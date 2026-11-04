# Unidad 5 — Métodos de ordenamiento

**Materia:** Estructura de Datos (AED-1026)
**Grupo:** S3C — Aula ED12
**Lenguaje:** Python 3

Continuación directa de la [Unidad 4](../estructura_de_datos_u4/README.md): ahí viste estructuras no lineales (árboles y grafos). Esta unidad cambia de tema dentro del mismo curso: dada una colección de datos, ¿cómo se ordena de forma eficiente? Se divide en dos grandes familias: algoritmos **internos** (toda la lista cabe en memoria) y algoritmos **externos** (los datos son demasiado grandes y hay que ordenar usando disco).

## Contenido

- [01_ordenamiento_internos.md](01_ordenamiento_internos.md) — 5.1 Algoritmos de ordenamiento internos: Burbuja, Quicksort, ShellSort y Radix Sort — implementación paso a paso, salidas reales, análisis de complejidad, tabla comparativa e implementaciones reales (Timsort, Introsort, `qsort`, GPU radix sort, índices de bases de datos)
- [02_ordenamiento_externos.md](02_ordenamiento_externos.md) — 5.2 Algoritmos de ordenamiento externos: Intercalación (la operación de mezcla base), Mezcla Directa y Mezcla Natural — con archivos reales en disco simulando datos que no caben en memoria, e implementaciones reales (`sort` de Unix, `ORDER BY` externo en bases de datos, shuffle de Hadoop/Spark, procesamiento de logs)

## Nota sobre el código

Todo el código de esta unidad es Python real, verificado ejecutándolo antes de publicarse — ninguna salida mostrada en los documentos está inventada. La práctica de ordenamiento externo usa archivos reales en disco (lectura/escritura con `open()`), no una simulación en memoria.

## Unidad anterior

- [Unidad 4 — Estructuras no lineales: árboles y grafos](../estructura_de_datos_u4/README.md)

## Fuentes

- Programa sinóptico oficial AED-1026 — TecNM
- Cormen, T., Leiserson, C., Rivest, R., Stein, C. — *Introduction to Algorithms*
- Knuth, D. — *The Art of Computer Programming, Vol. 3: Sorting and Searching*
