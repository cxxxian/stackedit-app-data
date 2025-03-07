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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3NDY1NzIxMDcsLTE4OTYzOTUxNSwxOT
A2OTU4NTM2LC0xNzYzNDczODUzLC0xNDc4MjkyODMwLC0xMzIz
NzkzMDcxXX0=
-->