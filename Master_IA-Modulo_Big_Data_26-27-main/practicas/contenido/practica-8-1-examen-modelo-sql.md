# 🧗🏽‍♂️ Practica 8.1 - Examen modelo - SQL

# Grupo Cierzo

**Sección 4 · SQL** · Subconsultas, CTE, vistas, `SELECT INTO` y tablas temporales · SQL Server 2025 + SSMS 22

## Contexto

Es viernes 08 de octubre de 2026. Trabajas como **Data Engineer** en el equipo de Datos de **Grupo Cierzo**, un distribuidor de material eléctrico y herramienta industrial con actividad en España, Portugal y México. Además de vender producto, el grupo presta servicios de mantenimiento y ejecuta proyectos llave en mano.

El T3 (tercer trimestre) se cerró el 30 de septiembre y esta semana coinciden el cierre trimestral, el seguimiento presupuestario y la revisión de riesgos. Tomás Egea, Head of Data, te ha reenviado las peticiones que han llegado al equipo. Todas se resuelven en T-SQL sobre la base de datos `CierzoDB`.

Esta práctica sirve de referencia para el examen: 15 preguntas sobre los mismos temas y con el mismo tipo de enunciado.

## Preparación

Antes de empezar, crea la base de datos con el script del **Anexo** que está al final de este documento: cópialo completo en una ventana de consulta de SSMS y ejecútalo. El script crea `CierzoDB` desde cero, así que puedes relanzarlo siempre que quieras volver al estado inicial. Al terminar, su consulta de verificación debe devolver:

| tabla | filas |
| --- | --- |
| periodos | 4 |
| unidades | 17 |
| cuentas | 20 |
| empleados | 20 |
| clientes | 14 |
| facturas | 68 |
| cobros | 48 |
| gastos | 350 |
| presupuestos | 196 |
| stg_cobros | 10 |

## Modelo de datos

| Tabla | Contenido | Columnas que conviene conocer |
| --- | --- | --- |
| `periodos` | Trimestres de 2026 | `fecha_inicio`, `fecha_fin`, `estado` (`Cerrado` o `Abierto`) |
| `unidades` | Estructura organizativa en 4 niveles: grupo, divisiones, departamentos y centros de coste | `unidad_padre_id` apunta a la propia tabla |
| `cuentas` | Plan de cuentas en 3 niveles. Solo las cuentas de último nivel reciben movimientos | `tipo` (`Ingreso` o `Gasto`), `cuenta_padre_id` apunta a la propia tabla |
| `empleados` | Plantilla del grupo | `puesto`, `unidad_id`, `jefe_id` apunta a la propia tabla |
| `clientes` | Clientes con su límite de crédito y su rating de riesgo (A es el mejor y D el peor) | `pais`, `segmento`, `limite_credito`, `rating`, `gestor_id` (empleado que lleva la cuenta; puede ser `NULL`) |
| `facturas` | Facturas emitidas: son los ingresos del grupo | `cliente_id`, `unidad_id`, `cuenta_id`, `fecha_emision`, `fecha_vencimiento`, `importe` |
| `cobros` | Cobros recibidos. Una factura puede tener varios cobros o ninguno | `factura_id`, `fecha_cobro`, `importe`, `metodo` |
| `gastos` | Gastos contabilizados | `fecha`, `unidad_id`, `cuenta_id`, `proveedor`, `importe` |
| `presupuestos` | Presupuesto trimestral por unidad y cuenta, tanto de ingresos como de gastos | `periodo_id`, `unidad_id`, `cuenta_id`, `importe` |
| `stg_cobros` | Lote de cobros recibido del banco, en staging, sin claves ni restricciones | Las mismas columnas que `cobros` |

## Reglas de negocio

- **Fecha de corte:** 30/09/2026, que es la fecha de fin del último periodo cerrado. Escríbela como literal.
- **T3 de 2026:** del 01/07/2026 al 30/09/2026, ambos incluidos.
- **Importe cobrado** de una factura: suma de todos sus cobros.
- **Saldo pendiente** de una factura: importe menos importe cobrado.
- **Factura vencida:** tiene saldo pendiente y su fecha de vencimiento es anterior a la fecha de corte.
- **Exposición** de un cliente: suma de los saldos pendientes de sus facturas.
- **Movimientos reales:** los ingresos son las facturas (se fechan por `fecha_emision`) y los gastos son las filas de `gastos` (se fechan por `fecha`).
- **Trimestre de un movimiento:** se obtiene de `periodos`, uniendo el movimiento con el periodo cuya `fecha_inicio` y `fecha_fin` contienen su fecha (`fecha BETWEEN fecha_inicio AND fecha_fin`).
## Instrucciones

- Entrega un único archivo `.sql`, con un comentario `- Pregunta N` delante de cada respuesta.
- Respeta los nombres de columna y el orden que pide cada enunciado. Tu resultado debe coincidir con la salida esperada.
- Las preguntas son independientes entre sí y **ninguna modifica las tablas del modelo**.
- Los objetos que crees (vistas y tablas) deben poder volver a crearse al relanzar tu script.
- En las preguntas 14 y 15, escribe la justificación como comentario encima del código.
---

## Sección 1 · Consultas con JOIN

### Pregunta 1

El equipo de Datos va a construir la tabla de hechos de ventas internacionales. Antes de automatizar la carga, Tomás te pide comprobar a mano que las facturas cruzan correctamente con sus tres dimensiones: cliente, unidad y cuenta.

Muestra las facturas emitidas en el **T3 de 2026** a clientes de **fuera de España** cuyo importe sea de **al menos 80.000 €**.

- Columnas: `factura_id`, `fecha_emision`, `razon_social`, `pais`, `unidad` (nombre de la unidad), `cuenta` (nombre de la cuenta), `importe`.
- Orden: de mayor a menor importe.
**Salida esperada:**

| factura_id | fecha_emision | razon_social | pais | unidad | cuenta | importe |
| --- | --- | --- | --- | --- | --- | --- |
| 56 | 2026-08-10 | Aceros Regiomontanos S.A. de C.V. | MX | Proyectos | Proyectos llave en mano | 128000.00 |
| 51 | 2026-07-16 | Aceros Regiomontanos S.A. de C.V. | MX | Ventas LatAm | Material eléctrico | 111800.00 |
| 64 | 2026-09-08 | Aceros Regiomontanos S.A. de C.V. | MX | Ventas LatAm | Herramienta industrial | 100400.00 |
| 52 | 2026-07-22 | Lusitana de Energia Lda. | PT | Ventas Iberia | Material eléctrico | 88200.00 |
| 57 | 2026-08-12 | Minera Sierra Alta S.A. de C.V. | MX | Ventas LatAm | Herramienta industrial | 88000.00 |

---

## Sección 2 · Subconsultas

### Pregunta 2

Antes de publicar la facturación en la capa de consumo, el pipeline debe señalar los valores atípicos para que alguien los revise.

Con una **subconsulta escalar**, lista las facturas cuyo importe supera **el doble del importe medio** de todas las facturas.

- Columnas: `factura_id`, `razon_social`, `fecha_emision`, `importe`.
- Orden: de mayor a menor importe.
**Salida esperada:**

| factura_id | razon_social | fecha_emision | importe |
| --- | --- | --- | --- |
| 42 | Naviera Atlántida S.A. | 2026-06-15 | 211200.00 |
| 10 | Naviera Atlántida S.A. | 2026-02-10 | 184000.00 |
| 59 | Naviera Atlántida S.A. | 2026-08-25 | 176000.00 |
| 26 | Construcciones Peñalara S.A. | 2026-04-15 | 170000.00 |
| 28 | Naviera Atlántida S.A. | 2026-04-21 | 156800.00 |
| 43 | Minera Sierra Alta S.A. de C.V. | 2026-06-18 | 144600.00 |
| 33 | Hormigones del Sur S.A. | 2026-05-12 | 142400.00 |
| 9 | Minera Sierra Alta S.A. de C.V. | 2026-02-05 | 134000.00 |
| 49 | Hormigones del Sur S.A. | 2026-07-08 | 133600.00 |

### Pregunta 3

Riesgos sospecha que los cobros de la cartera que gestionan los empleados con puesto `Comercial` no están bien conciliados y pide una cifra de control antes de lanzar el proceso de conciliación.

**Sin usar ****`JOIN`**, con subconsultas anidadas a **tres niveles** (cobros → facturas → clientes → empleados), obtén por método de cobro el número de cobros y el importe total cobrado en facturas de clientes cuyo gestor tiene el puesto `Comercial`.

- Columnas: `metodo`, `cobros`, `importe_cobrado`.
- Orden: de mayor a menor importe cobrado.
**Salida esperada:**

| metodo | cobros | importe_cobrado |
| --- | --- | --- |
| Transferencia | 3 | 228000.00 |
| Domiciliación | 7 | 81000.00 |
| Pagaré | 2 | 66000.00 |

---

## Sección 3 · CTE

### Pregunta 4

El cálculo «importe cobrado por factura» se va a reutilizar en varios procesos del equipo, así que conviene aislarlo como un paso con nombre.

Define una **CTE** que obtenga el importe cobrado de cada factura y úsala para listar las facturas con **cobro parcial**: han recibido algún cobro, pero no están totalmente cobradas.

- Columnas: `factura_id`, `razon_social`, `importe`, `cobrado`, `pendiente`.
- Orden: de mayor a menor saldo pendiente y, en caso de empate, por `factura_id`.
**Salida esperada:**

| factura_id | razon_social | importe | cobrado | pendiente |
| --- | --- | --- | --- | --- |
| 33 | Hormigones del Sur S.A. | 142400.00 | 40000.00 | 102400.00 |
| 42 | Naviera Atlántida S.A. | 211200.00 | 120000.00 | 91200.00 |
| 11 | Construcciones Peñalara S.A. | 108000.00 | 28000.00 | 80000.00 |
| 27 | Minera Sierra Alta S.A. de C.V. | 119000.00 | 50000.00 | 69000.00 |
| 16 | Hormigones del Sur S.A. | 117000.00 | 60000.00 | 57000.00 |
| 51 | Aceros Regiomontanos S.A. de C.V. | 111800.00 | 60000.00 | 51800.00 |
| 48 | Logística Moncayo S.L. | 63000.00 | 30000.00 | 33000.00 |
| 29 | Ferragens do Douro Lda. | 49000.00 | 24000.00 | 25000.00 |
| 41 | Talleres Ebro S.L. | 25200.00 | 12000.00 | 13200.00 |

### Pregunta 5

El equipo comercial quiere saber en qué países y segmentos de cliente se concentró la facturación del T3.

Resuélvelo como un pipeline de **CTE encadenadas**, donde cada una se apoya en la anterior:

1. `facturas_t3`: las facturas emitidas en el T3 de 2026.
2. `enriquecidas`: las facturas anteriores con el país y el segmento de su cliente.
3. `por_segmento`: número de facturas e importe por país y segmento.
La consulta final muestra las combinaciones de país y segmento que facturaron **200.000 € o más**.

- Columnas: `pais`, `segmento`, `facturas`, `importe`.
- Orden: de mayor a menor importe.
**Salida esperada:**

| pais | segmento | facturas | importe |
| --- | --- | --- | --- |
| ES | Gran cuenta | 8 | 528000.00 |
| MX | Gran cuenta | 4 | 428200.00 |
| PT | Distribuidor | 3 | 214600.00 |

### Pregunta 6

Para la dimensión organizativa del modelo analítico hay que aplanar la jerarquía que guarda `dbo.unidades`, de modo que cada unidad lleve su profundidad y su ruta completa.

Con una **CTE recursiva** que parta de la unidad raíz, devuelve todas las unidades con su nivel (la raíz es el nivel 0) y su ruta, separando los nombres con `>`.

- Columnas: `unidad_id`, `nombre`, `tipo`, `nivel`, `ruta`.
- Orden: por `ruta`.
**Salida esperada:**

