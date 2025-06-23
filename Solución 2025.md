# Teoría
## 1.Cursores en SQL Incorporado
### a) ¿Para qué sirven los cursores en SQL embebido?
Un **cursor** en SQL incorporado es una estructura especial que se utiliza para **manejar y recorrer los resultados de una consulta** (relación resultado) en el contexto de un programa escrito en un lenguaje anfitrión como C, C++ o Java.

### b) ¿Cómo recupera las tuplas un programa que utiliza cursores?
Primero se declara el **cursor** y a**sociarlo a una consulta** que devuelva varias filas. Después **se abre el cursor**, lo cual ejecuta la consulta y genera una relación temporal con todas las tuplas resultado.

Posteriormente, el programa utiliza la instrucción `fetch` para **recuperar las tuplas** **una por una**. Cada vez que se ejecuta un `fetch`, el cursor avanza a la siguiente tupla y sus valores se asignan a variables del lenguaje anfitrión.
```
EXEC SQL FETCH nombre_cursor INTO :var1, :var2, ...;
```
- `nombre_cursor`: es el cursor declarado y abierto previamente
- `:var1`. `var2`: son variables que reciben los valores de la tupla recuperada
Normalmente este proceso de `fetch` **se repite mediante un bucle** hasta que no queden tuplas por recuperar. Finalmente **se cierra el cursor.**

## 2. Algoritmo de recuperación
Una transacción está **comprometida** cuando el registro `<T comprometida>` se guarda en almacenamiento estable. 
- Si el sistema **falla antes** de esto, la transacción debe deshacerse cuando el sistema se recupere.
- Si el sistema **falla después**, la información guardada es suficiente para rehacer las operaciones de la transacción y asegurar que sus cambios no se pierdan.

Cada vez que una transacción modifica algo, se guarda un registro: qué cambió, valor anterior, valor nuevo. 
- `rehacer(T)`: vuelve a aplicar los cambios de la transacción  `T` (deja los valores nuevos)
- `deshacer(T)`: revierte los cambios de `T` (restaura los valores anteriores).

Para evitar leer todo el registro histórico, se usan **checkpoints**, dónde se anotan las transacciones activas y se asegura que todo el historial hasta ese punto está en disco.

Cuando el sistema se reinicia tras un fallo, lee el **registro histórico** y, usando el último **checkpoint**, determina qué transacciones:
- **Se deben rehacer:** las que estaban comprometidas
- **Se deben deshacer:** las que no se habían comprometido

Si falla todo el sistema se realiza en dos fases:
- **Fase rehacer:** se vuelven a aplicar los cambios de las transacciones comprometidas.
	- Se busca el último registro del **rh** de la forma `<revisión L>`. L es la lista de transacciones activa en ese momento
	- Se toma la lista `L` del checkpoint como punto de partida
	- **Lista deshacer inicial** es igual a `L`
	- Recorremos el **rh** desde el **checkpoint** hasta el final. Para cada registro de actualización se aplica **rehacer**. Cuando se encuentra un registro de la forma `<Ti iniada>` se añade a la lista de deshacer. Cuando se encuentra `<Ti comprometida>` o `<Ti abortada>`, se elimina `Ti` de la lista de deshacer.

- **Fase deshacer:** se revierten todos los cambios de las transacciones no comprometidas.
	-  Se recorre el historial desde el final hasta el checkpoint. Si se encuentra un registro de actualización de una transacción que está en la lista de deshacer, se aplica el **deshacer** y se escribe un registro de compensación (*solo-rehacer*) para dejar constancia de que se ha hecho ese cambio.
	- Si se encuentra un registro de la forma `<Ti iniciada>` para una transacción en la lista de deshacer se escribe `<Ti abortada>` y se borra de la lista de deshacer
	- Se termina cuando la lista de **deshacer** está vacía.

Si solo falla una transacción y no el sistema simplemente se aplica el **deshacer** a esa transacción

El algoritmo asegura que se ejecuten todas las operaciones de la transacción o ninguna, lo que asegura la **atomicidad**. La **consistencia** requiere que la bd pase de un estado válido a otro estado válido, lo que también se garantiza mediante el algoritmo.

## 3. Grafo de espera
Es un tipo de grafo que nos permite **detectar y prevenir interbloqueos**. Los vértices son todas las transacciones del sistema. 

Se añadirá una arista $T_i \rightarrow T_j$ cuando la transacción $T_i$ solicite un elemento de datos que posea en ese momento la transacción $T_j$. La arista de borrará cuando la transacción $T_j$ deje de poseer el elemento que necesita $T_i$

**Existe un interbloqueo** en el sistema si, y solo si, el **grafo de procedencia contiene un ciclo**. Para detectar los interbloqueos, el sistema debe mantener el grafo de espera e **invocar periódicamente** a un **algoritmo que busque un ciclo en él**.

## 4. Gestión de privilegios mediante roles
Para conceder un conjunto de privilegios e un conjunto de usuarios o roles usamos:
```sql
grant autorización1, autorización2...
	on relación/vista
to usuario/rol1, usuario/rol2
```

Los **roles** agrupan usuarios para poder concederles y revocarles autorizaciones a todos a la vez. Se pueden conceder roles a usuarios y roles a otros roles.

A un usuario puede permitírsele trasladar una autorización a otros usuarios añadiendo la cláusula `with grant option` al final de la instrucción de `grant` . El paso de una autorización de un usuario a otro se representa mediante un **grafo de autorización** para ese privilegio:
- Habrá una arista $U_i \rightarrow U_j$ , si el usuario $U_i$ le concedió la autorización al $U_j$
- Un usuario tendrá esa autorización si existe un camino desde la raíz

## 5. Secuencialidad en cuanto a conflictos
Se dice que una planificación $S$ es **secuenciable en cuanto a conflictos** si es equivalente en cuanto a conflictos a una planificación secuencial. $S$ y $S'$ son **equivalentes en cuanto a conflictos** si $S$ se puede transformar en $S'$ mediante intercambios de instrucciones **no conflictivas**.


# Prácticas
## 1. Planificación de transacciones
### a) ¿Es secuenciable en cuanto a conflictos?
Si porque es equivalente en cuanto a conflictos a una planificación secuencial

### b) ¿Se cumple el protocolo de bloqueo en dos fases estricto?
No, porque este protocolo impide liberar un **bloqueo X** hasta que se compromete la transacción, asegurando que ninguna otra puede leer el dato modificado hasta que se comprometa la transacción que lo escribió.