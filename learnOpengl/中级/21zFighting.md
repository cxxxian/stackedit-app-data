![输入图片说明](/imgs/2025-02-08/dr4v1kG4lwy4eRf6.png)

![输入图片说明](/imgs/2025-02-08/14hIsaHBBWQk5lGE.png)

![输入图片说明](/imgs/2025-02-08/YdiqQlBlB4j2JvMB.png)


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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg3MjE3MTQ0OSwtNTQyNDc3NzQzLDE2OT
I4NDkyOTgsMjA5NDk0NDkxXX0=
-->