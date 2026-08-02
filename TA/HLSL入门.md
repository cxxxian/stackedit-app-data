# 广告牌切换效果

## 蓝图实现

![输入图片说明](/imgs/2026-07-12/WzZYFToRG5cs8eFR.png)

### 简单一句话区分
**TexCoord** = 
1. 拿模型自带的 UV 坐标（附带横竖基础渐变），用来贴图片
2. 数据来源读取模型预烘焙 UV
3. 不依赖模型 UV输出类型二维向量 (U,V)

**LinearGradient** = 
1. 凭空生成一条可自由调角度、长短、位置的黑白渐变，专门做渐变光、遮罩、进度条
2. 依赖模型拓扑纯数学程序化生成
3. 同时包含横竖两套渐变一维标量，单方向线性渐变

![输入图片说明](/imgs/2026-07-12/fhjTOWpFqpl9ELxM.png)

利用`TexCoord * 10 * ceil`，可以讲原本`0-1`的`uv`转成`0-10`，然后用`ceil`之后就能实现上面这种画格子的效果，
但是因为`TexCoord * 10`的原因，整个颜色输出会变亮，所以我们在`ceil`之后要手动`divide`除以`10`
因为`uv`是`RG`两个数值，横向`R`（红色），纵向（绿色）`G`，所以我们利用**BreakOutFloat2Components**节点，可以拆分出两个通道，输出`R`的话就可以实现竖纹的效果，因为`R`代表`u`通道，如下

![输入图片说明](/imgs/2026-07-12/IcwmpUMNWHlVKZ48.png)

然后我们减去一个颜色值就可以得到这种效果了
但要想实现左右动态移动的效果，我们要引入一个`time`和`sin`正弦函数，因为`sin`值是`-1~1`，所以我们把原本的`R`通道值减去这个`sin`值，就可以得到一种往返黑白渐变的效果
可以看到原本最左边其实不是白色的，但是因为会有机会减去`-1`，所以他也能做到变成纯白色
这样做法会存在阶梯，所以我们可以利用先前学到的`step`节点，如下图，就可以做到只有纯黑纯白的切换了，因为低于`0.5`会变成黑色，高于`0.5`会变成白色，没有中间的灰度了

![输入图片说明](/imgs/2026-07-12/wMUbmKm3k5wbGKil.png)

如果希望更丝滑，就把刚刚的`*10`（乘大`uv`值和`divide 10`同步调整）拉大，如果不想丝滑那就拉小，拉到`2`的时候就会变成只有两格在移动变色
如果想调整来回滑动的速度，可以进入`sin`节点调整周期（`Period`）值，越大就越慢

![输入图片说明](/imgs/2026-07-12/HvXXE8VYuRXXlFpY.png)

我们刚刚做得所有计算，因为是灰度值，所以直接连接到`lerp`的`alpha`值，然后做两个输入即可进行自定义切换

## 程序实现
```hlsl
return (step(ceil(uv * 10) / 10 - sin(t), 0.5));
```
其实就是照着蓝图可以从内往外倒推，使用`custom`节点换上自己的代码，要自己调整输出类型以及输入的参数，然后把`TexCoord`和`time`连接到输入参数即可

![输入图片说明](/imgs/2026-07-12/6D4msmFWoHQ6eFiA.png)

# 画个圆
## 蓝图实现
### 方程做法
`X^2 + Y^2 = R^2`
其实就是满足圆形方程

![输入图片说明](/imgs/2026-07-12/Ml7icAjLcTyPA2kc.png)

实现出来长这样，但是会发现这就只是一个`1/4`个圆形，是因为`TexCoord`的`uv`左上角是`(0,0)`，右下角是`(1,1)`，所以会造成圆心在左上角的错误

所以一般我们的做法就直接`-0.5 * 2`（也不一定要`*2`，反正中心是`(0,0)`就行了）就能把`uv`映射到`-1~1`了，然后中心是`(0,0)`，但是这个`pow`节点，底数必须得是正数，负数的话会被按照`0`处理
`pow`相关文档说明如下：
Returns the Base value raised to the powerof Exponent. Base value must be positive,values less than 0 will be clamped.
所以解决办法也很简单，不用就好了，我们直接用`multiply`节点，输入的`A,B`都是自身就能得到`X^2, Y^2`的效果

