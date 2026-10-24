# 📋Clase 07 - Fundamentos de datos y servicios de Azure para Datos

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-001.png)

---

# PARTE 1 — Explorar conceptos de datos

## Sección 1: Qué son los datos y cómo se clasifican

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-002.png)

### 1.2 Datos semiestructurados

Tienen **estructura, pero flexible**: cada registro puede tener campos distintos o anidados. El formato más habitual es **JSON**.

```json
{
  "id_socio": "S-102938",
  "nombre": "Laura Méndez",
  "preferencias": ["ecológico", "sin gluten"],
  "cupones": [
    { "codigo": "FRUTA10", "canjeado": true, "tienda": "TND-042" },
    { "codigo": "LACT5" }
  ],
  "consentimiento_marketing": true
}
```

Observa que el segundo cupón no tiene los campos `canjeado` ni `tienda`. En una tabla relacional eso obligaría a rellenar con `NULL`; en JSON simplemente no aparecen.

**En Grupo Alimenta:** perfiles y eventos de la app de fidelización, pedidos de la tienda online recibidos por API, clics de la web.

### 1.3 Datos no estructurados

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-003.png)

### Ejercicio 1 — Clasifica las fuentes

Clasifica cada fuente como estructurada, semiestructurada o no estructurada y justifica tu respuesta:

1. Exportación diaria del ERP de compras en CSV.
2. Lecturas de temperatura de las cámaras frigoríficas enviadas cada 30 segundos en JSON.
3. Grabaciones MP3 del servicio de atención al cliente.
4. Tabla `Empleados` de la base de datos de RR. HH.
5. Logs de acceso de la tienda online.
6. Fotografías de los lineales tomadas por los reponedores.
> 💡 ¿Qué es un ERP y en que se diferencia de un CRM?
![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-004.png)

---

## Sección 2: Almacenamiento de datos en ficheros

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-005.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-006.png)

### 2.4 BLOB (Binary Large Object)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-007.png)

### 2.5 Formatos optimizados: el terreno del ingeniero de datos

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-008.png)

### 2.6 Comparativa rápida

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-009.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-010.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-011.png)

### 2.7 Demostración: de CSV a Delta en un notebook de Spark

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-012.png)

### Ejercicio 2 — Elige el formato

Para cada escenario de Grupo Alimenta, elige el formato más adecuado y justifica la decisión:

7. Un proveedor pequeño nos envía cada semana su tarifa de precios y solo sabe trabajar con Excel.
8. Almacenar 5 años de líneas de ticket para que los analistas estudien la estacionalidad de ventas por familia de producto.
9. Tabla de socios de fidelización que se actualiza a diario y sobre la que el delegado de protección de datos exige poder ver cómo estaba hace un mes.
10. Flujo continuo de lecturas de temperatura de las cámaras frigoríficas.
11. Recepción de pedidos de compra con un gran proveedor que usa EDI.
---

## Sección 3: Almacenamiento de datos en bases de datos

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-013.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-014.png)

Las bases relacionales se consultan con **SQL** (Structured Query Language):

```sql
SELECT t.ciudad,
       COUNT(k.id_ticket) AS num_tickets
FROM   Tiendas t
JOIN   Tickets k ON k.id_tienda = t.id_tienda
WHERE  k.fecha_hora >= '2026-09-01'
GROUP  BY t.ciudad
ORDER  BY num_tickets DESC;
```

### 3.2 Bases de datos no relacionales (NoSQL)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-015.png)

---

## Sección 4: Procesamiento de datos transaccional (OLTP)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-016.png)

```sql
BEGIN TRANSACTION;

    INSERT INTO PedidosOnline (id_pedido, id_socio, id_tienda, fecha)
    VALUES ('PO-77120', 'S-102938', 'TND-042', GETDATE());

    UPDATE Inventario
    SET    stock = stock - 6
    WHERE  id_producto = 501 AND id_tienda = 'TND-042' AND stock >= 6;

    IF @@ROWCOUNT = 0
        ROLLBACK TRANSACTION;   -- no hay stock: se deshace todo
    ELSE
        COMMIT TRANSACTION;     -- ambas operaciones quedan confirmadas
```

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-017.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-018.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-019.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-020.png)

---

## Sección 5: Procesamiento de datos analítico

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-021.png)

### 5.2 ETL frente a ELT

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-022.png)

### 5.3 Data lake, data warehouse y data lakehouse

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-023.png)

### 5.4 Modelo dimensional: hechos y dimensiones

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-024.png)

### 5.5 OLTP frente a OLAP

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-025.png)

### 5.6 Arquitectura medallón

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-026.png)

