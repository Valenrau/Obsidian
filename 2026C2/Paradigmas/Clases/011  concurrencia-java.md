# Concurrencia y Sincronización en Java: Guía Técnica y Práctica de Arquitectura Multihilo

---

## 1. Fundamentos de Concurrencia, Paralelismo y la CPU

La computación moderna requiere que el software no solo sea funcionalmente correcto, sino también capaz de responder en tiempos admisibles frente a múltiples demandas de procesamiento. Para comprender cómo se ejecutan las aplicaciones actuales, es imprescindible analizar la relación entre las instrucciones de software, los hilos de ejecución y el hardware subyacente.

### ¿Qué es un Thread (Hilo de Ejecución)?
Un **thread** (o hilo de ejecución) es la abstracción fundamental de una ejecución secuencial de instrucciones de código [T18, p. 5]. En un programa puramente lineal, el flujo de control avanza de una línea de código a la siguiente en un orden estrictamente predecible. El thread constituye la unidad básica de procesamiento asignada por el sistema para despachar instrucciones a la unidad central de procesamiento (CPU) [T18, p. 5, 22].

A nivel de arquitectura de memoria en un sistema operativo, un proceso puede ser **Single-Threaded** (un único hilo) o **Multi-Threaded** (múltiples hilos conviviendo en el mismo espacio) [T18, p. 7, 22]:
* **Memoria Compartida del Proceso**: Todos los threads de un mismo proceso comparten el segmento de código (*code*), el segmento de datos globales/estáticos (*data*), el montículo (*heap*) y los descriptores de archivos abiertos (*files*) [T18, p. 7, 22].
* **Memoria Privada del Thread**: Cada thread cuenta de forma exclusiva con su propio conjunto de **registros de CPU** (que almacenan el estado del procesador y el *Program Counter*) y una **pila de memoria propia (*stack*)** para almacenar variables locales y cuadros de invocación de métodos [T18, p. 7, 22].

Esta separación interna actúa como una suerte de aislamiento por diseño (*sandbox*): si un hilo comete un error en sus variables locales, no corrompe la pila de llamadas de los demás hilos; sin embargo, al compartir el *heap*, cualquier modificación sobre variables u objetos compartidos puede provocar anomalías catastróficas si no se sincroniza adecuadamente.

```
+-------------------------------------------------------------------+
|                        PROCESO (Process)                          |
|  +--------------------+  +-------------------+  +--------------+  |
|  | Código (Code)      |  | Datos (Data/Heap) |  | Files / I/O  |  |
|  +--------------------+  +-------------------+  +--------------+  |
|                                                                   |
|  +--------------------+  +-------------------+  +--------------+  |
|  | Thread 1           |  | Thread 2          |  | Thread N     |  |
|  | - Registros CPU    |  | - Registros CPU   |  | - Registros  |  |
|  | - Stack privado    |  | - Stack privado   |  | - Stack      |  |
|  +--------------------+  +-------------------+  +--------------+  |
+-------------------------------------------------------------------+
```

---

### Concurrencia vs. Paralelismo
Una de las distinciones conceptuales más críticas en la ingeniería de software radica en diferenciar la concurrencia del paralelismo [T18, p. 10-11, 21]:

* **Concurrencia**: Implica **lidiar con múltiples tareas al mismo tiempo**, pero no necesariamente ejecutarlas en el mismo instante cronológico [T18, p. 10, 21]. En un entorno concurrente sobre un único núcleo físico (core), el sistema operativo alterna rápidamente la ejecución de distintos threads (mediante una técnica conocida como *time-slicing* o multitarea) [T18, p. 11, 13]. Para el usuario humano —cuya percepción visual opera en el rango de los milisegundos mientras que el procesador conmuta en nanosegundos ($10^{-9}$ s)— se crea la **ilusión perceptiva** de que todo ocurre a la vez, aunque en un instante infinitesimal $t$ solo hay una instrucción ejecutándose en ese núcleo [T18, p. 10-11].
* **Paralelismo**: Implica **ejecutar múltiples tareas en simultáneo real** [T18, p. 10, 21]. En un instante de tiempo $t$, existen físicamente dos o más instrucciones procesándose en paralelo. Esto exige estrictamente que el hardware cuente con dos o más núcleos (cores) físicos independientes [T18, p. 11-12].

| Dimensión | Concurrencia | Paralelismo |
| :--- | :--- | :--- |
| **Definición** | Estructura del programa para gestionar múltiples flujos. | Ejecución física simultánea de múltiples cálculos. |
| **Hardware Requerido** | Funciona en 1 solo núcleo (o en múltiples). | Requiere estrictamente 2 o más núcleos físicos (cores). |
| **Instante de tiempo $t$** | Solo una tarea se ejecuta por core físico. | Múltiples tareas ejecutándose exactamente a la vez. |
| **Mecanismo Central** | Alternancia rápida (*Interleaving* / Multitasking). | Procesamiento distribuido en silicio independiente. |
| **Enfoque Arquitectónico** | Modularidad y gestión de tareas asincrónicas/I-O. | Rendimiento bruto y reducción de tiempo de CPU. |

---

### Metáfora Didáctica: Las Ventanillas de Atención Bancaria
Imaginemos una oficina de atención al público:
* **Concurrencia (Un solo core)**: Hay **una única ventanilla de atención** atendida por un único empleado. El empleado atiende a un cliente durante 30 segundos; mientras ese cliente saca sus documentos de la mochila, el empleado le dice que espere, atiende a un segundo cliente, sella un formulario, vuelve al primero y luego atiende a un tercero. Las tres gestiones están avanzando en el mismo período, pero el empleado solo puede hablar o firmar con una persona por segundo. Si un cliente se traba buscando un papel, el empleado atiende a otro.
* **Paralelismo (Múltiples cores)**: El banco abre **cuatro ventanillas independientes** con cuatro empleados dedicados. Cuatro clientes son atendidos física y simultáneamente en el mismo segundo de reloj.

---

### La CPU, los Cores y el Planificador (Scheduler)
El hardware real encargado de la ejecución son los **cores** de la CPU [T18, p. 9]. Cada core puede verse como un obrero que ejecuta operaciones de máquina una a una [T18, p. 12].

1. **¿Puede una computadora correr más de un proceso/thread a la vez?** Sí, en paralelo real puede ejecutar tantos threads como núcleos físicos posea [T18, p. 12].
2. **¿Puede un core correr más de un thread a la vez?** No en el sentido físico estricto de la palabra; un core ejecuta una sola instrucción por ciclo, pero realiza *multitasking* alternando entre ellos a frecuencias altísimas [T18, p. 12, 13].
3. **¿Tener más núcleos siempre mejora linealmente el rendimiento?** No es una relación estrictamente lineal [T18, p. 12]. Coordinar, comunicar y asignar hilos a distintos cores requiere un esfuerzo de gestión del sistema operativo y los buses de memoria compartida, introduciendo una sobrecarga (*overhead*) inevitable [T18, p. 12].

