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

# ShadowPass
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
我们要理解`OpenGL`是一个状态机的概念，
```cpp
void Renderer::render(
	Scene* scene, 
	Camera* camera,
	DirectionalLight* dirLight,
	AmbientLight* ambLight,
	unsigned int fbo
) {
	glBindFramebuffer(GL_FRAMEBUFFER, fbo);
	...

	//渲染shadowMap
	renderShadowMap(mOpacityObjects, dirLight, mShadowFBO);
	...
}
```
所以我们一开始是想要先渲染`pass01`，然后`pass02`，但此时我们插入了一个渲染`shadowMap`的过程，此时就会有问题，状态被改变了，那么后面两次的正式渲染就会绘制在`mShadowFBO`这个`fbo`上面

然后关于`viewPort`的问题
以前的`viewPort`就是在`main.cpp`中用鼠标进行监测并修改大小
```cpp
void OnResize(int width, int height) {
	GL_CALL(glViewport(0, 0, width, height));
}
```
但是这时候由于我们在这里：
```cpp
Renderer::Renderer() {
	...
	mShadowFBO = Framebuffer::createShadowFBO(2048, 2048);
}
```
修改了`viewPort`的大小，并且也没有调整回去，肯定会出问题

有了以上几点注意事项，现在我们正式开始设计`renderShadowMap`
```cpp
void Renderer::renderShadowMap(const std::vector<Mesh*>& meshes, DirectionalLight* dirLight, Framebuffer* fbo)
{
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
	glGetIntegerv(GL_FRAMEBUFFER_BINDING, &preFbo);//查找当前所绑定的fbo，并赋值给preFbo

	GLint preViewport[4];
	glGetIntegerv(GL_VIEWPORT, preViewport);
	
	//3 设置shadoPass绘制的时候所需的状态
	//4 开始绘制


	glBindFramebuffer(GL_FRAMEBUFFER, preFbo);
	glViewport(preViewport[0], preViewport[1], preViewport[2], preViewport[3]);
}
```
先搞定我们刚刚说的两个注意事项
我们通过遍历`meshes`判断`material`可以确保现在的绘制不是`postProcessPass`的绘制
然后通过`glGetIntegerv`记录需要保存的状态信息，然后我们最后再重新对`framebuffer`绑定和`viewport`赋值

我们再在`renderer.h`设计一个函数，用来计算`LightMatrix`，这是后续要传到`depthShader`中的，封装成函数比较有条理
```cpp
glm::mat4 getLightMatrix(DirectionalLight* dirLight);
```
实现如下：
```cpp
glm::mat4 Renderer::getLightMatrix(DirectionalLight* dirLight)
{
	//1 viewMatrix
	auto lightViewMatrix = glm::inverse(dirLight->getModelMatrix());

	//2 projection（正交投影）
	float size = 6.0f;
	auto lightCamera = new OrthographicCamera(-size, size, -size, size, 0.1, 1000);
	auto lightProjectionMatrix = lightCamera->getProjectionMatrix();
	
	//3 求lightMatrix并且返回
	return lightProjectionMatrix * lightViewMatrix;
}
```
总的`renderDepthMap`的函数如下：
由于`lightMatrix`是统一的，只需要传一次，其他关于`mesh`自身的（`modelMatrix`），进入循环依次传输
```cpp
void Renderer::renderShadowMap(const std::vector<Mesh*>& meshes, DirectionalLight* dirLight, Framebuffer* fbo)
{
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
	glGetIntegerv(GL_FRAMEBUFFER_BINDING, &preFbo);//查找当前所绑定的fbo，并赋值给preFbo

	GLint preViewport[4];
	glGetIntegerv(GL_VIEWPORT, preViewport);
	
	//3 设置shadoPass绘制的时候所需的状态
	glEnable(GL_DEPTH_TEST);
	glDepthFunc(GL_LESS);
	glDepthMask(GL_TRUE);

	glBindFramebuffer(GL_FRAMEBUFFER, fbo->mFBO);
	glViewport(0, 0, fbo->mWidth, fbo->mHeight);

	//4 开始绘制
	glClear(GL_DEPTH_BUFFER_BIT);

	auto lightMatrix = getLightMatrix(dirLight);
	mShadowShader->begin();
	mShadowShader->setMatrix4x4("lightMatrix", lightMatrix);
	for (int i = 0; i < meshes.size(); i++) {
		auto mesh = meshes[i];
		auto geometry = mesh->mGeometry;
		glBindVertexArray(geometry->getVao());
		mShadowShader->setMatrix4x4("modelMatrix", mesh->getModelMatrix());

		//4 执行绘制命令
		if (mesh->getType() == ObjectType::InstancedMesh) {
			InstancedMesh* im = (InstancedMesh*)mesh;
			glDrawElementsInstanced(GL_TRIANGLES, geometry->getIndicesCount(), GL_UNSIGNED_INT, 0, im->mInstanceCount);
		}
		else {
			glDrawElements(GL_TRIANGLES, geometry->getIndicesCount(), GL_UNSIGNED_INT, 0);
		}

	}

	mShadowShader->end();
	
	glBindFramebuffer(GL_FRAMEBUFFER, preFbo);
	glViewport(preViewport[0], preViewport[1], preViewport[2], preViewport[3]);
}
```
这时候我们去`main.cpp`构造试验场景，将`shadowMap`直接当作`pass02`进行输出
```cpp
void prepare() {
	...

	//pass 02
	auto sgeo = Geometry::createScreenPlane();
	auto smat = new ScreenMaterial();
	//smat->mScreenTexture = fbo->mColorAttachment;
	smat->mScreenTexture = renderer->mShadowFBO->mDepthAttachment;
	auto smesh = new Mesh(sgeo, smat);
	scene->addChild(smesh);

	dirLight = new DirectionalLight();
	dirLight->setPosition(glm::vec3(3.0f, 3.0f, 3.0f));
	dirLight->rotateY(45.0f);
	dirLight->rotateX(-45.0f);
	dirLight->mSpecularIntensity = 0.5f;

	ambLight = new AmbientLight();
	ambLight->mColor = glm::vec3(0.1f);
}
```
输出的结果如下，会发现是黑的
这是因为我们设计的正交相机给的数据中的`near`和`far`平面太大了，而我们的相机距离几何体很近，把（相机到几何体距离） 占 （`far - near`）距离的占比转化到`0~1`，就几乎没有了

