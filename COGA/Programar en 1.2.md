# 2.1 Crear una Ventana
```c++
#include <windows.h>
#include <glut.h>
#include <gl.h>
#include <glu.h>

const int W_WIDTH = 500;	// Ancho de la ventana
const int W_HEIGHT = 500;	// Alto de la ventana

void myDibujo() {
	glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT); //borramos ambos buffers

	glBegin(GL_POINTS); //Indicamos que vamos a dibujar puntos
	glColor3f(0.0, 1.0, 0.0); //Indicamos el color del punto
	glVertex3d(0.5, 0.5,0.5); 

	glFlush(); //limpia la pantalla (forzamos la impresion del dibujo por pantalla mejor dicho)
	glSwapBUffers();//al usar double buffer debemos forzar el intercambio
}


void iniciar() {
	glClearColor(1, 0, 0, 1); //le decimos que limpie la pantalla de rojo
	glClearDepth(1.0); //especificamos el valor de limpieza del z-buffer
}

int main(int argc, char** argv) {

	glutInit(&argc, argv);//con esto iniciamos glut

	glutInitDisplayMode(GLUT_RGBA | GLUT_DOUBLE);//Indicamos que vamos a usar RGBA y un buffer doble

	glutInitWindowSize(W_WIDTH, W_HEIGHT); //indicamos las dimensiones de nuestra ventana
	glutInitWindowPosition(100, 100); //indicamos la posicion de la ventana
	glutCreateWindow("FUCK USC"); //creamos nuestra ventana y le ponemmos un titulo descriptivo

	iniciar();

	glutDisplayFunc(myDibujo); //le indicamos que funcion debe usar para dibujar en pantalla

	glutMainLoop(); //esto es un bucle infinito que crea glut para que se muestre todo el rato la imagen en pantalla

	return 0;
}
```

