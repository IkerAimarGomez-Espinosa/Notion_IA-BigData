# 📋Clase 11 - Subconsultas

## El caso Sonora

![Imagen](images/clase-11-subconsultas/image-001.png)

---

## Preparación del entorno

> 💡 Antes de empezar hay que comprobar el motor, conectarse con SSMS 22 y crear la base de datos del caso con el nivel de compatibilidad de SQL Server 2025 (**170**). Esta base de datos se usará en las cinco sesiones de SQL.

### Paso 1. Comprobar la versión del motor

Abre SSMS 22, conéctate a tu instancia local y ejecuta:

```sql
SELECT @@VERSION AS version_completa,
       SERVERPROPERTY('ProductMajorVersion') AS version_principal;
```

![Imagen](images/clase-11-subconsultas/image-002.png)

**Salida esperada:** `version_principal` debe ser **17**, que corresponde a SQL Server 2025. Si aparece 16 (2022) o 15 (2019), estás conectado a otra instancia.

> Si al conectar aparece un error de certificado, en el diálogo de conexión de SSMS 22 marca la opción **Trust server certificate**. Es habitual en instancias locales de desarrollo con certificado autofirmado.

### Paso 2. Crear la base de datos con nivel de compatibilidad 170

> 💡 Una base de datos nueva hereda el nivel de la base `model`, que en una instalación limpia de 2025 ya es 170. Aun así, lo fijamos de forma explícita: si la instancia se actualizó desde una versión anterior, `model` puede conservar un nivel antiguo.

> **Atención:** este script borra `SonoraDB` si ya existe. Úsalo solo para crear la base de datos o para resetearla.

```sql
USE master;
GO

IF DB_ID(N'SonoraDB') IS NOT NULL
BEGIN
    ALTER DATABASE SonoraDB SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE SonoraDB;
END;
GO

CREATE DATABASE SonoraDB;
GO

ALTER DATABASE SonoraDB SET COMPATIBILITY_LEVEL = 170;
GO

SELECT name, compatibility_level
FROM sys.databases
WHERE name = N'SonoraDB';
GO
```

**Salida esperada:**

| name | compatibility_level |
| --- | --- |
| SonoraDB | 170 |

> Si en algún momento restauras una base de datos desde un backup (por ejemplo, AdventureWorks), comprueba su nivel con la misma consulta sobre `sys.databases`. Si no es 170, súbelo con `ALTER DATABASE <nombre> SET COMPATIBILITY_LEVEL = 170;` o algunas funciones de 2025, como las de expresiones regulares, no estarán disponibles.

### Paso 3. Crear las tablas

