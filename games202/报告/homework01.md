# games202-实验报告-01

## 任务0：实验环境
-   电脑品牌：惠普
-   电脑型号：暗影精灵8pro
-   CPU型号：12th Gen Intel(R) Core(TM) i7-12700H
    -   CPU核心（core）数量：4
    -   CPU默频/睿频：2.30 GHz
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
![输入图片说明](/imgs/2024-10-18/CMKTir5lJTY04oEM.png =550x400)

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
### 效果如下，
发现当data取值为1.0时可以有效解决锯齿问题
![输入图片说明](/imgs/2024-10-18/5Vl5GcL66CmPyUSJ.png =550x400)
![输入图片说明](/imgs/2024-10-18/xr9vvrtJ8CG7uctY.png =550x400)
## 任务2：PCF(Percentage Closer Filter)
1. 完善 phongFragment.glsl 中的 PCF(sampler2D shadowMap, vec4 shadowCoord, float filterSize) 函数
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
2. 使用main函数的第二行visibility
```
void main(void) {
  //归一化
  vec3 shadowCoord = vPositionFromLight.xyz / vPositionFromLight.w;
  //需要转化到NDC，才能在纹理uv坐标中使用
  shadowCoord.xyz=(shadowCoord.xyz + 1.0) / 2.0;

  float visibility;
  //visibility = useShadowMap(uShadowMap, vec4(shadowCoord, 1.0));
  visibility = PCF(uShadowMap, vec4(shadowCoord, 1.0));
  //visibility = PCSS(uShadowMap, vec4(shadowCoord, 1.0));

  vec3 phongColor = blinnPhong();

  gl_FragColor = vec4(phongColor * visibility, 1.0);
  //gl_FragColor = vec4(phongColor, 1.0);
}
```
### 运行如下（泊松采样）：
当`Stride = 10.0，NUM_SAMPLES = 20`时，噪点较多，因为此时泊松采样数较少，此情况会随着`NUM_SAMPLES`的增多而改善
![输入图片说明](/imgs/2024-10-18/hEni8o7AyCtoweQi.png =600x400)
当`Stride = 10.0，NUM_SAMPLES = 100`时，可以看到噪点明显变少很多
![输入图片说明](/imgs/2024-10-18/XSvAvGW8sIlOIMqZ.png =600x400)
当`Stride = 1.0，NUM_SAMPLES = 100`时，可以看到阴影的边缘变得清晰许多
![输入图片说明](/imgs/2024-10-18/ZLIG9E5XjEvCkrqc.png =600x400)
### 使用均匀圆盘采样：
`Stride = 10.0，NUM_SAMPLES = 100`
![输入图片说明](/imgs/2024-10-19/IjDLU5TxIASNJPr0.png =600x350)

### 综上
通过修改`Stride` 和`NUM_SAMPLES`参数
发现当`Stride`越小/大，阴影的边缘变得更清晰/模糊
当`NUM_SAMPLES`越小/大，噪点约多/少
泊松采样的效果会比均匀圆盘来的好

