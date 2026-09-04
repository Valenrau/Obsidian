# Arquitectura de Sistemas Operativos: Fundamentos, Abstracciones y la API de Unix

## 1. Fundamentos e Evolución Histórica de los Sistemas Operativos

La base conceptual de la computación moderna y los sistemas operativos descansa sobre dos pilares teóricos fundamentales formulados a mediados del siglo XX:

- **La Máquina de Turing (Alan Turing):** Propuesta como un modelo matemático abstracto, la Máquina de Turing sentó las bases teóricas de la computabilidad. Conceptualmente, consiste en una cinta de longitud infinita dividida en celdas (que actúa como memoria secuencial), un cabezal capaz de leer y escribir símbolos sobre ella, un registro de estado interno, y una tabla de transiciones (un conjunto de instrucciones equivalente al *instruction set* de un procesador moderno). Turing demostró que cualquier problema que sea algorítmicamente computable puede ser resuelto por esta máquina fundamental.
- **La Arquitectura de Von Neumann (John von Neumann):** Tradujo el concepto teórico de la Máquina de Turing en una arquitectura física viable para computadoras de propósito general. Esta estructura se compone principalmente de tres abstracciones físicas: 
  1. **Unidad Central de Procesamiento (CPU):** Encargada de ejecutar las instrucciones mediante el ciclo secuencial de **búsqueda de instrucción en memoria (*****fetch*****), decodificación (*****decode*****) y ejecución (*****execute*****)**.
  2. **Memoria de Acceso Aleatorio (RAM):** Una matriz gigante de bytes direccionable donde coexisten tanto los datos como las instrucciones del programa (concepto de programa almacenado).
  3. **Canales de Comunicación / Dispositivos de Entrada/Salida (I/O):** Canales secuenciales orientados a transferencias de flujos de datos (streams).

### Del Mainframe a la Computación Moderna

En los albores de la computación, las máquinas eran inmensas estructuras físicas ("heladeras" o mainframes) que carecían de sistemas operativos dinámicos. Al principio, para cargar un programa era necesario reconfigurar físicamente cables o interruptores. Con el tiempo, surgió la necesidad de automatizar la ejecución de tareas consecutivas sin intervención humana, dando origen a los primeros sistemas operativos de procesamiento por lotes (*batch processing*).

> **Caso Histórico de Clementina:**
>
> Un hito histórico de la computación científica en Latinoamérica ocurrió en 1961 con la llegada de "Clementina" (una computadora Mercury de Ferranti) a la Ciudad Universitaria de la Universidad de Buenos Aires (UBA), convirtiéndose en la primera computadora científica del continente utilizada para la investigación académica y la planificación estatal. Su historia quedó trágicamente interrumpida en 1966 durante la dictadura militar en el episodio conocido como "La Noche de los Bastones Largos", cuando las fuerzas de seguridad irrumpieron violentamente en las facultades de la UBA, provocando la renuncia masiva y el exilio de gran parte del cuerpo docente y científico.

En la actualidad, el ecosistema de hardware ha divergido significativamente del modelo secuencial puro de Von Neumann:

- **Cómputo en la Nube (*****Cloud Computing*****):** En lugar de una única máquina física aislada, los sistemas distribuidos modernos enlazan miles de servidores independientes mediante redes TCP/IP, compartiendo recursos de manera virtual y dinámica.
- **Coprecesamiento Heterogéneo (CPU + GPU):** Las aplicaciones modernas (como el entrenamiento de redes neuronales) ejecutan operaciones masivamente paralelas en Unidades de Procesamiento Gráfico (GPU), mientras que la CPU actúa como el orquestador principal del flujo de datos.

A pesar de esta masiva evolución y diversificación de hardware (servidores, GPUs, teléfonos inteligentes y PCs), **todos estos entornos comparten una capa común indispensable: el Sistema Operativo**. El S.O. moderno actúa como el software encargado de hacer que la ejecución de programas complejos sobre hardware real parezca una tarea sencilla, aislando a las aplicaciones de las complejidades de la electrónica física subyacente.

---

## 2. El Rol del Sistema Operativo Moderno: Tres Metáforas Funcionales

