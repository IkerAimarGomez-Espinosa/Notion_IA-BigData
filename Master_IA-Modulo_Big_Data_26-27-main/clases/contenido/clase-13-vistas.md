# 📋Clase 13 - Vistas

![Imagen](images/clase-13-vistas/image-001.png)

![Imagen](images/clase-13-vistas/image-002.png)

![Imagen](images/clase-13-vistas/image-003.png)

![Imagen](images/clase-13-vistas/image-004.png)

![Imagen](images/clase-13-vistas/image-005.png)

![Imagen](images/clase-13-vistas/image-006.png)

![Imagen](images/clase-13-vistas/image-007.png)

![Imagen](images/clase-13-vistas/image-008.png)

![Imagen](images/clase-13-vistas/image-009.png)

![Imagen](images/clase-13-vistas/image-010.png)

![Imagen](images/clase-13-vistas/image-011.png)

![Imagen](images/clase-13-vistas/image-012.png)

![Imagen](images/clase-13-vistas/image-013.png)

![Imagen](images/clase-13-vistas/image-014.png)

![Imagen](images/clase-13-vistas/image-015.png)

![Imagen](images/clase-13-vistas/image-016.png)

![Imagen](images/clase-13-vistas/image-017.png)

![Imagen](images/clase-13-vistas/image-018.png)

![Imagen](images/clase-13-vistas/image-019.png)

![Imagen](images/clase-13-vistas/image-020.png)

## Ejemplos

### Ejemplo 1. Vista que centraliza una regla de negocio

**Pregunta de negocio:** en la clase anterior escribimos `tipo_contenido = N'Canción' AND segundos_escuchados >= 30` en casi todas las consultas. Si mañana Producto decide que el umbral es de 35 segundos, habría que buscar y cambiar todas las copias. Queremos escribir la regla una sola vez.

**Paso 1.** Crear la vista con las columnas necesarias (nunca con `SELECT *`, lo veremos en el Ejemplo 8):

```sql
CREATE OR ALTER VIEW dbo.vw_reproducciones_validas
AS
SELECT reproduccion_id, usuario_id, cancion_id, fecha_hora,
       segundos_escuchados, dispositivo
FROM dbo.reproducciones
WHERE tipo_contenido = N'Canción'
  AND segundos_escuchados >= 30;
GO
```

**Salida esperada:** `Commands completed successfully.`

**Paso 2.** Usarla como si fuera una tabla:

```sql
SELECT (SELECT COUNT(*) FROM dbo.reproducciones)            AS total_reproducciones,
       (SELECT COUNT(*) FROM dbo.vw_reproducciones_validas) AS reproducciones_validas;
```

**Salida esperada:**

| total_reproducciones | reproducciones_validas |
| --- | --- |
| 43 | 32 |

**Paso 3.** Filtrar y ordenar desde fuera, como en cualquier tabla. Reproducciones válidas de sofi.trap (usuario 4):

```sql
SELECT reproduccion_id, cancion_id, fecha_hora, segundos_escuchados, dispositivo
FROM dbo.vw_reproducciones_validas
WHERE usuario_id = 4
ORDER BY fecha_hora;
```

**Salida esperada:**

| reproduccion_id | cancion_id | fecha_hora | segundos_escuchados | dispositivo |
| --- | --- | --- | --- | --- |
| 9 | 108 | 2026-09-15 17:30:00 | 167 | Móvil |
| 11 | 103 | 2026-09-15 17:35:00 | 176 | Móvil |
| 24 | 113 | 2026-09-18 20:00:00 | 158 | Móvil |
| 25 | 103 | 2026-09-18 20:03:00 | 176 | Móvil |

> sofi.trap tiene dos reproducciones más (20 y 22 segundos), pero son saltos y la vista ya las descarta. Quien consulta la vista no tiene que conocer la regla: la hereda.

### Ejemplo 2. Vista que oculta columnas sensibles

**Pregunta de negocio:** el equipo de Atención al cliente necesita consultar usuarios, pero no debe ver el email (dato personal).

```sql
CREATE OR ALTER VIEW dbo.vw_usuarios_directorio
AS
SELECT usuario_id, nombre_usuario, pais, plan_suscripcion, fecha_alta
FROM dbo.usuarios;
GO

SELECT usuario_id, nombre_usuario, plan_suscripcion, fecha_alta
FROM dbo.vw_usuarios_directorio
WHERE pais = 'ES'
ORDER BY fecha_alta;
```

**Salida esperada:**

| usuario_id | nombre_usuario | plan_suscripcion | fecha_alta |
| --- | --- | --- | --- |
| 1 | alexbeats | Premium | 2025-01-10 |
| 2 | marta_rock | Free | 2025-06-02 |
| 6 | lucia.m | Premium | 2026-09-01 |
| 9 | pau.synth | Free | 2026-09-21 |

> Una vista puede restringir **columnas** (seguridad vertical, como aquí) o **filas** con un `WHERE` (seguridad horizontal, lo veremos en el Ejemplo 9). Que la vista oculte el email no impide por sí sola que alguien consulte la tabla: para eso hace falta además gestionar permisos, que se estudian en la sesión de DCL.

### Ejemplo 3. Vista sobre vista: reproducciones enriquecidas

**Pregunta de negocio:** casi todos los informes necesitan la reproducción válida junto con el usuario, la canción, el artista y el género. Son cuatro `JOIN` que todo el mundo repite.

**Paso 1.** Crear una vista que parte de `vw_reproducciones_validas` (una vista puede leer de otra vista) y añade el contexto:

```sql
CREATE OR ALTER VIEW dbo.vw_reproducciones_detalle
AS
SELECT v.reproduccion_id,
       v.fecha_hora,
       u.usuario_id,
       u.nombre_usuario,
       u.pais            AS pais_usuario,
       u.plan_suscripcion,
       c.titulo,
       a.nombre          AS artista,
       g.nombre          AS genero,
       v.dispositivo,
       v.segundos_escuchados
FROM dbo.vw_reproducciones_validas AS v
JOIN dbo.usuarios  AS u ON u.usuario_id = v.usuario_id
JOIN dbo.canciones AS c ON c.cancion_id = v.cancion_id
JOIN dbo.artistas  AS a ON a.artista_id = c.artista_id
JOIN dbo.generos   AS g ON g.genero_id  = c.genero_id;
GO
```

**Paso 2.** Con la vista, una pregunta como *"¿qué artistas escuchan los usuarios de México?"* se resuelve sin ningún `JOIN`:

```sql
SELECT artista,
       COUNT(*)                 AS reproducciones,
       COUNT(DISTINCT usuario_id) AS oyentes
FROM dbo.vw_reproducciones_detalle
WHERE pais_usuario = 'MX'
GROUP BY artista
ORDER BY reproducciones DESC, artista;
```

**Salida esperada:**

| artista | reproducciones | oyentes |
| --- | --- | --- |
| Los Voltios | 2 | 1 |
| MC Brisa | 2 | 1 |
| Verso Libre | 2 | 2 |
| DJ Coral | 1 | 1 |
| Hierro Fundido | 1 | 1 |
| Kaiser Beats | 1 | 1 |

> SQL Server **expande** las vistas antes de ejecutar: sustituye `vw_reproducciones_detalle` por su definición, que a su vez contiene la de `vw_reproducciones_validas`, y optimiza la consulta completa. El filtro `pais_usuario = 'MX'` se aplica directamente sobre `usuarios`; no se calcula primero la vista entera. Aun así, encadenar muchas vistas unas sobre otras hace los planes difíciles de leer: dos o tres niveles es un límite razonable.

### Ejemplo 4. Vista con agregación: indicadores por canción

**Pregunta de negocio:** el equipo de catálogo consulta a diario los indicadores de cada canción. Deben aparecer todas las canciones, también las que no tienen ninguna reproducción válida.

**Paso 1.** Crear la vista con `LEFT JOIN` desde `canciones` y agregación:

```sql
CREATE OR ALTER VIEW dbo.vw_kpi_cancion
AS
SELECT c.cancion_id,
       c.titulo,
       a.nombre                     AS artista,
       c.fecha_lanzamiento,
       COUNT(v.reproduccion_id)     AS reproducciones_validas,
       COUNT(DISTINCT v.usuario_id) AS oyentes,
       CAST(COALESCE(SUM(v.segundos_escuchados), 0) / 60.0 AS decimal(6,1)) AS minutos
FROM dbo.canciones AS c
JOIN dbo.artistas AS a ON a.artista_id = c.artista_id
LEFT JOIN dbo.vw_reproducciones_validas AS v ON v.cancion_id = c.cancion_id
GROUP BY c.cancion_id, c.titulo, a.nombre, c.fecha_lanzamiento;
GO
```

**Paso 2.** Canciones sin ninguna reproducción válida:

```sql
SELECT titulo, artista, fecha_lanzamiento
FROM dbo.vw_kpi_cancion
WHERE reproducciones_validas = 0
ORDER BY titulo;
```

**Salida esperada:**

| titulo | artista | fecha_lanzamiento |
| --- | --- | --- |
| Latido | DJ Coral | 2025-07-07 |
| Sin Cobertura | Nébula | 2026-09-18 |

**Paso 3.** Las 3 canciones con más minutos escuchados:

```sql
SELECT TOP (3) titulo, artista, reproducciones_validas, oyentes, minutos
FROM dbo.vw_kpi_cancion
ORDER BY minutos DESC;
```

**Salida esperada:**

| titulo | artista | reproducciones_validas | oyentes | minutos |
| --- | --- | --- | --- | --- |
| Perreo Lunar | MC Brisa | 7 | 4 | 20.5 |
| Marea Alta | DJ Coral | 3 | 3 | 17.8 |
| Corriente Alterna | Los Voltios | 3 | 2 | 11.7 |

> En la pestaña Messages puede aparecer `Warning: Null value is eliminated by an aggregate or other SET operation.` Es normal: las canciones sin reproducciones aportan `NULL` a `COUNT(DISTINCT ...)` y `SUM(...)`. No es un error.

### Ejemplo 5. La trampa de `ORDER BY` en una vista

**Pregunta de negocio:** queremos que el catálogo salga "siempre ordenado" por duración.

**Paso 1.** El primer intento:

```sql
CREATE OR ALTER VIEW dbo.vw_canciones_ordenadas
AS
SELECT titulo, duracion_seg
FROM dbo.canciones
ORDER BY duracion_seg DESC;
GO
```

**Salida:**

```plain text
Msg 1033, Level 15, State 1
The ORDER BY clause is invalid in views, inline functions, derived tables, subqueries,
and common table expressions, unless TOP, OFFSET or FOR XML is also specified.
```

**Paso 2.** El "truco" que circula por internet: añadir `TOP (100) PERCENT`.

```sql
CREATE OR ALTER VIEW dbo.vw_canciones_ordenadas
AS
SELECT TOP (100) PERCENT titulo, duracion_seg
FROM dbo.canciones
ORDER BY duracion_seg DESC;
GO
```

La vista se crea, pero **el orden no está garantizado**. El optimizador sabe que `TOP (100) PERCENT` no filtra nada y descarta el `ORDER BY`. Puede que hoy salga ordenada y mañana, con más datos o un índice nuevo, no.

**Paso 3.** La forma correcta: la vista no ordena; ordena quien la consulta.

```sql
DROP VIEW IF EXISTS dbo.vw_canciones_ordenadas;
GO

SELECT titulo, duracion_seg
FROM dbo.canciones
ORDER BY duracion_seg DESC;
```

> Una vista representa un conjunto de filas, y un conjunto no tiene orden. Esto es especialmente importante en BI: Power BI ignora el orden de origen y aplica el suyo.

### Ejemplo 6. Vista con CTE recursiva: género principal de cada subgénero

**Pregunta de negocio:** cada subgénero, a cualquier profundidad, cuenta para su género principal (hijo directo de `Música`). La CTE recursiva que lo resuelve la hemos escrito varias veces. La guardamos como vista para no volver a escribirla.

**Paso 1.** Una vista puede contener CTE, incluidas las recursivas. La definición empieza directamente con `WITH`:

```sql
CREATE OR ALTER VIEW dbo.vw_genero_principal
AS
WITH arbol AS (
    SELECT genero_id, genero_id AS raiz_id
    FROM dbo.generos
    WHERE genero_padre_id = (SELECT genero_id FROM dbo.generos WHERE genero_padre_id IS NULL)

    UNION ALL

    SELECT g.genero_id, a.raiz_id
    FROM dbo.generos AS g
    JOIN arbol AS a ON g.genero_padre_id = a.genero_id
)
SELECT a.genero_id,
       g.nombre AS genero,
       r.nombre AS genero_principal
FROM arbol AS a
JOIN dbo.generos AS g ON g.genero_id = a.genero_id
JOIN dbo.generos AS r ON r.genero_id = a.raiz_id;
GO

SELECT genero_id, genero, genero_principal
FROM dbo.vw_genero_principal
ORDER BY genero_principal, genero;
```

**Salida esperada:**

| genero_id | genero | genero_principal |
| --- | --- | --- |
| 5 | Electrónica | Electrónica |
| 11 | House | Electrónica |
| 12 | Techno | Electrónica |
| 2 | Pop | Pop |
| 9 | Indie | Rock |
| 10 | Metal | Rock |
| 4 | Rock | Rock |
| 13 | Dembow | Urbano |
| 8 | Hip Hop | Urbano |
| 6 | Reguetón | Urbano |
| 7 | Trap | Urbano |
| 3 | Urbano | Urbano |

**Paso 2.** Ahora la pregunta *"reproducciones válidas por género principal"* combina dos vistas y una tabla:

```sql
SELECT gp.genero_principal,
       COUNT(*) AS reproducciones_validas,
       CAST(SUM(v.segundos_escuchados) / 60.0 AS decimal(6,1)) AS minutos
FROM dbo.vw_reproducciones_validas AS v
JOIN dbo.canciones           AS c  ON c.cancion_id = v.cancion_id
JOIN dbo.vw_genero_principal AS gp ON gp.genero_id = c.genero_id
GROUP BY gp.genero_principal
ORDER BY reproducciones_validas DESC;
```

**Salida esperada:**

| genero_principal | reproducciones_validas | minutos |
| --- | --- | --- |
| Urbano | 17 | 50.5 |
| Rock | 7 | 25.4 |
| Electrónica | 5 | 28.0 |
| Pop | 3 | 9.5 |

> La cláusula `OPTION (MAXRECURSION n)` no se puede escribir dentro de la vista. Si una jerarquía necesitara más de 100 niveles, el `OPTION` se añade al final de la consulta que **usa** la vista.

### Ejemplo 7. Consultar las vistas que existen y sus dependencias

**Contexto de ingeniería de datos:** antes de modificar una tabla hay que saber qué objetos dependen de ella. Es el **análisis de impacto**, y SQL Server guarda la información necesaria en sus vistas de catálogo.

**Paso 1.** Listar las vistas de la base de datos:

```sql
SELECT name AS vista
FROM sys.views
ORDER BY name;
```

**Salida esperada:**

vista

---

vw_genero_principal

---

vw_kpi_cancion

---

vw_reproducciones_detalle

---

vw_reproducciones_validas

---

vw_usuarios_directorio

---

**Paso 2.** Ver la definición de una vista:

```sql
EXEC sp_helptext N'dbo.vw_reproducciones_validas';
```

Devuelve, línea a línea, el texto completo del `CREATE OR ALTER VIEW`. En SSMS 22 también se obtiene con clic derecho sobre la vista → **Script View as** → **CREATE To**.

**Paso 3.** ¿Qué objetos dependen de la tabla `reproducciones`?

