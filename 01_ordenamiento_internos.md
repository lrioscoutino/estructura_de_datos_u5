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

### Visualización paso a paso en texto

El código anterior solo imprime el resultado final. Para explicar cada comparación en clase ayuda ver el arreglo completo en cada paso, con marcas (`^^`) debajo de los dos elementos que se están comparando:

```python
def mostrar(lista, marcar=None, etiqueta=""):
    fila = " ".join(f"{v:>3}" for v in lista)
    print(f"[{fila}]  {etiqueta}")
    if marcar:  # imprime una segunda línea con ^^ debajo de las posiciones marcadas
        marcas = "".join(" ^^ " if i in marcar else "    " for i in range(len(lista)))
        print(f" {marcas}")

def burbuja_visual(lista):
    lista = lista.copy()
    n = len(lista)
    mostrar(lista, etiqueta="(inicio)")
    for i in range(n - 1):
        huboIntercambio = False
        for j in range(n - 1 - i):
            if lista[j] > lista[j + 1]:
                mostrar(lista, marcar={j, j+1}, etiqueta=f"{lista[j]} > {lista[j+1]} -> se intercambian")
                lista[j], lista[j + 1] = lista[j + 1], lista[j]
                huboIntercambio = True
            else:
                mostrar(lista, marcar={j, j+1}, etiqueta=f"{lista[j]} <= {lista[j+1]} -> ya están en orden, no se mueven")
        mostrar(lista, etiqueta=f"fin de la pasada {i+1}: lista[{n-1-i}] ya quedó en su lugar final")
        if not huboIntercambio:
            break
    return lista

burbuja_visual([64, 25, 12, 22, 11, 90, 5])
```

**Salida real:**

```
[ 64  25  12  22  11  90   5]  (inicio)
[ 64  25  12  22  11  90   5]  64 > 25 -> se intercambian
  ^^  ^^
[ 25  64  12  22  11  90   5]  64 > 12 -> se intercambian
      ^^  ^^
[ 25  12  64  22  11  90   5]  64 > 22 -> se intercambian
          ^^  ^^
[ 25  12  22  64  11  90   5]  64 > 11 -> se intercambian
              ^^  ^^
[ 25  12  22  11  64  90   5]  64 <= 90 -> ya están en orden, no se mueven
                  ^^  ^^
[ 25  12  22  11  64  90   5]  90 > 5 -> se intercambian
                      ^^  ^^
[ 25  12  22  11  64   5  90]  fin de la pasada 1: lista[6] ya quedó en su lugar final
[ 25  12  22  11  64   5  90]  25 > 12 -> se intercambian
  ^^  ^^
[ 12  25  22  11  64   5  90]  25 > 22 -> se intercambian
      ^^  ^^
[ 12  22  25  11  64   5  90]  25 > 11 -> se intercambian
          ^^  ^^
[ 12  22  11  25  64   5  90]  25 <= 64 -> ya están en orden, no se mueven
              ^^  ^^
[ 12  22  11  25  64   5  90]  64 > 5 -> se intercambian
                  ^^  ^^
[ 12  22  11  25   5  64  90]  fin de la pasada 2: lista[5] ya quedó en su lugar final
[ 12  22  11  25   5  64  90]  12 <= 22 -> ya están en orden, no se mueven
  ^^  ^^
[ 12  22  11  25   5  64  90]  22 > 11 -> se intercambian
      ^^  ^^
[ 12  11  22  25   5  64  90]  22 <= 25 -> ya están en orden, no se mueven
          ^^  ^^
[ 12  11  22  25   5  64  90]  25 > 5 -> se intercambian
              ^^  ^^
[ 12  11  22   5  25  64  90]  fin de la pasada 3: lista[4] ya quedó en su lugar final
[ 12  11  22   5  25  64  90]  12 > 11 -> se intercambian
  ^^  ^^
[ 11  12  22   5  25  64  90]  12 <= 22 -> ya están en orden, no se mueven
      ^^  ^^
[ 11  12  22   5  25  64  90]  22 > 5 -> se intercambian
          ^^  ^^
[ 11  12   5  22  25  64  90]  fin de la pasada 4: lista[3] ya quedó en su lugar final
[ 11  12   5  22  25  64  90]  11 <= 12 -> ya están en orden, no se mueven
  ^^  ^^
[ 11  12   5  22  25  64  90]  12 > 5 -> se intercambian
      ^^  ^^
[ 11   5  12  22  25  64  90]  fin de la pasada 5: lista[2] ya quedó en su lugar final
[ 11   5  12  22  25  64  90]  11 > 5 -> se intercambian
  ^^  ^^
[  5  11  12  22  25  64  90]  fin de la pasada 6: lista[1] ya quedó en su lugar final
```

