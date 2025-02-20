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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc4MzgzMjQ3NSwtMTY2NzYxNDQyMCwxMT
g2MjQ1MTg0LDQxNTIzMDU5LC0zOTMxNzgwNzIsNDc3OTQwODQ3
LC02ODYyMDE3NTQsLTUzNTkxODk4MiwtOTgyMzQyMDIzLC01Nz
U4OTc0MywtMzIzMzQxMDkwLC0yNzc2OTU5MjgsLTMxMDUxODU2
MSwxNjEwNDkwMDI5LC0xMTc2MzM0NDg0XX0=
-->