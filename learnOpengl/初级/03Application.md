# 封装一个Application
![输入图片说明](/imgs/2024-10-13/VIxcnh6mId4MhoGT.png)
![输入图片说明](/imgs/2024-10-13/sVDUnB9Ry98gBWSi.png)

# 按三个环节设计接口
![输入图片说明](/imgs/2024-10-13/S2w0kZL0slRjxDTp.png)

```cpp
bool Application::init(const int& width, const int& height)
{
	mWidth = width;
	mHeight = height;
	glfwInit();

	//主要版本和次要版本，意为4.6
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 6);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

	mWindow = glfwCreateWindow(mWidth, mHeight, "LearnOpenGl", NULL, NULL);

	if (mWindow == NULL) {
		return false;
	}

	//设置当前窗体对象为Opengl的绘制舞台
	glfwMakeContextCurrent(mWindow);

	//使用glad加载所有当前版本的opengl函数
	if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
	{
		std::cout << "Failed to initialize GLAD" << std::endl;
		return -1;
	}

	return true;
}

bool Application::update()
{
	if (glfwWindowShouldClose(mWindow)) {
		return false;
	}
	//检查消息队列是否有需要处理的鼠标、键盘消息
	glfwPollEvents();

	//切换双缓存
	glfwSwapBuffers(mWindow);

	return true;
}

void Application::destroy()
{

	//退出程序前做相关处理
	glfwTerminate();
}
```
# 函数指针
![输入图片说明](/imgs/2024-10-13/CNaydYLtOfCeBkqe.png)

此例的MyFunc指的是两个形参是int类型的，返回类型也是int类型的
## 此处目的
为了外界不破坏Application类的情况下，通过函数指针去将自己在main函数设计的函数传给Application

![输入图片说明](/imgs/2024-10-13/G7ETkU4hUHeio9aM.png)

![输入图片说明](/imgs/2024-10-13/VIUxM347q0kIqYQN.png)

## 如何做一个回调函数
### 窗口变换例子
![输入图片说明](/imgs/2024-10-13/ntHIG4xCIvTcI34R.png)
在application.h中
1. 声明一个指针函数
```cpp
using ResizeCallback = void(*)(int width, int height);
```
2. 在private中声明一个ResizeCallback的成员变量
```cpp
ResizeCallback mResizeCallback{ nullptr };
```
3. 在public中，声明一个setResizeCallback，设置窗体变化相应回调函数。这个用来在main中接受自己做的函数！！！
```cpp
void setResizeCallback(ResizeCallback callback) { mResizeCallback = callback; }
```
4. 声明一个static的静态函数，用于相应glfw窗体变化
此处使用static即可以做到不用类对象即可调用

```cpp
	static void frameBufferSizeCallback(GLFWwindow* window, int width, int height);
```
在application.cpp实现如下
```cpp
void Application::frameBufferSizeCallback(GLFWwindow* window, int width, int height)
{
	std::cout << "Resize" << std::endl;
	if (Application::getInstance()->mResizeCallback != nullptr) {
		Application::getInstance()->mResizeCallback(width, height);
	}
}
```
5. 最后在application中的init，将静态函数设置到glfw的监听Resize监听当中
```cpp
glfwSetFramebufferSizeCallback(mWindow, frameBufferSizeCallback);
```
6. 所以最后在main.cpp中，
制作自己的OnResize函数，会发现此处不需要调用window变量
```cpp
void OnResize(int width, int height) {
    GL_CALL(glViewport(0, 0, width, height));
    std::cout << "OnResize" << std::endl;
}
```
在main函数中使用
```c[[
app->setResizeCallback(OnResize);
```
### 键盘消息例子（调整了顺序）

![输入图片说明](/imgs/2024-10-13/XWUMXkHwgsSR9YPc.png)

1. static静态函数
```cpp
static void KeyCallBack(GLFWwindow* window, int key, int scancode, int action, int mods);
```
2. 键盘响应
```cpp
glfwSetKeyCallback(mWindow, KeyCallBack); 
```
3.
```cpp
using KeyBoardCallback = void(*)(int key, int action, int mods);
```
4. 
```cpp
KeyBoardCallback mKeyBoardCallback{ nullptr };
```
5.
```cpp
void setKeyBoardCallback(KeyBoardCallback callback) { mKeyBoardCallback = callback; }
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMzYyNDkxNDgyLC0xODY1NzcxOTYwLC0zOD
Q2MzM5OCwtMzAxMDA1ODYsMzg2NzEwOTBdfQ==
-->