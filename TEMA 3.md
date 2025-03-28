# 3.1 Transformaciones

El **Local Space** es el sistema de coordenadas en el que están definidos los vértices de un objeto. A partir de ahí, la escena se compone en el **World Space**, que es el espacio global de la escena.

Para convertir coordenadas del **Local Space** al **World Space**, cada objeto se somete a una transformación mediante la **matriz del modelo** (_Model Matrix_), que aplica traslación, rotación y escala.

- **Local Space:** Coordenadas relativas al origen del propio objeto.

- **Matriz del modelo:** Transforma las coordenadas locales en coordenadas del mundo.
 
- **World Space:** Espacio donde se ubican todos los objetos transformados.
 
- La nueva posición de cada vértice se obtiene multiplicando sus coordenadas locales por la **matriz del modelo**.

# 3.2 Traslación
En **OpenGL**, las transformaciones se representan con matrices **4x4**, ya que se trabaja con coordenadas homogéneas en 3D. Esto permite combinar múltiples transformaciones (traslación, rotación y escala) en una sola matriz.

Si queremos trasladar un punto **P(x,y,z)P(x, y, z)** a una nueva posición **P′(x′,y′,z′)P'(x', y', z')**, usamos la ecuación:
$$x' = x + T_x$$ $$y' = y + T_y$$
$$z' = z + T_z$$

Donde $T_x, T_y, T_z$ son las cantidades de traslación en cada eje.

>[!Nota]
> Si te ralla que la matriz sea 4x4 es porque teniendo en cuenta las ecuaciones anteriores, te quedaría una matriz 3x4 multiplicando a una 3x1. Por eso añade esa fila extra, si no no sería posible realizar una traslación.
> POR ESO SE DICE QUE USAMOS UN **SISTEMA DE COORDENADAS HOMOGÉNEO**

En OpenGL, la traslación se representa con la siguiente matriz:

$$T = \begin{bmatrix} 1 & 0 & 0 & T_x \\ 0 & 1 & 0 & T_y \\ 0 & 0 & 1 & T_z \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

Y multiplicamos por el **vector de coordenadas homogéneas**:

$$P = \begin{bmatrix} x \\ y \\ z \\ 1 \end{bmatrix}$$

La multiplicación de matrices se resuelve así:
$$T \cdot P = \begin{bmatrix} 1 & 0 & 0 & T_x \\ 0 & 1 & 0 & T_y \\ 0 & 0 & 1 & T_z \\ 0 & 0 & 0 & 1 \end{bmatrix} \cdot \begin{bmatrix} x \\ y \\ z \\ 1 \end{bmatrix} = \begin{bmatrix} x + T_x \\ y + T_y \\ z + T_z \\ 1 \end{bmatrix}$$
Supongamos que tenemos el punto:
$$P = (2, 3, 5)$$

Y queremos trasladarlo **4 unidades en X**, **-2 en Y** y **3 en Z**. La matriz de traslación sería:

$$T = \begin{bmatrix} 1 & 0 & 0 & 4 \\ 0 & 1 & 0 & -2 \\ 0 & 0 & 1 & 3 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

Multiplicamos:
$$\begin{bmatrix} 1 & 0 & 0 & 4 \\ 0 & 1 & 0 & -2 \\ 0 & 0 & 1 & 3 \\ 0 & 0 & 0 & 1 \end{bmatrix} \cdot \begin{bmatrix} 2 \\ 3 \\ 5 \\ 1 \end{bmatrix} = \begin{bmatrix} 2 + 4 \\ 3 - 2 \\ 5 + 3 \\ 1 \end{bmatrix} = \begin{bmatrix} 6 \\ 1 \\ 8 \\ 1 \end{bmatrix}$$

**El punto trasladado es** $P'(6,1,8)$.

# 3.3 Rotación
La función en OpenGL:

```cpp
glRotatef(angle, vx, vy, vz);
```

Recibe:
- `angle`: el ángulo de rotación en grados.
- `(vx, vy, vz)`: el eje unitario alrededor del cual queremos rotar.

Cuando rotamos alrededor de un eje específico, aplicamos una de las siguientes matrices:

## 3.3.1 Rotación alrededor del eje X
Si giramos un punto $(x, y, z)$ alrededor del eje **X**, la coordenada $x$ no cambia, pero $y$ y $z$ rotan:

$$R_x(\theta) = \begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & \cos\theta & -\sin\theta & 0 \\ 0 & \sin\theta & \cos\theta & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

Las nuevas coordenadas son:
$$\begin{cases} x' = x \\ y' = y \cos\theta - z \sin\theta \\ z' = y \sin\theta + z \cos\theta \end{cases}$$

Esto significa que:

- Si $\theta = 0^\circ$, $y' = y$ y $z' = z$ (sin cambios).
- Si $\theta = 90^\circ$, $y' = -z$ y $z' = y$ (intercambio de valores con signo).
- Si $\theta = 180^\circ$, $y' = -y$ y $z' = -z$ (inversión total).


🔹 **Ejemplo OpenGL:**

```cpp
glRotatef(90, 1, 0, 0);  // Rota 90 grados alrededor del eje X
```

Este giro intercambia las coordenadas yy y zz.

## 3.3.2 Rotación alrededor del eje Y

Si giramos alrededor del eje **Y**, las coordenadas yy no cambian, pero xx y zz rotan:

$$R_y(\theta) = \begin{bmatrix} \cos\theta & 0 & \sin\theta & 0 \\ 0 & 1 & 0 & 0 \\ -\sin\theta & 0 & \cos\theta & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

Las nuevas coordenadas son:
$$\begin{cases} x' = x \cos\theta + z \sin\theta \\ y' = y \\ z' = -x \sin\theta + z \cos\theta \end{cases}$$

🔹 **Ejemplo OpenGL:**

```cpp
glRotatef(90, 0, 1, 0);  // Rota 90 grados alrededor del eje Y
```

Este giro intercambia x y z.

## 3.3.3 Rotación alrededor del eje Z
Si giramos alrededor del eje **Z**, las coordenadas z no cambian, pero x y y rotan:

$$R_z(\theta) = \begin{bmatrix} \cos\theta & -\sin\theta & 0 & 0 \\ \sin\theta & \cos\theta & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

Las nuevas coordenadas son:

$$\begin{cases} x' = x \cos\theta - y \sin\theta \\ y' = x \sin\theta + y \cos\theta \\ z' = z \end{cases}$$

🔹 **Ejemplo OpenGL:**

```cpp
glRotatef(90, 0, 0, 1);  // Rota 90 grados alrededor del eje Z
```

Esto rota los puntos en el plano XY.

---

## 3.3.4 Rotación en un Eje Arbitrario
Cuando usamos `glRotatef(angle, vx, vy, vz)`, OpenGL rota alrededor de un eje arbitrario **(vx, vy, vz)**. Para calcular la matriz de rotación en un eje cualquiera, usamos la fórmula de **Rodrigues**:

$$R(\theta) = \begin{bmatrix} t + vx^2(1-t) & vx vy(1-t) - vz s & vx vz(1-t) + vy s & 0 \\ vy vx(1-t) + vz s & t + vy^2(1-t) & vy vz(1-t) - vx s & 0 \\ vz vx(1-t) - vy s & vz vy(1-t) + vx s & t + vz^2(1-t) & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

Donde:
- $s = \sin\theta$
- $t = \cos\theta$
- $(vx, vy, vz)$ es un **vector unitario**, es decir:
	$\sqrt{vx^2 + vy^2 + vz^2} = 1$

```cpp
glRotatef(45, 1, 1, 1);  // Rota 45° alrededor del eje (1,1,1)
```

>[!Importante]
> Hay tener en cuenta si primero se hace una rotación o una traslación.
>
>- **`glRotatef()` antes de `glTranslatef()`** → El objeto rota sobre sí mismo antes de moverse.  
>
>- **`glTranslatef()` antes de `glRotatef()`** → El objeto se mueve primero y luego gira **alrededor del origen**.
>
>Imagina que tienes una pelota en el centro de una mesa.
>- **Primero rotar, luego trasladar** → La pelota gira sobre sí misma y luego se mueve.  
>- **Primero trasladar, luego rotar** → La pelota se aleja del centro y luego gira alrededor de la mesa.

