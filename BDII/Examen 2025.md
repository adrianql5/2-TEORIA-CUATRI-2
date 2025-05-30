# Teoría

## 1. Cursores en SQL Embebido

### a) ¿Para qué sirven los cursores en SQL embebido?
Explica para qué sirven los cursores en SQL embebido.

### b) ¿Cómo recupera las tuplas un programa que utiliza cursores?
Describe cómo recupera las tuplas un programa escrito en un lenguaje con SQL embebido que utiliza cursores.

---

## 2. Algoritmo de recuperación

Describe cómo funciona un algoritmo de recuperación en bases de datos, destacando:
- Atomicidad
- Consistencia

---

## 3. Grafo de espera

- ¿Qué es un grafo de espera?
- ¿Cómo se crean y destruyen aristas?
- ¿Cómo se detecta si hay interbloqueo?

---

## 4. Gestión de privilegios mediante roles

Explica cómo se otorgan y gestionan los privilegios de acceso a usuarios mediante **roles** en un sistema de bases de datos:
- Cadenas de roles
- Cómo se aplican los privilegios efectivos al usuario

---

## 5. Secuencialidad en cuanto a conflictos

- ¿Qué significa que dos planificaciones sean **secuenciables en cuanto a conflictos**?
- Define **secuencialidad en cuanto a conflictos**

---

# Prácticas

## 1. Planificación de transacciones

Se proporciona una planificación de transacciones `T1`, `T2` y `T3`.

### a) ¿Es secuenciable en cuanto a conflictos?

### b) ¿Se cumple el protocolo de bloqueo en dos fases estricto?
Justifica tu respuesta.

---

## 2. Planificación y recuperación

Se proporciona una planificación de operaciones sobre valores `A` y `B` (almacenamiento diferido).

- Calcula los valores de `A` y `B` y el contenido del **registro histórico (RH)**.
- Hay un error en el momento 12: explica cómo hace la recuperación la base de datos y los valores resultantes de `A` y `B`.

---

## 3. Ejercicio de hashing dinámico

Se presenta una tabla de **hashing dinámico** indexada por un atributo.

- Representar cómo queda el **directorio general** y los **cajones** después de las inserciones.

---

## 4. Control de concurrencia con planificación temporal

Se proporciona una tabla con los **tiempos de llegada** de transacciones y un **algoritmo de control de concurrencia**.
