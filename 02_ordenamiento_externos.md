# Unidad 5 — Algoritmos de ordenamiento externos

**Materia:** Estructura de Datos (AED-1026)
**Unidad:** 5. Métodos de ordenamiento
**Tema:** 5.2 Algoritmos de ordenamiento externos — Intercalación, Mezcla Directa, Mezcla Natural
**Lenguaje:** Python 3 — todo el código y las salidas de este documento se ejecutaron de verdad (con archivos reales en disco) antes de publicarse

## Por qué "externos"

En `01_ordenamiento_internos.md` cada algoritmo asumía que la lista completa cabe en memoria (`lista[i]` siempre accesible). Eso deja de ser cierto cuando los datos son demasiado grandes — por ejemplo, un archivo de **varios gigabytes de registros** que no cabe en la RAM disponible.

Un algoritmo **externo** resuelve esto usando disco como memoria auxiliar: lee **fragmentos pequeños** que sí caben en RAM, los ordena ahí (con cualquier algoritmo interno), los vuelve a escribir en disco ya ordenados (a esto se le llama una **"run"**), y después va **mezclando** esas runs entre sí hasta que queda un único archivo ordenado completo. La idea central que comparten los tres métodos de esta sección es la **intercalación** (merge): combinar dos secuencias ya ordenadas en una sola secuencia ordenada, sin tener que volver a compararlo todo desde cero.

---

## 5.2.1 Intercalación (Merge) — la operación base

Antes de ver los dos métodos externos, aquí está la operación que ambos reutilizan una y otra vez: mezclar dos listas **ya ordenadas** en una sola, recorriendo ambas una sola vez (nunca hay que regresar atrás).

```python
def mezclar(a, b):
    i = j = 0  # i recorre a, j recorre b — cada una solo avanza hacia adelante
    salida = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:  # el menor de los dos "frentes" siempre va primero en la salida
            salida.append(a[i]); i += 1
        else:
            salida.append(b[j]); j += 1
    salida.extend(a[i:])  # lo que sobró de a (si b se acabó primero)
    salida.extend(b[j:])  # lo que sobró de b (si a se acabó primero)
    return salida

print(mezclar([3, 12, 57, 90], [21, 45, 67, 88]))
```

**Salida real:**

```
[3, 12, 21, 45, 57, 67, 88, 90]
```

Esta mezcla cuesta O(n+m) — un solo recorrido de ambas listas — porque **ya vienen ordenadas**. Esa es la propiedad clave que explota todo el ordenamiento externo: en vez de comparar todo contra todo, solo hay que "ir avanzando por el frente" de cada secuencia ordenada.

---

## 5.2.2 Mezcla directa (Balanced Merge Sort)

**Idea:** partir el archivo en bloques de **tamaño fijo** (el tamaño que sí cabe en memoria), ordenar cada bloque por separado y guardarlo como una "run" en disco. Después, mezclar las runs de dos en dos, en pasadas sucesivas, hasta que quede una sola.

```python
def escribir_archivo(nombre, datos):
    # simula "volcar a disco" porque en ordenamiento externo real nunca cabe todo en RAM
    with open(nombre, 'w') as f:
        for d in datos:
            f.write(str(d) + '\n')

def leer_archivo(nombre):
    # simula "traer de disco a RAM" solo lo que se necesita en ese momento
    with open(nombre) as f:
        return [int(l) for l in f if l.strip()]

datos = [57, 12, 90, 3, 45, 88, 21, 67, 5, 99, 34, 72, 8, 61]
TAM_BLOQUE = 4  # tamaño máximo que "cabe en memoria" a la vez

def crear_runs(datos, tam_bloque, prefijo):
    # fase 1: partir el archivo completo en bloques de tamaño fijo, ordenar cada uno aparte y guardarlo
    runs = []
    for i in range(0, len(datos), tam_bloque):
        bloque = sorted(datos[i:i+tam_bloque])  # ordenamiento interno de un bloque chico (sí cabe en RAM)
        nombre = f"{prefijo}_{len(runs)}.txt"
        escribir_archivo(nombre, bloque)
        runs.append(nombre)
        print(f"  run creada {nombre}: {bloque}")
    return runs

def mezclar_dos_archivos(nombre_a, nombre_b, nombre_salida):
    # lee dos runs ya ordenadas, las mezcla (como mezclar() de la sección 5.2.1) y guarda el resultado
    a = leer_archivo(nombre_a)
    b = leer_archivo(nombre_b)
    i = j = 0
    salida = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            salida.append(a[i]); i += 1
        else:
            salida.append(b[j]); j += 1
    salida.extend(a[i:]); salida.extend(b[j:])
    escribir_archivo(nombre_salida, salida)
    return salida

runs = crear_runs(datos, TAM_BLOQUE, "run")
paso = 0
while len(runs) > 1:  # fase 2: mezclar runs de dos en dos, en pasadas, hasta que quede solo una
    paso += 1
    nuevas = []
    print(f"pasada {paso}:")
    for i in range(0, len(runs), 2):
        if i + 1 < len(runs):
            salida = f"merged_{paso}_{i//2}.txt"
            resultado = mezclar_dos_archivos(runs[i], runs[i+1], salida)
            print(f"  mezclar {runs[i]} + {runs[i+1]} -> {salida}: {resultado}")
            nuevas.append(salida)
        else:
            nuevas.append(runs[i])  # run impar sin pareja, pasa directo a la siguiente ronda
    runs = nuevas  # las runs mezcladas de esta pasada son la entrada de la siguiente
print("resultado final:", leer_archivo(runs[0]))
```