```sql
USE SonoraDB;
GO

CREATE TABLE dbo.generos (
    genero_id        int           NOT NULL PRIMARY KEY,
    nombre           nvarchar(50)  NOT NULL,
    genero_padre_id  int           NULL REFERENCES dbo.generos (genero_id)
);

CREATE TABLE dbo.artistas (
    artista_id  int            NOT NULL PRIMARY KEY,
    nombre      nvarchar(100)  NOT NULL,
    pais        char(2)        NOT NULL,
    genero_id   int            NOT NULL REFERENCES dbo.generos (genero_id)
);

CREATE TABLE dbo.canciones (
    cancion_id         int            NOT NULL PRIMARY KEY,
    titulo             nvarchar(150)  NOT NULL,
    artista_id         int            NOT NULL REFERENCES dbo.artistas (artista_id),
    genero_id          int            NOT NULL REFERENCES dbo.generos (genero_id),
    duracion_seg       int            NOT NULL,
    fecha_lanzamiento  date           NOT NULL
);

CREATE TABLE dbo.usuarios (
    usuario_id        int            NOT NULL PRIMARY KEY,
    nombre_usuario    nvarchar(50)   NOT NULL,
    email             nvarchar(150)  NOT NULL,
    pais              char(2)        NOT NULL,
    plan_suscripcion  nvarchar(20)   NOT NULL,
    fecha_alta        date           NOT NULL
);

CREATE TABLE dbo.reproducciones (
    reproduccion_id      int           NOT NULL PRIMARY KEY,
    usuario_id           int           NOT NULL REFERENCES dbo.usuarios (usuario_id),
    cancion_id           int           NULL REFERENCES dbo.canciones (cancion_id),
    fecha_hora           datetime2(0)  NOT NULL,
    segundos_escuchados  int           NOT NULL,
    dispositivo          nvarchar(20)  NOT NULL,
    tipo_contenido       nvarchar(10)  NOT NULL
        CHECK (tipo_contenido IN (N'Canción', N'Anuncio'))
);

-- Las tablas de staging no llevan restricciones: reciben los datos tal como llegan
CREATE TABLE dbo.stg_reproducciones (
    reproduccion_id      int,
    usuario_id           int,
    cancion_id           int,
    fecha_hora           datetime2(0),
    segundos_escuchados  int,
    dispositivo          nvarchar(20),
    tipo_contenido       nvarchar(10)
);

CREATE TABLE dbo.stg_usuarios (
    usuario_id        int,
    nombre_usuario    nvarchar(50),
    email             nvarchar(150),
    pais              char(2),
    plan_suscripcion  nvarchar(20),
    fecha_alta        date
);

CREATE TABLE dbo.empleados (
    empleado_id  int            NOT NULL PRIMARY KEY,
    nombre       nvarchar(100)  NOT NULL,
    puesto       nvarchar(50)   NOT NULL,
    jefe_id      int            NULL REFERENCES dbo.empleados (empleado_id)
);
GO
```

### Paso 4. Cargar los datos

