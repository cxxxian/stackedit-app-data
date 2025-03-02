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

# 实践（CSM结构搭建与视锥体划分算法实现）
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

# 实践（纹理数组应用与fbo实现）
## 1 Texture类加入CSMTextureArray的创建
在`texture.h`制作函数并实现
这里要**注意**，
`dTex->mTextureTarget = GL_TEXTURE_2D_ARRAY`这一句话别忘了，因为我们默认的`mTextureTarget`是`GL_TEXTURE_2D`
我们在`bind`函数中会根据`GL_TEXTURE_2D`进行动态绑定，因为我们之前由于天空球材质加入了不一样的材质种类
```cpp
Texture* Texture::createDepthAttachmentCSMArray(unsigned int width, unsigned int height, unsigned int layerNum, unsigned int unit)
{
	Texture* dTex = new Texture();
	unsigned int depth;
	glGenTextures(1, &depth);
	glBindTexture(GL_TEXTURE_BINDING_2D_ARRAY, depth);

	glTexImage3D(GL_TEXTURE_2D_ARRAY, 0, GL_DEPTH_COMPONENT, width, height, layerNum, 0, GL_DEPTH_COMPONENT, GL_FLOAT, NULL);

	glTexParameteri(GL_TEXTURE_2D_ARRAY, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
	glTexParameteri(GL_TEXTURE_2D_ARRAY, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
	GLfloat borderColor[] = { 0.0,0.0,0.0,0.0 };
	glTexParameterfv(GL_TEXTURE_2D_ARRAY, GL_TEXTURE_BORDER_COLOR, borderColor);
	glTexParameteri(GL_TEXTURE_2D_ARRAY, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_BORDER);//u
	glTexParameteri(GL_TEXTURE_2D_ARRAY, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_BORDER);//v
	glBindTexture(GL_TEXTURE_BINDING_2D_ARRAY, 0);

	dTex->mTexture = depth;
	dTex->mWidth = width;
	dTex->mHeight = height;
	dTex->mUnit = unit;
	dTex->mTextureTarget = GL_TEXTURE_2D_ARRAY;

	return dTex;
}
```
## 2 FBO类加入CSMFBO的创建
```cpp
Framebuffer* Framebuffer::createCSMShadowFbo(unsigned int width, unsigned int height, unsigned int layerNumber)
{
	Framebuffer* fb = new Framebuffer();
	unsigned int fbo;
	glGenFramebuffers(1, &fbo);
	glBindFramebuffer(GL_FRAMEBUFFER, fbo);

	//加入深度附件
	Texture* depthAttachment = Texture::createDepthAttachmentCSMArray(width, height, layerNumber, 0);
	glFramebufferTextureLayer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, depthAttachment->getTexture(), 0, 0);
	glDrawBuffer(GL_NONE);//显式的告诉opengl， 我们当前这个fbo没有颜色输出
	glBindFramebuffer(GL_FRAMEBUFFER, 0);

	fb->mFBO = fbo;
	fb->mDepthAttachment = depthAttachment;
	fb->mWidth = width;
	fb->mHeight = height;

	return fb;
}
```

# 实践（光源矩阵计算）
## 1 Tool类中，加入getFrustumCornersWorldSpace函数

复习一下这张图，我们通过相机`Projection*View`，可以将视锥体的`NDC`坐标反推回世界坐标

![输入图片说明](/imgs/2025-03-01/uY2ejfeckQ9el1b7.png)

