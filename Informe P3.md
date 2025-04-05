# 1. Introducción
Este programa implementa el **problema productor-consumidor** usando:
- **Múltiples hilos productores y consumidores**
- **Un buffer compartido (cola de tamaño fijo)**
- **Mecanismos de sincronización con mutexes y condiciones**
- **Barrera para sincronizar el inicio de los hilos**


# 2. Ejecución
El código se compila con `gcc` y se ejecuta desde la terminal con los siguientes pasos:

**1. Compilación:**
```bash
gcc -o prod_cons prod_cons.c -lpthread
```

**2. Ejecución:**
```bash
./prod_cons <número_de_productores> <número_de_consumidores>
```

Ejemplo:
```bash
./prodcons 2 2
```


Durante la ejecución:
- Para cada productor, se pide por consola:
    - El nombre del fichero de entrada (de donde se leerán caracteres)
    - El tiempo de espera ($T$)

- Para cada consumidor, solo se solicita el valor $T$

Cada consumidor guarda lo que consume en un fichero con nombre automático: `fileCons_<id>.txt`


# 3. Funcionamiento del código

## Productores:
- Leen caracteres de un fichero proporcionado por el usuario.
- Solo insertan caracteres alfanuméricos o `'*'`.
- Inserta los caracteres en el buffer circular mientras haya espacio.
- Cuando el carácter leído es `'*'`, se interpreta como **fin de la producción**.

## Consumidores:
- Retiran elementos del buffer.
- Escriben los caracteres en su propio fichero (`fileCons_x.txt`).
- Cuando reciben un `'*'`, entienden que un productor ha terminado y aumentan el contador global `contProd`.
- Terminan cuando todos los productores han terminado.


## Sincronización:
- Uso de `pthread_mutex_t` para evitar condiciones de carrera en el buffer
- Uso de `pthread_cond_t` para coordinar espera entre productores (cuando el buffer está lleno) y consumidores (cuando está vacío)
- Uso de `pthread_barrier_t` para asegurar que todos los hilos comiencen juntos


# 4. Conclusiones obtenidas
1. **Correcta implementación del patrón productor-consumidor** con múltiples hilos.
2. Se gestionan adecuadamente los accesos concurrentes a la memoria compartida mediante **mutexes y variables de condición**.
3. La lógica de finalización es clara: consumidores no terminan hasta que se ha recibido un `'*'` de cada productor.
4. El programa es **modular y escalable**, lo que permite probar distintos escenarios con diferentes cantidades de hilos.
5. El uso de **barreras** garantiza sincronía inicial entre hilos.