Para formalizar qué hace y por qué existe un sistema operativo de propósito general (como Linux), la literatura académica utiliza tres metáforas clásicas que agrupan sus funciones fundamentales:

```
                     ┌───────────────────────────────┐
                     │          APLICACIONES         │
                     └───────────────┬───────────────┘
                                     │
                                     ▼ (Llamadas al Sistema / API)
┌─────────────────────────────────────────────────────────────────────────┐
│                           SISTEMA OPERATIVO                             │
├───────────────────────┬─────────────────────────┬───────────────────────┤
│        ÁRBITRO        │       ILUSIONISTA       │       CONECTOR        │
│  - Gestión recursos   │  - Abstracción hardware │  - Servicios comunes  │
│  - Aislamiento fallos │  - Memoria Virtual      │  - APIs estándar      │
│  - Seguridad/Permisos │  - Concurrencia (CPU)   │  - Drivers e I/O      │
└───────────────────────┴────────────┬────────────┴───────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                                HARDWARE                                 │
└─────────────────────────────────────────────────────────────────────────┘

```

### A. El Árbitro (*Referee*)

El Árbitro se encarga de la **gestión y distribución de los recursos físicos** entre los múltiples programas y usuarios que compiten por ellos en un mismo sistema. Sus responsabilidades críticas son:

- **Asignación Equitativa:** Decide qué proceso obtiene acceso a qué recurso (CPU, memoria, disco) y durante cuánto tiempo.
- **Aislamiento y Protección:** Garantiza que un error en una aplicación (como una división por cero o un puntero inválido) no propague sus efectos al resto de los procesos independientes ni provoque la caída completa de la computadora. En sistemas antiguos como MS-DOS, un error en una aplicación provocaba pantallas azules o congelamientos totales de la máquina debido a la falta de aislamiento; en los sistemas operativos modernos, el núcleo detecta el fallo y aborta de forma segura únicamente al proceso infractor.
- **Seguridad y Permisos:** Evita que programas maliciosos o no autorizados accedan a partes de la memoria o archivos de otros usuarios.

### B. El Ilusionista (*Illusionist*)

El Ilusionista utiliza la **virtualización** para crear abstracciones poderosas sobre los componentes de hardware limitados y complejos. Su objetivo es simplificar drásticamente la vida de los programadores:

- **Ilusión de Monopolio de CPU:** Hace creer a cada programa que tiene el control exclusivo del procesador. En realidad, el planificador (*scheduler*) del S.O. interrumpe, extrae e introduce procesos en la CPU a un ritmo extremadamente veloz (por ejemplo, otorgando ráfagas o *quanta* de ejecución de 200 milisegundos a cada proceso), generando la ilusión óptica de concurrencia perfecta.
- **Ilusión de Memoria Infinita:** Provee a cada aplicación un espacio de direccionamiento virtual contiguo y limpio. Aunque la memoria física real esté sumamente fragmentada y ocupada por docenas de procesos ajenos, la aplicación opera bajo la ilusión de que se encuentra en un entorno de memoria dedicado únicamente para ella.

### C. El Conector (*Glue*)

El Conector actúa como un **unificador de interfaces y servicios comunes**, proveyendo abstracciones de software homogéneas que facilitan la comunicación y la interoperabilidad de las aplicaciones.

- **Independencia de Hardware:** Oculta los detalles mecánicos y electrónicos de los dispositivos periféricos a través de un diseño unificado. El desarrollador de un procesador de texto escribe datos a través de una API de archivos estándar, sin necesidad de saber si el almacenamiento físico subyacente es un disco de estado sólido (SSD), un disco mecánico SATA, o un volumen en la nube montado por red.
- **Servicios Compartidos:** Proporciona mecanismos para la interacción controlada entre aplicaciones, tales como portapapeles comunes (copiar y pegar), bases de datos de sistema, y canales de comunicación interprocesos (IPC).

---

## 3. El Concepto de Virtualización y Multiplexación

La virtualización es la técnica maestra mediante la cual el sistema operativo transforma un recurso físico real (frecuentemente limitado, crudo y complejo) en una abstracción virtual mucho más general, poderosa y fácil de utilizar. Esta técnica se divide en dos enfoques principales de administración de recursos: la multiplexación en tiempo y la multiplexación en espacio.

