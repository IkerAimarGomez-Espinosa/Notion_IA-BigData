# 🧗🏽‍♂️ Practica 03 - Python

**Entorno obligatorio:** Python **3.12**

**Herramienta de trabajo:** **JupyterLab**

**Formato de entrega:** notebook `.ipynb` dentro de un repositorio de **GitHub.**

Los archivos csv de muestra los puedes [descargar aquí.](https://tajamar365.sharepoint.com/:f:/s/3601-MasterIA2026-2027/IgDbW1c20a9gQpWfbhypvIZBAdLg6Xds1gbEJfrovWyyF5w?e=RvWGY2)

---

## Sección 0. Entorno de trabajo

Toda la práctica se resuelve en **Python 3.12** y se ejecuta en **JupyterLab**. No se admiten entregas realizadas en otras versiones del intérprete ni en otros entornos (scripts sueltos, Google Colab exportado sin celdas, capturas de pantalla, etc.).

### Preparación del entorno

Ejecuta estos comandos en tu terminal antes de empezar:

```bash
# 1. Crear el entorno virtual con Python 3.12
python3.12 -m venv .venv

# 2. Activar el entorno
source .venv/bin/activate        # Linux / macOS
.venv\Scripts\activate           # Windows (PowerShell)

# 3. Instalar dependencias
pip install --upgrade pip
pip install jupyterlab numpy pandas

# 4. Lanzar JupyterLab
jupyter lab
```

### Celda de verificación obligatoria

La **primera celda de código** de tu notebook debe ser exactamente esta. Sirve para demostrar que has trabajado sobre Python 3.12:

```python
import sys
import platform
import numpy as np
import pandas as pd

print("Python      :", sys.version)
print("Implementación:", platform.python_implementation())
print("NumPy       :", np.__version__)
print("pandas      :", pd.__version__)

assert sys.version_info[:2] == (3, 12), "Esta práctica requiere Python 3.12"
print("\nEntorno verificado correctamente.")
```

> Si el `assert` falla, el entorno no es válido y la práctica se considera no entregada. Revisa que el *kernel* seleccionado en JupyterLab (esquina superior derecha) apunte a tu entorno virtual con 3.12.

---

## Sección 1. Instrucciones de entrega

### Qué se entrega

Un **enlace a un repositorio público de GitHub** que contenga el notebook resuelto y los datos de partida. No se aceptan ficheros `.ipynb` sueltos por correo ni carpetas comprimidas.

### Estructura de carpetas exigida

El repositorio debe respetar esta estructura. La carpeta `data/` es obligatoria y debe contener los tres CSV de muestra que se proporcionan con el enunciado.

```plain text
practica3-python-<apellido>-<nombre>/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   ├── ventas_retail.csv
│   ├── empleados.csv
│   └── sensores_planta.csv
├── notebooks/
│   └── practica3_python.ipynb
├── src/
│   └── utilidades.py
└── outputs/
    └── (ficheros generados por el notebook)
```

| Elemento | Obligatorio | Contenido |
| --- | --- | --- |
| `README.md` | Sí | Ver plantilla más abajo |
| `requirements.txt` | Sí | Dependencias con versión fijada |
| `.gitignore` | Sí | Debe ignorar `.venv/`, `__pycache__/`, `.ipynb_checkpoints/` |
| `data/` | Sí | Los tres CSV de muestra, sin modificar |
| `notebooks/` | Sí | El notebook resuelto, con todas las celdas ejecutadas |
| `src/` | Sí | Módulo `utilidades.py` creado en el ejercicio 7 |
| `outputs/` | Sí | CSV generados por los ejercicios 15 y 18 |

### Plantilla mínima de `README.md`

Copia esta plantilla en tu repositorio y complétala:

```markdown
# Práctica 3: Python — <Nombre y Apellidos>

## Descripción
Resolución de los 20 ejercicios de la Práctica 3 sobre listas, funciones y
paquetes, NumPy, diccionarios y pandas, lógica y control de flujo, y bucles.

## Entorno
- Python 3.12
- JupyterLab
- NumPy, pandas (ver `requirements.txt`)

## Estructura del repositorio
| Carpeta | Contenido |
| --- | --- |
| `data/` | Ficheros CSV de partida |
| `notebooks/` | Notebook con la resolución |
| `src/` | Módulo de funciones auxiliares |
| `outputs/` | Ficheros generados durante la ejecución |

## Cómo reproducir
```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter lab notebooks/practica3_python.ipynb
```

```plain text

### Contenido de `requirements.txt`

```text
jupyterlab>=4.2
numpy>=1.26
pandas>=2.2
```

### Normas del notebook

1. **Una celda Markdown por ejercicio** con el título `## Ejercicio N — <título>` antes del código.
2. El notebook debe entregarse **con todas las celdas ejecutadas y en orden** (`Kernel → Restart Kernel and Run All Cells` antes del último commit).
3. Las rutas a los datos deben ser **relativas**: `../data/ventas_retail.csv`. Cualquier ruta absoluta del tipo `C:\Users\...` invalida el ejercicio.
4. Cada ejercicio debe terminar con al menos una llamada a `print()` o una salida visible que demuestre el resultado.
5. Comenta el código: se valora la explicación del *porqué*, no la descripción de la sintaxis.
6. Mínimo **5 commits** con mensajes descriptivos. Un único commit del tipo *"subo práctica"* penaliza.
---

## Sección 2. Datos de partida

Se proporcionan [tres ficheros CSV sintéticos](https://tajamar365.sharepoint.com/:f:/s/3601-MasterIA2026-2027/IgDbW1c20a9gQpWfbhypvIZBAdLg6Xds1gbEJfrovWyyF5w?e=RbJFAq). **Descárgalos y colócalos en la carpeta ****`data/`**** de tu repositorio sin modificarlos.**

### `ventas_retail.csv` — 420 filas

Pedidos de una cadena minorista durante los tres primeros trimestres de 2025.

| Columna | Tipo | Descripción |
| --- | --- | --- |
| `id_pedido` | texto | Identificador único del pedido |
| `fecha` | texto (ISO) | Fecha del pedido |
| `region` | texto | Norte, Sur, Este, Oeste, Centro |
| `categoria` | texto | Electronica, Hogar, Deporte, Oficina |
| `producto` | texto | Nombre comercial del artículo |
| `unidades` | entero | Unidades vendidas en el pedido |
| `precio_unitario` | decimal | Precio de catálogo por unidad (€) |
| `descuento_pct` | entero | Descuento aplicado (%) |
| `canal` | texto | Online, Tienda, Marketplace (**contiene nulos**) |
| `satisfaccion` | decimal | Valoración del cliente de 1 a 5 (**contiene nulos**) |

### `empleados.csv` — 180 filas

Plantilla de una empresa tecnológica.

| Columna | Tipo | Descripción |
| --- | --- | --- |
| `id_empleado` | texto | Identificador único |
| `nombre` | texto | Nombre y apellido |
| `departamento` | texto | Ingenieria, Ventas, Marketing, Operaciones, Finanzas, Soporte |
| `ciudad` | texto | Sede de trabajo |
| `antiguedad_anios` | entero | Años en la compañía |
| `salario_base` | entero | Salario bruto anual (€) |
| `bonus_pct` | entero | Bonus variable sobre el salario base (%) |
| `modalidad` | texto | Presencial, Hibrido, Remoto |
| `horas_formacion` | decimal | Horas de formación del último año (**contiene nulos**) |

### `sensores_planta.csv` — 600 filas

Telemetría de cuatro máquinas de una planta de producción, muestreada cada 15 minutos.

| Columna | Tipo | Descripción |
| --- | --- | --- |
| `marca_tiempo` | texto | Fecha y hora de la lectura |
| `id_maquina` | texto | MAQ-01 a MAQ-04 |
| `linea` | texto | Linea A (MAQ-01, MAQ-02) o Linea B (MAQ-03, MAQ-04) |
| `temperatura_c` | decimal | Temperatura del cabezal (°C) |
| `vibracion_mm_s` | decimal | Vibración RMS (mm/s) |
| `presion_bar` | decimal | Presión del circuito (bar) (**contiene nulos**) |
| `unidades_producidas` | entero | Piezas producidas en el intervalo |
| `estado` | texto | OK, AVISO, ALARMA |

---

## Sección 3. Listas

### Ejercicio 1 — Catálogo de productos como lista

Una tienda gestiona su catálogo en una lista plana que alterna nombre de producto y precio. Trabaja con esta estructura sin convertirla todavía a diccionario.

```python
catalogo = [
    "Auriculares BT", 59.90,
    "Smartwatch S2", 149.00,
    "Tablet 10", 219.00,
    "Cargador USB-C", 18.50,
    "Cafetera Expres", 189.00,
    "Robot Aspirador", 279.00,
]
```

**Se pide:**

7. Imprime el número total de elementos de la lista y deduce cuántos productos hay.
8. Usando *slicing con paso*, crea `nombres` con todos los nombres y `precios` con todos los precios.
9. Obtén el precio del `"Robot Aspirador"` **sin escribir el índice a mano**: localiza su posición con `.index()` y usa esa posición para llegar al precio.
10. Crea `ultimos_tres_nombres` con los tres últimos productos usando índices negativos.
11. Calcula e imprime el precio medio del catálogo redondeado a dos decimales, usando únicamente `sum()`, `len()` y `round()`.
**Comprobación:** `nombres` debe tener 6 elementos y `precios` debe sumar 914,40.

> **Pista:** `lista[inicio:fin:paso]`. Con paso 2 desde el índice 0 obtienes las posiciones pares, desde el índice 1 las impares.

---

### Ejercicio 2 — Inventario multialmacén con listas anidadas

El inventario de tres almacenes se representa como una lista de listas. Cada sublista sigue el formato `[codigo_almacen, producto, unidades, coste_unitario]`.

```python
inventario = [
    ["ALM-NORTE", "Robot Aspirador", 34, 201.50],
    ["ALM-NORTE", "Monitor 27", 58, 142.00],
    ["ALM-SUR",   "Silla Ergonomica", 12, 178.90],
    ["ALM-SUR",   "Robot Aspirador", 7, 201.50],
    ["ALM-ESTE",  "Cafetera Expres", 41, 131.20],
    ["ALM-ESTE",  "Monitor 27", 25, 142.00],
]
```

**Se pide:**

12. Imprime el nombre del producto de la **cuarta** fila usando doble indexación.
13. Crea una nueva lista `valor_stock` donde cada elemento sea `[producto, unidades * coste_unitario]` redondeado a dos decimales. Resuélvelo con un bucle `for` sobre `inventario`.
14. Calcula el **valor inmovilizado total** del inventario.
15. Extrae, mediante *slicing*, una sublista con solo las filas del `ALM-SUR` (usa el hecho de que están contiguas) y calcula su valor.
16. Explica en una celda Markdown de **dos frases** qué problema aparece si dos almacenes dejan de estar contiguos y por qué el *slicing* no es una estrategia robusta de filtrado.
**Comprobación:** el valor inmovilizado total debe ser 27 573,50 € y el del `ALM-SUR`, 3 557,30 €.

---

### Ejercicio 3 — Manipulación destructiva y copias de listas

Este ejercicio trabaja el comportamiento interno de las listas (referencias frente a copias).

```python
precios_originales = [59.90, 149.00, 219.00, 18.50, 189.00, 279.00]
```

**Se pide:**

17. Crea `precios_rebajados = precios_originales` y modifica el primer elemento de `precios_rebajados` a `49.90`. Imprime **las dos listas** y explica qué ha ocurrido.
18. Restaura el valor original. Ahora crea dos copias independientes: una con `list()` y otra con *slicing* completo `[:]`. Demuestra con `print()` que modificar la copia ya no afecta al original.
19. Sobre una copia, aplica una rebaja del 10 % a **todos** los elementos usando un bucle `for` con `range(len(...))`.
20. Añade dos productos nuevos con `.extend()` y un tercero en la **posición 2** con `.insert()`.
21. Elimina el elemento de la posición 4 con `del` y el último con `.pop()`, guardando el valor extraído en una variable `descartado`.
22. Comprueba con `id()` que `precios_originales` y cada copia apuntan a objetos distintos en memoria.
> **Pista:** una asignación con `=` no copia la lista, copia la referencia. Es el error más caro de depurar en proyectos de datos reales.

---

### Ejercicio 4 — Cola de pedidos pendientes

La cola de pedidos de un almacén se gestiona como lista ordenada por prioridad.

```python
cola = ["PED-10021", "PED-10007", "PED-10044", "PED-10012", "PED-10033", "PED-10008"]
```

**Se pide:**

23. Invierte la cola usando *slicing* con paso negativo (sin `.reverse()`) y guárdala en `cola_invertida`.
24. Ordena `cola` alfabéticamente con `sorted()` **sin modificar la original** y comprueba que la original sigue intacta.
25. Ordena ahora la lista **en sitio** con `.sort(reverse=True)`.
26. Atiende los dos primeros pedidos: extráelos con `.pop(0)` y guárdalos en `atendidos`.
27. Inserta un pedido urgente `"PED-99999"` al principio de la cola.
28. Imprime un informe final con el número de pedidos atendidos, los pendientes y el porcentaje de avance redondeado a un decimal.
---

## Sección 4. Funciones y Paquetes

### Ejercicio 5 — Funciones propias para el cálculo de importes

**Se pide:**

29. Define la función `calcular_importe(unidades, precio_unitario, descuento_pct=0)` que devuelva el importe neto redondeado a dos decimales. El parámetro `descuento_pct` debe tener valor por defecto.
30. Define `resumen_pedido(unidades, precio_unitario, descuento_pct=0)` que devuelva **una tupla de tres elementos**: `(importe_bruto, ahorro, importe_neto)`.
31. Llama a `resumen_pedido` usando **argumentos por palabra clave** y desempaqueta el resultado en tres variables.
32. Documenta ambas funciones con *docstring* y muestra la ayuda con `help(calcular_importe)`.
33. Usa `max()`, `min()`, `sorted()` y `len()` sobre la lista `importes` que obtengas al aplicar `calcular_importe` a estos cinco pedidos:
```python
pedidos = [
    (3, 59.90, 10),
    (1, 279.00, 0),
    (5, 18.50, 20),
    (2, 149.00, 5),
    (4, 89.90, 15),
]
```

**Comprobación:** `calcular_importe(3, 59.90, 10)` debe devolver `161.73`.
---

### Ejercicio 6 — Normalización de códigos SKU con métodos

El equipo de logística recibe códigos de producto en formatos inconsistentes.

```python
skus_crudos = [
    "  elc-0012-es ", "HOG-0045-ES", "dep-0003-pt  ",
    "ofi-0021-es", "  ELC-0012-ES", "hog-0099-fr ",
]
```

**Se pide:**

34. Crea `skus_limpios` aplicando a cada código: eliminación de espacios sobrantes (`.strip()`) y conversión a mayúsculas (`.upper()`).
35. Cuenta cuántas veces aparece `"ELC-0012-ES"` en la lista limpia con `.count()`.
36. Obtén la posición de la primera aparición de `"OFI-0021-ES"` con `.index()`.
37. Para cada SKU limpio, usa `.split("-")` para separar familia, número y país. Construye una lista de listas con esas tres partes.
38. Crea `skus_es` con solo los SKU cuyo país sea `"ES"` y sustituye en todos ellos el guion por un punto con `.replace()`.
39. Genera una cadena única separada por `" | "` usando `.join()` y muéstrala.
> **Pista:** `.strip()`, `.upper()` y `.replace()` **devuelven** una nueva cadena; no modifican la original. `.append()` y `.sort()` sobre listas, en cambio, actúan en sitio y devuelven `None`.

---

### Ejercicio 7 — Paquetes, importaciones y módulo propio

**Se pide:**

40. Importa el paquete completo con `import math` y calcula la raíz cuadrada de `2809` con `math.sqrt()`.
41. Importa **selectivamente** `pi` y `ceil` con `from math import pi, ceil`. Calcula cuántos palés completos de 24 unidades hacen falta para enviar 1 375 unidades.
42. Importa `numpy` con alias `np` y `pandas` con alias `pd`, e imprime la versión de ambos con `np.__version__` y `pd.__version__`.
43. Usa el módulo `random` con semilla fija para generar 10 valoraciones de cliente entre 1 y 5:
```python
import random

random.seed(42)
valoraciones = [random.randint(1, 5) for _ in range(10)]
print(valoraciones)
```
44. Explica en **una frase** por qué fijar la semilla hace que el resultado sea el mismo para ti y para el profesor.
45. Crea el fichero `src/utilidades.py` copiando este contenido tal cual:
```python
# src/utilidades.py
"""Funciones auxiliares de la Práctica 3."""

def calcular_importe(unidades, precio_unitario, descuento_pct=0):
    """Devuelve el importe neto de una línea de pedido."""
    bruto = unidades * precio_unitario
    return round(bruto * (1 - descuento_pct / 100), 2)


def clasificar_ticket(importe):
    """Clasifica un importe en 'Bajo', 'Medio' o 'Alto'."""
    if importe < 100:
        return "Bajo"
    elif importe < 400:
        return "Medio"
    return "Alto"
```
46. Impórtalo desde el notebook y comprueba que funciona:
```python
import sys
sys.path.append("../src")

from utilidades import calcular_importe, clasificar_ticket

print(calcular_importe(4, 89.90, 15))
print(clasificar_ticket(305.66))
```
**Comprobación:** `calcular_importe(4, 89.90, 15)` debe devolver `305.66` y `clasificar_ticket(305.66)` debe devolver `"Medio"`.

> **Pista:** `sys.path.append("../src")` le indica a Python dónde buscar tu módulo, porque el notebook vive en `notebooks/` y el módulo en `src/`.

### Ejercicio 8 — Del cálculo con listas al cálculo vectorizado

```python
unidades = [3, 1, 5, 2, 4, 6, 2, 8, 3, 1]
precios  = [59.90, 279.00, 18.50, 149.00, 89.90, 27.50, 219.00, 14.90, 64.00, 239.00]
```

**Se pide:**

47. Intenta calcular `unidades * precios` con las listas de Python y explica en una frase el resultado o el error obtenido.
48. Convierte ambas listas a arrays NumPy (`np_unidades`, `np_precios`) e imprime `dtype` y `shape` de cada uno.
49. Calcula el array `importes = np_unidades * np_precios` y su suma total.
50. Aplica un recargo logístico del 4 % a todos los importes con una única operación vectorizada.
51. Crea un array mixto a partir de `[1, "dos", 3.0, True]` y explica qué `dtype` resulta y por qué NumPy fuerza la homogeneidad de tipos.
52. Compara el tiempo de calcular los importes con un bucle `for` frente a la versión vectorizada usando `%timeit` en una celda mágica de JupyterLab.
**Comprobación:** la suma de `importes` debe ser 2 362,00 € y hay 4 productos con precio unitario superior a 100 €.

---

### Ejercicio 9 — Subsetting con máscaras booleanas

Partiendo de los arrays `np_unidades`, `np_precios` e `importes` del ejercicio anterior:

**Se pide:**

53. Crea la máscara booleana `caros = np_precios > 100` e imprímela. Observa que es un array de `True` y `False` con la misma longitud que `np_precios`.
54. Usa la máscara para extraer los precios que superan 100 €: `np_precios[caros]`.
55. Cuenta cuántos productos caros hay con `caros.sum()` y explica en una frase por qué sumar booleanos devuelve un número.
56. Aplica la **misma máscara** al array `importes` para obtener los importes de esas líneas. Calcula su suma.
57. Obtén las posiciones que cumplen la condición con `np.where(caros)`.
58. Crea una segunda máscara `varias = np_unidades >= 2` y combina ambas con `&`, recordando que cada condición va entre paréntesis:
```python
seleccion = (np_precios > 100) & (np_unidades >= 2)
print(importes[seleccion])
```
**Comprobación:** hay 4 productos con precio unitario superior a 100 € y la suma de sus importes es 1 254,00 €.

> **Pista:** con NumPy no se usan `and` ni `or`, sino `&` (y) y `|` (o). Sin los paréntesis el resultado es un error, porque `&` se evalúa antes que `>`.

### Ejercicio 10 — Matriz de ventas por trimestre y región

```python
import numpy as np

# Filas: T1, T2, T3  |  Columnas: Norte, Sur, Este, Oeste, Centro
ventas_2d = np.array([
    [15200.50, 11800.25, 12950.00,  9870.40, 13400.10],
    [16840.75, 12310.60, 13480.90, 11020.35, 14100.80],
    [12535.72, 10663.06, 11221.59, 12673.08, 12334.38],
])
```

**Se pide:**

59. Imprime `shape`, `ndim` y `size` del array y explica qué representa cada uno en este contexto de negocio.
60. Extrae la fila del segundo trimestre y la columna de la región Centro.
61. Obtén el valor de T3 en la región Este mediante indexación 2D.
62. Extrae con *slicing* la submatriz de T1 y T2 para las tres primeras regiones.
63. Calcula el total por trimestre (`axis=1`) y el total por región (`axis=0`).
64. Aplica una previsión de crecimiento del 6 % a T3 sumando el resultado como una **cuarta fila** con `np.vstack()`.
65. Identifica la región con mayor facturación acumulada usando `np.argmax()` sobre los totales por región, y traduce el índice a nombre con una lista de etiquetas.
---

### Ejercicio 11 — Estadística descriptiva sobre telemetría simulada

**Se pide:**

66. Genera con `rng = np.random.default_rng(2024)` dos arrays de 500 elementos: `temperatura` con `rng.normal(68, 6.5, 500)` y `vibracion` con `np.abs(rng.normal(2.4, 0.8, 500))`.
67. Calcula media, mediana, desviación típica, mínimo y máximo de `temperatura`. Compara media y mediana y razona en una frase qué indica la diferencia.
68. Calcula los percentiles 25, 50, 75 y 95 de `vibracion` con `np.percentile()`.
69. Apila ambos arrays en una matriz de forma `(500, 2)` con `np.column_stack()` y verifica el `shape`.
70. Calcula la matriz de correlación con `np.corrcoef()` y explica en una frase si existe relación lineal entre temperatura y vibración en estos datos simulados.
71. Define `umbral = media + 2 * desviación` de la temperatura, cuenta cuántas lecturas lo superan y calcula qué porcentaje del total representan.
72. Explica en dos frases por qué fijar la semilla (`default_rng(2024)`) es imprescindible para que un análisis sea reproducible por un tercero.
---

## Sección 6. Diccionarios y Pandas

### Ejercicio 12 — Diccionarios simples y anidados

**Se pide:**

73. Crea el diccionario `tarifas_envio` con estos pares: `Norte: 4.95`, `Sur: 5.50`, `Este: 5.20`, `Oeste: 6.10`, `Centro: 3.90`.
74. Accede a la tarifa de `"Este"` e imprímela.
75. Añade la región `"Insular"` con tarifa `9.80` y actualiza `"Oeste"` a `5.95`.
76. Elimina `"Sur"` con `del` y comprueba con el operador `in` que ya no existe.
77. Recorre el diccionario con `.items()` e imprime una línea por región con formato `Norte -> 4.95 €`.
78. Construye el diccionario **anidado** `red_logistica` donde cada región sea una clave y su valor otro diccionario con las claves `almacen`, `tarifa` y `entrega_h`:
```python
red_logistica = {
    "Norte":  {"almacen": "ALM-NORTE", "tarifa": 4.95, "entrega_h": 24},
    "Este":   {"almacen": "ALM-ESTE",  "tarifa": 5.20, "entrega_h": 48},
    "Centro": {"almacen": "ALM-CENTRO","tarifa": 3.90, "entrega_h": 24},
}
```
79. Accede al tiempo de entrega del almacén del Este mediante doble clave.
80. Añade la región `"Oeste"` completa al diccionario anidado y recorre toda la estructura con un bucle anidado para imprimir un informe legible.
81. Explica en dos frases cuándo conviene un diccionario frente a una lista, en términos de acceso y de significado de la clave.
---

### Ejercicio 13 — De diccionario a DataFrame y lectura de CSV

**Se pide:**

82. Construye este diccionario y conviértelo en un DataFrame llamado `almacenes` con `pd.DataFrame()`:
```python
almacenes = {
    "codigo":    ["ALM-NORTE", "ALM-SUR", "ALM-ESTE", "ALM-OESTE", "ALM-CENTRO"],
    "ciudad":    ["Bilbao", "Sevilla", "Valencia", "Vigo", "Madrid"],
    "m2":        [4200, 3100, 3800, 2600, 6500],
    "operarios": [48, 31, 39, 22, 74],
}
```
83. Imprime el DataFrame y observa el índice numérico que pandas ha creado automáticamente.
84. Convierte la columna `codigo` en índice con `.set_index("codigo")` y vuelve a imprimirlo.
85. Carga los tres ficheros CSV:
```python
ventas = pd.read_csv("../data/ventas_retail.csv")
empleados = pd.read_csv("../data/empleados.csv", index_col="id_empleado")
sensores = pd.read_csv("../data/sensores_planta.csv")
```
86. Para `ventas`, muestra `head(8)`, `tail(3)` y `shape`.
87. Ejecuta `ventas.info()` e identifica qué columnas son numéricas y cuáles de texto.
88. Cuenta los valores ausentes por columna con `ventas.isna().sum()` e indica en una frase en qué columnas se concentran.
89. Imprime el `shape` de los tres DataFrames.
**Comprobación:** `ventas` debe tener 420 filas; `empleados`, 180; `sensores`, 600.

### Ejercicio 14 — Selección con corchetes, `loc` e `iloc`

Trabaja sobre el DataFrame `empleados` con `id_empleado` como índice.

**Se pide:**

90. Selecciona la columna `salario_base` como **Serie** y como **DataFrame**. Muestra el `type()` de cada resultado.
91. Selecciona las columnas `departamento`, `ciudad` y `salario_base` a la vez.
92. Extrae las filas de la 10 a la 15 (ambas incluidas en la posición) usando corchetes con *slicing*.
93. Con `loc`, obtén la fila completa del empleado `EMP-1042`.
94. Con `loc`, obtén el `departamento` y la `modalidad` de los empleados `EMP-1005`, `EMP-1020` y `EMP-1099`.
95. Con `iloc`, obtén las tres primeras filas y las columnas en posición 2 y 5.
96. Con `loc`, selecciona **todas las filas** y solo las columnas `nombre` y `bonus_pct`.
97. Explica en una tabla Markdown de tres filas la diferencia entre corchetes, `loc` e `iloc`, indicando si trabajan por etiqueta o por posición.
---

### Ejercicio 15 — Columnas calculadas, agregación y exportación

Trabaja sobre `ventas`.

**Se pide:**

98. Crea la columna `importe_bruto = unidades * precio_unitario`.
99. Crea la columna `importe_neto` aplicando `descuento_pct`, redondeada a dos decimales.
100. Crea la columna `ticket_medio = importe_neto / unidades`.
101. Agrupa por `region` y calcula la suma de `importe_neto`, ordenada de mayor a menor.
102. Agrupa por `categoria` y `canal` a la vez, y calcula `importe_neto` medio y `unidades` totales usando `.agg()` con un diccionario de funciones.
103. Obtén el **top 5 de productos** por facturación neta con `.groupby()`, `.sum()` y `.nlargest()`.
104. Rellena los nulos de `satisfaccion` con la mediana de la columna y los de `canal` con el texto `"Desconocido"`.
105. Exporta el resultado agrupado por región a `../outputs/facturacion_por_region.csv`.
**Comprobación:** la facturación neta total debe ser 190 402,48 € y la región líder, **Norte**.

---

## Sección 7. Lógica, Control de Flujo y Filtrado

### Ejercicio 16 — Operadores de comparación y booleanos

**Se pide:**

106. Evalúa e imprime el resultado de estas expresiones, razonando **una frase por cada una**:
```python
print(2 < 3)
print("Norte" == "norte")
print(True + True + False)
print(3 != 3.0)
print("Zapatilla" > "Mochila")
```
107. Define `unidades = 6`, `precio = 149.0` y `descuento = 15`. Construye una expresión booleana que sea `True` solo si el pedido tiene más de 4 unidades **y** un descuento mayor o igual a 10.
108. Construye otra que sea `True` si el precio es inferior a 50 **o** superior a 200.
109. Aplica `not` a la expresión del punto 2 e interpreta el resultado en lenguaje de negocio.
110. Crea los arrays `temp = np.array([64.2, 88.1, 71.0, 92.5, 69.8])` y `vib = np.array([1.9, 3.4, 2.1, 4.0, 2.2])`.
111. Intenta ejecutar `temp > 70 and vib > 3` y explica el `ValueError` que se produce.
112. Resuélvelo con `np.logical_and()` y también con la sintaxis `(temp > 70) & (vib > 3)`, comprobando que el resultado es idéntico.
---

### Ejercicio 17 — `if` / `elif` / `else` aplicado a mantenimiento predictivo

**Se pide:**

113. Escribe la función `nivel_riesgo(temperatura)` que reciba una temperatura y devuelva:
- `"CRÍTICO"` si supera 85 °C
- `"ALTO"` si es mayor que 76 °C
- `"MEDIO"` si es mayor que 70 °C
- `"BAJO"` en cualquier otro caso
```python
def nivel_riesgo(temperatura):
    if temperatura > 85:
        return "CRÍTICO"
    elif temperatura > 76:
        return "ALTO"
    elif temperatura > 70:
        return "MEDIO"
    else:
        return "BAJO"
```

114. Prueba la función con estos cuatro valores y comprueba que cada uno cae en una rama distinta: `92.5`, `80.0`, `72.3`, `64.1`.
115. Aplica la función a la columna de temperatura del DataFrame `sensores` para crear la columna `riesgo`. **Pasa el nombre de la función, sin paréntesis:**
```python
sensores["riesgo"] = sensores["temperatura_c"].apply(nivel_riesgo)
print(sensores[["id_maquina", "temperatura_c", "riesgo"]].head())
```
116. Cuenta cuántas lecturas hay de cada nivel con `sensores["riesgo"].value_counts()`.
117. Filtra las lecturas con riesgo `"CRÍTICO"` y averigua qué máquina acumula más, usando `value_counts()` sobre la columna `id_maquina` del resultado filtrado.
118. Escribe una celda Markdown con **una recomendación de negocio de dos líneas** basada en lo observado.
**Comprobación:** el fichero contiene 20 lecturas con `estado` igual a `ALARMA` y 72 con `AVISO`.

### Ejercicio 18 — Filtrado avanzado de DataFrames

Trabaja sobre `ventas` (con las columnas calculadas del ejercicio 15) y sobre `empleados`.

**Se pide:**

119. Filtra los pedidos del canal `"Online"` con `importe_neto` superior a 500 €.
120. Filtra los pedidos de las regiones `"Norte"` o `"Centro"` usando `.isin()`.
121. Filtra los pedidos con `descuento_pct` entre 10 y 20 (incluidos) usando `.between()`.
122. Combina tres condiciones con `&` y `|`, cuidando el uso de paréntesis: categoría `"Electronica"`, más de 2 unidades y descuento distinto de 0.
123. Invierte un filtro con `~` para obtener los pedidos que **no** son de la categoría `"Oficina"`.
124. Repite el filtro del punto 1 usando el método `.query()` y compara la legibilidad de ambas sintaxis en una frase.
125. Sobre `empleados`, obtén la plantilla de `"Ingenieria"` en modalidad `"Remoto"` con más de 5 años de antigüedad, ordenada por salario descendente.
126. Exporta ese resultado a `../outputs/ingenieria_remoto_senior.csv`.
> **Pista:** en pandas, `and` / `or` producen un `ValueError` sobre Series. Usa `&` y `|` con **cada condición entre paréntesis**, porque estos operadores tienen mayor precedencia que las comparaciones.

---

## Sección 8. Bucles

### Ejercicio 19 — Bucles sobre estructuras de datos

**Se pide:**

127. **`while`****:** simula la reposición de stock. Partiendo de `stock = 12` y un objetivo de 60 unidades, añade lotes de 8 unidades e imprime en cada vuelta el número de lote y el stock resultante. El bucle termina al alcanzar o superar el objetivo.
```python
stock = 12
lote = 0

while stock < 60:
    lote = lote + 1
    stock = stock + 8
    print("Lote", lote, "-> stock:", stock)
```
128. **`for`**** sobre lista:** recorre `["Norte", "Sur", "Este", "Oeste", "Centro"]` e imprime cada región en mayúsculas.
129. **`enumerate`****:** recorre la misma lista imprimiendo `"Región 1: Norte"`, `"Región 2: Sur"`, etc. Usa `enumerate(regiones, start=1)`.
130. **Lista de listas:** recorre el `inventario` del ejercicio 2 desempaquetando cada fila en la cabecera del bucle e imprime una línea por producto con su valor de stock:
```python
for almacen, producto, unidades, coste in inventario:
    print(almacen, "|", producto, "|", round(unidades * coste, 2), "€")
```
131. **Diccionario:** recorre `tarifas_envio` del ejercicio 12 con `.items()` e imprime una línea por región con su tarifa.
132. **Array NumPy:** recorre `np_precios` con un `for` simple e imprime cada precio con un recargo del 4 % aplicado.
133. Explica en **una frase** por qué al recorrer un diccionario hace falta `.items()` y qué se obtiene si no se usa.
### Ejercicio 20 — Iteración sobre DataFrames

Trabaja sobre `ventas`, con las columnas calculadas del ejercicio 15.

**Se pide:**

134. Recorre las **5 primeras filas** con `.iterrows()` e imprime la región, el producto y el importe neto de cada una:
```python
for etiqueta, fila in ventas.head(5).iterrows():
    print(fila["region"], "|", fila["producto"], "|", fila["importe_neto"], "€")
```
135. Crea la columna `codigo_region` con las **tres primeras letras en mayúscula** de la columna `region`, usando el accesor de texto de pandas:
```python
ventas["codigo_region"] = ventas["region"].str[:3].str.upper()
```
136. Crea la columna `segmento_ticket` aplicando con `.apply()` la función `clasificar_ticket` que importaste de `src/utilidades.py` sobre `importe_neto`. Recuerda pasar la función **sin paréntesis**.
137. Cuenta cuántos pedidos hay en cada segmento con `.value_counts()`.
138. Crea en `sensores` la columna `eficiencia` como `unidades_producidas / temperatura_c`, redondeada a tres decimales con `.round(3)`. Esta operación no necesita bucle: pandas la aplica a toda la columna de golpe.
139. Construye con un bucle `for` el diccionario `resumen_por_region`, cuyas claves sean las regiones y cuyos valores sean la facturación neta total de cada una:
```python
resumen_por_region = {}

for region in ventas["region"].unique():
    filtrado = ventas[ventas["region"] == region]
    resumen_por_region[region] = round(filtrado["importe_neto"].sum(), 2)

print(resumen_por_region)
```
140. Compara el diccionario anterior con el resultado del `.groupby("region")` del ejercicio 15 y confirma que las cifras coinciden. Explica en **una frase** cuál de las dos formas prefieres y por qué.
**Comprobación:** la región **Norte** debe encabezar el diccionario con 44 576,97 € de facturación neta.