### 关于EPS数据的讨论
当`EPS = 0.001`时
![输入图片说明](/imgs/2024-10-19/dlDCu1X9zLQd7bBk.png =580x420)
当`EPS = 0.01`时
![输入图片说明](/imgs/2024-10-19/e8TxiSBCr3xVA7vN.png =600x400)
发现EPS主要作用变化在模型上的阴影
## 任务3：PCSS(Percentage Closer Soft Shadow)
1. 完善 phongFragment.glsl 中的 findBlocker(sampler2D shadowMap, vec2 uv, float zReceiver)
```
float findBlocker( sampler2D shadowMap,  vec2 uv, float zReceiver ) {
  // zReceiver就是当前coords的z值，PCSS调用findBlocker时会赋值
  int blockerNum = 0;
  float block_depth = 0.;
  float shadowmapSize = 2048.;
  float Stride = 50.;

  // 泊松采样得到采样点
  poissonDiskSamples(uv);

  // 判断是否是blocker并累加
  for(int i = 0; i < NUM_SAMPLES; i++){
    vec4 shadow_color = texture2D(shadowMap, uv + poissonDisk[i] * Stride / shadowmapSize);
    float shadow_depth = unpack(shadow_color);
    if(zReceiver > shadow_depth + 0.01){//在阴影内
      blockerNum++;
      block_depth += shadow_depth;
    }//else什么都不做
  }
  //在场景被光照到的地方，需要也给一个返回值，不然环境是全黑的
  if(blockerNum == 0){
    return 1.;
  }
  return float(block_depth) / float(blockerNum);
}
```
2. 完善PCSS(sampler2D shadowMap, vec4 shadowCoord) 函数
```
float PCSS(sampler2D shadowMap, vec4 coords){

  // STEP 1: avgblocker depth
  float d_Blocker = findBlocker(shadowMap, coords.xy, coords.z);
  float w_Light = 1.;
  float d_Receiver = coords.z;

  // STEP 2: penumbra size
  // 根据公式计算wpenumbra
  float w_penumbra = w_Light * (d_Receiver - d_Blocker) / d_Blocker;

  // STEP 3: filtering
  // PCF，只不过比PCF多了一重w_prenumbra加入了d_Receiber的影响
  float Stride = 20.;
  float shadowmapSize = 2048.;
  float visibility = 0.;
  float cur_depth = coords.z;

  //泊松采样已经在findBlocker中进行

  for(int i = 0; i < NUM_SAMPLES; i++){
    vec4 shadow_color = texture2D(shadowMap, coords.xy + poissonDisk[i] * Stride / shadowmapSize * w_penumbra);
    float shadow_depth = unpack(shadow_color);
    float res = cur_depth <= shadow_depth + 0.01 ? 1. : 0.;
    visibility += res;
  }
  
  return visibility / float(NUM_SAMPLES);
}
```
![输入图片说明](/imgs/2024-10-19/BFAW6FC1IAq6afUi.png =600x380)
## 任务4：Bonus
### 动态物体
1. engine.js中，在原有的transform基础上添加Rotate的参数
```
function setTransform(t_x, t_y, t_z, s_x, s_y, s_z, r_x, r_y, r_z) {
	return {
		...
		modelRotateX: r_x,
		modelRotateY: r_y,
		modelRotateZ: r_z,
	};
}
```
2. DirectionalLight.js中添加旋转的参数值
```
this.mesh = Mesh.cube(setTransform(0, 0, 0, 0.2, 0.2, 0.2, 0, 0, 0, 0));
```