### Multiplexación en Tiempo vs. Multiplexación en Espacio

| Tipo de Multiplexación Recurso Físico Principal Mecanismo de Funcionamiento Ejemplo Práctico  |                                         |                                                                                                                                                                  |                                                                                                                                                                      |
| --------------------------------------------------------------------------------------------- | --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **En Tiempo**                                                                            | **CPU (Procesador)**               | Un único recurso físico es asignado consecutivamente a diferentes procesos por pequeños intervalos temporales llamados *turnos* o *quanta*.                 | El planificador de la CPU rotando la ejecución de 100 procesos diferentes en ráfagas de milisegundos para simular multiprogramación simultánea.             |
| **En Espacio**                                                                           | **Memoria RAM** y **Disco Rígido** | Un gran recurso físico es fraccionado en múltiples porciones geográficas independientes, asignando un rango espacial exclusivo a cada proceso participante. | La memoria principal dividida en marcos de página virtuales asignados a las aplicaciones de forma aislada, o la división física de bloques lógicos en un disco. |

#### Un Ejemplo de la Física a la Virtualización

Cuando programamos software de alto nivel, rara vez nos percatamos de la existencia de colas de eventos físicas o límites electrónicos reales (memorias saturadas, cables de red sobrecargados). Vivimos en el "mundo de fantasía" estructurado por las abstracciones del S.O.. Sin embargo, cuando los sistemas sufren caídas de rendimiento o cuellos de botella en producción, un buen ingeniero de sistemas debe "levantar el capot" de esta virtualización y entender cómo el hardware físico subyacente (guiado por las leyes de la física y la electrónica real) interactúa con los recursos virtuales administrados por el kernel.

---

## 4. La Arquitectura Unix-like y la Filosofía de Diseño

El sistema operativo **Unix**, concebido originalmente en **1969 por Ken Thompson y Dennis Ritchie en los Laboratorios Bell**, sentó las bases de diseño arquitectónico y de API que rigen casi la totalidad de la computación contemporánea (incluyendo GNU/Linux, macOS, iOS y Android). 

### La Filosofía Unix

La asombrosa longevidad de Unix se explica por un conjunto de principios de diseño simples, elegantes y robustos que han resistido el paso de décadas de evolución tecnológica sin requerir modificaciones estructurales severas:

1. **Simplicidad en las Funcionalidades:** Cada programa debe hacer una sola cosa bien. Si se requiere funcionalidad adicional, en lugar de complejizar un programa existente agregando características parasitarias, se debe crear una herramienta nueva e independiente que se especialice en esa tarea.
2. **Interoperabilidad por Texto Plano:** Los programas deben poder conectarse de forma transparente. Para lograr esto, la salida de cualquier programa debe diseñarse de modo que pueda ser utilizada directamente como la entrada de otro. La ausencia de formatos propietarios, binarios complejos o encabezados rígidos (privilegiando el intercambio de flujos de bytes limpios) permite combinar comandos simples en potentes flujos de trabajo (*pipelines*).
3. **Modularidad Dinámica:** Promover la construcción rápida de herramientas modulares y temporales en lugar de monolitos pesados y difíciles de depurar.

### El [Kernel](<02 Kernel>): Línea de Demarcación de Privilegios

La división fundamental de seguridad y operación en una arquitectura Unix-like se compone de dos anillos de privilegio rigurosamente separados por el hardware del procesador:

```
┌─────────────────────────────────────────────────────────────────────────┐
│ USER-LAND (Espacio de Usuario - No Privilegiado)                        │
│   ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐      │
│   │   Aplicación     │  │   Aplicación     │  │      Bash        │      │
│   └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘      │
│            │ (Llamadas de Alto Nivel / Funciones Estándar de C)         │
│            ▼                                                            │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │ Biblioteca Estándar de C (glibc / unistd.h / stdlib)          │      │
│   └──────────────────────────────┬───────────────────────────────┘      │
└──────────────────────────────────┼──────────────────────────────────────┘
===================================┼=======================================
                                   ▼ (Interfaz de Llamadas al Sistema - Syscalls)
┌──────────────────────────────────┴──────────────────────────────────────┐
│ KERNEL-LAND (Espacio del Núcleo - Privilegiado)                         │
│                                                                         │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │                        EL KERNEL                             │      │
│   │  (Process Management, Memory Management, VFS, Scheduling)    │      │
│   └──────────────────────────────┬───────────────────────────────┘      │
│                                  │ (Drivers / Código Específico)         │
│                                  ▼                                      │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │                Controladores de Dispositivo (Drivers)        │      │
│   └──────────────────────────────┬───────────────────────────────┘      │
└──────────────────────────────────┼──────────────────────────────────────┘
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                                HARDWARE                                 │
│         (CPU, Memoria RAM, Discos, Placa de Red, Periféricos)           │
└─────────────────────────────────────────────────────────────────────────┘

```

1. **User-land (Espacio de Usuario):** Donde se ejecutan las aplicaciones de usuario (navegadores, editores de texto, procesos de servidor) y las bibliotecas estándar (como `unistd.h` o la biblioteca estándar de C). Los programas en user-land se ejecutan en un modo de CPU no privilegiado. Tienen estrictamente prohibido acceder directamente a la memoria física de otros procesos, escribir directamente en los registros del procesador, o enviar comandos físicos al hardware de disco.
2. **Kernel-land (Espacio del Núcleo):** El Kernel o núcleo es la barrera absoluta entre las aplicaciones y el hardware físico. Es un software altamente privilegiado que corre en el modo más protegido del procesador (Anillo 0) y posee control absoluto del hardware. Ningún programa ordinario puede saltarse el kernel para acceder a un periférico; todo paso requiere obligatoriamente solicitar formalmente un servicio al kernel.

Los servicios expuestos por el kernel se denominan **Llamadas al Sistema (**[**System Calls**](<System calls, procesos y su relación con el kernel>) **o** ***Syscalls*****)**. El pasaje del Espacio de Usuario al Espacio del Núcleo se realiza mediante instrucciones de hardware especiales de la CPU (como interrupciones de software o instrucciones específicas como `syscall` o `sysenter`), las cuales suspenden la ejecución segura del proceso en user-land, elevan el nivel de privilegios del CPU, y ejecutan una rutina específica y segura dentro del kernel.

---

## 5. La Abstracción de Archivos en Unix

Una de las ideas más brillantes y elegantes introducidas por Dennis Ritchie y Ken Thompson es la de generalizar el acceso de entrada/salida bajo un único axioma: **En Unix, (casi) todo es un archivo**.

Si un dispositivo físico o un canal lógico se comporta con semántica de archivo (es decir, acepta operaciones de abrir, leer flujos secuenciales de bytes, escribir en ellos y cerrarlos), el sistema operativo lo expone bajo la abstracción uniforme de un archivo. Esto permite unificar la API de bajo nivel a cuatro operaciones fundamentales:

- `open()`: Abre una ruta de acceso a un recurso.
- `close()`: Cierra el acceso al recurso.
- `read()`: Lee un bloque secuencial de bytes del recurso.
- `write()`: Escribe un bloque secuencial de bytes en el recurso.

### Tipos de Recursos Representados como Archivos

Gracias a esta asombrosa generalidad, un programador de Unix interactúa de la misma manera exacta con los siguientes recursos radicalmente dispares:

1. **Archivos Regulares:** Documentos de texto, imágenes, binarios compilados almacenados en un file system real.
2. **Dispositivos de Entrada/Salida Físicos (I/O Devices):** Representados bajo directorios especiales de dispositivos (ej. `/dev/sda` para discos enteros, `/dev/nvme0n1` para unidades SSD modernas). Escribir crudo en `/dev/sda` permite manipular directamente los bloques de almacenamiento físicos.
3. **Sockets de Red:** Conexiones de red TCP o UDP se exponen como archivos. Para enviar datos a un servidor remoto, basta con hacer un `write()` sobre el descriptor del socket de red; para recibir información, se realiza un `read()`.
4. **Pipes y Tuberías:** Canales de comunicación interprocesos en memoria.
5. **Pseudoterminales (PTY):** Usadas por emuladores de terminal o multiplexores como tmux para emular interfaces de terminal físicas (ej. `/dev/pts/X`).