| unidad_id | nombre | tipo | nivel | ruta |
| --- | --- | --- | --- | --- |
| 1 | Grupo Cierzo | Grupo | 0 | Grupo Cierzo |
| 4 | Corporativo | División | 1 | Grupo Cierzo > Corporativo |
| 9 | Finanzas | Departamento | 2 | Grupo Cierzo > Corporativo > Finanzas |
| 16 | Control de Gestión | Centro de coste | 3 | Grupo Cierzo > Corporativo > Finanzas > Control de Gestión |
| 15 | Riesgos | Centro de coste | 3 | Grupo Cierzo > Corporativo > Finanzas > Riesgos |
| 10 | Tecnología | Departamento | 2 | Grupo Cierzo > Corporativo > Tecnología |
| 17 | Datos | Centro de coste | 3 | Grupo Cierzo > Corporativo > Tecnología > Datos |
| 2 | División Industrial | División | 1 | Grupo Cierzo > División Industrial |
| 5 | Comercial Industrial | Departamento | 2 | Grupo Cierzo > División Industrial > Comercial Industrial |
| 11 | Ventas Iberia | Centro de coste | 3 | Grupo Cierzo > División Industrial > Comercial Industrial > Ventas Iberia |
| 12 | Ventas LatAm | Centro de coste | 3 | Grupo Cierzo > División Industrial > Comercial Industrial > Ventas LatAm |
| 6 | Operaciones y Logística | Departamento | 2 | Grupo Cierzo > División Industrial > Operaciones y Logística |
| 14 | Almacén Monterrey | Centro de coste | 3 | Grupo Cierzo > División Industrial > Operaciones y Logística > Almacén Monterrey |
| 13 | Almacén Zaragoza | Centro de coste | 3 | Grupo Cierzo > División Industrial > Operaciones y Logística > Almacén Zaragoza |
| 3 | División Servicios | División | 1 | Grupo Cierzo > División Servicios |
| 7 | Mantenimiento | Departamento | 2 | Grupo Cierzo > División Servicios > Mantenimiento |
| 8 | Proyectos | Departamento | 2 | Grupo Cierzo > División Servicios > Proyectos |

---

## Sección 4 · Vistas

### Pregunta 7

Hugo Barrios, **Financial Analyst**, monta cada trimestre la cuenta de resultados del grupo en su herramienta de BI. No debe consultar las tablas base ni conocer cómo se cruzan.

Crea la vista `dbo.vw_cuenta_resultados` con una fila por ejercicio, trimestre y cuenta.

- Columnas: `ejercicio`, `trimestre`, `tipo`, `cuenta` (nombre de la cuenta), `importe`.
- La vista reúne los dos tipos de movimiento: las facturas (por `fecha_emision`) y los gastos (por `fecha`).
**Comprobación:** importe total por trimestre y tipo, ordenado por `trimestre` y `tipo`.

**Salida esperada de la comprobación:**

| trimestre | tipo | importe |
| --- | --- | --- |
| 1 | Gasto | 1426490.00 |
| 1 | Ingreso | 1483000.00 |
| 2 | Gasto | 1530590.00 |
| 2 | Ingreso | 1608400.00 |
| 3 | Gasto | 1330600.00 |
| 3 | Ingreso | 1306500.00 |

### Pregunta 8

Clara Mendiola, **Budget Analyst**, hace el seguimiento del presupuesto de gastos y necesita comparar cada línea presupuestada con lo realmente gastado.

Crea la vista `dbo.vw_presupuesto_gastos` con una fila por cada línea de `dbo.presupuestos` cuya cuenta sea de tipo `Gasto`.

- Columnas: `ejercicio`, `trimestre`, `unidad`, `cuenta`, `presupuesto`, `gasto_real`, `desviacion`.
- `gasto_real`: suma de los gastos del mismo trimestre, unidad y cuenta. Si una línea no tiene gastos, vale 0.
- `desviacion` = gasto real − presupuesto.
- Sugerencia: calcula primero, en una CTE, el gasto real por periodo, unidad y cuenta, y únelo después con el presupuesto.
**Comprobación:** líneas del T3 de 2026 cuyo gasto real supera al presupuesto en más de 5.000 €, de mayor a menor desviación.

**Salida esperada de la comprobación:**

| unidad | cuenta | presupuesto | gasto_real | desviacion |
| --- | --- | --- | --- | --- |
| Proyectos | Servicios profesionales | 60000.00 | 89000.00 | 29000.00 |
| Datos | Sueldos y salarios | 18900.00 | 30900.00 | 12000.00 |
| Datos | Licencias de software | 10500.00 | 20400.00 | 9900.00 |
| Ventas LatAm | Sueldos y salarios | 14700.00 | 23400.00 | 8700.00 |
| Almacén Monterrey | Compra de mercaderías | 165500.00 | 174000.00 | 8500.00 |
| Ventas Iberia | Publicidad y marketing | 12000.00 | 18200.00 | 6200.00 |

### Pregunta 9

Iván Calvo, **Risk Analyst**, revisa cada semana la exposición de crédito de los clientes y necesita una única fuente fiable para su informe.

Crea la vista `dbo.vw_riesgo_cliente` con una fila por cliente, **incluidos los que no tienen facturas**.

- Columnas: `cliente_id`, `razon_social`, `rating`, `limite_credito`, `exposicion`, `saldo_vencido`.
- `exposicion`: suma de los saldos pendientes del cliente (0 si no tiene facturas).
- `saldo_vencido`: parte de la exposición que corresponde a facturas con vencimiento anterior a la fecha de corte (0 si no tiene).
- Sugerencia: apóyate en dos CTE, una con el importe cobrado por factura y otra con el saldo pendiente de cada factura.
**Comprobación:** clientes cuya exposición supera su límite de crédito, de mayor a menor exposición.

**Salida esperada de la comprobación:**

| razon_social | rating | limite_credito | exposicion | saldo_vencido |
| --- | --- | --- | --- | --- |
| Construcciones Peñalara S.A. | D | 400000.00 | 468600.00 | 468600.00 |
| Envases Tapatíos S.A. de C.V. | D | 100000.00 | 113800.00 | 113800.00 |

---

## Sección 5 · SELECT INTO

> En SQL Server, el patrón CTAS (`CREATE TABLE ... AS SELECT`) se escribe con `SELECT ... INTO`.

### Pregunta 10

El equipo de BI va a probar un cuadro de mando nuevo y necesita una copia estática de la facturación del T3 para no trabajar sobre `dbo.facturas`.

Con una instrucción **`SELECT ... INTO`**, crea la tabla `dbo.facturas_2026_t3` con las facturas emitidas en el T3 de 2026.

- Columnas: `factura_id`, `cliente_id`, `fecha_emision`, `fecha_vencimiento`, `importe`.
- El script debe poder ejecutarse varias veces sin dar error.
**Comprobación:** número de facturas, importe total y fechas de la primera y la última emisión de la tabla creada.

**Salida esperada de la comprobación:**

| facturas | importe_total | primera_emision | ultima_emision |
| --- | --- | --- | --- |
| 21 | 1306500.00 | 2026-07-06 | 2026-09-30 |

### Pregunta 11

La capa de consumo necesita una tabla de facturas con el cliente ya resuelto y el estado de cobro calculado, para que BI no tenga que cruzar tablas en cada informe.

Con **`SELECT ... INTO`**, crea la tabla `dbo.facturas_enriquecidas` con una fila por factura.

- Columnas: `factura_id`, `razon_social`, `pais`, `fecha_emision`, `importe`, `cobrado`, `pendiente`, `estado_cobro`.
- `estado_cobro` vale `Sin cobro` si la factura no ha recibido ningún cobro, `Cobro parcial` si ha recibido alguno pero queda saldo pendiente y `Cobrada` si no queda saldo.
- El script debe poder ejecutarse varias veces sin dar error.
**Comprobación:** por cada `estado_cobro`, número de facturas, importe facturado y saldo pendiente, de mayor a menor número de facturas.

**Salida esperada de la comprobación:**

| estado_cobro | facturas | importe | pendiente |
| --- | --- | --- | --- |
| Cobrada | 38 | 1784900.00 | 0.00 |
| Sin cobro | 21 | 1666400.00 | 1666400.00 |
| Cobro parcial | 9 | 946600.00 | 522600.00 |

---

## Sección 6 · Tablas temporales

### Pregunta 12

Tesorería pide dos cifras sobre los cobros del T3. Las dos salen del mismo cruce de tablas y no quieres calcularlo dos veces ni dejar ningún objeto nuevo en la base de datos.

Crea la **tabla temporal** `#cobros_t3` con los cobros cuya fecha de cobro cae en el T3 de 2026.

- Columnas: `cobro_id`, `razon_social`, `fecha_cobro`, `importe`, `metodo`.
A partir de la tabla temporal, obtén:

- **Consulta A:** número de cobros e importe cobrado por método, de mayor a menor importe.
- **Consulta B:** los tres clientes con mayor importe cobrado en el trimestre.
**Salida esperada de la consulta A:**

| metodo | cobros | importe_cobrado |
| --- | --- | --- |
| Transferencia | 10 | 483400.00 |
| Confirming | 2 | 276800.00 |
| Domiciliación | 5 | 85900.00 |
| Pagaré | 2 | 64000.00 |

**Salida esperada de la consulta B:**

| razon_social | importe_cobrado |
| --- | --- |
| Naviera Atlántida S.A. | 301800.00 |
| Lusitana de Energia Lda. | 160600.00 |
| Aceros Regiomontanos S.A. de C.V. | 153400.00 |

### Pregunta 13

El banco ha enviado el lote de cobros de principios de octubre, que está en `dbo.stg_cobros` tal como ha llegado. Antes de cargarlo hay que depurarlo. En esta pregunta **no se carga nada**: `dbo.cobros` no debe modificarse.

Resuélvelo con **tablas temporales**, en tres pasos:

4. `#lote_unico`: el lote sin filas repetidas.
5. `#lote_valido`: las filas de `#lote_unico` que cumplen las tres condiciones siguientes:
- Su `cobro_id` todavía no existe en `dbo.cobros`.
- Su factura existe en `dbo.facturas`.
- Su importe es mayor que 0.
6. A partir de las tablas temporales:
- **Consulta A:** número de filas de `dbo.stg_cobros`, de `#lote_unico` y de `#lote_valido`, con las columnas `etapa` y `filas`, de mayor a menor número de filas.
- **Consulta B:** el contenido de `#lote_valido`, ordenado por `cobro_id`.
**Salida esperada de la consulta A:**

| etapa | filas |
| --- | --- |
| stg_cobros | 10 |
| #lote_unico | 9 |
| #lote_valido | 5 |

**Salida esperada de la consulta B:**

| cobro_id | factura_id | fecha_cobro | importe | metodo |
| --- | --- | --- | --- | --- |
| 49 | 41 | 2026-10-01 | 13200.00 | Transferencia |
| 50 | 42 | 2026-10-01 | 91200.00 | Confirming |
| 51 | 48 | 2026-10-02 | 16000.00 | Transferencia |
| 54 | 16 | 2026-10-02 | 57000.00 | Pagaré |
| 55 | 63 | 2026-10-02 | 15400.00 | Domiciliación |

---

## Sección 7 · Decisiones de diseño

### Pregunta 14

El equipo de Datos va a restringir el acceso a la información financiera: solo podrán consultarla las personas que dependen, **directa o indirectamente**, del CFO (Álvaro Montes). La consulta que obtiene esa lista se ejecutará cada noche y debe seguir funcionando **sin modificarla** aunque la organización añada o elimine niveles de mando.

Tienes dos técnicas para recorrer la jerarquía de `dbo.empleados`: **subconsultas** o **CTE**. Solo una de las dos cumple el requisito.

