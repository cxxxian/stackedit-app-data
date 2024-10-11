# Hello World
```
#ifdef GL_ES
precision mediump float;
#endif

uniform float u_time;

void main() {
	gl_FragColor = vec4(1.0,0.0,0.0,1.0);
}
```
1.  最终的像素颜色取决于预设的全局变量 `gl_FragColor`。
2. 如果我们仔细观察 `vec4` 类型，可以推测这四个变元分别响应红，绿，蓝和透明度通道。同时我们也可以看到这些变量是**规范化**的，意思是它们的值是从0到1的。之后我们会学习如何规范化变量，使得在变量间**map**（映射）数值更加容易。
3. 另一个可以从本例看出来的很重要的类 C 语言特征是，预处理程序的宏指令。宏指令是预编译的一部分。有了宏才可以 `#define` （定义）全局变量和进行一些基础的条件运算（通过使用 `#ifdef` 和 `#endif`）。所有的宏都以 `#` 开头。预编译会在编译前一刻发生，把所有的命令复制到 `#defines` 里，检查`#ifdef` 条件句是否已被定义， `#ifndef` 条件句是否没有被定义。在我们刚刚的“hello world!”的例子中，如果定义了`GL_ES`这个变量，才会插入运行第2行的代码，这个通常用在移动端或浏览器的编译中。
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDM3MDA4MjY2XX0=
-->