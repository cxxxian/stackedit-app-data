# 算法介绍
 ![输入图片说明](/imgs/2025-02-28/MlvIKaHW69fbrCui.png)
 
![输入图片说明](/imgs/2025-02-28/cIszCS4ep3rp44ep.png)

![输入图片说明](/imgs/2025-02-28/LOtvUuFLq7ZLhKQ9.png)

![输入图片说明](/imgs/2025-02-28/EC5xuIQ1DfI0UxVl.png)

# 视锥体划分算法

![输入图片说明](/imgs/2025-02-28/JB6arthKfUApwilD.png)

![输入图片说明](/imgs/2025-02-28/z1pqFpFjkieK8F4e.png)

![输入图片说明](/imgs/2025-02-28/wjvyEYImwMOmdAHY.png)

# 子视锥体LightMatrix计算原理

![输入图片说明](/imgs/2025-03-01/CbTqYPCWZEIVqKn5.png)

![输入图片说明](/imgs/2025-03-01/857OU8SdEQ6vq1Q4.png)

整理一下我们得到的已知量，
`near`和`far`大小已知，以及`fov`视张角已知，就可以构建视锥体，得出投影矩阵
然后相机的位置和方向也已知，就可以得到`viewMatrix`

![输入图片说明](/imgs/2025-03-01/ZA5RZ2ATAn32t2P6.png)

`Pw = modelMatrix * P`

![输入图片说明](/imgs/2025-03-01/uY2ejfeckQ9el1b7.png)

![输入图片说明](/imgs/2025-03-01/wzXlRdHvOuJTmgv6.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxNDAzNjQ1NiwxNDA3NTk5NjgzLC0xMD
Y5ODIwODIxLC00ODEzMjAzMTIsLTIwOTQxMjQzMywzMjM2MDUz
OTIsMTEzOTIyOTEzLDIxNzkyNDc0MywtMTI0MDUyOTcxMiwtOD
I0NzY2NTY0LC0xNDI0Mzc1Nzk2LDEyOTc4NTczMjMsLTcwMjk5
NDk5XX0=
-->