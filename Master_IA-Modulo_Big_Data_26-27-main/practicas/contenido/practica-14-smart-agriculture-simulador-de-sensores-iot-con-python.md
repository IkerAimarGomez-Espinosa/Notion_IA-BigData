# 🚜Practica 14 - Smart Agriculture: Simulador de sensores IoT con Python

## 1. Contexto empresarial

La empresa ficticia **AgroSmart Solutions** gestiona una explotación agrícola inteligente. Dispone de varias parcelas con diferentes cultivos, sensores de suelo, una estación meteorológica y sistemas de riego.

El departamento de Ingeniería de Datos necesita un **simulador de dispositivos IoT** que produzca datos sintéticos periódicamente. Estos datos serán utilizados en futuras prácticas de ingesta, procesamiento y análisis, pero **en esta práctica únicamente se generarán y guardarán en archivos locales**.

**Reto:** Debes desarrollar, documentar y versionar en un repositorio GitHub una aplicación Python capaz de generar eventos coherentes con el comportamiento de una explotación agrícola.

## 2. Escenario de simulación

Se simularán cinco parcelas:

| Parcela | Cultivo | Superficie |
| --- | --- | --- |
| P001 | Tomates | 2 ha |
| P002 | Lechugas | 1,5 ha |
| P003 | Maíz | 5 ha |
| P004 | Fresas | 1 ha |
| P005 | Pimientos | 2 ha |

**Dispositivos simulados:** dos sensores de suelo por parcela, un dispositivo de monitorización del riego por parcela y una estación meteorológica compartida. Cada dispositivo tendrá un `device_id` estable y único durante la simulación.

## 3. Datos que debe generar el programa

Los siguientes rangos son **orientativos para la simulación**, no límites agronómicos universales.

### 3.1. Suelo

| Campo | Descripción | Rango orientativo |
| --- | --- | --- |
| `soil_moisture_pct` | Humedad volumétrica del suelo | 10–45 % |
| `soil_temperature_c` | Temperatura del suelo | 8–35 °C |
| `soil_ph` | pH del suelo | 5,5–8,0 |
| `soil_ec_ds_m` | Conductividad eléctrica | 0,2–3,0 dS/m |

### 3.2. Meteorología

| Campo | Descripción | Rango orientativo |
| --- | --- | --- |
| `air_temperature_c` | Temperatura del aire | 5–40 °C |
| `air_humidity_pct` | Humedad relativa del aire | 20–95 % |
| `solar_radiation_w_m2` | Radiación solar | 0–1.100 W/m² |
| `rainfall_mm` | Precipitación durante el intervalo | 0–15 mm |
| `wind_speed_ms` | Velocidad del viento | 0–15 m/s |

### 3.3. Riego

Registrar como mínimo: `irrigation_status` (`ON` / `OFF`), `valve_status` (`OPEN` / `CLOSED`), `water_flow_l_min`, `water_pressure_bar`, `water_volume_l` (volumen acumulado) y motivo de activación/desactivación cuando corresponda.

### 3.4. Estado de dispositivos

Registrar el identificador del dispositivo, el porcentaje de batería (`battery_pct`) y su estado operativo cuando aplique.

### 3.5. Diccionario de datos: ¿qué significa cada campo?

**Debes comprender el significado de los datos que genera tu simulador.** Utiliza esta referencia para interpretar las lecturas y justificar las reglas de comportamiento implementadas.

**Sensores del suelo**

| Campo | Significado |
| --- | --- |
| `soil_moisture_pct` | Porcentaje de volumen de agua presente en el suelo. Ayuda a decidir cuándo regar; no equivale directamente al porcentaje de agua disponible para la planta. |
| `soil_temperature_c` | Temperatura del suelo en grados Celsius; influye en la actividad de las raíces y los procesos biológicos. |
| `soil_ph` | Grado de acidez o alcalinidad del suelo; es una escala sin unidades que influye en la disponibilidad de nutrientes. |
| `soil_ec_ds_m` | Conductividad eléctrica del suelo, medida en decisiemens por metro; sirve como indicador indirecto de sales disueltas. |
| `sensor_depth_cm` | Profundidad, en centímetros, a la que está instalado el sensor de suelo. |