Esta misma idea — imprimir el arreglo y marcar con `^^` las posiciones activas — es la que reutilizan las versiones visuales de Quicksort, ShellSort y Radix más abajo.

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

### Visualización paso a paso en texto

Mismo truco que en Burbuja: imprimir el segmento activo completo, marcando el pivote (`PIV`) y la posición que se está comparando (`^^`) en cada paso de la partición:

```python
def mostrar(lista, marcar=None, pivote=None, etiqueta=""):
    fila = " ".join(f"{v:>3}" for v in lista)
    print(f"[{fila}]  {etiqueta}")
    if marcar or pivote is not None:
        marcas = []
        for i in range(len(lista)):
            if i == pivote:
                marcas.append(" PIV")
            elif marcar and i in marcar:
                marcas.append(" ^^ ")
            else:
                marcas.append("    ")
        print(" " + "".join(marcas))

def particionar_visual(lista, inicio, fin):
    pivote = lista[fin]
    print(f"\n-- partición de [{inicio}..{fin}] = {lista[inicio:fin+1]}, pivote = {pivote} --")
    i = inicio - 1
    for j in range(inicio, fin):
        cumple = lista[j] <= pivote
        mostrar(lista, marcar={j}, pivote=fin,
                etiqueta=f"j={j}: {lista[j]} <= {pivote}? {'Sí' if cumple else 'No'}")
        if cumple:
            i += 1
            lista[i], lista[j] = lista[j], lista[i]
            if i != j:
                mostrar(lista, marcar={i, j}, pivote=fin, etiqueta=f"  entra a la zona de menores: intercambia i={i} con j={j}")
    lista[i + 1], lista[fin] = lista[fin], lista[i + 1]
    mostrar(lista, marcar={i+1}, etiqueta=f"pivote {pivote} va a su posición final: {i+1}")
    return i + 1

def quicksort_visual(lista, inicio=0, fin=None):
    if fin is None:
        fin = len(lista) - 1
    if inicio < fin:
        p = particionar_visual(lista, inicio, fin)
        quicksort_visual(lista, inicio, p - 1)
        quicksort_visual(lista, p + 1, fin)
    return lista

datos = [64, 25, 12, 22, 11, 90, 5]
mostrar(datos, etiqueta="(inicio)")
quicksort_visual(datos)
print("\nordenado:", datos)
```

**Salida real:**

