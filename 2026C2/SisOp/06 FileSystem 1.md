# Arquitectura de Sistemas de Archivos: Inodos, VFS y Dispositivos en Unix



## 1. Arquitectura de Sistemas de Archivos: Principios y Decisiones de Diseño de Unix

Un **Sistema de Archivos** (*File System*) es una abstracción fundamental del sistema operativo que organiza y provee **datos persistentes dotados de un nombre legible**. Los datos persistentes son aquellos que sobreviven a la finalización de los procesos y al apagado o corte de energía de la computadora.

### Decisiones de Diseño Fundamentales de Unix
El sistema de archivos de Unix introdujo un conjunto de decisiones arquitectónicas que se convirtieron en el estándar de la industria:

1. **Estructura Jerárquica Unificada:** Organización de la información en un árbol lógico iniciado en un directorio raíz (`/`), compuesto por directorios intermedios y archivos hojas.
2. **Tratamiento Consistente como Flujos de Bytes (*Byte Streams*):** Para el kernel, un archivo es simplemente una secuencia unidimensional e ininterrumpida de bytes sin estructura ni tipo impuesto. La interpretación del contenido (texto ASCII, UTF-8, binario, imágenes) es responsabilidad exclusiva de las aplicaciones en espacio de usuario.
3. **Creación y Eliminación Dinámica:** Los archivos y directorios pueden crearse y destruirse en tiempo de ejecución de forma transparente.
4. **Crecimiento Dinámico de Archivos:** Los archivos pueden expandir su tamaño en disco según la demanda del programa, sin requerir asignación previa de tamaño máximo.
5. **Esquema de Protección Unificado:** Control de acceso mediante permisos tridimensionales (**Lectura/Read `r`**, **Escritura/Write `w`**, **Ejecución/Execute `x`**) aplicados sobre tres clases de usuarios: Propietario (*Owner*), Grupo (*Group*) y Otros (*Others*).
6. **Filosofía "Todo es un Archivo" (*Everything is a File*):** Abstracción uniforme donde periféricos de hardware (discos rígidos, puertos serie, terminales) y subsistemas del kernel (estadísticas de procesos) se exponen como nodos del sistema de archivos.

---

## 2. Anatomía Interna de un Archivo: Metadatos y Datos

Un archivo se compone internamente de dos elementos claramente diferenciados por el sistema operativo:

```
┌─────────────────────────────────────────────────────────┐
│                      METADATOS                          │
│ - Inodo ID       - Permisos (mode)   - UID / GID        │
│ - Tamaño (bytes) - Marcas de tiempo  - Punteros a Disco │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│                        DATOS                            │
│ [ Secuencia unidimensional no estructurada de Bytes ]  │
└─────────────────────────────────────────────────────────┘
```

1. **Metadatos:** Información estructural comprendida y gestionada por el kernel sobre el archivo. Incluye el tamaño en bytes, marcas de tiempo de acceso y modificación, identificadores de propietario/grupo, permisos de seguridad y punteros físicos a los bloques del almacenamiento.
2. **Datos:** El contenido propiamente dicho que la aplicación escribe o lee a través de las llamadas al sistema `write()` y `read()`.

### Inspección de Metadatos en la Terminal
Para inspeccionar la metadata de un archivo en Linux, se utiliza el comando `stat` o las opciones detalladas de `ls`:

```bash
$ stat prueba.txt
  Fichero: 'prueba.txt'
  Tamaño: 1457       Bloques: 8          Bloque E/S: 4096   fichero regular
Dispositivo: 803h/2051d  Nodo-i: 28332376    Enlaces: 1
Acceso: (0664/-rw-rw-r--)  Uid: ( 1000/ usuario)   Gid: ( 1000/ usuario)
Acceso: 2026-05-31 15:17:23.460862535 -0300
Modificación: 2026-01-13 20:48:39.716785878 -0300
Cambio: 2026-01-13 20:48:39.760786398 -0300
```

---

## 3. El Inodo (Index Node): Representación del Archivo en Disco

Un **Inodo** (*Index Node*) es la estructura de datos fundamental que representa un archivo individual en el sistema de archivos.

> **Regla de Oro:** Un Inodo almacena **únicamente la metadata y los punteros a los bloques de datos** en disco. El Inodo **NO contiene los datos del archivo** y **NO contiene el nombre del archivo**.

