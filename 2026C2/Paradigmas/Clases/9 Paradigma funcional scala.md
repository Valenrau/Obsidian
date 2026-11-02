# Paradigma Funcional y Programación en Scala: Guía Técnica y Práctica

## 1. Fundamentos del Paradigma Funcional y Transparencia Referencial

El **paradigma de programación funcional** es un enfoque declarativo basado en el concepto matemático de **función**. En lugar de estructurar un programa como una secuencia de instrucciones imperativas que modifican progresivamente el estado en memoria, la programación funcional concibe la ejecución de un sistema como la **evaluación y composición de funciones puras**.

```
Paradigma Imperativo:  Estado Inicial ---> Instrucción 1 ---> Instrucción 2 ---> Estado Mutado
Paradigma Funcional:   Entrada ---------> [ Función Pura ] ------------------> Resultado Nuevo
```

### Características Principales del Paradigma

1. **Programas como Funciones**: Un sistema se compone uniendo funciones donde el cálculo equivale a la reducción o evaluación de expresiones, asentado sobre las bases formales del Cálculo Lambda.
2. **Funciones Puras**: Funciones que dependen únicamente de los parámetros de entrada declarados en su firma, sin leer ni alterar estado externo.
3. **Datos Inmutables**: Una vez creada una estructura de datos o variable, su contenido no puede modificarse en memoria.
4. **Funciones de Primera Clase**: Las funciones son tratadas como valores de primer orden, pudiendo asignarse a variables, pasarse como argumentos y devolverse desde otras funciones.
5. **Funciones de Orden Superior (HOF)**: Funciones que operan tomando otras funciones como parámetros o retornándolas.
6. **Composición de Funciones**: Habilidad de encadenar la salida de una función con la entrada de otra para construir operaciones complejas.
7. **Recursividad**: Ausencia de bucles imperativos (`while`, `for` tradicionales mutables) sustituidos por recursión sobre estructuras de datos.

### Transparencia Referencial y Funciones Puras
Una expresión posee **transparencia referencial** si puede ser reemplazada por su valor evaluado sin alterar en absoluto el comportamiento ni el resultado final del programa. Para que esto sea posible, la función subyacente debe ser **pura**:

* **Determinismo**: Dada la misma entrada, retorna indefectiblemente el mismo resultado.
* **Ausencia de Efectos Secundarios (Side Effects)**: No modifica variables globales, no altera estructuras recibidas por referencia, no realiza escrituras I/O en consola o disco, ni lanza excepciones no controladas.

#### Beneficios de la Inmutabilidad y la Pureza:
* **Concurrencia y Paralelismo Seguros**: Al no existir estado mutable compartido, desaparecen de raíz las condiciones de carrera (*race conditions*) y la necesidad de bloqueos o locks.
* **Testabilidad y Debugging**: Cada función puede probarse de forma aislada sin preparar un estado global previo.
* **Memoización (Caching)**: Dado el determinismo, el resultado de una función costosa puede almacenarse en caché según su entrada.

---

### Metáfora Didáctica: La Máquina de Jugos Sellada
Una **función pura** se asemeja a una máquina extractora de jugo completamente sellada al vacío. Si ingresas tres manzanas por la ranura superior, la máquina procesa la fruta y entrega exactamente un vaso de jugo por la salida inferior. La máquina no mancha la mesada (sin *side effects*), no depende de si afuera es de día o de noche (no consulta estado externo) y, si vuelves a introducir tres manzanas idénticas mañana, obtendrás exactamente el mismo vaso de jugo (determinismo).

Por el contrario, un enfoque imperativo impreciso sería un extractor abierto que salpica las paredes, cambia el sabor del jugo según la temperatura ambiente y modifica el color de los vasos guardados en el mueble continuo.

---

### Ejemplo Práctico / Técnico: Función Pura vs. Impura en Scala

#### Código Impuro (Efectos Secundarios y Estado Global):
```scala
// Mal diseño: Depende y muta estado externo
var totalAcumulado: Int = 0

def agregarMontoImpuro(monto: Int): Int = {
  totalAcumulado += monto // Mutación de variable externa (Side Effect)
  totalAcumulado
}
```

#### Código Puro en Scala (Inmutable y Transparente Referencialmente):
```scala
// Diseño Funcional Puro: Inmutable y determinístico
def calcularNuevoTotal(totalActual: Int, monto: Int): Int = {
  totalActual + monto // Retorna un nuevo valor sin modificar totalActual
}

// Uso y transparencia referencial
val inicial = 100
val resultado1 = calcularNuevoTotal(inicial, 50) // 150
val resultado2 = calcularNuevoTotal(inicial, 50) // 150 (Siempre idéntico)
```