### 功能：传入一个相机Projection*View矩阵乘积，得到对应视锥体八个角点
方法声明在`tool.h`并实现
```cpp
std::vector<glm::vec4> Tools::getFrustumCornersWorldSpace(const glm::mat4& projView)
{
	const auto inv = glm::inverse(projView);
	std::vector<glm::vec4> corners{};
	for (int x = 0; x < 2; x++) {
		for (int y = 0; y < 2; y++) {
			for (int z = 0; z < 2; z++) {
				glm::vec4 ndc = glm::vec4(2.0f * x - 1.0, 2.0f * y - 1.0, 2.0f * z - 1.0, 1.0);
				glm::vec4 p_middle = inv * ndc;
				glm::vec4 p_world = p_middle / p_middle.w;
				corners.push_back(p_world);
			}
		}
	}
	return corners;
}
```
所以通过这一个方法，我们就成功把子视锥体的八个顶点转到世界坐标系下了
## 2 DirectionalLightCSMShadow类加入getLightMatrix函数.
### 功能：传入玩家相机+near+far+光，计算当前光源方向下，子椎体的LightMatrix
在`directionalLightCSMShadow.h`创建并实现
```cpp
glm::mat4 DirectionalLightCSMShadow::getLightMatrix(Camera* camera, glm::vec3 lightDir, float near, float far) {
	//1 求当前子视锥体的八个角点世界坐标系的值
	//此处的camera是玩家视角的相机
	auto perpCamera = (PerspectiveCamera*)camera;
	auto perpViewMatrix = perpCamera->getViewMatrix();
	auto perpProjectionMatrix = glm::perspective(glm::radians(perpCamera->mFovy), perpCamera->mAspect, near, far);
	auto corners = Tools::getFrustumCornersWorldSpace(perpProjectionMatrix * perpViewMatrix);

	//2 八个角点位置的平均值作为光源的位置，得到lightViewMatrix
	glm::vec3 center = glm::vec3(0.0f);
	for (int i = 0; i < corners.size(); i++) {
		//corners是四维向量要降为三维
		center += glm::vec3(corners[i]);
	}
	center /= corners.size();

	//glm::vec3(0.0, 1.0, 0.0)是穹顶向量
	auto lightViewMatrix = glm::lookAt(center, center + lightDir, glm::vec3(0.0, 1.0, 0.0));;

	//3 将八个角点转化到光源坐标系，并求最小的AABB包围盒
	//std::numeric_limits<float>::max()方法，可以取到float的最大值表示
	float minX = std::numeric_limits<float>::max();
	float maxX = std::numeric_limits<float>::min();
	float minY = std::numeric_limits<float>::max();
	float maxY = std::numeric_limits<float>::min();
	float minZ = std::numeric_limits<float>::max();
	float maxZ = std::numeric_limits<float>::min();

	//&v说明可能会改变v的值，但是不希望有改变的现象出现，可以加上const，称为不可改变的引用
	for (const auto& v : corners) {
		//将v点从世界坐标系下转换到了光源相机坐标系下
		const auto pt = lightViewMatrix * v;
		minX = std::min(minX, pt.x);
		maxX = std::max(maxX, pt.x);
		minY = std::min(minY, pt.y);
		maxY = std::max(maxY, pt.y);
		minZ = std::min(minZ, pt.z);
		maxZ = std::max(maxZ, pt.z);
	}

	//4 调整(包围盒以外的物体，也能够影响到其内部物体的阴影遮挡效果）
	maxZ *= 10;
	minZ *= 10;

	//5 计算当前光源的投影矩阵
	auto lightProjectionMatrix = glm::ortho(minX, maxX, minY, maxY, -maxZ, -minZ);

	return lightProjectionMatrix * lightViewMatrix;
}
```

理解一下这个正交相机的设计，按理来说，`near`平面和`far`平面都要在相机的前方，即传入的值都是正数
但是此时我们的相机是放在视景体内部的中心位置，所以就会导致`near`跑到相机的后面，即应该是一个负值
又因为我们的相机是朝向`-z`轴的
捋一下，现在我们的`maxZ`假如是`10`，`minZ`是`-10`的话，那就代表`near`对应的是`maxZ`，`far`对应的是`minZ`，（这个取值从包围盒的设计去理解）
那就不对了，因为`near`此时是在我们相机背后（`+z`轴方向）的，`far`是在我们相机前面（`-z`轴方向）的
所以我们在构造正交相机的时候，传入的`minZ`和`maxZ`都要加上负号