```sql
SELECT referencing_schema_name, referencing_entity_name
FROM sys.dm_sql_referencing_entities(N'dbo.reproducciones', N'OBJECT')
ORDER BY referencing_entity_name;
```

**Salida esperada:**

| referencing_schema_name | referencing_entity_name |
| --- | --- |
| dbo | vw_reproducciones_validas |

**Paso 4.** Solo aparecen las dependencias **directas**. Para seguir la cadena hay que preguntar por la vista:

```sql
SELECT referencing_schema_name, referencing_entity_name
FROM sys.dm_sql_referencing_entities(N'dbo.vw_reproducciones_validas', N'OBJECT')
ORDER BY referencing_entity_name;
```

**Salida esperada:**

| referencing_schema_name | referencing_entity_name |
| --- | --- |
| dbo | vw_kpi_cancion |
| dbo | vw_reproducciones_detalle |

> Si en sesiones anteriores creaste otros objetos sobre estas tablas, aparecerán también en el resultado. Conclusión práctica: cambiar `reproducciones` afecta a tres vistas, aunque solo una la lea directamente.

### Ejemplo 8. La trampa de `SELECT *` en una vista

**Contexto:** SQL Server guarda la lista de columnas de la vista **en el momento de crearla**. Si después cambia la tabla, la vista no se entera.

**Paso 1.** Preparar una copia de `artistas` para no tocar las tablas del caso y crear sobre ella una vista con `SELECT *`:

```sql
DROP VIEW IF EXISTS dbo.vw_demo_artistas;
DROP TABLE IF EXISTS dbo.demo_artistas;

CREATE TABLE dbo.demo_artistas (
    artista_id  int            NOT NULL,
    nombre      nvarchar(100)  NOT NULL,
    pais        char(2)        NOT NULL
);

INSERT INTO dbo.demo_artistas (artista_id, nombre, pais)
SELECT artista_id, nombre, pais
FROM dbo.artistas;
GO

CREATE OR ALTER VIEW dbo.vw_demo_artistas
AS
SELECT * FROM dbo.demo_artistas;
GO

SELECT * FROM dbo.vw_demo_artistas WHERE artista_id <= 2;
```

**Salida esperada:**

| artista_id | nombre | pais |
| --- | --- | --- |
| 1 | Luna Roja | ES |
| 2 | MC Brisa | PR |

**Paso 2.** Añadir una columna a la tabla y volver a consultar la vista:

```sql
ALTER TABLE dbo.demo_artistas ADD sello nvarchar(50) NULL;
GO

UPDATE dbo.demo_artistas SET sello = N'Sello Norte' WHERE artista_id = 1;

SELECT * FROM dbo.vw_demo_artistas WHERE artista_id <= 2;
```

**Salida esperada:** la misma de antes, **con tres columnas**. La columna `sello` no aparece aunque el `*` "debería" incluirla.

**Paso 3.** Refrescar los metadatos de la vista:

```sql
EXEC sp_refreshview N'dbo.vw_demo_artistas';

SELECT * FROM dbo.vw_demo_artistas WHERE artista_id <= 2;
```

**Salida esperada:**

| artista_id | nombre | pais | sello |
| --- | --- | --- | --- |
| 1 | Luna Roja | ES | Sello Norte |
| 2 | MC Brisa | PR | NULL |

**Paso 4.** Limpiar los objetos de la demostración:

```sql
DROP VIEW IF EXISTS dbo.vw_demo_artistas;
DROP TABLE IF EXISTS dbo.demo_artistas;
```

> Añadir una columna es el caso inofensivo. Si se elimina o se reordenan columnas de la tabla, una vista con `SELECT *` puede fallar o, peor, devolver datos bajo el nombre de otra columna sin dar ningún error. **Regla:** en una vista, siempre lista explícita de columnas.

---

## 2. Vistas actualizables y `WITH CHECK OPTION`

### Definición

Una vista es **actualizable** cuando SQL Server puede traducir un `INSERT`, `UPDATE` o `DELETE` sobre la vista en una operación sobre **una única tabla base**. La vista no guarda datos, así que cualquier modificación hecha "a través de la vista" cambia la tabla de verdad.

`WITH CHECK OPTION` añade una garantía: cualquier fila insertada o modificada a través de la vista **debe seguir cumpliendo el ****`WHERE`**** de la vista**. Sin esta opción, una modificación puede hacer que la fila "desaparezca" de la vista que se usó para modificarla.

Condiciones para que una modificación a través de una vista funcione:

| Condición | Error si no se cumple |
| --- | --- |
| La vista no contiene `GROUP BY`, `DISTINCT`, agregados, `PIVOT` ni `UNPIVOT` | Msg 4403 |
| Cada sentencia modifica columnas de **una sola** tabla base (aunque la vista tenga `JOIN`) | Msg 4405 |
| Las columnas modificadas son columnas reales, no expresiones calculadas ni constantes | Msg 4406 |
| En un `INSERT`, la vista incluye todas las columnas `NOT NULL` sin valor por defecto | Msg 515 |
| Con `WITH CHECK OPTION`, la fila resultante cumple el `WHERE` de la vista | Msg 550 |

### Analogía

Imagina el **mostrador de una oficina bancaria de barrio** que solo gestiona cuentas de clientes de su zona. El empleado puede actualizar tu dirección o tu teléfono, y el cambio queda registrado en el sistema central del banco (la tabla base). `WITH CHECK OPTION` es la norma interna que le prohíbe trasladar tu cuenta a otra zona desde su mostrador: si lo intenta, el sistema rechaza la operación, porque la cuenta dejaría de ser "de su zona".

### Sintaxis

```sql
CREATE OR ALTER VIEW esquema.nombre_vista
AS
SELECT columnas
FROM esquema.tabla
WHERE condicion
WITH CHECK OPTION;        -- va al final, después del WHERE
GO

-- Modificar a través de la vista
UPDATE esquema.nombre_vista SET columna = valor WHERE ...;
INSERT INTO esquema.nombre_vista (columnas) VALUES (...);
DELETE FROM esquema.nombre_vista WHERE ...;
```

> En los ejemplos que modifican datos usaremos `BEGIN TRAN` ... `ROLLBACK` para deshacer los cambios y dejar `SonoraDB` como estaba. Es la misma transacción con `ROLLBACK` que usaste en la práctica de repaso. Ejecuta cada bloque separado por `GO` de uno en uno y no olvides el `ROLLBACK` final.

### Ejemplo 9. Una vista de filas y el efecto de `WITH CHECK OPTION`

**Pregunta de negocio:** el equipo de conversión trabaja solo con usuarios Free.

**Paso 1.** Crear la vista sin `CHECK OPTION` y consultarla:

```sql
CREATE OR ALTER VIEW dbo.vw_usuarios_free
AS
SELECT usuario_id, nombre_usuario, pais, plan_suscripcion, fecha_alta
FROM dbo.usuarios
WHERE plan_suscripcion = N'Free';
GO

SELECT usuario_id, nombre_usuario, pais, plan_suscripcion
FROM dbo.vw_usuarios_free
ORDER BY usuario_id;
```

**Salida esperada:**

| usuario_id | nombre_usuario | pais | plan_suscripcion |
| --- | --- | --- | --- |
| 2 | marta_rock | ES | Free |
| 5 | dani_dj | CO | Free |
| 7 | nachox | MX | Free |
| 9 | pau.synth | ES | Free |

**Paso 2.** Cambiar el plan de nachox **a través de la vista**, dentro de una transacción:

```sql
BEGIN TRAN;

UPDATE dbo.vw_usuarios_free
SET plan_suscripcion = N'Premium'
WHERE nombre_usuario = N'nachox';

SELECT COUNT(*) AS usuarios_free FROM dbo.vw_usuarios_free;

SELECT nombre_usuario, plan_suscripcion
FROM dbo.usuarios
WHERE nombre_usuario = N'nachox';

ROLLBACK;
```