7. Indica cuál eliges y explica en dos o tres líneas por qué la otra no es válida.
8. Escribe la consulta que devuelve los empleados que dependen del CFO, sin incluirle a él.
- Columnas: `nombre`, `puesto`, `nivel` (quienes dependen directamente del CFO son el nivel 1).
- Orden: por `nivel` y `nombre`.
**Salida esperada:**

| nombre | puesto | nivel |
| --- | --- | --- |
| Nerea Azcona | Controller | 1 |
| Patricia Soler | Head of Risk | 1 |
| Clara Mendiola | Budget Analyst | 2 |
| Hugo Barrios | Financial Analyst | 2 |
| Iván Calvo | Risk Analyst | 2 |

### Pregunta 15

El lunes se cargará en `dbo.cobros` el lote de cobros de octubre. Antes, Tesorería quiere conservar la **foto de cobros a cierre del T3**: el importe cobrado a cada cliente hasta el 30/09/2026. Los requisitos son tres:

9. La foto **no debe cambiar** cuando se carguen cobros nuevos.
10. Debe **seguir existiendo en enero**, para compararla con el cierre del T4, y cualquier analista debe poder consultarla desde su propia sesión.
11. Debe crearse y cargarse en **una sola instrucción**.
Tienes tres opciones: una **vista**, una tabla creada con **`SELECT ... INTO`** o una **tabla temporal**. Solo una cumple los tres requisitos.

12. Indica cuál eliges y explica por qué las otras dos no son válidas.
13. Impleméntala con el nombre `cierre_cobros_2026_t3`.
- Columnas: `cliente_id`, `razon_social`, `importe_cobrado`, `fecha_foto` (30/09/2026).
**Comprobación:** contenido del objeto creado, de mayor a menor importe cobrado.

**Salida esperada de la comprobación:**

| cliente_id | razon_social | importe_cobrado | fecha_foto |
| --- | --- | --- | --- |
| 4 | Naviera Atlántida S.A. | 535800.00 | 2026-09-30 |
| 9 | Aceros Regiomontanos S.A. de C.V. | 354000.00 | 2026-09-30 |
| 7 | Lusitana de Energia Lda. | 321600.00 | 2026-09-30 |
| 2 | Hormigones del Sur S.A. | 264000.00 | 2026-09-30 |
| 10 | Minera Sierra Alta S.A. de C.V. | 184000.00 | 2026-09-30 |
| 12 | Hospital San Lorenzo | 156800.00 | 2026-09-30 |
| 13 | Logística Moncayo S.L. | 102800.00 | 2026-09-30 |
| 1 | Talleres Ebro S.L. | 70900.00 | 2026-09-30 |
| 5 | Bodegas Valdeolmos S.L. | 69500.00 | 2026-09-30 |
| 8 | Ferragens do Douro Lda. | 66000.00 | 2026-09-30 |
| 3 | Electro Montajes Rioja S.L. | 55500.00 | 2026-09-30 |
| 6 | Construcciones Peñalara S.A. | 28000.00 | 2026-09-30 |

---

## Anexo · Script de creación de CierzoDB

Copia el script completo y ejecútalo en SSMS. Requiere SQL Server 2025, porque fija el nivel de compatibilidad 170.