---

## 2. Colecciones e Combinadores de Orden Superior

En Scala, las colecciones (como `List`, `Vector` o `Set`) son **inmutables por defecto**. En lugar de iterar manualmente modificando índices, se utilizan **combinadores** o **funciones de orden superior** que aplican transformaciones sobre los elementos retornando una nueva colección.

### Catálogo de Combinadores Principales

| Combinador | Entrada de la Función | Comportamiento Estructural | Tipo de Retorno |
| :--- | :--- | :--- | :--- |
| **`map`** | `A => B` | Transforma cada elemento $A$ en $B$. Preserva el tamaño de la colección. | `Collection[B]` |
| **`filter`** | `A => Boolean` | Conserva únicamente los elementos que satisfacen el predicado booleano. | `Collection[A]` |
| **`reduce`** | `(A, A) => A` | Combina elementos de izquierda a derecha. Falla si la lista está vacía. | `A` |
| **`fold` / `foldLeft`** | `(B, A) => B` | Combina elementos arrancando desde un valor inicial explícito. | `B` |
| **`groupBy`** | `A => K` | Agrupa elementos según la clave $K$ calculada para cada uno. | `Map[K, Collection[A]]` |
| **`flatten`** | N/A | Colapsa un nivel de anidamiento en colecciones embebidas (`List[List[A]]`). | `Collection[A]` |
| **`flatMap`** | `A => Collection[B]` | Aplica `map` y realiza `flatten` del resultado en un único paso. | `Collection[B]` |

---

### Metáfora Didáctica: La Línea de Ensamblaje Industrial
Imaginemos una cinta transportadora en una fábrica de frutas:
* **`map`** es una estación que toma cada manzana de la cinta y le coloca un sello de calidad. Si entraron 10 manzanas, salen 10 manzanas selladas.
* **`filter`** es un sensor óptico que descarta las manzanas verdes y deja pasar solo las rojas.
* **`groupBy`** es un separador robótico que empuja las manzanas rojas hacia un cajón y las verdes hacia otro.
* **`fold`** es una prensa hidráulica con una caja inicial: arranca con una caja de 1 kg y va aplastando cada manzana entrante sumando su peso al total acumulado.

---

### Ejemplo Práctico / Técnico: Procesamiento de Transacciones en Scala

El siguiente código procesa un listado de transacciones financieras para calcular el total gastado por categoría en montos mayores a $100:

```scala
case class Transaccion(categoria: String, monto: Double)

val transacciones = List(
  Transaccion("Alimentos", 150.0),
  Transaccion("Electrónica", 1200.0),
  Transaccion("Alimentos", 80.0),
  Transaccion("Servicios", 200.0),
  Transaccion("Electrónica", 450.0)
)

// 1. Filtrar transacciones mayores a 100
val filtradas = transacciones.filter(t => t.monto > 100.0)

// 2. Agrupar por categoría
val agrupadas: Map[String, List[Transaccion]] = filtradas.groupBy(t => t.categoria)

// 3. Transformar los valores del Map sumando los montos mediante foldLeft
val totalPorCategoria: Map[String, Double] = agrupadas.map { case (cat, lista) =>
  val sumaMonto = lista.foldLeft(0.0)((acum, t) => acum + t.monto)
  (cat, sumaMonto)
}

// Resultado: Map("Alimentos" -> 150.0, "Electrónica" -> 1650.0, "Servicios" -> 200.0)
```

---

## 3. Pattern Matching y Tipos Algebraic (ADTs)

El **Pattern Matching** en Scala es una expresión evaluable que extiende el concepto tradicional de `switch` hacia un mecanismo de inspección estructural, verificación de tipos e inferencia de datos.

### Características Clave
* **Expresividad**: Retorna un valor (es una expresión, no una sentencia).
* **Desestructuración**: Permite "desarmar" el objeto inspeccionado para extraer sus componentes internos.
* **Guardas (`guards`)**: Condicionales adicionales `if` dentro del patrón.
* **Exhaustividad en el Compilador**: Al combinar `match` con **`sealed trait`**, el compilador de Scala verifica que todos los casos posibles estén cubiertos, emitiendo una advertencia (*warning*) si falta alguno.