**Estación meteorológica**

| Campo | Significado |
| --- | --- |
| `air_temperature_c` | Temperatura del aire en grados Celsius. |
| `air_humidity_pct` | Humedad relativa del aire: cantidad de vapor de agua respecto al máximo que podría contener a esa temperatura, expresada en porcentaje. |
| `solar_radiation_w_m2` | Potencia de radiación solar que llega a una superficie por metro cuadrado; ayuda a simular el ciclo diurno. |
| `rainfall_mm` | Cantidad de lluvia acumulada durante el intervalo de medición, en milímetros; **no** es necesariamente el total del día. |
| `wind_speed_ms` | Velocidad del viento en metros por segundo; puede influir en la evaporación. |

**Monitorización del riego**

| Campo | Significado |
| --- | --- |
| `irrigation_status` | Indica si el sistema de riego está activado (`ON`) o desactivado (`OFF`). |
| `valve_status` | Indica si la válvula que permite el paso del agua está abierta (`OPEN`) o cerrada (`CLOSED`). |
| `water_flow_l_min` | Caudal: litros de agua que circulan por minuto en el instante de la lectura. |
| `water_pressure_bar` | Presión del agua en la conducción, medida en bares. |
| `water_volume_l` | Volumen total de agua registrado por el dispositivo, en litros, acumulado desde el inicio de la simulación o desde un reinicio expresamente documentado. |
| `reason` | Motivo del evento de riego, por ejemplo, humedad baja (`LOW_SOIL_MOISTURE`) o telemetría periódica (`PERIODIC_TELEMETRY`). |

**Identificación, estado del dispositivo y alertas**

| Campo | Significado |
| --- | --- |
| `event_id` | Identificador único de cada evento generado, incluso si procede del mismo sensor. |
| `timestamp` | Momento al que corresponde el evento, expresado en fecha y hora UTC con formato ISO 8601. |
| `device_id` | Identificador del sensor, estación meteorológica o controlador de riego que origina el evento. |
| `event_type` | Categoría del evento: lectura de suelo, meteorología, riego o alerta. |
| `farm_id` | Identificador de la explotación agrícola simulada. |
| `plot_id` | Identificador de la parcela a la que corresponde el evento; puede no aparecer en datos de la estación compartida. |
| `crop` | Cultivo asociado a la parcela (por ejemplo, tomates o maíz). |
| `measurements` | Objeto JSON que agrupa las variables medidas por un sensor. |
| `battery_pct` | Porcentaje estimado de batería restante del dispositivo. |
| `device_status` | Estado operativo del dispositivo, por ejemplo, `OK` o `ERROR`. |
| `alert_code` | Código que identifica la anomalía detectada. |
| `severity` | Gravedad de la alerta, por ejemplo, `WARNING` o `CRITICAL`. |
| `measured_value` | Valor observado que ha provocado la alerta; su unidad depende de la variable que se esté supervisando. |
| `threshold_value` | Umbral con el que se compara el valor observado; utiliza la misma unidad que `measured_value`. |
| `message` | Explicación legible de la situación que ha provocado la alerta. |

**Nota:** los valores son sintéticos y los rangos son didácticos. No interpretes los umbrales como recomendaciones agronómicas para explotaciones reales.

## 4. Formato obligatorio de los eventos

**Todos los eventos, sin excepción, deben contener estos tres campos:**

1. **`event_id`**: identificador único del evento; no se puede reutilizar entre registros de una misma ejecución y debe evitar colisiones entre ejecuciones.
2. **`timestamp`**: fecha y hora UTC del evento en formato ISO 8601, por ejemplo, `2026-10-13T10:00:00Z`.
3. **`device_id`**: identificador del dispositivo físico simulado que originó el evento. En alertas, utilizar el identificador del dispositivo afectado o del controlador que detectó la incidencia.
Además, cada evento debe incluir `event_type`, `farm_id` y, cuando corresponda, `plot_id`.

### Tipos de eventos obligatorios

- `sensor_reading`: medidas del suelo.
- `weather_reading`: medidas de la estación meteorológica.
- `irrigation_event`: cambios de estado o telemetría del riego.
- `alert`: anomalías detectadas.
### Ejemplo orientativo de un evento JSON

