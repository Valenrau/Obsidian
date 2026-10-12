# Planificación de Procesos en Sistemas Operativos: De la Multiprogramación Tradicional al Completely Fair Scheduler (CFS) de Linux

## 1. Fundamentos de la Planificación de CPU

### 1.1 Evolución de la Monoprogramación a la Multiprogramación y Tiempo Compartido
En los inicios de la computación, los sistemas operaban bajo esquemas de **monoprogramación** (ejecución por lotes puros o *single-job execution*). Un único programa tomaba el control absoluto de la Unidad Central de Procesamiento (CPU) y los periféricos de Entrada/Salida (I/O). Si el programa necesitaba leer datos de una cinta magnética o tarjeta perforada, la CPU permanecía completamente inactiva (*idle*) hasta que la transferencia física finalizaba. Dada la inmensa brecha de velocidad entre los transistores del procesador y los dispositivos mecánicos de I/O, este modelo provocaba un desperdicio masivo del recurso más costoso del sistema.

Para resolver esta ineficiencia surgió la **multiprogramación**. Su principio fundamental consiste en mantener múltiples procesos cargados simultáneamente en la memoria principal. Cuando el proceso que se encuentra ejecutando en la CPU debe esperar por una operación de I/O, el sistema operativo intercala la ejecución realizando un cambio de contexto para otorgarle la CPU a otro proceso que esté listo para ejecutar (*RUNNABLE*).

Posteriormente, con la llegada de las terminales de usuario interactivas, la multiprogramación evolucionó hacia el **tiempo compartido (*time-sharing*)**. En lugar de esperar a que un proceso se bloquee por I/O para ceder la CPU, el sistema operativo incorpora un **temporizador de hardware (*timer interrupt*)** que interrumpe periódicamente el proceso en ejecución. Esto permite conmutar rápidamente la CPU entre múltiples usuarios o programas, creando la **ilusión de ejecución simultánea (pseudoparalelismo)**.

* **Explicación mediante Metáfora:**
  Imagínese un restaurante con un único cocinero de alta velocidad (la CPU) y varios meseros (dispositivos de I/O). En la monoprogramación, el cocinero toma la orden de la Mesa 1, hornea el pan, espera a que el mesero lleve el plato y la Mesa 1 termine de comer para recién tomar la orden de la Mesa 2. En la multiprogramación, mientras la Mesa 1 está masticando o esperando la bebida (bloqueada en I/O), el cocinero prepara el plato de la Mesa 2. En el tiempo compartido, el cocinero le da un bocado o una atención de 10 segundos a cada mesa de forma rotativa tan rápido que todos los comensales sienten que tienen un cocinero personal.

* **Ejemplo Práctico/Técnico:**
  Considere la diferencia de tiempos entre operaciones de CPU y accesos a disco o red:
  * Registro / Caché L1 de CPU: ~0.5 a 1 nanosegundo.
  * Memoria RAM: ~100 nanosegundos.
  * Lectura de Disco SSD / Red Local: ~100.000 a 10.000.000 nanosegundos (100 µs a 10 ms).

  Si un proceso realiza un cálculo de 1 milisegundo y luego solicita un bloque de datos a un disco duro que tarda 10 milisegundos en responder:
  * **Sin multiprogramación:** En un período de 11 ms, la CPU trabaja 1 ms y descansa 10 ms. Utilización de CPU = $\frac{1}{11} \approx 9.09\%$.
  * **Con multiprogramación (10 procesos idénticos):** Mientras el proceso 1 espera al disco, el sistema operativo ejecuta los procesos 2, 3, ..., 10. La CPU puede mantenerse ejecutando instrucciones cerca del $100\%$ del tiempo disponible.

---

### 1.2 Justificación Matemática e Impacto de la Entrada/Salida (I/O)
La ganancia de eficiencia que aporta la multiprogramación no es meramente cualitativa, sino que puede modelarse matemáticamente en función de la fracción de tiempo que un proceso pasa esperando por operaciones de I/O.

Sea $p$ la probabilidad de que un proceso dado esté esperando por una operación de I/O en un instante arbitrario (es decir, la proporción de tiempo de I/O respecto a su tiempo total de vida). Si en la memoria principal hay $n$ procesos independientes cargados simultáneamente, la probabilidad de que **todos** los $n$ procesos estén bloqueados esperando I/O al mismo tiempo es $p^n$ (asumiendo independencia estadística).

Por lo tanto, la **utilización de la CPU** ($U_{CPU}$) se expresa como:
$$U_{CPU} = 1 - p^n$$

```
Utilización de CPU (1 - p^n)
1.0 |                                      * * * * * (p = 0.5)
0.8 |                            * * * * *
0.6 |                  * * * * *                   (p = 0.8)
0.4 |        * * * * *
0.2 |  * * *
0.0 +---------------------------------------------------------
      1    2    3    4    5    6    7    8    9    10   (n procesos)
```

* **Explicación mediante Metáfora:**
  Imagine un taller mecánico donde la probabilidad de que un auto requiera un repuesto importado (I/O) y quede parado es del $80\%$ ($p = 0.8$). Si el mecánico solo acepta 1 auto en el taller ($n=1$), el $80\%$ del tiempo estará sin trabajar. Si acepta 5 autos ($n=5$), la probabilidad de que los 5 estén parados al mismo tiempo por repuestos es de $0.8^5 = 0.32768$, por lo que el mecánico estará trabajando el $67.2\%$ del tiempo. Si acepta 10 autos ($n=10$), la probabilidad de inactividad cae a $0.8^{10} \approx 0.107$, logrando una utilización del taller del $89.3\%$.

