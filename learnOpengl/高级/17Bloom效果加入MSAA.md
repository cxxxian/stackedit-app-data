在`framebuffer.h`设计一个用来专门针对`MSAA`版的`HDR`
```cpp
static Framebuffer* createMultiSampleHDRFbo(unsigned int width, unsigned int height, unsigned int samples = 4);
```
与普通的`createMultiSampleFbo`就是改变在
`auto colorAttachment = Texture::createMultiSampleTexture(width, height, samples, GL_RGB16F, 0);`这里使用的是`GL_RGB16F`
```cpp
Framebuffer* Framebuffer::createMultiSampleHDRFbo(unsigned int width, unsigned int height, unsigned int samples)
{
	Framebuffer* fb = new Framebuffer();
	unsigned int fbo;
	glGenFramebuffers(1, &fbo);
	glBindFramebuffer(GL_FRAMEBUFFER, fbo);

	auto colorAttachment = Texture::createMultiSampleTexture(width, height, samples, GL_RGB16F, 0);
	auto dsAttachment = Texture::createMultiSampleTexture(width, height, samples, GL_DEPTH24_STENCIL8, 0);

	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D_MULTISAMPLE, colorAttachment->getTexture(), 0);
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_TEXTURE_2D_MULTISAMPLE, dsAttachment->getTexture(), 0);

	glBindFramebuffer(GL_FRAMEBUFFER, 0);
	fb->mFBO = fbo;
	fb->mColorAttachment = colorAttachment;
	fb->mDepthStencilAttachment = dsAttachment;
	fb->mWidth = width;
	fb->mHeight = height;

	return fb;
}
```

然后到`main.cpp`中，创建相应的`fbo`并进行渲染
会发现，渲染出来的屏幕一片黑
无论只单纯运行`MSAA`或者`bloom`都没事，合并就出问题。
```cpp
Framebuffer* fboMulti = nullptr;
Framebuffer* fboResolve = nullptr;

void prepare() {

	fboMulti = Framebuffer::createMultiSampleHDRFbo(WIDTH, HEIGHT);
	fboResolve = Framebuffer::createHDRFbo(WIDTH, HEIGHT);

	...
}
int main() {
	...
	while (glApp->update()) {
		cameraControl->update();

		renderer->setClearColor(clearColor);
		renderer->render(sceneOff, camera, pointLight, ambLight, fboMulti->mFBO);
		renderer->msaaResolve(fboMulti, fboResolve);
		bloom->doBloom(fboResolve);
		renderer->render(scene, camera, pointLight, ambLight);

		renderIMGUI();
	}
	...
}
```

解析`BUG`：
1 由于`fboResolve`这个对象自创建以来，一次都没有清理过`DepthBuffer`，所以他的深度附件永远都充满了`0`；
2 由于其深度缓存内部都是`0`，那么对其进行绘制的时候，所有像素都无法通过深度检测

所以我们在进行合并渲染时，原本只清理的颜色信息，因为我们认为`bloom`不会有深度相关的操作，但是现在我们要多清理一下深度信息
```cpp
void Bloom::merge(Framebuffer* target, Framebuffer* origin, Framebuffer* bloom)
{
	glBindFramebuffer(GL_FRAMEBUFFER, target->mFBO);

	glViewport(0, 0, target->mWidth, target->mHeight);
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

	...
}

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU3NjQ3NzU1MCwtMTc0NjU3MjEwNywtMT
g5NjM5NTE1LDE5MDY5NTg1MzYsLTE3NjM0NzM4NTMsLTE0Nzgy
OTI4MzAsLTEzMjM3OTMwNzFdfQ==
-->