### Estructura de Direccionamiento de Bloques (Árbol Desbalanceado)
Los dispositivos físicos de almacenamiento leen y escriben en **bloques de datos** de tamaño fijo (típicamente 4 KB en Linux, equivalentes al tamaño de página de memoria). Para direccionar estos bloques, el inodo utiliza una estructura de árbol desbalanceado:

```
[ INODO ]
├── Permisos, UID, GID, Size
└── Punteros a Bloques (ej. 15 punteros de 32 bits):
    ├── Punteros Directos [0..11] ───► [Bloque Datos 0] ... [Bloque Datos 11]
    ├── Indirección Simple [12]   ───► [Bloque de Punteros] ───► 1024 Bloques
    ├── Indirección Doble [13]    ───► [Bloque N1] ───► [Bloques N2] ───► 1024x1024
    └── Indirección Triple [14]   ───► [Bloque N1] ───► [N2] ───► [N3] ───► 1024^3
```

#### Justificación del Diseño Desbalanceado
Esta estructura no es un árbol balanceado tradicional debido a un patrón empírico clave: **la inmensa mayoría de los archivos en un sistema de archivos son de tamaño pequeño** (pocos Kilobytes).
* Al disponer de 12 **punteros directos**, los archivos pequeños leen directamente las direcciones del disco almacenadas en el propio inodo sin requerir lecturas secundarias de metadatos.
* Para archivos medianos o gigantescos, se activan progresivamente los niveles de **indirección simple, doble y triple**, permitiendo acceder a gigabytes o terabytes de datos pagando un costo adicional de lecturas en bloque solo cuando el tamaño del archivo lo justifica.

---

### Tipos de Inodos (`mode`)
En Linux, el tipo de archivo se define en los bits de modo del inodo:

| Macro POSIX | Valor Octal | Descripción del Tipo de Inodo |
| :--- | :--- | :--- |
| `S_IFREG` | `0100000` | Archivo Regular (*Regular File*) |
| `S_IFDIR` | `0040000` | Directorio (*Directory*) |
| `S_IFBLK` | `0060000` | Dispositivo de Bloque (*Block Device File*) |
| `S_IFCHR` | `0020000` | Dispositivo de Carácter (*Character Device File*) |
| `S_IFLNK` | `0120000` | Enlace Simbólico (*Symbolic Link*) |
| `S_IFSOCK`| `0140000` | Socket Unix (*Socket File*) |
| `S_IFIFO` | `0010000` | Tubería Nombrada (*FIFO / Named Pipe*) |

---

## 4. Directorios y Dentries (Directory Entries)

Dado que un inodo no almacena el nombre de un archivo, ¿dónde se guardan los nombres legibles y cómo se implementa la jerarquía de directorios?

### El Dentry (Directory Entry)
Un **Dentry** es una estructura lógica que establece una asociación directa entre un **nombre textual de archivo** y su correspondiente **número de inodo**:

$$	ext{Dentry} = \langle 	ext{Nombre Textual}, 	ext{Número de Inodo} 
angle$$

### La Naturaleza de un Directorio
En Unix, **un directorio es un archivo especial cuyo contenido de datos es una lista estructurada de dentries**.

```
[ Bloque de Datos de un Directorio (ej. /home/usuario) ]
┌──────────┬──────────┬──────────┬────────────────┐
│ Inode ID │ Rec_Len  │ Name_Len │ Name           │
├──────────┼──────────┼──────────┼────────────────┤
│ 21       │ 12       │ 1        │ .              │
│ 22       │ 12       │ 2        │ ..             │
│ 53       │ 16       │ 5        │ home1          │
│ 67       │ 28       │ 3        │ usr            │
│ 28332376 │ 16       │ 10       │ prueba.txt     │
└──────────┴──────────┴──────────┴────────────────┘
```

### Las Tres Reglas Inviolables de la Jerarquía
Para comprender la arquitectura interna del árbol de archivos, deben cumplirse tres reglas estructurales:

1. **Un Dentry NUNCA apunta a otro Dentry**; apunta únicamente a un Inodo.
2. **Un Inodo NUNCA apunta a otros Inodos**; apunta únicamente a bloques de datos o dispositivos.
3. **La Jerarquía se construye cíclicamente:** Un Dentry apunta a un Inodo marcado como `S_IFDIR` (Directorio). Los bloques de datos apuntados por dicho Inodo contienen una lista de Dentries, los cuales a su vez apuntan a nuevos Inodos.