El componente del sistema operativo que orquesta esta dinámica es el **Planificador (*Scheduler*)** [T18, p. 13]. El scheduler decide a qué thread se le asigna un núcleo y durante cuánto tiempo (un *quantum* de procesamiento, por ejemplo unos microsegundos o nanosegundos). Cuando este tiempo expira o el thread se bloquea, el scheduler realiza un **cambio de contexto (*context switch*)**: guarda los registros del hilo saliente en memoria, carga los registros del nuevo hilo y salta a su instrucción. 

El scheduler también mantiene deliberadamente **tiempos ociosos (*idle time*)**, reservando capacidad de CPU para responder a eventos del sistema o interrupciones imprevistas [T18, p. 13-14].

#### El Costo Oculto de la Concurrencia (CPU-Bound vs. I/O-Bound)
Intercalar tareas **no es gratis** [T18, p. 12]. El *context switch*, la invalidación de memorias caché y la lógica del scheduler consumen tiempo y ciclos de reloj.
* Si una tarea está limitada por Entrada/Salida (**I/O-Bound**, como esperar datos de red, discos o bases de datos), la concurrencia ofrece mejoras gigantescas: cuando un hilo se frena esperando el disco, el núcleo procesa otro hilo en lugar de desperdiciar ciclos ociosos [T18, p. 16].
* Si una tarea está limitada exclusivamente por procesamiento matemático (**CPU-Bound**, como cálculos con matrices densas o compresión pura de imágenes): crear hilos adicionales sobre un mismo núcleo no reduce el tiempo total; de hecho, lo incrementa debido a la sobrecarga administrativa del scheduler [T18, p. 12].

---

### Beneficios y Desafíos de la Concurrencia

#### Beneficios Clave [T18, p. 16]:
1. **Mejora del Rendimiento**: Distribución de cargas paralelizables en múltiples núcleos físicos.
2. **Optimización de Recursos**: Aprovechamiento de los tiempos muertos de I/O para no dejar la CPU inactiva.
3. **Construcción de Sistemas Complejos y Escalables**: Capacidad de dar servicio a miles o millones de usuarios concurrentes en servidores modernos, desacoplando la interfaz de usuario de las tareas de fondo pesadas.

#### Desafíos Críticos [T18, p. 17]:
1. **No-Determinismo**: El orden de entrelazado de instrucciones lo decide el scheduler del sistema operativo y la carga en tiempo real; dos ejecuciones con los mismos datos pueden arrojar órdenes temporales diferentes.
2. **Acceso a Memoria Compartida**: Cuando múltiples hilos acceden y modifican el mismo espacio de memoria, surgen condiciones de carrera y corrupciones de datos invisibles al ojo humano.
3. **Dificultad de Depuración**: Los errores concurrentes son transitorios (*heisenbugs*): al intentar reproducirlos paso a paso con un depurador, la temporización se altera y el error desaparece.

---

## 2. Propiedades de Correctitud y Sección Crítica

En un programa secuencial tradicional, la correctitud se evalúa de manera sencilla: ante una entrada determinada, el sistema debe producir la salida esperada respetando los efectos secundarios [T19, p. 3]. En la programación concurrente, debido al no-determinismo, la correctitud se divide en dos categorías teóricas fundamentales postuladas por Leslie Lamport: **Propiedades de Safety** y **Propiedades de Liveness** [T19, p. 3].

```
                     PROPIEDADES DE CORRECTITUD
                                 |
         +-----------------------+-----------------------+
         |                                               |
  PROPIEDADES SAFETY                             PROPIEDADES LIVENESS
  "Nunca ocurre nada malo"                       "Eventualmente ocurre algo bueno"
  (Deben ser válidas siempre)                    (Deben cumplirse en el tiempo)
  - Exclusión Mutua                              - Ausencia de Starvation
  - Ausencia de Deadlocks                        - Fairness (Equidad de turnos)
```

### Propiedades de Tipo Safety (Seguridad)
Establecen que **"nada malo va a suceder durante la ejecución"**. Deben mantenerse verdaderas en cualquier punto y estado del ciclo de vida del programa [T19, p. 3]:
1. **Exclusión Mutua (*Mutual Exclusion*)**: Dos o más procesos/hilos no deben intercalar jamás instrucciones pertenecientes a sus secciones críticas [T19, p. 3, 5]. En todo instante $t$, como máximo un solo hilo puede encontrarse modificando la estructura compartida.
2. **Ausencia de Deadlocks (*Interbloqueo*)**: Un sistema que aún no ha finalizado debe ser capaz de continuar avanzando; no pueden existir conjuntos de hilos bloqueados mutuamente esperando recursos retenidos entre sí [T19, p. 3, 5].

### Propiedades de Tipo Liveness (Vivacidad)
Establecen que **"eventualmente algo bueno sucederá"**. No basta con que el sistema no se rompa; el programa debe realizar progreso útil en el tiempo [T19, p. 3]:
1. **Ausencia de Starvation (*Inanición*)**: Todo proceso o hilo que esté listo para utilizar un recurso debe poder obtenerlo eventualmente [T19, p. 3, 5]. Ningún hilo debe quedar postergado indefinidamente en una cola de espera mientras otros acaparan el procesador.
2. **Fairness (*Equidad*)**: El sistema debe garantizar un reparto justo y equitativo de los recursos de cómputo entre todos los procesos que compiten por ellos [T19, p. 3].

---

### La Sección Crítica (SC)
Formalmente, si consideramos $N$ procesos ejecutándose concurrentemente en bucles continuos, las instrucciones de cada proceso se dividen en dos categorías [T19, p. 4]:
* **Instrucciones No Críticas (no-SC)**: Procesamiento puramente local, transformaciones sobre variables privadas de la pila, cálculos matemáticos independientes.
* **Instrucciones de la Sección Crítica (SC)**: Fragmentos de código donde el hilo accede, lee o escribe sobre **estructuras de datos compartidas** en el *heap* o recursos externos compartidos (archivos, conexiones a base de datos, puertos) [T19, p. 4].

Para garantizar la correctitud, las operaciones sobre la sección crítica deben ser **atómicas**: o se ejecutan por completo como un bloque indivisible, o no se ejecutan; no pueden quedar en estados intermedios observables por otros hilos [T19, p. 4]. Por ello, el código concurrente se estructura bajo un protocolo estandarizado [T19, p. 6]:

```
loop forever {
    // 1. Código no crítico (cálculos locales, preparación)
    ...
    // 2. Pre-protocolo de sincronización (solicitar permiso / adquirir lock)
    ...
    // 3. SECCIÓN CRÍTICA (SC) (acceso a memoria o recurso compartido)
    ...
    // 4. Post-protocolo de sincronización (liberar permiso / ceder lock / notificar)
    ...
}
```

---

### Patología 1: Race Condition (Condición de Carrera)
Una **condición de carrera** ocurre cuando dos o más hilos compiten por acceder y modificar un recurso compartido simultáneamente, haciendo que el resultado final dependa del orden arbitrario en el que el scheduler intercaló sus instrucciones [T18, p. 19; T19, p. 4].

#### El Fenómeno del "Lost Update" (Actualización Perdida)
El caso más común ocurre con una simple operación de incremento: `contador++`. A nivel de lenguaje parece una única sentencia, pero a nivel de arquitectura de computadoras se descompone en tres pasos de bajo nivel [T18, p. 19]:
1. **Lectura (Read)**: Se carga el valor actual de la memoria al registro interno del procesador.
2. **Modificación (Increment)**: La Unidad Aritmético Lógica (ALU) le suma 1 al registro.
3. **Escritura (Write)**: Se escribe el nuevo valor del registro de regreso a la dirección de memoria.

Si dos hilos ($A$ y $B$) intentan incrementar una variable que inicialmente vale `17` sin sincronización [T18, p. 19]:
* El Hilo $A$ lee `17` en su registro local.
* Antes de que $A$ escriba, el scheduler interrumpe a $A$ y le da la CPU al Hilo $B$.
* El Hilo $B$ lee de memoria el valor `17` (que aún no fue modificado).
* El Hilo $B$ incrementa su registro local a `18` y lo escribe en memoria.
* El scheduler retoma al Hilo $A$. $A$ continúa donde quedó: tenía `17` en su registro, lo incrementa a `18` y lo escribe en memoria.
* **Resultado**: Se ejecutaron dos operaciones de incremento, pero el valor final en memoria es `18` en lugar de `19`. La actualización del Hilo $B$ se perdió por completo (*lost update*) [T18, p. 19].

#### Metáfora Didáctica: La Libreta del Departamento Compartido
Dos compañeros de departamento quieren actualizar la lista de gastos anotada en una libreta sobre la heladera. Ambos se acercan, ven que el total acumulado dice $10.000. El primero va a su cuarto a buscar un lápiz para anotar $2.000 (total $12.000). En ese segundo, el otro compañero saca una birome de su bolsillo, tacha $10.000 y anota $15.000 (+ $5.000). Cuando el primer compañero regresa con el lápiz, recuerda que había leído $10.000, le suma sus $2.000, tacha lo que ve y escribe $12.000. Los $5.000 del segundo compañero se esfumaron del registro.

---

### Patología 2: Deadlock (Abrazo Mortal o Interbloqueo)
Un **deadlock** es una condición crítica donde dos o más procesos quedan bloqueados indefinidamente, incapaces de continuar su ejecución, porque cada uno retiene un recurso que el otro necesita para avanzar [T18, p. 20; T19, p. 3].

Para que ocurra un deadlock es estrictamente necesario que participen **al menos dos procesos y al menos dos recursos** [T18, p. 20].

```
    [ Hilo 1 ]  --- Posee --->  ( Recurso R1 )
        |                             ^
    Espera por                    Retenido por
        v                             |
  ( Recurso R2 )  <--- Espera ---  [ Hilo 2 ]
```

#### Las Cuatro Condiciones de Coffman
En 1971, Edward Coffman formalizó que un deadlock solo puede ocurrir si se satisfacen simultáneamente cuatro condiciones necesarias [T19, p. 18]:
1. **Exclusión Mutua**: Los recursos no son compartibles; un recurso solo puede ser utilizado por un proceso a la vez [T19, p. 18].
2. **Retención y Espera (*Hold and Wait*)**: Un proceso retiene al menos un recurso mientras espera adquirir recursos adicionales que están asignados a otros procesos [T19, p. 18].
3. **No Expropiación (*No Preemption*)**: Los recursos no pueden ser quitados por la fuerza a un proceso; solo pueden ser liberados voluntariamente por el hilo que los retiene [T19, p. 18].
4. **Espera Circular (*Circular Wait*)**: Existe una cadena cerrada de procesos $\{P_0, P_1, \dots, P_n\}$ tal que $P_0$ espera un recurso retenido por $P_1$, $P_1$ espera a $P_2$, y $P_n$ espera un recurso retenido por $P_0$ [T18, p. 20; T19, p. 18].

Para resolver o prevenir un deadlock en el diseño de un sistema, **alcanza con invalidar matemáticamente al menos una de estas cuatro condiciones** [T19, p. 18].

#### Metáfora Didáctica: La Encrucijada de Tránsito
Cuatro autos llegan al mismo tiempo a una intersección estrecha sin semáforos desde las cuatro direcciones cardinales [T18, p. 20]. El auto del norte bloquea al del este, el del este al del sur, el del sur al del oeste y el del oeste al del norte. Ningún conductor puede avanzar porque el cruce está ocupado por la trompa del auto de adelante, y ninguno retrocede. Todos quedan paralizados para siempre.

---

### Patología 3: Starvation (Inanición)
A diferencia del deadlock, donde todos los hilos involucrados están trabados sin posibilidad alguna de avanzar, en la **inanición (*starvation*)** el sistema global continúa funcionando y realizando tareas útiles, pero **uno o más hilos específicos quedan bloqueados indefinidamente sin recibir recursos de procesamiento** [T18, p. 20; T19, p. 3, 5].

Esto ocurre comúnmente cuando:
* El planificador o la estructura de sincronización carece de equidad (*fairness*).
* Ciertos hilos poseen prioridades más altas y acaparan sistemáticamente los accesos.
* Un hilo que adquirió un recurso exclusivo realiza operaciones sumamente prolongadas o entra en ciclos que no liberan el lock con la debida celeridad [T18, p. 20].

### Patología 4: Livelock
El **livelock** es el inverso dinámico del deadlock [T19, p. 18]. En un livelock, los hilos **no están bloqueados ni dormidos**; continúan ejecutando código activamente y cambiando de estado, pero su comportamiento está atrapado en un ciclo infinito de reacción recíproca que no genera ningún progreso real [T19, p. 18].

