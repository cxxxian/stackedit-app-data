![输入图片说明](/imgs/2024-10-17/yqGCZeQ6aaO6VyUf.png)
![输入图片说明](/imgs/2024-10-17/naV9IKOS1UMOFebi.png)
# Vertex Shader
![输入图片说明](/imgs/2024-10-17/tnBG0s3bZiGZKoZ6.png)
### layout
![输入图片说明](/imgs/2024-10-17/XdjVYCXNYkkT1JIg.png)
### gl_Position
此处我们最终目的就是输出NDC坐标，此处由于我们的输入就已经是NDC坐标了，如果不是的话需要通过一些矩阵变换
![输入图片说明](/imgs/2024-10-17/sZt9j4e6kVbj5bgz.png)
# Fragment Shader
![输入图片说明](/imgs/2024-10-17/ZUnSyAWn1WHGSJ1q.png)
# Shader的编译
![输入图片说明](/imgs/2024-10-17/1ZoqeqyEmXG5gGWK.png)
此处的glShaderSource

 1. const GLchar *const* string本意上是一个*string的数组
 2. count用来记录String数组共有多少个字符串
 3. *length用来记录每个字符串的长度，也是数组
```
void prepareShader() {
    //1 完成vs与fs的源代码，并且装入字符串
    const char* vertexShaderSource =
        "#version 460 core\n"
        "layout (location = 0) in vec3 aPos;\n"
        "void main()\n"
        "{\n"
        "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
        "}\0";
    const char* fragmentShaderSource =
        "#version 330 core\n"
        "out vec4 FragColor;\n"
        "void main()\n"
        "{\n"
        "FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);\n"
        "}\n\0";
        
    //2 创建shader程序
    GLuint vertex, fragment;
    vertex = glCreateShader(GL_VERTEX_SHADER);
    fragment = glCreateShader(GL_FRAGMENT_SHADER);
    
    //3 为shader程序输入shader代码
    //此时无需告诉字符串长度，之间填NULL
    //因为我们用\0结尾，知道结尾在哪
    glShaderSource(vertex, 1, &vertexShaderSource, NULL);
    glShaderSource(fragment, 1, &fragmentShaderSource, NULL);

    int success = 0;
    char infoLog[1024];
    //4 执行shader代码编译
    glCompileShader(vertex);
    //检查vertex编译结果
    glGetShaderiv(vertex, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(vertex, 1024, NULL, infoLog);
        std::cout << "Error: SHADER COMPILE ERROR -- Vertex" << "\n" << infoLog << std::endl;
    }
    glCompileShader(fragment);
    //检查fragment编译结果
    glGetShaderiv(fragment, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(fragment, 1024, NULL, infoLog);
        std::cout << "Error: SHADER COMPILE ERROR -- Fragment" << "\n" << infoLog << std::endl;
    }
    
}
```
# Shader的链接
![输入图片说明](/imgs/2024-10-17/8d8Uxlge27oOiS6M.png)

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNTk2NjU5NjAsLTg2Nzk4MzI0Niw0Mj
YxMTc4ODldfQ==
-->