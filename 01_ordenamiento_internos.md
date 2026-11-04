# Unidad 5 — Algoritmos de ordenamiento internos

**Materia:** Estructura de Datos (AED-1026)
**Unidad:** 5. Métodos de ordenamiento
**Tema:** 5.1 Algoritmos de ordenamiento internos — Burbuja, Quicksort, ShellSort, Radix
**Lenguaje:** Python 3 — todo el código y las salidas de este documento se ejecutaron de verdad antes de publicarse

## Por qué "internos"

Un algoritmo de ordenamiento **interno** asume que toda la lista cabe en memoria principal (RAM) al mismo tiempo — puedes acceder a cualquier posición con `lista[i]` sin costo extra. Esta es la situación normal cuando ordenas una lista, un arreglo o cualquier colección que ya tienes cargada en tu programa.

Más adelante, en `02_ordenamiento_externos.md`, vas a ver qué cambia cuando los datos **no caben en memoria** y hay que ordenarlos usando archivos en disco.

---

## 5.1.1 Burbuja (Bubble Sort)

**Idea:** recorrer la lista comparando cada par de elementos adyacentes; si están en el orden incorrecto, intercambiarlos. Repetir el recorrido hasta que ya no haya intercambios — en cada pasada, el elemento más grande "burbujea" hasta su posición final.

```python
def burbuja(lista):
    lista = lista.copy()  # no mutar la lista original del llamador
    n = len(lista)
    comparaciones = 0
    intercambios = 0
    for i in range(n - 1):  # una pasada completa por cada elemento que falta "asentar"
        huboIntercambio = False
        # tras cada pasada, el último "n-1-i" ya quedó en su posición final -> no hace falta revisarlo de nuevo
        for j in range(n - 1 - i):
            comparaciones += 1
            if lista[j] > lista[j + 1]:  # están en el orden incorrecto -> intercambiar
                lista[j], lista[j + 1] = lista[j + 1], lista[j]
                intercambios += 1
                huboIntercambio = True
        if not huboIntercambio:
            break  # ya está ordenada, no hace falta seguir
    return lista, comparaciones, intercambios

datos = [64, 25, 12, 22, 11, 90, 5]
resultado, comp, inter = burbuja(datos)
print("original:", datos)
print("ordenado:", resultado)
print("comparaciones:", comp, "intercambios:", inter)
```

**Salida real:**

```
original: [64, 25, 12, 22, 11, 90, 5]
ordenado: [5, 11, 12, 22, 25, 64, 90]
comparaciones: 21 intercambios: 15
```

El corte anticipado (`if not huboIntercambio: break`) es lo que hace que Burbuja sea **adaptativo**: si la lista ya viene ordenada, termina en una sola pasada.

```python
datos_ordenados = [1,2,3,4,5,6,7]
_, comp2, inter2 = burbuja(datos_ordenados)
print("ya ordenado -> comparaciones:", comp2, "intercambios:", inter2)
```

**Salida real:**

```
ya ordenado -> comparaciones: 6 intercambios: 0
```

**Complejidad:** O(n²) en el peor caso y el promedio (lista en orden inverso); O(n) en el mejor caso (ya ordenada, gracias al corte anticipado). **Espacio:** O(1) extra — ordena *in place*.

**Cuándo se usa en la práctica:** casi nunca en producción (es de los más lentos), pero es el punto de partida obvio para enseñar la idea de "comparar e intercambiar", y es útil cuando la lista es muy pequeña o ya está casi ordenada.

---

## 5.1.2 Quicksort

**Idea:** elegir un **pivote**, reacomodar la lista para que todo lo menor o igual al pivote quede a su izquierda y todo lo mayor quede a su derecha (esto se llama **partición**), y repetir el proceso recursivamente en cada mitad. Cuando una sublista tiene 0 o 1 elementos, ya está ordenada.

