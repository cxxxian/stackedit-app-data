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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMjA3MjY4NTIsLTE3OTkwODEyOTUsLT
IwMjE1Njc3NjcsLTgxNzUzNTI5NSwxNjE1NDEzMDk0LDIwMzky
MzUxNjMsLTE2MDgwMTc1ODEsLTEyMDQyMDAwMDIsLTE3MjI3Nz
M1NzAsLTEyMDc3MzYwMjksLTEyNTg4NDc2NDksMTk1MTEyNTQw
Ml19
-->