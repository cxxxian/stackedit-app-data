# 结构体的封装
现在我们有三种不同的光照类型：
1. 平行光
2. 点光源
3. 聚光灯

我们希望在`shader`中封装一下光照的类型
利用`struct`结构体，声明一个`SpotLight`结构体，把参数都设计在结构体中
```
...
struct SpotLight{
    vec3 position;
    vec3 targetDirection;
    vec3 color;
    float outerLine;
    float innerLine;
    float specularIntensity;
};
uniform SpotLight spotLight;


void main()
{
    ...
    vec3 lightDirN = normalize(worldPosition - spotLight.position);

    vec3 viewDir = normalize(worldPosition - cameraPosition);
    vec3 targetDirN = normalize(spotLight.targetDirection);

    //计算spotLight的照射范围
    float cGamma = dot(lightDirN, targetDirN);
    float spotIntensity = clamp((cGamma - spotLight.outerLine) / (spotLight.innerLine - spotLight.outerLine), 0.0, 1.0);

    //计算diffuse
    float diffuse = clamp(dot(-lightDirN, normalN), 0.0, 1.0);
    vec3 diffuseColor = spotLight.color * diffuse * objectColor;
    ...

    vec3 specularColor = spotLight.color * specular * flag * spotLight.specularIntensity * specularMask;
	...
}
```
当然在`shader`中改完参数名字，`render.cpp`的传参到`shader`中的这一步也要同步修改
```
case MaterialType::PhongMaterial: {
	...
	//光源参数的uniform更新
	shader->setVector3("spotLight.position", spotLight->GetPosition());
	shader->setVector3("spotLight.color", spotLight->mColor);
	shader->setVector3("spotLight.targetDirection", spotLight->mTargetDirection);
	shader->setFloat("spotLight.specularIntensity", spotLight->mSpecularIntensity);
	shader->setFloat("spotLight.innerLine", glm::cos(glm::radians(spotLight->mInnerAngle)));
	shader->setFloat("spotLight.outerLine", glm::cos(glm::radians(spotLight->mOuterAngle)));
	...
		}
			break;
```
同理可以补上平行光和点光源的结构体。
```
struct DirectionalLight{
    vec3 direction;
    vec3 color;
    float specularIntensity;
};

struct PointLight{
    vec3 position;
    vec3 color;
    float specularIntensity;

    float k1;
    float k2;
    float kc;
};
```

# 反射的封装
### calculateDiffuse
```
//计算漫反射光照
vec3 calculateDiffuse(vec3 lightColor, vec3 objectColor, vec3 lightDir, vec3 normal){
    float diffuse = clamp(dot(-lightDir, normal), 0.0, 1.0);
    vec3 diffuseColor = lightColor * diffuse * objectColor;

    return diffuseColor;
}
```
### calculateSpecular
暂时先把`specularMask`拿出了镜面反射光照中，因为不一定所有的物体都要有这个功能
```
//计算镜面反射光照
vec3 calculateSpecular(vec3 lightColor, vec3 lightDir, vec3 normal, vec3 viewDir, float intensity){
    //防止背面光效果
    float dotResult = dot(-lightDir, normal);
    float flag = step(0.0, dotResult);

    vec3 lightReflect = normalize(reflect(lightDir, normal));
    float specular = clamp(dot(lightReflect, -viewDir), 0.0, 1.0);
    //控制光斑大小
    specular = pow(specular, shiness);

    vec3 specularColor = lightColor * specular * flag * intensity;

    return specularColor;
}
```
至此`main`函数就只需要调用方法计算`diffuse`和`specular`即可
```
void main()
{
    vec3 result = vec3(0.0, 0.0, 0.0);

    //计算光照的通用数据
    vec3 objectColor = texture(sampler, uv).xyz;
    vec3 normalN = normalize(normal);
    vec3 lightDirN = normalize(worldPosition - spotLight.position);

    vec3 viewDir = normalize(worldPosition - cameraPosition);
    vec3 targetDirN = normalize(spotLight.targetDirection);

    //计算spotLight的照射范围
    float cGamma = dot(lightDirN, targetDirN);
    float spotIntensity = clamp((cGamma - spotLight.outerLine) / (spotLight.innerLine - spotLight.outerLine), 0.0, 1.0);

    //计算diffuse
    result += calculateDiffuse(spotLight.color, objectColor, lightDirN, normalN);
    
    //计算specular
    result += calculateSpecular(spotLight.color, lightDirN, normalN, viewDir, spotLight.specularIntensity);

    //环境光计算
    vec3 ambientColor = objectColor * ambientColor;

    //vec3 finalColor = (diffuseColor + specularColor) * spotIntensity + ambientColor;

    FragColor = vec4(result, 1.0);
}
```
由于此时没有用`specularMask`，所以整个物体都会被镜面反射

![输入图片说明](/imgs/2024-11-29/iHIcBq0YHKmWJ2iC.png)

