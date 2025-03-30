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

## 2.7.2 Tipos de Luz en OpenGL
OpenGL 1.2 permite 4 tipos de luces:

1. **Luz Ambiente**
2. **Luz Direccional**
3. **Luz Local (Puntual)**
4. **Focos de Luz (Spotlights)**

### **2.7.2.1 Luz Ambiente**
Es la luz general que se dispersa en todas las direcciones. No tiene una dirección específica y afecta a todos los objetos de la misma manera.


```cpp
GLfloat luz_ambiente[] = {0.2, 0.2, 0.2, 1.0}; // Luz tenue global
glLightModelfv(GL_LIGHT_MODEL_AMBIENT, luz_ambiente);
```

### **2.7.2.2 Luz Direccional**
Simula una fuente de luz situada en el infinito, como el Sol. Todos los rayos de luz son paralelos.

```cpp
GLfloat direccion_luz[] = {1.0, 1.0, 1.0, 0.0}; // Último valor = 0 → Luz direccional
glLightfv(GL_LIGHT0, GL_POSITION, direccion_luz);
```

>[!Nota]
>``` 
>void glLightfv(GLenum light, GLenum pname, const GLfloat *params);
>```



### **2.7.2.3 Luz Local (Puntual)**

Se origina en un punto específico y se propaga en todas direcciones, como una bombilla.


```cpp
GLfloat posicion[] = {0.0, 5.0, 0.0, 1.0}; // Último valor = 1 → Luz local
glLightfv(GL_LIGHT0, GL_POSITION, posicion);
```

### **2.7.2.4 Focos de Luz (Spotlights)**

Es una luz direccional con un ángulo de apertura, como una linterna.

```cpp
GLfloat posicion[] = {0.0, 5.0, 0.0, 1.0}; // Posición de la luz
GLfloat direccion[] = {0.0, -1.0, 0.0};    // Dirección hacia donde apunta
glLightfv(GL_LIGHT0, GL_POSITION, posicion);
glLightfv(GL_LIGHT0, GL_SPOT_DIRECTION, direccion);
glLightf(GL_LIGHT0, GL_SPOT_CUTOFF, 30.0); // Ángulo de apertura en grados
```


## 2.7.3 Componentes de la Luz en OpenGL
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


## 2.7.4 Atenuación de la Luz**
Las luces locales y focos pueden perder intensidad con la distancia. OpenGL permite ajustar esto con tres parámetros:

- `GL_CONSTANT_ATTENUATION`: Factor de atenuación constante.
- `GL_LINEAR_ATTENUATION`: Factor de atenuación lineal.
- `GL_QUADRATIC_ATTENUATION`: Factor de atenuación cuadrática.


```cpp
glLightf(GL_LIGHT0, GL_CONSTANT_ATTENUATION, 1.0);
glLightf(GL_LIGHT0, GL_LINEAR_ATTENUATION, 0.1);
glLightf(GL_LIGHT0, GL_QUADRATIC_ATTENUATION, 0.01);
```


## 2.7.5 Modelos de Sombreado
OpenGL permite dos modelos de sombreado:

- `GL_FLAT`: Un solo color por cara → efectos de bordes marcados.
- `GL_SMOOTH`: Interpolación de color entre los vértices → transiciones suaves.

```cpp
glShadeModel(GL_FLAT);  // Bordes marcados
glShadeModel(GL_SMOOTH); // Transiciones suaves
```


## 2.7.6 Resumen General

|Tipo de Luz|Propiedades|Ejemplo de Uso|Consecuencias|
|---|---|---|---|
|**Ambiente**|Ilumina toda la escena uniformemente|Luz global de la escena|No genera sombras ni efectos de profundidad|
|**Direccional**|Rayo de luz paralelo (como el Sol)|Luz solar en juegos|No se atenúa con la distancia|
|**Local (Puntual)**|Se propaga desde un punto en todas direcciones|Bombilla, lámpara|Se atenúa con la distancia|
|**Foco**|Luz direccional con un cono de iluminación|Linterna, faro de coche|Efectos de iluminación más realistas|
## 2.7.8 Chuleta
Cambiar los valores en los vectores pasados a `glLightfv` afecta cómo se comporta la luz en la escena. Vamos a ver qué ocurre al modificar cada parámetro, con ejemplos y sus consecuencias.

## **1. `GL_AMBIENT` (Luz ambiente)**

```cpp
GLfloat luz_ambiente[] = {R, G, B, A};
glLightfv(GL_LIGHT0, GL_AMBIENT, luz_ambiente);
```

**Efecto:** Cambia el color y la intensidad de la luz ambiente en la escena.

|Valor|Efecto|
|---|---|
|`{1.0, 1.0, 1.0, 1.0}`|Luz ambiente blanca brillante (todo se ilumina).|
|`{0.5, 0.5, 0.5, 1.0}`|Luz grisácea, los objetos se ven menos iluminados.|
|`{0.0, 0.0, 0.0, 1.0}`|Sin luz ambiente, las sombras serán muy marcadas.|
|`{1.0, 0.0, 0.0, 1.0}`|Todo tendrá un tono rojo en las zonas sin luz directa.|

