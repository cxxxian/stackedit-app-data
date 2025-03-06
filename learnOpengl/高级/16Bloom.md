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

# 一、架构与初始化
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
计算每次下采样要进行多少次，按照下面的推导来：

![输入图片说明](/imgs/2025-03-06/YlAKJPg0j5O2dNxw.png)

注意此处有一个小重点，就是我们最后达到最小层级之后，还会再运行一次`w / 2`和`h / 2`，所以其实我们最后一张采样图片的大小是`2w, 2h`，
倒数第二张是`4w, 4h`，
```cpp
Bloom::Bloom(int width, int height, int minResolution = 32)
{
	mWidth = width;
	mHeight = height;

	float widthLevels = std::log2((float)width / (float)minResolution);
	float heightLevels = std::log2((float)height / (float)minResolution);

	mMipLevels = std::min(widthLevels, heightLevels);

	int w = mWidth, h = mHeight;
	for (int i = 0; i < mMipLevels; i++) {
		mDownSamples.push_back(Framebuffer::createHDRBloomFbo(w, h));
		w /= 2;
		h /= 2;
	}
	w = 4 * w, h = 4 * h;
	for (int i = 0; i < mMipLevels - 1; i++) {
		mDownSamples.push_back(Framebuffer::createHDRBloomFbo(w, h));
		w *= 2;
		h *= 2;
	}
}
```

# 二、提取高亮
## 1 Bloom类加入screenPlane的geometry（用其中的VAO）
在`bloom.h`中添加一个几何体`mQuad`
```cpp
#include "../geometry.h"
class Bloom {
private:
	...
	Geometry* mQuad{ nullptr };
};
```
并在构造函数中初始话为`screenPlane`
```cpp
Bloom::Bloom(int width, int height, int minResolution = 32)
{
	...
	mQuad = Geometry::createScreenPlane();
}
```
## 2 编写提取高亮用到的shader
制作`extractBright.vert/frag`，从`screen.vert/frag`复制过来
`extractBright.vert`如下，就是简单输出`pos`和`uv`给`frag`
这时候因为直接是设计好的屏幕坐标，即传入进来的时候就已经是`ndc`坐标了，所以不用对`pos`进行任何转化
```glsl
#version 460 core
layout (location = 0) in vec2 aPos;
layout (location = 1) in vec2 aUV;

out vec2 uv;


void main()
{
	gl_Position = vec4(aPos.x, aPos.y, 0.0, 1.0);
	uv = aUV;
}
```
## 3 编写extractBright函数，用来提取对应FBO的亮度
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMwOTE3ODQyNiwxMDE3Nzg1NTExLDE5MT
c3OTI5NzMsLTY4NzIwMzM5NSwyOTQ4MzcwNzIsNzY0ODYwODYz
LC0xOTcyOTQyNTc2LC0xNTA5MDYyODg0LDEyMDgxOTgxNTFdfQ
==
-->