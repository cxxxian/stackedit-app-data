![输入图片说明](/imgs/2025-04-08/06LhZMKZrXVp9RaD.png)

![输入图片说明](/imgs/2025-04-08/50awnfEQeSWu34EQ.png)

![输入图片说明](/imgs/2025-04-08/GyIHS2iXOsgRqOO6.png)

## 黎曼和

![输入图片说明](/imgs/2025-04-08/oS39XNvuY7mJbTRN.png)

![输入图片说明](/imgs/2025-04-08/5wJpcHKCBclPVSm2.png)

## 积分计算（先考虑diffuse）

**对内层和外层积分分别使用两次黎曼和**

![输入图片说明](/imgs/2025-04-08/E9Xqw1Ebus4RPh6y.png)

![输入图片说明](/imgs/2025-04-08/P9oEDfT9PgM7K6tq.png)

![输入图片说明](/imgs/2025-04-08/SV1exJqcS5fqUa9y.png)

![输入图片说明](/imgs/2025-04-08/c8f3ERlwJXefgF8B.png)

## 向量空间转换
把法线坐标系下的`P`乘上法线坐标系构成的矩阵，即可得到`P`在世界坐标系下的值

![输入图片说明](/imgs/2025-04-08/XeIErVIM2w3CKoIc.png)

![输入图片说明](/imgs/2025-04-08/PJsw238lPdd6ETvQ.png)

对应着公式看程序
`numSamples`循环结束后就会变成`n1 * n2`，
`localVec`是采样点在法线坐标系下的位置
然后我们利用法线坐标系矩阵将`localVec`转到世界坐标系下

![输入图片说明](/imgs/2025-04-08/iJzI5MwNxsH3Q6q6.png)

做预积分，把半球的与动态参数无关的积分结果存在`IBL`贴图

![输入图片说明](/imgs/2025-04-08/Y3xPs3ic2wj9KwtI.png)

# 实现预积分工具编写
## 1 清理代码，只渲染一次即可
`render.h`全部删光了，声明一个方法`renderIBLDiffuse`用来渲染预积分贴图
```cpp
class Renderer {
public:
	Renderer();
	~Renderer();

	void renderIBLDiffuse(Texture* hdrTex, Framebuffer* fbo);

	void setClearColor(glm::vec3 color);

private:
	Shader* mIBLDiffuseShader{ nullptr };
};
```
`main.cpp`也基本上删光了
声明一个`doRender`方法，用来将传入的环境贴图进行预积分
以前所有的渲染逻辑都删掉
```cpp
Renderer* renderer = nullptr;

Framebuffer* fbo = nullptr;

int WIDTH = 128;
int HEIGHT = 128;

glm::vec3 clearColor{};

void doRender(std::string path) {

}
int main() {
	if (!glApp->init(WIDTH, HEIGHT)) {
		return -1;
	}
	//设置opengl视口以及清理颜色
	GL_CALL(glViewport(0, 0, WIDTH, HEIGHT));
	GL_CALL(glClearColor(0.0f, 0.0f, 0.0f, 1.0f));
	
	doRender("xxx");
	glApp->destroy();
	return 0;
}
```

## 2 引入TinyExr，用于读取EXR的HDR贴图
此处专门说明一下这个`EXR`格式的图片，内部存储的亮度是可以超过一的，所以是`HDR`贴图，用这个比较好

在`application`目录下引入`tinyexr.h`，`tinyexr.cc`两个文件，
然后需要`application`目录下`cmakeList`添加配置，因为以前没有`.cc`文件
```cmake
#递归将本文件夹下所有cpp
file(GLOB_RECURSE APP ./  *.cpp *.cc)
add_library(app ${APP} )
```
还需要引入一个`miniz`文件夹，配合`tinyexr.h`，`tinyexr.cc`使用
要使用这个文件夹，就得去最外层的`cmakeList`添加文件配置
```
include_directories(
	SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/thirdParty/include
	SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/application/miniz
)
```
这样就可以编译成功了

