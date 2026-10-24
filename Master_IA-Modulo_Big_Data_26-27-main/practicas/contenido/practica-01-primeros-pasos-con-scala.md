# 🧗🏽‍♂️ Practica 01 - Primeros pasos con Scala

**Página relacionada:** 🈂️Práctica 1 - Parte 1: Entornos de Scala
## 1. Introducción

En esta primera parte de la práctica prepararás distintos **entornos de desarrollo para trabajar con Scala**. El objetivo es que conozcas varias formas de ejecutar, probar y organizar código Scala antes de comenzar con los ejercicios de programación de las siguientes partes.

A lo largo del curso utilizaremos Scala desde distintos contextos. El material del curso (Datacamp) presenta tanto el uso de un entorno interactivo como el desarrollo mediante IDE y herramientas de construcción. También introduce **sbt** como herramienta utilizada para compilar, ejecutar y probar aplicaciones Scala.

Además, el material (Datacamp) menciona específicamente el uso de un **kernel de Scala para Jupyter mediante Almond**, por lo que uno de los entornos de esta práctica estará basado en JupyterLab.

---

# Parte 1 — Entornos de trabajo

Deberás preparar **tres entornos independientes de desarrollo**.

Los tres entornos deberán quedar configurados para trabajar con:

- **Scala 2.12.21**
- **Java / JDK 17**, cuando sea necesario.
- Windows 11 como sistema operativo.
Los entornos serán:

1. **JupyterLab + Almond Kernel + Scala 2.12.21**
2. **Visual Studio Code + Metals + Scala 2.12.21 + JDK 17 + sbt**
3. **IntelliJ IDEA Community + Scala 2.12.21 + sbt**
> **Importante:** no es suficiente con instalar las herramientas. Debes demostrar mediante capturas de pantalla que cada entorno está correctamente configurado y que permite ejecutar código Scala.

---

# 1.1 Entorno 1 — JupyterLab + Almond Kernel + Scala 2.12.21

## Objetivo

Preparar un entorno interactivo basado en **JupyterLab** que permita crear notebooks y ejecutar código utilizando **Scala 2.12.21** mediante el **Almond Kernel**.

El uso de notebooks permite ejecutar instrucciones de forma interactiva y observar inmediatamente sus resultados, de manera similar al uso del intérprete de Scala presentado durante el curso.

## Actividades

### 1. Instalar los requisitos necesarios

Instala en Windows 11 las herramientas necesarias para poder utilizar:

- JupyterLab.
- Almond Kernel.
- Scala 2.12.21.
Documenta cualquier requisito adicional que hayas necesitado instalar para que el entorno funcione correctamente.

### 2. Instalar JupyterLab

Realiza la instalación de JupyterLab y comprueba que puedes iniciarlo correctamente desde Windows.

Debes documentar:

- Método de instalación utilizado.
- Comando o procedimiento utilizado para iniciar JupyterLab.
- Navegador o interfaz desde la que accedes al entorno.
### 3. Instalar Almond Kernel

Instala **Almond** como kernel de Scala para Jupyter.

Una vez instalado, comprueba que al crear un nuevo Notebook aparece una opción correspondiente a Scala.

### 4. Verificar la versión de Scala

Crea un nuevo Notebook utilizando Almond.

Ejecuta una celda que permita comprobar que el entorno está utilizando:

```plain text
Scala 2.12.21
```

Incluye una captura donde pueda observarse claramente la versión utilizada.

### 5. Ejecutar código Scala

En el Notebook crea varias celdas y ejecuta, como mínimo, ejemplos equivalentes a los siguientes:

```scala
val nombre = "Scala"
val version = "2.12.21"

println(s"Hola desde$nombre$version")
```

A continuación realiza una operación numérica:

```scala
val a = 10
val b = 20
val resultado = a + b

println(resultado)
```

Finalmente crea una colección sencilla:

```scala
val lenguajes = List("Scala", "Java", "Python")

println(lenguajes)
```

### Evidencias mínimas

Tu documentación deberá incluir capturas donde se observe:

- JupyterLab ejecutándose.
- Almond disponible como kernel.
- Notebook utilizando Scala.
- Versión de Scala utilizada.
- Ejecución correcta de las tres pruebas anteriores.
---

# 1.2 Entorno 2 — Visual Studio Code + Metals + Scala 2.12.21 + JDK 17 + sbt

## Objetivo

Preparar un entorno de desarrollo para proyectos Scala utilizando:

- Visual Studio Code.
- Metals.
- JDK 17.
- Scala 2.12.21.
- sbt.
El material del curso (Datacamp) distingue entre pequeños scripts y aplicaciones Scala estructuradas. Las aplicaciones pueden estar compuestas por varios archivos y ser compiladas antes de ejecutarse.

En esta parte comenzarás a trabajar con ese segundo enfoque.

## Actividades

### 1. Instalar JDK 17

Instala un **JDK 17** compatible con las herramientas que vas a utilizar.

Comprueba la instalación desde una terminal.

Ejecuta:

```bash
java -version
```

y:

```bash
javac -version
```

Documenta el resultado.

### 2. Instalar Visual Studio Code

Instala Visual Studio Code.

Una vez instalado, inicia el programa y documenta:

- La versión instalada.
- La pantalla principal del editor.
### 3. Instalar Metals

Desde el sistema de extensiones de Visual Studio Code, instala:

```plain text
Scala (Metals)
```

Documenta mediante una captura:

- Nombre de la extensión.
- Estado de instalación.
### 4. Instalar y comprobar sbt

Instala **sbt**.

Comprueba desde una terminal que el comando está disponible.

Por ejemplo:

```bash
sbt --version
```

Incluye una captura del resultado.

## 5. Crear un proyecto Scala con sbt

Crea una carpeta para el proyecto:

```plain text
scala-vscode
```

El proyecto deberá tener, como mínimo, una estructura equivalente a:

```plain text
scala-vscode/
├── build.sbt
├── project/
└── src/
    └── main/
        └── scala/
            └── Main.scala
```

## 6. Configurar Scala 2.12.21

En el archivo:

```plain text
build.sbt
```

configura explícitamente la versión:

```scala
scalaVersion := "2.12.21"
```

Añade también un nombre para el proyecto:

```scala
name := "scala-vscode"
```

## 7. Crear el programa

Dentro de:

```plain text
src/main/scala/Main.scala
```

crea un pequeño programa.

Por ejemplo:

```scala
object Main extends App {

  val entorno = "Visual Studio Code"

  println("Práctica de programación básica con Scala")
  println(s"Ejecutando desde:$entorno")
}
```

