![输入图片说明](/imgs/2025-02-08/dr4v1kG4lwy4eRf6.png)

![输入图片说明](/imgs/2025-02-08/14hIsaHBBWQk5lGE.png)

![输入图片说明](/imgs/2025-02-08/YdiqQlBlB4j2JvMB.png)

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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY3ODU3ODc2NywtNTQyNDc3NzQzLDE2OT
I4NDkyOTgsMjA5NDk0NDkxXX0=
-->