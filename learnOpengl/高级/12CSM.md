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

子锥体有八个点要确认，它是长得像一个棱台，这里二维方向所以只能看到四个点

![输入图片说明](/imgs/2025-03-01/857OU8SdEQ6vq1Q4.png)

整理一下我们得到的已知量，
`near`和`far`大小已知，以及`fov`视张角已知，就可以构建视锥体，得出投影矩阵
然后相机的位置和方向也已知，就可以得到`viewMatrix`

![输入图片说明](/imgs/2025-03-01/ZA5RZ2ATAn32t2P6.png)

`Pw = modelMatrix * P`

![输入图片说明](/imgs/2025-03-01/uY2ejfeckQ9el1b7.png)

![输入图片说明](/imgs/2025-03-01/wzXlRdHvOuJTmgv6.png)

![输入图片说明](/imgs/2025-03-01/iPH9ivqgSWEX3aMV.png)

# CSM算法概览即实现步骤规划

![输入图片说明](/imgs/2025-03-01/waoEQT0FoWKqu7Fa.png)

![输入图片说明](/imgs/2025-03-01/jXykrmSeTQ60BbAO.png)

# 实践
## 加入新结构
### 1 加入新影子DrectionalLightCSMShadow
就直接复制`DrectionalLightShadow.h`和`DrectionalLightShadow.cpp`，之后再做修改操作
### 2 加入新材质PhongCSMShadowMaterial
复制`PhongShadowMaterial`并改名为`PhongCSMShadowMaterial`
在`material.h`中添加一个新材质
```cpp
//使用C++的枚举类型
enum class MaterialType {
	...
	PhongCSMShadowMaterial
};
```
### 3 加入新shader，PhongCSMShadow.vert/frag

## 加入视锥体划分
### CSM阴影中，加入generateCascadeLayers函数 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMyNzQ3NDk3OSwxMDA4NjMzNDc4LC04Nj
M3OTQxMDQsLTE0Nzg2ODMyNjksLTkwMTE3OTY0NSwtMjE0MDM2
NDU2LDE0MDc1OTk2ODMsLTEwNjk4MjA4MjEsLTQ4MTMyMDMxMi
wtMjA5NDEyNDMzLDMyMzYwNTM5MiwxMTM5MjI5MTMsMjE3OTI0
NzQzLC0xMjQwNTI5NzEyLC04MjQ3NjY1NjQsLTE0MjQzNzU3OT
YsMTI5Nzg1NzMyMywtNzAyOTk0OTldfQ==
-->