### File Descriptors (Descriptores de Archivos)

Cuando un proceso invoca exitosamente la llamada al sistema `open()`, el kernel asocia el recurso abierto a una entrada en una estructura de datos interna del proceso llamada **Tabla de Descriptores de Archivos**. Esta tabla se almacena y administra con privilegios estrictos en el espacio del kernel para cada proceso por separado, impidiendo que el proceso manipule directamente sus punteros internos.

En lugar de exponer punteros complejos, el kernel devuelve al proceso un entero no negativo simple llamado **File Descriptor (FD)**. Este entero actúa únicamente como un índice (o llave de acceso) a la tabla del kernel para referenciar la estructura física abierta.

```
ESPACIO DE USUARIO (User Space)         │ ESPACIO DEL KERNEL (Kernel Space)
                                        │
┌─────────────────────────────────┐     │      Tabla de Descriptores del Proceso
│ Proceso en ejecución            │     │     ┌────┬─────────────────────────────┐
│                                 │     │     │ FD │  Puntero a Archivo Abierto  │
│   int fd = open("salida.txt")   ├─────┼────►├────┼─────────────────────────────┤
│   // fd toma valor entero, ej: 3│     │     │  0 │  [stdin -> tty]             │
│                                 │     │     ├────┼─────────────────────────────┤
│   write(fd, "Hola", 4);         │     │     │  1 │  [stdout -> tty]            │
│                                 │     │     ├────┼─────────────────────────────┤
│                                 │     │     │  2 │  [stderr -> tty]            │
│                                 │     │     ├────┼─────────────────────────────┤
└─────────────────────────────────┘     │     │  3 │  [Archivo Abierto salida.txt]─► Disco
                                        │     └────┴─────────────────────────────┘

```

#### Los Tres Descriptores Estándar Preabiertos

Por estricta convención de Unix, al iniciar, cada nuevo proceso hereda de forma predeterminada tres file descriptors que ya vienen "cableados" y listos para ser utilizados sin necesidad de una llamada explícita a `open()`:

- **Descriptor 0 (****`STDIN_FILENO`****):** Entrada estándar, cableada convencionalmente al teclado o dispositivo de entrada.
- **Descriptor 1 (****`STDOUT_FILENO`****):** Salida estándar, cableada por defecto a la terminal.
- **Descriptor 2 (****`STDERR_FILENO`****):** Salida de errores estándar, típicamente cableada a la terminal para canalizar mensajes de diagnóstico de manera independiente de la salida estándar.

> **Análisis de la Redirección y el Comando** **`echo`****:**
>
> El comando estándar `echo` es un ejemplo perfecto de desacoplamiento. La implementación interna de `echo` simplemente recibe una lista de argumentos y ejecuta la syscall `write(1, mensaje, longitud)`. `echo` no posee código específico para abrir la pantalla o manipular la GPU; asume en abstracto que el Descriptor 1 es donde debe escribir. Si el usuario ejecuta `echo "Hola" > salida.txt` en la consola, el programa *shell* (como Bash), antes de ejecutar el programa `echo`, intercepta la redirección, abre el archivo `salida.txt` y modifica la tabla de file descriptors para que el Descriptor 1 de `echo` apunte directamente al archivo y no a la consola. El comando `echo` corre exactamente igual, escribe en el descriptor 1, pero su salida termina físicamente en el disco gracias a la mediación del S.O..

---

## 6. La Abstracción de Proceso y la API de Unix

Un **Proceso** es la abstracción dinámica primordial de un sistema operativo, formalizado como **un programa en ejecución**. Mientras que el *programa* es un ente estático (un archivo ejecutable inerte almacenado en el disco), el *proceso* es una entidad viva que se ejecuta activamente en el procesador.

Cada proceso se ejecuta de manera estrictamente aislada en el espacio de usuario (*user-land*). Internamente, cuenta con una estructura de memoria bien estructurada:

```
Dirección Alta ┌────────────────────────┐
               │    STACK (Pila)        │  ◄── Crece hacia abajo (Llamadas a funciones, variables locales)
               ├────────────────────────┤
               │           ▼            │
               │                        │
               │           ▲            │
               ├────────────────────────┤
               │    HEAP (Montículo)    │  ◄── Crece hacia arriba (Asignación dinámica de memoria: malloc)
               ├────────────────────────┤
               │    DATA (Datos)        │  ◄── Variables globales e inicializadas
               ├────────────────────────┤
               │    CODE (Código)       │  ◄── Instrucciones binarias puras del ejecutable
Dirección Baja └────────────────────────┘

```

A nivel administrativo, el kernel asocia cada proceso a un identificador numérico único e intransferible en un instante dado, denominado **Process ID (PID)**. El kernel mantiene una estructura de datos central e inaccesible llamada **Tabla de Procesos (*****Process Table*****)** que almacena de forma aislada la información de control de cada proceso en ejecución, incluyendo su mapa de memoria física, sus credenciales de usuario y su tabla de descriptores de archivos.

### La API de Procesos de Unix

La API estándar para administrar procesos en Unix se compone de un conjunto pequeño de llamadas al sistema altamente combinables, entre las que destacan `fork()`, `wait()`, `execve()`, `dup2()` y `pipe()`.

#### A. La Syscall `fork()`: La Bifurcación Borgiana

La única forma de crear un nuevo proceso en Unix desde el espacio de usuario consiste en invocar la syscall `fork()`. Al ser llamada, ocurre una bifurcación conceptual que recuerda al cuento *"El jardín de senderos que se bifurcan"* de Jorge Luis Borges: el flujo de ejecución único se divide instantáneamente en dos universos paralelos e independientes:

1. **El Proceso Padre:** Aquel que originalmente invocó la llamada.
2. **El Proceso Hijo:** Un nuevo proceso creado como una **copia exacta e idéntica en memoria** del proceso padre.

##### Qué ocurre internamente durante un `fork()`:

- Se crea una nueva entrada en la *Process Table* del kernel con un PID único para el hijo.
- Se genera una copia idéntica del espacio de memoria del padre (Código, Datos, Heap y Stack) para el hijo.
- **Importante:** A partir de ese instante, las memorias de ambos procesos quedan **completamente aisladas**. Cualquier modificación de variable realizada por el hijo no afectará las variables del padre, ni viceversa. No es memoria compartida.
- El hijo hereda de forma exacta una **copia de la tabla de descriptores de archivos** del padre. Ambos procesos tendrán los mismos descriptores numéricos apuntando a las mismas estructuras de archivos abiertos en el kernel.
- **Retorno Diferencial:** La única forma que poseen el padre y el hijo para saber en qué "universo" se encuentran tras la llamada es el valor de retorno de `fork()`: 
  - En el **proceso hijo**, `fork()` retorna estrictamente el valor **`0`**.
  - En el **proceso padre**, `fork()` retorna el **PID del hijo recién creado** (un valor entero mayor a cero).
  - Si la llamada falla, retorna `-1`.

```c
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>

int main() {
    int variable_control = 42;
    pid_t pid = fork();

    if (pid < 0) {
        // Ocurrió un error al duplicar el proceso
        perror("Fallo de fork");
        return 1;
    } 
    else if (pid == 0) {
        // Bloque ejecutado exclusivamente por el hijo
        variable_control = 99; // Modificación en memoria aislada
        printf("[Hijo] PID: %d, variable_control modificada: %d\n", getpid(), variable_control);
    } 
    else {
        // Bloque ejecutado exclusivamente por el padre
        printf("[Padre] PID: %d, hijo creado con PID: %d, variable_control intacta: %d\n", 
               getpid(), pid, variable_control);
    }
    // Código común que ejecutan ambos de forma independiente tras la bifurcación
    return 0;
}

```

*Nota técnica:* Tras invocar `fork()`, el orden de ejecución física de los procesos es completamente indeterminado (concurrencia pura). No hay garantías de si el procesador elegirá planificar primero al padre o al hijo.

---

#### B. La Syscall `wait()`: Mecanismo de Sincronización

Debido al indeterminismo en la planificación física de procesos, el padre necesita un método formal para sincronizarse con sus hijos. La syscall `wait()` suspende la ejecución del proceso llamador (padre) de forma bloqueante hasta que alguno de sus procesos hijos termine o cambie de estado.