```json
{
  "event_id": "1b11cf5e-c5c2-4d07-8c26-312b2e124985",
  "timestamp": "2026-10-13T10:00:00Z",
  "device_id": "SOIL-P001-01",
  "event_type": "sensor_reading",
  "farm_id": "FARM001",
  "plot_id": "P001",
  "measurements": {
    "soil_moisture_pct": 24.8,
    "soil_temperature_c": 21.5,
    "soil_ph": 6.5,
    "soil_ec_ds_m": 1.2
  },
  "battery_pct": 92
}
```

**Nota:** el ejemplo define un esquema, pero no constituye la solución del simulador. **Debes definir** los esquemas específicos de los cuatro tipos de eventos y documentarlos en el `README.md`.

## 5. Reglas de simulación y coherencia

**No se aceptará** un generador que produzca únicamente números aleatorios independientes. Debe mantener un estado por parcela y simular cambios coherentes a lo largo del tiempo.

Implementar, como mínimo:

4. **Ciclo diario:** la temperatura y la radiación solar varían entre el día y la noche; la radiación nocturna debe aproximarse a cero.
5. **Suelo sin aportes:** si no llueve ni se riega, su humedad tenderá a disminuir gradualmente.
6. **Riego:** cuando la humedad cae por debajo del umbral configurado para la parcela, el riego puede activarse; debe poder detenerse al alcanzar otro umbral o una condición de seguridad.
7. **Efecto del riego:** la humedad del suelo aumenta progresivamente después de recibir agua.
8. **Lluvia:** las precipitaciones influyen en la humedad del suelo.
9. **Coherencia de dispositivos:** los caudales, estados de válvula y valores acumulados deben ser compatibles entre sí, salvo anomalías simuladas expresamente.
Los umbrales y las reglas deben estar documentados y ser configurables. Es aceptable utilizar aproximaciones didácticas, sin pretender reproducir un modelo agronómico científico completo.

## 6. Anomalías obligatorias

El simulador debe poder generar y registrar eventos `alert` para, al menos:

| Código | Situación |
| --- | --- |
| `LOW_SOIL_MOISTURE` | Humedad demasiado baja |
| `HIGH_TEMPERATURE` | Temperatura excesiva |
| `IRRIGATION_FAILURE` | Riego activado sin caudal esperado |
| `WATER_LEAK` | Flujo inesperado con válvula cerrada |
| `SENSOR_FAILURE` | Lecturas defectuosas o ausencia de lectura |

Cada alerta deberá incluir `event_id`, `timestamp`, `device_id`, `event_type`, código de anomalía, nivel de gravedad y una descripción comprensible.

## 7. Ejecución del simulador

El programa debe ofrecer:

- **Modo continuo:** genera eventos hasta que el usuario lo detiene de forma controlada.
- **Modo limitado:** se detiene tras un tiempo o cantidad de eventos configurada.
- **Intervalo de generación configurable**, expresado en segundos.
- **Semilla aleatoria configurable** para reproducir los escenarios sintéticos (sin sacrificar la unicidad de `event_id`).
- **Salida configurable** para indicar dónde almacenar los archivos.
Parámetros de referencia (puedes diseñar tu propio sistema de configuración):

```python
NUM_PLOTS = 5
GENERATION_INTERVAL = 5
SIMULATION_DURATION = 300
RANDOM_SEED = 42
ENABLE_ANOMALIES = True
OUTPUT_DIR = "data"
```

**Ampliación recomendada (opcional):** generación acelerada de varios días históricos mediante un reloj virtual, conservando marcas temporales coherentes.

## 8. Almacenamiento local obligatorio

Guardar los datos en formato **JSON Lines (****`.jsonl`****)**: **un objeto JSON por línea**, codificado en UTF-8. Los archivos deberán poder ampliarse sin sobrescribir los registros anteriores.

Se crearán, como mínimo, los siguientes archivos:

```plain text
data/
├── sensor_readings.jsonl
├── weather_readings.jsonl
├── irrigation_events.jsonl
└── alerts.jsonl
```

