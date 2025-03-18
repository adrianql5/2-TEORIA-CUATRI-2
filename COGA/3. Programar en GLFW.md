# 3.1 Crear una ventana
``` C++
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

// Dimensiones de la ventana
const int SCR_WIDTH = 800;
const int SCR_HEIGHT = 600;

// Función para manejar entradas del teclado
void processInput(GLFWwindow* window) {
    if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}

int main() {
    // Inicializa GLFW
    if (!glfwInit()) {
        std::cerr << "Error al inicializar GLFW" << std::endl;
        return -1;
    }

    // Especificamos la versión de OpenGL (3.3 Core Profile)
    //lo de major es el 3 y lo de minor el .3
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    //esto hace que elimine caracteristicas obsoletas
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    // Creamos la ventana, poniendo tamaño, y titulo.
    GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "Ventana con GLFW", NULL, NULL);
    if (!window) {
        std::cerr << "Error al crear la ventana" << std::endl;
        glfwTerminate();
        return -1;
    }

    // Asignamos el contexto de OpenGL a la ventana, hace que todas las llamadas de OpenGL afecten a esa ventana
    glfwMakeContextCurrent(window);

    // Cargamos las funciones de OpenGL con GLAD
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
        std::cerr << "Error al inicializar GLAD" << std::endl;
        return -1;
    }

    // Configuramos el viewport, área de ña vemta donde OpenGLdibujará
    // El (0,0) son las coordenadas de la esquina inferior izquierda
    glViewport(0, 0, SCR_WIDTH, SCR_HEIGHT);

    // Bucle de renderizado, comprobamos si se debería cerrar la ventana
    while (!glfwWindowShouldClose(window)) {
        // Procesamos entradas por teclado
        processInput(window);

        // Limpiamos la pantalla con un color de fondo
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // Intercambiamos buffers y procesamos eventos pendientes como el teclado o del raton
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // Terminamos GLFW
    glfwTerminate();
    return 0;
}
``` 

# 3.2 Shaders
Un **shader** es un programa pequeño que se ejecuta en la **GPU** para procesar gráficos en tiempo real. Su principal objetivo es manipular la apariencia de los objetos en la pantalla, permitiendo efectos visuales como iluminación, sombras, reflejos, desenfoques, entre otros.

Los shaders son esenciales en **OpenGL**, **Vulkan**, **DirectX**, y otros motores gráficos, ya que permiten a los programadores personalizar cómo se renderizan los objetos en la pantalla.

Los shaders se escriben en **GLSL** (OpenGL Shading Language), un lenguaje de programación específico para la GPU.


## 3.2.1 Vertex Shader
Se encarga de procesar los **vértices** de los objetos en la escena.
Convierte las coordenadas del modelo 3D a coordenadas en pantalla.
Maneja atributos de los vértices como:
    - **Posición** en el espacio 3D.
    - **Color** del vértice.
    - **Coordenadas de textura** (para aplicar imágenes a los modelos).
    - **Normales** (para iluminación).

**Flujo de trabajo:**
1. Recibe los vértices de un objeto.
2. Aplica transformaciones para colocarlos en el espacio de la pantalla.
3. Envía los datos procesados al siguiente paso en la renderización.

🔹 **Ejemplo de un Vertex Shader en GLSL:**

```glsl
#version 330 core

layout (location = 0) in vec3 aPos;  // Entrada: posición del vértice
layout (location = 1) in vec3 aColor; // Entrada: color del vértice

out vec3 vertexColor; // Pasamos el color al fragment shader

void main() {
    gl_Position = vec4(aPos, 1.0); // Transformamos el vértice
    vertexColor = aColor; // Pasamos el color al Fragment Shader
}
```

🔹 **Explicación del código:**

- `aPos`: Recibe la posición del vértice desde la CPU.
- `aColor`: Recibe el color del vértice desde la CPU.
- `gl_Position`: Devuelve la posición del vértice transformada.
- `vertexColor`: Se pasa al Fragment Shader para el color final.


## 3.2.2 Fragment Shader
- Se ejecuta después del **Vertex Shader** y trabaja con **fragmentos**.
- Un **fragmento** es un posible píxel en la pantalla (incluye color, profundidad, etc.).
- Calcula el **color final** que se dibujará en cada píxel.
- Es donde se aplican efectos como:
    - **Iluminación**
    - **Texturas**
    - **Sombreado**
    - **Reflejos**