#### Metáfora Didáctica: El Baile de Pasillo
Dos personas caminando en direcciones opuestas se encuentran de frente en un pasillo muy angosto. Para no chocar, ambas personas dan un paso hacia la derecha simultáneamente; al notar que se siguen bloqueando, ambas dan un paso hacia la izquierda al mismo tiempo; luego a la derecha, y así sucesivamente. Ninguna persona está quieta ni bloqueada físicamente, pero ninguna puede avanzar por el pasillo.

---

## 3. Concurrencia Práctica en Java: De la API Base a Locks y Monitores

La plataforma Java incorpora soporte de concurrencia desde su primera versión. Para coordinar los flujos de ejecución, el lenguaje ofrece mecanismos que van desde el control directo de hilos hasta construcciones estructuradas de exclusión mutua.

### Creación y Ciclo de Vida de Threads en Java
En Java, los threads se gestionan mediante la clase `java.lang.Thread` y la interfaz funcional `java.lang.Runnable` [P14, p. 4]. Existen tres maneras tradicionales de instanciar un hilo:

1. **Expresiones Lambda o Funciones Anónimas**: Pasando una implementación directa de `Runnable` al constructor de `Thread` [P14, p. 4].
2. **Implementación de la interfaz `Runnable`**: Creando una clase separada que encapsula la tarea a realizar en su método `run()` [P14, p. 4].
3. **Herencia de la clase `Thread`**: Creando una subclase que sobreescribe directamente el método `run()`.

```java
// Forma estándar moderna mediante Lambda (Runnable)
Thread hilo = new Thread(() -> {
    System.out.println("Hilo secundario en ejecución");
});
```

#### Métodos Fundamentales de la API de Thread [P14, p. 4-5]:
* **`start()`**: Solicita al sistema operativo la creación del hilo de plataforma nativo y programa la invocación de su método `run()` dentro de ese nuevo contexto concurrente [P14, p. 4, 5]. *(Invocar directamente a `run()` NO crea un hilo; simplemente ejecuta el código secuencialmente en el hilo actual)*.
* **`join()`**: Sincronización de cooperación. Bloquea al hilo que realiza la llamada (por ejemplo, el hilo `main`) hasta que el hilo sobre el cual se invoca el `join()` complete su ejecución [P14, p. 4, 5, 8]. Al finalizar, se liberan y devuelven al sistema operativo los recursos asignados al hilo [P14, p. 4, 5].
* **`sleep(ms)`**: Pausa temporalmente la ejecución del hilo actual durante una cantidad determinada de milisegundos, cediendo ciclos de procesador al scheduler [P14, p. 5].
* **`interrupt()`**: Envía una señal de interrupción al hilo; si el hilo se encuentra bloqueado en una llamada a `sleep()` o `wait()`, se lanza una excepción de tipo `InterruptedException` [P14, p. 4, 5].

---

### Ejemplo Práctico: El Orden de Ejecución y el No-Determinismo
El siguiente ejemplo ilustra el no-determinismo intrínseco del scheduler cuando se levantan múltiples hilos simultáneos sin barreras de exclusión mutua [P14, p. 6-7]:

```java
public class OrdenEjecucionDemo {
    public static void main(String[] args) throws InterruptedException {
        // Hilo inicial
        Thread primero = new Thread(() -> System.out.println("Artistas"));
        
        // Tres hilos concurrentes
        Thread a = new Thread(() -> System.out.println("Fito"));
        Thread b = new Thread(() -> System.out.println("Charly"));
        Thread c = new Thread(() -> System.out.println("Flaco"));

        // 1. Iniciar y esperar obligatoriamente al primer hilo
        primero.start();
        primero.join(); // Barrera: garantiza que "Artistas" sea el primer mensaje

        // 2. Disparar los tres hilos concurrentemente
        a.start();
        b.start();
        c.start();

        System.out.println("Gracias totales");

        // 3. Esperar que todos terminen para cerrar el programa
        c.join();
        b.join();
        a.join();
    }
}
```

#### Análisis del Comportamiento [P14, p. 6-7]:
1. **Línea Garantizada**: `"Artistas"` siempre será la primera línea en imprimirse, ya que el hilo `main` se bloquea explícitamente con `primero.join()` antes de continuar.
2. **Líneas Indeterminadas**: El orden de impresión de `"Fito"`, `"Charly"`, `"Flaco"` y `"Gracias totales"` es totalmente impredecible. El hilo `main` puede imprimir `"Gracias totales"` antes de que el procesador le asigne un ciclo a los hilos `a`, `b` o `c`, o los hilos de los artistas pueden imprimirse en cualquier ordenación temporal posible.

---

### Locks Explícitos (`ReentrantLock`)
Para implementar exclusión mutua directa sobre secciones críticas, Java provee la interfaz `java.util.concurrent.locks.Lock` y su implementación principal `ReentrantLock` [P14, p. 9-11; T19, p. 8-9].

Un lock funciona como un candado físico: solo un thread puede poseer el lock en un instante dado. Si otro hilo intenta adquirirlo mediante `lock()`, queda bloqueado en una cola hasta que el poseedor lo libere mediante `unlock()` [P14, p. 10-11; T19, p. 8-9].

#### Patrón Obligatorio de Código en Java: `try-finally`
Debido a que una excepción imprevista dentro de la sección crítica dejaría el lock tomado permanentemente (provocando inanición o deadlock en el resto del sistema), la llamada a `unlock()` debe colocarse **obligatoriamente dentro de un bloque `finally`** [P14, p. 11; T19, p. 9]:

```java
import java.util.concurrent.locks.ReentrantLock;

public class ContadorSeguro {
    private final ReentrantLock lock = new ReentrantLock();
    private int saldo = 0;

    public void depositar(int monto) {
        lock.lock(); // Pre-protocolo: Solicita el candado (bloquea si está ocupado)
        try {
            // SECCIÓN CRÍTICA: Acceso exclusivo a la variable compartida
            saldo += monto;
            System.out.println("Nuevo saldo: " + saldo);
        } finally {
            // Post-protocolo: Garantiza la liberación incluso ante excepciones
            lock.unlock(); 
        }
    }
}
```

---

### Condition Variables (Variables de Condición)
Las **variables de condición (`Condition`)** proporcionan un medio para que un hilo suspenda su ejecución hasta que otro hilo le notifique que una condición lógica de negocio se ha cumplido [P14, p. 12-13; T19, p. 10-11].

En Java, un objeto `Condition` no existe en el vacío; se crea **directamente vinculado a una instancia de `Lock`** mediante `lock.newCondition()` [P14, p. 12; T19, p. 10].

