# 🧗🏽‍♂️ Practica 02 - Ejercicios modelo SQL

#  Parte 1: Instalación y puesta en marcha de Postgresql

Esta primera parte deja el entorno listo para trabajar: servidor PostgreSQL 18 instalado en Windows 11, la base de datos `northwind` cargada con sus 14 tablas y su diagrama entidad-relación generado desde el propio PostgreSQL.

Nada de lo que hagas aquí requiere escribir consultas complejas. Lo importante es que entiendas **qué** estás instalando y **por qué** cada opción importa, porque los errores más habituales de las semanas siguientes se originan en decisiones tomadas en estos primeros diez minutos.

---

## Sección 1. Descarga del instalador

PostgreSQL no distribuye instaladores propios para Windows. El instalador oficial recomendado lo mantiene **EDB (EnterpriseDB)**, una empresa que contribuye activamente al proyecto.

1. Abre `https://www.postgresql.org/download/windows/`
2. Pulsa en **Download the installer**. El enlace te lleva al sitio de EDB.
3. En la tabla de versiones, elige la fila **PostgreSQL 18** y la columna **Windows x86-64**.
4. Descarga el `.exe` (ocupa unos 350 MB).
> **Nota sobre compatibilidad.** EDB certifica el instalador de la versión 18 sobre Windows Server 2025 y 2022. Windows 11 no aparece en esa lista de certificación, pero funciona sin problemas porque comparte el mismo núcleo. Si algún compañero tiene Windows 10, también le funcionará.

**Antes de continuar, comprueba tu nombre de usuario de Windows.** Abre el símbolo del sistema y escribe:

```plain text
echo %USERNAME%
```

Si el resultado contiene tildes, eñes o caracteres no ASCII (`José`, `Begoña`, `Martín`), **lee la Sección 9 antes de instalar**. Es la causa número uno de instalaciones fallidas y se resuelve en dos minutos si lo sabes de antemano.

---

## Sección 2. Instalación del servidor

### Lanzar el instalador

Haz clic derecho sobre el `.exe` descargado y elige **Ejecutar como administrador**. Sin privilegios de administrador el instalador no puede crear el servicio de Windows ni inicializar el clúster de datos.

### Pantalla "Installation Directory"

Acepta la ruta por defecto:

```plain text
C:\Program Files\PostgreSQL\18
```

El número de versión forma parte de la ruta a propósito: PostgreSQL permite tener varias versiones mayores instaladas en paralelo, cada una con su propia carpeta, su propio servicio y su propio puerto.

### Pantalla "Select Components"

| Componente | ¿Marcar? | Para qué sirve |
| --- | --- | --- |
| PostgreSQL Server | Sí | El motor de base de datos. Es lo que realmente instalamos. |
| pgAdmin 4 | Sí | El cliente gráfico que usaremos en toda la práctica. |
| Stack Builder | Si | Gestor de paquetes para extensiones y drivers adicionales. No lo necesitamos y alarga la instalación. |
| Command Line Tools | Sí | Incluye `psql`, `pg_dump`, `pg_restore`. Imprescindible aunque trabajes en modo gráfico. |

Desmarcar Stack Builder es una decisión consciente, no un descuido: si más adelante necesitas PostGIS o un driver ODBC, puedes instalarlo por separado.

### Pantalla "Data Directory"

Acepta el valor por defecto:

```plain text
C:\Program Files\PostgreSQL\18\data
```

Esta carpeta es el **clúster de datos**. Contiene físicamente todas las bases de datos, los ficheros de configuración y los registros de transacciones. En un entorno de producción nunca estaría en `Program Files`, sino en un disco dedicado, pero para el aula es perfecto.

### Pantalla "Password" — la más importante

El instalador pide la contraseña del superusuario `postgres`. Este usuario es el equivalente a `root`: puede hacer absolutamente todo en el servidor.

- **Anótala.** No hay forma sencilla de recuperarla; si la pierdes, hay que editar el fichero `pg_hba.conf` a mano.
- Para el aula, usa una contraseña corta y compartida por todos (por ejemplo `postgres`). Esto simplifica el soporte y no supone riesgo porque el servidor solo escucha en `localhost`.
- En cualquier entorno real, esta contraseña sería larga y única.
### Pantalla "Port"

Deja el **5432**, que es el puerto estándar de PostgreSQL.

Si el instalador propone 5433, significa que ya hay otro PostgreSQL instalado en la máquina ocupando el 5432. Anótalo, porque tendrás que indicarlo en cada conexión.

### Pantalla "Advanced Options" (configuración regional)

El instalador pide la *locale* del clúster. Determina el orden alfabético de los `ORDER BY` sobre texto y el formato de fechas y números.

Deja el valor **[Default locale]**. No vamos a depender de esta elección: en la Sección 5 forzaremos explícitamente la codificación UTF-8 al crear la base de datos, que es lo que realmente nos importa para el dataset Northwind.

### Finalizar

Al terminar, el instalador ofrece lanzar Stack Builder. **Desmarca la casilla** y pulsa Finish.

---

## Sección 3. Verificación de la instalación

Antes de abrir pgAdmin conviene confirmar que el servidor está realmente vivo. Es un hábito profesional: verificar cada capa antes de subir a la siguiente.

### Comprobar el servicio de Windows

Pulsa `Win + R`, escribe `services.msc` y busca la entrada:

```plain text
postgresql-x64-18
```

Su estado debe ser **En ejecución** y su tipo de inicio **Automático**. Si está detenido, haz clic derecho e **Iniciar**.

### Añadir las herramientas de línea de comandos al PATH

Esto te permitirá usar `psql` desde cualquier carpeta en lugar de tener que navegar hasta el directorio de instalación.

5. Pulsa `Win`, escribe `variables de entorno` y abre **Editar las variables de entorno del sistema**.
6. Pulsa **Variables de entorno...**
7. En **Variables del sistema**, selecciona `Path` y pulsa **Editar**.
8. **Nuevo** y pega:
```plain text
C:\Program Files\PostgreSQL\18\bin
```

9. Acepta todas las ventanas y **abre una consola nueva** (las consolas ya abiertas conservan el PATH antiguo).
### Primera conexión desde la consola

Abre el símbolo del sistema y ejecuta:

```plain text
psql -U postgres -c "SELECT version();"
```

Introduce la contraseña cuando la pida. Deberías ver una línea que empieza por `PostgreSQL 18.x on x86_64-windows`.