### Clases de Caso (`case class`) y Jerarquías Cerradas (`sealed trait`)
* **`case class`**: Modificador de clases que autogenera inmutabilidad por defecto, comparación de igualdad por valor (`==`), desestructuración para pattern matching y el método `.copy()` para clonar objetos modificando atributos específicos.
* **`sealed trait`**: Interfaz o rasgo abstracto que solo puede ser extendido por clases definidas dentro del **mismo archivo fuente**, creando un **Tipo de Dato Algebraico (ADT)** cerrado.

---

### Metáfora Didáctica: El Escáner y la Matriz de Geometría
El `sealed trait` es como un juego de piezas de madera que solo contiene dos formas permitidas: Círculo y Rectángulo. El `pattern matching` actúa como una matriz calada de clasificación. Cuando introduces una figura desconocida, la matriz prueba la forma (¿es Círculo?), desensambla su medida interna (obtiene el radio) y la envía por el riel correspondiente. Si intentas agregar una forma de triángulo sin actualizar la matriz, el control de calidad de la fábrica (el compilador) suena una alarma advirtiendo que la matriz no es exhaustiva.

---

### Ejemplo Práctico / Técnico: Evaluador de Expresiones Matemáticas (ADT)

```scala
// 1. Definición del ADT con sealed trait y case classes
sealed trait Expresion
case class Valor(numero: Double) extends Expresion
case class Suma(izq: Expresion, der: Expresion) extends Expresion
case class Multiplicacion(izq: Expresion, der: Expresion) extends Expresion
case class Division(izq: Expresion, der: Expresion) extends Expresion

// 2. Función evaluadora exhaustiva con Pattern Matching y Recursión
def evaluar(expr: Expresion): Double = expr match {
  case Valor(n) => n
  case Suma(e1, e2) => evaluar(e1) + evaluar(e2)
  case Multiplicacion(e1, e2) => evaluar(e1) * evaluar(e2)
  case Division(e1, e2) => 
    val denom = evaluar(e2)
    if (denom == 0) throw new ArithmeticException("División por cero")
    else evaluar(e1) / denom
}

// 3. Ejemplo de uso (Expresión: (5 + 3) * 2)
val arbolExpr: Expresion = Multiplicacion(Suma(Valor(5), Valor(3)), Valor(2))
val resultado = evaluar(arbolExpr) // Retorna 16.0
```

---

## 4. Recursión de Cola (Tail Recursion) y Evaluación Perezosa (Lazy Evaluation)

### Recursión de Cola (`@tailrec`)
Dado que los lenguajes funcionales prescinden de los bucles mutables, la repetición de tareas se realiza mediante recursión. Sin embargo, una recursión estándar acumula marcos de pila (*stack frames*) por cada llamada pendiente, arriesgando un desbordamiento de memoria (**`StackOverflowError`**).

La **recursión de cola** es una optimización donde la llamada recursiva es la **última instrucción exacta** que ejecuta la función antes de retornar. El compilador de Scala la optimiza transformándola internamente en un bucle imperativo a nivel de bytecode de la JVM. Se utiliza la anotación `@scala.annotation.tailrec` para exigir al compilador que verifique dicha optimización.

### Evaluación Perezosa (`Lazy Evaluation`) y *Call-by-Name*
Por defecto, Scala utiliza evaluación estricta (*Call-by-Value*): los argumentos de una función se evalúan en el momento en que son pasados.

* **`call-by-name` (`param: => Tipo`)**: Pasa una expresión sin evaluar. La expresión se evalúa **cada vez** que se menciona dentro del cuerpo de la función.
* **`lazy val`**: Retarda la evaluación de una variable hasta su primer acceso explícito. El resultado se **memoriza (cachea)** para accesos posteriores.
* **`LazyList`**: Colección perezosa evaluada por demanda, lo que permite representar secuencias teóricamente **infinitas**.

---

### Metáfora Didáctica: El Tramite en Ventanilla vs. Pedido por Encargo
La recursión común es como una fila de correos donde cada empleado sostiene un paquete y espera a que el empleado de al lado termine para poder soltar el suyo. Si la fila es muy larga, la oficina se llena de gente y colapsa (`StackOverflowError`). La **recursión de cola** con acumulador es como un solo empleado que va anotando los datos en una libreta (acumulador) y le pasa la libreta limpia al siguiente; nunca se junta gente esperando en el mostrador.

Por otro lado, la **evaluación perezosa (`lazy`)** es como un chef que prepara un platillo gourmet únicamente cuando el cliente lo pide desde la mesa, en lugar de cocinar 100 platillos al abrir el restaurante "por si acaso".

---

### Ejemplo Práctico / Técnico: Recursión de Cola y Secuencias Infinitas

