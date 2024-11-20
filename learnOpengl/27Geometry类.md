![输入图片说明](/imgs/2024-11-20/BJZMhf09N3OgxPwj.png)
![输入图片说明](/imgs/2024-11-20/K6uoexsFgZ9dulge.png)
#pragma once
#include "core.h"

class Geometry {
public:
	Geometry();
	~Geometry();

	static Geometry* createBox(float size);
	static Geometry* createSphere(float size);

	GLuint getVao() const { return mVao; }
	uint32_t getIndicesCount() const { return mIndicesCount; }

private:
	GLuint mVao{ 0 };
	GLuint mPosVbo{ 0 };
	GLuint mUvVbo{ 0 };
	GLuint Ebo{ 0 };

	uint32_t mIndicesCount{ 0 };
};

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1ODY5NjkyODQsLTE5NTc5OTA4NCwtMj
A4ODc0NjYxMl19
-->