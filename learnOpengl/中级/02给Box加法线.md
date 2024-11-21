# 制作Plane
先前做了box，sphere，现在多做一个plane
```
Geometry* Geometry::createPlane(float width, float height)
{
	Geometry* geometry = new Geometry();
	geometry->mIndicesCount = 6;

	float halfW = width / 2.0f;
	float halfH = height / 2.0f;

	float positions[] = {
		-halfW, -halfH, 0.0f,
		halfW, -halfH, 0.0f,
		halfW, halfH, 0.0f,
		-halfW, halfH, 0.0f,
	};

	float uvs[] = {
		0.0f, 0.0f,
		1.0f, 0.0f,
		1.0f, 1.0f,
		0.0f, 1.0f
	};

	unsigned int indices[] = {
		0, 1, 2,
		2, 3, 0
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
# 添加法线
### 法线数据
去到`createBox()`方法中，给`box`添加法线数据
```
float normals[] = {
		//Front face
		0.0f, 0.0f, 1.0f,
		0.0f, 0.0f, 1.0f,
		0.0f, 0.0f, 1.0f,
		0.0f, 0.0f, 1.0f,
		//Back face
		0.0f, 0.0f, -1.0f,
		0.0f, 0.0f, -1.0f,
		0.0f, 0.0f, -1.0f,
		0.0f, 0.0f, -1.0f,
		//Top face
		0.0f, 1.0f, 0.0f,
		0.0f, 1.0f, 0.0f,
		0.0f, 1.0f, 0.0f,
		0.0f, 1.0f, 0.0f,
		//Botton face
		0.0f, -1.0f, 0.0f,
		0.0f, -1.0f, 0.0f,
		0.0f, -1.0f, 0.0f,
		0.0f, -1.0f, 0.0f,
		//Right face
		1.0f, 0.0f, 0.0f,
		1.0f, 0.0f, 0.0f,
		1.0f, 0.0f, 0.0f,
		1.0f, 0.0f, 0.0f,
		//Left face
		-1.0f, 0.0f, 0.0f,
		-1.0f, 0.0f, 0.0f,
		-1.0f, 0.0f, 0.0f,
		-1.0f, 0.0f, 0.0f
	};
```
### 制作normalVbo
去geometry.h中，添加一个`mNormalVbo`
```
private:
	GLuint mVao{ 0 };
	GLuint mPosVbo{ 0 };
	GLuint mUvVbo{ 0 };
	GLuint mNormalVbo{ 0 };
	GLuint mEbo{ 0 };
```
当然我们就去析构函数中删除这个vbo
```
Geometry::~Geometry()
{
	...
	if (mNormalVbo != 0) {
		glDeleteBuffers(1, &mNormalVbo);
	}
}
```
既然我们有了`normal`数据，创建并绑`vbo`到`vao`中
```
Geometry* Geometry::createBox(float size)
{
	...
	GLuint& posVbo = geometry->mPosVbo, uvVbo = geometry->mUvVbo, normalVbo = geometry->mNormalVbo;
	
	glGenBuffers(1, &normalVbo);
	glBindBuffer(GL_ARRAY_BUFFER, normalVbo);
	glBufferData(GL_ARRAY_BUFFER, sizeof(normals), normals, GL_STATIC_DRAW);

	//5.3  加入normal属性描述信息
	glBindBuffer(GL_ARRAY_BUFFER, normalVbo);
	glEnableVertexAttribArray(2);
	glVertexAttribPointer(2, 3, GL_FLOAT, GL_FALSE, sizeof(float) * 3, (void*)0);
	...
	return geometry;
}
```
### 验证法线数据
一个很好的验证方法：
通过将**法线值当作颜色输出**，因为normal也有x，y，z三个变量，可以当作颜色值的rgb输出
先去`vertex.glsl`用layout接一下法线数据，然后`out`给`fragment`
```
#version 460 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aUV;
layout (location = 2) in vec3 aNormal;

out vec2 uv;
out vec3 normal;

uniform mat4 transform;
uniform mat4 viewMatrix;
uniform mat4 projectionMatrix;

void main()
{
    vec4 position = vec4(aPos, 1.0);
    position = projectionMatrix * viewMatrix * transform * position;
    gl_Position = position;
    uv = aUV;
    normal = aNormal;
}
```
然后去`fragment.glsl`用`in`接一下法线，将颜色归一和夹值后输出为颜色
```
#version 330 core
out vec4 FragColor;

uniform float time;

in vec2 uv;
in vec3 normal;

uniform sampler2D sampler;

void main()
{
    //1 将vs中输入的normal做一下归一化
    vec3 normalN = normalize(normal);
    //2 将负数的情况直接清理为0，使用clamp函数
    vec3 normalColor = clamp(normalN, 0.0, 1.0);
    FragColor = vec4(normalColor, 1.0);

    //FragColor = texture(sampler, uv);
}
```

![输入图片说明](/imgs/2024-11-21/gVnhbLe8x10nZWKU.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg2NjgzNDM1NCwtMTE1MDk3OTM1LDcxMD
A4MDcyNCwtNzY3NTE0OTMxLDEwNTY1NTg3MTldfQ==
-->