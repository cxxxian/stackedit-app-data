# 概念

![输入图片说明](/imgs/2025-03-04/B9n1v5GEoutVtS8l.png)

![输入图片说明](/imgs/2025-03-04/OWjZqnQNfpq7Babh.png)

![输入图片说明](/imgs/2025-03-04/jJFBgc4xG0urG1sN.png)

意思就是，`fragment`不会变大，变大的是针对每一个`fragment`的采样点

![输入图片说明](/imgs/2025-03-04/8sPQGq21hNJl78ji.png)

 以上就是`MSAA`过程，最后的`PostProcessPass`其实就是我们做的后处理`Gamma`校正
 
# 代码分析

![输入图片说明](/imgs/2025-03-04/IP1xRacTyMcXlFWJ.png)

![输入图片说明](/imgs/2025-03-04/T1TQCBtJe8F4dACp.png)

这个函数`glBlitFramebuffer`代表：
从源头`src`的什么位置开始，源头的宽度，源头的高度
复制到`dst`的什么位置开始，目标用多少宽度来承接，目标用多少高度来承接

![输入图片说明](/imgs/2025-03-04/M3fj6hqjoIRxpDEu.png)

# 实践
## 1 创建MSAA专用纹理
在`texture.h`声明函数并实现
步骤上没有大的差别，几个新参数：
`samples`用来指定有多少个采样点
`format`用来指定格式（例如`GL_RGBA`或者`GL_DEPTH24_STENCIL8`）
按照以前我们需要设计该纹理的采样（`filter`）方式（`GL_NEAREST`或者`GL_LINEAR`）以及超过`0, 1`后要怎么处理这个纹理
但是现在是超采样，所以没有必要指定采样方式，我们更不会有超过`0, 1`的情况
```cpp
Texture* Texture::createMultiSampleTexture(unsigned int width, unsigned int height, unsigned int samples, unsigned int format, unsigned int unit)
{
	Texture* tex = new Texture();

	unsigned int glTex;
	glGenTextures(1, &glTex);
	glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, glTex);

	glTexImage2DMultisample(GL_TEXTURE_2D_MULTISAMPLE, samples, format, width, height, GL_TRUE);
	glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, 0);

	tex->mTexture = glTex;
	tex->mWidth = width;
	tex->mHeight = height;
	tex->mUnit = unit;
	tex->mTextureTarget = GL_TEXTURE_2D_MULTISAMPLE;

	return tex;
}
```
## 2 创建MSAA专用FBO
这里`glFramebufferTexture2D`方法中的：
`GL_TEXTURE_2D_MULTISAMPLE`代表要把哪个目标点的`texture`连入进来，后面附上对应的`texture`：`colorAttachment->getTexture()`和`dsAttachment->getTexture()`
```cpp
Framebuffer* Framebuffer::createMultiSampleFbo(unsigned int width, unsigned int height, unsigned int samples)
{
	Framebuffer* fb = new Framebuffer();
	unsigned int fbo;
	glGenFramebuffers(1, &fbo);
	glBindFramebuffer(GL_FRAMEBUFFER, fbo);

	auto colorAttachment = Texture::createMultiSampleTexture(width, height, samples, GL_RGBA, 0);
	auto dsAttachment = Texture::createMultiSampleTexture(width, height, samples, GL_DEPTH24_STENCIL8, 0);

	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D_MULTISAMPLE, colorAttachment->getTexture(), 0);
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_TEXTURE_2D_MULTISAMPLE, dsAttachment->getTexture(), 0);

	fb->mFBO = fbo;
	fb->mColorAttachment = colorAttachment;
	fb->mDepthAttachment = dsAttachment;
	fb->mWidth = width;
	fb->mHeight = height;

	return fb;
}
```
## 3 Renderer中增加msaaResolve函数
`renderer.h`新建并实现函数
```cpp
void Renderer::msaaResolve(Framebuffer* src, Framebuffer* dst)
{
	glBindFramebuffer(GL_READ_FRAMEBUFFER, src->mFBO);
	glBindFramebuffer(GL_DRAW_FRAMEBUFFER, dst->mFBO);
	glBlitFramebuffer(0, 0, src->mWidth, src->mHeight, 0, 0, dst->mWidth, dst->mHeight, GL_COLOR_BUFFER_BIT, GL_NEAREST);
}

```
## 4 绘制流程更改
我们到`main.cpp`中，需要声明两个`fbo`，一个用来做超采样，一个用来承接超采样得到的下采样
```cpp
Framebuffer* fboMultiSample = nullptr;
Framebuffer* fboResolve = nullptr;
```
然后到`prepare`函数中，初始化两个`fbo`
`fboMultiSample`用我们设计的函数`createMultiSampleFbo`
`fboResolve`就普通初始化即可
`pass01`并没有和`fbo`有什么关系，
关键在于`pass02`，我们要把`smat->mScreenTexture = fboResolve->mColorAttachment`，即把下采样过后的`mColorAttachment`赋给屏幕进行渲染
```cpp
void prepare() {
	fboMultiSample = Framebuffer::createMultiSampleFbo(WIDTH, HEIGHT, 4);
	fboResolve = new Framebuffer(WIDTH, HEIGHT);

	...
	//pass 01
	...
	//pass 02 postProcessPass:后处理pass
	auto sgeo = Geometry::createScreenPlane();
	auto smat = new ScreenMaterial();
	smat->mScreenTexture = fboResolve->mColorAttachment;
	auto smesh = new Mesh(sgeo, smat);
	scene->addChild(smesh);
	...
}
```
最后最重要的在这一步，
第一次离屏渲染，我们用的是`fboMultiSample->mFBO`进行超采样
然后调用`renderer->msaaResolve`函数，把超采样得到的颜色xia'cai'y
```cpp
int main() {
	...
	while (glApp->update()) {
		cameraControl->update();

		renderer->setClearColor(clearColor);
		renderer->render(sceneOff, camera, pointLight, ambLight, fboMultiSample->mFBO);
		renderer->msaaResolve(fboMultiSample, fboResolve);
		renderer->render(scene, camera, pointLight, ambLight);

		renderIMGUI();
	}
	...
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg0NzUyMTk1MiwtODAwNjQ4NTg0LDcyMz
Q1NTQxMCwtMTM2MTE1ODM1MiwxMTkyNzc5NjA2LDExODQ3ODk5
MjgsOTk1NDQxMTQ2LC0xNjcxMjc0NDE3LC0xNjAxNDUyNjQ2LD
ExMDcyMjc2MDksLTEwNzA0ODI2MDldfQ==
-->