## 8. Importar el proyecto con Metals

Abre la carpeta del proyecto desde Visual Studio Code.

Permite que Metals detecte e importe el proyecto sbt.

Documenta este proceso.

La captura deberá mostrar que Metals reconoce correctamente el proyecto Scala.

## 9. Compilar el proyecto

Desde una terminal situada dentro del proyecto ejecuta:

```bash
sbt compile
```

Comprueba que el proyecto compila correctamente.

## 10. Ejecutar el proyecto

Ejecuta:

```bash
sbt run
```

Comprueba que aparece la salida del programa.

## Evidencias mínimas

Incluye capturas donde se observe:

- Resultado de `java -version`.
- Visual Studio Code instalado.
- Extensión Metals instalada.
- Resultado de `sbt --version`.
- Estructura del proyecto.
- Contenido de `build.sbt`.
- Scala `2.12.21` configurado.
- Archivo `Main.scala`.
- Proyecto reconocido por Metals.
- Ejecución de `sbt compile`.
- Ejecución correcta de `sbt run`.
---

# 1.3 Entorno 3 — IntelliJ IDEA Community + Scala 2.12.21 + sbt

## Objetivo

Preparar un segundo entorno de desarrollo basado en un IDE completo utilizando:

- IntelliJ IDEA Community Edition.
- Plugin de Scala.
- JDK 17.
- Scala 2.12.21.
- sbt.
El material del curso (Datacamp) presenta los IDE como especialmente útiles para proyectos con múltiples archivos e identifica IntelliJ IDEA como uno de los entornos utilizados para desarrollar en Scala.

## Actividades

### 1. Instalar IntelliJ IDEA Community Edition

Descarga e instala:

```plain text
IntelliJ IDEA Community Edition
```

Inicia el programa y documenta la instalación.

## 2. Instalar el soporte para Scala

Instala el plugin necesario para trabajar con Scala desde IntelliJ IDEA.

Documenta:

- Nombre del plugin.
- Estado de instalación.
Si el programa solicita reiniciar IntelliJ después de instalar el plugin, realiza el reinicio antes de continuar.

## 3. Configurar JDK 17

Configura IntelliJ IDEA para utilizar **JDK 17**.

Comprueba que el JDK seleccionado para el proyecto corresponde a Java 17.

Incluye una captura donde pueda observarse esta configuración.

## 4. Crear un proyecto sbt

Crea un nuevo proyecto Scala utilizando **sbt** como herramienta de construcción.

Utiliza como nombre:

```plain text
scala-intellij
```

El proyecto deberá configurarse para utilizar:

```plain text
Scala 2.12.21
```

## 5. Revisar `build.sbt`

Comprueba que el archivo:

```plain text
build.sbt
```

contiene una configuración equivalente a:

```scala
scalaVersion := "2.12.21"

name := "scala-intellij"
```

## 6. Crear un programa Scala

Crea:

```plain text
src/main/scala/Main.scala
```

con un programa similar al siguiente:

```scala
object Main extends App {

  val entorno = "IntelliJ IDEA"

  println("Práctica de programación básica con Scala")
  println(s"Ejecutando desde:$entorno")
}
```

## 7. Ejecutar desde IntelliJ IDEA

Ejecuta el programa utilizando las herramientas del propio IDE.

Comprueba que la consola integrada muestra correctamente la salida.

## 8. Ejecutar utilizando sbt

Abre una terminal en el proyecto y ejecuta también:

```bash
sbt compile
```

y posteriormente:

```bash
sbt run
```

Documenta los resultados.

## Evidencias mínimas

Incluye capturas donde se observe:

- IntelliJ IDEA Community instalado.
- Plugin de Scala instalado.
- JDK 17 configurado.
- Proyecto sbt creado.
- Scala 2.12.21 configurado.
- Estructura del proyecto.
- Archivo `build.sbt`.
- Archivo `Main.scala`.
- Ejecución desde IntelliJ IDEA.
- Ejecución mediante `sbt run`.
---

# 1.4 Documentación obligatoria

Toda la práctica deberá estar documentada utilizando **Markdown**.

No se aceptará como entrega únicamente una colección de capturas de pantalla.

Cada captura deberá estar acompañada de una explicación de lo que se está realizando.

Por ejemplo:

```markdown
## Instalación de Metals

Desde la sección de extensiones de Visual Studio Code busqué la extensión
Scala (Metals) y procedí a instalarla.

![Instalación de Metals](images/vscode-metals.png)

Después de la instalación, Metals detectó automáticamente el archivo
build.sbt del proyecto.
```

---

# 1.5 Capturas de pantalla

Las capturas deberán:

- Ser legibles.
- Mostrar únicamente información relevante.
- Incluir suficiente contexto para identificar la herramienta utilizada.
- Mostrar los comandos ejecutados y sus resultados cuando corresponda.
- Mostrar los mensajes de éxito o error relevantes.
- Estar referenciadas desde los archivos Markdown.
- Guardarse dentro del propio repositorio.
Organiza las imágenes dentro de una carpeta:

```plain text
images/
```

Puedes utilizar nombres descriptivos como:

```plain text
images/
├── jupyter-inicio.png
├── jupyter-almond.png
├── jupyter-scala-version.png
├── vscode-metals.png
├── vscode-sbt-run.png
├── intellij-plugin-scala.png
├── intellij-jdk17.png
└── intellij-run.png
```

---

# 1.6 Estructura recomendada del repositorio

Tu repositorio deberá tener una estructura organizada.

Por ejemplo:

```plain text
practica-scala/
│
├── README.md
│
├── parte1/
│   ├── README.md
│   │
│   ├── notebook/
│   │   └── entorno-scala.ipynb
│   │
│   ├── vscode/
│   │   └── scala-vscode/
│   │
│   └── intellij/
│       └── scala-intellij/
│
└── images/
    ├── jupyter-inicio.png
    ├── jupyter-almond.png
    ├── vscode-metals.png
    └── intellij-scala.png
```

Puedes modificar esta estructura siempre que el repositorio permanezca claro y fácil de navegar.

---

# 1.7 Archivo `README.md`

En la raíz del repositorio deberás incluir un archivo:

```plain text
README.md
```

Como mínimo deberá contener:

```markdown
# Práctica de programación básica con Scala

## Autor

Nombre y apellidos:

## Entorno

-Sistema operativo: Windows 11
-Scala: 2.12.21
-Java: JDK 17

## Parte 1 — Entornos de trabajo

### Entorno 1
JupyterLab + Almond Kernel

[Ver documentación](parte1/README.md)

### Entorno 2
Visual Studio Code + Metals + sbt

### Entorno 3
IntelliJ IDEA Community + sbt
```