```
[ Dentry: "usr" ] ───► [ Inodo 67 (S_IFDIR) ] ───► [ Bloque de Datos ]
                                                          │
                                         ┌────────────────┴────────────────┐
                                         ▼                                 ▼
                               [ Dentry: "bin" ]                 [ Dentry: "src" ]
                                       │                                 │
                                       ▼                                 ▼
                            [ Inodo 101 (S_IFDIR) ]           [ Inodo 205 (S_IFREG) ]
```

---

### Enlaces Duros (*Hard Links*)
Debido a que los nombres son simplemente entradas de directorio (dentries) externas al inodo, **múltiples dentries pueden apuntar al mismo número de inodo**. Esto se conoce como un **Hard Link**.

```
[ Dentry: /tmp/test1 ] ────────┐
                               ├──────► [ Inodo 5664 ] ───► [ Datos en Disco ]
[ Dentry: /home/test2 ] ───────┘
```

No existe un "nombre original" primario frente a un "alias": todos los dentries que apuntan al mismo inodo poseen la misma jerarquía. El inodo mantiene un contador de referencias (`Link Count`). Cuando se ejecuta la llamada al sistema `unlink()` (borrar un archivo), el kernel decrementa dicho contador; los bloques de datos físicos en disco solo se liberan cuando el `Link Count` llega a cero y ningún proceso mantiene el archivo abierto.

---

## 5. Integración con Procesos: La Tabla de Archivos Abiertos (Open File Table)

Para conectar el sistema de archivos persistente con los procesos dinámicos en ejecución, el kernel de Unix implementa una arquitectura de resolución de tres niveles:

```
[ Proceso A ]
  Tabla Descriptores (Por Proceso)
  ┌────┬──────────┐
  │ FD │ File Ptr │
  ├────┼──────────┤
  │ 3  │   ───────┼────────┐
  └────┴──────────┘        │
                           ▼
[ Proceso B ]       [ Tabla de Archivos Abiertos (System-Wide) ]
  Tabla Descriptores  ┌────────┬─────────────┬──────────┐
  ┌────┬──────────┐   │ Offset │ Status Flags│ InodePtr │
  │ FD │ File Ptr │   ├────────┼─────────────┼──────────┤
  ├────┼──────────┤   │ 1024   │ O_RDONLY    │   ───────┼───┐
  │ 3  │   ───────┼──►│ 0      │ O_WRONLY    │   ───────┼─┐ │
  └────┴──────────┘   └────────┴─────────────┴──────────┘ │ │
                                                          │ │
                    [ Tabla de Inodos (System-Wide) ]     │ │
                      ┌──────────┬────────────┐           │ │
                      │ Inode ID │ File Type  │           │ │
                      ├──────────┼────────────┤           │ │
                      │ 28332376 │ S_IFREG    │◄──────────┼─┘
                      │ 5139     │ S_IFCHR    │◄──────────┘
                      └──────────┴────────────┘
```

1. **Tabla de Descriptores de Archivo (Por Proceso):** Mantenida dentro de la estructura `task_struct` de cada proceso. Indiza números enteros pequeños (`fd 0, 1, 2, 3...`) a punteros de la estructura `struct file`.
2. **Tabla de Archivos Abiertos (*Open File Table* - System-Wide):** Tabla global mantenida por el kernel que representa cada llamada `open()` activa. Almacena:
   * El **Offset actual** (cursor de lectura/escritura).
   * Los **Flags de Estado** (modo de acceso como `O_RDONLY`, `O_WRONLY`, `O_APPEND`).
   * El **Contador de referencias** de la apertura.
   * Un puntero al Inodo en memoria.
3. **Tabla de Inodos (*System-Wide Inode Table*):** Copia en memoria RAM de los inodos activos leídos del disco o generados dinámicamente.

