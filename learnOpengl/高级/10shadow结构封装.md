![输入图片说明](/imgs/2025-02-26/aL4BSM1rT8dVSror.png)

这里设计的几个参数，解释几个看起来比较抽象的
当场景中的灯多起来后，就需要多个`shadowMap`，所以`camera, renderTarget`就要设计在`shadow`类下，每一个不同阴影都要有自己的`camera`和`renderTarget`
而这个`setRenderTargetSize`方法，在前面加上`virtual`，后面加上`=0`，说明这是一个纯虚函数，需要子类强制进行实现，
以前我们是直接写死在`renderer.cpp`中，如下：
`mShadowFBO = Framebuffer::createShadowFbo(2048, 2048);`
大小为`2048*2048`，而我们以后肯定会有不同`shadowMap`大小的定制需求，所以子类需要强制进行重写方法

![输入图片说明](/imgs/2025-02-26/DsjfF0lOja219sUp.png)

![输入图片说明](/imgs/2025-02-26/mfl7Gdi2fZhSKIvj.png)

# 实现
## 1.加入Shadow父类，并且派生DirectionalLightShadow子类
`shadow`父类的设计
`mCamera`是用来获得`projectionMatrix`（投影矩阵）的
而我们还需要光源的矩阵，这个设计在子类中
```cpp
#pragma once
#include "../../core.h"
#include "../../framebuffer/framebuffer.h"
#include "../../../application/Application.h"
#include "../../../application/camera/camera.h"

class Shadow {
	Shadow();
	~Shadow();

	virtual void setRenderTargetSize(int width, int height) = 0;

public:
	Camera*		mCamera{ nullptr };
	Framebuffer*	mRenderTarget{ nullptr };

	float	mBias{ 0.0003f };
	float	mPcfRadius{ 0.1f };
	float	mDiskTightness{ 1.0f };
};
```
`directionalLightShadow`设计如下：
```cpp
#pragma once
#include "shadow.h"

class DirectionalLightShadow :public Shadow {
public:
	DirectionalLightShadow();
	~DirectionalLightShadow();

	void setRenderTargetSize(int width, int height)override;
	glm::mat4 getLightMatrix(glm::mat4 lightModelMatrix);
};
```
### 1.1 注意setRenderTargetSize的实现
### 1.2 DirectionalLightShadow中，加入getLightMatrix函数
根据以上两点注意事项，实现`directionalLightShadow.cpp`
```cpp
#include "directionalLightShadow.h"
#include "../../../application/camera/orthographicCamera.h"

DirectionalLightShadow::DirectionalLightShadow()
{
	float size = 10.0;
	mCamera = new OrthographicCamera(-size, size, -size, size, 0.1f, 80.0f);
	mRenderTarget = Framebuffer::createShadowFbo(1024, 1024);
}

DirectionalLightShadow::~DirectionalLightShadow()
{
	delete mCamera;
	delete mRenderTarget;
}

void DirectionalLightShadow::setRenderTargetSize(int width, int height)
{
	if (mRenderTarget != nullptr) {
		delete mRenderTarget;
	}

	mRenderTarget = Framebuffer::createShadowFbo(width, height);

}
glm::mat4 DirectionalLightShadow::getLightMatrix(glm::mat4 lightModelMatrix)
{
	auto viewMatrix = glm::inverse(lightModelMatrix);
	auto projMatrix = mCamera->getProjectionMatrix();

	return projMatrix * viewMatrix;

}
```
## 2.在Light父类中，加入Shadow类型对象，并且在平行光中对其进行初始化（其余暂时不管）	
在`light.h`中初始化一个`mShadow`，在平行光中对其进行初始化
```cpp
#pragma once
#include "../core.h"
#include "../object.h"
#include "shadow/shadow.h"

class Light:public Object {
public:
	Light();
	~Light();

public:
	glm::vec3	mColor{ 1.0f };
	float		mSpecularIntensity{ 1.0f };
	float		mIntensity{ 1.0 };

	Shadow*		mShadow{ nullptr };
};
```
在平行光中对其进行初始化如下：
```cpp
#include "shadow/directionalLightShadow.h"

DirectionalLight::DirectionalLight() {
	mShadow = new DirectionalLightShadow();
}
```
## 3.RenderShadowMap中更改FBO与lightMatrix获取方式
以前我们是在`renderer.h`中声明了一个`mShadow`，现在可以将其删除了。
然后修改相关报错
先前我们这里使用的是`mShadow`
```cpp
renderShadowMap(mOpacityObjects, dirLight, mShadow);
```
改为调用`dirLight->mShadow->mRenderTarget`，这是刚刚设计的，并且我们方法参数里面也有`dirLight`
```cpp
void Renderer::render(
	Scene* scene, 
	Camera* camera,
	DirectionalLight* dirLight,
	AmbientLight* ambLight,
	unsigned int fbo
) {
	...
	//渲染Shadowmap
	renderShadowMap(mOpacityObjects, dirLight, dirLight->mShadow->mRenderTarget);
	...
	}
}
```
原本是在`renderer.h`中设计了一个`getLightMatrix`，现在我们直接删除即可，因为我们在`DirectionalLightShadow`下也设计了一个
所以将`dirLight->mShadow`强转为`DirectionalLightShadow`即可
```cpp
void Renderer::renderShadowMap(const std::vector<Mesh*>& meshes, DirectionalLight* dirLight, Framebuffer* fbo)
{
	...
	DirectionalLightShadow* dirShadow = (DirectionalLightShadow*)dirLight->mShadow;
	auto lightMatrix = dirShadow->getLightMatrix(dirLight->getModelMatrix());
	...
}

```
## 4.RenderObject中，更改ShadowMap以及其他系列参数的更新
```cpp
case MaterialType::PhongShadowMaterial: {
	...
	//----shadow相关------
	shader->setInt("shadowMapSampler", 1);
	dirShadow->mRenderTarget->mDepthAttachment->setUnit(1);
	dirShadow->mRenderTarget->mDepthAttachment->bind();

	shader->setMatrix4x4("lightMatrix", dirShadow->getLightMatrix(dirLight->getModelMatrix()));
	//--------------------
	...
	//bias
	shader->setFloat("bias", dirShadow->mBias);
	//tightness
	shader->setFloat("diskTightness", dirShadow->mDiskTightness);
	//pcfRadius
	shader->setFloat("pcfRadius", dirShadow->mPcfRadius);
}
	break;
```
## 5.IMGUI中修改对bias、pcfRadius、tightness、renderTarget大小的调整
`bias, pcfRadius, tightness`都很简单，直接调用`&dirLight->mShadow`下的变量即可
`renderTarget`大小的调整就比较麻烦了，需要监测`imgui`的对应滑块是否有被滑动（因为`imgui`是每帧进行调用的）
如果有滑动就调用`dirLight->mShadow->setRenderTargetSize(width, height)`方法
```cpp
void renderIMGUI() {
	...

	//2 决定当前的GUI上面有哪些控件，从上到下
	ImGui::Begin("MaterialEditor");
	ImGui::SliderFloat("Bias:", &dirLight->mShadow->mBias, 0.0f, 0.01f, "%.4f");
	ImGui::SliderFloat("Tightness:", &dirLight->mShadow->mDiskTightness, 0.0f, 5.0f, "%.3f");
	ImGui::SliderFloat("PcfRadius:", &dirLight->mShadow->mPcfRadius, 0.0f, 1.0f, "%.4f");

	int width = dirLight->mShadow->mRenderTarget->mWidth;
	int height = dirLight->mShadow->mRenderTarget->mHeight;
	if (
		ImGui::SliderInt("FBO width", &width, 1, 4096) ||
		ImGui::SliderInt("FBO height", &height, 1, 4096))
	{
		dirLight->mShadow->setRenderTargetSize(width, height);
	}
	...
	ImGui::End();
	...
}
```

可以发现，如果`shadowMap`的分辨率特别小的话，即使我们`Bias`很大，也会出现自遮挡现象

![输入图片说明](/imgs/2025-02-27/69XOvAgyCbDwPCap.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTQxMDAzMDUzLC0xNTU5MjkxMDAzLDExNz
YyNDQzODgsLTE2OTQ4NzMyNjEsNDYwNzQxMDAzLC0yNTc4MTQ3
OTIsLTE4MjgwMjAzMDEsLTEwMzA2Mjc3NzYsOTI1MjI3NjM3LD
Q2Njc1NjUxNF19
-->