**Salida esperada:** `(1 row affected)` en el `UPDATE`, y después:

usuarios_free

---

3

---

| nombre_usuario | plan_suscripcion |
| --- | --- |
| nachox | Premium |

El `UPDATE` ha cambiado la tabla `usuarios` y nachox ha "desaparecido" de la vista con la que se modificó. El `ROLLBACK` deshace el cambio.

**Paso 3.** Añadir `WITH CHECK OPTION` y repetir el mismo `UPDATE`:

```sql
CREATE OR ALTER VIEW dbo.vw_usuarios_free
AS
SELECT usuario_id, nombre_usuario, pais, plan_suscripcion, fecha_alta
FROM dbo.usuarios
WHERE plan_suscripcion = N'Free'
WITH CHECK OPTION;
GO

UPDATE dbo.vw_usuarios_free
SET plan_suscripcion = N'Premium'
WHERE nombre_usuario = N'nachox';
```

**Salida:**

```plain text
Msg 550, Level 16, State 1
The attempted insert or update failed because the target view either specifies
WITH CHECK OPTION or spans a view that specifies WITH CHECK OPTION and one or more rows
resulting from the operation did not qualify under the CHECK OPTION constraint.
The statement has been terminated.
```

**Paso 4.** Los cambios que mantienen la fila dentro de la vista sí se permiten:

```sql
BEGIN TRAN;

UPDATE dbo.vw_usuarios_free
SET pais = 'AR'
WHERE nombre_usuario = N'nachox';

ROLLBACK;
```

**Salida esperada:** `(1 row affected)`. nachox sigue siendo Free, así que cumple el `WHERE` de la vista.

> `WITH CHECK OPTION` convierte el filtro de la vista en una regla de escritura. Es la forma de dar a un equipo permiso para mantener "sus" filas sin que pueda sacarlas, por error, de su ámbito.

### Ejemplo 10. Cuándo una vista no es actualizable

**Paso 1.** Intentar renombrar una canción a través de `vw_kpi_cancion`, que tiene `GROUP BY`:

```sql
UPDATE dbo.vw_kpi_cancion
SET titulo = N'Perreo Lunar (Remix)'
WHERE titulo = N'Perreo Lunar';
```

**Salida:**

```plain text
Msg 4403, Level 16, State 1
Cannot update the view or function 'dbo.vw_kpi_cancion' because it contains aggregates,
or a DISTINCT or GROUP BY clause, or PIVOT or UNPIVOT operator.
```

**Paso 2.** Intentar cambiar a la vez el título (tabla `canciones`) y el artista (tabla `artistas`) a través de `vw_reproducciones_detalle`:

```sql
UPDATE dbo.vw_reproducciones_detalle
SET titulo  = N'Perreo Lunar (Remix)',
    artista = N'MC Brisa & Nébula'
WHERE reproduccion_id = 1;
```

**Salida:**

```plain text
Msg 4405, Level 16, State 1
View or function 'dbo.vw_reproducciones_detalle' is not updatable because the modification
affects multiple base tables.
```

> Ninguna de las dos sentencias llega a ejecutarse: SQL Server las rechaza al compilarlas, así que no hace falta transacción. Que una vista **tenga** `JOIN` no la hace no actualizable: lo que no se permite es que **una misma sentencia** toque columnas de más de una tabla base. Lo comprobarás en los ejercicios.

---

## 3. Vistas con `SCHEMABINDING` y vistas materializadas (indexadas)

### Definición

Una vista normal es frágil: alguien puede cambiar el tipo de una columna, renombrarla o borrar la tabla, y la vista solo fallará cuando alguien la consulte. **`WITH SCHEMABINDING`** enlaza la vista con las tablas que usa: mientras la vista exista, SQL Server **impide** modificar o eliminar las columnas y tablas de las que depende.

Una **vista materializada** es una vista cuyo resultado **se guarda en disco** en lugar de calcularse en cada consulta. En otros motores (PostgreSQL, Oracle) se crea con `CREATE MATERIALIZED VIEW`. En SQL Server se llama **vista indexada** y se construye en dos pasos:

1. Se crea la vista con `WITH SCHEMABINDING`.
2. Se crea sobre ella un **índice clustered único** (`CREATE UNIQUE CLUSTERED INDEX`). Un índice clustered es la estructura en la que SQL Server guarda físicamente las filas, ordenadas por una clave; al crearlo sobre una vista, SQL Server calcula el resultado y lo almacena. La clave debe ser **única**: una fila guardada por cada valor (por ejemplo, una fila por canción).
A partir de ese momento, cada `INSERT`, `UPDATE` o `DELETE` en las tablas base actualiza también el resultado guardado, dentro de la misma operación. A diferencia de las vistas materializadas de PostgreSQL, en SQL Server **nunca hay que refrescarla**: siempre está al día.

Requisitos principales:

| Requisito | `SCHEMABINDING` | Vista indexada |
| --- | --- | --- |
| Lista explícita de columnas: prohibido `SELECT *` | Sí (Msg 1054) | Sí |
| Nombres de dos partes (`dbo.tabla`) | Sí (Msg 4512) | Sí |
| Si lee de otra vista, esa vista también debe tener `SCHEMABINDING` | Sí | — |
| Solo lee tablas, no otras vistas | No | Sí |
| Si hay `GROUP BY`, incluir `COUNT_BIG(*)` (un `COUNT(*)` que devuelve `bigint`) | No | Sí |
| Prohibido `AVG`, `MIN`, `MAX`, `DISTINCT`, `TOP`, `LEFT JOIN`, subconsultas y CTE | No | Sí |
| `SUM` solo sobre columnas que no admiten `NULL` | No | Sí |
| Prohibidas las expresiones sobre agregados (por ejemplo, `SUM(...) / 60.0`) | No | Sí |

### Analogía

`SCHEMABINDING` es como un **contrato de alquiler de un local** con una cláusula que impide al propietario tirar tabiques mientras el inquilino siga dentro. El propietario puede pintar las paredes que el inquilino no usa, pero si quiere hacer obra en el local alquilado, primero hay que rescindir o renegociar el contrato.

La vista materializada es la **vitrina de un bar de tapas**. En lugar de cocinar cada tapa cuando se pide (vista normal), la tortilla ya está hecha y se sirve al instante. A cambio, cada vez que entran ingredientes nuevos hay que rehacerla, y eso cuesta trabajo en la cocina. Compensa con las tapas que se piden muchísimo, no con las que se piden una vez al mes.

### Sintaxis

```sql
-- Vista enlazada al esquema
CREATE OR ALTER VIEW esquema.nombre_vista
WITH SCHEMABINDING                 -- va antes de AS
AS
SELECT t.columna1, t.columna2      -- lista explícita de columnas
FROM esquema.tabla AS t;           -- nombres de dos partes
GO

-- Vista indexada (materializada)
DROP VIEW IF EXISTS esquema.vw_ix_nombre;
GO

CREATE VIEW esquema.vw_ix_nombre
WITH SCHEMABINDING
AS
SELECT clave,
       COUNT_BIG(*)         AS filas,
       SUM(columna_no_nula) AS total
FROM esquema.tabla
GROUP BY clave;
GO

CREATE UNIQUE CLUSTERED INDEX IX_vw_ix_nombre
    ON esquema.vw_ix_nombre (clave);
GO

-- Leerla usando el resultado guardado
SELECT clave, filas, total
FROM esquema.vw_ix_nombre WITH (NOEXPAND);
```

> Para las vistas indexadas usamos `DROP VIEW IF EXISTS` + `CREATE VIEW` en lugar de `CREATE OR ALTER`: modificar una vista indexada con `ALTER` **elimina su índice** (y con él, el resultado guardado), y volver a ejecutar `CREATE INDEX` sobre un índice que ya existe da error. Borrar y crear de nuevo deja el script re-ejecutable.

### Ejemplo 11. Proteger una vista con `SCHEMABINDING`

