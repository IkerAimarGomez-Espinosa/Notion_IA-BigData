# 🐍 Practica 13 - Sprints de Python

> **Instrucciones de entrega:
> **- Enlace al repositorio de Github. El formato de entrega son notebooks de Jupyter (.ipynb).
> - Se entrega un notebook por cada sprint, por ejemplo: sprint1.ipynb , sprint2.ipynb, sprint3.ipynb, sprint4.ipynb.
> - No hace falta entregar los sprints en markdown ni pdf

**Página relacionada:** 🚵🏽‍♂️Sprint_1-Estructuras básicas
**Temas:** listas, tuplas, set y diccionarios

En cada sección hay primero ejemplos resueltos y después ejercicios propuestos. Todos trabajan con situaciones habituales en un equipo de datos: lotes de ingesta, logs, registros JSON, esquemas o particiones de un data lake.

Cada ejemplo incluye el código, la explicación paso a paso y la salida esperada. Cada ejercicio propuesto incluye los datos de partida y la salida que debe producir tu solución, para que puedas comprobarla. Copia los datos tal cual en tu notebook o en tu script.

---

## Sección 1. Listas

Una lista es una colección **ordenada y modificable**. En ingeniería de datos aparece en todas partes: los ficheros pendientes de un lote, las filas leídas de un CSV, las métricas de una ejecución o la cola de tareas de un orquestador.

### Ejemplos resueltos

Cada noche llegan a la zona de aterrizaje los ficheros de ventas del día. El proceso de ingesta mantiene una lista con los ficheros pendientes.

```python
ficheros = ["ventas_0101.csv", "ventas_0102.csv", "ventas_0104.csv"]

# Llega un fichero nuevo: va al final del lote
ficheros.append("ventas_0105.csv")

# Llega con retraso el fichero del día 3: lo colocamos en su posición
ficheros.insert(2, "ventas_0103.csv")
print(ficheros)

print("Ficheros en el lote:", len(ficheros))
print("Primero:", ficheros[0], "| Último:", ficheros[-1])
print("Dos últimos:", ficheros[-2:])

# El fichero del día 1 ya se procesó: lo retiramos
ficheros.remove("ventas_0101.csv")
print(ficheros)
print("Posición de ventas_0104.csv:", ficheros.index("ventas_0104.csv"))
```

**Paso a paso**

1. `append()` añade siempre al final de la lista.
2. `insert(2, ...)` coloca el elemento en la posición 2 y desplaza el resto a la derecha. Así el fichero que llegó tarde queda en su sitio.
3. `len()` devuelve el número de elementos. Los índices empiezan en 0, y los negativos cuentan desde el final: `[-1]` es el último.
4. El *slicing* `[-2:]` devuelve una **lista nueva** con los dos últimos elementos.
5. `remove()` elimina la primera aparición del valor indicado. Si el valor no existe, lanza `ValueError`.
6. `index()` devuelve la posición del valor. Como ya hemos quitado el primer fichero, `ventas_0104.csv` pasa a la posición 2.
**Salida esperada**

```plain text
['ventas_0101.csv', 'ventas_0102.csv', 'ventas_0103.csv', 'ventas_0104.csv', 'ventas_0105.csv']
Ficheros en el lote: 5
Primero: ventas_0101.csv | Último: ventas_0105.csv
Dos últimos: ['ventas_0104.csv', 'ventas_0105.csv']
['ventas_0102.csv', 'ventas_0103.csv', 'ventas_0104.csv', 'ventas_0105.csv']
Posición de ventas_0104.csv: 2
```

Un pipeline lee datos de una API y guarda cuánto tarda cada petición. Queremos un pequeño informe de rendimiento.

```python
latencias_ms = [120, 85, 430, 95, 110, 1500, 88, 102, 97, 640]

total = sum(latencias_ms)
media = total / len(latencias_ms)
print("Peticiones:", len(latencias_ms))
print("Mínima:", min(latencias_ms), "ms | Máxima:", max(latencias_ms), "ms")
print("Media:", round(media, 1), "ms")

# sorted() devuelve una lista NUEVA: la original no cambia
ordenadas = sorted(latencias_ms)
n = len(ordenadas)
mediana = (ordenadas[n // 2 - 1] + ordenadas[n // 2]) / 2   # n es par
print("Mediana:", mediana, "ms")

print("Tres más lentas:", sorted(latencias_ms, reverse=True)[:3])
print("Original sin tocar:", latencias_ms)
```

**Paso a paso**

7. `sum()`, `min()`, `max()` y `len()` son funciones integradas que aceptan una lista.
8. La media (326,7 ms) se dispara por dos peticiones muy lentas (1500 y 640 ms). Por eso en monitorización se mira también la **mediana**, que no se deja arrastrar por los valores extremos.
9. Para la mediana hace falta ordenar. `sorted()` devuelve una lista nueva y deja la original intacta. Con 10 valores, la mediana es la media de los dos centrales: posiciones 4 y 5 de la lista ordenada (102 y 110).
10. `sorted(..., reverse=True)[:3]` ordena de mayor a menor y se queda con los tres primeros.
11. La última línea confirma que `latencias_ms` conserva su orden original. Si hubiéramos usado `latencias_ms.sort()`, la lista original sí habría cambiado.
**Salida esperada**

```plain text
Peticiones: 10
Mínima: 85 ms | Máxima: 1500 ms
Media: 326.7 ms
Mediana: 106.0 ms
Tres más lentas: [1500, 640, 430]
Original sin tocar: [120, 85, 430, 95, 110, 1500, 88, 102, 97, 640]
```

En una arquitectura medallón, la capa **bronce** guarda los datos tal como llegan y la capa **plata** guarda los datos limpios. La regla es no modificar nunca bronce.

```python
capa_bronce = [100, -5, 230, 0, 75]

# Error habitual: parece una copia, pero es la MISMA lista
capa_plata = capa_bronce
capa_plata.remove(-5)
print("Bronce:", capa_bronce)
print("Plata: ", capa_plata)
print("¿Misma lista?", capa_plata is capa_bronce)

# Forma correcta: copiar la lista
capa_bronce = [100, -5, 230, 0, 75]
capa_plata = capa_bronce.copy()       # también vale capa_bronce[:]
capa_plata.remove(-5)
print("Bronce:", capa_bronce)
print("Plata: ", capa_plata)
print("¿Misma lista?", capa_plata is capa_bronce)
```

**Paso a paso**

12. `capa_plata = capa_bronce` **no copia** la lista. Crea un segundo nombre (un *alias*) para el mismo objeto en memoria.
13. Por eso, al quitar el `5` desde `capa_plata`, también desaparece de `capa_bronce`. El operador `is` lo confirma: devuelve `True` porque ambas variables apuntan al mismo objeto.
14. Con `.copy()` (o `[:]`) se crea una lista nueva con el mismo contenido. Ahora los cambios en plata no afectan a bronce.
> Este error es muy frecuente en los pipelines: una función de limpieza recibe una lista y, sin querer, modifica los datos originales.

**Salida esperada**

```plain text
Bronce: [100, 230, 0, 75]
Plata:  [100, 230, 0, 75]
¿Misma lista? True
Bronce: [100, -5, 230, 0, 75]
Plata:  [100, 230, 0, 75]
¿Misma lista? False
```

Un orquestador ejecuta los jobs en el orden en que llegan, como una **cola**. En cambio, cuando hay que deshacer migraciones de base de datos, se revierte primero la última aplicada, como en una **pila**.

```python
from collections import deque

# Cola FIFO: los jobs se ejecutan en el orden en que llegan
cola_jobs = deque()
cola_jobs.append("ingesta_clientes")
cola_jobs.append("ingesta_pedidos")
cola_jobs.append("calculo_kpis")
print("Cola:", list(cola_jobs))

siguiente = cola_jobs.popleft()
print("Se ejecuta:", siguiente)
print("Pendientes:", list(cola_jobs))

# Pila LIFO: las migraciones se deshacen en orden inverso
migraciones = []
migraciones.append("v1_crear_tablas")
migraciones.append("v2_add_columna_email")
migraciones.append("v3_crear_indice")
print("Aplicadas:", migraciones)

while migraciones:                     # una lista vacía equivale a False
    print("Revirtiendo:", migraciones.pop())
print("Quedan:", migraciones)
```

**Paso a paso**

15. Quitar el primer elemento de una lista con `pop(0)` es lento en listas grandes, porque Python tiene que desplazar todos los demás. Para colas se usa `deque` del módulo `collections`, cuyo `popleft()` es inmediato.
16. Para la pila basta una lista normal: `append()` apila y `pop()` sin argumento desapila el último elemento.
17. `while migraciones:` se repite mientras la lista tenga elementos, porque una lista vacía se evalúa como `False`.
**Salida esperada**

```plain text
Cola: ['ingesta_clientes', 'ingesta_pedidos', 'calculo_kpis']
Se ejecuta: ingesta_clientes
Pendientes: ['ingesta_pedidos', 'calculo_kpis']
Aplicadas: ['v1_crear_tablas', 'v2_add_columna_email', 'v3_crear_indice']
Revirtiendo: v3_crear_indice
Revirtiendo: v2_add_columna_email
Revirtiendo: v1_crear_tablas
Quedan: []
```

Cuando se lee un CSV sin librerías, cada fila es una lista y el fichero completo es una lista de listas.

```python
from operator import itemgetter

# Cada fila: [fecha, tienda, importe]
filas = [
    ["2026-10-01", "MAD01", 1250.0],
    ["2026-10-01", "BCN02", 980.5],
    ["2026-10-02", "MAD01", 1530.0],
    ["2026-10-02", "VLC03", 410.0],
    ["2026-10-03", "BCN02", 1105.0],
]

print("Primera fila:", filas[0])
print("Tienda de la segunda fila:", filas[1][1])

importes = []
for fila in filas:
    importes.append(fila[2])
print("Columna importe:", importes)
print("Total vendido:", sum(importes))

por_importe = sorted(filas, key=itemgetter(2), reverse=True)
print("Venta más alta:", por_importe[0])
print("Venta más baja:", por_importe[-1])
```

**Paso a paso**

18. `filas[1][1]` accede primero a la fila 1 y, dentro de ella, a la columna 1 (la tienda).
19. Para extraer una columna recorremos las filas y guardamos el elemento 2 de cada una.
20. `itemgetter(2)` (del módulo `operator`) crea una función que, dada una fila, devuelve su elemento 2. `sorted()` la usa como **criterio de ordenación** mediante el parámetro `key`.
**Salida esperada**

```plain text
Primera fila: ['2026-10-01', 'MAD01', 1250.0]
Tienda de la segunda fila: BCN02
Columna importe: [1250.0, 980.5, 1530.0, 410.0, 1105.0]
Total vendido: 5275.5
Venta más alta: ['2026-10-02', 'MAD01', 1530.0]
Venta más baja: ['2026-10-02', 'VLC03', 410.0]
```

HDFS no guarda los ficheros enteros: los divide en **bloques** de 128 MB y guarda **3 copias** (réplicas) de cada bloque en nodos distintos (DataNodes). Así, si un nodo falla, los datos siguen disponibles en otro.

```python
tamano_fichero_mb = 700
tamano_bloque_mb = 128
replicas = 3
nodos = ["datanode1", "datanode2", "datanode3", "datanode4"]

# 1. Trocear el fichero: todos los bloques de 128 MB salvo el último
bloques = []
restante = tamano_fichero_mb
while restante > 0:
    bloques.append(min(tamano_bloque_mb, restante))
    restante -= tamano_bloque_mb
print("Bloques (MB):", bloques)
print("Número de bloques:", len(bloques))

# 2. Una lista vacía por nodo: lo que guardará cada DataNode
contenido = []
for _ in nodos:
    contenido.append([])

# 3. Repartir las 3 réplicas de cada bloque en nodos distintos
for b in range(len(bloques)):
    for r in range(replicas):
        destino = (b + r) % len(nodos)
        contenido[destino].append(f"blk{b}")

for i in range(len(nodos)):
    print(nodos[i], contenido[i])
print("Espacio ocupado en el clúster:", sum(bloques) * replicas, "MB")
```

**Paso a paso**

21. El bucle `while` va restando 128 MB hasta agotar el fichero. `min()` hace que el último bloque tenga solo lo que queda (60 MB). Los bloques no se rellenan hasta 128 MB.
22. `contenido` es una lista de listas: una lista por cada DataNode.
23. La réplica `r` del bloque `b` va al nodo `(b + r) % 4`. El operador `%` hace que, al llegar al último nodo, se vuelva a empezar por el primero, y garantiza que las 3 réplicas de un bloque caen en nodos distintos.
24. Un fichero de 700 MB ocupa 2100 MB en el clúster. Es el precio de la tolerancia a fallos.
**Salida esperada**

```plain text
Bloques (MB): [128, 128, 128, 128, 128, 60]
Número de bloques: 6
datanode1 ['blk0', 'blk2', 'blk3', 'blk4']
datanode2 ['blk0', 'blk1', 'blk3', 'blk4', 'blk5']
datanode3 ['blk0', 'blk1', 'blk2', 'blk4', 'blk5']
datanode4 ['blk1', 'blk2', 'blk3', 'blk5']
Espacio ocupado en el clúster: 2100 MB
```

En Spark, cada partición la procesa una tarea, y una etapa no termina hasta que acaba **la tarea más lenta**. Si unas particiones tienen muchos más datos que otras (*data skew*), todo el job espera por ellas.

```python
registros_por_particion = [10250, 9870, 10110, 48300, 9930, 10020, 9990, 51200]

total = sum(registros_por_particion)
media = total / len(registros_por_particion)
print("Particiones:", len(registros_por_particion), "| Registros:", total)
print("Media por partición:", round(media))

mayor = max(registros_por_particion)
print("Partición más grande:", registros_por_particion.index(mayor), "con", mayor, "registros")
print("Factor de skew (máx / media):", round(mayor / media, 2))

sesgadas = []
for i in range(len(registros_por_particion)):
    if registros_por_particion[i] > 2 * media:
        sesgadas.append(i)
print("Particiones con más del doble de la media:", sesgadas)

dos_mayores = sorted(registros_por_particion, reverse=True)[:2]
print("Las 2 mayores concentran el", round(sum(dos_mayores) / total * 100, 1), "% de los datos")
```

**Paso a paso**

25. Con la suma y la longitud de la lista se obtiene la media de registros por partición.
26. El **factor de skew** compara la partición más grande con la media. Un valor cercano a 1 indica un reparto equilibrado; aquí es 2,57.
27. El bucle con `range(len(...))` recorre los índices para poder informar del **número de partición**, no solo de su tamaño.
28. Dos particiones de ocho concentran más de la mitad de los datos: esas dos tareas marcarán la duración del job. La solución habitual es reparticionar (Ejercicio 1.6).
**Salida esperada**

```plain text
Particiones: 8 | Registros: 159670
Media por partición: 19959
Partición más grande: 7 con 51200 registros
Factor de skew (máx / media): 2.57
Particiones con más del doble de la media: [3, 7]
Las 2 mayores concentran el 62.3 % de los datos
```

### Ejercicios propuestos

Un sensor de temperatura envía `-999.0` cuando falla, y a veces envía valores imposibles. El rango físico válido es de -40 a 85 ºC, ambos incluidos.

```python
lecturas = [21.5, 22.1, -999.0, 23.4, 150.2, 22.8, -999.0, 24.0, 21.9, 87.3]
```

Construye dos listas, `validas` y `descartadas`, y muestra:

- Las dos listas.
- El porcentaje de lecturas descartadas, con un decimal.
- La media de las lecturas válidas, con dos decimales.
- Cuántas veces aparece el código de fallo `999.0` (usa un método de lista).
**Salida esperada**

```plain text
Válidas: [21.5, 22.1, 23.4, 22.8, 24.0, 21.9]
Descartadas: [-999.0, 150.2, -999.0, 87.3]
Porcentaje descartado: 40.0 %
Media de las válidas: 22.62
Códigos -999.0 (sensor caído): 2
```

Una API de clientes devuelve identificadores repetidos. Hay que eliminarlos **conservando el orden en que aparecen por primera vez**. Resuélvelo solo con listas (los `set` se ven en la Sección 3).

```python
ids_recibidos = ["C003", "C001", "C003", "C007", "C001", "C002", "C007", "C003"]
```

**Salida esperada**

```plain text
Únicos en orden de llegada: ['C003', 'C001', 'C007', 'C002']
Duplicados eliminados: 4
```

La API de destino acepta como máximo 5 registros por petición. Divide la lista en lotes con *slicing* y muestra cada lote con su número y su tamaño. El último lote puede ser más pequeño. Calcula el número de lotes con una fórmula, sin contar a mano.

```python
registros = list(range(1, 24))   # 23 registros: 1, 2, ..., 23
tamano_lote = 5
```

**Salida esperada**

```plain text
Lote 1: 5 registros -> [1, 2, 3, 4, 5]
Lote 2: 5 registros -> [6, 7, 8, 9, 10]
Lote 3: 5 registros -> [11, 12, 13, 14, 15]
Lote 4: 5 registros -> [16, 17, 18, 19, 20]
Lote 5: 3 registros -> [21, 22, 23]
Número de lotes: 5
```

Un compañero ha escrito este código para eliminar los importes negativos de un lote:

```python
importes = [120, -10, -35, 80, -5, -60, 200]
for x in importes:
    if x < 0:
        importes.remove(x)
print(importes)
```

Y obtiene esta salida:

```plain text
[120, -35, 80, -60, 200]
```

29. Explica por qué siguen apareciendo `35` y `60`. Pista: piensa qué le ocurre al índice interno del `for` cuando se elimina un elemento.
30. Corrige el código **sin modificar la lista mientras se recorre**.
**Salida esperada tras la corrección**

```plain text
[120, 80, 200]
```

```python
# [pedido_id, tienda, importe, estado]
pedidos = [
    ["P01", "MAD01", 250.0, "ENTREGADO"],
    ["P02", "BCN02", 89.9, "PENDIENTE"],
    ["P03", "MAD01", 1200.0, "ENTREGADO"],
    ["P04", "VLC03", 45.5, "CANCELADO"],
    ["P05", "MAD01", 310.0, "PENDIENTE"],
    ["P06", "BCN02", 760.0, "ENTREGADO"],
    ["P07", "MAD01", 99.0, "ENTREGADO"],
]
```

Escribe el código necesario para:

31. Obtener la lista de importes de los pedidos `ENTREGADO` de la tienda `MAD01` y su total.
32. Obtener los identificadores de los 3 pedidos de mayor importe, sea cual sea su estado.
33. Cambiar **en la propia lista** el estado `PENDIENTE` por `EN_CURSO` y mostrar la columna de estados.
**Salida esperada**

```plain text
Importes entregados en MAD01: [250.0, 1200.0, 99.0] -> total 1549.0
Top 3 por importe: ['P03', 'P06', 'P05']
Estados actualizados: ['ENTREGADO', 'EN_CURSO', 'ENTREGADO', 'CANCELADO', 'EN_CURSO', 'ENTREGADO', 'ENTREGADO']
```

La función `repartition()` de Spark redistribuye los registros para equilibrar las particiones. Vamos a simular la versión más sencilla, el reparto *round-robin*.

```python
# Tres particiones de entrada muy desequilibradas
entrada = [
    [101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112],
    [113, 114],
    [115, 116, 117, 118, 119, 120],
]
num_particiones = 4
```

