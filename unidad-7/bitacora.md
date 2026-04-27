# Unidad 7


## Bitácora de aplicación 



### Fase 1



```

#include <iostream>
#include <glad/glad.h>
#include <GLFW/glfw3.h>


void framebuffer_size_callback(GLFWwindow* window, int width, int height) {
    glViewport(0, 0, width, height);
}


void processInput(GLFWwindow* window) {
    if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}


const unsigned int SCR_WIDTH = 400;
const unsigned int SCR_HEIGHT = 400;


const char* vertexShaderSrc = R"glsl(
    #version 460 core
    layout(location = 0) in vec3 aPos;
    uniform vec2 offset;
    void main() {
        vec3 newPos = aPos;
        newPos.x += offset.x;
        newPos.y += offset.y;
        gl_Position = vec4(newPos, 1.0);
    }
)glsl";

const char* fragmentShaderSrc = R"glsl(
    #version 460 core
    out vec4 FragColor;
    uniform vec4 ourColor;
    void main() {
        FragColor = ourColor;
    }
)glsl";


unsigned int VAO, VBO;
unsigned int shaderProg;


unsigned int buildShaderProgram() {
    int success;
    char log[512];

    unsigned int vs = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vs, 1, &vertexShaderSrc, nullptr);
    glCompileShader(vs);
    glGetShaderiv(vs, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(vs, 512, nullptr, log);
        std::cerr << "ERROR VERTEX SHADER:\n" << log << "\n";
    }

    unsigned int fs = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fs, 1, &fragmentShaderSrc, nullptr);
    glCompileShader(fs);
    glGetShaderiv(fs, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(fs, 512, nullptr, log);
        std::cerr << "ERROR FRAGMENT SHADER:\n" << log << "\n";
    }

    unsigned int prog = glCreateProgram();
    glAttachShader(prog, vs);
    glAttachShader(prog, fs);
    glLinkProgram(prog);
    glGetProgramiv(prog, GL_LINK_STATUS, &success);
    if (!success) {
        glGetProgramInfoLog(prog, 512, nullptr, log);
        std::cerr << "ERROR LINKING PROGRAM:\n" << log << "\n";
    }

    glDeleteShader(vs);
    glDeleteShader(fs);
    return prog;
}


void setupTriangle() {
    float vertices[] = {
        -0.5f, -0.5f, 0.0f,
         0.5f, -0.5f, 0.0f,
         0.0f,  0.5f, 0.0f
    };
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);

    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);
    glBindVertexArray(0);
}

int main() {
    
    if (!glfwInit()) {
        std::cerr << "Fallo al inicializar GLFW\n";
        return -1;
    }
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 6);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    
    GLFWwindow* mainWindow = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "Ventana Actividad 06", nullptr, nullptr);
    if (!mainWindow) {
        std::cerr << "Error creando ventana\n";
        glfwTerminate();
        return -1;
    }

   
    int bufferWidth, bufferHeight;
    glfwGetFramebufferSize(mainWindow, &bufferWidth, &bufferHeight);

  
    glfwSetFramebufferSizeCallback(mainWindow, framebuffer_size_callback);

    
    glfwMakeContextCurrent(mainWindow);
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
        std::cerr << "Fallo al cargar GLAD\n";
        return -1;
    }

    
    glfwSwapInterval(1);

   
    shaderProg = buildShaderProgram();

    
    setupTriangle();

    
    glViewport(0, 0, bufferWidth, bufferHeight);

   
    glUseProgram(shaderProg);
    int offsetLocation = glGetUniformLocation(shaderProg, "offset");
    int colorLocation = glGetUniformLocation(shaderProg, "ourColor");

    
    while (!glfwWindowShouldClose(mainWindow)) {
        
        glfwPollEvents();

        
        processInput(mainWindow);

        
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

       
        glUseProgram(shaderProg);

        
        double xpos, ypos;
        glfwGetCursorPos(mainWindow, &xpos, &ypos);

        
        float x = (float)xpos / (float)SCR_WIDTH;
        x = (x < 0) ? 0 : ((x > 1) ? 1 : x);

        float y = (float)ypos / (float)SCR_HEIGHT;
        y = (y < 0) ? 0 : ((y > 1) ? 1 : y);

        
        glUniform4f(colorLocation, x, y, 0.0f, 1.0f);

        
        glUniform2f(offsetLocation, x * 2 - 1.0f, 1.0f - y * 2.0f);
        

        
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, 3);

       
        glfwSwapBuffers(mainWindow);
    }

    
    glfwMakeContextCurrent(mainWindow);
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProg);

    glfwDestroyWindow(mainWindow);
    glfwTerminate();
    return 0;
}




```




### Fase 2


#### Evidencia 1 — Contexto y carga de OpenGL

Demuestra que comprendes la secuencia de inicialización. Explica por qué GLFW debe aparecer antes de GLAD y qué función cumple cada uno.


<img width="1919" height="1012" alt="image" src="https://github.com/user-attachments/assets/68559b8a-239b-4fe1-acb7-916618a506b1" />


##### Punto de inspeccion 





##### Explicacion 






##### Justificacion 




#### Evidencia 2 — Del arreglo al shader

Demuestra con el depurador o con una evidencia técnica equivalente cómo el arreglo de vértices termina alimentando el atributo del vertex shader.



<img width="1918" height="1017" alt="image" src="https://github.com/user-attachments/assets/15c0c759-1827-44b7-8336-456e4333ec84" />



##### Punto de inspeccion 






##### Explicacion 








##### Justificacion 





#### Evidencia 3 — Uniform y cambio visual

Demuestra que un uniform modifica el resultado visual sin cambiar el VBO. Explica por qué esto es posible.
   




<img width="1915" height="1012" alt="image" src="https://github.com/user-attachments/assets/249dac9f-4989-4547-82ab-3a55962362db" />







##### Punto de inspeccion 










##### Explicacion 






##### Justificacion 





#### Evidencia 4 — Prueba de borde

Diseña una prueba deliberada. Por ejemplo:

usar un offset fuera del rango esperado,
enviar un color extremo,
desactivar un atributo,
cambiar un location y observar qué se rompe.





<img width="1919" height="1017" alt="image" src="https://github.com/user-attachments/assets/3f86c960-ed06-428c-bbd5-275372381dec" />


##### Punto de inspeccion 







##### Explicacion 





##### Justificacion 




#### Evidencia 5 — Responsabilidad del pipeline

Elige una decisión técnica que tomaste y justifícala. Por ejemplo:

por qué un dato va como uniform y no como atributo,
por qué cierto cálculo se hace en C++ y no en GLSL,
por qué el problema visual observado se debe a un error de estado y no a un error del arreglo de vértices.




<img width="1919" height="1014" alt="image" src="https://github.com/user-attachments/assets/c787a9a2-0161-4b29-b0eb-686f80ac4459" />






##### Punto de inspeccion 






##### Explicacion 






##### Justificacion 








## Bitácora de reflexión