```python
def quicksort(lista, inicio=0, fin=None, profundidad=0):
    if fin is None:
        fin = len(lista) - 1  # primera llamada: cubrir toda la lista
    if inicio < fin:  # segmento de 0 o 1 elementos -> ya está ordenado, caso base
        p = particionar(lista, inicio, fin)  # p = posición final del pivote
        print("  " * profundidad + f"pivote={lista[p]} -> izq={lista[inicio:p]} der={lista[p+1:fin+1]}")
        quicksort(lista, inicio, p - 1, profundidad + 1)  # ordenar la mitad izquierda (menores)
        quicksort(lista, p + 1, fin, profundidad + 1)     # ordenar la mitad derecha (mayores)
    return lista

def particionar(lista, inicio, fin):
    pivote = lista[fin]  # esquema de Lomuto: el pivote es siempre el último elemento
    i = inicio - 1  # i marca el final de la zona "ya confirmada <= pivote"
    for j in range(inicio, fin):
        if lista[j] <= pivote:
            i += 1
            lista[i], lista[j] = lista[j], lista[i]  # mete lista[j] dentro de la zona de menores
    lista[i + 1], lista[fin] = lista[fin], lista[i + 1]  # coloca el pivote justo después de los menores
    return i + 1  # posición final del pivote: a su izquierda todo <= él, a su derecha todo >= él

datos = [64, 25, 12, 22, 11, 90, 5]
print("original:", datos)
resultado = quicksort(datos.copy())
print("ordenado:", resultado)
```

**Salida real:**

```
original: [64, 25, 12, 22, 11, 90, 5]
pivote=5 -> izq=[] der=[25, 12, 22, 11, 90, 64]
  pivote=64 -> izq=[25, 12, 22, 11] der=[90]
    pivote=11 -> izq=[] der=[12, 22, 25]
      pivote=25 -> izq=[12, 22] der=[]
        pivote=22 -> izq=[12] der=[]
ordenado: [5, 11, 12, 22, 25, 64, 90]
```

Nota algo importante en esta corrida real: el primer pivote elegido (5, el último elemento) resultó ser el valor **más pequeño** de toda la lista — por eso la partición quedó muy desbalanceada (0 elementos a la izquierda, 6 a la derecha). Esto no es un error: es el caso que explica por qué la elección del pivote importa tanto.

**Complejidad:** O(n log n) en el promedio y el mejor caso (particiones balanceadas). **O(n²) en el peor caso** — ocurre precisamente cuando el pivote elegido siempre es el menor o el mayor del segmento, como estuvo a punto de pasar aquí (en listas ya ordenadas, elegir siempre el último elemento como pivote produce exactamente este peor caso). **Espacio:** O(log n) por la pila de llamadas recursivas (en el caso balanceado).

**Cuándo se usa en la práctica:** es el que usan por default muchos lenguajes para ordenar arreglos genéricos (con variantes como *introsort*, que cambia a heapsort si detecta que la recursión se está desbalanceando demasiado) — rápido en el caso típico, y el riesgo de peor caso se mitiga eligiendo el pivote de forma aleatoria o como la mediana de tres valores.

---

## 5.1.3 ShellSort

**Idea:** una generalización de la inserción directa. En vez de comparar solo elementos adyacentes, compara elementos separados por un **salto (gap)** grande, y va reduciendo ese salto en cada pasada hasta llegar a 1 (que es una inserción normal, pero sobre una lista ya "casi ordenada" por los saltos anteriores).

```python
def shellsort(lista):
    lista = lista.copy()
    n = len(lista)
    gap = n // 2  # salto inicial: la mitad del tamaño de la lista
    while gap > 0:
        print(f"gap={gap}")
        # inserción directa, pero comparando elementos separados "gap" posiciones en vez de 1
        for i in range(gap, n):
            temp = lista[i]  # elemento que se va a insertar en su sub-lista (la de su mismo "carril" de gap)
            j = i
            while j >= gap and lista[j - gap] > temp:
                lista[j] = lista[j - gap]  # recorre hacia atrás, "gap" en "gap", para abrir el hueco
                j -= gap
            lista[j] = temp  # inserta temp en el hueco que quedó libre
        print("  estado:", lista)
        gap //= 2  # reduce el salto para la siguiente pasada, hasta llegar a gap=1
    return lista

datos = [64, 25, 12, 22, 11, 90, 5, 33]
print("original:", datos)
resultado = shellsort(datos)
print("ordenado:", resultado)
```

**Salida real:**

