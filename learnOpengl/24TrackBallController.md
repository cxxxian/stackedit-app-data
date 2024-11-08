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
最后运行，w'm
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg4Mjg3OTUyMiwxMzM3ODQ5MDY3LDkxMD
M3MDY5MywxNDg0MTE1MTA3LC0yMDg4NzQ2NjEyXX0=
-->