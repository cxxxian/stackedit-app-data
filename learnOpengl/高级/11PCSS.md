![输入图片说明](/imgs/2025-02-27/lRBIhSWFrGCcSUb8.png)

![输入图片说明](/imgs/2025-02-27/w0FL3CzGIfJhf5sF.png)

![输入图片说明](/imgs/2025-02-27/S7mc54dvcXWWecpz.png)

![输入图片说明](/imgs/2025-02-27/2YMnTtkuLBebcBwW.png)

![输入图片说明](/imgs/2025-02-27/9A1yz7mHFgNrTGeT.png)

![输入图片说明](/imgs/2025-02-27/wy9bahSOFezdCk7P.png)

观察上图，其实在同一个遮挡物前，算出来的`penumbra`是一样的，`penumbra`只是用来控制`pcf`采样的大小（`pcfRadius`），
`penumbra`越大，说明采样半径要越大，
`penumbra`越小，说明采样半径要越小

![输入图片说明](/imgs/2025-02-27/fJZYhSY3QSORzalY.png)

![输入图片说明](/imgs/2025-02-27/mhWpKPWNGP3ZD2li.png)

依旧是相似三角形的思想，此处不可以直接乘上`projectionMatrix`，因为我们`light`的位置到`shadowMap`之间的距离`n`，并不在视景体之内

所以我们只能乘上`viewMatrix`将其转到光源坐标系下进行操作

此处的`lightSpacePosition.z`是负值，因为我们是朝向`-z`轴的，所以需要加上负号再进行运算
最后的得到一个`searchRadius`，但是注意，因为我们没有乘`projectionMatrix`，此时的`searchRadius`是光源相机空间下的度量尺度，我们需要转到`shadowMap`的`uv`空间，所以除上`frustumSize`，这个是`shadowMap`的`uv`尺度得来的

![输入图片说明](/imgs/2025-02-27/ldDe3nV4hcxRTQP4.png)

这里求得两个参数，`blockerNum`是采样点中有几个点是遮挡物的范围，`blockerSumDepth`是几个被遮挡的总深度值，
通过这两个参数就可以求得遮挡物的平均深度值

![输入图片说明](/imgs/2025-02-27/F4PYwiWOv6he3T7W.png)

# 实现
## 准备工作：搭建实验环境，注意参数调节（尤其是光源位置，方向以及光源视景体大小）
一个平面和一个大平面
```cpp
void prepare() {
	fbo = new Framebuffer(WIDTH, HEIGHT);

	renderer = new Renderer();
	sceneOff = new Scene();
	scene = new Scene();

	//pass 01
	auto geo = Geometry::createPlane(1.0, 1.0);
	auto upPlaneMat = new PhongShadowMaterial();
	upPlaneMat->mDiffuse = new Texture("assets/textures/grass.jpg", 0, GL_SRGB_ALPHA);
	upPlaneMat->mShiness = 32;
	upPlane = new Mesh(geo, upPlaneMat);
	upPlane->rotateX(-90.0f);
	sceneOff->addChild(upPlane);

	auto groundGeo = Geometry::createPlane(10, 10);
	auto groundMat = new PhongShadowMaterial();
	groundMat->mDiffuse = new Texture("assets/textures/grass.jpg", 0, GL_SRGB_ALPHA);
	groundMat->mShiness = 32;
	auto groundMesh = new Mesh(groundGeo, groundMat);
	groundMesh->setPosition(glm::vec3(0.0, 0.0, 0.0f));
	groundMesh->rotateX(-90.0f);
	sceneOff->addChild(groundMesh);
	...
	dirLight = new DirectionalLight();
	dirLight->setPosition(glm::vec3(3.0f, 15.0f, 0.0f));
	dirLight->rotateX(-90.0f);
	dirLight->mSpecularIntensity = 0.5f;

	ambLight = new AmbientLight();
	ambLight->mColor = glm::vec3(0.1f);

}
```
## 1 加入dBlocker所需的uniform参数
`lightSize`：光源尺寸（可调整）
`frustrum`：近平面大小
`nearPlane`：近平面到相机距离

## 2 shader中加入findBlocker函数，计算dBlocker，如果没有阻挡物则返回-1（说明不在阴影中）
需要更新的`uniform`：
`uniform mat4 lightViewMatrix;`
`uniform float lightSize;`
`uniform float frustum;`
`uniform float nearPlane;`

## 3 将计算的dBlocker绘制在屏幕上进行观察
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0OTUzNDcwMDEsLTc3Njg3NDA2OSwtMj
AxNzM3NTcyNywtMTI5Mzc1NjA4LC0yNjE5OTI2MjQsMTQyMTYy
MzI4OCw2NDk0OTA1MzYsLTUxMTA0MDYzNywxMTk0MTE2NDIxLD
Y4NTA4NjczOCwtMjg0NjY0OTE5XX0=
-->