El README deberá permitir que otra persona pueda comprender y navegar fácilmente por tu trabajo.

---

# 1.8 Uso de GitHub

Todo el trabajo deberá entregarse mediante un **repositorio de GitHub**.

No será necesario utilizar Git desde la línea de comandos.

Puedes crear y gestionar el repositorio utilizando directamente la interfaz web de GitHub.

Deberás:

4. Crear un repositorio para la práctica.
5. Subir los archivos Markdown.
6. Subir las capturas de pantalla.
7. Subir el Notebook utilizado en JupyterLab.
8. Subir los proyectos desarrollados con Visual Studio Code e IntelliJ IDEA.
9. Mantener una estructura clara y ordenada de carpetas.
10. Comprobar que los enlaces relativos y las imágenes incluidas en los archivos Markdown funcionan correctamente desde GitHub.
El repositorio deberá ser accesible por el profesor en el momento de la entrega.

---

# 1.9 Entrega

La entrega se realizará **exclusivamente mediante un enlace al repositorio de GitHub**.

El enlace deberá tener un formato similar a:

```plain text
https://github.com/usuario/practica-scala
```

El repositorio deberá contener:

- La documentación en Markdown.
- Las capturas de pantalla.
- El Notebook utilizado en JupyterLab.
- El proyecto utilizado con Visual Studio Code.
- El proyecto utilizado con IntelliJ IDEA.
- Los archivos `build.sbt` correspondientes.
- El código fuente Scala.
- Un `README.md` principal que permita navegar por el proyecto.
> **Antes de entregar el enlace, comprueba que el repositorio es accesible por el profesor y que todos los enlaces e imágenes del Markdown funcionan correctamente.**

---

# Resultado esperado

Al finalizar la **Parte 1** deberás disponer de **tres entornos funcionales capaces de ejecutar código Scala 2.12.21**:

```plain text
JupyterLab
    │
    └── Almond Kernel
            │
            └── Scala 2.12.21


Visual Studio Code
    │
    ├── Metals
    ├── JDK 17
    └── sbt
         │
         └── Scala 2.12.21


IntelliJ IDEA Community
    │
    ├── Scala Plugin
    ├── JDK 17
    └── sbt
         │
         └── Scala 2.12.21
```

Estos entornos serán la base para desarrollar las actividades de programación correspondientes a las **Partes 2 y 3 de la práctica**.

**Página relacionada:** ⛩️Practica 1 - Parte 2: Programación con Scala en JupyterLab
# 1. Introducción

En esta segunda parte de la práctica trabajarás exclusivamente con el entorno configurado en la Parte 1:

- **JupyterLab**
- **Almond Kernel**
- **Scala 2.12.21**
Los ejercicios están basados en los contenidos estudiados en los capítulos 1, 2 y 3 del material del curso (Datacamp). Durante esta parte trabajarás con variables, tipos de datos, inferencia de tipos, funciones, arrays, listas, estructuras de control, operadores relacionales y lógicos, bucles `while`, `foreach`, mutabilidad, inmutabilidad y los estilos imperativo y funcional.

Todo el trabajo deberá realizarse dentro de uno o varios **Notebooks de Jupyter** utilizando Scala 2.12.21.

---

## 2. Instrucciones generales

Para cada ejercicio deberás:

1. Incluir una celda **Markdown** con:
- Número y título del ejercicio.
- Breve explicación de la solución planteada.
2. Incluir una o varias celdas de **código Scala** con la solución.
3. Ejecutar todas las celdas y dejar visible el resultado.
4. Añadir una breve conclusión en Markdown cuando el ejercicio solicite comparar comportamientos, explicar errores o reflexionar sobre mutabilidad, tipos o estilo de programación.
5. Utilizar nombres de variables y funciones descriptivos.
6. Evitar utilizar características de Scala que no hayan sido estudiadas en los capítulos 1, 2 y 3 del material del curso (Datacamp).
> **Importante:** el Notebook entregado deberá poder ejecutarse de principio a fin en JupyterLab con Almond Kernel y Scala 2.12.21.

---

# 3. Ejercicios

## Ejercicio 1 — Variables, tipos e inferencia

Crea variables que representen la información de una partida de cartas:

- Nombre del jugador.
- Número de partidas jugadas.
- Puntuación actual.
- Porcentaje de victorias.
- Indicación de si el jugador continúa activo.
Realiza dos versiones:

### Versión A

Declara explícitamente el tipo de cada variable.

Ejemplo de estructura:

```scala
val nombre: String = ...
val partidas: Int = ...
```

### Versión B

Declara las mismas variables permitiendo que Scala infiera los tipos.

Finalmente, muestra los valores por pantalla.

### Documentación

En una celda Markdown explica:

- Qué tipos ha inferido Scala.
- Qué diferencia observas entre ambas versiones.
- Qué variables deberían ser `val` y cuáles podrían ser `var`.
---

## Ejercicio 2 — `val`, `var` y reasignación

Crea las siguientes variables:

```scala
val jugador = "Marta"
var puntuacion = 10
```

Realiza las siguientes operaciones:

7. Incrementa la puntuación en 5 puntos.
8. Vuelve a incrementarla en 3 puntos.
9. Muestra el resultado final.
10. Intenta reasignar el valor de `jugador`.
La última operación deberá producir un error.

### Documentación

Incluye una celda Markdown explicando:

- Por qué `puntuacion` puede modificarse.
- Por qué `jugador` no puede reasignarse.
- Qué mensaje genera Scala al intentar modificar un `val`.
No elimines la celda que contiene el error. Debe quedar documentada en el Notebook.

---

## Ejercicio 3 — Tipos numéricos y precisión

Declara un mismo valor decimal utilizando:

```scala
Double
Float
```

Utiliza un número con muchos decimales, por ejemplo:

```plain text
3.14159265358979323846264338327
```

Muestra ambos valores.

Después:

11. Crea una variable de tipo `Int`.
12. Crea una variable de tipo `Boolean`.
13. Crea una variable de tipo `String`.
### Documentación

Explica:

- Qué diferencia observas entre `Double` y `Float`.
- Qué tipo utilizarías para representar el número de estudiantes de una clase.
- Qué tipo utilizarías para representar si un estudiante ha aprobado.
---

## Ejercicio 4 — Función para determinar si una mano se pasa de 21

Define una función llamada:

```scala
bust
```

que reciba la puntuación de una mano como `Int` y devuelva un `Boolean`.

La función deberá devolver:

- `true` si la puntuación es mayor que 21.
- `false` en caso contrario.
Prueba la función con los siguientes valores:

```plain text
18
21
22
30
```

### Resultado esperado

Los resultados deberán corresponder conceptualmente a:

```plain text
false
false
true
true
```

### Requisito

La función deberá devolver el resultado de la comparación y no deberá modificar variables externas.

---

## Ejercicio 5 — Comparación de dos manos

Define una función:

```scala
maxHand
```

que reciba dos valores enteros y devuelva el mayor de los dos utilizando `if` y `else`.

Prueba la función al menos con:

```plain text
17 y 19
20 y 18
21 y 21
```

### Después

Explica qué sucede cuando ambas manos tienen el mismo valor.

Modifica la función si consideras necesario mejorar el comportamiento para ese caso y documenta tu decisión.

---

## Ejercicio 6 — Decidir el ganador de una partida

Utiliza la función `bust` del ejercicio 4.

Crea una función:

```scala
ganador
```

que reciba dos manos:

```scala
handA: Int
handB: Int
```

y aplique las siguientes reglas:

14. Si ambas manos superan 21, devuelve `0`.
15. Si solamente `handA` supera 21, devuelve `handB`.
16. Si solamente `handB` supera 21, devuelve `handA`.
17. Si ninguna supera 21, devuelve la mano con mayor puntuación.
Prueba como mínimo los siguientes casos:

```plain text
26 y 20
18 y 22
24 y 25
17 y 19
21 y 20
```

### Documentación

Para cada prueba indica qué condición del bloque `if / else if / else` se ha cumplido.

---

## Ejercicio 7 — Arrays y mutabilidad

Crea el siguiente array:

```scala
val jugadores = Array("Alex", "Chen", "Marta")
```

Realiza las siguientes operaciones:

18. Muestra el array original.
19. Sustituye `"Alex"` por `"Sindhu"`.
20. Muestra nuevamente el array.
21. Intenta asignar el número `500` a una de sus posiciones.
### Documentación

Explica:

- Por qué los elementos del array pueden cambiar aunque la variable se haya declarado con `val`.
- Por qué Scala no permite introducir el valor `500` en un `Array[String]`.
- La diferencia entre reasignar la variable y modificar un elemento del array.
Mantén en el Notebook la prueba que genera el error de tipos.

---

## Ejercicio 8 — Creación e inicialización de Arrays

Crea un array para almacenar las puntuaciones de cuatro jugadores utilizando:

```scala
new Array[Int](4)
```

Realiza las siguientes tareas:

22. Muestra el array inmediatamente después de crearlo.
23. Asigna manualmente una puntuación a cada posición.
24. Muestra el array completo.
25. Muestra su longitud utilizando:
```scala
.length
```

Utiliza las puntuaciones:

```plain text
17, 24, 21, 19
```

### Pregunta

¿Qué valores contiene un `Array[Int]` recién creado antes de que asignes manualmente sus elementos?

---

## Ejercicio 9 — Recorrer un Array con `while`

Utiliza el array:

```scala
val manos = Array(17, 24, 21, 19, 26)
```

Recórrelo utilizando un bucle `while`.

Para cada posición deberás mostrar:

- El valor de la mano.
- El resultado de aplicar la función `bust`.
El ejercicio deberá utilizar:

```scala
var i = 0
```

y la propiedad:

```scala
manos.length
```

para determinar cuándo termina el bucle.

### Documentación

Explica:

- Por qué el contador debe ser mutable.
- Qué ocurriría si no incrementaras `i`.
- Qué elementos del ejercicio corresponden al estilo imperativo.
---

## Ejercicio 10 — Listas e inmutabilidad

Crea la siguiente lista:

```scala
val jugadores = List("Alex", "Chen", "Marta")
```

Realiza las siguientes operaciones:

26. Muestra la lista original.
27. Crea una nueva lista añadiendo `"Sindhu"` al principio mediante `::`.
28. Muestra ambas listas.
29. Comprueba que la lista original no ha cambiado.
30. Muestra:
- Su longitud.
- La lista invertida mediante `reverse`.
### Documentación

Explica por qué la operación con `::` produce una lista nueva en lugar de modificar la lista original.

---

## Ejercicio 11 — Construcción y concatenación de listas

Construye una lista utilizando `Nil` y el operador `::`.

La lista deberá contener:

```plain text
Ana
Luis
Marta
```

A continuación crea una segunda lista:

```plain text
Pedro
Sofia
```

Concatena ambas listas utilizando:

```scala
:::
```

Finalmente muestra:

- Primera lista.
- Segunda lista.
- Lista resultante.
### Requisito

Demuestra mediante la salida que las dos listas originales permanecen sin cambios después de la concatenación.

---

## Ejercicio 12 — Operadores relacionales y lógicos

Considera las siguientes puntuaciones:

```scala
val handA = 18
val handB = 21
val handC = 25
```

Crea expresiones que respondan a las siguientes preguntas:

31. ¿Es `handA` mayor que `handB`?
32. ¿Es `handB` igual a 21?
33. ¿Es `handC` diferente de 21?
34. ¿Son `handA` y `handB` menores o iguales a 21?
35. ¿Alguna de las manos `handA` o `handC` supera 21?
36. ¿No se ha pasado `handB` de 21?
Utiliza según corresponda:

```plain text
>
<
>=
<=
==
!=
&&
||
!
```

### Documentación

Incluye una tabla Markdown con tres columnas:

| Expresión | Resultado | Explicación |
| --- | --- | --- |
| … | … | … |

---

## Ejercicio 13 — `foreach` y funciones como valores

Utiliza:

```scala
val manos = Array(17, 24, 21, 26, 18)
```

y la función `bust`.

Recorre el array utilizando:

```scala
foreach
```

de forma que se evalúe cada mano.

Debes realizar dos versiones:

### Versión A

Utiliza un bucle `while`.

### Versión B

Utiliza `foreach` pasando una función que procese cada elemento.

### Documentación

Compara ambas versiones e identifica:

- Cuál necesita un contador.
- Cuál necesita una variable `var` para recorrer la colección.
- Cuál se aproxima más al estilo funcional presentado en el material del curso (Datacamp).
---

## Ejercicio 14 — Efectos secundarios y estilo de programación

Analiza el siguiente planteamiento:

```scala
var total = 0

def sumarAlTotal(valor: Int) = {
  total = total + valor
}
```

Realiza las siguientes tareas:

37. Ejecuta la función varias veces.
38. Muestra el valor de `total` después de cada llamada.
39. Explica por qué la función modifica una variable situada fuera de su ámbito local.
40. Identifica el efecto secundario.
Después crea una segunda versión:

```scala
def sumar(a: Int, b: Int): Int = {
  ...
}
```

que reciba valores y devuelva un nuevo resultado sin modificar variables externas.

### Documentación

Compara ambas funciones en una tabla:

| Característica | `sumarAlTotal` | `sumar` |
| --- | --- | --- |
| Modifica datos externos |  |  |
| Devuelve un resultado calculado |  |  |
| Utiliza efecto secundario |  |  |
| Estilo predominante |  |  |

---

## Ejercicio 15 — Programa integrado: torneo de Twenty-One

Desarrolla un pequeño programa que combine los principales conceptos trabajados en esta parte.

Dispones de los siguientes jugadores:

```scala
val jugadores = List("Alex", "Chen", "Marta", "Sindhu")
```

y de sus puntuaciones:

```scala
val manos = Array(18, 24, 21, 20)
```

El programa deberá:

41. Mantener los nombres de los jugadores en una `List`.
42. Mantener las puntuaciones en un `Array[Int]`.
43. Utilizar la función `bust` para determinar qué manos superan 21.
44. Recorrer todas las puntuaciones y mostrar si cada mano se ha pasado o no.
45. Determinar cuál es la mejor puntuación válida.
46. Utilizar `if`, `else if` y `else` cuando sea necesario.
47. Utilizar operadores relacionales o lógicos.
48. Utilizar al menos una función definida por ti.
49. Utilizar al menos una estructura `while` o `foreach`.
50. Evitar modificar las listas originales.
### Ampliación dentro del mismo ejercicio

Añade una segunda ronda:

```scala
val manosRonda2 = Array(22, 19, 20, 21)
```

Determina nuevamente qué jugadores continúan con una puntuación válida y cuál es la mejor puntuación de la ronda.

### Informe final del ejercicio

Añade una celda Markdown explicando:

- Qué partes de tu solución son mutables.
- Qué partes son inmutables.
- Dónde utilizas funciones.
- Dónde utilizas estructuras de control.
- Qué parte de tu solución consideras más cercana al estilo funcional.
- Qué parte consideras más cercana al estilo imperativo.
---

# 4. Organización del Notebook

El trabajo deberá estar claramente organizado.

Puedes utilizar un único Notebook:

```plain text
parte2-scala.ipynb
```

con la siguiente estructura:

```plain text
# Parte 2 — Programación con Scala

## Ejercicio 1
Explicación en Markdown
Código Scala
Resultado

## Ejercicio 2
Explicación en Markdown
Código Scala
Resultado

...

## Ejercicio 15
Explicación en Markdown
Código Scala
Resultado
Conclusiones
```

También puedes dividir el trabajo en varios Notebooks si lo prefieres, siempre que estén correctamente numerados y enlazados desde el `README.md`.

---

# 5. Documentación y evidencias

La documentación forma parte de la práctica.

El Notebook deberá mostrar:

- Las celdas Markdown con las explicaciones solicitadas.
- Todo el código Scala.
- Las salidas obtenidas después de ejecutar las celdas.
- Los errores solicitados expresamente en algunos ejercicios.
- Las conclusiones y comparaciones correspondientes.
Además, incluye en el repositorio una carpeta:

```plain text
images/
```

con capturas que demuestren, como mínimo:

- JupyterLab abierto.
- Notebook ejecutándose con Almond.
- Scala 2.12.21 como versión utilizada.
- Ejecución de varios ejercicios.
- Ejecución del ejercicio 15 completo.
Las capturas deberán ser legibles y estar referenciadas desde el archivo `README.md` o desde otro documento Markdown del repositorio.

---

# 6. Estructura recomendada del repositorio

Puedes continuar utilizando el repositorio de la práctica completa o utilizar el repositorio indicado por el profesor.

Una estructura recomendada es:

```plain text
practica-scala/
│
├── README.md
│
├── parte1/
│   └── ...
│
├── parte2/
│   ├── README.md
│   ├── parte2-scala.ipynb
│   └── images/
│       ├── jupyter-parte2.png
│       ├── scala-version.png
│       ├── ejercicio-01.png
│       ├── ejercicio-09.png
│       ├── ejercicio-13.png
│       └── ejercicio-15.png
│
└── ...
```

Si utilizas varios Notebooks:

```plain text
parte2/
├── README.md
├── notebooks/
│   ├── ejercicios-01-05.ipynb
│   ├── ejercicios-06-10.ipynb
│   └── ejercicios-11-15.ipynb
└── images/
```

---

# 7. Archivo `README.md`

La Parte 2 deberá estar documentada desde un archivo Markdown.

Como mínimo deberá indicar:

```markdown
# Parte 2 — Programación con Scala

## Entorno utilizado

-JupyterLab
-Almond Kernel
-Scala 2.12.21

## Contenido

Esta parte contiene 15 ejercicios de programación realizados
en Scala utilizando JupyterLab.

## Notebook

[Ver Notebook de la Parte 2](parte2-scala.ipynb)

## Evidencias

![JupyterLab con Scala](images/jupyter-parte2.png)
```

Si la Parte 2 se encuentra dentro del mismo repositorio de la Parte 1, actualiza también el `README.md` principal para incluir un enlace a esta sección.

Por ejemplo:

```markdown
## Parte 2 — Programación con Scala

[Ver Parte 2](parte2/README.md)
```

---

# 8. Uso de GitHub

Todo el trabajo de esta parte deberá almacenarse en **GitHub**.

No será necesario utilizar Git desde la línea de comandos.

Desde GitHub deberás asegurarte de que estén disponibles:

- El Notebook o los Notebooks de la Parte 2.
- El archivo `README.md`.
- Las capturas de pantalla.
- Cualquier archivo adicional necesario para comprender la solución.
Antes de entregar:

51. Abre el repositorio desde GitHub.
52. Comprueba que el `README.md` se visualiza correctamente.
53. Comprueba que las imágenes aparecen correctamente.
54. Comprueba que el Notebook puede abrirse desde GitHub.
55. Comprueba que el Notebook contiene las salidas de las celdas ejecutadas.
56. Comprueba que los 15 ejercicios están completos.
57. Verifica que el repositorio sea accesible por el profesor.
---

# 9. Entrega

La entrega de la Parte 2 se realizará mediante un **enlace al repositorio de GitHub**.

Ejemplo:

```plain text
https://github.com/usuario/practica-scala
```

El repositorio deberá contener como mínimo:

- `README.md`.
- Notebook de Jupyter con los 15 ejercicios.
- Código Scala ejecutado.
- Resultados de ejecución visibles.
- Explicaciones y conclusiones en Markdown.
- Capturas de pantalla.
- Evidencia de uso de JupyterLab + Almond Kernel + Scala 2.12.21.
> **Importante:** no entregues únicamente capturas de pantalla. El código, las explicaciones y los resultados deberán estar contenidos en el Notebook.

---

# Resultado esperado

Al finalizar la Parte 2 deberás disponer de un Notebook reproducible que demuestre que puedes utilizar los principales elementos de programación estudiados en los capítulos 1, 2 y 3 del material del curso (Datacamp), incluyendo:

```plain text
Scala 2.12.21
│
├── val y var
├── tipos e inferencia
├── funciones
├── Array
├── List
│   ├── Nil
│   ├── ::
│   └── :::
├── if / else if / else
├── operadores relacionales y lógicos
├── while
├── foreach
├── mutabilidad e inmutabilidad
└── estilos imperativo y funcional
```

La Parte 2 servirá como base para las actividades que se desarrollarán posteriormente en la Parte 3.

**Página relacionada:** 🥷🏽Practica 1 - Parte 3: Mini proyectos en entornos de desarrollo
## 1. Introducción

En esta tercera parte de la práctica aplicarás de forma integrada los contenidos trabajados en los capítulos 1, 2 y 3 del material del curso (Datacamp).

La Parte 3 se divide en dos mini proyectos:

- **Parte 3.1:** Visual Studio Code + Metals + Scala 2.12.21 + JDK 17 + sbt.
- **Parte 3.2:** IntelliJ IDEA Community + Scala 2.12.21 + JDK 17 + sbt.
Los dos mini proyectos deberán resolverse utilizando únicamente conceptos estudiados previamente, entre ellos:

- `val` y `var`.
- Tipos básicos.
- Inferencia de tipos.
- Funciones.
- `Array`.
- `List`.
- `Nil`.
- `::` y `:::`.
- `if`, `else if` y `else`.
- Operadores relacionales y lógicos.
- `while`.
- `foreach`.
- Mutabilidad e inmutabilidad.
- Estilo imperativo y estilo funcional.
No es necesario utilizar clases propias, herencia, `case class`, `match`, colecciones avanzadas ni otras características no trabajadas en el material del curso (Datacamp).

---

# 2. Metodología general

Para ambos mini proyectos deberás:

1. Leer el enunciado completo antes de comenzar.
2. Diseñar una solución sencilla utilizando los conceptos estudiados.
3. Dividir el problema en varias funciones.
4. Utilizar nombres de variables y funciones descriptivos.
5. Ejecutar y comprobar diferentes casos de prueba.
6. Documentar el proceso con Markdown.
7. Incluir capturas de pantalla de los pasos más importantes.
8. Entregar el trabajo mediante un repositorio de GitHub.
> **Importante:** no se evaluará únicamente que el programa produzca una salida. También se tendrá en cuenta la organización del código, el uso correcto de los conceptos trabajados y la calidad de la documentación.

---

# Parte 3.1 — Visual Studio Code + Metals + sbt

## 3.1.1 Entorno de trabajo

Este mini proyecto deberá realizarse utilizando:

- **Visual Studio Code**
- **Metals**
- **Scala 2.12.21**
- **JDK 17**
- **sbt**
El proyecto deberá ser una aplicación Scala organizada con una estructura sbt.

Antes de comenzar el proyecto, incluye capturas donde pueda observarse:

- Visual Studio Code abierto.
- Extensión **Scala (Metals)** instalada y activa.
- JDK 17 disponible.
- Proyecto sbt abierto en Visual Studio Code.
- Scala 2.12.21 configurado en `build.sbt`.
- Metals reconociendo correctamente el proyecto.
---

## 3.1.2 Mini proyecto — Clasificador de resultados de un torneo de Twenty-One

### Descripción

Vas a desarrollar un pequeño programa que analice los resultados de varias manos de un torneo de **Twenty-One**.

Cada jugador tendrá:

- Un nombre.
- Una puntuación.
El programa deberá determinar:

- Si cada jugador se ha pasado de 21.
- Qué jugadores tienen una puntuación válida.
- Cuál es la mejor puntuación válida de la ronda.
- Cuántos jugadores se han pasado.
- Cuántos jugadores siguen dentro de la partida.
El proyecto está inspirado en los ejemplos de Twenty-One utilizados en el material del curso (Datacamp), pero deberás construir una solución más completa combinando los conceptos aprendidos.

---

## 3.1.3 Estructura del proyecto

Crea un proyecto sbt con una estructura equivalente a:

```plain text
torneo-twenty-one/
├── build.sbt
├── project/
└── src/
    └── main/
        └── scala/
            └── Main.scala
```

---

## 3.1.4 Archivo `build.sbt`

Configura como mínimo:

```scala
scalaVersion := "2.12.21"

name := "torneo-twenty-one"
```

---

## 3.1.5 Programa principal

Crea:

```plain text
src/main/scala/Main.scala
```

con una estructura equivalente a:

```scala
object Main extends App {
  // Código del mini proyecto
}
```

---

## 3.1.6 Datos iniciales

Utiliza las siguientes colecciones:

```scala
val jugadores = List(
  "Alex",
  "Chen",
  "Marta",
  "Sindhu",
  "Luis"
)

val puntuaciones = Array(
  18,
  24,
  21,
  20,
  26
)
```

Cada posición de `jugadores` se corresponde con la misma posición de `puntuaciones`.

Por ejemplo:

```plain text
Alex   -> 18
Chen   -> 24
Marta  -> 21
Sindhu -> 20
Luis   -> 26
```

---

## 3.1.7 Función `bust`

Define una función:

```scala
def bust(puntuacion: Int): Boolean
```

que devuelva:

- `true` si la puntuación es mayor que 21.
- `false` en caso contrario.
Ejemplo:

```scala
println(bust(24))
```

Resultado:

```plain text
true
```

---

## 3.1.8 Función `estadoMano`

Define una función que reciba una puntuación y devuelva:

```plain text
"VALIDA"
```

o:

```plain text
"BUST"
```

según corresponda.

---

## 3.1.9 Función `mejorMano`

Define una función:

```scala
def mejorMano(handA: Int, handB: Int): Int
```

que aplique las siguientes reglas:

9. Si ambas puntuaciones superan 21, devuelve `0`.
10. Si solo `handA` supera 21, devuelve `handB`.
11. Si solo `handB` supera 21, devuelve `handA`.
12. Si ninguna supera 21, devuelve la mayor.
Utiliza `if`, `else if` y `else`.

---

## 3.1.10 Procesamiento de la primera ronda

