# Uniform
现在我们知道了 GPU 如何处理并行线程，每个线程负责给完整图像的一部分配置颜色。尽管每个线程和其他线程之间不能有数据交换，但我们能从 CPU 给每个线程输入数据。因为显卡的架构，所有线程的输入值必须**统一**（uniform），而且必须设为**只读**。也就是说，每条线程接收相同的数据，并且是不可改变的数据。

这些输入值叫做 `uniform` （统一值），它们的数据类型通常为：`float`, `vec2`, `vec3`, `vec4`, `mat2`, `mat3`, `mat4`, `sampler2D` and `samplerCube`。uniform 值需要数值类型前后一致。且在 shader 的开头，在设定精度之后，就对其进行定义。

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution; // 画布尺寸（宽，高）
uniform vec2 u_mouse;      // 鼠标位置（在屏幕上哪个像素）
uniform float u_time;     // 时间（加载后的秒数）
```

你可以把 uniforms 想象成连通 GPU 和 CPU 的许多小的桥梁。

## gl_FragCoord

就像 GLSL 有个默认输出值 `vec4 gl_FragColor` 一样，它也有一个默认输入值（ `vec4 gl_FragCoord` ）。`gl_FragCoord`存储了活动线程正在处理的**像素**或**屏幕碎片**的坐标。有了它我们就知道了屏幕上的哪一个线程正在运转。为什么我们不叫 `gl_FragCoord` uniform （统一值）呢？因为每个像素的坐标都不同，所以我们把它叫做 **varying**（变化值）。
```
#ifdef GL_ES
precision mediump float;
#endif
uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;
void main() {
    vec2 st = gl_FragCoord.xy/u_resolution;
	gl_FragColor = vec4(st.x,st.y,0.0,1.0);
}
```
![输入图片说明](/imgs/2024-10-11/L4bspxPAdJkOpAQb.png)
---
- 在代码中，渐变是通过将 `st.x` 和 `st.y` 分别用作红色（`r`）和绿色（`g`）分量来实现的：
        
        `gl_FragColor = vec4(st.x, st.y, 0.0, 1.0);`
        
    -   这意味着：
        -   当片元靠近屏幕的左边（`x` 较小）时，红色分量 `st.x` 接近于 0，因此红色较弱。
        -   当片元靠近屏幕的右边（`x` 较大）时，红色分量 `st.x` 接近于 1，因此红色较强。
        -   同样，当片元靠近顶部（`y` 较大）时，绿色分量 `st.y` 接近于 1，导致绿色较强，而靠近底部时，绿色分量接近于 0，导致绿色较弱。
-   **渐变效果的示意**:
    
    -   当 `x` 从 0 变化到 1 时，颜色从左到右逐渐变成红色。
    -   当 `y` 从 0 变化到 1 时，颜色从底部到顶部逐渐变成绿色。
    -   因此，结合 `x` 和 `y` 的变化，形成了一个从左下角（黑色）到右上角（亮绿色）的渐变效果。
---
上述代码中我们用 `gl_FragCoord.xy` 除以 `u_resolution`，对坐标进行了**规范化**。这样做是为了使所有的值落在 `0.0` 到 `1.0` 之间，这样就可以轻松把 X 或 Y 的值映射到红色或者绿色通道。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQ0NTE2MjM5NiwxMjQwODI4ODEyLDMxND
A3MDY5NiwtMjA4ODc0NjYxMl19
-->