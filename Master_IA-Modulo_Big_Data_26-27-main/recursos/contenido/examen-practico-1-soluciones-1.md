# 👨🏽‍💻 Examen Práctico 1 — Soluciones (1)

---

# Parte 1: Scala

**Entorno:** Jupyter Lab con kernel Almond · Scala 2.12.x

> **Cómo leer las salidas de esta parte.** Almond muestra primero lo que la celda imprime con `println` y, a continuación, una línea `nombre: Tipo = valor` por cada definición de la celda, con el valor que tiene al terminar de ejecutarse. Las cadenas aparecen entre comillas en esas líneas automáticas, pero no en las de `println`. El número de `cmdN.sc` en los errores depende del orden en que se ejecuten las celdas, y el aspecto de las líneas automáticas puede variar ligeramente según la versión de Almond: lo que debe coincidir son los valores.

---

## Sección 1 — Variables, tipos e inferencia

### Pregunta 1

**Apartado 1**

```scala
val tarifaEnvioEstandar = 4.95
val unidadesPorPalet: Int = 48

println(tarifaEnvioEstandar)
println(unidadesPorPalet)
println(tarifaEnvioEstandar.getClass)
```

Salida:

```plain text
4.95
48
double
tarifaEnvioEstandar: Double = 4.95
unidadesPorPalet: Int = 48
```

`.getClass` devuelve `double` porque en tiempo de ejecución un `Double` de Scala es el primitivo `double` de la JVM.

**Apartado 2** (celda aparte)

```scala
tarifaEnvioEstandar = 5.50
```

Salida (error esperado):

```plain text
cmd1.sc:1: reassignment to val
val res1 = tarifaEnvioEstandar = 5.50
                               ^
Compilation Failed
```

---

### Pregunta 2

**Apartado 1**

```scala
var pedidosPendientes = 12
println(pedidosPendientes)

pedidosPendientes = 9
println(pedidosPendientes)
```

Salida:

```plain text
12
9
pedidosPendientes: Int = 9
```

**Apartado 2**

```scala
var facturacionAcumulada: Double = 0
facturacionAcumulada += 340.5
facturacionAcumulada += 120.0
println(facturacionAcumulada)
```

Salida:

```plain text
460.5
facturacionAcumulada: Double = 460.5
```

El literal `0` es un `Int`, pero al anotar `Double` el compilador lo convierte en `0.0`.

---

### Pregunta 3

**Apartado 1**

```scala
val codigoCliente = "CLI-00421"
val descuentoAplicado = 0.15
val importeBruto = 120.0
val importeNeto = importeBruto * (1 - descuentoAplicado)
val etiquetaCliente = codigoCliente * 2
```

Salida:

```plain text
codigoCliente: String = "CLI-00421"
descuentoAplicado: Double = 0.15
importeBruto: Double = 120.0
importeNeto: Double = 102.0
etiquetaCliente: String = "CLI-00421CLI-00421"
```

**El fragmento no produce ningún error de compilación.** En Scala un `String` se convierte implícitamente en `StringOps`, que define el método `*(n: Int)` y devuelve la cadena repetida `n` veces. Por eso `codigoCliente * 2` es válido y ya vale `"CLI-00421CLI-00421"`. La respuesta correcta al apartado es constatarlo, no forzar un error.

**Apartado 2**

No hay nada que corregir: la última línea ya produce el valor pedido. Se ejecuta el fragmento completo y se muestran los dos valores.

```scala
val codigoCliente = "CLI-00421"
val descuentoAplicado = 0.15
val importeBruto = 120.0
val importeNeto = importeBruto * (1 - descuentoAplicado)
val etiquetaCliente = codigoCliente * 2

println(importeNeto)
println(etiquetaCliente)
```

Salida:

```plain text
102.0
CLI-00421CLI-00421
codigoCliente: String = "CLI-00421"
descuentoAplicado: Double = 0.15
importeBruto: Double = 120.0
importeNeto: Double = 102.0
etiquetaCliente: String = "CLI-00421CLI-00421"
```

La versión `codigoCliente + codigoCliente` produce exactamente el mismo resultado.

---

## Sección 2 — Decisiones y funciones

### Pregunta 4

**Apartado 1**

```scala
val importePedido = 340.0

val categoriaPedido =
  if (importePedido >= 500) "Premium"
  else if (importePedido >= 100) "Estandar"
  else "Basico"

println(categoriaPedido)
```

Salida:

```plain text
Estandar
importePedido: Double = 340.0
categoriaPedido: String = "Estandar"
```

**Apartado 2**

```scala
val importePedido2 = 620.0
val categoriaPedido2 =
  if (importePedido2 >= 500) "Premium"
  else if (importePedido2 >= 100) "Estandar"
  else "Basico"

val importePedido3 = 75.0
val categoriaPedido3 =
  if (importePedido3 >= 500) "Premium"
  else if (importePedido3 >= 100) "Estandar"
  else "Basico"

println(s"$importePedido ->$categoriaPedido")
println(s"$importePedido2 ->$categoriaPedido2")
println(s"$importePedido3 ->$categoriaPedido3")
```

Salida:

```plain text
340.0 -> Estandar
620.0 -> Premium
75.0 -> Basico
importePedido2: Double = 620.0
categoriaPedido2: String = "Premium"
importePedido3: Double = 75.0
categoriaPedido3: String = "Basico"
```

---

### Pregunta 5

**Apartado 1**

```scala
def calcularRecargo(pesoKg: Double): Double = {
  if (pesoKg <= 5) 0.0
  else if (pesoKg <= 20) 3.5
  else 9.0
}
```

Salida:

```plain text
defined function calcularRecargo
```

**Apartado 2**

```scala
println(calcularRecargo(2.0))
println(calcularRecargo(12.5))
println(calcularRecargo(31.0))
```

Salida:

```plain text
0.0
3.5
9.0
```

Los límites son inclusivos: un peso de 5 da `0.0` y un peso de 20 da `3.5`.

---

## Sección 3 — Iteración: estilo imperativo y estilo funcional

### Pregunta 6

```scala
val unidadesPorAlmacen = Array(120, 45, 300, 78, 210, 15)
```

Salida:

```plain text
unidadesPorAlmacen: Array[Int] = Array(120, 45, 300, 78, 210, 15)
```

**Apartado 1**

```scala
var totalUnidades = 0
var i = 0

while (i < unidadesPorAlmacen.length) {
  totalUnidades += unidadesPorAlmacen(i)
  i += 1
}

println(s"Total de unidades:$totalUnidades")
```

Salida:

```plain text
Total de unidades: 768
totalUnidades: Int = 768
i: Int = 6
```