### 向量做法
这个做法利用了向量，我们用一个二维常数值`(0.5,0.5)`减去`TexCoord`，然后去算二维向量的`length`并做二次幂，其实得出来的也是圆形
并且把`exp`值和左上角的二维常数改成`time`控制的`sin`函数之后，圆形就会对应进行收缩放大和左上角到右下角的周期变化运动

![输入图片说明](/imgs/2026-07-12/pkpzAzbG8rueCDyk.png)

## 程序做法
这样就行，很简单。
两个输入分别是`uv`，`pos`，对应`TexCoord`和我们自定义的一个二维常数输入，用来控制圆心位置
```hlsl
float d = length(uv - pos);
return d;
```
加上一个参数，就可以控制圆的半径，只有在`radius`之内才是`1`
```hlsl
float d = length(uv - pos);
return d <= radius;
```

![输入图片说明](/imgs/2026-07-12/ZDzEL3llDPs7m4F5.png)

现在我们希望做很多个圆，可以利用`frac`函数（蓝图同名）
我们将`uv`扩大`gridSize`倍数，然后用`frac`处理`uv`
`frac`定义如下：`Frac(X)` = **取小数部分**，舍去整数部分，只保留小数点后的数值
`Frac(x) = x - Floor(x);`
输出范围永远固定：`[0, 1)`，最大值无限接近 `1`，不会等于 `1`。
所以代码就变成了：
```hlsl
float d = length(frac(uv * gridSize) - pos);
return d <= radius;
```
效果如下

![输入图片说明](/imgs/2026-07-12/rq0IYllG4HvyXzvw.png)

# 砖块效果
利用`uv`实现，这样即可实现`grid.x * grid.y`个数的砖墙
```hlsl
float2 tex = frac(float2(uv.x * grid.x, uv.y * grid.y));
return tex;
```
注意这里`return tex`，所以对应虚幻输出节点要调整为`float2`
要实现自定义砖块样式，只需要做一个输出的判断即可
```hlsl
float2 tex = frac(float2(uv.x * grid.x, uv.y * grid.y));
if(tex.x >= dim.x || tex.x <= -dim.x || tex.y >= dim.y || tex.y <= -dim.y){
    return (0);
}
return (1);
```
但其实这里`tex.x <= -dim.x`小于一个负数没用，不知道为什么教程这样写
然后我们将输出改为`float3`，这样子就可以输出黑白了

![输入图片说明](/imgs/2026-07-19/wyLhRtrYckJ5j6GU.png)

现在会发现每一个面的线对应不上
加上一个`offset`，值为`dim / 2`
```hlsl
float2 tex = frac(float2(uv.x * grid.x + offset.x, uv.y * grid.y + offset.x));
if(tex.x >= dim.x || tex.x <= -dim.x || tex.y >= dim.y || tex.y <= -dim.y){
    return (colMortar);
}
return (colBrick);
```
这样子就能实现左上角向下向右偏移`dim / 2`个单位，得到砖缝
效果如图

![输入图片说明](/imgs/2026-07-19/CBSEHxEqpXuNblKB.png)

如果我们希望作为蒙版的话，也可以将混合模式改成遮罩，然后输出连到`opacityMask`（不透明蒙版），可以利用蒙版作为`alpha`做更多效果

# 圆形动画
先简单画个圆
```hlsl
float result;
result = length(pos - uv) < size;
return result;
```
优化一下，能做出`nSides`个圆围着中心的效果
-   `float2(cosθ, sinθ)`：固定长度 1，只记录朝向；
-   乘以标量 radius：把向量整体拉长到 radius，朝向 / 角度保持不变；
-   最后 `+ center` 只是把整个点平移到画布中心，不改变长度和角度。
```hlsl
float result = 0;
for(int i = 0; i < nSides; i++){
	float angle = (i / nSides) * 2 * 3.14;
	float2 pos = center + radius * float2(cos(angle), sin(angle));
	result += length(pos - uv) < size;
}
return saturate(result);
```
就像这样：

