# 📋Clase 12 - CTE’s

## 2. CTE (Common Table Expressions)

![Imagen](images/clase-12-cte-s/image-001.png)

### Analogía

![Imagen](images/clase-12-cte-s/image-002.png)

### Sintaxis

```sql
-- CTE simple
WITH nombre_cte AS (
    SELECT ...
)
SELECT ... FROM nombre_cte;

-- CTE encadenadas
WITH paso1 AS (
    SELECT ...
),
paso2 AS (
    SELECT ... FROM paso1 ...
),
paso3 AS (
    SELECT ... FROM paso2 ...
)
SELECT ... FROM paso3;

-- CTE recursiva
WITH nombre_cte AS (
    -- Miembro ancla: punto de partida
    SELECT ... FROM tabla WHERE condicion_inicial
    UNION ALL
    -- Miembro recursivo: se une con la propia CTE
    SELECT ... FROM tabla AS t JOIN nombre_cte AS c ON t.padre_id = c.id
)
SELECT ... FROM nombre_cte
OPTION (MAXRECURSION n);
```

**Reglas de T-SQL que conviene recordar:**

- Si hay otra instrucción antes del `WITH`, esa instrucción debe terminar en punto y coma. Si no, aparece el error **Msg 319**. Por eso muchos equipos escriben `;WITH`.
- Una CTE solo vive durante la instrucción que la sigue. En la siguiente instrucción ya no existe.
- No admite `ORDER BY` dentro, salvo con `TOP` u `OFFSET`.
- En una CTE recursiva, las columnas del ancla y del miembro recursivo deben tener **exactamente el mismo tipo**. Si no coinciden, aparece el error **Msg 240**.
- Por defecto, la recursión se detiene a los **100 niveles** (error **Msg 530**). Se cambia con `OPTION (MAXRECURSION n)`, donde `0` significa sin límite.
![Imagen](images/clase-12-cte-s/image-003.png)

>
### **Características**

![Imagen](images/clase-12-cte-s/image-004.gif)

- **Legibilidad mejorada**: Divide consultas complejas en partes lógicas más simples
- **Reutilización**: Puedes referenciar el mismo CTE múltiples veces en la misma consulta
- **Recursividad**: Soporta consultas recursivas (aunque no lo cubriremos en esta clase básica)
- **Ámbito limitado**: Solo existe durante la ejecución de la consulta principal
- **Alternativa a subconsultas**: Más legible que subconsultas anidadas complejas
- **Mantenibilidad**: Facilita el debugging y modificaciones futuras
- **No almacena datos**: No es una tabla física, solo un resultado temporal en memoria

![Imagen](images/clase-12-cte-s/image-005.png)

![Imagen](images/clase-12-cte-s/image-006.png)

![Imagen](images/clase-12-cte-s/image-007.png)

![Imagen](images/clase-12-cte-s/image-008.png)

![Imagen](images/clase-12-cte-s/image-009.png)

![Imagen](images/clase-12-cte-s/image-010.png)

![Imagen](images/clase-12-cte-s/image-011.png)

![Imagen](images/clase-12-cte-s/image-012.png)

![Imagen](images/clase-12-cte-s/image-013.png)

![Imagen](images/clase-12-cte-s/image-014.png)

![Imagen](images/clase-12-cte-s/image-015.png)

![Imagen](images/clase-12-cte-s/image-016.png)

![Imagen](images/clase-12-cte-s/image-017.png)

![Imagen](images/clase-12-cte-s/image-018.png)

![Imagen](images/clase-12-cte-s/image-019.png)

![Imagen](images/clase-12-cte-s/image-020.png)

![Imagen](images/clase-12-cte-s/image-021.png)

![Imagen](images/clase-12-cte-s/image-022.png)

![Imagen](images/clase-12-cte-s/image-023.png)

![Imagen](images/clase-12-cte-s/image-024.png)