![输入图片说明](/imgs/2025-03-01/PdQfJp4yG3cwW0s3.png)

## 3 DirectionalLightCSMShadow类加入getLightMatrices函数
### 功能：传入玩家相机+光源+视锥体划分数据，计算每个子视锥体的LightMatrix
`clips`是视锥体切片数据，即每一个子视锥体的划分界限

![输入图片说明](/imgs/2025-03-01/eb26DYwKDcTVSsfR.png)

假设有四个切片边界，那就会划分出三个子视锥体，所以循环的次数为`clips.size() - 1`
每次依次传入两个作为`near`和`far`平面
比如第一次就是`0`作为近平面，`1`作为远平面
**tips**：
`const std::vector<float>& clips`如果传入一个数组并且不修改，这样子声明会更加高效，不修改的引用
```cpp
std::vector<glm::mat4> DirectionalLightCSMShadow::getLightMatrices(Camera* camera, glm::vec3 lightDir, const std::vector<float>& clips) {
	std::vector<glm::mat4> matrices;
	for (int i = 0; i < clips.size() - 1; i++) {
		auto lightMatrix = getLightMatrix(camera, lightDir, clips[i], clips[i + 1]);
		matrices.push_back(lightMatrix);
	}

	return matrices;
}

```

# 实践（shadowPass函数改版）
## 1 查缺补漏，查看DirectionalLightCSMShadow中有没有需要补充的内容
其他都和`directionalLightShadow`差不多，
只是`directionalLightCSMShadow`就不需要在构造函数中创建正交相机了，因为正交相机我们通过`getLightMatrix`方法创建在了子视锥体的中心位置
```cpp
DirectionalLightCSMShadow::DirectionalLightCSMShadow() {
	mRenderTarget = Framebuffer::createCSMShadowFbo(1024, 1024, mLayerCount);
}

DirectionalLightCSMShadow::~DirectionalLightCSMShadow() {
	delete mRenderTarget;
}

void DirectionalLightCSMShadow::setRenderTargetSize(int width, int height) {
	if (mRenderTarget != nullptr) {
		delete mRenderTarget;
	}

	mRenderTarget = Framebuffer::createCSMShadowFbo(width, height, mLayerCount);
}
```
## 2 修改renderShadowMap，每一次渲染N张ShadowMap给到每个子视锥体
原本我们在`frameBuffer.cpp`中，默认取出的是第`0`张图片作为`attachment`
```cpp
Framebuffer* Framebuffer::createCSMShadowFbo(unsigned int width, unsigned int height, unsigned int layerNumber)
{
	...
	glFramebufferTextureLayer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, depthAttachment->getTexture(), 0, 0);
	...
}
```
现在我们要设计动态切换`attachment`（渲染`N`张`ShadowMap`），因为这里的`depthAttachment`是我们用`static Texture* createDepthAttachmentCSMArray(unsigned int width, unsigned int height, unsigned int layerNum, unsigned int unit);`制作出来的，是一个类型为`GL_TEXTURE_2D_ARRAY`的`texture`的`depthAttachment`
所以我们去`renderer.cpp`中的`renderShadowMap`完善切换渲染的步骤
```cpp
void Renderer::renderShadowMap(
	const std::vector<Mesh*>& meshes, 
	DirectionalLight* dirLight, 
	Framebuffer* fbo) {
	...
	DirectionalLightCSMShadow* csmShadow = (DirectionalLightCSMShadow*)dirLight->mShadow;
	
	//4 循环为每个子视锥体渲染shadowMap
	for (int i = 0; i < csmShadow->mLayerCount; i++) {
		glFramebufferTextureLayer(
			GL_FRAMEBUFFER,
			GL_DEPTH_ATTACHMENT,
			csmShadow->mRenderTarget->mDepthAttachment->getTexture(),
			0, i);
	}
	...
}
```
慢慢来解释，
完整实现如下：
```cpp
void Renderer::renderShadowMap(
	Camera* camera,
	const std::vector<Mesh*>& meshes, 
	DirectionalLight* dirLight) {
	//1 确保现在的绘制不是postProcessPass的绘制，如果是，则不执行渲染
	bool isPostProcessPass = true;
	for (int i = 0; i < meshes.size(); i++) {
		auto mesh = meshes[i];
		if (mesh->mMaterial->mType != MaterialType::ScreenMaterial) {
			isPostProcessPass = false;
			break;
		}
	}
	if (isPostProcessPass) {
		return;
	}

	//2 保存原始状态，绘制shadowMap完毕后，要恢复原始状态
	GLint preFbo;
	glGetIntegerv(GL_FRAMEBUFFER_BINDING, &preFbo);

	GLint preViewport[4];
	glGetIntegerv(GL_VIEWPORT, preViewport);


	//3 设置ShadowPass绘制的时候所需的状态
	glEnable(GL_DEPTH_TEST);
	glDepthFunc(GL_LESS);
	glDepthMask(GL_TRUE);

	DirectionalLightCSMShadow* csmShadow = (DirectionalLightCSMShadow*)dirLight->mShadow;
	std::vector<float>layers;
	csmShadow->generateCascadeLayers(layers, camera->mNear, camera->mFar);
	auto lightMatrices = csmShadow->getLightMatrices(camera, dirLight->getDirection(), layers);

	glBindFramebuffer(GL_FRAMEBUFFER, csmShadow->mRenderTarget->mFBO);
	glViewport(0, 0, csmShadow->mRenderTarget->mWidth, csmShadow->mRenderTarget->mHeight);

	//4 循环为每个子视锥体渲染shadowMap
	for (int i = 0; i < csmShadow->mLayerCount; i++) {
		glFramebufferTextureLayer(
			GL_FRAMEBUFFER,
			GL_DEPTH_ATTACHMENT,
			csmShadow->mRenderTarget->mDepthAttachment->getTexture(),
			0, i);
		glClear(GL_DEPTH_BUFFER_BIT);//!!!重要
		mShadowShader->begin();
		mShadowShader->setMatrix4x4("lightMatrix", lightMatrices[i]);
		for (int i = 0; i < meshes.size(); i++) {
			auto mesh = meshes[i];
			auto geometry = mesh->mGeometry;

			glBindVertexArray(geometry->getVao());
			mShadowShader->setMatrix4x4("modelMatrix", mesh->getModelMatrix());

			if (mesh->getType() == ObjectType::InstancedMesh) {
				InstancedMesh* im = (InstancedMesh*)mesh;
				glDrawElementsInstanced(GL_TRIANGLES, geometry->getIndicesCount(), GL_UNSIGNED_INT, 0, im->mInstanceCount);
			}
			else {
				glDrawElements(GL_TRIANGLES, geometry->getIndicesCount(), GL_UNSIGNED_INT, 0);
			}
		}
		mShadowShader->end();
	}

	glBindFramebuffer(GL_FRAMEBUFFER, preFbo);
	glViewport(preViewport[0], preViewport[1], preViewport[2], preViewport[3]);
}
```