```sql
/* =====================================================================
   CierzoDB · Script de creación y carga
   Práctica tipo examen · Sección 4 · SQL
   Motor: SQL Server 2025 · Herramienta: SSMS 22

   Caso: Grupo Cierzo, distribuidor de material eléctrico y herramienta
   industrial con actividad en España, Portugal y México. El grupo vende
   producto, presta servicios de mantenimiento y ejecuta proyectos.

   La base de datos recoge los nueve primeros meses de 2026:
     - Estructura organizativa, plan de cuentas y plantilla (jerarquías)
     - Facturas emitidas a clientes y cobros recibidos
     - Gastos contabilizados y presupuesto trimestral
     - Un lote de cobros en staging pendiente de cargar

   Fecha de corte de todos los análisis: 30/09/2026.

   El script se puede ejecutar tantas veces como haga falta: elimina la
   base de datos si existe y la vuelve a crear desde cero.
   ===================================================================== */

/* =====================================================================
   1. CREACIÓN DE LA BASE DE DATOS
   ===================================================================== */

USE master;
GO

IF DB_ID(N'CierzoDB') IS NOT NULL
BEGIN
    ALTER DATABASE CierzoDB SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE CierzoDB;
END
GO

CREATE DATABASE CierzoDB;
GO

ALTER DATABASE CierzoDB SET COMPATIBILITY_LEVEL = 170;
GO

USE CierzoDB;
GO


/* =====================================================================
   2. CREACIÓN DE TABLAS
   ===================================================================== */

-- Periodos contables (trimestres)
CREATE TABLE dbo.periodos (
    periodo_id    int          NOT NULL CONSTRAINT PK_periodos PRIMARY KEY,
    ejercicio     int          NOT NULL,
    trimestre     int          NOT NULL CONSTRAINT CK_periodos_trimestre CHECK (trimestre BETWEEN 1 AND 4),
    fecha_inicio  date         NOT NULL,
    fecha_fin     date         NOT NULL,
    estado        nvarchar(10) NOT NULL CONSTRAINT CK_periodos_estado CHECK (estado IN (N'Abierto', N'Cerrado')),
    CONSTRAINT UQ_periodos UNIQUE (ejercicio, trimestre)
);

-- Estructura organizativa (jerarquía: unidad_padre_id apunta a la misma tabla)
CREATE TABLE dbo.unidades (
    unidad_id        int          NOT NULL CONSTRAINT PK_unidades PRIMARY KEY,
    nombre           nvarchar(60) NOT NULL,
    tipo             nvarchar(20) NOT NULL CONSTRAINT CK_unidades_tipo
                         CHECK (tipo IN (N'Grupo', N'División', N'Departamento', N'Centro de coste')),
    unidad_padre_id  int          NULL CONSTRAINT FK_unidades_padre REFERENCES dbo.unidades (unidad_id),
    pais             char(2)      NOT NULL
);

-- Plan de cuentas (jerarquía: cuenta_padre_id apunta a la misma tabla)
CREATE TABLE dbo.cuentas (
    cuenta_id        int          NOT NULL CONSTRAINT PK_cuentas PRIMARY KEY,
    nombre           nvarchar(60) NOT NULL,
    tipo             nvarchar(10) NOT NULL CONSTRAINT CK_cuentas_tipo CHECK (tipo IN (N'Ingreso', N'Gasto')),
    cuenta_padre_id  int          NULL CONSTRAINT FK_cuentas_padre REFERENCES dbo.cuentas (cuenta_id)
);

-- Plantilla (jerarquía: jefe_id apunta a la misma tabla)
CREATE TABLE dbo.empleados (
    empleado_id    int           NOT NULL CONSTRAINT PK_empleados PRIMARY KEY,
    nombre         nvarchar(60)  NOT NULL,
    puesto         nvarchar(60)  NOT NULL,
    unidad_id      int           NOT NULL CONSTRAINT FK_empleados_unidad REFERENCES dbo.unidades (unidad_id),
    jefe_id        int           NULL CONSTRAINT FK_empleados_jefe REFERENCES dbo.empleados (empleado_id),
    salario_anual  decimal(12,2) NOT NULL,
    fecha_alta     date          NOT NULL
);

-- Clientes con su límite de crédito y su rating interno de riesgo (A = mejor, D = peor)
CREATE TABLE dbo.clientes (
    cliente_id      int           NOT NULL CONSTRAINT PK_clientes PRIMARY KEY,
    razon_social    nvarchar(80)  NOT NULL,
    sector          nvarchar(40)  NOT NULL,
    pais            char(2)       NOT NULL,
    segmento        nvarchar(20)  NOT NULL CONSTRAINT CK_clientes_segmento
                        CHECK (segmento IN (N'Gran cuenta', N'Pyme', N'Distribuidor')),
    limite_credito  decimal(12,2) NOT NULL,
    rating          char(1)       NOT NULL CONSTRAINT CK_clientes_rating CHECK (rating IN ('A', 'B', 'C', 'D')),
    fecha_alta      date          NOT NULL,
    gestor_id       int           NULL CONSTRAINT FK_clientes_gestor REFERENCES dbo.empleados (empleado_id)
);

-- Facturas emitidas (ingresos). Cada factura se imputa a una unidad y a una cuenta de ingreso
CREATE TABLE dbo.facturas (
    factura_id         int           NOT NULL CONSTRAINT PK_facturas PRIMARY KEY,
    cliente_id         int           NOT NULL CONSTRAINT FK_facturas_cliente REFERENCES dbo.clientes (cliente_id),
    unidad_id          int           NOT NULL CONSTRAINT FK_facturas_unidad REFERENCES dbo.unidades (unidad_id),
    cuenta_id          int           NOT NULL CONSTRAINT FK_facturas_cuenta REFERENCES dbo.cuentas (cuenta_id),
    fecha_emision      date          NOT NULL,
    fecha_vencimiento  date          NOT NULL,
    importe            decimal(12,2) NOT NULL CONSTRAINT CK_facturas_importe CHECK (importe > 0),
    CONSTRAINT CK_facturas_fechas CHECK (fecha_vencimiento >= fecha_emision)
);

-- Cobros recibidos de clientes
CREATE TABLE dbo.cobros (
    cobro_id     int           NOT NULL CONSTRAINT PK_cobros PRIMARY KEY,
    factura_id   int           NOT NULL CONSTRAINT FK_cobros_factura REFERENCES dbo.facturas (factura_id),
    fecha_cobro  date          NOT NULL,
    importe      decimal(12,2) NOT NULL CONSTRAINT CK_cobros_importe CHECK (importe > 0),
    metodo       nvarchar(20)  NOT NULL CONSTRAINT CK_cobros_metodo
                     CHECK (metodo IN (N'Transferencia', N'Domiciliación', N'Confirming', N'Pagaré'))
);

-- Gastos contabilizados. Cada gasto se imputa a una unidad y a una cuenta de gasto
CREATE TABLE dbo.gastos (
    gasto_id   int           NOT NULL CONSTRAINT PK_gastos PRIMARY KEY,
    fecha      date          NOT NULL,
    unidad_id  int           NOT NULL CONSTRAINT FK_gastos_unidad REFERENCES dbo.unidades (unidad_id),
    cuenta_id  int           NOT NULL CONSTRAINT FK_gastos_cuenta REFERENCES dbo.cuentas (cuenta_id),
    proveedor  nvarchar(80)  NULL,
    importe    decimal(12,2) NOT NULL CONSTRAINT CK_gastos_importe CHECK (importe > 0)
);

-- Presupuesto trimestral por unidad y cuenta (tanto de ingresos como de gastos)
CREATE TABLE dbo.presupuestos (
    presupuesto_id  int           NOT NULL CONSTRAINT PK_presupuestos PRIMARY KEY,
    periodo_id      int           NOT NULL CONSTRAINT FK_presupuestos_periodo REFERENCES dbo.periodos (periodo_id),
    unidad_id       int           NOT NULL CONSTRAINT FK_presupuestos_unidad REFERENCES dbo.unidades (unidad_id),
    cuenta_id       int           NOT NULL CONSTRAINT FK_presupuestos_cuenta REFERENCES dbo.cuentas (cuenta_id),
    importe         decimal(12,2) NOT NULL,
    CONSTRAINT UQ_presupuestos UNIQUE (periodo_id, unidad_id, cuenta_id)
);

-- Staging de cobros: recibe el lote del banco tal como llega, sin claves ni restricciones
CREATE TABLE dbo.stg_cobros (
    cobro_id     int           NULL,
    factura_id   int           NULL,
    fecha_cobro  date          NULL,
    importe      decimal(12,2) NULL,
    metodo       nvarchar(20)  NULL
);
GO


/* =====================================================================
   3. CARGA DE DATOS
   ===================================================================== */

-- Periodos contables: los tres primeros trimestres de 2026 están cerrados.
-- La fecha de corte para los análisis es el fin del último periodo cerrado (30/09/2026).

INSERT INTO dbo.periodos (periodo_id, ejercicio, trimestre, fecha_inicio, fecha_fin, estado) VALUES
(1, 2026, 1, '2026-01-01', '2026-03-31', N'Cerrado'),
(2, 2026, 2, '2026-04-01', '2026-06-30', N'Cerrado'),
(3, 2026, 3, '2026-07-01', '2026-09-30', N'Cerrado'),
(4, 2026, 4, '2026-10-01', '2026-12-31', N'Abierto');

-- Estructura organizativa: Grupo > División > Departamento > Centro de coste (4 niveles).
-- Los gastos se pueden imputar a una unidad de cualquier nivel, no solo a los centros de coste

INSERT INTO dbo.unidades (unidad_id, nombre, tipo, unidad_padre_id, pais) VALUES
(1, N'Grupo Cierzo', N'Grupo', NULL, 'ES'),
(2, N'División Industrial', N'División', 1, 'ES'),
(3, N'División Servicios', N'División', 1, 'ES'),
(4, N'Corporativo', N'División', 1, 'ES'),
(5, N'Comercial Industrial', N'Departamento', 2, 'ES'),
(6, N'Operaciones y Logística', N'Departamento', 2, 'ES'),
(7, N'Mantenimiento', N'Departamento', 3, 'ES'),
(8, N'Proyectos', N'Departamento', 3, 'ES'),
(9, N'Finanzas', N'Departamento', 4, 'ES'),
(10, N'Tecnología', N'Departamento', 4, 'ES'),
(11, N'Ventas Iberia', N'Centro de coste', 5, 'ES'),
(12, N'Ventas LatAm', N'Centro de coste', 5, 'MX'),
(13, N'Almacén Zaragoza', N'Centro de coste', 6, 'ES'),
(14, N'Almacén Monterrey', N'Centro de coste', 6, 'MX'),
(15, N'Riesgos', N'Centro de coste', 9, 'ES'),
(16, N'Control de Gestión', N'Centro de coste', 9, 'ES'),
(17, N'Datos', N'Centro de coste', 10, 'ES');

-- Plan de cuentas: dos raíces (Gastos e Ingresos) y 3 niveles. Solo las cuentas de último nivel reciben apuntes

INSERT INTO dbo.cuentas (cuenta_id, nombre, tipo, cuenta_padre_id) VALUES
(6, N'Gastos', N'Gasto', NULL),
(7, N'Ingresos', N'Ingreso', NULL),
(60, N'Aprovisionamientos', N'Gasto', 6),
(62, N'Servicios exteriores', N'Gasto', 6),
(64, N'Gastos de personal', N'Gasto', 6),
(70, N'Ventas de productos', N'Ingreso', 7),
(71, N'Prestación de servicios', N'Ingreso', 7),
(600, N'Compra de mercaderías', N'Gasto', 60),
(601, N'Transporte y logística', N'Gasto', 60),
(621, N'Alquileres', N'Gasto', 62),
(623, N'Servicios profesionales', N'Gasto', 62),
(627, N'Publicidad y marketing', N'Gasto', 62),
(629, N'Licencias de software', N'Gasto', 62),
(640, N'Sueldos y salarios', N'Gasto', 64),
(642, N'Seguridad Social a cargo de la empresa', N'Gasto', 64),
(649, N'Formación', N'Gasto', 64),
(700, N'Material eléctrico', N'Ingreso', 70),
(701, N'Herramienta industrial', N'Ingreso', 70),
(710, N'Contratos de mantenimiento', N'Ingreso', 71),
(711, N'Proyectos llave en mano', N'Ingreso', 71);

-- Plantilla: jefe_id define la línea jerárquica; unidad_id, el centro donde se imputa su nómina

INSERT INTO dbo.empleados (empleado_id, nombre, puesto, unidad_id, jefe_id, salario_anual, fecha_alta) VALUES
(1, N'Marta Ibarra', N'CEO', 4, NULL, 145200.00, '2015-03-01'),
(2, N'Sergio Lanza', N'Director División Industrial', 2, 1, 98400.00, '2016-09-12'),
(3, N'Beatriz Oyarzun', N'Directora División Servicios', 3, 1, 96000.00, '2018-01-15'),
(4, N'Álvaro Montes', N'CFO', 9, 1, 110400.00, '2017-05-02'),
(5, N'Inés Carrasco', N'CTO', 10, 1, 105600.00, '2019-10-07'),
(6, N'Rubén Tejero', N'Jefe de Ventas Iberia', 11, 2, 62400.00, '2018-04-03'),
(7, N'Daniela Rojas', N'Jefa de Ventas LatAm', 12, 2, 58800.00, '2020-02-17'),
(8, N'Óscar Pineda', N'Responsable de Almacén', 13, 2, 40800.00, '2019-06-24'),
(9, N'Lucía Ferrer', N'Responsable de Mantenimiento', 7, 3, 52800.00, '2020-09-01'),
(10, N'Gonzalo Rey', N'Jefe de Proyectos', 8, 3, 60000.00, '2021-03-08'),
(11, N'Patricia Soler', N'Head of Risk', 15, 4, 72000.00, '2019-11-04'),
(12, N'Iván Calvo', N'Risk Analyst', 15, 11, 44400.00, '2023-02-13'),
(13, N'Nerea Azcona', N'Controller', 16, 4, 69600.00, '2018-07-16'),
(14, N'Hugo Barrios', N'Financial Analyst', 16, 13, 46800.00, '2022-09-05'),
(15, N'Clara Mendiola', N'Budget Analyst', 16, 13, 45600.00, '2024-01-08'),
(16, N'Tomás Egea', N'Head of Data', 17, 5, 75600.00, '2021-06-01'),
(17, N'Aitana Vega', N'Data Engineer', 17, 16, 48000.00, '2026-04-01'),
(18, N'Mario Quintana', N'Key Account Manager', 11, 6, 39600.00, '2021-10-18'),
(19, N'Elisa Navarro', N'Comercial', 11, 6, 37200.00, '2023-05-22'),
(20, N'Fernando Luján', N'Comercial', 12, 7, 34800.00, '2026-07-01');

-- Clientes: el cliente 14 es un alta reciente, sin gestor asignado (gestor_id NULL) y sin facturas

INSERT INTO dbo.clientes (cliente_id, razon_social, sector, pais, segmento, limite_credito, rating, fecha_alta, gestor_id) VALUES
(1, N'Talleres Ebro S.L.', N'Automoción', 'ES', N'Pyme', 120000.00, 'B', '2021-03-15', 18),
(2, N'Hormigones del Sur S.A.', N'Construcción', 'ES', N'Gran cuenta', 500000.00, 'C', '2019-06-01', 18),
(3, N'Electro Montajes Rioja S.L.', N'Instalaciones', 'ES', N'Pyme', 80000.00, 'A', '2022-01-20', 19),
(4, N'Naviera Atlántida S.A.', N'Naval', 'ES', N'Gran cuenta', 600000.00, 'B', '2018-09-10', 6),
(5, N'Bodegas Valdeolmos S.L.', N'Alimentación', 'ES', N'Pyme', 60000.00, 'A', '2023-05-05', 19),
(6, N'Construcciones Peñalara S.A.', N'Construcción', 'ES', N'Gran cuenta', 400000.00, 'D', '2020-02-11', 18),
(7, N'Lusitana de Energia Lda.', N'Energía', 'PT', N'Distribuidor', 300000.00, 'B', '2021-11-30', 6),
(8, N'Ferragens do Douro Lda.', N'Distribución', 'PT', N'Distribuidor', 160000.00, 'C', '2022-07-19', 19),
(9, N'Aceros Regiomontanos S.A. de C.V.', N'Siderurgia', 'MX', N'Gran cuenta', 440000.00, 'B', '2020-10-01', 7),
(10, N'Minera Sierra Alta S.A. de C.V.', N'Minería', 'MX', N'Gran cuenta', 360000.00, 'C', '2021-04-22', 20),
(11, N'Envases Tapatíos S.A. de C.V.', N'Packaging', 'MX', N'Pyme', 100000.00, 'D', '2024-02-14', 20),
(12, N'Hospital San Lorenzo', N'Sanidad', 'ES', N'Gran cuenta', 240000.00, 'A', '2019-01-08', 9),
(13, N'Logística Moncayo S.L.', N'Logística', 'ES', N'Pyme', 90000.00, 'B', '2023-09-12', 10),
(14, N'Agroquímica Turia S.L.', N'Química', 'ES', N'Pyme', 70000.00, 'B', '2026-08-25', NULL);

-- Facturas emitidas entre el 01/01/2026 y el 30/09/2026 (ingresos del grupo)

INSERT INTO dbo.facturas (factura_id, cliente_id, unidad_id, cuenta_id, fecha_emision, fecha_vencimiento, importe) VALUES
(1, 1, 11, 701, '2026-01-14', '2026-02-13', 16800.00),
(2, 9, 12, 701, '2026-01-15', '2026-03-16', 96000.00),
(3, 2, 11, 700, '2026-01-20', '2026-03-21', 128000.00),
(4, 7, 11, 700, '2026-01-22', '2026-03-23', 76000.00),
(5, 3, 11, 700, '2026-01-27', '2026-02-26', 10600.00),
(6, 4, 7, 710, '2026-01-31', '2026-03-02', 25000.00),
(7, 12, 7, 710, '2026-01-31', '2026-03-02', 19600.00),
(8, 5, 11, 700, '2026-02-03', '2026-03-05', 8400.00),
(9, 10, 12, 701, '2026-02-05', '2026-05-06', 134000.00),
(10, 4, 11, 701, '2026-02-10', '2026-05-11', 184000.00),
(11, 6, 11, 700, '2026-02-12', '2026-04-13', 108000.00),
(12, 1, 11, 701, '2026-02-18', '2026-03-20', 22500.00),
(13, 8, 11, 701, '2026-02-24', '2026-04-25', 42000.00),
(14, 12, 7, 710, '2026-02-28', '2026-03-30', 19600.00),
(15, 11, 12, 700, '2026-03-03', '2026-04-02', 37000.00),
(16, 2, 11, 700, '2026-03-05', '2026-05-04', 117000.00),
(17, 13, 8, 711, '2026-03-09', '2026-05-08', 56000.00),
(18, 9, 12, 700, '2026-03-12', '2026-05-11', 104600.00),
(19, 3, 11, 700, '2026-03-16', '2026-04-15', 14300.00),
(20, 7, 11, 700, '2026-03-18', '2026-05-17', 85000.00),
(21, 6, 11, 701, '2026-03-26', '2026-05-25', 123000.00),
(22, 2, 7, 710, '2026-03-31', '2026-04-30', 36000.00),
(23, 12, 7, 710, '2026-03-31', '2026-04-30', 19600.00),
(24, 11, 12, 700, '2026-04-07', '2026-05-07', 44000.00),
(25, 1, 11, 700, '2026-04-09', '2026-05-09', 19600.00),
(26, 6, 8, 711, '2026-04-15', '2026-06-14', 170000.00),
(27, 10, 12, 701, '2026-04-16', '2026-07-15', 119000.00),
(28, 4, 11, 700, '2026-04-21', '2026-07-20', 156800.00),
(29, 8, 11, 701, '2026-04-28', '2026-06-27', 49000.00),
(30, 4, 7, 710, '2026-04-30', '2026-05-30', 25000.00),
(31, 12, 7, 710, '2026-04-30', '2026-05-30', 19600.00),
(32, 5, 11, 700, '2026-05-06', '2026-06-05', 7800.00),
(33, 2, 11, 701, '2026-05-12', '2026-07-11', 142400.00),
(34, 9, 12, 701, '2026-05-14', '2026-07-13', 93400.00),
(35, 11, 12, 700, '2026-05-19', '2026-06-18', 32800.00),
(36, 7, 11, 701, '2026-05-20', '2026-07-19', 72400.00),
(37, 3, 11, 700, '2026-05-22', '2026-06-21', 12800.00),
(38, 6, 11, 700, '2026-05-28', '2026-07-27', 95600.00),
(39, 12, 7, 710, '2026-05-31', '2026-06-30', 19600.00),
(40, 13, 7, 710, '2026-05-31', '2026-06-30', 8400.00),
(41, 1, 11, 701, '2026-06-03', '2026-08-02', 25200.00),
(42, 4, 11, 701, '2026-06-15', '2026-09-13', 211200.00),
(43, 10, 12, 701, '2026-06-18', '2026-09-16', 144600.00),
(44, 5, 8, 711, '2026-06-22', '2026-08-21', 44000.00),
(45, 2, 7, 710, '2026-06-30', '2026-07-30', 36000.00),
(46, 8, 11, 701, '2026-06-30', '2026-08-29', 39600.00),
(47, 12, 7, 710, '2026-06-30', '2026-07-30', 19600.00),
(48, 13, 8, 711, '2026-07-06', '2026-09-04', 63000.00),
(49, 2, 11, 700, '2026-07-08', '2026-09-06', 133600.00),
(50, 3, 11, 700, '2026-07-14', '2026-08-13', 17800.00),
(51, 9, 12, 700, '2026-07-16', '2026-09-14', 111800.00),
(52, 7, 11, 700, '2026-07-22', '2026-09-20', 88200.00),
(53, 4, 7, 710, '2026-07-31', '2026-08-30', 25000.00),
(54, 12, 7, 710, '2026-07-31', '2026-08-30', 19600.00),
(55, 5, 11, 700, '2026-08-04', '2026-09-03', 9300.00),
(56, 9, 8, 711, '2026-08-10', '2026-11-08', 128000.00),
(57, 10, 12, 701, '2026-08-12', '2026-11-10', 88000.00),
(58, 1, 11, 701, '2026-08-20', '2026-10-19', 21800.00),
(59, 4, 11, 700, '2026-08-25', '2026-11-23', 176000.00),
(60, 8, 11, 701, '2026-08-31', '2026-10-30', 46800.00),
(61, 12, 7, 710, '2026-08-31', '2026-09-30', 19600.00),
(62, 13, 7, 710, '2026-08-31', '2026-09-30', 8400.00),
(63, 3, 11, 700, '2026-09-03', '2026-10-03', 15400.00),
(64, 9, 12, 701, '2026-09-08', '2026-11-07', 100400.00),
(65, 2, 11, 700, '2026-09-10', '2026-11-09', 98600.00),
(66, 7, 11, 700, '2026-09-15', '2026-11-14', 79600.00),
(67, 2, 7, 710, '2026-09-30', '2026-10-30', 36000.00),
(68, 12, 7, 710, '2026-09-30', '2026-10-30', 19600.00);

-- Cobros recibidos hasta el 30/09/2026. Una factura puede tener varios cobros parciales o ninguno

INSERT INTO dbo.cobros (cobro_id, factura_id, fecha_cobro, importe, metodo) VALUES
(1, 1, '2026-02-12', 16800.00, N'Transferencia'),
(2, 5, '2026-02-26', 10600.00, N'Domiciliación'),
(3, 6, '2026-03-02', 25000.00, N'Transferencia'),
(4, 7, '2026-03-02', 19600.00, N'Domiciliación'),
(5, 8, '2026-03-04', 8400.00, N'Domiciliación'),
(6, 2, '2026-03-16', 96000.00, N'Transferencia'),
(7, 4, '2026-03-20', 76000.00, N'Transferencia'),
(8, 12, '2026-03-25', 22500.00, N'Transferencia'),
(9, 14, '2026-03-30', 19600.00, N'Domiciliación'),
(10, 3, '2026-04-10', 128000.00, N'Pagaré'),
(11, 19, '2026-04-15', 14300.00, N'Domiciliación'),
(12, 23, '2026-04-30', 19600.00, N'Domiciliación'),
(13, 22, '2026-05-05', 36000.00, N'Transferencia'),
(14, 17, '2026-05-08', 28000.00, N'Transferencia'),
(15, 25, '2026-05-08', 19600.00, N'Transferencia'),
(16, 10, '2026-05-11', 184000.00, N'Confirming'),
(17, 18, '2026-05-11', 104600.00, N'Transferencia'),
(18, 20, '2026-05-15', 85000.00, N'Transferencia'),
(19, 16, '2026-05-20', 60000.00, N'Transferencia'),
(20, 30, '2026-05-29', 25000.00, N'Transferencia'),
(21, 13, '2026-05-30', 42000.00, N'Pagaré'),
(22, 31, '2026-05-30', 19600.00, N'Domiciliación'),
(23, 32, '2026-06-05', 7800.00, N'Domiciliación'),
(24, 17, '2026-06-08', 28000.00, N'Transferencia'),
(25, 9, '2026-06-20', 134000.00, N'Transferencia'),
(26, 37, '2026-06-21', 12800.00, N'Domiciliación'),
(27, 40, '2026-06-29', 8400.00, N'Transferencia'),
(28, 11, '2026-06-30', 28000.00, N'Pagaré'),
(29, 39, '2026-06-30', 19600.00, N'Domiciliación'),
(30, 36, '2026-07-17', 72400.00, N'Transferencia'),
(31, 28, '2026-07-20', 156800.00, N'Confirming'),
(32, 34, '2026-07-20', 93400.00, N'Transferencia'),
(33, 47, '2026-07-30', 19600.00, N'Domiciliación'),
(34, 29, '2026-07-31', 24000.00, N'Pagaré'),
(35, 33, '2026-08-01', 40000.00, N'Pagaré'),
(36, 41, '2026-08-10', 12000.00, N'Transferencia'),
(37, 50, '2026-08-13', 17800.00, N'Domiciliación'),
(38, 44, '2026-08-20', 44000.00, N'Transferencia'),
(39, 27, '2026-08-28', 50000.00, N'Transferencia'),
(40, 54, '2026-08-30', 19600.00, N'Domiciliación'),
(41, 53, '2026-09-02', 25000.00, N'Transferencia'),
(42, 55, '2026-09-03', 9300.00, N'Domiciliación'),
(43, 48, '2026-09-04', 30000.00, N'Transferencia'),
(44, 42, '2026-09-13', 120000.00, N'Confirming'),
(45, 51, '2026-09-14', 60000.00, N'Transferencia'),
(46, 52, '2026-09-25', 88200.00, N'Transferencia'),
(47, 62, '2026-09-29', 8400.00, N'Transferencia'),
(48, 61, '2026-09-30', 19600.00, N'Domiciliación');

-- Gastos contabilizados entre el 01/01/2026 y el 30/09/2026. Las nóminas (cuenta 640) no llevan proveedor

INSERT INTO dbo.gastos (gasto_id, fecha, unidad_id, cuenta_id, proveedor, importe) VALUES
-- Enero de 2026
(1, '2026-01-01', 4, 621, N'Oficinas Torre Aragonia', 9000.00),
(2, '2026-01-01', 13, 621, N'Inmobiliaria Ribera', 6500.00),
(3, '2026-01-01', 14, 621, N'Parque Industrial Apodaca', 4200.00),
(4, '2026-01-03', 10, 629, N'ERP Nimbus', 7400.00),
(5, '2026-01-05', 14, 623, N'Operador Logístico Regio', 5500.00),
(6, '2026-01-12', 13, 600, N'Cables y Conductores Aragón', 125000.00),
(7, '2026-01-14', 14, 600, N'Proveedora Industrial del Norte', 55700.00),
(8, '2026-01-20', 7, 623, N'Servicios Técnicos Aragón', 23200.00),
(9, '2026-01-25', 13, 601, N'Transportes Jalón', 9300.00),
(10, '2026-01-26', 14, 601, N'Fletes Sierra Madre', 4800.00),
(11, '2026-01-28', 17, 629, N'DataCloud Services', 3100.00),
(12, '2026-01-31', 2, 640, NULL, 8200.00),
(13, '2026-01-31', 2, 642, N'Seguridad Social', 2460.00),
(14, '2026-01-31', 3, 640, NULL, 8000.00),
(15, '2026-01-31', 3, 642, N'Seguridad Social', 2400.00),
(16, '2026-01-31', 4, 640, NULL, 12100.00),
(17, '2026-01-31', 4, 642, N'Seguridad Social', 3630.00),
(18, '2026-01-31', 7, 640, NULL, 4400.00),
(19, '2026-01-31', 7, 642, N'Seguridad Social', 1320.00),
(20, '2026-01-31', 8, 640, NULL, 5000.00),
(21, '2026-01-31', 8, 642, N'Seguridad Social', 1500.00),
(22, '2026-01-31', 9, 640, NULL, 9200.00),
(23, '2026-01-31', 9, 642, N'Seguridad Social', 2760.00),
(24, '2026-01-31', 10, 640, NULL, 8800.00),
(25, '2026-01-31', 10, 642, N'Seguridad Social', 2640.00),
(26, '2026-01-31', 11, 640, NULL, 11600.00),
(27, '2026-01-31', 11, 642, N'Seguridad Social', 3480.00),
(28, '2026-01-31', 12, 640, NULL, 4900.00),
(29, '2026-01-31', 12, 642, N'IMSS', 1470.00),
(30, '2026-01-31', 13, 640, NULL, 3400.00),
(31, '2026-01-31', 13, 642, N'Seguridad Social', 1020.00),
(32, '2026-01-31', 15, 640, NULL, 9700.00),
(33, '2026-01-31', 15, 642, N'Seguridad Social', 2910.00),
(34, '2026-01-31', 16, 640, NULL, 13500.00),
(35, '2026-01-31', 16, 642, N'Seguridad Social', 4050.00),
(36, '2026-01-31', 17, 640, NULL, 6300.00),
(37, '2026-01-31', 17, 642, N'Seguridad Social', 1890.00),
-- Febrero de 2026
(38, '2026-02-01', 4, 621, N'Oficinas Torre Aragonia', 9000.00),
(39, '2026-02-01', 13, 621, N'Inmobiliaria Ribera', 6500.00),
(40, '2026-02-01', 14, 621, N'Parque Industrial Apodaca', 4200.00),
(41, '2026-02-03', 10, 629, N'ERP Nimbus', 7400.00),
(42, '2026-02-05', 14, 623, N'Operador Logístico Regio', 5500.00),
(43, '2026-02-12', 13, 600, N'Herramientas Bidasoa', 197000.00),
(44, '2026-02-14', 14, 600, N'Conductores Monterrey', 77700.00),
(45, '2026-02-16', 15, 623, N'Informes Mercantiles Ibéricos', 3200.00),
(46, '2026-02-20', 7, 623, N'Servicios Técnicos Aragón', 10200.00),
(47, '2026-02-25', 13, 601, N'Transportes Jalón', 14600.00),
(48, '2026-02-26', 14, 601, N'Fletes Sierra Madre', 6700.00),
(49, '2026-02-28', 2, 640, NULL, 8200.00),
(50, '2026-02-28', 2, 642, N'Seguridad Social', 2460.00),
(51, '2026-02-28', 3, 640, NULL, 8000.00),
(52, '2026-02-28', 3, 642, N'Seguridad Social', 2400.00),
(53, '2026-02-28', 4, 640, NULL, 12100.00),
(54, '2026-02-28', 4, 642, N'Seguridad Social', 3630.00),
(55, '2026-02-28', 7, 640, NULL, 4400.00),
(56, '2026-02-28', 7, 642, N'Seguridad Social', 1320.00),
(57, '2026-02-28', 8, 640, NULL, 5000.00),
(58, '2026-02-28', 8, 642, N'Seguridad Social', 1500.00),
(59, '2026-02-28', 9, 640, NULL, 9200.00),
(60, '2026-02-28', 9, 642, N'Seguridad Social', 2760.00),
(61, '2026-02-28', 10, 640, NULL, 8800.00),
(62, '2026-02-28', 10, 642, N'Seguridad Social', 2640.00),
(63, '2026-02-28', 11, 640, NULL, 11600.00),
(64, '2026-02-28', 11, 642, N'Seguridad Social', 3480.00),
(65, '2026-02-28', 12, 640, NULL, 4900.00),
(66, '2026-02-28', 12, 642, N'IMSS', 1470.00),
(67, '2026-02-28', 13, 640, NULL, 3400.00),
(68, '2026-02-28', 13, 642, N'Seguridad Social', 1020.00),
(69, '2026-02-28', 15, 640, NULL, 9700.00),
(70, '2026-02-28', 15, 642, N'Seguridad Social', 2910.00),
(71, '2026-02-28', 16, 640, NULL, 13500.00),
(72, '2026-02-28', 16, 642, N'Seguridad Social', 4050.00),
(73, '2026-02-28', 17, 629, N'DataCloud Services', 3300.00),
(74, '2026-02-28', 17, 640, NULL, 6300.00),
(75, '2026-02-28', 17, 642, N'Seguridad Social', 1890.00),
-- Marzo de 2026
(76, '2026-03-01', 4, 621, N'Oficinas Torre Aragonia', 9000.00),
(77, '2026-03-01', 13, 621, N'Inmobiliaria Ribera', 6500.00),
(78, '2026-03-01', 14, 621, N'Parque Industrial Apodaca', 4200.00),
(79, '2026-03-03', 10, 629, N'ERP Nimbus', 7400.00),
(80, '2026-03-05', 8, 623, N'Ingeniería Moncayo', 24000.00),
(81, '2026-03-05', 14, 623, N'Operador Logístico Regio', 5500.00),
(82, '2026-03-12', 11, 627, N'Ferias y Congresos Norte', 14500.00),
(83, '2026-03-12', 13, 600, N'Suministros Eléctricos Levante', 183200.00),
(84, '2026-03-14', 14, 600, N'Proveedora Industrial del Norte', 82100.00),
(85, '2026-03-20', 7, 623, N'Servicios Técnicos Aragón', 28900.00),
(86, '2026-03-20', 9, 623, N'Auditores Asociados del Ebro', 28000.00),
(87, '2026-03-25', 13, 601, N'Transportes Jalón', 13600.00),
(88, '2026-03-26', 14, 601, N'Fletes Sierra Madre', 7100.00),
(89, '2026-03-28', 17, 629, N'DataCloud Services', 3600.00),
(90, '2026-03-31', 2, 640, NULL, 8200.00),
(91, '2026-03-31', 2, 642, N'Seguridad Social', 2460.00),
(92, '2026-03-31', 3, 640, NULL, 8000.00),
(93, '2026-03-31', 3, 642, N'Seguridad Social', 2400.00),
(94, '2026-03-31', 4, 640, NULL, 12100.00),
(95, '2026-03-31', 4, 642, N'Seguridad Social', 3630.00),
(96, '2026-03-31', 7, 640, NULL, 4400.00),
(97, '2026-03-31', 7, 642, N'Seguridad Social', 1320.00),
(98, '2026-03-31', 8, 640, NULL, 5000.00),
(99, '2026-03-31', 8, 642, N'Seguridad Social', 1500.00),
(100, '2026-03-31', 9, 640, NULL, 9200.00),
(101, '2026-03-31', 9, 642, N'Seguridad Social', 2760.00),
(102, '2026-03-31', 10, 640, NULL, 8800.00),
(103, '2026-03-31', 10, 642, N'Seguridad Social', 2640.00),
(104, '2026-03-31', 11, 640, NULL, 11600.00),
(105, '2026-03-31', 11, 642, N'Seguridad Social', 3480.00),
(106, '2026-03-31', 12, 640, NULL, 4900.00),
(107, '2026-03-31', 12, 642, N'IMSS', 1470.00),
(108, '2026-03-31', 13, 640, NULL, 3400.00),
(109, '2026-03-31', 13, 642, N'Seguridad Social', 1020.00),
(110, '2026-03-31', 15, 640, NULL, 9700.00),
(111, '2026-03-31', 15, 642, N'Seguridad Social', 2910.00),
(112, '2026-03-31', 16, 640, NULL, 13500.00),
(113, '2026-03-31', 16, 642, N'Seguridad Social', 4050.00),
(114, '2026-03-31', 17, 640, NULL, 6300.00),
(115, '2026-03-31', 17, 642, N'Seguridad Social', 1890.00),
-- Abril de 2026
(116, '2026-04-01', 4, 621, N'Oficinas Torre Aragonia', 9000.00),
(117, '2026-04-01', 13, 621, N'Inmobiliaria Ribera', 6500.00),
(118, '2026-04-01', 14, 621, N'Parque Industrial Apodaca', 4200.00),
(119, '2026-04-03', 10, 629, N'ERP Nimbus', 7400.00),
(120, '2026-04-05', 14, 623, N'Operador Logístico Regio', 5500.00),
(121, '2026-04-10', 8, 623, N'Ingeniería Moncayo', 96000.00),
(122, '2026-04-12', 13, 600, N'Cables y Conductores Aragón', 121700.00),
(123, '2026-04-14', 14, 600, N'Conductores Monterrey', 94500.00),
(124, '2026-04-20', 7, 623, N'Servicios Técnicos Aragón', 23200.00),
(125, '2026-04-22', 12, 627, N'Expo Industrial Monterrey', 7800.00),
(126, '2026-04-25', 13, 601, N'Transportes Jalón', 9000.00),
(127, '2026-04-26', 14, 601, N'Fletes Sierra Madre', 8200.00),
(128, '2026-04-28', 17, 629, N'DataCloud Services', 4200.00),
(129, '2026-04-30', 2, 640, NULL, 8200.00),
(130, '2026-04-30', 2, 642, N'Seguridad Social', 2460.00),
(131, '2026-04-30', 3, 640, NULL, 8000.00),
(132, '2026-04-30', 3, 642, N'Seguridad Social', 2400.00),
(133, '2026-04-30', 4, 640, NULL, 12100.00),
(134, '2026-04-30', 4, 642, N'Seguridad Social', 3630.00),
(135, '2026-04-30', 7, 640, NULL, 4400.00),
(136, '2026-04-30', 7, 642, N'Seguridad Social', 1320.00),
(137, '2026-04-30', 8, 640, NULL, 5000.00),
(138, '2026-04-30', 8, 642, N'Seguridad Social', 1500.00),
(139, '2026-04-30', 9, 640, NULL, 9200.00),
(140, '2026-04-30', 9, 642, N'Seguridad Social', 2760.00),
(141, '2026-04-30', 10, 640, NULL, 8800.00),
(142, '2026-04-30', 10, 642, N'Seguridad Social', 2640.00),
(143, '2026-04-30', 11, 640, NULL, 11600.00),
(144, '2026-04-30', 11, 642, N'Seguridad Social', 3480.00),
(145, '2026-04-30', 12, 640, NULL, 4900.00),
(146, '2026-04-30', 12, 642, N'IMSS', 1470.00),
(147, '2026-04-30', 13, 640, NULL, 3400.00),
(148, '2026-04-30', 13, 642, N'Seguridad Social', 1020.00),
(149, '2026-04-30', 15, 640, NULL, 9700.00),
(150, '2026-04-30', 15, 642, N'Seguridad Social', 2910.00),
(151, '2026-04-30', 16, 640, NULL, 13500.00),
(152, '2026-04-30', 16, 642, N'Seguridad Social', 4050.00),
(153, '2026-04-30', 17, 640, NULL, 10300.00),
(154, '2026-04-30', 17, 642, N'Seguridad Social', 3090.00),
-- Mayo de 2026
(155, '2026-05-01', 4, 621, N'Oficinas Torre Aragonia', 9000.00),
(156, '2026-05-01', 13, 621, N'Inmobiliaria Ribera', 6500.00),
(157, '2026-05-01', 14, 621, N'Parque Industrial Apodaca', 4200.00),
(158, '2026-05-03', 10, 629, N'ERP Nimbus', 7400.00),
(159, '2026-05-05', 14, 623, N'Operador Logístico Regio', 5500.00),
(160, '2026-05-12', 13, 600, N'Herramientas Bidasoa', 178700.00),
(161, '2026-05-14', 14, 600, N'Proveedora Industrial del Norte', 73200.00),
(162, '2026-05-15', 15, 623, N'Informes Mercantiles Ibéricos', 3200.00),
(163, '2026-05-20', 7, 623, N'Servicios Técnicos Aragón', 14600.00),
(164, '2026-05-25', 13, 601, N'Transportes Jalón', 13200.00),
(165, '2026-05-26', 14, 601, N'Fletes Sierra Madre', 6300.00),
(166, '2026-05-28', 17, 629, N'DataCloud Services', 4900.00),
(167, '2026-05-29', 11, 627, N'Agencia Brújula', 6000.00),
(168, '2026-05-31', 2, 640, NULL, 8200.00),
(169, '2026-05-31', 2, 642, N'Seguridad Social', 2460.00),
(170, '2026-05-31', 3, 640, NULL, 8000.00),
(171, '2026-05-31', 3, 642, N'Seguridad Social', 2400.00),
(172, '2026-05-31', 4, 640, NULL, 12100.00),
(173, '2026-05-31', 4, 642, N'Seguridad Social', 3630.00),
(174, '2026-05-31', 7, 640, NULL, 4400.00),
(175, '2026-05-31', 7, 642, N'Seguridad Social', 1320.00),
(176, '2026-05-31', 8, 640, NULL, 5000.00),
(177, '2026-05-31', 8, 642, N'Seguridad Social', 1500.00),
(178, '2026-05-31', 9, 640, NULL, 9200.00),
(179, '2026-05-31', 9, 642, N'Seguridad Social', 2760.00),
(180, '2026-05-31', 10, 640, NULL, 8800.00),
(181, '2026-05-31', 10, 642, N'Seguridad Social', 2640.00),
(182, '2026-05-31', 11, 640, NULL, 11600.00),
(183, '2026-05-31', 11, 642, N'Seguridad Social', 3480.00),
(184, '2026-05-31', 12, 640, NULL, 4900.00),
(185, '2026-05-31', 12, 642, N'IMSS', 1470.00),
(186, '2026-05-31', 13, 640, NULL, 3400.00),
(187, '2026-05-31', 13, 642, N'Seguridad Social', 1020.00),
(188, '2026-05-31', 15, 640, NULL, 9700.00),
(189, '2026-05-31', 15, 642, N'Seguridad Social', 2910.00),
(190, '2026-05-31', 16, 640, NULL, 13500.00),
(191, '2026-05-31', 16, 642, N'Seguridad Social', 4050.00),
(192, '2026-05-31', 17, 640, NULL, 10300.00),
(193, '2026-05-31', 17, 642, N'Seguridad Social', 3090.00),
-- Junio de 2026
(194, '2026-06-01', 4, 621, N'Oficinas Torre Aragonia', 9000.00),
(195, '2026-06-01', 13, 621, N'Inmobiliaria Ribera', 6500.00),
(196, '2026-06-01', 14, 621, N'Parque Industrial Apodaca', 4200.00),
(197, '2026-06-03', 10, 629, N'ERP Nimbus', 7400.00),
(198, '2026-06-05', 14, 623, N'Operador Logístico Regio', 5500.00),
(199, '2026-06-12', 13, 600, N'Suministros Eléctricos Levante', 149000.00),
(200, '2026-06-14', 14, 600, N'Conductores Monterrey', 83900.00),
(201, '2026-06-18', 8, 623, N'Montajes Industriales Gállego', 31500.00),
(202, '2026-06-20', 7, 623, N'Servicios Técnicos Aragón', 28900.00),
(203, '2026-06-25', 9, 623, N'Asesores Fiscales Pilar', 12000.00),
(204, '2026-06-25', 13, 601, N'Transportes Jalón', 11000.00),
(205, '2026-06-26', 14, 601, N'Fletes Sierra Madre', 7200.00),
(206, '2026-06-28', 17, 629, N'DataCloud Services', 5600.00),
(207, '2026-06-30', 2, 640, NULL, 8200.00),
(208, '2026-06-30', 2, 642, N'Seguridad Social', 2460.00),
(209, '2026-06-30', 3, 640, NULL, 8000.00),
(210, '2026-06-30', 3, 642, N'Seguridad Social', 2400.00),
(211, '2026-06-30', 4, 640, NULL, 12100.00),
(212, '2026-06-30', 4, 642, N'Seguridad Social', 3630.00),
(213, '2026-06-30', 7, 640, NULL, 4400.00),
(214, '2026-06-30', 7, 642, N'Seguridad Social', 1320.00),
(215, '2026-06-30', 8, 640, NULL, 5000.00),
(216, '2026-06-30', 8, 642, N'Seguridad Social', 1500.00),
(217, '2026-06-30', 9, 640, NULL, 9200.00),
(218, '2026-06-30', 9, 642, N'Seguridad Social', 2760.00),
(219, '2026-06-30', 10, 640, NULL, 8800.00),
(220, '2026-06-30', 10, 642, N'Seguridad Social', 2640.00),
(221, '2026-06-30', 11, 640, NULL, 11600.00),
(222, '2026-06-30', 11, 640, NULL, 9000.00),
(223, '2026-06-30', 11, 642, N'Seguridad Social', 3480.00),
(224, '2026-06-30', 12, 640, NULL, 4900.00),
(225, '2026-06-30', 12, 640, NULL, 4500.00),
(226, '2026-06-30', 12, 642, N'IMSS', 1470.00),
(227, '2026-06-30', 13, 640, NULL, 3400.00),
(228, '2026-06-30', 13, 642, N'Seguridad Social', 1020.00),
(229, '2026-06-30', 15, 640, NULL, 9700.00),
(230, '2026-06-30', 15, 642, N'Seguridad Social', 2910.00),
(231, '2026-06-30', 16, 640, NULL, 13500.00),
(232, '2026-06-30', 16, 642, N'Seguridad Social', 4050.00),
(233, '2026-06-30', 17, 640, NULL, 10300.00),
(234, '2026-06-30', 17, 642, N'Seguridad Social', 3090.00),
-- Julio de 2026
(235, '2026-07-01', 4, 621, N'Oficinas Torre Aragonia', 9000.00),
(236, '2026-07-01', 13, 621, N'Inmobiliaria Ribera', 6500.00),
(237, '2026-07-01', 14, 621, N'Parque Industrial Apodaca', 4200.00),
(238, '2026-07-02', 8, 623, N'Montajes Industriales Gállego', 18000.00),
(239, '2026-07-03', 10, 629, N'ERP Nimbus', 7400.00),
(240, '2026-07-05', 14, 623, N'Operador Logístico Regio', 5500.00),
(241, '2026-07-12', 13, 600, N'Cables y Conductores Aragón', 129400.00),
(242, '2026-07-14', 14, 600, N'Proveedora Industrial del Norte', 64800.00),
(243, '2026-07-20', 7, 623, N'Servicios Técnicos Aragón', 23200.00),
(244, '2026-07-25', 13, 601, N'Transportes Jalón', 9600.00),
(245, '2026-07-26', 14, 601, N'Fletes Sierra Madre', 5600.00),
(246, '2026-07-28', 17, 629, N'DataCloud Services', 6100.00),
(247, '2026-07-31', 2, 640, NULL, 8200.00),
(248, '2026-07-31', 2, 642, N'Seguridad Social', 2460.00),
(249, '2026-07-31', 3, 640, NULL, 8000.00),
(250, '2026-07-31', 3, 642, N'Seguridad Social', 2400.00),
(251, '2026-07-31', 4, 640, NULL, 12100.00),
(252, '2026-07-31', 4, 642, N'Seguridad Social', 3630.00),
(253, '2026-07-31', 7, 640, NULL, 4400.00),
(254, '2026-07-31', 7, 642, N'Seguridad Social', 1320.00),
(255, '2026-07-31', 8, 640, NULL, 5000.00),
(256, '2026-07-31', 8, 642, N'Seguridad Social', 1500.00),
(257, '2026-07-31', 9, 640, NULL, 9200.00),
(258, '2026-07-31', 9, 642, N'Seguridad Social', 2760.00),
(259, '2026-07-31', 10, 640, NULL, 8800.00),
(260, '2026-07-31', 10, 642, N'Seguridad Social', 2640.00),
(261, '2026-07-31', 11, 640, NULL, 11600.00),
(262, '2026-07-31', 11, 642, N'Seguridad Social', 3480.00),
(263, '2026-07-31', 12, 640, NULL, 7800.00),
(264, '2026-07-31', 12, 642, N'IMSS', 2340.00),
(265, '2026-07-31', 13, 640, NULL, 3400.00),
(266, '2026-07-31', 13, 642, N'Seguridad Social', 1020.00),
(267, '2026-07-31', 15, 640, NULL, 9700.00),
(268, '2026-07-31', 15, 642, N'Seguridad Social', 2910.00),
(269, '2026-07-31', 16, 640, NULL, 13500.00),
(270, '2026-07-31', 16, 642, N'Seguridad Social', 4050.00),
(271, '2026-07-31', 17, 640, NULL, 10300.00),
(272, '2026-07-31', 17, 642, N'Seguridad Social', 3090.00),
-- Agosto de 2026
(273, '2026-08-01', 4, 621, N'Oficinas Torre Aragonia', 9000.00),
(274, '2026-08-01', 13, 621, N'Inmobiliaria Ribera', 6500.00),
(275, '2026-08-01', 14, 621, N'Parque Industrial Apodaca', 4200.00),
(276, '2026-08-03', 10, 629, N'ERP Nimbus', 7400.00),
(277, '2026-08-05', 8, 623, N'Ingeniería Moncayo', 71000.00),
(278, '2026-08-05', 14, 623, N'Operador Logístico Regio', 5500.00),
(279, '2026-08-12', 13, 600, N'Herramientas Bidasoa', 137100.00),
(280, '2026-08-14', 14, 600, N'Conductores Monterrey', 51000.00),
(281, '2026-08-17', 15, 623, N'Informes Mercantiles Ibéricos', 4800.00),
(282, '2026-08-20', 7, 623, N'Servicios Técnicos Aragón', 14600.00),
(283, '2026-08-25', 13, 601, N'Transportes Jalón', 10200.00),
(284, '2026-08-26', 14, 601, N'Fletes Sierra Madre', 4400.00),
(285, '2026-08-27', 12, 627, N'Expo Industrial Monterrey', 9400.00),
(286, '2026-08-28', 17, 629, N'DataCloud Services', 6800.00),
(287, '2026-08-31', 2, 640, NULL, 8200.00),
(288, '2026-08-31', 2, 642, N'Seguridad Social', 2460.00),
(289, '2026-08-31', 3, 640, NULL, 8000.00),
(290, '2026-08-31', 3, 642, N'Seguridad Social', 2400.00),
(291, '2026-08-31', 4, 640, NULL, 12100.00),
(292, '2026-08-31', 4, 642, N'Seguridad Social', 3630.00),
(293, '2026-08-31', 7, 640, NULL, 4400.00),
(294, '2026-08-31', 7, 642, N'Seguridad Social', 1320.00),
(295, '2026-08-31', 8, 640, NULL, 5000.00),
(296, '2026-08-31', 8, 642, N'Seguridad Social', 1500.00),
(297, '2026-08-31', 9, 640, NULL, 9200.00),
(298, '2026-08-31', 9, 642, N'Seguridad Social', 2760.00),
(299, '2026-08-31', 10, 640, NULL, 8800.00),
(300, '2026-08-31', 10, 642, N'Seguridad Social', 2640.00),
(301, '2026-08-31', 11, 640, NULL, 11600.00),
(302, '2026-08-31', 11, 642, N'Seguridad Social', 3480.00),
(303, '2026-08-31', 12, 640, NULL, 7800.00),
(304, '2026-08-31', 12, 642, N'IMSS', 2340.00),
(305, '2026-08-31', 13, 640, NULL, 3400.00),
(306, '2026-08-31', 13, 642, N'Seguridad Social', 1020.00),
(307, '2026-08-31', 15, 640, NULL, 9700.00),
(308, '2026-08-31', 15, 642, N'Seguridad Social', 2910.00),
(309, '2026-08-31', 16, 640, NULL, 13500.00),
(310, '2026-08-31', 16, 642, N'Seguridad Social', 4050.00),
(311, '2026-08-31', 17, 640, NULL, 10300.00),
(312, '2026-08-31', 17, 642, N'Seguridad Social', 3090.00),
-- Septiembre de 2026
(313, '2026-09-01', 4, 621, N'Oficinas Torre Aragonia', 9000.00),
(314, '2026-09-01', 13, 621, N'Inmobiliaria Ribera', 6500.00),
(315, '2026-09-01', 14, 621, N'Parque Industrial Apodaca', 4200.00),
(316, '2026-09-03', 10, 629, N'ERP Nimbus', 7400.00),
(317, '2026-09-05', 14, 623, N'Operador Logístico Regio', 5500.00),
(318, '2026-09-12', 13, 600, N'Suministros Eléctricos Levante', 104500.00),
(319, '2026-09-14', 14, 600, N'Proveedora Industrial del Norte', 58200.00),
(320, '2026-09-18', 11, 627, N'Ferias y Congresos Norte', 18200.00),
(321, '2026-09-20', 7, 623, N'Servicios Técnicos Aragón', 28900.00),
(322, '2026-09-25', 13, 601, N'Transportes Jalón', 7700.00),
(323, '2026-09-26', 14, 601, N'Fletes Sierra Madre', 5000.00),
(324, '2026-09-28', 17, 629, N'DataCloud Services', 7500.00),
(325, '2026-09-30', 2, 640, NULL, 8200.00),
(326, '2026-09-30', 2, 642, N'Seguridad Social', 2460.00),
(327, '2026-09-30', 3, 640, NULL, 8000.00),
(328, '2026-09-30', 3, 642, N'Seguridad Social', 2400.00),
(329, '2026-09-30', 4, 640, NULL, 12100.00),
(330, '2026-09-30', 4, 642, N'Seguridad Social', 3630.00),
(331, '2026-09-30', 7, 640, NULL, 4400.00),
(332, '2026-09-30', 7, 642, N'Seguridad Social', 1320.00),
(333, '2026-09-30', 8, 640, NULL, 5000.00),
(334, '2026-09-30', 8, 642, N'Seguridad Social', 1500.00),
(335, '2026-09-30', 9, 640, NULL, 9200.00),
(336, '2026-09-30', 9, 642, N'Seguridad Social', 2760.00),
(337, '2026-09-30', 10, 640, NULL, 8800.00),
(338, '2026-09-30', 10, 642, N'Seguridad Social', 2640.00),
(339, '2026-09-30', 11, 640, NULL, 11600.00),
(340, '2026-09-30', 11, 642, N'Seguridad Social', 3480.00),
(341, '2026-09-30', 12, 640, NULL, 7800.00),
(342, '2026-09-30', 12, 642, N'IMSS', 2340.00),
(343, '2026-09-30', 13, 640, NULL, 3400.00),
(344, '2026-09-30', 13, 642, N'Seguridad Social', 1020.00),
(345, '2026-09-30', 15, 640, NULL, 9700.00),
(346, '2026-09-30', 15, 642, N'Seguridad Social', 2910.00),
(347, '2026-09-30', 16, 640, NULL, 13500.00),
(348, '2026-09-30', 16, 642, N'Seguridad Social', 4050.00),
(349, '2026-09-30', 17, 640, NULL, 10300.00),
(350, '2026-09-30', 17, 642, N'Seguridad Social', 3090.00);

-- Presupuesto trimestral de 2026 por unidad y cuenta (el T4 está presupuestado pero aún no tiene datos reales)

INSERT INTO dbo.presupuestos (presupuesto_id, periodo_id, unidad_id, cuenta_id, importe) VALUES
-- Trimestre 1 de 2026
(1, 1, 2, 640, 24600.00),
(2, 1, 2, 642, 7380.00),
(3, 1, 3, 640, 24000.00),
(4, 1, 3, 642, 7200.00),
(5, 1, 4, 621, 27000.00),
(6, 1, 4, 640, 36300.00),
(7, 1, 4, 642, 10890.00),
(8, 1, 7, 623, 60000.00),
(9, 1, 7, 640, 13200.00),
(10, 1, 7, 642, 3960.00),
(11, 1, 7, 710, 124500.00),
(12, 1, 8, 623, 60000.00),
(13, 1, 8, 640, 15000.00),
(14, 1, 8, 642, 4500.00),
(15, 1, 8, 711, 46000.00),
(16, 1, 9, 623, 15000.00),
(17, 1, 9, 640, 27600.00),
(18, 1, 9, 642, 8280.00),
(19, 1, 10, 629, 22200.00),
(20, 1, 10, 640, 26400.00),
(21, 1, 10, 642, 7920.00),
(22, 1, 11, 627, 12000.00),
(23, 1, 11, 640, 34800.00),
(24, 1, 11, 642, 10440.00),
(25, 1, 11, 649, 4000.00),
(26, 1, 11, 700, 591000.00),
(27, 1, 11, 701, 361000.00),
(28, 1, 12, 640, 14700.00),
(29, 1, 12, 642, 4410.00),
(30, 1, 12, 700, 184000.00),
(31, 1, 12, 701, 234500.00),
(32, 1, 13, 600, 525500.00),
(33, 1, 13, 601, 36000.00),
(34, 1, 13, 621, 19500.00),
(35, 1, 13, 640, 10200.00),
(36, 1, 13, 642, 3060.00),
(37, 1, 14, 600, 207000.00),
(38, 1, 14, 601, 20000.00),
(39, 1, 14, 621, 12600.00),
(40, 1, 14, 623, 16500.00),
(41, 1, 15, 640, 29100.00),
(42, 1, 15, 642, 8730.00),
(43, 1, 16, 640, 40500.00),
(44, 1, 16, 642, 12150.00),
(45, 1, 16, 649, 3000.00),
(46, 1, 17, 629, 10500.00),
(47, 1, 17, 640, 18900.00),
(48, 1, 17, 642, 5670.00),
(49, 1, 17, 649, 5000.00),
-- Trimestre 2 de 2026
(50, 2, 2, 640, 24600.00),
(51, 2, 2, 642, 7380.00),
(52, 2, 3, 640, 24000.00),
(53, 2, 3, 642, 7200.00),
(54, 2, 4, 621, 27000.00),
(55, 2, 4, 640, 36300.00),
(56, 2, 4, 642, 10890.00),
(57, 2, 7, 623, 68000.00),
(58, 2, 7, 640, 13200.00),
(59, 2, 7, 642, 3960.00),
(60, 2, 7, 710, 119000.00),
(61, 2, 8, 623, 60000.00),
(62, 2, 8, 640, 15000.00),
(63, 2, 8, 642, 4500.00),
(64, 2, 8, 711, 175500.00),
(65, 2, 9, 623, 15000.00),
(66, 2, 9, 640, 27600.00),
(67, 2, 9, 642, 8280.00),
(68, 2, 10, 629, 22200.00),
(69, 2, 10, 640, 26400.00),
(70, 2, 10, 642, 7920.00),
(71, 2, 11, 627, 12000.00),
(72, 2, 11, 640, 34800.00),
(73, 2, 11, 642, 10440.00),
(74, 2, 11, 649, 4000.00),
(75, 2, 11, 700, 286500.00),
(76, 2, 11, 701, 572000.00),
(77, 2, 12, 640, 14700.00),
(78, 2, 12, 642, 4410.00),
(79, 2, 12, 700, 100000.00),
(80, 2, 12, 701, 339000.00),
(81, 2, 13, 600, 418000.00),
(82, 2, 13, 601, 34000.00),
(83, 2, 13, 621, 19500.00),
(84, 2, 13, 640, 10200.00),
(85, 2, 13, 642, 3060.00),
(86, 2, 14, 600, 256500.00),
(87, 2, 14, 601, 21500.00),
(88, 2, 14, 621, 12600.00),
(89, 2, 14, 623, 16500.00),
(90, 2, 15, 640, 29100.00),
(91, 2, 15, 642, 8730.00),
(92, 2, 16, 640, 40500.00),
(93, 2, 16, 642, 12150.00),
(94, 2, 16, 649, 3000.00),
(95, 2, 17, 629, 10500.00),
(96, 2, 17, 640, 18900.00),
(97, 2, 17, 642, 5670.00),
(98, 2, 17, 649, 5000.00),
-- Trimestre 3 de 2026
(99, 3, 2, 640, 24600.00),
(100, 3, 2, 642, 7380.00),
(101, 3, 3, 640, 24000.00),
(102, 3, 3, 642, 7200.00),
(103, 3, 4, 621, 27000.00),
(104, 3, 4, 640, 36300.00),
(105, 3, 4, 642, 10890.00),
(106, 3, 7, 623, 63500.00),
(107, 3, 7, 640, 13200.00),
(108, 3, 7, 642, 3960.00),
(109, 3, 7, 710, 136000.00),
(110, 3, 8, 623, 60000.00),
(111, 3, 8, 640, 15000.00),
(112, 3, 8, 642, 4500.00),
(113, 3, 8, 711, 156500.00),
(114, 3, 9, 623, 15000.00),
(115, 3, 9, 640, 27600.00),
(116, 3, 9, 642, 8280.00),
(117, 3, 10, 629, 22200.00),
(118, 3, 10, 640, 26400.00),
(119, 3, 10, 642, 7920.00),
(120, 3, 11, 627, 12000.00),
(121, 3, 11, 640, 34800.00),
(122, 3, 11, 642, 10440.00),
(123, 3, 11, 649, 4000.00),
(124, 3, 11, 700, 643000.00),
(125, 3, 11, 701, 66000.00),
(126, 3, 12, 640, 14700.00),
(127, 3, 12, 642, 4410.00),
(128, 3, 12, 700, 145500.00),
(129, 3, 12, 701, 203500.00),
(130, 3, 13, 600, 393500.00),
(131, 3, 13, 601, 26000.00),
(132, 3, 13, 621, 19500.00),
(133, 3, 13, 640, 10200.00),
(134, 3, 13, 642, 3060.00),
(135, 3, 14, 600, 165500.00),
(136, 3, 14, 601, 15500.00),
(137, 3, 14, 621, 12600.00),
(138, 3, 14, 623, 16500.00),
(139, 3, 15, 640, 29100.00),
(140, 3, 15, 642, 8730.00),
(141, 3, 16, 640, 40500.00),
(142, 3, 16, 642, 12150.00),
(143, 3, 16, 649, 3000.00),
(144, 3, 17, 629, 10500.00),
(145, 3, 17, 640, 18900.00),
(146, 3, 17, 642, 5670.00),
(147, 3, 17, 649, 5000.00),
-- Trimestre 4 de 2026
(148, 4, 2, 640, 24600.00),
(149, 4, 2, 642, 7380.00),
(150, 4, 3, 640, 24000.00),
(151, 4, 3, 642, 7200.00),
(152, 4, 4, 621, 27000.00),
(153, 4, 4, 640, 36300.00),
(154, 4, 4, 642, 10890.00),
(155, 4, 7, 623, 67000.00),
(156, 4, 7, 640, 13200.00),
(157, 4, 7, 642, 3960.00),
(158, 4, 7, 710, 133000.00),
(159, 4, 8, 623, 60000.00),
(160, 4, 8, 640, 15000.00),
(161, 4, 8, 642, 4500.00),
(162, 4, 8, 711, 132500.00),
(163, 4, 9, 623, 15000.00),
(164, 4, 9, 640, 27600.00),
(165, 4, 9, 642, 8280.00),
(166, 4, 10, 629, 22200.00),
(167, 4, 10, 640, 26400.00),
(168, 4, 10, 642, 7920.00),
(169, 4, 11, 627, 12000.00),
(170, 4, 11, 640, 34800.00),
(171, 4, 11, 642, 10440.00),
(172, 4, 11, 649, 4000.00),
(173, 4, 11, 700, 532000.00),
(174, 4, 11, 701, 349500.00),
(175, 4, 12, 640, 14700.00),
(176, 4, 12, 642, 4410.00),
(177, 4, 12, 700, 150500.00),
(178, 4, 12, 701, 272000.00),
(179, 4, 13, 600, 468000.00),
(180, 4, 13, 601, 33500.00),
(181, 4, 13, 621, 19500.00),
(182, 4, 13, 640, 10200.00),
(183, 4, 13, 642, 3060.00),
(184, 4, 14, 600, 220000.00),
(185, 4, 14, 601, 20000.00),
(186, 4, 14, 621, 12600.00),
(187, 4, 14, 623, 16500.00),
(188, 4, 15, 640, 29100.00),
(189, 4, 15, 642, 8730.00),
(190, 4, 16, 640, 40500.00),
(191, 4, 16, 642, 12150.00),
(192, 4, 16, 649, 3000.00),
(193, 4, 17, 629, 10500.00),
(194, 4, 17, 640, 18900.00),
(195, 4, 17, 642, 5670.00),
(196, 4, 17, 649, 5000.00);

-- Staging: lote de cobros recibido del banco el 02/10/2026, tal como llega.
-- Contiene cobros ya cargados, una fila repetida dentro del lote, una factura inexistente y un importe negativo

INSERT INTO dbo.stg_cobros (cobro_id, factura_id, fecha_cobro, importe, metodo) VALUES
(47, 62, '2026-09-29', 8400.00, N'Transferencia'),
(48, 61, '2026-09-30', 19600.00, N'Domiciliación'),
(49, 41, '2026-10-01', 13200.00, N'Transferencia'),
(50, 42, '2026-10-01', 91200.00, N'Confirming'),
(51, 48, '2026-10-02', 16000.00, N'Transferencia'),
(51, 48, '2026-10-02', 16000.00, N'Transferencia'),
(52, 999, '2026-10-01', 10000.00, N'Transferencia'),
(53, 27, '2026-10-02', -2400.00, N'Transferencia'),
(54, 16, '2026-10-02', 57000.00, N'Pagaré'),
(55, 63, '2026-10-02', 15400.00, N'Domiciliación');

GO

/* =====================================================================
   4. VERIFICACIÓN DE LA CARGA
   ===================================================================== */

SELECT 'periodos' AS tabla, COUNT(*) AS filas FROM dbo.periodos
UNION ALL SELECT 'unidades', COUNT(*) FROM dbo.unidades
UNION ALL SELECT 'cuentas', COUNT(*) FROM dbo.cuentas
UNION ALL SELECT 'empleados', COUNT(*) FROM dbo.empleados
UNION ALL SELECT 'clientes', COUNT(*) FROM dbo.clientes
UNION ALL SELECT 'facturas', COUNT(*) FROM dbo.facturas
UNION ALL SELECT 'cobros', COUNT(*) FROM dbo.cobros
UNION ALL SELECT 'gastos', COUNT(*) FROM dbo.gastos
UNION ALL SELECT 'presupuestos', COUNT(*) FROM dbo.presupuestos
UNION ALL SELECT 'stg_cobros', COUNT(*) FROM dbo.stg_cobros;
GO

/* Salida esperada:

   tabla          filas
   -------------  -----
   periodos           4
   unidades          17
   cuentas           20
   empleados         20
   clientes          14
   facturas          68
   cobros            48
   gastos           350
   presupuestos     196
   stg_cobros        10
*/
```