**Pregunta de negocio:** el catálogo que consume la app de los sellos no puede romperse porque alguien cambie una tabla sin avisar.

**Paso 1.** Primer intento, con `SELECT *`:

```sql
CREATE OR ALTER VIEW dbo.vw_catalogo_canciones
WITH SCHEMABINDING
AS
SELECT * FROM dbo.canciones;
GO
```

**Salida:**

```plain text
Msg 1054, Level 15, State 6
Syntax '*' is not allowed in schema-bound objects.
```

**Paso 2.** Segundo intento, con columnas explícitas pero el nombre de la tabla sin esquema:

```sql
CREATE OR ALTER VIEW dbo.vw_catalogo_canciones
WITH SCHEMABINDING
AS
SELECT cancion_id, titulo FROM canciones;
GO
```

**Salida:**

```plain text
Msg 4512, Level 16, State 3
Cannot schema bind view 'dbo.vw_catalogo_canciones' because name 'canciones' is invalid
for schema binding. Names must be in two-part format and an object cannot reference itself.
```

**Paso 3.** La versión correcta:

```sql
CREATE OR ALTER VIEW dbo.vw_catalogo_canciones
WITH SCHEMABINDING
AS
SELECT c.cancion_id,
       c.titulo,
       a.nombre AS artista,
       c.duracion_seg,
       c.fecha_lanzamiento
FROM dbo.canciones AS c
JOIN dbo.artistas  AS a ON a.artista_id = c.artista_id;
GO
```

**Salida esperada:** `Commands completed successfully.`

**Paso 4.** Un compañero intenta ampliar la columna `titulo`:

```sql
ALTER TABLE dbo.canciones ALTER COLUMN titulo nvarchar(200) NOT NULL;
```

**Salida:**

```plain text
Msg 5074, Level 16, State 1
The object 'vw_catalogo_canciones' is dependent on column 'titulo'.
Msg 4922, Level 16, State 9
ALTER TABLE ALTER COLUMN titulo failed because one or more objects access this column.
```

> El cambio se bloquea **antes** de romper nada. Para hacerlo de forma controlada hay que: (1) quitar el `SCHEMABINDING` de la vista o eliminarla, (2) modificar la tabla, (3) volver a crear la vista. Añadir una columna nueva a `canciones` sí estaría permitido, porque la vista no la usa.

### Ejemplo 12. Vista materializada para el indicador más consultado

**Pregunta de negocio:** el contador de reproducciones válidas por canción aparece en la app, en la web y en los informes de los sellos. Se consulta miles de veces al día y hoy se recalcula en cada consulta.

**Paso 1.** Crear la vista con los requisitos de una vista indexada (tabla base, nombres de dos partes, `COUNT_BIG(*)` y `SUM` sobre una columna `NOT NULL`):

```sql
DROP VIEW IF EXISTS dbo.vw_ix_consumo_cancion;
GO

CREATE VIEW dbo.vw_ix_consumo_cancion
WITH SCHEMABINDING
AS
SELECT cancion_id,
       COUNT_BIG(*)             AS reproducciones_validas,
       SUM(segundos_escuchados) AS segundos
FROM dbo.reproducciones
WHERE tipo_contenido = N'Canción'
  AND segundos_escuchados >= 30
GROUP BY cancion_id;
GO
```

**Paso 2.** Crear el índice clustered único. En este momento el resultado se calcula y se guarda:

```sql
CREATE UNIQUE CLUSTERED INDEX IX_vw_ix_consumo_cancion
    ON dbo.vw_ix_consumo_cancion (cancion_id);
GO

SELECT COUNT(*) AS filas_guardadas
FROM dbo.vw_ix_consumo_cancion WITH (NOEXPAND);
```

**Salida esperada:**

filas_guardadas

---

13

---

Hay 13 filas porque Latido y Sin Cobertura no tienen ninguna reproducción válida (una vista indexada no admite `LEFT JOIN` para incluirlas).

**Paso 3.** Consultarla con `NOEXPAND`:

```sql
SELECT TOP (5) c.titulo,
       v.reproducciones_validas,
       CAST(v.segundos / 60.0 AS decimal(6,1)) AS minutos
FROM dbo.vw_ix_consumo_cancion AS v WITH (NOEXPAND)
JOIN dbo.canciones AS c ON c.cancion_id = v.cancion_id
ORDER BY v.reproducciones_validas DESC, v.segundos DESC;
```

**Salida esperada:**

| titulo | reproducciones_validas | minutos |
| --- | --- | --- |
| Perreo Lunar | 7 | 20.5 |
| Marea Alta | 3 | 17.8 |
| Corriente Alterna | 3 | 11.7 |
| Galaxia Dembow | 3 | 7.9 |
| Fábrica 4AM | 2 | 10.2 |

> La media y los minutos no se guardan (`AVG` y las expresiones sobre agregados no están permitidos), pero se calculan al consultar: `segundos / 60.0` o `segundos * 1.0 / reproducciones_validas`. `WITH (NOEXPAND)` es una indicación para el optimizador que obliga a leer el resultado guardado. Sin la sugerencia, el optimizador puede decidir expandir la vista y recalcular desde `reproducciones`; en algunas ediciones de SQL Server nunca usa la vista indexada automáticamente, así que la práctica recomendada es indicar siempre `NOEXPAND`.

**Paso 4.** Comprobar que el resultado guardado se mantiene solo. Latido no tiene reproducciones válidas; insertamos una dentro de una transacción:

```sql
BEGIN TRAN;

INSERT INTO dbo.reproducciones
    (reproduccion_id, usuario_id, cancion_id, fecha_hora, segundos_escuchados, dispositivo, tipo_contenido)
VALUES (44, 8, 115, '2026-09-22 12:00', 330, N'Móvil', N'Canción');

SELECT cancion_id, reproducciones_validas, segundos
FROM dbo.vw_ix_consumo_cancion WITH (NOEXPAND)
WHERE cancion_id = 115;

ROLLBACK;
```

**Salida esperada:**

| cancion_id | reproducciones_validas | segundos |
| --- | --- | --- |
| 115 | 1 | 330 |

El `INSERT` en `reproducciones` ha actualizado también la vista indexada, en la misma transacción. Tras el `ROLLBACK` todo vuelve a su estado.

**¿Cuándo compensa una vista materializada?**

| Compensa | No compensa |
| --- | --- |
| Agregaciones costosas que se leen mucho más de lo que se escriben | Tablas con cargas continuas de muchas filas (cada fila nueva actualiza la vista) |
| Informes y cuadros de mando que repiten la misma agregación | Consultas que se ejecutan pocas veces |
| Datos que cambian en lotes controlados (cargas nocturnas) | Lógica que necesita `LEFT JOIN`, `AVG`, `DISTINCT` o subconsultas |

### Vistas frente a subconsultas, CTE y vistas materializadas

| Criterio | Subconsulta / CTE | Vista | Vista materializada (indexada) |
| --- | --- | --- | --- |
| Dónde vive | Dentro de una consulta | En la base de datos, con nombre | En la base de datos, con nombre |
| Quién la puede reutilizar | Solo esa consulta | Cualquier consulta, script o persona | Cualquier consulta, script o persona |
| Guarda datos | No | No, solo la definición | Sí, el resultado completo |
| Datos siempre actuales | Sí | Sí | Sí, se mantiene sola |
| Admite parámetros | — | No: los filtros los pone quien consulta la vista | No |
| Oculta columnas o filas | No | Sí | Sí |
| Coste | Al ejecutar la consulta | Al ejecutar la consulta | En cada escritura en las tablas base; la lectura es inmediata |

# Parte 02 · Ejercicios de refuerzo

> Todos los ejercicios se resuelven sobre `SonoraDB` con el entorno preparado al inicio de la clase (43 reproducciones y 10 usuarios) y las vistas de la teoría ya creadas. Recuerda: una reproducción es válida si es de tipo `Canción` y tiene al menos 30 segundos escuchados. Los ejercicios que modifican datos se hacen dentro de `BEGIN TRAN` ... `ROLLBACK`, ejecutando cada bloque separado por `GO` de uno en uno.