34. Junta todos los registros en una sola lista, en orden. Puedes usar `extend()`.
35. Crea 4 particiones vacías y asigna el registro número `i` a la partición `i % num_particiones`.
36. Muestra el contenido de cada partición y compara el factor de skew (máximo / media) antes y después, con dos decimales.
**Salida esperada**

```plain text
Partición 0: [101, 105, 109, 113, 117]
Partición 1: [102, 106, 110, 114, 118]
Partición 2: [103, 107, 111, 115, 119]
Partición 3: [104, 108, 112, 116, 120]
Tamaños antes: [12, 2, 6] -> skew 1.8
Tamaños después: [5, 5, 5, 5] -> skew 1.0
```

Un sistema de streaming cuenta los eventos que llegan cada minuto. Para suavizar el ruido se calcula una **media móvil** con una ventana de 3 minutos.

```python
eventos_por_minuto = [120, 135, 128, 410, 140, 132, 125, 390, 395, 130]
ventana = 3
```

37. Calcula con *slicing* la media de cada ventana de 3 minutos consecutivos (minutos 0-2, 1-3, 2-4…), redondeada a un decimal.
38. Indica qué ventana tiene la media más alta.
39. Detecta los minutos con **pico**: aquellos cuyo valor supera el doble de la media de los 3 minutos anteriores. Empieza a comprobar en el minuto 3.
**Salida esperada**

```plain text
Medias móviles: [127.7, 224.3, 226.0, 227.3, 132.3, 215.7, 303.3, 305.0]
Ventanas calculadas: 8
Ventana más cargada: minutos 7-9 (media 305.0)
Minutos con pico: [3, 7]
```

---

## Sección 2. Tuplas

Una tupla es una colección **ordenada e inmutable**: una vez creada, no se puede cambiar. Por eso encaja muy bien para representar un registro cerrado, una clave compuesta o una fila que ningún paso del pipeline debe alterar por accidente.

### Ejemplos resueltos

```python
evento = ("2026-10-01T08:15:00", "sensor_07", 21.4)

print("Evento completo:", evento)
print("Sensor:", evento[1])
print("Campos:", len(evento))

# Desempaquetado: cada campo en su variable
momento, sensor, temperatura = evento
print(sensor, "midió", temperatura, "ºC a las", momento[11:16])

# Una tupla no se puede modificar:
# evento[2] = 22.0   ->  TypeError: 'tuple' object does not support item assignment

# Para "corregir" un valor se construye una tupla nueva
evento_corregido = evento[:2] + (22.0,)
print("Original: ", evento)
print("Corregido:", evento_corregido)

un_campo = ("sensor_07",)    # con coma: tupla de un elemento
sin_coma = ("sensor_07")     # sin coma: es solo un str entre paréntesis
print(type(un_campo).__name__, type(sin_coma).__name__)
```

**Paso a paso**

40. Se accede por índice igual que en una lista, y `len()` funciona igual.
41. El **desempaquetado** asigna cada campo a una variable. Debe haber tantas variables como elementos.
42. `momento[11:16]` extrae la hora (`08:15`) del texto con formato ISO.
43. Intentar asignar `evento[2] = 22.0` produce un `TypeError`. Para “corregir” un valor se construye una tupla nueva concatenando trozos: `evento[:2] + (22.0,)`.
44. Una tupla de un solo elemento necesita la coma final. Sin la coma, los paréntesis solo agrupan y el resultado es un `str`.
**Salida esperada**

```plain text
Evento completo: ('2026-10-01T08:15:00', 'sensor_07', 21.4)
Sensor: sensor_07
Campos: 3
sensor_07 midió 21.4 ºC a las 08:15
Original:  ('2026-10-01T08:15:00', 'sensor_07', 21.4)
Corregido: ('2026-10-01T08:15:00', 'sensor_07', 22.0)
tuple str
```

```python
from operator import itemgetter

lecturas = [
    ("sensor_01", "Madrid", 21.4),
    ("sensor_02", "Sevilla", 31.8),
    ("sensor_03", "Bilbao", 17.2),
    ("sensor_04", "Sevilla", 29.5),
    ("sensor_05", "Madrid", 24.9),
]

# Desempaquetado directo en el for
for sensor, ciudad, temp in lecturas:
    if temp > 25:
        print("ALERTA", sensor, ciudad, temp)

ordenadas = sorted(lecturas, key=itemgetter(2))
print("Más fría:  ", ordenadas[0])
print("Más cálida:", ordenadas[-1])

# Ordenar por ciudad y, dentro de cada ciudad, por temperatura
for fila in sorted(lecturas, key=itemgetter(1, 2)):
    print(fila)
```

**Paso a paso**

45. `for sensor, ciudad, temp in lecturas` desempaqueta cada tupla directamente en el bucle, sin usar índices.
46. `itemgetter(2)` ordena por temperatura.
47. `itemgetter(1, 2)` ordena por **dos criterios**: primero ciudad y, si hay empate, temperatura. Funciona porque las tuplas se comparan elemento a elemento, de izquierda a derecha.
**Salida esperada**

```plain text
ALERTA sensor_02 Sevilla 31.8
ALERTA sensor_04 Sevilla 29.5
Más fría:   ('sensor_03', 'Bilbao', 17.2)
Más cálida: ('sensor_02', 'Sevilla', 31.8)
('sensor_03', 'Bilbao', 17.2)
('sensor_01', 'Madrid', 21.4)
('sensor_05', 'Madrid', 24.9)
('sensor_04', 'Sevilla', 29.5)
('sensor_02', 'Sevilla', 31.8)
```

```python
particiones = ("2026-09-26", "2026-09-27", "2026-09-28", "2026-09-29", "2026-09-30")

primera, *intermedias, ultima = particiones
print("Desde:", primera, "hasta:", ultima)
print("Intermedias:", intermedias)

# Duración de un job en segundos -> horas, minutos y segundos
duracion = 7384
horas, resto = divmod(duracion, 3600)
minutos, segundos = divmod(resto, 60)
print(f"El job tardó{horas} h{minutos} min{segundos} s")

# Intercambio de variables en una sola línea
origen, destino = "staging", "produccion"
origen, destino = destino, origen
print(origen, destino)
```

**Paso a paso**

48. `primera, *intermedias, ultima` asigna el primer y el último elemento a sus variables. Lo que queda en medio va a `intermedias`, que siempre es una **lista**.
49. `divmod(a, b)` devuelve una tupla `(cociente, resto)`. 7384 s entre 3600 da 2 h y sobran 184 s. Esos 184 s son 3 min y 4 s.
50. `origen, destino = destino, origen` funciona porque Python primero crea la tupla `(destino, origen)` y después la desempaqueta. No hace falta una variable auxiliar.
**Salida esperada**

```plain text
Desde: 2026-09-26 hasta: 2026-09-30
Intermedias: ['2026-09-27', '2026-09-28', '2026-09-29']
El job tardó 2 h 3 min 4 s
produccion staging
```

Los formatos como Parquet guardan los datos **por columnas**, mientras que un CSV o una API los entregan **por filas**. `zip()` permite pasar de un formato al otro.

```python
ids = [101, 102, 103, 104]
clientes = ["Acme", "Globex", "Initech", "Umbrella"]
importes = [1200.0, 340.5, 980.0, 15.0]

# De columnas a filas: cada fila es una tupla
filas = list(zip(ids, clientes, importes))
print("Filas:")
for fila in filas:
    print(" ", fila)

# De filas a columnas: zip(*filas) desempaqueta las filas
col_ids, col_clientes, col_importes = zip(*filas)
print("Columna clientes:", col_clientes)
print("Total importes:", sum(col_importes))
```

**Paso a paso**

51. `zip(ids, clientes, importes)` toma un elemento de cada lista en cada vuelta y los junta en una tupla.
52. `zip(*filas)` hace la operación inversa. El  pasa cada fila como un argumento separado, y `zip` agrupa los primeros elementos de todas las filas, después los segundos, etc.
53. Cada columna que devuelve `zip` es una tupla, por eso `col_clientes` aparece entre paréntesis.
**Salida esperada**

```plain text
Filas:
  (101, 'Acme', 1200.0)
  (102, 'Globex', 340.5)
  (103, 'Initech', 980.0)
  (104, 'Umbrella', 15.0)
Columna clientes: ('Acme', 'Globex', 'Initech', 'Umbrella')
Total importes: 2535.5
```

MapReduce, el modelo de procesamiento de Hadoop, trabaja siempre con **pares clave-valor**, y en Python un par es una tupla. Vamos a simular un conteo de palabras sobre líneas de log en tres fases.

```python
from operator import itemgetter

# Cada línea la procesaría un nodo distinto del clúster
splits = [
    "ERROR disco lleno en datanode3",
    "WARN latencia alta en datanode1",
    "ERROR timeout en datanode1",
]

# MAP: cada palabra se emite como una tupla (clave, 1)
pares = []
for linea in splits:
    for palabra in linea.split():
        pares.append((palabra, 1))
print("Pares emitidos:", len(pares))
print("Primeros pares:", pares[:3])

# SHUFFLE / SORT: al ordenar por clave, las tuplas iguales quedan juntas
pares.sort(key=itemgetter(0))

# REDUCE: sumar los valores de cada grupo de claves consecutivas
resultado = []
clave_actual, suma = pares[0]
for clave, valor in pares[1:]:
    if clave == clave_actual:
        suma += valor
    else:
        resultado.append((clave_actual, suma))
        clave_actual, suma = clave, valor
resultado.append((clave_actual, suma))

print("Claves distintas:", len(resultado))
for clave, total in resultado:
    if total > 1:
        print(clave, total)
```

**Paso a paso**

54. **Map:** cada nodo procesa su trozo de datos (*split*) y emite una tupla `(palabra, 1)` por cada palabra, sin contar nada todavía.
55. **Shuffle / sort:** el sistema ordena los pares por clave para que las tuplas de una misma palabra queden juntas. En un clúster, esta fase mueve datos por la red entre nodos y es la más costosa.
56. **Reduce:** se recorren los pares ordenados. Mientras la clave se repite, se acumula; cuando cambia, se guarda el total de la clave anterior y se empieza a contar la nueva.
57. El último grupo se añade después del bucle, porque no hay ninguna clave nueva que provoque su guardado.
58. Python ordena las mayúsculas antes que las minúsculas, por eso `ERROR` aparece primero.
**Salida esperada**

```plain text
Pares emitidos: 14
Primeros pares: [('ERROR', 1), ('disco', 1), ('lleno', 1)]
Claves distintas: 10
ERROR 2
datanode1 2
en 3
```

Kafka identifica cada mensaje por tres datos: el topic, la partición y su posición dentro de ella (*offset*). El **lag** es el número de mensajes que el consumidor todavía no ha procesado.

```python
from operator import itemgetter

# (topic, partición, offset): último mensaje escrito en el broker
ultimos = [("pedidos", 0, 15230), ("pedidos", 1, 14980), ("pedidos", 2, 15410)]
# (topic, partición, offset): último mensaje confirmado por el consumidor
confirmados = [("pedidos", 0, 15228), ("pedidos", 1, 13100), ("pedidos", 2, 15410)]

lags = []
for (topic, particion, fin), (_, _, leido) in zip(ultimos, confirmados):
    lags.append((topic, particion, fin - leido))

for lag in sorted(lags, key=itemgetter(2), reverse=True):
    print(lag)

total = 0
for _, _, pendientes in lags:
    total += pendientes
print("Lag total del consumidor:", total)
print("Partición más retrasada:", max(lags, key=itemgetter(2))[:2])
```

**Paso a paso**

59. `zip()` empareja la tupla de cada partición en las dos listas. Funciona porque ambas listas están en el mismo orden de partición.
60. En el `for` se desempaquetan **dos tuplas a la vez**. El guion bajo `_` indica los campos que no necesitamos.
61. Lag = offset escrito − offset confirmado. La partición 1 acumula 1880 mensajes pendientes: el consumidor asignado a ella va retrasado.
62. `max(...)[:2]` toma la tupla con más lag y se queda con sus dos primeros campos, topic y partición.
**Salida esperada**

```plain text
('pedidos', 1, 1880)
('pedidos', 0, 2)
('pedidos', 2, 0)
Lag total del consumidor: 1882
Partición más retrasada: ('pedidos', 1)
```

### Ejercicios propuestos

```python
# (pedido_id, cliente, importe, estado)
pedidos = [
    (5001, "Acme", 250.0, "ENTREGADO"),
    (5002, "Globex", 1340.0, "ENTREGADO"),
    (5003, "Initech", 89.9, "CANCELADO"),
    (5004, "Acme", 610.0, "ENTREGADO"),
    (5005, "Umbrella", 45.0, "CANCELADO"),
    (5006, "Globex", 300.0, "PENDIENTE"),
]
```

Recorre los pedidos con desempaquetado en el `for` y muestra:

- La facturación de los pedidos `ENTREGADO`.
- El id y el cliente del pedido de mayor importe. Usa `max()` con `key`.
- Una lista de tuplas `(cliente, importe)` con los pedidos `CANCELADO`.
- Cuántos pedidos hay en cada estado. Guarda los estados en una lista y usa `count()`.
**Salida esperada**

```plain text
Facturación entregada: 2200.0
Pedido de mayor importe: 5002 de Globex
Cancelados: [('Initech', 89.9), ('Umbrella', 45.0)]
ENTREGADO: 3 | CANCELADO: 2 | PENDIENTE: 1
```

El esquema de una tabla se describe como una tupla de tuplas `(columna, tipo)`. Comprueba cada fila con `isinstance()` e informa de cada valor que no tenga el tipo esperado.

```python
esquema = (("id", int), ("cliente", str), ("importe", float), ("activo", bool))

filas = [
    (1, "Acme", 120.5, True),
    (2, "Globex", "340", True),
    ("3", "Initech", 99.9, False),
    (4, "Umbrella", 15.0, "no"),
]
```

Pistas: `enumerate(filas, start=1)` numera las filas desde 1, y `zip(esquema, fila)` empareja cada columna del esquema con su valor. El nombre de un tipo se obtiene con `tipo.__name__`.

**Salida esperada**

```plain text
Fila 2: 'importe' esperaba float y recibió str
Fila 3: 'id' esperaba int y recibió str
Fila 4: 'activo' esperaba bool y recibió str
Filas: 4 | Errores: 3
```

Un registro de esquemas guarda versiones como texto. Compararlas como texto da resultados erróneos, porque `"1.10.0"` se considera menor que `"1.9.7"`.

```python
versiones = ["1.4.2", "1.10.0", "1.9.7", "2.0.0", "1.10.3"]
```

63. Muestra el resultado incorrecto al comparar y ordenar como texto.
64. Convierte cada versión en una tupla de enteros `(mayor, menor, parche)`.
65. Ordénalas, obtén la última y muéstrala con el formato `v2.0.0`.
**Salida esperada**

```plain text
Como texto, ¿1.10.0 > 1.9.7? False
Ordenadas como texto: ['1.10.0', '1.10.3', '1.4.2', '1.9.7', '2.0.0']
Como tuplas: [(1, 4, 2), (1, 10, 0), (1, 9, 7), (2, 0, 0), (1, 10, 3)]
Ordenadas: [(1, 4, 2), (1, 9, 7), (1, 10, 0), (1, 10, 3), (2, 0, 0)]
Última versión: v2.0.0
¿1.10.0 es posterior a 1.9.7? True
```

Cada tupla indica el minuto del día en que arranca y termina un job. Dos jobs que comparten clúster no deberían solaparse.

```python
# (job, minuto_inicio, minuto_fin) dentro del día
ejecuciones = [
    ("job_c", 130, 170),
    ("job_a", 60, 95),
    ("job_d", 165, 200),
    ("job_b", 90, 120),
]
```

Ordena las ejecuciones por hora de inicio. Después compara cada ejecución con la siguiente: hay solape si la siguiente arranca antes de que termine la actual. Para cada solape, muestra los jobs implicados y los minutos de solape.

**Salida esperada**

```plain text
Orden de arranque: ['job_a', 'job_b', 'job_c', 'job_d']
SOLAPE: job_b arranca en 90 y job_a termina en 95 (5 min)
SOLAPE: job_d arranca en 165 y job_c termina en 170 (5 min)
Solapes detectados: 2
```

Cada línea de log tiene el formato `método ruta código_http bytes`.

```python
# método ruta código_http bytes
logs = [
    "GET /pedidos 200 5120",
    "GET /clientes 404 310",
    "POST /pedidos 201 880",
    "GET /pedidos 200 4980",
    "GET /stock 500 120",
    "GET /clientes 200 2210",
    "GET /stock 500 95",
    "POST /pedidos 201 910",
]
```

Simula un job MapReduce como el del Ejemplo 2.5:

66. **Map:** emite una tupla `(código, bytes)` por línea. Los bytes deben ser `int`.
67. **Shuffle / sort:** ordena los pares por código.
68. **Reduce:** obtén una lista de tuplas `(código, peticiones, bytes_totales)` y muéstrala.
**Salida esperada**

```plain text
Fase map: [('200', 5120), ('404', 310), ('201', 880)] ...
200: 3 peticiones, 12310 bytes
201: 2 peticiones, 1790 bytes
404: 1 peticiones, 310 bytes
500: 2 peticiones, 215 bytes
```

Cuando una consulta filtra por fecha sobre una tabla particionada, el motor solo lee las carpetas de las fechas pedidas y descarta el resto sin abrirlas. Esto se llama *partition pruning*.

```python
# (año, mes, día, número de ficheros) de cada partición de la tabla ventas
particiones = [
    (2026, 9, 27, 14), (2026, 9, 28, 12), (2026, 9, 29, 15), (2026, 9, 30, 13),
    (2026, 10, 1, 16), (2026, 10, 2, 11), (2026, 10, 3, 12),
]
desde = (2026, 9, 29)
hasta = (2026, 10, 1)
```

69. Selecciona las particiones cuya fecha `(año, mes, día)` esté entre `desde` y `hasta`, ambos incluidos. Compara tuplas directamente, sin convertirlas a texto.
70. Muestra la ruta de cada partición seleccionada con el formato `ventas/year=AAAA/month=MM/day=DD` y su número de ficheros.
71. Indica cuántos ficheros se leen del total y qué porcentaje se ha descartado.
**Salida esperada**

```plain text
ventas/year=2026/month=09/day=29 -> 15 ficheros
ventas/year=2026/month=09/day=30 -> 13 ficheros
ventas/year=2026/month=10/day=01 -> 16 ficheros
Ficheros leídos: 44 de 93
Ficheros descartados: 52.7 %
```

---

## Sección 3. Set

Un `set` es una colección **sin orden y sin duplicados**. Sus dos usos principales en ingeniería de datos son **deduplicar** y **comparar colecciones**: qué registros faltan, cuáles sobran y cuáles coinciden. Además, comprobar si un elemento pertenece a un `set` es mucho más rápido que buscarlo en una lista.

> Como un `set` no tiene orden, los ejemplos usan `sorted()` para mostrarlos. Así la salida es siempre la misma.

### Ejemplos resueltos

```python
crm = ["C001", "C002", "C003", "C002", "C004", "C001"]
ecommerce = ["C003", "C005", "C004", "C006", "C005"]

print("Registros en CRM:", len(crm), "| únicos:", len(set(crm)))

clientes = set(crm)
clientes.update(ecommerce)      # añade todos los elementos de la otra fuente
print("Clientes únicos totales:", len(clientes))
print(sorted(clientes))

clientes.add("C007")
clientes.add("C001")            # ya existe: no cambia nada
clientes.discard("C999")        # no existe: discard() no da error
print("Tras las altas:", len(clientes))
```