```sql
INSERT INTO dbo.generos (genero_id, nombre, genero_padre_id) VALUES
(1, N'Música', NULL),
(2, N'Pop', 1), (3, N'Urbano', 1), (4, N'Rock', 1), (5, N'Electrónica', 1),
(6, N'Reguetón', 3), (7, N'Trap', 3), (8, N'Hip Hop', 3),
(9, N'Indie', 4), (10, N'Metal', 4),
(11, N'House', 5), (12, N'Techno', 5),
(13, N'Dembow', 6);

INSERT INTO dbo.artistas (artista_id, nombre, pais, genero_id) VALUES
(1, N'Luna Roja', 'ES', 2),
(2, N'MC Brisa', 'PR', 6),
(3, N'Kaiser Beats', 'DE', 12),
(4, N'Los Voltios', 'AR', 9),
(5, N'Nébula', 'MX', 7),
(6, N'Hierro Fundido', 'ES', 10),
(7, N'DJ Coral', 'CO', 11),
(8, N'Verso Libre', 'ES', 8),
(9, N'Sara Cometa', 'ES', 2);

INSERT INTO dbo.canciones (cancion_id, titulo, artista_id, genero_id, duracion_seg, fecha_lanzamiento) VALUES
(101, N'Verano en Bucle',   1, 2,  185, '2026-06-12'),
(102, N'Neón',              1, 2,  201, '2025-11-03'),
(103, N'Perreo Lunar',      2, 6,  176, '2026-07-01'),
(104, N'Calle 23',          2, 6,  190, '2026-03-15'),
(105, N'Fábrica 4AM',       3, 12, 412, '2025-09-20'),
(106, N'Corriente Alterna', 4, 9,  234, '2026-02-10'),
(107, N'Cables Sueltos',    4, 9,  198, '2024-05-08'),
(108, N'Humo Violeta',      5, 7,  167, '2026-08-22'),
(109, N'Diamantes Rotos',   5, 7,  182, '2026-04-30'),
(110, N'Forja',             6, 10, 305, '2023-10-31'),
(111, N'Marea Alta',        7, 11, 356, '2026-05-19'),
(112, N'Rimas de Barrio',   8, 8,  214, '2025-12-01'),
(113, N'Galaxia Dembow',    2, 13, 158, '2026-09-04'),
(114, N'Sin Cobertura',     5, 7,  175, '2026-09-18'),
(115, N'Latido',            7, 11, 330, '2025-07-07');

INSERT INTO dbo.usuarios (usuario_id, nombre_usuario, email, pais, plan_suscripcion, fecha_alta) VALUES
(1, N'alexbeats',  N'alex.beats@mail.com',  'ES', N'Premium',  '2025-01-10'),
(2, N'marta_rock', N'marta.rock@mail.com',  'ES', N'Free',     '2025-06-02'),
(3, N'juanpi',     N'juanpi@mail.com',      'MX', N'Premium',  '2024-11-20'),
(4, N'sofi.trap',  N'sofi.trap@mail.com',   'AR', N'Familiar', '2026-02-14'),
(5, N'dani_dj',    N'dani.dj@mail.com',     'CO', N'Free',     '2026-08-30'),
(6, N'lucia.m',    N'lucia.m@mail.com',     'ES', N'Premium',  '2026-09-01'),
(7, N'nachox',     N'nachox@mail.com',      'MX', N'Free',     '2025-03-18'),
(8, N'vale_indie', N'vale.indie@mail.com',  'AR', N'Premium',  '2026-05-05');

INSERT INTO dbo.reproducciones (reproduccion_id, usuario_id, cancion_id, fecha_hora, segundos_escuchados, dispositivo, tipo_contenido) VALUES
(1,  1, 103,  '2026-09-14 08:15', 176, N'Móvil',      N'Canción'),
(2,  1, 108,  '2026-09-14 08:18', 167, N'Móvil',      N'Canción'),
(3,  1, 103,  '2026-09-14 19:40', 176, N'Web',        N'Canción'),
(4,  2, 106,  '2026-09-14 10:02', 234, N'Web',        N'Canción'),
(5,  2, NULL, '2026-09-14 10:06', 30,  N'Web',        N'Anuncio'),
(6,  2, 110,  '2026-09-14 10:07', 120, N'Web',        N'Canción'),
(7,  3, 105,  '2026-09-14 23:10', 412, N'Escritorio', N'Canción'),
(8,  3, 111,  '2026-09-15 00:05', 356, N'Escritorio', N'Canción'),
(9,  4, 108,  '2026-09-15 17:30', 167, N'Móvil',      N'Canción'),
(10, 4, 109,  '2026-09-15 17:33', 20,  N'Móvil',      N'Canción'),
(11, 4, 103,  '2026-09-15 17:35', 176, N'Móvil',      N'Canción'),
(12, 5, NULL, '2026-09-15 21:00', 30,  N'Móvil',      N'Anuncio'),
(13, 5, 111,  '2026-09-15 21:01', 356, N'Móvil',      N'Canción'),
(14, 1, 101,  '2026-09-16 07:50', 185, N'Móvil',      N'Canción'),
(15, 6, 101,  '2026-09-16 12:00', 185, N'Smart TV',   N'Canción'),
(16, 6, 102,  '2026-09-16 12:04', 201, N'Smart TV',   N'Canción'),
(17, 6, 101,  '2026-09-16 12:08', 15,  N'Smart TV',   N'Canción'),
(18, 7, 112,  '2026-09-16 16:20', 214, N'Móvil',      N'Canción'),
(19, 7, NULL, '2026-09-16 16:24', 30,  N'Móvil',      N'Anuncio'),
(20, 7, 104,  '2026-09-16 16:25', 190, N'Móvil',      N'Canción'),
(21, 3, 106,  '2026-09-17 09:10', 234, N'Web',        N'Canción'),
(22, 3, 107,  '2026-09-17 09:14', 198, N'Web',        N'Canción'),
(23, 2, 107,  '2026-09-17 18:45', 198, N'Móvil',      N'Canción'),
(24, 4, 113,  '2026-09-18 20:00', 158, N'Móvil',      N'Canción'),
(25, 4, 103,  '2026-09-18 20:03', 176, N'Móvil',      N'Canción'),
(26, 1, 113,  '2026-09-18 22:30', 158, N'Móvil',      N'Canción'),
(27, 5, 105,  '2026-09-19 02:15', 200, N'Móvil',      N'Canción'),
(28, 6, 109,  '2026-09-19 11:00', 182, N'Smart TV',   N'Canción'),
(29, 7, 103,  '2026-09-19 13:40', 176, N'Móvil',      N'Canción'),
(30, 1, 103,  '2026-09-20 09:05', 176, N'Móvil',      N'Canción'),
(31, 3, 112,  '2026-09-20 10:30', 214, N'Escritorio', N'Canción'),
(32, 2, NULL, '2026-09-20 18:00', 30,  N'Web',        N'Anuncio'),
(33, 2, 106,  '2026-09-20 18:01', 234, N'Web',        N'Canción');

-- Lote recibido el 21/09: las filas 32 y 33 llegan repetidas (ya estaban cargadas)
INSERT INTO dbo.stg_reproducciones (reproduccion_id, usuario_id, cancion_id, fecha_hora, segundos_escuchados, dispositivo, tipo_contenido) VALUES
(32, 2, NULL, '2026-09-20 18:00', 30,  N'Web',        N'Anuncio'),
(33, 2, 106,  '2026-09-20 18:01', 234, N'Web',        N'Canción'),
(34, 6, 113,  '2026-09-21 08:00', 158, N'Smart TV',   N'Canción'),
(35, 4, 114,  '2026-09-21 18:30', 22,  N'Móvil',      N'Canción'),
(36, 3, 110,  '2026-09-21 21:00', 305, N'Escritorio', N'Canción'),
(37, 5, NULL, '2026-09-21 22:00', 30,  N'Móvil',      N'Anuncio'),
(38, 5, 104,  '2026-09-21 22:01', 190, N'Móvil',      N'Canción');

-- Lote de altas: los usuarios 6 y 8 ya existen
INSERT INTO dbo.stg_usuarios (usuario_id, nombre_usuario, email, pais, plan_suscripcion, fecha_alta) VALUES
(6,  N'lucia.m',      N'lucia.m@mail.com',      'ES', N'Premium', '2026-09-01'),
(8,  N'vale_indie',   N'vale.indie@mail.com',   'AR', N'Premium', '2026-05-05'),
(9,  N'pau.synth',    N'pau.synth@mail.com',    'ES', N'Free',    '2026-09-21'),
(10, N'kiara_perreo', N'kiara.perreo@mail.com', 'PR', N'Premium', '2026-09-21');

INSERT INTO dbo.empleados (empleado_id, nombre, puesto, jefe_id) VALUES
(1,  N'Irene Salas',   N'CEO',               NULL),
(2,  N'Tomás Vidal',   N'CTO',               1),
(3,  N'Nuria Campos',  N'CMO',               1),
(4,  N'Raúl Benet',    N'Head of Data',      2),
(5,  N'Elena Ruiz',    N'Data Engineer',     4),
(6,  N'Pablo Ortega',  N'Data Engineer',     4),
(7,  N'Carmen Lozano', N'Data Analyst',      4),
(8,  N'Hugo Marín',    N'Backend Lead',      2),
(9,  N'Laura Gil',     N'Marketing Manager', 3),
(10, N'Andrés Pozo',   N'Community Manager', 9);
GO
```

