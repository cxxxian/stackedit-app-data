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

	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, colorAttachment->getTexture(), 0);

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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTkwNjk1ODUzNiwtMTc2MzQ3Mzg1MywtMT
Q3ODI5MjgzMCwtMTMyMzc5MzA3MV19
-->