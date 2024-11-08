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
使用PerspectiveCamera的构造矩阵声明一个camera，使用TrackBallCameraControl声明一个cameraControl，并且将刚刚的camera利用方法setCamera(camera)传入
```
void prepareCamera() {
    camera = new PerspectiveCamera(60.f, (float)app->getWidth() / (float)app->getHeight(), 0.1f, 1000.0f);

    cameraControl = new TrackBallCameraControl();
    cameraControl->setCamera(camera);
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMxOTA1NzkyOCw5MTAzNzA2OTMsMTQ4ND
ExNTEwNywtMjA4ODc0NjYxMl19
-->