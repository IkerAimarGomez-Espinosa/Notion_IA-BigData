# 🧗🏽‍♂️ Practica 04 - Desplegar un recurso real y medir su coste en Azure

---

> 💡 Instrucciones de entrega:
- Enlace al repositorio de GitHub que contenga a detalle los pasos realizados mediante capturas de pantalla. La practica debe estar desarrollada en markdown y pdf

# Sección 1 — Escenario

> El equipo de analítica de **DataNova Engineering** necesita un servidor Linux temporal para probar unos scripts de ingesta antes de pasarlos a producción.
El responsable de plataforma acepta, pero con condiciones:

- El servidor solo puede existir durante la duración del laboratorio.
- Antes de crearlo hay que presentar una estimación de coste.
- El laboratorio tendrá su propio presupuesto, independiente del presupuesto general de 30 €.
- Todos los recursos deben estar etiquetados.
- Al terminar, hay que entregar un informe comparando **coste estimado** y **coste real**.

La práctica se desarrolla a lo largo de **tres días**, porque los datos de coste en Azure no aparecen al instante (suelen tardar entre 8 y 24 horas) y las alertas de presupuesto se evalúan periódicamente, no en tiempo real.

| Momento | Qué harás |
| --- | --- |
| Día 1 | Estimar, crear Resource Group, Budget, alertas y la máquina virtual |
| Día 2 | Primera revisión en Cost Analysis y experimento de apagado |
| Día 3 | Revisión final, comparación estimado vs. real y limpieza |

⚠️ **Coste esperado de la práctica:** entre 1 € y 3 € aproximadamente, siempre que sigas las instrucciones y elimines los recursos el Día 3. Muy por debajo del Budget general de 30 €.

---

# Sección 2 — El servicio elegido: una máquina virtual Linux

Vamos a desplegar una **Virtual Machine (VM) Linux** de tamaño pequeño. Es el servicio ideal para esta práctica por varios motivos:

- Se crea en pocos minutos desde el portal.
- Genera un coste **por hora**, fácil de relacionar con el Pricing Calculator que ya usaste en los ejercicios de la clase 6
- No es un único cargo: una VM crea **varios recursos** que se facturan por separado, lo que da mucho juego en Cost Analysis.
- Permite demostrar la diferencia entre **apagar** una VM y **desasignarla**, una de las fuentes de gasto inesperado más habituales en empresas.
## Qué genera coste en una VM

| Recurso creado | ¿Genera coste? | ¿Cuándo? |
| --- | --- | --- |
| Máquina virtual (cómputo) | Sí | Por hora, **solo mientras está asignada** (encendida o apagada desde el sistema operativo) |
| Disco del sistema operativo | Sí | **Siempre**, aunque la VM esté desasignada |
| IP pública (Standard) | Sí | **Siempre**, mientras exista |
| Interfaz de red (NIC) | No | — |
| Red virtual (VNet) | No | — |
| Network Security Group (NSG) | No | — |
| Tráfico de salida | Mínimo | Despreciable en esta práctica |

## Por qué usaremos el tamaño Standard_B2s

Algunas cuentas gratuitas o de estudiante incluyen horas gratis de ciertos tamaños muy pequeños (por ejemplo, **B1s**). Si usáramos uno de esos tamaños, Cost Analysis podría mostrar **0 €** y no podríamos medir nada.

Por eso utilizaremos:

```plain text
Standard_B2s  →  2 vCPU, 4 GB de RAM
```

Es un tamaño barato, pero con coste visible.

---

# Sección 3 — Estimar el coste antes de desplegar

## Paso 1. Abrir el Pricing Calculator

```plain text
https://azure.microsoft.com/pricing/calculator/
```

## Paso 2. Añadir los tres componentes con coste

Añade a la estimación:

```plain text
Virtual Machines
Managed Disks
IP Addresses
```

## Paso 3. Configurar la máquina virtual