### Ejercicio 1

El equipo comercial vende los huecos publicitarios y quiere una vista `dbo.vw_anuncios` con las reproducciones de tipo `Anuncio`: identificador de reproducción, usuario, fecha y hora, y dispositivo. Después, consulta la vista para obtener cuántos anuncios se han emitido por dispositivo y a cuántos usuarios distintos. Ordena por número de anuncios descendente y por dispositivo.

**Salida esperada de la consulta:**

| dispositivo | anuncios | usuarios |
| --- | --- | --- |
| Móvil | 3 | 2 |
| Web | 3 | 2 |

### Ejercicio 2

Crea la vista `dbo.vw_usuarios_pago` con los usuarios de los planes `Premium` y `Familiar`, sin la columna `email`. Consulta la vista ordenando por fecha de alta.

**Salida esperada:**

| usuario_id | nombre_usuario | pais | plan_suscripcion | fecha_alta |
| --- | --- | --- | --- | --- |
| 3 | juanpi | MX | Premium | 2024-11-20 |
| 1 | alexbeats | ES | Premium | 2025-01-10 |
| 4 | sofi.trap | AR | Familiar | 2026-02-14 |
| 8 | vale_indie | AR | Premium | 2026-05-05 |
| 6 | lucia.m | ES | Premium | 2026-09-01 |
| 10 | kiara_perreo | PR | Premium | 2026-09-21 |

### Ejercicio 3

Producto estudia por qué los usuarios saltan canciones. Crea la vista `dbo.vw_saltos` con las reproducciones de tipo `Canción` de menos de 30 segundos: identificador, usuario, canción, fecha y hora, y segundos escuchados. Después, consulta la vista uniéndola con `usuarios` y `canciones` para mostrar el nombre de usuario, el título, los segundos escuchados y la fecha y hora, ordenado por fecha y hora.

**Salida esperada de la consulta:**

| nombre_usuario | titulo | segundos_escuchados | fecha_hora |
| --- | --- | --- | --- |
| sofi.trap | Diamantes Rotos | 20 | 2026-09-15 17:33:00 |
| lucia.m | Verano en Bucle | 15 | 2026-09-16 12:08:00 |
| sofi.trap | Sin Cobertura | 22 | 2026-09-21 18:30:00 |
| vale_indie | Corriente Alterna | 12 | 2026-09-22 07:30:00 |
| alexbeats | Sin Cobertura | 25 | 2026-09-22 10:00:00 |

### Ejercicio 4

El equipo editorial necesita una ficha de catálogo. Crea la vista `dbo.vw_catalogo_detalle` con el identificador y el título de la canción, el nombre del artista (`artista`), el país del artista (`pais_artista`), el nombre del género (`genero`), la duración en segundos (`duracion_seg`) y la duración en minutos con un decimal (`duracion_min`). Consulta las canciones de artistas españoles ordenadas por título. ¿Por qué no aparece Sara Cometa, que es española?

**Salida esperada de la consulta:**

| titulo | artista | genero | duracion_seg | duracion_min |
| --- | --- | --- | --- | --- |
| Forja | Hierro Fundido | Metal | 305 | 5.1 |
| Neón | Luna Roja | Pop | 201 | 3.4 |
| Rimas de Barrio | Verso Libre | Hip Hop | 214 | 3.6 |
| Verano en Bucle | Luna Roja | Pop | 185 | 3.1 |

### Ejercicio 5

Un compañero ha intentado crear esta vista para el equipo de producto y no consigue que funcione:

```sql
CREATE VIEW dbo.vw_canciones_por_dispositivo
AS
SELECT dispositivo, COUNT(*)
FROM dbo.reproducciones
WHERE tipo_contenido = N'Canción'
GROUP BY dispositivo
ORDER BY COUNT(*) DESC;
```

Explica los dos problemas, corrige la vista (la columna del conteo debe llamarse `reproducciones`) y escribe la consulta que la usa ordenada de más a menos reproducciones.

**Salida esperada de la consulta:**

| dispositivo | reproducciones |
| --- | --- |
| Móvil | 21 |
| Web | 7 |
| Smart TV | 5 |
| Escritorio | 4 |

### Ejercicio 6

Crea la vista `dbo.vw_kpi_usuario` con una fila por usuario, **incluidos los que no tienen actividad**: identificador, nombre de usuario, plan, país, número de reproducciones válidas (`reproducciones_validas`), minutos escuchados con un decimal (`minutos`) y fecha y hora de su última reproducción válida (`ultima_reproduccion_valida`). Reutiliza `dbo.vw_reproducciones_validas`. Después, con dos consultas sobre la vista:

a) Obtén los usuarios sin ninguna reproducción válida.

**Salida esperada:**

| nombre_usuario | plan_suscripcion | reproducciones_validas | minutos | ultima_reproduccion_valida |
| --- | --- | --- | --- | --- |
| vale_indie | Premium | 0 | 0.0 | NULL |

b) Obtén los usuarios de pago ordenados por minutos de mayor a menor.

**Salida esperada:**

| nombre_usuario | plan_suscripcion | reproducciones_validas | minutos |
| --- | --- | --- | --- |
| juanpi | Premium | 6 | 28.7 |
| alexbeats | Premium | 6 | 17.3 |
| lucia.m | Premium | 4 | 12.1 |
| sofi.trap | Familiar | 4 | 11.3 |
| kiara_perreo | Premium | 1 | 2.9 |
| vale_indie | Premium | 0 | 0.0 |

### Ejercicio 7

Recursos Humanos consulta el organigrama varias veces por semana. Guarda la CTE recursiva del organigrama en la vista `dbo.vw_organigrama` con el identificador, el nombre, el puesto, el nivel (el CEO es nivel 0) y la ruta desde el CEO (`Irene Salas > Tomás Vidal > ...`). Con la vista:

a) Obtén las personas que dependen, directa o indirectamente, de Raúl Benet, ordenadas por nombre.

**Salida esperada:**

| nombre | puesto | nivel |
| --- | --- | --- |
| Carmen Lozano | Data Analyst | 3 |
| Elena Ruiz | Data Engineer | 3 |
| Pablo Ortega | Data Engineer | 3 |

b) Obtén cuántos empleados hay en cada nivel.

**Salida esperada:**

| nivel | empleados |
| --- | --- |
| 0 | 1 |
| 1 | 2 |
| 2 | 3 |
| 3 | 4 |

### Ejercicio 8

El equipo de lanzamientos solo debe gestionar canciones publicadas en 2026. Crea la vista `dbo.vw_canciones_2026` con todas las columnas de `canciones` (escritas de forma explícita), filtrada por `fecha_lanzamiento >= '2026-01-01'` y con `WITH CHECK OPTION`. Antes de ejecutar las siguientes sentencias, **predice** el resultado de cada una. Después, ejecútalas una a una dentro de una transacción y deshaz los cambios al final.

```sql
-- a)
UPDATE dbo.vw_canciones_2026 SET duracion_seg = 170 WHERE titulo = N'Humo Violeta';
-- b)
UPDATE dbo.vw_canciones_2026 SET fecha_lanzamiento = '2025-12-15' WHERE titulo = N'Humo Violeta';
-- c)
UPDATE dbo.vw_canciones_2026 SET duracion_seg = 210 WHERE titulo = N'Neón';
-- d)
INSERT INTO dbo.vw_canciones_2026 (cancion_id, titulo, artista_id, genero_id, duracion_seg, fecha_lanzamiento)
VALUES (116, N'Eco Antiguo', 9, 2, 190, '2025-10-01');
-- e)
INSERT INTO dbo.vw_canciones_2026 (cancion_id, titulo, artista_id, genero_id, duracion_seg, fecha_lanzamiento)
VALUES (117, N'Primer Vuelo', 9, 2, 195, '2026-10-02');
```

