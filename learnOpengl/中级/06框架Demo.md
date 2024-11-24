利用我们的框架做个`demo`
做两个不同`material`的`mesh`
都做圆形，所以可以用同一种`geometry`
两种不同材质，所以要声明两种`material`，
利用`geometry`和`material`生成两种`mesh`
将`mesh01`和`mesh02`放到数组中
```
void prepare() {
    renderer = new Renderer();

    //1 创建geometry
    auto geometry = Geometry::createSphere(1.5f);

    //2 创建一个material并且配置参数
    auto material01 = new PhongMaterial();
    material01->mShiness = 32.0f;
    material01->mDiffuse = new Texture("assets/textures/goku.jpg", 0);

    auto material02 = new PhongMaterial();
    material02->mShiness = 32.0f;
    material02->mDiffuse = new Texture("assets/textures/wall.jpg", 0);

    //3 生成mesh
    auto mesh01 = new Mesh(geometry, material01);
    auto mesh02 = new Mesh(geometry, material02);
    mesh02->setPosition(glm::vec3(3.0f, 0.0f, 0.0f));

    meshes.push_back(mesh01);
    meshes.push_back(mesh02);

    dirLight = new DirectionalLight();
    ambLight = new AmbientLight();
    ambLight->mColor = glm::vec3(0.1f);
}
```
通过以上操作我们就已经生成了两个`mesh`，并将`mesh02`往`x`轴正方向移动了`3`单位长度

想让`mesh02`转起来，那么只要调用之前在`mesh`设计的`rotate`函数即可，在`while`中持续调用，持续旋转
而且由于我们之前是根据本地坐标系设计的`modelMatrix`，所以会根据自己球体的中心进行旋转
```
int main() {
	...
    while (app->update()) {

        meshes[1]->rotateX(1.0f);
        meshes[1]->rotateY(5.0f);

        cameraControl->update();
        renderer->render(meshes, camera, dirLight, ambLight);

    }
	...
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAxMzM5MzEzMSwxNzg4NzU3ODMzXX0=
-->