* **Ejemplo Práctico/Técnico:**
  En la infraestructura de servidores en la nube (*Cloud Computing*), las instancias de procesamiento se pagan por hora de uso de CPU. Suponga un servidor web interactivo donde cada proceso pasa el $80\%$ de su tiempo esperando peticiones de red o consultas a base de datos ($p = 0.8$).
  * Si el servidor se configura para ejecutar un solo proceso ($n=1$), la utilización es $1 - 0.8^1 = 20\%$. Se está desperdiciando el $80\%$ de la capacidad pagada.
  * Si se configuran 10 procesos/hilos concurrentes ($n=10$), la utilización sube a $1 - 0.8^{10} = 1 - 0.10737 = 89.26\%$.
  * Si se eleva a 20 procesos concurrentes ($n=20$), $U_{CPU} = 1 - 0.8^{20} \approx 98.85\%$.
  Esta formulación explica por qué los servidores web modernos (como Nginx, Apache o Node.js) manejan cientos o miles de conexiones concurrentes para maximizar el rendimiento económico del hardware.

---

### 1.3 Clasificación de Cargas de Trabajo y Sistemas
El diseño de un planificador (*scheduler*) depende críticamente de la naturaleza de los procesos y del objetivo del sistema operativo.

#### Clasificación según el Comportamiento del Proceso:
1. **Procesos Orientados a CPU (*CPU-bound*):** Pasan la mayor parte de su tiempo realizando cálculos numéricos intensivos. Hacen pocas llamadas al sistema y rara vez se bloquean por I/O. Ejemplos: compilación de código, renderizado 3D, entrenamiento de modelos de Inteligencia Artificial, simulación científica.
2. **Procesos Orientados a I/O (*I/O-bound*):** Pasan la mayor parte de su tiempo esperando eventos externos (interacción del usuario, disco, red). Ejecutan ráfagas (*bursts*) muy cortas de CPU y de inmediato realizan una llamada al sistema bloqueante. Ejemplos: editores de texto, reproductores de audio/video, servidores de archivos.

#### Clasificación según el Tipo de Sistema:
1. **Sistemas Por Lotes (*Batch Systems*):** No hay usuarios interactivos esperando respuestas inmediatas en una pantalla. El objetivo es procesar un gran volumen de trabajos de principio a fin optimizando el tiempo total.
2. **Sistemas Interactivos (*Interactive Systems*):** Hay usuarios humanos operando la máquina o clientes esperando respuestas por red. El objetivo prioritario es responder rápidamente a los estímulos.

#### Mecanismo de Desalojo:
* **No Preemptivo (*Non-preemptive* / Sin desalojo):** Un proceso toma la CPU y la retiene hasta que decide cederla voluntariamente (mediante una llamada al sistema de I/O o finalizando). Si el programa entra en un bucle infinito, cuelga todo el sistema.
* **Preemptivo (*Preemptive* / Con desalojo):** El sistema operativo, apoyado por interrupciones de reloj del hardware, puede remover por la fuerza a un proceso de la CPU en cualquier momento para otorgársela a otro.

---

### 1.4 Métricas Fundamentales de Planificación
Para evaluar y comparar las políticas de planificación, se utilizan métricas cuantitativas estandarizadas.

1. **Tiempo de Retorno / Finalización (*Turnaround Time - $T_{turnaround}$*):** Es el tiempo total transcurrido desde que un proceso ingresa al sistema (arriba) hasta que completa su ejecución. Mide la **performance o rendimiento global**.
   $$T_{turnaround} = T_{completion} - T_{arrival}$$

2. **Tiempo de Respuesta (*Response Time - $T_{response}$*):** Es el tiempo transcurrido desde que un proceso llega o se vuelve ejecutable hasta que es seleccionado por primera vez por el planificador y comienza a ejecutar en la CPU. Mide la **responsividad o interactividad**.
   $$T_{response} = T_{firstrun} - T_{arrival}$$

3. **Equidad (*Fairness*):** Mide la distribución justa de la CPU entre todos los procesos competidores. Existe una tensión o *trade-off* fundamental en teoría de planificación: **optimizar el tiempo de retorno suele arruinar el tiempo de respuesta y la equidad, y viceversa**.

* **Ejemplo Práctico/Técnico:**
  Suponga que un proceso $P_1$ llega al sistema en el instante $t = 0$ ms, es ejecutado por primera vez en $t = 5$ ms y finaliza completamente en $t = 30$ ms.
  * $T_{turnaround} = 30	ext{ ms} - 0	ext{ ms} = 30	ext{ ms}$.
  * $T_{response} = 5	ext{ ms} - 0	ext{ ms} = 5	ext{ ms}$.

---

## 2. Algoritmos de Planificación Por Lotes (Batch)

Para analizar los algoritmos clásicos, se establecen inicialmente **cinco supuestos didácticos simplificados**:
1. Todos los trabajos (*jobs*) duran la misma cantidad de tiempo.
2. Todos los trabajos llegan al mismo tiempo ($T_{arrival} = 0$).
3. Una vez que un trabajo comienza, ejecuta hasta completarse (no preemptivo).
4. Todos los trabajos usan exclusivamente la CPU (sin I/O).
5. El tiempo de ejecución (*runtime*) de cada trabajo es conocido a priori por el planificador.

---

### 2.1 First In, First Out (FIFO) / First Come, First Served (FCFS)
Es la política más elemental: atiende los procesos estrictamente en el orden de su llegada, utilizando una cola First-In, First-Out.

#### Propósito y Funcionamiento:
Mantiene los procesos listos en una lista enlazada o arreglo circular. Toma el primero de la cola, lo ejecuta hasta su finalización y luego toma el siguiente.

#### El Efecto Convoy (*Convoy Effect*):
Si se relaja el supuesto 1 y se permite que los trabajos tengan diferentes duraciones, FIFO sufre un problema grave de rendimiento cuando un proceso intensivo de larga duración llega justo antes de varios procesos extremadamente cortos.

* **Explicación mediante Metáfora:**
  Es la fila de un supermercado donde una persona con dos carros llenos de mercadería (proceso $A$ de 100 segundos) está primera en la caja, y detrás hay dos personas que solo quieren pagar un paquete de chicles cada una (procesos $B$ y $C$ de 10 segundos). Aunque los clientes $B$ y $C$ podrían pagar en un segundo, deben esperar a que pase toda la carga del primer cliente.