```
[ 64  25  12  22  11  90   5]  (inicio)

-- partición de [0..6] = [64, 25, 12, 22, 11, 90, 5], pivote = 5 --
[ 64  25  12  22  11  90   5]  j=0: 64 <= 5? No
  ^^                      PIV
[ 64  25  12  22  11  90   5]  j=1: 25 <= 5? No
      ^^                  PIV
[ 64  25  12  22  11  90   5]  j=2: 12 <= 5? No
          ^^              PIV
[ 64  25  12  22  11  90   5]  j=3: 22 <= 5? No
              ^^          PIV
[ 64  25  12  22  11  90   5]  j=4: 11 <= 5? No
                  ^^      PIV
[ 64  25  12  22  11  90   5]  j=5: 90 <= 5? No
                      ^^  PIV
[  5  25  12  22  11  90  64]  pivote 5 va a su posición final: 0
  ^^

-- partición de [1..6] = [25, 12, 22, 11, 90, 64], pivote = 64 --
[  5  25  12  22  11  90  64]  j=1: 25 <= 64? Sí
      ^^                  PIV
[  5  25  12  22  11  90  64]  j=2: 12 <= 64? Sí
          ^^              PIV
[  5  25  12  22  11  90  64]  j=3: 22 <= 64? Sí
              ^^          PIV
[  5  25  12  22  11  90  64]  j=4: 11 <= 64? Sí
                  ^^      PIV
[  5  25  12  22  11  90  64]  j=5: 90 <= 64? No
                      ^^  PIV
[  5  25  12  22  11  64  90]  pivote 64 va a su posición final: 5
                      ^^

-- partición de [1..4] = [25, 12, 22, 11], pivote = 11 --
[  5  25  12  22  11  64  90]  j=1: 25 <= 11? No
      ^^          PIV
[  5  25  12  22  11  64  90]  j=2: 12 <= 11? No
          ^^      PIV
[  5  25  12  22  11  64  90]  j=3: 22 <= 11? No
              ^^  PIV
[  5  11  12  22  25  64  90]  pivote 11 va a su posición final: 1
      ^^

-- partición de [2..4] = [12, 22, 25], pivote = 25 --
[  5  11  12  22  25  64  90]  j=2: 12 <= 25? Sí
          ^^      PIV
[  5  11  12  22  25  64  90]  j=3: 22 <= 25? Sí
              ^^  PIV
[  5  11  12  22  25  64  90]  pivote 25 va a su posición final: 4
                  ^^

-- partición de [2..3] = [12, 22], pivote = 22 --
[  5  11  12  22  25  64  90]  j=2: 12 <= 22? Sí
          ^^  PIV
[  5  11  12  22  25  64  90]  pivote 22 va a su posición final: 3
              ^^

ordenado: [5, 11, 12, 22, 25, 64, 90]
```

Esta trazabilidad es útil en clase para mostrar por qué el primer pivote (5, el valor más pequeño de la lista) produce una partición muy desbalanceada (0 elementos a la izquierda, 6 a la derecha) — es la misma observación de la sección anterior, pero ahora se ve "elemento por elemento" en vez de solo en el resumen final.

### Tres formas de implementar la partición

El código de arriba usa el esquema de **Lomuto**, pero no es la única forma de partir la lista. Existen (al menos) tres enfoques distintos y vale la pena verlos juntos porque cada uno expone una idea diferente de cómo funciona la partición.

**1. Con listas auxiliares** — la forma más fácil de leer, pero la que normalmente no se usa en código de producción:

```python
def quicksort_auxiliar(lista):
    # Caso base: 0 o 1 elementos ya está ordenado
    if len(lista) <= 1:
        return lista

    pivote = lista[-1]  # mismo criterio que Lomuto: el último elemento
    resto = lista[:-1]

    # Dos listas NUEVAS en vez de reordenar el mismo arreglo
    menores = [x for x in resto if x <= pivote]
    mayores = [x for x in resto if x > pivote]

    # Se ordena cada lista por separado (recursión) y se unen con el pivote en medio
    return quicksort_auxiliar(menores) + [pivote] + quicksort_auxiliar(mayores)

datos = [64, 25, 12, 22, 11, 90, 5]
print("auxiliar:", quicksort_auxiliar(datos.copy()))
```

**Salida real:**

```
auxiliar: [5, 11, 12, 22, 25, 64, 90]
```

No hay índices que cuidar ni intercambios que razonar: se lee casi como la definición matemática de Quicksort ("menores, pivote, mayores"). El costo es la memoria: cada llamada crea listas nuevas, O(n) de memoria extra por nivel de recursión, en vez de O(log n) que logran Lomuto y Hoare trabajando directamente sobre el mismo arreglo ("in place").

**2. Partición de Hoare** — la original, propuesta por Tony Hoare en 1960. Dos índices arrancan en los extremos del segmento y se acercan uno al otro: `izq` avanza mientras encuentra valores menores que el pivote, `der` retrocede mientras encuentra valores mayores. Cuando ambos se detienen sin haberse cruzado, se intercambian esos dos valores y se repite; cuando se cruzan, la partición terminó:

```python
def particion_hoare(lista, lo, hi):
    pivote = lista[lo]  # aquí el pivote se toma del extremo IZQUIERDO
    izq = lo - 1
    der = hi + 1

    while True:
        izq += 1
        while lista[izq] < pivote:  # avanza mientras encuentre menores
            izq += 1

        der -= 1
        while lista[der] > pivote:  # retrocede mientras encuentre mayores
            der -= 1

        if izq >= der:  # los índices se cruzaron: la partición terminó
            return der  # der es el punto de corte, no necesariamente la posición final del pivote

        lista[izq], lista[der] = lista[der], lista[izq]

def quicksort_hoare(lista, lo=0, hi=None):
    if hi is None:
        hi = len(lista) - 1
    if lo < hi:
        p = particion_hoare(lista, lo, hi)
        quicksort_hoare(lista, lo, p)
        quicksort_hoare(lista, p + 1, hi)
    return lista

d = datos.copy()
quicksort_hoare(d)
print("hoare:", d)
```

**Salida real:**

```
hoare: [5, 11, 12, 22, 25, 64, 90]
```

Diferencia clave frente a Lomuto: en Hoare, al terminar una partición el pivote **no** queda garantizado en su posición final — el valor que devuelve la función (`der`) es solo el punto donde se corta la lista en dos sublistas a seguir ordenando. A cambio, Hoare hace en promedio **menos intercambios** que Lomuto y se comporta mejor cuando hay muchos valores repetidos (Lomuto puede degradar a O(n²) con más facilidad en esos casos).

**3. Partición de Lomuto** — la que ya se mostró arriba: un solo índice `i` (la "frontera de menores") recorre el segmento una sola vez de izquierda a derecha. Es la más fácil de implementar sin errores y la más común en libros de texto, aunque hace más intercambios en promedio que Hoare.

| | Listas auxiliares | Hoare | Lomuto |
|---|---|---|---|
| Facilidad de lectura | Muy alta | Media-baja | Alta |
| Memoria extra | O(n) por nivel | O(1) (in place) | O(1) (in place) |
| Intercambios promedio | — (no aplica, no muta) | Menos | Más |
| Pivote en posición final al terminar la partición | No aplica | No | Sí |
| Con muchos valores repetidos | Normal | Buen comportamiento | Puede degradar a O(n²) |
| Uso típico | Enseñanza / prototipos | Implementaciones de alto rendimiento | Libros de texto, la más enseñada |

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

### Visualización paso a paso en texto

Aquí lo importante de marcar no es solo "quién se compara", sino el **gap**: qué tan separados están los dos elementos que se comparan en cada pasada.

```python
def mostrar(lista, marcar=None, etiqueta=""):
    fila = " ".join(f"{v:>3}" for v in lista)
    print(f"[{fila}]  {etiqueta}")
    if marcar:
        marcas = "".join(" ^^ " if i in marcar else "    " for i in range(len(lista)))
        print(" " + marcas)

def shellsort_visual(lista):
    lista = lista.copy()
    n = len(lista)
    gap = n // 2
    while gap > 0:
        print(f"\n-- gap = {gap} --")
        for i in range(gap, n):
            temp = lista[i]
            j = i
            mostrar(lista, marcar={i}, etiqueta=f"elemento a insertar: lista[{i}]={temp}")
            while j >= gap and lista[j - gap] > temp:
                mostrar(lista, marcar={j-gap, j}, etiqueta=f"{lista[j-gap]} (pos {j-gap}) > {temp}? Sí -> recorre y abre hueco")
                lista[j] = lista[j - gap]
                j -= gap
            if j != i:
                lista[j] = temp
                mostrar(lista, marcar={j}, etiqueta=f"{temp} se inserta en la posición {j}")
            else:
                mostrar(lista, marcar={j}, etiqueta="ya estaba en su lugar dentro de este gap, no se mueve")
        mostrar(lista, etiqueta=f"fin de la pasada con gap={gap}")
        gap //= 2
    return lista

datos = [64, 25, 12, 22, 11, 90, 5, 33]
mostrar(datos, etiqueta="(inicio)")
resultado = shellsort_visual(datos)
print("\nordenado:", resultado)
```

**Salida real:**

