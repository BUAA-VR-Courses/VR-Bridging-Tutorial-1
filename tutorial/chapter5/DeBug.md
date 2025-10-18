# Debug Hint

## 检查shadow map

### 一、调试核心思路

阴影贴图本质上是一张存储深度信息的纹理，其可视化原理与普通纹理（如牛模型的漫反射贴图、平板的棋盘格贴图）完全一致。既然普通纹理能正常贴在模型上显示，我们也可以将阴影贴图“贴”在一个屏幕空间的矩形上，通过观察灰度分布来验证阴影贴图是否正确生成——这是定位阴影渲染问题的高效手段。

核心实现逻辑：在屏幕空间绘制一个全屏四边形，通过片段着色器对阴影贴图进行采样，将深度值转换为灰度颜色输出，直观判断阴影贴图的生成效果。

>感兴趣的同学可以尝试自行实现，当然也可以直接使用以下代码

### 二、着色器代码配置

#### 1. 顶点着色器（texture.vert）

负责传递顶点位置和纹理坐标，直接使用标准化设备坐标（NDC）确保四边形覆盖整个屏幕。

```glsl
#version 330 core  

// 输入顶点属性：位置（3分量）和纹理坐标（2分量）
layout (location = 0) in vec3 aPos;       // 顶点位置（与VAO属性索引0绑定）
layout (location = 1) in vec2 aTexCoord;  // 纹理坐标（与VAO属性索引1绑定）

// 输出到片段着色器的纹理坐标（需与片段着色器输入变量完全匹配）
out vec2 TexCoord;

void main()
{
    // 直接使用输入位置作为NDC坐标，确保四边形全屏显示
    gl_Position = vec4(aPos, 1.0);
    // 将纹理坐标传递给片段着色器，用于后续纹理采样
    TexCoord = aTexCoord;
}
```

#### 2. 片段着色器（texture.frag）

对阴影贴图（深度纹理）进行采样，将深度值映射为灰度色输出，便于观察深度分布。

```glsl
#version 330 core
// 从顶点着色器接收的纹理坐标（名称、类型需与顶点着色器输出一致）
in vec2 TexCoord;
// 输出最终像素颜色
out vec4 FragColor;

// 2D纹理采样器（绑定到指定纹理单元，用于采样阴影贴图）
uniform sampler2D ourTexture;

void main() {
    // 采样阴影贴图的深度值（深度信息存储在r通道，范围[0,1]）
    float depth = texture(ourTexture, TexCoord).r;
    // 将深度值转换为灰度颜色输出（默认近物暗、远物亮）
    FragColor = vec4(vec3(depth), 1.0f);
    
    // 调试备用：输出纯色（用于验证着色器是否正常执行）
    // FragColor = vec4(1.0f, 0.0f, 0.0f, 1.0f);
}
```

### 三、调试数据准备

#### 1. 着色器程序初始化

创建调试专用着色器程序，加载上述顶点/片段着色器文件。

```cpp
// 初始化调试着色器（传入顶点/片段着色器文件路径）
Shader debug_shader(SHADER_DIR"/texture.vert", SHADER_DIR"/texture.frag");
```

#### 2. 全屏四边形数据定义

通过顶点数组和索引数组定义一个覆盖全屏的四边形（由两个三角形组成），包含位置和纹理坐标信息。

```cpp
// 顶点数据：位置（x,y,z）+ 纹理坐标（u,v）
float vertices[] = {
    // 位置          // 纹理坐标
    1.0f,  1.0f, 0.0f, 1.0f, 1.0f,  // 右上角
    1.0f, -1.0f, 0.0f, 1.0f, 0.0f,  // 右下角
    -1.0f, -1.0f, 0.0f, 0.0f, 0.0f,  // 左下角
    -1.0f,  1.0f, 0.0f, 0.0f, 1.0f   // 左上角
};

// 索引数据：通过索引复用顶点，组成两个三角形
unsigned int indices[] = {
    0, 1, 3,  // 第一个三角形（右上角、右下角、左上角）
    1, 2, 3   // 第二个三角形（右下角、左下角、左上角）
};
```

#### 3. VAO/VBO/EBO配置