![输入图片说明](/imgs/2026-07-19/uUUhyzCftGLKPytS.png)

但是这是静止的，我们想让它动起来
原本是这样，意思是整个圆，那如果`*1`就是半圆，`*0.5`就是`1/4`圆，所以我们把这个值改成`time`，就可以让它动起来了
`float angle = (i / nSides) * 2 * 3.14;`
```hlsl
float result = 0;
for(int i = 0; i < nSides; i++){
    float angle = (i / nSides) * time * 3.14;
    float2 pos = center + radius * float2(cos(angle), sin(angle));
    result += length(pos - uv) < size;
}
return saturate(result);
```
然后我希望向外层扩散
可以利用第二层`for`，取`(j / nCopies) * radius`，就可以对半径做均分取值绘制
```hlsl
float result = 0;
for(int i = 0; i < nSides; i++){
    for(int j = 0; j < nCopies; j++){
        float angle = (i / nSides) * time * 3.14;
        float2 pos = center + (j / nCopies) * radius * float2(cos(angle), sin(angle));
        result += length(pos - uv) < size;
    }
}
return saturate(result);
```

![输入图片说明](/imgs/2026-07-19/2MJ7iqraZ3OCEa7E.png)

如果想做出一种`3d`的感觉的话可以操作正余弦
例如：`float2 pos = center + (j / nCopies) * radius * float2(cos(1 - angle), sin(3 * angle));`

# 圆形动画2.0
做出类似于加载圆形进度条的感觉，重点就是要把刚刚的`time`用`sin(time)`，控制在`-1~1`，要不然`time`增大之后`angle`的速度就会原来越大不是均分分布
```hlsl
float result = 0;
for(int i = 0; i < nSides; i++){
    for(int j = 0; j < nCopies; j++){
        float angle = (i / nSides) * sin(time) * 2 * 3.14;
        float2 pos = center + (j / nCopies) * radius * float2(cos(angle), sin(angle));
        result += length(pos - uv) < size;
    }
}
return result;
```
这样是逆时针的，如果改成`float2(cos(1 - angle), sin(1 - angle))`初始就会是顺时针
加入一个`outEmissive`并且开启遮罩我们就可以做出颜色变化
```hlsl
float result = 0;
for(int i = 0; i < nSides; i++){
    for(int j = 0; j < nCopies; j++){
        float angle = (i / nSides) * sin(time) * 2 * 3.14;
        float2 pos = center + (j / nCopies) * radius * float2(cos(angle), sin(angle));
        result += length(pos - uv) < size;
    }
}
outEmissive = float3(sin(time), 0, 0.1);
return result;
```

![输入图片说明](/imgs/2026-07-19/s4XKdis71In6sJDY.png)

这里解释一下遮罩区别
### 区别 1：像素生存逻辑（你说的裁剪）
-   **BaseColor（基础颜色）**
    永远保留全部像素，不会删除任何区域：
    val=0 → 纯黑色像素；val=1 → 白色像素；整张面片完整存在，挡住后方物体。
-   **Opacity Mask（不透明蒙版）**
    硬裁剪丢弃像素，阈值默认 0.5：
    val ≥ 0.5：像素保留正常渲染；
    val ＜ 0.5：像素直接**彻底删除、镂空透明**，能看到模型背后的东西。
### 区别 2：通道职能完全不一样
-   BaseColor：**控制像素本身的颜色**，决定这个点是什么颜色；
-   Opacity Mask：**只控制像素要不要存在**，完全不影响颜色，颜色由自发光 / 基础色单独控制（你图里颜色走的是 outEmissive）。

如果我们想让它发光，可以注意到我们`result`专门没做`saturate`
所以直接把`result`乘上
```hlsl
float result = 0;
for(int i = 0; i < nSides; i++){
    for(int j = 0; j < nCopies; j++){
        float angle = (i / nSides) * sin(time) * 2 * 3.14;
        float2 pos = center + (j / nCopies) * radius * float2(cos(angle), sin(angle));
        result += length(pos - uv) < size;
    }
}
outEmissive = result * float3(sin(time), 0, 0.1);
return result;
```
这样越中心的位置会越亮，因为越中心圆形重叠的个数越多
![输入图片说明](/imgs/2026-07-19/1HVJpjjIgqgvNrZk.png)