**Salida esperada:**

| Sentencia | Resultado |
| --- | --- |
| a) | `(1 row affected)` |
| b) | `Msg 550` |
| c) | `(0 rows affected)` |
| d) | `Msg 550` |
| e) | `(1 row affected)` |

Tras el `ROLLBACK`, `SELECT COUNT(*) FROM dbo.canciones;` devuelve **15** y Humo Violeta conserva sus 167 segundos.

### Ejercicio 9

Usando la vista `dbo.vw_catalogo_detalle` del Ejercicio 4, que une `canciones`, `artistas` y `generos`, predice qué ocurre con cada sentencia. Después ejecútalas una a una dentro de una transacción (separadas por `GO`) y, antes del `ROLLBACK`, consulta las canciones cuyo artista empiece por `Luna Roja`.

```sql
-- a)
UPDATE dbo.vw_catalogo_detalle SET duracion_seg = 205 WHERE titulo = N'Neón';
-- b)
UPDATE dbo.vw_catalogo_detalle
SET titulo = N'Neón (Remix)', artista = N'Luna Roja & DJ Coral'
WHERE titulo = N'Neón';
-- c)
UPDATE dbo.vw_catalogo_detalle SET duracion_min = 3.5 WHERE titulo = N'Neón';
-- d)
UPDATE dbo.vw_catalogo_detalle SET artista = N'Luna Roja Oficial' WHERE titulo = N'Neón';
```

**Salida esperada:**

| Sentencia | Resultado |
| --- | --- |
| a) | `(1 row affected)` |
| b) | `Msg 4405` |
| c) | `Msg 4406` |
| d) | `(1 row affected)` |

**Consulta antes del ****`ROLLBACK`****:**

| titulo | artista | duracion_seg |
| --- | --- | --- |
| Neón | Luna Roja Oficial | 205 |
| Verano en Bucle | Luna Roja Oficial | 185 |

¿Por qué ha cambiado también el artista de Verano en Bucle si el `UPDATE` filtraba por Neón?

### Ejercicio 10

Un compañero quiere proteger la ficha de artistas con `SCHEMABINDING`, pero su vista no se crea:

```sql
CREATE OR ALTER VIEW dbo.vw_artistas_ficha
WITH SCHEMABINDING
AS
SELECT *
FROM artistas AS a
JOIN dbo.generos AS g ON g.genero_id = a.genero_id;
```

a) Identifica los tres problemas y corrige la vista para que devuelva el identificador del artista, su nombre (`artista`), su país y el nombre de su género (`genero`). Consulta los artistas españoles ordenados por nombre.

**Salida esperada:**

| artista | pais | genero |
| --- | --- | --- |
| Hierro Fundido | ES | Metal |
| Luna Roja | ES | Pop |
| Sara Cometa | ES | Pop |
| Verso Libre | ES | Hip Hop |

b) Intenta ampliar la columna `nombre` de `artistas` a `nvarchar(150)`. ¿Qué ocurre y por qué aparecen dos vistas en los mensajes?

**Salida esperada:** un `Msg 5074` por cada vista enlazada que usa la columna (`vw_artistas_ficha` y `vw_catalogo_canciones`, del Ejemplo 11) seguido de `Msg 4922`.

c) Con `sys.dm_sql_referencing_entities`, obtén las vistas que dependen directamente de `dbo.artistas`, ordenadas por nombre. ¿Cuáles de ellas impiden el cambio del apartado b) y cuáles no? ¿Por qué?

**Salida esperada** (si has creado otros objetos sobre `artistas`, aparecerán también):

| referencing_schema_name | referencing_entity_name |
| --- | --- |
| dbo | vw_artistas_ficha |
| dbo | vw_catalogo_canciones |
| dbo | vw_catalogo_detalle |
| dbo | vw_kpi_cancion |
| dbo | vw_reproducciones_detalle |

### Ejercicio 11

El panel de dispositivos de Producto se refresca cada minuto. Un compañero ha preparado esta vista para materializarla, pero al crear el índice SQL Server la rechaza:

```sql
DROP VIEW IF EXISTS dbo.vw_ix_consumo_dispositivo;
GO

CREATE VIEW dbo.vw_ix_consumo_dispositivo
WITH SCHEMABINDING
AS
SELECT dispositivo,
       AVG(segundos_escuchados) AS media_segundos
FROM dbo.reproducciones
WHERE tipo_contenido = N'Canción'
  AND segundos_escuchados >= 30
GROUP BY dispositivo;
GO

CREATE UNIQUE CLUSTERED INDEX IX_vw_ix_consumo_dispositivo
    ON dbo.vw_ix_consumo_dispositivo (dispositivo);
```

a) Explica por qué falla y corrígela para que guarde el número de reproducciones válidas (`reproducciones_validas`) y los segundos totales (`segundos`) por dispositivo. Crea el índice.

b) Consulta la vista con `NOEXPAND` calculando los minutos y la media de segundos por reproducción, ambos con un decimal. Ordena por reproducciones descendente y por dispositivo.

**Salida esperada:**

| dispositivo | reproducciones_validas | minutos | media_segundos |
| --- | --- | --- | --- |
| Móvil | 17 | 54.0 | 190.5 |
| Web | 7 | 25.9 | 221.7 |
| Escritorio | 4 | 21.5 | 321.8 |
| Smart TV | 4 | 12.1 | 181.5 |

c) Dentro de una transacción, inserta la reproducción 44 (usuario 6, canción 102, `'2026-09-22 21:00'`, 201 segundos, `Smart TV`, `Canción`), consulta la fila de `Smart TV` de la vista y deshaz el cambio.

**Salida esperada antes del ****`ROLLBACK`****:**

| dispositivo | reproducciones_validas | segundos |
| --- | --- | --- |
| Smart TV | 5 | 927 |

# Parte 03. Caso de estudio Sonora: las vistas para el cuadro de mando

## Contexto

El informe del martes funcionó. El comité de dirección de Sonora vio las cifras de la semana y tomó una decisión: a partir de ahora quiere un **cuadro de mando en Power BI** que se actualice solo, en lugar de pedir cifras por mensaje cada semana.

Carmen Lozano (Data Analyst) se encargará del cuadro de mando. Antes de que conecte Power BI, Raúl Benet (Head of Data) te envía este mensaje:

> *"Carmen va a conectar Power BI esta semana y no quiero que el cuadro de mando lea las tablas directamente. Prepárale un conjunto de vistas, todas con el prefijo **`vw_bi_`**, que sean lo único que use. Las reglas de negocio tienen que estar dentro de las vistas, no en Power BI, para que no haya dos versiones de 'reproducción válida'. Aprovecha también para resolver dos peticiones que tengo pendientes: Laura quiere que su equipo de LATAM pueda corregir datos de sus usuarios sin tocar los de España, y Hugo quiere ampliar un campo de **`usuarios`**. Todo en un script que se pueda ejecutar varias veces."*

Todo el trabajo se hace sobre `SonoraDB`.

**Reglas de negocio de Sonora:**

- Una reproducción es **válida** cuando es de tipo `Canción` y se han escuchado al menos **30 segundos**. Por debajo es un **salto**.
- Los **géneros principales** son los hijos directos del género raíz `Música`. Cada subgénero, a cualquier profundidad, cuenta para su género principal.
- Los **mercados LATAM** son México (`MX`), Argentina (`AR`), Colombia (`CO`) y Puerto Rico (`PR`).
- La **semana de análisis** del cuadro de mando va del 16/09/2026 al 22/09/2026, ambos incluidos.
---

## Preparación

Ejecuta este script antes de empezar. Garantiza el punto de partida (43 reproducciones y 10 usuarios) aunque no hayas hecho las cargas de clases anteriores. Es idempotente.