```
CalcLightMVP(translate, scale, rotate) {
...
mat4.translate(modelMatrix, modelMatrix, translate);
mat4.scale(modelMatrix, modelMatrix, scale);
mat4.rotateX(modelMatrix, modelMatrix, rotate[0])
mat4.rotateY(modelMatrix, modelMatrix, rotate[1])
mat4.rotateZ(modelMatrix, modelMatrix, rotate[2])
...
```
3. 在PhongMaterial.js中加入旋转参数
```
constructor(color, specular, light, translate, scale, rotate, vertexShader, fragmentShader) {
	let lightMVP = light.CalcLightMVP(translate, scale, rotate);
	let lightIntensity = light.mat.GetIntensity();
...
}
async function buildPhongMaterial(color, specular, light, translate, scale, rotate, vertexPath, fragmentPath) {
	let vertexShader = await getShaderString(vertexPath);
	let fragmentShader = await getShaderString(fragmentPath);
	return new PhongMaterial(color, specular, light, translate, scale, rotate, vertexShader, fragmentShader);
}
```
4. 同理ShadowMaterial.js也需添加
```
constructor(light, translate, scale, rotate, vertexShader, fragmentShader) {
	let lightMVP = light.CalcLightMVP(translate, scale, rotate);
...
}
async function buildShadowMaterial(light, translate, scale, rotate, vertexPath, fragmentPath) {
	let vertexShader = await getShaderString(vertexPath);
	let fragmentShader = await getShaderString(fragmentPath);
	return new ShadowMaterial(light, translate, scale, rotate, vertexShader, fragmentShader);
}
```
5. 在Mesh.js中加入旋转参数
```
constructor(translate = [0, 0, 0], scale = [1, 1, 1], rotate = [0, 0, 0]) {
	this.translate = translate;
	this.scale = scale;
	this.rotate = rotate;
}

const modelTranslation = [transform.modelTransX, transform.modelTransY, transform.modelTransZ];
const modelScale = [transform.modelScaleX, transform.modelScaleY, transform.modelScaleZ];
const modelRatation = [transform.modelRotateX, transform.modelRotateY, transform.modelRotateZ];
let meshTrans = new TRSTransform(modelTranslation, modelScale, modelRatation);
this.transform = meshTrans;
```
6. MeshRender.js添加旋转
```
// Model transform
mat4.identity(modelMatrix);
mat4.translate(modelMatrix, modelMatrix, this.mesh.transform.translate);
mat4.scale(modelMatrix, modelMatrix, this.mesh.transform.scale);
mat4.rotateX(modelMatrix, modelMatrix, this.mesh.transform.rotate[0])
mat4.rotateY(modelMatrix, modelMatrix, this.mesh.transform.rotate[1])
mat4.rotateZ(modelMatrix, modelMatrix, this.mesh.transform.rotate[2])
```
7. loadOBJ.js中添加旋转参数
```
let material, shadowMaterial;
let Translation = [transform.modelTransX, transform.modelTransY, transform.modelTransZ];
let Scale = [transform.modelScaleX, transform.modelScaleY, transform.modelScaleZ];
let Rotation = [transform.modelRotateX, transform.modelRotateY, transform.modelRotateZ];
let light = renderer.lights[0].entity;
switch (objMaterial) {
	case 'PhongMaterial':
	material = buildPhongMaterial(colorMap, mat.specular.toArray(), light, Translation, Scale, Rotation, "./src/shaders/phongShader/phongVertex.glsl", "./src/shaders/phongShader/phongFragment.glsl");
	shadowMaterial = buildShadowMaterial(light, Translation, Scale, Rotation, "./src/shaders/shadowShader/shadowVertex.glsl", "./src/shaders/shadowShader/shadowFragment.glsl");
	break;
}
```
8. 以上我们准备好了支持旋转的模型变换需要修改的部分，去engine.js中把Rotation参数补齐
```
let floorTransform = setTransform(0, 0, -30, 4, 4, 4, 0, 0, 0);
let obj1Transform = setTransform(0, 0, 0, 20, 20, 20, 0, 0, 0);
let obj2Transform = setTransform(40, 0, -40, 10, 10, 10, 0, 0, 0);
```
且为了得到时间的概念，我们需要在渲染主循环中进行时间的获取
```
let prevTime = 0;
function mainLoop(now) {
cameraControls.update();
let deltaime = (now - prevTime) / 1000;
renderer.render(now, deltaime);
requestAnimationFrame(mainLoop);
prevTime = now;
}
requestAnimationFrame(mainLoop);
```
9. 因为Render需要参数，所以请WebGLRender.js中添加接收时间的参数
``` 
render(time, deltaime) { }
```
10. 在engine.js中添加一个方法用来实现角度转弧度制，因为控制旋转时接受的时弧度参数，我们希望使用角度
```
//角度转弧度
function degrees2Radians(degrees){
	return 3.1415927 / 180 * degrees;
}
```
11. WebGLRender.js中加入旋转循环，此处针对顶点数大于8的mesh进行旋转，是因为地面的顶点数为6，就可以做到只旋转人物而不旋转地面.
`degrees2Radians(10)`此函数是我们刚刚做的角度转弧度制，在此处输入我们希望旋转的角度即可
```
for (let i = 0; i < this.meshes.length; i++) {
	if(this.meshes[i].mesh.count > 8){
		this.meshes[i].mesh.transform.rotate[1] = this.meshes[i].mesh.transform.rotate[1] + degrees2Radians(10) * deltaime;
	}
}
```
![输入图片说明](/imgs/2024-10-26/GBzcKFsztW8oRCTO.png =600x380)
但是此处我们会发现阴影并没有跟着人物模型进行旋转，是因为没有更新lightMVP矩阵，导致物体变动并没有反映在ShadowMap上。
12. 更新lightMVP
先在WebGLRenderer.js进行shadowMap的清除工作。然后在两个Pass（Shadow pass/ Camera pass）中进行lightMVP矩阵的更新
```
for (let l = 0; l < this.lights.length; l++) {
	gl.bindFramebuffer(gl.FRAMEBUFFER, this.lights[l].entity.fbo); // 绑定到当前光源的framebuffer
	gl.clearColor(1.0, 1.0, 1.0, 1.0); // shadowmap默认白色（无遮挡，解决地面边缘产生阴影的问题
	gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT); // 清除shadowmap上一帧的颜色、深度缓存，否则会一直叠加每一帧的结果

// Shadow pass
if (this.lights[l].entity.hasShadowMap == true) {
	for (let i = 0; i < this.shadowMeshes.length; i++) {
		//每帧更新shader中uniforms的LightMVP
		this.gl.useProgram(this.shadowMeshes[i].shader.program.glShaderProgram);
		let translation = this.shadowMeshes[i].mesh.transform.translate;
		let scale = this.shadowMeshes[i].mesh.transform.scale;
		let rotation = this.shadowMeshes[i].mesh.transform.rotate;
		let lightMVP = this.lights[l].entity.CalcLightMVP(translation, scale, rotation);
		this.shadowMeshes[i].material.uniforms.uLightMVP = { type: 'matrix4fv', 		value: lightMVP };
		this.shadowMeshes[i].draw(this.camera);
	}
}

// Camera pass
for (let i = 0; i < this.meshes.length; i++) {
	this.gl.useProgram(this.meshes[i].shader.program.glShaderProgram);
	// 每帧更新shader中uniforms参数
	// this.gl.uniform3fv(this.meshes[i].shader.program.uniforms.uLightPos, 	this.lights[l].entity.lightPos); //这里改用下面写法
	let translation = this.meshes[i].mesh.transform.translate;
	let rotation = this.meshes[i].mesh.transform.rotate;
	let scale = this.meshes[i].mesh.transform.scale;
	let lightMVP = this.lights[l].entity.CalcLightMVP(translation, scale, rotation);
	this.meshes[i].material.uniforms.uLightMVP = { type: 'matrix4fv', value: lightMVP };
	this.meshes[i].material.uniforms.uLightPos = { type: '3fv', value: 		this.lights[l].entity.lightPos }; // 光源方向计算、光源强度衰减
	this.meshes[i].draw(this.camera);
}
```
![输入图片说明](/imgs/2024-10-26/WlJU0GnBAbTtyTYs.png =600x380)
### 多光源
在Metrial.js中添加一个lightIndex用来区分不同光源
```
constructor(uniforms, attribs, vsSrc, fsSrc, frameBuffer, lightIndex) {
	...
	this.lightIndex = lightIndex;
}
```
ShadowMaterial.js中添加lightIndex参数
```
class ShadowMaterial extends Material {
	constructor(light, translate, scale, rotate, lightIndex, vertexShader, fragmentShader) {
		let lightMVP = light.CalcLightMVP(translate, scale, rotate);
		super({
			'uLightMVP': { type: 'matrix4fv', value: lightMVP }
		}, [], vertexShader, fragmentShader, light.fbo, lightIndex);
	}
}
async function buildShadowMaterial(light, translate, scale, rotate, lightIndex, vertexPath, fragmentPath) {
	...
	return new ShadowMaterial(light, translate, scale, rotate, lightIndex, vertexShader, fragmentShader);
}
```
同样PhongMaterial中添加lightIndex参数
```
constructor(color, specular, light, translate, scale, rotate, lightIndex, vertexShader, fragmentShader) {
	let lightMVP = light.CalcLightMVP(translate, scale, rotate);
	let lightIntensity = light.mat.GetIntensity();
	super({
		...
		// Shadow
		'uShadowMap': { type: 'texture', value: light.fbo },
		'uLightMVP': { type: 'matrix4fv', value: lightMVP },
	}, [], vertexShader, fragmentShader, null, lightIndex);
}
async function buildPhongMaterial(color, specular, light, translate, scale, rotate, lightIndex, vertexPath, fragmentPath) {
	...
	return new PhongMaterial(color, specular, light, translate, scale, rotate, vertexShader, fragmentShader, lightIndex);
}
```
loadOBJ.js中添加参数lightIndex
```
//let light = renderer.lights[0].entity;
//原本只添加第一个light的材质，改成添加所有light的材质，并添加旋转参数
for(let i = 0; i < renderer.lights.length; i++){
	let light = renderer.lights[i].entity;
	switch (objMaterial) {
		case 'PhongMaterial':
		//添加旋转参数、光源索引参数
	material = buildPhongMaterial(colorMap, mat.specular.toArray(), light, Translation, Scale, Rotation, i, "./src/shaders/phongShader/phongVertex.glsl", "./src/shaders/phongShader/phongFragment.glsl");
	shadowMaterial = buildShadowMaterial(light, Translation, Scale, Rotation, i, "./src/shaders/shadowShader/shadowVertex.glsl", "./src/shaders/shadowShader/shadowFragment.glsl");
		break;
	}
	...
}
```
以上即为支持多光源的参数支持，将原本的一个光源改为通过for循环添加多个，接下来我们只需往上添加光源
```
## 实验总结

-   请简述实验的心得体会。欢迎对实验形式、内容提出意见和建议。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0MjQzNTA2OTQsMjAzNjk1NDMwMCwtMT
kyOTk4MTEyNywxMzg4MTk0NzU3LC00NzExODI3ODksLTY3MzUz
MDY0OSwxMzQ3ODE3Nzk2LC01MDkwMTcwNjgsMTI3NTA3MzEyOS
wxMjkxMDY1MjQ5LDYzNTYxMzI2MywtMTQwOTg1MzY1MywxODgx
NjMxOTkxLC0xNzUwMzYzODM0LC0xOTU4MDQ5MTg3LC0xNjkxMj
cwNzI1LDExMTU1MzA3MDYsLTIxMTQ4MTgyMzksMjU4NDEzMDk2
LC0xMTQ4NDEzNzcwXX0=
-->