No es necesario crear un archivo por evento. Cada archivo agrupará registros del mismo tipo. **Debes verificar** que los archivos puedan leerse línea por línea y que cada línea sea un objeto JSON válido.

## 9. Ejemplos de salida esperada de los datos generados

Los siguientes ejemplos muestran **la estructura esperada de los cuatro archivos JSONL**. Son muestras ilustrativas; los valores, identificadores y fechas deben generarse desde Python, no copiarse como datos fijos. Cada línea de un archivo debe contener un objeto JSON completo y válido. Los objetos de los distintos tipos de eventos pueden tener campos diferentes.

### 9.1. `sensor_readings.jsonl` — sensores de suelo

```json
{"event_id":"9edb99c2-0e44-4874-bd02-0b483d0e15a1","timestamp":"2026-10-13T10:00:00Z","device_id":"SOIL-P001-01","event_type":"sensor_reading","farm_id":"FARM001","plot_id":"P001","crop":"tomates","sensor_depth_cm":20,"measurements":{"soil_moisture_pct":24.8,"soil_temperature_c":21.5,"soil_ph":6.5,"soil_ec_ds_m":1.2},"battery_pct":92,"device_status":"OK"}
{"event_id":"63e5875e-7d67-4bf9-8e8e-b532655a8ab1","timestamp":"2026-10-13T10:00:05Z","device_id":"SOIL-P002-01","event_type":"sensor_reading","farm_id":"FARM001","plot_id":"P002","crop":"lechugas","sensor_depth_cm":20,"measurements":{"soil_moisture_pct":33.1,"soil_temperature_c":19.4,"soil_ph":6.8,"soil_ec_ds_m":0.9},"battery_pct":85,"device_status":"OK"}
```

### 9.2. `weather_readings.jsonl` — estación meteorológica

```json
{"event_id":"a7eeea1f-d19f-44c7-8854-8a93a57e8bd6","timestamp":"2026-10-13T10:00:00Z","device_id":"WEATHER-001","event_type":"weather_reading","farm_id":"FARM001","measurements":{"air_temperature_c":27.8,"air_humidity_pct":48.2,"solar_radiation_w_m2":720.0,"rainfall_mm":0.0,"wind_speed_ms":3.2},"battery_pct":96,"device_status":"OK"}
{"event_id":"8b67fa0e-e95d-4a2d-afbb-6c0081b0a55d","timestamp":"2026-10-13T10:00:30Z","device_id":"WEATHER-001","event_type":"weather_reading","farm_id":"FARM001","measurements":{"air_temperature_c":28.0,"air_humidity_pct":47.8,"solar_radiation_w_m2":735.0,"rainfall_mm":0.0,"wind_speed_ms":3.4},"battery_pct":96,"device_status":"OK"}
```

### 9.3. `irrigation_events.jsonl` — sistema de riego

```json
{"event_id":"8422888b-6710-412b-a58e-157723c42a5b","timestamp":"2026-10-13T10:01:00Z","device_id":"IRR-P001-01","event_type":"irrigation_event","farm_id":"FARM001","plot_id":"P001","irrigation_status":"ON","valve_status":"OPEN","water_flow_l_min":35.5,"water_pressure_bar":2.3,"water_volume_l":0.0,"reason":"LOW_SOIL_MOISTURE","battery_pct":91,"device_status":"OK"}
{"event_id":"3c69855e-e595-4277-bda0-9d93c1e0e2c4","timestamp":"2026-10-13T10:02:00Z","device_id":"IRR-P001-01","event_type":"irrigation_event","farm_id":"FARM001","plot_id":"P001","irrigation_status":"ON","valve_status":"OPEN","water_flow_l_min":35.5,"water_pressure_bar":2.3,"water_volume_l":35.5,"reason":"PERIODIC_TELEMETRY","battery_pct":91,"device_status":"OK"}
```

### 9.4. `alerts.jsonl` — alertas y anomalías

