# 📋Clase 06 - Fundamentos de Cloud Computing

> Esta clase recorre los fundamentos del cloud computing (modelos de despliegue, beneficios y modelos de servicio) pero siempre desde la perspectiva de quien diseña, construye y opera plataformas de datos en Microsoft Azure.

---

## Sección 1: La nube desde la mirada del ingeniero de datos

### ¿Qué es la computación en la nube?

> La computación en la nube permite **acceder a recursos tecnológicos (almacenamiento, cómputo, bases de datos, redes, analítica) a través de internet**, sin necesidad de poseer ni mantener la infraestructura física. El proveedor, en nuestro caso Microsoft, es dueño de los centros de datos; nosotros consumimos sus capacidades como un servicio.

### ¿Qué cambia para un ingeniero de datos?

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-001.png)

### Mapa de servicios de Azure para ingeniería de datos

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-002.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-003.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-004.png)

## Sección 2: Nube pública vs. nube privada

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-005.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-006.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-007.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-008.png)

---

## Sección 3: Nube híbrida y multinube

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-009.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-010.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-011.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-012.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-013.png)

---

## Sección 4: Comparativa de modelos y cómo elegir

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-014.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-015.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-016.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-017.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-018.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-019.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-020.png)

---

## Sección 6: Modelos de servicio — IaaS, PaaS y SaaS en el ecosistema de datos

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-021.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-022.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-023.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-024.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-025.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-026.png)

![Imagen](images/clase-06-fundamentos-de-cloud-computing/image-027.png)

---

# 2. Primeros pasos en Azure

## 1. Verificación por Entorno Gráfico (Portal de Azure)

> El portal permite verificar las regiones autorizadas por directivas de la suscripción estudiantil y comprobar la disponibilidad de servicios específicos antes de desplegar.

### Paso A: Ver las regiones autorizadas por Azure Policy