### Paso 5. Verificar la carga

```sql
SELECT 'generos' AS tabla, COUNT(*) AS filas FROM dbo.generos
UNION ALL SELECT 'artistas', COUNT(*) FROM dbo.artistas
UNION ALL SELECT 'canciones', COUNT(*) FROM dbo.canciones
UNION ALL SELECT 'usuarios', COUNT(*) FROM dbo.usuarios
UNION ALL SELECT 'reproducciones', COUNT(*) FROM dbo.reproducciones
UNION ALL SELECT 'stg_reproducciones', COUNT(*) FROM dbo.stg_reproducciones
UNION ALL SELECT 'stg_usuarios', COUNT(*) FROM dbo.stg_usuarios
UNION ALL SELECT 'empleados', COUNT(*) FROM dbo.empleados;
```

**Salida esperada:**

| tabla | filas |
| --- | --- |
| generos | 13 |
| artistas | 9 |
| canciones | 15 |
| usuarios | 8 |
| reproducciones | 33 |
| stg_reproducciones | 7 |
| stg_usuarios | 4 |
| empleados | 10 |

---

# Parte 01 · Teoría

## 1. Subconsultas

![Imagen](images/clase-11-subconsultas/image-003.png)

### Analogía

![Imagen](images/clase-11-subconsultas/image-004.png)