| Opción | Valor |
| --- | --- |
| Region | West Europe (o la región que vayas a usar) |
| Operating system | Linux |
| Type | Ubuntu |
| Tier | Standard |
| Instance | B2s |
| Virtual machines | 1 |
| Savings options | Pay as you go |
| Horas | Ver escenarios del Paso 6 |

> En la sección de la VM, deja los discos gestionados a 0: los estimaremos en su propio apartado para verlos por separado.

## Paso 4. Configurar el disco

| Opción | Valor |
| --- | --- |
| Tier | Standard SSD |
| Disk size | E4 (32 GiB) |
| Redundancy | LRS |
| Disks | 1 |

💡 El portal propone por defecto **Premium SSD** para el disco del sistema operativo, que es bastante más caro. En esta práctica lo cambiaremos a **Standard SSD** al crear la VM. Es un buen ejemplo de cómo una opción por defecto puede encarecer un entorno de pruebas sin que nadie lo decida conscientemente.

## Paso 5. Configurar la IP pública

| Opción | Valor |
| --- | --- |
| Type | Standard (ARM) |
| IP address type | Static |
| Cantidad | 1 |

## Paso 6. Calcular tres escenarios

El laboratorio dura unas 72 horas desde que creas la VM hasta que la eliminas. Calcula el coste de la VM en cada escenario cambiando las horas en el calculador:

| Escenario | Horas de cómputo | Coste cómputo | Coste disco (72 h) | Coste IP (72 h) | Total estimado |
| --- | --- | --- | --- | --- | --- |
| A. VM encendida todo el laboratorio | 72 |  |  |  |  |
| B. VM con apagado automático nocturno (≈ 12 h/día) | 36 |  |  |  |  |
| C. VM encendida solo 8 h en total | 8 |  |  |  |  |

Para el disco y la IP, el calculador muestra coste mensual. Conviértelo a 72 horas:

```plain text
Coste 72 h = Coste mensual × 72 / 730
```

> Como referencia orientativa, una B2s Linux en West Europe ronda los **0,04–0,05 € por hora**. Los precios cambian y dependen de la región y la moneda: utiliza siempre el valor que te dé el calculador.

## Paso 7. Guardar la estimación

Guarda o exporta la estimación (botón **Export** o **Save**). La necesitarás el Día 3 para comparar.

---

# Sección 4 — Crear el Resource Group etiquetado

Si varios alumnos comparten la misma suscripción, los nombres deben ser únicos. Sustituye `XX` por tu número de alumno.

## Paso 1. Crear el Resource Group

En **Resource groups → Create**:

| Campo | Valor |
| --- | --- |
| Resource group | `rg-practica04-vm-alumnoXX` |
| Region | West Europe |

## Paso 2. Añadir Tags

En la pestaña **Tags**:

| Tag | Valor |
| --- | --- |
| Project | Practica04 |
| Department | Data |
| Environment | Lab |
| Owner | AlumnoXX |
| CostCenter | DataNova-Analytics |

Selecciona **Review + create** y después **Create**.

📌 Recuerda lo aprendido en la Práctica 04: los recursos que crees dentro del Resource Group **no heredan automáticamente** sus Tags. Por eso en la Sección 6 aplicaremos los Tags directamente a los recursos de la VM.

---

# Sección 5 — Crear un Budget específico para el laboratorio

En la clase 06 el Budget de 30 € estaba asociado a la suscripción y no se esperaba que las alertas llegaran a activarse. Ahora vamos a crear un Budget **pequeño y asociado solo al Resource Group**, para que las alertas sí se disparen con el gasto real del laboratorio.

## Paso 1. Abrir Budgets con el ámbito correcto

```plain text
Cost Management
      ↓
Scope → selecciona rg-practica04-vm-alumnoXX
      ↓
Budgets → Add
```

> Comprueba en la parte superior que el ámbito (**Scope**) es el Resource Group y no la suscripción. Es el error más frecuente en este paso.

## Paso 2. Definir el importe a partir de tu estimación