## 3 纹理类新增工具函数
### 3.1 createExrTexture
这里运用了新的库，`TINYEXR_IMPLEMENTATION`表示启用
正常使用`tiny`库进行图片的读取，不同的地方在于，`tiny`库并没有内置图片旋转的方法，而一般图片的原点在左上角，`OpenGL`是以左下角建立坐标系的
所以我们要手动实现反转
就是把上下一行一行对调即可
```cpp
#define TINYEXR_IMPLEMENTATION
#include "../application/tinyexr.h"

Texture* Texture::createExrTexture(const std::string& path)
{
	Texture* tex = new Texture();
	//1 数据读取
	float* data = nullptr;
	int width, height;
	//有报错的话装给err
	const char* err = nullptr;
	//成功与否的信号
	int ret;

	ret = LoadEXR(&data, &width, &height, path.c_str(), &err);
	if (ret != TINYEXR_SUCCESS) {
		if (err) {
			std::cerr << "Error Loading Exr:" << err << std::endl;
			FreeEXRErrorMessage(err);
		}
		return nullptr;
	}
	//y方向进行反向，因为tiny库没有反向的函数，我们要自己写
	int channels = 4;//这里的channels可以去看LoadEXR方法的返回值，无论输入的格式是什么输出的data一定是rgba格式的
	for (int y = 0; y < height / 2; ++y) {
		int opposite_y = height - 1 - y;
		for (int x = 0; x < width * channels; x++) {
			std::swap(data[width * channels * y + x], data[width * channels * opposite_y + x]);
		}
	}

	//2 生成纹理
	GLuint glTex;
	glGenTextures(1, &glTex);
	glBindTexture(GL_TEXTURE_2D, glTex);

	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB16F, width, height, 0, GL_RGBA, GL_FLOAT, data);

	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);

	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_BORDER);//u
	glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_BORDER);//v

	tex->mTexture = glTex;
	tex->mWidth = width;
	tex->mHeight = height;
	tex->mUnit = 0;

	return tex;
}
```
### 3.2 createHDRCubeMap
这里`glTexImage2D`最后是`nullptr`，是因为我们暂时只是开辟空间，还没传数据
```cpp
Texture* Texture::createHDRCubeMap(int width, int height)
{
	Texture* tex = new Texture();

	GLuint glTex;
	glGenTextures(1, &glTex);
	glBindTexture(GL_TEXTURE_CUBE_MAP, glTex);

	for (int i = 0; i < 6; i++) {
		glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i,
			0, GL_RGB16F,
			width, height, 0,
			GL_RGB, GL_FLOAT, nullptr);
	}
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);

	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_BORDER);//s
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_BORDER);//t
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_BORDER);//r

	glBindTexture(GL_TEXTURE_CUBE_MAP, 0);

	tex->mTexture = glTex;
	tex->mWidth = width;
	tex->mHeight = height;
	tex->mUnit = 0;
	tex->mTextureTarget = GL_TEXTURE_CUBE_MAP;

	return tex;
}
```

## FBO类新增工具函数
### 3.3 createHDRCubeMapFBO
```cpp
Framebuffer* Framebuffer::createCubeMapHDRFbo(int width, int height)
{
	Framebuffer* fb = new Framebuffer();
	unsigned int fbo;
	glGenFramebuffers(1, &fbo);
	glBindFramebuffer(GL_FRAMEBUFFER, fbo);

	Texture* colorAttachment = Texture::createHDRCubeMap(width, height);
	glFramebufferTexture2D(
		GL_FRAMEBUFFER,
		GL_COLOR_ATTACHMENT0,
		GL_TEXTURE_CUBE_MAP_POSITIVE_X,
		colorAttachment->getTexture(),
		0
	);

	glBindFramebuffer(GL_FRAMEBUFFER, 0);
	fb->mFBO = fbo;
	fb->mColorAttachment = colorAttachment;
	fb->mWidth = width;
	fb->mHeight = height;

	return fb;
}
```

## 4 预积分shader代码编写
`iblDiffuse.vert`如下：
```glsl
#version 460
layout(location = 0) in vec3 aPos;
out vec3 uvw;

uniform mat4 viewMatrix;
uniform mat4 projectionMatrix;

void main(){
	vec4 transformPosition = vec4(aPos, 1.0);
	gl_Position = projectionMatrix * viewMatrix * transformPosition;

	uvw = normalize(aPos);
}
```
`iblDiffuse.frag`如下：

![输入图片说明](/imgs/2025-04-08/Y3xPs3ic2wj9KwtI.png)

对应着公式来
```cpp
#version 460

out vec4 FragColor;
in vec3 uvw;

uniform sampler2D envMap;

const float PI = 3.1415926535897932384626433832795;

vec2  uvwToUv(vec3 uvwN){
	float  phi = asin(uvwN.y);
	float theta = atan(uvwN.z, uvwN.x);
	float u = theta / (2.0 * PI) + 0.5;
	float v = phi / PI + 0.5;
	return vec2(u, v);
}

void main(){
	vec3 normal = normalize(uvw);
	vec3 irradiance = vec3(0.0);

	//准备以法线为z轴的本地坐标系
	vec3 up = vec3(0.0, 1.0, 0.0);
	vec3 right = normalize(cross(up, normal));
	up = normalize(cross(normal, right));

	float delta = 0.025;
	float sampleNum = 0.0;
	for(float phi = 0.0; phi < 2.0 * PI; phi += delta){
		for(float theta = 0.0; theta < 0.5 * PI; theta += delta){
			//1 极坐标转化为采样的local向量
			vec3 localVec = vec3(sin(theta) * cos(phi), sin(theta) * sin(phi), cos(theta));

			//2 local向量转化为world坐标系中
			vec3 worldVec = localVec.x * right + localVec.y * up + localVec.z * normal;

			//3 通过world向量采样球面贴图
			vec2 uv = uvwToUv(normalize(worldVec));
			vec3 Li = texture(envMap, uv).rgb;

			//4 将Radiance通过公式转化为Irradiance累加到变量上
			irradiance += Li * cos(theta) * sin(theta);
			

			sampleNum++;
		}
	}

	irradiance = PI * irradiance * 1.0 / sampleNum;
	FragColor = vec4(irradiance, 1.0);
}
```