### Sintaxis

```sql
-- Escalar
SELECT columnas
FROM tabla
WHERE columna > (SELECT AGREGADO(columna) FROM otra_tabla);

-- De lista
SELECT columnas
FROM tabla
WHERE columna IN (SELECT columna FROM otra_tabla WHERE condicion);

-- De tabla (tabla derivada): el alias es obligatorio
SELECT t.columna
FROM (SELECT ... FROM ... GROUP BY ...) AS t;

-- Correlacionada: la subconsulta usa una columna de la consulta externa
SELECT e.columna
FROM tabla AS e
WHERE e.valor = (SELECT MAX(i.valor) FROM tabla AS i WHERE i.clave = e.clave);

-- De existencia
SELECT columnas
FROM tabla AS e
WHERE EXISTS (SELECT 1 FROM otra_tabla AS i WHERE i.clave = e.clave);
```

**Reglas de T-SQL que conviene recordar:**

- Una subconsulta escalar que devuelve más de una fila provoca el error **Msg 512** (*Subquery returned more than 1 value*).
- No se puede usar `ORDER BY` dentro de una subconsulta salvo que vaya acompañado de `TOP` u `OFFSET` (error **Msg 1033**).
- Una tabla derivada siempre necesita alias: `FROM (SELECT ...) AS t`.
- Dentro de `EXISTS` da igual lo que pongas en el `SELECT`; por convención se escribe `SELECT 1`.
![Imagen](images/clase-11-subconsultas/image-005.png)

![Imagen](images/clase-11-subconsultas/image-006.png)

![Imagen](images/clase-11-subconsultas/image-007.png)

![Imagen](images/clase-11-subconsultas/image-008.png)

### ¿ Cuando usar subqueries?

![Imagen](images/clase-11-subconsultas/image-009.png)

![Imagen](images/clase-11-subconsultas/image-010.png)

### Categorías de las Subqueries

![Imagen](images/clase-11-subconsultas/image-011.png)

![Imagen](images/clase-11-subconsultas/image-012.png)

![Imagen](images/clase-11-subconsultas/image-013.png)

### Como ejecutan una subquerie las bases de datos?

![Imagen](images/clase-11-subconsultas/image-014.png)

### SubQuery en la cláusula SELECT

![Imagen](images/clase-11-subconsultas/image-015.png)

![Imagen](images/clase-11-subconsultas/image-016.png)

![Imagen](images/clase-11-subconsultas/image-017.png)

### SubQuery en la cláusula JOIN

![Imagen](images/clase-11-subconsultas/image-018.png)

### SubQuery en la cláusula WHERE

![Imagen](images/clase-11-subconsultas/image-019.png)

![Imagen](images/clase-11-subconsultas/image-020.png)

![Imagen](images/clase-11-subconsultas/image-021.png)

![Imagen](images/clase-11-subconsultas/image-022.png)

### SubQuery con el operador IN

![Imagen](images/clase-11-subconsultas/image-023.png)

![Imagen](images/clase-11-subconsultas/image-024.png)

### SubQuery con ANY | ALL

![Imagen](images/clase-11-subconsultas/image-025.png)

### Subquieries correlacionadas y no correlacionadas

