# 准备工作
## 1.改造Light类，继承Object（并为Object增加getDirection函数）
因为之前我们说过，`light`其实也是有位置的，所以我们可以把`Object`做为`Light`的父类（当初设计`Object`类就有位置等参数），
继承后就可以利用`Object`的`getModelMatrix`得到`M`矩阵，而上节课说到的，`M`的逆矩阵就是光源摄像机的`viewMatrix`
然后我们原本的`directionLight`有一个参数是：
```cpp
public:
	glm::vec3 mDirection{-1.0};
```
但现在光源既然继承于`Object`，那它就可以进行旋转什么的，所以我们要为`Object`增加`getDirection`函数，因为此时的`mDirection`可能会因为旋转什么的发生变化

开始修改代码：
将`Light`继承于`Object`，这样所有的光源就都继承于`Object`了，因为`light.h`是所有光源的父类
```cpp
#pragma once
#include "../core.h"
#include "../object.h"
class Light: public Object {
	...
};
```
然后去把所有光源有关于方向的可以直接删除了
检查过后发现只有`directionLight`和`spotLight`有
分别是
```cpp
public:
	glm::vec3 mDirection{-1.0};
```
和
```cpp
glm::vec3	mTargetDirection{ -1.0f };
```

现在我们开始设计`geiDirection`方法

![输入图片说明](/imgs/2024-11-01/rKYoiuZ2ttQqLKy0.png)

我们先回忆一下先前设计的旋转矩阵，第一列是`right`轴，第二列是`up`轴，第三列是`front`轴（取`-f`是因为我们的向前对应的是`z`方向），分别对应`xyz`

因为我们要得到光照的方向，所以只要关注旋转矩阵的`front`方向即可，代表向前的向量
有了这个回忆，我们就可以开始设计了
在`object.h`声明并在`object.cpp`实现：
```cpp
glm::vec3 Object::getDirection() const {
	auto modelMatrix = glm::mat3(getModelMatrix());
	auto dir = glm::normalize(-modelMatrix[2]) ;

	return dir;
}
```
解释一下，可以利用上面那张图，`getModelMatrix()`此时得到的是一个四维矩阵左上角的`3*3`矩阵是旋转矩阵，第四列的前三个代表平移，而第四行是我们为了齐次进行的补全
所以我们通过`glm::mat3(getModelMatrix())`就可以获得旋转矩阵
而此时我们只需要`front`向量，所以`modelMatrix[2]`就可以取得`modelMatrix`的第三列即`front`，最后还需要加上一个负号，因为此时是朝向`z`轴的，但是`-z`轴才是我们的前方

## 2.更改使用光源的代码们
由于我们删除了`directionLight`和`spotLight`的方向，肯定会有很多报错
慢慢来改
 
```cpp
void prepare() {
	...
	dirLight = new DirectionalLight();
	dirLight->mDirection = glm::vec3(0.0f, -0.4f,-1.0f);
	dirLight->mSpecularIntensity = 0.5f;

	ambLight = new AmbientLight();
	ambLight->mColor = glm::vec3(0.1f);
}
```
这里的`main.cpp`中的`dirLight->mDirection = glm::vec3(0.0f, -0.4f,-1.0f);`就已经在报错了，因为我们把这个变量已经删了
所以我们现在要利用`rotate`函数进行更改方向（继承自`Object`），并且要注意这里的旋转是根据`dirLight`本地的坐标轴进行旋转的
```cpp
dirLight = new DirectionalLight();
dirLight->rotateY(45.0f);
dirLight->rotateX(-25.0f);
dirLight->mSpecularIntensity = 0.5f;
```
然后就是取`render.cpp`中修改，我们传到`shader`用的都是`dirLight->mDirection`
要改成我们自己设计的方法`getDirection()`
举个例子：
```cpp
shader->setVector3("directionalLight.direction", dirLight->getDirection());
```
全改完就可以正常运行了
构造一个小场景先：

![输入图片说明](/imgs/2025-02-25/yhUUeos5oPbPZy3v.png)

# ShadowMap
## shader制作
### 1.创建shadow.vert/frag，渲染阴影贴图专用
`shadow.vert`如下：
```glsl
#version 460 core
layout(location = 0) in vec3 aPos;

uniform mat4 lightMatrix;//lightProjection * lightView
uniform mat4 modelMatrix;

void main(){
	gl_Position = lightMatrix * modelMatrix * vec4(aPos, 1.0);
}
```
`shadow.frag`如下：
```glsl
#version 460 core
//不需要输出任何颜色
//用于绘制shadowMap的FBO只有深度附件没有颜色附件
void main(){
}
```
### 2.在Renderer中创建shadowShader，用于做ShadowMap渲染
```cpp
private:
	...
	Shader* mShadowShader{ nullptr };
```
```cpp
Renderer::Renderer() {
	...
	mShadowShader = new Shader("assets/shaders/advanced/shadow.vert", "assets/shaders/advanced/shadow.frag");
}
```

