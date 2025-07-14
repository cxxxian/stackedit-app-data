
# GPU为什么不适合用if或者switch
这个问题关键在于 GPU 的执行模型，叫做 **SIMD**（Single Instruction, Multiple Data）或 **SIMT**（NVIDIA 的变种：Single Instruction, Multiple Threads）

### GPU 是“锁步执行”的 —— 一组线程（Warp/Wavefront）执行同一条指令

> 在 GPU 上，32 或 64 个线程是打包在一起执行的（一个 Warp/Wave），他们 **必须执行同一条指令**。

#### 如果出现 `if` 分支会发生什么？

```cpp
if (a > 0) {     
	color = red; 
} else {     
	color = blue; 
}
```

![输入图片说明](/imgs/2025-07-14/Tj3ZjRISqyVcSAzq.png)

### 正常我们以为会怎样执行？

我们希望每个线程直接走它自己的分支，互不影响，对吧？

但 GPU **不行**，因为这 4 个线程是在同一个 Warp 中，它们**必须执行一模一样的指令流**！

----------

### 实际 GPU 会怎么做？

1.  **先执行 if 分支：**
    
    -   所有线程**都执行** `color = red;`
        
    -   但 GPU 会“屏蔽”不该执行的线程（1 和 3）→ 它们执行这条指令但不写结果
        
2.  **再执行 else 分支：**
    
    -   所有线程**再执行一次** `color = blue;`
        
    -   这次屏蔽掉线程 0 和 2，只让 1 和 3 写入结果

可以改写为：
`mix` 线性插值写法（推荐）

```glsl
vec3 red  = vec3(1.0, 0.0, 0.0);
vec3 blue = vec3(0.0, 0.0, 1.0);

float t   = step(0.5, a);          // a ≤ 0.5 → 0，a > 0.5 → 1
vec3 color = mix(blue, red, t);    // 等价于 if/else
```
-   `step(edge, x)` 在硬件里是无分支的比较；
-   `mix(x, y, t)` = `x * (1‑t) + y * t`，同样无分支。


# 如何降低DrawCall
降低Draw Call主要是为了减轻CPU向GPU提交绘制命令的压力，提高渲染效率。常用方法包括：

-   **GPU Instancing（实例化）**：一次Draw Call绘制多个相同模型的不同实例，减少重复调用。
-   **合批（Batching）**：把多个小物体的顶点合并成一个大顶点缓冲，一次提交，减少调用次数。（
 **静态合批（Static Batching）**：针对静态不动的物体，预先把它们的顶点和索引合并，渲染时只需一次提交。
 **动态合批（Dynamic Batching）**：对频繁移动的物体，实时在CPU端把它们的顶点合并成一个缓冲，但合批开销较大。）
-   **使用纹理图集（Texture Atlas）**：将多个纹理合并成一张大纹理，避免频繁切换材质。
-   **减少状态切换**：按材质、渲染状态排序物体，减少GPU切换开销。
-   **合并绘制调用**：合并多种绘制命令，尽量用一个Draw Call完成。
-   **裁剪和视锥剔除**：不渲染不可见物体，避免无用Draw Call。
-   **使用延迟渲染**：减少光照计算重复，间接降低Draw Call负担。
<!--stackedit_data:
eyJoaXN0b3J5IjpbODYxNTM2NjUwLC0yNzU4NzgyMjRdfQ==
-->