🔹 **Flujo de trabajo:**
1. Recibe información interpolada de los vértices.
2. Determina el color final del píxel en pantalla.
3. Envía el color a la tarjeta gráfica para dibujarlo.

🔹 **Ejemplo de un Fragment Shader en GLSL:**

```glsl
#version 330 core

in vec3 vertexColor; // Color del Vertex Shader

void main() {
    gl_FragColor = vec4(vertexColor, 1.0); // Asigna el color al píxel
}
```

🔹 **Explicación del código:**

- `vertexColor`: Recibe el color interpolado desde el Vertex Shader.
- `FragColor`: Devuelve el color final del píxel a la pantalla.

## 3.2.3 ¿Qué ocurre después del Fragment Shader?
**Alpha Test**
- Verifica la transparencia del píxel.
- Si el píxel es completamente transparente, no se dibuja.

**Blending Test**
- Se usa para mezclar colores (por ejemplo, hacer un objeto semi-transparente).
- Se calcula combinando el color del fragmento con el color del fondo.

**Depth Test**
- OpenGL verifica si el fragmento está **más cerca o más lejos** de la cámara comparado con otros píxeles.
- Si el fragmento está detrás de otro, se descarta.

| **Shader**          | **¿Qué procesa?**    | **Propósito**                                             |
| ------------------- | -------------------- | --------------------------------------------------------- |
| **Vertex Shader**   | Vértices             | Transforma posiciones, colores, y coordenadas de textura. |
| **Fragment Shader** | Fragmentos (píxeles) | Calcula el color final de cada píxel en pantalla.         |

# 3.3 Como se envían los vértices
Aquí no tenemos el `glVertex3f`, pero es ineficiente porque le mandamos de uno en uno los vértices a la GPU. Ahora tenemos que almacenar los vértices en **buffers** en la GPU y se envían de manera más eficiente. Usamos el **VAO** y el **VBO**.

## 3.3.1 VAO (vertex array object)
Es un **contenedor** que guarda la configuración de los datos de los vértices. Es como una carpeta donde OpenGl guarda de los datos de los vértices para usarlos más tarde. Esto permite **organizar y reutilizar datos** sin definirlos varias veces. **Solo creamos un VAO una vez y lo usamos cada vez que queremos dibujar el mismo objeto**.

## 3.3.2 VBO (vertex buffer object)
Es un **buffer** en la GPU que almacena los datos de los vértices. Es como un archivo del VAO que contiene la info de los vertices. En lugar de enviar los vértices uno a uno, los **almacenamos en un VBO**. Esto acelera el renderizado porque la información ya está en la memoria de la gpu.

## 3.3.3 Procedimiento

```C++
float vertices []={
	0.0f,0.0f,0.0f, //V1
	0.5f,0.0f,0.0f, //V2
	0.0f,0.5f,0.0f //V3            
}
```

Definimos vertices en un array de floats

1. **Creamos un VAO**
```C++
unsigned int VAO;
glGenVertexArrays(1, &VAO); 
glBindVertexArray(VAO);
```
- `glGenVertexArrays(1, &VAO); ` crea un **VAO** y le asigna un **ID**
- `glBindVertexArray(VAO);` activa el **VAO**.

2. **Creamos un BVO**
```C++
unsigned int VBO;
glGenBuffers(1, &VBO);
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
``` 
- `glGenBuffers(1, &VBO);` crea un **VBO** y le asigna un **ID**.
- `glBindBuffer(GL_ARRAY_BUFFER, VBO);` activa el **VBO** para que OpenGl lo use.
- `glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);` copia los datos del array de **vértices** a la memoria de la GPU. GL_STATIC_DRAW le dice a OpenGl que los datos no van a cambiar.

3. **Configuramos los atributos de los vértices**
```C++
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
``` 