#### ¿Por qué existe la Tabla Intermedia de Archivos Abiertos?
La separación entre la tabla de descriptores del proceso y la tabla de archivos abiertos global permite dos escenarios fundamentales:
* **Múltiples `open()` independientes:** Si dos procesos abren el mismo archivo por su ruta, cada uno obtiene una entrada independiente en la Tabla de Archivos Abiertos. Cada proceso posee su propio **offset**, permitiéndoles leer o escribir de forma autónoma sin interferir en la posición del otro.
* **Compartición de Offset (`fork()` o `dup()`):** Si un proceso padre llama a `fork()`, el proceso hijo hereda una copia idéntica de la tabla de descriptores apuntando a las **mismas entradas de la Tabla de Archivos Abiertos global**. Por lo tanto, si el hijo escribe en el archivo y avanza el offset, el padre verá inmediatamente el offset actualizado.

---

## 6. Virtual File System (VFS): Capa de Abstracción y Polimorfismo en C

El **Virtual File System (VFS)** es el subsistema del kernel de Linux que proporciona una capa de abstracción sobre los sistemas de archivos físicos o virtuales concretos (Ext4, FAT32, NTFS, NFS, procfs).

```
[ Aplicación en Espacio de Usuario ]
                │
                ▼ (System Calls: open, read, write)
┌─────────────────────────────────────────────────────────┐
│               Virtual File System (VFS)                 │
├───────────────┬───────────────┬─────────────┬───────────┤
│ Driver Ext4   │ Driver FAT32  │ Driver NFS  │ procfs    │
└───────┬───────┴───────┬───────┴──────┬──────┴─────┬─────┘
        │               │              │            │
        ▼               ▼              ▼            ▼
   [ Disco SSD ]   [ Pendrive ]     [ Red ]     [ Memoria ]
```

El VFS actúa como un "pegamento" polimórfico: permite que las llamadas al sistema del usuario (`read()`, `write()`) se ejecuten de manera transparente e idéntica sin importar el sistema de archivos ni el dispositivo físico subyacente.

### Los Cuatro Objetos Principales del VFS

El VFS define cuatro estructuras orientadas a objetos (implementadas en C mediante estructuras de punteros a funciones):

1. **Objeto Superbloque (`struct super_block`):** Representa a un sistema de archivos montado completo. Almacena la metadata global del medio físico y un puntero al Inodo raíz.
2. **Objeto Inodo (`struct inode`):** Representa la metadata de un archivo individual en el modelo abstracto del VFS.
3. **Objeto Dentry (`struct dentry`):** Representa un componente individual del nombre de la ruta (*path*), facilitando la traducción de nombres y el almacenamiento en caché de directorio (*dentry cache*).
4. **Objeto Archivo (`struct file`):** Representa un archivo abierto por un proceso (asociado a la entrada de la Open File Table).

### Tablas de Operaciones (Polimorfismo en C)
Cada objeto del VFS contiene un puntero a una tabla de operaciones compuestas por punteros a funciones. Cuando un creador de un sistema de archivos implementa su driver, asigna sus propias funciones específicas a estas tablas:

```c
// Estructura de Operaciones de Archivo del VFS (f_ops)
struct file_operations {
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    int (*open) (struct inode *, struct file *);
    int (*release) (struct inode *, struct file *);
};

// Ejemplo: Implementación para un Driver de Disco Físico
struct file_operations ext4_file_operations = {
    .read = ext4_file_read,
    .write = ext4_file_write,
    .open = ext4_file_open,
};
```

Cuando el usuario invoca `write(fd, buf, len)`, el kernel ejecuta internamente:

```c
file->f_op->write(file, buf, len, &file->f_pos);
```

Invocando de forma totalmente polimórfica la función correcta del sistema de archivos correspondiente.

---

## 7. Tratamiento de Dispositivos como Archivos (`/dev`)

Fiel al principio de Unix "Todo es un archivo", los dispositivos de entrada/salida físicos y virtuales se exponen a los programas como archivos especiales ubicados convencionalmente dentro del directorio `/dev`.

```
[ Aplicación / Usuario ]
           │
           ▼ (open / read / write en /dev/ttyS0)
┌─────────────────────────────────────────────────────────┐
│               Virtual File System (VFS)                 │
└──────────────────────────┬──────────────────────────────┘
                           │ Ruteo por Major & Minor Number
                           ▼
┌─────────────────────────────────────────────────────────┐
│                Driver del Dispositivo                   │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                 Hardware Periférico                     │
└─────────────────────────────────────────────────────────┘
```