```
[ 64  25  12  22  11  90   5  33]  (inicio)

-- gap = 4 --
[ 64  25  12  22  11  90   5  33]  elemento a insertar: lista[4]=11
                  ^^
[ 64  25  12  22  11  90   5  33]  64 (pos 0) > 11? Sí -> recorre y abre hueco
  ^^              ^^
[ 11  25  12  22  64  90   5  33]  11 se inserta en la posición 0
  ^^
[ 11  25  12  22  64  90   5  33]  elemento a insertar: lista[5]=90
                      ^^
[ 11  25  12  22  64  90   5  33]  ya estaba en su lugar dentro de este gap, no se mueve
                      ^^
[ 11  25  12  22  64  90   5  33]  elemento a insertar: lista[6]=5
                          ^^
[ 11  25  12  22  64  90   5  33]  12 (pos 2) > 5? Sí -> recorre y abre hueco
          ^^              ^^
[ 11  25   5  22  64  90  12  33]  5 se inserta en la posición 2
          ^^
[ 11  25   5  22  64  90  12  33]  elemento a insertar: lista[7]=33
                              ^^
[ 11  25   5  22  64  90  12  33]  ya estaba en su lugar dentro de este gap, no se mueve
                              ^^
[ 11  25   5  22  64  90  12  33]  fin de la pasada con gap=4

-- gap = 2 --
[ 11  25   5  22  64  90  12  33]  elemento a insertar: lista[2]=5
          ^^
[ 11  25   5  22  64  90  12  33]  11 (pos 0) > 5? Sí -> recorre y abre hueco
  ^^      ^^
[  5  25  11  22  64  90  12  33]  5 se inserta en la posición 0
  ^^
[  5  25  11  22  64  90  12  33]  elemento a insertar: lista[3]=22
              ^^
[  5  25  11  22  64  90  12  33]  25 (pos 1) > 22? Sí -> recorre y abre hueco
      ^^      ^^
[  5  22  11  25  64  90  12  33]  22 se inserta en la posición 1
      ^^
[  5  22  11  25  64  90  12  33]  elemento a insertar: lista[4]=64
                  ^^
[  5  22  11  25  64  90  12  33]  ya estaba en su lugar dentro de este gap, no se mueve
                  ^^
[  5  22  11  25  64  90  12  33]  elemento a insertar: lista[5]=90
                      ^^
[  5  22  11  25  64  90  12  33]  ya estaba en su lugar dentro de este gap, no se mueve
                      ^^
[  5  22  11  25  64  90  12  33]  elemento a insertar: lista[6]=12
                          ^^
[  5  22  11  25  64  90  12  33]  64 (pos 4) > 12? Sí -> recorre y abre hueco
                  ^^      ^^
[  5  22  11  25  12  90  64  33]  12 se inserta en la posición 4
                  ^^
[  5  22  11  25  12  90  64  33]  elemento a insertar: lista[7]=33
                              ^^
[  5  22  11  25  12  90  64  33]  90 (pos 5) > 33? Sí -> recorre y abre hueco
                      ^^      ^^
[  5  22  11  25  12  33  64  90]  33 se inserta en la posición 5
                      ^^
[  5  22  11  25  12  33  64  90]  fin de la pasada con gap=2

-- gap = 1 --
[  5  22  11  25  12  33  64  90]  elemento a insertar: lista[1]=22
      ^^
[  5  22  11  25  12  33  64  90]  ya estaba en su lugar dentro de este gap, no se mueve
      ^^
[  5  22  11  25  12  33  64  90]  elemento a insertar: lista[2]=11
          ^^
[  5  22  11  25  12  33  64  90]  22 (pos 1) > 11? Sí -> recorre y abre hueco
      ^^  ^^
[  5  11  22  25  12  33  64  90]  11 se inserta en la posición 1
      ^^
[  5  11  22  25  12  33  64  90]  elemento a insertar: lista[3]=25
              ^^
[  5  11  22  25  12  33  64  90]  ya estaba en su lugar dentro de este gap, no se mueve
              ^^
[  5  11  22  25  12  33  64  90]  elemento a insertar: lista[4]=12
                  ^^
[  5  11  22  25  12  33  64  90]  25 (pos 3) > 12? Sí -> recorre y abre hueco
              ^^  ^^
[  5  11  22  25  25  33  64  90]  22 (pos 2) > 12? Sí -> recorre y abre hueco
          ^^  ^^
[  5  11  12  22  25  33  64  90]  12 se inserta en la posición 2
          ^^
[  5  11  12  22  25  33  64  90]  elemento a insertar: lista[5]=33
                      ^^
[  5  11  12  22  25  33  64  90]  ya estaba en su lugar dentro de este gap, no se mueve
                      ^^
[  5  11  12  22  25  33  64  90]  elemento a insertar: lista[6]=64
                          ^^
[  5  11  12  22  25  33  64  90]  ya estaba en su lugar dentro de este gap, no se mueve
                          ^^
[  5  11  12  22  25  33  64  90]  elemento a insertar: lista[7]=90
                              ^^
[  5  11  12  22  25  33  64  90]  ya estaba en su lugar dentro de este gap, no se mueve
                              ^^
[  5  11  12  22  25  33  64  90]  fin de la pasada con gap=1

ordenado: [5, 11, 12, 22, 25, 33, 64, 90]
```