**Comentarios:**
- Usamos un buffer doble-> implica que se va dibujando en un buffer la imagen y después se intercambia con el otro buffer.
![Double Buffering and Page Flipping (The Java™ Tutorials > Bonus >  Full-Screen Exclusive Mode API)](https://docs.oracle.com/javase/tutorial/figures/extra/fullscreen/doubleBuffering.gif)
- RGBA-> red green blue y a es para la intensidad
- Usamos el z-buffer-> que almacena información sobre que objetos están más alejados de la cámara, le damos como valor por defecto 1.0 al borrado de este buffer, si ponemos 0.0 no se verá casi ningún objeto.
![[Pasted image 20250228223346.png]]

# 2.2 Dibujar Objetos
**Cosas muy importantes:**
- Los ejes no son como los de toda la vida
![[Pasted image 20250228223916.png]]
- Regla de la mano derecha muy importante porque solo se va a ver la cara de la figura a donde se dirija el vector normal, por lo que el orden de colocación de los puntos importa: 
![[Pasted image 20250228224504.png]]
- La imagen derecha se ve de frente y la izquierda de espaldas.
![[Pasted image 20250228224522.png]]

``` c++
#include <windows.h>
#include <glut.h>
#include <gl.h>
#include <glu.h>



void myDibujo() {
	glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT);

	glBegin(GL_LINES); //Indicamos que vamos a dibujar puntos
	glColor3f(1.0f, 0.0f, 0.0f); //Indicamos el color del punto
	glVertex3f(0.0f, 0.0f,0.0f);
	glVertex3f(0.0f, 5.0f,0.0f);
	glEnd(); //Le decimos que hemos acabado

	//No es necesario poner todo el rato glEnd y GLbegin si vais a usar un mismo tipo de figura, pero es util para entender vuestro c�digo
	
	glFlush();
	glutSwapBuffers(); 
}

```

**Argumentos del `glBegin()`:**
![[Pasted image 20250228224206.png]]

**Funciones útiles:**
- `glEnable(GL_CULL_FACE)`: El culling se refiere a la eliminación de las caras de los objetos que no son visibles desde la cámara, con el objetivo de mejorar el rendimiento. (Se desactiva usando `glDisable(GL_CULL_FACE)`)
- `glEnable(GL_NORMALIZE)`: es una función que asegura que los **vectores normales** se mantengan normalizados después de una transformación. Normalizar un vector significa asegurarse de que su longitud sea de **1 unidad**
- `glPolygonMode(GLenum cara, GLenum modo)`: 
	- `glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)`: muestra solo las aristas
	- `glPolygonMode(GL_FRONT_AND_BACK, GL_FILL)`: colorea todo
- `glEnable(GL_DEPTH_TEST)`: **activa la prueba de profundidad** (Depth Test) en OpenGL. Esto significa que OpenGL comenzará a comparar la profundidad (Z) de cada fragmento que intenta dibujar con el valor almacenado en el **Depth Buffer** (Z-Buffer).

# 2.3 Listas
Una **Display List** en OpenGL es una secuencia de comandos de renderizado que se almacena en la GPU para su reutilización. Solo las usamos con glut, no con glfw3.

```c++
#include <GL/glut.h>

GLuint miLista; //declaro la lista

void crearDisplayList() {
    miLista = glGenLists(1);  // Generar un id 
    glNewList(miLista, GL_COMPILE);  // Inicia la lista en modo COMPILE -> se guarda pero no se ejecuta
        glColor3f(1.0, 0.0, 0.0);  // Color rojo
        glBegin(GL_TRIANGLES);  // Dibujar un triángulo
            glVertex2f(-0.5, -0.5);
            glVertex2f(0.5, -0.5);
            glVertex2f(0.0, 0.5);
        glEnd();
    glEndList();  // Finalizar la lista
}

void display() {
    glClear(GL_COLOR_BUFFER_BIT);
    glCallList(miLista);  // Llamar a la lista para dibujar el triángulo
    glFlush();
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);
    glutInitWindowSize(500, 500);
    glutCreateWindow("Display Lists en OpenGL");
    
    crearDisplayList();  // Crear la Display List

    glutDisplayFunc(display);
    glutMainLoop();

    return 0;
}
``` 

# 2.4 Matrices
Se usan para transformar los objetos de la escena **3D** y proyectarlos en la pantalla **2D**. Tenemos 3 matrices principales:
- **Model Matrix (Matriz de Modelo):** define la posición, rotación y escala de un objeto en el mundo.
- **View Matrix (Matriz de Vista):** define la posición y orientación de la cámara.
- **Projection Matrix (Matriz de Proyección):** define cómo se proyecta la escena en la pantalla (*perspectiva o ortogonal*).

La función `glMatrixMode(mode)` se usa para seleccionar cuál de estas matrices se va a modificar:
- `GL_MODELVIEW`: modifica la **Model Matrix** y la **View Matrix** juntas.
- `GL_PROJECTION`: modifica la **Projection Matrix**.
- `GL_TEXTURE`: modifica la matriz de transformación de coordenadas de textura.

Cuando seleccionamos una matriz con `glMatrixMode(mode)`, podemos reiniciarla con `glLoadIndentity()`. Esto la resetea a la matriz identidad (*sin transformaciones*).
![[Pasted image 20250301155409.png]]
## 2.4.1 Model Matrix
Transforma un objeto desde su espacio local al espacio del mundo. Se usa para:
- **Trasladar** (*mover*) el objeto `glTranslatef(x,y,z)`
- **Rotar** el objeto `glRotatef(ángulo, x,y,z)`
- **Escalar** el objeto `glScalef(x,y,z)`

```c++
glMatrixMode(GL_MODELVIEW);
glLoadIdentity();
glTranslatef(1.0f, 2.0f, -5.0f); // Mueve el objeto a (1,2,-5)
glRotatef(45, 0, 1, 0);          // Rota 45° alrededor del eje Y
glScalef(2.0f, 2.0f, 2.0f);      // Doble de tamaño en cada eje
```

![[Pasted image 20250301155440.png]]
## 2.4.2 View Matrix
Mueve el **mundo** para simular el movimiento de la cámara.
Para definir una cámara usamos:
`gluLookAt(eyeX, eyeY, eyeZ, centerX, centerY, centerZ, upX, upY, upZ);`

- `(eyeX, eyeY, eyeZ)`: Posición de la cámara.
- `(centerX, centerY, centerZ)`: Punto donde mira la cámara.
- `(upX, upY, upZ)`: Dirección "arriba" de la cámara.

```c++
glMatrixMode(GL_MODELVIEW);
glLoadIdentity();
gluLookAt(0.0f, 0.0f, 5.0f,  // Cámara en (0,0,5)
          0.0f, 0.0f, 0.0f,  // Mira hacia (0,0,0)
          0.0f, 1.0f, 0.0f); // Arriba es (0,1,0)
```

## 2.4.3 Projection Matrix
Define cómo la escena se proyecta en la pantalla. Dos tipos principales:
- **Perspectiva** (`glFrustrum` o `gluPerspective`): Simula la visión humana, los objetos más lejanos se ven más pequeños.
```c++
glMatrixMode(GL_PROJECTION);
glLoadIdentity();
gluPerspective(45.0,  // Ángulo de visión vertical
               4.0/3.0, // Relación de aspecto (ancho/alto)
               0.1, 100.0); // Distancia del near y far plane
```

![[Pasted image 20250301160144.png]]
>[!Nota]
> `gluPerspective( fovy, aspect, near, far)
> - **Fovy** se trata del ángulo con el que se abre el foco, normalmente 45 o 60 grados
> - **Aspect** es el tamaño de la ventana que suele ser el **alto/ancho**.
> - **Near y Far** que determinan desde donde ve la cámara hasta donde ve la cámara


- **Ortogonal** (`glOrtho`): No hay perspectiva, los objetos mantienen su tamaño sin importar la distancia.
```c++
glMatrixMode(GL_PROJECTION);
glLoadIdentity();
glOrtho(-10, 10, -10, 10, -1, 1); // Espacio visible de -10 a 10 en X e Y
```
![[Pasted image 20250301155947.png]]

>[!IMPORTANTE]
>Si estás hasta los huevos del salto este que mete la cámara cuando quieres rotar un objeto, tirate este código y disfruta (no se ni por qué funciona, cosas de matemáticos supongo):
>```C
> gluLookAt(((float)DISTANCIA * (float)sin(alpha) * cos(beta)), ((float)DISTANCIA * (float)sin(beta)), ((float)DISTANCIA * cos(alpha) * cos(beta)), 0, 0, 0, 0, (cos(beta) >= 0 ? 1 : -1), 0);
> ```


## 2.4.4 Push y Pop Matrix
Se usan para recordar una transformación anterior antes de aplicar nuevas transformaciones temporales:
- `glPushMatrix()`: Guarda la matriz actual en una pila (stack).
- `glPopMatrix()`: Restaura la última matriz guardada en la pila.


# 2.5 Mover Cosas
Usamos `glutIdleFunc` y le pasamos una función que actualice la posición del objeto. Usamos `glutPostRedisplay()` para redibujar la escena.

```c++
#include <GL/glut.h>

float posX = -2.0f; // Posición inicial del cubo
float velocidad = 0.01f; // Velocidad de movimiento

void actualizar() {
    posX += velocidad; // Mover el cubo

    // Rebotar en los límites
    if (posX > 2.0f || posX < -2.0f) {
        velocidad = -velocidad;
    }

    glutPostRedisplay(); // Redibujar la escena
}

void display() {
    glClear(GL_COLOR_BUFFER_BIT);
    glLoadIdentity();

    glTranslatef(posX, 0.0f, 0.0f); // Aplicar la transformación de traslación
    glutSolidCube(0.5); // Dibujar un cubo

    glFlush();
}

void iniciar() {
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f); // Fondo negro
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluOrtho2D(-3, 3, -3, 3); // Espacio de dibujo 2D
    glMatrixMode(GL_MODELVIEW);
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);
    glutInitWindowSize(500, 500);
    glutCreateWindow("Movimiento con glutIdleFunc");

    iniciar();
    glutDisplayFunc(display);
    glutIdleFunc(actualizar); // Se ejecutará continuamente

    glutMainLoop();
    return 0;
}
```

# 2.6 Teclado
Para poder interactuar con el programa usamos estas funciones:
- `glutKeyboardFunc(myTeclado)`
- `glutSpecialFunc(myTeclasespeciales)`

```c++
void myTeclado(unsigned char tras, int x, int y)
{
	switch (tras) {
	case 'l':
		glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);
		break;
	case 'c':
		glDisable(GL_CULL_FACE);
		break;
	case 'k':
		glPolygonMode(GL_FRONT_AND_BACK, GL_FILL);
		break;
	case 'x':
		glEnable(GL_CULL_FACE);
		break;
	default:
		break;
	}
	// Se se modificou algo redebúxase
	glutPostRedisplay();
}