#### Métodos de Control [P14, p. 13; T19, p. 10]:
* **`await()`**: El hilo actual **libera automáticamente el lock asociado** y entra en un estado suspendido (*WAITING*), esperando una señal [P14, p. 13; T19, p. 10].
* **`signal()`**: Despierta a uno de los hilos que se encuentran esperando sobre esa condición [P14, p. 13; T19, p. 10]. El hilo despertado es trasladado de la cola de la condición a la cola del lock, compitiendo nuevamente por adquirirlo antes de continuar [T19, p. 10].
* **`signalAll()`**: Despierta a todos los hilos que están esperando en la condición [P14, p. 13; T19, p. 10].

#### La Necesidad del Bucle `while` en lugar de `if`
Al evaluar la condición antes de invocar `await()`, se debe utilizar siempre una estructura de repetición `while` y jamás un simple `if` [T19, p. 11]:
```java
// CORRECTO: Protege contra despertares espurios y carreras entre hilos
while (buffer == 0) {
    notEmpty.await();
}

// PELIGROSO: Vulnerable a race conditions tras ser notificado
if (buffer == 0) {
    notEmpty.await();
}
```
**Causa Técnica**: Cuando un hilo es despertado mediante `signal()`, no obtiene el lock inmediatamente. Para cuando el hilo finalmente adquiere el lock y se reanuda, otro hilo pudo haber entrado en el medio y consumido el elemento. Además, las arquitecturas de procesadores admiten *despertares espurios* (*spurious wakeups*), donde un hilo puede despertar sin haber recibido señal alguna. El bucle `while` garantiza que la condición sea verificada nuevamente antes de avanzar sobre la sección crítica.

---

### Semáforos (`Semaphore`)
Inventados por Edsger Dijkstra, los semáforos son mecanismos de sincronización basados en un **contador de permisos entero** [T19, p. 12-14]. Mientras que un lock permite el ingreso de un único hilo a la sección crítica, un semáforo permite el acceso de hasta $N$ hilos simultáneos [T19, p. 12].

#### Arquitectura Interna del Semáforo
Internamente, un semáforo puede modelarse mediante un candado de exclusión mutua, una variable de condición y un contador [T19, p. 14]:

```
Pseudocódigo de Implementación:
--------------------------------
mutex lock;
condition cv;
int count = valor_inicial;

function wait() / acquire():
    lock.acquire()
    while count == 0:
        cv.wait(lock)      // Libera el lock y espera
    count = count - 1
    lock.release()

function signal() / release():
    lock.acquire()
    count = count + 1
    cv.signal()            // Notifica a un hilo en espera
    lock.release()
```

#### Operaciones en Java (`java.util.concurrent.Semaphore`) [T19, p. 13]:
* **`acquire()`**: Solicita un permiso. Si el contador es mayor a 0, lo decrementa en 1 y continúa. Si el contador es 0, el hilo se bloquea hasta que otro libere un permiso [T19, p. 12, 13].
* **`release()`**: Devuelve un permiso al semáforo, incrementando el contador en 1 y despertando a hilos bloqueados si los hubiera [T19, p. 12, 13].
* **Semáforo Binario vs. General**: Un semáforo con valor inicial `1` se denomina binario y actúa de forma análoga a un Mutex. Un semáforo con valor inicial $N > 1$ se denomina general o contador [T19, p. 12].

#### Caso de Uso Principal: Pool de Conexiones a Base de Datos
Los semáforos no se utilizan habitualmente para proteger una variable numérica contra condiciones de carrera (para eso se requiere un lock de capacidad 1), sino para **controlar y limitar el acceso a un conjunto finito de recursos costosos (*resource pool*)** [T19, p. 12-13]. 

Por ejemplo, si un servidor web maneja 10.000 peticiones concurrentes pero la base de datos solo admite 20 conexiones simultáneas, se utiliza un semáforo inicializado en `20`. Las peticiones que no consiguen permiso esperan en cola, evitando que la base de datos colapse por sobrecarga de memoria o sufra un ataque de Denegación de Servicio (*DoS*) [T19, p. 13].

---

### Monitores y Locks Intrínsecos (`synchronized`)
Los **monitores** son abstracciones de más alto nivel formuladas por C.A.R. Hoare y Per Brinch Hansen que combinan la exclusión mutua y la sincronización condicional dentro de una estructura unificada [P14, p. 14-15; T19, p. 7].

En Java, **todo objeto posee intrínsecamente un lock asociado y una cola de condición implícita** [P14, p. 14-15; T19, p. 7]. Para utilizar este mecanismo, el lenguaje provee la palabra clave `synchronized`:

```java
public class MonitorCuenta {
    private int saldo = 1000;

    // Métodos sincronizados sobre el lock intrínseco de 'this'
    public synchronized void depositar(int monto) {
        saldo += monto;
        notifyAll(); // Despierta hilos que esperen en el monitor
    }

    public synchronized void retirar(int monto) throws InterruptedException {
        while (saldo < monto) {
            wait(); // Libera el lock intrínseco del objeto y espera
        }
        saldo -= monto;
    }
}
```

#### Características del Monitor en Java [P14, p. 14; T19, p. 7]:
* Al declarar un método con `synchronized`, cualquier hilo que desee invocarlo debe adquirir primero el lock intrínseco de la instancia (`this`) [P14, p. 14; T19, p. 7].
* Si se declara un método estático con `synchronized`, el lock se toma sobre el objeto `Class` correspondiente (`NombreClase.class`).
* Los métodos de sincronización condicional heredados de `java.lang.Object` son:
  * **`wait()`**: Equivalente a `await()`. Libera el lock intrínseco y suspende el hilo [P14, p. 13].
  * **`notify()`**: Equivalente a `signal()`. Despierta a un hilo arbitrario [P14, p. 13].
  * **`notifyAll()`**: Equivalente a `broadcast()` / `signalAll()`. Despierta a todos los hilos en espera [P14, p. 13].

| Criterio | `synchronized` (Monitor Implícito) | `ReentrantLock` (Lock Explícito) |
| :--- | :--- | :--- |
| **Sintaxis** | Palabra clave nativa, código más limpio. | Clase en `java.util.concurrent`, requiere `try-finally`. |
| **Flexibilidad** | Bloque estructurado; se adquiere y libera en el mismo método. | Se puede adquirir en un método y liberar en otro. |
| **Condiciones** | Solo una cola de espera implícita por objeto. | Admite múltiples condiciones independientes (`newCondition()`). |
| **Adquisición Avanzada** | No permite timeout ni cancelación. | Métodos `tryLock()` y `lockInterruptibly()`. |

---

## 4. Problemas Clásicos de Concurrencia y Soluciones en Java

Para integrar las herramientas de sincronización, analizaremos los problemas clásicos de la literatura de sistemas concurrentes.

---

### Problema 1: Productor - Consumidor (Bounded Buffer)
Varios procesos de tipo **Productor** generan información y la colocan en un buffer, mientras que procesos de tipo **Consumidor** extraen información de él [T19, p. 15-16].