![输入图片说明](/imgs/2025-02-25/gPnp9AmVRvDV6rZC.png)

![输入图片说明](/imgs/2025-02-25/jawyhBiIfoHTri1E.png)

```cpp
glm::mat4 Renderer::getLightMatrix(DirectionalLight* dirLight)
{
	//1 viewMatrix
	auto lightViewMatrix = glm::inverse(dirLight->getModelMatrix());

	//2 projection（正交投影）
	float size = 6.0f;
	auto lightCamera = new OrthographicCamera(-size, size, -size, size, 0.1, 100);
	auto lightProjectionMatrix = lightCamera->getProjectionMatrix();
	
	//3 求lightMatrix并且返回
	return lightProjectionMatrix * lightViewMatrix;
}
```
将`far`换成`100`之后，就不会全是黑色了

![输入图片说明](/imgs/2025-02-25/fXdczrMNuEvEPRpY.png)

然后全是红色的，这是什么原因？
因为我们的`screen.frag`是对`screenTexSampler`进行采样，此时`screenTexSampler`我们传过来的是`shadowMap`，只有`r`通道有值，所以会是红色的
```glsl
void main()
{
	...
	vec3 color = texture(screenTexSampler, uv).rgb;

	//2 最终颜色要抵抗屏幕gamma
	color = pow(color, vec3(1.0/2.2));
	FragColor = vec4(color, 1.0);
}
```