![Imagen](images/clase-11-subconsultas/image-026.png)

![Imagen](images/clase-11-subconsultas/image-027.png)

![Imagen](images/clase-11-subconsultas/image-028.png)

![Imagen](images/clase-11-subconsultas/image-029.png)

![Imagen](images/clase-11-subconsultas/image-030.png)

![Imagen](images/clase-11-subconsultas/image-031.png)

![Imagen](images/clase-11-subconsultas/image-032.png)

![Imagen](images/clase-11-subconsultas/image-033.png)

> 💡

### Ejemplo 1. Subconsulta escalar en `WHERE`

> **Pregunta de negocio:** ¿Qué canciones del catálogo duran más que la media?

**Paso 1.** Resolver la pregunta interna por separado:

```sql
SELECT AVG(duracion_seg) FROM dbo.canciones;
```

Devuelve **232**. 

Ojo: la media real es 232,2, pero `AVG` sobre una columna `int` devuelve un `int` y trunca los decimales.

**Paso 2.** Usar ese valor dentro de la consulta principal:

```sql
SELECT titulo, duracion_seg
FROM dbo.canciones
WHERE duracion_seg > (SELECT AVG(duracion_seg) FROM dbo.canciones)
ORDER BY duracion_seg DESC;
```

**Salida esperada:**

| titulo | duracion_seg |
| --- | --- |
| Fábrica 4AM | 412 |
| Marea Alta | 356 |
| Latido | 330 |
| Forja | 305 |
| Corriente Alterna | 234 |

### Ejemplo 2. Subconsulta escalar en `SELECT`

> **Pregunta de negocio:** para las canciones de Luna Roja (artista 1), ¿Cuánto se desvía su duración de la media del catálogo?

```sql
SELECT titulo,
       duracion_seg,
       (SELECT AVG(duracion_seg) FROM dbo.canciones) AS media_catalogo,
       duracion_seg - (SELECT AVG(duracion_seg) FROM dbo.canciones) AS diferencia
FROM dbo.canciones
WHERE artista_id = 1;
```

**Salida esperada:**

| titulo | duracion_seg | media_catalogo | diferencia |
| --- | --- | --- | --- |
| Verano en Bucle | 185 | 232 | -47 |
| Neón | 201 | 232 | -31 |

> Para obtener la media con decimales basta con forzar un tipo decimal: `AVG(duracion_seg * 1.0)` devuelve **232.200000**. En ingeniería de datos este truncamiento silencioso es una fuente clásica de errores en informes.

### Ejemplo 3. Subconsulta de lista con `IN` (y subconsultas anidadas)

**Pregunta de negocio:** Marketing quiere enviar una notificación a los usuarios que han escuchado alguna canción de **Nébula**.

**Paso 1.** La consulta más interna obtiene el `artista_id` de Nébula (5).
**Paso 2.** La siguiente obtiene las canciones de ese artista (108, 109, 114).
**Paso 3.** La siguiente obtiene los usuarios que han reproducido alguna de esas canciones.
**Paso 4.** La consulta externa muestra los datos de esos usuarios.

```sql
SELECT nombre_usuario, plan_suscripcion
FROM dbo.usuarios
WHERE usuario_id IN (
    SELECT usuario_id
    FROM dbo.reproducciones
    WHERE cancion_id IN (
        SELECT cancion_id
        FROM dbo.canciones
        WHERE artista_id = (SELECT artista_id FROM dbo.artistas WHERE nombre = N'Nébula')
    )
)
ORDER BY nombre_usuario;
```

**Salida esperada:**

| nombre_usuario | plan_suscripcion |
| --- | --- |
| alexbeats | Premium |
| lucia.m | Premium |
| sofi.trap | Familiar |

> `IN` elimina duplicados de forma natural: alexbeats ha escuchado varias veces a Nébula, pero aparece una sola vez. Con un `JOIN` habría que añadir `DISTINCT`.

