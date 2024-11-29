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

# 方法的封装
```
//计算漫反射光照
vec3 calculateDiffuse(vec3 lightColor, vec3 objectColor, vec3 lightDir, vec3 normal){
    float diffuse = clamp(dot(-lightDir, normal), 0.0, 1.0);
    vec3 diffuseColor = lightColor * diffuse * objectColor;

    return diffuseColor;
}
```
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

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM5MTc0MDkzNCwzNzkxMTg4NDEsLTEyOT
Y4NTY2MzhdfQ==
-->