#### 1. Factorial con Recursión de Cola (`@tailrec`):
```scala
import scala.annotation.tailrec

def factorial(n: Int): BigInt = {
  @tailrec
  def acumular(actual: Int, acum: BigInt): BigInt = {
    if (actual <= 1) acum
    else acumular(actual - 1, acum * actual) // Última instrucción pura
  }
  
  acumular(n, 1)
}

val f100 = factorial(100) // Se ejecuta en O(1) espacio de pila
```

#### 2. Serie Infinita de Fibonacci con `LazyList`:
```scala
// Generación perezosa e infinita de Fibonacci
def fibonacci(a: BigInt = 0, b: BigInt = 1): LazyList[BigInt] = {
  a #:: fibonacci(b, a + b) // Operador #:: construye la lista de forma lazy
}

val fiboStream = fibonacci()

// No evalúa la secuencia completa; solo toma los primeros 5 elementos
val primerosCinco = fiboStream.take(5).toList // List(0, 1, 1, 2, 3)
```

---

## 5. Aplicación Parcial y Currying

### Aplicación Parcial
La **aplicación parcial** de una función consiste en fijar un subconjunto de sus parámetros de entrada, produciendo una **nueva función** que acepta únicamente los parámetros restantes.

### Currying (Currificación)
El **Currying** es un caso particular de aplicación parcial donde una función que acepta $n$ argumentos se transforma en una cadena secuencial de $n$ funciones, cada una de las cuales acepta exactamente **un único argumento**:

$$(X 	imes Y) 	o Z \quad \implies \quad X 	o (Y 	o Z)$$

En Scala, el currying se expresa nativamente mediante **múltiples listas de parámetros**: `def f(a: Int)(b: String): Boolean`.

#### Utilidad de Currying:
1. Permite fijar configuraciones generales en la primera lista de parámetros y pasar datos cambiantes en las posteriores.
2. Facilita la **inferencia de tipos** en bloques de código pasados como lambdas (`{ ... }`).

---

### Metáfora Didáctica: La Impresora de Sellos Personalizados
Pensar en una función tradicional de dos parámetros `(Color, Texto)` es como tener una imprenta donde debes especificar el bote de tinta y el molde del texto al mismo tiempo para imprimir cada hoja. 

**Currificación** es configurar primero la imprenta con tinta azul (fijar el primer parámetro). El resultado de esa acción es una "máquina selladora azul" lista para usar. Ahora, a esa nueva máquina solo necesitas pasarle la hoja con el texto que deseas imprimir.

---

### Ejemplo Práctico / Técnico: Sistema de Formateo y Log

```scala
// 1. Función tradicional de dos parámetros
def formatearMensaje(nivel: String, mensaje: String): String = {
  s"[$nivel] - $mensaje"
}

// 2. Función Currificada nativa en Scala (Múltiples listas de parámetros)
def logCurried(nivel: String)(mensaje: String): String = {
  s"[$nivel] - $mensaje"
}

// 3. Aplicación Parcial fijando el nivel de Log
val logError: String => String = logCurried("ERROR")
val logInfo: String => String = logCurried("INFO")

// Uso simplificado:
val msj1 = logError("Conexión rehusada a la base de datos") // [ERROR] - Conexión rehusada...
val msj2 = logInfo("Servicio iniciado en puerto 8080")       // [INFO] - Servicio iniciado...

// Conversión explícita con .curried
val formatearCurried = (formatearMensaje _).curried
val logWarn = formatearCurried("WARN")
```

---

## 6. Mónadas y Manejo Seguro de Efectos (`Option`, `Either`, `Try`)

En la programación puramente funcional no se utilizan referencias nulas (`null`) ni se lanzan excepciones no controladas mediante `throw`, ya que estas destruyen la transparencia referencial. En su lugar, se utilizan **Mónadas**.

### ¿Qué es una Mónada?
Una **Mónada** es un contenedor abstracto que envuelve un valor dentro de un **contexto determinado** y define cómo aplicarle transformaciones de forma segura sin romper dicho contexto.

Para ser considerada una mónada, la estructura debe proveer principalmente dos operaciones fundamentalmente asociadas a las leyes monádicas:
1. **`unit` / `pure`**: Coloca un valor dentro del contenedor monádico (ej. `Option(v)`).
2. **`flatMap`**: Aplica una función que toma el valor contenido y retorna una **nueva mónada**, aplanando el resultado para evitar contenedores anidados (`M[M[A]]` -> `M[B]`).

### Mónadas Estándar en Scala

