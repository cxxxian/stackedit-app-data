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
最后到`main.cpp`中
先声明`geometry`，然后我们用`createBox`赋值，最后在`render`中绑定`vao`
```
Geometry* geometry = nullptr;
void prepareVAO() {
    geometry = Geometry::createBox(6.0f);
}
void render(){
	...
    //2 绑定当前的vao
    glBindVertexArray(geometry->getVao());
    //3 发出绘制指令
    //glDrawArrays(GL_LINE_STRIP, 0, 6);
    glDrawElements(GL_TRIANGLES, geometry->getIndicesCount(), GL_UNSIGNED_INT, 0);

    shader->end();
}
```
就可以看到立方体了，不过此时我们还没开启深度检测
![输入图片说明](/imgs/2024-11-20/tVtV6GX8Bq7XemJc.png)
复习一遍深度测试流程
制作`prepareState()`用来开启深度测试，在`main`函数中调用，**最重要的是，别忘记清理深度缓存信息**
```
void prepareState() {
    glEnable(GL_DEPTH_TEST);
    glDepthFunc(GL_LESS);
}
void render(){
    //执行opengl画布清理操作
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	...
}
int main() {
	...
    prepareShader();
    //prepareInterleavedBuffer();
    //prepareVAOForGLTriangles();
    prepareVAO();
    prepareTexture();
    prepareCamera();
    prepareState();
	...
}
```
![输入图片说明](/imgs/2024-11-20/02ldfXzuYfLfTwL6.png)
# 球体
### 位置分析
![输入图片说明](/imgs/2024-11-20/jZCCAaqMtWofLJZG.png)
![输入图片说明](/imgs/2024-11-20/cxaje47wVFc5vSOg.png)
![输入图片说明](/imgs/2024-11-20/QBIsPXH4NAq211Yt.png)
**纬线只需要0-180度，经线0-360度**，就可以确定球上的每一个顶点的值
`glm::pi<float>()`是派的值
![输入图片说明](/imgs/2024-11-20/dM6dsgL7VLgTTZyK.png)
此处注意我们的 `i <= latitude, j <= long`，为什么要取等，这样是数据冗余的。
原因：
如下图，我们最左边的一列和最右边一列其实是同一条经线，但是但是，两边需要存储的uv值其实是有区别的，虽然是数据冗余，但是我们可以更方便地进行uv绑定
![输入图片说明](/imgs/2024-11-20/dcSRo15azmXF0sOG.png)
### uv值分析
![输入图片说明](/imgs/2024-11-20/kvNhu9aQcU0bvnS9.png)
### 索引值分析
此处不需要等于 `i < latitude, j < long`，是因为，我们通过p1，可以知道p2，p3，p4.
![输入图片说明](/imgs/2024-11-20/Ba5qJ15CVUxRnnYo.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQwODg5NDQwNSwxNTM2NjkyNzQsMTg4Mj
AxNDA3MCwtMjAzOTEwNDgwNSwtNzMyNzQ3MCwtMzU2MjU5OTQ1
LDEwMjgwNDQxMDEsLTE5NTc5OTA4NCwtMjA4ODc0NjYxMl19
-->