**Paso a paso**

72. `set(crm)` elimina automáticamente los duplicados: 6 registros se quedan en 4 clientes.
73. `update()` añade todos los elementos de otra colección. Los repetidos se ignoran.
74. `add()` añade un elemento. Si ya existía, no pasa nada.
75. `discard()` elimina un elemento sin dar error si no existe. `remove()`, en cambio, lanzaría `KeyError`.
**Salida esperada**

```plain text
Registros en CRM: 6 | únicos: 4
Clientes únicos totales: 6
['C001', 'C002', 'C003', 'C004', 'C005', 'C006']
Tras las altas: 7
```

Después de una carga, hay que comprobar que todos los ids del sistema origen han llegado al data warehouse.

```python
origen = {101, 102, 103, 104, 105, 106}
destino = {101, 102, 104, 106, 107}

print("Cargados correctamente:", sorted(origen & destino))
print("Faltan en destino:     ", sorted(origen - destino))
print("Sobran en destino:     ", sorted(destino - origen))
print("Discrepancias totales: ", sorted(origen ^ destino))
print("¿Carga completa?", origen <= destino)
```

**Paso a paso**

| Operador | Operación | Pregunta que responde |
| --- | --- | --- |
| `a & b` | Intersección | ¿Qué está en los dos? |
| `a - b` | Diferencia | ¿Qué está en `a` pero no en `b`? |
| `a ^ b` | Diferencia simétrica | ¿Qué está solo en uno de los dos? |
| `a <= b` | Subconjunto | ¿Está todo `a` dentro de `b`? |

Los ids 103 y 105 no se cargaron, y el 107 está en destino aunque no viene del origen. Puede ser un registro antiguo que debería haberse borrado.

**Salida esperada**

```plain text
Cargados correctamente: [101, 102, 104, 106]
Faltan en destino:      [103, 105]
Sobran en destino:      [107]
Discrepancias totales:  [103, 105, 107]
¿Carga completa? False
```

```python
esperadas = {"id_pedido", "fecha", "cliente", "importe", "moneda"}
recibidas = ["id_pedido", "fecha", "importe", "cliente", "canal", "importe"]

recibidas_set = set(recibidas)
print("¿Columnas duplicadas en origen?", len(recibidas) != len(recibidas_set))
print("Faltan:", sorted(esperadas - recibidas_set))
print("Nuevas:", sorted(recibidas_set - esperadas))
print("¿Esquema válido?", esperadas.issubset(recibidas_set))
```

**Paso a paso**

76. Si al convertir la lista en `set` se reduce su tamaño, había columnas duplicadas: `importe` aparece dos veces.
77. `esperadas - recibidas_set` indica qué columnas faltan, y la resta inversa indica las columnas nuevas.
78. `issubset()` es la versión con método del operador `<=`.
**Salida esperada**

```plain text
¿Columnas duplicadas en origen? True
Faltan: ['moneda']
Nuevas: ['canal']
¿Esquema válido? False
```

```python
ips_bloqueadas = {"10.0.0.13", "192.168.1.50", "172.16.4.2"}

accesos = [
    ("08:01", "10.0.0.7"),
    ("08:02", "192.168.1.50"),
    ("08:02", "10.0.0.8"),
    ("08:03", "172.16.4.2"),
    ("08:05", "192.168.1.50"),
]

sospechosos = []
ips_vistas = set()
for hora, ip in accesos:
    ips_vistas.add(ip)
    if ip in ips_bloqueadas:          # búsqueda muy rápida en un set
        sospechosos.append((hora, ip))

print("Accesos bloqueados:", len(sospechosos))
for s in sospechosos:
    print(" ", s)
print("IPs distintas vistas:", len(ips_vistas))
print("Bloqueadas que no han aparecido:", ips_bloqueadas - ips_vistas)
```

**Paso a paso**

79. `ip in ips_bloqueadas` es casi instantáneo aunque el `set` tenga millones de IPs, porque internamente se busca por *hash*. En una lista habría que revisar los elementos uno a uno.
80. Un `set` puede contener tuplas, porque son inmutables, pero no listas. `{[1, 2]}` produce `TypeError: unhashable type: 'list'`.
81. Al restar las IPs vistas de las bloqueadas sabemos qué IPs de la lista negra no han aparecido en este log.
**Salida esperada**

```plain text
Accesos bloqueados: 3
  ('08:02', '192.168.1.50')
  ('08:03', '172.16.4.2')
  ('08:05', '192.168.1.50')
IPs distintas vistas: 4
Bloqueadas que no han aparecido: {'10.0.0.13'}
```

Contar elementos distintos (`COUNT(DISTINCT ...)`) es una de las operaciones más delicadas en un sistema distribuido, porque un mismo usuario puede aparecer en varias particiones.

```python
# Usuarios que generaron eventos, repartidos en 3 particiones
particion_0 = ["u1", "u2", "u3", "u2", "u4"]
particion_1 = ["u3", "u5", "u1", "u6"]
particion_2 = ["u2", "u7", "u5", "u5"]

unicos_0 = set(particion_0)
unicos_1 = set(particion_1)
unicos_2 = set(particion_2)
print("Únicos por partición:", len(unicos_0), len(unicos_1), len(unicos_2))

suma_parcial = len(unicos_0) + len(unicos_1) + len(unicos_2)
print("Suma de los únicos parciales:", suma_parcial, "(incorrecto)")

globales = unicos_0 | unicos_1 | unicos_2
print("Únicos globales:", len(globales), sorted(globales))
print("Usuarios contados de más:", suma_parcial - len(globales))
```

**Paso a paso**

82. Cada partición calcula su `set` de usuarios únicos de forma independiente, como haría cada nodo del clúster.
83. **Sumar los tamaños de los ****`set`**** parciales es un error**: `u1`, `u2`, `u3` y `u5` aparecen en más de una partición y se cuentan varias veces.
84. El resultado correcto exige **unir** los `set` (`|`) y contar después. En un clúster, esto obliga a mover los identificadores por la red hasta un mismo nodo.
85. Con miles de millones de usuarios, unir los `set` exactos es muy caro. Por eso los motores de Big Data ofrecen conteos aproximados, como `approx_count_distinct()` en Spark.
**Salida esperada**

```plain text
Únicos por partición: 4 4 3
Suma de los únicos parciales: 11 (incorrecto)
Únicos globales: 7 ['u1', 'u2', 'u3', 'u4', 'u5', 'u6', 'u7']
Usuarios contados de más: 4
```

Muchos sistemas de mensajería garantizan que cada evento llega **al menos una vez** (*at-least-once*). Si hay un fallo, el evento se reenvía, así que el consumidor debe descartar los repetidos para no escribirlos dos veces.

```python
procesados = set()        # ids de eventos ya escritos en destino

micro_lotes = [
    ["e01", "e02", "e03"],
    ["e03", "e04", "e05"],          # e03 se reenvió tras un fallo
    ["e05", "e06", "e02", "e07"],   # e05 y e02 llegan otra vez
]

for num, lote in enumerate(micro_lotes, start=1):
    nuevos = []
    for evento in lote:
        if evento not in procesados:
            nuevos.append(evento)
            procesados.add(evento)
    descartados = len(lote) - len(nuevos)
    print(f"Lote{num}: recibidos{len(lote)}, escritos{nuevos}, descartados{descartados}")

print("Eventos escritos en total:", len(procesados))
```

**Paso a paso**

86. `procesados` es un `set` que se mantiene entre micro-lotes y recuerda qué eventos ya se han escrito.
87. Para cada evento, `in` comprueba de forma inmediata si ya se procesó. Si es nuevo, se escribe y se añade al `set`.
88. `enumerate(..., start=1)` numera los lotes desde 1.
89. Gracias a esta comprobación, procesar varias veces el mismo evento produce el mismo resultado. Esta propiedad se llama **idempotencia**.
90. En un sistema real este `set` no puede crecer sin límite. Spark Structured Streaming, por ejemplo, olvida los ids más antiguos que un tiempo configurado (*watermark*).
**Salida esperada**

```plain text
Lote 1: recibidos 3, escritos ['e01', 'e02', 'e03'], descartados 0
Lote 2: recibidos 3, escritos ['e04', 'e05'], descartados 1
Lote 3: recibidos 4, escritos ['e06', 'e07'], descartados 2
Eventos escritos en total: 7
```

### Ejercicios propuestos

Cada `set` contiene los usuarios que abrieron la app ese día.

```python
lunes = {"u01", "u02", "u03", "u04", "u05", "u06"}
martes = {"u02", "u03", "u05", "u07", "u08"}
miercoles = {"u01", "u03", "u05", "u08", "u09"}
```

Calcula, solo con operadores de `set`:

- Los usuarios activos los tres días.
- Cuántos usuarios distintos estuvieron activos al menos un día.
- Los que solo entraron el lunes.
- Los usuarios nuevos del miércoles, es decir, los que no estuvieron ni el lunes ni el martes.
- La retención del lunes al martes: porcentaje de usuarios del lunes que volvieron el martes.
**Salida esperada**

```plain text
Activos los tres días: ['u03', 'u05']
Activos al menos un día: 9
Solo el lunes: ['u04', 'u06']
Nuevos el miércoles: ['u09']
Retención lunes -> martes: 50.0 %
```

Antes de publicar datasets en una zona compartida, el equipo de gobierno del dato pide revisar si contienen columnas con información personal (PII).

```python
pii = {"dni", "email", "telefono", "iban", "fecha_nacimiento"}

datasets = [
    ("ventas", {"id_venta", "fecha", "importe", "id_cliente"}),
    ("clientes", {"id_cliente", "nombre", "email", "telefono", "ciudad"}),
    ("pagos", {"id_pago", "iban", "importe", "fecha"}),
    ("logs_web", {"timestamp", "url", "user_agent"}),
]
```

Para cada dataset, usa `isdisjoint()` para saber si está libre de PII. Si no lo está, muestra qué columnas sensibles contiene. Al final, indica cuántos datasets necesitan enmascarado.

**Salida esperada**

```plain text
ventas    sin PII
clientes  PII -> ['email', 'telefono']
pagos     PII -> ['iban']
logs_web  sin PII
Datasets que requieren enmascarado: 2
```

`set(transacciones)` elimina los duplicados, pero no dice **cuáles** estaban repetidos. Usa dos `set`, `vistos` y `repetidos`, en un solo recorrido.

```python
transacciones = ["T10", "T11", "T12", "T11", "T13", "T14", "T12", "T11", "T15"]
```

**Salida esperada**

```plain text
Repetidos: ['T11', 'T12']
Únicos: 6 de 9
```

```python
rutas_hoy = [("MAD", "BCN"), ("BCN", "MAD"), ("MAD", "LIS"), ("MAD", "BCN"),
             ("SVQ", "MAD"), ("LIS", "MAD"), ("BIO", "PMI")]
rutas_manana = [("PMI", "BIO"), ("MAD", "VLC")]
```

91. Crea un `set` de rutas distintas. Una ruta es una tupla `(origen, destino)`.
92. Obtén las rutas que no tienen vuelta, es decir, aquellas en las que `(destino, origen)` no está en el `set`.
93. Incorpora las rutas de mañana con el operador `|=` y repite el cálculo.
**Salida esperada**

```plain text
Rutas distintas hoy: 6
Sin vuelta: [('BIO', 'PMI'), ('SVQ', 'MAD')]
Rutas acumuladas: 8
Sin vuelta tras añadir mañana: [('MAD', 'VLC'), ('SVQ', 'MAD')]
```

Un job diario escribe una carpeta por fecha en HDFS y, al terminar con éxito, deja dentro un fichero vacío llamado `_SUCCESS`. Una carpeta sin ese marcador puede tener datos a medias.

```python
esperadas = {"2026-10-01", "2026-10-02", "2026-10-03", "2026-10-04",
             "2026-10-05", "2026-10-06", "2026-10-07"}
# Carpetas que existen en HDFS
presentes = {"2026-09-30", "2026-10-01", "2026-10-02", "2026-10-03",
             "2026-10-05", "2026-10-06", "2026-10-07"}
# Carpetas que contienen el marcador _SUCCESS (escritura terminada)
con_success = {"2026-09-30", "2026-10-01", "2026-10-02", "2026-10-05", "2026-10-07"}
```

Usando solo operaciones de `set`, obtén:

- Las particiones esperadas que no existen.
- Las particiones esperadas que existen pero no tienen `_SUCCESS`.
- La lista de particiones a reprocesar (unión de las dos anteriores).
- Las carpetas que existen pero no corresponden al rango esperado.
- Cuántas particiones esperadas están completas y qué porcentaje representan.
**Salida esperada**

```plain text
Faltan: ['2026-10-04']
Incompletas: ['2026-10-03', '2026-10-06']
A reprocesar: ['2026-10-03', '2026-10-04', '2026-10-06']
Fuera de rango: ['2026-09-30']
Completas: 4 de 7 (57.1 %)
```

Dos DataNodes del clúster han dejado de responder. El NameNode necesita saber qué bloques han perdido réplicas.

```python
nodos_vivos = {"dn1", "dn2", "dn4", "dn5"}

# (bloque, set de nodos que guardan una réplica)
bloques = [
    ("blk_001", {"dn1", "dn2", "dn3"}),
    ("blk_002", {"dn3", "dn6", "dn4"}),
    ("blk_003", {"dn2", "dn4", "dn5"}),
    ("blk_004", {"dn3", "dn6", "dn1"}),
    ("blk_005", {"dn6", "dn3", "dn2"}),
    ("blk_006", {"dn3", "dn6"}),
]
```

94. Para cada bloque, calcula sus réplicas vivas (intersección con `nodos_vivos`) y muéstralas ordenadas.
95. Reúne en un `set` todos los nodos que aparecen en los bloques (usa `|=`) y obtén los nodos caídos.
96. Clasifica los bloques: **en riesgo** si les queda 1 réplica viva y **perdidos** si no les queda ninguna.
**Salida esperada**

```plain text
blk_001: 2 réplicas vivas ['dn1', 'dn2']
blk_002: 1 réplicas vivas ['dn4']
blk_003: 3 réplicas vivas ['dn2', 'dn4', 'dn5']
blk_004: 1 réplicas vivas ['dn1']
blk_005: 1 réplicas vivas ['dn2']
blk_006: 0 réplicas vivas []
Nodos caídos: ['dn3', 'dn6']
Bloques en riesgo: ['blk_002', 'blk_004', 'blk_005']
Bloques perdidos: ['blk_006']
```

---

## Sección 4. Diccionarios

Un diccionario guarda pares **clave → valor** y permite acceder a cada valor por su clave de forma muy rápida. Es la estructura que corresponde a un objeto JSON. También es la herramienta básica para contar, agrupar y cruzar datos sin una base de datos.

### Ejemplos resueltos

```python
pedido = {
    "id": 50231,
    "cliente": "Acme",
    "importe": 1250.0,
    "moneda": "EUR",
}

print(pedido["cliente"])
print(pedido.get("canal"))               # la clave no existe: None
print(pedido.get("canal", "online"))     # valor por defecto

pedido["canal"] = "tienda"      # añadir
pedido["importe"] = 1190.0      # modificar
moneda = pedido.pop("moneda")   # extraer y eliminar
print("Moneda retirada:", moneda)
print(pedido)

print("Claves:", list(pedido.keys()))
for campo, valor in pedido.items():
    print(f"{campo:<8} ->{valor}")
print("¿Tiene 'importe'?", "importe" in pedido)
```

**Paso a paso**

97. `pedido["cliente"]` lanza `KeyError` si la clave no existe. `get()` devuelve `None` en ese caso, o el valor por defecto que le indiquemos.
98. Asignar a una clave nueva la **añade**, y asignar a una clave existente la **modifica**.
99. `pop()` elimina la clave y devuelve su valor.
100. `keys()`, `values()` e `items()` permiten recorrer las claves, los valores o los pares. Los diccionarios conservan el orden de inserción.
101. `in` comprueba si existe una **clave**, no un valor.
**Salida esperada**

```plain text
Acme
None
online
Moneda retirada: EUR
{'id': 50231, 'cliente': 'Acme', 'importe': 1190.0, 'canal': 'tienda'}
Claves: ['id', 'cliente', 'importe', 'canal']
  id       -> 50231
  cliente  -> Acme
  importe  -> 1190.0
  canal    -> tienda
¿Tiene 'importe'? True
```

```python
eventos = ["login", "click", "click", "compra", "login", "click",
           "error", "click", "logout", "error"]

conteo = {}
for e in eventos:
    conteo[e] = conteo.get(e, 0) + 1
print(conteo)

for tipo in sorted(conteo):
    print(f"{tipo:<7}{'#' * conteo[tipo]}{conteo[tipo]}")
```

**Paso a paso**

102. `conteo.get(e, 0)` devuelve el contador actual o 0 si es la primera vez que aparece el evento. Así no hace falta comprobar antes si la clave existe.
103. `sorted(conteo)` devuelve las **claves** ordenadas alfabéticamente.
104. `'#' * n` repite el carácter `n` veces y produce un histograma de texto.
**Salida esperada**

```plain text
{'login': 2, 'click': 4, 'compra': 1, 'error': 2, 'logout': 1}
click   #### 4
compra  # 1
error   ## 2
login   ## 2
logout  # 1
```

```python
ventas = [
    ("MAD01", 1250.0), ("BCN02", 980.5), ("MAD01", 1530.0),
    ("VLC03", 410.0), ("BCN02", 1105.0), ("MAD01", 300.0),
]

resumen = {}
for tienda, importe in ventas:
    if tienda not in resumen:
        resumen[tienda] = {"total": 0.0, "pedidos": 0}
    resumen[tienda]["total"] += importe
    resumen[tienda]["pedidos"] += 1

print(resumen)
for tienda, datos in resumen.items():
    media = datos["total"] / datos["pedidos"]
    print(f"{tienda}:{datos['pedidos']} pedidos, total{datos['total']:.2f}, media{media:.2f}")
```

**Paso a paso**

105. Cada tienda es una clave, y su valor es otro diccionario con dos acumuladores.
106. La primera vez que aparece una tienda se inicializan sus acumuladores. Las siguientes veces solo se actualizan.
107. Es lo mismo que haría en SQL `SELECT tienda, COUNT(*), SUM(importe), AVG(importe) ... GROUP BY tienda`.
**Salida esperada**

```plain text
{'MAD01': {'total': 3080.0, 'pedidos': 3}, 'BCN02': {'total': 2085.5, 'pedidos': 2}, 'VLC03': {'total': 410.0, 'pedidos': 1}}
MAD01: 3 pedidos, total 3080.00, media 1026.67
BCN02: 2 pedidos, total 2085.50, media 1042.75
VLC03: 1 pedidos, total 410.00, media 410.00
```

```python
clientes = {
    "C001": {"nombre": "Acme", "pais": "ES"},
    "C002": {"nombre": "Globex", "pais": "PT"},
    "C003": {"nombre": "Initech", "pais": "MX"},
}
pedidos = [("P1", "C001", 300.0), ("P2", "C003", 125.5),
           ("P3", "C009", 80.0), ("P4", "C001", 47.0)]

desconocido = {"nombre": "DESCONOCIDO", "pais": "??"}
for id_pedido, id_cliente, importe in pedidos:
    cli = clientes.get(id_cliente, desconocido)
    print(id_pedido, id_cliente, cli["nombre"], cli["pais"], importe)
```