```
original: [64, 25, 12, 22, 11, 90, 5, 33]
gap=4
  estado: [11, 25, 5, 22, 64, 90, 12, 33]
gap=2
  estado: [5, 22, 11, 25, 12, 33, 64, 90]
gap=1
  estado: [5, 11, 12, 22, 25, 33, 64, 90]
```

Con `gap=4`, solo se comparan pares separados por 4 posiciones (posición 0 con 4, 1 con 5, etc.) — por eso el estado tras esa pasada todavía no está ordenado, pero ya movió los valores grandes mucho más cerca de su posición final de lo que lo haría una sola comparación adyacente. Con `gap=1` se hace una inserción directa normal, pero sobre una lista que ya está "casi lista", así que trabaja mucho menos que si hubiera empezado directo con `gap=1`.

**Complejidad:** depende de la secuencia de gaps elegida; con la secuencia simple `n/2, n/4, ..., 1` (la que se usó aquí) es O(n²) en el peor caso, pero en la práctica se comporta notablemente mejor que inserción directa. Con secuencias de gap más sofisticadas (como la de Hibbard o Sedgewick) se puede llegar a O(n^1.3) aproximadamente. **Espacio:** O(1) extra.

**Cuándo se usa en la práctica:** para listas medianas donde Quicksort sería excesivo de implementar (no necesita recursión ni memoria extra) y Burbuja/Inserción directa serían demasiado lentos — es una opción intermedia simple de programar.

---

## 5.1.4 Radix Sort

**Idea:** a diferencia de los tres anteriores, Radix **no compara elementos entre sí**. Ordena números procesando un dígito a la vez (de menos significativo a más significativo), usando en cada pasada un *counting sort* estable por ese dígito.

```python
def radix_sort(lista):
    lista = lista.copy()
    max_val = max(lista)
    exp = 1  # exp = 1, 10, 100... selecciona qué dígito mirar
    while max_val // exp > 0:
        lista = counting_sort_por_digito(lista, exp)
        print(f"exp={exp} (dígito en posición {len(str(exp))}) -> {lista}")
        exp *= 10
    return lista

def counting_sort_por_digito(lista, exp):
    n = len(lista)
    salida = [0] * n
    conteo = [0] * 10  # un contador por cada dígito posible (0-9)
    for num in lista:
        digito = (num // exp) % 10  # extrae el dígito que toca en esta pasada
        conteo[digito] += 1
    for i in range(1, 10):
        conteo[i] += conteo[i - 1]  # conteo acumulado -> cuántos elementos van antes o igual a cada dígito
    for i in range(n - 1, -1, -1):  # recorrido de atrás hacia adelante: así se mantiene estable
        digito = (lista[i] // exp) % 10
        salida[conteo[digito] - 1] = lista[i]  # coloca el elemento en su posición final para este dígito
        conteo[digito] -= 1  # libera esa posición para el siguiente elemento con el mismo dígito
    return salida

datos = [170, 45, 75, 90, 802, 24, 2, 66]
print("original:", datos)
resultado = radix_sort(datos)
print("ordenado:", resultado)
```

**Salida real:**

```
original: [170, 45, 75, 90, 802, 24, 2, 66]
exp=1 (dígito en posición 1) -> [170, 90, 802, 2, 24, 45, 75, 66]
exp=10 (dígito en posición 2) -> [802, 2, 24, 45, 66, 170, 75, 90]
exp=100 (dígito en posición 3) -> [2, 24, 45, 66, 75, 90, 170, 802]
ordenado: [2, 24, 45, 66, 75, 90, 170, 802]
```

Observa la primera pasada (`exp=1`, el dígito de las unidades): `170` y `90` terminan primero porque su último dígito es `0`; `802` y `2` van después porque su último dígito es `2`. Después de las tres pasadas (unidades, decenas, centenas) la lista queda completamente ordenada — nunca se comparó un número contra otro directamente, solo se contaron dígitos.

**Por qué funciona (el detalle que suele confundir):** cada pasada de *counting sort* es **estable** (nunca cambia el orden relativo de dos elementos con el mismo dígito) — eso es obligatorio, porque la pasada de las decenas depende de que la pasada anterior de las unidades ya haya dejado un orden parcial correcto que no se debe romper.