创建并配置顶点数组对象（VAO）、顶点缓冲对象（VBO）和索引缓冲对象（EBO），确保顶点数据能正确传递给着色器。

```cpp
unsigned int VAO, VBO, EBO;
// 生成缓冲对象ID
glGenVertexArrays(1, &VAO);
glGenBuffers(1, &VBO);
glGenBuffers(1, &EBO);

// 绑定VAO（后续顶点属性配置均关联到此VAO）
glBindVertexArray(VAO);

// 绑定VBO并上传顶点数据
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

// 绑定EBO并上传索引数据
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

// 配置位置属性（location=0）
glVertexAttribPointer(
    0,                  // 属性索引（与顶点着色器layout(location=0)对应）
    3,                  // 每个属性的分量数（vec3）
    GL_FLOAT,           // 数据类型
    GL_FALSE,           // 是否归一化
    5 * sizeof(float),  // 顶点步长（位置3个float + 纹理坐标2个float）
    (void*)0            // 属性在缓冲中的偏移量
);
glEnableVertexAttribArray(0);  // 启用位置属性

// 配置纹理坐标属性（location=1）
glVertexAttribPointer(
    1,                  // 属性索引（与顶点着色器layout(location=1)对应）
    2,                  // 每个属性的分量数（vec2）
    GL_FLOAT,           // 数据类型
    GL_FALSE,           // 是否归一化
    5 * sizeof(float),  // 顶点步长
    (void*)(3 * sizeof(float))  // 偏移量（跳过前3个float的位置数据）
);
glEnableVertexAttribArray(1);  // 启用纹理坐标属性

// 解绑VAO（避免后续操作意外修改配置）
glBindVertexArray(0);
```

### 四、调试渲染循环配置

在主渲染循环中，注释原有光照渲染逻辑，专注于阴影贴图的可视化调试，核心步骤包括帧缓冲切换、状态配置、纹理绑定和绘制调用。

```cpp
// 注释原有光照渲染代码，仅保留调试逻辑
// ...

// 1. 切换到默认帧缓冲（屏幕），确保调试内容显示在窗口上
glBindFramebuffer(GL_FRAMEBUFFER, 0);
// 设置视口为窗口实际尺寸（避免调试内容超出显示范围）
glViewport(0, 0, SCR_WIDTH + GUI_WIDTH, SCR_HEIGHT);

// 2. 临时禁用干扰性渲染状态（避免深度测试、面剔除等导致绘制失败）
glDisable(GL_DEPTH_TEST);
glDisable(GL_CULL_FACE);
glDisable(GL_BLEND);

// 3. 清除颜色缓冲，设置蓝色背景（与灰度阴影贴图形成明显对比，便于观察）
glClearColor(0.0f, 0.0f, 1.0f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT);

// 4. 激活调试着色器程序（关键步骤，确保使用目标着色器进行渲染）
debug_shader.use_program();

// 5. 绑定阴影贴图到纹理单元0
glActiveTexture(GL_TEXTURE0);  // 激活纹理单元0
shadow_map.bind();             // 绑定阴影贴图（确保内部调用glBindTexture(GL_TEXTURE_2D, 纹理ID)）

//第4,5步等价于debug_shader.activate_texture(0,&shadow_map)

// 6. 配置采样器Uniform变量（检查变量位置有效性，避免采样失败）
GLint texLoc = glGetUniformLocation(debug_shader.get_id(), "ourTexture");
if (texLoc == -1) {
    std::cerr << "[E] 片段着色器中未找到uniform变量ourTexture！请检查变量名拼写" << std::endl;
} else {
    glUniform1i(texLoc, 0);  // 告诉采样器使用纹理单元0
}

// 7. 绑定VAO并执行绘制调用
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);  // 绘制6个索引对应的两个三角形

// 8. 恢复原有渲染状态（避免影响后续其他渲染逻辑）
glEnable(GL_DEPTH_TEST);
glEnable(GL_CULL_FACE);
glEnable(GL_BLEND);

// 9. 交换缓冲并处理窗口事件（确保调试内容实时更新）
glfwSwapBuffers(window);
glfwPollEvents();
```