**Apartado 2**

```scala
var almacenesBajoStock = 0
var j = 0

while (j < unidadesPorAlmacen.length) {
  if (unidadesPorAlmacen(j) < 100) almacenesBajoStock += 1
  j += 1
}

println(s"Almacenes con menos de 100 unidades:$almacenesBajoStock")
```

Salida:

```plain text
Almacenes con menos de 100 unidades: 3
almacenesBajoStock: Int = 3
j: Int = 6
```

Los tres almacenes son los de 45, 78 y 15 unidades.

---

### Pregunta 7

**Apartado 1**

```scala
var totalUnidadesForeach = 0
unidadesPorAlmacen.foreach(unidades => totalUnidadesForeach += unidades)
println(s"Total de unidades:$totalUnidadesForeach")
```

Salida:

```plain text
Total de unidades: 768
totalUnidadesForeach: Int = 768
```

**Apartado 2**

```scala
unidadesPorAlmacen.foreach(unidades => println(s"Almacen con$unidades unidades"))
```

Salida:

```plain text
Almacen con 120 unidades
Almacen con 45 unidades
Almacen con 300 unidades
Almacen con 78 unidades
Almacen con 210 unidades
Almacen con 15 unidades
```

---

## Sección 4 — Colecciones: arrays y listas

### Pregunta 8

**Apartado 1**

```scala
val identificadoresContador = new Array[String](3)
println(identificadoresContador.mkString("Array(", ", ", ")"))

identificadoresContador(0) = "CNT-A01"
identificadoresContador(1) = "CNT-B02"
identificadoresContador(2) = "CNT-C03"

println(identificadoresContador.mkString("Array(", ", ", ")"))
```

Salida:

```plain text
Array(null, null, null)
Array(CNT-A01, CNT-B02, CNT-C03)
identificadoresContador: Array[String] = Array("CNT-A01", "CNT-B02", "CNT-C03")
```

Un array de `String` recién creado contiene `null` en todas sus posiciones. La línea automática del final muestra el array ya relleno porque Almond la escribe al terminar la celda. Con `println(identificadoresContador)` a secas se imprime la referencia interna (`[Ljava.lang.String;@...`), de ahí el `mkString`.

**Apartado 2**

```scala
val lecturasKwh = Array(154.2, 98.7, 230.5)
lecturasKwh(1) = 101.4
println(lecturasKwh.mkString("Array(", ", ", ")"))
```

Salida:

```plain text
Array(154.2, 101.4, 230.5)
lecturasKwh: Array[Double] = Array(154.2, 101.4, 230.5)
```

La segunda lectura es el índice `1`.

---

### Pregunta 9

**Apartado 1**

```scala
val referenciasIniciales = "REF-100" :: "REF-200" :: "REF-300" :: Nil
val referenciasAmpliadas = "REF-050" :: referenciasIniciales

println(referenciasIniciales)
println(referenciasAmpliadas)
```

Salida:

```plain text
List(REF-100, REF-200, REF-300)
List(REF-050, REF-100, REF-200, REF-300)
referenciasIniciales: List[String] = List("REF-100", "REF-200", "REF-300")
referenciasAmpliadas: List[String] = List("REF-050", "REF-100", "REF-200", "REF-300")
```

**Apartado 2**

```scala
val referenciasExternas = List("REF-900", "REF-901")
val catalogoCompleto = referenciasAmpliadas ::: referenciasExternas

var referenciasSerie9: List[String] = Nil
catalogoCompleto.foreach { referencia =>
  if (referencia.startsWith("REF-9")) referenciasSerie9 = referencia :: referenciasSerie9
}

println(catalogoCompleto)
println(referenciasSerie9)
```

Salida:

```plain text
List(REF-050, REF-100, REF-200, REF-300, REF-900, REF-901)
List(REF-901, REF-900)
referenciasExternas: List[String] = List("REF-900", "REF-901")
catalogoCompleto: List[String] = List("REF-050", "REF-100", "REF-200", "REF-300", "REF-900", "REF-901")
referenciasSerie9: List[String] = List("REF-901", "REF-900")
```

La lista resultante sale en orden inverso porque cada referencia se añade al principio con `::`. Con `.reverse` al final se obtiene `List(REF-900, REF-901)`; las dos salidas son correctas.

---

## Sección 5 — Del cuaderno al script

### Pregunta 10

**Apartado 1** — Celda Markdown con el contenido íntegro de `informe_ventas.scala`:

```scala
val ventasDiarias = Array(1250.0, 980.5, 1740.25, 640.0, 2100.75)

def clasificarJornada(importe: Double): String = {
  if (importe > 1500) "Jornada alta"
  else if (importe < 800) "Jornada baja"
  else "Jornada normal"
}

var totalFacturado = 0.0
var numeroJornada = 0

ventasDiarias.foreach { importe =>
  numeroJornada += 1
  println(s"Dia$numeroJornada:$importe ->${clasificarJornada(importe)}")
  totalFacturado += importe
}

println(s"Total facturado:$totalFacturado")
```

**Apartado 2** — El mismo código ejecutado en una celda de código.

Salida:

```plain text
Dia 1: 1250.0 -> Jornada normal
Dia 2: 980.5 -> Jornada normal
Dia 3: 1740.25 -> Jornada alta
Dia 4: 640.0 -> Jornada baja
Dia 5: 2100.75 -> Jornada alta
Total facturado: 6711.5
ventasDiarias: Array[Double] = Array(1250.0, 980.5, 1740.25, 640.0, 2100.75)
defined function clasificarJornada
totalFacturado: Double = 6711.5
numeroJornada: Int = 5
```

Celda Markdown con el comando de terminal:

```bash
scala informe_ventas.scala
```

Lanzado así desde la terminal, el script imprime las mismas seis líneas, sin las líneas automáticas de Almond. El formato de cada línea es libre siempre que muestre el importe y su clasificación; el total es **6711.5**. Un importe de exactamente 1500 o de 800 sería `Jornada normal`.

---

# Parte 2: SQL

**Base de datos:** Chinook 1.4.5 (script `chinook.sql` del examen) sobre PostgreSQL 18

> Los recuentos de las once tablas y el rango de fechas de facturación (del 2021-01-01 al 2025-12-22, 412 facturas) coinciden con los del enunciado.

---

## Sección 1 — Consultas sobre una sola tabla

### Pregunta 1

```sql
SELECT
    t.track_id,
    t.name                             AS pista,
    t.composer                         AS compositor,
    ROUND(t.milliseconds / 60000.0, 2) AS minutos
FROM track AS t
WHERE t.milliseconds BETWEEN 180000 AND 240000
  AND (t.composer LIKE '%Jagger%' OR t.composer LIKE '%Lennon%')
ORDER BY t.name,
         t.track_id;
```

