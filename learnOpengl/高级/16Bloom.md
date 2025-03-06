# 概念
辉光效果，亮到有点朦胧感

![输入图片说明](/imgs/2025-03-05/19E8GAhmZbG9b1ZJ.png)

![输入图片说明](/imgs/2025-03-05/W3qAfbhg9LA8ejEk.png)

![输入图片说明](/imgs/2025-03-05/slRBC1ZfWoJsu1ss.png)

![输入图片说明](/imgs/2025-03-05/DMw90Rym202hqm3l.png)

![输入图片说明](/imgs/2025-03-05/SlS8r1j4y6GpO5mZ.png)

![输入图片说明](/imgs/2025-03-05/yhCihKjmoMEgQYAK.png)

![输入图片说明](/imgs/2025-03-05/DswyvpydLLp5apLC.png)

![输入图片说明](/imgs/2025-03-05/i69ZauxWGm1crciL.png)

![输入图片说明](/imgs/2025-03-05/gO2uQmVNNxnPAMhK.png)

# 实现方式解析

![输入图片说明](/imgs/2025-03-05/QAyI6USOoz6rY7s4.png)

![输入图片说明](/imgs/2025-03-05/GotyZblQOrTGgjqx.png)

下采样完之后我们进行上采样，上采样同样可以用`glBlitFrameBuffer`，但是我们自定义一个`poisson`采样效果更好

![输入图片说明](/imgs/2025-03-05/g6vfUxsWRPHpIs7a.png)

![输入图片说明](/imgs/2025-03-05/I1q3SwtUQB0xAwBY.png)

# 实践
## 1 FrameBuffer类增加HDR下Bloom专用的fbo
特点：只需要颜色`attachment`，因为是`screenSpace`的绘制，所以不用进行深度检测什么七七八八的
相比普通的`createHDRFbo`，就是把深度相关的`attachment`删去
```cpp
Framebuffer* Framebuffer::createHDRBloomFbo(unsigned int width, unsigned int height)
{
	Framebuffer* fb = new Framebuffer();
	unsigned int fbo;
	glGenFramebuffers(1, &fbo);
	glBindFramebuffer(GL_FRAMEBUFFER, fbo);

	auto colorAttachment = Texture::createHDRAttachment(width, height, 0);

	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, colorAttachment->getTexture(), 0);

	glBindFramebuffer(GL_FRAMEBUFFER, 0);
	fb->mFBO = fbo;
	fb->mColorAttachment = colorAttachment;
	fb->mWidth = width;
	fb->mHeight = height;

	return fb;
}

```

## 2 创建Bloom类（初步）
- 下采样`FBO`数组
- 上采样`FBO`数组
- 初始化宽度高度，有多少级的下采样图片数量
```cpp
#pragma once
#include "../core.h"
#include "../framebuffer/framebuffer.h"

class Bloom {
public:
	Bloom();
	~Bloom();

private:
	//下/上 采样FBO数组
	std::vector<Framebuffer*> mDownSamples{};
	std::vector<Framebuffer*> mUpSamples{};

	//初始化宽度高度，有多少级的下采样图片数量
	int mWidth{ 0 };
	int mHeight{ 0 };
	int mMipLevels{ 0 };
};
```
## 编写Bloom构造函数（按需创建上下采样的fbo数组）
计算每次下采样要缩小

![输入图片说明](/imgs/2025-03-06/YlAKJPg0j5O2dNxw.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYyMTQ3ODY1OSw3NjQ4NjA4NjMsLTE5Nz
I5NDI1NzYsLTE1MDkwNjI4ODQsMTIwODE5ODE1MV19
-->