# 概念

![输入图片说明](/imgs/2025-02-25/bt5KiPgBKRaSixIZ.png)

从光源相机视角：
`shadowMap`由于可能分辨率不够，玩家视角的三个像素对应到`shadowMap`可能是同一个像素，（也可能玩家视角离得近，自然看到的像素就更多）
但是`shadowMap`肯定没办法记录那么多个，所以会选取中间那个进行存储，
那么从玩家视角下，很明显深度`d0 > d1 > d2`，所以`d0`会被误认为遮挡

![输入图片说明](/imgs/2025-02-25/ah8tjNTdtm30xfea.png)

所以`bias`不是一个固定值，可以暴露出来给美术人员调整，不同场景需要不同的`bias`

# 代码实现
## 1.shader中加入uniform变量bias，并且在阴影判断中使用
用`selfDepth - bias`，就可以把观察到的像素向光源推进`bias`个距离，消除子遮挡现象
```glsl
uniform float bias;
...
float calculateShadow(){
	...
	float shadow = (selfDepth - bias) > closestDepth? 1.0:0.0;
	return shadow;
}
```
相应的我们要在`cpu`端设计参数并传输，
在`phongShadowMaterial.h`设计参数`mBias`
```cpp
public:
	...
	float		mBias{ 0.0f };
```
然后就到`renderer.cpp`中的`renderObject`找到对应`case`进行传输即可
```cpp
case MaterialType::PhongShadowMaterial: {
	...
	shader->setFloat("bias", phongShadowMat->mBias);
}
	break;
```
## 2.在IMGUI中加入对bias的调节

来到`mian.cpp`中，把`mat`提到外面作为全局变量（因为`imgui`拿到这个变量），而且地面和几何体都是用的`PhongShadowMaterial`，所以统一即可
```cpp
PhongShadowMaterial* mat = nullptr;
void prepare() {
	...
	mat = new PhongShadowMaterial();

	//pass 01
	auto geo = Geometry::createBox(2.0);
	mat->mDiffuse = new Texture("assets/textures/parallax/bricks.jpg", 0, GL_SRGB_ALPHA);

	mat->mShiness = 32;
	auto mesh = new Mesh(geo, mat);
	sceneOff->addChild(mesh);

	auto groundGeo = Geometry::createPlane(10, 10);
	mat->mDiffuse = new Texture("assets/textures/grass.jpg", 0, GL_SRGB_ALPHA);
	mat->mShiness = 32;

	auto groundMesh = new Mesh(groundGeo, mat);
	groundMesh->setPosition(glm::vec3(0.0, 0.0, 0.0f));
	groundMesh->rotateX(-90.0f);
	sceneOff->addChild(groundMesh);

	...
}
```
`%.3f`这个参数值是`format`，代表滑动滑块移动的最小单位是`10`的`-3`次方

![输入图片说明](/imgs/2025-02-25/Sq0j5zehEMyzInOB.png)

由于`bias`非常敏感，所以我们使用`%.4f`
```cpp
void renderIMGUI() {
	...
	ImGui::Begin("MaterialEditor");
	ImGui::SliderFloat("Bias:", &mat->mBias, 0.0f, 0.01f, "%.4f");
	ImGui::End();
	...
}
```
`0.0001`就可以解决自遮挡问题了

![输入图片说明](/imgs/2025-02-25/C50szzfw9RfHywrS.png)

当调到`0.01`时，发现阴影不对了，这是由于`bias`的值过大了，把原本该被遮挡的也认为是无阴影的

![输入图片说明](/imgs/2025-02-25/aQXWDrcd3HdaOTUC.png)

# 自适应bias

![输入图片说明](/imgs/2025-02-26/VudYjCyKTwhcSVur.png)

随着夹角变大，看上面的图，`sin`值也跟着变大

![输入图片说明](/imgs/2025-02-26/WtkzxcZNSvqa6MdK.png)

这里其实利用的是`1.0 - cos`，在数学上这个算法是错误的，但是这里我们这样即可，只需要表现出随着`sin`增大，`bias`也要跟着变大
实现如下：
设计方法`getBias`，此处使用`max(bias * (1.0 - dot(normalN, lightDirN)), 0.0005)`，是为了防止`bias`为`0`的情况，最小为`0.0005`
```glsl
float getBias(vec3 normal, vec3 lightDir){
	vec3 normalN = normalize(normal);
	vec3 lightDirN = normalize(lightDir);

	return max(bias * (1.0 - dot(normalN, lightDirN)), 0.0005);
}


float calculateShadow(vec3 normal, vec3 lightDir){
	...
	float shadow = (selfDepth - getBias(normal, lightDir)) > closestDepth? 1.0:0.0;
	return shadow;
}
void main()
{
	...
	float shadow = calculateShadow(normal, -directionalLight.direction);
	
	vec3 finalColor = result * (1.0 - shadow) + ambientColor;
	FragColor = vec4(finalColor,alpha * opacity);
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg4NjEyMDYxMywtMTA4NDg5MDI5OCwtOT
g5Mzg2MzEzLC0xNTQ2MDkwMDQ2LC0xNDE5MDI1ODkwLC0xMjIz
MTg3OTYyLC0yMDg4NzQ2NjEyXX0=
-->