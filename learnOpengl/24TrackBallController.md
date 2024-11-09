# 鼠标左键旋转
![输入图片说明](/imgs/2024-11-08/3ni32chSZxBU2fQS.png)
![输入图片说明](/imgs/2024-11-08/hckRujHW7Ie4nZpZ.png)
![输入图片说明](/imgs/2024-11-08/K2OLiybgkvSpPVS7.png)
制作trackBallCameraControl.h
```
#pragma once
#include "cameraControl.h"
class TrackBallCameraControl :public CameraControl {
public:
	TrackBallCameraControl();
	~TrackBallCameraControl();

	//父类当中的接口函数，是否需要重写
	void onCursor(double xpos, double ypos) override;
private:
	void pitch(float angle);
private:
};
```
和trackBallCameraControl.cpp
```
#include "trackBallControl.h"

TrackBallCameraControl::TrackBallCameraControl()
{
}

TrackBallCameraControl::~TrackBallCameraControl()
{
}

void TrackBallCameraControl::onCursor(double xpos, double ypos)
{
	if (mLeftMouseDown) {
		//调整相机的各类参数
		//1 计算经线和纬线旋转的增量角度（正负都有可能）
		float deltaX = (xpos - mCurrentX) * mSensitivity;
		float deltaY = (ypos - mCurrentY) * mSensitivity;

		//2 分开pitch跟yaw各自计算
		pitch(deltaY);
	}

	mCurrentX = xpos;
	mCurrentY = ypos;
}

void TrackBallCameraControl::pitch(float angle)
{
	//绕着mRight向量在旋转
	auto mat = glm::rotate(glm::mat4(1.0f), glm::radians(angle), mCamera->mRight);

	//影响当前相机的up向量和位置
	//此处需要将mCamera->mUp升维到四维，最后一维补0.0f
	mCamera->mUp = mat * vec4(mCamera->mUp, 0.0f);//vec4给到vec3，给了xyz
	mCamera->mPosition = mat * glm::vec4(mCamera->mPosition, 1.0f);
}

```
此时理一遍`main`中关于`camera`和`cameraControl`的使用
引入头文件以及声明`camera`和`cameraControl`
```
//引入相机+控制器
#include "application/camera/perspectiveCamera.h"
#include "application/camera/trackBallCameraControl.h"

PerspectiveCamera* camera = nullptr;
TrackBallCameraControl* cameraControl = nullptr;
```
使用`PerspectiveCamera`的构造矩阵声明一个`camera`，使用`TrackBallCameraControl`声明一个`cameraControl`，并且将刚刚的`camera`利用方法`setCamera(camera)`传入
```
void prepareCamera() {
    camera = new PerspectiveCamera(60.f, (float)app->getWidth() / (float)app->getHeight(), 0.1f, 1000.0f);

    cameraControl = new TrackBallCameraControl();
    cameraControl->setCamera(camera);
}
```
然后我们在`render`中，利用`camera`的`getViewMatrix()`和`getProjectionMatrix()`进行`shader`绑定
```
void render(){
	...
    shader->begin();

    shader->setInt("sampler", 0);//此处值为0是因为我们的纹理绑定在0号位上
    shader->setMatrix4x4("transform", transform);
    shader->setMatrix4x4("viewMatrix", camera->getViewMatrix());
    shader->setMatrix4x4("projectionMatrix", camera->getProjectionMatrix());
	...
}
```
最后运行，我们就可以移动三角形的`pitch`角，因为目前我们只做了`pitch`角函数并且在`onCursor`函数调用
`yaw`角方向实现如下：
与`pitch`角方向多更新`mRight`向量，因为移动`pitch`角度不会影响到`mRight`，而`yaw`会
```
void TrackBallCameraControl::yaw(float angle)
{
	//绕着世界坐标系的y轴旋转
	auto mat = glm::rotate(glm::mat4(1.0f), glm::radians(angle), glm::vec3(0.0f, 1.0f, 0.0f));
	mCamera->mUp = mat * glm::vec4(mCamera->mUp, 0.0f);//vec4给到vec3，给了xyz
	mCamera->mRight = mat * glm::vec4(mCamera->mRight, 0.0f);//vec4给到vec3，给了xyz
	mCamera->mPosition = mat * glm::vec4(mCamera->mPosition, 1.0f);
}
```
# 鼠标中键平移
![输入图片说明](/imgs/2024-11-08/3XZfScHP5iM7MGwR.png)
在`trackBallCameraControl.h`中声明一个`mMoveSpeed`，类似于之前的`mSensitivity`
```
private:
	float mMoveSpeed = 0.005f;
```
到`trackBallCameraControl.cpp`的`onCursor`方法中，添加`else if`中键点下的情况，
```
else if (mMiddleMouseDown) {
		float deltaX = ((float)xpos - mCurrentX) * mMoveSpeed;
		float deltaY = ((float)ypos - mCurrentY) * mMoveSpeed;

		mCamera->mPosition += mCamera->mUp * deltaY;
		mCamera->mPosition -= mCamera->mRight * deltaX;
}
```
# 鼠标中键滚轮缩放
![输入图片说明](/imgs/2024-11-09/CnCZ27pnwFKivWJU.png)
### 缩放策略：
透视相机：沿着front向量运动摄像机，所以会存在透过物体的可能
正交相机：变化投影盒的大小，所以放大只会无限放大物体并不会透过它
![输入图片说明](/imgs/2024-11-09/gE2xoolnPVjfPXEz.png)
![输入图片说明](/imgs/2024-11-09/B0VUfo098XCrKz4i.png)
![输入图片说明](/imgs/2024-11-09/MMAbgxNtnAGgDQyo.png)
![输入图片说明](/imgs/2024-11-09/X2LpSiNNKrsLBdeg.png)
![输入图片说明](/imgs/2024-11-09/qeOYHJRsZzUGvtQv.png)
到`camera.h`中声明一个`scale`方法，
```
virtual void scale(float deltaScale);
```
`perspectiveCamera.h`中`override scale`函数
```
void scale(float deltaScale) override;
```
并到`perspectiveCamera.cpp`中实现
```
void PerspectiveCamera::scale(float deltaScale)
{
	auto front = glm::cross(mUp, mRight);
	mPosition += (front * deltaScale);
}
```
`orthographicCamera.h`中同样`override scale`函数，并且声明一个mScale进行累加
```
private:
	float mScale{ 0.0f };
```
利用`scale`对`getProjectionMatrix()`的包围盒进行缩放
```
glm::mat4 OrthographicCamera::getProjectionMatrix()
{
	float scale = std::pow(2.0f, mScale);
	return glm::ortho(mLeft * scale, mRight * scale, mTop * scale, mBottom * scale, mNear, mFar);
}

void OrthographicCamera::scale(float deltaScale)
{
	mScale += deltaScale;
}
```
以上就完成了对于缩放效果的设计，但是现在还没和鼠标扯上关系
到cameraControl.h中声明一个虚函数`onScroll(float offset)`
```
virtual void onScroll(float offset);//+1,-1
```
然后因为只有`trackBallController`需要，游戏相机并不需要这个功能，所以我们到`trackBallController.h`中`override`这个函数，
但在此之前，我们需要回到`cameraControl`中，设计一个用来调整缩放速度的系数，这样使得更简洁以及调节性更高
```
public:
	void setScaleSppeed(float s) { mScaleSpeed = s; }
protected:
	//6 记录相机缩放的速度
	float mScaleSpeed = 0.2f;
```
最后到`trackBallController.cpp`中实现这个`onScroll`函数
```
void TrackBallCameraControl::onScroll(float offset)
{
	mCamera->scale(mScaleSpeed * offset);
}
```
以上我们在`camera`和`cameraControl`实现了`scale和onScroll`函数，最后我们去`Application`中加入鼠标滚动的响应
老样子，重新盘一遍：
在application.h中
声明一个`scrollCallBack`
```
static void scrollCallBack(GLFWwindow* window, double xoffset, double yoffset);
```
声明一个函数指针，
```
using ScrollCallback = void(*)(double offset);
```
利用函数指针声明一个变量
```
ScrollCallback mScrollCallback{ nullptr };
```
声明一个`setScrollCallback`
```
void setScrollCallback(ScrollCallback callback) { mScrollCallback = callback; }
```
在application.cpp中实现`static void scrollCallBack`
```
//滚动消息的xoffset没有意义
void Application::scrollCallBack(GLFWwindow* window, double xoffset, double yoffset)
{
	Application* self = (Application*)glfwGetWindowUserPointer(window);
	if (self->mScrollCallback != nullptr) {
		self->mScrollCallback(yoffset);
	}
}
```
到`application.cpp`的`init`方法中绑定
```
//鼠标滚轮消息
glfwSetScrollCallback(window, scrollCallBack);
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIwMDI4NDA0NywxNzM1NTI4MDMsODIwNz
g4NDQ5LDM0NDMwOTYwOCwtMTUzMzA5NjUwMywtMTUyNzU5MDU3
LC04MzM3NTM5MDMsMTI3ODY3NTE2MCwtMTk1MDYyMDMyMywxMz
M3ODQ5MDY3LDkxMDM3MDY5MywxNDg0MTE1MTA3LC0yMDg4NzQ2
NjEyXX0=
-->