void myTeclasespeciales(int cursor, int x, int y)
{
	switch (cursor)
	{
		//Traslaciones:
	case GLUT_KEY_F1:
		break;
	case GLUT_KEY_F2:
		break;
	case GLUT_KEY_F3:
		break;
	case GLUT_KEY_F4:
		break;
	case GLUT_KEY_F5:
		break;
	case GLUT_KEY_F6:
		break;

		//Giros:
	case GLUT_KEY_UP:
		beta += INCREMENTO;
		break;
	case GLUT_KEY_DOWN:
		beta -= INCREMENTO;
		break;
	case GLUT_KEY_RIGHT:
		alpha -= INCREMENTO;
		break;
	case GLUT_KEY_LEFT:
		alpha += INCREMENTO;
		break;
	default:
		break;
	}

	//if (alpha >= PI * 2.0 && alpha <= 0) alpha = 0;
	//if (beta >= PI * 2.0 && beta <= 0) beta = 0; //hay que repasarlo para evitar el salto

	glutPostRedisplay();
}
```

# 2.7 Iluminación (crazy shi)

## 2.7.1 Concepto General de Luces en OpenGL 1.2
En OpenGL 1.2, las luces se manejan manualmente. Se deben encender y configurar adecuadamente para iluminar los objetos de la escena.

1. **Encender las luces** con `glEnable(GL_LIGHTING)`. 
2. **Activar luces específicas** con `glEnable(GL_LIGHT0)`, `GL_LIGHT1`, etc.
3. **Definir las propiedades de la luz** (ambiente, difusa, especular).
4. **Definir las propiedades del material** de los objetos.
5. **Configurar el modelo de sombreado** (`GL_FLAT` o `GL_SMOOTH`).

**Ejemplo**
```cpp
glEnable(GL_CULL_FACE); //Al dibujar la mitad de caras aumenta el rendimiento
glEnable(GL_DEPTH_TEST);  // Activar el buffer de profundidad
glEnable(GL_LIGHTING);    // Activar el sistema de iluminación
glEnable(GL_LIGHT0);      // Activar la luz 0
glEnable(GL_COLOR_MATERIAL);  // Permitir que los colores del material afecten la luz
glShadeModel(GL_SMOOTH);  // Sombreado suave
```


## 2.7.1 Luz Ambiente
Es la luz general que se dispersa en todas las direcciones. No tiene una dirección específica y afecta a todos los objetos de la misma manera.

```cpp
GLfloat luz_ambiente[] = {0.2, 0.2, 0.2, 1.0}; //indicamos de que color es la luz ambiente (RGBA)
glLightModelfv(GL_LIGHT_MODEL_AMBIENT, luz_ambiente);