Salida (15 filas):

| `track_id` | `pista` | `compositor` | `minutos` |
| --- | --- | --- | --- |
| 2671 | 19th Nervous Breakdown | Jagger/Richards | 3.97 |
| 2987 | Helter Skelter | Lennon, John/McCartney, Paul | 3.12 |
| 2675 | Let’s Spend The Night Together | Jagger/Richards | 3.62 |
| 2685 | Live With Me | Jagger/Richards | 3.91 |
| 2690 | Love Is Strong | Jagger/Richards | 3.85 |
| 2695 | Moon Is Up | Jagger/Richards | 3.70 |
| 2672 | Paint It Black | Jagger/Richards | 3.77 |
| 2719 | Paint It Black | Keith Richards/Mick Jagger | 3.58 |
| 2686 | Respectable | Jagger/Richards | 3.59 |
| 2674 | Ruby Tuesday | Jagger/Richards | 3.29 |
| 2667 | Satisfaction | Jagger/Richards | 3.78 |
| 2692 | Sparks Will Fly | Jagger/Richards | 3.27 |
| 2673 | Under My Thumb | Jagger/Richards | 3.69 |
| 2677 | You Got Me Rocking | Jagger/Richards | 3.43 |
| 2691 | You Got Me Rocking | Jagger/Richards | 3.60 |

Son 14 pistas de `Jagger` y 1 de `Lennon`. `Paint It Black` y `You Got Me Rocking` aparecen dos veces cada una: el enunciado solo pide ordenar por nombre, así que cualquier orden dentro de cada pareja es válido. El `track_id` del `ORDER BY` solo sirve para que la salida sea reproducible.

---

### Pregunta 2

```sql
SELECT
    COUNT(*)                                  AS total_pistas,
    ROUND(AVG(t.milliseconds) / 60000.0, 2)   AS duracion_media_min,
    ROUND(AVG(t.unit_price), 2)               AS precio_medio,
    ROUND(SUM(t.milliseconds) / 3600000.0, 1) AS catalogo_horas
FROM track AS t;
```

Salida (1 fila):

| `total_pistas` | `duracion_media_min` | `precio_medio` | `catalogo_horas` |
| --- | --- | --- | --- |
| 3503 | 6.56 | 1.05 | 383.0 |

El redondeo es opcional según el enunciado. Valores sin redondear: 6,5599… minutos, 1,0508… de precio medio y 382,9939 horas.

---

### Pregunta 3

```sql
SELECT
    i.billing_country      AS pais,
    COUNT(*)               AS num_facturas,
    ROUND(SUM(i.total), 2) AS facturado
FROM invoice AS i
WHERE i.invoice_date >= DATE '2024-01-01'
  AND i.invoice_date <  DATE '2025-01-01'
GROUP BY i.billing_country
HAVING SUM(i.total) > 40
ORDER BY facturado DESC;
```

Salida (3 filas):

| `pais` | `num_facturas` | `facturado` |
| --- | --- | --- |
| USA | 21 | 127.98 |
| Brazil | 9 | 53.46 |
| Canada | 9 | 42.57 |

El año va en `WHERE` y el umbral en `HAVING`. El siguiente país es Francia, con 36,66, que queda fuera. En total son 83 facturas y 477,53 en 2024.

---

### Pregunta 4

```sql
-- Desempate: a igual recuento va primero el playlist_id más bajo (la lista creada antes), para que el orden sea siempre el mismo.
SELECT
    pt.playlist_id,
    COUNT(*) AS num_pistas
FROM playlist_track AS pt
GROUP BY pt.playlist_id
ORDER BY num_pistas DESC,
         pt.playlist_id ASC
LIMIT 5;
```

Salida (5 filas):

| `playlist_id` | `num_pistas` |
| --- | --- |
| 1 | 3290 |
| 8 | 3290 |
| 5 | 1477 |
| 3 | 213 |
| 10 | 213 |

Hay dos empates: las listas 1 y 8 (3 290 pistas) y las listas 3 y 10 (213 pistas). La sexta lista tiene 75 pistas, así que el conjunto de cinco es siempre el mismo y solo cambia el orden dentro de cada pareja. Con desempate descendente la salida válida es 8, 1, 5, 10, 3.

---

## Sección 2 — Combinación de tablas

### Pregunta 5

```sql
SELECT
    c.country                     AS pais,
    COUNT(DISTINCT c.customer_id) AS clientes,
    ROUND(SUM(i.total), 2)        AS facturado
FROM customer AS c
INNER JOIN invoice AS i
        ON i.customer_id = c.customer_id
GROUP BY c.country
ORDER BY facturado DESC
LIMIT 5;
```

Salida (5 filas):

| `pais` | `clientes` | `facturado` |
| --- | --- | --- |
| USA | 13 | 523.06 |
| Canada | 8 | 303.96 |
| France | 5 | 195.10 |
| Brazil | 5 | 190.10 |
| Germany | 4 | 156.48 |

`COUNT(DISTINCT c.customer_id)` cuenta clientes; `COUNT(*)` contaría facturas y daría 91, 56, 35, 35 y 28.

---

### Pregunta 6

```sql
-- Total de pistas nunca vendidas en el catálogo completo: 1 519 (de 3 503).
SELECT
    g.name   AS genero,
    COUNT(*) AS pistas_sin_vender
FROM track AS t
INNER JOIN genre AS g
        ON g.genre_id = t.genre_id
LEFT JOIN invoice_line AS il
       ON il.track_id = t.track_id
WHERE il.invoice_line_id IS NULL
GROUP BY g.name
ORDER BY pistas_sin_vender DESC
LIMIT 5;
```

Salida (5 filas):

| `genero` | `pistas_sin_vender` |
| --- | --- |
| Rock | 552 |
| Latin | 239 |
| Metal | 143 |
| Alternative & Punk | 129 |
| Jazz | 62 |

El total del comentario sale de 3 503 pistas del catálogo menos 1 984 pistas distintas vendidas: **1 519**.

---

### Pregunta 7

```sql
SELECT
    e.employee_id,
    e.first_name || ' ' || e.last_name                           AS empleado,
    e.title                                                      AS puesto,
    COALESCE(s.first_name || ' ' || s.last_name, 'Sin superior') AS superior
FROM employee AS e
LEFT JOIN employee AS s
       ON s.employee_id = e.reports_to
ORDER BY e.employee_id;
```

Salida (8 filas):

