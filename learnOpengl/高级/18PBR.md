![输入图片说明](/imgs/2025-03-07/BG2svCQWM9kGUdnF.png)

![输入图片说明](/imgs/2025-03-07/FlLYjRJaIu7grJJm.png)

# 辐射度量学
用于光的能量与传播的定义与计算

![输入图片说明](/imgs/2025-03-07/XHpZApk0oA4v9a0q.png)

# Radiant Intensity

## 辐射能量

![输入图片说明](/imgs/2025-03-07/eklgiY6JhKm1kruV.png)

## 辐射通量

![输入图片说明](/imgs/2025-03-07/mVUI4L8AkcuN7E93.png)

## 立体角

![输入图片说明](/imgs/2025-03-07/R5MpAqBR0kBzTqhv.png)

问题：
如何描述点光源在某个方向上有多么的亮？
注意：点光源没有面积，没有体积

## radiant intensity

![输入图片说明](/imgs/2025-03-07/Mxvau6tc3y6tCHSi.png)

`dΦ`表示`dw`内对应的功率大小
可以得出两个结论：
1. `intensity`和距离远近没有关系，只和`dΦ`和`dw`相关
2. 当`dw`无限小的时候，可以视为`intensity`是一个方向上的功率

# 微分立体角
 
![输入图片说明](/imgs/2025-03-07/QQPq8LjfcTjn7V9o.png)

![输入图片说明](/imgs/2025-03-07/Qbom7GTVdC09lPpr.png)

# Irradiance

![输入图片说明](/imgs/2025-03-07/3v7TDuR6k6gh48gA.png)

![输入图片说明](/imgs/2025-03-07/jMdxqINHf0oDjcPa.png)

画图推导一下可以得知：
真正的面积是`ds * cosθ`

![输入图片说明](/imgs/2025-03-07/Yq0yTsRZKUHJq1d5.png)

# Radiance

![输入图片说明](/imgs/2025-03-08/qS8aYOaiTgDV0MEY.png)

发送方：

![输入图片说明](/imgs/2025-03-08/NhlK2GeSwBo8oXrq.png)

这里的二次不是二次方的意思，而是二次求导

![输入图片说明](/imgs/2025-03-08/YifQQoxvmy5J5KMM.png)

接收方：

`L`其实就是`w`方向发过来的`Radiance`，
通过以下计算可以得出表面接收的`Irradiance`，这里要配合`Irradiance`的公式一起看

![输入图片说明](/imgs/2025-03-08/Lbb2laxvsPgZ32L9.png)

![输入图片说明](/imgs/2025-03-08/Byypax7tnWae758P.png)

捋一下，由光源这边的`dΦ`，以及对应的`dw`和`ds`，然后可知物体的`dw`和`ds`，就可以计算出物体对应的`dΦ`了

# BRDF
双向反射分布方程
`Bidirectional Reflectance Distribution Funtion`

![输入图片说明](/imgs/2025-03-08/si9pEaOPfLvIG17r.png)

这里为什么`E`和`L`前面都要加上一个微分，这是因为现实中是有很多光线共同作用得到结果，此时我们只考虑了一个方向（即一根光线），所以加上微分

![输入图片说明](/imgs/2025-03-08/2XlHY3Hro2hr56Rd.png)

这个式子的意思就是，
光线的`Radiance`转化为反射处的`Irradiance`，然后乘上`BRDF`系数，最后转到人眼处的`Radiance`

![输入图片说明](/imgs/2025-03-08/m4fVMzi2qL9fREyA.png)

可以把`BRDF`理解为材质相关的系数，因为我们计算光线`Radiance`->反射处的`Irradiance`->人眼处的`Radiance`，这之间并没有引入任何材质相关的概念，通过`BRDF`系数，就相当于一个系数告诉你应该要怎么反射，应该反射多少出去

# Rendering Equation
渲染方程

![输入图片说明](/imgs/2025-03-08/Bch7WmAeu7HlG9Sh.png)