# RenderPass
## 1.加入新的phongShadow.vert/frag，加入是否位于阴影中的判断
`phongShadow.vert`如下：
计算完`lightSpaceClipCoord`往后传到`frag`中
```glsl
#version 460 core
...
out vec4 lightSpaceClipCoord;//光源空间内的剪裁空间坐标
....
uniform mat4 lightMatrix;//lightProjection * lightView

void main()
{
	// 将输入的顶点位置，转化为齐次坐标（3维-4维）
	vec4 transformPosition = vec4(aPos, 1.0);
	//做一个中间变量TransformPosition，用于计算四位位置与modelMatrix相乘的中间结果
	transformPosition = modelMatrix * transformPosition;
	...

	lightSpaceClipCoord = lightMatrix * transformPosition;
}
```
`phongShadow.frag`如下：
我们做一个`calculateShadow`方法用来判断`shadow`值是`0`还是`1`，分别对应无阴影和有阴影
```glsl
in vec4 lightSpaceClipCoord;
uniform sampler2D shadowMapSampler;

float calculateShadow(){
	//1 找到当前像素在光源空间内的NDC坐标
	vec3 lightNDC = lightSpaceClipCoord.xyz / lightSpaceClipCoord.w;
	//2 找到当前像素在shadowMap上的uv
	vec3 projCoord = lightNDC * 0.5 + 0.5;//+0.5会被自动扩展为三维向量
	vec2 uv = projCoord.xy;

	//3 使用这个uv对SM进行采样，得到closestDepth
	float closestDepth = texture(shadowMapSampler, uv).r;
	
	//4 对比当前像素在光源空间内的深度值与closestDepth的大小
	float selfDepth = projCoord.z;

	float shadow = selfDepth > closestDepth? 1.0:0.0;
	return shadow;
}
void main()
{
	...

	float shadow = calculateShadow();
	vec3 finalColor = result * (1.0 - shadow) + ambientColor;
	FragColor = vec4(finalColor,alpha * opacity);
}
```
## 2.加入新的PhongShadowMaterial材质
把`phongMaterial`复制改为`phongShadowMaterial`
并且在`material.h`加上`PhongShadowMaterial`枚举
## 3.在Renderer中对新材质进行解析，并且更新uniform
### 3.1加入新的shader对象
这里别搞混了
`mShadowShader`是我们用来渲染`shadowMap`的`shader`
`mPhongShadowShader`是我们用来根据`shadowMap`的结果进行真实的阴影判断并渲染场景的`shader`
```cpp
private:
	...
	Shader* mShadowShader{ nullptr };
	Shader* mPhongShadowShader{ nullptr };
```
### 3.2pickShader加入对新材质的兼容
```cpp
Shader* Renderer::pickShader(MaterialType type) {
	...
	case MaterialType::PhongShadowMaterial:
		result = mPhongShadowShader;
		break;
	...
	return result;
}

```
### 3.3对新的材质进行uniform更新
其实就是多了`shadowMapSampler`和`lightMatrix`
`shadowMapSampler`就是`shadowMap`
`lightMatrix`调用我们设计的方法得到
```cpp
 case MaterialType::PhongShadowMaterial: {
	PhongShadowMaterial* phongShadowMat = (PhongShadowMaterial*)material;

	//diffuse贴图帧更新
	//将纹理采样器与纹理单元进行挂钩
	shader->setInt("sampler", 0);
	//将纹理与纹理单元进行挂钩
	phongShadowMat->mDiffuse->bind();

	shader->setInt("shadowMapSampler", 1);
	mShadowFBO->mDepthAttachment->setUnit(1);
	mShadowFBO->mDepthAttachment->bind();

	shader->setMatrix4x4("lightMatrix", getLightMatrix(dirLight));
	...
}
	break;
```
## 4.修改场景当中物体的材质（兼容阴影）
都修改`material`为`PhongShadowMaterial`，
以及用回颜色贴图`smat->mScreenTexture = fbo->mColorAttachment;`
```cpp
void prepare() {
	fbo = new Framebuffer(WIDTH, HEIGHT);

	renderer = new Renderer();
	sceneOff = new Scene();
	scene = new Scene();

	//pass 01
	auto geo = Geometry::createBox(2.0);
	auto mat = new PhongShadowMaterial();
	mat->mDiffuse = new Texture("assets/textures/parallax/bricks.jpg", 0, GL_SRGB_ALPHA);

	mat->mShiness = 32;
	auto mesh = new Mesh(geo, mat);
	sceneOff->addChild(mesh);

	auto groundGeo = Geometry::createPlane(10, 10);
	mat = new PhongShadowMaterial();
	mat->mDiffuse = new Texture("assets/textures/grass.jpg", 0, GL_SRGB_ALPHA);
	mat->mShiness = 32;

	auto groundMesh = new Mesh(groundGeo, mat);
	groundMesh->setPosition(glm::vec3(0.0, 0.0, 0.0f));
	groundMesh->rotateX(-90.0f);
	sceneOff->addChild(groundMesh);

	//pass 02 postProcessPass:后处理pass
	auto sgeo = Geometry::createScreenPlane();
	auto smat = new ScreenMaterial();
	smat->mScreenTexture = fbo->mColorAttachment;
//	smat->mScreenTexture = renderer->mShadowFBO->mDepthAttachment;
	auto smesh = new Mesh(sgeo, smat);
	scene->addChild(smesh);

	
	dirLight = new DirectionalLight();
	dirLight->setPosition(glm::vec3(3.0f, 3.0f, 3.0f));
	dirLight->rotateY(45.0f);
	dirLight->rotateX(-45.0f);
	dirLight->mSpecularIntensity = 0.5f;

	ambLight = new AmbientLight();
	ambLight->mColor = glm::vec3(0.1f);

}
```
运行后就可以得到正确的阴影计算了，此时出现了摩尔纹，后续解决

![输入图片说明](/imgs/2025-02-25/R5RLbT65kAAPG1Lm.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU2NTc3MTc0OCwxNTE5MzMwMDcxLC0zOT
YxODIyNTYsLTE5Nzg4OTQ1MTksMTM2Nzk5MjU5MSwxMTgwMDkz
NzgxLDEyMjc2MzA1NjcsMTc1MDQyMTI0MCwyMDQ0Njg1MzE4LD
EyNjcxMjQ2MTUsLTY5ODc1OTAyNywtNzkwMTY3MTE0LC0xMzYy
ODc2MjgxLC0yMTQzODIyNDA0LC0xNjc2NjY1Njk2LC04NTg0Mj
UwNTMsMTU2MjQ4OTk1MSw0Mjc5ODMyMTAsLTgxNzI1MDQyOCwx
OTM0MjM3NTMyXX0=
-->