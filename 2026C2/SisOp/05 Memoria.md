# Arquitectura de la Gestión de Memoria: Virtualización, Paginación Multinivel, TLB, Asignación Dinámica y Concurrencia

## 1. Fundamentos y Evolución de la Virtualización de Memoria

### 1.1 El Espacio de Direcciones (Address Space) y la Necesidad de Aislamiento

#### Explicación Conceptual
En los inicios de la computación (*Early Days*), la totalidad de la memoria RAM física de la máquina era compartida de forma continua entre una pequeña rutina del sistema operativo (ubicada desde la dirección 0) y un único programa en ejecución. Bajo este esquema monoprogramado, el programa tenía acceso directo irrestricto a todas las posiciones de la memoria física. 

Con el advenimiento de la multiprogramación y los sistemas de tiempo compartido (*time-sharing*), surgió la necesidad imperiosa de ejecutar múltiples procesos simultáneamente en la misma memoria RAM. Para lograr esto sin comprometer la seguridad ni la estabilidad del sistema, el sistema operativo implementa la **virtualización de memoria**.

La virtualización de memoria crea la abstracción del **espacio de direcciones** (*Address Space* o VAS, por sus siglas en inglés *Virtual Address Space*). Un espacio de direcciones es la visión contigua e independiente de la memoria que el sistema operativo le presenta a cada proceso. Dentro de su espacio de direcciones, un proceso asume que posee la totalidad del mapa de memoria de la máquina (por ejemplo, 4 GB en una arquitectura de 32 bits), conteniendo sus segmentos tradicionales:
*   **Código (Text):** Las instrucciones ejecutables del programa.
*   **Datos globales/estáticos (Data/BSS):** Variables globales e inicializadas o no.
*   **Heap:** Memoria dinámica administrada en tiempo de ejecución que crece hacia direcciones altas.
*   **Pila (Stack):** Memoria secuencial para marcos de llamadas a funciones y variables locales, que crece hacia direcciones bajas.

Esta ilusión garantiza el **aislamiento y la protección**: un proceso es incapaz de leer, escribir o modificar la memoria perteneciente a otro proceso o al kernel, ya que carece de mecanismos para direccionar físicamente la memoria ajena.

```
       VISIÓN DEL PROCESO                     MEMORIA FÍSICA REAL
     (Espacio de Direcciones)                      (PAS)
     ┌──────────────────────┐               ┌──────────────────────┐
0x00 │       Código         │               │  Sistema Operativo   │
     ├──────────────────────┤               ├──────────────────────┤
     │    Datos Globais     │               │  Proceso A (Código)  │
     ├──────────────────────┤               ├──────────────────────┤
     │     Heap (grow ↓)    │               │  Proceso B (Stack)   │
     │                      │   Traducción  ├──────────────────────┤
     │     Stack (grow ↑)   │  ───────────► │  Proceso A (Heap)    │
     ├──────────────────────┤     (MMU)     ├──────────────────────┤
0xFF │   Kernel Space       │               │      Libre / I/O     │
     └──────────────────────┘               └──────────────────────┘
```

#### Ejemplo mediante una Metáfora
Imagine un edificio corporativo de oficinas. Si no existiera aislamiento, todos los empleados de diferentes empresas competirían por el mismo piso sin paredes ni puertas, pudiendo revisar los escritorios ajenos o borrar archivos de otros. 

La virtualización de memoria actúa como un sistema de arquitectura mágica donde a cada empresa se le entrega una llave para entrar a su propia oficina. Todas las oficinas tienen la misma distribución interna de salas (código, datos, heap, stack) y el mismo número de puerta "101", pero la puerta "101" de la Empresa A conduce físicamente a una habitación totalmente distinta de la puerta "101" de la Empresa B. Ninguna empresa sabe ni puede ver dónde se encuentra físicamente ubicada la otra.

#### Ejemplo Práctico/Técnico
El siguiente programa en C demuestra la naturaleza virtual de las direcciones de memoria. Si ejecutamos dos instancias simultáneas de este código en la misma máquina, ambas imprimirán la misma dirección virtual de memoria para sus respectivas variables locales, pero los valores almacenados en ellas serán completamente independientes y aislados:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int global_var = 42; // Ubicada en el segmento Data