**Paso a paso**

108. El diccionario `clientes` funciona como una **tabla de búsqueda** indexada por `id_cliente`. Así funciona internamente un *hash join*.
109. `get(id_cliente, desconocido)` resuelve los clientes que no existen, como `C009`, sin que el proceso falle. Es el equivalente a un `LEFT JOIN` con un valor por defecto.
**Salida esperada**

```plain text
P1 C001 Acme ES 300.0
P2 C003 Initech MX 125.5
P3 C009 DESCONOCIDO ?? 80.0
P4 C001 Acme ES 47.0
```

```python
config_base = {
    "formato": "parquet",
    "compresion": "snappy",
    "particiones": 8,
    "destino": {"zona": "plata", "ruta": "/lake/plata"},
}
config_prod = {"particiones": 64, "compresion": "zstd"}

# Fusión: las claves de config_prod sobrescriben a las de config_base
config = {**config_base, **config_prod}
print(config)
print("Particiones:", config["particiones"])
print("Ruta destino:", config["destino"]["ruta"])

# Cuidado: el diccionario anidado "destino" es compartido
config["destino"]["zona"] = "oro"
print("Zona en config_base:", config_base["destino"]["zona"])
```

**Paso a paso**

110. `{**a, **b}` crea un diccionario nuevo con todas las claves de `a` y de `b`. Si una clave está en los dos, gana el valor de `b`, que está más a la derecha.
111. La fusión es **superficial**: el diccionario anidado `destino` no se copia, sino que se comparte. Por eso, al cambiar `config["destino"]["zona"]` también cambia `config_base`. Es el mismo problema de alias que vimos en el Ejemplo 1.3.
112. Para copiar también los niveles internos se usa `copy.deepcopy()`.
**Salida esperada**

```plain text
{'formato': 'parquet', 'compresion': 'zstd', 'particiones': 64, 'destino': {'zona': 'plata', 'ruta': '/lake/plata'}}
Particiones: 64
Ruta destino: /lake/plata
Zona en config_base: oro
```

Spark y Kafka deciden en qué partición va cada registro calculando un número a partir de su clave (*hash*) y quedándose con el resto de dividirlo entre el número de particiones.

```python
import zlib

num_particiones = 3
claves = ["C001", "C002", "C003", "C004", "C005", "C001", "C006", "C003"]

particiones = {}
for clave in claves:
    p = zlib.crc32(clave.encode()) % num_particiones
    particiones.setdefault(p, []).append(clave)

for p in sorted(particiones):
    print(f"Partición{p}:{particiones[p]}")
```

**Paso a paso**

113. `zlib.crc32()` convierte el texto de la clave (pasado a bytes con `encode()`) en un número entero. Para la misma clave, el número es siempre el mismo.
114. `% num_particiones` convierte ese número en una partición entre 0 y 2.
115. `setdefault(p, [])` devuelve la lista de la partición y, si todavía no existe, la crea vacía. Así se puede hacer `append()` directamente.
116. Todos los registros de `C001` y de `C003` caen siempre en la misma partición. Por eso un `groupBy` o un `join` por cliente puede resolverse dentro de cada partición.
117. No se usa la función `hash()` de Python porque, para los textos, cambia en cada ejecución del programa. Un particionado debe ser estable entre ejecuciones y entre máquinas.
**Salida esperada**

```plain text
Partición 0: ['C003', 'C003']
Partición 1: ['C002', 'C005', 'C006']
Partición 2: ['C001', 'C004', 'C001']
```

En el Ejemplo 2.5 cada palabra viajaba por la red como un par `(palabra, 1)`. Un *combiner* hace un conteo **local** en cada nodo antes de enviar nada, y así reduce el volumen de datos del shuffle.

```python
splits = [
    ["click", "view", "click", "view", "view", "compra"],
    ["view", "view", "click", "view"],
    ["click", "compra", "view", "click", "click"],
]

# MAP + COMBINER: cada nodo cuenta primero su propio split
parciales = []
for split in splits:
    local = {}
    for evento in split:
        local[evento] = local.get(evento, 0) + 1
    parciales.append(local)
    print("Conteo local:", local)

# REDUCE: se fusionan los conteos parciales
total = {}
for local in parciales:
    for evento, n in local.items():
        total[evento] = total.get(evento, 0) + n
print("Conteo global:", total)

sin_combiner = 0
con_combiner = 0
for i in range(len(splits)):
    sin_combiner += len(splits[i])
    con_combiner += len(parciales[i])
print("Pares enviados por la red sin combiner:", sin_combiner)
print("Pares enviados por la red con combiner:", con_combiner)
```

**Paso a paso**

118. Cada nodo construye su propio diccionario de conteos con el patrón `get(clave, 0) + 1`.
119. En la fase reduce se recorren los diccionarios parciales y se suman sus valores en un diccionario global.
120. Sin combiner, cada evento viajaría por la red como un par: 15 pares. Con combiner, cada nodo envía un par por cada clave distinta: 8 pares.
121. Esta técnica funciona porque la suma es asociativa: da igual sumar por partes que todo junto. Las operaciones `reduceByKey` de Spark aplican esta optimización automáticamente.
**Salida esperada**

```plain text
Conteo local: {'click': 2, 'view': 3, 'compra': 1}
Conteo local: {'view': 3, 'click': 1}
Conteo local: {'click': 3, 'compra': 1, 'view': 1}
Conteo global: {'click': 6, 'view': 7, 'compra': 2}
Pares enviados por la red sin combiner: 15
Pares enviados por la red con combiner: 8
```

### Ejercicios propuestos

```python
tienda_region = {
    "MAD01": "Centro", "MAD02": "Centro", "BCN01": "Noreste",
    "VLC01": "Levante", "BCN02": "Noreste", "TOL01": "Centro",
}
```

Construye un diccionario `region → lista de tiendas` y muéstralo con las regiones y las tiendas ordenadas alfabéticamente. Pista: el método `setdefault()` permite crear la lista vacía la primera vez que aparece una región.

**Salida esperada**

```plain text
Centro -> ['MAD01', 'MAD02', 'TOL01']
Levante -> ['VLC01']
Noreste -> ['BCN01', 'BCN02']
```

```python
ventas = [
    ("2026-09-15", "MAD01", 200.0),
    ("2026-09-28", "BCN02", 150.0),
    ("2026-10-02", "MAD01", 320.0),
    ("2026-09-30", "MAD01", 80.0),
    ("2026-10-05", "BCN02", 410.0),
    ("2026-10-09", "MAD01", 95.5),
]
```

Usa como clave una tupla `(tienda, mes)`, donde el mes son los 7 primeros caracteres de la fecha. Muestra los totales ordenados por tienda y mes, con dos decimales.

**Salida esperada**

```plain text
BCN02 2026-09: 150.00
BCN02 2026-10: 410.00
MAD01 2026-09: 280.00
MAD01 2026-10: 415.50
```

Un informe de calidad de datos necesita saber cuántos valores faltan en cada campo. Un valor cuenta como nulo si es `None`, si es una cadena vacía o si la clave no aparece en el registro.

```python
registros = [
    {"id": 1, "email": "ana@acme.com", "ciudad": "Madrid", "edad": 34},
    {"id": 2, "email": None, "ciudad": "Sevilla", "edad": None},
    {"id": 3, "email": "luis@globex.com", "edad": 41},
    {"id": 4, "email": "", "ciudad": None, "edad": 29},
]
campos = ["id", "email", "ciudad", "edad"]
```

**Salida esperada**

```plain text
id      0 nulos (0 %)
email   2 nulos (50 %)
ciudad  2 nulos (50 %)
edad    1 nulos (25 %)
```

```python
lineas = [
    ("teclado", 3), ("raton", 10), ("monitor", 2), ("raton", 5),
    ("cable_hdmi", 8), ("teclado", 4), ("monitor", 1), ("cable_hdmi", 9),
]
```

Suma las unidades por producto. Después ordena los pares de `items()` por unidades, de mayor a menor, con `itemgetter(1)`, y muestra el top 3 numerado.

**Salida esperada**

```plain text
{'teclado': 7, 'raton': 15, 'monitor': 3, 'cable_hdmi': 17}
1. cable_hdmi (17 uds)
2. raton (15 uds)
3. teclado (7 uds)
```

Un pipeline combina tres niveles de configuración. El nivel `ejecucion` tiene prioridad sobre `entorno`, y `entorno` sobre `defaults`.

```python
defaults = {"formato": "csv", "separador": ",", "reintentos": 3, "timeout_s": 60}
entorno = {"formato": "parquet", "timeout_s": 120}
ejecucion = {"reintentos": 5}
```

Fusiona los tres niveles en un solo diccionario. Después indica, para cada clave, de qué nivel procede su valor final.

**Salida esperada**

```plain text
{'formato': 'parquet', 'separador': ',', 'reintentos': 5, 'timeout_s': 120}
formato    = parquet  (viene de entorno)
separador  = ,        (viene de defaults)
reintentos = 5        (viene de ejecucion)
timeout_s  = 120      (viene de entorno)
```

Varios equipos leen el mismo topic, cada uno con su propio grupo de consumidores. Hay que revisar cuál va retrasado.

```python
# Último offset escrito en cada partición del topic "pedidos"
ultimo_offset = {0: 15230, 1: 14980, 2: 15410, 3: 9800}

# Último offset confirmado por cada grupo de consumidores
grupos = {
    "etl_lakehouse": {0: 15230, 1: 14975, 2: 15400, 3: 9800},
    "alertas_fraude": {0: 13900, 1: 14980, 2: 12010, 3: 9790},
    "dashboard_rt": {0: 15229, 1: 14980, 2: 15410},
}
umbral_alerta = 1000
```

Para cada grupo:

122. Construye un diccionario `partición → lag`. Si un grupo no tiene offset confirmado para una partición, considera que no ha leído nada (offset 0).
123. Calcula el lag total y la partición más retrasada. Pista: `max(lags, key=lags.get)` devuelve la clave con el valor más alto.
124. Marca el grupo como `ALERTA` si su lag total supera el umbral, o como `OK` en caso contrario.
**Salida esperada**

```plain text
etl_lakehouse   lag    15 | peor partición 2 (10) | OK
alertas_fraude  lag  4740 | peor partición 2 (3400) | ALERTA
dashboard_rt    lag  9801 | peor partición 3 (9800) | ALERTA
```

Cuando una tabla es muy pequeña, Spark envía una copia completa a cada nodo (*broadcast*). Así cada partición de la tabla grande se cruza localmente, sin mover sus datos por la red.

```python
# Tabla pequeña: se envía completa a todos los nodos (broadcast)
paises = {"ES": "Europa", "PT": "Europa", "MX": "América", "AR": "América", "JP": "Asia"}

# Tabla grande, ya repartida en 3 particiones: (id_venta, país, importe)
particiones = [
    [("V1", "ES", 120.0), ("V2", "MX", 80.0), ("V3", "ES", 45.5)],
    [("V4", "JP", 300.0), ("V5", "AR", 60.0), ("V6", "FR", 99.0)],
    [("V7", "PT", 210.0), ("V8", "MX", 150.0)],
]
```

125. Para cada partición, enriquece cada venta con su continente usando el diccionario `paises`. Los países que no estén en él se asignan a `"Desconocido"`.
126. Calcula en cada partición un diccionario local `continente → importe` y muéstralo.
127. Fusiona los resultados locales en un total global y muéstralo ordenado por continente, con dos decimales.
**Salida esperada**

```plain text
Partición 0: {'Europa': 165.5, 'América': 80.0}
Partición 1: {'Asia': 300.0, 'América': 60.0, 'Desconocido': 99.0}
Partición 2: {'Europa': 210.0, 'América': 150.0}
Ventas por continente:
  América       290.00
  Asia          300.00
  Desconocido    99.00
  Europa        375.50
```

**Página relacionada:** 🚵🏽Sprint_2-Comprehensions
**Tema:** comprehensions

---

## Sección 5. Comprehensions

Una *comprehension* construye una lista, un `set` o un diccionario en una sola expresión, a partir de otra colección. Su forma general es:

```python
[expresion for elemento in coleccion if condicion]           # lista
{expresion for elemento in coleccion if condicion}           # set
{clave: valor for elemento in coleccion if condicion}        # diccionario
```

Es la forma habitual de escribir en Python las operaciones de **transformar** (`SELECT`) y **filtrar** (`WHERE`) una colección.

### Ejemplos resueltos

```python
crudos = ["12.5", "8", "", "N/A", "103.75", "  44 ", "-3"]

limpios = [v.strip() for v in crudos]
print(limpios)

numericos = [float(v) for v in limpios if v.replace(".", "", 1).isdigit()]
print(numericos)
```

**Paso a paso**

1. La primera comprehension solo **transforma**: aplica `strip()` a cada valor.
2. La segunda **filtra y transforma**: se queda con los textos que representan un número y los convierte a `float`.
3. `v.replace(".", "", 1).isdigit()` quita el primer punto decimal y comprueba que lo que queda son dígitos. Por eso `"-3"` se descarta: en este ejemplo, un importe negativo no es válido.
4. La segunda comprehension equivale a este bucle:
```python
numericos = []
for v in limpios:
    if v.replace(".", "", 1).isdigit():
        numericos.append(float(v))
```

**Salida esperada**

```plain text
['12.5', '8', '', 'N/A', '103.75', '44', '-3']
[12.5, 8.0, 103.75, 44.0]
```

```python
latencias = [120, 85, 430, 95, 1500, 88]

estado = ["LENTA" if ms > 300 else "OK" for ms in latencias]
print(estado)
print(list(zip(latencias, estado)))
print("Lentas:", estado.count("LENTA"))
```

**Paso a paso**

5. Cuando la condición sirve para **elegir el valor**, el `if-else` va **antes** del `for`.
6. Cuando la condición sirve para **filtrar elementos**, el `if` va **después** del `for` (Ejemplo 5.1).
**Salida esperada**

```plain text
['OK', 'OK', 'LENTA', 'OK', 'LENTA', 'OK']
[(120, 'OK'), (85, 'OK'), (430, 'LENTA'), (95, 'OK'), (1500, 'LENTA'), (88, 'OK')]
Lentas: 2
```

```python
lotes = [[101, 102, 103], [104], [], [105, 106]]

registros = [r for lote in lotes for r in lote]
print(registros)
print("Total registros:", len(registros))
```

**Paso a paso**

Los `for` se leen en el mismo orden que si fueran bucles anidados: el primero es el exterior.

```python
registros = []
for lote in lotes:          # primer for: bucle exterior
    for r in lote:          # segundo for: bucle interior
        registros.append(r)
```

**Salida esperada**

```plain text
[101, 102, 103, 104, 105, 106]
Total registros: 6
```

```python
emails = ["ana@Acme.com", "luis@globex.com ", "marta@acme.com",
          "pablo@Initech.io", "eva@globex.com"]

dominios = {e.strip().lower().split("@")[1] for e in emails}
print(sorted(dominios))
print("Empresas distintas:", len(dominios))
```

**Paso a paso**

7. Para cada email: se quitan los espacios, se pasa a minúsculas y se toma lo que hay después de `@`.
8. Al usar llaves `{}`, el resultado es un `set`. `Acme.com` y `acme.com` quedan unificados tras normalizarlos.
**Salida esperada**

```plain text
['acme.com', 'globex.com', 'initech.io']
Empresas distintas: 3
```

```python
stock = {"SKU-001": 120, "SKU-002": 0, "SKU-003": 15, "SKU-004": 0, "SKU-005": 340}

agotados = {sku: uds for sku, uds in stock.items() if uds == 0}
print(agotados)

estado = {sku: ("REPONER" if uds < 20 else "OK") for sku, uds in stock.items()}
print(estado)

columnas = ["ID Pedido", "Fecha Alta", "Importe Total"]
renombrar = {c: c.lower().replace(" ", "_") for c in columnas}
print(renombrar)
```

**Paso a paso**

9. `stock.items()` entrega los pares `(clave, valor)`, que se desempaquetan en `sku, uds`.
10. El primer diccionario **filtra**, el segundo **transforma los valores** y el tercero crea un mapeo para **renombrar columnas** a *snake_case*.
**Salida esperada**

```plain text
{'SKU-002': 0, 'SKU-004': 0}
{'SKU-001': 'OK', 'SKU-002': 'REPONER', 'SKU-003': 'REPONER', 'SKU-004': 'REPONER', 'SKU-005': 'OK'}
{'ID Pedido': 'id_pedido', 'Fecha Alta': 'fecha_alta', 'Importe Total': 'importe_total'}
```

Las transformaciones básicas de Spark (`map`, `filter`, `flatMap`, `distinct`) tienen un equivalente directo en las comprehensions. Vamos a procesar un micro-lote de logs con el formato `timestamp|usuario|nivel|etiquetas`.

```python
lineas = [
    "2026-10-15T08:00:01|u17|ERROR|pago,timeout",
    "2026-10-15T08:00:02|u03|INFO|login",
    "2026-10-15T08:00:04|u17|ERROR|pago,tarjeta",
    "2026-10-15T08:00:05|u21|WARN|latencia",
    "2026-10-15T08:00:07|u08|ERROR|stock",
]

# map: cada línea se convierte en una lista de campos
registros = [linea.split("|") for linea in lineas]
print("Registros:", len(registros))
print("Primero:", registros[0])

# filter: solo los errores
errores = [r for r in registros if r[2] == "ERROR"]
print("Errores:", len(errores))

# flatMap: cada error produce tantas etiquetas como tenga
etiquetas = [etiqueta for r in errores for etiqueta in r[3].split(",")]
print("Etiquetas:", etiquetas)

# distinct: usuarios afectados
usuarios = {r[1] for r in errores}
print("Usuarios con errores:", sorted(usuarios))
```

**Paso a paso**

| Paso | Comprehension | Equivalente en Spark |
| --- | --- | --- |
| Convertir cada elemento | `[f(x) for x in datos]` | `map` |
| Quedarse con algunos elementos | `[x for x in datos if cond]` | `filter` |
| Un elemento produce varios | `[y for x in datos for y in f(x)]` | `flatMap` |
| Eliminar repetidos | `{x for x in datos}` | `distinct` |

11. `registros` aplica `split("|")` a cada línea: de 5 textos pasamos a 5 listas de campos.
12. `errores` filtra por el campo 2 (nivel) y se queda con 3 registros.
13. `etiquetas` usa dos `for`: el primero recorre los errores y el segundo recorre las etiquetas de cada error. De 3 registros salen 5 etiquetas.
14. La comprehension de `set` devuelve los usuarios sin repetir: `u17` tuvo dos errores, pero aparece una vez.
**Salida esperada**

```plain text
Registros: 5
Primero: ['2026-10-15T08:00:01', 'u17', 'ERROR', 'pago,timeout']
Errores: 3
Etiquetas: ['pago', 'timeout', 'pago', 'tarjeta', 'stock']
Usuarios con errores: ['u08', 'u17']
```

En el particionado por hash, cada clave va a la partición `hash(clave) % num_particiones`. Una comprehension de diccionario con otra de lista dentro construye todas las particiones de una vez.

