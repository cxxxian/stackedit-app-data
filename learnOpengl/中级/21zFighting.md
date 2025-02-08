## zFighting现象解释

![输入图片说明](/imgs/2025-02-08/dr4v1kG4lwy4eRf6.png)

## 平行条件下过于接近引起的在Fighting

![输入图片说明](/imgs/2025-02-08/14hIsaHBBWQk5lGE.png)

![输入图片说明](/imgs/2025-02-08/YdiqQlBlB4j2JvMB.png)

在`main.cpp`中，我们准备两种间隔十分紧密的平面，就会出现`zFighting`现象
```cpp
void prepare() {
	renderer = new Renderer();
	scene = new Scene();

	auto geometry = Geometry::createPlane(5.0f, 5.0f);
	auto materialA = new PhongMaterial();
	materialA->mDiffuse = new Texture("assets/textures/goku.jpg", 0);
	auto meshA = new Mesh(geometry, materialA);

	scene->addChild(meshA);

	auto materialB = new PhongMaterial();
	materialB->mDiffuse = new Texture("assets/textures/box.png", 0);
	auto meshB = new Mesh(geometry, materialB);
	meshB->setPosition(glm::vec3(2.0f,0.5f, -0.0000001f));
	scene->addChild(meshB);
	
	...
}
```
去到`render.cpp`中，开启`PolygonOffset`，并将第二个面片向后偏移`1.0f`即可解决
```cpp
//2 遍历object的子节点，对每个子节点都需要调用renderObject
auto children = object->getChildren();
for (int i = 0; i < children.size(); i++) {
	if (i == 1) {
		glEnable(GL_POLYGON_OFFSET_FILL);
		glPolygonOffset(0.0f, 1.0f);
	}
	else {
		glDisable(GL_POLYGON_OFFSET_FILL);
	}
	renderObject(children[i], camera, dirLight, ambLight);
}
```

![输入图片说明](/imgs/2025-02-08/EzXPwQbidRjGTbOz.png)

## 倾斜条件下导致的zFighting

![输入图片说明](/imgs/2025-02-08/77m3JVKe7UL0wDRI.png)

![输入图片说明](/imgs/2025-02-08/CQ8TQQhqYYQLsazO.png)

![输入图片说明](/imgs/2025-02-08/L8zhRA9FKeJ4WbFm.png)

如下图：

![输入图片说明](/imgs/2025-02-08/iX3hdCv4SGq9EhNw.png)

倾斜后，远处的地方精度不高，就会出现`zFighting`现象，我们通过计算偏导可以判断哪部分离我们远，哪部分近

![输入图片说明](/imgs/2025-02-08/jGhoyrGSKeHLElCf.png)

如以下代码：
前者`factor`参数，可以做到离我们近的少往后偏移一点，离我们远的多往后偏移一点
后者`units`参数，整体向后面平移

![输入图片说明](/imgs/2025-02-08/6MkP1dRlHh99b4wm.png)

## polygonOffset封装
`material.h`
```cpp
public:
	//polygonOffset
	bool	mPolygonOffset{ false };
	unsigned int	mPolygonType{ GL_POLYGON_OFFSET_FILL };
	float	mFactor{ 0.0f };
	float	mUnit{ 0.0f };
```
在`render.cpp`中，我们每一帧需要将`polygon`相关状态给设为`false`，什么时候要用什么时候再打开
```cpp
void Renderer::render(
	Scene* scene, 
	Camera* camera,
	DirectionalLight* dirLight,
	AmbientLight* ambLight
) {
	////1 设置当前帧绘制的时候，opengl的必要状态机参数
	glEnable(GL_DEPTH_TEST);
	glDepthFunc(GL_LESS);
	glDepthMask(GL_TRUE);

	glDisable(GL_POLYGON_OFFSET_FILL);
	glDisable(GL_POLYGON_OFFSET_LINE);

	...
}
```
在`render.cpp`中和先前检测深度状态一样，开始检测`polygonOffset`状态
```cpp
//针对单个object进行渲染
void Renderer::renderObject(
	Object* object,
	Camera* camera,
	DirectionalLight* dirLight,
	AmbientLight* ambLight
) {
		...
		//1 检测深度状态
		if (material->mDepthTest) {
			glEnable(GL_DEPTH_TEST);
			glDepthFunc(material->mDepthFunc);
		}
		else {
			glDisable(GL_DEPTH_TEST);
		}

		if (material->mDepthWrite) {
			glDepthMask(GL_TRUE);
		}
		else {
			glDepthMask(GL_FALSE);
		}
		//2 检测polygonOffset状态
		if (material->mPolygonOffset) {
			glEnable(material->mPolygonType);
			glPolygonOffset(material->mFactor, material->mUnit);
		}
		else {
			glDisable(GL_POLYGON_OFFSET_FILL);
			glDisable(GL_POLYGON_OFFSET_LINE);
		}
		...
}
```
以上就准备好了所有工作，剩下的只需要去`main.cpp`中调用，
由于大面片旋转更好观察到`zFighting`现象，将面片尺寸设大一点，然后针对`B`面片应用`polygon`相关的参数设置
```cpp
void prepare() {
	renderer = new Renderer();
	scene = new Scene();

	//auto geometry = Geometry::createPlane(5.0f, 5.0f);
	auto geometry = Geometry::createPlane(500.0f, 500.0f);
	auto materialA = new PhongMaterial();
	materialA->mDiffuse = new Texture("assets/textures/goku.jpg", 0);
	auto meshA = new Mesh(geometry, materialA);
	meshA->rotateX(-88.0f);

	scene->addChild(meshA);

	auto materialB = new PhongMaterial();
	materialB->mDiffuse = new Texture("assets/textures/box.png", 0);
	auto meshB = new Mesh(geometry, materialB);
	meshB->setPosition(glm::vec3(0.0f,0.0f, -0.1f));
	meshB->rotateX(-88.0f);
	materialB->mPolygonOffset = true;
	materialB->mFactor = 1.0f;
	materialB->mUnit = 1.0f;
	scene->addChild(meshB);
	...
}
```
完美解决问题

![输入图片说明](/imgs/2025-02-08/cfMpyU6g0wvbxlbz.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4NDEyNzA4OTcsLTYyNDYwNTk1Niw0ND
gxMDQxMzYsMTU5NzE0NDY2OSwtNzE0MjA2MjA5LDE5NjIwNjA4
NzEsLTU1NTM0MDc4OCwtNTQyNDc3NzQzLDE2OTI4NDkyOTgsMj
A5NDk0NDkxXX0=
-->