当`dw`无穷小的时候，我们就可以改写成积分的形式
回顾一开始的立体角概念，`w`可以用极坐标的`θ`和`Φ`表示

![输入图片说明](/imgs/2025-03-08/8IfZr9j3f9YWHWPU.png)

加入物体本身的自发光，就能构成完整的渲染方程

![输入图片说明](/imgs/2025-03-08/PrfOlGv1cLO1q9t9.png)

# BRDF构造-漫反射
如下图的公式所示，我们分开两个`BRDF`进行研究构造：`diffuse`和`specular`

![输入图片说明](/imgs/2025-03-08/c08IQijLttdVaJaK.png)

![输入图片说明](/imgs/2025-03-08/AvIMK3Zt2wAeXG9i.png)

因为漫反射情况下，入射光线打到平面上后，会向四面八方发射出相同的出射光，所以可得与入射方向出射方向都无关
所以也就是`fr`与立体角参数`w`没有关系，即极坐标的`θ`和`Φ`
所以我们可以直接把`fr`提到积分外面

![输入图片说明](/imgs/2025-03-08/HdV9ZGBXKAtRkypR.png)

`L0(p)`四面八方的值也是一样的，跟`w`没有关系，直接提到积分外面
然后外面把`dw`拆成`sinθ*dθ*dΦ`

![输入图片说明](/imgs/2025-03-08/usoY0KdsjAJqMZCV.png)

![输入图片说明](/imgs/2025-03-08/Ud7zJkB6eavG0g8P.png)

# Specular微表面模型

![输入图片说明](/imgs/2025-03-08/udrdlvsTzd7rmBNt.png)

![输入图片说明](/imgs/2025-03-08/OhJu0KMlKgTBbmvQ.png)

## NDF

![输入图片说明](/imgs/2025-03-08/PVz5C4W5JgXmowDY.png)

我们通过`D(w)`函数，可以列出式子
`D(w) * dw = P(w)`，然后`P(w) * S`就可以得到`w`方向上，`dw`内的平面面积

![输入图片说明](/imgs/2025-03-09/JEUHmgQZRV7sQXyD.png)

![输入图片说明](/imgs/2025-03-09/pBx4l0TeLYzLNflg.png)

关于第二条性质的推导：

`D(w) * dw = S(w) / Smacro`
所以写成`S(w) = D(w) * dw * Smacro`
把所有微平面求和就可以得到`Smicro`

![输入图片说明](/imgs/2025-03-09/PXMy5D8kQkmW0imz.png)

![输入图片说明](/imgs/2025-03-09/5bsmMUSnJGrktSAt.png)

![输入图片说明](/imgs/2025-03-09/GujLh67uEwRsyojO.png)

![输入图片说明](/imgs/2025-03-09/pVJ0uoYGbHyQgOXU.png)

该函数图像长这样，当`a = 0.35`时
因为宏平面的法向总体还是朝上的，所以在`n * h = 1`时，占的比例应该大，所以概率密度很高
在`n * h = 0`时，概率密度应该变得很小

但是当`a`特别小时，整个函数会趋向于一条直线，也很好理解，因为tai'cu

![输入图片说明](/imgs/2025-03-09/00LhwnMJF5RAREhJ.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3ODU2ODA3NTMsOTQ4MjI2ODg2LDE0MD
cxODcwMjQsLTk3NzkxNzA2NCwyMTEzMzk4MDU5LDQwNTU1MTc0
MCwtMjU4NDkyMDIzLC0xMzkxMDM1NzIyLC0xNzQwNDMxNDE4LD
k0NzMzMDUsMTE2ODAxNzk2MiwtMTg4MTQ5OTM5OSwtNDIwNzk1
NTA1LDIwMDAwMDkwMzIsLTEwNDEyNzYyNTEsLTEyOTMzMTM2OT
UsLTE2Mjg0NDM2MiwtMTg4MjgxMjgwOSwxNDg3Mjg1ODA4LC0x
Nzk5MDgxMjk1XX0=
-->