现在理解一下我们要做什么
我们的`shader`代码过于冗长，我希望可以创建一个`commonLight.glsl`，专门用来放光照的结构声明以及计算
然后我们去`phong.frag`使用`include`
就像我们正常的`.cpp`用`include`得到`.h`一样

![输入图片说明](/imgs/2025-02-24/qFPC4IMXHcmwgY0w.png)

观察一下现在的文件夹结构，我们希望`advanced`下的`phong.frag`可以`include`到`common`文件夹下的`commonLight.glsl`

相应的`commonLight.glsl`文件如下：
全都是有关光照的代码
```glsl
struct DirectionalLight{
	vec3 direction;
	vec3 color;
	float specularIntensity;
	float intensity;
};

struct PointLight{
	vec3 position;
	vec3 color;
	float specularIntensity;

	float k2;
	float k1;
	float kc;
};

struct SpotLight{
	vec3 position;
	vec3 targetDirection;
	vec3 color;
	float outerLine;
	float innerLine;
	float specularIntensity;
};

//计算漫反射光照
vec3 calculateDiffuse(vec3 lightColor, vec3 objectColor, vec3 lightDir, vec3 normal){
	float diffuse = clamp(dot(-lightDir, normal), 0.0,1.0);
	vec3 diffuseColor = lightColor * diffuse * objectColor;

	return diffuseColor;
}

//计算镜面反射光照
vec3 calculateSpecular(vec3 lightColor, vec3 lightDir, vec3 normal, vec3 viewDir, float intensity){
	//1 防止背面光效果
	float dotResult = dot(-lightDir, normal);
	float flag = step(0.0, dotResult);
	vec3 lightReflect = normalize(reflect(lightDir,normal));

	//2 jisuan specular
	float specular = max(dot(lightReflect,-viewDir), 0.0);

	//3 控制光斑大小
	specular = pow(specular, shiness);

	//float specularMask = texture(specularMaskSampler, uv).r;

	//4 计算最终颜色
	vec3 specularColor = lightColor * specular * flag * intensity;

	return specularColor;
}

vec3 calculateSpotLight(SpotLight light, vec3 normal, vec3 viewDir){
	//计算光照的通用数据
	vec3 objectColor  = texture(sampler, uv).xyz;
	vec3 lightDir = normalize(worldPosition - light.position);
	vec3 targetDir = normalize(light.targetDirection);

	//计算spotlight的照射范围
	float cGamma = dot(lightDir, targetDir);
	float intensity =clamp( (cGamma - light.outerLine) / (light.innerLine - light.outerLine), 0.0, 1.0);

	//1 计算diffuse
	vec3 diffuseColor = calculateDiffuse(light.color,objectColor, lightDir,normal);

	//2 计算specular
	vec3 specularColor = calculateSpecular(light.color, lightDir,normal, viewDir,light.specularIntensity); 

	return (diffuseColor + specularColor)*intensity;
}

vec3 calculateDirectionalLight(vec3 objectColor, DirectionalLight light, vec3 normal ,vec3 viewDir){
	light.color *= light.intensity;

	//计算光照的通用数据
	vec3 lightDir = normalize(light.direction);

	//1 计算diffuse
	vec3 diffuseColor = calculateDiffuse(light.color,objectColor, lightDir,normal);

	//2 计算specular
	vec3 specularColor = calculateSpecular(light.color, lightDir,normal, viewDir,light.specularIntensity); 

	return diffuseColor + specularColor;
}

vec3 calculatePointLight(vec3 objectColor, PointLight light, vec3 normal ,vec3 viewDir){
	//计算光照的通用数据
	vec3 lightDir = normalize(worldPosition - light.position);

	//计算衰减
	float dist = length(worldPosition - light.position);
	float attenuation = 1.0 / (light.k2 * dist * dist + light.k1 * dist + light.kc);

	//1 计算diffuse
	vec3 diffuseColor = calculateDiffuse(light.color,objectColor, lightDir,normal);

	//2 计算specular
	vec3 specularColor = calculateSpecular(light.color, lightDir,normal, viewDir,light.specularIntensity); 

	return (diffuseColor + specularColor)*attenuation;
}
```
然后对应的`phong.frag`把以上部分删除即可
并且在`phong.frag`这样写就好了`#include "../common/commonLight.glsl"`