* **Ejemplo Práctico/Técnico (Cálculo de Métricas):**
  Tres procesos llegan en $T_{arrival} = 0$: $A$ (dura 100s), $B$ (dura 10s), $C$ (dura 10s).
  Orden de ejecución en FIFO: $A ightarrow B ightarrow C$.

  ```
  [0 -------------- A -------------- 100][100 - B - 110][110 - C - 120]
  ```

  * Tiempos de finalización: $T_{comp}(A) = 100$, $T_{comp}(B) = 110$, $T_{comp}(C) = 120$.
  * Turnaround individual: $T_{turn}(A) = 100$, $T_{turn}(B) = 110$, $T_{turn}(C) = 120$.
  * **Turnaround Promedio:**
    $$ar{T}_{turnaround} = rac{100 + 110 + 120}{3} = rac{330}{3} = 110	ext{ segundos}$$

---

### 2.2 Shortest Job First (SJF)
Para solucionar el efecto convoy de FIFO, surge la política **SJF**: ejecuta siempre primero el trabajo con la duración total de CPU más corta.

#### Propósito y Funcionamiento:
Ordena la lista de procesos según su tiempo de ejecución de menor a mayor. Es una política no preemptiva.

* **Explicación mediante Metáfora:**
  El cajero del supermercado analiza las canastas de todos los clientes en la fila que llegaron al abrir la caja y hace pasar primero a los dos clientes que llevan un solo artículo antes de atender al cliente del carro lleno.

* **Ejemplo Práctico/Técnico (Comparación con FIFO):**
  Usando los mismos procesos: $A$ (100s), $B$ (10s), $C$ (10s), todos arribando en $t = 0$.
  Orden de ejecución en SJF: $B ightarrow C ightarrow A$.

  ```
  [0 - B - 10][10 - C - 20][20 -------------- A -------------- 120]
  ```

  * Tiempos de finalización: $T_{comp}(B) = 10$, $T_{comp}(C) = 20$, $T_{comp}(A) = 120$.
  * **Turnaround Promedio:**
    $$ar{T}_{turnaround} = rac{10 + 20 + 120}{3} = rac{150}{3} = 50	ext{ segundos}$$
  Se logra una reducción dramática del tiempo de retorno promedio (de 110s a 50s).

#### La Limitación de SJF (Llegadas Diferidas):
Si relajamos el supuesto 2 ($T_{arrival}$ heterogéneo), suponga que $A$ (100s) llega en $t=0$, mientras que $B$ (10s) y $C$ (10s) llegan en $t=10$.
Como SJF no es preemptivo, cuando $B$ y $C$ llegan a los 10 segundos, $A$ ya tomó la CPU y no puede ser desalojado.
* $T_{comp}(A) = 100$, $T_{comp}(B) = 110$, $T_{comp}(C) = 120$.
* Turnaround individuales ($T_{comp} - T_{arr}$):
  * $A: 100 - 0 = 100$
  * $B: 110 - 10 = 100$
  * $C: 120 - 10 = 110$
* $ar{T}_{turnaround} = rac{100 + 100 + 110}{3} = 103.33	ext{ segundos}$. El beneficio de SJF se destruye.

---

### 2.3 Shortest Time-to-Completion First (STCF)
Para resolver la rigidez de SJF ante llegadas diferidas, se relaja el supuesto 3 incorporando **desalojo (preemption)**. Esta política se conoce como **STCF** o *Preemptive SJF*.

#### Propósito y Funcionamiento:
Cada vez que un nuevo proceso ingresa al sistema (o un proceso se desbloquea), el planificador compara el **tiempo restante de ejecución** del proceso actual en CPU contra el tiempo total/restante del nuevo proceso. Si el nuevo trabajo requiere menos tiempo para terminar que lo que le resta al trabajo actual, el planificador desaloja al proceso en ejecución y le otorga la CPU al nuevo.

* **Explicación mediante Metáfora:**
  El cajero está pasando los artículos de un carro gigante. A los 10 segundos, llega un cliente con una botella de agua. El cajero detiene momentáneamente la cuenta del carro grande, le cobra la botella de agua al nuevo cliente en 2 segundos, y luego reanuda el pase del carro grande.

* **Ejemplo Práctico/Técnico:**
  * $A$: llega en $t=0$, dura 100s.
  * $B$ y $C$: llegan en $t=10$, duran 10s cada uno.

  Cronograma de Ejecución en STCF:
  1. $t=0$ a $t=10$: Ejecuta $A$. En $t=10$, a $A$ le restan 90s.
  2. $t=10$: Llegan $B$ (restan 10s) y $C$ (restan 10s). Como $10	ext{s} < 90	ext{s}$, se desaloja a $A$.
  3. $t=10$ a $t=20$: Ejecuta $B$. En $t=20$, $B$ finaliza.
  4. $t=20$ a $t=30$: Ejecuta $C$. En $t=30$, $C$ finaliza.
  5. $t=30$ a $t=120$: Se reanuda $A$ hasta completar los 90s restantes.

  ```
  [0-A-10][10-B-20][20-C-30][30 ------------------ A ------------------ 120]
  ```

  Cálculo de $T_{turnaround}$:
  * $A: T_{comp} = 120 ightarrow T_{turn} = 120 - 0 = 120$
  * $B: T_{comp} = 20 ightarrow T_{turn} = 20 - 10 = 10$
  * $C: T_{comp} = 30 ightarrow T_{turn} = 30 - 10 = 20$
  * **Turnaround Promedio:**
    $$\bar{T}_{turnaround} = \frac{120 + 10 + 20}{3} = \frac{150}{3} = 50\text{ segundos}$$

---

## 3. Planificación de Sistemas Interactivos y Tiempo Compartido

Aunque STCF es óptimo para minimizar el tiempo de retorno promedio, es **pésimo para el tiempo de respuesta y la interactividad**. Si en STCF evaluamos el tiempo de respuesta del ejemplo anterior:
* $T_{response}(A) = 0 - 0 = 0$
* $T_{response}(B) = 10 - 10 = 0$
* $T_{response}(C) = 20 - 10 = 10$

Si un usuario estuviera escribiendo en una terminal asociada al proceso $C$, experimentaría un congelamiento de 10 segundos antes de ver la primera respuesta.

