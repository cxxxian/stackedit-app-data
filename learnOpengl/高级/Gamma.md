# 理论
## 图片制作中的Gamma现象

![输入图片说明](/imgs/2025-02-20/A1yXOH1zq4ZvfL4F.png)

通过人眼会觉得上面的灰度图比较均匀，但实际上下面的灰度图才是真正均匀变换的

![输入图片说明](/imgs/2025-02-20/9fHjOACBX7sbBFwe.png)

![输入图片说明](/imgs/2025-02-20/ek3VoXmYMyB8IFb0.png)

![输入图片说明](/imgs/2025-02-20/cHqbi1VatlFS3Y1Y.png)

![输入图片说明](/imgs/2025-02-20/aTA0oXUxrFtmYlsW.png)

这里的`color-srgb`不是减法，时`srgb`空间下的`color`值的意思

## 图片存储的Gamma现象

![输入图片说明](/imgs/2025-02-20/2XzOzac53WpulaD2.png)

![输入图片说明](/imgs/2025-02-20/Zylj3ilFAZ2iE6G6.png)

![输入图片说明](/imgs/2025-02-20/7DeRo2yo5ijHrFab.png)

这张图的意思是，我们从`ps`导出图片时，图片会从`32`位精度转化为`8`位精度，即从`0~1`的`float`转为`255`，
但是我们将图片读取到`OpenGL`中时，又会被转回去`0~1`的`float`类型，因为我们在`OpenGL`需要对图片做一些计算，转化为`0~1`比较方便
此时就会发现丢失精度，比如：
`0.218`和`0.22`都会被转为`56`，`56`再转回来两个就都变成`0.22`了

![输入图片说明](/imgs/2025-02-20/uDqFLhbQl2jOAusP.png)

![输入图片说明](/imgs/2025-02-20/ZGlplixmTgm5KvWz.png)

我们在导出图片时，先做运算转化为`sRGB`空间，这样`32`转化为`8`位存储空间的时候，暗部就不至于丢掉太多细节精度，然后在我们将图片读入`OpenGL`时，再做相应的运算把`sRGB`转化回`RGB`空间

![输入图片说明](/imgs/2025-02-20/I9uhYuLekoPbj64p.png)

![输入图片说明](/imgs/2025-02-20/b45R57UWlAkkR7IK.png)

## 屏幕显示器的Gamma现象

![输入图片说明](/imgs/2025-02-20/8OoTVsUTofkFySlj.png)

![输入图片说明](/imgs/2025-02-20/I65fa7Oravmm7mZK.png)

![输入图片说明](/imgs/2025-02-20/9PTHY9kv6gaF8czf.png)

![输入图片说明](/imgs/2025-02-20/gJwSWBlCBj8NfBrL.png)

![输入图片说明](/imgs/2025-02-20/Jd8EoZQYJj8fNJPv.png)

# 实践
## Gamma矫正实验
我们先构建一个试验场景，就是渲染一张屏幕贴图出来
```cpp
void prepare() {
	...
	auto sgeo = Geometry::createScreenPlane();
	auto smat = new ScreenMaterial();
	smat->mScreenTexture = new Texture("assets/textures/wall.jpg", 0);
	auto smesh = new Mesh(sgeo, smat);
	scene->addChild(smesh);
	...
}
```
运行之后就是很普通的一个屏幕贴图

![输入图片说明](/imgs/2025-02-20/OJligs7XwVFBTXEY.png)

### 1.抵抗屏幕的Gamma，输出颜色之前做1/2.2次方运算
这时候我们去`screen.frag`中将最终颜色要抵抗屏幕`gamma`给加上，其实就是颜色乘上`1/2.2`次方

```glsl
void main()
{
	vec3 color = texture(screenTexSampler, uv).rgb;
	//2 最终颜色要抵抗屏幕gamma
	color = pow(color, vec3(1.0/2.2));
	FragColor = vec4(color, 1.0);
}
```

![输入图片说明](/imgs/2025-02-20/js9XBTXaLv4ANHmd.png)