1. `std::vector<float>layers`声明一个`layers`数组，用来存放子视锥体的分层平面数据，通过调用`csmShadow->generateCascadeLayers`方法获得
2. 通过`csmShadow->getLightMatrices`方法获得每一个子视锥体在`light`坐标系下的`lightMatrix`（`lightProjectionMatrix * lightViewMatrix`）
3. 然后就是进入循环，为每一个子视锥体进行渲染`shadowMap`，以前我们只渲染了一张，而现在引入`CSM`的概念我们要渲染多张

可以重新捋一遍，`csmShadow->getLightMatrices`这个方法，里面调用了`getLightMatrice`，主要是做到了：
利用玩家的摄像机的投影矩阵和视角矩阵，将子视锥体的`ndc`转到世界空间坐标系下面，然后我们把子视锥体的八个点求中点作为光源位置，在那个位置建立正交相机，我们就可以把子视锥体转到光源坐标系下面，得到`lightMatrix`，这个`lightMatrix`是要传入`shader`中使用的
然后进入循环，渲染出我们的`csmShadow`，此时它就包含了很多张`shadowMap`在其中，因为我们的`mDepthAttachment`（`texture`）是用`texture2DArray`声明的，所以所有渲染出来的`shadowMap`都会被存在里面

# 实践（RenderPass完成）
## 1 修改phongCSMShadowShader，根据当前像素位于的分层信息选择
- 采样哪一个ShadowMap
- 增加uniform mat4 lightMatrices[20];
- 修改pcf函数，需要传入采样shadowmap的哪一个layer