### Ejemplo 4. Tabla derivada en `FROM`

**Pregunta de negocio:** ¿qué usuarios han reproducido 5 o más canciones?

**Paso 1.** La tabla derivada `t` calcula cuántas canciones ha reproducido cada usuario.
**Paso 2.** La consulta externa trata `t` como una tabla más: la une con `usuarios` y filtra.

```sql
SELECT u.nombre_usuario, u.plan_suscripcion, t.num_reproducciones
FROM (
    SELECT usuario_id, COUNT(*) AS num_reproducciones
    FROM dbo.reproducciones
    WHERE tipo_contenido = N'Canción'
    GROUP BY usuario_id
) AS t
JOIN dbo.usuarios AS u ON u.usuario_id = t.usuario_id
WHERE t.num_reproducciones >= 5
ORDER BY t.num_reproducciones DESC, u.nombre_usuario;
```

**Salida esperada:**

| nombre_usuario | plan_suscripcion | num_reproducciones |
| --- | --- | --- |
| alexbeats | Premium | 6 |
| juanpi | Premium | 5 |
| sofi.trap | Familiar | 5 |

### Ejemplo 5. Subconsulta correlacionada

**Pregunta de negocio:** Para cada usuario, ¿cuál fue su última reproducción?

**Paso 1.** Para cada fila `r` de la consulta externa, la subconsulta busca la fecha máxima **de ese mismo usuario** (`r2.usuario_id = r.usuario_id`).
**Paso 2.** Solo se quedan las filas cuya fecha coincide con esa fecha máxima.

```sql
SELECT u.nombre_usuario, r.fecha_hora, c.titulo
FROM dbo.reproducciones AS r
JOIN dbo.usuarios AS u ON u.usuario_id = r.usuario_id
LEFT JOIN dbo.canciones AS c ON c.cancion_id = r.cancion_id
WHERE r.fecha_hora = (
    SELECT MAX(r2.fecha_hora)
    FROM dbo.reproducciones AS r2
    WHERE r2.usuario_id = r.usuario_id
)
ORDER BY r.fecha_hora;
```

**Salida esperada:**

| nombre_usuario | fecha_hora | titulo |
| --- | --- | --- |
| sofi.trap | 2026-09-18 20:03:00 | Perreo Lunar |
| dani_dj | 2026-09-19 02:15:00 | Fábrica 4AM |
| lucia.m | 2026-09-19 11:00:00 | Diamantes Rotos |
| nachox | 2026-09-19 13:40:00 | Perreo Lunar |
| alexbeats | 2026-09-20 09:05:00 | Perreo Lunar |
| juanpi | 2026-09-20 10:30:00 | Rimas de Barrio |
| marta_rock | 2026-09-20 18:01:00 | Corriente Alterna |

> `vale_indie` no aparece porque no tiene reproducciones. Conceptualmente, la subconsulta se evalúa una vez por cada fila externa; el optimizador suele reescribirla de forma eficiente, pero en tablas grandes conviene revisar el plan de ejecución (lo veremos en la Sesión 13).

### Ejemplo 6. `EXISTS`

**Pregunta de negocio:** ¿qué artistas han publicado al menos una canción en 2026?

```sql
SELECT a.nombre, a.pais
FROM dbo.artistas AS a
WHERE EXISTS (
    SELECT 1
    FROM dbo.canciones AS c
    WHERE c.artista_id = a.artista_id
      AND c.fecha_lanzamiento >= '2026-01-01'
)
ORDER BY a.nombre;
```

**Salida esperada:**

| nombre | pais |
| --- | --- |
| DJ Coral | CO |
| Los Voltios | AR |
| Luna Roja | ES |
| MC Brisa | PR |
| Nébula | MX |

> `EXISTS` se detiene en cuanto encuentra la primera fila que cumple la condición. No le importa cuántas canciones haya: solo si hay alguna.

### Ejemplo 7. La trampa de `NOT IN` con valores `NULL`

