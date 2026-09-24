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

Puede implementarse sobre [[array]], [[dynamic array]] o [[linked list]], y la elección importa. Sobre una lista enlazada basta con guardar dos punteros (`front` y `rear`) para tener ambas operaciones en $O(1)$. Sobre un array la implementación ingenua es una trampa: si se desencola desplazando todos los elementos una posición a la izquierda, `dequeue` cuesta $O(n)$. La solución estándar es tratar el arreglo como **circular** (*ring buffer*), moviendo los índices con módulo en lugar de mover los datos.

## 2. Operaciones y complejidad

### Operaciones principales
- `enqueue(x)` inserta un elemento en el fondo
- `dequeue()` elimina y retorna el elemento del frente
- `peek()` / `front()` consulta el frente sin eliminarlo
- `isEmpty()` verifica si está vacía

### Complejidad

| Operación | Tiempo |
| :--- | :--- |
| `enqueue` | $O(1)$ |
| `dequeue` | $O(1)$ |
| `peek` | $O(1)$ |
| Búsqueda / recorrido | $O(n)$ |
| Espacio | $O(n)$ |

> **Nota:** el $O(1)$ de `enqueue` es *amortizado* si la cola se apoya en un [[dynamic array]] que se redimensiona al llenarse. Con array circular de capacidad fija, o con lista enlazada, es $O(1)$ en el peor caso.

### Detalles operativos
- **Underflow:** hacer `dequeue` o `peek` sobre una cola vacía es un error, no un valor por defecto.
- **Overflow:** con capacidad fija, encolar en una cola llena falla (o descarta, según la política elegida).
- **Ambigüedad vacía/llena:** en un array circular, `front == rear` ocurre tanto cuando está vacía como cuando está llena. Se desambigua guardando el `size`, o dejando un slot siempre libre.
- **Búsqueda:** no hay acceso a elementos intermedios; encontrar uno exige desencolar todo, $O(n)$.

## 3. Implementación

### Idea de implementación
Mantener dos referencias a los extremos y no tocar nunca el medio.
Con lista enlazada: `front` es el `head` y `rear` es el `tail`; se inserta en el tail y se elimina desde el head (nunca al revés, porque eliminar el tail en una lista simple es $O(n)$).
Con array circular: los índices avanzan con `(i + 1) % capacidad`, de modo que el espacio liberado al frente se reutiliza sin desplazar datos.

![](/attachments/grimorio/data-structures/queue-circular-animada.svg)

En la animación se ve el punto clave: al encolar `F` en el índice 5, `rear` no se sale del arreglo sino que vuelve al 0, y el siguiente `enqueue` ocupa el slot que los `dequeue` habían liberado. Los datos nunca se mueven de lugar; lo único que avanza son los dos índices.

### Invariantes
- `front` referencia siempre al elemento más antiguo vivo; `rear`, al más reciente
- La cola está vacía si y solo si `size == 0` (y entonces `front` y `rear` son nulos o irrelevantes)
- Todo elemento sale exactamente en el mismo orden relativo en que entró
- Ninguna operación elemental recorre ni desplaza el contenido

### Ejemplo de código

```python
class Queue:
    def __init__(self, capacidad):
        self.data = [None] * capacidad
        self.front = 0
        self.size = 0

    def enqueue(self, x):
        if self.size == len(self.data):
            raise Exception("Queue overflow")
        self.data[(self.front + self.size) % len(self.data)] = x
        self.size += 1

    def dequeue(self):
        if self.size == 0:
            raise Exception("Queue underflow")
        x = self.data[self.front]
        self.front = (self.front + 1) % len(self.data)
        self.size -= 1
        return x

    def is_empty(self):
        return self.size == 0
```

#### Ejemplo de uso típico

Recorrido por niveles (BFS) sobre un grafo: la cola garantiza que se visite todo lo que está a distancia $k$ antes que lo que está a distancia $k+1$.

```python
def bfs(grafo, inicio):
    visitados = {inicio}
    q = Queue(len(grafo))
    q.enqueue(inicio)

    while not q.is_empty():
        nodo = q.dequeue()
        print(nodo)
        for vecino in grafo[nodo]:
            if vecino not in visitados:
                visitados.add(vecino)
                q.enqueue(vecino)
```

## 4. Uso y criterio

### Casos de uso
- BFS (Breadth-First Search) y cualquier recorrido por niveles
- Planificación de tareas *round-robin* en sistemas operativos
- Buffers productor/consumidor entre procesos de distinta velocidad (I/O, red, streaming)
- Colas de impresión, de pedidos, de mensajes: cualquier atención por orden de llegada

### Cuándo NO usarlo
- Cuando el orden de atención depende de una **prioridad** y no de la llegada: ahí corresponde una *priority queue* (heap), no una cola.
- Cuando hay que procesar lo más reciente primero: eso es un [[stack]].
- Cuando se necesita acceso aleatorio o búsqueda frecuente por posición o clave: [[array]], [[map]] o [[hash table]].
- Cuando hace falta insertar o eliminar en ambos extremos: [[deque]].

