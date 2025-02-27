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
## 5.IMGUI中修改对bias、pcfRadius、tightness、renderTarget大小的调整
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE3NjI0NDM4OCwtMTY5NDg3MzI2MSw0Nj
A3NDEwMDMsLTI1NzgxNDc5MiwtMTgyODAyMDMwMSwtMTAzMDYy
Nzc3Niw5MjUyMjc2MzcsNDY2NzU2NTE0XX0=
-->