### 2.处理图片的sRGB到RGB线性变换
所以正常步骤应该是这样的，这样输出就是正常颜色了
将`sRGB`变换为`RGB`，是为了将图片变为线性空间之后，与光照计算才是正确的
```glsl
void main()
{
	vec3 color = texture(screenTexSampler, uv).rgb;
	//1 将sRGB变换位RGB
	color = pow(color, vec3(2.2));
	//2 与光照进行计算

	//3 最终颜色要抵抗屏幕gamma
	color = pow(color, vec3(1.0/2.2));
	FragColor = vec4(color, 1.0);
}
```
### 3.改造Texture构造函数，允许外部传入InternalFormat参数

![输入图片说明](/imgs/2025-02-20/MklUqFKdBnqEKzSZ.png)

这里并不是所有的贴图都用来表达颜色：
例如蒙版贴图

去到`texture.h`中，添加参数
```cpp
Texture(const std::string& path, unsigned int unit, unsigned int internalFormat = GL_RGBA);
Texture(
	unsigned int unit,
	unsigned char* dataIn,
	uint32_t widthIn,
	uint32_t heightIn,
	unsigned int internalFormat = GL_RGBA
);
Texture(unsigned int width, unsigned int height, unsigned int unit, unsigned int internalFormat = GL_RGBA);
Texture(const std::vector<std::string>& paths, unsigned int unit, unsigned int internalFormat = GL_RGBA);
```
然后就到具体的方法里面中的`glTexImage2D`方法替换参数。
以前我们是直接用`GL_RGBA`，现在改为参数`internalFormat`，这样如果我们再构造时要手动声明类型就很方便了
```cpp
Texture::Texture(const std::string& path, unsigned int unit, unsigned int internalFormat) {
	...
	glTexImage2D(GL_TEXTURE_2D, 0, internalFormat, mWidth, mHeight, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
	...
}

Texture::Texture(
	unsigned int unit,
	unsigned char* dataIn,
	uint32_t widthIn,
	uint32_t heightIn,
	unsigned int internalFormat
) {
	...
	glTexImage2D(GL_TEXTURE_2D, 0, internalFormat, mWidth, mHeight, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
	...
}

Texture::Texture(unsigned int width, unsigned int height, unsigned int unit, unsigned int internalFormat) {
	...
	glTexImage2D(GL_TEXTURE_2D, 0, internalFormat, mWidth, mHeight, 0, GL_RGBA, GL_UNSIGNED_BYTE, NULL);
	...
}

//paths:右左上下后前(+x -x +y -y +z -z)
Texture::Texture(const std::vector<std::string>& paths, unsigned int unit, unsigned int internalFormat) {
	...
	glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, internalFormat, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
	...
}
```
然后进行修改以下两个方法，不过这两个方法我们可以确定是用来读取颜色贴图的，所以我们直接使用`GL_SRGB_ALPHA`
```cpp
Texture* Texture::createTexture(const std::string& path, unsigned int unit) {
	...
	auto texture = new Texture(path, unit, GL_SRGB_ALPHA);
	...
}

Texture* Texture::createTextureFromMemory(
	const std::string& path,
	unsigned int unit,
	unsigned char* dataIn,
	uint32_t widthIn,
	uint32_t heightIn
) {
	...
	//2 如果本路径对应的texture没有生成过，则重新生成
	auto texture = new Texture(unit, dataIn, widthIn, heightIn, GL_SRGB_ALPHA);
	...
}
```
做完以上的准备工作，我们就可以进行实验
我们在构造纹理的时候，手动填写参数`GL_SRGB_ALPHA`，这样的话，我们读入的图片就会在进入`shader`计算前，自动从`sRGB`转化为`RGB`
```cpp
void prepare() {
	...
	auto sgeo = Geometry::createScreenPlane();
	auto smat = new ScreenMaterial();
	smat->mScreenTexture = new Texture("assets/textures/wall.jpg", 0, GL_SRGB_ALPHA);
	auto smesh = new Mesh(sgeo, smat);
	scene->addChild(smesh);
	...
}
```
所以我们也没必要在`screen.frag`做`sRGB`转化为`RGB`的操作了
注释掉后输出的就是正常颜色
```glsl
void main()
{
	...
	//1 将sRGB变换位RGB
	//color = pow(color, vec3(2.2));
	//2 与光照进行计算
	//3 最终颜色要抵抗屏幕gamma
	color = pow(color, vec3(1.0/2.2));
	FragColor = vec4(color, 1.0);
}
```

![输入图片说明](/imgs/2025-02-20/SooqnUl2ZWGxZzzT.png)

