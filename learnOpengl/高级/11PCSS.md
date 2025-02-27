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
一个平面和一个大平面，设计一个函数`transform()`循环调用，使小平面可以反复上下运动
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

bool goUp = true;
void transform() {
	auto pos = upPlane->getPosition();
	if (goUp) {
		pos.y += 0.05;
		upPlane->setPosition(pos);
		if (pos.y > 10) {
			goUp = false;
		}
	}
	else {
		pos.y -= 0.05;
		upPlane->setPosition(pos);
		if (pos.y < 0.2) {
			goUp = true;
		}
	}
}
int main() {
	...
	prepare();
	initIMGUI();
	while (glApp->update()) {
		cameraControl->update();
		transform();
		...
	}
	...
}
```
此时会发现无论上下怎么运动，阴影都一动不动（不合常理），因为我们之前只用了`pcf`，并没有远近距离导致阴影变化软硬一说

![输入图片说明](/imgs/2025-02-27/edoSnJYhd6RYk6r4.png)

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

`lightSize`我们可以设计为外部可调控，所以在`shadow.h`中设计参数
```cpp
public:
	...
	float	mLightSize{ 0.04 };
```
还记得我们计算`dBlocker`需要用到的参数，要一个在光源坐标系下的位置，用来判断深度，这个无需乘上`projectionMatrix`，
但是我们之前设计的`lightMatrix`是通过`lightProjection * lightView`得到的
所以我们要重新在`vert`设计一个`lightViewMatrix`，并传到`frag`中
区分一下：
`lightSpaceClipCoord`是乘上投影矩阵的
`lightSpacePosition`没有，是一个在光源坐标系下的位置
```glsl
out vec3 lightSpacePosition;
uniform mat4 lightViewMatrix;
void main()
{
...
	lightSpaceClipCoord = lightMatrix * transformPosition;
	lightSpacePosition = (lightViewMatrix * transformPosition).xyz;
}
```
然后在`phongShadow.frag`接收`vert`的`lightSpacePosition`，
并设计三个参数，以及实现`findBlocker`
```glsl
in vec3 lightSpacePosition;

//PCSS相关参数
uniform float lightSize;
uniform float frustum;
uniform float nearPlane;

float findBlocker(vec3 lightSpacePosition, vec2 shadowUV, float depthReceiver, vec3 normal, vec3 lightDir){

	poissonDiskSamples(shadowUV);

	//根据相似三角形得到的在shadowMap上对应的光源长度，此时还不是以uv为单位
	float searchRadius = (-lightSpacePosition.z - nearPlane) / (-lightSpacePosition.z) * lightSize;
	//从光源坐标系转uv
	float searchRadiusUV = searchRadius / frustum;

	float blockerNum = 0.0;
	float blockerSumDepth = 0.0;
	for(int i = 0; i < NUM_SAMPLES; i++){
		float sampleDepth = texture(shadowMapSampler, shadowUV + disk[i] * searchRadiusUV).r;
		if(depthReceiver - getBias(normal, lightDir) > sampleDepth){
			blockerNum += 1.0;
			blockerSumDepth += sampleDepth;
		}
	}

	return blockerNum != 0.0? blockerSumDepth / blockerNum : -1;
}
```
解释一下：
`float sampleDepth = texture(shadowMapSampler, shadowUV + disk[i] * searchRadiusUV).r;`
这句话，`searchRadiusUV`是我们通过`float searchRadiusUV = searchRadius / frustum`算出来的，与光源长度挂钩
所以通过`shadowUV + disk[i] * searchRadiusUV`进行采样，就可以根据光源长度更改采样范围
最后返回的是一个点周围的`blocker`的平均深度
## 3 将计算的dBlocker绘制在屏幕上进行观察
```glsl
void main()
{
	...
	//---------临时实验---------------
	//1 找到当前像素在光源空间内的NDC坐标
	vec3 lightNDC = lightSpaceClipCoord.xyz/lightSpaceClipCoord.w;

	//2 找到当前像素在ShadowMap上的uv
	vec3 projCoord = lightNDC * 0.5 + 0.5;
	vec2 uv = projCoord.xy;
	float depth = projCoord.z;
	
	float dBlocker = findBlocker(lightSpacePosition, uv, depth, normal, -directionalLight.direction);
	if(dBlocker < 0.0){
		FragColor = vec4(1.0, 0.0, 0.0, 1.0);
	}else{
		FragColor = vec4(dBlocker, dBlocker, dBlocker, 1.0);
	}

	//FragColor = vec4(finalColor,alpha * opacity);
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg1NzQ3NjQyNiw4NDQwMTM0MTMsMTU3Nz
c1ODQ3MywtMTc2NzYzMTgyMSwtMzU3NzczNjc4LC0xOTY3NDI3
NjM4LC03NzY4NzQwNjksLTIwMTczNzU3MjcsLTEyOTM3NTYwOC
wtMjYxOTkyNjI0LDE0MjE2MjMyODgsNjQ5NDkwNTM2LC01MTEw
NDA2MzcsMTE5NDExNjQyMSw2ODUwODY3MzgsLTI4NDY2NDkxOV
19
-->