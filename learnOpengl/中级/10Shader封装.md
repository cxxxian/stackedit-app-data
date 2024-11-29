现在我们有三种不同的光照类型：
1. 平行光
2. 点光源
3. 聚光灯

我们希望在`shader`中封装一下光照的类型
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM0NzIwODI2MV19
-->