- `glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);`
	- Define como OpenGL debe **interpretar** los datos del VBO
	- `0` → Número del atributo (posición del vértice). Basicamente le decimos al VAO que la posicion de los vertices se almacena en el atributo 0.
    - `3` → Cada vértice tiene **3 valores (X, Y, Z)**.
    - `GL_FLOAT` → Los valores son de tipo `float`.
    - `GL_FALSE` → No necesitamos normalizar los valores.
    - `3 * sizeof(float)` → Tamaño de cada vértice en la memoria.
    - `(void*)0` → La información empieza desde el inicio del array.
- `glEnableVertexAttribArray(0);`
    - Activa el **atributo de vértices** número `0` (posición).

## 3.3.4 Shaders
Los vertices se envían al pipe.
>[!Nota]
> **Pipeline Gráfico** es el proceso por el cual la tarjeta gráfica (GPU) convierte los datos de los vértices en una imagen final en la pantalla.
>
>🔹 **Piensa en el pipeline gráfico como una fábrica**:
>
>1. **Recibe datos** (vértices de un objeto).
>2. **Los procesa** (con shaders).
>3. **Dibuja los píxeles en la pantalla**.
>
>Cada paso del **pipeline** tiene un **shader**, que es un programa que se ejecuta en la GPU. Las principales etapas son el **vertex shader** y el **fragment shader**.

En primer lugar pasan al **Vertex Shader** donde se calcula la posición de los vértices en la pantalla.
```glsl
#version 330 core //Esto indica que usamos openGl 3.3

layout (location = 0) in vec3 aPos;  // Entrada: posición del vértice

void main() {
    gl_Position = vec4(aPos, 1.0); // Transformamos el vértice
}
```

- `#version 330 core`
	- Indica que estamos usando **OpenGL 3.3**.
	- La sintaxis y las funciones disponibles dependen de la versión de OpenGL.

- `layout (location = 0) in vec3 aPos;`
	- **`layout (location = 0)`** → Indica que la información de los vértices **viene desde el VAO en la posición 0**.
	- **`in vec3 aPos;`** → Declara una variable de **entrada** que recibe un **vector de 3 valores (X, Y, Z)**.

- `gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);`
	- `gl_Position` es una **variable especial** de salida que indica la **posición final del vértice en la pantalla**.
	- **`vec4(aPos.x, aPos.y, aPos.z, 1.0)`** convierte el `vec3` en `vec4` agregando un `1.0` extra (para cálculos de perspectiva).


**Este shader no cambia la posición de los vértices**, simplemente los pasa tal cual. Más adelante, podemos hacer transformaciones (escalado, rotación, traslación) aquí.

Posteriomente pasamos al **fragment shader**, que colorea los píxeles.
```glsl
#version 330 core
out vec4 FragColor; // Salida: color del fragmento

void main() {
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f); // Color RGBA (Rojo-Naranja)
}
```

- `#version 330 core`
	- Igual que en el Vertex Shader, indica que estamos usando **OpenGL 3.3**.

- `out vec4 FragColor;`
	- Declara una **variable de salida** que representa el **color del píxel**.
	- **`vec4`** tiene **4 valores**: `(Red, Green, Blue, Alpha)`.

- `FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);`
	- Asigna un color **naranja** `(R = 1.0, G = 0.5, B = 0.2, A = 1.0)`.
	- **El valor Alpha (`A`) indica transparencia**, pero en este caso es `1.0` (completamente opaco).


# 3.4 Usar Shaders
## 3.4.1 Cargar y Compilar Shaders desde Archivos
El profe nos da la función `setShaders()` se encarga de **leer el código de los shaders desde archivos externos**, lo que permite mantenerlos organizados fuera del código principal.
1. Se **leen los archivos** (`textFileRead(nVertx)` y `textFileRead(nFrag)`) para obtener los códigos de los shaders.
2. Se **crean los identificadores** con `glCreateShader(GL_VERTEX_SHADER)` y `glCreateShader(GL_FRAGMENT_SHADER)`.
3. Se **asigna el código fuente** de los shaders con `glShaderSource()`.
4. Se **compilan los shaders** con `glCompileShader()`.


## 3.4.2 Crear el Programa de Shaders
Una vez compilados, los shaders se deben **unir en un programa** que la GPU pueda ejecutar:
1. Se **crea un programa de shaders** con `glCreateProgram()`.
2. Se **adjuntan** los shaders con `glAttachShader()`.
3. Se **enlaza el programa** con `glLinkProgram()`, combinando los shaders en un único ejecutable.
4. Se verifica si hay errores con `printProgramInfoLog()`.

