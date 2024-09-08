# goal

 1. 将三角形变成立方体
 2. 主要修改createCommandBuffers()
 3. 封装函数

### 5. **命令缓冲区是否提交**

检查你的命令缓冲区是否正确提交。确保你在渲染循环中有调用 `vkQueueSubmit` 提交命令缓冲区，并通过信号量和栅栏同步确保渲染完成。

### 6. **索引数据类型**

你在 `vkCmdBindIndexBuffer` 中使用 `VK_INDEX_TYPE_UINT32`，确保你的索引数据确实为 `uint32_t` 类型。如果使用了 `uint16_t` 索引类型，需要将其更改为 `VK_INDEX_TYPE_UINT16`。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc1OTg5MzA2XX0=
-->