El importe del Budget será el **total estimado del Escenario B**, redondeado hacia arriba al euro.

```plain text
Ejemplo:
Total estimado Escenario B = 1,62 €
Budget del laboratorio      = 2 €
```

Si no has podido completar la estimación, utiliza **2 €**.

## Paso 3. Configurar el Budget

| Campo | Valor |
| --- | --- |
| Name | `Budget-Practica04-VM-AlumnoXX` |
| Reset period | Monthly |
| Creation date | Mes actual |
| Expiration date | Final del mes siguiente |
| Amount | El importe calculado en el Paso 2 |

## Paso 4. Configurar las alertas

Crea las siguientes condiciones de alerta:

| Tipo | Umbral | Importe con Budget de 2 € | Qué significa |
| --- | --- | --- | --- |
| Actual | 50 % | 1,00 € | Se ha gastado la mitad de lo previsto |
| Actual | 80 % | 1,60 € | El laboratorio se acerca al límite |
| Actual | 100 % | 2,00 € | Se ha alcanzado lo estimado |
| Forecasted | 100 % | 2,00 € | Azure prevé que el mes superará el Budget |

En **Alert recipients (email)** introduce el correo indicado por el profesor.

Selecciona **Create**.

🔎 **Predicción para el Día 2:** la alerta **Forecasted** probablemente se active antes que las demás. Azure proyecta el gasto diario hasta final de mes: si la VM consume algo más de 1 € al día, la proyección mensual superará con creces un Budget de 2 €. Anota ahora si crees que ocurrirá y compruébalo después.

---

# Sección 6 — Desplegar la máquina virtual

## Paso 1. Iniciar la creación

En Azure Portal busca **Virtual machines → Create → Azure virtual machine**.

## Paso 2. Pestaña Basics

| Campo | Valor |
| --- | --- |
| Resource group | `rg-practica04-vm-alumnoXX` |
| Virtual machine name | `vm-practica04-alumnoXX` |
| Region | West Europe |
| Availability options | No infrastructure redundancy required |
| Security type | Standard |
| Image | Ubuntu Server 24.04 LTS - x64 Gen2 |
| Size | **Standard_B2s** (usa *See all sizes* si no aparece) |
| Authentication type | SSH public key |
| Username | `azureuser` |
| Public inbound ports | **None** |

> Observa que junto al tamaño el portal muestra un **precio mensual aproximado**. Anótalo y compáralo con tu estimación.

> Seleccionamos **None** en los puertos de entrada porque en esta práctica no necesitamos conectarnos a la máquina. Así evitamos exponerla a Internet sin motivo.

## Paso 3. Pestaña Disks

| Campo | Valor |
| --- | --- |
| OS disk type | **Standard SSD (locally-redundant storage)** |
| Delete with VM | ✅ Activado |

## Paso 4. Pestaña Networking

Deja la red virtual, la subred y la IP pública que propone el portal.

Activa:

```plain text
✅ Delete public IP and NIC when VM is deleted
```

## Paso 5. Pestaña Management — Apagado automático

| Campo | Valor |
| --- | --- |
| Enable auto-shutdown | ✅ Activado |
| Shutdown time | 21:00 |
| Time zone | (UTC+01:00) Brussels, Copenhagen, Madrid, Paris |
| Notification before shutdown | Opcional, con tu correo |

💡 El apagado automático **desasigna** la VM, por lo que deja de facturarse el cómputo por la noche. Es uno de los controles de coste más sencillos y eficaces en entornos de desarrollo.

## Paso 6. Pestaña Tags

Añade los mismos Tags que el Resource Group:

| Tag | Valor |
| --- | --- |
| Project | Practica04 |
| Department | Data |
| Environment | Lab |
| Owner | AlumnoXX |
| CostCenter | DataNova-Analytics |

Asegúrate de que en la columna **Resource** aparece seleccionado **All resources** (o todos los tipos de recurso de la lista).