Recorre las puntuaciones y muestra un resultado similar a:

```plain text
Alex -> 18 -> VALIDA
Chen -> 24 -> BUST
Marta -> 21 -> VALIDA
Sindhu -> 20 -> VALIDA
Luis -> 26 -> BUST
```

Para recorrer las colecciones puedes utilizar un `while`.

Deberás utilizar:

```scala
var i = 0
```

y avanzar mientras:

```scala
i < puntuaciones.length
```

---

## 3.1.11 Estadísticas de la primera ronda

Calcula y muestra:

- Número total de jugadores.
- Número de manos válidas.
- Número de manos que se han pasado.
- Mejor puntuación válida.
La salida deberá ser clara, por ejemplo:

```plain text
--- Resumen de la ronda ---

Jugadores: 5
Manos válidas: 3
Bust: 2
Mejor puntuación válida: 21
```

---

## 3.1.12 Segunda ronda

Añade una segunda ronda:

```scala
val puntuacionesRonda2 = Array(
  22,
  19,
  20,
  21,
  17
)
```

Repite el análisis utilizando las mismas funciones.

No modifiques la lista original de jugadores.

---

## 3.1.13 Comparación de rondas

Al finalizar las dos rondas, muestra:

- Mejor puntuación de la primera ronda.
- Mejor puntuación de la segunda ronda.
- Qué ronda tuvo la mejor puntuación.
- Si ambas tuvieron la misma mejor puntuación.
Utiliza `if`, `else if` y `else`.

---

## 3.1.14 Uso de `foreach`

Después de completar una versión con `while`, realiza una segunda versión de alguna parte del procesamiento utilizando:

```scala
foreach
```

Por ejemplo, puedes utilizar `foreach` para mostrar si cada puntuación es válida o se ha pasado.

En el `README.md`, compara brevemente:

- La versión con `while`.
- La versión con `foreach`.
Indica:

- Cuál necesita un contador.
- Cuál necesita una variable mutable para recorrer la colección.
- Cuál se aproxima más al estilo funcional presentado en el material del curso (Datacamp).
---

## 3.1.15 Compilación y ejecución

Desde la terminal integrada de Visual Studio Code ejecuta:

```bash
sbt compile
```

y posteriormente:

```bash
sbt run
```

Incluye capturas donde se observe:

- Compilación correcta.
- Ejecución del programa.
- Salida de la primera ronda.
- Salida de la segunda ronda.
- Comparación final.
---

## 3.1.16 Documentación del mini proyecto

Dentro de la carpeta del proyecto incluye un:

```plain text
README.md
```

que contenga como mínimo:

```markdown
# Mini proyecto 3.1 — Torneo de Twenty-One

## Entorno

-Visual Studio Code
-Metals
-Scala 2.12.21
-JDK 17
-sbt

## Descripción

Breve descripción del proyecto.

## Estructura

Descripción de los archivos principales.

## Funciones utilizadas

-bust
-estadoMano
-mejorMano

## Ejecución

```bash
sbt compile
sbt run
```

## Resultados

Descripción de los resultados obtenidos.

---

---

# Parte 3.2 — IntelliJ IDEA Community + sbt

## 3.2.1 Entorno de trabajo

Este mini proyecto deberá realizarse utilizando:

- **IntelliJ IDEA Community**
- **Scala 2.12.21**
- **JDK 17**
- **sbt**
El proyecto deberá ser una aplicación Scala organizada con una estructura sbt.

Antes de comenzar, incluye capturas donde pueda observarse:

- IntelliJ IDEA abierto.
- Plugin de Scala activo.
- JDK 17 seleccionado.
- Proyecto sbt cargado.
- Scala 2.12.21 configurado.
---

## 3.2.2 Mini proyecto — Analizador de calificaciones de un grupo

### Descripción

Vas a desarrollar una pequeña aplicación que analice las calificaciones de un grupo de estudiantes.

El programa deberá:

- Almacenar nombres.
- Almacenar notas.
- Determinar quién aprueba y quién suspende.
- Calcular estadísticas básicas.
- Identificar la mejor nota.
- Mostrar un resumen final.
El propósito del ejercicio es aplicar los conceptos estudiados en una aplicación Scala estructurada y ejecutada mediante `sbt`.

---

## 3.2.3 Datos iniciales

Utiliza:

```scala
val estudiantes = List(
  "Ana",
  "Luis",
  "Marta",
  "Pedro",
  "Sofia"
)

val notas = Array(
  8,
  4,
  10,
  6,
  3
)
```

Cada nombre se corresponde con la nota que ocupa la misma posición.

---

## 3.2.4 Estructura del proyecto

El proyecto deberá tener, como mínimo:

```plain text
analizador-notas/
├── build.sbt
├── project/
└── src/
    └── main/
        └── scala/
            └── Main.scala
```

---

## 3.2.5 Archivo `build.sbt`

Configura:

```scala
scalaVersion := "2.12.21"

name := "analizador-notas"
```

---

## 3.2.6 Programa principal

Crea:

```plain text
src/main/scala/Main.scala
```

con:

```scala
object Main extends App {
  // Código del proyecto
}
```

---

## 3.2.7 Función `aprobado`

Define:

```scala
def aprobado(nota: Int): Boolean
```

que devuelva:

- `true` si la nota es mayor o igual que 5.
- `false` en caso contrario.
---

## 3.2.8 Función `estadoNota`

Define una función que devuelva:

```plain text
"APROBADO"
```

o:

```plain text
"SUSPENSO"
```

según corresponda.

---

## 3.2.9 Función `maxNota`

Define:

```scala
def maxNota(a: Int, b: Int): Int
```

que devuelva la mayor de dos notas utilizando `if` y `else`.

---

## 3.2.10 Listado de estudiantes

Recorre los datos y muestra:

```plain text
Ana -> 8 -> APROBADO
Luis -> 4 -> SUSPENSO
Marta -> 10 -> APROBADO
Pedro -> 6 -> APROBADO
Sofia -> 3 -> SUSPENSO
```

Puedes utilizar un bucle `while`.

---

## 3.2.11 Estadísticas

Calcula:

- Número total de estudiantes.
- Número de aprobados.
- Número de suspensos.
- Nota más alta.
Muestra:

```plain text
--- Resumen del grupo ---

