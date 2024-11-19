# 鼠标右键旋转
![输入图片说明](/imgs/2024-11-18/dUFty3H3vhqjkISX.png)
创建`GameCameraControl.h`和`GameCameraControl.cpp`
```
#pragma once
#include "cameraControl.h"

class GameCameraControl :public CameraControl {
public:
	GameCameraControl();
	~GameCameraControl();

	void onCursor(double xpos, double ypos) override;
private:
	void pitch(float angle);
	void yaw(float angle);

private:
	float mPitch = {0.0f};
};
```
`GameCameraControl.cpp`实现如下，`mPitch`用来做上下抬头的限制：
```
#include "gameCameraControl.h"

GameCameraControl::GameCameraControl()
{
}

GameCameraControl::~GameCameraControl()
{
}

void GameCameraControl::onCursor(double xpos, double ypos)
{
	float deltaX = (xpos - mCurrentX) * mSensitivity;
	float deltaY = (ypos - mCurrentY) * mSensitivity;

	if (mRightMouseDown) {
		pitch(deltaY);
		yaw(deltaX);
	}

	mCurrentX = xpos;
	mCurrentY = ypos;
}

void GameCameraControl::pitch(float angle)
{
	mPitch += angle;
	if (mPitch > 89.0f || mPitch < -89.0f) {
		mPitch -= angle;
		return;
	}
	//在gameCameraControl的情况下，pitch不会影响mPostion
	auto mat = glm::rotate(glm::mat4(1.0f), glm::radians(angle), mCamera->mRight);
	mCamera->mUp = mat * glm::vec4(mCamera->mUp, 0.0f);
}

void GameCameraControl::yaw(float angle)
{
	auto mat = glm::rotate(glm::mat4(1.0f), glm::radians(angle), glm::vec3(0.0f, 1.0f, 0.0f));
	mCamera->mUp = mat * glm::vec4(mCamera->mUp, 0.0f);
	mCamera->mRight = mat * glm::vec4(mCamera->mRight, 0.0f);
}
```
# 键盘移动
![输入图片说明](/imgs/2024-11-18/Zum6dc7T5CkRyiAV.png)
![输入图片说明](/imgs/2024-11-18/hP09EbvgBFy5L7fI.png)
按情况来逐一分析，
第三列的情况，需要进行`direction`的单位归一化，因为我们都是对单位向量进行操作，自然希望也得到一个单位向量
第四列的话`direction`为0需要特殊判断，否则归一化除0会出问题
```
#pragma once

#include "cameraControl.h"

class GameCameraControl :public CameraControl {
public:
	GameCameraControl();
	~GameCameraControl();

	void onCursor(double xpos, double ypos) override;
	void update() override;

	void setSpeed(float s) { mSpeed = s; }
private:
	void pitch(float angle);
	void yaw(float angle);

private:
	float mPitch{ 0.0f };
	float mSpeed{ 0.1f };
};
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjI5OTMwMTYzLDE3OTg1ODYyNDNdfQ==
-->