| `employee_id` | `empleado` | `puesto` | `superior` |
| --- | --- | --- | --- |
| 1 | Andrew Adams | General Manager | Sin superior |
| 2 | Nancy Edwards | Sales Manager | Andrew Adams |
| 3 | Jane Peacock | Sales Support Agent | Nancy Edwards |
| 4 | Margaret Park | Sales Support Agent | Nancy Edwards |
| 5 | Steve Johnson | Sales Support Agent | Nancy Edwards |
| 6 | Michael Mitchell | IT Manager | Andrew Adams |
| 7 | Robert King | IT Staff | Michael Mitchell |
| 8 | Laura Callahan | IT Staff | Michael Mitchell |

---

### Pregunta 8

```sql
-- Combinaciones vacías: 87 de 125 posibles (5 formatos x 25 géneros).
SELECT
    mt.name  AS formato,
    COUNT(*) AS generos_sin_catalogo
FROM media_type AS mt
CROSS JOIN genre AS g
LEFT JOIN track AS t
       ON t.media_type_id = mt.media_type_id
      AND t.genre_id      = g.genre_id
WHERE t.track_id IS NULL
GROUP BY mt.name
ORDER BY generos_sin_catalogo DESC,
         formato;
```

Salida (5 filas):

| `formato` | `generos_sin_catalogo` |
| --- | --- |
| Purchased AAC audio file | 23 |
| AAC audio file | 19 |
| Protected MPEG-4 video file | 19 |
| Protected AAC audio file | 18 |
| MPEG audio file | 8 |

Existen 38 combinaciones con pistas, de ahí las **87 vacías de 125**. `AAC audio file` y `Protected MPEG-4 video file` empatan a 19, y cualquier orden entre ellas es válido.

---

## Sección 3 — Subconsultas, CTE y funciones de ventana

### Pregunta 9

```sql
SELECT
    CASE
        WHEN t.milliseconds > (SELECT AVG(t2.milliseconds)
                               FROM track AS t2
                               WHERE t2.album_id = t.album_id)
            THEN 'Por encima de la media de su album'
        ELSE 'En la media o por debajo'
    END      AS categoria,
    COUNT(*) AS num_pistas
FROM track AS t
GROUP BY categoria
ORDER BY num_pistas DESC;
```

Salida (2 filas):

| `categoria` | `num_pistas` |
| --- | --- |
| En la media o por debajo | 1944 |
| Por encima de la media de su album | 1559 |

Desglose: 1 559 pistas por encima de la media, 82 exactamente en la media (las de álbumes con una sola pista) y 1 862 por debajo.

---

### Pregunta 10

```sql
WITH ventas_mes AS (
    SELECT
        DATE_TRUNC('month', i.invoice_date)::date AS mes,
        SUM(i.total)                              AS facturado
    FROM invoice AS i
    WHERE i.invoice_date >= DATE '2024-01-01'
    GROUP BY DATE_TRUNC('month', i.invoice_date)
)
SELECT
    vm.mes,
    ROUND(vm.facturado, 2) AS facturado,
    ROUND(SUM(vm.facturado) OVER (
        PARTITION BY EXTRACT(YEAR FROM vm.mes)
        ORDER BY vm.mes
    ), 2)                  AS acumulado_anio,
    ROUND(AVG(vm.facturado) OVER (
        PARTITION BY EXTRACT(YEAR FROM vm.mes)
        ORDER BY vm.mes
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2)                  AS media_movil_3m
FROM ventas_mes AS vm
ORDER BY vm.mes;
```

Salida (24 filas):

| `mes` | `facturado` | `acumulado_anio` | `media_movil_3m` |
| --- | --- | --- | --- |
| 2024-01-01 | 37.62 | 37.62 | 37.62 |
| 2024-02-01 | 37.62 | 75.24 | 37.62 |
| 2024-03-01 | 37.62 | 112.86 | 37.62 |
| 2024-04-01 | 37.62 | 150.48 | 37.62 |
| 2024-05-01 | 37.62 | 188.10 | 37.62 |
| 2024-06-01 | 37.62 | 225.72 | 37.62 |
| 2024-07-01 | 39.62 | 265.34 | 38.29 |
| 2024-08-01 | 47.62 | 312.96 | 41.62 |
| 2024-09-01 | 46.71 | 359.67 | 44.65 |
| 2024-10-01 | 42.62 | 402.29 | 45.65 |
| 2024-11-01 | 37.62 | 439.91 | 42.32 |
| 2024-12-01 | 37.62 | 477.53 | 39.29 |
| 2025-01-01 | 37.62 | 37.62 | 37.62 |
| 2025-02-01 | 27.72 | 65.34 | 32.67 |
| 2025-03-01 | 37.62 | 102.96 | 34.32 |
| 2025-04-01 | 33.66 | 136.62 | 33.00 |
| 2025-05-01 | 37.62 | 174.24 | 36.30 |
| 2025-06-01 | 37.62 | 211.86 | 36.30 |
| 2025-07-01 | 37.62 | 249.48 | 37.62 |
| 2025-08-01 | 37.62 | 287.10 | 37.62 |
| 2025-09-01 | 37.62 | 324.72 | 37.62 |
| 2025-10-01 | 37.62 | 362.34 | 37.62 |
| 2025-11-01 | 49.62 | 411.96 | 41.62 |
| 2025-12-01 | 38.62 | 450.58 | 41.95 |

El acumulado y la media móvil se reinician en enero de 2025 por el `PARTITION BY` anual. La columna `mes` también es válida como `TIMESTAMP` (`2024-01-01 00:00:00`) si no se hace el `::date`.

---

# Parte 3: Python

**Entorno:** JupyterLab · Python 3.12 · NumPy 2.1.3 · pandas 2.2.3

> Los DataFrame se presentan como tablas Markdown, equivalentes a la tabla que pinta JupyterLab. Las preguntas 3, 5, 6, 7 y 10 piden además una celda Markdown: se incluye una respuesta modelo en cada una.

---

## Sección 1 — Celda de verificación del entorno

```python
import sys
import numpy as np
import pandas as pd

print("Python :", sys.version)
print("NumPy  :", np.__version__)
print("pandas :", pd.__version__)

assert sys.version_info[:2] == (3, 12), "El kernel no está ejecutando Python 3.12"
```

Salida:

```plain text
Python : 3.12.3 (main, Mar  3 2026, 12:15:18) [GCC 13.3.0]
NumPy  : 2.1.3
pandas : 2.2.3
```

La primera línea cambia según el sistema operativo; lo que debe coincidir es `3.12.x`, `2.1.3` y `2.2.3`.

---

## Sección 2 — Fundamentos de Python

### Pregunta 1

**Apartado 1**