```python
# Bronze → Silver: limpieza básica con PySpark
from pyspark.sql import functions as F

bronze = spark.read.json("Files/bronze/app_fidelizacion/2026/09/")

silver = (bronze
    .dropDuplicates(["id_socio", "id_evento"])
    .withColumn("ts_utc", F.to_timestamp("timestamp_evento"))
    .withColumn("importe", F.col("importe").cast("decimal(10,2)"))
    .filter(F.col("id_socio").isNotNull()))

silver.write.format("delta").mode("append").saveAsTable("silver_eventos_socios")

# Silver → Gold: agregación para negocio
gold = (spark.table("silver_eventos_socios")
    .filter(F.col("tipo_evento") == "cupon_canjeado")
    .groupBy("codigo_cupon", F.to_date("ts_utc").alias("fecha"))
    .agg(F.count("*").alias("canjes"),
         F.sum("importe").alias("importe_asociado")))

gold.write.format("delta").mode("overwrite").saveAsTable("gold_rendimiento_cupones")
```

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-027.png)

### Ejercicio 3 — OLTP u OLAP

Indica si cada necesidad corresponde a un sistema transaccional o analítico:

12. Aplicar un cupón de descuento en la caja.
13. Calcular la evolución de las ventas de productos ecológicos en los últimos 3 años.
14. Consultar si queda aceite de oliva en la tienda de Valencia ahora mismo.
15. Detectar qué tiendas tienen más roturas de stock los lunes.
16. Actualizar la dirección de entrega de un pedido online en curso.
---

# PARTE 2 — Roles y servicios de datos en Azure

## Sección 6: Los roles del mundo del dato

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-028.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-029.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-030.png)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-031.png)

### 6.5 Científico de datos (Data Scientist)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-032.png)

### 6.6 Arquitecto de datos (Data Architect)

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-033.png)

### 6.7 Analytics Engineer

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-034.png)

### Ejercicio 4 — ¿Qué rol es responsable?

17. El informe de ventas lleva dos días sin actualizarse porque ha fallado la carga nocturna.
18. Hay que restaurar la base de datos de la tienda online tras un borrado accidental.
19. Dirección quiere un nuevo gráfico de margen por familia de producto en el cuadro de mando.
20. Atención al cliente quiere un asistente que responda "¿cuándo llega mi pedido?".
21. Hay que enmascarar los teléfonos de los socios antes de que lleguen a la capa analítica.
22. Marketing quiere saber qué socios tienen mayor probabilidad de dejar de comprar en los próximos 3 meses.
23. Se debe decidir si la plataforma se construye en Fabric, en Databricks o combinando ambos.
24. Dos departamentos presentan en el comité cifras distintas de "venta neta" para el mismo mes.
---

## Sección 7: Modelos de servicio en la nube: IaaS, PaaS y SaaS

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-035.png)

---

## Sección 8: Servicios de datos de Azure organizados por capa

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-036.png)

### 8.1 Capa de fuentes: bases de datos operativas

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-037.png)

### 8.2 Capa de almacenamiento

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-038.png)

### 8.3 Capa de ingesta y orquestación

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-039.png)

### 8.4 Capa de procesamiento y modelado

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-040.png)

### 8.5 Capa de consumo

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-041.png)

### 8.6 Gobierno transversal

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-042.png)

### 8.7 Tabla resumen: servicio, rol y tipo de carga

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-043.png)

---

## Sección 9: Cómo elegir: Fabric, Databricks o combinación

![Imagen](images/clase-07-fundamentos-de-datos-y-servicios-de-azure-para-datos/image-044.png)

---

## Ejercicio 5 : Investiga y arma un diagrama de arquitectura de capas (como el de la sección 8) con cada una de estas tecnologías:

- Databricks (no Azure Databricks)
- Microsoft Fabric. Es decir todo enteramente dentro de Fabric
- AWS
- GCP
- Herramientas Open Source ( el mas importante de todos )
- ¿Y que pasa con Snowflake , dbt y DuckDB?  ¿En que casos se utilizan? ¿En que capas se pueden incluir o con cuales otras tecnologías se puede combinar? Crear al menos 3 diagramas para este caso
>
1. Para cada diagrama debes incluir el logo oficial de cada servicio o tecnología y explicar brevemente cada uno. 
2. Para cada diagrama investiga casos de éxito.

## Sección 10: Caso práctico — Plataforma de datos de Grupo Alimenta

>
### Contexto

Grupo Alimenta quiere tres cosas:

1. Un **cuadro de mando de ventas, margen y roturas de stock** por tienda y familia de producto.
2. Un **modelo de previsión de demanda** de productos frescos para reducir la merma.
3. Un **asistente para atención al cliente** que responda sobre el estado de los pedidos online.

**Fuentes disponibles:**

