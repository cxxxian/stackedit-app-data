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
	glm::vec3(300.0f, 300.0f, 300.0f),
	glm::vec3(300.0f, 300.0f, 300.0f),
	glm::vec3(300.0f, 300.0f, 300.0f),
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
`pbr.frag`如下：
```glsl
...
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

我们现在要做的就是正确构建`pbr`计算公式，然后通过`renderObject`把相应的变量传到`shader`中

# 完善frag中的main函数
就是完善全流程的`pbr`计算，利用`D G F`三个函数来进行计算，得到最终的颜色输出
```glsl
void main()
{
	//1 准备通用数据
	vec3 albedo  = texture(albedoTex, uv).xyz ;
	vec3 N =  normalize(normal);
	vec3 V = normalize(cameraPosition - worldPosition);
	
	//2 计算基础反射率
	vec3 F0 = vec3(0.04);
	//因为金属颜色可以用自身的颜色表示，所以使用mix函数，metallic越大，表示金属度越高，自身颜色占的权重就更大
	F0 = mix(F0, albedo, metallic);

	//3 遍历四个点光源，计算反射总共的radiance
	vec3 Lo = vec3(0.0);
	for(int i = 0; i < 4; i++){
		//3.1 准备计算单个光源贡献的时候用到的通用数据
		vec3 L = normalize(pointLights[i].position - worldPosition);
		//半程向量H
		vec3 H = normalize(L + V);
		float NdotL = max(dot(N,L),0.0);
		float NdotV = max(dot(N,V),0.0);

		//3.2 计算光源打到平面上的irradiance
		float dis = length(pointLights[i].position - worldPosition);
		float attenuation = 1.0 / (dis * dis);
		vec3 irradiance = pointLights[i].color * NdotL * attenuation;

		//3.3 计算NDF G F各项值
		float D = NDF_GGX(N, H, roughness);
		float G = GeometrySmith(N, V, L, roughness);
		vec3 F = fresnelSchlick(F0, max(dot(H, V), 0.0));

		//3.4 决定diffuse与specular各自比例多少
		//因为F考虑的就是有多少光被反射出去了，所以用来做ks的权重刚刚好
		vec3 ks = F;
		vec3 kd = vec3(1.0) - ks;
		//考虑问题：对于金属而言是没有diffuse反射的！
		kd *= (1.0 - metallic);

		//3.5 计算cook-torrance BRDF的值
		vec3 num = D * G * F;//分子
		float denom = max(4.0 * NdotL * NdotV, 0.0000001);//分母，这里用max函数防止分母为0的情况
		vec3 specularBRDF = num / denom; 

		//3.6 考虑specular + diffuse最终的反射结果
		//此处specularBRDF不用再乘一次ks，因为num = D * G * F，以及考虑了F项，而F = ks
		Lo += (kd * albedo / PI + specularBRDF) * irradiance;
	}
	vec3 ambient = vec3(0.1) * albedo;
	Lo += ambient;

	FragColor = vec4(Lo, 1.0);
}
```

# 完成uniform更新
## 1 PbrMaterial中加入albedo贴图、roughness、metallic参数
`PbrMaterial.h`中加入三个参数
```cpp
public:
	Texture* mAlbedo{ nullptr };
	float mRoughness{ 0.1f };
	float mMetallic{ 0.5f };
```
然后顺便去`main.cpp`中的`prepare`加上材质初始化
```cpp
void prepare() {
	...
	//pass 01
	auto geometry = Geometry::createSphere(1.0f);
	auto material = new PbrMaterial();
	material->mAlbedo = new Texture("assets/textures/box.png", 0, GL_SRGB_ALPHA);
	...
}
```
## 2 更新uniform
将相关参数都传到`shader`中
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

	//pbr相关参数更新
	shader->setFloat("roughness", pbrMat->mRoughness);
	shader->setFloat("metallic", pbrMat->mMetallic);
	shader->setInt("albedoTex", 0);
	pbrMat->mAlbedo->setUnit(0);
	pbrMat->mAlbedo->bind();

	for (int i = 0; i < pointLights.size(); i++) {
		shader->setVector3("pointLights[" + std::to_string(i) + "].color", pointLights[i]->mColor);
		shader->setVector3("pointLights[" + std::to_string(i) + "].position", pointLights[i]->getPosition());
	}
}
	break;
```
之前的箱子贴图效果不明显
先暂时将贴图颜色直接输出为红色
将`metallic`金属度直接拉到`1`，就能得到金属的感觉，在调一下`roughness`，就可以得到以下效果

![输入图片说明](/imgs/2025-03-10/qatyUcZI2WTpKjiA.png)

# 相关纹理替代
将`Shader`中的`roughness normal metallic albedo `都是用相关纹理替代