**Salida real:**

```
  run creada run_0.txt: [3, 12, 57, 90]
  run creada run_1.txt: [21, 45, 67, 88]
  run creada run_2.txt: [5, 34, 72, 99]
  run creada run_3.txt: [8, 61]
pasada 1:
  mezclar run_0.txt + run_1.txt -> merged_1_0.txt: [3, 12, 21, 45, 57, 67, 88, 90]
  mezclar run_2.txt + run_3.txt -> merged_1_1.txt: [5, 8, 34, 61, 72, 99]
pasada 2:
  mezclar merged_1_0.txt + merged_1_1.txt -> merged_2_0.txt: [3, 5, 8, 12, 21, 34, 45, 57, 61, 67, 72, 88, 90, 99]
resultado final: [3, 5, 8, 12, 21, 34, 45, 57, 61, 67, 72, 88, 90, 99]
```

Con 14 elementos y bloques de 4, se crearon 4 runs iniciales. Se necesitaron **2 pasadas de mezcla** para llegar a una sola run final — en general, con `r` runs iniciales, hacen falta `⌈log₂ r⌉` pasadas, porque cada pasada reduce el número de runs a la mitad (se llama "mezcla **directa**" porque el tamaño de los bloques iniciales es siempre el mismo, fijo de antemano, sin importar si el archivo ya trae partes ordenadas por casualidad).

**Costo:** cada pasada completa lee y escribe todo el archivo una vez, así que el costo total es O(n · log(n / B)), donde `B` es el tamaño del bloque que cabe en memoria — mientras más grande sea `B` (más RAM disponible), menos pasadas hacen falta.

---

## 5.2.3 Mezcla natural (Natural Merge Sort)

**Idea:** en vez de cortar el archivo en bloques de tamaño fijo sin mirar los datos, se recorre el archivo una vez y se detectan los tramos que **ya vienen ordenados por casualidad** — a eso se le llama una "run natural". Estas runs pueden tener tamaños distintos entre sí, a diferencia de la mezcla directa.

```python
def detectar_runs_naturales(datos):
    # un solo recorrido del archivo: agrupa en el mismo run mientras los valores sigan subiendo
    runs = []
    actual = [datos[0]]
    for i in range(1, len(datos)):
        if datos[i] >= datos[i-1]:
            actual.append(datos[i])  # el dato sigue la racha ascendente -> mismo run
        else:
            runs.append(actual)
            actual = [datos[i]]  # se rompió el orden -> empieza un run nuevo
    runs.append(actual)  # no olvidar el último run en construcción
    return runs

def mezclar(a, b):
    # misma operación de intercalación de la sección 5.2.1, reutilizada aquí también
    i = j = 0
    salida = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            salida.append(a[i]); i += 1
        else:
            salida.append(b[j]); j += 1
    salida.extend(a[i:]); salida.extend(b[j:])
    return salida

datos = [3, 12, 57, 90, 2, 45, 88, 1, 67, 99]
print("datos:", datos)
runs = detectar_runs_naturales(datos)  # fase 1: detectar tramos ya ordenados, sin tamaño fijo
print("runs naturales detectadas (sin tamaño fijo, aprovechando el orden ya existente):")
for r in runs:
    print(" ", r)

paso = 0
while len(runs) > 1:  # fase 2: mezclar runs de dos en dos hasta que quede solo una (igual que en 5.2.2)
    paso += 1
    nuevas = []
    print(f"pasada {paso}:")
    for i in range(0, len(runs), 2):
        if i+1 < len(runs):
            m = mezclar(runs[i], runs[i+1])
            print(f"  {runs[i]} + {runs[i+1]} -> {m}")
            nuevas.append(m)
        else:
            nuevas.append(runs[i])  # run impar sin pareja, pasa directo a la siguiente ronda
    runs = nuevas
print("resultado final:", runs[0])
```

**Salida real:**