> Las cuentas de estudiante restringen el despliegue a un subconjunto de regiones mediante directivas automáticas.
1. Inicia sesión en [Azure Portal](https://portal.azure.com/) con tu cuenta institucional.
2. En la barra de búsqueda superior, escribe **Directiva** (o **Policy**) y entra en el servicio.
3. En el menú lateral izquierdo, ve a la sección **Creación** (*Authoring*) y haz clic en **Asignaciones** (*Assignments*).
4. Localiza una directiva con el nombre **“Allowed resource deployment regions”** o **“Allowed locations”** y haz clic sobre ella.
5. Abre la pestaña **Parámetros** (*Parameters*): allí verás el listado de regiones exactas (por ejemplo: `spaincentral`, `germanywestcentral`, `westeurope`, `eastus`) donde tu cuenta tiene permiso para crear infraestructura.

### Paso B: Comprobar la disponibilidad real de un servicio

>
1. Escribe en el buscador el servicio que deseas crear:
- **Azure Data Factory**
- **Azure Databricks**
- **Máquinas virtuales**
2. Haz clic en **Crear**.
3. En la pestaña **Aspectos básicos** (*Basics*), localiza el desplegable **Región** (*Region*).
4. Selecciona una de las regiones autorizadas obtenidas en el Paso A.
5. Si la región seleccionada no admite el servicio para tu suscripción o tiene agotada la cuota, el asistente mostrará un mensaje de advertencia o bloqueará el avance antes de la validación final.

---

## 2. Verificación por Consola (Azure CLI)

> Puedes ejecutar estos comandos en **Azure Cloud Shell** (el icono `>_` en la barra superior del portal) o directamente en la terminal de VS Code en Windows.

### Paso 1: Listar las regiones globales admitidas por tu suscripción

Muestra todas las regiones de Azure donde tu suscripción puede operar:

```bash
az account list-locations --query "[].{Region:name, Display:displayName}" -o table
```

### Paso 2: Filtrar las regiones autorizadas por Azure Policy

Para cuentas de estudiante con directivas activas, este comando extrae únicamente los identificadores técnicos de las zonas permitidas:

```powershell
az policy assignment list --query "[?contains(name, 'Allowed') || contains(displayName, 'Allowed')].parameters.listOfAllowedLocations.value" -o tsv
```

### Paso 3: Verificar si un servicio específico está disponible en una región

Cada servicio pertenece a un proveedor de recursos (*Resource Provider*).

- **Para Azure Data Factory (****`Microsoft.DataFactory`****):**
Comprueba las regiones donde está disponible el recurso `factories`:
```sql
az provider show --namespace Microsoft.DataFactory --query "resourceTypes[?resourceType=='factories'].locations | [0]" -o tsv
```
- **Para Azure Databricks (****`Microsoft.Databricks`****):**
Comprueba las regiones donde está disponible el recurso `workspaces`:
```sql
az provider show --namespace Microsoft.Databricks --query "resourceTypes[?resourceType=='workspaces'].locations | [0]" -o tsv
```
- **Para Máquinas Virtuales (****`Microsoft.Compute`****):**
Comprueba los tamaños de máquina virtual (SKUs) disponibles y sin restricciones de cuota en una región específica (por ejemplo, `spaincentral`):
```sql
az vm list-skus --location spaincentral --resource-type virtualMachines --query "[?restrictions[0].reasonCode == null].{Size:name, Cores:capabilities[?name=='vCPUs'].value | [0]}" -o table
```
---

## 3. Conexión y Uso de Azure CLI en Visual Studio Code (Windows)

> Configurar Azure CLI en Visual Studio Code para Windows te permite desplegar y validar recursos desde la terminal integrada de PowerShell o Command Prompt.

### Paso 1: Instalar Azure CLI en Windows

Puedes instalar la CLI de Azure mediante cualquiera de estas dos opciones:

- **Opción A (Recomendada - PowerShell):** Abre PowerShell como Administrador y ejecuta:
```powershell
winget install -e --id Microsoft.AzureCLI
```
- **Opción B (Instalador manual):** Descarga y ejecuta el instalador MSI para Windows de 64 bits desde el sitio oficial de Microsoft Learn.
> **Nota:** Tras la instalación, reinicia VS Code si ya lo tenías abierto para que detecte las nuevas variables de entorno del sistema.

### Paso 2: Extensiones recomendadas en VS Code (Opcional)

1. En VS Code, abre la vista de **Extensiones** pulsando `Ctrl + Shift + X`.
2. Busca e instala **Azure Tools** (Microsoft) o **Azure Account**.
3. Estas extensiones habilitan la exploración visual de tus grupos de recursos y suscripciones en la barra lateral.
### Paso 3: Abrir la terminal integrada y verificar la herramienta

4. Abre VS Code en Windows.
5. Abre la terminal integrada con el atajo de teclado: `Ctrl + `` (o menú superior *Terminal -> New Terminal*).
6. Asegúrate de que el shell sea PowerShell o Command Prompt y ejecuta:
```powershell
az version
```

Verás la versión de Azure CLI instalada y sus componentes.
### Paso 4: Iniciar sesión con tu cuenta institucional

7. En la terminal de VS Code, ejecuta:
```powershell
az login
```
8. Se abrirá automáticamente una pestaña en tu navegador web predeterminado.
9. Inicia sesión con tus credenciales de **correo de estudiante** asociadas al crédito de Azure for Students.
10. Una vez completado el inicio de sesión en el navegador, vuelve a VS Code. La terminal mostrará un resumen en formato JSON con la suscripción conectada.
### Paso 5: Fijar la suscripción activa de estudiante

Si tienes más de una suscripción vinculada a tu cuenta, debes asegurarte de que la CLI apunta a tu suscripción de estudiante:

11. Lista tus suscripciones:
```powershell
az account list --output table
```
12. Establece como activa la suscripción de estudiante:
```powershell
az account set --subscription "Azure for Students"
```
13. Comprueba que está correctamente seleccionada:
```powershell
az account show --output table
```
A partir de este punto, puedes ejecutar directamente en la terminal de VS Code todos los comandos de comprobación de directivas, Azure Data Factory y Azure Databricks detallados en la **Sección 2**.

# 3. Ejercicios básicos en Azure

## Facturación, costes, presupuestos, alertas y FinOps

Esta guía reúne diez ejercicios prácticos para aprender a supervisar y controlar los costes en Microsoft Azure.

Está diseñada para comenzar con una cuenta en la que:

- No hay Resource Groups creados.
- No hay servicios desplegados.
- No hay máquinas virtuales.
- No hay bases de datos.
- No hay cuentas de almacenamiento.
- No existe todavía consumo significativo.
> **Importante:** el objetivo de estos ejercicios no es generar gasto para comprobar las alertas, sino aprender a preparar los mecanismos de control antes de desplegar recursos.

La secuencia general será:

```plain text
EXPLORAR
   ↓
ESTIMAR
   ↓
ORGANIZAR
   ↓
PRESUPUESTAR
   ↓
ALERTAR
   ↓
MONITORIZAR
   ↓
LIMPIAR
```

---

---

# Ejercicio 1. Explorar Cost Management

## Objetivo

Aprender a localizar la información de facturación y costes de una suscripción y conocer las principales vistas disponibles en **Microsoft Cost Management**.

## Escenario

Acabas de recibir acceso a una cuenta de Azure que todavía no contiene recursos. Antes de comenzar a crear servicios quieres saber dónde se controlarán los costes.

## Paso 1. Acceder a Azure Portal

14. Abre un navegador.
15. Accede a:
`https://portal.azure.com`
16. Inicia sesión con tu cuenta de Azure.
## Paso 2. Abrir Cost Management

17. En la barra de búsqueda superior escribe:
`Cost Management`
18. Selecciona **Cost Management + Billing**.
19. Localiza el ámbito o **Scope** correspondiente a tu suscripción.
20. Selecciona la suscripción con la que vas a trabajar.
## Paso 3. Abrir Cost Analysis

21. Selecciona **Cost analysis**.
22. Observa el periodo seleccionado.
23. Revisa la información disponible.
Como todavía no existen recursos, puedes encontrar:

```plain text
0 € de consumo
```

o:

```plain text
No hay datos suficientes
```

o una vista sin costes significativos.

Esto es normal.

## Paso 4. Explorar las opciones de análisis

Busca la opción:

**Group by**

Observa, cuando estén disponibles:

- **Service name**
- **Resource**
- **Resource group name**
- **Location**
- **Tag**
No necesitas obtener resultados todavía.

El objetivo es aprender dónde se encuentran estas opciones.

## Paso 5. Registrar el estado inicial

Completa:

| Dato | Valor |
| --- | --- |
| Fecha de consulta |  |
| Periodo |  |
| Coste mostrado |  |
| Moneda |  |
| ¿Existen recursos con consumo? | Sí / No |

## Comprobación

Debes saber localizar:

```plain text
Azure Portal
   ↓
Cost Management
   ↓
Cost analysis
```

## Preguntas

24. ¿Por qué es útil conocer Cost Management antes de desplegar recursos?
25. ¿Qué información esperas encontrar aquí después de realizar futuros laboratorios?
---

# Ejercicio 2. Crear un Budget

## Objetivo

Crear un presupuesto mensual antes de desplegar recursos.

## Escenario

Se establece un presupuesto de referencia de:

```plain text
30 € al mes
```

No es necesario consumir esa cantidad.

## Paso 1. Abrir Budgets

26. Abre **Cost Management**.
27. Selecciona el ámbito de la suscripción.
28. Abre **Budgets**.
## Paso 2. Crear un nuevo Budget

Selecciona **Add** o la opción equivalente.

Configura:

```plain text
Name: Budget-Laboratorio-Azure
Period: Monthly
Amount: 30 €
```

Si tu suscripción utiliza otra moneda, utiliza el valor equivalente indicado por el profesor.

## Paso 3. Revisar el ámbito

Como todavía no tienes Resource Groups con servicios desplegados, crea el Budget a nivel de la **suscripción**, salvo que el profesor indique otro ámbito.

## Paso 4. Guardar

Completa la configuración requerida y guarda el Budget.

## Paso 5. Verificar

Comprueba que aparece en **Budgets**.

## Concepto importante

Un Budget:

```plain text
MONITORIZA
      +
GENERA AVISOS
```

pero no debe interpretarse como:

```plain text
LÍMITE AUTOMÁTICO QUE APAGA AZURE
```

## Comprobación

Debes poder localizar un Budget mensual de 30 €.

## Preguntas

29. ¿Qué diferencia existe entre un Budget y el gasto real?
30. ¿Por qué es útil crear el Budget antes de desplegar recursos?
---

# Ejercicio 3. Crear alertas de presupuesto

## Objetivo

Configurar una notificación para recibir un aviso antes de alcanzar el presupuesto.

## Paso 1. Abrir el Budget

Accede a:

```plain text
Cost Management
   ↓
Budgets
   ↓
Budget-Laboratorio-Azure
```

## Paso 2. Crear una condición

Configura:

```plain text
Type: Actual
Threshold: 80 %
```

Con un Budget de 30 €:

```plain text
30 € × 0,80 = 24 €
```

## Paso 3. Configurar una notificación

Introduce el correo indicado para el laboratorio.

## Paso 4. Guardar

Guarda la configuración.

## Paso 5. Comprender lo que ocurrirá

No debes intentar gastar 24 € para probar la alerta.

La configuración queda preparada para futuros consumos.

## Comprobación

El Budget debe contener al menos una condición de alerta.

## Preguntas

31. ¿Por qué puede ser útil avisar al 80 %?
32. ¿Qué revisarías si recibieras ese aviso?
---

# Ejercicio 4. Simular umbrales 50/80/100 %

## Objetivo

Preparar un sistema de avisos progresivos.

## Escenario

Utiliza:

```plain text
Budget = 30 €
```

Calcula:

| Umbral | Importe |
| --- | --- |
| 50 % | 15 € |
| 80 % | 24 € |
| 100 % | 30 € |

## Paso 1. Configurar el 50 %

```plain text
Type: Actual
Threshold: 50 %
```

## Paso 2. Configurar el 80 %

```plain text
Type: Actual
Threshold: 80 %
```

## Paso 3. Configurar el 100 %

```plain text
Type: Actual
Threshold: 100 %
```

## Paso 4. Preparar una alerta Forecasted

Si tu cuenta muestra la opción:

```plain text
Type: Forecasted
Threshold: 100 %
```

Si no aparece, continúa con el ejercicio.

## Paso 5. Resolver un caso simulado

Supón:

```plain text
Budget = 30 €
Actual Cost = 18 €
Forecasted Cost = 34 €
```

Responde:

33. ¿Se ha superado actualmente el Budget?
34. ¿Existe riesgo de superarlo?
35. ¿Qué dato permite anticiparse mejor?
## Comprobación

Debes comprender:

```plain text
Actual Cost     = gasto ya generado
Forecasted Cost = previsión de gasto
```

---

# Ejercicio 5. Preparar el análisis por Resource Group

## Objetivo

Comprender cómo se utilizarán los Resource Groups para organizar los laboratorios y analizar sus costes cuando exista consumo.

## Escenario

La cuenta no contiene ningún Resource Group.

Vas a crear uno vacío para aprender a organizar los futuros recursos.

## Paso 1. Abrir Resource Groups

Busca:

```plain text
Resource groups
```

## Paso 2. Crear un Resource Group

Selecciona:

```plain text
Create
```

Utiliza:

```plain text
rg-ejercicios-costes
```

Selecciona una región disponible, por ejemplo:

```plain text
West Europe
```

y crea el Resource Group.

## Paso 3. Verificar que está vacío

Abre:

```plain text
rg-ejercicios-costes
```

Comprueba que no contiene servicios desplegados.

## Paso 4. Volver a Cost Analysis

Accede a:

```plain text
Cost Management
   ↓
Cost analysis
```

Busca:

```plain text
Group by → Resource group name
```

Como el Resource Group está vacío y no genera consumo, puede no aparecer en los datos de costes.

Esto es correcto.

## Paso 5. Comprender el uso futuro

En próximos laboratorios podrás tener:

```plain text
rg-practica05
rg-practica06
rg-practica07
```

y utilizar Cost Analysis para comparar el consumo asociado a cada grupo cuando sus recursos hayan generado costes.

## Comprobación

Debes poder explicar:

```plain text
Suscripción
   ↓
Resource Group
   ↓
Recursos
   ↓
Consumo
   ↓
Costes
```

## Preguntas

36. ¿Por qué conviene utilizar un Resource Group distinto para laboratorios independientes?
37. ¿Por qué el Resource Group vacío no muestra necesariamente un coste en Cost Analysis?
---

# Ejercicio 6. Preparar una estrategia de Tags

## Objetivo

Aprender a utilizar Tags para clasificar recursos y comprender cómo se utilizarán posteriormente en el análisis de costes.

## Paso 1. Abrir el Resource Group

Abre:

```plain text
rg-ejercicios-costes
```

## Paso 2. Añadir Tags al Resource Group

Añade:

| Tag | Valor |
| --- | --- |
| Project | AzureLabs |
| Department | Data |
| Environment | Lab |
| Owner | AlumnoXX |

Sustituye `AlumnoXX` por el identificador indicado.

## Paso 3. Comprender una limitación importante

Los Tags aplicados al Resource Group sirven para organizarlo, pero:

```plain text
LOS RECURSOS NO HEREDAN AUTOMÁTICAMENTE
LOS TAGS DEL RESOURCE GROUP
```

Por tanto, cuando en futuras prácticas crees recursos y quieras analizar sus costes mediante Tags, deberás:

```plain text
Aplicar también los Tags a los recursos
```

o utilizar mecanismos de herencia configurados expresamente cuando corresponda.

## Paso 4. Preparar la estrategia futura

En próximos laboratorios, los recursos podrán utilizar Tags como:

```plain text
Project = Practica05
Department = Data
Environment = Lab
Owner = Alumno07
```

## Paso 5. Explorar Cost Analysis

Abre:

```plain text
Cost Management
   ↓
Cost analysis
```

Busca las opciones relacionadas con **Tag**.

Como todavía no hay recursos con consumo etiquetado, no necesitas obtener resultados.

## Comprobación

Debes comprender la diferencia entre:

```plain text
Tag del Resource Group
```

y:

```plain text
Tag aplicado al recurso que genera consumo
```

## Preguntas

38. ¿Para qué pueden utilizarse los Tags?
39. ¿Un recurso hereda automáticamente los Tags de su Resource Group?
40. ¿Qué Tags utilizarías en todos los laboratorios del curso?
---

# Ejercicio 7. Usar Azure Pricing Calculator

## Objetivo

Estimar el coste de una solución sin desplegar recursos.

## Paso 1. Abrir Azure Pricing Calculator

Accede a:

`https://azure.microsoft.com/pricing/calculator/`

## Paso 2. Añadir Virtual Machines

Busca:

```plain text
Virtual Machines
```

Añade el producto a la estimación.

## Paso 3. Revisar

Observa:

- Región.
- Sistema operativo.
- Tamaño.
- Número de máquinas.
- Horas mensuales.
- Precio estimado.
## Paso 4. Comparar dos usos

### Escenario A

```plain text
730 horas/mes
```

### Escenario B

```plain text
8 horas/día × 22 días = 176 horas/mes
```

Completa:

| Escenario | Horas/mes | Coste estimado |
| --- | --- | --- |
| Todo el mes | 730 |  |
| Horario de trabajo | 176 |  |

## Paso 5. Cambiar una variable

Prueba a cambiar:

- Región.
- Tamaño de VM.
- Sistema operativo.
Observa si cambia el precio.

## Paso 6. Añadir otro servicio a la estimación

Puedes añadir **Storage** únicamente dentro de la calculadora.

No debes desplegarlo.

## Comprobación

Debes disponer de al menos dos estimaciones sin haber creado una VM real.

---

# Ejercicio 8. Comparar configuraciones de VM

## Objetivo

Comprender cómo el tamaño y el tiempo de utilización afectan al coste sin desplegar ninguna máquina virtual.

## Paso 1. Utilizar Azure Pricing Calculator

Crea tres estimaciones:

```plain text
VM A = tamaño pequeño
VM B = tamaño intermedio
VM C = tamaño mayor
```

Utiliza inicialmente:

```plain text
176 horas/mes
```

## Paso 2. Registrar

| VM | vCPU | RAM | 176 h/mes |
| --- | --- | --- | --- |
| A |  |  |  |
| B |  |  |  |
| C |  |  |  |

## Paso 3. Cambiar a 730 horas

Completa:

| VM | 176 h/mes | 730 h/mes | Diferencia |
| --- | --- | --- | --- |
| A |  |  |  |
| B |  |  |  |
| C |  |  |  |

## Paso 4. Analizar

Responde:

41. ¿Cuál tiene menor coste?
42. ¿Cuál tiene mayor coste?
43. ¿Qué efecto tiene aumentar las horas?
44. ¿Qué significa sobredimensionar una VM?
## Concepto

```plain text
Right-sizing
=
adaptar el tamaño del recurso
a la necesidad real
```

---

# Ejercicio 9. Aprender a detectar recursos innecesarios

## Objetivo

Preparar una rutina de revisión y limpieza antes de comenzar a crear servicios en los siguientes laboratorios.

## Escenario

Actualmente la cuenta no contiene servicios desplegados. Por tanto, este ejercicio se realiza mediante una **simulación**.

## Paso 1. Revisar la cuenta actual

Abre:

```plain text
Resource groups
```

Comprueba qué grupos existen.

Deberías encontrar únicamente los creados durante estos ejercicios, si todavía no los has eliminado.

## Paso 2. Simular un laboratorio futuro

Imagina que un Resource Group contiene:

```plain text
VM
Managed Disk
Public IP
Storage Account
Snapshot
```

El laboratorio ha terminado.

## Paso 3. Clasificar

Completa:

| Recurso | ¿Debo revisarlo al terminar? | Motivo |
| --- | --- | --- |
| VM | Sí | Puede generar coste |
| Managed Disk |  |  |
| Public IP |  |  |
| Storage Account |  |  |
| Snapshot |  |  |

## Paso 4. Aprender la regla de limpieza

Al finalizar un laboratorio:

```plain text
1. Abrir el Resource Group
2. Revisar todos sus recursos
3. Confirmar qué debe conservarse
4. Eliminar lo que ya no sea necesario
5. Revisar Cost Analysis
```

## Paso 5. Preparar el uso futuro de Auto-shutdown

Cuando más adelante trabajes con VMs, revisarás:

```plain text
Virtual Machine
   ↓
Operations
   ↓
Auto-shutdown
```

En este ejercicio no necesitas crear una VM para practicarlo.

## Comprobación

Debes poder identificar varios tipos de recursos que deben revisarse al finalizar un laboratorio.

## Preguntas

45. ¿Por qué no basta con apagar una única VM y olvidarse del resto de recursos?
46. ¿Por qué la limpieza forma parte del control de costes?
---

# Ejercicio 10. Diseñar una política FinOps básica

## Objetivo

Integrar los ejercicios anteriores en una pequeña política de trabajo sin desplegar servicios de pago.

## Escenario

Un equipo de DataNova Engineering dispone de:

```plain text
Presupuesto de referencia = 30 €/mes
```

Antes de iniciar futuros proyectos debe dejar preparado su procedimiento de control.

## Fase 1. Estimar

Utiliza **Azure Pricing Calculator** para preparar una estimación conceptual que incluya:

- Una VM.
- Storage.
- Otro servicio elegido por el profesor.
No despliegues esos servicios.

Completa:

| Servicio | Coste estimado |
| --- | --- |
| VM |  |
| Storage |  |
| Servicio adicional |  |
| **Total estimado** |  |

## Fase 2. Comprobar el presupuesto

Comprueba:

```plain text
Coste estimado ≤ 30 €
```

Si la estimación lo supera, modifica:

- Horas.
- Tamaño.
- Servicios incluidos.
## Fase 3. Organizar

Utiliza:

```plain text
rg-ejercicios-costes
```

o crea, si el profesor lo indica:

```plain text
rg-finops-lab
```

No es necesario desplegar servicios dentro.

## Fase 4. Etiquetar el Resource Group

Utiliza:

| Tag | Valor |
| --- | --- |
| Project | FinOpsLab |
| Department | Data |
| Environment | Lab |
| Owner | AlumnoXX |

Recuerda:

> Estos Tags no se copiarán automáticamente a futuros recursos.

Cuando haya recursos reales, deberán etiquetarse también si se quieren analizar costes por Tag.

## Fase 5. Verificar Budget y alertas

Comprueba que tienes:

```plain text
Budget = 30 €
Alertas = 50 %, 80 %, 100 %
```

y, cuando esté disponible:

```plain text
Forecasted = 100 %
```

## Fase 6. Explorar Cost Analysis

Localiza:

```plain text
Group by → Service name
Group by → Resource
Group by → Resource group name
Group by → Tag
```

No necesitas obtener costes reales todavía.

## Fase 7. Diseñar la política

Completa:

```plain text
ANTES:
Estimar y revisar Budget

DURANTE:
Organizar y etiquetar

AL FINAL:
Revisar Cost Analysis y limpiar
```

## Fase 8. Limpieza

Si el Resource Group está vacío y ya no será utilizado:

47. Ábrelo.
48. Confirma que no contiene recursos.
49. Elimínalo.
El profesor puede indicar que se conserve el Budget para los siguientes laboratorios.

## Entregable

| Elemento | Resultado |
| --- | --- |
| Budget de referencia | 30 € |
| Coste estimado |  |
| Alertas preparadas |  |
| Resource Group utilizado |  |
| Tags utilizados |  |
| Recursos de pago desplegados | Ninguno |
| ¿Se realizó limpieza? |  |

---

# Reto final

Imagina que en un futuro laboratorio aparecen estos datos:

```plain text
Budget = 30 €
Actual Cost = 21 €
Forecasted Cost = 35 €
```

Responde:

50. ¿Se ha superado ya el Budget?
51. ¿Existe riesgo de superarlo?
52. ¿Qué alertas deberían haberse activado?
53. ¿Qué revisarías primero en Cost Analysis?
54. ¿Qué harías con recursos que ya no sean necesarios?

