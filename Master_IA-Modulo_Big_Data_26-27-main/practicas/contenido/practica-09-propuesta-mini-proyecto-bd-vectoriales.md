# 🧗🏽‍♂️ Practica 09 - Propuesta mini-proyecto BD Vectoriales

## Desarrollo de una solución inteligente con SQL Server 2025

### 1. Objetivo

El objetivo de este mini proyecto es diseñar, implementar, documentar y presentar una solución de base de datos utilizando **SQL Server 2025 en local**, aplicando conocimientos relacionados con la certificación **Microsoft DP-800: SQL AI Developer Associate**.

El proyecto deberá demostrar de forma práctica al menos una** de las siguientes áreas**:

1. **Diseñar y desarrollar soluciones de base de datos**, incluyendo funcionalidades de desarrollo SQL asistido por IA.
2. **Proteger, optimizar e implementar soluciones de base de datos**.
3. **Implementar capacidades de IA en soluciones de base de datos**, incluyendo uno o varios de los siguientes elementos:
- Modelos y embeddings.
- Intelligent Search.
- RAG — Retrieval-Augmented Generation.
> 💡 Se valorará especialmente que el proyecto combine varias de estas áreas dentro de una única solución.

---

# 2. Escenario del proyecto

Cada persona deberá seleccionar un pequeño caso de negocio y desarrollar una solución utilizando SQL Server 2025.

Algunos ejemplos posibles:

- Buscador inteligente de productos.
- Base de conocimiento empresarial.
- Sistema de búsqueda de incidencias técnicas.
- Buscador de documentación.
- Sistema de preguntas y respuestas sobre documentos.
- Buscador semántico de películas o libros.
- Catálogo inteligente de cursos.
- Sistema de soporte técnico.
- Buscador de CV y perfiles profesionales.
- Base de conocimiento para una empresa ficticia.
- Sistema inteligente de preguntas frecuentes.
- Buscador semántico de noticias.
- Sistema de recomendaciones basado en similitud.
> 💡 También puede proponerse cualquier otro escenario que permita demostrar las competencias técnicas requeridas.

---

# 3. Restricciones técnicas

Todo el proyecto deberá ejecutarse **en local**. No será necesario utilizar recursos de Azure ni Microsoft Fabric.

El entorno mínimo será:

- Windows 11.
- SQL Server 2025.
- SQL Server Management Studio.
- Git.
- GitHub.
- Editor de código, por ejemplo:
- Visual Studio Code. O cualquier otro
- Visual Studio.
- SSMS.
Para los proyectos que implementen IA se podrán utilizar modelos ejecutados localmente, por ejemplo:

- Ollama.
- ONNX Runtime. Este se puede ejecutar en SSMS 22
- Otro modelo local compatible.
- GitHub Copilot
> No deben almacenarse claves, contraseñas ni secretos directamente en los scripts SQL o en el repositorio GitHub.

---

# 4. Requisitos funcionales

El proyecto deberá implementar al menos un** bloque** de los siguientes.

---

## BLOQUE A — Diseño y desarrollo de la base de datos

Crear una base de datos SQL Server 2025 que represente correctamente el escenario seleccionado.

Debe incluir como mínimo:

- Varias tablas relacionadas.
- PRIMARY KEY.
- FOREIGN KEY.
- Constraints cuando sean necesarias.
- Tipos de datos adecuados.
- Índices.
- Datos de prueba suficientes.
Además, deberán implementarse estos** objetos de programación**, por ejemplo:

- Views.
- Stored Procedures.
- Functions.
- Triggers.
Se deberá incluir también alguna consulta T-SQL avanzada utilizando uno o varios elementos como:

- CTE.
- Window Functions.
- JSON.
- Expresiones regulares.
- Fuzzy String Matching.
- Consultas correlacionadas.
---

## BLOQUE B — Desarrollo SQL asistido por IA

Utilizar una herramienta de IA como asistente durante alguna fase del desarrollo.

Por ejemplo:

- Generación de consultas SQL.
- Explicación de consultas.
- Generación de datos de prueba.
- Creación de procedimientos almacenados.
- Optimización de consultas.
- Generación de documentación.
- Identificación de errores.
- Propuesta de índices.
Deberás documentar:

4. El prompt utilizado.
5. La respuesta proporcionada por la IA.
6. Qué partes fueron aceptadas.
7. Qué partes tuvieron que ser corregidas.
8. Cómo se verificó que el código generado era correcto.
> 💡 No será suficiente incluir código generado directamente por IA sin explicar y validar su funcionamiento.

---

# 5. BLOQUE C — Seguridad de SQL Server

Implementar al menos **dos mecanismos de seguridad**.

Por ejemplo:

- Usuarios y roles.
- Permisos mediante `GRANT`, `DENY` y `REVOKE`.
- Row-Level Security.
- Dynamic Data Masking.
- Cifrado de columnas.
- Auditoría.
- Principio de mínimo privilegio.
Debe demostrarse mediante consultas que diferentes usuarios tienen diferentes permisos sobre los datos.

Ejemplo:

```plain text
Administrador
     │
     ├── Puede consultar todos los registros
     │
UsuarioVentas
     │
     └── Solo puede consultar determinados datos
```