```
datos: [3, 12, 57, 90, 2, 45, 88, 1, 67, 99]
runs naturales detectadas (sin tamaño fijo, aprovechando el orden ya existente):
  [3, 12, 57, 90]
  [2, 45, 88]
  [1, 67, 99]
pasada 1:
  [3, 12, 57, 90] + [2, 45, 88] -> [2, 3, 12, 45, 57, 88, 90]
pasada 2:
  [2, 3, 12, 45, 57, 88, 90] + [1, 67, 99] -> [1, 2, 3, 12, 45, 57, 67, 88, 90, 99]
resultado final: [1, 2, 3, 12, 45, 57, 67, 88, 90, 99]
```

Con estos mismos 10 datos, la mezcla natural encontró **3 runs** en lugar de las 3 runs de tamaño fijo que habría forzado la mezcla directa con bloques de tamaño 4 (que hubiera partido el archivo en `[3,12,57,90]`, `[2,45,88,1]`, `[67,99]` sin aprovechar que `2,45,88` ya venía ordenado por sí solo, pero cortando `1` después de `88` aunque rompiera esa racha). Cuando los datos de entrada ya tienen tramos ordenados "naturalmente" (algo común en datos del mundo real, como transacciones casi en orden cronológico), la mezcla natural necesita **menos runs** y por lo tanto **menos pasadas** que la mezcla directa — el caso extremo es un archivo ya completamente ordenado, donde la mezcla natural lo detecta como una sola run y termina de inmediato, sin hacer ninguna mezcla.

**Costo:** en el peor caso (datos en orden descendente total, donde cada elemento rompe la racha anterior) se comporta igual que la mezcla directa con bloques de tamaño 1. En el mejor caso (datos ya ordenados) termina en una sola pasada de detección, sin mezclas — ahí es donde la mezcla natural le gana claramente a la directa.

---

## Mezcla directa vs. mezcla natural

| | Mezcla directa | Mezcla natural |
|---|---|---|
| Tamaño de las runs iniciales | Fijo, elegido de antemano (el que cabe en memoria) | Variable, detectado según el orden que ya traen los datos |
| Número de runs iniciales | Siempre `⌈n / tam_bloque⌉` | Depende de qué tan ordenados vengan los datos — puede ser mucho menor |
| Mejor caso | Igual que el peor caso (no sabe si los datos ya vienen ordenados) | Una sola run — termina sin mezclar nada |
| Peor caso | O(n log(n/B)) | Igual que la mezcla directa con bloques de tamaño 1 |
| Cuándo conviene | Datos sin ningún orden previo conocido | Datos con tramos ya ordenados (registros casi cronológicos, logs, etc.) |

---

## Implementaciones reales en tecnología actual

El ordenamiento externo no es un tema de museo — es exactamente lo que hacen estos sistemas cuando el volumen de datos supera la memoria disponible:

- **El comando `sort` de Unix/Linux** — cuando el archivo a ordenar es más grande que la memoria disponible, `sort` cambia automáticamente a un algoritmo de **mezcla externa**: parte el archivo en bloques que sí caben en RAM, los ordena y los guarda como runs temporales en disco, y luego los va mezclando — el mismo patrón de `crear_runs` + `mezclar_dos_archivos` de la sección 5.2.2, solo que a escala de gigabytes.
- **`ORDER BY` en bases de datos (PostgreSQL, MySQL)** — cuando un `ORDER BY` o un `GROUP BY` necesita ordenar más filas de las que caben en la memoria asignada a la consulta (`work_mem` en PostgreSQL), el motor recurre a un **external merge sort**: genera runs ordenadas en disco temporal y las mezcla, igual que en esta práctica. Puedes verlo tú mismo con `EXPLAIN ANALYZE` sobre una consulta grande: el plan muestra literalmente un paso llamado "External Merge".
- **La fase *shuffle* de Hadoop MapReduce y Apache Spark** — al procesar datasets de terabytes repartidos en cientos de máquinas, cada nodo ordena sus propios datos localmente (fase *map*) y los escribe a disco; después, la fase *reduce* **mezcla** los fragmentos ya ordenados de todos los nodos — una mezcla directa distribuida, donde cada "run" vive en una máquina distinta.
- **Edición de video y procesamiento de logs a gran escala** — herramientas que procesan archivos de registro (logs) de varios gigabytes, demasiado grandes para cargar completos en memoria, usan el mismo patrón: ordenar fragmentos pequeños por separado (por ejemplo, por marca de tiempo) y mezclarlos — frecuentemente aprovechando que los logs **ya vienen casi ordenados cronológicamente**, el caso ideal para **mezcla natural**.
- **Motores de bases de datos columnares para analítica (ej. construcción de índices en ClickHouse, BigQuery)** — al construir un índice ordenado sobre una tabla de miles de millones de filas, el ordenamiento se hace por partes en disco/almacenamiento distribuido y se mezcla, nunca cargando la tabla completa en memoria de una sola vez.

## Fuentes

- Programa sinóptico oficial AED-1026 — TecNM (5.2 Algoritmos de ordenamiento externos)
- Knuth, D. — *The Art of Computer Programming, Vol. 3: Sorting and Searching* (capítulo de ordenamiento externo y mezcla)