![输入图片说明](/imgs/2025-03-10/1dQcdACMhzDIgJa5.png)

解释一下这个`arm`贴图，分别代表`AO`，`Roughness`，`matellic`三个参数的值，分别对应`RGB`三个通道
`AO`是指环境遮蔽，还用不上
`Roughness`也用不上因为我们有专门`Roughness`的贴图
只需要用`matellic`，即`B`通道的值

我们去`pbrMaterial.h`中声明对应的材质贴图
但是对于这几个贴图我们要有不同的滤波的方式
`mAlbedo`用双线性插值，因为他是最主要的颜色贴图
剩下三个使用的是最邻近插值，因为我们需要有颗粒分明的效果，不会被周围的像素插值
```cpp
public:
	Texture* mAlbedo{ nullptr };//双线性插值
	Texture* mRoughness{ nullptr };//最邻近插值
	Texture* mNormal{ nullptr };//最邻近插值
	Texture* mMetallic{ nullptr };//最邻近插值
```
所以我们要去在`texture.h`创建一个最邻近插值滤波方法含函数并实现
```cpp
Texture* Texture::createNearestAttachment(const std::string& path)
{
	auto texture = new Texture(path, 0);
	glBindTexture(GL_TEXTURE_2D, texture->mTexture);

	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);

	glBindTexture(GL_TEXTURE_2D, 0);
	return texture;
}
```
然后去`main.cpp`中的`prepare`函数中初始化相关贴图
```cpp
void prepare() {
	...

	//pass 01
	auto geometry = Geometry::createSphere(1.0f);
	material = new PbrMaterial();
	material->mAlbedo = new Texture("assets/textures/pbr/slab_tiles_diff_2k.jpg", 0, GL_SRGB_ALPHA);
	material->mNormal = Texture::createNearestAttachment("assets/textures/pbr/slab_tiles_nor_gl_2k.jpg");
	material->mRoughness = Texture::createNearestAttachment("assets/textures/pbr/slab_tiles_rough_2k.jpg");
	material->mMetallic = Texture::createNearestAttachment("assets/textures/pbr/slab_tiles_arm_2k.jpg");
	...
}
```

然后去`pbr.frag`中设计相关贴图变量
但是我们要先去`pbr.vert`中计算一个`tbn`传到`frag`，后续的`normal`计算需要用（因为默认的`normal`贴图都是朝向z轴的，我们利用`tbn`才能支持旋转后也正确得到法线，其实就是将法线转到世界坐标系下）
所以如下，
我们需要从外界传入`aTangent`，利用`aTangent`和`normal`可以计算出`bitangent`，最后构成`tbn`矩阵
小回忆：
`tangent`是可以直接利用`modelMatrix`进行几何变换的
但是`normal`不行，比如放大后斜边的法向量就不一定垂直了
```glsl
#version 460 core
...
layout (location = 3) in vec3 aTangent;
...
out mat3 tbn;

//aPos作为attribute（属性）传入shader
//不允许更改的
void main()
{
	...
	normal =  normalMatrix * aNormal;
	vec3 tangent = normalize(mat3(modelMatrix) * aTangent);
	vec3 bitangent = normalize(cross(normal, tangent));
	tbn = mat3(tangent, bitangent, normal);
}
```

然后就可以回到`vert`了
```glsl
...
in mat3 tbn;
uniform sampler2D albedoTex;//物体颜色(反照率）
uniform sampler2D roughnessTex;//粗糙度贴图
uniform sampler2D metallicTex;//金属度贴图
uniform sampler2D normalTex;//法线贴图

void main()
{
	//1 准备通用数据
	vec3 albedo = texture(albedoTex, uv).xyz;
	vec3 V = normalize(cameraPosition - worldPosition);

	vec3 N = texture(normalTex, uv).xyz;
	N = N * 2.0 - 1.0;
	N = normalize(tbn * N);

	float metallic = texture(metallicTex, uv).b;
	float roughness = texture(roughnessTex, uv).r;
	
	...
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk3Mzg4Mjc3MSwtODgwOTk2Njc3LC0yNz
M3MzE1MzYsMTM1NDA4NzAyOSwtNjQxMTg4MTU0LC04MDY0MTk4
NzAsNzgyODk2Njg0LDE3OTAzMTU4MzQsLTIyMDE5OTkyNCwtOT
Q4OTgwMDM3LC0yNDc4OTA1NzUsMTMyNTcwMzQ0NiwtMzcxNDMx
MzM4LDE0OTk0ODQ3NzAsNjA5NDA0MDI1LC00NjU4MzkzNDUsLT
IyMDQ5NTk4Niw3Mjg0OTgwNCwtMTYyMTMyMDkzMSwtMTczNzU5
NTU1Nl19
-->