```python
import zlib

num_particiones = 3
claves = ["C001", "C002", "C003", "C004", "C005", "C001", "C006", "C003", "C007", "C001"]

# Una lista de claves por partición
particiones = {
    p: [c for c in claves if zlib.crc32(c.encode()) % num_particiones == p]
    for p in range(num_particiones)
}
for p, contenido in particiones.items():
    print(f"Partición{p}:{contenido}")

tamanos = {p: len(contenido) for p, contenido in particiones.items()}
print("Tamaños:", tamanos)

media = len(claves) / num_particiones
print("Factor de skew:", round(max(tamanos.values()) / media, 2))
```

**Paso a paso**

15. La comprehension exterior crea una clave por partición (`for p in range(num_particiones)`).
16. Para cada partición, la comprehension interior recorre todas las claves y se queda con las que tienen ese resto. `zlib.crc32()` da siempre el mismo número para la misma clave, así que `C001` cae siempre en la misma partición.
17. Una comprehension se puede escribir en varias líneas para que sea más legible: Python sabe que continúa hasta que se cierran las llaves.
18. `tamanos` se construye con otra comprehension sobre `particiones.items()`, y con ella se calcula el factor de skew (máximo / media).
19. Esta versión recorre la lista de claves una vez por partición. Con millones de claves sería más eficiente un bucle que recorra la lista una sola vez; la comprehension aporta claridad, pero no siempre es la opción más rápida.
**Salida esperada**

```plain text
Partición 0: ['C003', 'C003', 'C007']
Partición 1: ['C002', 'C005', 'C006']
Partición 2: ['C001', 'C004', 'C001', 'C001']
Tamaños: {0: 3, 1: 3, 2: 4}
Factor de skew: 1.2
```

### Ejercicios propuestos

```python
registros = [
    {"id": 1, "cliente": "Acme", "importe": 250.0, "canal": "web"},
    {"id": 2, "cliente": "Globex", "importe": 80.0, "canal": "tienda"},
    {"id": 3, "cliente": "Initech", "importe": 1200.0, "canal": "web"},
    {"id": 4, "cliente": "Umbrella", "importe": 99.99, "canal": "app"},
    {"id": 5, "cliente": "Acme", "importe": 430.0, "canal": "app"},
]
```

Con comprehensions:

20. Crea una lista de diccionarios que contenga solo `id` e `importe` de los registros con importe mayor que 100.
21. Calcula el total de esos importes.
22. Obtén el `set` de canales distintos y muéstralo ordenado.
**Salida esperada**

```plain text
[{'id': 1, 'importe': 250.0}, {'id': 3, 'importe': 1200.0}, {'id': 5, 'importe': 430.0}]
Total: 1880.0
Canales: ['app', 'tienda', 'web']
```

```python
# 3 filas x 4 columnas (id, unidades, precio, descuento)
filas = [
    [1, 10, 2.5, 0.0],
    [2, 4, 10.0, 0.1],
    [3, 7, 3.2, 0.0],
]
```

Con una comprehension anidada, construye `columnas`: una lista con una lista por cada columna. No uses `zip()`. Después suma las unidades, que son la columna 1.

**Salida esperada**

```plain text
[[1, 2, 3], [10, 4, 7], [2.5, 10.0, 3.2], [0.0, 0.1, 0.0]]
Unidades totales: 21
```

```python
muestra = {"id": 1043, "cliente": "Acme", "importe": 99.9, "activo": True,
           "fecha": "2026-10-01", "descuento": None}
```

Con una comprehension de diccionario, construye `columna → nombre del tipo` (usa `type(v).__name__`). Después obtén con una comprehension de lista las columnas cuyo tipo no se puede inferir porque el valor es `None`.

**Salida esperada**

```plain text
{'id': 'int', 'cliente': 'str', 'importe': 'float', 'activo': 'bool', 'fecha': 'str', 'descuento': 'NoneType'}
No se puede inferir: ['descuento']
```

```python
anio = 2026
meses = [9, 10]
dias = [1, 2, 3]
```

Con una sola comprehension, genera las rutas con el formato `ventas/year=AAAA/month=MM/day=DD`. El mes y el día deben tener dos dígitos: usa `{m:02d}` en un f-string.

**Salida esperada**

```plain text
ventas/year=2026/month=09/day=01
ventas/year=2026/month=09/day=02
ventas/year=2026/month=09/day=03
ventas/year=2026/month=10/day=01
ventas/year=2026/month=10/day=02
ventas/year=2026/month=10/day=03
Particiones: 6
```

Este código guarda la duración de los jobs que no terminaron en error:

```python
lineas = ["job_a;120;OK", "job_b;45;ERROR", "job_c;300;OK", "job_d;12;OK"]

resultado = {}
for linea in lineas:
    partes = linea.split(";")
    if partes[2] != "ERROR":
        resultado[partes[0]] = int(partes[1])
print(resultado)
```

Reescríbelo como **una sola comprehension de diccionario**. Pista: puedes llamar varias veces a `linea.split(";")`.

**Salida esperada**

```plain text
{'job_a': 120, 'job_c': 300, 'job_d': 12}
```

En un data lake con particionado estilo Hive, la ruta de cada fichero contiene los valores de las columnas de partición con el formato `columna=valor`.

```python
rutas = [
    "ventas/year=2026/month=09/day=30/part-0000.parquet",
    "ventas/year=2026/month=10/day=01/part-0000.parquet",
    "ventas/year=2026/month=10/day=01/part-0001.parquet",
    "ventas/year=2026/month=10/day=02/part-0000.parquet",
    "ventas/year=2026/month=10/day=02/part-0001.parquet",
    "ventas/year=2026/month=10/day=02/part-0002.parquet",
]
```

Resuelve cada paso con comprehensions:

23. Construye `metadatos`: una lista con un diccionario por ruta que contenga solo los segmentos `columna=valor`. Muestra el primero y el último.
24. Obtén las particiones distintas como tuplas `(mes, día)`, ordenadas.
25. Cuenta cuántos ficheros pertenecen al mes `"10"`. Pista: `zip(rutas, metadatos)` recorre las dos listas a la vez.
26. Construye un diccionario `"mes-día" → número de ficheros`.
**Salida esperada**

```plain text
{'year': '2026', 'month': '09', 'day': '30'}
{'year': '2026', 'month': '10', 'day': '02'}
Particiones distintas: [('09', '30'), ('10', '01'), ('10', '02')]
Ficheros de octubre: 5 de 6
Ficheros por partición: {'09-30': 1, '10-01': 2, '10-02': 3}
```

La función `explode()` de Spark convierte un registro con un array en varias filas, una por cada elemento del array. Vamos a simularla.

```python
pedidos = [
    {"id": "P1", "lineas": [{"sku": "A1", "uds": 2}, {"sku": "B7", "uds": 1}]},
    {"id": "P2", "lineas": [{"sku": "A1", "uds": 5}]},
    {"id": "P3", "lineas": []},
    {"id": "P4", "lineas": [{"sku": "C3", "uds": 4}, {"sku": "B7", "uds": 3}, {"sku": "A1", "uds": 1}]},
]
```

27. Con una comprehension de dos `for`, genera una fila `(id_pedido, sku, uds)` por cada línea de pedido y muéstralas.
28. Comprueba qué pedidos desaparecen en el *explode* porque no tienen líneas.
29. Con una comprehension de `set`, obtén los SKU distintos. Después, con una comprehension de diccionario, calcula las unidades totales de cada SKU.
**Salida esperada**

```plain text
('P1', 'A1', 2)
('P1', 'B7', 1)
('P2', 'A1', 5)
('P4', 'C3', 4)
('P4', 'B7', 3)
('P4', 'A1', 1)
Filas tras el explode: 6
Pedidos sin líneas: ['P3']
Unidades por SKU: {'A1': 8, 'B7': 4, 'C3': 4}
```

**Página relacionada:** 🚴🏽‍♂️Sprint_3-Funciones
**Tema:** funciones

---

## Sección 6. Funciones

Una función agrupa un bloque de código con un nombre para poder **reutilizarlo**. Python trae muchas funciones integradas (`len()`, `sorted()`, `max()`…) y permite definir las nuestras con `def`. En ingeniería de datos, cada paso de un pipeline (leer, validar, transformar, cargar) suele ser una función. Así cada paso se puede probar por separado y combinar con otros.

```python
def nombre_funcion(parametros):
    """Docstring: qué hace la función."""
    # cuerpo de la función
    return resultado
```

Por convención, los nombres de función van en minúsculas y con las palabras separadas por `_`: `calcular_kpis()`, `validar_registro()`.

### Ejemplos resueltos

```python
def resumen_lote(valores):
    """Devuelve (n, mínimo, máximo, media) de una lista de números."""
    n = len(valores)
    media = sum(valores) / n
    return n, min(valores), max(valores), round(media, 2)

lote_lunes = [120.0, 85.5, 430.0, 95.25]
lote_martes = [300.0, 310.5]

print(resumen_lote(lote_lunes))
n, minimo, maximo, media = resumen_lote(lote_martes)
print(f"Martes:{n} registros, media{media}")
```

**Paso a paso**

1. El texto entre triples comillas es el *docstring*: documenta qué hace la función.
2. `return n, min(...), ...` devuelve **una tupla**. El que llama puede guardarla entera o desempaquetarla en variables.
3. La misma función sirve para cualquier lote. Esa es la ventaja de definirla una vez.
**Salida esperada**

```plain text
(4, 85.5, 430.0, 182.69)
Martes: 2 registros, media 305.25
```

```python
def normalizar(texto, minusculas=True, quitar_espacios=True, relleno=""):
    if texto is None or texto.strip() == "":
        return relleno
    if quitar_espacios:
        texto = texto.strip()
    if minusculas:
        texto = texto.lower()
    return texto

print(repr(normalizar("  Madrid ")))
print(repr(normalizar("  Madrid ", minusculas=False)))
print(repr(normalizar("", relleno="SIN_DATO")))
print(repr(normalizar(None, relleno="SIN_DATO")))
print(repr(normalizar("  BCN  ", quitar_espacios=False)))
```

**Paso a paso**

4. `minusculas`, `quitar_espacios` y `relleno` tienen **valor por defecto**. Si no se pasan, se usan esos valores.
5. Al llamar con `minusculas=False` (**argumento por nombre**) cambiamos solo esa opción, sin tener que pasar las anteriores en orden.
6. En la definición, los parámetros con valor por defecto deben ir **después** de los obligatorios.
7. `repr()` muestra el texto entre comillas, así que se ven los espacios que quedan.
**Salida esperada**

```plain text
'madrid'
'Madrid'
'SIN_DATO'
'SIN_DATO'
'  bcn  '
```

```python
def publicar_metrica(nombre, valor, unidad):
    print(f"métrica={nombre} valor={valor} unidad={unidad}")

publicar_metrica("filas_cargadas", 15230, "filas")            # por posición
publicar_metrica(valor=42.7, unidad="s", nombre="duracion")   # por nombre: el orden da igual
publicar_metrica("errores", unidad="filas", valor=3)          # mezcla: posicionales primero

# publicar_metrica(nombre="errores", 3, "filas")
#   -> SyntaxError: positional argument follows keyword argument
# publicar_metrica("errores", 3)
#   -> TypeError: publicar_metrica() missing 1 required positional argument: 'unidad'

resultado = publicar_metrica("latencia_p95", 230, "ms")
print("La función devuelve:", resultado)

def formato_fecha():
    return "AAAA-MM-DD"

def formato_fecha():           # redefinición: sustituye a la anterior
    return "DD/MM/AAAA"

print(formato_fecha())
```

**Paso a paso**

8. **Por posición**, cada valor va al parámetro que ocupa su mismo lugar. Hay que pasar exactamente tantos valores como parámetros obligatorios tenga la función.
9. **Por nombre**, cada valor se asigna al parámetro indicado y el orden da igual.
10. Se pueden mezclar, pero los argumentos posicionales siempre van **antes** que los argumentos por nombre. Si no, Python ni siquiera ejecuta el programa (`SyntaxError`).
11. Si falta un argumento obligatorio, la llamada falla con `TypeError` e indica cuál falta.
12. Una función sin `return` **devuelve ****`None`**. `publicar_metrica()` imprime, pero no devuelve nada.
13. Si se define dos veces una función con el mismo nombre, la segunda definición sustituye a la primera. En un notebook es una fuente habitual de confusión: se ejecuta la versión de la última celda que se ejecutó.
**Salida esperada**

```plain text
métrica=filas_cargadas valor=15230 unidad=filas
métrica=duracion valor=42.7 unidad=s
métrica=errores valor=3 unidad=filas
métrica=latencia_p95 valor=230 unidad=ms
La función devuelve: None
DD/MM/AAAA
```

```python
def total_bytes(*tamanos):
    print("  recibido:", tamanos)
    return sum(tamanos)

print(total_bytes(1024, 2048))
print(total_bytes(500, 500, 500, 500))
print(total_bytes())

def log_evento(nivel, mensaje, **contexto):
    linea = f"[{nivel}]{mensaje}"
    for clave, valor in contexto.items():
        linea += f" |{clave}={valor}"
    print(linea)

log_evento("INFO", "Carga finalizada", tabla="ventas", filas=15230)
log_evento("ERROR", "Timeout en origen", origen="api_crm", intento=3, espera_s=30)
log_evento("DEBUG", "Sin contexto")
```

**Paso a paso**

14. `tamanos` recoge todos los argumentos posicionales en una **tupla**, que puede estar vacía.
15. `*contexto` recoge todos los argumentos por nombre que sobran en un **diccionario**. Es muy útil para logs, porque cada evento puede llevar un contexto distinto.
16. Los nombres `args` y `kwargs` son solo una convención: lo que importa son el  y el `*`.
**Salida esperada**

```plain text
  recibido: (1024, 2048)
3072
  recibido: (500, 500, 500, 500)
2000
  recibido: ()
0
[INFO] Carga finalizada | tabla=ventas | filas=15230
[ERROR] Timeout en origen | origen=api_crm | intento=3 | espera_s=30
[DEBUG] Sin contexto
```

Cuando una función combina todos los tipos de parámetros, deben aparecer en este orden:

| Orden | Tipo | En el ejemplo |
| --- | --- | --- |
| 1 | Posicionales obligatorios | `nombre`, `entorno` |
| 2 | Posicionales variables | `*tablas` |
| 3 | Solo por nombre | `reintentos`, `notificar` |
| 4 | Por nombre variables | `**spark_conf` |

```python
def lanzar_job(nombre, entorno, *tablas, reintentos=3, notificar=False, **spark_conf):
    print(f"Job:{nombre} [{entorno}]")
    print("  Tablas:", tablas)
    print("  Reintentos:", reintentos, "| Notificar:", notificar)
    print("  Configuración Spark:", spark_conf)

lanzar_job("carga_diaria", "prod", "ventas", "clientes", "stock",
           notificar=True, executor_memory="8g", shuffle_partitions=200)

lanzar_job("recalculo_kpis", "dev")

# Cuidado: el 5 no llega a reintentos, se queda dentro de tablas
lanzar_job("backfill", "dev", "ventas", 5)
```

**Paso a paso**

17. `nombre` y `entorno` reciben los dos primeros valores. Todos los posicionales que siguen van a la tupla `tablas`.
18. Los parámetros escritos **después** de `tablas` solo se pueden pasar **por nombre**. Por eso `notificar=True` llega a su parámetro.
19. Los argumentos por nombre que no corresponden a ningún parámetro (`executor_memory`, `shuffle_partitions`) van al diccionario `spark_conf`.
20. En la última llamada, el `5` no llega a `reintentos`: al ser posicional, se queda dentro de `tablas`. Para cambiar los reintentos hay que escribir `reintentos=5`.
**Salida esperada**

```plain text
Job: carga_diaria [prod]
  Tablas: ('ventas', 'clientes', 'stock')
  Reintentos: 3 | Notificar: True
  Configuración Spark: {'executor_memory': '8g', 'shuffle_partitions': 200}
Job: recalculo_kpis [dev]
  Tablas: ()
  Reintentos: 3 | Notificar: False
  Configuración Spark: {}
Job: backfill [dev]
  Tablas: ('ventas', 5)
  Reintentos: 3 | Notificar: False
  Configuración Spark: {}
```

Es habitual que los parámetros de conexión estén en un fichero de configuración. Al leerlo, llegan a Python como tupla o como diccionario.

```python
def cadena_conexion(host, puerto, base_datos, usuario="lector"):
    return f"{usuario}@{host}:{puerto}/{base_datos}"

params_tupla = ("db-prod.interno", 5432, "ventas")
print(cadena_conexion(*params_tupla))

params_dict = {"host": "db-dev.interno", "puerto": 5433,
               "base_datos": "pruebas", "usuario": "etl"}
print(cadena_conexion(**params_dict))

print(cadena_conexion("db-qa.interno", 5432, base_datos="qa"))

# Sin ** se desempaquetan las CLAVES del diccionario, no los valores
print(cadena_conexion(*params_dict))
```

**Paso a paso**

21. `params_tupla` reparte los elementos de la tupla como argumentos posicionales, en orden.
22. `*params_dict` reparte el diccionario como argumentos por nombre. Las claves deben coincidir con los nombres de los parámetros.
23. En la tercera llamada se mezclan argumentos posicionales y por nombre. Los posicionales siempre van primero.
24. Con un solo , un diccionario reparte sus **claves** como argumentos posicionales. Por eso la última línea muestra los nombres de los campos en lugar de sus valores.
**Salida esperada**

```plain text
lector@db-prod.interno:5432/ventas
etl@db-dev.interno:5433/pruebas
lector@db-qa.interno:5432/qa
usuario@host:puerto/base_datos
```

```python
def registrar_error_mal(error, errores=[]):
    errores.append(error)
    return errores

print(registrar_error_mal("fila 3: importe vacío"))
print(registrar_error_mal("fila 9: fecha inválida"))   # ¡arrastra el error anterior!

def registrar_error(error, errores=None):
    if errores is None:
        errores = []
    errores.append(error)
    return errores

print(registrar_error("fila 3: importe vacío"))
print(registrar_error("fila 9: fecha inválida"))
```

**Paso a paso**

25. El valor por defecto de un parámetro se crea **una sola vez**, al definir la función, no en cada llamada.
26. Por eso, en la versión incorrecta, todas las llamadas comparten la misma lista `errores`, y el error de la fila 3 aparece en la segunda llamada.
27. La forma correcta es usar `None` como valor por defecto y crear la lista dentro de la función.
**Salida esperada**

```plain text
['fila 3: importe vacío']
['fila 3: importe vacío', 'fila 9: fecha inválida']
['fila 3: importe vacío']
['fila 9: fecha inválida']
```

Una función se puede definir dentro de otra. La función interna solo existe mientras se ejecuta la externa, así que queda protegida: nadie puede llamarla desde fuera por error.

```python
def validar_lote(registros, importe_maximo=10000):
    def es_importe_valido(importe):        # función interna
        return importe is not None and 0 <= importe <= importe_maximo

    validos = 0
    for r in registros:
        if es_importe_valido(r["importe"]):
            validos += 1
    return validos, len(registros) - validos

lote = [
    {"id": 1, "importe": 250.0},
    {"id": 2, "importe": -4.0},
    {"id": 3, "importe": None},
    {"id": 4, "importe": 12500.0},
    {"id": 5, "importe": 99.9},
]
print("(válidos, rechazados):", validar_lote(lote))
print("Con máximo 20000:     ", validar_lote(lote, importe_maximo=20000))

# es_importe_valido(100)  -> NameError: name 'es_importe_valido' is not defined
```

**Paso a paso**

