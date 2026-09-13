---
title: Queue
tags:
  - data-structures
alias:
  - cola
  - FIFO
---

## 1. Qué es y cómo funciona

### Intuición

Una cola es como la fila de un banco: el primero que llega es el primero que se atiende.
Resuelve problemas donde el orden de llegada debe preservarse (FIFO). Es una estructura con acceso restringido a sus dos extremos: se inserta por un lado y se elimina por el otro.

### Definición / propiedades

- Estructura FIFO (First-In, First-Out)
- Solo se inserta por el fondo (`rear`) y solo se elimina por el frente (`front`)
- No hay acceso directo a elementos intermedios
- Invariante clave: el frente siempre representa el elemento más antiguo que sigue en la cola

### Representación

![](/attachments/grimorio/data-structures/queue.svg)

Puede implementarse sobre [[array]], [[dynamic array]] o [[linked list]], y la elección importa. Sobre una lista enlazada basta con guardar dos punteros (`front` y `rear`) para tener ambas operaciones en $O(1)$. Sobre un array la implementación ingenua es una trampa: si se desencola desplazando todos los elementos una posición a la izquierda, `dequeue` cuesta $O(n)$. La solución estándar es tratar el arreglo como **circular** (_ring buffer_), moviendo los índices con módulo en lugar de mover los datos.

## 2. Operaciones y complejidad

### Operaciones principales

- `enqueue(x)` inserta un elemento en el fondo
- `dequeue()` elimina y retorna el elemento del frente
- `peek()` / `front()` consulta el frente sin eliminarlo
- `isEmpty()` verifica si está vacía

### Complejidad

| Operación | Tiempo |
| :-------- | :----- |
| `enqueue` | $O(1)$ |
| `dequeue` | $O(1)$ |
| `peek`    | $O(1)$ |
| Espacio   | $O(n)$ |

> **Nota:** el $O(1)$ de `enqueue` es _amortizado_ si la cola se apoya en un [[dynamic array]] que se redimensiona al llenarse. Con array circular de capacidad fija, o con lista enlazada, es $O(1)$ en el peor caso.

### Detalles operativos

- **Underflow:** hacer `dequeue` o `peek` sobre una cola vacía es un error, no un valor por defecto.
- **Overflow:** con capacidad fija, encolar en una cola llena falla (o descarta, según la política elegida).
- **Ambigüedad vacía/llena:** en un array circular, `front == rear` ocurre tanto cuando está vacía como cuando está llena. Se desambigua guardando el `size`, o dejando un slot siempre libre.
- **Búsqueda:** no hay acceso a elementos intermedios; encontrar uno exige desencolar todo, $O(n)$.