---

### 3.1 Round Robin (RR) y el Manejo de Time Slices
Para satisfacer a los sistemas interactivos de tiempo compartido nació **Round Robin (RR)**.

#### Propósito y Funcionamiento:
En lugar de ejecutar los procesos hasta su finalización, Round Robin ejecuta un proceso durante una pequeña porción fija de tiempo llamada **intervalo de tiempo (*time slice*)** o **quantum**. Una vez agotado el quantum, el proceso es desalojado mediante una interrupción de reloj y movido al final de una cola circular de procesos listos.

#### El Dilema del Tamaño del Quantum:
* **Quantum muy pequeño (ej. 1 ms):** Mejora enormemente el tiempo de respuesta. Sin embargo, el **costo de cambio de contexto (*context switch overhead*)** empieza a dominar la CPU. Cada cambio de contexto requiere guardar y restaurar registros, cambiar tablas de páginas y vaciar la caché de la CPU.
* **Quantum muy grande (ej. 1000 ms):** Amortiza el costo del cambio de contexto a casi $0\%$, pero degrada el algoritmo transformándolo prácticamente en FIFO, arruinando la responsividad.

$$\text{Porcentaje de Overhead} = \frac{\text{Tiempo de Cambio de Contexto}}{\text{Quantum} + \text{Tiempo de Cambio de Contexto}} \times 100$$

* **Explicación mediante Metáfora:**
  Un profesor dando consulta a 5 alumnos en lugar de atender a cada uno por 1 hora completa (FIFO/SJF), le dedica 5 minutos a cada uno en ronda. Si el tiempo de consulta es de 5 minutos y el profesor tarda 1 minuto en guardar los papeles de un alumno y abrir la libreta del siguiente, pasa el $16.6\%$ del día administrando carpetas. Si atiende 10 segundos a cada uno, pasará más tiempo cambiando de carpeta que respondiendo preguntas.

* **Ejemplo Práctico/Técnico (Cálculo comparativo de métricas):**
  Suponga tres procesos $A$, $B$ y $C$, cada uno con un tiempo de ejecución de 5 segundos, todos llegando en $t=0$.
  * **Bajo SJF / FIFO:**
    $A$ ejecuta de 0 a 5s, $B$ de 5 a 10s, $C$ de 10 a 15s.
    * $T_{response}(A) = 0$, $T_{response}(B) = 5$, $T_{response}(C) = 10$. $\bar{T}_{response} = 5\text{s}$.
    * $T_{turn}(A) = 5$, $T_{turn}(B) = 10$, $T_{turn}(C) = 15$. $\bar{T}_{turnaround} = 10\text{s}$.
  * **Bajo Round Robin (quantum = 1s):**
    Secuencia: $A(1s) \rightarrow B(1s) \rightarrow C(1s) \rightarrow A(1s) \rightarrow B(1s) \rightarrow \dots$
    * $T_{response}(A) = 0$, $T_{response}(B) = 1$, $T_{response}(C) = 2$.
    * **$\bar{T}_{response} = 1\text{ segundo}$** (Excelente responsividad).
    * Tiempos de finalización: $C$ termina en $t=13$, $B$ en $t=14$, $A$ en $t=15$.
    * **$\bar{T}_{turnaround} = \frac{15 + 14 + 13}{3} = 14\text{ segundos}$** (Degradación del turnaround respecto a los 10s de SJF).

  > **Conclusión Clave:** Round Robin es óptimo para tiempo de respuesta, pero es una de las peores políticas para tiempo de retorno cuando los procesos tienen duraciones similares.

---

### 3.2 Implementación Práctica de Round Robin (Caso de Estudio: xv6)
En el sistema operativo de enseñanza **xv6** (basado en Unix para arquitecturas x86 / RISC-V), el planificador implementa un esquema Round Robin básico recorriendo la tabla global de procesos `proc`.

```c
// Bucle principal del planificador en kernel/proc.c para xv6
void scheduler(void) {
  struct proc *p;
  struct cpu *c = mycpu();
  c->proc = 0;
  
  for(;;){
    // Habilita interrupciones para evitar bloqueos del sistema
    intr_on();

    int found = 0;
    for(p = proc; p < &proc[NPROC]; p++) {
      acquire(&p->lock);
      if(p->state == RUNNABLE) {
        // Cambia el estado del proceso a RUNNING
        p->state = RUNNING;
        c->proc = p;

        // Realiza el cambio de contexto del kernel hacia el proceso p
        swtch(&c->context, &p->context);

        // El proceso ha sido desalojado o se bloqueó y retornó al kernel
        c->proc = 0;
        found = 1;
      }
      release(&p->lock);
    }
    if(found == 0) {
      // Si no hay ningún proceso RUNNABLE, la CPU entra en bajo consumo
      asm volatile("wfi");
    }
  }
}
```

---

## 4. Multi-Level Feedback Queue (MLFQ): Planificación Basada en Historia

### 4.1 Principios y Filosofía de MLFQ
En la vida real, el sistema operativo **no conoce a priori cuánto va a durar un trabajo ni si será orientado a CPU o a I/O**. 

Desarrollado originalmente por Fernando Corbató en el sistema CTSS (lo que le valió el Premio Turing), el **Multi-Level Feedback Queue (MLFQ)** busca resolver simultáneamente dos objetivos contrapuestos:
1. Minimizar el tiempo de respuesta para tareas interactivas (sin conocerlas de antemano).
2. Minimizar el tiempo de retorno para tareas pesadas de CPU.

La filosofía central de MLFQ es **aprender del pasado reciente del proceso para predecir su comportamiento futuro**.

---

### 4.2 Las Reglas Básicas y la Dinámica de Prioridades
MLFQ mantiene un conjunto de colas discretas de prioridad en el sistema, numeradas desde $Q_{max}$ (prioridad más alta) hasta $Q_0$ (prioridad más baja).

```
[Prioridad Alta]   Q2:  --- (A) ---> --- (B) --->  (Round Robin)
                   Q1:  --- (C) --->
[Prioridad Baja]   Q0:  --- (D) --->               (CPU-bound)
```