| Fuente | Tecnología | Tipo de dato | Frecuencia |
| --- | --- | --- | --- |
| TPV de tiendas y ERP de compras | Azure SQL Database | Estructurado | Cambios continuos |
| App de fidelización | Azure Cosmos DB (JSON) | Semiestructurado | Eventos continuos |
| Sensores de cámaras frigoríficas | Dispositivos IoT | Semiestructurado (streaming) | Cada 30 segundos |
| Tarifas de proveedores | CSV por SFTP | Estructurado | Semanal |
| Facturas de proveedores | PDF escaneados | No estructurado | Por factura |
| Meteorología y festivos | API pública | Semiestructurado | Diario |

### Tareas

25. **Clasificación.** Clasifica cada fuente (estructurada, semiestructurada, no estructurada) e indica si es OLTP, fichero o stream.
26. **Arquitectura.** Dibuja  una arquitectura con capas Bronze, Silver y Gold. Indica qué servicio de Azure usarías en cada paso y justifica si optas por Fabric, Databricks u otro stack
27. **Formatos.** Indica el formato de almacenamiento de cada capa y por qué.
28. **Modelo Gold.** Diseña el esquema en estrella de ventas: tabla de hechos, granularidad, medidas y al menos cuatro dimensiones.
29. **Roles.** Asigna cada paso de la arquitectura a uno de los siete roles vistos, incluyendo arquitecto de datos, analytics engineer y científico de datos.
30. **Métricas.** Como analytics engineer, redacta la definición oficial de "rotura de stock" y dos pruebas de calidad que aplicarías a la tabla Gold.
31. **Ciencia de datos.** Indica qué tablas y variables necesitaría el científico de datos para el modelo de previsión de demanda y cómo escribirías sus predicciones de vuelta en la plataforma.
32. **Gobierno.** Los datos de socios incluyen nombre, teléfono y correo. Explica qué harías en cada capa y qué papel juega Purview.
33. **IA.** Explica qué datos necesitaría el ingeniero de IA para construir el asistente en Foundry y qué requisitos de calidad le exigirías como ingeniero de datos.
---

## Sección 12: Repaso de conceptos

34. Un fichero donde cada evento puede tener campos distintos y anidados es un ejemplo de dato:
a) Estructurado b) Semiestructurado c) No estructurado d) Binario
35. ¿Qué formato almacena juntos los valores de cada columna y es el estándar de facto de los lakehouses?
a) Avro b) CSV c) Parquet d) XML
36. ¿Qué aporta Delta Lake sobre Parquet?
a) Legibilidad humana b) Un registro de transacciones con ACID y versionado c) Almacenamiento orientado a filas d) Soporte exclusivo para imágenes
37. Si una transacción crea un pedido pero falla la reserva de stock y se deshace todo, se está garantizando la:
a) Durabilidad b) Atomicidad c) Consistencia d) Disponibilidad
38. En el patrón ELT, las transformaciones se realizan:
a) Antes de extraer b) En un servidor intermedio c) En el sistema de destino d) En la aplicación de origen
39. ¿Qué capa de la arquitectura medallón contiene datos limpios, deduplicados y con tipos estandarizados?
a) Bronze b) Silver c) Gold d) Platinum
40. ¿Qué rol es el responsable de monitorizar que los pipelines de carga se ejecutan correctamente?
a) DBA b) Analista de datos c) Ingeniero de datos d) Ingeniero de IA
41. ¿Qué característica de Azure Storage permite usarlo como data lake?
a) File shares b) Tables c) Espacio de nombres jerárquico sobre Blob d) Colas
42. ¿Cuál es la opción recomendada de orquestación cuando todo el trabajo de datos se realiza dentro de Microsoft Fabric?
a) Azure Data Factory b) Fabric Data Factory c) Azure Stream Analytics d) Azure Data Explorer
43. ¿Qué servicio usarías para trazar el linaje de un dato desde el TPV hasta un informe de Power BI?
a) Microsoft Foundry b) Azure Cosmos DB c) Microsoft Purview d) Azure Databricks
44. Microsoft Fabric se ofrece como:
a) IaaS b) PaaS c) SaaS d) On-premises
45. Una empresa necesita detectar en tiempo real cámaras frigoríficas que superan un umbral de temperatura. ¿Qué servicio encaja mejor?
a) Power BI b) Azure Stream Analytics c) Azure SQL Managed Instance d) Microsoft Purview
46. ¿Qué rol es responsable de que la métrica "venta neta" tenga una única definición oficial, probada y documentada?
a) DBA b) Analytics engineer c) Ingeniero de IA d) Científico de datos
47. ¿Qué rol construiría un modelo para predecir qué clientes dejarán de comprar en los próximos meses?
a) Analista de datos b) Arquitecto de datos c) Científico de datos d) DBA
48. ¿Qué rol decide si la plataforma de la empresa se organiza por dominios y qué servicios se usan en cada capa?
a) Arquitecto de datos b) Analytics engineer c) Analista de datos d) Ingeniero de IA