```python
referencia = "MON-27-4K"
precio_unitario = 289.9
unidades = 14
es_urgente = True
recargo_urgencia = 18.5

print(type(referencia))
print(type(precio_unitario))
print(type(unidades))
print(type(es_urgente))
print(type(recargo_urgencia))
```

Salida:

```plain text
<class 'str'>
<class 'float'>
<class 'int'>
<class 'bool'>
<class 'float'>
```

**Apartado 2**

```python
importe_final = precio_unitario * unidades + recargo_urgencia * es_urgente

print(f"Pedido{referencia}:{unidades} uds. | Importe final:{importe_final:.2f} EUR")
```

Salida:

```plain text
Pedido MON-27-4K: 14 uds. | Importe final: 4077.10 EUR
```

En una expresión aritmética `True` vale `1` y `False` vale `0`, así que el recargo solo se suma cuando el pedido es urgente: `289.9 × 14 + 18.5 = 4077.10`.

---

### Pregunta 2

**Apartado 1**

```python
catalogo = [
    ["MON-27-4K", "Monitores", 42, 289.90],
    ["POR-14-I7", "Portatiles", 15, 1149.00],
    ["TEC-MEC-01", "Perifericos", 130, 79.95],
    ["SSD-1TB-NV", "Almacenamiento", 58, 112.50],
    ["ROU-WIFI6", "Redes", 24, 199.00],
]

print(catalogo[2][0])
print(catalogo[-1][3])

primeros_tres = catalogo[:3]
print(primeros_tres)
```

Salida:

```plain text
TEC-MEC-01
199.0
[['MON-27-4K', 'Monitores', 42, 289.9], ['POR-14-I7', 'Portatiles', 15, 1149.0], ['TEC-MEC-01', 'Perifericos', 130, 79.95]]
```

**Apartado 2**

```python
catalogo[3][2] = 46
catalogo[3][3] = 119.90

catalogo.append(["WEB-CAM-4K", "Perifericos", 67, 89.90])

del catalogo[4]

for producto in catalogo:
    print(producto)
```

Salida:

```plain text
['MON-27-4K', 'Monitores', 42, 289.9]
['POR-14-I7', 'Portatiles', 15, 1149.0]
['TEC-MEC-01', 'Perifericos', 130, 79.95]
['SSD-1TB-NV', 'Almacenamiento', 46, 119.9]
['WEB-CAM-4K', 'Perifericos', 67, 89.9]
```

`append` añade al final, de modo que `"ROU-WIFI6"` sigue en la posición `4` cuando se ejecuta el `del`.

**Apartado 3**

```python
referencias = [producto[0] for producto in catalogo]

print(sorted(referencias))
```

Salida:

```plain text
['MON-27-4K', 'POR-14-I7', 'SSD-1TB-NV', 'TEC-MEC-01', 'WEB-CAM-4K']
```

---

### Pregunta 3

**Apartado 1**

```python
def importe_neto(precio, unidades, descuento=0.0):
    return precio * unidades * (1 - descuento)

print(importe_neto(289.90, 14, 0.10))
print(importe_neto(79.95, 30, descuento=0.15))
print(importe_neto(1149.00, 2))
```

Salida:

```plain text
3652.74
2038.725
2298.0
```

Los valores de las llamadas son libres. Aquí hay una por posición, otra con `descuento` por nombre y una tercera que comprueba el valor por defecto.

**Apartado 2**

```python
texto = " monitor profesional 27 pulgadas "

texto_limpio = texto.strip().upper().replace(" ", "_")
print(texto_limpio)

import numpy as np

print(np.sqrt(2025))
```

Salida:

```plain text
MONITOR_PROFESIONAL_27_PULGADAS
45.0
```

**Apartado 3** — Celda Markdown (respuesta modelo)

> **Función.** Fragmento de código con nombre que se invoca por sí solo, pasándole los datos como argumentos. En esta pregunta: `importe_neto(...)`, definida con `def`, y `print(...)`, que es una función integrada de Python.
**Método.** Función que pertenece a un objeto y se invoca sobre él con un punto. En esta pregunta: `texto.strip()`, `.upper()` y `.replace(" ", "_")`, los tres métodos del tipo `str`. Como cada uno devuelve una cadena nueva, se pueden encadenar.

**Paquete.** Colección de módulos que amplía el lenguaje y que hay que importar antes de usarla. En esta pregunta: NumPy, cargado con `import numpy as np`. `np.sqrt(2025)` es una función que vive dentro de ese paquete.

---

### Pregunta 4

**Apartado 1**

```python
almacenes = {
    "Madrid":    {"capacidad": 12000, "ocupado": 10450, "operativo": True},
    "Barcelona": {"capacidad":  8500, "ocupado":  6120, "operativo": True},
    "Sevilla":   {"capacidad":  6000, "ocupado":  5880, "operativo": True},
    "Bilbao":    {"capacidad":  4500, "ocupado":  1980, "operativo": False},
    "Valencia":  {"capacidad":  7200, "ocupado":  6900, "operativo": True},
}

almacenes["Zaragoza"] = {"capacidad": 5400, "ocupado": 2700, "operativo": True}

for ciudad, datos in almacenes.items():
    porcentaje = datos["ocupado"] / datos["capacidad"] * 100
    print(f"{ciudad}:{porcentaje:.1f} %")
```

Salida:

```plain text
Madrid: 87.1 %
Barcelona: 72.0 %
Sevilla: 98.0 %
Bilbao: 44.0 %
Valencia: 95.8 %
Zaragoza: 50.0 %
```

**Apartado 2**

```python
en_riesgo = []

for ciudad, datos in almacenes.items():
    porcentaje = datos["ocupado"] / datos["capacidad"] * 100
    if datos["operativo"] and porcentaje > 85:
        en_riesgo.append(ciudad)

print(en_riesgo)
```

Salida:

```plain text
['Madrid', 'Sevilla', 'Valencia']
```

**Apartado 3**

```python
pendientes = 1500
lote = 250
capacidad = almacenes["Bilbao"]["capacidad"]
ocupado = almacenes["Bilbao"]["ocupado"]

while pendientes > 0 and ocupado < capacidad:
    descarga = min(lote, pendientes, capacidad - ocupado)
    pendientes -= descarga
    ocupado += descarga
    print(f"Pendientes:{pendientes} | Ocupado en Bilbao:{ocupado}")

almacenes["Bilbao"]["ocupado"] = ocupado
print(f"Ocupacion final de Bilbao:{ocupado / capacidad * 100:.1f} %")
```

Salida:

```plain text
Pendientes: 1250 | Ocupado en Bilbao: 2230
Pendientes: 1000 | Ocupado en Bilbao: 2480
Pendientes: 750 | Ocupado en Bilbao: 2730
Pendientes: 500 | Ocupado en Bilbao: 2980
Pendientes: 250 | Ocupado en Bilbao: 3230
Pendientes: 0 | Ocupado en Bilbao: 3480
Ocupacion final de Bilbao: 77.3 %
```

El bucle se detiene porque se agotan las unidades del camión: Bilbao tenía 2 520 huecos libres y solo llegan 1 500 unidades.

---

## Sección 3 — NumPy

### Pregunta 5

**Apartado 1**

```python
consumos = [140.5, 152.3, 128.9, 161.0, 147.2, 133.8]
arr_consumos = np.array(consumos)

print(arr_consumos.dtype)
print(arr_consumos.shape)
print(arr_consumos.ndim)

tarifa = 0.164
costes = arr_consumos * tarifa
print(costes)
```

Salida:

```plain text
float64
(6,)
1
[23.042  24.9772 21.1396 26.404  24.1408 21.9432]
```

**Apartado 2**

```python
try:
    costes_lista = consumos * tarifa
    print(costes_lista)
except TypeError as error:
    print("Error capturado:", error)
```

Salida:

```plain text
Error capturado: can't multiply sequence by non-int of type 'float'
```

**Apartado 3** — Celda Markdown (respuesta modelo)

> **Operaciones aritméticas.** Un array de NumPy aplica la operación elemento a elemento: `arr_consumos * 0.164` devuelve los seis costes sin escribir ningún bucle. En una lista, el operador `*` significa repetición y solo admite un entero. Con un `float` lanza `TypeError`, que es justo el error capturado.
**Tipo de los datos.** Un array es homogéneo: todos sus elementos comparten un único tipo, visible en `dtype` (`float64`), y tiene estructura dimensional (`shape`, `ndim`). Una lista puede mezclar tipos y no tiene esos atributos.

---

### Pregunta 6

**Apartado 1**

```python
datos = np.loadtxt("planta_sensores.csv", delimiter=",", skiprows=1)

print(datos.shape)
```

Salida:

```plain text
(60, 5)
```

**Apartado 2**

```python
temperatura = datos[:, 0]
unidades_prod = datos[:, 2]
consumo = datos[:, 3]
defectuosas = datos[:, 4]

variables = {
    "temperatura_c": temperatura,
    "consumo_kwh": consumo,
    "unidades_producidas": unidades_prod,
}

print(f"{'variable':<22}{'media':>10}{'mediana':>10}{'desv_tip':>10}{'minimo':>10}{'maximo':>10}")
for nombre, columna in variables.items():
    print(
        f"{nombre:<22}"
        f"{np.mean(columna):>10.2f}"
        f"{np.median(columna):>10.2f}"
        f"{np.std(columna):>10.2f}"
        f"{np.min(columna):>10.2f}"
        f"{np.max(columna):>10.2f}"
    )
```

Salida:

```plain text
variable                   media   mediana  desv_tip    minimo    maximo
temperatura_c              68.43     68.35      6.12     54.40     80.00
consumo_kwh               140.94    143.40     18.18     86.90    187.50
unidades_producidas       915.32    919.50     43.15    818.00   1022.00
```

La desviación típica es la poblacional, que NumPy calcula por defecto (`ddof=0`). Con la muestral (`ddof=1`, la que devuelve pandas) salen **6.18, 18.34 y 43.51**. El enunciado no fija cuál usar, así que ambas son válidas.

**Apartado 3**

```python
umbral = np.mean(consumo) + np.std(consumo)
turnos_sobre_umbral = np.sum(consumo > umbral)
porcentaje = np.mean(consumo > umbral) * 100

print(f"Umbral (media + 1 desviacion tipica):{umbral:.2f} kWh")
print(f"Turnos por encima del umbral:{turnos_sobre_umbral} de{consumo.size}")
print(f"Porcentaje:{porcentaje:.2f} %")
```

Salida:

```plain text
Umbral (media + 1 desviacion tipica): 159.13 kWh
Turnos por encima del umbral: 7 de 60
Porcentaje: 11.67 %
```

Con `ddof=1` el umbral es 159.28 kWh y el resultado es el mismo: 7 turnos, 11.67 %.

Celda Markdown (respuesta modelo)

> La media de `consumo_kwh` (140,94 kWh) queda por debajo de la mediana (143,40 kWh). Cuando la media es menor que la mediana, hay valores bajos que tiran de ella hacia abajo: la distribución presenta una **asimetría negativa**, con la cola hacia la izquierda. La causa es visible en el mínimo, un turno de 86,9 kWh muy alejado del resto.
La diferencia es pequeña, 2,46 kWh, así que la asimetría es leve y la distribución es casi simétrica.

---

### Pregunta 7

**Apartado 1**

```python
seleccion = datos[:, [0, 3, 2]]   # temperatura_c, consumo_kwh, unidades_producidas
correlacion = np.round(np.corrcoef(seleccion, rowvar=False), 3)

print(correlacion)
```

Salida:

```plain text
[[ 1.     0.955 -0.644]
 [ 0.955  1.    -0.638]
 [-0.644 -0.638  1.   ]]
```

Filas y columnas siguen el orden `temperatura_c`, `consumo_kwh`, `unidades_producidas`.

Celda Markdown (respuesta modelo)

> **Correlación positiva más fuerte: temperatura y consumo (0,955).** Es una relación casi lineal: los turnos más calientes son también los de mayor consumo eléctrico.
**Correlación negativa más fuerte: temperatura y unidades producidas (−0,644).** Es una relación inversa moderada: cuando sube la temperatura, la producción del turno tiende a bajar. La de consumo y unidades es prácticamente igual (−0,638), así que los turnos calientes resultan los menos eficientes.

La correlación mide asociación, no causalidad.

**Apartado 2**

```python
mascara = (temperatura > 72) & (defectuosas >= 8)

print("Turnos que cumplen la condicion:", np.sum(mascara))
print(f"Media de unidades (seleccionados):{np.mean(unidades_prod[mascara]):.2f}")
print(f"Media de unidades (resto)        :{np.mean(unidades_prod[~mascara]):.2f}")
print(f"Diferencia                       :{np.mean(unidades_prod[mascara]) - np.mean(unidades_prod[~mascara]):.2f}")
```

Salida:

```plain text
Turnos que cumplen la condicion: 7
Media de unidades (seleccionados): 868.86
Media de unidades (resto)        : 921.45
Diferencia                       : -52.60
```

Los 7 turnos seleccionados producen de media 52,6 unidades menos que los 53 restantes.

**Apartado 3**