glLightModelfv(GL_LIGH_MODEL_LOCAL_VIEWER, TRUE); //se usa para que las luces afecten a ambar caras de un material

glLightModelfv(GL_LIGHT_MODEL_LOCAL_VIEWER, TRUE); //colocal el modo de cáculo con el observador lcoal, si no todos los cálculos se hacen como si el observador estuviese situadosobre el eje z
```

## 2.7.2 Componentes de la Luz GL
Cada luz en OpenGL tiene tres componentes principales:

1. **Ambiente (`GL_AMBIENT`)** → Luz difusa general.
2. **Difusa (`GL_DIFFUSE`)** → Luz que se refleja en todas direcciones (según Lambert).
3. **Especular (`GL_SPECULAR`)** → Reflejos brillantes.

```cpp
GLfloat light_ambient[] = {0.2, 0.2, 0.2, 1.0};  // Componente ambiente
GLfloat light_diffuse[] = {1.0, 1.0, 1.0, 1.0};  // Componente difusa
GLfloat light_specular[] = {1.0, 1.0, 1.0, 1.0}; // Componente especular
glLightfv(GL_LIGHT0, GL_AMBIENT, light_ambient);
glLightfv(GL_LIGHT0, GL_DIFFUSE, light_diffuse);
glLightfv(GL_LIGHT0, GL_SPECULAR, light_specular);
```

#### **Consecuencias en la escena**
`GL_AMBIENT`: Proporciona iluminación base para evitar sombras negras.  
`GL_DIFFUSE`: Define el color principal de los objetos iluminados.  
`GL_SPECULAR`: Agrega reflejos brillantes y realistas.


## 2.7.3 Luz Direccional

Simula una fuente de luz situada en el infinito, como el Sol. Todos los rayos de luz son paralelos y no sufren atenuación con la distancia. Para definir una luz direccional en OpenGL, el cuarto componente del vector de posición debe ser 0.

```cpp
GLfloat direccion[] = {0.0, -1.0, 0.0, 0.0}; // Dirección de la luz direccional
 glLightfv(GL_LIGHT0, GL_POSITION, direccion);