# 光照计算的封装
### spotLight
在`phong.frag`设计一个函数用来计算`spotLight`的光照
```
vec3 calculateSpotLight(SpotLight light, vec3 normal, vec3 viewDir){
    //计算光照的通用数据
    vec3 objectColor = texture(sampler, uv).xyz;
    vec3 lightDir = normalize(worldPosition - spotLight.position);

    vec3 targetDir = normalize(spotLight.targetDirection);

    //计算spotLight的照射范围
    float cGamma = dot(lightDir, targetDir);
    float intensity = clamp((cGamma - light.outerLine) / (light.innerLine - light.outerLine), 0.0, 1.0);

    //计算diffuse
    vec3 diffuseColor = calculateDiffuse(light.color, objectColor, lightDir, normal);
    
    //计算specular
    vec3 specularColor = calculateSpecular(light.color, lightDir, normal, viewDir, light.specularIntensity);

    return (diffuseColor + specularColor) * intensity;

}
```
所以就能把原来在`main`函数中原本一堆的计算改为调用`calculateSpotLight`的计算
```
void main()
{
    vec3 result = vec3(0.0, 0.0, 0.0);

    //计算光照的通用数据
    vec3 objectColor = texture(sampler, uv).xyz;
    vec3 normalN = normalize(normal);
    vec3 lightDirN = normalize(worldPosition - spotLight.position);

    vec3 viewDir = normalize(worldPosition - cameraPosition);
    vec3 targetDirN = normalize(spotLight.targetDirection);

    result += calculateSpotLight(spotLight, normalN, viewDir);

    //环境光计算
    vec3 ambientColor = objectColor * ambientColor;

    vec3 finalColor = result + ambientColor;


    FragColor = vec4(finalColor, 1.0);
}
```
### directionalLight
并且这样我们就可以设计不同光源的计算方法，要用的时候替换即可
平行光计算方法如下：
在`phong.frag`中
```
vec3 calculateDirectionalLight(DirectionalLight light, vec3 normal, vec3 viewDir){
    //计算光照的通用数据
    vec3 objectColor = texture(sampler, uv).xyz;
    vec3 lightDir = normalize(light.direction);

    //计算diffuse
    vec3 diffuseColor = calculateDiffuse(light.color, objectColor, lightDir, normal);
    
    //计算specular
    vec3 specularColor = calculateSpecular(light.color, lightDir, normal, viewDir, light.specularIntensity);

    return diffuseColor + specularColor;
}
```
我们在`spotLight`的基础上，多声明`directionalLight`，可以弄出不同光源照射的效果

```
uniform SpotLight spotLight;
uniform DirectionalLight directionalLight;
```
到`main`函数中累加到`result`即可
```
void main()
{
    vec3 result = vec3(0.0, 0.0, 0.0);

    ...
    result += calculateSpotLight(spotLight, normalN, viewDir);
    result += calculateDirectionalLight(directionalLight, normalN, viewDir);

    //环境光计算
    vec3 ambientColor = objectColor * ambientColor;
    vec3 finalColor = result + ambientColor;
    FragColor = vec4(finalColor, 1.0);
}
```
但是注意到我们的`renderer`中的函数现在只传了`spotLight`的数据，并没有`directionalLight`的，要去`renderer.cpp`中补全
在方法声明的时候加入`DirectionalLight* dirLight`，并实现，将对应的参数传给`shader`
```
void Renderer::render(const std::vector<Mesh*>& meshes, Camera* camera, DirectionalLight* dirLight, SpotLight* spotLight, AmbientLight* ambLight){
	//光源参数的uniform更新
	//spotLight的更新
	shader->setVector3("spotLight.position", spotLight->GetPosition());
	shader->setVector3("spotLight.color", spotLight->mColor);
	shader->setVector3("spotLight.targetDirection", spotLight->mTargetDirection);
	shader->setFloat("spotLight.specularIntensity", spotLight-	>mSpecularIntensity);
	shader->setFloat("spotLight.innerLine", glm::cos(glm::radians(spotLight-	>mInnerAngle)));
	shader->setFloat("spotLight.outerLine", glm::cos(glm::radians(spotLight->mOuterAngle)));
	shader->setVector3("ambientColor", ambLight->mColor);

	//directionalLight的更新
	shader->setVector3("directionalLight.color", dirLight->mColor);
	shader->setVector3("directionalLight.direction", dirLight->mDirection);
	shader->setFloat("directionalLight.specularIntensity", dirLight->mSpecularIntensity);
}
```
最后回到`main.cpp`中补充平行光的声明和初始化即可
```
DirectionalLight* dirLight = nullptr;
SpotLight* spotLight = nullptr;
```
在`prepare`函数中利用`dirLight = new DirectionalLight()`构造，并设定方向参数
```
void prepare() {
    ...
    meshes.push_back(meshWhite);
    spotLight = new SpotLight();
    spotLight->setPosition(meshWhite->GetPosition());
    spotLight->mTargetDirection = glm::vec3(-1.0f, 0.0f, 0.0f);
    spotLight->mInnerAngle = 30.0f;
    spotLight->mOuterAngle = 45.0f;

    dirLight = new DirectionalLight();
    dirLight->mDirection = glm::vec3(1.0f);
	...
}
```
最后到`main`函数中，由于我们刚刚修改了`render`函数，需要将`dirLight`添加进其中
```
while (app->update()) {

    cameraControl->update();
    renderer->render(meshes, camera, dirLight, spotLight, ambLight);

}
```
输出的图像可以看到，照向`glm::vec3(-1.0f, 0.0f, 0.0f)`是`spotLight`