```python
clasificacion = np.where(consumo > 150, "ALTO", "NORMAL")

etiquetas, recuentos = np.unique(clasificacion, return_counts=True)
for etiqueta, recuento in zip(etiquetas, recuentos):
    print(f"{etiqueta}:{recuento}")
```

Salida:

```plain text
ALTO: 21
NORMAL: 39
```

---

## Sección 4 — pandas

### Pregunta 8

Los datos los inventa cada alumno, así que las salidas de esta pregunta son un ejemplo. Lo que debe coincidir es la estructura: cinco proveedores, las cinco claves pedidas, dimensiones `(5, 5)`, `pedidos_mensuales` con un decimal y la ordenación descendente por `valoracion` con tres columnas.

**Apartado 1**

```python
proveedores = {
    "proveedor": ["Iberchip", "NordData", "TecnoSur", "Altavia Systems", "LogiPort"],
    "pais": ["España", "Alemania", "Portugal", "Francia", "Italia"],
    "pedidos_anuales": [148, 96, 210, 64, 132],
    "plazo_medio_dias": [4.5, 7.0, 3.2, 9.5, 5.8],
    "valoracion": [4.6, 3.9, 4.8, 3.4, 4.2],
}

df_prov = pd.DataFrame(proveedores)
df_prov
```

Salida:

|  | `proveedor` | `pais` | `pedidos_anuales` | `plazo_medio_dias` | `valoracion` |
| --- | --- | --- | --- | --- | --- |
| 0 | Iberchip | España | 148 | 4.5 | 4.6 |
| 1 | NordData | Alemania | 96 | 7.0 | 3.9 |
| 2 | TecnoSur | Portugal | 210 | 3.2 | 4.8 |
| 3 | Altavia Systems | Francia | 64 | 9.5 | 3.4 |
| 4 | LogiPort | Italia | 132 | 5.8 | 4.2 |

**Apartado 2**

```python
df_prov.head(3)
```

Salida:

|  | `proveedor` | `pais` | `pedidos_anuales` | `plazo_medio_dias` | `valoracion` |
| --- | --- | --- | --- | --- | --- |
| 0 | Iberchip | España | 148 | 4.5 | 4.6 |
| 1 | NordData | Alemania | 96 | 7.0 | 3.9 |
| 2 | TecnoSur | Portugal | 210 | 3.2 | 4.8 |

```python
print(df_prov.shape)
```

Salida:

```plain text
(5, 5)
```

```python
print(df_prov.dtypes)
```

Salida:

```plain text
proveedor            object
pais                 object
pedidos_anuales       int64
plazo_medio_dias    float64
valoracion          float64
dtype: object
```

```python
df_prov.describe()
```

Salida:

|  | `pedidos_anuales` | `plazo_medio_dias` | `valoracion` |
| --- | --- | --- | --- |
| count | 5.000000 | 5.000000 | 5.00000 |
| mean | 130.000000 | 6.000000 | 4.18000 |
| std | 55.317267 | 2.417643 | 0.55857 |
| min | 64.000000 | 3.200000 | 3.40000 |
| 25% | 96.000000 | 4.500000 | 3.90000 |
| 50% | 132.000000 | 5.800000 | 4.20000 |
| 75% | 148.000000 | 7.000000 | 4.60000 |
| max | 210.000000 | 9.500000 | 4.80000 |

**Apartado 3**

```python
df_prov["pedidos_mensuales"] = (df_prov["pedidos_anuales"] / 12).round(1)
df_prov
```

Salida:

|  | `proveedor` | `pais` | `pedidos_anuales` | `plazo_medio_dias` | `valoracion` | `pedidos_mensuales` |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | Iberchip | España | 148 | 4.5 | 4.6 | 12.3 |
| 1 | NordData | Alemania | 96 | 7.0 | 3.9 | 8.0 |
| 2 | TecnoSur | Portugal | 210 | 3.2 | 4.8 | 17.5 |
| 3 | Altavia Systems | Francia | 64 | 9.5 | 3.4 | 5.3 |
| 4 | LogiPort | Italia | 132 | 5.8 | 4.2 | 11.0 |

```python
df_prov.sort_values("valoracion", ascending=False)[["proveedor", "valoracion", "plazo_medio_dias"]]
```

Salida:

|  | `proveedor` | `valoracion` | `plazo_medio_dias` |
| --- | --- | --- | --- |
| 2 | TecnoSur | 4.8 | 3.2 |
| 0 | Iberchip | 4.6 | 4.5 |
| 4 | LogiPort | 4.2 | 5.8 |
| 1 | NordData | 3.9 | 7.0 |
| 3 | Altavia Systems | 3.4 | 9.5 |

---

### Pregunta 9

**Apartado 1**

```python
pedidos = pd.read_csv("pedidos_ecommerce.csv", parse_dates=["fecha"])

pedidos.head()
```

Salida:

|  | `id_pedido` | `fecha` | `cliente` | `pais` | `segmento` | `categoria` | `canal` | `unidades` | `precio_unitario` | `descuento` | `dias_envio` | `devuelto` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | PED-2554 | 2025-01-10 | Cliente_029 | Francia | PYME | Almacenamiento | Web | 13 | 109.38 | 0.10 | 3 | Si |
| 1 | PED-2522 | 2025-01-11 | Cliente_027 | Francia | Autonomo | Monitores | Marketplace | 4 | 324.04 | 0.00 | 11 | No |
| 2 | PED-2576 | 2025-01-11 | Cliente_024 | Alemania | PYME | Monitores | Marketplace | 4 | 301.90 | 0.10 | 4 | No |
| 3 | PED-2512 | 2025-01-14 | Cliente_025 | Portugal | Corporativo | Perifericos | Comercial | 13 | 49.12 | 0.00 | 5 | No |
| 4 | PED-2585 | 2025-01-15 | Cliente_009 | España | PYME | Perifericos | Comercial | 10 | 43.54 | 0.05 | 2 | No |

```python
print(pedidos.shape)
print(pedidos["fecha"].dtype)
```

Salida:

```plain text
(90, 12)
datetime64[ns]
```

```python
pedidos["importe"] = (
    pedidos["unidades"] * pedidos["precio_unitario"] * (1 - pedidos["descuento"])
).round(2)

pedidos[["id_pedido", "unidades", "precio_unitario", "descuento", "importe"]].head()
```

Salida:

|  | `id_pedido` | `unidades` | `precio_unitario` | `descuento` | `importe` |
| --- | --- | --- | --- | --- | --- |
| 0 | PED-2554 | 13 | 109.38 | 0.10 | 1279.75 |
| 1 | PED-2522 | 4 | 324.04 | 0.00 | 1296.16 |
| 2 | PED-2576 | 4 | 301.90 | 0.10 | 1086.84 |
| 3 | PED-2512 | 13 | 49.12 | 0.00 | 638.56 |
| 4 | PED-2585 | 10 | 43.54 | 0.05 | 413.63 |