28. `es_importe_valido()` se define dentro de `validar_lote()`. Fuera de ella no existe: intentar llamarla produce `NameError`.
29. La función interna puede **leer** las variables de la externa. Usa `importe_maximo` sin recibirlo como parámetro.
30. Por eso, al llamar a `validar_lote()` con `importe_maximo=20000`, la regla interna cambia y el importe de 12500 pasa a ser válido.
31. Es una buena opción para reglas auxiliares que solo tienen sentido dentro de un proceso concreto.
**Salida esperada**

```plain text
(válidos, rechazados): (2, 3)
Con máximo 20000:      (3, 2)
```

```python
def leer():
    return ["  101;Acme;1200 ", "102;Globex;", "103;initech;300", "  104;ACME;50"]

def parsear(linea):
    id_, cliente, importe = linea.strip().split(";")
    return {
        "id": int(id_),
        "cliente": cliente.title(),
        "importe": float(importe) if importe else None,
    }

def es_valido(registro):
    return registro["importe"] is not None

def ejecutar_pipeline():
    validos, rechazados = [], []
    for linea in leer():
        registro = parsear(linea)
        if es_valido(registro):
            validos.append(registro)
        else:
            rechazados.append(registro)
    return validos, rechazados

ok, ko = ejecutar_pipeline()
print("Válidos:", ok)
print("Rechazados:", ko)
print("Total cargado:", sum([r["importe"] for r in ok]))
```

**Paso a paso**

32. Cada función hace **una sola cosa**: `leer()` obtiene los datos, `parsear()` convierte una línea en un diccionario, `es_valido()` aplica una regla de calidad y `ejecutar_pipeline()` coordina el proceso.
33. `parsear()` normaliza el nombre del cliente con `title()`, de modo que `ACME` e `initech` pasan a `Acme` e `Initech`.
34. Las filas sin importe no se pierden: se apartan en `rechazados`, igual que una tabla de cuarentena.
**Salida esperada**

```plain text
Válidos: [{'id': 101, 'cliente': 'Acme', 'importe': 1200.0}, {'id': 103, 'cliente': 'Initech', 'importe': 300.0}, {'id': 104, 'cliente': 'Acme', 'importe': 50.0}]
Rechazados: [{'id': 102, 'cliente': 'Globex', 'importe': None}]
Total cargado: 1550.0
```

En Hadoop, el programador solo escribe dos funciones, la de **map** y la de **reduce**, y el framework se encarga de ejecutarlas en paralelo y de mover los datos entre ellas. Vamos a simular un conteo de las palabras más buscadas en una tienda online.

```python
def mapear(linea):
    """Fase map: emite (palabra, 1) por cada palabra buscada."""
    usuario, termino = linea.split(";")
    return [(palabra.lower(), 1) for palabra in termino.split()]

def agrupar(pares):
    """Fase shuffle: reúne todos los valores de cada clave."""
    grupos = {}
    for clave, valor in pares:
        grupos.setdefault(clave, []).append(valor)
    return grupos

def reducir(clave, valores):
    """Fase reduce: combina los valores de una clave."""
    return clave, sum(valores)

def ejecutar_job(lineas):
    pares = []
    for linea in lineas:
        pares.extend(mapear(linea))
    grupos = agrupar(pares)
    return [reducir(clave, valores) for clave, valores in grupos.items()]

busquedas = [
    "u1;Zapatillas running", "u2;zapatillas mujer", "u3;Camiseta running",
    "u1;zapatillas trail", "u4;mochila trail", "u5;Running",
]

print("Map de una línea:", mapear(busquedas[0]))
resultado = ejecutar_job(busquedas)
from operator import itemgetter
ordenado = sorted(sorted(resultado), key=itemgetter(1), reverse=True)
for palabra, total in ordenado:
    print(f"{palabra:<11}{total}")
```

**Paso a paso**

35. `mapear()` procesa **una sola línea** y devuelve sus pares `(palabra, 1)`. Como no depende de ninguna otra línea, el framework puede ejecutarla en muchos nodos a la vez.
36. `agrupar()` simula el **shuffle**: reúne en una lista todos los valores de una misma clave.
37. `reducir()` recibe **una clave y sus valores** y los combina en un resultado.
38. `ejecutar_job()` hace el papel del framework: llama a las tres fases en orden.
39. Para ordenar por frecuencia y, en caso de empate, alfabéticamente, se ordena primero por palabra y después por total. `sorted()` respeta el orden previo de los empates.
**Salida esperada**

```plain text
Map de una línea: [('zapatillas', 1), ('running', 1)]
running     3
zapatillas  3
trail       2
camiseta    1
mochila     1
mujer       1
```

En Spark, una referencia habitual es que cada partición pese unos 128 MB. Con pocas particiones se desaprovechan núcleos del clúster; con demasiadas, se pierde tiempo gestionando tareas muy pequeñas.

```python
import math

def calcular_particiones(tamano_gb, objetivo_mb=128, nucleos=None):
    """Número de particiones para que cada una pese unos objetivo_mb."""
    particiones = math.ceil(tamano_gb * 1024 / objetivo_mb)
    if nucleos and particiones > nucleos:
        # redondear hacia arriba a un múltiplo del número de núcleos
        particiones = math.ceil(particiones / nucleos) * nucleos
    return particiones

datasets = [("logs_web", 50), ("ventas", 2.5), ("clientes", 0.04)]

for nombre, gb in datasets:
    simple = calcular_particiones(gb)
    ajustado = calcular_particiones(gb, nucleos=48)
    print(f"{nombre:<9}{gb:>5} GB ->{simple:>3} particiones | con 48 núcleos:{ajustado}")

print("logs_web con particiones de 256 MB:", calcular_particiones(50, objetivo_mb=256))
```

**Paso a paso**

40. `math.ceil()` redondea hacia arriba: si el dataset pesa 40 MB, hace falta al menos una partición.
41. `objetivo_mb` y `nucleos` tienen valores por defecto, así que la función sirve tanto para un cálculo rápido como para uno ajustado al clúster.
42. Si se indican núcleos y hay más particiones que núcleos, el resultado se redondea a un múltiplo de los núcleos. Así la última “oleada” de tareas ocupa el clúster entero: 400 particiones en 48 núcleos dejarían una última oleada a medias, y 432 no.
43. Si hay menos particiones que núcleos (`ventas`, `clientes`), no se ajusta nada: crear particiones casi vacías no aporta.
**Salida esperada**

```plain text
logs_web     50 GB -> 400 particiones | con 48 núcleos: 432
ventas      2.5 GB ->  20 particiones | con 48 núcleos: 20
clientes   0.04 GB ->   1 particiones | con 48 núcleos: 1
logs_web con particiones de 256 MB: 200
```

En las aplicaciones de IA Generativa, los prompts no se escriben a mano cada vez: se generan con funciones que combinan una plantilla fija con los datos de cada petición.

```python
def construir_prompt(tarea, contexto, *ejemplos, idioma="español",
                     formato="lista", **restricciones):
    partes = [f"Tarea:{tarea}", f"Contexto:{contexto}"]
    if ejemplos:
        partes.append("Ejemplos:")
        for i, ejemplo in enumerate(ejemplos, start=1):
            partes.append(f"{i}.{ejemplo}")
    partes.append(f"Responde en{idioma} con formato{formato}.")
    for clave, valor in restricciones.items():
        partes.append(f"-{clave.replace('_', ' ')}:{valor}")
    return "\n".join(partes)

prompt = construir_prompt(
    "Resumir las incidencias del pipeline nocturno",
    "3 jobs fallaron por timeout en la API del CRM y 1 job tardó el doble de lo habitual.",
    "job_ventas: timeout tras 3 reintentos -> revisar el límite de la API",
    longitud_maxima="5 líneas",
    tono="técnico",
)
print(prompt)
print()
print(construir_prompt("Traducir el mensaje de error", "Connection refused", idioma="inglés",
                       formato="texto plano"))
```

**Paso a paso**

44. `tarea` y `contexto` son obligatorios: sin ellos el prompt no tiene sentido.
45. `ejemplos` admite cero o más ejemplos (*few-shot*). Si no se pasa ninguno, la sección de ejemplos no aparece.
46. `idioma` y `formato` tienen un valor por defecto y solo se pueden pasar por nombre.
47. `*restricciones` admite cualquier instrucción adicional. El nombre del argumento se convierte en texto sustituyendo `_` por espacios.
48. `"\n".join(partes)` une las líneas con saltos de línea.
**Salida esperada**

```plain text
Tarea: Resumir las incidencias del pipeline nocturno
Contexto: 3 jobs fallaron por timeout en la API del CRM y 1 job tardó el doble de lo habitual.
Ejemplos:
  1. job_ventas: timeout tras 3 reintentos -> revisar el límite de la API
Responde en español con formato lista.
- longitud maxima: 5 líneas
- tono: técnico

Tarea: Traducir el mensaje de error
Contexto: Connection refused
Responde en inglés con formato texto plano.
```

En un sistema RAG (*Retrieval-Augmented Generation*), los documentos se trocean en fragmentos (*chunks*) antes de convertirlos en embeddings. Los fragmentos se **solapan** un poco para que una idea no quede partida entre dos trozos sin contexto.

```python
def dividir_en_chunks(texto, tamano=8, solape=2):
    palabras = texto.split()
    paso = tamano - solape
    chunks = []
    for inicio in range(0, len(palabras), paso):
        trozo = palabras[inicio:inicio + tamano]
        chunks.append({"id": len(chunks), "inicio": inicio, "texto": " ".join(trozo)})
        if inicio + tamano >= len(palabras):
            break
    return chunks

politica = ("La política de devoluciones permite devolver cualquier producto en un plazo "
            "de 30 días. El cliente debe conservar el ticket de compra. Los productos de "
            "higiene no admiten devolución una vez abiertos.")

print("Palabras:", len(politica.split()))
for chunk in dividir_en_chunks(politica):
    print(chunk)
print("Chunks con tamano=12 y solape=3:", len(dividir_en_chunks(politica, tamano=12, solape=3)))
```

**Paso a paso**

49. Cada chunk tiene `tamano` palabras, y el siguiente empieza `tamano - solape` palabras más adelante. Con 8 y 2, avanza de 6 en 6, así que cada chunk repite las 2 últimas palabras del anterior.
50. Cada chunk se guarda como un diccionario con su `id` y su posición de `inicio`. Esos metadatos permiten saber después de qué parte del documento salió cada fragmento.
51. El `break` detiene el bucle cuando un chunk ya llega al final del texto, para no generar un último fragmento formado solo por el solape.
52. Los sistemas reales trocean por **tokens** en lugar de palabras, pero la lógica es la misma.
**Salida esperada**

```plain text
Palabras: 32
{'id': 0, 'inicio': 0, 'texto': 'La política de devoluciones permite devolver cualquier producto'}
{'id': 1, 'inicio': 6, 'texto': 'cualquier producto en un plazo de 30 días.'}
{'id': 2, 'inicio': 12, 'texto': '30 días. El cliente debe conservar el ticket'}
{'id': 3, 'inicio': 18, 'texto': 'el ticket de compra. Los productos de higiene'}
{'id': 4, 'inicio': 24, 'texto': 'de higiene no admiten devolución una vez abiertos.'}
Chunks con tamano=12 y solape=3: 4
```

### Ejercicios propuestos

```python
registros = [
    {"id": 1, "cliente": "Acme", "importe": 120.0},
    {"id": 2, "cliente": "", "importe": 80.0},
    {"id": 3, "cliente": "Globex", "importe": -15.0},
    {"cliente": "Initech", "importe": None},
]
obligatorios = ["id", "cliente", "importe"]
```

Escribe `validar_registro(registro, obligatorios)`, que devuelva una tupla `(es_valido, lista_de_errores)`. Un campo obligatorio falla si no existe, si es `None` o si es una cadena vacía. Además, si el importe existe y es negativo, se añade el error `"importe negativo"`. Aplica la función a todos los registros y muestra `?` cuando falte el id.

**Salida esperada**

```plain text
OK  1 []
KO  2 ["falta 'cliente'"]
KO  3 ['importe negativo']
KO  ? ["falta 'id'", "falta 'importe'"]
```

Escribe `calcular_kpis(*importes, moneda="EUR", decimales=2)`, que devuelva un diccionario con `n`, `total`, `media` y `moneda`. Si no recibe importes, la media debe ser `None`. Comprueba la función con estas llamadas:

```python
print(calcular_kpis(120.5, 80.25, 99.99))
print(calcular_kpis(1000, 2500, moneda="USD", decimales=0))
print(calcular_kpis())
lote = [10.0, 20.0, 30.0]
print(calcular_kpis(*lote, moneda="MXN"))
```

**Salida esperada**

```plain text
{'n': 3, 'total': 300.74, 'media': 100.25, 'moneda': 'EUR'}
{'n': 2, 'total': 3500, 'media': 1750.0, 'moneda': 'USD'}
{'n': 0, 'total': 0, 'media': None, 'moneda': 'EUR'}
{'n': 3, 'total': 60.0, 'media': 20.0, 'moneda': 'MXN'}
```

Escribe `construir_ruta(base, *carpetas, **particiones)`. La función debe unir la base (sin `/` final), las carpetas y las particiones con el formato `clave=valor`, separando cada parte con `/`. Comprueba la función con estas llamadas:

```python
print(construir_ruta("abfss://lake@cuenta.dfs.core.windows.net/", "plata", "ventas",
                     year=2026, month="10", day="15"))
print(construir_ruta("/lake", "bronce", "clientes"))
print(construir_ruta("/lake", region="EU"))
```

**Salida esperada**

```plain text
abfss://lake@cuenta.dfs.core.windows.net/plata/ventas/year=2026/month=10/day=15
/lake/bronce/clientes
/lake/region=EU
```

```python
pedidos = [
    {"id": 1, "canal": "web", "pais": "ES"},
    {"id": 2, "canal": "app", "pais": "PT"},
    {"id": 3, "canal": "web", "pais": "ES"},
    {"id": 4, "canal": "tienda", "pais": "ES"},
    {"id": 5, "canal": "web", "pais": "MX"},
    {"id": 6, "canal": "app"},
]
```

Escribe `contar_por(registros, campo, ausente="SIN_VALOR")`, que cuente cuántos registros hay para cada valor del campo indicado. Los registros que no tengan el campo se cuentan con el valor de `ausente`. Llama a la función tres veces: por `canal`, por `pais` y por `pais` con `ausente="??"`.

**Salida esperada**

```plain text
{'web': 3, 'app': 2, 'tienda': 1}
{'ES': 3, 'PT': 1, 'MX': 1, 'SIN_VALOR': 1}
{'ES': 3, 'PT': 1, 'MX': 1, '??': 1}
```

Tienes estas líneas de un fichero de movimientos por tienda: `fecha;tienda;tipo;importe`.

```python
lineas_crudas = [
    "2026-10-15;MAD01;  ventas ;1250.0",
    "2026-10-15;BCN02;Ventas;",
    "2026-10-15;MAD01;DEVOLUCIONES;-80.0",
    "2026-10-15;VLC03;ventas;410.0",
    "2026-10-15;BCN02;ventas;980.5",
]
```

Organiza la solución en cuatro funciones:

- `parsear_linea(linea)`: devuelve un diccionario. El tipo se normaliza quitando espacios y pasando a minúsculas. Si no hay importe, se guarda `None`.
- `es_valida(registro)`: comprueba que haya importe.
- `agregar_por_tienda(registros, tipo)`: devuelve un diccionario `tienda → total` con los registros del tipo indicado.
- `informe(lineas)`: coordina el proceso y muestra cuántas líneas se han leído, cuántas son válidas y cuántas se han rechazado, seguido del total de `ventas` por tienda, ordenado.
**Salida esperada**

```plain text
Leídas: 5 | Válidas: 4 | Rechazadas: 1
  BCN02: 980.50
  MAD01: 1250.00
  VLC03: 410.00
```

Sin ejecutar el código, analiza estas seis llamadas a la misma función:

```python
def cargar_tabla(tabla, esquema, *columnas, modo="append", **opciones):
    print(f"{esquema}.{tabla} | columnas={columnas} | modo={modo} | opciones={opciones}")

# Llamada 1
cargar_tabla("ventas", "plata")
# Llamada 2
cargar_tabla("ventas", "plata", "id", "importe", modo="overwrite")
# Llamada 3
cargar_tabla(esquema="oro", tabla="kpis", particion="fecha")
# Llamada 4
cargar_tabla("ventas")
# Llamada 5
cargar_tabla("clientes", "plata", "id", "overwrite")
# Llamada 6
cargar_tabla("clientes", "plata", tabla="clientes_v2")
```

53. Indica qué llamadas producen un error y explica por qué.
54. Para las que funcionan, escribe qué imprimen.
55. Una de las llamadas que funcionan tiene un error lógico: hace algo distinto de lo que probablemente pretendía quien la escribió. Identifícala y corrígela.
Después, ejecútalas una a una para comprobar tus respuestas.

**Salida esperada** (los errores se muestran con su mensaje de `TypeError`)

```plain text
Llamada 1: plata.ventas | columnas=() | modo=append | opciones={}
Llamada 2: plata.ventas | columnas=('id', 'importe') | modo=overwrite | opciones={}
Llamada 3: oro.kpis | columnas=() | modo=append | opciones={'particion': 'fecha'}
Llamada 4: ERROR -> cargar_tabla() missing 1 required positional argument: 'esquema'
Llamada 5: plata.clientes | columnas=('id', 'overwrite') | modo=append | opciones={}
Llamada 6: ERROR -> cargar_tabla() got multiple values for argument 'tabla'
```

Un job tiene que procesar varios ficheros de tamaños muy distintos y dispone de varios ejecutores. Para equilibrar la carga se usa una regla sencilla: se ordenan los ficheros de mayor a menor y cada uno se asigna al ejecutor que **menos megas lleva acumulados** en ese momento. Si hay empate, se elige el de número más bajo.

```python
# (fichero, tamaño en MB)
ficheros = [
    ("f1.parquet", 512), ("f2.parquet", 128), ("f3.parquet", 900), ("f4.parquet", 256),
    ("f5.parquet", 640), ("f6.parquet", 300), ("f7.parquet", 128),
]
```

56. Escribe `asignar_tareas(ficheros, num_ejecutores=3)`, que devuelva un diccionario `"ejecutor_N" → {"ficheros": [...], "mb": total}`.
57. Escribe `desequilibrio(asignacion)`, que devuelva la carga máxima dividida entre la mínima, con dos decimales.
58. Muestra el reparto con 3 ejecutores y compara su desequilibrio con el de 4 ejecutores.
**Salida esperada**

```plain text
ejecutor_0:  900 MB -> ['f3.parquet']
ejecutor_1: 1024 MB -> ['f5.parquet', 'f4.parquet', 'f7.parquet']
ejecutor_2:  940 MB -> ['f1.parquet', 'f6.parquet', 'f2.parquet']
Desequilibrio (máx / mín): 1.14
Con 4 ejecutores: [900, 640, 640, 684] -> desequilibrio 1.41
```

Para lanzar un job de Spark en un clúster se usa el comando `spark-submit`, con opciones como el gestor de recursos (`--master`), el modo de despliegue o el número de ejecutores, y parámetros de configuración `--conf clave=valor`.

