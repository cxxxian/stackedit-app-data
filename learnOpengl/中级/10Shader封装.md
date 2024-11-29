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
当然
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNjk0MTAwMjcsLTEyOTY4NTY2MzhdfQ
==
-->