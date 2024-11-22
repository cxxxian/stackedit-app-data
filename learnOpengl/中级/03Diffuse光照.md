在main.cpp中声明光照的参数，然后将这些参数搞到shader中
```
//平行光：参数（方向、光强）uniform变量形式
glm::vec3 lightDirection = glm::vec3(-1.0f, -1.0f, -1.0f);
glm::vec3 lightColor = glm::vec3(0.9f, 0.85f, 0.75f);
```
到`fragment.glsl`中声明参数，并且先暂时将`lightColor`直接输出验证是否正确
```
...
//光源参数
uniform vec3 lightDirection;
uniform vec3 lightColor;

void main()
{
    //1 将vs中输入的normal做一下归一化
    vec3 normalN = normalize(normal);
    //2 将负数的情况直接清理为0，使用clamp函数
    vec3 normalColor = clamp(normalN, 0.0, 1.0);
    FragColor = vec4(lightColor, 1.0);

    //FragColor = texture(sampler, uv);
}
```
之前设计的传输vec3不好用，设计一个专门传vector3向量的
```
void Shader::setVector3(const std::string& name, const glm::vec3 value)
{
    //1 通过名称拿到Uniform变量位置location
    GLuint location = GL_CALL(glGetUniformLocation(mProgram, name.c_str()));

    //2 通过location更新Uniform变量的值
    GL_CALL(glUniform3f(location, value.x, value.y, value.z));
}
```
到`render()`中调用方法并且绑定光照参数到`shader`中
```
void render(){
	...
    //光源参数的uniform更新
    shader->setVector3("lightDirection", lightDirection);
    shader->setVector3("lightColor", lightColor);
    ...
}
```
就可以得到目标效果啦

![输入图片说明](/imgs/2024-11-22/eR7Dk8ZpNNqxIKQW.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4ODg5MDExNDEsMzUzMjQyMjgyLDE5Nj
UxMjI0NDQsLTE3MjU1MjI1ODUsLTIwODg3NDY2MTJdfQ==
-->