#### Reglas Fundamentales Iniciales:
* **Regla 1:** Si $\text{Prioridad}(A) > \text{Prioridad}(B)$, ejecuta $A$ y no $B$.
* **Regla 2:** Si $\text{Prioridad}(A) == \text{Prioridad}(B)$, $A$ y $B$ ejecutan en Round Robin en ese nivel.
* **Regla 3:** Cuando un trabajo ingresa al sistema, se lo coloca en la cola de mayor prioridad ($Q_{max}$).

#### Dinámica de Descenso (Primer Intento):
* **Regla 4a:** Si un proceso utiliza todo su quantum de tiempo de un solo tirón, su prioridad se reduce y baja una cola ($Q_i \rightarrow Q_{i-1}$).
* **Regla 4b:** Si un proceso cede voluntariamente la CPU antes de agotar su quantum (por ejemplo, para realizar una I/O), se mantiene en el mismo nivel de prioridad.

---

### 4.3 Fallas del Modelo Ingenuo: Inanición (Starvation) y Gaming del Scheduler

El algoritmo básico con las Reglas 1 a 4 presenta dos vulnerabilidades críticas en sistemas reales:

1. **Inanición (*Starvation*):**
   Si ingresa un volumen alto de procesos interactivos cortos (o si hay muchos procesos de usuario moviendo el ratón o tecleando), estos ocuparán constantemente las colas superiores ($Q_2$ y $Q_1$). Los procesos largos orientados a CPU que cayeron a la cola inferior ($Q_0$) nunca recibirán tiempo de procesador.

2. **Engaño/Ventajeo (*Gaming the Scheduler*):**
   Un programador malicioso o un código astuto puede adivinar el tamaño del quantum de tiempo (ej. 10 ms). Justo en el milisegundo 9.9, el proceso realiza una operación ficticia de I/O (escribir un byte en la consola o disco).
   * Bajo la **Regla 4b**, el planificador observa que el proceso cedió la CPU voluntariamente antes de agotar los 10 ms.
   * En consecuencia, el proceso retiene la prioridad máxima en $Q_{max}$ de forma indefinida, monopolizando el $99\%$ de la CPU.

---

### 4.4 Soluciones Definitivas: Accounting Acumulado y Priority Boost

Para sellar los agujeros de seguridad e inanición de MLFQ, la teoría refinó el conjunto de reglas agregando contabilidad estricta y reajuste periódico.

#### Solución a la Inanición (Priority Boost):
* **Regla 5 (Priority Boost):** Después de un período de tiempo estipulado $S$ (un intervalo global), **todos** los procesos existentes en el sistema se mueven a la cola de máxima prioridad ($Q_{max}$).

Esta regla mata dos pájaros de un tiro:
1. Garantiza que los procesos pesados atrapados en $Q_0$ no sufran inanición; cada $S$ segundos reciben tiempo de CPU en $Q_{max}$.
2. Si un proceso que era de CPU se transforma en interactivo (por ejemplo, terminó de procesar un archivo y ahora espera comandos del usuario), el *boost* revalúa dinámicamente su comportamiento.

```
Escenario SIN Boost:
Q2 | [B]-------------------------------------- (B) monopoliza
Q1 |     [B]
Q0 |         [========== A (Atrapado sin CPU) ==========]

Escenario CON Boost (Cada S ms):
Q2 | [B]-----[BOOST]--[A]--[B]-----------------
Q0 |         [=== A ===]      [=== A ===]
```

#### Solución al Ventajeo (Accounting / Allotment de Tiempo):
Para evitar que los programas engañen al sistema haciendo I/O antes de vencer el quantum, se reescriben las Reglas 4a y 4b en una **Regla 4 Unificada**:

* **Regla 4 (Definitiva):** Una vez que un proceso consume su **asignación total de tiempo (*time allotment*)** en un nivel dado (independientemente de cuántas veces haya cedido voluntariamente la CPU para I/O), su prioridad se reduce y baja un nivel en la cola.

El planificador ahora mantiene un contador acumulador en la estructura de control del proceso (`PCB`).

```c
// Lógica de contabilidad en el PCB del proceso para MLFQ
void update_process_time(struct proc *p, int time_spent) {
    p->time_spent_in_queue += time_spent;
    
    if (p->time_spent_in_queue >= QUEUE_ALLOTMENT(p->current_queue)) {
        p->time_spent_in_queue = 0; // Resetea el contador
        if (p->current_queue > 0) {
            p->current_queue--;     // Degrada de prioridad
        }
    }
}
```

---

### 4.5 Ajuste de Parámetros y Constantes Empíricas ("Voodoo Constants")
A pesar de su elegancia, MLFQ introduce una serie de parámetros de configuración que son difíciles de calcular analíticamente:
* ¿Cuántas colas de prioridad deben crearse? (Ej. 3, 8 o 16 colas).
* ¿Cuál debe ser el tamaño del quantum en cada nivel? (Es una buena práctica asignar quantums cortos a las colas altas para acelerar la respuesta de I/O, y quantums largos a las colas bajas para amortizar cambios de contexto en tareas pesadas).
* ¿Cada cuánto tiempo debe ejecutarse el *Priority Boost* ($S$)?

En la literatura de sistemas operativos, a estos parámetros ajustados empíricamente mediante pruebas de laboratorio se los conoce como **Constantes Vudú (*Voodoo Constants*)**, ya que parecen requerir cierta "magia negra" para funcionar de forma óptima en cargas de trabajo variadas.

---

## 5. El Completely Fair Scheduler (CFS) de Linux

A partir de la versión 2.6.23 del kernel de Linux, Ingo Molnar introdujo el **Completely Fair Scheduler (CFS)**, el cual reemplazó al antiguo planificador $O(1)$. CFS abandonó el concepto tradicional de colas múltiples y quantums fijos, adoptando un enfoque de **planificación proporcional de recursos (*Proportional Share Scheduling*)**.

---

### 5.1 Filosofía del Fair Scheduling y el Modelo Multitarea Ideal
CFS se basa en una abstracción teórica denominada **"Ideal Multi-tasking Hardware"**.