### Comparaciones
- **vs [[stack]].** Orden opuesto sobre la misma restricción de acceso: el stack procesa en LIFO y la cola en FIFO. Es la comparación que más cambia el resultado de un algoritmo: reemplazar la cola de un BFS por una pila lo convierte en un DFS, y el recorrido deja de garantizar caminos mínimos en aristas. Elegí cola cuando el orden de llegada (o la distancia) debe preservarse; elegí pila cuando querés invertirlo o deshacerlo.
- **vs [[deque]].** El deque es una generalización que incluye a la cola: permite insertar y eliminar en ambos extremos. Usar un deque como cola funciona, pero expone operaciones que rompen el contrato FIFO. Preferí la cola cuando querés que la estructura garantice ese contrato por diseño.
- **vs [[linked list]].** La cola es una lista enlazada con acceso restringido a los extremos. Esa restricción es una ventaja: hace imposible insertar en el medio y romper la semántica. Usá la lista cuando necesites operar en posiciones arbitrarias.

### Ventajas / desventajas

Ventajas:

- Operaciones $O(1)$ en ambos extremos
- Modelo mental simple y justo: nadie se adelanta
- Desacopla productores de consumidores sin coordinación explícita

Desventajas:

- Muy limitada: sin acceso aleatorio ni búsqueda eficiente
- Con array circular hay que administrar capacidad y wrap-around a mano
- No expresa prioridades ni vencimientos; si el problema los tiene, la cola no alcanza

### Señales de reconocimiento
- "Atender / procesar en orden de llegada"
- "Nivel por nivel", "camino más corto en cantidad de pasos" (BFS)
- "Productor y consumidor", "buffer", "pipeline"
- "Turnos", "espera", "cola de trabajos"

## 5. Relaciones y extensiones

### Variantes
- **Cola circular (*ring buffer*):** implementación sobre arreglo de capacidad fija; es la variante estándar en sistemas embebidos y buffers de I/O.
- **Cola de prioridad:** el frente no es el más antiguo sino el de mayor prioridad; se implementa con un heap binario, con `enqueue`/`dequeue` en $O(\log n)$.
- **Cola acotada / bloqueante:** con capacidad máxima; el productor espera si está llena y el consumidor si está vacía.
- **Cola con dos pilas:** dos [[stack]] (entrada y salida) simulan una cola con `dequeue` en $O(1)$ amortizado; es el truco clásico para obtener una cola persistente o funcional.

### Relación con otras estructuras
- Es el motor de BFS en grafos y árboles, igual que el [[stack]] lo es de DFS
- Caso particular de [[deque]] (restringido a un extremo por operación)
- Se implementa sobre [[linked list]], [[array]] o [[dynamic array]]

### Notas avanzadas

#### Concurrencia
La cola es la estructura de intercambio por excelencia entre hilos. Protegerla con un mutex es simple pero genera contención, ya que productores y consumidores compiten por el mismo lock aunque toquen extremos distintos. Las implementaciones *lock-free* (Michael–Scott) usan `compare-and-swap` sobre los punteros de cabeza y cola para dejar avanzar a ambos lados en paralelo; escalan mejor, pero deben resolver el problema ABA y la recuperación de memoria.

#### Persistencia
Una cola persistente (que conserva sus versiones anteriores) no sale gratis con la representación de dos punteros. La construcción funcional clásica —dos listas, "entrada" invertida y "salida"— da $O(1)$ amortizado, pero la amortización se rompe si se reutiliza una versión vieja muchas veces; para $O(1)$ en el peor caso hay que recurrir a *real-time queues* con inversión incremental.

#### Costos ocultos de memoria
En la versión con lista enlazada, cada elemento paga un puntero extra y una asignación dinámica, con mala localidad de caché. El array circular es lo contrario: acceso contiguo y predecible, pero capacidad fija. Para colas de alto throughput casi siempre gana el ring buffer.

## 6. Referencias y recursos
- [[COR2011]] - Chapter 10.1 Stacks and queues; Chapter 22.2 Breadth-first search
- [[GOO2005]] - Chapter 5 Stacks, Queues, and Deques
- [[LAF2002]] - Chapter 4 Stacks and Queues
- Michael, M. M., & Scott, M. L. (1996). *Simple, Fast, and Practical Non-Blocking and Blocking Concurrent Queue Algorithms*. Recuperado de: [PODC '96](https://dl.acm.org/doi/10.1145/248052.248106)
- Okasaki, C. (1995). *Simple and Efficient Purely Functional Queues and Deques*. Journal of Functional Programming. Recuperado de: [JFP](https://www.cambridge.org/core/journals/journal-of-functional-programming/article/simple-and-efficient-purely-functional-queues-and-deques/7B3036772616B39E87BF7FBD119015AB)
- Visualización interactiva de cola y cola circular: [VisuAlgo - Linked List / Queue](https://visualgo.net/en/list)
- Python Software Foundation. *queue - A synchronized queue class*. Recuperado de: [Python docs](https://docs.python.org/3/library/queue.html)
