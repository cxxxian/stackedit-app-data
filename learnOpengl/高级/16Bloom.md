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
以及声明一个变量`mThreshold`用来作为`shader`中阈值的调整
```cpp
#include "../geometry.h"
class Bloom {
private:
	...
	Geometry* mQuad{ nullptr };
	float mThreshold{ 1.0f };
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
`extractBright.frag`如下，
用来找寻传入的`srcTex`纹理中超过阈值`threshold`的部分，也就是比较亮的地方
如果超过我们就输出`color`，否则就是直接输出黑色`vec4(0.0, 0.0, 0.0, 1.0)`
```glsl
#version 460 core
out vec4 FragColor;

in vec2 uv;

uniform sampler2D srcTex;

uniform float threshold;

void main()
{
	vec3 color = texture(srcTex, uv).rgb;

	float brightness = dot(color, vec3(0.2126, 0.7152, 0.0722));

	if(brightness > threshold){
		FragColor = vec4(color, 1.0);
	}else{
		FragColor = vec4(0.0, 0.0, 0.0, 1.0);		
	}
}
```

并且因为这个`shader`只可能用在`bloom`上面，所以我们直接声明在`bloom.h`当中，而不是像以前一样声明在`renderer.h`当中
```cpp
#include "../shader.h"
class Bloom {
private:
	...
	Shader* mExtractBrightShader{ nullptr };
```
在`bloom`的构造函数中初始化
```cpp
#include "bloom.h"
Bloom::Bloom(int width, int height, int minResolution = 32)
{
	...
	mQuad = Geometry::createScreenPlane();
	mExtractBrightShader = new Shader("assets/shaders/advanced/bloom/extractBright.vert", "assets/shaders/advanced/bloom/extractBright.frag")
}
```
## 3 编写extractBright函数，用来提取对应FBO的亮度
在`bloom.h`声明函数`extractBright`并实现
在一开始，我们需要绑定`glBindFramebuffer(GL_FRAMEBUFFER, dst->mFBO);`，
说明我们此时绘制的目标是`dst->mFBO`
接下来就是将视口调整为`glViewport(0, 0, dst->mWidth, dst->mHeight)`，这里是出于安全性的考量，防止我们在调用这个方法之前，如果修改过`viewport`的大下导致和屏幕尺寸不对应，就会有问题

接下来就是读取`shader`，并向`shader`中传递`uniform`变量以及`vao`
```cpp
void Bloom::extractBright(Framebuffer* src, Framebuffer* dst)
{
	glBindFramebuffer(GL_FRAMEBUFFER, dst->mFBO);

	glViewport(0, 0, dst->mWidth, dst->mHeight);
	glClear(GL_COLOR_BUFFER_BIT);

	mExtractBrightShader->begin();
	{
		auto srcTex = src->mColorAttachment;
		srcTex->setUnit(0);
		srcTex->bind();
		mExtractBrightShader->setInt("srcTex", 0);

		mExtractBrightShader->setFloat("threshold", mThreshold);

		glBindVertexArray(mQuad->getVao());
		glDrawElements(GL_TRIANGLES, mQuad->getIndicesCount(), GL_UNSIGNED_INT, 0);
	}
	mExtractBrightShader->end();
}

```

# 三、 编写下采样函数
就直接用`glBlitFramebuffer`进行编写即可
根据读入的`src->mFBO`，写到`dst->mFBO`即可，写入的是`GL_COLOR_BUFFER_BIT`即颜色信息，采用`GL_LINEAR`方式
```cpp

void Bloom::downSample(Framebuffer* src, Framebuffer* dst)
{
	glBindFramebuffer(GL_READ_FRAMEBUFFER, src->mFBO);
	glBindFramebuffer(GL_DRAW_FRAMEBUFFER, dst->mFBO);

	glBlitFramebuffer(0, 0, src->mWidth, src->mHeight, 0, 0, dst->mWidth, dst->mHeight, GL_COLOR_BUFFER_BIT, GL_LINEAR);
}
```

# 四、编写上采样函数
上采样我们就不能像下采样那样简单做了，我们要引入`poisson`模糊，效果更好

![输入图片说明](/imgs/2025-03-05/yhCihKjmoMEgQYAK.png)

![输入图片说明](/imgs/2025-03-05/DswyvpydLLp5apLC.png)

## 1 编写上采样的shader：upSample.vert/frag
`upSample.vert`就是很普通的得到`pos`和`uv`信息
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
`upSample.frag`如下，这里引入之前制作的`poisson`采样函数
`bloomRadius`用来控制采样的半径距离

`bloomAttenuation`用来控制辉光效果的剧烈程度，越小辉光周围蔓延就会越小，越大的话周围亮度朦胧程度就会越大
`lowerResColor * bloomAttenuation`，`lowerResColor`被上采样得到的`texture`的颜色，主要影响的是外面一圈朦胧的效果
```glsl
#version 460 core
out vec4 FragColor;

in vec2 uv;
uniform sampler2D lowerResTex;//被上采样的低分辨率图像
uniform sampler2D higherResTex;//来自下采样链条，与当前绘制目标大小相同的纹理

#define NUM_SAMPLES 32
#define PI 3.141592653589793
#define PI2 6.283185307179586

float rand_2to1(vec2 uv ) { 
  ...
}
uniform float diskTightness;
vec2 disk[NUM_SAMPLES];
void poissonDiskSamples(vec2 randomSeed){
	...
}

uniform float bloomRadius;

vec3 doPoissonSample(sampler2D tex, vec2 uv){
	poissonDiskSamples(uv);
	 
	vec3 sumColor = vec3(0.0);
	for(int i = 0; i < NUM_SAMPLES; i++){
		sumColor += texture(tex, uv + disk[i] * bloomRadius).rgb;
	}
	return sumColor / NUM_SAMPLES;
}

uniform float bloomAttenuation;

void main()
{
	vec3 lowerResColor = doPoissonSample(lowerResTex, uv);
	vec3 higherResColor = doPoissonSample(higherResTex, uv);

	vec3 color = higherResColor + lowerResColor * bloomAttenuation;

	FragColor = vec4(color, 1.0);
}
```
准备好以上`shader`，我们就到`bloom.h`声明变量`mUpSampleShader`并初始化
```cpp
Bloom::Bloom(int width, int height, int minResolution = 32)
{
	...
	mUpSampleShader = new Shader("assets/shaders/advanced/bloom/upSample.vert", "assets/shaders/advanced/bloom/upSample.frag");
}
```
## 2 编写上采样的函数
先在`bloom.h`创建两个`shader`中对应的`uniform`变量
```cpp
float mBloomRadius{ 0.1f };
float mBloomAttenuation{ 1.0f };
```
然后在`bloom.h`定义一个函数并实现
传入`shader`中对应的变量即可
```cpp
void Bloom::upSample(Framebuffer* target, Framebuffer* lowerResFbo, Framebuffer* higherResFbo)
{
	glBindFramebuffer(GL_FRAMEBUFFER, target->mFBO);

	glViewport(0, 0, target->mWidth, target->mHeight);
	glClear(GL_COLOR_BUFFER_BIT);

	mUpSampleShader->begin();
	{
		lowerResFbo->mColorAttachment->setUnit(0);
		lowerResFbo->mColorAttachment->bind();
		mUpSampleShader->setInt("lowerResTex", 0);

		higherResFbo->mColorAttachment->setUnit(1);
		higherResFbo->mColorAttachment->bind();
		mUpSampleShader->setInt("higherResTex", 1);

		mUpSampleShader->setFloat("bloomRadius", mBloomRadius);
		mUpSampleShader->setFloat("bloomAttenuation", mBloomAttenuation);

		glBindVertexArray(mQuad->getVao());
		glDrawElements(GL_TRIANGLES, mQuad->getIndicesCount(), GL_UNSIGNED_INT, 0);
	}
	mUpSampleShader->end();
}

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE3NTg4OTUyOSwxNjkyOTE4MTc5LC0xMj
gyOTEyMTU2LDE1NzIyOTY3OTcsLTE0NTI4NTg1NzgsMTY3MjA1
OTgwMywyMDEzOTc0MTAwLC0xODg2NDU1NzczLC0yMjI2NDY1NT
csNzk0Mjk3MjEsMTAxNzc4NTUxMSwxOTE3NzkyOTczLC02ODcy
MDMzOTUsMjk0ODM3MDcyLDc2NDg2MDg2MywtMTk3Mjk0MjU3Ni
wtMTUwOTA2Mjg4NCwxMjA4MTk4MTUxXX0=
-->