#### Restricciones del Sistema [T19, p. 15-16]:
1. El consumidor no puede extraer elementos si el buffer se encuentra **vacío** [T19, p. 15].
2. El productor no puede insertar elementos si el buffer alcanzó su **capacidad máxima $N$** (en buffer acotado) [T19, p. 16].
3. Los accesos a la estructura interna deben ser estrictamente atómicos y respetar el orden FIFO [T19, p. 15].

#### Implementación Elegante con Doble Semáforo en Java:
Una de las soluciones más potentes evita bucles `while` y variables de condición utilizando **dos semáforos cruzados y un lock de exclusión mutua**:
* `semEspacios`: Inicializado en la capacidad máxima $N$ (representa los casilleros libres para producir).
* `semItems`: Inicializado en `0` (representa los elementos disponibles para consumir).
* `mutex`: Inicializado en `1` (protege la estructura de la cola).

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.Semaphore;

public class BoundedBufferConcurrente<T> {
    private final Queue<T> cola = new LinkedList<>();
    private final Semaphore semEspacios;
    private final Semaphore semItems;
    private final Semaphore mutex;

    public BoundedBufferConcurrente(int capacidad) {
        this.semEspacios = new Semaphore(capacidad); // Arranca con N lugares
        this.semItems = new Semaphore(0);           // Arranca con 0 ítems
        this.mutex = new Semaphore(1);              // Mutex para exclusión mutua
    }

    public void producir(T item) throws InterruptedException {
        semEspacios.acquire(); // Decrementa espacio disponible (bloquea si está lleno)
        mutex.acquire();       // Protege la sección crítica de la cola
        try {
            cola.add(item);
            System.out.println("Producido: " + item + " | Total en buffer: " + cola.size());
        } finally {
            mutex.release();
        }
        semItems.release();    // Notifica que hay un nuevo ítem disponible
    }

    public T consumir() throws InterruptedException {
        semItems.acquire();    // Decrementa ítem disponible (bloquea si está vacío)
        mutex.acquire();       // Protege la sección crítica de la cola
        T elemento;
        try {
            elemento = cola.remove();
            System.out.println("Consumido: " + elemento + " | Total en buffer: " + cola.size());
        } finally {
            mutex.release();
        }
        semEspacios.release(); // Libera un casillero para futuros productores
        return elemento;
    }
}
```

> **Aclaración Técnica sobre Semáforos Inicializados en 0**:
> Inicializar `semItems` en 0 parece antiintuitivo si se piensa al semáforo como un pool de conexiones donde el valor inicial es el máximo permitido. Sin embargo, un semáforo representa un contador de eventos. Al iniciarse en 0, cualquier consumidor que intente hacer `acquire()` se bloquea de inmediato hasta que un productor invoque `release()`, actuando como una barrera de sincronización limpia.

---

### Problema 2: Lectores - Escritores (Acceso a Base de Datos)
Múltiples hilos compiten por acceder a un área compartida de datos (como una base de datos o archivo en memoria) [T19, p. 17].

#### Reglas de Invarianza [T19, p. 17]:
1. **Múltiples lectores** pueden acceder y leer datos simultáneamente sin peligro de condición de carrera [T18, p. 18; T19, p. 17].
2. No se puede leer mientras un escritor está modificando datos [T19, p. 17].
3. **Solo un escritor** puede modificar la base de datos a la vez, excluyendo tanto a otros escritores como a todos los lectores [T19, p. 17].

#### Implementación con Monitor en Java:
```java
public class GestorLectoresEscritores {
    private int lectoresActivos = 0;
    private boolean escribiendo = false;

    // Protocolo de inicio de lectura
    public synchronized void empezarLectura() throws InterruptedException {
        while (escribiendo) {
            wait(); // Espera si hay un escritor activo
        }
        lectoresActivos++;
    }

    // Protocolo de finalización de lectura
    public synchronized void terminarLectura() {
        lectoresActivos--;
        // Si soy el último lector en salir, notifico a posibles escritores en espera
        if (lectoresActivos == 0) {
            notifyAll();
        }
    }

    // Protocolo de inicio de escritura
    public synchronized void empezarEscritura() throws InterruptedException {
        // Un escritor exige exclusión total: ni escritores ni lectores activos
        while (escribiendo || lectoresActivos > 0) {
            wait();
        }
        escribiendo = true;
    }

    // Protocolo de finalización de escritura
    public synchronized void terminarEscritura() {
        escribiendo = false;
        notifyAll(); // Despierta tanto a lectores como a escritores que esperan
    }
}
```

*Análisis de Starvation en Lectores-Escritores*: En esta implementación con prioridad para lectores, si un flujo incesante de nuevos lectores ingresa antes de que `lectoresActivos` llegue a 0, un hilo escritor sufrirá **inanición (*starvation*)**, quedando bloqueado indefinidamente.

---

### Problema 3: La Cena de los Filósofos
Formulado por Dijkstra, $N$ filósofos se sientan alrededor de una mesa circular [T19, p. 18]. Entre cada par de filósofos hay un único tenedor (en total $N$ filósofos y $N$ tenedores). Cada filósofo solo realiza dos actividades: **pensar** y **comer** [T19, p. 18].

```
                  [Filósofo 0]
                 /                      (Tenedor 0)      (Tenedor 1)
             /                         [Filósofo 4]              [Filósofo 1]
          |                          |
     (Tenedor 4)                (Tenedor 2)
             \                    /
          [Filósofo 3] ---- (Tenedor 3) ---- [Filósofo 2]
