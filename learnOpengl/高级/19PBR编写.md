![输入图片说明](/imgs/2025-03-09/H5fQTdOQ9h2nCRVH.png)

# 准备工作
## 1 创建pbr的Material以及shader
`pbrMaterial.h`如下
```cpp
#pragma once
#include "../material.h"
#include "../../texture.h"

class PbrMaterial :public Material {
public:
	PbrMaterial();
	~PbrMaterial();
	...
};
```
`pbrMaterial.cpp`如下，以及去到`material.h`中添加对应枚举类
```cpp
#include "pbrMaterial.h"

PbrMaterial::PbrMaterial() {
	mType = MaterialType::PbrMaterial;
}

PbrMaterial::~PbrMaterial() {

}
```
`pbr.vert`如下，和`phong.vert`没区别
```glsl
#version 460 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aUV;
layout (location = 2) in vec3 aNormal;

out vec2 uv;
out vec3 normal;
out vec3 worldPosition;

uniform mat4 modelMatrix;
uniform mat4 viewMatrix;
uniform mat4 projectionMatrix;

uniform mat3 normalMatrix;

//aPos作为attribute（属性）传入shader
//不允许更改的
void main()
{
// 将输入的顶点位置，转化为齐次坐标（3维-4维）
	vec4 transformPosition = vec4(aPos, 1.0);

	//做一个中间变量TransformPosition，用于计算四位位置与modelMatrix相乘的中间结果
	transformPosition = modelMatrix * transformPosition;

	//计算当前顶点的worldPosition，并且向后传输给FragmentShader
	worldPosition = transformPosition.xyz;

	gl_Position = projectionMatrix * viewMatrix * transformPosition;
	
	uv = aUV;
	normal =  normalMatrix * aNormal;
}
```
`pbr.frag`如下：
```glsl
#version 460 core
out vec4 FragColor;

in vec2 uv;
in vec3 normal;
in vec3 worldPosition;

uniform sampler2D albedoTex;	//物体颜色(反照率）

//相机世界位置
uniform vec3 cameraPosition;
#include "../../common/lightStruct.glsl"

uniform PointLight pointLights[4];

#define PI 3.141592653589793

void main()
{
//环境光计算
	vec3 objectColor  = texture(albedoTex, uv).xyz ;

	//计算光照的通用数据
	vec3 normalN = normalize(normal);
	vec3 viewDirN = normalize(worldPosition - cameraPosition);
	

	FragColor = vec4(1.0);
}
```
## 2 pbr材质解析加入管线中
在`renderer.h`中声明新的`shader`对象
```cpp
//pbr相关
Shader* mPbrShader{ nullptr };
```
并且到`renderer.cpp`中进行初始化
```cpp
Renderer::Renderer() {
	mPbrShader = new Shader("assets/shaders/advanced/pbr/pbr.vert", "assets/shaders/advanced/pbr/pbr.frag");
}
```
## 3 编写D G F三大函数

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM0MzgzMTUyOCwtMTczNzU5NTU1NiwtMj
AxMzI0ODM2NSwxMTI5NjgyNTE0LC0yMDg4NzQ2NjEyXX0=
-->