有了这句话后，我们也不需要进行抵抗屏幕`gamma`了，而且抵抗屏幕`gamma`要手动到每一个`shader`中都进行更改，很麻烦
全部注释掉，相当于直接输出纹理采样的颜色
```glsl
void main()
{
	...
	vec3 color = texture(screenTexSampler, uv).rgb;
	//1 将sRGB变换位RGB
	//color = pow(color, vec3(2.2));
	//2 与光照进行计算
	//3 最终颜色要抵抗屏幕gamma
	//color = pow(color, vec3(1.0/2.2));
	FragColor = vec4(color, 1.0);
}
```
但是现在我们是`rgb`空间下的，直接输出会偏黑，因为我们构造的实验场景手动修改了参数`GL_SRGB_ALPHA`，
`smat->mScreenTexture = new Texture("assets/textures/wall.jpg", 0, GL_SRGB_ALPHA);`
所以我们调用`glEnable(GL_FRAMEBUFFER_SRGB);`
这样子就完美得到正常颜色了
```cpp
void prepare() {
	glEnable(GL_FRAMEBUFFER_SRGB);
	...
	//pass 02
	auto sgeo = Geometry::createScreenPlane();
	auto smat = new ScreenMaterial();
	smat->mScreenTexture = new Texture("assets/textures/wall.jpg", 0, GL_SRGB_ALPHA);
	auto smesh = new Mesh(sgeo, smat);
	scene->addChild(smesh);
	...
}
```

### 4.加入PostProcess后处理Pass，专门用来调节Gamma

![输入图片说明](/imgs/2025-02-20/B71lAiDWNCk6QUzR.png)

如果用统一矫正的话，`pass01`处理的时候是正确的线性空间，但是到`pass02`就不对了
我们创建两个`pass`，然后将统一的矫正关闭
```cpp
Scene* sceneOff = nullptr;
Scene* scene = nullptr;
Framebuffer* fbo = nullptr;
void prepare() {
	//glEnable(GL_FRAMEBUFFER_SRGB);
	fbo = new Framebuffer(WIDTH, HEIGHT);
	...
	//pass 01
	auto boxGeo = Geometry::createBox(5.0f);
	auto boxMat = new PhongMaterial();
	boxMat->mDiffuse = new Texture("assets/textures/wall.jpg", 0, GL_SRGB_ALPHA);
	boxMat->mShiness = 64;
	auto boxMesh = new Mesh(boxGeo, boxMat);
	sceneOff->addChild(boxMesh);
	
	//pass 02
	auto sgeo = Geometry::createScreenPlane();
	auto smat = new ScreenMaterial();
	smat->mScreenTexture = fbo->mColorAttachment;
	//smat->mScreenTexture = new Texture("assets/textures/wall.jpg", 0, GL_SRGB_ALPHA);
	auto smesh = new Mesh(sgeo, smat);
	scene->addChild(smesh);
	...
}
int main() {
	...
	while (glApp->update()) {
		cameraControl->update();
		renderer->setClearColor(clearColor);
		renderer->render(sceneOff, camera,dirLight, ambLight, fbo->mFBO);
		renderer->render(scene, camera, dirLight, ambLight);
		renderIMGUI();
	}
	...
}
```
现在第一个`pass`处理的就是线性空间的颜色，并且传到第二`pass`也是线性的，然后我们只需要在`screen.frag`中对输出颜色进行矫正即可
```glsl
void main()
{
	
	vec3 color = texture(screenTexSampler, uv).rgb;
	//1 将sRGB变换位RGB
	//color = pow(color, vec3(2.2));

	//2 与光照进行计算

	//3 最终颜色要抵抗屏幕gamma
	color = pow(color, vec3(1.0/2.2));

	FragColor = vec4(color, 1.0);
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzQ1ODY0NjUsODg1ODc3OTQzLDE4ODM1Nz
k1MzUsLTE2Njc2MTQ0MjAsMTE4NjI0NTE4NCw0MTUyMzA1OSwt
MzkzMTc4MDcyLDQ3Nzk0MDg0NywtNjg2MjAxNzU0LC01MzU5MT
g5ODIsLTk4MjM0MjAyMywtNTc1ODk3NDMsLTMyMzM0MTA5MCwt
Mjc3Njk1OTI4LC0zMTA1MTg1NjEsMTYxMDQ5MDAyOSwtMTE3Nj
MzNDQ4NF19
-->