> El repositorio deberá incluir scripts para reproducir la configuración.

---

# 6. BLOQUE D — Optimización de consultas

Seleccionar una consulta significativa del proyecto y estudiar su rendimiento.

Se deberá mostrar:

### Estado inicial

Ejecutar la consulta y analizar:

- Execution Plan.
- Table Scan / Index Scan / Index Seek.
- Costes principales.
- Logical reads.
- Tiempo de ejecución cuando resulte significativo.
### Optimización

Aplicar alguna mejora, por ejemplo:

- Crear un índice.
- Modificar la consulta.
- Modificar la estructura de los datos.
- Crear un índice compuesto.
- Crear un índice filtrado.
### Resultado

Comparar:

```plain text
ANTES
Consulta
   ↓
Table Scan
   ↓
Mayor coste

DESPUÉS
Consulta
   ↓
Index Seek
   ↓
Menor coste
```

> Se deberá explicar por qué la optimización mejora —o no mejora— el rendimiento.

---

# 7. BLOQUE E — Modelos y embeddings

Este bloque permite utilizar las nuevas capacidades de IA disponibles en SQL Server 2025.

La propuesta deberá almacenar información textual en SQL Server y convertirla en representaciones vectoriales.

Ejemplo:

```plain text
Producto
────────────────────────────
Nombre
Descripción
Categoría
        │
        ▼
Texto
        │
        ▼
Modelo de embeddings
        │
        ▼
VECTOR
        │
        ▼
SQL Server 2025
```

Puede utilizarse un modelo ejecutado localmente.

Por ejemplo:

```plain text
SQL Server 2025
      │
      │ CREATE EXTERNAL MODEL
      ▼
Ollama / ONNX Runtime
      │
      ▼
Modelo de embeddings
      │
      ▼
AI_GENERATE_EMBEDDINGS
      │
      ▼
VECTOR
```

Se deberá explicar:

- Qué modelo ha utilizado.
- Qué columnas se utilizan para crear el embedding.
- Qué dimensión tiene el vector.
- Cómo se generan los embeddings.
- Cómo se almacenan en SQL Server.
---

# 8. BLOQUE F — Intelligent Search

Construir un pequeño motor de búsqueda sobre los datos almacenados.

Podrá utilizarse:

- Full-Text Search.
- Vector Search.
- Semantic Search.
- Hybrid Search.
Por ejemplo, en un catálogo de productos:

Usuario:

```plain text
"portátil ligero para trabajar viajando"
```

La aplicación deberá devolver productos semánticamente similares aunque la descripción no contenga exactamente las mismas palabras.

Ejemplo:

```plain text
Pregunta del usuario
        │
        ▼
Generar embedding
        │
        ▼
[0.12, -0.44, 0.83, ...]
        │
        ▼
VECTOR SEARCH
        │
        ▼
SQL Server 2025
        │
        ▼
Productos más similares
```

Se deberá devolver un ranking de resultados. El alumno tendrá que justificar la métrica de similitud o mecanismo de búsqueda utilizado.

---

# 9. BLOQUE G — RAG

Como parte avanzada del proyecto puede implementarse una pequeña solución de **Retrieval-Augmented Generation**.

Ejemplo:

Base de conocimiento de soporte técnico.

El usuario pregunta:

```plain text
¿Cómo puedo solucionar el error de conexión 18456?
```

El sistema:

```plain text
Pregunta
   │
   ▼
Embedding
   │
   ▼
SQL Server 2025
   │
   ▼
Vector Search
   │
   ▼
Documentos relevantes
   │
   ▼
Construcción del contexto
   │
   ▼
LLM local
   │
   ▼
Respuesta
```

El modelo deberá responder utilizando información recuperada desde SQL Server.

Debe poder identificarse claramente:

9. La pregunta.
10. Los documentos recuperados.
11. El contexto proporcionado al modelo.
12. La respuesta final.
Se valorará que el estudiante explique claramente la diferencia entre:

```plain text
LLM
```

y

```plain text
SQL Server
      +
Vector Search
      +
Contexto
      +
LLM
      =
RAG
```

---

# 10. Arquitectura

El proyecto deberá incluir un pequeño diagrama de arquitectura.

Por ejemplo:

```plain text
                    ┌───────────────────┐
                    │      Usuario      │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Aplicación / SQL  │
                    └─────────┬─────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │    SQL Server 2025      │
                 │                         │
                 │ Tablas                  │
                 │ Stored Procedures       │
                 │ Índices                 │
                 │ VECTOR                  │
                 │ Embeddings              │
                 └───────────┬─────────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   Modelo local   │
                    │ Ollama / ONNX    │
                    └──────────────────┘
```

La arquitectura real dependerá de las funcionalidades seleccionadas.

---

# 11. GitHub

Todo el proyecto deberá almacenarse en un repositorio GitHub.

Ejemplo de estructura:

```plain text
dp800-mini-project/
│
├── README.md
│
├── docs/
│   ├── arquitectura.png
│   ├── proyecto.md
│   └── proyecto.pdf
│
├── database/
│   ├── 01_create_database.sql
│   ├── 02_create_tables.sql
│   ├── 03_insert_data.sql
│   ├── 04_views.sql
│   ├── 05_procedures.sql
│   ├── 06_security.sql
│   └── 07_indexes.sql
│
├── ai/
│   ├── create_external_model.sql
│   ├── generate_embeddings.sql
│   ├── vector_search.sql
│   └── rag.sql
│
├── tests/
│   └── tests.sql
│
├── presentation/
│   └── dp800-project.pptx
│
└── .gitignore
```

La estructura puede modificarse según las necesidades del proyecto.

---

# 12. README.md

El `README.md` deberá permitir que otro estudiante pueda descargar el repositorio y reproducir el proyecto.

Debe contener como mínimo:

## Descripción

Explicar qué problema resuelve el proyecto.

## Arquitectura

Mostrar el diagrama de la solución.

## Tecnologías

Por ejemplo:

```plain text
SQL Server 2025
T-SQL
SSMS
Git
GitHub
Ollama
ONNX Runtime
```

## Instalación

Explicar paso a paso cómo reproducir el proyecto.

## Base de datos

Describir las tablas y relaciones principales.

## Funcionalidades DP-800 implementadas

Identificar claramente qué requisitos se han cubierto.

Por ejemplo:

```plain text
✓ Diseño de base de datos
✓ Stored Procedures
✓ Seguridad
✓ Optimización
✓ Embeddings
✓ Vector Search
```

## Ejemplos

Mostrar algunas consultas y sus resultados.

## Conclusiones

Explicar:

- Qué se ha aprendido.
- Qué problemas surgieron.
- Cómo se resolvieron.
- Qué mejoras podrían implementarse.
---

# 13. Documento PDF

Además del repositorio se entregará un documento PDF.

Extensión recomendada:

**5–10 páginas.**

Debe contener:

13. Portada.
14. Objetivo del proyecto.
15. Caso de uso.
16. Arquitectura.
17. Modelo de datos.
18. Funcionalidades implementadas.
19. Consultas o código más importante.
20. Seguridad y/o optimización.
21. Funcionamiento de la parte de IA, si aplica.
22. Capturas de resultados.
23. Problemas encontrados.
24. Conclusiones.
No se busca incluir grandes cantidades de código en el PDF.

El código completo deberá estar disponible en GitHub.

---

# 14. Presentación

Cada estudiante o equipo realizará una presentación breve del proyecto.

Duración recomendada:

**5–8 minutos.**

Presentación recomendada:

### Slide 1

**Título y objetivo**

### Slide 2

**Problema que queremos resolver**

### Slide 3

**Arquitectura**

### Slide 4

**Base de datos SQL Server 2025**

### Slide 5

**Funcionalidades DP-800 implementadas**

### Slide 6

**Demostración**

### Slide 7

**Resultados**

### Slide 8

**Conclusiones**

Durante la presentación deberá realizarse, si es posible, una pequeña demostración en directo.

---

# 15. Entregables

Cada estudiante o equipo deberá entregar:

### 1. Repositorio GitHub

Debe contener todo el código necesario para reproducir el proyecto.

### 2. README.md

Documentación principal del proyecto.

### 3. Documento PDF

Informe técnico del proyecto.

### 4. PowerPoint

Presentación utilizada para exponer el proyecto.

### 5. Scripts SQL

Scripts necesarios para crear la solución desde cero.

---

# 16. Criterios de evaluación

| Área | Peso |
| --- | --- |
| Diseño de la base de datos | 15 % |
| Calidad del código T-SQL | 15 % |
| Implementación de requisitos DP-800 | 25 % |
| Seguridad / optimización | 10 % |
| IA, embeddings, búsqueda o RAG | 15 % |
| GitHub y reproducibilidad | 10 % |
| Documentación | 5 % |
| Presentación | 5 % |
| **TOTAL** | **100 %** |

---

# 17. Niveles del proyecto

Para permitir distintos grados de dificultad se establecen tres niveles.

## Nivel 1 — Básico

Implementar:

```plain text
SQL Server 2025
+
Diseño de BD
+
T-SQL
+
Seguridad u optimización
```

---

## Nivel 2 — Intermedio

Implementar:

```plain text
Nivel 1
+
Modelo de embeddings
+
VECTOR
+
Semantic Search
```

---

## Nivel 3 — Avanzado

Implementar:

```plain text
Nivel 2
+
Vector Search
+
RAG
+
Modelo local
```

El nivel seleccionado deberá indicarse en el `README.md`.

---

# 18. Requisito fundamental: demostrar que funciona

No será suficiente entregar únicamente código.

El estudiante deberá proporcionar evidencias de ejecución:

```plain text
Código
   │
   ▼
Ejecución
   │
   ▼
Resultado
   │
   ▼
Explicación
```

Cada funcionalidad importante debe poder reproducirse.

---

> 💡 El objetivo no es construir una aplicación empresarial completa, sino crear una solución pequeña, reproducible y técnicamente bien explicada que permita demostrar de forma práctica varias de las competencias evaluadas en **Microsoft DP-800: SQL AI Developer Associate**.