Comprueba también la codificación del servidor:

```plain text
psql -U postgres -c "SHOW server_encoding;"
```

Anota el resultado. Si dice `UTF8`, perfecto. Si dice `WIN1252` no pasa nada: la Sección 5 lo resuelve.

---

## Sección 4. Primer arranque de pgAdmin 4

pgAdmin 4 no es una aplicación de escritorio convencional: es una aplicación web que se ejecuta en un servidor local dentro de tu propio equipo. Por eso el primer arranque tarda entre 30 segundos y dos minutos. Ten paciencia y **no lo abras dos veces**.

### Las dos contraseñas

Aquí se produce la confusión más frecuente del curso. Hay **dos contraseñas distintas** y no tienen ninguna relación entre sí:

| Contraseña | Cuándo aparece | Qué protege |
| --- | --- | --- |
| **Master password** de pgAdmin | La primera vez que abres pgAdmin | El almacén local donde pgAdmin guarda tus credenciales guardadas |
| Contraseña del usuario `postgres` | Al conectar con el servidor | El acceso real a la base de datos |

La master password te la inventas tú ahora mismo. Para simplificar, usa la misma que la de `postgres`.

### Conectar con el servidor

10. En el panel izquierdo (**Object Explorer**), despliega **Servers**.
11. Verás una entrada llamada **PostgreSQL 18** que el instalador creó automáticamente. Haz doble clic.
12. Introduce la contraseña del usuario `postgres`.
13. Marca **Save Password** para no tener que teclearla en cada sesión.
Si el árbol se despliega y aparecen `Databases`, `Login/Group Roles` y `Tablespaces`, estás conectado.

---

## Sección 5. Creación de la base de datos `northwind`

El script que vamos a ejecutar **no crea la base de datos**: da por hecho que ya existe y que estás conectado a ella. Tenemos que crearla nosotros.

### Por qué forzamos UTF-8

El fichero `northwind.sql` contiene 402 líneas con caracteres no ingleses: clientes mexicanos (`México D.F.`), suecos (`Berglunds snabbköp`, `Luleå`) y franceses (`Blondesddsl père et fils`). El fichero está codificado en UTF-8.

Si la base de datos se crea heredando una codificación regional de Windows como WIN1252, esos caracteres pueden almacenarse mal o directamente provocar errores de conversión. Forzar UTF-8 elimina el problema de raíz.

### Opción 1: Creación por consulta

Selecciona el servidor en el árbol, abre **Tools → Query Tool** y ejecuta:

```sql
CREATE DATABASE northwind
    WITH ENCODING  = 'UTF8'
         TEMPLATE  = template0;
```

Dos detalles que conviene entender:

- `ENCODING = 'UTF8'` fija la codificación de la base independientemente de la del servidor.
- `TEMPLATE = template0` es obligatorio cuando cambias la codificación. PostgreSQL clona una base plantilla al crear una nueva; `template1` puede contener objetos y una codificación incompatible, mientras que `template0` es una plantilla virgen e inmutable.
### Opción 2: Creación por interfaz gráfica (alternativa)

14. Clic derecho sobre **Databases → Create → Database...**
15. Pestaña **General**: campo `Database` = `northwind`
16. Pestaña **Definition**: `Encoding` = `UTF8`, `Template` = `template0`
17. **Save**
### Comprobación

```sql
SELECT datname, pg_encoding_to_char(encoding) AS codificacion
FROM pg_database
WHERE datname = 'northwind';
```

Resultado esperado:

```plain text
 datname   | codificacion
-----------+--------------
 northwind | UTF8
```

---

## Sección 6. Ejecución del script `northwind.sql`

