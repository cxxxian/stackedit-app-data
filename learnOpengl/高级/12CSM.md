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
也是复制`PhongShadow.vert/frag`即可

## 合并到渲染流程当中
快速过一遍流程
声明`shader`变量
```cpp
Shader* mPhongCSMShadowShader{ nullptr };
```
初始化
```cpp
mPhongCSMShadowShader = new Shader("assets/shaders/advanced/phongCSMShadow.vert", "assets/shaders/advanced/phongCSMShadow.frag");
```
完善`pickShader`
```cpp
Shader* Renderer::pickShader(MaterialType type) {
	Shader* result = nullptr;
	switch (type) {
	...
	case MaterialType::PhongCSMShadowMaterial:
		result = mPhongCSMShadowShader;
		break;
	...
	}

	return result;
}
```
然后在`renderObject`加入对应`case`即可

搭建一个实验场景
会发现后面是黑色，这是因为超出了我们视景体的范围，我们先前设计的是将其`shadowMap`的`uv`值超出的部分直接设为`0`，所以和`0`相比什么东西都被遮住了，即是阴影区域

![输入图片说明](/imgs/2025-03-01/3lNhLKYsqKzqNFNu.png)

## 加入视锥体划分
### CSM阴影中，加入generateCascadeLayers函数 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU2NzEwMTU0NCwtMzI1NDYyLDEwNjc2MD
gxNDcsMTAwODYzMzQ3OCwtODYzNzk0MTA0LC0xNDc4NjgzMjY5
LC05MDExNzk2NDUsLTIxNDAzNjQ1NiwxNDA3NTk5NjgzLC0xMD
Y5ODIwODIxLC00ODEzMjAzMTIsLTIwOTQxMjQzMywzMjM2MDUz
OTIsMTEzOTIyOTEzLDIxNzkyNDc0MywtMTI0MDUyOTcxMiwtOD
I0NzY2NTY0LC0xNDI0Mzc1Nzk2LDEyOTc4NTczMjMsLTcwMjk5
NDk5XX0=
-->