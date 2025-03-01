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
然后我们去到`phongCSMShadow.frag`当中，设计一个方法`getCurrentLayer`， 返回的值是当前我们在哪一个分层（子锥体）中
`i <= csmLayerCount`，这里为什么要用`<=`，是因为我们的层边界有五个，分层（子锥体）是四个，就好像`0--1--2--3--4`，`--`有四个
然后我们把对应的`layer`声明不同颜色并输出
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
然后我们去`renderer.cpp`中的`renderObject`进行数据的传输到`shader`中

暂时先把所有的阴影相关的数据传输都注释掉，我们传输`layer`相关的参数过去，然后就可以暂时得到一个目的效果
```cpp
case MaterialType::PhongCSMShadowMaterial: {
	PhongCSMShadowMaterial* phongShadowMat = (PhongCSMShadowMaterial*)material;

	DirectionalLightCSMShadow* dirCSMShadow = (DirectionalLightCSMShadow*)dirLight->mShadow;
	shader->setInt("sampler", 0);
	phongShadowMat->mDiffuse->bind();

	shader->setInt("csmLayerCount", dirCSMShadow->mLayerCount);

	std::vector<float> layers;
	dirCSMShadow->generateCascadeLayers(layers, camera->mNear, camera->mFar);
	shader->setFloatArray("csmLayers", layers.data(), layers.size());
	...
}
	 break;
```
如下，分层很好的显示了出来

![输入图片说明](/imgs/2025-03-01/o1yOnDndgp3sxtN3.png)

# 纹理数组介绍

这里调用的方法就是`glTexImage3D`，以前是`2D`，现在多了一个层级的维度
`layerNum`代表的是一共有多少层

![输入图片说明](/imgs/2025-03-01/dcKmgRD1MWmzx4Si.png)

`layer`代表第`n`张`texture`
所以我们使用循环，渲染`n`张贴图

![输入图片说明](/imgs/2025-03-01/M4q5paNf9uhANRjk.png)

`shader`中所使用的采样器也不一样，原本是`sampler2D`，现在是`sampler2DArray`

![输入图片说明](/imgs/2025-03-01/Ln0hRrFQtN9RqVAx.png)

# 实践
## 1 Texture类加入CSMTextureArray的创建
## 2 FBO类加入CSMFBO的创建
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMTA2OTE1OTcsMzMwNTA4MTY3LC0xNj
Q5MjE5NDc1LC0xMjQ3ODM5MzUsLTY4ODQ3ODI5OSwxNDA0OTUy
ODk0LDE4NDIzNjMyMTksLTMyNTQ2MiwxMDY3NjA4MTQ3LDEwMD
g2MzM0NzgsLTg2Mzc5NDEwNCwtMTQ3ODY4MzI2OSwtOTAxMTc5
NjQ1LC0yMTQwMzY0NTYsMTQwNzU5OTY4MywtMTA2OTgyMDgyMS
wtNDgxMzIwMzEyLC0yMDk0MTI0MzMsMzIzNjA1MzkyLDExMzky
MjkxM119
-->