> 💡 El fichero `northwind.sql` lo puedes [descargar aqui](https://tajamar365.sharepoint.com/:u:/s/3601-MasterIA2026-2027/IQDOXUAfPqviSoFHLwESoz-cAWsY4pUCQPmsQmqsQdOGY5Q?e=TDuzAR).

### Preparar el fichero

Guarda `northwind.sql` en una ruta **corta y sin tildes ni espacios**. Por ejemplo:

```plain text
C:\bbdd\northwind.sql
```

Rutas como `C:\Users\José\Mis Documentos\Máster\` provocan errores difíciles de diagnosticar en las herramientas de línea de comandos.

### El paso que todo el mundo se salta

**Selecciona primero la base de datos ****`northwind`**** en el árbol de la izquierda.** El Query Tool de pgAdmin siempre se abre conectado al objeto que tengas seleccionado en ese momento. Si tienes seleccionada la base `postgres`, el script creará las 14 tablas ahí y luego ninguna consulta de la práctica funcionará.

Despliega **Servers → PostgreSQL 18 → Databases** y haz clic sobre `northwind` para que quede resaltada.

### Abrir y ejecutar

18. **Tools → Query Tool** (o el icono del rayo en la barra superior).
19. Verifica la pestaña que se acaba de abrir: debe decir `northwind/postgres@PostgreSQL 18`. Si dice `postgres/postgres@...`, ciérrala y vuelve al paso anterior.
20. Pulsa el icono de **carpeta** (*Open File*) en la barra del Query Tool.
21. Navega hasta `C:\bbdd\northwind.sql` y ábrelo. Verás las casi 3.900 líneas cargadas en el editor.
22. Pulsa **F5** o el botón ▶ (*Execute script*).
La ejecución tarda entre dos y diez segundos. Al terminar, el panel inferior mostrará un mensaje del tipo:

```plain text
Query returned successfully in 3 secs 421 msec.
```

### Qué hace el script exactamente

Conocer su estructura te permitirá diagnosticar cualquier fallo:

23. Establece parámetros de sesión (codificación del cliente, nivel de mensajes).
24. Ejecuta `DROP TABLE IF EXISTS` sobre las 14 tablas, en orden inverso de dependencias.
25. Crea las 14 tablas con `CREATE TABLE`.
26. Inserta unas 3.400 filas mediante sentencias `INSERT` individuales.
27. Añade al final 14 claves primarias y 13 claves ajenas.
El punto 2 tiene una consecuencia práctica muy útil: **el script se puede volver a ejecutar tantas veces como quieras**. Si te equivocas o quieres partir de cero, basta con lanzarlo de nuevo.

El punto 5 explica por qué los datos se cargan tan rápido: al aplicar las restricciones después de la carga, PostgreSQL no tiene que validar la integridad referencial fila a fila.

### Alternativa desde línea de comandos

Si prefieres la consola, o si pgAdmin da problemas:

```plain text
psql -U postgres -d northwind -f C:\bbdd\northwind.sql
```

Si trabajas en el símbolo del sistema y los mensajes salen con caracteres extraños, ejecuta antes `chcp 65001` para poner la consola en UTF-8.

---

## Sección 7. Validación de la carga

No des por buena una carga solo porque no haya salido un mensaje en rojo. Verifica.

### Refrescar el árbol

Haz clic derecho sobre la base `northwind` y elige **Refresh**. Después despliega:

```plain text
northwind → Schemas → public → Tables
```

Deben aparecer 14 tablas.

### Consulta de verificación rápida

Copia y ejecuta esto en el Query Tool:

```sql
SELECT
    (SELECT count(*) FROM customers)     AS clientes,
    (SELECT count(*) FROM orders)        AS pedidos,
    (SELECT count(*) FROM order_details) AS lineas,
    (SELECT count(*) FROM products)      AS productos,
    (SELECT count(*) FROM employees)     AS empleados,
    (SELECT count(*) FROM suppliers)     AS proveedores;
```

Resultado esperado:

| clientes | pedidos | lineas | productos | empleados | proveedores |
| --- | --- | --- | --- | --- | --- |
| 91 | 830 | 2155 | 77 | 9 | 29 |

Si algún valor difiere, la carga se ha interrumpido. Vuelve a ejecutar el script completo.

### Verificación de las restricciones

```sql
SELECT contype AS tipo, count(*) AS total
FROM pg_constraint c
JOIN pg_namespace n ON n.oid = c.connamespace
WHERE n.nspname = 'public'
GROUP BY contype
ORDER BY 1;
```

Resultado esperado:

```plain text
 tipo | total
------+-------
 f    |    13
 n    |    31
 p    |    14
```

Interpretación de cada código:

| Código | Significado | Total | De dónde sale |
| --- | --- | --- | --- |
| `p` | *primary* — clave primaria | 14 | Una por tabla |
| `f` | *foreign* — clave ajena | 13 | Las relaciones del modelo |
| `n` | *not null* — obligatoriedad de columna | 31 | Las declaraciones `NOT NULL` del script |

### Verificación de la codificación

Esta es la comprobación que delata un problema de UTF-8:

```sql
SELECT customer_id, company_name, city
FROM customers
WHERE customer_id IN ('ANATR', 'BERGS', 'BLONP');
```

Resultado esperado:

```plain text
 customer_id |            company_name            |    city
-------------+------------------------------------+-------------
 ANATR       | Ana Trujillo Emparedados y helados | México D.F.
 BERGS       | Berglunds snabbköp                 | Luleå
 BLONP       | Blondesddsl père et fils           | Strasbourg
```

Si en lugar de `México` ves `MÃ©xico`, o interrogantes, o rombos negros, la base no está en UTF-8. Bórrala con `DROP DATABASE northwind;` y repite la Sección 5 asegurándote de indicar `ENCODING = 'UTF8'`.

---

## Sección 8. Generación del diagrama entidad-relación

pgAdmin 4 incorpora una herramienta **ERD** capaz de hacer ingeniería inversa sobre una base existente: lee el catálogo del sistema, localiza las claves ajenas y dibuja el modelo automáticamente.

Esto significa que el diagrama no es una interpretación ni un dibujo decorativo: es **el reflejo exacto de lo que el motor sabe sobre las relaciones**. Si dos tablas están relacionadas conceptualmente pero nadie declaró la clave ajena, la línea no aparecerá. Ese es precisamente el valor didáctico del ejercicio.

### Generar el diagrama

28. En el árbol, haz **clic derecho sobre la base de datos ****`northwind`**.
29. Selecciona **ERD For Database** (en algunas versiones aparece como *Generate ERD*).
30. Se abrirá una pestaña nueva con las 14 tablas. Tardará unos segundos.
Es importante hacer clic derecho sobre **la base de datos**, no sobre el esquema ni sobre una tabla. Las tres opciones existen y hacen cosas distintas:

- **ERD For Database**: recupera todas las tablas de todos los esquemas.
- **ERD For Schema**: solo las tablas de un esquema; las relaciones que salgan fuera de él se descartan.
- **ERD For Table**: la tabla elegida y aquellas con las que se relaciona, directa o indirectamente. La profundidad se ajusta en `File → Preferences → ERD Tool → Options → Table Relation Depth`.
### Ordenar y exportar

El diagrama inicial suele salir desordenado, con tablas superpuestas.

31. Busca en la barra de herramientas el botón **Auto align** (icono de varita). Reorganiza las tablas y las líneas automáticamente.
32. Ajusta manualmente lo que quieras arrastrando las cajas.
33. Usa el zoom para encuadrar el conjunto.
34. Pulsa **Download image** para exportar el diagrama como imagen.
Guarda la imagen; la necesitarás como entregable y como referencia mientras escribas consultas.

### Lectura del diagrama

Dedica unos minutos a observarlo antes de pasar a las consultas. Localiza estos elementos:

- **`orders`**** y ****`order_details`**** forman el núcleo transaccional.** Casi todas las consultas de negocio pasarán por ahí.
- **`order_details`**** tiene clave primaria compuesta** (`order_id`, `product_id`). Es una tabla puente clásica entre pedidos y productos.
- **`employees`**** se relaciona consigo misma** a través de `reports_to`. Esa línea que sale y entra en la misma caja es la jerarquía de mandos, y se consulta con auto-JOIN.
- **`us_states`**** no tiene ninguna línea.** Está completamente aislada del modelo: nadie declaró clave ajena hacia ella.
- **`customer_demographics`**** y ****`customer_customer_demo`** aparecen conectadas pero están vacías. El modelo las previó y el negocio nunca las usó.
### Complemento: las relaciones en modo texto

El diagrama es visual, pero el mismo dato está en el catálogo y se puede consultar. Ejecuta esto y compáralo con la imagen:

```sql
SELECT conrelid::regclass AS tabla_origen,
       confrelid::regclass AS tabla_destino,
       conname AS nombre_restriccion
FROM pg_constraint
WHERE contype = 'f'
ORDER BY 1, 2;
```

Devuelve las 13 relaciones. Fíjate en la fila donde `tabla_origen` y `tabla_destino` son ambas `employees`: es la autorreferencia que viste en el diagrama.

Saber leer el catálogo del sistema es una habilidad que te servirá en cualquier motor relacional, mucho más allá de esta práctica.

---

## Sección 9. Problemas frecuentes

### "Problem running post-install step. The database cluster installation failed."

**Causa más probable:** tu nombre de usuario de Windows contiene tildes, eñes u otros caracteres no ASCII. EDB documenta explícitamente que no admite caracteres multibyte en nombres de usuario o de máquina.

**Soluciones, por orden de preferencia:**

35. Crea una cuenta local de Windows con nombre ASCII (`alumno`, `dataeng`) e instala desde ella.
36. Desactiva temporalmente el antivirus y Windows Defender durante la instalación, que es la segunda causa más frecuente del mismo mensaje.
37. Desinstala por completo, **borra la carpeta ****`C:\Program Files\PostgreSQL`** si quedó residuo, y reinstala.
Si la instalación falló a medias, la desinstalación limpia es obligatoria antes de reintentar.

### "El puerto 5432 está ocupado"

Hay otro PostgreSQL instalado, o un Docker con PostgreSQL corriendo. Comprueba con:

```plain text
netstat -ano | findstr :5432
```

Desinstala la versión antigua o acepta el puerto 5433 que propone el instalador, anotándolo.

### pgAdmin no arranca o se queda en blanco

Es una aplicación web local. Cierra todas las instancias desde el Administrador de tareas (busca procesos `pgAdmin4`), espera diez segundos y vuelve a abrir. No hagas doble clic repetidamente en el icono.

### "relation does not exist" al consultar las tablas

Ejecutaste el script contra la base equivocada, casi con seguridad `postgres`. Comprueba la pestaña del Query Tool: debe decir `northwind/postgres@...`. Vuelve a la Sección 6.

Para limpiar lo que se creó por error:

```sql
DROP TABLE IF EXISTS customer_customer_demo, customer_demographics,
     employee_territories, order_details, orders, customers, products,
     shippers, suppliers, territories, us_states, categories, region, employees;
```

Ejecútalo **conectado a la base ****`postgres`**, no a `northwind`.

### "permission denied for schema public"

Estás conectado con un rol que no es el propietario de la base. Desde PostgreSQL 15, el esquema `public` ya no permite crear objetos a cualquier usuario. Conéctate como `postgres` o pide que te concedan permisos:

```sql
GRANT CREATE ON SCHEMA public TO tu_usuario;
```

### El diagrama ER sale vacío o sin líneas

Generaste el ERD antes de que se aplicaran las claves ajenas, o el script se interrumpió. Verifica con la consulta de la Sección 7 que existen 13 restricciones de tipo `f` y vuelve a generar el diagrama.

### Los acentos se ven mal solo en la consola, no en pgAdmin

Es un problema de la consola de Windows, no de la base de datos. Ejecuta `chcp 65001` antes de lanzar `psql`.

---

## Comprobación final antes de la Parte 2

Marca cada punto antes de continuar:

- [ ] El servicio `postgresql-x64-18` aparece en ejecución.
- [ ] `psql -U postgres -c "SELECT version();"` devuelve PostgreSQL 18.
- [ ] La base `northwind` existe y su codificación es `UTF8`.
- [ ] La consulta de conteo devuelve 91 / 830 / 2155 / 77 / 9 / 29.
- [ ] Existen 14 claves primarias y 13 claves ajenas.
- [ ] Los nombres con acentos se muestran correctamente.
- [ ] Tienes exportada la imagen del diagrama ER.
Con esto el entorno queda operativo. La Parte 2 empieza directamente sobre este modelo.

# Parte 2: 20 consultas de análisis

Northwind Traders es un distribuidor mayorista de productos alimentarios que importa de 16 países y vende a 91 clientes repartidos por 21 países. Su histórico de operaciones cubre desde julio de 1996 hasta mayo de 1998: 830 pedidos y 2.155 líneas de detalle.

Te incorporas al equipo de datos de la compañía. Las 20 consultas que siguen son peticiones reales del negocio, ordenadas por dificultad técnica creciente.

## Cómo trabajar esta práctica

**Consulta el diagrama ER que generaste en la Parte 1 antes de escribir cada consulta.** Es tu mapa: te dice qué tablas se pueden unir directamente y cuáles necesitan pasar por una tabla intermedia.

**El cálculo del importe de una línea de pedido aparecerá una y otra vez.** La tabla `order_details` no almacena el importe: hay que derivarlo. La fórmula acordada para toda la práctica es:

```sql
ROUND((unit_price::numeric) * quantity * (1 - discount::numeric), 2)
```

El `::numeric` no es decorativo. Las columnas `unit_price`, `discount` y `freight` están declaradas como `real`, un tipo de coma flotante que arrastra errores de redondeo al agregar. Si sumas directamente en `real`, la facturación total de la compañía te sale 1.265.793,0387; si castea a `numeric`, 1.265.793,0395. **Usa siempre ****`::numeric`**** en los cálculos monetarios** o tus resultados no coincidirán con los de tus compañeros.

**Las fechas del dataset son de 1996 a 1998.** Cualquier consulta que uses con `CURRENT_DATE` o `NOW()` devolverá resultados vacíos o sin sentido. Trabaja siempre con fechas literales del rango histórico.

---

## Sección 1. Fundamentos: filtrado y agregación

### Pregunta 1 — Catálogo comercial activo

El equipo de ventas prepara la tarifa de la próxima campaña y necesita el catálogo depurado.

Obtén los productos que **no** están descatalogados y cuyo precio unitario esté entre 10 y 50 euros, ambos incluidos. Muestra el nombre del producto y su precio redondeado a dos decimales, ordenado de mayor a menor precio.

**Columnas esperadas:** `producto`, `precio`

**Técnicas:** `WHERE`, `BETWEEN`, `ROUND()`, alias de columna, `ORDER BY`

> **Pista:** la columna `discontinued` es de tipo `integer`, no booleana. Un producto activo tiene valor 0.

---

### Pregunta 2 — Concentración geográfica de la cartera

Dirección quiere saber en qué mercados está realmente concentrada la base de clientes antes de decidir dónde abrir delegación.

Cuenta cuántos clientes hay en cada país y muestra únicamente aquellos países con **5 o más clientes**, ordenados de mayor a menor. Indica también cuántas ciudades distintas hay en cada uno de esos países.

**Columnas esperadas:** `pais`, `num_clientes`, `num_ciudades`

**Técnicas:** `GROUP BY`, `COUNT()`, `COUNT(DISTINCT ...)`, `HAVING`

> **Pista:** `HAVING` filtra después de agrupar; `WHERE` filtra antes. Aquí la condición se aplica sobre el resultado de un conteo, así que solo una de las dos cláusulas sirve.

---

### Pregunta 3 — Alerta de reposición

Logística necesita detectar qué referencias están en riesgo de rotura de stock.

Localiza los productos activos cuyas unidades en stock sean **inferiores o iguales** a su nivel de reposición. Muestra el nombre, las unidades en stock, el nivel de reposición, las unidades ya pedidas al proveedor y una columna de texto que indique `'CRÍTICO'` cuando el stock sea 0 y `'AVISO'` en el resto de casos.

**Columnas esperadas:** `producto`, `stock`, `nivel_reposicion`, `pedido_a_proveedor`, `situacion`

**Técnicas:** `WHERE` con comparación entre columnas, `CASE WHEN`

> **Pista:** comprueba antes si alguna de estas columnas admite nulos. Un `NULL` en una comparación no devuelve ni verdadero ni falso, y la fila desaparece del resultado sin avisarte.

---

## Sección 2. INNER JOIN

### Pregunta 4 — Ficha completa de producto

Marketing va a rehacer el catálogo impreso y necesita cada producto con su categoría y los datos de contacto de quien lo suministra.

Para los productos suministrados por empresas de **Italia, Francia o España**, muestra el nombre del producto, el nombre de la categoría, el nombre del proveedor, su país y su ciudad. Ordena por país y, dentro de cada país, por nombre de producto.

**Columnas esperadas:** `producto`, `categoria`, `proveedor`, `pais`, `ciudad`

**Técnicas:** `INNER JOIN` de tres tablas, alias de tabla, `WHERE ... IN`

> **Pista:** `products` no se une directamente con nada geográfico. Mira el diagrama: la información de país está en `suppliers`.

---

### Pregunta 5 — Detalle valorizado de un pedido

Atención al cliente recibe una reclamación sobre el pedido **10248** y necesita reconstruir la factura línea a línea.

Muestra, para ese pedido, el nombre del producto, el precio unitario aplicado, la cantidad, el descuento y el importe final de cada línea. Añade el nombre del cliente y la fecha del pedido.

**Columnas esperadas:** `cliente`, `fecha_pedido`, `producto`, `precio_unitario`, `cantidad`, `descuento`, `importe_linea`

**Técnicas:** `INNER JOIN` con `USING`, aritmética entre columnas, `ROUND()`

> **Pista:** `orders` y `order_details` comparten el nombre de columna `order_id`; `order_details` y `products` comparten `product_id`. Cuando los nombres coinciden a ambos lados, `USING(columna)` es más limpio que `ON a.col = b.col` y además evita que la columna aparezca duplicada en el resultado.

---

### Pregunta 6 — Ranking de categorías por facturación

Comité de dirección: ¿qué familias de producto sostienen realmente el negocio?

Calcula la facturación total de cada categoría durante toda la historia de la compañía. Muestra el nombre de la categoría, el número de líneas de pedido que ha generado, el número de productos distintos vendidos y la facturación total. Incluye únicamente las categorías que superen los **100.000 euros** de facturación, ordenadas de mayor a menor.

**Columnas esperadas:** `categoria`, `num_lineas`, `num_productos`, `facturacion`

**Técnicas:** `INNER JOIN` de tres tablas, `GROUP BY`, `SUM()`, `COUNT(DISTINCT ...)`, `HAVING`, `ROUND()`

> **Pista:** el `HAVING` se aplica sobre la expresión agregada completa, no sobre el alias. En PostgreSQL puedes repetir la expresión o envolver la consulta.

---

## Sección 3. Uniones externas, reflexivas y cruzadas

### Pregunta 7 — Clientes sin actividad comercial

Dirección comercial sospecha que hay cuentas abiertas que nunca han llegado a comprar.

Lista **todos** los clientes con el número de pedidos que ha realizado cada uno y la fecha de su último pedido. Los clientes sin ningún pedido deben aparecer igualmente, con un 0 en el conteo y el texto `'SIN PEDIDOS'` en lugar de la fecha. Ordena de forma que los clientes inactivos aparezcan primero.

**Columnas esperadas:** `cliente`, `pais`, `num_pedidos`, `ultimo_pedido`

**Técnicas:** `LEFT JOIN`, `COUNT()` sobre columna de la tabla derecha, `COALESCE()`, `MAX()`

> **Pista:** `COUNT(*)` cuenta filas, incluidas las que el `LEFT JOIN` rellenó con nulos, y te dará 1 para los clientes sin pedidos. `COUNT(columna)` ignora los nulos. Esa diferencia es exactamente el objetivo del ejercicio.

---

### Pregunta 8 — Organigrama de la fuerza de ventas

Recursos Humanos necesita el organigrama del departamento comercial en formato tabla.

Muestra cada empleado con su nombre completo, su cargo, el nombre completo de la persona a la que reporta y el cargo de esa persona. El empleado que no reporta a nadie debe aparecer también, con el texto `'DIRECCIÓN GENERAL'` en el campo del responsable.

**Columnas esperadas:** `empleado`, `cargo`, `responsable`, `cargo_responsable`

**Técnicas:** `SELF JOIN` con `LEFT JOIN`, alias de tabla obligatorios, concatenación de texto, `COALESCE()`

> **Pista:** la misma tabla aparece dos veces en el `FROM`, así que los alias dejan de ser una comodidad y pasan a ser imprescindibles. Piensa en `emp` y `jefe` como si fueran dos tablas distintas.

---

### Pregunta 9 — Rejilla de cobertura categoría × año

Control de gestión quiere una rejilla completa de facturación por categoría y año, **sin huecos**: si una categoría no vendió nada en un año concreto, debe aparecer con un 0, no desaparecer de la tabla.

Genera todas las combinaciones posibles de las 8 categorías con los 3 años del histórico (24 filas) y asocia a cada combinación su facturación. Ordena por categoría y año.

**Columnas esperadas:** `categoria`, `anio`, `facturacion`

**Técnicas:** `CROSS JOIN` para generar la rejilla, `LEFT JOIN` contra los datos reales, `COALESCE()`, `EXTRACT()`

> **Pista:** este es el patrón clásico para informes con huecos. Primero construyes el "esqueleto" de todas las combinaciones posibles con un `CROSS JOIN`, y solo después cuelgas los datos reales con un `LEFT JOIN`. Si lo haces al revés, las combinaciones sin datos nunca aparecerán.

---

### Pregunta 10 — Mapa de países: clientes frente a proveedores

Expansión internacional quiere una única tabla que muestre, para cada país en el que la compañía tiene presencia, cuántos clientes y cuántos proveedores hay. Deben aparecer los países que solo tienen clientes, los que solo tienen proveedores y los que tienen ambos.

**Columnas esperadas:** `pais`, `num_clientes`, `num_proveedores`, `tipo_presencia`

Donde `tipo_presencia` toma los valores `'SOLO CLIENTES'`, `'SOLO PROVEEDORES'` o `'AMBOS'`.

**Técnicas:** `FULL JOIN` entre dos subconsultas agregadas, `COALESCE()`, `CASE WHEN`

> **Pista:** en un `FULL JOIN` la columna de unión puede venir nula por cualquiera de los dos lados. Si escribes `SELECT a.country`, perderás el nombre de los países que solo existen en la tabla `b`.

---

## Sección 4. Operadores de conjunto

### Pregunta 11 — Directorio unificado de contactos

Sistemas va a migrar el CRM y necesita una exportación única con todos los contactos de la compañía, vengan de donde vengan.

Construye una sola tabla que reúna los contactos de clientes, los de proveedores y los empleados. Cada fila debe indicar el origen (`'CLIENTE'`, `'PROVEEDOR'`, `'EMPLEADO'`), el nombre de la persona de contacto **en mayúsculas**, la organización a la que pertenece, la ciudad y el país. Para los empleados, la organización es el literal `'NORTHWIND TRADERS'` y el nombre de contacto se forma concatenando nombre y apellidos.

Ordena por origen y luego por país.

**Columnas esperadas:** `origen`, `contacto`, `organizacion`, `ciudad`, `pais`

**Técnicas:** `UNION ALL`, `UPPER()`, concatenación con `||` o `CONCAT()`, literales como columna

> **Pista:** las tres consultas deben devolver el mismo número de columnas, en el mismo orden y con tipos compatibles. Razona por qué aquí conviene `UNION ALL` y no `UNION`: ¿qué pasaría si un cliente y un proveedor compartieran nombre de contacto y ciudad?

---

### Pregunta 12 — Mercados con desequilibrio

Compras y Ventas mantienen una discusión recurrente: ¿en qué países vendemos sin tener proveedor local, y en cuáles coincidimos?

Resuelve las dos preguntas en dos consultas independientes:

**a)** Países donde hay clientes pero **ningún** proveedor.
**b)** Países donde hay **a la vez** clientes y proveedores.

Ordena ambos resultados alfabéticamente.

**Columnas esperadas:** `pais`

**Técnicas:** `EXCEPT`, `INTERSECT`

> **Pista:** los operadores de conjunto eliminan duplicados automáticamente, a diferencia de `UNION ALL`. Compara el resultado del apartado (a) con el que obtendrías usando un `LEFT JOIN ... WHERE ... IS NULL`: llegan al mismo sitio por caminos distintos, y conviene que sepas escribir los dos.

---

## Sección 5. Subconsultas

### Pregunta 13 — Clientes que nunca han comprado pescado

El responsable de la categoría Seafood quiere una lista de cuentas sobre las que hacer campaña de captación.

Localiza los clientes que **nunca** han incluido un producto de la categoría `'Seafood'` en ninguno de sus pedidos. Muestra el nombre del cliente, su país y el número total de pedidos que sí ha realizado, de mayor a menor.

**Columnas esperadas:** `cliente`, `pais`, `pedidos_realizados`

**Técnicas:** anti join con `NOT EXISTS`, subconsulta correlacionada, `INNER JOIN` en la subconsulta

> **Pista:** hay tres formas de escribir un anti join: `NOT EXISTS`, `NOT IN` y `LEFT JOIN ... WHERE ... IS NULL`. Escribe la versión con `NOT EXISTS` y después prueba con `NOT IN`. Si la subconsulta de `NOT IN` puede devolver algún `NULL`, el resultado será una tabla vacía sin ningún mensaje de error. Es uno de los fallos más difíciles de detectar en SQL.

---

### Pregunta 14 — Productos por encima de la media

El comité de precios quiere identificar el segmento premium del catálogo.

Muestra los productos activos cuyo precio unitario supere el precio medio de **todo** el catálogo. Incluye en cada fila el precio del producto, el precio medio general y la diferencia entre ambos, todo redondeado a dos decimales. Ordena por diferencia descendente.

**Columnas esperadas:** `producto`, `precio`, `precio_medio_catalogo`, `diferencia`

**Técnicas:** subconsulta escalar en `WHERE`, subconsulta escalar en `SELECT`, aritmética

> **Pista:** una subconsulta escalar es aquella que devuelve exactamente una fila y una columna, y por eso se puede usar donde iría un valor. Fíjate en que la misma subconsulta aparece en dos sitios; más adelante verás cómo evitar esa repetición con un CTE.

---

### Pregunta 15 — Ticket medio por cliente

Dirección comercial quiere segmentar la cartera por valor medio de pedido, no por volumen total.

Calcula, para cada cliente que haya comprado alguna vez, el número de pedidos, el importe total acumulado y el importe medio por pedido. Muestra los 15 clientes con mayor ticket medio.

El cálculo tiene dos niveles: primero hay que obtener el importe de cada pedido sumando sus líneas, y solo después promediar esos importes por cliente. **Promediar directamente las líneas daría un resultado distinto y equivocado.**

**Columnas esperadas:** `cliente`, `pais`, `num_pedidos`, `importe_total`, `ticket_medio`

**Técnicas:** subconsulta en `FROM` (tabla derivada), agregación en dos niveles, `LIMIT`

> **Pista:** toda subconsulta en `FROM` necesita un alias en PostgreSQL, aunque no lo uses. Si lo olvidas, el error que verás es `subquery in FROM must have an alias`.

---

## Sección 6. Subconsultas correlacionadas y CTE

### Pregunta 16 — El producto más caro de cada categoría

El equipo de compras quiere revisar el posicionamiento de precio en cada familia.

Para cada categoría, muestra el producto con el precio unitario más alto. Incluye el nombre de la categoría, el nombre del producto, su precio y el precio medio de su categoría.

Resuélvelo con una **subconsulta correlacionada**: para cada producto, comprueba si su precio coincide con el máximo de su propia categoría.

**Columnas esperadas:** `categoria`, `producto`, `precio`, `precio_medio_categoria`

**Técnicas:** subconsulta correlacionada en `WHERE`, subconsulta correlacionada en `SELECT`, `INNER JOIN`

> **Pista:** una subconsulta correlacionada se ejecuta conceptualmente una vez por cada fila de la consulta externa, porque hace referencia a una columna de esa fila. Eso la hace potente pero costosa. Cuando termines, plantéate cuál sería el coste sobre una tabla de diez millones de filas.

---

### Pregunta 17 — Segmentación ABC de la cartera de clientes

Dirección quiere clasificar a los clientes en tres tramos de valor para asignar recursos comerciales.

Usando expresiones de tabla común (CTE), construye una consulta que:

38. Calcule la facturación total de cada cliente.
39. Divida los clientes en **cuartiles** según esa facturación.
40. Asigne una etiqueta de segmento: `'A - Estratégico'` al cuartil superior, `'B - Consolidado'` al segundo, `'C - Ocasional'` al tercero y `'D - Marginal'` al cuarto.
41. Devuelva, por segmento, el número de clientes, la facturación total del segmento y el porcentaje que representa sobre el total de la compañía.
**Columnas esperadas:** `segmento`, `num_clientes`, `facturacion_segmento`, `porcentaje_sobre_total`

**Técnicas:** `WITH` con varias CTE encadenadas, `NTILE()`, `CASE WHEN`, agregación sobre el resultado de una CTE, cálculo de porcentaje

> **Pista:** encadenar CTE permite leer la consulta de arriba abajo como una receta, en lugar de descifrarla de dentro hacia fuera como ocurre con las subconsultas anidadas. Una CTE puede referirse a las declaradas antes que ella.

---

## Sección 7. Funciones de ventana

### Pregunta 18 — Los tres productos más vendidos de cada categoría

El equipo de categoría necesita el podio de cada familia para negociar con proveedores.

Para cada categoría, obtén los **tres productos con mayor facturación**. Muestra la categoría, la posición dentro de la categoría, el nombre del producto, las unidades vendidas y la facturación.

Incluye además una columna con la posición global del producto en el conjunto de la compañía, para que se vea qué productos son líderes de su nicho pero irrelevantes en el total.

**Columnas esperadas:** `categoria`, `posicion_en_categoria`, `producto`, `unidades`, `facturacion`, `posicion_global`

**Técnicas:** `RANK()` o `ROW_NUMBER()` con `OVER (PARTITION BY ... ORDER BY ...)`, CTE para poder filtrar por la posición, función de ventana sin `PARTITION BY`

> **Pista:** no se puede filtrar por una función de ventana en el `WHERE`, porque las funciones de ventana se evalúan después del filtrado. Necesitas calcularla en una CTE o subconsulta y filtrar fuera.
Piensa también qué ocurriría con `RANK()` frente a `DENSE_RANK()` frente a `ROW_NUMBER()` si dos productos empatasen exactamente en facturación.

---

### Pregunta 19 — Evolución mensual con acumulado y media móvil

Control de gestión prepara el cuadro de mando de la evolución del negocio durante 1997.

Para cada mes de 1997, calcula:

- La facturación del mes.
- El total acumulado desde enero.
- La media móvil de los tres últimos meses (el mes actual y los dos anteriores).
- La facturación del mes anterior.
- La variación porcentual respecto al mes anterior.
**Columnas esperadas:** `mes`, `facturacion`, `acumulado`, `media_movil_3m`, `mes_anterior`, `variacion_pct`

**Técnicas:** `DATE_TRUNC()`, `SUM() OVER (ORDER BY ...)` como total acumulado, definición explícita de marco con `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`, `LAG()`, CTE

> **Pista:** cuando una función de ventana agregada lleva `ORDER BY` pero no especificas marco, PostgreSQL aplica por defecto `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, que es justo lo que quieres para el acumulado. Para la media móvil ese comportamiento por defecto no sirve: ahí tienes que declarar el marco tú.
La primera fila no tiene mes anterior. Decide qué mostrar en ese caso.

---

### Pregunta 20 — Cuadro de mando anual por categoría

Última petición, y la más ambiciosa: el informe anual que se presenta al consejo.

Construye una tabla donde cada fila sea una categoría y las columnas muestren la facturación de 1996, 1997 y 1998 en columnas separadas, más el total de los tres años. Añade al final una fila de totales generales.

Incluye además una columna que indique el peso de cada categoría sobre la facturación total de la compañía, y otra que muestre si la categoría creció o decreció entre 1997 y 1998.

**Columnas esperadas:** `categoria`, `f_1996`, `f_1997`, `f_1998`, `total`, `peso_pct`, `tendencia`

**Técnicas:** pivotado manual con `CASE WHEN` dentro de `SUM()` (o `FILTER`), `ROLLUP` para la fila de totales, `COALESCE()`, `CASE WHEN` para la tendencia, funciones de ventana para el peso

> **Pista:** el pivotado en SQL estándar consiste en convertir filas en columnas mediante una función de agregación que solo suma cuando se cumple una condición. PostgreSQL ofrece dos sintaxis equivalentes: `SUM(CASE WHEN anio = 1997 THEN importe ELSE 0 END)` y la más moderna `SUM(importe) FILTER (WHERE anio = 1997)`. Escribe la versión con `FILTER`, que es específica de PostgreSQL y mucho más legible.
Ten en cuenta que 1996 solo tiene medio año de datos (desde julio) y 1998 llega solo hasta mayo. La tendencia entre 1997 y 1998 no es comparable sin normalizar. Menciónalo en un comentario dentro de tu consulta: detectar que una comparación no es válida vale más que calcularla bien.

---

## Cobertura de técnicas

Esta tabla te sirve para repasar antes del examen y para localizar dónde practicar cada concepto.

| Técnica | Preguntas |
| --- | --- |
| `WHERE`, `BETWEEN`, `IN`, `LIKE` | 1, 3, 4 |
| `NULL` y `COALESCE()` | 3, 7, 9, 10, 19 |
| Funciones de agregación y `ROUND()` | 1, 2, 6, 14 |
| `GROUP BY` y `HAVING` | 2, 6 |
| `INNER JOIN`, alias, `USING` | 4, 5, 6 |
| `LEFT JOIN` | 7, 9 |
| `SELF JOIN` | 8 |
| `CROSS JOIN` | 9 |
| `FULL JOIN` | 10 |
| `UNION ALL` | 11 |
| `EXCEPT`, `INTERSECT` | 12 |
| Semi join y anti join | 12, 13 |
| Subconsulta en `WHERE` | 13, 14, 16 |
| Subconsulta en `SELECT` | 14, 16 |
| Subconsulta en `FROM` | 15 |
| Subconsultas correlacionadas | 13, 16 |
| CTE (`WITH`) | 17, 18, 19, 20 |
| `CASE WHEN` | 3, 10, 17, 20 |
| `NTILE()` | 17 |
| `RANK()` / `ROW_NUMBER()` | 18 |
| `PARTITION BY` | 18 |
| Marcos de ventana (`ROWS BETWEEN`) | 19 |
| `LAG()` | 19 |
| Pivotado y `ROLLUP` | 20 |
| Funciones de texto | 11 |
| Funciones de fecha (`EXTRACT`, `DATE_TRUNC`) | 9, 19, 20 |

## Sección 8. Entrega en GitHub

La entrega no se hace por correo ni como fichero adjunto. **Cada alumno publica su trabajo en un repositorio de GitHub y entrega únicamente el enlace.**

Esto no es un capricho administrativo. En cualquier equipo de datos, el código que escribes vive en un repositorio: es donde se revisa, se comenta y se recupera cuando alguien necesita entender por qué una métrica cambió hace ocho meses. Documentar consultas en Markdown dentro de un repositorio es exactamente lo que harás en tu primer empleo.

### Qué se entrega

Un único enlace público a un repositorio de GitHub, con este contenido:

```plain text
northwind-sql/
├── README.md
├── respuestas.md
└── img/
    ├── diagrama-er.png
    ├── p01.png
    ├── p02.png
    └── ...
```

### Contenido de `README.md`

Es la portada del repositorio: lo primero que se ve y lo primero que se corrige. Debe incluir:

- Título de la práctica y tu nombre completo.
- Versión de PostgreSQL y de pgAdmin que has utilizado.
- Instrucciones breves para reproducir tu trabajo: cómo crear la base de datos y cómo cargar el script.
- El diagrama ER que generaste en la Parte 1, insertado como imagen.
- Un índice con enlace a `respuestas.md`.
### Contenido de `respuestas.md`

Una sección por pregunta, las 20, en orden. Cada una con esta estructura:

```markdown
## Pregunta 7 — Clientes sin actividad comercial

**Enunciado:** Lista todos los clientes con el número de pedidos que ha
realizado cada uno...

**Consulta:**

```sql
-- Clientes con su volumen de pedidos, incluidos los que nunca han comprado
SELECT c.company_name AS cliente,
       c.country      AS pais,
       COUNT(o.order_id) AS num_pedidos
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.company_name, c.country
ORDER BY num_pedidos;
```

**Resultado:**

![Resultado pregunta 7](img/p07.png)

**Comentario:** He usado `COUNT(o.order_id)` en lugar de `COUNT(*)` porque...
```

El bloque de comentario es obligatorio y es lo que más peso tiene en la corrección. Dos o tres frases explicando **por qué** has resuelto la consulta así: qué tipo de JOIN elegiste y por qué, qué alternativa descartaste, qué te sorprendió del resultado. Una consulta correcta sin explicación demuestra que funciona; con explicación demuestra que la entiendes.

### Requisitos técnicos del código

- **Todo el SQL va en bloques de código con la etiqueta del lenguaje** (````sql`). Sin etiqueta no hay coloreado de sintaxis y la corrección se vuelve penosa.
- **Sangrado consistente.** Una cláusula por línea: `SELECT`, `FROM`, `JOIN`, `WHERE`, `GROUP BY`, `ORDER BY`.
- **Alias de tabla en toda consulta con más de una tabla**, y alias de columna en toda expresión calculada. Una columna llamada `?column?` en el resultado es un error de entrega.
- **Cast a ****`numeric`**** en todo cálculo monetario.**
- **Un comentario de una línea al inicio de cada consulta** explicando qué devuelve.
### Capturas de resultado

- Formato PNG, dentro de la carpeta `img/`.
- Nombradas `p01.png` a `p20.png`. Nada de `Captura de pantalla 2026-03-14 a las 18.32.15.png`.
- Deben verse las cabeceras de columna y las primeras filas. Si el resultado tiene cientos de filas, basta con las 10 o 15 primeras más el contador que muestra pgAdmin abajo a la derecha.
- Referéncialas con rutas relativas: `![...](img/p07.png)`. Una ruta absoluta de tu disco duro no se verá en GitHub.
### Pasos para publicar

42. Crea una cuenta en GitHub si aún no la tienes.
43. Crea un repositorio **público** llamado `northwind-sql`. Si lo pones privado, no se podrá corregir.
44. Marca la casilla **Add a README file** al crearlo.
45. Sube los ficheros. Si no manejas Git todavía, la interfaz web sirve: botón **Add file → Upload files**, arrastras y confirmas con **Commit changes**.
46. Verifica el resultado abriendo tu repositorio **en una ventana de incógnito**. Es la única forma fiable de comprobar que es realmente público y que las imágenes cargan.
47. Entrega el enlace, con este formato: `https://github.com/tu-usuario/northwind-sql`
### Errores que invalidan la entrega

| Error | Consecuencia |
| --- | --- |
| Repositorio privado | No se puede corregir. Entrega no evaluada. |
| Imágenes con ruta absoluta o no subidas | Aparecen rotas. Esas preguntas cuentan como no entregadas. |
| SQL pegado como texto plano, sin bloque de código | Penalización en la nota de forma. |
| Enlace a un fichero suelto en lugar de al repositorio | Se solicita reenvío, con la penalización de retraso que corresponda. |
| Un único fichero `.sql` sin Markdown | No cumple el formato de entrega. |

### Sobre la comprobación de autoría

Los commits quedan registrados con fecha y hora. Un repositorio con un único commit de 20 consultas subidas de golpe a las 23:58 del día del plazo no es motivo de suspenso por sí mismo, pero sí una señal. **Ve subiendo el trabajo a medida que lo haces.** El historial de commits es tu mejor defensa si alguien cuestiona la autoría, y es además el hábito profesional que interesa adquirir.

Prioriza la corrección sobre la brevedad. Una consulta larga y legible vale más que una compacta que nadie pueda mantener dentro de seis meses.

