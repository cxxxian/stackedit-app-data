# 封装一个Application
![输入图片说明](/imgs/2024-10-13/VIxcnh6mId4MhoGT.png)
![输入图片说明](/imgs/2024-10-13/sVDUnB9Ry98gBWSi.png)
# 按三个环节设计接口
![输入图片说明](/imgs/2024-10-13/S2w0kZL0slRjxDTp.png)
```
bool Application::init(const int& width, const int& height)
{
	mWidth = width;
	mHeight = height;
	glfwInit();

	//主要版本和次要版本，意为3.3
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
![输入图片说明](/imgs/2024-10-13/G7ETkU4hUHeio9aM.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE4NTY2MTAwMSwxMzUxMDczODg1LC0xMj
YxNzMxNjYyLDEzMTExMzIyNzddfQ==
-->