## Paso 7. Revisar y crear

Selecciona **Review + create**. En la pantalla de validación, revisa el precio por hora que muestra el portal y regístralo:

| Dato | Valor |
| --- | --- |
| Precio por hora mostrado por el portal |  |
| Precio por hora según tu estimación |  |
| Fecha y hora de creación |  |

Selecciona **Create** y descarga la clave privada cuando el portal lo solicite.

---

## Alternativa: despliegue con Azure CLI

Si el profesor lo indica, puedes hacer lo mismo desde **Cloud Shell** (Bash). Sustituye `XX` antes de ejecutar.

```bash
# Variables
ALUMNO="alumnoXX"
RG="rg-practica04-vm-$ALUMNO"
VM="vm-practica04-$ALUMNO"
LOCATION="westeurope"
TAGS="Project=Practica04 Department=Data Environment=Lab Owner=$ALUMNO CostCenter=DataNova-Analytics"

# 1. Resource Group con Tags
az group create --name $RG --location $LOCATION --tags $TAGS

# 2. Máquina virtual B2s con disco Standard SSD, sin puertos abiertos
az vm create \
  --resource-group $RG \
  --name $VM \
  --image Ubuntu2404 \
  --size Standard_B2s \
  --storage-sku StandardSSD_LRS \
  --os-disk-delete-option Delete \
  --nic-delete-option Delete \
  --public-ip-sku Standard \
  --nsg-rule NONE \
  --admin-username azureuser \
  --generate-ssh-keys \
  --tags $TAGS

# 3. Apagado automático (la hora se indica en UTC: 19:00 UTC = 21:00 en Madrid en horario de verano)
az vm auto-shutdown --resource-group $RG --name $VM --time 1900
```

> El Budget y sus alertas se crean desde el portal (Sección 5).

---

# Sección 7 — Verificar el etiquetado de todos los recursos

Abre el Resource Group. Deberías ver varios recursos creados automáticamente junto a la VM.

Completa la tabla revisando la columna **Tags** de cada uno (puedes añadirla desde **Manage view → Edit columns**):

| Recurso | Tipo | ¿Tiene los 5 Tags? | ¿Genera coste? |
| --- | --- | --- | --- |
| vm-practica04-alumnoXX | Virtual machine |  |  |
|  | Disk |  |  |
|  | Public IP address |  |  |
|  | Network interface |  |  |
|  | Network security group |  |  |
|  | Virtual network |  |  |

Si algún recurso con coste **no tiene Tags**, añádelos manualmente desde su pestaña **Tags**. Si no lo haces, su gasto aparecerá como *untagged* cuando agrupes por Tag en Cost Analysis.

---

# Sección 8 — Primera revisión en Cost Analysis

Esta sección se realiza **al día siguiente de crear la VM**.

## Paso 1. Abrir Cost Analysis con el ámbito del Resource Group

```plain text
Cost Management
      ↓
Scope → rg-practica04-vm-alumnoXX
      ↓
Cost analysis
```

## Paso 2. Ver el coste por recurso

Selecciona la vista **Resources** (o **Group by → Resource**).

Registra:

| Recurso | Coste acumulado |
| --- | --- |
| Máquina virtual |  |
| Disco |  |
| IP pública |  |
| Otros |  |
| **Total (Actual Cost)** |  |

## Paso 3. Ver el coste por medidor

Cambia a **Group by → Meter**.

Aquí verás exactamente qué se está cobrando: horas de cómputo B2s, capacidad del disco, horas de IP pública, etc. Es la vista más útil para entender **por qué** se paga.

## Paso 4. Ver el coste diario

Cambia la granularidad a **Daily** y el tipo de gráfico a **Column (grouped)**.

## Paso 5. Ver el coste por Tag

Selecciona **Group by → Tag → Project**.

Comprueba si todo el gasto aparece bajo `Practica04` o si hay una parte sin etiquetar.

## Paso 6. Consultar la previsión

