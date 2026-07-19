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

如果想做出一种`3d`的感觉的话ke'yi
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA1Njk2MDUxNSwtMjc3ODY5NDI5LC0xMD
k2NjE3NTg1LC0xMTcxNTI0ODQsMTc5NjY0NzkwNiwxNTgzMjg4
MTc0LC05ODQ3Nzg3MDMsMjExMzg0OTEzNSw4MjMxMTM1MTksLT
g3NDkzNDIxOCwtNzIwMDg2OTc3LC0xNTkzMzcyMTQwLDE1Njk0
OTAzMjMsNzYzNDA5MDE1LC0yMDA3MjY4MTc2LC02NDgxOTc3NS
wxMDAwNzQwMTM5LDE5Mjk2OTU1MzUsLTY2MTg3NzI2XX0=
-->