## 5 编写执行代码
### 5.1 读取球面贴图，制作FBO
`main.cpp`中的`doRender`方法如下，我们读取贴图制作材质，然后创建一个`fbo`，然后送到`renderer`那边展开渲染工作
```cpp
void doRender(std::string path) {
	//1 读取exr的球面环境贴图
	auto exrTex = Texture::createExrTexture(path);

	//2 创建带有cubemapColorAttachment的FBO
	auto fbo = Framebuffer::createCubeMapHDRFbo(WIDTH, HEIGHT);

	//3 进行渲染工作
	renderer = new Renderer();
	renderer->renderIBLDiffuse(exrTex, fbo);

}
```
最后在`main`方法中调用，我们只需要渲染一次，不用像以前一样一直渲染
```cpp
int main() {
	if (!glApp->init(WIDTH, HEIGHT)) {
		return -1;
	}


	//设置opengl视口以及清理颜色
	GL_CALL(glViewport(0, 0, WIDTH, HEIGHT));
	GL_CALL(glClearColor(0.0f, 0.0f, 0.0f, 1.0f));
	
	doRender("assets/textures/ibl/warm_bar_4k.exr");

	glApp->destroy();

	return 0;
}
```
### 5.2 渲染六个CubeMap的面，并且导出EXR图片
完善`renderer`中的方法
此处的相机矩阵（`viewMatrix`）尊崇`OpenGL`给的规范设计，在点光源笔记有详细解释
最后依旧是利用`tiny`库去实现导出`exr`图片到我们的文件中
```cpp
void Renderer::renderIBLDiffuse(Texture* hdrTex, Framebuffer* fbo)
{
	//rgb三个通道所以乘3
	std::vector<float> pixels(fbo->mWidth * fbo->mHeight * 3);

	//1 准备渲染cubeMap六个面所需要的投影+相机矩阵
	auto projection = glm::perspective(glm::radians(90.0f), 1.0f, 0.1f, 1000.0f);
	glm::mat4 views[] = {
	   glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(1.0f,  0.0f,  0.0f), glm::vec3(0.0f, -1.0f,  0.0f)),
	   glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(-1.0f,  0.0f,  0.0f), glm::vec3(0.0f, -1.0f,  0.0f)),
	   glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f,  1.0f,  0.0f), glm::vec3(0.0f,  0.0f,  1.0f)),
	   glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, -1.0f,  0.0f), glm::vec3(0.0f,  0.0f, -1.0f)),
	   glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f,  0.0f,  1.0f), glm::vec3(0.0f, -1.0f,  0.0f)),
	   glm::lookAt(glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f,  0.0f, -1.0f), glm::vec3(0.0f, -1.0f,  0.0f))
	};
	//2 开始渲染
	glBindFramebuffer(GL_FRAMEBUFFER, fbo->mFBO);
	glViewport(0, 0, fbo->mWidth, fbo->mHeight);
	//shader需要传输vao
	auto geometry = Geometry::createBox(1.0f);
	glBindVertexArray(geometry->getVao());

	mIBLDiffuseShader->begin();
	hdrTex->setUnit(0);
	hdrTex->bind();
	mIBLDiffuseShader->setInt("envMap", 0);

	mIBLDiffuseShader->setMatrix4x4("projectionMatrix", projection);

	for (int i = 0; i < 6; i++) {
		glFramebufferTexture2D(
			GL_FRAMEBUFFER,
			GL_COLOR_ATTACHMENT0,
			GL_TEXTURE_CUBE_MAP_POSITIVE_X + i,
			fbo->mColorAttachment->getTexture(),
			0
		);
		glClear(GL_COLOR_BUFFER_BIT);
		mIBLDiffuseShader->setMatrix4x4("viewMatrix", views[i]);

		glDrawElements(GL_TRIANGLES, geometry->getIndicesCount(), GL_UNSIGNED_INT, 0);

		//渲染完毕一个面之后，立刻输出为图片
		glReadPixels(0, 0, fbo->mWidth, fbo->mHeight, GL_RGB, GL_FLOAT, pixels.data());

		//读取到的像素数据写入到文件当中
		const char* err = nullptr;
		SaveEXR(pixels.data(), fbo->mWidth, fbo->mHeight, 3, 1, ("env_" + std::to_string(i) + ".exr").c_str(), &err);
	}
	mIBLDiffuseShader->end();
}
```


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxNzM2MDk5NCwtMTI0NTQ5NjI0OSwxMj
c5MTUyMDc1XX0=
-->