**Apartado 2**

```python
filtro = (
    (pedidos["importe"] > 1500)
    & (pedidos["canal"] != "Comercial")
    & (pedidos["dias_envio"] <= 7)
)
pedidos_filtrados = pedidos[filtro]

print("Pedidos que cumplen el filtro:", len(pedidos_filtrados))

(
    pedidos_filtrados
    .sort_values("importe", ascending=False)
    [["id_pedido", "pais", "canal", "dias_envio", "importe"]]
    .head(5)
)
```

Salida:

```plain text
Pedidos que cumplen el filtro: 32
```

|  | `id_pedido` | `pais` | `canal` | `dias_envio` | `importe` |
| --- | --- | --- | --- | --- | --- |
| 63 | PED-2508 | Alemania | Marketplace | 7 | 15743.70 |
| 6 | PED-2552 | Francia | Web | 4 | 13675.14 |
| 43 | PED-2539 | Alemania | Marketplace | 4 | 13615.70 |
| 89 | PED-2587 | Italia | Web | 1 | 13060.97 |
| 55 | PED-2502 | Portugal | Web | 2 | 11463.76 |

**Apartado 3**

```python
pedidos["envio_lento"] = (pedidos["dias_envio"] > 8) | (pedidos["devuelto"] == "Si")

print("Pedidos marcados:", pedidos["envio_lento"].sum(), "de", len(pedidos))
print(f"Porcentaje sobre el total:{pedidos['envio_lento'].mean() * 100:.2f} %")
```

Salida:

```plain text
Pedidos marcados: 36 de 90
Porcentaje sobre el total: 40.00 %
```

Hay 25 pedidos con más de 8 días de envío y 12 devueltos; uno cumple las dos condiciones, de ahí `25 + 12 − 1 = 36`.

> **Tolerancia de un céntimo.** Tres pedidos tienen un importe exacto que termina en medio céntimo: `PED-2544` (1124.125), `PED-2542` (875.925) y `PED-2511` (1385.865). `Series.round(2)` los deja en 1124.12, 875.92 y 1385.87. Con `round()` de Python aplicado fila a fila, `PED-2542` sale 875.93. Una diferencia de un céntimo en esas filas no es un error de cálculo.

---

### Pregunta 10

**Apartado 1**

```python
resumen_pais = (
    pedidos.groupby("pais")
    .agg(
        importe_total=("importe", "sum"),
        importe_medio=("importe", "mean"),
        num_pedidos=("importe", "count"),
    )
    .sort_values("importe_total", ascending=False)
)

resumen_pais.round(2)
```

Salida:

| `pais` | `importe_total` | `importe_medio` | `num_pedidos` |
| --- | --- | --- | --- |
| España | 105693.73 | 3302.93 | 32 |
| Alemania | 56864.13 | 4738.68 | 12 |
| Italia | 43720.99 | 3643.42 | 12 |
| México | 35564.17 | 2963.68 | 12 |
| Francia | 32820.28 | 2983.66 | 11 |
| Portugal | 30891.69 | 2808.34 | 11 |

```python
for pais, fila in resumen_pais.iterrows():
    print(
        f"{pais} |{int(fila['num_pedidos'])} pedidos | "
        f"Total:{fila['importe_total']:.2f} EUR | "
        f"Media:{fila['importe_medio']:.2f} EUR"
    )
```

Salida:

```plain text
España | 32 pedidos | Total: 105693.73 EUR | Media: 3302.93 EUR
Alemania | 12 pedidos | Total: 56864.13 EUR | Media: 4738.68 EUR
Italia | 12 pedidos | Total: 43720.99 EUR | Media: 3643.42 EUR
México | 12 pedidos | Total: 35564.17 EUR | Media: 2963.68 EUR
Francia | 11 pedidos | Total: 32820.28 EUR | Media: 2983.66 EUR
Portugal | 11 pedidos | Total: 30891.69 EUR | Media: 2808.34 EUR
```

**Apartado 2**

```python
tabla_cruzada = pedidos.pivot_table(
    index="categoria",
    columns="canal",
    values="importe",
    aggfunc="sum",
    fill_value=0,
)

tabla_cruzada.round(2)
```

Salida:

| `categoria / canal` | `Comercial` | `Marketplace` | `Web` |
| --- | --- | --- | --- |
| Almacenamiento | 3697.27 | 8966.36 | 9800.95 |
| Monitores | 11071.05 | 19388.31 | 13759.61 |
| Perifericos | 3321.52 | 3188.04 | 3077.40 |
| Portatiles | 40075.69 | 70020.74 | 86749.82 |
| Redes | 3172.12 | 16807.08 | 12459.03 |

**Apartado 3**

```python
pedidos["mes"] = pedidos["fecha"].dt.month

importe_por_mes = pedidos.groupby("mes")["importe"].sum()
print(importe_por_mes.round(2))
```

Salida:

```plain text
mes
1      6167.10
2     20056.34
3     24566.13
4     23520.44
5     21442.38
6     22808.63
7     64047.49
8     11786.03
9     27913.42
10    15474.80
11    54711.26
12    13060.97
Name: importe, dtype: float64
```

```python
mes_top = importe_por_mes.idxmax()
total_anual = importe_por_mes.sum()
peso = importe_por_mes.max() / total_anual * 100

print(f"Mes de mayor facturacion:{mes_top}")
print(f"Importe del mes:{importe_por_mes.max():.2f} EUR")
print(f"Total del año:{total_anual:.2f} EUR")
print(f"Peso sobre el total:{peso:.2f} %")
```

Salida:

```plain text
Mes de mayor facturacion: 7
Importe del mes: 64047.49 EUR
Total del año: 305554.99 EUR
Peso sobre el total: 20.96 %
```

Celda Markdown (respuesta modelo)

> El mes de mayor facturación ha sido **julio (mes 7)**, con 64 047,49 EUR. Sobre un total anual de 305 554,99 EUR, representa el **20,96 %**: uno de cada cinco euros del año se facturó en ese mes.

> **Tolerancia de céntimos.** Si se agrega sobre el importe sin redondear, o se redondea con otro método, algunos totales cambian en uno o dos céntimos: España 105693.75, Portugal 30891.68 con media 2808.33, julio 64047.51, enero 6167.09, marzo 24566.12, mayo 21442.37, Portatiles–Web 86749.83 y Portatiles–Marketplace 70020.75. Es efecto del redondeo, no un error de cálculo. El 20,96 % de julio se mantiene en todos los casos.

