#version 460 core
out vec4 FragColor;

uniform float time;

in vec2 uv;
in vec3 normal;
in vec3 worldPosition;

uniform sampler2D sampler;//diffuse贴图采样器
uniform sampler2D specularMaskSampler;//specularMask贴图采样器


uniform vec3 ambientColor;

uniform float shiness;

//相机世界位置
uniform vec3 cameraPosition;

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
    float diffuse = clamp(dot(-lightDirN, normalN), 0.0, 1.0);
    vec3 diffuseColor = spotLight.color * diffuse * objectColor;

    //计算specular
    //防止背面光效果
    float dotResult = dot(-lightDirN, normalN);
    float flag = step(0.0, dotResult);

    vec3 lightReflect = normalize(reflect(lightDirN, normalN));
    float specular = clamp(dot(lightReflect, -viewDir), 0.0, 1.0);
    //控制光斑大小
    specular = pow(specular, shiness);

    float specularMask = texture(specularMaskSampler, uv).r;

    vec3 specularColor = spotLight.color * specular * flag * spotLight.specularIntensity * specularMask;

    //环境光计算
    vec3 ambientColor = objectColor * ambientColor;

    vec3 finalColor = (diffuseColor + specularColor) * spotIntensity + ambientColor;


    FragColor = vec4(finalColor, 1.0);
}
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE5ODg4NDI1MF19
-->