Imagine una CPU teórica capaz de ejecutar $N$ procesos simultáneamente dividiendo la frecuencia del procesador en $N$ partes iguales. En este entorno físico ideal, si hay 2 procesos, cada uno avanza exactamente al $50\%$ de la velocidad de la CPU en paralelo continuo, sin necesidad de realizar cambios de contexto ni sufrir interrupciones.

Dado que el hardware real solo tiene un número finito de núcleos y debe alternar la ejecución en el tiempo, CFS busca aproximar esta equidad matemática en una ventana temporal concreta.

#### El Objetivo de Optimización de CFS:
El objetivo fundamental de CFS es **minimizar la varianza del tiempo de ejecución virtual (`vruntime`) entre todos los procesos ejecutables**. CFS intenta mantener a todos los procesos alineados en la misma cantidad consumida de CPU ajustada por prioridad.

---

### 5.2 El Tiempo de Ejecución Virtual (`vruntime`)
El corazón de CFS es una variable mantenida en el `sched_entity` de cada `task_struct` llamada **`vruntime` (Virtual Runtime)**.

`vruntime` mide la cantidad acumulada de tiempo de reloj real que un proceso ha ejecutado en la CPU, ponderada inversamente por su peso de prioridad.

```
Proceso A (Prioridad Alta / Peso Alto):   vruntime avanza LENTAMENTE.
Proceso B (Prioridad Normal / Peso 1024): vruntime avanza en TIEMPO REAL.
Proceso C (Prioridad Baja / Peso Bajo):   vruntime avanza RÁPIDAMENTE.
```

* **Explicación mediante Metáfora:**
  Imagine una maratón donde los corredores tienen diferentes pesos de mochila (prioridades). El comité calificador no mide la distancia en kilómetros reales recorridos en el mapa, sino en "kilómetros virtuales". A un corredor profesional sin carga (alta prioridad) se le cuentan los kilómetros lentamente (cada 2 km reales le anotan 1 km virtual). A un corredor con una mochila pesada (baja prioridad) se le anotan los kilómetros rápidamente (cada 1 km real le anotan 3 km virtuales). El árbitro siempre manda a correr al que tenga **menos kilómetros virtuales acumulados** en la tabla.

---

### 5.3 Mapeo de Prioridades: `nice` y Pesos Exponenciales

En Unix y Linux, el usuario o el sistema asignan una prioridad a los procesos mediante el valor de **`nice`**, que oscila entre **$-20$ (máxima prioridad, "menos afable")** y **$+19$ (mínima prioridad, "muy afable")**. El valor por defecto de `nice` es $0$.

CFS convierte cada valor de `nice` a un valor de **peso (*weight*)** mediante una tabla fija definida en el código fuente del kernel (`prio_to_weight[40]`):

```c
// Tabla de mapeo de nice (-20 a +19) a peso en kernel/sched/core.c
static const int prio_to_weight[40] = {
 /* -20 */ 88761, 71755, 56483, 46273, 36291,
 /* -15 */ 29154, 23254, 18705, 14949, 11916,
 /* -10 */  9548,  7620,  6100,  4904,  3906,
 /*  -5 */  3121,  2501,  1991,  1586,  1277,
 /*   0 */  1024,   820,   655,   526,   423,
 /*   5 */   335,   272,   215,   172,   137,
 /*  10 */   110,    87,    70,    56,    45,
 /*  15 */    36,    29,    23,    18,    15,
};
```

Esta tabla sigue una progresión geométrica con una razón aproximada de $1.25$. Esto implica que por cada cambio de $1$ punto en el valor de `nice`, la cantidad de tiempo de CPU asignada al proceso cambia en aproximadamente un $10\%$.

#### Fórmula de Actualización del `vruntime`:
Cuando un proceso ejecuta durante un intervalo de tiempo real $\Delta\text{exec}$ (tiempo de reloj de pared), CFS actualiza su `vruntime` mediante la siguiente fórmula:

$$\text{vruntime} += \Delta\text{exec} \times \frac{\text{NICE\_0\_LOAD}}{\text{weight}}$$

Donde $\text{NICE\_0\_LOAD} = 1024$ (el peso correspondiente a `nice = 0`).

* Si un proceso tiene `nice = 0` ($	ext{weight} = 1024$):
  $$	ext{vruntime} += \Delta	ext{exec} 	imes rac{1024}{1024} = \Delta	ext{exec}$$
  El tiempo virtual es exactamente igual al tiempo real.
* Si un proceso tiene `nice = -5` ($	ext{weight} = 3121$):
  $$	ext{vruntime} += \Delta	ext{exec} 	imes rac{1024}{3121} pprox \Delta	ext{exec} 	imes 0.328$$
  Su `vruntime` crece mucho más despacio, por lo que permanecerá más tiempo a la izquierda del árbol y ejecutará más.

> **Importante:** El valor de `nice` tiene un significado **relativo, no absoluto**. Dos procesos con `nice = 0` y `nice = 5` se repartirán la CPU exactamente en la misma proporción que dos procesos con `nice = 10` y `nice = 15`, ya que la relación de pesos entre sus niveles es idéntica ($rac{1024}{335} pprox rac{110}{36} pprox 3.05$).

---

### 5.4 Estructura de Datos: Árbol Rojo-Negro (`tasks_timeline`)

Para organizar todos los procesos listos (*RUNNABLE*) en orden estricto de `vruntime`, CFS no utiliza listas ni colas, sino un **Árbol Rojo-Negro (*Red-Black Tree*)** auto-balanceado, apuntado por el campo `tasks_timeline` de la estructura `struct cfs_rq`.

```
                    ( vruntime = 27 )
                       /                             /                    ( vruntime = 19 )      ( vruntime = 34 )
           /        \              /           [rb_leftmost]  (vr = 25)    (vr = 31)   (vr = 65)
   ( vr = 7 )
```