```

- No tiene una posición definida, solo una dirección. 
- No se atenúa con la distancia.
- Ilumina de manera uniforme en toda la escena.

![[Pasted image 20250401125724.png]]
## 2.7.4 Luz Local (Puntual)

Es una luz que emana desde un punto específico en el espacio y se propaga en todas direcciones, similar a una bombilla.

```cpp
GLfloat posicion[] = {0.0, 5.0, 0.0, 1.0}; // Último valor = 1 → Luz local
 glLightfv(GL_LIGHT0, GL_POSITION, posicion);
```

- Se define con un vector de posición donde el cuarto componente es 1. 
- Se atenúa con la distancia al objeto iluminado.
- Puede ser usada para simular fuentes de luz naturales o artificiales dentro de la escena.

 ![[Pasted image 20250401125734.png]]

## 2.7.5 Focos de Luz (Spotlights)

Un foco es una luz localizada que emite luz en una dirección específica dentro de un cono de apertura determinado.

```cpp
GLfloat posicion[] = {0.0, 5.0, 0.0, 1.0}; // Posición del foco
GLfloat direccion[] = {0.0, -1.0, 0.0};    // Dirección hacia donde apunta
 glLightfv(GL_LIGHT0, GL_POSITION, posicion);
 glLightfv(GL_LIGHT0, GL_SPOT_DIRECTION, direccion);
 glLightf(GL_LIGHT0, GL_SPOT_CUTOFF, 30.0); // Ángulo de apertura en grados
 glLightf(GL_LIGHT0, GL_SPOT_EXPONENT, 10.0); // Atenuación dentro del cono
```

- Posee una posición y una dirección. 
- Ilumina en un cono de apertura definido por `GL_SPOT_CUTOFF`.
- La intensidad de la luz se degrada desde el centro del cono hacia los bordes según `GL_SPOT_EXPONENT`.

![[Pasted image 20250401130203.png]]

## 2.7.6 Atenuación de la Luz
Las luces locales y focos pueden perder intensidad con la distancia. OpenGL permite ajustar esto con tres parámetros:

- `GL_CONSTANT_ATTENUATION`: Factor de atenuación constante.
- `GL_LINEAR_ATTENUATION`: Factor de atenuación lineal.
- `GL_QUADRATIC_ATTENUATION`: Factor de atenuación cuadrática.

```cpp
glLightf(GL_LIGHT0, GL_CONSTANT_ATTENUATION, 1.0);
glLightf(GL_LIGHT0, GL_LINEAR_ATTENUATION, 0.1);
glLightf(GL_LIGHT0, GL_QUADRATIC_ATTENUATION, 0.01);
```


## 2.7.7 Modelos de Sombreado
En OpenGL 1.2, el modelo de iluminación se basa en el uso de normales para determinar cómo se refleja la luz en una superficie. A diferencia de otros sistemas que asignan una única normal a una cara, OpenGL permite definir normales en los vértices, permitiendo así diferentes modelos de sombreado. Los modelos principales de sombreado en OpenGL 1.2 son:

- **Sombreado plano (Flat Shading)**
- **Sombreado de Gouraud (Gouraud Shading)**
- **Sombreado de Phong (Phong Shading)** (no implementado directamente en hardware en OpenGL 1.2)

### Sombreado Plano (Flat Shading)
El sombreado plano es el modelo más simple y eficiente. En este modelo, todo el polígono se representa con un único color, que es calculado usando la normal de una de sus caras.

- **Eficiencia alta**: Solo se calcula un color por cara. 
- **Menos realista**: No representa bien superficies curvas, ya que los cambios de color entre polígonos son abruptos.
- **Útil para objetos de apariencia facetada**.

Para habilitar el sombreado plano en OpenGL, se utiliza la función:

```c
 glShadeModel(GL_FLAT);
```

La normal de la cara se puede definir manualmente con `glNormal3fv`, por ejemplo:

```c
 glNormal3fv(normals[1]);
 glVertex3fv(vertices[1]);
 glVertex3fv(vertices[5]);
 glVertex3fv(vertices[6]);
 glVertex3fv(vertices[2]);