## 3.4.3 Usar el Programa de Shaders
Una vez creado el programa, se puede **activar con `glUseProgram(progShader);`** para que la GPU lo utilice.
```c
GLuint setShaders(const char *nVertx, const char *nFrag) {
    // Leer archivos de shaders
    char *ficherovs = textFileRead(nVertx);
    char *ficherofs = textFileRead(nFrag);
    const char *codigovs = ficherovs;
    const char *codigofs = ficherofs;

    // Crear y compilar Vertex Shader
    GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &codigovs, NULL);
    glCompileShader(vertexShader);

    // Crear y compilar Fragment Shader
    GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &codigofs, NULL);
    glCompileShader(fragmentShader);

    // Liberar memoria de archivos
    free(ficherovs);
    free(ficherofs);

    // Crear y enlazar el programa de shaders
    GLuint progShader = glCreateProgram();
    glAttachShader(progShader, vertexShader);
    glAttachShader(progShader, fragmentShader);
    glLinkProgram(progShader);
    printProgramInfoLog(progShader); // Verificar errores

    return progShader; // Retorna el ID del programa
}
```

# 3.5 Ejemplo Detallado

##  Definir los Datos de los Vértices
Se declara un array `vertices[]`, donde cada fila representa un **vértice** con:

- **3 valores de posición** `(x, y, z)`.
- **3 valores de color** `(r, g, b)`.

```cpp
float vertices[] = {
    // Posición (x, y, z)   // Color (r, g, b)
    0.0f,  0.0f, 0.0f,      1.0f, 1.0f, 1.0f,  // Vértice 0
    0.5f,  0.0f, 0.0f,      1.0f, 0.0f, 0.0f,  // Vértice 1
    0.0f,  0.5f, 0.0f,      0.0f, 1.0f, 0.0f,  // Vértice 2
    0.0f,  0.5f, 0.0f,      0.0f, 0.0f, 1.0f,  // Vértice 3
    0.5f,  0.5f, 0.5f,      1.0f, 1.0f, 1.0f   // Vértice 4
};
```


## Definir los Índices (Element Buffer Object - EBO)
Se crea un array `indices[]`, que define el orden en que los vértices se deben **conectar para formar primitivas (líneas, triángulos, etc.)**.

```cpp
unsigned int indices[] = {0, 2, 0, 3, 0, 4};
```

Este array indica que:
- El **vértice 0 se conecta con el 2**.
- El **vértice 0 se conecta con el 3**.
- El **vértice 0 se conecta con el 4**.


## Crear los Buffers (VAO, VBO y EBO)
Se **generan identificadores** para cada buffer.

```cpp
unsigned int VAO, VBO, EBO;
glGenVertexArrays(1, &VAO);  // Crea un VAO
glGenBuffers(1, &VBO);       // Crea un VBO
glGenBuffers(1, &EBO);       // Crea un EBO
```

## Configurar el VAO
El **VAO (Vertex Array Object)** es el objeto que agrupa todos los buffers.

```cpp
glBindVertexArray(VAO);
```

Se activa para almacenar la configuración de los buffers.

## Enviar los Datos de los Vértices al VBO
Se enlaza el **Vertex Buffer Object (VBO)** y se copia la información de `vertices[]` a la GPU.