# rayMatching
## 画圆
还是从画一个圆开始
很普通的光线步进逻辑
```hlsl
float3 ro = camWorldPos;
float3 rd = normalize(worldPos - camWorldPos);
float3 p = ro;

for(int i = 0; i < 512; i++){
    float dist = length(p - sphereCenter) - sphereRadius;
    if(dist < 0.01){
        return float3(1, 0, 0);
    }
    p += rd;
}
return float3(0, 0, 0);
```
但是这样，`sphereCenter`我们在外面显式写位置`(0, 0, 0)`的话，我们会造成在场景中拖动物体的时候，里面的球位置不会变，还是在`(0, 0, 0)`
就像这样：

![输入图片说明](/imgs/2026-07-25/1KwjQSV2ToySVg1s.png)

把`ActorPosition`作为`sphereCenter`传入，即可解决问题。

做一个输出`opacityMask`，并且把混合模式改成已遮罩，就会只显示圆了
```hlsl
float3 ro = camWorldPos;
float3 rd = normalize(worldPos - camWorldPos);
float3 p = ro;

for(int i = 0; i < 512; i++){
    float dist = length(p - sphereCenter) - sphereRadius;
    if(dist < 0.01){
        opacityMask = 1;
        return float3(1, 0, 0);
    }
    p += rd;
}
opacityMask = 0;
return float3(0, 0, 0);
```
## Diffuse
`p`是我们真实步进的点，所以`p - sphereCenter`是法线方向
以下代码就可以做出`diffuse`效果了
```hlsl
float3 ro = camWorldPos;
float3 rd = normalize(worldPos - camWorldPos);
float3 p = ro;
float3 lightDirection = normalize(lightPos);

for(int i = 0; i < 512; i++){
    float dist = length(p - sphereCenter) - sphereRadius;
    if(dist < 0.01){
        float3 normal = normalize(p - sphereCenter);
        float diffuse = max(dot(normal, lightDirection), 0);
        opacityMask = 1;
        return diffuse * float3(1, 0, 0);
    }
    p += rd;
}
opacityMask = 0;
return float3(0, 0, 0);
```
## Speclar
很基础的高光逻辑
```hlsl
float3 ro = camWorldPos;
float3 rd = normalize(worldPos - camWorldPos);
float3 p = ro;
float3 lightDirection = normalize(lightPos);

for(int i = 0; i < 512; i++){
    float dist = length(p - sphereCenter) - sphereRadius;
    if(dist < 0.01){
        float3 normal = normalize(p - sphereCenter);
        float diffuse = max(dot(normal, lightDirection), 0);
        float3 reflection = reflect(lightDirection, normal);
        float3 viewDirection = normalize(p - camWorldPos);
        float specular = pow(max(dot(reflection, viewDirection), 0), 16);
        opacityMask = 1;
        return (diffuse * float3(1, 0, 0)) + (specular * float3(1, 1, 1));
    }
    p += rd;
}
opacityMask = 0;
return float3(0, 0, 0);
```
## 动画
`displace`作为新圆心，然后`+ float3( sin(p.x * sin(time)/3), sin(p.y * sin(time)/3), sin(p.z * sin(time)/3) )`加的这个东西类似于一个`offset`
至于为什么是乘法，因为：
正弦函数通用形式：
`y = sin(A * x)`
A：**频率系数**
A 越大 → 同样 x 区间内，正弦震荡越多，波纹越密；
A 越小 → 波纹越稀疏。
`y = sin(A + x)`
这是**相位偏移**！
只会让波纹整体左右平移，**波纹疏密完全不变**。
```hlsl
float3 ro = camWorldPos;
float3 rd = normalize(worldPos - camWorldPos);
float3 p = ro;
float3 lightDirection = normalize(lightPos);

for(int i = 0; i < 512; i++){
    float3 displace = sphereCenter 
    + float3( sin(p.x * sin(time)/3),
              sin(p.y * sin(time)/3),
              sin(p.z * sin(time)/3) );

    float dist = length(p - displace) - sphereRadius;
    if(dist < 0.01){
        float3 normal = normalize(p - sphereCenter);
        float diffuse = max(dot(normal, lightDirection), 0);
        float3 reflection = reflect(lightDirection, normal);
        float3 viewDirection = normalize(p - camWorldPos);
        float specular = pow(max(dot(reflection, viewDirection), 0), 16);
        opacityMask = 1;
        return (diffuse * float3(1, 0, 0)) + (specular * float3(1, 1, 1));
    }
    p += rd;
}
opacityMask = 0;
return float3(0, 0, 0);
```
## 甜甜圈
重点是`struct`的用法
以及甜甜圈的画法，法线先置为`0`不管
`length(p.xz)`是水平面上距离原点`(0, 0)`的长度，然后`- size`之后我们就能得到一个距离甜甜圈圆环的距离，在圆环内就是负数，圆环外就是正数，`p.y`不做处理，这个是朝着我们的方向，单纯这玩意的距离那就是一根线
然后再减去`cutout`就有体积了，`cutout`就是甜甜圈这个环的宽度
```hlsl
float3 ro = camWorldPos;
float3 rd = normalize(worldPos - camWorldPos);
float3 p = ro;
float3 lightDirection = normalize(lightPos);

struct sdfShapes{
    float donut(float3 p, float size, float cutout){
        float2 q = float2(length(p.xz) - size, p.y);
        return length(q) - cutout;
    }
};
sdfShapes sdf;

for(int i = 0; i < 512; i++){

    float dist = sdf.donut(p, 50, 25);

    if(dist < 0.01){
        //float3 normal = normalize(p - displace);
        float3 normal = 0;
        float diffuse = max(dot(normal, lightDirection), 0);
        float3 reflection = reflect(lightDirection, normal);
        float3 viewDirection = normalize(p - camWorldPos);
        float specular = pow(max(dot(reflection, viewDirection), 0), 16);
        opacityMask = 1;
        return (diffuse * float3(1, 0, 0)) + (specular * float3(1, 1, 1));
    }
    p += rd;
}
opacityMask = 0;
return float3(0, 0, 0);
```
补上法线实现
`eps` = 微小偏移步长。
不能太大：太大采样点跑到曲面外面；
不能太小：GPU 浮点精度不足，推荐 `0.001`。