* **`Option[T]`**: Encapsula la presencia (`Some(v)`) o ausencia (`None`) de un valor, eliminando los `NullPointerException`.
* **`Either[A, B]`**: Encapsula dos posibles resultados: una falla del lado izquierdo (`Left(a)`) o un éxito del lado derecho (`Right(b)`).
* **`Try[T]`**: Encapsula el resultado de una operación computacional que puede lanzar una excepción (`Success(v)` o `Failure(exception)`).

---

### Metáfora Didáctica: El Contenedor de Materiales Peligrosos
Una mónada es como una **caja blindada con manipuladores de guantes integrados**. Si tienes un reactivo químico peligroso (un valor que puede ser nulo o lanzar un error), no lo tocas con las manos descubiertas. Lo colocas dentro de la caja blindada (`Option`). Si deseas mezclar ese reactivo con otro ingrediente (`map` / `flatMap`), introduces las manos por los guantes y haces la mezcla dentro de la caja. Si la caja estaba vacía (`None`), la operación de mezclado simplemente no hace nada de forma segura sin explotar en la cara del operador.

---

### Ejemplo Práctico / Técnico: Encadenamiento de Operaciones con `Option` y `for-comprehension`

Dado un sistema de usuarios donde la búsqueda en base de datos y la extracción de email pueden fallar o retornar nulo:

```scala
case class Usuario(id: Int, nombre: String, email: Option[String])

// Simuladores de base de datos
def buscarUsuario(id: Int): Option[Usuario] = {
  if (id == 1) Some(Usuario(1, "Ana", Some("ana@empresa.com")))
  else if (id == 2) Some(Usuario(2, "Pedro", None)) // Usuario sin email registrado
  else None // Usuario inexistente
}

def formatearEmail(email: String): String = email.trim.toLowerCase

// 1. Encadenamiento monádico clásico usando flatMap y map
def obtenerEmailDominio(id: Int): Option[String] = {
  buscarUsuario(id).flatMap(usr => usr.email).map(email => formatearEmail(email))
}

// 2. Sintaxis de Azúcar Sintáctico: for-comprehension (Equivalente a flatMap/map)
def obtenerEmailDominioElegante(id: Int): Option[String] = {
  for {
    usuario <- buscarUsuario(id)
    emailRaw <- usuario.email
  } yield formatearEmail(emailRaw)
}

// Evaluación de resultados:
val res1 = obtenerEmailDominioElegante(1) // Some("ana@empresa.com")
val res2 = obtenerEmailDominioElegante(2) // None (Manejo limpio de ausencia)
val res3 = obtenerEmailDominioElegante(99) // None (Manejo limpio de usuario inexistente)
```

---

## 7. Tabla Resumen e Integración de Conceptos

### Matriz Comparativa de Mecanismos del Paradigma Funcional

| Concepto | Propósito Principal | Problema Imperativo que Resuelve | Mecanismo en Scala |
| :--- | :--- | :--- | :--- |
| **Inmutabilidad** | Eliminar la mutación de estado compartido | Condición de carrera (*race conditions*) y acoplamiento temporal | `val`, `case class`, Colecciones inmutables |
| **Funciones Puras** | Garantizar transparencia referencial y determinismo | Efectos secundarios incontrolados (*side effects*) y código difícil de testear | `def` sin lectura/escritura de estado mutable |
| **Combinadores** | Declarar *qué* transformar sobre listas | Bucles manuales `for`/`while` con índices mutables acumulativos | `map`, `filter`, `foldLeft`, `groupBy` |
| **Pattern Matching** | Inspección estructural y desestructuración | Bloques `switch` rígidos y cascadas complejas de `if/else` | Expresión `match { case ... }` |
| **Sealed Traits (ADTs)**| Definir jerarquías cerradas de datos exhaustivas | Polimorfismo descontrolado en clases abiertas | `sealed trait` + `case class` |
| **Tail Recursion** | Iterar inductivamente sin agotar memoria | Desbordamiento de pila (*StackOverflowError*) en recursión | Anotación `@tailrec` optimizada a bucle JVM |
| **Lazy Evaluation** | Diferir cálculos costosos o infinitos | Carga prematura de datos no utilizados en memoria | `lazy val`, `param: => T`, `LazyList` |
| **Currying** | Dividir funciones multivariable en secuencias de 1 parámetro | Dificultad para fijar configuraciones y reutilizar funciones | Múltiples listas de parámetros `def f(a)(b)` |
| **Mónadas** | Encapsular valores y manejar efectos de forma segura | Excepciones no atrapadas y errores de puntero nulo (`NullPointerException`) | `Option`, `Either`, `Try` con `flatMap` |