int main(void) {
    int local_var = 100; // Ubicada en el Stack
    void *heap_var = malloc(1024); // Ubicada en el Heap

    printf("PID: %d
", getpid());
    printf("Dirección virtual de Código (main):  %p
", (void*)main);
    printf("Dirección virtual de Var Global:     %p
", (void*)&global_var);
    printf("Dirección virtual de Var Heap:       %p
", heap_var);
    printf("Dirección virtual de Var Stack (local): %p
", (void*)&local_var);

    // Mantenemos el proceso vivo para inspeccionar su mapa de memoria
    sleep(30);

    free(heap_var);
    return 0;
}
```

Al inspeccionar el archivo pseudo-filesystem `/proc/<PID>/maps` en Linux para el proceso anterior, observamos cómo el kernel mapea estos rangos de direcciones virtuales a regiones protegidas:

```text
00400000-00401000 r-xp 00000000 08:01 123456  /home/user/app (Código)
00600000-00601000 rw-p 00001000 08:01 123456  /home/user/app (Datos)
01000000-01021000 rw-p 00000000 00:00 0       [heap]
7ffde000-7ffdf000 rw-p 00000000 00:00 0       [stack]
```

---

### 1.2 Mapeo Teórico de Direcciones

#### Explicación Conceptual
Formalmente, el proceso de traducción de direcciones (*Address Translation*) es una transformación matemática entre un conjunto de direcciones virtuales de $N$ elementos ($VAS$) y un conjunto de direcciones físicas de $M$ elementos ($PAS$, *Physical Address Space*).

La función de mapeo $MAP$ se define como:

$$MAP: VAS \implies PAS \cup \{\emptyset\}$$

$$MAP(A) = egin{cases} 
A' & 	ext{si los datos en la dirección virtual } A 	ext{ están presentes en la dirección física } A' \in PAS \
\emptyset & 	ext{si los datos en la dirección virtual } A 	ext{ no están presentes en la memoria física}
\end{cases}$$

Cuando $MAP(A) = \emptyset$, el intento de acceder a la dirección virtual $A$ desencadena un evento de excepción en el hardware. Si la dirección es no válida por permisos o fuera de rango, el procesador genera una falla de protección de memoria (como un *Segmentation Fault*). Si la dirección es válida pero no está cargada físicamente en la RAM (por ejemplo, en sistemas con memoria virtual respaldada por swap en disco), se genera un *Page Fault* para que el sistema operativo cargue el bloque solicitado desde el disco.

---

### 1.3 Primera Generación: Registros Base y Límite (Base and Bound)

#### Explicación Conceptual
Introducida a finales de la década de 1950 en las primeras máquinas con tiempo compartido, la técnica de **Base and Bound** (o relocalización dinámica por hardware) representa la forma más elemental de traducción de direcciones.

Para soportar esta técnica, la Unidad de Gestión de Memoria (MMU) en la CPU incluye dos registros de control dedicados por hardware:
1.  **Registro Base (*Base Register*):** Almacena la dirección física de memoria donde comienza el espacio de direcciones del proceso actual.
2.  **Registro Límite o Límite/Segmento (*Bound / Limit Register*):** Almacena el tamaño máximo permitido del espacio de direcciones virtual del proceso.

Cada dirección generada por el procesador dentro de un programa es tratada explícitamente como un desplazamiento (*offset*) relativo a 0. La traducción física y el chequeo de seguridad se realizan en hardware para cada instrucción de acceso a memoria mediante las siguientes operaciones lógicas simultáneas:

$$	ext{Dirección Física} = 	ext{Dirección Virtual} + 	ext{Registro Base}$$

$$	ext{Si } 	ext{Dirección Virtual} \ge 	ext{Registro Límite} \implies 	ext{Lanzar Excepción de Hardware (Fault)}$$

```
                   DIRECCIÓN VIRTUAL (Offset)
                               │
            ┌──────────────────┴──────────────────┐
            │                                     │
            ▼                                     ▼
     ┌─────────────┐                       ┌─────────────┐
     │  Sumador    │                       │ Comparador  │
     │     (+)     │                       │     (>)     │
     └──────┬──────┘                       └──────┬──────┘
            │                                     │
    + Base  │                             Límite  │
            ▼                                     ▼
   DIRECCIÓN FÍSICA                       ¿Mayor o Igual?
            │                                  ├─ SÍ ──► Excepción (Fault)
            ▼                                  └─ NO ──► Acceso Válido
    [ MEMORIA RAM ]
```

#### Ejemplo mediante una Metáfora
Considere una regla graduada de 30 cm que representa el espacio virtual de un proceso (de 0 a 30 cm). Si queremos colocar este trabajo sobre una mesa de madera gigante (memoria física), el sistema operativo decide apoyar el inicio de la regla (el 0 cm) en la marca de 150 cm de la mesa (Registro Base = 150 cm). Si el proceso quiere leer la marca de "5 cm" en su regla, el hardware calcula $150 + 5 = 155	ext{ cm}$ en la mesa. Si el proceso intenta leer la marca "35 cm" de su regla, el comparador ve que $35 > 30$ (Registro Límite = 30 cm) y detiene el acceso con un error antes de medir fuera de la regla.

#### Ejemplo Práctico/Técnico
Caso de estudio histórico: la arquitectura del microprocesador **Intel 8086**. El Intel 8086 poseía un bus de direcciones físico de 20 bits (capaz de direccionar hasta $2^{20} = 1	ext{ MB}$ de RAM), pero sus registros de propósito general eran de solo 16 bits (capaces de direccionar $2^{16} = 64	ext{ KB}$).

Para resolver esta limitación, Intel implementó un esquema de relocalización dinámica basado en registros de segmento de 16 bits (`CS` para Código, `DS` para Datos, `SS` para Pila, `ES` para Extra). El hardware de la Bus Interface Unit (BIU) del 8086 combinaba el registro de segmento (Base) y el registro de índice/puntero (Offset) desplazando el registro de segmento 4 bits a la izquierda (multiplicando por 16) y sumando el offset:

$$	ext{Dirección Física 8086} = (	ext{Registro de Segmento} \ll 4) + 	ext{Offset}$$

Si `CS` = `0x2000` y el Instruction Pointer `IP` = `0x0150`:

$$	ext{Dirección Física} = (0x2000 \ll 4) + 0x0150 = 0x20000 + 0x0150 = 0x20150$$

**Desventaja de Base and Bound:** Toda la memoria asignada a un proceso debe ser físicamente **contigua**. Si un proceso utiliza 10 KB para código y 10 KB para la pila, pero deja 1 MB libre en el medio entre el heap y el stack, ese espacio no utilizado se desperdicia en la memoria RAM física, generando **fragmentación interna** masiva y dificultando empaquetar múltiples procesos en la memoria RAM.

---

### 1.4 Segunda Generación: Memoria Segmentada (Segmentation)

#### Explicación Conceptual
Para resolver la rigidez de exigir memoria física contigua, surge la **segmentación**. En lugar de asociar un proceso a un único par de registros Base y Límite, el espacio de direcciones se divide lógicamente en varios **segmentos independientes de tamaño variable** que corresponden a los bloques naturales del programa (Código, Datos, Heap, Stack).

Cada segmento puede ubicarse de forma totalmente independiente y no contigua en la memoria física RAM. Para soportar esto, la MMU mantiene una **Tabla de Segmentos** (*Segment Table*) por proceso, cuyos renglones contienen:
*   Dirección física base del segmento.
*   Límite / Tamaño del segmento.
*   Bits de permisos de acceso (Lectura, Escritura, Ejecución).

Bajo segmentación, una dirección virtual se descompone en dos campos explícitos de bits:

$$	ext{Dirección Virtual} = [	ext{Número de Segmento (Index)} \mid 	ext{Desplazamiento (Offset)}]$$

```
 DIRECCIÓN VIRTUAL:  [ Segmento (2 bits) | Offset (12 bits) ]
                             │                      │
                             ▼                      │
                   ┌──────────────────┐             │
                   │ Tabla Segmentos  │             │
                   ├────────┬─────────┤             │
         Index 00  │ Base 0 │ Bound 0 │             │
         Index 01  │ Base 1 │ Bound 1 │             │
         Index 10  │ Base 2 │ Bound 2 │             │
                   └───┬────┴────┬────┘             │
                       │         │                  │
                       ▼         └────────┐         ▼
                  (+) Sumador             │    (>) Comparador
                       │                  │         │
                       ▼                  ▼         ▼
                Dirección Física       ¿Offset >= Bound? ──► Fault
```

#### El Origen del Término "Segmentation Fault"
Cuando un programa intenta acceder a un offset que supera el tamaño asignado al registro *Bound* de un segmento particular (por ejemplo, desbordar un buffer en el segmento de Datos) o intenta escribir en un segmento marcado como solo lectura (como el segmento de Código), la MMU de hardware detecta la violación y notifica al SO mediante una excepción física. Este evento histórico es la causa directa del error **Segmentation Fault** (`SIGSEGV` en sistemas Unix).

#### Ejemplo Práctico/Técnico
Analicemos la traducción de direcciones en un sistema con direcciones virtuales de 16 bits, donde los 2 bits superiores representan el segmento ($2^2 = 4$ segmentos posibles) y los 14 bits inferiores representan el offset ($2^{14} = 16	ext{ KB}$ de tamaño máximo por segmento):

```
Bits [15-14]: Número de Segmento (00 = Código, 01 = Datos, 10 = Heap, 11 = Stack)
Bits [13-0]:  Offset dentro del segmento
```

**Tabla de Segmentos en el Kernel:**

| Segmento | Índice Binario | Base Física | Límite (Bound) | Permisos |
| :--- | :--- | :--- | :--- | :--- |
| **Código** | `00` | `0x32000` | `0x2000` (8 KB) | Read / Execute |
| **Datos** | `01` | `0x40000` | `0x1000` (4 KB) | Read / Write |
| **Heap** | `10` | `0x48000` | `0x3000` (12 KB)| Read / Write |
| **Stack** | `11` | `0x58000` | `0x2000` (8 KB) | Read / Write |

**Caso 1: Traducción Válida**
Se solicita acceder a la dirección virtual `0x4200` (`0010 0010 0000 0000` en binario).
1.  Extraer Segmento: Bits `[15-14]` = `01` (Segmento de Datos).
2.  Extraer Offset: Bits `[13-0]` = `0x0200` (512 bytes en decimal).
3.  Chequeo de Límite: Offset `0x0200` < Límite `0x1000` (4096 bytes). **Válido.**
4.  Cálculo de Dirección Física: $	ext{Base} + 	ext{Offset} = 0x40000 + 0x0200 = 0x40200$.

**Caso 2: Violación de Segmentación (Segmentation Fault)**
Se solicita acceder a la dirección virtual `0x5500` (`0101 0101 0000 0000` en binario).
1.  Extraer Segmento: Bits `[15-14]` = `01` (Segmento de Datos).
2.  Extraer Offset: Bits `[13-0]` = `0x1500` (5376 bytes en decimal).
3.  Chequeo de Límite: Offset `0x1500` $\ge$ Límite `0x1000` (4096 bytes). **Violación de Límite.**
4.  El hardware detiene la traducción de inmediato y lanza una excepción `#GP` (*General Protection Fault*), haciendo que el kernel envíe `SIGSEGV` al proceso.

**Desventaja Principal de la Segmentación: Fragmentación Externa**
Debido a que los segmentos son de tamaño variable y se asignan/liberan dinámicamente, la memoria RAM física se llena de pequeños huecos no contiguos. Aunque la suma total de espacio libre en la RAM sea de 100 MB, si está dividida en 1000 huecos dispersos de 100 KB cada uno, el sistema será incapaz de asignar un nuevo segmento contiguo de 2 MB, requiriendo algoritmos costosos de compactación de memoria.

---

## 2. Memoria Paginada y Traducción de Direcciones

### 2.1 El Modelo de Paginación Fija: Páginas Virtuales y Marcos Físicos (Frames)

#### Explicación Conceptual
Para eliminar por completo el problema de la fragmentación externa, los sistemas operativos modernos utilizan **paginación** (*Paging*). En lugar de dividir la memoria en bloques de tamaño variable, la paginación divide tanto el espacio de direcciones virtual como la memoria física RAM en bloques de **tamaño fijo e invariable**, que son siempre potencias de 2 (típicamente de **4 KB = 4096 bytes**).

*   **Página Virtual (*Virtual Page* / VP):** Bloque de tamaño fijo dentro del espacio de direcciones virtual del proceso.
*   **Marco de Página Físico (*Physical Page Frame* / Frame / PFN):** Bloque de tamaño fijo idéntico dentro de la memoria RAM física.

Bajo este esquema, cualquier página virtual de un proceso puede ser ubicada en **cualquier marco físico disponible** de la memoria RAM, sin importar si los marcos físicos están contiguos o dispersos por todo el chip de memoria.

```
 ESPACIO VIRTUAL (Proceso)                        MEMORIA RAM FÍSICA
┌──────────────────────┐                        ┌──────────────────────┐
│  Página Virtual 0    │ ──────────────────────►│  Frame Físico 105    │
├──────────────────────┤                        ├──────────────────────┤
│  Página Virtual 1    │ ────────┐              │  Frame Físico 106    │
├──────────────────────┤         │              ├──────────────────────┤
│  Página Virtual 2    │ ──┐     └─────────────►│  Frame Físico 12     │
└──────────────────────┘   │                    ├──────────────────────┤
                           └───────────────────►│  Frame Físico 89     │
                                                └──────────────────────┘
```

---

### 2.2 Anatomía de una Dirección Virtual (VPN y Offset)

#### Explicación Conceptual
Toda dirección virtual bajo un sistema paginado se compone de dos partes bien definidas:

$$	ext{Dirección Virtual} = [	ext{VPN (Virtual Page Number)} \mid 	ext{Offset}]$$

1.  **VPN (Virtual Page Number):** Los bits más significativos (superiores) de la dirección. Se utiliza como el **índice numérico directo** dentro de la Tabla de Páginas para encontrar qué marco de página físico alberga esa memoria.
2.  **Offset (Desplazamiento):** Los bits menos significativos (inferiores) de la dirección. Indica el byte exacto dentro de la página solicitada (desde el byte 0 hasta el byte $4095$ para páginas de 4 KB).

**Cálculo del tamaño de bits:**
Para una página de tamaño $S = 4	ext{ KB} = 4096	ext{ bytes} = 2^{12}	ext{ bytes}$, se requieren exactamente **12 bits para el Offset**.
En una arquitectura de 32 bits ($2^{32} = 4	ext{ GB}$ de VAS):
$$	ext{Bits de Offset} = 12$$
$$	ext{Bits de VPN} = 32 - 12 = 20	ext{ bits}$$
$$	ext{Cantidad de Páginas Virtuales posibles} = 2^{20} = 1.048.576	ext{ páginas}$$

Dado que el tamaño de una página física y de una página virtual es idéntico, **el Offset se traslada exactamente igual y sin cambios** desde la dirección virtual a la dirección física. La MMU solo traduce el VPN al PFN (*Physical Frame Number*):

$$	ext{Dirección Física} = [	ext{PFN} \mid 	ext{Offset}]$$

```
 DIRECCIÓN VIRTUAL (32 bits):
 ┌──────────────────────────────────────────────┬───────────────────────────┐
 │          VPN (Bits 31 a 12 = 20 bits)        │ Offset (Bits 11 a 0 = 12) │
 └──────────────────────┬───────────────────────┴─────────────┬─────────────┘
                        │ (Búsqueda en Tabla)                 │
                        ▼                                     │
 ┌──────────────────────────────────────────────┐             │ (Copia directa)
 │            PFN (Physical Frame Number)       │             │
 └──────────────────────┬───────────────────────┘             │
                        ▼                                     ▼
 DIRECCIÓN FÍSICA:      [          PFN          │          Offset           ]
```

---

### 2.3 Estructura de la Tabla de Páginas (Page Table) y Entradas (PTE)

#### Explicación Conceptual
La **Tabla de Páginas** (*Page Table*) es la estructura de datos fundamental que mantiene la cartografía entre las páginas virtuales de un proceso y los marcos de memoria física RAM. Es conceptualmente un arreglo ordenado (*array*) en memoria, donde el índice del elemento es el número de página virtual (VPN).

Cada elemento del arreglo se denomina **Entrada de la Tabla de Páginas** (*Page Table Entry* / PTE). En una arquitectura estándar de 32 bits, cada PTE ocupa **4 bytes (32 bits)** de memoria física.

#### Anatomía del Registro de una PTE de 32 bits (x86)

```
 31                                 12 11 10 9  8  7  6  5  4  3  2  1  0
┌─────────────────────────────────────┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
│   Physical Page Frame Number (PFN)  │? │? │? │G │PS│D │A │CD│WT│U │W │P │
└─────────────────────────────────────┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
```

**Significado de los Bits de Control (Flags) en la PTE:**
*   **P (Present - Bit 0):** `1` si la página está presente en memoria RAM física; `0` si la página no está asignada o fue enviada al disco (*swap*). Si un proceso intenta acceder a una PTE con $P=0$, el hardware genera un **Page Fault**.
*   **W o R/W (Read/Write - Bit 1):** `1` si la página permite lectura y escritura; `0` si la página es de solo lectura (por ejemplo, el código ejecutable).
*   **U o U/S (User/Supervisor - Bit 2):** `1` si la página puede ser accedida por código ejecutado en Ring 3 (espacio de usuario); `0` si la página es de nivel Kernel (Ring 0).
*   **WT (Write-Through - Bit 3):** Política de caché del procesador para la página.
*   **CD (Cache Disable - Bit 4):** `1` inhabilita el almacenamiento en caché de la CPU para esta página (crucial para regiones de memoria I/O mapeadas a dispositivos).
*   **A (Accessed - Bit 5):** Marcado automáticamente en `1` por el hardware cada vez que se lee o escribe en la página. Utilizado por los algoritmos de reemplazo de páginas del SO.
*   **D (Dirty - Bit 6):** Marcado automáticamente en `1` por el hardware cuando se realiza una operación de escritura en la página. Indica si la página fue modificada respecto a lo guardado en disco.
*   **PS (Page Size - Bit 7):** Si está en `1`, indica que la entrada apunta directamente a una página grande (ej. 4 MB) en lugar de una tabla de segundo nivel.
*   **G (Global - Bit 8):** Si está en `1`, evita que la traducción sea eliminada del TLB durante un *flush* causado por un cambio de contexto (usado para páginas del kernel compartidas).
*   **PFN (Bits 31 a 12 - 20 bits):** Los 20 bits superiores que almacenan la dirección física base del Marco de Página en la RAM.

---

### 2.4 Paginación Multinivel (Jerárquica)

#### El Problema de la Tabla de Páginas Plana (Simple Level)
Supongamos un sistema de 32 bits con páginas de 4 KB ($2^{12}$ bytes). Como vimos, existen $2^{20} = 1.048.576$ páginas virtuales por proceso.
Si utilizamos una tabla de páginas simple (un arreglo plano):
$$	ext{Tamaño de la Tabla por Proceso} = 1.048.576 	ext{ entradas} 	imes 4 	ext{ bytes/PTE} = 4.194.304 	ext{ bytes} = \mathbf{4 	ext{ MB}}$$

Dado que **cada proceso requiere su propia tabla de páginas**, si ejecutamos 1000 procesos simultáneos en el sistema, ¡necesitaríamos **4 GB de memoria RAM física solo para almacenar las tablas de páginas**, consumiendo toda la RAM del sistema antes de ejecutar cualquier código!

Además, la mayor parte del espacio de direcciones de un proceso está totalmente **vacío** (*sparse*), ya que hay un enorme hueco no utilizado entre el crecimiento del Heap y de la Pila. En una tabla plana, nos vemos obligados a reservar memoria física para las 1.048.576 PTEs, contengan direcciones válidas o datos nulos.

#### La Solución: Paginación Multinivel (Multi-Level Paging)
La **paginación multinivel** resuelve este problema convirtiendo la tabla de páginas plana en una **estructura de árbol jerárquico**. 

En lugar de tener una sola tabla gigante contigua, dividimos la tabla de páginas en páginas pequeñas de 4 KB. Luego, creamos una tabla de primer nivel (llamada **Directorio de Páginas** o *Page Directory*) que contiene punteros hacia las tablas de segundo nivel (*Page Tables*).

**La ventaja fundamental de la paginación multinivel:** Si un rango entero del espacio de direcciones virtual no está siendo utilizado por el programa (por ejemplo, el espacio vacío entre el Heap y la Pila), la entrada correspondiente en el Directorio de Páginas se marca con $P=0$ (No Presente). Con esto, **no hace falta instanciar ni reservar en RAM las tablas de segundo nivel para esas regiones vacías**.

```
                DIRECCIÓN VIRTUAL (32 bits)
  ┌──────────────────────┬──────────────────────┬───────────────────────────┐
  │ Directory (10 bits)  │   Table (10 bits)    │   Offset (12 bits)        │
  └──────────┬───────────┴──────────┬───────────┴─────────────┬─────────────┘
             │                      │                         │
             ▼                      │                         │
   ┌──────────────────┐             │                         │
   │ Page Directory   │             │                         │
   │ (Nivel 1: 4 KB)  │             │                         │
   ├──────────────────┤             │                         │
   │ PDE 0            │             │                         │
   │ PDE 1 ─────────┐ │             │                         │
   │ PDE 2 (P=0)    │ │             │                         │
   └────────────────┼─┘             │                         │
                    │               ▼                         │
                    │     ┌──────────────────┐                │
                    └────►│    Page Table    │                │
                          │ (Nivel 2: 4 KB)  │                │
                          ├──────────────────┤                │
                          │ PTE 0            │                │
                          │ PTE 1 ─────────┐ │                │
                          └────────────────┼─┘                │
                                           │                  │
                                           ▼                  ▼
                                ┌──────────────────────────────────────────┐
                                │       Marco de Memoria RAM Físico        │
                                └──────────────────────────────────────────┘
```

---

### 2.5 Caso de Estudio Riguroso: x86 de 32 bits en 2 Niveles

#### División de Bits de la Dirección Virtual x86
En la arquitectura Intel x86 de 32 bits en modo paginado estándar (sin PAE), una dirección virtual de 32 bits se descompone exactamente en:
*   **Directory Index (Bits 31 a 22 - 10 bits):** Índice dentro del *Page Directory* ($2^{10} = 1024$ entradas posibles).
*   **Table Index (Bits 21 a 12 - 10 bits):** Índice dentro de la *Page Table* de nivel 2 ($2^{10} = 1024$ entradas posibles).
*   **Offset (Bits 11 a 0 - 12 bits):** Desplazamiento en el marco de página ($2^{12} = 4096$ bytes).

#### Tamaño Perfecto de las Tablas
Observe la elegancia matemática del diseño del hardware:
*   Cada tabla (Directory o Table) tiene $1024$ entradas.
*   Cada entrada (PDE o PTE) ocupa $4$ bytes.
*   $$	ext{Tamaño total de una tabla} = 1024 	ext{ entradas} 	imes 4 	ext{ bytes} = \mathbf{4096 	ext{ bytes}} = \mathbf{4 	ext{ KB}}$$

Por lo tanto, **cada tabla de páginas ocupa exactamente una página física de 4 KB en la RAM**, encajando perfectamente en el alocador de páginas del kernel.

#### El Registro de Control CR3
El punto de partida físico para la traducción de direcciones es el registro **CR3** (o PDBR, *Page Directory Base Register*) en la CPU x86 (equivalente al registro `SATP` en la arquitectura RISC-V).

El registro CR3 contiene la **dirección física base del Page Directory del proceso activo**. 
Dado que el Page Directory ocupa 4 KB y está siempre alineado a un límite de página física, sus 12 bits inferiores son siempre cero (`0x000`). Por ende, el registro CR3 solo necesita almacenar los 20 bits superiores de la dirección física.

Cualquier cambio de contexto (*Context Switch*) entre dos procesos requiere únicamente actualizar el registro CR3 con la dirección del Page Directory del nuevo proceso.

#### Algoritmo Paso a Paso de Traducción Realizada por la MMU

```c
// Pseudocódigo del comportamiento implementado en el Hardware de la MMU
uint32_t translate_virtual_to_physical(uint32_t virtual_address) {
    uint32_t dir_index   = (virtual_address >> 22) & 0x3FF; // Bits 31-22
    uint32_t table_index = (virtual_address >> 12) & 0x3FF; // Bits 21-12
    uint32_t offset      = virtual_address & 0xFFF;        // Bits 11-0

    // 1. Obtener dirección base del Page Directory desde el registro CR3
    uint32_t page_directory_phys_base = CR3 & 0xFFFFF000;

    // 2. Leer la Entrada del Directorio de Páginas (PDE)
    uint32_t *page_directory = (uint32_t*) page_directory_phys_base;
    uint32_t pde = page_directory[dir_index];

    // Verificar si la tabla de nivel 2 está presente
    if (!(pde & PTE_PRESENT)) {
        raise_page_fault_exception(virtual_address, FAULT_NOT_PRESENT);
        return 0;
    }

    // 3. Obtener dirección base de la Page Table desde el PDE
    uint32_t page_table_phys_base = pde & 0xFFFFF000;

    // 4. Leer la Entrada de la Tabla de Páginas (PTE)
    uint32_t *page_table = (uint32_t*) page_table_phys_base;
    uint32_t pte = page_table[table_index];

    // Verificar si la página física final está presente y validar permisos
    if (!(pte & PTE_PRESENT)) {
        raise_page_fault_exception(virtual_address, FAULT_NOT_PRESENT);
        return 0;
    }
    
    if (user_mode() && !(pte & PTE_USER)) {
        raise_page_fault_exception(virtual_address, FAULT_PROTECTION_VIOLATION);
        return 0;
    }

    // 5. Calcular la Dirección Física Final combinando PFN y Offset
    uint32_t pfn = pte & 0xFFFFF000;
    uint32_t physical_address = pfn | offset;

    return physical_address;
}
```

---

### 2.6 Páginas Grandes (Large Pages / Huge Pages)

#### Explicación Conceptual
En el esquema x86 de 32 bits, si el bit **PS (Page Size)** en una PDE del Directorio de Páginas se establece en `1`, la MMU modifica su comportamiento y salta completamente el segundo nivel de traducción.

En este modo de **Paginación Extendida**, la PDE apunta directamente a un marco de página gigante de **4 MB** ($2^{22}$ bytes). La dirección virtual se divide en:
*   **Directory Index (Bits 31 a 22 - 10 bits):** Selecciona el marco de 4 MB.
*   **Offset (Bits 21 a 0 - 22 bits):** Desplazamiento directo dentro del marco de 4 MB.

#### Propósito y Aplicación
Las páginas grandes reducen la presión sobre el caché de traducción (TLB) y eliminan niveles de búsqueda en memoria. Son ampliamente utilizadas por los kernels de sistemas operativos para mapear su propio código ejecutable y por motores de bases de datos o máquinas virtuales con consumos masivos de RAM.

---

## 3. Aceleración del Hardware: Translation Lookaside Buffer (TLB)

### 3.1 El Costo del Recorrido Multinivel (Page Walk)

#### Explicación Conceptual
A pesar de la elegancia de la paginación multinivel para ahorrar memoria RAM, introduce un severo problema de rendimiento: **la penalización por acceso a memoria**.

En un sistema de 2 niveles sin aceleración, por **cada única lectura/escritura de datos** que un programa solicita en el código (por ejemplo, `mov eax, [ebx]`), el procesador debe realizar **3 accesos secuenciales a la memoria RAM física**:
1.  Leer la PDE en el Page Directory (RAM).
2.  Leer la PTE en la Page Table (RAM).
3.  Leer o escribir finalmente el byte solicitado en la memoria RAM del marco físico.

En arquitecturas de 64 bits con 4 o 5 niveles de paginación (como x86-64), ¡se requerirían 5 o 6 accesos a RAM física por cada instrucción! Dado que la RAM es cientos de veces más lenta que la CPU, la computadora se volvería extremadamente ineficiente.

---

### 3.2 Arquitectura Interna de la TLB

#### Explicación Conceptual
Para resolver este cuello de botella, la MMU incorpora un componente de hardware de memoria estática ultrarrápida integrado dentro de la CPU denominado **Translation Lookaside Buffer (TLB)** (Caché de Traducción de Direcciones).

La TLB es una **memoria caché de direcciones** (no almacena datos del programa, solo pares de traducción procesados). Cada registro en la TLB guarda la tupla:

$$	ext{TLB Entry} = \{	ext{VPN} \mid 	ext{PFN} \mid 	ext{Flags / Permisos}\}$$

```
                               DIRECCIÓN VIRTUAL
                               ┌────────┬────────┐
                               │  VPN   │ Offset │
                               └───┬────┴───┬────┘
                                   │        │
                   ┌───────────────┴──┐     │
                   ▼                  │     │
         ┌──────────────────┐         │     │
         │  TLB (MMU Cache) │         │     │
         ├──────┬──────┬────┤         │     │
         │ VPN  │ PFN  │Flg │         │     │
         ├──────┼──────┼────┤         │     │
         │ 0x05 │ 0x1A │r-x │         │     │
         └──────┴──┬───┴────┘         │     │
                   │ (Hit)            │     │
                   ▼                  │ (Miss)
            ┌──────────────┐          ▼     │
            │  Frame PFN   │   ┌──────────┐ │
            └──────┬───────┘   │Page Walk │ │
                   │           └────┬─────┘ │
                   │                │       │
                   ▼                ▼       ▼
            [ DIRECCIÓN FÍSICA = PFN | Offset ]
```

#### Organización de la TLB: Fully-Associative vs. Set-Associative
Para ser efectiva, la búsqueda dentro de la TLB debe tomar una fracción de nanosegundo. Por ello, la TLB se implementa en hardware mediante dos arquitecturas:
*   **Fully-Associative (Completamente Asociativa):** El VPN buscado se compara **en paralelo al mismo tiempo** contra todas las entradas de la tabla usando comparadores lógicos en hardware. Es extremadamente rápida pero limitada en capacidad (ej. 64 a 128 entradas).
*   **Set-Associative (Asociativa por Conjuntos):** Se utilizan ciertos bits del VPN para seleccionar un conjunto de bancos en la memoria caché y luego se busca en paralelo dentro de ese pequeño grupo.

Un chip procesador moderno posee múltiples niveles de TLB (ej. L1 i-TLB de 64 entradas para instrucciones, L1 d-TLB de 64 entradas para datos, y un L2 TLB unificado de 512 a 1024 entradas).

---

### 3.3 Secuencia de Traducción: TLB Hit y TLB Miss

#### Algoritmo de Traducción con TLB
Cuando la CPU necesita traducir una dirección virtual:
1.  La MMU extrae el **VPN** de la dirección virtual.
2.  Consulta en la **TLB**:
    *   **TLB Hit (Éxito):** El VPN está presente en la TLB y los permisos son válidos. La MMU toma inmediatamente el **PFN** cacheado, le concatena el **Offset** original y accede a la memoria física en un solo ciclo de reloj. Se evita el recorrido por las tablas en RAM.
    *   **TLB Miss (Fallo):** El VPN no está en la TLB. La MMU (o el SO mediante una interrupción, dependiendo de la arquitectura) realiza el costoso recorrido por el árbol de páginas en la RAM (*Page Walk*). Una vez hallado el PFN, **almacena la nueva traducción en la TLB** para futuros accesos y completa la instrucción.

#### Principio de Localidad
La TLB funciona con un nivel de éxito (*Hit Rate*) superior al $99\%$ en la mayoría de las aplicaciones gracias al principio de **localidad espacial y temporal**: al acceder a un byte dentro de una página de 4 KB, las subsecuentes miles de operaciones en variables contiguas o bucles reutilizarán la misma entrada de página cargada en la TLB.

---

### 3.4 Consistencia de la TLB y Gestión de Context Switch

#### El Problema del Cambio de Contexto (Context Switch)
Dado que la TLB guarda pares de `VPN -> PFN`, ¿qué sucede cuando el planificador del sistema operativo detiene la ejecución del Proceso A y conmuta al Proceso B?

Si la TLB conservara las entradas creadas por el Proceso A, cuando el Proceso B solicite acceder a su propia dirección virtual `0x1000`, la TLB registraría un **Hit erróneo** (*TLB Hit*) devolviendo el marco físico del Proceso A. Esto provocaría una gravísima falla de seguridad y corrupción de memoria.

#### Solución 1: TLB Flush (Vaciado Completo)
Cada vez que el sistema operativo ejecuta un cambio de contexto escribiendo un nuevo valor en el registro `CR3` (o `SATP`), el procesador **invalida automáticamente todas las entradas no globales de la TLB** (poniendo sus bits válidos en 0).

*   **Efecto secundario:** Tras cada *context switch*, el nuevo proceso sufre una ráfaga de *TLB Misses* (*Cold Cache*) hasta volver a "calentar" la TLB con sus traducciones frecuentes. Esta es una de las razones principales por las cuales los cambios de contexto son costosos.

#### Solución 2: Etiquetado de Proceso (ASID / PCID Tagging)
Los procesadores modernos incluyen un campo adicional en cada entrada de la TLB denominado **ASID** (*Address Space Identifier* en RISC-V/ARM) o **PCID** (*Process Context Identifier* en x86).

El hardware almacena el ID del proceso activo junto con la entrada. Al consultar la TLB, la MMU solo genera un *Hit* si el VPN coincide **y** el ASID actual coincide con el ID guardado. Esto permite conservar las traducciones de múltiples procesos en la TLB a través de cambios de contexto sin necesidad de vaciarla.

---

### 3.5 Invalidación en Multiprocesamiento (TLB Shootdown)

#### Explicación Conceptual
En un sistema multiprocesador (SMP) con múltiples núcleos de CPU, cada núcleo posee su propia unidad MMU y su propia caché TLB privada.

Si el kernel que se ejecuta en el Núcleo 1 decide desmapear una página o reducir sus permisos (por ejemplo, al liberar memoria con `free()` o marcar una página como de solo lectura para *Copy-on-Write*), debe asegurarse de que ningún otro núcleo mantenga la traducción antigua en su TLB privada.

Para lograr esto, el kernel en el Núcleo 1 emite una interrupción entre procesadores (**IPI** - *Inter-Processor Interrupt*) a todos los demás núcleos. Al recibir la IPI, los demás núcleos pausan su ejecución e invalidan forzosamente la página en sus TLBs privadas. Este mecanismo se conoce como **TLB Shutdown** o **TLB Shootdown**.

---

## 4. Gestión de Memoria Física y Asignación Dinámica

### 4.1 Asignación a Nivel de Kernel: Manejo de Memoria Física (kalloc y kfree)

#### Explicación Conceptual
El kernel del sistema operativo necesita administrar la memoria RAM física disponible para asignar páginas a las tablas de páginas, pilas del kernel o estructuras de procesos.

En sistemas académicos reales como **xv6**, la memoria RAM física libre se administra mediante una estructura de **Lista Enlazada de Páginas Libres (`freelist`)**.

#### La Técnica de la Lista Enlazada In-Situ (Embedded Allocation)
Para evitar gastar memoria extra administrando la memoria física, el kernel guarda los nodos de la lista enlazada **dentro de las mismas páginas físicas libres**. Dado que las páginas libres no contienen datos útiles en ese momento, los primeros bytes de cada página física libre se reinterpretan como una estructura que contiene un puntero `next` a la siguiente página libre.

```
 Global Kernel Variable: [ freelist ] ──► Ptr a Página Libre 1
                                                │
                                                ▼ (4 KB Physical Page)
                                         ┌──────────────┐
                                         │ struct run *next ──► Ptr a Página Libre 2
                                         ├──────────────┤
                                         │ (Unused RAM) │
                                         └──────────────┘
```

#### Código de Implementación del Alocador Físico de xv6

```c
// Estructura guardada en los primeros bytes de cada página física libre
struct run {
    struct run *next;
};

struct {
    struct spinlock lock; // Cerrojo para proteger la lista concurrente
    struct run *freelist; // Puntero al primer nodo de la lista
} kmem;

// Asigna una página física de 4 KB. Retorna la dirección física o 0 si falla.
void *kalloc(void) {
    struct run *r;

    acquire(&kmem.lock); // Sección crítica: proteger la lista compartida
    r = kmem.freelist;
    if (r) {
        kmem.freelist = r->next; // Extraer el elemento cabecera de la pila
    }
    release(&kmem.lock);

    if (r) {
        // Llenamos la página con basura (carácter 5) para detectar usos indebidos
        memset((char*)r, 5, 4096); 
    }
    return (void*)r; // Retorna la dirección física del frame
}

// Libera una página física regresándola a la lista de páginas libres
void kfree(void *pa) {
    struct run *r;

    // Validación: la dirección debe estar alineada a 4 KB y ser válida
    if (((uint64_t)pa % 4096) != 0 || (char*)pa < end || (uint64_t)pa >= PHYSTOP)
        panic("kfree invalid address");

    // Llenamos la página liberada con basura (carácter 1)
    memset(pa, 1, 4096);

    r = (struct run*)pa;

    acquire(&kmem.lock);
    r->next = kmem.freelist; // Insertar al frente de la lista (comportamiento de Pila)
    kmem.freelist = r;
    release(&kmem.lock);
}
```

---

### 4.2 Recorrido y Mapeo por Software (mappage y walk)

#### Explicación Conceptual
Cuando un proceso solicita más memoria o el kernel crea un nuevo espacio de direcciones, el sistema operativo debe modificar la tabla de páginas ejecutando por software el mismo recorrido que realiza la MMU por hardware.

La función `walk()` inspecciona los niveles de la tabla de páginas. Si durante el recorrido encuentra que una tabla de nivel intermedio no existe ($P=0$), **llama a `kalloc()` para asignar una nueva página física**, la limpia con ceros y actualiza la entrada en el nivel superior para enlazar la nueva tabla.

```c
// Algoritmo de mapeo de páginas por software
int mappages(pagetable_t pagetable, uint64_t va, uint64_t size, uint64_t pa, int perm) {
    uint64_t a, last;
    pte_t *pte;

    a = PGROUNDDOWN(va);
    last = PGROUNDDOWN(va + size - 1);

    for (;;) {
        // Obtenemos la dirección de la PTE final recorriendo el árbol
        if ((pte = walk(pagetable, a, 1)) == 0)
            return -1; // Error por falta de memoria física
        
        if (*pte & PTE_P)
            panic("mappages: remap"); // La página ya estaba mapeada

        // Inicializamos la PTE apuntando a la dirección física 'pa' con sus permisos
        *pte = PA2PTE(pa) | perm | PTE_P;

        if (a == last)
            break;
        a += 4096;
        pa += 4096;
    }
    return 0;
}
```

---

### 4.3 La Llamada al Sistema brk() y sbrk()

#### Explicación Conceptual
En los sistemas operativos de la familia Unix, el Heap de un proceso es un área contigua de memoria virtual que comienza justo después del segmento BSS y crece hacia direcciones altas.

El límite superior del Heap se denomina el **Break** (*Puntero de Ruptura*). La llamada al sistema `brk(void *addr)` establece la ubicación del break en la dirección especificada, mientras que `sbrk(intptr_t increment)` incrementa o decrementa el break por la cantidad de bytes indicada.

Cuando un proceso invoca `sbrk(4096)`:
1.  El kernel verifica que el nuevo break no colisione con el Stack.
2.  El kernel invoca a `kalloc()` para obtener páginas físicas libres.
3.  El kernel llama a `mappages()` para mapear las nuevas páginas físicas justo en la dirección virtual del break antiguo.
4.  El kernel actualiza el valor del break en la estructura del proceso (`p->sz` o `p->brk`) y retorna la dirección anterior.

```
  [ Código ] [ Datos ] [ Heap ] ──► (Break Antiguo) ───► sbrk(4096) ───► (Nuevo Break)
```

---

### 4.4 Asignación en Espacio de Usuario: Funciones malloc() y free()

#### La Necesidad de un Administrador en Espacio de Usuario
Invocar la llamada al sistema `sbrk()` o `brk()` directamente para cada solicitud de memoria pequeña (por ejemplo, pedir 16 bytes para un nodo de una lista enlazada) sería extremadamente ineficiente debido al costo de conmutar al modo Kernel mediante una interrupción de software (*syscall*). Además, las llamadas al sistema solo pueden asignar memoria en **múltiplos de páginas enteras (4 KB)**.

La biblioteca estándar de C (`glibc` / `libc`) implementa en **espacio de usuario** las funciones `malloc()` y `free()`. `malloc()` actúa como un intermediario eficiente: le solicita al kernel páginas grandes de 4 KB mediante `sbrk()` o `mmap()`, y luego administra y subdivide esas páginas en pequeños bloques dentro del espacio de usuario.

#### Estructura de Bloques en el Heap: Encabezados (Headers)
Para administrar la memoria libre dentro del Heap, `malloc()` antepone a cada bloque de memoria asignado o libre un **Encabezado (*Header*)** invisible para el programador.

```c
// Encabezado de bloque en el Heap
typedef long Align; // Alineamiento al límite de palabra de la arquitectura

union header {
    struct {
        union header *ptr; // Puntero al siguiente bloque libre en la freelist
        unsigned size;     // Tamaño del bloque medido en unidades de Header
    } s;
    Align x; // Forzar alineación de memoria física
};

typedef union header Header;
```

Cuando un usuario solicita `malloc(N)`:
1.  Calcula el número de unidades de `Header` requeridas para albergar los $N$ bytes más el tamaño del propio `Header`.
2.  Recorre la lista enlazada circular de bloques libres (*Free List* en espacio de usuario).
3.  Aplica una política de búsqueda:
    *   **First-Fit:** Elige el primer bloque libre que tenga un tamaño mayor o igual al solicitado.
    *   **Best-Fit:** Busca en toda la lista el bloque que mejor se ajuste minimizando el desperdicio.
4.  **División de Bloques (*Splitting*):** Si el bloque libre encontrado es más grande que lo solicitado, el bloque se parte en dos: la porción posterior se le entrega al usuario y la porción anterior permanece en la lista libre con su tamaño actualizado.
5.  **Retorno:** Pasa al usuario un puntero que apunta **justo después del Header**.

```
 BLOQUE COMPLETO EN EL HEAP:
 ┌──────────────────────┬─────────────────────────────────────────────────┐
 │ Header (s.size, ptr) │ Memoria Retornada al Usuario (Puntero devuelto) │
 └──────────────────────┴─────────────────────────────────────────────────┘
 ▲                      ▲
 │                      └─ Puntero retornado por malloc()
 └─ Puntero real administrado por malloc
```

#### Liberación de Memoria y Coalescencia (Coalescing)
Cuando el usuario llama a `free(void *ap)`:
1.  Obtiene la dirección del `Header` restando el tamaño de `sizeof(Header)` al puntero `ap`.
2.  Inserta el bloque de vuelta en la lista enlazada de bloques libres.
3.  **Coalescencia (*Coalescing* / Fusión):** `free()` comprueba si el bloque recién liberado es físicamente adyacente a sus bloques vecinos en la lista libre. Si son adyacentes, **los fusiona inmediatamente en un único bloque libre continuo más grande**. Esto previene la fragmentación externa dentro del Heap del usuario.

```c
// Algoritmo de Coalescencia al liberar un bloque (K&R)
void free(void *ap) {
    Header *bp, *p;

    if (ap == NULL) return;

    bp = (Header *)ap - 1; // Obtener puntero al Header original

    // Recorrer la lista libre para encontrar la posición ordenada del bloque
    for (p = freep; !(bp > p && bp < p->s.ptr); p = p->s.ptr) {
        if (p >= p->s.ptr && (bp > p || bp < p->s.ptr))
            break; // Caso borde: envolver la lista circular
    }

    // Coalescencia con el vecino superior (fusión a la derecha)
    if (bp + bp->s.size == p->s.ptr) {
        bp->s.size += p->s.ptr->s.size;
        bp->s.ptr = p->s.ptr->s.ptr;
    } else {
        bp->s.ptr = p->s.ptr;
    }

    // Coalescencia con el vecino inferior (fusión a la izquierda)
    if (p + p->s.size == bp) {
        p->s.size += bp->s.size;
        p->s.ptr = bp->s.ptr;
    } else {
        p->s.ptr = bp;
    }

    freep = p;
}
```

---

### 4.5 Mecanismos Modernos con mmap() y Mapeo de Archivos

#### Explicación Conceptual
En los sistemas operativos modernos, el uso de `brk()` ha sido desplazado ampliamente por la llamada al sistema más flexible **`mmap()`**.

`mmap()` permite mapear páginas de memoria virtual en cualquier región libre del espacio de direcciones del proceso, sin obligar a que la memoria sea contigua al Heap tradicional.

#### Firma de la API POSIX mmap

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);
```

**Parámetros Principales:**
*   `addr`: Dirección virtual sugerida (usualmente `NULL` para que el kernel elija la dirección).
*   `length`: Tamaño en bytes de la región a mapear (múltiplo de páginas de 4 KB).
*   `prot`: Banderas de protección de memoria (`PROT_READ`, `PROT_WRITE`, `PROT_EXEC`).
*   `flags`:
    *   `MAP_ANONYMOUS`: La memoria no está respaldada por ningún archivo físico; sus contenidos se inicializan automáticamente en cero por el kernel. Utilizado por `malloc()` para pedir bloques grandes (ej. mayores a 128 KB).
    *   `MAP_PRIVATE`: Las modificaciones a la memoria son privadas para el proceso y no se escriben al archivo subyacente (*Copy-on-Write*).
    *   `MAP_SHARED`: La memoria es compartida entre múltiples procesos. Las escrituras son visibles por otros procesos que mapeen el mismo archivo o región.
*   `fd`: Descriptor del archivo a mapear en memoria (o `-1` si es `MAP_ANONYMOUS`).

#### Mapeo de Archivos en Memoria (Memory-Mapped Files)
`mmap()` permite reemplazar llamadas costosas a `read()` y `write()` asignando un archivo directamente al espacio de direcciones del proceso. El contenido del archivo se carga perezosamente (*Lazy Demand Paging*) a medida que el proceso lee o escribe en los punteros de memoria, delegando la gestión de I/O al subsistema de Caché de Páginas (*Page Cache*) del kernel.

```c
// Ejemplo práctico de asignación anónima compartida para IPC con mmap
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/wait.h>
#include <unistd.h>

int main(void) {
    // Asignamos una página de memoria compartida entre proceso padre e hijo
    int *shared_counter = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
                              MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    *shared_counter = 100;

    pid_t pid = fork();

    if (pid == 0) { // Proceso Hijo
        *shared_counter += 50;
        printf("Hijo modificó el contador compartido a: %d
", *shared_counter);
        exit(0);
    } else { // Proceso Padre
        wait(NULL); // Esperar que el hijo termine
        printf("Padre lee el contador compartido: %d
", *shared_counter); // Imprime 150
        munmap(shared_counter, 4096);
    }

    return 0;
}
```

---

## 5. Sincronización y Concurrencia en el Acceso a Memoria

### 5.1 Condiciones de Carrera (Race Conditions) en Estructuras Compartidas

#### Explicación Conceptual
Cuando múltiples hilos de ejecución (*threads*) o procesos con memoria compartida intentan leer y escribir simultáneamente sobre la misma estructura de datos (como la lista libre de páginas `freelist` del kernel o el Heap administrado por `malloc()`), el resultado final depende impredeciblemente del orden exacto de intercalado de las instrucciones en la CPU. Este fenómeno no determinista se denomina **Condición de Carrera (*Race Condition*)**.

#### Análisis de un Entrelazado Violatorio en C
Considere la operación aparentemente simple de incrementar una variable de saldo global: `saldo += 10`.

A nivel de lenguaje ensamblador, la instrucción en C se descompone en **tres instrucciones de máquina separadas**:
1.  `MOV EAX, [saldo]` (Leer valor desde la RAM a un registro)
2.  `ADD EAX, 10`      (Sumar 10 en el registro de la CPU)
3.  `MOV [saldo], EAX` (Escribir el resultado desde el registro a la RAM)

Si dos hilos ejecutan esta función al mismo tiempo sin sincronización:

```
    Hilo 1 (CPU Core 1)                  Hilo 2 (CPU Core 2)               Valor en RAM (saldo = 100)
┌───────────────────────────┐        ┌───────────────────────────┐        ────────────────────────────
│ MOV EAX, [saldo] (EAX=100)│        │                           │                  100
│                           │        │ MOV EAX, [saldo] (EAX=100)│                  100
│ ADD EAX, 10      (EAX=110)│        │                           │                  100
│                           │        │ ADD EAX, 10      (EAX=110)│                  100
│ MOV [saldo], EAX          │        │                           │                  110
│                           │        │ MOV [saldo], EAX          │                  110 (¡Pérdida de actualización!)
└───────────────────────────┘        └───────────────────────────┘
```

**Resultado:** A pesar de que dos hilos sumaron 10 cada uno, el valor final en la RAM es 110 en lugar de 120. La actualización del Hilo 1 fue destruida y sobrescrita por el Hilo 2.

---

### 5.2 Secciones Críticas y Exclusión Mutua

#### Explicación Conceptual
*   **Sección Crítica (*Critical Section*):** El bloque de código que accede a un recurso compartido (memoria, archivos, listas enlazadas) y que no debe ser ejecutado por más de un hilo a la vez.
*   **Exclusión Mutua (*Mutual Exclusion*):** La propiedad garantizada por el sistema que asegura que si un hilo está ejecutando su sección crítica, ningún otro hilo puede ingresar a su propia sección crítica para el mismo recurso.

---

### 5.3 Primitivas de Sincronización Basadas en Hardware: Operaciones Atómicas

#### Explicación Conceptual
Para construir mecanismos de exclusión mutua, el software requiere soporte directo del procesador mediante **Instrucciones Atómicas**. Una instrucción atómica se ejecuta en el hardware de la CPU de forma indivisible ("todo o nada"): la CPU garantiza que ninguna otra interrupción, lectura o escritura de otro núcleo puede ocurrir entre el inicio y el fin de la instrucción.

#### La Instrucción Test-and-Set
La instrucción atómica clásica `Test-and-Set` escribe un valor en una posición de memoria y retorna el valor antiguo que contenía la memoria, todo en un solo ciclo ininterrumpido de bus de memoria:

```c
// Comportamiento semántico equivalente ejecutado atómicamente por la CPU
int test_and_set(int *old_ptr, int new_value) {
    int old = *old_ptr;
    *old_ptr = new_value;
    return old; // Retorna el valor previo
}
```

---

### 5.4 Implementación de Locks: Spinlocks vs. Sleeplocks

#### 1. Spinlocks (Cerrojos de Espera Activa)
Un **Spinlock** es un cerrojo que utiliza una instrucción atómica dentro de un bucle activo (*Busy Waiting*) para adquirir el lock.

*   **Ventaja:** Inmediatez. No pone a dormir al hilo ni provoca un costoso cambio de contexto (*context switch*). Ideal para secciones críticas extremadamente cortas dentro del kernel (ej. manipular la `freelist`).
*   **Desventaja:** Desperdicio de CPU. Si el lock está ocupado, el hilo gira en un bucle consumiendo el $100\%$ de un núcleo de CPU hasta que el lock se libere.

```c
// Implementación de un Spinlock en el Kernel (xv6)
struct spinlock {
    uint locked; // 0 = libre, 1 = ocupado
};

void acquire(struct spinlock *lk) {
    // __sync_lock_test_and_set es una intrínseca del compilador que emite la instrucción atómica
    while(__sync_lock_test_and_set(&lk->locked, 1) != 0) {
        // Busy wait (espera activa): gira infinitamente hasta obtener el lock
    }
    __sync_synchronize(); // Barrera de memoria para prevenir reordenamiento de instrucciones
}

void release(struct spinlock *lk) {
    __sync_synchronize();
    __sync_lock_release(&lk->locked); // Establece lk->locked en 0 atómicamente
}
```

#### 2. Sleeplocks / Mutex Bloqueantes
Un **Sleeplock** (o `pthread_mutex_t` en POSIX) es un cerrojo que pone a dormir al hilo si el lock está ocupado.

*   **Ventaja:** Eficiencia de CPU. Libera la CPU para que otros procesos puedan ejecutarse mientras el hilo espera. Ideal para secciones críticas largas (ej. esperar I/O de disco o red).
*   **Desventaja:** Sobrecarga de latencia. Si el lock está ocupado, la llamada debe cambiar al modo kernel y poner el proceso en estado `SLEEPING`, incurriendo en el costo de dos cambios de contexto.

```c
// Implementación conceptual de un Sleeplock en el Kernel (xv6)
struct sleeplock {
    uint locked;         // Estado del cerrojo
    struct spinlock lk;  // Spinlock interno para proteger la estructura
    char *name;
};

void acquiresleep(struct sleeplock *lk) {
    acquire(&lk->lk); // Adquirir el spinlock de la estructura
    while (lk->locked) {
        // Cede la CPU: pone el proceso en estado SLEEPING y libera lk->lk atómicamente
        sleep(lk, &lk->lk); 
    }
    lk->locked = 1;
    release(&lk->lk);
}

void releasesleep(struct sleeplock *lk) {
    acquire(&lk->lk);
    lk->locked = 0;
    wakeup(lk); // Despierta a todos los procesos que dormían esperando este cerrojo
    release(&lk->lk);
}
```

---

### 5.5 Contención (Lock Contention) y Prevención de Interbloqueos (Deadlocks)

#### Contención de Cerrojos (Lock Contention)
La **contención** ocurre cuando múltiples hilos intentan adquirir el mismo lock simultáneamente. Si 100 hilos compiten por un único lock global que protege toda la memoria del sistema, el programa perderá todo el paralelismo y se ejecutará de forma secuencial, sufriendo degradación masiva de rendimiento.

Para mitigar la contención, los sistemas avanzados aplican **Granularidad Fina (*Fine-Grained Locking*)**, dividiendo un lock global gigante en múltiples locks pequeños para subestructuras independientes.

#### Interbloqueos (Deadlocks)
Un **Deadlock** es una condición catastrófica en la cual dos o más hilos están bloqueados permanentemente, esperando por recursos que los otros hilos sostienen en un círculo vicioso de dependencia.

```
 Hilo 1:                                Hilo 2:
 ┌──────────────────────────┐           ┌──────────────────────────┐
 │ Adquiere Lock A          │           │ Adquiere Lock B          │
 │ Intenta adquirir Lock B  │           │ Intenta adquirir Lock A  │
 │ (Bloqueado esperando H2) │           │ (Bloqueado esperando H1) │
 └──────────────────────────┘           └──────────────────────────┘
```

#### Regla de Oro para Prevenir Deadlocks: Ordenamiento Global de Locks
La estrategia principal para prevenir deadlocks en el diseño de kernels y software concurrente es imponer un **Orden de Adquisición de Cerrojos Estricto y Global**.

Si el sistema define la regla inviolable de que *el Lock A siempre debe adquirirse antes que el Lock B*, un interbloqueo es matemáticamente imposible:

```c
// Solución Correcta: Ambos hilos adquieren los locks exactamente en el mismo orden
void hilo_1(void) {
    acquire(&lock_A);
    acquire(&lock_B);
    // Sección crítica...
    release(&lock_B);
    release(&lock_A);
}

void hilo_2(void) {
    acquire(&lock_A); // Ambos compiten primero por A, eliminando la espera circular
    acquire(&lock_B);
    // Sección crítica...
    release(&lock_B);
    release(&lock_A);
}
```