#### Propiedades del Árbol Rojo-Negro en CFS:
1. **Selección del Próximo Proceso en $O(1)$:** CFS siempre selecciona para ejecutar al proceso con el `vruntime` más bajo. Este nodo se encuentra en el extremo más a la izquierda del árbol (`rb_leftmost`). Para lograr un tiempo de selección de $O(1)$, el kernel mantiene un puntero directo al nodo `rb_leftmost` en memoria, evitando recorrer el árbol en cada decisión de planificación.
2. **Inserción y Eliminación en $O(\log N)$:** Cuando el proceso actual en CPU consume su tiempo, su `vruntime` se incrementa, se reinserta en el árbol en su nueva posición hacia la derecha, y el árbol se auto-balancea en $O(\log N)$.

---

### 5.5 Control Dinámico de Latencia: `sched_latency` y `min_granularity`

CFS no utiliza un quantum fijo, sino que calcula dinámicamente la ventana de tiempo que le otorga a un proceso en función de dos parámetros del sistema:

1. **Latencia Objetivo (*Target Latency* - `sched_latency`):**
   Es el período de tiempo de reloj dentro del cual el planificador **intenta que todos los procesos ejecutables en el sistema se ejecuten al menos una vez**. El valor típico por defecto en Linux es de $6	ext{ ms}$ a $20	ext{ ms}$.

#### Cálculo del Timeslice Dinámico:
Dado un entorno con $N$ procesos y una latencia objetivo `sched_latency`, el tiempo de CPU asignado individualmente al proceso $i$ es:

$$	ext{timeslice}_i = 	ext{sched\_latency} 	imes rac{	ext{weight}_i}{\sum_{j=1}^{N} 	ext{weight}_j}$$

2. **Granularidad Mínima (*Minimum Granularity* - `min_granularity`):**
   Si hay miles de procesos en ejecución, dividir `sched_latency` entre $N$ resultaría en un `timeslice` infinitesimal (por ejemplo, nanosegundos). Esto provocaría un *thrashing* destructivo por cambios de contexto masivos.
   Para evitarlo, el parámetro `min_granularity` (típicamente entre $0.75	ext{ ms}$ y $1.5	ext{ ms}$) establece un piso absoluto. Ningún proceso recibirá un `timeslice` menor que `min_granularity`.

> **Mecanismo de Expansión de Latencia:** Si $N 	imes 	ext{min\_granularity} > 	ext{sched\_latency}$, la latencia objetivo se expande dinámicamente a $N 	imes 	ext{min\_granularity}$. En este escenario extremo, la equidad estricta se sacrifica momentáneamente para preservar el rendimiento del sistema amortizando el overhead de cambio de contexto.

* **Ejemplo Práctico/Técnico de Cálculo de Timeslice:**
  Suponga un sistema con `sched_latency = 20 ms`.
  * **Caso 1: Dos procesos de igual prioridad (`nice = 0`, peso 1024).**
    $$	ext{timeslice}(A) = 20	ext{ ms} 	imes rac{1024}{1024 + 1024} = 10	ext{ ms}$$
    $$	ext{timeslice}(B) = 20	ext{ ms} 	imes rac{1024}{1024 + 1024} = 10	ext{ ms}$$

  * **Caso 2: Proceso $A$ (`nice = 0`, peso 1024) y Proceso $B$ (`nice = 5`, peso 335).**
    $$	ext{Peso Total} = 1024 + 335 = 1359$$
    $$	ext{timeslice}(A) = 20	ext{ ms} 	imes rac{1024}{1359} pprox 15.07	ext{ ms}$$
    $$	ext{timeslice}(B) = 20	ext{ ms} 	imes rac{335}{1359} pprox 4.93	ext{ ms}$$

---

### 5.6 Normalización de Tiempos y Manejo de Desbloqueos

¿Qué sucede cuando un proceso se bloquea a esperar I/O (o se duerme con `sleep()`) durante 10 minutos y luego se despierta?

Si el proceso durmió durante 10 minutos, su `vruntime` se quedó congelado en un valor antiguo, mientras que los demás procesos en el árbol continuaron ejecutando y aumentando sus `vruntime` a miles de milisegundos.

Si CFS reinsertara al proceso recién despierto con su `vruntime` antiguo intacto, el proceso se colocaría en el extremo más a la izquierda del árbol y **monopolizaría la CPU de forma ininterrumpida durante minutos** hasta equiparar su `vruntime` con el de los demás.

#### La Solución de Normalización (`min_vruntime`):
Para evitar este monopolio al despertar:
1. CFS mantiene una variable por cola llamada `min_vruntime`, que rastrea el `vruntime` del proceso más a la izquierda del árbol activo.
2. Cuando un proceso se despierta, CFS ajusta su `vruntime` estableciéndolo como:
   $$	ext{vruntime}_{	ext{despierto}} = \max(	ext{vruntime}_{	ext{antiguo}}, 	ext{min\_vruntime} - \Delta_{	ext{latencia}})$$
3. Esto le permite al proceso recién despierto colocarse hacia la izquierda del árbol para responder rápidamente al evento de I/O, pero sin permitirle acumular un "crédito masivo" que destruya la equidad del sistema.

Asimismo, periódicamente CFS sustrae `min_vruntime` de todos los nodos del árbol para mantener los números en rangos pequeños y prevenir el desbordamiento de enteros (*integer overflow*).

---

## 6. Planificación en Entornos Modernos: Grupos, Hilos y Multiprocesamiento

---

### 6.1 Planificación de Grupos (Group Scheduling y cgroups)

Suponga un servidor compartido por dos usuarios: el Usuario $A$ abre una sola terminal con $1$ proceso. El Usuario $B$ ejecuta un script que lanza $99$ procesos concurrentes.

Bajo un esquema de planificación equitativa por proceso puro, el sistema detecta $100$ procesos en total. El Usuario $A$ recibirá solo el $1\%$ de la CPU, mientras que el Usuario $B$ monopolizará el $99\%$.

Para evitar este abuso, Linux implementa **Group Scheduling** apoyado en la tecnología de **cgroups (Control Groups)**.

```
                   [ CPU Total ]
                  /                     ( Grupo Usuario A )   ( Grupo Usuario B )
             [ 50% ]               [ 50% ]
                |                 /   |               Proc A1 (50%)     Proc B1 Proc B2 ... Proc B99 (0.5% c/u)
```

