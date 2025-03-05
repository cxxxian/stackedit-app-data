# 概念

![输入图片说明](/imgs/2025-03-05/Wi1Krth0HeFpxpOI.png)

![输入图片说明](/imgs/2025-03-05/WwxDgu0Tdx5ZnowX.png)

`GL_RGBA`一共占`32`位，每个通道占`8bit`，即`1byte`
`GL_RGBA16F`每个通道占`16bit`
`GL_RGBA32F`每个通道占`32bit`

`format`是外界给入的数据的格式
`internalFormat`是把外界给的数据存储到`internalFormat`这种格式

![输入图片说明](/imgs/2025-03-05/owkZcV9qniGdemWq.png)

![输入图片说明](/imgs/2025-03-05/1lZ2X7IJcZOmNzPD.png)

![输入图片说明](/imgs/2025-03-05/SdaXFIhJ7sAxsEfg.png)

![输入图片说明](/imgs/2025-03-05/uJUqnnkzhrcRjiCz.png)

`exposure`大的话，颜色分配亮的部分就会比较多
`exposure`小的话，颜色分配亮暗就会比较平均
以下是`exposure = 5`的情况

![输入图片说明](/imgs/2025-03-05/ZdNE79o6jitm2JY8.png)

以下是`exposure = 1`的情况

![输入图片说明](/imgs/2025-03-05/hDCD2Ja3lbxCujlk.png)

# 实践
## 1 创建HDR专用纹理
```cpp
Texture* Texture::createHDRAttachment(unsigned int width, unsigned int height, unsigned int unit)
{
	Texture* tex = new Texture();

	GLuint glTex;
	glGenTextures(1, &glTex);
	glBindTexture(GL_TEXTURE_2D, glTex);

	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB16F, width, height, 0, GL_RGB, GL_FLOAT, NULL);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);//u
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);//v

	tex->mTexture = glTex;
	tex->mWidth = width;
	tex->mHeight = height;
	tex->mUnit = unit;
	tex->mTextureTarget = GL_TEXTURE_2D;

	return tex;
}
```
## 2 创建HDR专用FBO
其实和之前创建的`fbo`区别就是，颜色的`attachment`用`createHDRAttachment(width, height, 0)`创建
```cpp
Framebuffer* Framebuffer::createHDRFbo(unsigned int width, unsigned int height)
{
	Framebuffer* fb = new Framebuffer();
	unsigned int fbo;
	glGenFramebuffers(1, &fbo);
	glBindFramebuffer(GL_FRAMEBUFFER, fbo);

	auto colorAttachment = Texture::createHDRAttachment(width, height, 0);
	auto dsAttachment = Texture::createDepthStencilAttachment(width, height, 0);

	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, colorAttachment->getTexture(), 0);
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_TEXTURE_2D, dsAttachment->getTexture(), 0);

	glBindFramebuffer(GL_FRAMEBUFFER, 0);
	fb->mFBO = fbo;
	fb->mColorAttachment = colorAttachment;
	fb->mDepthAttachment = dsAttachment;
	fb->mWidth = width;
	fb->mHeight = height;

	return fb;
}
```
## 3 screen.frag/vert的更改与uniform
```glsl
vec3 toneMappingReinhard(vec3 hdrColor){
	return hdrColor / (hdrColor + vec3(1.0));
}

uniform float exposure;
vec3 toneMappingExposure(vec3 hdrColor){
	return (vec3(1.0) - exp(-hdrColor * exposure));
}

void main()
{
	...
	vec3 color = texture(screenTexSampler, uv).rgb;
	color = toneMappingReinhard(color);

	//2 最终颜色要抵抗屏幕gamma
	color = pow(color, vec3(1.0/2.2));

	FragColor = vec4(color, 1.0);
}
```
在对应`screenMaterial.h`创建对应变量
```cpp
float mExposure{ 1.0f };
```
## 4 绘制流程更改
在`renderer.cpp`中的`renderObject`方法中将变量传输到`shader`当中
```cpp
case MaterialType::WhiteMaterial: {
	...
	shader->setFloat("exposure", screenMat->mExposure);
}
	break;
```
在`imgui`加入`exposure`的调控
```cpp
ImGui::SliderFloat("exposure:", &screenMat->mExposure, 0.0f, 3.0f);
```
## 自己小研究：
在`screen.frag`设计一个`uniform`变量`k`，
用于选择不同的`HDR`处理方式
```glsl
uniform int k;
void main()
{
	vec3 color = texture(screenTexSampler, uv).rgb;
	switch(k){
	case 0:
		color = color;
		break;
	case 1:
		color = toneMappingReinhard(color);
		break;
	case 2:
		color = toneMappingExposure(color);
		break;
	}
	//2 最终颜色要抵抗屏幕gamma
	color = pow(color, vec3(1.0/2.2));

	FragColor = vec4(color, 1.0);
}
```
然后在`screenMaterial.h`设计一个变量`k`
```cpp
int k{ 0 };
```
加入渲染流程传到`shader`中
```cpp
shader->setFloat("exposure", screenMat->mExposure);
shader->setInt("k", screenMat->k);
```
然后再到`imgui`中添加调控
```cpp
ImGui::InputInt("HDRWays:", &screenMat->k);
```

对应没有任何处理，即超过`1`被截断了直接输出`1`亮度

![输入图片说明](/imgs/2025-03-05/30E2fJNsgVAY6fb5.png)

对应`toneMappingReinhard`

![输入图片说明](/imgs/2025-03-05/hKr5Ac0Qrow2QXkF.png)

对应`toneMappingExposure`
可以通过调整`exposure`的值，
越小越暗，越大越亮

![输入图片说明](/imgs/2025-03-05/QokQFJ1XVKA3v22F.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMjc0OTkwMywxNTExOTUwNzAwLC0xNT
I1OTI4MzE1LC01MTkwNjc1NzgsMjEzMDA5MTI0NiwtMTQ2MjU0
MTMwOCwtNTE3OTA3NDk2LC0yMDg4NzQ2NjEyXX0=
-->