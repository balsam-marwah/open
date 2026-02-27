#include <iostream>

// تضمين المكتبات
#define GLEW_STATIC
#include <GL/glew.h>
#include <GLFW/glfw3.h>

// إعدادات النافذة
const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

// --- كود المظللات (Shaders) مع إضافة uniform للون ---

// Vertex Shader: فقط ينقل موقع الرأس (بدون لون)
const char* vertexShaderSource = "#version 330 core\n"
"layout (location = 0) in vec3 aPos;\n"
"void main()\n"
"{\n"
"   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
"}\0";

// Fragment Shader: يستقبل لوناً موحداً (uniform) من البرنامج
const char* fragmentShaderSource = "#version 330 core\n"
"out vec4 FragColor;\n"
"uniform vec4 ourColor;\n"
"void main()\n"
"{\n"
"   FragColor = ourColor;\n"
"}\n\0";

// دالة لتغيير حجم النافذة
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    glViewport(0, 0, width, height);
}

// متغير عام لحالة النافذة (مغلقة/مفتوحة)
bool windowOpen = false;

// دالة معالجة المدخلات
void processInput(GLFWwindow* window)
{
    // إغلاق البرنامج عند الضغط على ESC أو Q
    if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
    if (glfwGetKey(window, GLFW_KEY_Q) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);

    // تبديل حالة النافذة (فتح/غلق) عند الضغط على F
    static bool fPressed = false;  // لمنع التبديل المتكرر أثناء الضغط المستمر
    if (glfwGetKey(window, GLFW_KEY_F) == GLFW_PRESS && !fPressed)
    {
        windowOpen = !windowOpen;
        fPressed = true;
    }
    if (glfwGetKey(window, GLFW_KEY_F) == GLFW_RELEASE)
        fPressed = false;
}

int main()
{
    // --- تهيئة GLFW وإنشاء النافذة ---
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "My Simple House with Grass", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

    // --- تهيئة GLEW ---
    glewExperimental = GL_TRUE;
    if (glewInit() != GLEW_OK)
    {
        std::cout << "Failed to initialize GLEW" << std::endl;
        return -1;
    }

    // --- بناء وتجميع برنامج الشيدر ---
    unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glCompileShader(vertexShader);

    unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    glCompileShader(fragmentShader);

    unsigned int shaderProgram = glCreateProgram();
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);

    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);

    // --- الحصول على موقع uniform (اللون) ---
    int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");

    // --- بيانات الرؤوس (جميع أشكال المنزل + أعشاب) ---
    float vertices[] = {
        // جسم المنزل (مستطيل - 6 رؤوس) [0-5]
        -0.6f, -0.5f, 0.0f,  // 0
         0.6f, -0.5f, 0.0f,  // 1
        -0.6f,  0.3f, 0.0f,  // 2
         0.6f, -0.5f, 0.0f,  // 3
         0.6f,  0.3f, 0.0f,  // 4
        -0.6f,  0.3f, 0.0f,  // 5

        // سقف (مثلث - 3 رؤوس) [6-8]
        -0.7f, 0.3f, 0.0f,   // 6
         0.7f, 0.3f, 0.0f,   // 7
         0.0f, 0.8f, 0.0f,   // 8

         // باب (مستطيل - 6 رؤوس) [9-14]
         -0.25f, -0.5f, 0.0f, // 9
          0.25f, -0.5f, 0.0f, // 10
         -0.25f,  0.0f, 0.0f, // 11
          0.25f, -0.5f, 0.0f, // 12
          0.25f,  0.0f, 0.0f, // 13
         -0.25f,  0.0f, 0.0f, // 14

         // إطار النافذة (مربع - 6 رؤوس) [15-20]
          0.3f, 0.0f, 0.0f,   // 15
          0.5f, 0.0f, 0.0f,   // 16
          0.3f, 0.2f, 0.0f,   // 17
          0.5f, 0.0f, 0.0f,   // 18
          0.5f, 0.2f, 0.0f,   // 19
          0.3f, 0.2f, 0.0f,   // 20

          // خطوط النافذة (عند الفتح - 4 رؤوس لخطين) [21-24]
           0.3f, 0.0f, 0.0f,   // 21
           0.5f, 0.2f, 0.0f,   // 22
           0.3f, 0.2f, 0.0f,   // 23
           0.5f, 0.0f, 0.0f,   // 24

           // --- 5 مثلثات عشب أمام المنزل (3 رؤوس لكل مثلث) [25-39] ---
           // مثلث 1 (يسار)
           -0.8f, -0.5f, 0.0f,
           -0.6f, -0.5f, 0.0f,
           -0.7f, -0.3f, 0.0f,

           // مثلث 2
           -0.4f, -0.5f, 0.0f,
           -0.2f, -0.5f, 0.0f,
           -0.3f, -0.35f, 0.0f,

           // مثلث 3 (وسط)
            0.0f, -0.5f, 0.0f,
            0.2f, -0.5f, 0.0f,
            0.1f, -0.25f, 0.0f,

            // مثلث 4
             0.3f, -0.5f, 0.0f,
             0.5f, -0.5f, 0.0f,
             0.4f, -0.4f, 0.0f,

             // مثلث 5 (يمين)
              0.7f, -0.5f, 0.0f,
              0.9f, -0.5f, 0.0f,
              0.8f, -0.3f, 0.0f
    };

    // --- إعداد VAO و VBO ---
    unsigned int VAO, VBO;
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);

    glBindVertexArray(VAO);

    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    // تحديد كيفية قراءة بيانات الرأس (الموقع فقط)
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);

    // فك الربط (اختياري)
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindVertexArray(0);

    // --- حلقة الرسم ---
    while (!glfwWindowShouldClose(window))
    {
        processInput(window);

        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        glUseProgram(shaderProgram);
        glBindVertexArray(VAO);

        // رسم جسم المنزل (لون بيج)
        glUniform4f(vertexColorLocation, 0.9f, 0.8f, 0.7f, 1.0f);
        glDrawArrays(GL_TRIANGLES, 0, 6);

        // رسم السقف (أحمر)
        glUniform4f(vertexColorLocation, 0.8f, 0.3f, 0.1f, 1.0f);
        glDrawArrays(GL_TRIANGLES, 6, 3);

        // رسم الباب (بني)
        glUniform4f(vertexColorLocation, 0.5f, 0.25f, 0.0f, 1.0f);
        glDrawArrays(GL_TRIANGLES,9,6);

        // رسم النافذة حسب حالتها
        if (windowOpen)
        {
            // نافذة مفتوحة: لون أزرق فاتح (يمثل الزجاج)
            glUniform4f(vertexColorLocation, 0.5f, 0.8f, 1.0f, 1.0f);
            glDrawArrays(GL_TRIANGLES, 15, 6);
            // رسم خطي التقاطع باللون الأبيض
            glUniform4f(vertexColorLocation, 1.0f, 1.0f, 1.0f, 1.0f);
            glDrawArrays(GL_LINES, 21, 4);
        }
        else
        {
            // نافذة مغلقة: لون بني غامق (مثل الخشب)
            glUniform4f(vertexColorLocation, 0.3f, 0.15f, 0.0f, 1.0f);
            glDrawArrays(GL_TRIANGLES, 15, 6);
        }

        // رسم العشب (مثلثات خضراء) - الإزاحة 25، عدد الرؤوس 15
        glUniform4f(vertexColorLocation, 0.0f, 0.8f, 0.0f, 1.0f); // أخضر
        glDrawArrays(GL_TRIANGLES, 25, 15);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // --- تنظيف ---
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProgram);

    glfwTerminate();
    return 0;
}
