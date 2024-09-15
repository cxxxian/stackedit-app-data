# goal

 1. 将三角形变成立方体
 2. 主要修改createCommandBuffers()
 3. 封装函数
    size_t bufferSize = sizeof(vertices[0]) * vertices.size();
    VkPhysicalDeviceProperties properties;
    vkGetPhysicalDeviceProperties(physicalDevice, &properties);
    size_t nonCoherentAtomSize = properties.limits.nonCoherentAtomSize;
    bufferSize = (bufferSize + nonCoherentAtomSize - 1) / nonCoherentAtomSize * nonCoherentAtomSize;

    VkBuffer stagingBuffer;
    VkDeviceMemory stagingBufferMemory;

    // 创建临时缓冲区
    createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT,
        VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT,
        &stagingBuffer, &stagingBufferMemory);

    // 将顶点数据复制到临时缓冲区
    void* data;
    if (vkMapMemory(logicalDevice, stagingBufferMemory, 0, bufferSize, 0, &data) != VK_SUCCESS) {
        throw std::runtime_error("Failed to map memory for staging buffer!");
    }
    memcpy(data, vertices.data(), sizeof(vertices[0]) * vertices.size()); // 复制数据
    vkUnmapMemory(logicalDevice, stagingBufferMemory); // 解除映射


## temp
1. 把createVertex和createIndex方法做出形参，可以传入vertexBuffer和vertexBuffer2以及indexBuffer。
2. createCommandBuffer最后一段绑定管线和渲染也可以做一个函数传入？
3. createpipeline的函数太丑了，需要优化

---
void setPerspective(float fov, float aspect, float znear, float zfar, VkDevice device, VkBuffer cameraBuffer, VkDeviceMemory cameraBufferMemory) {
    glm::mat4 currentMatrix = matrices.perspective;
    this->fov = fov;
    this->znear = znear;
    this->zfar = zfar;
    matrices.perspective = glm::perspective(glm::radians(fov), aspect, znear, zfar);
    if (flipY) {
        matrices.perspective[1][1] *= -1.0f;
    }
    if (matrices.perspective != currentMatrix) {
        updated = true;
    }

    // 更新 uniform 缓冲区
    updateCameraBuffer(device, cameraBuffer, cameraBufferMemory);
}

void updateViewMatrix(VkDevice device, VkBuffer cameraBuffer, VkDeviceMemory cameraBufferMemory) {
    // 原有的视图矩阵更新代码
    glm::mat4 currentMatrix = matrices.view;

    glm::mat4 rotM = glm::mat4(1.0f);
    glm::mat4 transM;

    rotM = glm::rotate(rotM, glm::radians(rotation.x * (flipY ? -1.0f : 1.0f)), glm::vec3(1.0f, 0.0f, 0.0f));
    rotM = glm::rotate(rotM, glm::radians(rotation.y), glm::vec3(0.0f, 1.0f, 0.0f));
    rotM = glm::rotate(rotM, glm::radians(rotation.z), glm::vec3(0.0f, 0.0f, 1.0f));

    glm::vec3 translation = position;
    if (flipY) {
        translation.y *= -1.0f;
    }
    transM = glm::translate(glm::mat4(1.0f), translation);

    if (type == CameraType::firstperson) {
        matrices.view = rotM * transM;
    } else {
        matrices.view = transM * rotM;
    }

    viewPos = glm::vec4(position, 0.0f) * glm::vec4(-1.0f, 1.0f, -1.0f, 1.0f);

    if (matrices.view != currentMatrix) {
        updated = true;
    }

    // 更新 uniform 缓冲区
    updateCameraBuffer(device, cameraBuffer, cameraBufferMemory);
}

void updateCameraBuffer(VkDevice device, VkBuffer cameraBuffer, VkDeviceMemory cameraBufferMemory) {
    void* data;
    vkMapMemory(device, cameraBufferMemory, 0, sizeof(glm::mat4) * 2, 0, &data);
    memcpy(data, &matrices.view, sizeof(glm::mat4)); // 将视图矩阵拷贝到缓冲区
    memcpy(static_cast<char*>(data) + sizeof(glm::mat4), &matrices.perspective, sizeof(glm::mat4)); // 将投影矩阵拷贝到缓冲区
    vkUnmapMemory(device, cameraBufferMemory);
}



<!--stackedit_data:
eyJoaXN0b3J5IjpbMTczMjEwMTUwNywxNjQ3NzcxODA1LC03NT
k4OTMwNl19
-->