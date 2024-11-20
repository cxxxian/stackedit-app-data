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
一套完整的vbo，vao，ebo绑定过程
```
Geometry* Geometry::createBox(float size)
{
	Geometry* geometry = new Geometry();
	geometry->mIndicesCount = 36;

	float halfSize = size / 2.0f;
	float positions[] = {
		//Front face
		-halfSize, -halfSize, halfSize, halfSize, -halfSize, halfSize, halfSize, halfSize, halfSize, -halfSize, halfSize, halfSize,
		//Back face
		-halfSize, -halfSize, -halfSize, halfSize, -halfSize, halfSize, halfSize, halfSize, -halfSize, -halfSize, halfSize, -halfSize,
		//Top face
		-halfSize, halfSize, -halfSize, halfSize, halfSize, -halfSize, halfSize, halfSize, halfSize, -halfSize, halfSize, halfSize,
		//Botton face
		-halfSize, -halfSize, -halfSize, halfSize, -halfSize, -halfSize, halfSize, -halfSize, halfSize, -halfSize, -halfSize, halfSize,
		//Right face
		halfSize, -halfSize, -halfSize, halfSize, halfSize, -halfSize, halfSize, halfSize, halfSize, halfSize, -halfSize, halfSize,
		//Left face
		-halfSize, -halfSize, -halfSize, -halfSize, halfSize, -halfSize, -halfSize, halfSize, halfSize, -halfSize, -halfSize, halfSize
	};

	float uvs[] = {
		0.0, 0.0, 1.0, 0.0, 1.0, 1.0, 0.0, 1.0,
		1.0, 0.0, 0.0, 0.0, 0.0, 1.0, 1.0, 1.0,
		0.0, 1.0, 1.0, 1.0, 1.0, 0.0, 0.0, 0.0,
		1.0, 1.0, 0.0, 1.0, 0.0, 0.0, 1.0, 0.0,
		1.0, 0.0, 1.0, 1.0, 0.0, 1.0, 0.0, 0.0,
		0.0, 0.0, 1.0, 0.0, 1.0, 1.0, 0.0, 1.0
	};

	unsigned int indices[] = {
		0, 1, 2, 2, 3, 0,//Front face
		4, 5, 6, 6, 7, 4,//Back face
		8, 9, 10, 10, 11, 8,//Top face
		12, 13, 14, 14, 15, 12,//Botton face
		16, 17, 18, 18, 19, 16,//Right face
		20, 21, 22, 22, 23, 20//Left face
	};

	//2 VBO创建
	GLuint& posVbo = geometry->mPosVbo, uvVbo = geometry->mUvVbo;
	glGenBuffers(1, &posVbo);
	glBindBuffer(GL_ARRAY_BUFFER, posVbo);
	glBufferData(GL_ARRAY_BUFFER, sizeof(positions), positions, GL_STATIC_DRAW);

	glGenBuffers(1, &uvVbo);
	glBindBuffer(GL_ARRAY_BUFFER, uvVbo);
	glBufferData(GL_ARRAY_BUFFER, sizeof(uvs), uvs, GL_STATIC_DRAW);

	//3 EBO创建
	glGenBuffers(1, &geometry->mEbo);
	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, geometry->mEbo);
	glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

	//4 VAO创建
	glGenVertexArrays(1, &geometry->mVao);
	glBindVertexArray(geometry->mVao);

	//5 绑定vbo ebo 加入属性描述信息
	//5.1 加入位置属性描述信息
	glBindBuffer(GL_ARRAY_BUFFER, posVbo);
	glEnableVertexAttribArray(0);
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(float) * 3, (void*)0);

	//5.2  加入uv属性描述信息
	glBindBuffer(GL_ARRAY_BUFFER, uvVbo);
	glEnableVertexAttribArray(1);
	glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, sizeof(float) * 2, (void*)0);

	//5.3 加入ebo到当前的vao
	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, geometry->mEbo);
	
	glBindVertexArray(0);
	return geometry;
}

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjIzMjE4OTc0LDEwMjgwNDQxMDEsLTE5NT
c5OTA4NCwtMjA4ODc0NjYxMl19
-->