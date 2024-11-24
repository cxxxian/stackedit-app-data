利用我们的框架做个demo
做两个不同material的mesh
都做圆形，所以可以用同一种geometry
两种不同材质，所以要声明两种material，
利用geometry和material生成liang'z
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTExOTUyNDU5MywxNzg4NzU3ODMzXX0=
-->