Escribe `construir_spark_submit(script, *args_app, master="yarn", deploy_mode="cluster", num_executors=2, **conf)`. La función devuelve el comando como texto, con este orden: opciones fijas, opciones `--conf`, script y argumentos de la aplicación. En los nombres de `conf`, los `_` se sustituyen por `.` (`spark_executor_memory` → `spark.executor.memory`). Comprueba la función con estas llamadas:

```python
print(construir_spark_submit("carga_ventas.py"))
print(construir_spark_submit("carga_ventas.py", "--fecha", "2026-10-15",
                             num_executors=10, spark_executor_memory="8g",
                             spark_sql_shuffle_partitions=400))
print(construir_spark_submit("prueba.py", master="local[4]", deploy_mode="client"))
```

**Salida esperada**

```plain text
spark-submit --master yarn --deploy-mode cluster --num-executors 2 carga_ventas.py
spark-submit --master yarn --deploy-mode cluster --num-executors 10 --conf spark.executor.memory=8g --conf spark.sql.shuffle.partitions=400 carga_ventas.py --fecha 2026-10-15
spark-submit --master local[4] --deploy-mode client --num-executors 2 prueba.py
```

Los proveedores de modelos de lenguaje cobran por **tokens**, con un precio distinto para los de entrada (el prompt) y los de salida (la respuesta).

```python
# USD por millón de tokens (entrada, salida). Tarifas ficticias para el ejercicio.
TARIFAS = {"modelo_ligero": (0.25, 1.25), "modelo_avanzado": (3.00, 15.00)}

# (modelo, tokens de entrada, tokens de salida) de las peticiones de un día
peticiones = [
    ("modelo_ligero", 1200, 300),
    ("modelo_avanzado", 5400, 900),
    ("modelo_ligero", 800, 150),
    ("modelo_avanzado", 12000, 2000),
    ("modelo_ligero", 2500, 600),
]
```

59. Escribe `coste_peticion(modelo, tokens_entrada, tokens_salida)`, que devuelva el coste en USD de una petición. Pruébala con la primera petición desempaquetando la tupla con .
60. Escribe `resumen_costes(peticiones, presupuesto_diario=0.05)`, que devuelva tres valores: un diccionario por modelo con su número de peticiones y su coste, el coste total y si se ha superado el presupuesto.
61. Escribe `proyectar(coste_diario, dias=30)`, que devuelva el coste estimado para ese número de días, con dos decimales.
**Salida esperada**

```plain text
Coste de la primera petición: 0.000675 USD
modelo_ligero    3 peticiones -> 0.002437 USD
modelo_avanzado  2 peticiones -> 0.095700 USD
Total del día: 0.098138 USD | ¿Supera el presupuesto? True
Proyección a 30 días: 2.94 USD
Proyección a 365 días: 35.82 USD
```

Un LLM clasifica los tickets de soporte y devuelve un diccionario por ticket. Antes de guardar sus respuestas en la base de datos hay que comprobarlas: los modelos a veces omiten campos o inventan valores que no existen.

```python
CATEGORIAS = {"facturacion", "tecnico", "envio"}
PRIORIDADES = {"baja", "media", "alta"}

# Respuestas de un LLM que clasifica tickets de soporte
respuestas = [
    {"ticket": "T1", "categoria": "envio", "prioridad": "alta", "resumen": "Pedido no entregado"},
    {"ticket": "T2", "categoria": "Facturación", "prioridad": "media", "resumen": "Cobro duplicado"},
    {"ticket": "T3", "categoria": "tecnico", "resumen": "Error al iniciar sesión"},
    {"ticket": "T4", "categoria": "tecnico", "prioridad": "urgente", "resumen": ""},
    {"ticket": "T5", "categoria": "facturacion", "prioridad": "baja", "resumen": "Pide factura"},
]
```

62. Escribe `validar_respuesta(respuesta, obligatorios=("categoria", "prioridad", "resumen"), **permitidos)`, que devuelva `(es_valida, errores)`. Un campo obligatorio falla si no existe o está vacío. Cada argumento de `permitidos` indica los valores aceptados de un campo, como `categoria=CATEGORIAS`.
63. Escribe `procesar_respuestas(respuestas, **permitidos)`, que separe los tickets aceptados de los que necesitan revisión humana. Debe pasar `permitidos` a `validar_respuesta()` con `*`.
64. Muestra los resultados y la tasa de respuestas válidas.
**Salida esperada**

```plain text
Aceptadas: ['T1', 'T5']
A revisión humana:
  T2: ["'categoria' no válido: Facturación"]
  T3: ["falta 'prioridad'"]
  T4: ["falta 'resumen'", "'prioridad' no válido: urgente"]
Tasa de respuestas válidas: 40 %
```

**Página relacionada:** 🚴🏽‍♀️Sprint_4-Recursiones
**Tema:** recursión

---

## Sección 7. Recursión

Una función recursiva es una función que **se llama a sí misma**. Siempre tiene dos partes:

- **Caso base:** la situación en la que la función ya no necesita llamarse y devuelve un resultado directamente. Sin él, la recursión no termina nunca.
- **Caso recursivo:** la función resuelve una parte del problema y se llama a sí misma con un problema **más pequeño**.
```python
def funcion(datos):
    if condicion_de_parada:              # caso base
        return resultado_directo
    return combinar(parte, funcion(resto))   # caso recursivo
```

La recursión es útil en dos tipos de problema:

1. **Problemas que se dividen en subproblemas iguales pero más pequeños.** Por ejemplo, el tamaño de una carpeta es la suma del tamaño de sus subcarpetas.
2. **Problemas con un número desconocido de bucles anidados.** Por ejemplo, recorrer un JSON del que no se sabe cuántos niveles tiene, o combinar los valores de un número variable de parámetros.
En ingeniería de datos y en IA aparece al recorrer **estructuras anidadas**: documentos JSON, árboles de carpetas de un data lake, grafos de dependencias, planes de un agente o trazas de ejecución.

### Ejemplos resueltos

```python
def total_bytes(tamanos):
    if not tamanos:                                  # caso base: lista vacía
        return 0
    return tamanos[0] + total_bytes(tamanos[1:])     # caso recursivo

print(total_bytes([512, 1024, 256]))

# Misma función con traza para ver las llamadas
def total_bytes_traza(tamanos, nivel=0):
    sangria = "  " * nivel
    print(f"{sangria}total_bytes({tamanos})")
    if not tamanos:
        print(f"{sangria}-> 0  (caso base)")
        return 0
    resultado = tamanos[0] + total_bytes_traza(tamanos[1:], nivel + 1)
    print(f"{sangria}->{resultado}")
    return resultado

total_bytes_traza([512, 1024, 256])
```

**Paso a paso**

3. Caso base: una lista vacía suma 0 bytes.
4. Caso recursivo: el total es el primer elemento más el total del resto de la lista (`tamanos[1:]`), que es una lista más corta.
5. La traza muestra que las llamadas **bajan** hasta el caso base y que las sumas se resuelven al **subir**: 0 → 256 → 1280 → 1792. Mientras tanto, cada suma queda pendiente esperando el resultado de la llamada siguiente.
6. Cada llamada tiene sus propias variables. No se comparten entre llamadas.
**Salida esperada**

```plain text
1792
total_bytes([512, 1024, 256])
  total_bytes([1024, 256])
    total_bytes([256])
      total_bytes([])
      -> 0  (caso base)
    -> 256
  -> 1280
-> 1792
```

Según dónde esté la llamada recursiva, cambia el orden en que se hace el trabajo.

```python
def procesar_cola(lotes):
    """Recursión de cola: primero procesa, después se llama."""
    if not lotes:
        return
    print("cola     ->", lotes[0])
    procesar_cola(lotes[1:])

def procesar_cabeza(lotes):
    """Recursión de cabeza: primero se llama, después procesa."""
    if not lotes:
        return
    procesar_cabeza(lotes[1:])
    print("cabeza   ->", lotes[0])

particiones = ["day=01", "day=02", "day=03"]
procesar_cola(particiones)
procesar_cabeza(particiones)
```

**Paso a paso**

7. **Recursión de cola:** la función procesa el primer elemento y **después** se llama con el resto. El trabajo se hace al bajar, en el orden original, como en un bucle.
8. **Recursión de cabeza:** la función se llama **primero** y procesa el elemento después, cuando la llamada interior ya ha terminado. El trabajo se hace al subir, así que el orden se invierte.
9. Las dos versiones tienen el mismo caso base y la misma llamada. Solo cambia el lugar del `print()`.
**Salida esperada**

```plain text
cola     -> day=01
cola     -> day=02
cola     -> day=03
cabeza   -> day=03
cabeza   -> day=02
cabeza   -> day=01
```

Una política de reintentos con *backoff* exponencial duplica la espera tras cada fallo: 2 s, 4 s, 8 s… Queremos saber cuánto tiempo se espera en total.

```python
# Versión iterativa: espera total de una política de reintentos
def espera_total_iterativa(reintentos, espera_inicial):
    total = 0
    espera = espera_inicial
    while reintentos > 0:
        total += espera
        espera *= 2
        reintentos -= 1
    return total

# Versión recursiva, siguiendo los pasos de conversión
def espera_total(reintentos, espera):
    if reintentos == 0:                                  # condición del bucle -> caso base
        return 0
    return espera + espera_total(reintentos - 1, espera * 2)   # cuerpo -> caso recursivo

print("Iterativa:", espera_total_iterativa(5, 2), "s")
print("Recursiva:", espera_total(5, 2), "s")
for n in range(1, 6):
    print(f"{n} reintentos ->{espera_total(n, 2):>2} s de espera acumulada")
```

**Paso a paso**

Para convertir un bucle en recursión se siguen estos pasos:

10. Las variables que cambia el bucle (`reintentos`, `espera`) pasan a ser **parámetros** de la función recursiva.
11. La condición en la que el bucle termina (`reintentos == 0`) se convierte en el **caso base**.
12. El cuerpo del bucle se convierte en el **caso recursivo**: se suma la espera actual y se llama con un reintento menos y la espera duplicada.
13. Se comprueba que las dos versiones dan el mismo resultado y se eliminan las variables que ya no hacen falta. La versión recursiva no necesita el acumulador `total`.
**Salida esperada**

```plain text
Iterativa: 62 s
Recursiva: 62 s
  1 reintentos ->  2 s de espera acumulada
  2 reintentos ->  6 s de espera acumulada
  3 reintentos -> 14 s de espera acumulada
  4 reintentos -> 30 s de espera acumulada
  5 reintentos -> 62 s de espera acumulada
```

```python
# Carpetas = diccionarios; ficheros = tamaño en MB (int)
lake = {
    "bronce": {
        "ventas": {"2026-09.csv": 120, "2026-10.csv": 135},
        "clientes.csv": 40,
    },
    "plata": {
        "ventas": {"part-0000.parquet": 30, "part-0001.parquet": 28},
    },
    "oro": {"kpis_mensuales.parquet": 5},
}

def tamano_total(nodo):
    if isinstance(nodo, int):          # caso base: es un fichero
        return nodo
    total = 0
    for hijo in nodo.values():         # caso recursivo: es una carpeta
        total += tamano_total(hijo)
    return total

print("Data lake completo:", tamano_total(lake), "MB")
for capa, contenido in lake.items():
    print(f"{capa:<6}{tamano_total(contenido):>4} MB")
```

**Paso a paso**

14. `isinstance(nodo, int)` distingue un fichero, que es el caso base, de una carpeta, que es un diccionario.
15. Para una carpeta se suman los tamaños de todos sus hijos, y cada hijo puede ser a su vez otra carpeta.
16. Con un bucle `for` normal habría que saber de antemano cuántos niveles hay. La recursión funciona con cualquier profundidad.
**Salida esperada**

```plain text
Data lake completo: 358 MB
  bronce  295 MB
  plata    58 MB
  oro       5 MB
```

Antes de cargar un JSON en una tabla, hay que convertir sus objetos anidados en columnas planas, como hace la función `flatten` de muchas herramientas ETL.

```python
evento = {
    "id": "e-1001",
    "cliente": {
        "id": "C001",
        "direccion": {"ciudad": "Madrid", "cp": "28001"},
    },
    "importe": 99.9,
    "dispositivo": {"tipo": "movil", "so": {"nombre": "Android", "version": "14"}},
}

def aplanar(diccionario, prefijo=""):
    plano = {}
    for clave, valor in diccionario.items():
        nombre = f"{prefijo}.{clave}" if prefijo else clave
        if isinstance(valor, dict):
            plano.update(aplanar(valor, nombre))    # caso recursivo
        else:
            plano[nombre] = valor                   # caso base
    return plano

for columna, valor in aplanar(evento).items():
    print(f"{columna:<26}{valor}")
```

**Paso a paso**

17. `prefijo` acumula la ruta de claves: `cliente` → `cliente.direccion` → `cliente.direccion.ciudad`.
18. Si el valor es un diccionario, se aplana recursivamente y su resultado se incorpora con `update()`.
19. Si el valor es simple, es una hoja: se guarda con su nombre completo.
**Salida esperada**

```plain text
id                         e-1001
cliente.id                 C001
cliente.direccion.ciudad   Madrid
cliente.direccion.cp       28001
importe                    99.9
dispositivo.tipo           movil
dispositivo.so.nombre      Android
dispositivo.so.version     14
```

En un orquestador como Airflow, una tarea solo puede ejecutarse cuando han terminado todas sus dependencias.

```python
dependencias = {
    "kpis_ventas": ["ventas_limpias", "clientes_limpios"],
    "ventas_limpias": ["ventas_raw"],
    "clientes_limpios": ["clientes_raw"],
    "ventas_raw": [],
    "clientes_raw": [],
}

def orden_ejecucion(tarea, ejecutadas):
    for previa in dependencias[tarea]:
        if previa not in ejecutadas:
            orden_ejecucion(previa, ejecutadas)     # primero, sus dependencias
    ejecutadas.append(tarea)                        # después, la propia tarea
    return ejecutadas

orden = orden_ejecucion("kpis_ventas", [])
for paso, tarea in enumerate(orden, start=1):
    print(paso, tarea)
```

**Paso a paso**

20. Antes de ejecutar una tarea, se ejecutan recursivamente sus dependencias.
21. La comprobación `if previa not in ejecutadas` evita repetir una tarea que comparte dependencias con otra.
22. La tarea se añade a la lista **después** de sus dependencias. Por eso `kpis_ventas` aparece en último lugar. Es una recursión de cabeza: el trabajo se hace al volver de las llamadas.
**Salida esperada**

```plain text
1 ventas_raw
2 ventas_limpias
3 clientes_raw
4 clientes_limpios
5 kpis_ventas
```

Un fichero de log guarda un índice ordenado de *offsets*: la posición en bytes en la que empieza cada bloque. Buscar en él por mitades es mucho más rápido que recorrerlo entero.

```python
import sys

def buscar(ordenada, objetivo, inicio, fin):
    if inicio > fin:                     # caso base 1: no está
        return -1
    medio = (inicio + fin) // 2
    if ordenada[medio] == objetivo:      # caso base 2: encontrado
        return medio
    if objetivo < ordenada[medio]:
        return buscar(ordenada, objetivo, inicio, medio - 1)
    return buscar(ordenada, objetivo, medio + 1, fin)

offsets = [0, 120, 245, 380, 512, 640, 777, 901, 1024]
print("640 está en la posición", buscar(offsets, 640, 0, len(offsets) - 1))
print("300 está en la posición", buscar(offsets, 300, 0, len(offsets) - 1))
print("Límite de recursión:", sys.getrecursionlimit())

def profundidad_lineal(n):
    if n == 0:
        return 0
    return 1 + profundidad_lineal(n - 1)

print(profundidad_lineal(900))
# profundidad_lineal(5000)  -> RecursionError: maximum recursion depth exceeded

sys.setrecursionlimit(10_000)      # se puede subir, con precaución
print(profundidad_lineal(5000))
```

**Paso a paso**

23. En cada llamada se mira el elemento central. Si no es el buscado, se repite la búsqueda solo en la mitad donde puede estar.
24. Hay dos casos base: el elemento se encuentra, o el rango queda vacío (`inicio > fin`).
25. Con 1.000.000 de elementos bastan unas 20 llamadas, porque cada una descarta la mitad de los datos.
26. Python limita la profundidad de recursión a 1000 llamadas por defecto. `profundidad_lineal(900)` funciona, pero con 5000 se produce un `RecursionError`.
27. `sys.setrecursionlimit()` permite subir el límite, pero cada llamada pendiente ocupa memoria, y un límite demasiado alto puede hacer que el programa se cierre de golpe. Por eso la recursión es adecuada para estructuras con pocos niveles, como un JSON o un árbol de carpetas, y no para recorrer millones de elementos de uno en uno.
**Salida esperada**

```plain text
640 está en la posición 5
300 está en la posición -1
Límite de recursión: 1000
900
5000
```

Antes de llevar un job a producción, se prueba con distintas configuraciones de recursos. Con tres parámetros harían falta tres bucles anidados, pero si mañana se añade un cuarto parámetro, habría que reescribir el código. Es un problema con un **número desconocido de bucles**.

```python
parametros = {
    "executors": [4, 8],
    "memoria": ["4g", "8g"],
    "shuffle_partitions": [200, 400],
}

def combinaciones(nombres, parametros, actual, resultado):
    if len(actual) == len(nombres):          # caso base: ya hay un valor por parámetro
        resultado.append(dict(zip(nombres, actual)))
        return
    nombre = nombres[len(actual)]            # siguiente parámetro a fijar
    for valor in parametros[nombre]:
        actual.append(valor)
        combinaciones(nombres, parametros, actual, resultado)
        actual.pop()                         # deshacer para probar el siguiente valor

configuraciones = []
combinaciones(list(parametros), parametros, [], configuraciones)
for i, conf in enumerate(configuraciones, start=1):
    print(i, conf)

# Añadir un parámetro más no exige añadir un bucle más
parametros["compresion"] = ["snappy", "zstd"]
configuraciones = []
combinaciones(list(parametros), parametros, [], configuraciones)
print("Con 4 parámetros:", len(configuraciones), "configuraciones")
```

**Paso a paso**

28. `actual` es la configuración que se está construyendo. Cada nivel de recursión fija el valor de un parámetro.
29. **Caso base:** cuando `actual` tiene un valor por cada parámetro, la configuración está completa y se guarda como diccionario con `dict(zip(...))`.
30. **Caso recursivo:** para cada valor del siguiente parámetro, se añade a `actual`, se llama a la función para fijar el resto y, al volver, se quita con `pop()` para probar el siguiente valor. Esta técnica se llama *backtracking*.
31. Al añadir `compresion`, el código no cambia y genera 2 × 2 × 2 × 2 = 16 configuraciones.
**Salida esperada**

```plain text
1 {'executors': 4, 'memoria': '4g', 'shuffle_partitions': 200}
2 {'executors': 4, 'memoria': '4g', 'shuffle_partitions': 400}
3 {'executors': 4, 'memoria': '8g', 'shuffle_partitions': 200}
4 {'executors': 4, 'memoria': '8g', 'shuffle_partitions': 400}
5 {'executors': 8, 'memoria': '4g', 'shuffle_partitions': 200}
6 {'executors': 8, 'memoria': '4g', 'shuffle_partitions': 400}
7 {'executors': 8, 'memoria': '8g', 'shuffle_partitions': 200}
8 {'executors': 8, 'memoria': '8g', 'shuffle_partitions': 400}
Con 4 parámetros: 16 configuraciones
```