![输入图片说明](/imgs/2024-11-29/wBVo4BABbH6mpH8n.png)

照向`glm::vec3(1.0f)`是`dirLight`

![输入图片说明](/imgs/2024-11-29/jyyAKF9TBMvzOwlX.png)

### pointLight
在`phong.frag`中，声明一个`pointLight`
```
uniform SpotLight spotLight;
uniform DirectionalLight directionalLight;
uniform PointLight pointLight;
```
计算点光源强度代码：
```
vec3 calculatePointLight(PointLight light, vec3 normal, vec3 viewDir){
    //计算光照的通用数据
    vec3 objectColor = texture(sampler, uv).xyz;
    vec3 lightDir = normalize(worldPosition - light.position);
    
    //计算衰减
    float dist = length(worldPosition - light.position);
    float attenuation = 1.0 / (light.k2 * dist * dist + light.k1 * dist + light.kc);

    //计算diffuse
    vec3 diffuseColor = calculateDiffuse(light.color, objectColor, lightDir, normal);
    
    //计算specular
    vec3 specularColor = calculateSpecular(light.color, lightDir, normal, viewDir, light.specularIntensity);

    return (diffuseColor + specularColor) * attenuation;;
}
```
最后在`main`函数中调用`calculatePointLight`并将值累加到`result`中
```
void main()
{
    ...
    result += calculateSpotLight(spotLight, normalN, viewDir);
    result += calculateDirectionalLight(directionalLight, normalN, viewDir);
    result += calculatePointLight(pointLight, normalN, viewDir);

    //环境光计算
    vec3 ambientColor = objectColor * ambientColor;
    vec3 finalColor = result + ambientColor;

    FragColor = vec4(finalColor, 1.0);
}
```
老样子去到`renderer.cpp`中对`render`函数添加一个`pointLight`的参数，并实现传参数到`shader`的功能
```
void Renderer::render(const std::vector<Mesh*>& meshes, Camera* camera, DirectionalLight* dirLight, PointLight* pointLight, SpotLight* spotLight, AmbientLight* ambLight)
{
	...
	//光源参数的uniform更新
	//spotLight的更新
	shader->setVector3("spotLight.position", spotLight->GetPosition());
	shader->setVector3("spotLight.color", spotLight->mColor);
	shader->setVector3("spotLight.targetDirection", spotLight->mTargetDirection);
	shader->setFloat("spotLight.specularIntensity", spotLight->mSpecularIntensity);
	shader->setFloat("spotLight.innerLine", glm::cos(glm::radians(spotLight->mInnerAngle)));
	shader->setFloat("spotLight.outerLine", glm::cos(glm::radians(spotLight->mOuterAngle)));
	shader->setVector3("ambientColor", ambLight->mColor);

	//directionalLight的更新
	shader->setVector3("directionalLight.color", dirLight->mColor);
	shader->setVector3("directionalLight.direction", dirLight->mDirection);
	shader->setFloat("directionalLight.specularIntensity", dirLight->mSpecularIntensity);

	//pointLight的更新
	shader->setVector3("pointLight.color", pointLight->mColor);
	shader->setVector3("pointLight.position", pointLight->GetPosition());
	shader->setFloat("pointLight.specularIntensity", pointLight->mSpecularIntensity);
	shader->setFloat("pointLight.k2", pointLight->mK2);
	shader->setFloat("pointLight.k1", pointLight->mK1);
	shader->setFloat("pointLight.kc", pointLight->mKc);
	...
	}
}
```

最后到`main.cpp`中初始化`pointLight`和初始化对应参数即可
```
//灯光们
PointLight* pointLight = nullptr;
DirectionalLight* dirLight = nullptr;
SpotLight* spotLight = nullptr;
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDgyMTkwMTA2LDE3MDI5MDQzNTMsLTE3OT
Q4MjM2NCwtMTk3Mjg2NDEwOSwxMDAzMzAzNDIyLDIyMzM1NTkx
MCw1MTAwODUzMTgsMTcwOTAzNTU2MywtMzkxNzQwOTM0LDM3OT
ExODg0MSwtMTI5Njg1NjYzOF19
-->