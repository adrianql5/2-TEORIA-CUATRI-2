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