### 数学概念：梯度 Gradient
SDF 函数 `S(p)`，输入空间点 p，输出到物体表面距离。
梯度 `∇S` 就是法线方向：
-   梯度指向：**距离场增长最快的方向**
-   永远垂直几何体表面、向外
我们没有解析求导公式，用**中心有限差分**近似导数。
一维导数近似公式：

![输入图片说明](/imgs/2026-07-25/PCWyMbAgIMZdKDdH.png)

分母`2ε`只是统一缩放，最后会被 normalize 归一化抵消，代码里直接省略分母。
```hlsl
float eps = 0.001;
float3 normal = normalize(float3(
	// X轴正向偏移 - X轴负向偏移
	sdf.donut(p + float3(eps, 0, 0), 50, 25) - sdf.donut(p - 	float3(eps, 0, 0), 50, 25),
	// Y轴正向偏移 - Y轴负向偏移
	sdf.donut(p + float3(0, eps, 0), 50, 25) - sdf.donut(p - float3(0, eps, 0), 50, 25),
	// Z轴正向偏移 - Z轴负向偏移
	sdf.donut(p + float3(0, 0, eps), 50, 25) - sdf.donut(p - float3(0, 0, eps), 50, 25)
));
```
# raymarch深度贴图
利用步进去求颜色
当遇到满足`inputTex.r > 0.1 && inputTex.g > 0.1 && inputTex.b > 0.1`的时候我们就可以输出颜色
```glsl
float3 rayStep = viewDir * -1;
float4 inputTex = Texture2DSample(texObject, texObjectSampler, uv);

for(int i = 0; i < 50; i++){
    if(inputTex.r > 0.1 && inputTex.g > 0.1 && inputTex.b > 0.1){
        return float3(i, 0, 0);
    }

    uv += rayStep * 0.15;
    inputTex = Texture2DSample(texObject, texObjectSampler, uv.xy);
}

return inputTex;
```
确实有点抽象，对照图片理解会稍微好点，会发现原本白色的地方黑了，黑的地方红了，因为返回的是`(i, 0, 0)`，就是不同的红
意思差不多就是黑色的`uv`位置会随着`rayStep`的方向去找，当找到白色的时候，就说明可以`return`一个颜色回来作为当前`uv`的颜色值了，所以越远的地方会发现越亮，越近越暗
然后`i < 50`，`i`越大就越深，`rayStep * 0.15`，`0.15`越小层之间就越紧密

