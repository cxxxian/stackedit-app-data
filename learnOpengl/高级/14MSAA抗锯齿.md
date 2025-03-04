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

![输入图片说明](/imgs/2025-03-04/M3fj6hqjoIRxpDEu.png)

# 实践
## 1 创建MSAA专用纹理
在`texture.h`声明函数并实现
步骤上没有大的差别，几个新参数：
`samples`用来指定有多少个采样点
`format`用来指定格式（例如`GL_RGBA`或者`GL_DEPTH24_STENCIL8`）

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
## 3 Renderer中增加msaaResolve函数
## 4 绘制流程更改
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTUxMzI1MTg4NywxMTg0Nzg5OTI4LDk5NT
Q0MTE0NiwtMTY3MTI3NDQxNywtMTYwMTQ1MjY0NiwxMTA3MjI3
NjA5LC0xMDcwNDgyNjA5XX0=
-->