```cpp
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

`GL_STATIC_DRAW`: Indica que los datos **no cambiarán** frecuentemente.

## Configurar los Atributos de los Vértices
Cada vértice tiene **dos atributos**: **posición (x, y, z) y color (r, g, b)**.

**Posición (Atributo 0)**
```cpp
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
```

- `0`: Índice del atributo **(posición)**.
- `3`: Cantidad de valores (**x, y, z**).
- `GL_FLOAT`: Tipo de datos (`float`).
- `6 * sizeof(float)`: Distancia entre atributos (`stride`).
- `(void*)0`: Desplazamiento (**offset = 0** porque la posición es el primer dato).

**Color (Atributo 1)**
```cpp
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3 * sizeof(float)));
glEnableVertexAttribArray(1);
```

- `1`: Índice del atributo **(color)**.
- `3`: Cantidad de valores (**r, g, b**).
- `(void*)(3 * sizeof(float))`: **Desplazamiento de 3 floats** para saltar la posición.

## Configurar el EBO
Se enlaza el **Element Buffer Object (EBO)** y se copian los `indices[]` a la GPU.

```cpp
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
```

>[!Nota]
> En el `glBindBuffer` si es un buffer de indices se especifica `GL_ELEMENT_ARRAY_BUFFER` si es un buffer de vértices se especifica `GL_ARRAY_BUFFER` 
## Liberar Memoria

Una vez configurados los buffers, **se pueden liberar los datos de la CPU**.

```cpp
glDeleteBuffers(1, &VBO);
glDeleteBuffers(1, &EBO);
```

**No se borra el VAO** porque mantiene la configuración.

# 3.6 Usar el VAO

**Tenemos nuestro VAO y nuestros shaders**
- `VAO` almacena la configuración de los vértices.
- `setShaders("shader.vert", "shader.frag")` carga y compila los shaders.

**Dentro del bucle de renderizado (`while`):**
- Se limpia el buffer de color con `glClearColor()` y `glClear()`.
- Se usa el **programa de shaders** con `glUseProgram(shaderProgram)`.
- Se **activa el VAO** con `glBindVertexArray(VAO)`, lo que carga los atributos de los vértices previamente configurados.
- Se llama a `glDrawArrays(GL_LINES, 0, 6)`, que **dibuja líneas** usando los datos almacenados en el VAO.
- Se **desvincula el VAO** con `glBindVertexArray(0)`, aunque no es obligatorio en cada iteración.
- Se actualiza la ventana con `glfwSwapBuffers(window)`, mostrando la imagen renderizada.
- Se procesan eventos de entrada con `glfwPollEvents()`.

## `glDrawArrays(GL_LINES, 0, 6)`
Este comando **dibuja líneas** a partir de los vértices almacenados en el VAO.

📌 **Parámetros:**

- `GL_LINES` → Dibuja líneas conectando pares de vértices.
- `0` → Índice del primer vértice a usar.
- `6` → Número de vértices a dibujar.

Esto significa que OpenGL **dibujará 3 líneas** usando los 6 primeros vértices.

# 3.7 Transformaciones (crazy shit)
Primero tenemos que modificar el **Vertex Shader**:
```glsl
layout (location = 0) in vec3 aPos;
uniform mat4 transform;
void main() {
gl_Position = transform * vec4(aPos, 1.0f);
}
```

Para evitar andar usando el glm::nombrefunc, vamos a tirar de `C++` y le chantamos en la primera línea del programa: 
``` C++
using namespace glm;
```

Y para que el shader reciba la matriz:
```C++
unsigned int transformLoc = glGetUniformLocation(shaderProgram, "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE,glm::value_ptr(transform));
``` 

Tenemos lo mismo que en la 1.2:
```C++
mat4 transform = mat4(1.0f);
transform = translate(transform, vec3(0.25f, 0.0f, 0.0f));
transform = rotate(transform, angulo, vec3(0.0f, 0.0f, 1.0f));
transform = scale(transform, vec3(0.1f, 0.1f, 0.1f));
```

# 3.8 Cámara
En OpenGL 3.3, la **cámara** se implementa mediante **tres matrices clave** que transforman los vértices de los objetos en el mundo 3D hasta la pantalla:
1. **Matriz Modelo (`Mmodel`)**: Ubica los objetos en el mundo.
2. **Matriz Vista (`Mview`)**: Define la posición y orientación de la cámara.
3. **Matriz de Proyección (`Mprojection`)**: Define el tipo de proyección (perspectiva u ortográfica).

El **pipeline de transformación** en OpenGL se ve así:

$$V_{clip} = M_{projection} \times M_{view} \times M_{model} \times V_{local}$$

Donde:
- $V_{local}$ son las coordenadas originales del vértice.
- $M_{model}$ lo posiciona en el mundo.
- $M_{view}$ lo transforma según la cámara.
- $M_{projection}$ ajusta la proyección a la pantalla.

## 3.8.1 Matriz de Proyección (`Mprojection`)
Define **cómo se proyectan los objetos en la pantalla**. Hay dos tipos principales:

### Volumen Ortográfico (`glm::ortho`)
```cpp
glm::mat4 proj = glm::ortho(-100.0f, 100.0f, -100.0f, 100.0f, 0.1f, 100.0f);
```

No hay perspectiva ni profundidad, los objetos mantienen su tamaño.  

```cpp
glm::ortho(left, right, bottom, top, near, far);
```

- `left, right`: Límites izquierdo y derecho del volumen ortográfico.
- `bottom, top`: Límites inferior y superior.
- `near, far`: Distancia del recorte cercano y lejano.


### Volumen en Perspectiva (`glm::perspective`)

```cpp
glm::mat4 proj = glm::perspective(glm::radians(45.0f), aspect, 0.1f, 100.0f);
```


```cpp
glm::perspective(fov, aspect, near, far);
```
- `fov`: **Campo de visión** en radianes (se recomienda `glm::radians(45.0f)`).
- `aspect`: Relación de aspecto `(float) width / (float) height`.
- `near, far`: **Distancia de recorte**, los objetos fuera de este rango no se dibujan.


## 3.8.2 Matriz de Vista (`Mview`)
Define **dónde está la cámara y hacia dónde mira**. Se usa `glm::lookAt()`, que devuelve una matriz de vista.

```cpp
glm::mat4 view = glm::lookAt(
    glm::vec3(0.0f, 0.0f, 3.0f), // Posición de la cámara
    glm::vec3(0.0f, 0.0f, 0.0f), // Hacia dónde mira
    glm::vec3(0.0f, 1.0f, 0.0f)  // Vector "arriba"
);
```

```cpp
glm::lookAt(posCam, objetivo, up);
```

- `posCam`: **Posición de la cámara** en el mundo.
- `objetivo`: **Hacia dónde mira la cámara**.
- `up`: **Vector "arriba"** para definir la inclinación.


```cpp
glm::vec3 posCam(0.0f, 2.0f, 5.0f); // Cámara arriba y atrás
glm::vec3 objetivo(0.0f, 0.0f, 0.0f); // Mira al origen
glm::mat4 view = glm::lookAt(posCam, objetivo, glm::vec3(0.0f, 1.0f, 0.0f));
```


## 3.8.3 Matriz de Modelo (`Mmodel`)
Define **dónde está cada objeto en la escena**. Se combinan transformaciones:

```cpp
glm::mat4 model = glm::mat4(1.0f); // Matriz identidad
model = glm::translate(model, glm::vec3(2.0f, 0.0f, 0.0f)); // Traslación
model = glm::rotate(model, glm::radians(45.0f), glm::vec3(0.0f, 1.0f, 0.0f)); // Rotación
model = glm::scale(model, glm::vec3(1.0f, 2.0f, 1.0f)); // Escala
```


## 3.8.4 Transformaciones en el Shader

El **shader de vértices** usa las 3 matrices (`model`, `view`, `projection`) para transformar los vértices.

```glsl
#version 330 core
layout (location = 0) in vec3 aPos; // Posición del vértice
uniform mat4 model; 
uniform mat4 view;
uniform mat4 projection;

void main() {
    gl_Position = projection * view * model * vec4(aPos, 1.0); // Transformaciones
}
```

📌 **Explicación**  
Primero, `model` coloca el objeto en la escena.  
Luego, `view` transforma todo según la cámara.  
Finalmente, `projection` aplica la proyección.

## 3.8.5 Cómo Enviar las Matrices a OpenGL
Antes de dibujar, **enviamos las matrices al shader** usando `glUniformMatrix4fv()`.

```cpp
unsigned int modelLoc = glGetUniformLocation(shaderProgram, "model");
glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));

unsigned int viewLoc = glGetUniformLocation(shaderProgram, "view");
glUniformMatrix4fv(viewLoc, 1, GL_FALSE, glm::value_ptr(view));

unsigned int projectionLoc = glGetUniformLocation(shaderProgram, "projection");
glUniformMatrix4fv(projectionLoc, 1, GL_FALSE, glm::value_ptr(proj));
```

- `glGetUniformLocation(shaderProgram, "nombre_uniform")`: Busca la variable en el shader.
- `glUniformMatrix4fv(loc, 1, GL_FALSE, glm::value_ptr(matrix))`: Envía la matriz a la GPU.