**Pregunta de negocio:** ¿Qué canciones del catálogo no se han reproducido nunca?

**Paso 1.** El primer intento con `NOT IN`:

```sql
SELECT cancion_id, titulo
FROM dbo.canciones
WHERE cancion_id NOT IN (SELECT cancion_id FROM dbo.reproducciones);
```

**Salida:** 0 filas. Sin error y sin aviso.

**Paso 2.** Entender por qué. La subconsulta devuelve también los `NULL` de los anuncios. `NOT IN (101, 103, NULL, ...)` equivale a `cancion_id <> 101 AND cancion_id <> 103 AND cancion_id <> NULL ...`, y cualquier comparación con `NULL` da `UNKNOWN`. Como la condición completa nunca llega a ser `TRUE`, no se devuelve ninguna fila.

**Paso 3.** La versión correcta con `NOT EXISTS`, que no se ve afectada por los `NULL`:

```sql
SELECT c.cancion_id, c.titulo
FROM dbo.canciones AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM dbo.reproducciones AS r
    WHERE r.cancion_id = c.cancion_id
);
```

**Salida esperada:**

| cancion_id | titulo |
| --- | --- |
| 114 | Sin Cobertura |
| 115 | Latido |

> **Regla práctica:** para “lo que no está en”, usa `NOT EXISTS`. Si usas `NOT IN`, filtra los nulos dentro de la subconsulta (`WHERE cancion_id IS NOT NULL`). Este fallo es especialmente peligroso en pipelines porque no rompe nada: simplemente deja de cargar datos.

### Ejemplo 8. `NOT EXISTS` como anti-join para la carga incremental

**Contexto de ingeniería de datos:** el lote del 21/09 está en `stg_reproducciones`. La app reenvía a veces filas que ya se cargaron (las 32 y 33). Hay que cargar solo las nuevas y que la carga pueda repetirse sin duplicar.

**Paso 1.** Identificar las filas del staging que aún no están en destino:

```sql
SELECT s.reproduccion_id, s.usuario_id, s.cancion_id, s.fecha_hora
FROM dbo.stg_reproducciones AS s
WHERE NOT EXISTS (
    SELECT 1
    FROM dbo.reproducciones AS r
    WHERE r.reproduccion_id = s.reproduccion_id
)
ORDER BY s.reproduccion_id;
```

**Salida esperada:**

| reproduccion_id | usuario_id | cancion_id | fecha_hora |
| --- | --- | --- | --- |
| 34 | 6 | 113 | 2026-09-21 08:00:00 |
| 35 | 4 | 114 | 2026-09-21 18:30:00 |
| 36 | 3 | 110 | 2026-09-21 21:00:00 |
| 37 | 5 | NULL | 2026-09-21 22:00:00 |
| 38 | 5 | 104 | 2026-09-21 22:01:00 |

**Paso 2.** Convertir esa consulta en la carga:

```sql
INSERT INTO dbo.reproducciones
    (reproduccion_id, usuario_id, cancion_id, fecha_hora, segundos_escuchados, dispositivo, tipo_contenido)
SELECT s.reproduccion_id, s.usuario_id, s.cancion_id, s.fecha_hora,
       s.segundos_escuchados, s.dispositivo, s.tipo_contenido
FROM dbo.stg_reproducciones AS s
WHERE NOT EXISTS (
    SELECT 1
    FROM dbo.reproducciones AS r
    WHERE r.reproduccion_id = s.reproduccion_id
);
```

**Salida esperada (pestaña Messages de SSMS):** `(5 rows affected)`. La tabla `reproducciones` pasa de 33 a 38 filas.

**Paso 3.** Ejecutar la misma carga una segunda vez: `(0 rows affected)`. La carga es **idempotente**: repetirla no duplica datos. Este patrón es la base de la carga incremental que automatizaremos en las próximas sesiones.

> **A partir de aquí, todos los ejemplos y ejercicios asumen que esta carga ya se ha realizado** (38 reproducciones).

