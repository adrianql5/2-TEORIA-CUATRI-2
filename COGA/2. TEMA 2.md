# 2.1 Ejes, Puntos y Vectores
Tanto los puntos, como los vectores y los objetos, se representarán sobre los tres ejes cartesianos.
- $y$, altura
- $x$, anchura
- $z$, profundidad

Los **puntos** informan sobre la **ubicación del objeto**, mientras que los **vectores** indican la **dirección hacia la que este se dirige**.

Tenemos:
- **Escena:** compuesta por los objetos situados en una determinada posición (coordenadas locales de los objetos).
- **Cámara:** determinada por su posición, su orientación y un foco (frustrum).

# 2.2 Triangulación
Una **primitiva** es la interpretación de un conjunto de vértices dibujados de una manera específica en pantalla (*puntos, líneas, triángulos...*).

Para modelar cualquier objeto se usa la técnica de **teselación**, que consiste en la creación o transformación de figuras geométricas a partir de polígonos. Se suelen usar **triángulos**, porque es la figura geométrica mas simple que delimita un plano.

$$\text{Número de triángulos} = \text{Número de lados del polígono} -2$$

# 2.3 Identificación de Caras
Un plano está definido por su **vector normal** (*vector perpendicular al plano*). Para calcular la normal se debe hallar el producto vectorial de dos vectores $\vec{u}$ y $\vec{v}$, contenido en el plano.

>[!Nota] Calcular Vectores entre 2 puntos
>$A(x_1​,y_1​,z_1​),B(x_2​,y_2​,z_2​)$
>$AB=(x_2​−x_1​,y_2​−y_1​,z_2​−z_1​)$

El producto vectorial $\vec{u} \times \vec{v}$ en $R^3$ se calcula como el determinante de la matriz $3 \times 3$ conformada por los vectores unitarios de los ejes, los componentes del vector $\vec{u}$ y los del $\vec{v}$:

$$
\begin{vmatrix} 
\vec{i} & \vec{j} & \vec{k} \\ 
\vec{u}_1 & \vec{u}_2 & \vec{u}_3 \\ 
\vec{v}_1 & \vec{v}_2 & \vec{v}_3 
\end{vmatrix}
$$

Dados los vectores:
$$\mathbf{u} = (1, 0, 0), \quad \mathbf{v} = (0, 1, 0)$$

Calculamos el producto cruz:
$$\mathbf{u} \times \mathbf{v} = \begin{vmatrix} \mathbf{i} & \mathbf{j} & \mathbf{k} \\ 1 & 0 & 0 \\ 0 & 1 & 0 \end{vmatrix}$$

Si simplemente queremos saber la dirección del vector normal aplicamos **la regla de la mano derecha** según el recorrido de los vértices de la cara de una figura. Si el pulgar apunta hacia nosotros es positiva, si no es negativa.

Saber dónde apuntar los vectores normales es muy importante, porque en **OpenGL** solo se ven los objetos cuya normal se dirige a la cámara, por lo que hay que tener en cuenta el orden de creación de los vértices, puesto que determinan el vector normal.

Se puede forzar que se vean todas las caras desde cualquier posición con `glDisable(GL_CULL_FACE)`.

El **vector normal** de un plano puede ser **normalizado** (se puede transformar en un vector unitario).

>[!Nota] **Normalizar un Vector**
> Hace que nuestro vector mida como mucho 1 unidad:
> 
> $\mathbf{\hat{v}} = \frac{\mathbf{v}}{|\mathbf{v}|}$
> 
> $|\mathbf{v}| = \sqrt{x^2 + y^2 + z^2}$

Es útil para procesos de iluminación de objetos (como calcular el valor de un rayo reflejado) por lo que conviene normalizar dicho vector mediante **glEnable(GL_NORMALICE)** al inicio del código.

