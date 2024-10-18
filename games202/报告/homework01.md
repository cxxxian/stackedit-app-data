# games202-实验报告-01

## 任务0：实验环境
-   电脑品牌：惠普
-   电脑型号：暗影精灵8pro
-   CPU型号：12th Gen Intel(R) Core(TM) i7-12700H
    -   CPU核心（core）数量：4
    -   CPU默频/睿频：2.30 GHz
-   内存规格：// 例如：DDR2-400
-   内存容量：512G
-   存储类型：SSD
-   显卡型号：Nvidia GeForce RTX 3060
-   屏幕分辨率：1920*1080
-   最大功率：280W
-   操作系统：Windows11

## 任务1：Shadow Map

 1. 完善DirectionalLight中的CalcLightMVP(translate, scale)函数
```
CalcLightMVP(translate, scale) {
let lightMVP = mat4.create();
let modelMatrix = mat4.create();
let viewMatrix = mat4.create();
let projectionMatrix = mat4.create();
// Model transform
//translate(输出的矩阵，处理的矩阵，vec3矩阵)
mat4.translate(modelMatrix, modelMatrix, translate);
mat4.scale(modelMatrix, modelMatrix, scale);
// View transform
//参数在engine定义
//let lightPos = [0, 80, 80];
//let focalPoint = [0, 0, 0];
//let lightUp = [0, 1, 0]
//focalPoint是lookat的方向
mat4.lookAt(viewMatrix, this.lightPos, this.focalPoint, this.lightUp);
// Projection transform
//n 是近裁剪面，f 是远裁剪面
var r = 100;
var l = -100;
var t = 100;
var b = -100;
var n = 1e-2;
var f = 400;
mat4.ortho(projectionMatrix, l, r, b, t, n, f);
mat4.multiply(lightMVP, projectionMatrix, viewMatrix);
mat4.multiply(lightMVP, lightMVP, modelMatrix);
return lightMVP;
}
```
2. 完善phongFragment.glsl中的 useShadowMap(sampler2D shadowMap, vec4 shadowCoord) 函数
```
//shadowMap: 提取来自方向光中创建的FBO中的深度信息
//shadowCoord: 纹理图片上像素对应的坐标，main()中有对应的归一化坐标计算
float useShadowMap(sampler2D shadowMap, vec4 shadowCoord){
  //查询纹理图片对应坐标上的深度值，而实现深度值查询首先要查对应的颜色
  //获取颜色值
  //第一个参数: 图片纹理
  //第二个参数: 纹理坐标
  vec4 shadow_color = texture2D(shadowMap, shadowCoord.xy);
  //将RGBA值转化成float
  float shadow_depth = unpack(shadow_color);
  float cur_depth = shadowCoord.z;
  if(cur_depth >= shadow_depth + EPS){
    return 0.;//不可视
  }else{
    return 1.0;
  }
}
```
3. 完善main函数，将像素坐标归一化。
```
void main(void) {
  //归一化
  vec3 shadowCoord = vPositionFromLight.xyz / vPositionFromLight.w;
  //需要转化到NDC，才能在纹理uv坐标中使用
  shadowCoord.xyz=(shadowCoord.xyz + 1.0) / 2.0;

  float visibility;
  visibility = useShadowMap(uShadowMap, vec4(shadowCoord, 1.0));
  //visibility = PCF(uShadowMap, vec4(shadowCoord, 1.0));
  //visibility = PCSS(uShadowMap, vec4(shadowCoord, 1.0));

  vec3 phongColor = blinnPhong();

  gl_FragColor = vec4(phongColor * visibility, 1.0);
  //gl_FragColor = vec4(phongColor, 1.0);
}
```
此时会存在自遮挡导致锯齿的问题，效果如下
![输入图片说明](/imgs/2024-10-18/CMKTir5lJTY04oEM.png)

4. 解决锯齿问题
引入bias的概念，利用函数getBias得到合适的Bias值，在`if(cur_depth - bias >= shadow_depth + EPS)`判断中加入bias，可以有效解决锯齿问题
```
//  1 - dot(normal, lightDir)用来近似tan和sin
float getBias(float data){
  vec3 lightDir = normalize(uLightPos - vFragPos);
  vec3 normal = normalize(vNormal);
  float m = 200. / 2048. / 2.;
  float bias = max(m , m * (1. - dot(normal, lightDir))) * data;
  return bias;
}

//shadowMap: 提取来自方向光中创建的FBO中的深度信息
//shadowCoord: 纹理图片上像素对应的坐标，main()中有对应的归一化坐标计算
float useShadowMap(sampler2D shadowMap, vec4 shadowCoord){

  //查询纹理图片对应坐标上的深度值，而实现深度值查询首先要查对应的颜色
  //获取颜色值
  //第一个参数: 图片纹理
  //第二个参数: 纹理坐标
  vec4 shadow_color = texture2D(shadowMap, shadowCoord.xy);
  //将RGBA值转化成float
  float shadow_depth = unpack(shadow_color);

  float cur_depth = shadowCoord.z;

  float data = 1.;
  float bias = getBias(data);
  if(cur_depth - bias >= shadow_depth + EPS){
    return 0.;//不可视
  }else{
    return 1.0;
  }
}
```
效果如下，发现当data取值为1.0时可以有效解决锯齿问题
![输入图片说明](/imgs/2024-10-18/5Vl5GcL66CmPyUSJ.png)
![输入图片说明](/imgs/2024-10-18/xr9vvrtJ8CG7uctY.png)
## 任务2：PCF(Percentage Closer Filter)
完善 phongFragment.glsl 中的 PCF(sampler2D shadowMap, vec4 shadowCoord, float filterSize) 函数
```
float PCF(sampler2D shadowMap, vec4 coords) {
  // 1 给定步长，分辨率，初始输出值，卷积范围当前的深度
  float Stride = 10.;
  float shadowmapSize = 2048.;
  float visibility = 0.;
  float cur_depth = coords.z;
  
  //2 泊松采样得到采样点
  //coords.xy 是从光源空间转换到阴影贴图空间的 2D 坐标，决定了在阴影贴图上采样的位置
  poissonDiskSamples(coords.xy);
  
  //3 对每个点进行比较深度值并累加
  for(int i = 0; i < NUM_SAMPLES; i++){
    //Stride / shadowmapSize 来对泊松采样的偏移进行归一化
    vec4 shadow_color = texture2D(shadowMap, coords.xy + poissonDisk[i] * Stride / shadowmapSize);
    float shadow_depth = unpack(shadow_color);
    float res = cur_depth < shadow_depth + EPS ? 1. : 0.;
    visibility += res;
  }
  return visibility / float(NUM_SAMPLES);
}
```
```
将像素坐标归一化了
```
## 任务3：修正程序（Fixme）

## 实验总结

-   请简述实验的心得体会。欢迎对实验形式、内容提出意见和建议。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMjY2NDk2NF19
-->