Cuando Spark ordena un dataset grande, cada partición se ordena por separado y después se combinan los resultados. La función recursiva divide la lista de particiones por la mitad hasta quedarse con una sola, y al volver fusiona los resultados de dos en dos.

```python
def fusionar(a, b):
    """Une dos listas ordenadas en una sola lista ordenada."""
    resultado = []
    i = j = 0
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            resultado.append(a[i])
            i += 1
        else:
            resultado.append(b[j])
            j += 1
    return resultado + a[i:] + b[j:]

def fusionar_particiones(particiones, nivel=0):
    print("  " * nivel + f"fusionar{len(particiones)} particiones")
    if len(particiones) == 1:                       # caso base
        return particiones[0]
    mitad = len(particiones) // 2
    izquierda = fusionar_particiones(particiones[:mitad], nivel + 1)
    derecha = fusionar_particiones(particiones[mitad:], nivel + 1)
    return fusionar(izquierda, derecha)

# Timestamps ya ordenados dentro de cada partición
particiones = [[3, 9, 15], [1, 12], [5, 6, 20], [2, 18]]
ordenado = fusionar_particiones(particiones)
print("Resultado:", ordenado)
```

**Paso a paso**

32. `fusionar()` no es recursiva: une dos listas ya ordenadas comparando sus primeros elementos y tomando siempre el menor.
33. `fusionar_particiones()` divide la lista de particiones en dos mitades y se llama con cada una.
34. **Caso base:** una sola partición ya está ordenada y se devuelve tal cual.
35. Al volver de las dos llamadas, se fusionan los dos resultados. La traza muestra el árbol de llamadas: 4 → 2 + 2 → 1 + 1 + 1 + 1.
36. Este mismo esquema, dividir el problema, resolver cada parte y combinar los resultados, se llama **divide y vencerás**, y es la base del algoritmo *merge sort*.
**Salida esperada**

```plain text
fusionar 4 particiones
  fusionar 2 particiones
    fusionar 1 particiones
    fusionar 1 particiones
  fusionar 2 particiones
    fusionar 1 particiones
    fusionar 1 particiones
Resultado: [1, 2, 3, 5, 6, 9, 12, 15, 18, 20]
```

Un agente de IA que recibe una tarea compleja la **descompone** en subtareas, y cada subtarea puede descomponerse a su vez, hasta llegar a acciones concretas que se resuelven con una herramienta (una consulta SQL, una búsqueda web, una llamada al LLM…). El plan resultante es un árbol.

```python
plan = {
    "tarea": "Informe de ventas de octubre",
    "subtareas": [
        {"tarea": "Obtener datos", "subtareas": [
            {"tarea": "Consultar ventas", "herramienta": "sql"},
            {"tarea": "Consultar objetivos", "herramienta": "api_crm"},
        ]},
        {"tarea": "Analizar", "subtareas": [
            {"tarea": "Calcular KPIs", "herramienta": "python"},
            {"tarea": "Detectar anomalías", "subtareas": [
                {"tarea": "Comparar con septiembre", "herramienta": "python"},
                {"tarea": "Buscar causas en noticias", "herramienta": "web_search"},
            ]},
        ]},
        {"tarea": "Redactar el informe", "herramienta": "llm"},
    ],
}

def mostrar_plan(nodo, nivel=0):
    if "herramienta" in nodo:                         # caso base: acción concreta
        print("  " * nivel + f"-{nodo['tarea']}  [{nodo['herramienta']}]")
        return 1
    print("  " * nivel + f"+{nodo['tarea']}")
    acciones = 0
    for sub in nodo["subtareas"]:                     # caso recursivo: descomponer
        acciones += mostrar_plan(sub, nivel + 1)
    return acciones

total = mostrar_plan(plan)
print("Acciones que ejecutará el agente:", total)
```

**Paso a paso**

37. Cada nodo del plan es un diccionario. Si tiene la clave `herramienta`, es una **acción concreta**: el caso base. Se imprime y cuenta como 1.
38. Si tiene `subtareas`, es una tarea compuesta: se imprime y se llama a la función con cada subtarea, un nivel más adentro.
39. `nivel` controla la sangría, así que la salida refleja la estructura del árbol.
40. La función devuelve el número de acciones de cada rama, y la tarea compuesta suma las de sus hijos. Así sabemos cuántas herramientas tendrá que ejecutar el agente.
**Salida esperada**

```plain text
+ Informe de ventas de octubre
  + Obtener datos
    - Consultar ventas  [sql]
    - Consultar objetivos  [api_crm]
  + Analizar
    - Calcular KPIs  [python]
    + Detectar anomalías
      - Comparar con septiembre  [python]
      - Buscar causas en noticias  [web_search]
  - Redactar el informe  [llm]
Acciones que ejecutará el agente: 6
```

Los *splitters* recursivos de herramientas como LangChain intentan cortar un documento por los separadores más “naturales” primero: párrafos, después líneas, después frases y, solo si no queda más remedio, palabras. Así cada chunk respeta al máximo la estructura del texto.

```python
def trocear(texto, max_caracteres, separadores=("\n\n", "\n", ". ", " ")):
    if len(texto) <= max_caracteres:                  # caso base: ya cabe
        return [texto]
    if not separadores:                               # caso base: corte duro
        return [texto[i:i + max_caracteres] for i in range(0, len(texto), max_caracteres)]
    separador = separadores[0]
    piezas = []
    for parte in texto.split(separador):              # caso recursivo
        if parte.strip():
            piezas.extend(trocear(parte.strip(), max_caracteres, separadores[1:]))
    chunks = []                                       # juntar piezas pequeñas
    for pieza in piezas:
        if chunks and len(chunks[-1]) + len(separador) + len(pieza) <= max_caracteres:
            chunks[-1] = chunks[-1] + separador + pieza
        else:
            chunks.append(pieza)
    return chunks

manual = """Política de envíos. Los pedidos se envían en 24 horas.

Devoluciones. El plazo es de 30 días. El producto debe estar sin usar. Los gastos de devolución corren a cargo del cliente salvo defecto de fábrica.

Garantía. Dos años."""

for i, chunk in enumerate(trocear(manual, 60)):
    print(f"{i} ({len(chunk):>2} car.){chunk!r}")
```

**Paso a paso**

41. **Caso base 1:** si el texto ya cabe en el límite, se devuelve tal cual.
42. **Caso base 2:** si no quedan separadores, se corta a la fuerza cada `max_caracteres`.
43. **Caso recursivo:** se divide el texto con el primer separador y cada parte se trocea de nuevo con los separadores **restantes**. Una frase demasiado larga acabará dividida por espacios.
44. Al volver, se juntan las piezas pequeñas consecutivas mientras quepan en el límite. Sin este paso, la frase larga quedaría convertida en palabras sueltas.
45. Se muestra cada chunk con `!r` para que se vea el salto de párrafo (`\n\n`) del último.
**Salida esperada**

```plain text
0 (54 car.) 'Política de envíos. Los pedidos se envían en 24 horas.'
1 (36 car.) 'Devoluciones. El plazo es de 30 días'
2 (31 car.) 'El producto debe estar sin usar'
3 (57 car.) 'Los gastos de devolución corren a cargo del cliente salvo'
4 (40 car.) 'defecto de fábrica.\n\nGarantía. Dos años.'
```

### Ejercicios propuestos

Algunos sistemas de destino limitan el número de niveles de anidamiento que aceptan. Escribe `profundidad(nodo)` con estas reglas: un valor simple tiene profundidad 0, y un diccionario o una lista tienen profundidad 1 más la mayor profundidad de sus elementos. Un diccionario o una lista vacíos tienen profundidad 1.

```python
documento = {
    "id": 1,
    "cliente": {"nombre": "Acme", "contacto": {"email": "a@acme.com", "tel": {"pref": "+34"}}},
    "items": [{"sku": "A1", "atributos": {"color": "rojo"}}, {"sku": "B2"}],
}
```

Calcula la profundidad de `documento`, de `documento["items"]` y del valor `42`.

**Salida esperada**

```plain text
Profundidad: 4
Profundidad de 'items': 3
Profundidad de un valor simple: 0
```

```python
lake = {
    "bronce": {"ventas": {"2026-09.csv": 120, "2026-10.csv": 135}, "clientes.csv": 40},
    "plata": {
        "ventas": {"year=2026": {"part-0000.parquet": 30, "part-0001.parquet": 28}},
        "clientes": {"part-0000.parquet": 12},
    },
    "oro": {"kpis_mensuales.parquet": 5, "README.md": 1},
}
```

Escribe `buscar_ficheros(nodo, extension, ruta="")`, que devuelva la lista de rutas completas de los ficheros con esa extensión. Usa `nombre.endswith(extension)`. Muestra las rutas `.parquet` y su total, y después solo el total de `.csv`.

**Salida esperada**

```plain text
/plata/ventas/year=2026/part-0000.parquet
/plata/ventas/year=2026/part-0001.parquet
/plata/clientes/part-0000.parquet
/oro/kpis_mensuales.parquet
Total .parquet: 4
Total .csv: 3
```

Este diccionario indica las tablas de origen directas de cada tabla.

```python
origenes = {
    "dashboard_ventas": ["kpis_ventas", "dim_tiempo"],
    "kpis_ventas": ["ventas_limpias", "clientes_limpios"],
    "ventas_limpias": ["ventas_raw", "tipos_cambio"],
    "clientes_limpios": ["clientes_raw"],
    "dim_tiempo": [],
    "ventas_raw": [],
    "clientes_raw": [],
    "tipos_cambio": [],
}
```

Escribe `ancestros(tabla)`, que devuelva un `set` con todas las tablas de las que depende, directa o indirectamente. Muestra los ancestros de `dashboard_ventas`, de `ventas_limpias` y de `ventas_raw`. Por último, con una comprehension, obtén las fuentes originales del dashboard: los ancestros que no tienen ningún origen.

**Salida esperada**

```plain text
['clientes_limpios', 'clientes_raw', 'dim_tiempo', 'kpis_ventas', 'tipos_cambio', 'ventas_limpias', 'ventas_raw']
['tipos_cambio', 'ventas_raw']
[]
Fuentes originales del dashboard: ['clientes_raw', 'dim_tiempo', 'tipos_cambio', 'ventas_raw']
```

La respuesta mezcla diccionarios y listas a varios niveles.

```python
respuesta_api = {
    "pedido": 9001,
    "cliente": {"nombre": "Acme", "email": None, "telefonos": ["600111222", None]},
    "lineas": [
        {"sku": "A1", "precio": 10.0, "descuento": None},
        {"sku": None, "precio": None, "descuento": 0.1},
    ],
    "notas": None,
}
```

Escribe `contar_nulos(nodo)`, que cuente todos los `None` de la estructura, estén dentro de diccionarios o de listas.

**Salida esperada**

```plain text
Nulos en la respuesta: 6
Nulos en 'lineas': 3
```

Un orquestador rechaza un flujo de tareas si sus dependencias forman un ciclo (por ejemplo, `b → c → d → b`), porque ninguna de esas tareas podría arrancar nunca.

```python
dag_ok = {"a": ["b", "c"], "b": ["d"], "c": ["d"], "d": []}
dag_ko = {"a": ["b"], "b": ["c"], "c": ["d"], "d": ["b"]}
```

46. Escribe `tiene_ciclo(grafo, nodo, camino)`, que devuelva `True` si, partiendo de `nodo`, se vuelve a un nodo que ya está en el camino actual. `camino` es un `set` con los nodos visitados en la rama que se está recorriendo.
47. Escribe `validar_dag(grafo)`, que compruebe todos los nodos y devuelva `True` si no hay ningún ciclo.
**Salida esperada**

```plain text
dag_ok es un DAG válido: True
dag_ko es un DAG válido: False
```

Los sistemas de almacenamiento informan del tamaño en bytes, pero en los informes se muestran en la unidad más adecuada: KB, MB, GB…

```python
tamanos_bytes = [512, 2048, 5_368_709_120, 1_099_511_627_776, 734_003_200]
```

Escribe `legible(valor, nivel=0)` de forma **recursiva**, usando la lista de unidades `["B", "KB", "MB", "GB", "TB", "PB"]`:

- **Caso base:** si el valor es menor que 1024, o ya no quedan unidades mayores, se devuelve con un decimal y su unidad.
- **Caso recursivo:** se divide el valor entre 1024 y se llama con el nivel siguiente.
**Salida esperada**

```plain text
             512 -> 512 B
            2048 -> 2.0 KB
      5368709120 -> 5.0 GB
   1099511627776 -> 1.0 TB
       734003200 -> 700.0 MB
```

Cuando hay muchas particiones, combinar todos sus resultados en un solo nodo puede saturarlo. Spark ofrece `treeReduce()`, que combina los resultados parciales por parejas, en varios niveles, como un torneo.

```python
# Conteo de eventos calculado por cada una de las 6 particiones
parciales = [
    {"click": 120, "view": 400},
    {"click": 98, "compra": 7},
    {"view": 310, "compra": 3},
    {"click": 150, "view": 280, "compra": 12},
    {"view": 95},
    {"click": 40, "compra": 1},
]
```

48. Escribe `combinar(a, b)`, que devuelva un diccionario nuevo con la suma de los conteos de los dos diccionarios. No es recursiva.
49. Escribe `tree_reduce(parciales, nivel=0)`, que divida la lista en dos mitades, se llame con cada una y combine los dos resultados. El caso base es una lista con un solo diccionario. Imprime en cada llamada el nivel y el número de parciales que recibe, con una sangría de dos espacios por nivel.
50. Muestra el resultado final con las claves ordenadas.
**Salida esperada**

```plain text
nivel 0: 6 parciales
  nivel 1: 3 parciales
    nivel 2: 1 parciales
    nivel 2: 2 parciales
      nivel 3: 1 parciales
      nivel 3: 1 parciales
  nivel 1: 3 parciales
    nivel 2: 1 parciales
    nivel 2: 2 parciales
      nivel 3: 1 parciales
      nivel 3: 1 parciales
Resultado: {'click': 408, 'compra': 23, 'view': 1085}
```

En HDFS y en los data lakes, miles de ficheros diminutos degradan el rendimiento: cada fichero supone metadatos y una tarea de lectura. Se recomienda compactar las carpetas donde predominan los ficheros pequeños.

```python
# Carpetas = diccionarios; ficheros = tamaño en MB
tabla_eventos = {
    "fecha=2026-10-13": {"part-0.parquet": 210, "part-1.parquet": 190},
    "fecha=2026-10-14": {f"part-{i}.parquet": 3 for i in range(6)},
    "fecha=2026-10-15": {
        "hora=00": {"part-0.parquet": 2, "part-1.parquet": 4},
        "hora=01": {"part-0.parquet": 150},
    },
}
umbral_mb = 32
```

51. Escribe `ficheros_pequenos(nodo, umbral)`, que recorra una carpeta y todas sus subcarpetas y devuelva una tupla `(total_de_ficheros, ficheros_por_debajo_del_umbral)`.
52. Aplícala a cada partición de fecha y marca `COMPACTAR` las que tengan más de la mitad de sus ficheros por debajo del umbral.
53. Aplícala también a la tabla completa.
**Salida esperada**

```plain text
fecha=2026-10-13: 2 ficheros, 0 pequeños -> ok
fecha=2026-10-14: 6 ficheros, 6 pequeños -> COMPACTAR
fecha=2026-10-15: 3 ficheros, 2 pequeños -> COMPACTAR
Tabla completa (ficheros, pequeños): (11, 8)
```

Las plataformas de observabilidad de agentes guardan cada ejecución como un árbol de pasos: llamadas al LLM, llamadas a herramientas y subagentes, que a su vez lanzan otros pasos.

```python
# Traza de ejecución de un agente: cada paso puede lanzar sub-pasos
traza = {
    "paso": "agente_principal", "tipo": "llm", "tokens": 1200,
    "hijos": [
        {"paso": "buscar_clientes", "tipo": "tool", "herramienta": "sql", "tokens": 0, "hijos": []},
        {"paso": "subagente_analista", "tipo": "llm", "tokens": 2300, "hijos": [
            {"paso": "calcular_churn", "tipo": "tool", "herramienta": "python", "tokens": 0, "hijos": []},
            {"paso": "buscar_contexto", "tipo": "tool", "herramienta": "rag", "tokens": 450, "hijos": [
                {"paso": "reescribir_consulta", "tipo": "llm", "tokens": 300, "hijos": []},
            ]},
        ]},
        {"paso": "redactar_respuesta", "tipo": "llm", "tokens": 1800, "hijos": []},
    ],
}
```

Escribe cuatro funciones recursivas:

- `tokens_totales(nodo)`: suma los tokens del nodo y de todos sus descendientes.
- `profundidad(nodo)`: un paso sin hijos tiene profundidad 1.
- `herramientas(nodo)`: devuelve el `set` de herramientas usadas en los pasos de tipo `tool`.
- `llamadas_llm(nodo)`: cuenta los pasos de tipo `llm`.
Muestra los resultados para la traza completa y los tokens consumidos solo por `subagente_analista`.

**Salida esperada**

```plain text
Tokens totales: 6050
Profundidad máxima: 4
Herramientas usadas: ['python', 'rag', 'sql']
Llamadas al LLM: 4
Tokens del subagente: 3050
```

Cuando un documento no cabe en el contexto de un LLM, se usa un resumen jerárquico: se resumen los fragmentos por grupos, después se resumen esos resúmenes, y así hasta obtener uno solo. Como aquí no llamamos a un modelo real, la función `resumir()` lo simula quedándose con las 3 primeras palabras de cada texto.

```python
# Simulamos el LLM: "resume" un grupo de textos quedándose con
# las 3 primeras palabras de cada uno
def resumir(textos):
    return " ".join([" ".join(t.split()[:3]) for t in textos])

capitulos = [
    "Ventas crecen un 12% en el canal online",
    "Tiendas físicas estables respecto al trimestre anterior",
    "Devoluciones bajan tras el cambio de política",
    "Coste logístico sube por el precio del combustible",
    "Nuevo almacén en Valencia reduce plazos de entrega",
]
```

54. Escribe `resumen_jerarquico(textos, grupo=2, nivel=0)`. En cada llamada imprime el nivel y cuántos textos recibe. Si solo queda un texto, lo devuelve. Si no, resume los textos en grupos de `grupo` elementos y se llama con la lista de resúmenes.
55. Ejecútala con grupos de 2 y de 3 y compara cuántos niveles necesita cada una.
56. Observa el resumen final: ¿qué información de los capítulos se ha perdido? Con un LLM real ocurre algo parecido, por eso el tamaño de grupo y las instrucciones de cada nivel influyen en la calidad del resultado.
**Salida esperada**

```plain text
Nivel 0: 5 textos
Nivel 1: 3 textos
Nivel 2: 2 textos
Nivel 3: 1 textos
Resumen final: Ventas crecen un Nuevo almacén en

Nivel 0: 5 textos
Nivel 1: 2 textos
Nivel 2: 1 textos
Resumen final: Ventas crecen un Coste logístico sube
```

** **

Estructura sugerida del repo:

```plain text
Practica-13_Sprints_de_Python/
├── README.md
├── .gitignore
├── sprint_01_estructuras_datos/
│   └── sprint_01_estructuras_datos.ipynb    # listas, tuplas, set y diccionarios
├── sprint_02_comprehensions/
│   └── sprint_02_comprehensions.ipynb
├── sprint_03_funciones/
│   └── sprint_03_funciones.ipynb
└── sprint_04_recursion/
    └── sprint_04_recursion.ipynb
```