#### Funcionamiento de Group Scheduling:
CFS aplica el algoritmo de equidad jerárquicamente en dos niveles:
1. **Nivel 1 (Grupos):** Distribuye la CPU equitativamente entre los cgroups definidos (ej. 50% para el Grupo A, 50% para el Grupo B).
2. **Nivel 2 (Procesos individuales):** Dentro de la cuota del Grupo B, divide su 50% asignado equitativamente entre sus 99 procesos.

Cada grupo mantiene su propia estructura `cfs_rq` y su propio árbol Rojo-Negro interno, actuando como una entidad planificable (*schedulable entity*) de nivel superior.

---

### 6.2 Manejo de Hilos (Threads), `clone()` y NPTL
A diferencia de otros sistemas operativos que tienen estructuras separadas y complejas para manejar procesos e hilos, **el kernel de Linux no distingue conceptualmente entre un proceso y un hilo**. Ambos son representados internamente por la misma estructura de datos: `struct task_struct`.

La creación de hilos en Linux mediante la biblioteca POSIX (NPTL - *Native POSIX Threads Library*) utiliza la llamada al sistema de bajo nivel **`clone()`**, pasando modificadores de comportamiento (*flags*):

```c
// Invocación conceptual de clone() para crear un hilo de usuario
clone(CLONE_VM | CLONE_FILES | CLONE_FS | CLONE_SIGHAND | CLONE_THREAD, stack_ptr);
```

#### Interpretación de los Flags:
* `CLONE_VM`: Padre e hijo comparten el mismo espacio de direcciones virtuales (memoria).
* `CLONE_FILES`: Comparten la tabla de descriptores de archivos.
* `CLONE_SIGHAND`: Comparten los manejadores de señales.
* `CLONE_THREAD`: El nuevo hilo se coloca en el mismo **Thread Group** que el padre.

#### Estructura de Identificación:
* **PID (Process ID):** Para el kernel, cada hilo tiene un ID único de tarea llamado **TID (Thread ID)**.
* **TGID (Thread Group ID):** Es el PID del hilo principal (lider de grupo). Cuando una aplicación llama a `getpid()`, la biblioteca C retorna el `TGID`, dando la ilusión en espacio de usuario de que todos los hilos pertenecen al mismo proceso.

CFS planifica cada `task_struct` (hilo) de forma individual en los núcleos de CPU.

---

### 6.3 Arquitecturas Multiprocesador: Afinidad de Caché y Coherencia

Planificar en sistemas con múltiples núcleos físicas o CPUs (*SMP - Symmetric Multiprocessing*) introduce dos desafíos físicos principales:

1. **Coherencia y Afinidad de Caché (*Cache Affinity*):**
   Cuando un proceso ejecuta en la CPU 0, llena las memorias caché L1 y L2 locales con sus datos e instrucciones. Si el planificador desaloja al proceso y luego lo reanuda en la CPU 1, todas las cachés locales de la CPU 1 contendrán "basura". El proceso sufrirá fallos masivos de caché (*cache misses*), teniendo que traer todo desde la memoria RAM lenta. Por lo tanto, el planificador debe intentar **mantener un proceso ejecutando en la misma CPU (afinidad de caché)**.

2. **Contención de Locks (*Lock Contention*):**
   Si varios núcleos intentan acceder y modificar la misma estructura de datos del planificador al mismo tiempo, los mecanismos de sincronización (mutexes/spinlocks) congelarán los núcleos esperando su turno.

---

### 6.4 Modelos de Colas en Multiprocesamiento: SQMS vs. MQMS y Balanceo de Carga

#### 1. Single-Queue Multiprocessor Scheduling (SQMS):
Toma el diseño de un solo núcleo y coloca una **única cola global de procesos** compartida por todas las CPUs.

```
           [ Cola Global Única (Lock) ]
          /            |                   ( CPU 0 )     ( CPU 1 )     ( CPU 2 )
```

* **Desventajas Críticas:**
  * **Contención de Lock masiva:** Cada CPU debe adquirir un *spinlock* global cada vez que busca un proceso. A medida que aumenta el número de núcleos (ej. 64 cores), la CPU pasa más tiempo esperando el lock que ejecutando código.
  * **Cero Afinidad de Caché:** Un proceso $A$ ejecuta un quantum en la CPU 0, el siguiente en la CPU 2, el siguiente en la CPU 1, destruyendo el rendimiento de las memorias caché.

#### 2. Multi-Queue Multiprocessor Scheduling (MQMS):
Es el esquema utilizado por los sistemas operativos modernos (incluyendo Linux CFS). Consiste en asignar **una cola/árbol Rojo-Negro independiente a cada CPU/núcleo del sistema**.

```
    ( Árbol CPU 0 )       ( Árbol CPU 1 )       ( Árbol CPU 2 )
           |                     |                     |
        [ CPU 0 ]             [ CPU 1 ]             [ CPU 2 ]
```

* **Ventajas:**
  * **Sin contención de locks:** Cada CPU accede exclusivamente a su propio árbol en el $99\%$ de los casos.
  * **Excelente afinidad de caché:** Un proceso permanece en la misma CPU a lo largo de toda su vida útil por defecto.

#### Mecanismo de Balanceo de Carga (*Load Balancing* y *Work Stealing*):
Si la CPU 0 recibe 10 procesos pesados mientras que la CPU 1 se queda sin procesos (inactiva), el sistema sufre un desbalance severo.

Para corregirlo, MQMS implementa **Robo de Trabajo (*Work Stealing*)**:
1. Periódicamente, o cuando una CPU entra en estado inactivo (*idle*), examina los árboles de las demás CPUs.
2. Si detecta que otra CPU está sobrecargada, adquiere el lock de esa cola específica y **migra/roba** una o más tareas hacia su propia cola.
3. Al migrar una tarea de una CPU a otra, CFS **reajusta y normaliza su `vruntime`** respecto al `min_vruntime` del nuevo árbol destino para integrarlo de forma justa en el nuevo núcleo.