```

#### Reglas de Invarianza [T19, p. 18]:
* Para comer, un filósofo necesita **obligatoriamente los dos tenedores adyacentes** (el izquierdo y el derecho) [T19, p. 18].
* Mientras piensa, no retiene tenedores [T19, p. 18].

#### El Deadlock Ingenuo:
Si todos los filósofos deciden comer al mismo tiempo y aplican el algoritmo simétrico:
```
1. Tomar tenedor izquierdo
2. Tomar tenedor derecho
3. Comer
4. Soltar ambos tenedores
```
Si los 5 filósofos toman en simultáneo su tenedor izquierdo, todos se quedarán reteniendo el tenedor izquierdo mientras esperan que el vecino suelte el derecho. El sistema entra en un **deadlock circular clásico** [T19, p. 18].

#### El Livelock por Cortesía:
Si intentamos solucionar el deadlock diciendo: *"Si tomás el izquierdo y el derecho no está disponible, soltá el izquierdo, esperá un instante y volvé a intentar"*. En una CPU paralela, todos pueden tomar el izquierdo simultáneamente, ver que el derecho no está, soltarlo al mismo tiempo, y repetir la secuencia una y otra vez. Se produce un **livelock**: alto uso de recursos sin que nadie logre comer jamás [T19, p. 18].

#### Tres Soluciones de Diseño:
1. **Árbitro Central (Lock de Mesa)**: Se introduce un lock global para tomar cubiertos. Solo un filósofo puede intentar levantar tenedores a la vez. Es seguro y libre de deadlocks, pero limita la concurrencia: como máximo comerá un solo filósofo a la vez aunque los comensales no adyacentes podrían comer en paralelo.
2. **Limitación de Comensales (Semáforo de Asientos)**: Se coloca un semáforo inicializado en $N - 1$ (para 5 filósofos, 4 permisos). Al impedir que los 5 coman al mismo tiempo, al menos un filósofo tendrá garantizados ambos tenedores, rompiendo la condición de retención y espera de Coffman.
3. **Ruptura de la Simetría (Filósofo Zurdo)**: Se asigna un orden asimétrico a uno de los filósofos. Mientras los filósofos del $0$ al $3$ toman primero el tenedor izquierdo y luego el derecho, el filósofo $4$ intenta tomar **primero el derecho y luego el izquierdo**. Esta inversión rompe matemáticamente la cadena de espera circular de Coffman [T19, p. 18].

---

## 5. Evolución del Modelo Concurrente: De Threads del SO a Asincronismo y Virtual Threads

A medida que los sistemas crecieron para soportar servicios distribuidos e Internet a escala masiva, las limitaciones del modelo clásico de concurrencia en Java forzaron la evolución de la arquitectura.

### Los Desafíos de los Hilos Tradicionales de Plataforma (*Platform Threads*)
En las versiones clásicas de Java, cada instancia de `Thread` equivale a un **Platform Thread**, el cual se mapea de manera directa ($1:1$) a un hilo del núcleo del sistema operativo (*OS Kernel Thread*) [T18, p. 23]:

```
  JVM:         [ Thread 1 ]     [ Thread 2 ]     [ Thread 3 ]
                    |                |                |
             ( Mapeo 1:1 con llamadas al sistema nativas )
                    |                |                |
  OS Kernel:   [ Thread 1 ]     [ Thread 2 ]     [ Thread 3 ]
```

#### Cuatro Desventajas Críticas de los Platform Threads [T18, p. 24]:
1. **Creación Muy Costosa**: Instanciar un hilo exige llamadas al sistema operativo (*system calls*), inicialización de estructuras del kernel y asignación de memoria nativa [T18, p. 24].
2. **Consumo Excesivo de Memoria**: Cada thread reserva un tamaño fijo de memoria para su pila (*stack*, típicamente 1 MB por defecto). Un millón de hilos requeriría un terabyte solo en memoria de pilas [T18, p. 24].
3. **Sobrecarga de Cambio de Contexto**: Cuando la cantidad de hilos supera con creces el número de núcleos físicos de la CPU, el sistema operativo colapsa intentando balancear y migrar hilos entre cores [T18, p. 24].
4. **Desperdicio en I/O Bloqueante**: Cuando un hilo realiza una operación de lectura o espera una llamada de red, permanece bloqueado consumiendo 1 MB de memoria y recursos del kernel sin producir trabajo [T18, p. 24].

---

### La Alternativa Single-Thread: El Event Loop de Node.js
Ante el costo de los múltiples hilos, plataformas como **Node.js** adoptaron un modelo radicalmente opuesto: **un único hilo principal de ejecución asistido por un bucle de eventos (*Event Loop*)** [T18, p. 31; Clase Youtube 1].

```
                       ARQUITECTURA DEL EVENT LOOP
                                   |
                   +---> [ 1. Timers (setTimeout) ]
                   |                 |
                   |     [ 2. Pending Callbacks (I/O) ]
                   |                 |
                   |     [ 3. Idle, Prepare ]
                   |                 |
                   |     [ 4. POLL (I/O Events) ] <--- Nuevas Conexiones / Datos
                   |                 |
                   |     [ 5. Check (setImmediate) ]
                   |                 |
                   +---- [ 6. Close Callbacks ]
```

#### Funcionamiento Operativo [T18, p. 31; Clase Youtube 1]:
* **Operaciones No Bloqueantes**: Asignaciones de variables, llamadas a funciones y transformaciones locales en memoria se ejecutan de inmediato en el hilo principal [T18, p. 31].
* **Operaciones Bloqueantes**: Lecturas de archivos en disco, consultas a bases de datos o peticiones HTTP se delegan inmediatamente al sistema operativo o a un pool interno de hilos auxiliares en segundo plano (*libuv*) [T18, p. 31].
* El hilo principal **nunca se bloquea**: despacha la tarea de I/O y queda libre inmediatamente para atender peticiones de otros usuarios [T18, p. 31].
* Cuando la operación bloqueante finaliza, su callback se encola en la fase de **Poll** del Event Loop, donde el hilo principal retoma el resultado y continúa la lógica [T18, p. 31].

> **Aclaración sobre el "Single-Thread"**:
> Afirmar que Node.js es puramente mono-hilo es un mito. El código JavaScript del usuario corre en un solo hilo (el Event Loop), pero por debajo el motor utiliza un pool de hilos nativos para resolver las operaciones pesadas de disco y red, consumiendo típicamente entre un 10% y un 30% de capacidad adicional de CPU (*1.1 a 1.3 cores*) [Clase Youtube 1]. 
> El peligro supremo en este modelo es ejecutar código matemático pesado (CPU-bound) en el hilo principal: si el Event Loop se traba calculando una matriz, ninguna otra conexión entrante podrá ser atendida, congelando el servidor completo [T18, p. 31; Clase Youtube 1].

---

### Programación Asincrónica con `CompletableFuture` en Java
Para evitar el bloqueo de hilos sin abandonar la JVM, Java 8 introdujo `CompletableFuture`, equivalente a las *Promises* de JavaScript [T18, p. 25].

#### Características Principales [T18, p. 25]:
* Representa el resultado pendiente de una operación asincrónica que puede completarse con un valor o fallar con una excepción [T18, p. 25].
* Utiliza internamente un pool de hilos reutilizables (`ForkJoinPool.commonPool()`), desacoplando al programador de la creación manual de threads [T18, p. 25].
* Permite encadenar transformaciones de forma no bloqueante mediante funciones de orden superior [T18, p. 25]:
  * `supplyAsync()`: Dispara una tarea asincrónica que produce un valor [T18, p. 26].
  * `thenApply()`: Transforma el resultado cuando esté listo (`map`) [T18, p. 32].
  * `thenCompose()`: Encadena otra operación asincrónica dependiente (`flatMap`) [T18, p. 32].
  * `thenAccept()`: Consume el resultado final (`forEach`) [T18, p. 26].
  * `CompletableFuture.allOf()`: Espera la finalización concurrente de múltiples futuros [T18, p. 28].

```java
// Ejemplo: Agregador de servicios de red en paralelo sin bloqueo
CompletableFuture<Usuario> futUsuario = CompletableFuture.supplyAsync(() -> api.obtenerUsuario(id));
CompletableFuture<List<Pedido>> futPedidos = CompletableFuture.supplyAsync(() -> api.obtenerPedidos(id));

