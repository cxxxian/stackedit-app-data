# 框架搭建
![输入图片说明](/imgs/2024-11-20/BJZMhf09N3OgxPwj.png)
![输入图片说明](/imgs/2024-11-20/K6uoexsFgZ9dulge.png)
```
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
```
先基础实现一下：
```
#include "geometry.h"

Geometry::Geometry(){}
Geometry::~Geometry()
{
	if (mVao != 0) {
		glDeleteVertexArrays(1, &mVao);
	}
	if (mPosVbo != 0) {
		glDeleteBuffers(1, &mPosVbo);
	}
	if (mUvVbo != 0) {
		glDeleteBuffers(1, &mUvVbo);
	}
	if (mEbo != 0) {
		glDeleteBuffers(1, &mEbo);
	}
}
Geometry* Geometry::createBox(float size)
{
	Geometry* geometry = new Geometry();
	return geometry;
}
Geometry* Geometry::createSphere(float size)
{
	Geometry* geometry = new Geometry();
	return geometry;
}
```
# 立方体
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAyODA0NDEwMSwtMTk1Nzk5MDg0LC0yMD
g4NzQ2NjEyXX0=
-->