### Clasificación de Dispositivos

1. **Dispositivos de Carácter (*Character Devices* - `c`):** Transferencia de datos flujo a flujo, byte a byte, sin almacenamiento en bloques de tamaño fijo.
   * *Ejemplos:* Terminales (`/dev/tty`), puertos serie (`/dev/ttyS0`), generadores de números aleatorios (`/dev/random`), el pozo sin fondo (`/dev/null`).
2. **Dispositivos de Bloque (*Block Devices* - `b`):** Transferencia de datos en bloques direccionables de tamaño fijo (ej. 512 o 4096 bytes) que admiten acceso aleatorio eficiente.
   * *Ejemplos:* Discos rígidos (`/dev/sda`), particiones (`/dev/sda1`), memorias USB, dispositivos de bucle (`/dev/loop0`).

---

### Ruteo mediante Major y Minor Numbers

Los archivos especiales de dispositivo no almacenan datos en disco; en su lugar, su inodo guarda un par de identificadores numéricos que el VFS utiliza para enrutar las peticiones E/S hacia el driver correspondiente:

* **Major Number:** Identifica el **driver o controlador específico** en el kernel que administra ese tipo de hardware.
* **Minor Number:** Identifica el **dispositivo específico o partición** individual dentro del grupo administrado por ese controlador.

```bash
# Ejemplo: Creación manual de un nodo de dispositivo de caracteres con mknod
$ sudo mknod /dev/ttyS1 c 4 65
```
* `/dev/ttyS1`: Ruta del nuevo archivo especial.
* `c`: Especifica que es un dispositivo de caracteres (`S_IFCHR`).
* `4`: Major number ( Driver de puerto serie ).
* `65`: Minor number ( Subdispositivo o puerto número 1 ).

En los sistemas modernos, el demonio `udev` en espacio de usuario monitorea dinámicamente los eventos del kernel y crea automáticamente los nodos de dispositivo en `/dev`.

---

### Montaje de Sistemas de Archivos (`mount`)

Para vincular la estructura interna de bloques almacenada en un dispositivo de bloques (como `/dev/sda1`) con el árbol de directorios global, se utiliza la llamada al sistema y comando `mount`:

```bash
$ sudo mount -t ext4 /dev/sda1 /mnt/mi_directorio
```

```
[ Árbol Existente ]                 [ Volumen sin Montar ]
       /                                   / (Raíz del volumen)
       ├── bin                             ├── documento.txt
       └── mnt                             └── fotos
            └── mi_directorio (Punto Montaje)
                     │
                     ▼ (Tras ejecutar mount)
[ Árbol Resultante Unificado ]
       /
       ├── bin
       └── mnt
            └── mi_directorio
                     ├── documento.txt
                     └── fotos
```

El directorio `/mnt/mi_directorio` actúa como el **Punto de Montaje** (*Mount Point*). Durante el tiempo que el volumen permanezca montado, el VFS intercepta los accesos a dicho dentry y los redirige transparente al inodo raíz del sistema de archivos contenido dentro del dispositivo `/dev/sda1`.

---

### Sistemas de Archivos Virtuales: El Caso de `procfs` (`/proc`)

No todos los sistemas de archivos están respaldados por un almacenamiento físico persistente. Los **Sistemas de Archivos Virtuales** existen únicamente en la memoria RAM y son generados dinámicamente por el kernel al ser leídos.

Un ejemplo emblemático es el **`procfs`** (montado en `/proc`). Este sistema de archivos expone la estructura interna del kernel y el estado de los procesos como si fuesen archivos de texto:

```bash
# Montaje del sistema de archivos virtual procfs
$ sudo mount -t proc proc /proc

# Consulta de los sistemas de archivos soportados por el kernel actual
$ cat /proc/filesystems
nodev   sysfs
nodev   tmpfs
nodev   proc
nodev   cgroup
        ext4
```

La etiqueta `nodev` indica que dichos sistemas de archivos son enteramente virtuales y no poseen un dispositivo de bloques subyacente. Al hacer `cat /proc/cpuinfo` o `cat /proc/meminfo`, el VFS no lee ningún disco físico: ejecuta funciones internas del kernel que recolectan el estado del hardware en tiempo real y lo devuelven formateado como un flujo de bytes.
