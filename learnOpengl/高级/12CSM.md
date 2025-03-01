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

在`directionalLightCSMShadow.h`中，
设计一个方法用来划分层次
```cpp
public:
	...
	void generateCascadeLayers(std::vector<float>& layers, float near, float far);
	...
public:
	int mLayerCount = 5;
};
```
实现如下，根据公式计算每一层的距离然后存入`layer`数组中
```cpp
void DirectionalLightCSMShadow::generateCascadeLayers(std::vector<float>& layers, float near, float far) {
	layers.clear();

	for (int i = 0; i <= mLayerCount; i++) {
		float layer = near * glm::pow(far / near, (float)i / (float)mLayerCount);
		layers.push_back(layer);
	}
}
```
然后我们去到`phongCSMShadow.frag`当中
```glsl
uniform int csmLayerCount;
uniform float csmLayers[20];
uniform mat4 viewMatrix;

int getCurrentLayer(){
	//求当前像素在相机坐标系下的坐标
	vec3 positionCameraSpace = (viewMatrix * vec4(worldPosition, 1.0)).xyz;
	float z = -positionCameraSpace.z;

	int layer = 0;
	for(int i = 0;i <= csmLayerCount;i++){
		if(z < csmLayers[i]){
			layer = i - 1;
			break;
		}
	}

	return layer;
}

void main()
{
//环境光计算
	...
	int layer = getCurrentLayer();
	vec3 maskColor = vec3(0.0,0.0,0.0);
	switch(layer){
		case 0:
			maskColor = vec3(1.0,0.0,0.0);
		break;
		case 1:
			maskColor = vec3(0.0,1.0,0.0);
		break;
		case 2:
			maskColor = vec3(0.0,0.0,1.0);
		break;
		case 3:
			maskColor = vec3(0.0,1.0,1.0);
		break;
		case 4:
			maskColor = vec3(1.0,0.0,1.0);
		break;
	}
	finalColor = finalColor * maskColor;
	FragColor = vec4(finalColor,alpha * opacity);
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQwNDk1Mjg5NCwxODQyMzYzMjE5LC0zMj
U0NjIsMTA2NzYwODE0NywxMDA4NjMzNDc4LC04NjM3OTQxMDQs
LTE0Nzg2ODMyNjksLTkwMTE3OTY0NSwtMjE0MDM2NDU2LDE0MD
c1OTk2ODMsLTEwNjk4MjA4MjEsLTQ4MTMyMDMxMiwtMjA5NDEy
NDMzLDMyMzYwNTM5MiwxMTM5MjI5MTMsMjE3OTI0NzQzLC0xMj
QwNTI5NzEyLC04MjQ3NjY1NjQsLTE0MjQzNzU3OTYsMTI5Nzg1
NzMyM119
-->