```sql
USE SonoraDB;
GO

-- 1. Cargas de la clase de subconsultas y CTE
INSERT INTO dbo.reproducciones
    (reproduccion_id, usuario_id, cancion_id, fecha_hora, segundos_escuchados, dispositivo, tipo_contenido)
SELECT s.reproduccion_id, s.usuario_id, s.cancion_id, s.fecha_hora,
       s.segundos_escuchados, s.dispositivo, s.tipo_contenido
FROM dbo.stg_reproducciones AS s
WHERE NOT EXISTS (SELECT 1 FROM dbo.reproducciones AS r WHERE r.reproduccion_id = s.reproduccion_id);

INSERT INTO dbo.usuarios (usuario_id, nombre_usuario, email, pais, plan_suscripcion, fecha_alta)
SELECT s.usuario_id, s.nombre_usuario, s.email, s.pais, s.plan_suscripcion, s.fecha_alta
FROM dbo.stg_usuarios AS s
WHERE NOT EXISTS (SELECT 1 FROM dbo.usuarios AS u WHERE u.usuario_id = s.usuario_id);
GO

-- 2. Lote del 22/09
DROP TABLE IF EXISTS dbo.stg_reproducciones_2209;

CREATE TABLE dbo.stg_reproducciones_2209 (
    reproduccion_id      int,
    usuario_id           int,
    cancion_id           int,
    fecha_hora           datetime2(0),
    segundos_escuchados  int,
    dispositivo          nvarchar(20),
    tipo_contenido       nvarchar(10)
);

INSERT INTO dbo.stg_reproducciones_2209
    (reproduccion_id, usuario_id, cancion_id, fecha_hora, segundos_escuchados, dispositivo, tipo_contenido) VALUES
(37, 5,  NULL, '2026-09-21 22:00', 30,  N'Móvil', N'Anuncio'),
(38, 5,  104,  '2026-09-21 22:01', 190, N'Móvil', N'Canción'),
(39, 8,  106,  '2026-09-22 07:30', 12,  N'Móvil', N'Canción'),
(40, 10, 103,  '2026-09-22 09:00', 176, N'Móvil', N'Canción'),
(41, 9,  NULL, '2026-09-22 09:10', 30,  N'Web',   N'Anuncio'),
(42, 9,  111,  '2026-09-22 09:11', 356, N'Web',   N'Canción'),
(43, 1,  114,  '2026-09-22 10:00', 25,  N'Móvil', N'Canción');

INSERT INTO dbo.reproducciones
    (reproduccion_id, usuario_id, cancion_id, fecha_hora, segundos_escuchados, dispositivo, tipo_contenido)
SELECT s.reproduccion_id, s.usuario_id, s.cancion_id, s.fecha_hora,
       s.segundos_escuchados, s.dispositivo, s.tipo_contenido
FROM dbo.stg_reproducciones_2209 AS s
WHERE NOT EXISTS (SELECT 1 FROM dbo.reproducciones AS r WHERE r.reproduccion_id = s.reproduccion_id);
GO

-- 3. Comprobación
SELECT (SELECT COUNT(*) FROM dbo.reproducciones) AS reproducciones,
       (SELECT COUNT(*) FROM dbo.usuarios)       AS usuarios;
```

**Salida esperada:**

| reproducciones | usuarios |
| --- | --- |
| 43 | 10 |

---

## Requerimientos

### Requerimiento 1. Tabla de hechos de reproducciones válidas

**Solicitado por:** Carmen Lozano (Data Analyst)

Crea la vista `dbo.vw_bi_hechos_reproduccion`: **una fila por reproducción válida** con todo lo que el cuadro de mando necesita para filtrar y agrupar sin hacer ningún `JOIN` en Power BI: identificador de la reproducción, fecha (solo el día, sin hora), fecha y hora, identificador y nombre del usuario, país del usuario, plan, título de la canción, artista, género, género principal, dispositivo y segundos escuchados.

La vista debe ser independiente de las vistas creadas en clase (solo lee tablas) y debe resolver el género principal internamente. Comprueba que tiene exactamente una fila por reproducción válida y entrega el resumen por país del usuario: reproducciones, oyentes distintos y minutos con un decimal.

### Requerimiento 2. Evolución diaria

**Solicitado por:** Carmen Lozano (Data Analyst)

Crea la vista `dbo.vw_bi_kpi_diario` con una fila por día: reproducciones válidas, oyentes distintos y minutos escuchados con un decimal. Debe construirse sobre la vista del Requerimiento 1. Entrega la consulta que usará Power BI para mostrar la semana de análisis, ordenada por fecha.

### Requerimiento 3. Indicadores por género principal

**Solicitado por:** Nuria Campos (CMO)

Crea la vista `dbo.vw_bi_kpi_genero_principal` con el acumulado histórico de cada género principal: reproducciones válidas, oyentes distintos y minutos con un decimal. Deben aparecer los cuatro géneros principales aunque alguno no tenga actividad. Entrega la consulta ordenada por reproducciones de mayor a menor.

### Requerimiento 4. Rendimiento de canciones

**Solicitado por:** equipo de relación con sellos

Los sellos quieren saber qué canciones "enganchan" y cuáles se saltan. Crea la vista `dbo.vw_bi_rendimiento_canciones` con todas las canciones del catálogo: título, artista, fecha de lanzamiento, reproducciones totales de la canción (válidas o no), reproducciones válidas, saltos y **tasa de salto** (porcentaje de saltos sobre el total, con un decimal). Una canción sin reproducciones debe aparecer con tasa `NULL`, no con un error de división entre cero. Entrega la consulta de las canciones con algún salto, ordenadas por tasa de salto descendente.

### Requerimiento 5. Mantenimiento de usuarios LATAM

**Solicitado por:** Laura Gil (Marketing)

El equipo de Marketing LATAM tiene que poder corregir el plan y el país de sus usuarios, pero nunca ver emails ni tocar usuarios de otros mercados, ni siquiera por error. Crea la vista `dbo.vw_usuarios_latam` (no lleva el prefijo `vw_bi_` porque no es para el cuadro de mando, sino para mantenimiento) con identificador, nombre de usuario, país, plan y fecha de alta de los usuarios LATAM, y garantiza que ninguna modificación hecha a través de ella pueda sacar a un usuario del ámbito LATAM.

Entrega el listado de la vista ordenado por país y nombre, y demuestra dentro de una transacción que se deshace al final:

- a) Que se puede cambiar el plan de nachox a `Premium`.
- b) Que no se puede cambiar el país de juanpi a `ES`.
- c) Que un intento de modificar a marta_rock (España) a través de la vista no afecta a ninguna fila.
### Requerimiento 6. La página principal, blindada

**Solicitado por:** Carmen Lozano (Data Analyst) y Hugo Marín (Backend Lead)

La página principal del cuadro de mando muestra el consumo por país y plan, y Carmen no quiere que deje de funcionar porque alguien cambie una tabla sin avisar. Crea la vista `dbo.vw_bi_consumo_pais_plan` **enlazada al esquema** con el país del usuario, el plan, las reproducciones válidas y los minutos con un decimal. Entrega la consulta ordenada por minutos de mayor a menor.

Después, responde a Hugo: Backend quiere lanzar el plan `Premium Estudiante Anual` (24 caracteres) y necesita ampliar `usuarios.plan_suscripcion` de `nvarchar(20)` a `nvarchar(30)`. Ejecuta el `ALTER TABLE`, explica qué ocurre y describe los pasos para hacer el cambio sin dejar el cuadro de mando roto.

### Requerimiento 7. Mapa de dependencias

**Solicitado por:** Raúl Benet (Data)

Raúl quiere saber qué habría que tocar si un día cambia la regla de reproducción válida. Con `sys.dm_sql_referencing_entities`, obtén:

- a) Las vistas `vw_bi_` que leen directamente la tabla `dbo.reproducciones`.
- b) Las vistas que leen directamente `dbo.vw_bi_hechos_reproduccion`.