```

![[Pasted image 20250401133854.png]]
### Sombreado de Gouraud
El sombreado de Gouraud mejora la suavidad de la iluminación interpolando colores entre los vértices de una cara.

- **Más realista que el sombreado plano**.  
- **Mayor costo computacional**: Se deben calcular colores en cada vértice y luego interpolarlos.
- **Funciona bien con superficies curvas**.

Para habilitar el sombreado de Gouraud:
```c
 glEnable(GL_SMOOTH);
 glShadeModel(GL_SMOOTH);
```

El cálculo del color en cada vértice se realiza con normales por vértice:

```c
 for (i = 0; i < 3; i++) {
     glNormal3fv(normAvg[i]);
     glVertex3fv(vertices[i]);
 }
```

**Errores comunes**
- No normalizar las normales de los vértices. 
- Usar Gouraud Shading en escenas con iluminación especular fuerte, lo que puede causar efectos visuales incorrectos.

![[Pasted image 20250401133911.png]]
### Sombreado de Phong
El modelo de sombreado de Phong es una mejora sobre el de Gouraud, en el que en lugar de interpolar colores, se interpolan las normales y luego se calcula la iluminación en cada píxel.

- **Mayor realismo**: Captura mejor los reflejos especulares y las transiciones de luz. 
- **Mayor costo computacional**: Se deben calcular las normales interpoladas en cada píxel.


OpenGL 1.2 no implementa directamente el sombreado de Phong, por lo que debe hacerse manualmente mediante programación en shaders o interpolación en software.

**Errores comunes**
- No realizar la interpolación correcta de las normales.
- Usar el modelo en hardware antiguo sin soporte para cálculo en fragmentos.


## 2.7.8 Materiales y Luces
El color percibido de un objeto depende no solo de su material, sino también de la luz que incide sobre él y de cómo esta se refleja en su superficie.

### Interacción entre la Luz y los Materiales
Cuando un objeto es iluminado, la luz puede interactuar con su superficie de distintas maneras:

- **Reflexión ambiente**: Es la luz que se dispersa uniformemente en todas direcciones.
- **Reflexión difusa**: Se basa en la orientación de la superficie respecto a la fuente de luz.
- **Reflexión especular**: Genera brillos y destaca zonas de reflejo intenso.  
- **Emisión**: Define si el objeto emite luz propia.
  
Por ejemplo, si un objeto es rojo y es iluminado por una luz azul, este se verá negro porque absorbe la luz azul y no refleja ninguna componente roja.

### Parámetros de Material en OpenGL
OpenGL permite definir materiales usando la función:

```c
void glMaterial{if}{v}(GLenum face, GLenum pname, TYPE *param);
```

Donde:
- **`face`**: Indica si el material afecta a las caras delanteras (`GL_FRONT`), traseras (`GL_BACK`) o ambas (`GL_FRONT_AND_BACK`).

- **`pname`**: Especifica la propiedad a modificar.
 
- **`param`**: Define los valores de la propiedad.
 

|Propiedad|Descripción|Valor por defecto|
|---|---|---|
|`GL_AMBIENT`|Componente ambiental del material|(0.2, 0.2, 0.2, 1.0)|
|`GL_DIFFUSE`|Componente difusa del material|(0.8, 0.8, 0.8, 1.0)|
|`GL_SPECULAR`|Reflexión especular del material|(0.0, 0.0, 0.0, 1.0)|
|`GL_SHININESS`|Controla la concentración del brillo especular|0.0 (rango [0, 128])|
|`GL_EMISSION`|Color de luz emitida por el objeto|(0.0, 0.0, 0.0, 1.0)|


### Configuración de Material en OpenGL
Los materiales se definen para que respondan de manera específica a la luz. Ejemplo de configuración de material:

```c
GLfloat ambient[] = { 0.7, 0.7, 0.7, 1.0 };
GLfloat diffuse[] = { 0.1, 0.5, 0.8, 1.0 };
GLfloat specular[] = { 1.0, 1.0, 1.0, 1.0 };
GLfloat shininess = 50.0;