**Complejidad:** O(d·(n+k)), donde `d` es el número de dígitos del valor más grande y `k` el rango de cada dígito (10, en base decimal). Si `d` es constante (como con enteros de tamaño fijo), esto es efectivamente **O(n)** — más rápido que cualquier algoritmo basado en comparaciones, que tiene un límite teórico de O(n log n). **Espacio:** O(n+k) por el arreglo de salida y el conteo.

**Cuándo se usa en la práctica:** cuando se ordenan enteros (o cadenas de longitud fija) y se puede garantizar que el número de dígitos es pequeño frente a `n` — por ejemplo, ordenar millones de números telefónicos o IDs.

---

## Comparación de los cuatro algoritmos

| Algoritmo | Peor caso | Promedio | Mejor caso | Espacio extra | Estable | Compara elementos |
|---|---|---|---|---|---|---|
| Burbuja | O(n²) | O(n²) | O(n) | O(1) | Sí | Sí |
| Quicksort | O(n²) | O(n log n) | O(n log n) | O(log n) | No | Sí |
| ShellSort | O(n²)* | ~O(n^1.3)* | O(n log n) | O(1) | No | Sí |
| Radix | O(d·(n+k)) | O(d·(n+k)) | O(d·(n+k)) | O(n+k) | Sí | No |

*Depende de la secuencia de gaps elegida.

La elección correcta no es "¿cuál es el mejor algoritmo?" sino "¿qué forma tienen mis datos?": Quicksort gana cuando hay suficiente memoria para la recursión y el pivote se puede elegir bien; Radix gana cuando se ordenan enteros con pocos dígitos; ShellSort es una alternativa simple cuando no quieres implementar recursión; Burbuja casi nunca gana, pero demuestra la idea base de la que parten los demás.

---

## Implementaciones reales en tecnología actual

Estos mismos algoritmos (o variantes directas de ellos) están corriendo hoy dentro de herramientas que usas todos los días:

- **Timsort (Python `sorted()`/`list.sort()`, Java `Arrays.sort()` para objetos)** — un híbrido entre Mezcla (la que viste en `02_ordenamiento_externos.md`) e Inserción directa: detecta tramos ya ordenados en los datos de entrada (igual que la **mezcla natural**) y los mezcla, pero usa inserción directa para ordenar tramos cortos antes de mezclarlos. Fue diseñado específicamente para datos reales, que casi nunca están en orden totalmente aleatorio.
- **Introsort (C++ `std::sort`, .NET `Array.Sort`)** — empieza como **Quicksort**, pero si detecta que la recursión se está desbalanceando demasiado (el caso O(n²) que viste en la sección 5.1.2), cambia a Heapsort para garantizar que nunca se dispare el tiempo de ejecución. Combina la velocidad típica de Quicksort con una cota de seguridad que Quicksort puro no tiene.
- **`qsort` de la librería estándar de C** — es, literalmente, una implementación de **Quicksort** (con variantes de partición según la implementación de cada sistema) expuesta como función de propósito general desde los años 70.
- **GPU radix sort (motores gráficos, CUDA, renderizado de partículas)** — las tarjetas gráficas ordenan millones de elementos (partículas, triángulos por profundidad) usando **Radix Sort** en paralelo: cada núcleo de la GPU procesa un dígito/bit a la vez sobre miles de elementos simultáneamente, aprovechando que Radix no necesita comparar elementos entre sí.
- **ShellSort en sistemas embebidos** — algunas implementaciones ligeras de la librería estándar de C para sistemas con muy poca memoria (como uClibc, usada en routers y dispositivos IoT) usan ShellSort para `qsort()` en vez de Quicksort, porque no necesita memoria extra para recursión.
- **Índices de bases de datos (PostgreSQL, MySQL)** — al construir un índice `B-Tree` desde cero sobre una tabla existente, el motor ordena todas las filas por la columna indexada antes de construir el árbol; para tablas que caben en memoria usa variantes internas (tipo Quicksort/Timsort), y para tablas grandes recurre a ordenamiento externo (ver `02_ordenamiento_externos.md`).

## Fuentes

- Programa sinóptico oficial AED-1026 — TecNM (5.1 Algoritmos de ordenamiento internos)
- Cormen, T., Leiserson, C., Rivest, R., Stein, C. — *Introduction to Algorithms* (capítulos de ordenamiento por comparación y counting/radix sort)
