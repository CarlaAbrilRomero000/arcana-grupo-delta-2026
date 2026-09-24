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

![](/attachments/grimorio/data-structures/queue-circular-animada.svg)

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

## 3. Implementación

### Idea de implementación
Hay dos estrategias típicas. Sobre [[linked list]] basta mantener dos punteros: `front` (para desencolar) y `rear` (para encolar); cada operación mueve un puntero y reengancha un enlace. Sobre array, la implementación ingenua desplaza elementos en cada `dequeue`, lo cual es $O(n)$; la solución estándar es un **buffer circular**, donde `front` y `rear` avanzan con aritmética modular (`(i + 1) % capacidad`) y reciclan el espacio liberado sin mover datos.

### Invariantes
- `front` siempre apunta al elemento más antiguo disponible para desencolar.
- `rear` siempre apunta a la próxima posición libre donde insertar.
- En el buffer circular, se mantiene `size` explícito para distinguir "vacía" de "llena" cuando `front == rear`.
- Ningún `dequeue` ni `peek` se ejecuta sobre una cola vacía.

### Ejemplo de código

```python
class Queue:
    def __init__(self, capacidad):
        self.data = [None] * capacidad
        self.capacidad = capacidad
        self.front = 0
        self.size = 0

    def enqueue(self, x):
        if self.size == self.capacidad:
            raise Exception("Queue overflow")
        rear = (self.front + self.size) % self.capacidad
        self.data[rear] = x
        self.size += 1

    def dequeue(self):
        if self.size == 0:
            raise Exception("Queue underflow")
        x = self.data[self.front]
        self.front = (self.front + 1) % self.capacidad
        self.size -= 1
        return x
```

#### Ejemplo de uso típico: BFS por niveles

Recorrido por niveles (BFS) sobre un grafo: la cola garantiza que se visite todo lo que está a distancia $k$ antes que lo que está a distancia $k+1$.


```python
from collections import deque

def bfs(grafo, inicio):
    visitados = {inicio}
    orden = []
    q = deque([inicio])
    while q:
        nodo = q.popleft()
        orden.append(nodo)
        for vecino in grafo[nodo]:
            if vecino not in visitados:
                visitados.add(vecino)
                q.append(vecino)
    return orden
```

**Entrada:** el siguiente grafo, partiendo del nodo `A`.

```
    A
   / \
  B   C
   \ /
    D
    |
    E
```

```python
grafo = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A", "D"],
    "D": ["B", "C", "E"],
    "E": ["D"],
}

print(bfs(grafo, "A"))
```

**Salida:**

```
['A', 'B', 'C', 'D', 'E']
```

`A` se visita primero (nivel 0). Sus vecinos `B` y `C` entran a la cola y se visitan a continuación (nivel 1). `D` es vecino tanto de `B` como de `C`, pero ya está en `visitados` la segunda vez que se lo encuentra, así que solo se encola una vez, al procesar `B` (nivel 2). Por último se visita `E` (nivel 3). Ningún nodo de nivel 2 se visita antes que uno de nivel 1, que es justamente la garantía que da BFS con una cola.

## 4. Uso y criterio

### Casos de uso
- BFS (Breadth-First Search) y cualquier recorrido por niveles
- Planificación de tareas *round-robin* en sistemas operativos
- Buffers productor/consumidor entre procesos de distinta velocidad (I/O, red, streaming)
- Colas de impresión, de pedidos, de mensajes: cualquier atención por orden de llegada

### Cuándo NO usarlo
- Si el orden relevante no es el de llegada sino una prioridad explícita (usar cola de prioridad / heap).
- Si hace falta procesar lo más reciente primero (usar [[stack]]).
- Si se necesita insertar o eliminar por ambos extremos (usar [[deque]]).
- Si se necesita acceso aleatorio frecuente a posiciones intermedias (usar [[array]]).