# 3.4 Escalado
El **escalado** es una transformación geométrica que cambia el tamaño de un objeto en el espacio 3D. No es una **transformación de cuerpo rígido**, porque modifica las dimensiones del objeto, lo que puede cambiar su forma.

Cuando escalamos un objeto en OpenGL con `glScalef(Sx, Sy, Sz)`, estamos multiplicando cada coordenada del objeto por un factor:

- `Sx` → Escalado en el eje X. 
- `Sy` → Escalado en el eje Y.
- `Sz` → Escalado en el eje Z.

💡 **Ejemplo:**
- Si `Sx = 2`, el objeto se hace **el doble de ancho**.
- Si `Sy = 0.5`, el objeto se hace **la mitad de alto**.
- Si `Sz = 1`, el objeto **no cambia en el eje Z**.

En OpenGL, el escalado se representa con una **matriz de transformación 4x4**, que se multiplica con las coordenadas del objeto:

$$M_{escala} = \begin{bmatrix} Sx & 0 & 0 & 0 \\ 0 & Sy & 0 & 0 \\ 0 & 0 & Sz & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

Multiplicando esta matriz por un punto $P(x, y, z, 1)$:

$$P' = \begin{bmatrix} Sx & 0 & 0 & 0 \\ 0 & Sy & 0 & 0 \\ 0 & 0 & Sz & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \cdot \begin{bmatrix} x \\ y \\ z \\ 1 \end{bmatrix} = \begin{bmatrix} Sx \cdot x \\ Sy \cdot y \\ Sz \cdot z \\ 1 \end{bmatrix}$$

Esto significa que cada coordenada del objeto se **multiplica** por su respectivo factor de escala.

## 3.4.1 Tipos de Escalado
**1. Escalado Uniforme (`Sx = Sy = Sz`)**
Si usamos el **mismo factor** en todos los ejes (`Sx = Sy = Sz`), el objeto mantiene su proporción y solo cambia de tamaño.

```cpp
glScalef(2, 2, 2); // Escala el objeto al doble en todas las direcciones
```
🔹 **Ejemplo:** Una esfera duplicará su tamaño, pero seguirá siendo una esfera.

**2. Escalado No Uniforme (`Sx ≠ Sy ≠ Sz`)**
Si usamos **factores diferentes** en cada eje, el objeto cambia de forma.

```cpp
glScalef(2, 1, 0.5); // Doble de ancho, misma altura, mitad en Z
```

🔹 **Ejemplo:** Un cubo puede convertirse en un rectángulo alargado.

## 3.4.2 Problema del Escalado y el Origen
📌 **Si escalamos un objeto directamente, se moverá si no está en el origen (0,0,0).**  
Esto pasa porque la matriz de escalado multiplica todas las coordenadas, incluyendo su posición.

🔹 **Ejemplo:**
- Si tenemos un objeto en `(5,0,0)` y aplicamos `glScalef(2,2,2)`, **también se duplicará su distancia al origen**, quedando en `(10,0,0)`.

Para evitar que el objeto **se desplace** al escalarlo, usamos esta secuencia:
1. **Trasladamos el objeto al origen (`T(-xc, -yc, -zc)`).**
2. **Aplicamos el escalado (`Sx, Sy, Sz`).**
3. **Devolvemos el objeto a su posición original (`T(xc, yc, zc)`).**

# 3.5 Matrices en OpenGL
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

# 3.6 Recordatorio
- **Matrices 4×4**: Se utilizan para todas las transformaciones.
 
- **Modos de matriz (`glMatrixMode`)**: 
    - `GL_PROJECTION`: Define la proyección de la escena en la pantalla.
    - `GL_MODELVIEW`: Se usa para transformar los objetos en la escena.
    
- **Matriz identidad**: Se restablece con `glLoadIdentity()`.

- **Coordenadas homogéneas**: OpenGL usa un sistema de coordenadas en 4D.
 
- **Orden de aplicación**: Las transformaciones se aplican de derecha a izquierda.
 
- **No conmutativas**: Cambiar el orden altera el resultado.
 
- **Pila de matrices**: OpenGL usa una estructura **FILO (First In, Last Out)** para almacenar matrices.

- **Estados de matrices**: Se pueden copiar y restaurar con `glPushMatrix()` y `glPopMatrix()`.