glMaterialfv(GL_FRONT, GL_AMBIENT, ambient);
glMaterialfv(GL_FRONT, GL_DIFFUSE, diffuse);
glMaterialfv(GL_FRONT, GL_SPECULAR, specular);
glMaterialf(GL_FRONT, GL_SHININESS, shininess);
```


# 2.8 Texturas

### **Habilitar el mapeado de texturas**
Antes de cualquier dibujo:
```c
glEnable(GL_TEXTURE_2D);
```

> Para dejar de usar textura en un momento puntual:

```c
glDisable(GL_TEXTURE_2D);
```

> También se puede "no usar ninguna" sin deshabilitar:

```c
glBindTexture(GL_TEXTURE_2D, 0);
```

### **Generar un índice para la textura**
Creamos un ID de textura (como un puntero o referencia):

```c
GLuint textura;
glGenTextures(1, &textura);
```

### **Cargar la imagen (desde fichero, no procedural)**
Necesitas una librería como `stb_image`:
```c
#define STB_IMAGE_IMPLEMENTATION
#include <stb_image.h> // cuidado con el nombre correcto: no std_image.h

int width, height, nrChannels;
unsigned char *data = stbi_load("nombre_de_la_imagen.jpg", &width, &height, &nrChannels, 0);
```

### **Vincular la textura y establecer sus parámetros**
```c
glBindTexture(GL_TEXTURE_2D, textura);

// Repetir textura si se pasa del rango [0,1]
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);

// Filtrado (mejor calidad o rendimiento)
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

### **Subir la imagen como textura a la GPU**
```c
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0,
             GL_RGB, GL_UNSIGNED_BYTE, data);

glTextImage(GLenum objetivo, GLint nive, GLint componentes, GLsizei ancho, GLsizei alto, GLint borde, GLenum formato, GLenum tipo, const Glvoid *pixels) 
```

- **Objetivo:** define que textura hay que definir y debe ser GL_TEXTURE_23
- **Nivel:** Indica el nivel de datalle de las texturas y es normalmente 0
- **Componenetes:** Inidca el número de componentes de color usados en cada píxel
- **Ancho:** especifica la anchura de la textura. Debe ser potencia entera de 2
- **Alto**: especifica la anchura de la textura. Debe ser potencia entera de 2
- **Borde:** controla el número de pixels de borde que OpenGl usará
- **FOrmato:** indica el tipo de valores de color esperados
- **Tipo:** indica el tipo de la variables que almacena los valores de color por píxel
- **Pixels:** puntero a la variable que almacena los colores por pixel.

> ⚠️ Solo usar `GL_RGB` si tu imagen no tiene canal alfa. Si tiene transparencia, usar `GL_RGBA`.


### **Liberar memoria del sistema**
```c
stbi_image_free(data);
```

### **Asignar la textura antes de dibujar**
```c
glBindTexture(GL_TEXTURE_2D, textura);
```


### **Dibujar con coordenadas de textura**
```c
glBegin(GL_QUADS);
    glTexCoord2f(0.0f, 0.0f); glVertex3f(-1.0f, -1.0f, 0.0f);
    glTexCoord2f(1.0f, 0.0f); glVertex3f( 1.0f, -1.0f, 0.0f);
    glTexCoord2f(1.0f, 1.0f); glVertex3f( 1.0f,  1.0f, 0.0f);
    glTexCoord2f(0.0f, 1.0f); glVertex3f(-1.0f,  1.0f, 0.0f);
glEnd();
```

> 📌 Las coordenadas `(u, v)` siempre están en el rango `[0, 1]`.

## Uso de listas de display
Para asociar una textura a un objeto que dibujas con `glCallList(lista)`:

```c
glBindTexture(GL_TEXTURE_2D, objeto.textura);
glCallList(objeto.lista);
```

### Ejemplo completo de carga de textura

```c
int myCargaTexturas(char *nome) {
    int textura;
    glGenTextures(1, &textura);
    glBindTexture(GL_TEXTURE_2D, textura);

    int width, height, nrChannels;
    unsigned char *data = stbi_load(nome, &width, &height, &nrChannels, 0);

    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0,
                 GL_RGB, GL_UNSIGNED_BYTE, data);

    stbi_image_free(data);
    return textura;
}
```

### Si tienes dos texturas (ejemplo suelo y pared)

```c
glBindTexture(GL_TEXTURE_2D, suelo.textura);
glCallList(suelo.lista);

glBindTexture(GL_TEXTURE_2D, pared.textura);
glCallList(pared.lista);
```