### Comparaciones
- **vs [[stack]].** orden opuesto: la cola procesa FIFO, la pila LIFO. BFS (Breadth-First Search) usa cola porque explora nivel por nivel, visitando primero los nodos más cercanos al origen; DFS (Depth-First Search) usa pila porque profundiza primero antes de retroceder.
- **vs [[deque]].** el deque generaliza a la cola, permitiendo operar en ambos extremos. Usar una cola en lugar de un deque comunica y garantiza la restricción FIFO por diseño, en vez de dejar disponibles operaciones que romperían esa semántica.
- **vs cola de prioridad (heap).** la cola ordena estrictamente por orden de llegada; la cola de prioridad ordena por una prioridad explícita, sin importar cuándo se insertó cada elemento. Cuando "más antiguo" y "más urgente" no coinciden, corresponde una cola de prioridad.

### Ventajas / desventajas

| Ventajas | Desventajas |
| :--- | :--- |
| Operaciones $O(1)$ garantizadas (buffer circular o linked list) | No permite acceso ni búsqueda eficiente a elementos intermedios |
| Modelo mental simple: refleja el orden natural de llegada | Rígida: no sirve si hace falta reordenar o acceder por prioridad |
| Buena localidad de caché con buffer circular | Con capacidad fija hay que resolver overflow y la ambigüedad vacía/llena |

### Señales de reconocimiento
- "Procesar en el orden en que llegan" / "primero en entrar, primero en salir".
- Recorrido "por niveles" o "por oleadas" (BFS, propagación de estados).
- Simulación de una fila de espera real, o "los primeros k en llegar".

## 5. Relaciones y extensiones

### Variantes

- Cola circular: normalmente implementada sobre un array. Cuando se alcanza el final de este, se puede volver al comienzo aprovechando las posiciones que quedaron libres. Esto permite utilizar eficientemente el espacio disponible y evita tener que desplazar elementos.
- Cola de prioridad: cada elemento de la cola tiene asociada una prioridad. El siguiente elemento en ser procesado no necesariamente es el que llegó primero, sino el que posee mayor prioridad. Una estructura común para implementarla eficientemente es el heap.
- Deque (double-ended queue): permite insertar y eliminar elementos desde ambos extremos. Es más flexible que una cola tradicional, ya que permite comportamientos similares tanto a una cola como a una pila.

### Relación con otras estructuras

- Pila: ambas restringen el acceso directo a los elementos y definen claramente dónde se insertan y eliminan.
- Árboles y grafos: el algoritmo BFS utiliza una cola para garantizar que los nodos se procesen por niveles.
- Arrays y linked lists: una cola puede implementarse utilizando arrays o listas enlazadas.

### Notas avanzadas

#### Persistencia
Una cola puede almacenarse en memoria secundaria para conservar los elementos pendientes incluso cuando el programa deja de ejecutarse. Esto resulta útil en sistemas de procesamiento de tareas y colas de mensajes, donde no se debe perder el trabajo pendiente ante una interrupción.

#### Concurrencia
En sistemas donde varios procesos o hilos producen y consumen elementos simultáneamente, es necesario utilizar mecanismos de sincronización para evitar problemas de condiciones de carrera. Para estos escenarios existen implementaciones de colas concurrentes diseñadas específicamente para permitir el acceso seguro de múltiples procesos o hilos.

#### Paralelismo
Una cola puede utilizarse para distribuir tareas entre distintos workers, de modo que los productores agregan trabajo y los consumidores lo toman para ejecutarlo. Esto permite repartir dinámicamente la carga de trabajo.

#### Colas de mensajes
Las colas de mensajes se utilizan para comunicar componentes de una aplicación de forma asíncrona. Un productor coloca mensajes en la cola y un consumidor los procesa posteriormente. Esto permite desacoplar componentes y manejar picos de demanda mediante el almacenamiento temporal de tareas.

## 6. Referencias y recursos
- [Queue Data Structure - Devopedia](https://devopedia.org/queue-data-structure)
- [[COR2011]] - Chapter 10.1 Stacks and queues
- [[DRO1995]] - Chapter 3 Stacks and queues
- Geek for Geeks. Queue Data Structure. Recuperado de: [Queue Data Structure - GeeksforGeeks](https://www.geeksforgeeks.org/dsa/queue-data-structure/)
- Geek for Geeks. Queue in Python. Recuperado de: [Queue in Python - GeeksforGeeks](https://www.geeksforgeeks.org/python/queue-in-python/)
- [VisuAlgo - Queue](https://visualgo.net/en/queue) _(visualización interactiva de enqueue/dequeue y del buffer circular)_