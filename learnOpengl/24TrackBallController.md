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
![输入图片说明](/imgs/2024-11-09/gE2xoolnPVjfPXEz.png)
![输入图片说明](/imgs/2024-11-09/B0VUfo098XCrKz4i.png)
![输入图片说明](/imgs/2024-11-09/MMAbgxNtnAGgDQyo.png)
![输入图片说明](/imgs/2024-11-09/X2LpSiNNKrsLBdeg.png)
![输入图片说明](/imgs/2024-11-09/qeOYHJRsZzUGvtQv.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMzQ0MzA5NjA4LC0xNTMzMDk2NTAzLC0xNT
I3NTkwNTcsLTgzMzc1MzkwMywxMjc4Njc1MTYwLC0xOTUwNjIw
MzIzLDEzMzc4NDkwNjcsOTEwMzcwNjkzLDE0ODQxMTUxMDcsLT
IwODg3NDY2MTJdfQ==
-->