En la vista **Accumulated costs**, localiza el valor de **Forecast**.

| Dato | Valor |
| --- | --- |
| Actual Cost |  |
| Forecasted Cost (fin de mes) |  |
| Budget del laboratorio |  |
| ¿Actual supera el Budget? | Sí / No |
| ¿Forecast supera el Budget? | Sí / No |

## Paso 7. Revisar las alertas

Consulta tu correo y abre **Cost Management → Cost alerts**.

| Alerta | ¿Se ha activado? | Fecha y hora |
| --- | --- | --- |
| Actual 50 % |  |  |
| Actual 80 % |  |  |
| Actual 100 % |  |  |
| Forecasted 100 % |  |  |

¿Se cumplió la predicción que hiciste en la Sección 5?

---

# Sección 9 — Experimento: apagar no es lo mismo que desasignar

Este experimento demuestra uno de los errores de coste más comunes en empresas.

## Los dos tipos de “apagado”

| Acción | Estado resultante | ¿Se cobra el cómputo? |
| --- | --- | --- |
| Apagar desde el sistema operativo (`sudo shutdown`) | **Stopped** | ✅ Sí, se sigue cobrando |
| Botón **Stop** del portal o `az vm deallocate` | **Stopped (deallocated)** | ❌ No |

En ambos casos **el disco y la IP pública se siguen cobrando**.

## Paso 1. Encender la VM

Si el apagado automático la detuvo por la noche, pulsa **Start** y déjala encendida unas horas.

## Paso 2. Desasignarla

Pulsa **Stop** en el portal y confirma. Comprueba que el estado es:

```plain text
Stopped (deallocated)
```

Desde CLI sería:

```bash
az vm deallocate --resource-group $RG --name $VM
```

## Paso 3. Observar el efecto al día siguiente

El Día 3, en Cost Analysis con **Group by → Meter** y granularidad **Daily**, compara los días:

| Medidor | Coste Día 1 | Coste Día 2 | Coste Día 3 |
| --- | --- | --- | --- |
| Cómputo B2s |  |  |  |
| Disco Standard SSD |  |  |  |
| IP pública |  |  |  |

Responde:

1. ¿Qué medidor baja cuando la VM está desasignada?
2. ¿Qué medidores siguen generando coste?
3. Si un compañero deja 20 VMs apagadas desde el sistema operativo durante un fin de semana, ¿qué está pagando DataNova?
4. ¿Qué acción garantizaría que **ningún** medidor siga cobrando?
---

# Sección 10 — Comparar estimación y coste real

Esta sección se realiza el Día 3, antes de la limpieza.

| Concepto | Estimado (Sección 3) | Real (Cost Analysis) | Diferencia |
| --- | --- | --- | --- |
| Cómputo |  |  |  |
| Disco |  |  |  |
| IP pública |  |  |  |
| **Total** |  |  |  |

Responde:

5. ¿Qué escenario de la Sección 3 se parece más a lo que realmente ocurrió?
6. ¿Qué componente se desvió más de lo estimado? ¿Por qué?
7. ¿Algún coste no estaba en tu estimación?
8. Si DataNova tuviera que mantener este servidor para 10 analistas durante un mes, 8 horas al día y 22 días, ¿qué coste estimarías con los datos reales que has obtenido?
---

# Sección 11 — Limpieza

## Paso 1. Eliminar el Resource Group

```plain text
Resource groups
      ↓
rg-practica04-vm-alumnoXX
      ↓
Delete resource group
```

Escribe el nombre del Resource Group para confirmar.

Desde CLI:

```bash
az group delete --name $RG --yes --no-wait
```

## Paso 2. Comprobar que no queda nada

Busca en el portal **All resources** y filtra por el Tag `Project = Practica04`.

El resultado debe ser **vacío**.

⚠️ Si creaste algún recurso fuera del Resource Group (por ejemplo, una cuenta de almacenamiento para Cloud Shell), no se habrá eliminado. Revisa **All resources** y consulta al profesor antes de borrarlo.

