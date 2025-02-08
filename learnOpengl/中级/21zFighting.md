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

![输入图片说明](/imgs/2025-02-08/77m3JVKe7UL0wDRI.png)

![输入图片说明](/imgs/2025-02-08/CQ8TQQhqYYQLsazO.png)

![输入图片说明](/imgs/2025-02-08/L8zhRA9FKeJ4WbFm.png)

倾斜后，远处的地方精度不高，就会出现`zFighting`现象，我们通过计算偏导可以判断哪部分离我们远，哪部分近

![输入图片说明](/imgs/2025-02-08/jGhoyrGSKeHLElCf.png)

如以下代码：
前者`factor`参数，可以做到离我们近的少往后偏移一点，离我们远的多往后偏移一点
后者`units`参数，整体向后面平移

![输入图片说明](/imgs/2025-02-08/6MkP1dRlHh99b4wm.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMTY0OTkxNjMsLTcxNDIwNjIwOSwxOT
YyMDYwODcxLC01NTUzNDA3ODgsLTU0MjQ3Nzc0MywxNjkyODQ5
Mjk4LDIwOTQ5NDQ5MV19
-->