```json
{"event_id":"c2d1a6a5-f0ec-41b7-8303-a184cc78b20f","timestamp":"2026-10-13T10:00:55Z","device_id":"SOIL-P001-01","event_type":"alert","farm_id":"FARM001","plot_id":"P001","alert_code":"LOW_SOIL_MOISTURE","severity":"WARNING","measured_value":18.5,"threshold_value":20.0,"message":"Humedad del suelo por debajo del umbral"}
{"event_id":"6d223313-cf08-4172-8867-0ba892e37bca","timestamp":"2026-10-13T10:03:00Z","device_id":"IRR-P003-01","event_type":"alert","farm_id":"FARM001","plot_id":"P003","alert_code":"IRRIGATION_FAILURE","severity":"CRITICAL","measured_value":0.0,"threshold_value":5.0,"message":"Riego activado sin caudal suficiente"}
```

**Debes generar todos los campos de medición definidos en la sección 3** y todos los campos comunes obligatorios de la sección 4. Las muestras anteriores representan distintas lecturas y situaciones, no una secuencia completa de simulación. Los eventos de riego y las alertas pueden producirse en momentos distintos de las lecturas periódicas.

## 10. Repositorio GitHub obligatorio

**Debes crear tu propio repositorio GitHub** para desarrollar y entregar la práctica. Nombre sugerido: `smart-agriculture-iot-simulator`.

Estructura propuesta:

```plain text
smart-agriculture-iot-simulator/
├── main.py
├── config.py
├── sensors.py
├── weather.py
├── irrigation.py
├── anomalies.py
├── data_writer.py
├── data/
│   ├── sensor_readings.jsonl
│   ├── weather_readings.jsonl
│   ├── irrigation_events.jsonl
│   └── alerts.jsonl
├── docs/
│   └── tutorial.md
├── tests/
├── requirements.txt
├── .gitignore
└── README.md
```

Se admite una estructura modular alternativa si está justificada.

**Requisitos Git/GitHub:**

10. Crear el repositorio y subir el proyecto mediante commits descriptivos.
11. Realizar **al menos cinco commits significativos** a lo largo del desarrollo; no subir todo únicamente al final.
12. Incorporar un `README.md` con descripción, requisitos, instalación, ejecución, estructura del proyecto, esquemas de eventos y explicación de las reglas.
13. **Debes elaborar un tutorial propio, paso a paso, que explique cómo desarrollaste el script Python**: preparación del entorno, creación de módulos, generación de datos, implementación de reglas, escritura de JSONL, ejecución y comprobación de resultados. Incluye fragmentos de tu código, comandos y capturas o ejemplos reales de salida. Publica el tutorial en `docs/tutorial.md` y enlázalo desde el `README.md`.
14. Añadir `requirements.txt` con las dependencias necesarias (si solo se utiliza la biblioteca estándar, indicarlo).
15. Configurar `.gitignore` para excluir entornos virtuales, cachés, logs y archivos grandes de ejecución.
16. Incluir en `data/` **una muestra pequeña y representativa** de los cuatro tipos de eventos, sin subir datasets voluminosos.
17. No publicar secretos, credenciales, tokens ni información sensible.
**Nota:** el repositorio podrá ser privado si así lo indica el profesor; en ese caso **debes facilitar acceso al docente** para la evaluación.

## 11. Pruebas de validación

**Debes ejecutar y documentar** las siguientes pruebas:

| Prueba | Resultado esperado |
| --- | --- |
| Validación JSONL | Todas las líneas son JSON válido |
| Campos obligatorios | Todos los eventos contienen `event_id`, `timestamp`, `device_id` |
| Unicidad | No hay `event_id` duplicados |
| Tiempos | Los `timestamp` se expresan en UTC e ISO 8601 |
| Dispositivos | Los identificadores se corresponden con dispositivos simulados |
| Comportamiento temporal | La temperatura y la humedad evolucionan coherentemente |
| Riego y lluvia | Las condiciones del suelo responden a los aportes de agua |
| Anomalías | Se generan y almacenan alertas identificables |
| Modo limitado | Finaliza automáticamente según la configuración |
| Modo continuo | Puede detenerse sin corromper los archivos |
| Volumen mínimo | Se generan **al menos 1.000 eventos en total** en una ejecución de prueba |

Se valorará positivamente implementar pruebas automatizadas, por ejemplo, con `unittest` o `pytest`.