CompletableFuture<DashboardDTO> dashboard = futUsuario.thenCombine(futPedidos, (u, p) -> {
    return new DashboardDTO(u, p);
});

// Se procesa de forma no bloqueante cuando ambos responden
dashboard.thenAccept(dto -> System.out.println("Dashboard cargado: " + dto));
```

#### Limitaciones de `CompletableFuture` [T18, p. 33]:
1. **Callback Hell**: El encadenamiento excesivo de transformaciones asincrónicas degrada severamente la legibilidad del código [T18, p. 32-33].
2. **Dificultad de Depuración**: Las trazas de excepción (*stack traces*) pierden la causalidad lineal de los métodos invocados al saltar entre distintos hilos del pool [T18, p. 33].
3. **Incompatibilidad con Estructuras Secuenciales**: Resulta sumamente complejo traducir estructuras de control tradicionales (bucles `for`, bloques `try-catch`) al paradigma declarativo de futuros [T18, p. 33].

---

### Virtual Threads (Project Loom, Java 19+)
Para resolver simultáneamente las desventajas de los Platform Threads y la complejidad de los futuros asincrónicos, Java 19 introdujo los **Virtual Threads (Hilos Virtuales)** [T18, p. 34-37].

Un Virtual Thread es un hilo ultraliviano gestionado **directamente por la Máquina Virtual de Java (JVM)** en lugar de por el kernel del sistema operativo [T18, p. 35].

```
  JVM:           [VT 1]  [VT 2]  [VT 3]     [VT 4]  [VT 5]  [VT 6]   (Miles o millones de VTs)
                    \       |       /          \       |       /
                     \      |      /            \      |      /
  Platform Threads:    [ Carrier 1 ]               [ Carrier 2 ]     (Pocos hilos nativos)
                            |                           |
  OS Kernel:           [ Thread OS ]               [ Thread OS ]
```

#### Mecanismo Operativo $M:N$ [T18, p. 34-36]:
* La JVM monta millones de Virtual Threads sobre un pequeño grupo de Platform Threads subyacentes, denominados **Carrier Threads** [T18, p. 34, 36].
* **Suspensión Automática en I/O Bloqueante**: Cuando un Virtual Thread ejecuta una llamada bloqueante (acceso a base de datos, lectura de red, `Thread.sleep`), la JVM intercepta la llamada, **desmonta el Virtual Thread del Carrier Thread y libera el Carrier Thread del sistema operativo** para que ejecute inmediatamente otro Virtual Thread [T18, p. 35-36].
* Cuando la operación de I/O finaliza, la JVM vuelve a montar el Virtual Thread en cualquier Carrier Thread disponible y continúa su ejecución exactamente desde donde se suspendió [T18, p. 36].

#### Ventajas Radicales de los Virtual Threads [T18, p. 35]:
1. **Costo de Creación Despreciable**: Se crean en el *heap* de Java como cualquier objeto normal; no realizan llamadas de sistema nativas ni asignan 1 MB de memoria de stack [T18, p. 35].
2. **Sin Sobrecarga de Context Switch del SO**: La conmutación ocurre enteramente en el espacio de usuario gestionado por la JVM [T18, p. 35].
3. **Retorno al Código Imperativo Sincrónico**: Permite escribir código lineal y fácil de leer (`Thread.sleep()`, llamadas bloqueantes simples) con la escalabilidad y eficiencia de los sistemas reactivos y asincrónicos [T18, p. 35].

#### Ejemplo de Código en Java Moderno [T18, p. 37]:
```java
public class VirtualThreadsDemo {
    public static void main(String[] args) throws InterruptedException {
        // Creación e inicio directo de un Virtual Thread
        Thread vt = Thread.ofVirtual().start(() -> {
            System.out.println("Hola desde un Virtual Thread: " + Thread.currentThread());
        });
        
        vt.join();
    }
}
```

---

## 6. Cuadro Comparativo Global de Paradigmas y Mecanismos de Concurrencia

| Mecanismo / Enfoque | Nivel de Abstracción | Mecanismo Central | Principal Ventaja | Principal Desventaja | Caso de Uso Óptimo |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Platform Threads (`Thread`)** | Bajo nivel | Mapeo 1:1 con hilos nativos del SO | Control total del hardware y afinidad de CPU | Muy costosos en memoria (1MB) y context switch | Cómputo matemático intensivo en paralelo (CPU-bound) |
| **Locks Explícitos (`ReentrantLock`)** | Medio | Adquisición / Liberación de candado en exclusión mutua | Alta flexibilidad, soporte de timeouts (`tryLock`) y múltiples condiciones | Verbosidad obligatoria; riesgo de fugas si se omite `finally` | Secciones críticas con múltiples condiciones de espera |
| **Monitores (`synchronized`)** | Alto | Lock intrínseco de cada objeto Java | Sintaxis limpia, integración directa con el lenguaje | Una sola cola de condición implícita por objeto | Protección simple de métodos y objetos de dominio |
| **Semáforos (`Semaphore`)** | Medio | Contador entero de permisos | Control cuantitativo de concurrencia | No protege variables únicas contra race conditions si $N > 1$ | Pools de conexiones a base de datos y limitadores de tasa |
| **Event Loop (Node.js)** | Arquitectónico | Hilo único despachador + I/O asincrónico delegado | Cero problemas de sincronización de memoria compartida | Una tarea CPU-bound bloquea a todo el servidor | Servidores I/O intensivos con conexiones masivas concurrentes |
| **`CompletableFuture`** | Funcional / Reactivo | Composición de callbacks asincrónicos sobre Thread Pool | Ejecución no bloqueante sin hilos ociosos | Código difícil de leer (*Callback Hell*) y stack traces rotos | Orquestación de microservicios y agregación de APIs |
| **Virtual Threads (Java 19+)** | Alto (Runtime) | Mapeo $M:N$ gestionado por la JVM sobre Carrier Threads | Millones de hilos posibles con código secuencial simple | No aporta ventajas significativas en tareas puramente CPU-bound | Aplicaciones empresariales masivas limitadas por I/O |