我们先一步一步来实现
先制作`csm`方法并调用`getCurrentLayer`
```cpp
int getCurrentLayer(vec3 positionWorldSpace){
	//求当前像素在相机坐标系下的坐标
	vec3 positionCameraSpace = (viewMatrix * vec4(positionWorldSpace, 1.0)).xyz;
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

float csm(vec3 positionWorldSpace, vec3 normal, vec3 lightDir, float pcfRadius){
	int layer = getCurrentLayer(positionWorldSpace);

}
```
接下来要调用`pcf`方法，先看一下原来的
我们会发现用到了一个`lightSpaceClipCoord`是从`vert`传过来的
```glsl
in vec4 lightSpaceClipCoord;
uniform float pcfRadius;
float pcf(vec3 normal, vec3 lightDir,float pcfUVRadius){
	//1 找到当前像素在光源空间内的NDC坐标
	vec3 lightNDC = lightSpaceClipCoord.xyz/lightSpaceClipCoord.w;
	...
}
```
去到`vert`会发现，我们只有一个全局的`lightMatrix`，但是现在有了`csm`概念之后，每一个子视锥体的`lightMatrix`都是不一样的
```glsl
uniform mat4 lightViewMatrix;
uniform mat4 lightMatrix;//lightProjection * lightView

void main()
{
	...
	lightSpaceClipCoord = lightMatrix * transformPosition;
	lightSpacePosition = (lightViewMatrix * transformPosition).xyz;
}
```
怎么解决？全删了！
把`vert`相关的`lightSpaceClipCoord`和`lightMatrix`都删了，放到`frag`进行处理（`PCSS`也先删了）

我们在`pcf`方法中加入参数`lightSpaceClipCoord`，这样的话`csm`方法进行调用的时候动态计算传入不同子视锥体对应的`lightSpaceClipCoord`
```glsl
uniform float pcfRadius;
float pcf(vec4 lightSpaceClipCoord, vec3 normal, vec3 lightDir,float pcfUVRadius){
	...
}
```
我们从`cpu`传入
```glsl
uniform mat4 lightMatrices[20];
float csm(vec3 positionWorldSpace, vec3 normal, vec3 lightDir, float pcfRadius){
	int layer = getCurrentLayer(positionWorldSpace);
	vec4 lightSpaceClipCoord = lightMatrices[layer] * vec4(positionWorldSpace, 1.0);
	return 1.0;
}
```

## 2 修改renderObject函数，将uniform更新做好
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4MjUyMTM0MSwxNTk5Mjk4NTcwLDcwND
Y0NTAxMSw4NjUxMzk1OTYsMTkzNjUzMDAzMSwxMzY3ODQxMTQx
LDIwMjcxNDgwODgsNDQxNTc3NDAzLDE2MzAzNTk0MDQsMTI2Nz
Y2OTYyLDE1NzI4NzI2MjgsMTYyMTY2NDY1MSw4NTc4NTMwNTEs
LTE1NDAxNzM3MTIsMTEwMjY5MjYxNywtMTE3NDMwNTE4MCw3NT
E2NzM3OTIsLTkxMjYzMjA1MSwtMTExMDY5MTU5NywzMzA1MDgx
NjddfQ==
-->