但是问题来了，`shader`并看不懂这个代码
回忆一下我们是怎么编译`shader`的，是将`vert`和`frag`作为字符串进行编译的，
那么我们在读取`phong.frag`时，只要将`include`作为关键字，当读取到`include`的时候，我们就把`commonLight.glsl`内的内容替换成`#include "../common/commonLight.glsl"`

所以我们去到`shader.h`中创建一个新方法：
`std::string loadShader(const std::string& filePath);`
实现如下：
用了很多`stl`语法，慢慢来盘一下
```cpp
std::string Shader::loadShader(const std::string& filePath) {
	std::ifstream file(filePath);
	std::stringstream shaderStream;
	std::string line;

	while (std::getline(file, line)) {
		//判断是否含有#include
		if (line.find("#include") != std::string::npos) {
			//找到include包含的文件路径
			auto start = line.find("\"");
			auto end = line.find_last_of("\"");
			std::string includeFile = line.substr(start + 1, end - start - 1);

			//找到当前文件的文件目录
			auto lastSlashPos = filePath.find_last_of("/\\");
			auto folder = filePath.substr(0, lastSlashPos + 1);
			auto totalPath = folder + includeFile;
			shaderStream << loadShader(totalPath);
		}
		//如果不含有#include就将line装入shaderStream
		else {
			shaderStream << line << "\n";
		}
	}

	return shaderStream.str();
}
```
`std::ifstream file(filePath);`读取输入的文件流，根据地址进行打开
`std::stringstream shaderStream;`字符串流
`std::string line;`用来记录每一行的字符串
`while (std::getline(file, line))`然后我们就可以循环地去读取每一行字符串，判断有没有遇到`#include`
`if (line.find("#include") != std::string::npos)`中的这个`std::string::npos`就代表没有找到的意思

以上是在`shader`中进行字符串的读取
然后我们找到了`#include`之后，我们得到的是一个相对路径，如下：
`#include "../common/commonLight.glsl"`

所以我们还要找到头进行拼接，而头其实就可以通过`filePath`得到
例如：
`mPhongShader = new Shader("assets/shaders/advanced/phong.vert", "assets/shaders/advanced/phong.frag");`
我们要得到的头部就是
`"assets/shaders/advanced/`
所以就用这样找：
`auto lastSlashPos = filePath.find_last_of("/\\");`
这里的`/\\`是代表最后一个是`/`或者`\`，因为根据每个人书写规范不同，两种写法都有可能，`\\`用到了转义字符
最后拼出来的完整路径就是
`assets/shaders/advanced/phong.vert", "assets/shaders/advanced/common/commonLight.glsl`
很重要的就是要记得`substr`包前不包后

最后递归调用`loadShader`，如果下一层还有`#include`就持续递归使用
`shaderStream << loadShader(totalPath);`

以下是原本的构造函数，然后我们的新方法`loadShader`以及做好了字符串处理，所以我们可以进行调用
```cpp
Shader::Shader(const char* vertexPath, const char* fragmentPath) {
	//声明装入shader代码字符串的两个string
	std::string vertexCode;
	std::string fragmentCode;

	//声明用于读取vs跟fs文件的inFileStream
	std::ifstream vShaderFile;
	std::ifstream fShaderFile;

	//保证ifstream遇到问题的时候可以抛出异常
	vShaderFile.exceptions(std::ifstream::failbit | std::ifstream::badbit);
	fShaderFile.exceptions(std::ifstream::failbit | std::ifstream::badbit);
	try {
		//1 打开文件
		vShaderFile.open(vertexPath);
		fShaderFile.open(fragmentPath);
		
		//2 将文件输入流当中的字符串输入到stringStream里面
		std::stringstream vShaderStream, fShaderStream;
		vShaderStream << vShaderFile.rdbuf();
		fShaderStream << fShaderFile.rdbuf();

		//3 关闭文件
		vShaderFile.close();
		fShaderFile.close();

		//4 将字符串从stringStream当中读取出来，转化到code String当中
		vertexCode = vShaderStream.str();
		fragmentCode = fShaderStream.str();
	}
	...
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk4MjUzMTk5LDEwNjYzNTUyNzksNDM0ND
cwNDYzLDExNzA2MDAwNSwtMTg0OTAyNDY5MCw4NTk0MDY4NzUs
LTIwODg3NDY2MTJdfQ==
-->