![输入图片说明](/imgs/2026-07-28/2OlyxMO3SirwdSI8.png)

# 混合
这个很简单就不做了
简单来说就是一张混凝土贴图，然后一个浅灰色的`rgb`
利用一张蒙版贴图，`RGB`三通道有各自的颜色，把三通道分别作为`alpha`输入进行混合得到结果
`custom`里面的对应代码如下
```hlsl
float result = (Pick == 1.0)?In1
							 (Pick == 2.0)?In2
							 (Pick==3.0)?In3 :float(0,0,0);
return result;
```

![输入图片说明](/imgs/2026-07-28/htonXBoIjI2Q3n1g.png)

然后利用这个材质我们可以做一个`instance`实例，这样就好像一个对象一样，右侧就是我们在材质里面写的参数

![输入图片说明](/imgs/2026-07-28/LqGNknoKMmYAVU0V.png)

# 数学变换
`uvScale`：是`uv`变换，`(uv - 0.5)`可以把`(0, 0)`挪到中心点，基于中心点进行变换，然后`+ 0.5`再把中心点挪回来，这里返回的`uv`就已经不在`0~1`了，我们会有其他策略例如`Repeat, Clamp, Mirror`
`texRotation`：旋转贴图，注意点是把弧度制传入做正余弦`radians(45)`

```hlsl
struct texDistort{
    float2 uvScale(float2 uv, float scale){
        uv = (uv - 0.5) * scale + 0.5;
        return uv;
    }

    float2 texRotation(float2 uv, float angle){
        //顺时针
        float2x2 rotationMatrix = float2x2(cos(angle), -sin(angle),
                                           sin(angle), cos(angle));

        return mul(uv - 0.5, rotationMatrix) + 0.5;
    }
};

texDistort txd;

//float4 color = Texture2DSample(texObject, texObjectSampler, txd.uvScale(uv, 2.0));
float4 color = Texture2DSample(texObject, texObjectSampler, txd.texRotation(uv, radians(45)));

return color;
```
核心逻辑：
距离中心越远的像素，被施加的旋转角度不一样；同时角度随时间持续变化 → 形成螺旋扭动动画

#### 1. radius（到中心距离）起了什么作用？

`sin(3 * radius + 2 * time)`
-   越靠近中心：radius ≈ 0 → sin 值接近 0 → **旋转力度几乎为 0**
-   越靠近外圈：radius 变大 → sin 来回正负震荡 → **旋转力度很大**

✅ 关键：**中心几乎不转，外圈疯狂扭转**

一张圆图，中间固定、外圈强行扭转，天然会拉出螺旋线条。

#### 2. `sin(6.0 * angle)` 是什么效果？
angle 是点朝向中心的方向，绕中心完整一圈 = 360°
`6.0 * angle`：沿着圆圈走一圈，正弦函数会上下震荡 6 次。
效果：
圆圈上平均分成 6 段：
3 段区域：顺时针扭
3 段区域：逆时针扭
所以画面会出现**6 瓣对称花纹**。
#### 3. time 时间项 `+2*time`

随着时间推进，sin 里面的值持续变大。
相当于：**扭转的波纹，持续从中心向外扩散滚动**，动画就动起来了。
#### 4. 整合到一起发生了什么？

1.  图片中心点锁死，基本不动；
2.  越往外的像素，允许扭转的幅度越大；
3.  一圈上，6 个区域往左拧、6 个区域往右拧；
4.  随着时间变化，拧动的波纹不断向外移动；

