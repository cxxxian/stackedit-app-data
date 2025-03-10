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
完善`pickShader`函数
```cpp
case MaterialType::PbrMaterial:
	result = mPbrShader;
	break;
```
在`renderObject`加入对应`case`，先做一个简单基础款，只有`vert`部分对应的变量传输，后面随着`frag`的实现慢慢往上加东西
```cpp
case MaterialType::PbrMaterial: {
	PbrMaterial* pbrMat = (PbrMaterial*)material;

	
	//mvp以及基础参数
	shader->setMatrix4x4("modelMatrix", mesh->getModelMatrix());
	shader->setMatrix4x4("viewMatrix", camera->getViewMatrix());
	shader->setMatrix4x4("projectionMatrix", camera->getProjectionMatrix());

	auto normalMatrix = glm::mat3(glm::transpose(glm::inverse(mesh->getModelMatrix())));
	shader->setMatrix3x3("normalMatrix", normalMatrix);

	shader->setVector3("cameraPosition", camera->mPosition);
	//pbr相关参数

}
	break;
```
## 3 编写D G F三大函数
在`pbr.frag`实现如下三大函数
注意`Fresnel`返回的是`vec3`
```glsl
#define PI 3.141592653589793

//NDF：α本应该表示粗糙度（roughness），α= r^2，便于美术调控
float NDF_GGX(vec3 N, vec3 H, float roughness){
	float a = roughness * roughness;
	float a2 = a*a;
	float NdotH = max(dot(N,H), 0.0);

	float num = a2;
	float denom =PI * (NdotH * NdotH *(a2 - 1.0) + 1.0);//分母

	denom = max(denom, 0.0001);//不能为0

	return num / denom;
}

//Geometry
float GeometrySchlickGGX(float NdotV, float roughness){
	float r = (roughness + 1.0);
	float k = r * r / 8.0;

	float num = NdotV;
	float denom = NdotV * (1.0 - k) + k;

	denom = max(denom, 0.00001);

	return num / denom;
}

//考虑几何遮蔽和几何阴影的共同作用得到的值
//此处的N是宏平面的法线方向
float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness){
	float NdotV = max(dot(N,V),0.0);
	float NdotL = max(dot(N,L),0.0);

	float ggx1 = GeometrySchlickGGX(NdotV, roughness);
	float ggx2 = GeometrySchlickGGX(NdotL, roughness);

	return ggx1 * ggx2;
}

//Fresnel
vec3 fresnelSchlick(vec3 F0, float HdotV){
	return F0 + (1.0 - F0) * pow((1.0 - HdotV), 5.0);
}
```

# 构造实验环境
先把`renderer.h`中的两个函数改成`std::vector<PointLight*> pointLights`点光源数组
```cpp
void render(
	Scene* scene,
	Camera* camera,
	std::vector<PointLight*> pointLights,
	AmbientLight* ambLight,
	unsigned int fbo = 0
);

void renderObject(
	Object* object,
	Camera* camera,
	std::vector<PointLight*> pointLights,
	AmbientLight* ambLight
);
```
并且`renderObject`我们只留下三个`case`
`case MaterialType::WhiteMaterial`和`case MaterialType::ScreenMaterial`和`case MaterialType::PbrMaterial`
由于这三个没有涉及到`pointLight`的使用，所以不会报错
暂时关闭`pointShadow`的使用
## 1 渲染环境改为四个点光源
## 2 生成一堆阵列小球
```cpp
std::vector<PointLight*> pointLights{};
void prepare() {
	//fbo = Framebuffer::createHDRFbo(WIDTH, HEIGHT);

	fboMulti = Framebuffer::createMultiSampleHDRFbo(WIDTH, HEIGHT);
	fboResolve = Framebuffer::createHDRFbo(WIDTH, HEIGHT);

	renderer = new Renderer();
	bloom = new Bloom(WIDTH, HEIGHT);
	sceneOff = new Scene();
	scene = new Scene();

	//pass 01
	auto geometry = Geometry::createSphere(1.0f);
	auto material = new PbrMaterial();

	for (int i = 0; i < 5; i++) {
		for (int j = 0; j < 5; j++) {
			auto mesh = new Mesh(geometry, material);
			mesh->setPosition(glm::vec3(i * 2.5, j * 2.5, 0.0f));
			sceneOff->addChild(mesh);
		}
	}

	//pass 02 postProcessPass:后处理pass
	auto sgeo = Geometry::createScreenPlane();
	screenMat = new ScreenMaterial();
	screenMat->mScreenTexture = fboResolve->mColorAttachment;
	auto smesh = new Mesh(sgeo, screenMat);
	scene->addChild(smesh);


	glm::vec3 lightPositions[] = {
			glm::vec3(-3.0f,  3.0f, 10.0f),
			glm::vec3(3.0f,  3.0f, 10.0f),
			glm::vec3(-3.0f, -3.0f, 10.0f),
			glm::vec3(3.0f, -3.0f, 10.0f),
	};
	glm::vec3 lightColors[] = {
		glm::vec3(300.0f, 0.0f, 0.0f),
		glm::vec3(0.0f, 300.0f, 0.0f),
		glm::vec3(0.0f, 0.0f, 300.0f),
		glm::vec3(300.0f, 300.0f, 300.0f)
	};
	for (int i = 0; i < 4; i++) {
		auto pointLight = new PointLight();
		pointLight->setPosition(lightPositions[i]);
		pointLight->mColor = lightColors[i];
		pointLights.push_back(pointLight);
	}
}
int main() {
	...
	while (glApp->update()) {
		...
		renderer->render(sceneOff, camera, pointLights, ambLight, fboMulti->mFBO);
		renderer->msaaResolve(fboMulti, fboResolve);
		bloom->doBloom(fboResolve);
		renderer->render(scene, camera, pointLights, ambLight);
	...
	}
	...
}
```
此时的场景就是这样全白色的阵列小球，为什么是白色的，因为我们还没进行`pbr`流程计算，现在`pbr.frag`里面直接输出的是`vec3(1.0)`，相应的`shader`传参数也还没做好
`pbr.frag`
```glsl
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

![输入图片说明](/imgs/2025-03-10/YuV4DCAa4sNGBq9V.png)

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU1NTk4OTAzNywxMzI1NzAzNDQ2LC0zNz
E0MzEzMzgsMTQ5OTQ4NDc3MCw2MDk0MDQwMjUsLTQ2NTgzOTM0
NSwtMjIwNDk1OTg2LDcyODQ5ODA0LC0xNjIxMzIwOTMxLC0xNz
M3NTk1NTU2LC0yMDEzMjQ4MzY1LDExMjk2ODI1MTQsLTIwODg3
NDY2MTJdfQ==
-->