> 💡
1. Ejecutar todas las querys de ejemplo del [script compartido en clase](https://tajamar365.sharepoint.com/:f:/s/3601-MasterIA2026-2027/IgAGzO5PiswXRIemIgzl7C8jAeyQ_CbBXIHErxM2SonTgbQ?e=ujJU9k), muestra las salidas.
2. Las siguientes imágenes muestran el paso paso de una CTE recursiva. Ejecuta el código y muestra la salida. Es el último ejercicio del script: `01_CTE.sql`
3. Una vez finalizado el apartado 2 crear un ejemplo propio de CTE recursiva. 
![Imagen](images/clase-12-cte-s/image-025.png)

![Imagen](images/clase-12-cte-s/image-026.png)

![Imagen](images/clase-12-cte-s/image-027.png)

![Imagen](images/clase-12-cte-s/image-028.png)

# Ejemplos resueltos

> 💡 Ejecuta los ejemplos que se muestran a continuación. Para cada ejemplo de CTE añade también su equivalente pero usando Subconsultas.

### Ejemplo 9. CTE simple

**Pregunta de negocio:** ¿Cuáles son las 3 canciones con más reproducciones válidas?

**Paso 1.** La CTE `validas` aplica la regla de negocio (canción y al menos 30 segundos).
**Paso 2.** La consulta final cuenta por canción y se queda con las 3 primeras.

```sql
WITH validas AS (
    SELECT cancion_id
    FROM dbo.reproducciones
    WHERE tipo_contenido = N'Canción'
      AND segundos_escuchados >= 30
)
SELECT TOP (3) c.titulo, COUNT(*) AS reproducciones_validas
FROM validas AS v
JOIN dbo.canciones AS c ON c.cancion_id = v.cancion_id
GROUP BY c.titulo
ORDER BY reproducciones_validas DESC, c.titulo;
```

**Salida esperada:**

| titulo | reproducciones_validas |
| --- | --- |
| Perreo Lunar | 6 |
| Corriente Alterna | 3 |
| Galaxia Dembow | 3 |

### Ejemplo 10. CTE encadenadas como pipeline

**Pregunta de negocio:** ¿Cuántos minutos válidos se han escuchado por género?

La consulta se escribe como un pequeño pipeline: **limpiar → enriquecer → agregar**.

```sql
WITH limpias AS (          -- 1. Limpiar: solo reproducciones válidas
    SELECT cancion_id, segundos_escuchados
    FROM dbo.reproducciones
    WHERE tipo_contenido = N'Canción'
      AND segundos_escuchados >= 30
),
enriquecidas AS (          -- 2. Enriquecer: añadir el género
    SELECT l.segundos_escuchados, g.nombre AS genero
    FROM limpias AS l
    JOIN dbo.canciones AS c ON c.cancion_id = l.cancion_id
    JOIN dbo.generos   AS g ON g.genero_id  = c.genero_id
),
agregadas AS (             -- 3. Agregar: totales por género
    SELECT genero,
           COUNT(*) AS reproducciones,
           CAST(SUM(segundos_escuchados) / 60.0 AS decimal(6,1)) AS minutos
    FROM enriquecidas
    GROUP BY genero
)
SELECT genero, reproducciones, minutos
FROM agregadas
ORDER BY minutos DESC, genero;
```

**Salida esperada:**

| genero | reproducciones | minutos |
| --- | --- | --- |
| Reguetón | 8 | 23.9 |
| Indie | 5 | 18.3 |
| House | 2 | 11.9 |
| Techno | 2 | 10.2 |
| Pop | 3 | 9.5 |
| Trap | 3 | 8.6 |
| Dembow | 3 | 7.9 |
| Hip Hop | 2 | 7.1 |
| Metal | 2 | 7.1 |

> Para depurar un pipeline de CTE, cambia el `SELECT` final por `SELECT * FROM limpias` o `SELECT * FROM enriquecidas` y revisa cada paso por separado. Se divide en `60.0` y no en `60` para evitar la división entera.

### Ejemplo 11. CTE recursiva: árbol de géneros

**Pregunta de negocio:** mostrar el árbol completo de géneros con su nivel de profundidad y su ruta.

**Paso 1. Ancla:** el género raíz, el que no tiene padre (`Música`), con nivel 0.
**Paso 2. Miembro recursivo:** los géneros cuyo padre ya está en la CTE, con nivel + 1 y la ruta ampliada.
**Paso 3.** La recursión termina cuando una vuelta no encuentra más hijos.

```sql
WITH arbol AS (
    SELECT genero_id, nombre, genero_padre_id,
           0 AS nivel,
           CAST(nombre AS nvarchar(200)) AS ruta
    FROM dbo.generos
    WHERE genero_padre_id IS NULL

    UNION ALL

    SELECT g.genero_id, g.nombre, g.genero_padre_id,
           a.nivel + 1,
           CAST(a.ruta + N' > ' + g.nombre AS nvarchar(200))
    FROM dbo.generos AS g
    JOIN arbol AS a ON g.genero_padre_id = a.genero_id
)
SELECT genero_id, nombre, nivel, ruta
FROM arbol
ORDER BY ruta;
```

**Salida esperada:**

| genero_id | nombre | nivel | ruta |
| --- | --- | --- | --- |
| 1 | Música | 0 | Música |
| 5 | Electrónica | 1 | Música > Electrónica |
| 11 | House | 2 | Música > Electrónica > House |
| 12 | Techno | 2 | Música > Electrónica > Techno |
| 2 | Pop | 1 | Música > Pop |
| 4 | Rock | 1 | Música > Rock |
| 9 | Indie | 2 | Música > Rock > Indie |
| 10 | Metal | 2 | Música > Rock > Metal |
| 3 | Urbano | 1 | Música > Urbano |
| 8 | Hip Hop | 2 | Música > Urbano > Hip Hop |
| 6 | Reguetón | 2 | Música > Urbano > Reguetón |
| 13 | Dembow | 3 | Música > Urbano > Reguetón > Dembow |
| 7 | Trap | 2 | Música > Urbano > Trap |

> Los dos `CAST(... AS nvarchar(200))` son obligatorios. Sin ellos, el ancla devuelve `nvarchar(50)` (el tipo de `nombre`) y el miembro recursivo un `nvarchar` más largo, y SQL Server lanza el error Msg 240 por tipos distintos.

### Ejemplo 12. CTE recursiva combinada con agregación

**Pregunta de negocio:** ¿cuántas reproducciones válidas tiene cada género principal, **contando todos sus subgéneros**? Por ejemplo, Urbano debe sumar Reguetón, Trap, Hip Hop y también Dembow, que cuelga de Reguetón.

**Paso 1. Ancla:** los géneros principales (hijos directos de la raíz). Cada uno se guarda como su propia raíz (`raiz_id`).
**Paso 2. Miembro recursivo:** los descendientes heredan el `raiz_id` de su antepasado.
**Paso 3.** Se unen canciones y reproducciones y se agrupa por la raíz.

```sql
WITH arbol AS (
    SELECT genero_id, genero_id AS raiz_id
    FROM dbo.generos
    WHERE genero_padre_id = (SELECT genero_id FROM dbo.generos WHERE genero_padre_id IS NULL)

    UNION ALL

    SELECT g.genero_id, a.raiz_id
    FROM dbo.generos AS g
    JOIN arbol AS a ON g.genero_padre_id = a.genero_id
)
SELECT gr.nombre AS genero_principal,
       COUNT(r.reproduccion_id) AS reproducciones_validas
FROM arbol AS a
JOIN dbo.generos AS gr ON gr.genero_id = a.raiz_id
LEFT JOIN dbo.canciones AS c ON c.genero_id = a.genero_id
LEFT JOIN dbo.reproducciones AS r
       ON r.cancion_id = c.cancion_id
      AND r.tipo_contenido = N'Canción'
      AND r.segundos_escuchados >= 30
GROUP BY gr.nombre
ORDER BY reproducciones_validas DESC;
```

**Salida esperada:**

| genero_principal | reproducciones_validas |
| --- | --- |
| Urbano | 16 |
| Rock | 7 |
| Electrónica | 4 |
| Pop | 3 |

> Fíjate en que el ancla combina una CTE con una subconsulta escalar para localizar la raíz sin escribir su id a mano.

### Ejemplo 13. CTE recursiva para generar fechas y `MAXRECURSION`

**Pregunta de negocio:** reproducciones de canciones por día entre el 14/09 y el 22/09, **incluidos los días sin actividad**.

Si agrupamos directamente la tabla `reproducciones`, los días sin datos no aparecen. La solución es generar primero la lista completa de días y después unirla con `LEFT JOIN`.

```sql
WITH dias AS (
    SELECT CAST('2026-09-14' AS date) AS dia
    UNION ALL
    SELECT DATEADD(DAY, 1, dia)
    FROM dias
    WHERE dia < '2026-09-22'
)
SELECT d.dia, COUNT(r.reproduccion_id) AS reproducciones
FROM dias AS d
LEFT JOIN dbo.reproducciones AS r
       ON CAST(r.fecha_hora AS date) = d.dia
      AND r.tipo_contenido = N'Canción'
GROUP BY d.dia
ORDER BY d.dia
OPTION (MAXRECURSION 400);
```

**Salida esperada:**

| dia | reproducciones |
| --- | --- |
| 2026-09-14 | 6 |
| 2026-09-15 | 5 |
| 2026-09-16 | 6 |
| 2026-09-17 | 3 |
| 2026-09-18 | 3 |
| 2026-09-19 | 3 |
| 2026-09-20 | 3 |
| 2026-09-21 | 4 |
| 2026-09-22 | 0 |

> Aquí solo hay 8 recursiones, pero si generas un año completo (365 días) sin `OPTION (MAXRECURSION ...)`, la consulta falla con el error Msg 530 al llegar a 100. En las próximas sesiones veremos `GENERATE_SERIES`, que genera series sin recursión y es la opción preferente en SQL Server 2022 y 2025.

### Ejemplo 14. CTE frente a subconsulta: reutilizar un resultado

**Pregunta de negocio:** ¿Qué usuarios han escuchado más segundos de música que la media de los usuarios?

La lógica “segundos totales por usuario” se necesita **dos veces**: para listar a cada usuario y para calcular la media. Con una CTE se escribe una sola vez.

```sql
WITH totales AS (
    SELECT usuario_id, SUM(segundos_escuchados) AS segundos
    FROM dbo.reproducciones
    WHERE tipo_contenido = N'Canción'
    GROUP BY usuario_id
)
SELECT u.nombre_usuario,
       t.segundos,
       (SELECT AVG(segundos) FROM totales) AS media_usuarios
FROM totales AS t
JOIN dbo.usuarios AS u ON u.usuario_id = t.usuario_id
WHERE t.segundos > (SELECT AVG(segundos) FROM totales)
ORDER BY t.segundos DESC;
```

**Salida esperada:**

| nombre_usuario | segundos | media_usuarios |
| --- | --- | --- |
| juanpi | 1719 | 904 |
| alexbeats | 1038 | 904 |

La misma consulta con tablas derivadas obliga a repetir el bloque de agregación en cada sitio donde se usa:

```sql
SELECT u.nombre_usuario, t.segundos
FROM (SELECT usuario_id, SUM(segundos_escuchados) AS segundos
      FROM dbo.reproducciones
      WHERE tipo_contenido = N'Canción'
      GROUP BY usuario_id) AS t
JOIN dbo.usuarios AS u ON u.usuario_id = t.usuario_id
WHERE t.segundos > (SELECT AVG(t2.segundos)
                    FROM (SELECT usuario_id, SUM(segundos_escuchados) AS segundos
                          FROM dbo.reproducciones
                          WHERE tipo_contenido = N'Canción'
                          GROUP BY usuario_id) AS t2)
ORDER BY t.segundos DESC;
```

| Criterio | Subconsulta / tabla derivada | CTE |
| --- | --- | --- |
| Legibilidad | Se lee de dentro hacia fuera | Se lee de arriba abajo, como una secuencia de pasos |
| Reutilización en la misma consulta | Hay que repetir el código | Se define una vez y se referencia varias veces |
| Recursividad | No | Sí |
| Rendimiento | Equivalente en la mayoría de casos | SQL Server **no materializa** la CTE: si se referencia dos veces, se calcula dos veces |
| Alcance | La instrucción donde se escribe | La instrucción que sigue al `WITH` |

> Si un resultado intermedio se va a usar en varias instrucciones o es costoso de calcular, la opción correcta es materializarlo en una tabla temporal. 

---

# Parte 02 · Ejercicios de refuerzo 

> Todos los ejercicios se resuelven sobre `SonoraDB` con la carga del Ejemplo 8 ya aplicada (38 reproducciones). Recuerda: una reproducción es válida si es de tipo `Canción` y tiene al menos 30 segundos escuchados.

### Ejercicio 1

El sello de Luna Roja quiere saber qué canciones del catálogo duran más que la canción más larga de Luna Roja. Resuélvelo con una subconsulta escalar que obtenga esa duración máxima a partir del **nombre** del artista. Ordena de mayor a menor duración.

**Salida esperada:**

| titulo | duracion_seg |
| --- | --- |
| Fábrica 4AM | 412 |
| Marea Alta | 356 |
| Latido | 330 |
| Forja | 305 |
| Corriente Alterna | 234 |
| Rimas de Barrio | 214 |

### Ejercicio 2

El equipo de producto está diseñando una nueva interfaz para televisores. Obtén, con una subconsulta de lista (`IN`), los títulos de las canciones que se han reproducido alguna vez desde un dispositivo `Smart TV`. Ordena alfabéticamente.

**Salida esperada:**

titulo

---

Diamantes Rotos

---

Galaxia Dembow

---

Neón

---

Verano en Bucle

---

### Ejercicio 3

El equipo de catálogo necesita detectar artistas fichados que todavía no tienen ninguna canción publicada. Resuélvelo con `NOT EXISTS`.

**Salida esperada:**

nombre

---

Sara Cometa

---

### Ejercicio 4

El equipo comercial quiere ofrecer un descuento en Premium a los usuarios que han escuchado al menos un anuncio. Obtén su nombre de usuario y su plan con `EXISTS`. Ordena por nombre de usuario.

**Salida esperada:**

| nombre_usuario | plan_suscripcion |
| --- | --- |
| dani_dj | Free |
| marta_rock | Free |
| nachox | Free |

### Ejercicio 5

Obtén los usuarios con 4 o más reproducciones válidas, mostrando cuántas tienen. Resuélvelo dos veces: primero con una tabla derivada en `FROM` y después reescríbelo con una CTE. Ordena por número de reproducciones descendente y, en caso de empate, por nombre de usuario.

**Salida esperada:**

| nombre_usuario | reproducciones_validas |
| --- | --- |
| alexbeats | 6 |
| juanpi | 6 |
| lucia.m | 4 |
| marta_rock | 4 |
| sofi.trap | 4 |

### Ejercicio 6

Para las canciones de Nébula y DJ Coral, muestra el título y el número total de reproducciones (válidas o no) de cada una, **incluidas las que no tienen ninguna**. Usa una subconsulta correlacionada en el `SELECT` para el conteo y una subconsulta de lista para filtrar los artistas por nombre. Ordena por número de reproducciones descendente y por título.

**Salida esperada:**

| titulo | total_reproducciones |
| --- | --- |
| Diamantes Rotos | 2 |
| Humo Violeta | 2 |
| Marea Alta | 2 |
| Sin Cobertura | 1 |
| Latido | 0 |

### Ejercicio 7

Un compañero ha escrito esta consulta para encontrar las canciones que ningún usuario Free ha escuchado nunca, pero devuelve 0 filas:

```sql
SELECT titulo
FROM dbo.canciones
WHERE cancion_id NOT IN (
    SELECT r.cancion_id
    FROM dbo.reproducciones AS r
    JOIN dbo.usuarios AS u ON u.usuario_id = r.usuario_id
    WHERE u.plan_suscripcion = N'Free'
);
```

Explica por qué falla y reescríbela correctamente con `NOT EXISTS`. Ordena por título.

**Salida esperada:**

titulo

---

Diamantes Rotos

---

Galaxia Dembow

---

Humo Violeta

---

Latido

---

Neón

---

Sin Cobertura

---

Verano en Bucle

---

### Ejercicio 8

La tabla `stg_usuarios` contiene el último lote de altas. Algunos usuarios ya existen en `usuarios`. Primero, escribe la consulta que identifica los usuarios del staging que aún no están cargados. Después, conviértela en un `INSERT ... SELECT` idempotente y comprueba que una segunda ejecución no inserta nada.

**Salida esperada de la consulta:**

| usuario_id | nombre_usuario | plan_suscripcion |
| --- | --- | --- |
| 9 | pau.synth | Free |
| 10 | kiara_perreo | Premium |

**Salida esperada de la carga:** `(2 rows affected)` en la primera ejecución, `(0 rows affected)` en la segunda. La tabla `usuarios` queda con 10 filas.

### Ejercicio 9

El equipo de finanzas quiere saber cuánta música válida se consume según el país del usuario y su plan. Escribe un pipeline de tres CTE encadenadas (**limpiar → enriquecer → agregar**) que devuelva el país, el plan, el número de reproducciones válidas y los minutos escuchados con un decimal. Ordena por minutos de mayor a menor.

**Salida esperada:**

| pais | plan_suscripcion | reproducciones | minutos |
| --- | --- | --- | --- |
| ES | Premium | 10 | 29.4 |
| MX | Premium | 6 | 28.7 |
| ES | Free | 4 | 13.1 |
| CO | Free | 3 | 12.4 |
| AR | Familiar | 4 | 11.3 |
| MX | Free | 3 | 9.7 |

### Ejercicio 10

Recursos Humanos necesita el organigrama completo de Sonora. Con una CTE recursiva sobre `empleados`, muestra el nombre, el puesto, el nivel (el CEO es nivel 0) y la ruta desde el CEO con el formato `Irene Salas > Tomás Vidal > ...`. Ordena por la ruta.

**Salida esperada:**

| nombre | puesto | nivel | ruta |
| --- | --- | --- | --- |
| Irene Salas | CEO | 0 | Irene Salas |
| Nuria Campos | CMO | 1 | Irene Salas > Nuria Campos |
| Laura Gil | Marketing Manager | 2 | Irene Salas > Nuria Campos > Laura Gil |
| Andrés Pozo | Community Manager | 3 | Irene Salas > Nuria Campos > Laura Gil > Andrés Pozo |
| Tomás Vidal | CTO | 1 | Irene Salas > Tomás Vidal |
| Hugo Marín | Backend Lead | 2 | Irene Salas > Tomás Vidal > Hugo Marín |
| Raúl Benet | Head of Data | 2 | Irene Salas > Tomás Vidal > Raúl Benet |
| Carmen Lozano | Data Analyst | 3 | Irene Salas > Tomás Vidal > Raúl Benet > Carmen Lozano |
| Elena Ruiz | Data Engineer | 3 | Irene Salas > Tomás Vidal > Raúl Benet > Elena Ruiz |
| Pablo Ortega | Data Engineer | 3 | Irene Salas > Tomás Vidal > Raúl Benet > Pablo Ortega |

### Ejercicio 11

El equipo editorial prepara una playlist “Todo Urbano”. Obtén todas las canciones del género `Urbano` y de **cualquiera de sus subgéneros**, a cualquier profundidad, con su género concreto y su artista. La CTE recursiva debe partir del nombre del género, no de su id. Ordena por género y título.

**Salida esperada:**

| titulo | genero | artista |
| --- | --- | --- |
| Galaxia Dembow | Dembow | MC Brisa |
| Rimas de Barrio | Hip Hop | Verso Libre |
| Calle 23 | Reguetón | MC Brisa |
| Perreo Lunar | Reguetón | MC Brisa |
| Diamantes Rotos | Trap | Nébula |
| Humo Violeta | Trap | Nébula |
| Sin Cobertura | Trap | Nébula |

### Ejercicio 12

El CTO, Tomás Vidal, quiere saber quién forma parte de su área, directa o indirectamente. Con una CTE recursiva que parta de Tomás Vidal, muestra el nombre, el puesto y el nivel relativo (sus subordinados directos son nivel 1). No incluyas al propio CTO. Ordena por nivel y nombre.

**Salida esperada:**

| nombre | puesto | nivel |
| --- | --- | --- |
| Hugo Marín | Backend Lead | 1 |
| Raúl Benet | Head of Data | 1 |
| Carmen Lozano | Data Analyst | 2 |
| Elena Ruiz | Data Engineer | 2 |
| Pablo Ortega | Data Engineer | 2 |

---