## Paso 3. Revisar el Budget del laboratorio

Abre **Cost Management → Budgets** con el ámbito de la suscripción y comprueba si `Budget-Practica04-VM-AlumnoXX` sigue existiendo. Si aparece, elimínalo: el laboratorio ha terminado.

## Paso 4. Verificación final al día siguiente

Vuelve a Cost Analysis 24 horas después de eliminar los recursos. El coste diario asociado a `Practica04` debe ser **0 €**.

---

# Sección 12 — Informe final

Completa la plantilla:

| Elemento | Resultado |
| --- | --- |
| Nombre de la práctica | Práctica 04 — VM Linux con control de costes |
| Resource Group | rg-practica04-vm-alumnoXX |
| Servicios utilizados |  |
| Coste estimado antes de desplegar |  |
| Budget disponible |  |
| Alertas configuradas |  |
| Alertas activadas |  |
| Coste observado |  |
| Recurso con mayor coste |  |
| Recursos eliminados |  |
| Observaciones |  |

---

# Sección 13 — Ejercicio final

Un equipo de DataNova Engineering despliega cinco VMs de pruebas con esta configuración:

```plain text
Budget del proyecto          = 50 €
Actual Cost (día 10 del mes) = 22 €
Forecasted Cost              = 68 €
Apagado automático           = No configurado
Estado de 3 VMs              = Stopped (no deallocated)
```

Responde:

### 1. ¿Qué alertas se habrán activado con umbrales Actual 50 %, 80 %, 100 % y Forecasted 100 %?

---

### 2. ¿Por qué el Forecasted Cost es tan superior al Actual Cost?

---

### 3. ¿Qué parte del gasto de las 3 VMs detenidas se podría eliminar sin borrarlas?

---

### 4. Ordena estas acciones de mayor a menor impacto en el ahorro inmediato:

- [ ] Desasignar las 3 VMs detenidas.
- [ ] Configurar apagado automático en las 5 VMs.
- [ ] Cambiar los discos de Premium SSD a Standard SSD.
- [ ] Eliminar las IP públicas que no se usan.
- [ ] Añadir Tags a todos los recursos.
---

### 5. ¿Cuál de las acciones anteriores no ahorra dinero pero es imprescindible para analizarlo?

---

# Sección 14 — Referencias oficiales

- Microsoft Learn — Tutorial: Create and manage budgets
[https://learn.microsoft.com/azure/cost-management-billing/costs/tutorial-acm-create-budgets](https://learn.microsoft.com/azure/cost-management-billing/costs/tutorial-acm-create-budgets)
- Microsoft Learn — Cost Analysis
[https://learn.microsoft.com/azure/cost-management-billing/costs/quick-acm-cost-analysis](https://learn.microsoft.com/azure/cost-management-billing/costs/quick-acm-cost-analysis)
- Microsoft Learn — Group and filter options in Cost Analysis and Budgets
[https://learn.microsoft.com/azure/cost-management-billing/costs/group-filter](https://learn.microsoft.com/azure/cost-management-billing/costs/group-filter)
- Microsoft Learn — Create a Linux virtual machine in the Azure portal
[https://learn.microsoft.com/azure/virtual-machines/linux/quick-create-portal](https://learn.microsoft.com/azure/virtual-machines/linux/quick-create-portal)
- Microsoft Learn — States and billing status of Azure Virtual Machines
[https://learn.microsoft.com/azure/virtual-machines/states-billing](https://learn.microsoft.com/azure/virtual-machines/states-billing)
- Microsoft Learn — Auto-shutdown a virtual machine
[https://learn.microsoft.com/azure/virtual-machines/auto-shutdown-vm](https://learn.microsoft.com/azure/virtual-machines/auto-shutdown-vm)
- Azure Pricing Calculator
[https://azure.microsoft.com/pricing/calculator/](https://azure.microsoft.com/pricing/calculator/)