```c
#include <sys/wait.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        printf("[Hijo] Ejecutando tarea pesada...\n");
        sleep(2); // Simula cómputo
        printf("[Hijo] Terminado.\n");
        return 7; // Código de salida de ejemplo
    } else {
        int status;
        printf("[Padre] Esperando de forma bloqueante a que mi hijo termine...\n");
        pid_t hijo_terminado = wait(&status); // Syscall bloqueante
        
        if (WIFEXITED(status)) {
            printf("[Padre] El hijo con PID %d ha terminado de forma segura con código: %d\n", 
                   hijo_terminado, WEXITSTATUS(status)); //
        }
    }
    return 0;
}

```

*Estado Zombie:* Si un proceso hijo termina y su padre no invoca `wait()` para recolectar su código de terminación, el kernel retiene la información mínima del hijo en la Process Table para que el padre pueda consultarla en el futuro. En este punto, el proceso se encuentra en un estado "zombie" (muerto pero retenido en las tablas).

---

#### C. La Syscall `execve()`: Mutación del Proceso

A menudo, no deseamos que el proceso hijo continúe ejecutando una copia exacta del mismo código del padre; queremos que ejecute un programa binario completamente diferente (como el shell Bash ejecutando herramientas del sistema como `ls` o `echo`).

La syscall `execve()` carga un nuevo archivo ejecutable en memoria, **reemplazando y pisando la totalidad del espacio de memoria actual** (segmentos de Código, Datos, Heap e Hilos de Pila de ejecución del llamador) con un entorno limpio para el nuevo programa.

- **Punto Crítico:** `execve()` **no crea un nuevo proceso ni altera el PID**. Es el mismo proceso con la misma ranura en la Process Table, pero su mente y memoria han mutado de manera irreversible por un ejecutable completamente nuevo.
- Si `execve()` tiene éxito, **nunca retorna** al código del llamador, dado que ese código ha dejado de existir en la memoria física del proceso.

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    char *argv;
    argv = "echo";       // Convención: el nombre del binario
    argv = "Hola Mundo";
    argv = NULL;         // El arreglo debe terminar en NULL

    printf("PID actual antes del exec: %d\n", getpid());
    printf("Pisando proceso actual con /bin/echo...\n");

    // Ejecuta el comando y pisa la memoria actual
    execve("/bin/echo", argv, NULL);

    // Esta sección solo se alcanza si execve falla
    perror("Fallo en execve");
    return 1;
}

```

---

#### D. La Syscall `dup2()`: Clonando File Descriptors

Para redireccionar flujos de entrada/salida (por ejemplo, direccionar stdout a un archivo), Unix proporciona la syscall `dup2()`. Esta llamada duplica un descriptor de archivo de origen sobre un descriptor de destino, haciendo que ambos descriptores apunten exactamente a la misma posición lógica de archivo en el kernel.

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main() {
    // Abrimos un archivo con permisos de escritura, creándolo si no existe
    int fd = open("bitacora.log", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    
    // Clonamos el FD sobre el descriptor estándar 1 (stdout)
    // A partir de esta línea, todo write al descriptor 1 (como printf) irá al archivo
    dup2(fd, STDOUT_FILENO); 
    
    // Cerramos el FD original dado que STDOUT_FILENO ya apunta a bitacora.log
    close(fd);

    printf("Este mensaje no saldrá en pantalla, se registrará en bitacora.log\n");
    return 0;
}

```

##### Comportamiento de la Redirección paso a paso:

1. Se abre `bitacora.log` con `open()`, obteniendo el descriptor `3`.
2. `dup2(3, 1)` hace que la entrada `1` de la tabla de file descriptors del proceso apunte a la misma estructura abierta que apunta la entrada `3`.
3. `close(3)` cierra el descriptor original sobrante. La entrada `1` permanece apuntando con firmeza al archivo `bitacora.log`.

---

#### E. La Syscall `pipe()`: Comunicación Unidireccional en el Kernel

La llamada al sistema `pipe()` es la primitiva de comunicación interprocesos fundamental de Unix. Al ser invocada, el kernel genera un pequeño **búfer intermedio en memoria reservada (Kernel Space)** que expone formalmente a los procesos a través de un **par de file descriptors acoplados**:

- `pipefd`: Extremo de **Lectura** (los datos ingresados se extraen de aquí).
- `pipefd`: Extremo de **Escritura** (los datos se empujan aquí).

Un pipe es estrictamente **unidireccional**. Los datos viajan exclusivamente de `pipefd` a `pipefd`. Para simular comunicación bidireccional, es obligatorio instanciar dos pipes independientes en sentidos opuestos.

```
                     ┌───────────────────────────────┐
                     │          KERNEL SPACE         │
                     │  ┌─────────────────────────┐  │
     Escritura ─────┼─►│ Búfer Interno del Pipe  ├─┼─────► Lectura
     (pipefd)     │  └─────────────────────────┘  │      (pipefd)
                     └───────────────────────────────┘

```

##### Sincronización en Pipes y el Problema de las Flechas Redundantes

Cuando un proceso crea un pipe mediante `pipe()` y luego invoca `fork()`, el hijo hereda copias exactas de los descriptores de lectura y escritura. Como consecuencia, ambos procesos quedan interconectados al mismo búfer físico del kernel.

```
┌─────────────────────────────────┐                 ┌─────────────────────────────────┐
│         PROCESO PADRE           │                 │          PROCESO HIJO           │
│  (pipefd)       (pipefd)  │                 │  (pipefd)       (pipefd)  │
└───────┬─────────────────┬───────┘                 └───────┬─────────────────┬───────┘
        │                 │                                 │                 │
        │                 ▼                                 │                 ▼
        │        ┌───────────────────────────────────────────────────┐        │
        │        │                   KERNEL SPACE                    │        │
        │        │   ┌───────────────────────────────────────────┐   │        │
        └───────┼─► │             BÚFER DEL PIPE                │ ◄─┼────────┘
                 │   └───────────────────────────────────────────┘   │
                 └───────────────────────────────────────────────────┘

```

Esta duplicación crea **flechas o descriptores redundantes** que atentan contra la correcta finalización de la comunicación. 

En la semántica de Unix, una operación de lectura `read()` sobre un pipe solo retornará un indicador de fin de archivo (**EOF / retorna 0**) una vez que **todos** los descriptores de escritura de la tubería que existen en el sistema (tanto en el padre como en el hijo) hayan sido cerrados. Si un proceso intenta leer datos en bucle de un pipe y deja su propia flecha de escritura preabierta o el proceso emisor olvida cerrar la suya, la syscall `read()` se bloqueará infinitamente esperando datos que nunca llegarán, provocando un congelamiento de ejecución (*deadlock*).

Por esta razón, la regla de oro tras realizar un `fork()` en un flujo de tuberías es **cerrar de inmediato en cada proceso los extremos del pipe que no se vayan a utilizar**:

```c
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    int pipefd;
    char buf;

    if (pipe(pipefd) < 0) { // Creación del canal en el kernel
        perror("Error de pipe");
        return 1;
    }

    pid_t pid = fork(); // Bifurcación

    if (pid == 0) {
        // --- PROCESO HIJO (Escritor) ---
        // El hijo solo va a escribir, cerramos el descriptor de lectura redundante
        close(pipefd); 

        char mensaje[] = "Transmisión desde el subproceso hijo";
        write(pipefd, mensaje, strlen(mensaje) + 1); // Escribe en el búfer
        
        // Cerramos el extremo de escritura para indicar EOF al receptor
        close(pipefd); 
        return 0;
    } 
    else {
        // --- PROCESO PADRE (Lector) ---
        // El padre solo va a leer, cerramos el descriptor de escritura redundante
        // Si no cerramos pipefd aquí, la llamada read() de abajo nunca recibirá EOF
        close(pipefd); 

        // Espera bloqueante de flujo hasta que el hijo escriba y cierre el pipe
        int bytes_leidos = read(pipefd, buf, sizeof(buf)); 
        printf("[Padre] Mensaje recibido del canal virtual: \"%s\" (%d bytes)\n", buf, bytes_leidos);

        close(pipefd); // Cerramos el receptor de lectura
        wait(NULL);       // Sincronización del hijo
    }
    return 0;
}

```