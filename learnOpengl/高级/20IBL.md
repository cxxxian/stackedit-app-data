![输入图片说明](/imgs/2025-04-08/06LhZMKZrXVp9RaD.png)

![输入图片说明](/imgs/2025-04-08/50awnfEQeSWu34EQ.png)

![输入图片说明](/imgs/2025-04-08/GyIHS2iXOsgRqOO6.png)

## 黎曼和

![输入图片说明](/imgs/2025-04-08/oS39XNvuY7mJbTRN.png)

![输入图片说明](/imgs/2025-04-08/5wJpcHKCBclPVSm2.png)

## 积分计算

**对内层和外层积分分别使用两次黎曼和**

![输入图片说明](/imgs/2025-04-08/E9Xqw1Ebus4RPh6y.png)

![输入图片说明](/imgs/2025-04-08/P9oEDfT9PgM7K6tq.png)

![输入图片说明](/imgs/2025-04-08/SV1exJqcS5fqUa9y.png)

![输入图片说明](/imgs/2025-04-08/c8f3ERlwJXefgF8B.png)

## 向量空间转换
把法线坐标系下的`P`乘上法线坐标系构成的矩阵，即可得到`P`在世界坐标系下的值

![输入图片说明](/imgs/2025-04-08/XeIErVIM2w3CKoIc.png)

![输入图片说明](/imgs/2025-04-08/PJsw238lPdd6ETvQ.png)

对应着公式看程序
`numSamples`循环结束后就会变成`n1 * n2`，
`localVec`是采样点在法线坐标系下的位置
然后我们利用法线坐标系矩阵将`localVec`转到世界坐标系下

![输入图片说明](/imgs/2025-04-08/iJzI5MwNxsH3Q6q6.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc2MTE3MTg1OSwxNzE0NDQ3MDg0LDIzND
M4OTg5LDY2MjM1MTUsLTE5MDY4MjM1NzMsMTc0NTAxMjcyNiwx
NTM1NDQwMjE4LC0yMDg4NzQ2NjEyXX0=
-->