前面所有代码**只算出了「这个像素应该旋转多少角度 primDist」，但没有真正执行旋转操作**。
`texRotate(uv, primDist)` 才是**真正把 UV 坐标旋转**的函数。
```hlsl
// 极坐标螺旋扭曲函数
float2 texDistortion(float2 uv, float time){
// 1. 转换uv到【以纹理中心(0.5,0.5)为原点】的极坐标
	float2 offset = uv - 0.5;
	float angle = atan2(offset.y, offset.x); // 当前点极角 θ
	float radius = length(offset);           // 当前点距离中心半径 r
	// 2. 基础扭曲幅度：随半径 + 时间正弦波动
	float distortion = 4 * sin(3 * radius + 2 * time);
	// 3. 扭曲角度：随极角周期性变化（6个周期 = 6瓣花纹）
	float primDist = sin(2.0 * angle) * distortion;

	// 4. 根据计算出的动态角度旋转UV
	return texRotation(uv, primDist);
}
```

# 雨水效果
这样就可以做出一个圆环，内部有点渐变的感觉，是第一步，涟漪的感觉
```hlsl
float4 result = float4(0, 0, 0, 0);
float2 pointCtr = float2(0.5, 0.5);
float2 uvOffset = uv - float2(0.5, 0.5);
float radiusMin = 0.05;
float radiusMax = 0.1;
float ringThickness = 0.005;
float pointDist = length(uvOffset);

float alpha = saturate(smoothstep(radiusMin - ringThickness, radiusMin + ringThickness, pointDist));

if(pointDist <= radiusMax + ringThickness && pointDist >= radiusMin - ringThickness){
    result += alpha;
}

return result;
```

![输入图片说明](/imgs/2026-08-02/EYiOgovbSEUnHUuA.png)

以上是绘制一个，水滴是很多的，开始准备绘制多个
`frac(x)`：取小数部分
例：frac (3.1415) = 0.1415；frac (5.98) = 0.98
运行流程（循环每一次）
1.  初始 seed 固定 `(123.456,789.012)`
2.  seed × 一个大浮点数（123.456），数值剧烈放大
3.  frac () 砍掉整数，只保留 0~1 之间的小数
4.  结果重新赋值给 seed，**作为下一轮计算的输入**
```hlsl
float4 result = float4(0, 0, 0, 0);

float ringThickness = 0.005;
float fadeInner = 0.005;

float2 seed = float2(123.456, 789.012);
float2 offsetRange = float2(-1, 1);
// 水滴个数
float drops = 100;

for(int i = 0; i < drops; i++){
    //随机数
    seed = frac(seed * 123.456);
    // 利用seed生成 [-1,1] 的随机偏移坐标
    float2 randOffset = lerp(offsetRange.x, offsetRange.y, seed);

    float radius = 0.05;
    // 当前uv减去随机偏移 = 把圆环中心挪到随机位置
    float2 offset = (uv - 0.5) - randOffset;

    float pointDist = length(offset);

    float alpha = saturate(smoothstep(radius - fadeInner, radius + fadeInner, pointDist));

    if(pointDist <= radius + ringThickness && pointDist >= radius - ringThickness){
        result += alpha;
    }
}

return result;
```

![输入图片说明](/imgs/2026-08-02/Kr5CSbhyfLp3phq5.png)

学个用法，可以`debug`显示时间，可以看到随着虚幻运行时间会变得越来越大，所以我们可以通过`frac`来限制只取得小数点

![输入图片说明](/imgs/2026-08-02/DHPEFBFd4EifYFRg.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE3Mjg5ODQyOSwxMTE5MzE4Nzk2LC0xMj
EzODA2NjIzLDE4MDgzNTI0OTMsLTg4NTUxMDUzMCwxODg0ODM3
ODgxLDkyMjgwNDgyOSwtMTI3MjAxODM3MywxNDMyNjkxOTc5LD
U4MTM1MDYxMSwxMzg0NjY1Nzk2LC05NTQwMjcyNTgsNTg0NjYy
NzgyLDIwNTkyNjI2NDIsLTIwNTUxNDg4NzEsLTkwMTUzNjkwMy
wtMzI3NjEzNzE2LDM5OTk1ODUzNCwxMDExMjAzNzYyLDIwMzQw
Mzk5MDNdfQ==
-->