**Consecuencia:** Si se pone un valor muy alto, la escena parecerá "lavada" porque no habrá sombras fuertes.

## **2. `GL_DIFFUSE` (Luz difusa)**

```cpp
GLfloat luz_difusa[] = {R, G, B, A};
glLightfv(GL_LIGHT0, GL_DIFFUSE, luz_difusa);
```

**Efecto:** Cambia la iluminación sobre superficies que reciben luz directa.

|Valor|Efecto|
|---|---|
|`{1.0, 1.0, 1.0, 1.0}`|Iluminación blanca intensa en áreas expuestas.|
|`{0.5, 0.5, 0.5, 1.0}`|Iluminación más suave en superficies.|
|`{0.0, 0.0, 0.0, 1.0}`|No hay iluminación difusa (los objetos no reflejan luz).|
|`{1.0, 0.0, 0.0, 1.0}`|Las áreas iluminadas serán rojas.|

**Consecuencia:** Si la luz difusa es demasiado baja, los objetos pueden parecer planos y sin volumen.

## **3. `GL_SPECULAR` (Luz especular o reflejos)**

```cpp
GLfloat luz_especular[] = {R, G, B, A};
glLightfv(GL_LIGHT0, GL_SPECULAR, luz_especular);
```

**Efecto:** Afecta los reflejos en superficies brillantes.

|Valor|Efecto|
|---|---|
|`{1.0, 1.0, 1.0, 1.0}`|Reflejos blancos y brillantes (metal o plástico pulido).|
|`{0.5, 0.5, 0.5, 1.0}`|Reflejos más suaves y naturales.|
|`{0.0, 0.0, 0.0, 1.0}`|Sin reflejos (parece material mate).|
|`{1.0, 0.0, 0.0, 1.0}`|Reflejos rojizos en superficies brillantes.|

**Consecuencia:** Si se usa un color de especular muy diferente al difuso, los reflejos pueden verse irreales.

## **4. `GL_POSITION` (Posición de la luz)**

```cpp
GLfloat posicion[] = {X, Y, Z, W};
glLightfv(GL_LIGHT0, GL_POSITION, posicion);
```

**Efecto:** Controla la posición de la luz en la escena.

|Valor|Efecto|
|---|---|
|`{0.0, 10.0, 0.0, 1.0}`|Luz sobre la escena, similar al sol en el cielo.|
|`{10.0, 0.0, 0.0, 1.0}`|Luz a un costado, genera sombras largas.|
|`{0.0, 0.0, 10.0, 1.0}`|Luz frontal, minimiza sombras.|
|`{0.0, 0.0, 1.0, 0.0}`|Luz direccional, apunta siempre en una dirección fija.|

**Consecuencia:**

- Si `W=1.0`, la luz es **puntual** (parecida a un foco).
- Si `W=0.0`, la luz es **direccional** (como la luz del sol, sin punto de origen).


## **5. `GL_SPOT_DIRECTION` (Dirección de un foco)**

```cpp
GLfloat direccion_foco[] = {X, Y, Z};
glLightfv(GL_LIGHT0, GL_SPOT_DIRECTION, direccion_foco);
```

**Efecto:** Define la dirección en la que apunta un foco de luz.

|Valor|Efecto|
|---|---|
|`{0.0, -1.0, 0.0}`|Foco que apunta hacia abajo.|
|`{1.0, 0.0, 0.0}`|Foco que apunta a la derecha.|
|`{0.0, 0.0, -1.0}`|Foco que apunta hacia el frente.|

**Consecuencia:** Si la dirección no coincide con la posición de la luz, el efecto será extraño.


## **6. `GL_SPOT_CUTOFF` (Ángulo del foco)**

```cpp
glLightf(GL_LIGHT0, GL_SPOT_CUTOFF, angulo);
```

**Efecto:** Controla qué tan amplio es el cono de luz del foco.

|Valor|Efecto|
|---|---|
|`180.0`|Se convierte en una luz normal sin recorte.|
|`90.0`|Ilumina todo lo que está en un hemisferio.|
|`30.0`|Cono de luz más cerrado, como una linterna.|
|`5.0`|Haz de luz muy fino, como un láser.|

**Consecuencia:**

- Si el ángulo es **muy grande**, la luz parece una luz normal.
- Si el ángulo es **muy pequeño**, puede que la luz sea difícil de ver.


## **7. `GL_SPOT_EXPONENT` (Intensidad del foco)**

```cpp
glLightf(GL_LIGHT0, GL_SPOT_EXPONENT, valor);
```

**Efecto:** Controla qué tan fuerte es la luz en el centro del cono.

|Valor|Efecto|
|---|---|
|`0`|La luz es uniforme dentro del cono.|
|`10`|Más brillante en el centro, se atenúa hacia los lados.|
|`100`|Muy concentrada en el centro, los bordes casi no iluminan.|

 **Consecuencia:** Un valor alto simula una linterna potente con un centro más brillante.
