# OpenGL 简明教程

## 1. 什么是OpenGL

OpenGL（Open Graphics Library，开放图形库）是一套跨平台、跨语言的图形编程接口（API），主要用于高效渲染 2D 和 3D 图形。它由 Khronos Group（一个开源技术联盟）维护，是图形编程领域的行业标准之一。

## 2. 基本渲染流程

1. **初始化**
   - 创建窗口
   - 初始化GLAD
   - 设置视口（Viewport）

2. **准备数据**
   - 准备绘制对象（FrameBuffer，这一点在本教程中不做过多体现，可以简单的理解为OpenGL为程序员提供不止屏幕这一张“画布”）
   - 定义顶点数据（位置、颜色等）
   - 创建VBO（顶点缓冲对象）存储数据
   - 创建VAO（顶点数组对象）管理顶点格式

3. **创建着色器**
   - 顶点着色器：处理顶点位置
   - 片段着色器：处理像素颜色
   - 编译并链接为着色器程序

4. **渲染循环**
   - 清空缓冲区
   - 激活着色器程序
   - 绑定VAO并绘制
   - 交换缓冲、处理事件

## 3. 示例：绘制三角形

### 顶点着色器（vertex_shader.glsl）
```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

void main() {
    gl_Position = vec4(aPos, 1.0);
}
```

### 片段着色器（fragment_shader.glsl）
```glsl
#version 330 core
out vec4 FragColor;

void main() {
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
}
```

### 主程序（main.cpp）
```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

// 窗口尺寸
const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

// 顶点数据
float vertices[] = {
    -0.5f, -0.5f, 0.0f, // 左下
     0.5f, -0.5f, 0.0f, // 右下
     0.0f,  0.5f, 0.0f  // 顶部
};

// 编译着色器
unsigned int compileShader(unsigned int type, const char* source) {
    unsigned int id = glCreateShader(type);
    glShaderSource(id, 1, &source, nullptr);
    glCompileShader(id);
    return id;
}

int main() {
    // 初始化GLFW
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    // 创建窗口
    GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "OpenGL Demo", NULL, NULL);
    glfwMakeContextCurrent(window);

    // 初始化GLAD
    gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);
    glViewport(0, 0, SCR_WIDTH, SCR_HEIGHT);

    // 编译着色器，并非唯一方法
    const char* vertexShaderSource = "#version 330 core\nlayout (location = 0) in vec3 aPos;\nvoid main() { gl_Position = vec4(aPos, 1.0); }";
    const char* fragmentShaderSource = "#version 330 core\nout vec4 FragColor;\nvoid main() { FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f); }";
    
    unsigned int vertexShader = compileShader(GL_VERTEX_SHADER, vertexShaderSource);
    unsigned int fragmentShader = compileShader(GL_FRAGMENT_SHADER, fragmentShaderSource);
    
    // 链接程序
    unsigned int shaderProgram = glCreateProgram();
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);

    // 设置VAO和VBO
    unsigned int VAO, VBO;
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
    
    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);

    // 渲染循环
    while (!glfwWindowShouldClose(window)) {
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // 绘制三角形
        glUseProgram(shaderProgram);
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, 3);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // 清理资源
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProgram);

    glfwTerminate();
    return 0;
}
```

## 4. 学习资源

- [LearnOpenGL](https://learnopengl.com/)：详细的入门教程
- [OpenGL官方文档](https://www.khronos.org/opengl/)：完整API参考
- [OpenGL SuperBible](https://www.openglsuperbible.com/)：经典书籍