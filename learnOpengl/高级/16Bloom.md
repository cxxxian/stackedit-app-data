# 概念
辉光效果，亮到有点朦胧感

![输入图片说明](/imgs/2025-03-05/19E8GAhmZbG9b1ZJ.png)

![输入图片说明](/imgs/2025-03-05/W3qAfbhg9LA8ejEk.png)

![输入图片说明](/imgs/2025-03-05/slRBC1ZfWoJsu1ss.png)

![输入图片说明](/imgs/2025-03-05/DMw90Rym202hqm3l.png)

![输入图片说明](/imgs/2025-03-05/SlS8r1j4y6GpO5mZ.png)

![输入图片说明](/imgs/2025-03-05/yhCihKjmoMEgQYAK.png)

![输入图片说明](/imgs/2025-03-05/DswyvpydLLp5apLC.png)

![输入图片说明](/imgs/2025-03-05/i69ZauxWGm1crciL.png)

![输入图片说明](/imgs/2025-03-05/gO2uQmVNNxnPAMhK.png)

# 实现方式解析

![输入图片说明](/imgs/2025-03-05/QAyI6USOoz6rY7s4.png)

![输入图片说明](/imgs/2025-03-05/GotyZblQOrTGgjqx.png)

下采样完之后我们进行上采样，上采样同样可以用`glBlitFrameBuffer`，但是我们自定义一个`poisson`采样效果更好

![输入图片说明](/imgs/2025-03-05/g6vfUxsWRPHpIs7a.png)

![输入图片说明](/imgs/2025-03-05/I1q3SwtUQB0xAwBY.png)

# 实践
## 1 FrameBuffer类增加HDR下Bloom专用的fbo
特点：只需要颜色`attachment`，因为是`screenSpace`的绘制

## 2 创建Bloom类（初步）
- 下采样`FBO`数组
- 上采样`FBO`数组
- 初始化宽度高度，有多少级的下采样图片数量
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDkwNjI4ODQsMTIwODE5ODE1MV19
-->