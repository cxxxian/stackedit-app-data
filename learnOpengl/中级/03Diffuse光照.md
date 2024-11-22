在main.cpp中声明光照的参数，然后将这些参数搞到shader中
```
//平行光：参数（方向、光强）uniform变量形式
glm::vec3 lightDirection = glm::vec3(-1.0f, -1.0f, -1.0f);
glm::vec3 lightColor = glm::vec3(0.9f, 0.85f, 0.75f);
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMzUzMjQyMjgyLDE5NjUxMjI0NDQsLTE3Mj
U1MjI1ODUsLTIwODg3NDY2MTJdfQ==
-->