Estudiantes: 5
Aprobados: 3
Suspensos: 2
Mejor nota: 10
```

---

## 3.2.12 Clasificación adicional

Utiliza `if`, `else if` y `else` para clasificar cada nota:

```plain text
9 o 10 -> EXCELENTE
7 u 8  -> NOTABLE
5 o 6  -> APROBADO
0 a 4  -> SUSPENSO
```

Define:

```scala
def clasificacion(nota: Int): String
```

Después muestra:

```plain text
Ana -> 8 -> NOTABLE
Luis -> 4 -> SUSPENSO
Marta -> 10 -> EXCELENTE
...
```

---

## 3.2.13 Segunda evaluación

Añade:

```scala
val notasSegundaEvaluacion = Array(
  9,
  5,
  8,
  7,
  6
)
```

Analiza la segunda evaluación utilizando las mismas funciones.

No modifiques la lista original de estudiantes.

---

## 3.2.14 Comparación de evaluaciones

Muestra:

- Mejor nota de la primera evaluación.
- Mejor nota de la segunda evaluación.
- Número de aprobados de la primera.
- Número de aprobados de la segunda.
Finalmente indica si el grupo:

- Ha mejorado.
- Ha empeorado.
- Se ha mantenido igual.
Para decidirlo, compara el número de aprobados de ambas evaluaciones.

---

## 3.2.15 Uso de listas

Crea una nueva lista de estudiantes incorporando un estudiante nuevo al principio mediante:

```scala
::
```

Por ejemplo:

```scala
val nuevosEstudiantes = "Carlos" :: estudiantes
```

Muestra:

- Lista original.
- Lista nueva.
Explica en la documentación por qué la lista original no se modifica.

---

## 3.2.16 Ejecución con sbt

Desde la terminal integrada de IntelliJ IDEA ejecuta:

```bash
sbt compile
```

y posteriormente:

```bash
sbt run
```

Incluye capturas donde se observe:

- Compilación correcta.
- Ejecución del programa.
- Salida completa.
---

---

# 4. Documentación obligatoria

Los dos mini proyectos deberán estar documentados mediante archivos Markdown.

La documentación deberá explicar:

- Qué hace el programa.
- Cómo está organizado.
- Qué funciones se han creado.
- Qué colecciones se utilizan.
- Qué resultados se obtienen.
- Qué problemas aparecieron durante el desarrollo.
- Cómo se solucionaron.
No se aceptará como documentación únicamente una colección de capturas de pantalla.

---

# 5. Capturas de pantalla

Incluye capturas legibles que demuestren el funcionamiento de ambos entornos.

## Parte 3.1

Como mínimo:

- Visual Studio Code con el proyecto abierto.
- Metals instalado y activo.
- Estructura del proyecto sbt.
- `build.sbt`.
- `Main.scala`.
- Resultado de `sbt compile`.
- Resultado de `sbt run`.
- Salida de la primera ronda.
- Salida de la segunda ronda.
## Parte 3.2

Como mínimo:

- IntelliJ IDEA con el proyecto abierto.
- Plugin de Scala.
- Estructura sbt.
- `build.sbt`.
- `Main.scala`.
- Resultado de `sbt compile`.
- Resultado de `sbt run`.
- Salida final de la aplicación.
---

# 6. Estructura recomendada del repositorio

Puedes continuar utilizando el mismo repositorio de las Partes 1 y 2.

La estructura recomendada es:

```plain text
practica-scala/
│
├── README.md
│
├── parte1/
│   └── ...
│
├── parte2/
│   └── ...
│
└── parte3/
    │
    ├── README.md
    │
    ├── parte3-1-vscode/
    │   ├── README.md
    │   ├── torneo-twenty-one/
    │   │   ├── build.sbt
    │   │   ├── project/
    │   │   └── src/
    │   │       └── main/
    │   │           └── scala/
    │   │               └── Main.scala
    │   └── images/
    │       ├── vscode-metals.png
    │       ├── sbt-compile.png
    │       └── sbt-run.png
    │
    └── parte3-2-intellij/
        ├── README.md
        ├── analizador-notas/
        │   ├── build.sbt
        │   ├── project/
        │   └── src/
        │       └── main/
        │           └── scala/
        │               └── Main.scala
        └── images/
            ├── intellij-proyecto.png
            ├── sbt-compile.png
            └── sbt-run.png
```

Puedes modificar esta estructura siempre que el repositorio siga siendo claro y fácil de navegar.

---

# 7. README de la Parte 3

Crea:

```plain text
parte3/README.md
```

con una estructura similar a:

```markdown
# Parte 3 — Mini proyectos

## Parte 3.1 — Visual Studio Code + Metals + sbt

Mini proyecto: Clasificador de resultados de un torneo de Twenty-One.

[Ver Parte 3.1](parte3-1-vscode/README.md)

## Parte 3.2 — IntelliJ IDEA + sbt

Mini proyecto: Analizador de calificaciones de un grupo.

[Ver Parte 3.2](parte3-2-intellij/README.md)
```

Actualiza también el `README.md` principal del repositorio para incluir un enlace a la Parte 3.

---

# 8. Uso de GitHub

Todo el trabajo deberá estar almacenado en **GitHub**.

No será necesario utilizar Git desde la línea de comandos.

Desde la interfaz de GitHub deberás comprobar que:

- Los archivos Markdown se visualizan correctamente.
- Las imágenes aparecen correctamente.
- Los proyectos incluyen sus archivos fuente.
- Los archivos `build.sbt` están incluidos.
- El código fuente Scala está incluido.
- El repositorio es accesible por el profesor.
---

# 9. Entrega

La entrega de la Parte 3 se realizará mediante un **enlace al repositorio de GitHub**.

Ejemplo:

```plain text
https://github.com/usuario/practica-scala
```

El repositorio deberá contener:

- Documentación Markdown.
- Mini proyecto de la Parte 3.1.
- Mini proyecto de la Parte 3.2.
- Código Scala.
- Archivos `build.sbt`.
- Capturas de pantalla.
- Resultados de ejecución.
- `README.md` principal actualizado.
> **Antes de entregar**, revisa que todos los enlaces internos, imágenes y archivos puedan abrirse correctamente desde GitHub.

---

# Resultado esperado

Al finalizar la Parte 3 deberás haber desarrollado dos mini proyectos funcionales:

```plain text
Parte 3
│
├── 3.1 Visual Studio Code + Metals + sbt
│   └── Torneo de Twenty-One
│       ├── funciones
│       ├── Array
│       ├── List
│       ├── while
│       ├── foreach
│       ├── if / else
│       ├── compilación con sbt
│       └── ejecución desde VSCode
│
└── 3.2 IntelliJ IDEA + sbt
    └── Analizador de calificaciones
        ├── funciones
        ├── Array
        ├── List
        ├── while
        ├── if / else if / else
        ├── compilación con sbt
        └── ejecución desde IntelliJ IDEA
```

