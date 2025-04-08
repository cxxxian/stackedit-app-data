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
tinyexr.h`
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyNDkyMzIzMDUsMTI3NjA3NjI2NCwtMz
I4NjA1ODY4LDEwMzkzMDIwMTUsMTc2MTE3MTg1OSwxNzE0NDQ3
MDg0LDIzNDM4OTg5LDY2MjM1MTUsLTE5MDY4MjM1NzMsMTc0NT
AxMjcyNiwxNTM1NDQwMjE4LC0yMDg4NzQ2NjEyXX0=
-->