## 渲染目标
### 1.在Texture中增加创建DepthAttachment的创建函数
`texture.h`声明函数
```cpp
static Texture* createDepthAttachment(
	unsigned int width,
	unsigned int height,
	unsigned int unit
);
```
然后再到`texture.cpp`实现
```cpp
Texture* Texture::createDepthAttachment(
	unsigned int width,
	unsigned int height,
	unsigned int unit
) {
	Texture* depthTex = new Texture();

	unsigned int depth;
	glGenTextures(1, &depth);
	glBindTexture(GL_TEXTURE_2D, depth);

	glTexImage2D(GL_TEXTURE_2D, 0, GL_DEPTH_COMPONENT, width, height, 0, GL_DEPTH_COMPONENT, GL_FLOAT, NULL);

	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);

	//设置纹理的包裹方式
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);//u
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);//v

	depthTex->mTexture = depth;
	depthTex->mWidth = width;
	depthTex->mHeight = height;
	depthTex->mUnit = unit;

	return depthTex;
}
```
### 2.在FrameBuffer中增加创建ShadowFBO的创建函数
`frameBuffer.h`中，我们不仅要创建函数`createShadowFBO`，并且还要创建一个空的构造函数，因为我们以前的构造函数用来创建了另一套`fbo`（颜色）
```cpp
static Framebuffer* createShadowFBO(unsigned int width, unsigned int height);
Framebuffer();
```
然后`frameBuffer.cpp`实现如下：
```cpp
Framebuffer* Framebuffer::createShadowFBO(unsigned int width, unsigned int height)
{
	Framebuffer* fb = new Framebuffer();
	fb->mWidth = width;
	fb->mHeight = height;

	glGenFramebuffers(1, &fb->mFBO);
	glBindFramebuffer(GL_FRAMEBUFFER, fb->mFBO);

	fb->mDepthAttachment = Texture::createDepthAttachment(width, height, 0);
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_TEXTURE_2D, fb->mDepthAttachment->getTexture(), 0);

	glDrawBuffer(GL_NONE);//显式地告诉opengl，我们当前这个fbo没有颜色输出
	glBindFramebuffer(GL_FRAMEBUFFER, 0);

	return fb;
}
Framebuffer::Framebuffer()
{
}
```
### 3.在Renderer中创建ShadowFBO，用于做阴影ShadowMap的渲染目标（RenderTarget）
```cpp
#include "../framebuffer/framebuffer.h"
public:
	Framebuffer* mShadowFBO{ nullptr };
```
我们`shadowMap`的大小是可以自己设置大小的
```cpp
Renderer::Renderer() {
	...
	mShadowFBO = Framebuffer::createShadowFBO(2048, 2048);
}
```
这里要对应我们的`viewPort`，如果我们的`viewPort`是`1024*1024`的话，那么只会把东西都绘制到`shadowMap`的左下角四分之一的位置，很浪费空间。

![输入图片说明](/imgs/2025-02-25/tUy6L1Yyn2nCEDci.png)

## 渲染器修改
在`Renderer`中加入`RenderShadowMap`函数，在真正渲染物体之前，先把`ShadowMap`做出来
在`renderer.h`中设计一个函数用来渲染`shadowMap`
```cpp
void renderShadowMap(const std::vector<Mesh*> &meshes, DirectionalLight* dirLight, Framebuffer* fbo);
```
然后对应实现，但是在实现之前，我们先思考一下要在哪里调用
```cpp
void Renderer::render(
	Scene* scene, 
	Camera* camera,
	DirectionalLight* dirLight,
	AmbientLight* ambLight,
	unsigned int fbo
) {
	...

	std::sort(
		mTransparentObjects.begin(), 
		mTransparentObjects.end(),
		[camera](const Mesh* a, const Mesh* b) {
			...
		}
	);

	//渲染shadowMap
	renderShadowMap(mOpacityObjects, dirLight, mShadowFBO);

	//3 渲染两个队列
	for (int i = 0; i < mOpacityObjects.size(); i++) {
		renderObject(mOpacityObjects[i], camera, dirLight, ambLight);
	}

	for (int i = 0; i < mTransparentObjects.size(); i++) {
		renderObject(mTransparentObjects[i], camera, dirLight, ambLight);
	}
}
```
如以上，我们在排序完之后就要进行渲染`shadowMap`
因为再往后就已经是渲染离屏场景了，已经要用到`shadowMap`了，所以只能在此之前准备好
然后我们在这里只传入`mOpacityObjects`，即不透明的物体
因为`shadowMap`不好处理半透明物体，我们判断完遮挡后就会直接挡住光，并不能做到半透过一点光，所以干脆就直接只处理不透明物体

### 注意1：
做好排除工作，`ScreenMaterial`的物体不参与`ShadowPass`渲染，若是`PostProcessPass`则不进行`RenderShadowMap`的操作（防止污染`ShadowMap`）

解释：
这是因为我们离屏渲染调用的是`render`函数，而`postProcessPass`调用的也是`render`渲染，而我们的`shadowMap`渲染方法调用正是在`render`函数中的，这就导致，我们`pass01`渲染出一张阴影贴图后，`pass02`又会渲染一次并覆盖

![输入图片说明](/imgs/2025-02-25/ZhOlpB7LvSyFCELo.png)

### 注意2：
做好备份工作，先前的`fbo`，先前的`viewport`等参数，都需要做备份与恢复
解释：
我们要理解`OpenGL`是一个状态机的概念，suo

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTE4NzU0MjYzLC04NTg0MjUwNTMsMTU2Mj
Q4OTk1MSw0Mjc5ODMyMTAsLTgxNzI1MDQyOCwxOTM0MjM3NTMy
LDg1MjQyMTI5NiwtMTE0MzA0Njk2NCwtMzAzMTEwOTUzLDE4Mz
M3ODU1NzksMTI5MTc4NTk5MSw3Nzk1Mjc2MzcsMzEzMTEyNDQz
LC0xODYwMTY5NjExLC0yMTg3NzcxMzUsLTMzODIxMDYwMl19
-->