Con `gap=4` las marcas `^^` quedan muy separadas: eso es justo lo que distingue a ShellSort de la inserción directa (gap=1 sería comparar solo adyacentes desde el principio). Nota el paso intermedio raro `[5 11 22 25 25 33 64 90]` en `gap=1`: es el "hueco" que deja temporalmente el elemento que se está recorriendo hacia atrás, antes de insertarse en su posición final — por eso aparece un 25 duplicado por un instante.

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

### Visualización paso a paso en texto

Radix no compara elementos, así que marcar `^^` no aplica aquí. Lo que ayuda a visualizar es **en qué cubeta cae cada número**, porque eso es lo único que decide el orden de cada pasada:

```python
def mostrar_cubetas(cubetas, etiqueta=""):
    print(etiqueta)
    for d in range(10):
        if cubetas[d]:
            print(f"  cubeta {d}: {cubetas[d]}")

def counting_sort_por_digito_visual(lista, exp):
    cubetas = [[] for _ in range(10)]
    for num in lista:
        digito = (num // exp) % 10
        cubetas[digito].append(num)
    mostrar_cubetas(cubetas, f"reparto por el dígito con exp={exp} (dígito = (num // {exp}) % 10):")
    salida = [num for cubeta in cubetas for num in cubeta]
    print(f"  -> se juntan las cubetas en orden 0..9: {salida}\n")
    return salida

def radix_sort_visual(lista):
    lista = lista.copy()
    max_val = max(lista)
    exp = 1
    while max_val // exp > 0:
        print(f"== pasada con exp={exp} ==")
        lista = counting_sort_por_digito_visual(lista, exp)
        exp *= 10
    return lista

datos = [170, 45, 75, 90, 802, 24, 2, 66]
print("original:", datos, "\n")
resultado = radix_sort_visual(datos)
print("ordenado:", resultado)
```

**Salida real:**

```
original: [170, 45, 75, 90, 802, 24, 2, 66]

== pasada con exp=1 ==
reparto por el dígito con exp=1 (dígito = (num // 1) % 10):
  cubeta 0: [170, 90]
  cubeta 2: [802, 2]
  cubeta 4: [24]
  cubeta 5: [45, 75]
  cubeta 6: [66]
  -> se juntan las cubetas en orden 0..9: [170, 90, 802, 2, 24, 45, 75, 66]

== pasada con exp=10 ==
reparto por el dígito con exp=10 (dígito = (num // 10) % 10):
  cubeta 0: [802, 2]
  cubeta 2: [24]
  cubeta 4: [45]
  cubeta 6: [66]
  cubeta 7: [170, 75]
  cubeta 9: [90]
  -> se juntan las cubetas en orden 0..9: [802, 2, 24, 45, 66, 170, 75, 90]

== pasada con exp=100 ==
reparto por el dígito con exp=100 (dígito = (num // 100) % 10):
  cubeta 0: [2, 24, 45, 66, 75, 90]
  cubeta 1: [170]
  cubeta 8: [802]
  -> se juntan las cubetas en orden 0..9: [2, 24, 45, 66, 75, 90, 170, 802]

ordenado: [2, 24, 45, 66, 75, 90, 170, 802]
```

Ver las cubetas explícitamente aclara la duda más común de los estudiantes con Radix: por qué `170` y `75` terminan en la misma cubeta (7) en la segunda pasada — porque `170 // 10 = 17` y `75 // 10 = 7`... en realidad es `(170 // 10) % 10 = 7` y `(75 // 10) % 10 = 7`: ambos tienen un 7 en la posición de las decenas, aunque tengan distinto número de dígitos en total.

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
