# Arquitectura de Sistemas de Archivos: Estructura en Disco, Asignación, Caché y Consistencia

## 1. Diseño e Implementación de un Sistema de Archivos Simplificado (VSFS)

### 1.1 Objetivos de Diseño y Abstracción sobre Medios Orientados a Bloques
El objetivo fundamental de un sistema de archivos (*file system*) es construir la abstracción de alto nivel denominada **archivo** y la estructura jerárquica de **directorios** sobre un medio de almacenamiento secundario no volátil que opera físicamente orientado a bloques fijados por el hardware o el controlador de disco.

Para transformar una matriz lineal de sectores o bloques físicos sin estructura intrínseca en un sistema organizado de almacenamiento persistente, todo sistema de archivos debe resolver de manera concurrente y eficiente dos necesidades de diseño primarias:
1. **La Estructura de Índice de Archivo:** Un mecanismo orientado a datos que permita identificar y localizar de forma unívoca todos los bloques de almacenamiento que componen el contenido de un archivo determinado. Esta estructura debe soportar accesos secuenciales y aleatorios, garantizando escalabilidad para archivos de diversos tamaños y maximizando la localidad espacial.
2. **El Mapa de Espacio Libre:** Una estructura de datos persistente que lleve un registro detallado y actualizado de cuáles bloques de disco y cuáles metadatos están asignados a archivos existentes y cuáles se encuentran disponibles para la creación o expansión de nuevos archivos.

---

### 1.2 Layout Físico del Disco en VSFS
El sistema de archivos simplificado **VSFS** (*Very Simple File System*) modela el disco como un arreglo unidimensional de $N$ bloques de tamaño uniforme (típicamente $4	ext{ KB}$ o $4096	ext{ bytes}$). Para visualizar su diseño, considérese una partición pequeña dividida en 64 bloques contiguos (indexados de 0 a 63). 

El layout físico del disco en VSFS organiza los bloques en regiones bien definidas:

```
+---+---+---+---+---+---+---+---+-----------------------------------+
| S | i | d | I | I | I | I | I | D D D D D D D D D D D D D D D D D | ...
+---+---+---+---+---+---+---+---+-----------------------------------+
  0   1   2   3   4   5   6   7   8                             63
```

1. **Superbloque (`S` - Bloque 0):** Ocupa el primer bloque de la partición. Almacena la información administrativa global de la estructura del sistema de archivos. Contiene parámetros críticos como:
   * Cantidad total de inodos y cantidad total de bloques en la partición.
   * Ubicación inicial de los bitmaps y de la tabla de inodos.
   * Identificador mágico (*Magic Number*) para validar el tipo de sistema de archivos al momento del montaje (*mount*).
   * Número de inodo correspondiente al directorio raíz (`/`).

2. **Bitmap de Inodos (`i` - Bloque 1) y Bitmap de Datos (`d` - Bloque 2):** Son mapas de bits empleados para la administración eficiente del espacio libre. Cada bit representa la disponibilidad de un recurso: un valor `0` indica que el recurso está libre y un valor `1` indica que está ocupado.
   * **Bitmap de Inodos (`i-bmap`):** Registra el estado de ocupación de cada entrada en la tabla de inodos.
   * **Bitmap de Datos (`d-bmap`):** Registra el estado de ocupación de cada bloque en la región de datos.

3. **Tabla de Inodos (`I` - Bloques 3 a 7):** Es una región reservada contigua que contiene un arreglo de estructuras de metadatos denominadas **inodos** (*index nodes*). Cada inodo representa a un archivo o directorio individual dentro del sistema de archivos y posee un tamaño fijo (por ejemplo, $256	ext{ bytes}$).

4. **Región de Datos (`D` - Bloques 8 a 63):** Es la sección más extensa del disco, destinada a almacenar el contenido real de los archivos de usuario y las entradas que componen los bloques de directorios.

#### Cálculo Directo de Posición de Inodos en Disco
Dado que la tabla de inodos se almacena como un arreglo contiguo de estructuras de tamaño uniforme, la localización física en disco del inodo asociado a un número de inodo (*inumber*) $i$ se calcula mediante una conversión de dirección base más offset:

$$	ext{Dirección Byte en Disco} = 	ext{Inicio Tabla Inodos} + (i 	imes 	ext{sizeof}(	ext{inode}))$$

*Ejemplo de cálculo:* En un sistema donde la tabla de inodos comienza a los $12	ext{ KB}$ ($12288	ext{ bytes}$) y cada inodo mide $256	ext{ bytes}$, el acceso al inodo $i = 32$ requiere calcular:

$$	ext{Offset Byte} = 12288 + (32 	imes 256) = 12288 + 8192 = 20480	ext{ bytes} \quad (20	ext{ KB})$$

El controlador del sistema de archivos lee el bloque de $4	ext{ KB}$ en la posición de $20	ext{ KB}$ (bloque físico 5) e inspecciona la estructura correspondiente.

---

### 1.3 Metodología de Cálculo de Capacidad e Inodos (Criterio Práctico de Cátedra)

#### Explicación Conceptual
Al diseñar un sistema de archivos VSFS para una capacidad física dada, surge el dilema de cuántos bloques asignar a la tabla de inodos frente a la región de datos. Asignar demasiados bloques a inodos reduce el espacio disponible para almacenar datos; asignar muy pocos limita la cantidad total de archivos que pueden crearse, provocando que el sistema quede inoperable por falta de inodos (*out of inodes*) aun teniendo espacio libre en disco.

#### Criterio Práctico de Examen
Para el cálculo de dimensiones en problemas prácticos y exámenes, se adopta el criterio simplificado estándar: **en un disco de $N$ bloques físicos se asume la necesidad teórica de soportar hasta $N$ archivos**. Por lo tanto, se dimensiona el sistema para albergar $N$ inodos, garantizando que en el peor escenario (donde cada archivo ocupe un solo bloque) no falten inodos.

#### Metáfora
Imagínese un hotel con $N$ habitaciones. El administrador debe construir un casillero para guardar las fichas de registro (inodos) de los huéspedes. Para evitar que el casillero quede chico si cada huésped alquila una sola habitación por separado, el administrador construye espacio para $N$ fichas, independientemente de que algunas familias alquilen varias habitaciones juntas.

#### Caso Práctico Desarrollado: Dimensionamiento de un Disco de 1024 Bloques

**Consigna:** Diseñar el layout físico de un sistema de archivos VSFS para un disco de 1024 bloques de $4	ext{ KB}$ ($4096	ext{ bytes}$) cada uno, donde la estructura de cada inodo mide $256	ext{ bytes}$.

**Paso 1: Determinar la cantidad de inodos requeridos**
Aplicando el criterio estándar de $N$ archivos para $N$ bloques:
$$	ext{Cantidad de inodos requeridos} = 1024	ext{ inodos}$$

**Paso 2: Calcular la densidad de inodos por bloque**
$$	ext{Inodos por bloque} = rac{	ext{Tamaño de bloque}}{	ext{sizeof}(	ext{inode})} = rac{4096	ext{ bytes}}{256	ext{ bytes}} = 16	ext{ inodos/bloque}$$

**Paso 3: Calcular la cantidad de bloques requeridos para la Tabla de Inodos**
$$	ext{Bloques de Inodos} = rac{	ext{Inodos totales}}{	ext{Inodos por bloque}} = rac{1024}{16} = 64	ext{ bloques}$$

**Paso 4: Validar la capacidad de los Bitmaps**
Un bloque de $4	ext{ KB}$ contiene $4096 	imes 8 = 32768	ext{ bits}$.
* 1 bloque de bitmap de inodos puede administrar hasta $32768$ inodos (cubre holgadamente los 1024 requeridos).
* 1 bloque de bitmap de datos puede administrar hasta $32768$ bloques de datos (cubre holgadamente los 1024 bloques totales del disco).

**Paso 5: Distribución Final de Bloques en Disco**

| Región | Descripción | Cantidad de Bloques |
| :--- | :--- | :--- |
| **Superbloque (`S`)** | Información global del sistema | 1 bloque |
| **Bitmap de Inodos (`i-bmap`)** | Registro de ocupación de inodos | 1 bloque |
| **Bitmap de Datos (`d-bmap`)** | Registro de ocupación de datos | 1 bloque |
| **Tabla de Inodos (`Inode Table`)** | Arreglo de 1024 inodos ($64 	imes 16$) | 64 bloques |
| **Región de Datos (`Data Region`)** | Espacio restante para datos | $1024 - 1 - 1 - 1 - 64 = 957	ext{ bloques}$ |

---

## 2. Modelado de Accesos y Resolución de Rutas en Disco

### 2.1 La Estructura de Directorios y Registros Dentry
En la arquitectura de sistemas tipo Unix, un directorio no es un contenedor físico especial, sino un **archivo regular con un tipo de inodo marcado como directorio** (`S_IFDIR`). 

El bloque de datos asignado a un directorio contiene una lista estructurada de entradas de directorio conocidas como **dentries** (*directory entries*). Cada dentry enlaza un nombre de archivo legible por humanos con su respectivo número de inodo (*inumber*).

```
 +---------------------------------------------------+
 | Bloque de Datos de Directorio (/etc)             |
 +---------------------------------------------------+
 | Nombre Entrada (string) | Número de Inodo (uint) |
 +-------------------------+-------------------------+
 | .                       | 7                       |
 | ..                      | 2                       |
 | group                   | 81                      |
 | passwd                  | 82                      |
 | shadow                  | 83                      |
 +---------------------------------------------------+
```

---

### 2.2 Algoritmo de Recorrido Componente a Componente (Lookup)
Para acceder a un archivo identificado por una ruta absoluta (por ejemplo, `/etc/passwd`), el sistema operativo debe resolver la ruta componente por componente partiendo del directorio raíz (`/`), cuyo número de inodo es conocido a priori por el kernel (típicamente el inodo 2).

#### Secuencia de Operaciones para la Resolución de `/etc/passwd`
1. **Acceso al Inodo Raíz:** El kernel lee el inodo 2 (`/`) desde la tabla de inodos para obtener la ubicación de sus bloques de datos.
2. **Lectura de Datos del Raíz:** Se lee el bloque de datos del directorio `/` y se escanea la lista de dentries en busca de la cadena `"etc"`. Se extrae su número de inodo asociado (por ejemplo, inodo 7).
3. **Acceso al Inodo `/etc`:** Se lee el inodo 7 desde la tabla de inodos, verificando sus permisos y obteniendo la ubicación de sus bloques de datos.
4. **Lectura de Datos de `/etc`:** Se lee el bloque de datos de `/etc` y se busca la entrada `"passwd"`. Se obtiene su número de inodo (por ejemplo, inodo 82).
5. **Acceso al Inodo `/etc/passwd`:** Se lee el inodo 82. Se inspeccionan los permisos y metadatos del archivo regular.
6. **Lectura de Datos del Archivo:** Si la operación requiere acceder al contenido (como en `cat`), se leen los bloques de datos apuntados por el inodo 82.

---

### 2.3 Hard Links vs. Soft Links (Symbolic Links)

La diferencia estructural entre un enlace duro (*hard link*) y un enlace simbólico (*soft link*) es fundamental para comprender el modelado de accesos a disco:

```
HARD LINK:
 [Dentry: "file1" -> Inode 61]                                  ==> [Inode 61 (link_count = 2)] ==> [Bloque Datos "Hola"]
 [Dentry: "file2" -> Inode 61] /

SOFT LINK:
 [Dentry: "soft1" -> Inode 309] ==> [Inode 309 (type=symlink)] ==> [Bloque Datos: "/home/user/file1"]
                                                                        | (re-resolución de ruta)
                                                                        v
                                                                 [Inode 61] ==> [Bloque Datos]
```

#### Tabla Comparativa de Características

| Propiedad | Hard Link (`ln file target`) | Soft / Symbolic Link (`ln -s target link`) |
| :--- | :--- | :--- |
| **Definición Estructural** | Múltiples dentries apuntando al mismo número de inodo. | Archivo especial con inodo propio que almacena la ruta en texto. |
| **Inodo Dedicado** | No. Utiliza el mismo inodo del archivo original. | Sí. Se asigna un nuevo inodo de tipo `S_IFLNK`. |
| **Consumo de Bloque de Datos** | Ninguno adicional (solo una entrada dentry en el directorio). | Ocupa un inodo y un bloque de datos para la cadena del path. |
| **Contador de Enlaces (`link_count`)** | Incrementa el campo `link_count` del inodo en 1. | Mantiene el `link_count` del inodo objetivo intacto. |
| **Comportamiento ante Eliminación** | Eliminar un nombre ejecuta `unlink()`, reduciendo `link_count`. El archivo persiste mientras `link_count > 0`. | Eliminar el archivo destino deja al enlace simbólico "huérfano" (*dangling link*). |
| **Soporte entre Distintos Filesystems** | **Imposible.** Los inodos son locales a cada partición/disco. | **Sí.** La cadena del path es universal y cruza particiones. |

---

### 2.4 Resolución Paso a Paso de Accesos a Disco (Ejercicios de Parcial)

A continuación se analizan cuatro casos de estudio típicos de evaluación, donde se requiere determinar a cuántos inodos y bloques de datos accede el sistema de archivos para ejecutar un comando específico tras una preparación previa del entorno.

---

#### Caso 1: Inspección de Metadatos con `ls -l`
**Comandos de preparación:**
```bash
# mkdir /dir
# echo 'hola' > /dir/x
```
**Comando evaluado:**
```bash
# ls -l /dir/x
```

**Secuencia de accesos a disco:**
1. Lee el **Inodo `/`** (tipo directorio).
2. Lee el **Bloque de datos de `/`** (busca la entrada `"dir"`, obtiene inodo de `/dir`).
3. Lee el **Inodo `/dir`** (tipo directorio).
4. Lee el **Bloque de datos de `/dir`** (busca la entrada `"x"`, obtiene inodo de `/dir/x`).
5. Lee el **Inodo `/dir/x`** (tipo regular).

**Análisis técnico:**
El comando `ls -l` solo requiere mostrar la metadata del archivo (permisos, propietario, tamaño, marcas de tiempo), la cual se encuentra almacenada íntegramente dentro del inodo. Por lo tanto, **no se lee el bloque de datos de `/dir/x`**.

**Resultado final:**
* **Inodos accedidos:** 3 (inodo `/`, inodo `/dir`, inodo `/dir/x`)
* **Bloques de datos accedidos:** 2 (bloque datos `/`, bloque datos `/dir`)

---

#### Caso 2: Lectura de Contenido con `cat`
**Comandos de preparación:**
```bash
# mkdir /dir
# mkdir /dir/s
# echo 'mundo' > /dir/s/y
```
**Comando evaluado:**
```bash
# cat /dir/s/y
```

**Secuencia de accesos a disco:**
1. Lee el **Inodo `/`** (directorio).
2. Lee el **Bloque de datos de `/`** (busca `"dir"`).
3. Lee el **Inodo `/dir`** (directorio).
4. Lee el **Bloque de datos de `/dir`** (busca `"s"`).
5. Lee el **Inodo `/dir/s`** (directorio).
6. Lee el **Bloque de datos de `/dir/s`** (busca `"y"`).
7. Lee el **Inodo `/dir/s/y`** (archivo regular).
8. Lee el **Bloque de datos de `/dir/s/y`** (imprime la cadena `'mundo'`).

**Análisis técnico:**
A diferencia de `ls`, la utilidad `cat` requiere volcar el contenido real del archivo a la salida estándar, exigiendo la lectura efectiva del bloque de datos del archivo regular.

**Resultado final:**
* **Inodos accedidos:** 4 (inodo `/`, inodo `/dir`, inodo `/dir/s`, inodo `/dir/s/y`)
* **Bloques de datos accedidos:** 4 (bloque `/`, bloque `/dir`, bloque `/dir/s`, bloque `/dir/s/y`)

---

#### Caso 3: Persistencia de Datos tras `rm` sobre Enlace Duro
**Comandos de preparación:**
```bash
# mkdir /dir
# echo 'hola' > /dir/x
# ln /dir/x /dir/h
# rm /dir/x
```
**Comando evaluado:**
```bash
# cat /dir/h
```

**Secuencia de accesos a disco:**
1. Lee el **Inodo `/`** (directorio).
2. Lee el **Bloque de datos de `/`** (busca `"dir"`).
3. Lee el **Inodo `/dir`** (directorio).
4. Lee el **Bloque de datos de `/dir`** (busca la entrada `"h"`, obtiene inodo compartido).
5. Lee el **Inodo `/dir/h`** (inodo compartido del archivo original).
6. Lee el **Bloque de datos de `/dir/h`** (obtiene el contenido `'hola'`).

**Análisis técnico:**
Al ejecutar `ln /dir/x /dir/h`, se creó un nuevo dentry `"h"` en el directorio `/dir` que apunta al mismo número de inodo de `x`, incrementando su `link_count` a 2. La posterior ejecución de `rm /dir/x` invocó la llamada al sistema `unlink("/dir/x")`, eliminando únicamente el dentry `"x"` y decrementando `link_count` a 1. El inodo y los bloques de datos no se liberaron. Por ende, `cat /dir/h` accede directamente al archivo sin penalización.

**Resultado final:**
* **Inodos accedidos:** 3 (inodo `/`, inodo `/dir`, inodo del archivo)
* **Bloques de datos accedidos:** 3 (bloque `/`, bloque `/dir`, bloque de datos del archivo)

---

#### Caso 4: Doble Resolución por Enlace Simbólico
**Comandos de preparación:**
```bash
# mkdir /dir
# mkdir /dir/s
# echo 'mundo' > /dir/s/y
# ln -s /dir/s/y /dir/y
```
**Comando evaluado:**
```bash
# cat /dir/y
```

**Secuencia de accesos a disco:**

*Fase A: Resolución del Enlace Simbólico (`/dir/y`)*
1. Lee **Inodo `/`** $
ightarrow$ 2. Lee **Bloque datos `/`** (obtiene inodo `/dir`).
3. Lee **Inodo `/dir`** $
ightarrow$ 4. Lee **Bloque datos `/dir`** (obtiene inodo de `/dir/y`).
5. Lee **Inodo `/dir/y`** (detecta tipo `S_IFLNK`).
6. Lee **Bloque de datos de `/dir/y`** (obtiene la cadena de texto con el path destino: `"/dir/s/y"`).

*Fase B: Resolución de la Ruta Destino (`/dir/s/y`)*
7. Lee **Inodo `/`** $
ightarrow$ 8. Lee **Bloque datos `/`**.
9. Lee **Inodo `/dir`** $
ightarrow$ 10. Lee **Bloque datos `/dir`**.
11. Lee **Inodo `/dir/s`** $
ightarrow$ 12. Lee **Bloque datos `/dir/s`** (obtiene inodo de `/dir/s/y`).
13. Lee **Inodo `/dir/s/y`** (tipo regular).
14. Lee **Bloque de datos de `/dir/s/y`** (contenido `'mundo'`).

**Análisis técnico:**
El sistema operativo debe procesar el enlace simbólico leyendo su bloque de datos para extraer la ruta de destino y, posteriormente, reiniciar el proceso de resolución de ruta desde la raíz para el nuevo path. Sin considerar optimizaciones de caché del kernel, el acceso requiere resolver ambas rutas de forma secuencial.

**Resultado final:**
* **Inodos accedidos:** 7 (Fase A: 3 inodos; Fase B: 4 inodos)
* **Bloques de datos accedidos:** 7 (Fase A: 3 bloques; Fase B: 4 bloques)

---

## 3. Estudio de Casos Arquitectónicos: FAT vs. FFS

### 3.1 MS-DOS FAT (File Allocation Table)

#### Explicación Conceptual y Funcionamiento
Desarrollado en la década de 1970, el sistema de archivos **FAT** prescinde por completo del concepto de inodos. En su lugar, basa toda la organización de archivos y la administración de espacio libre en una tabla centralizada cargada en el área reservada del disco, denominada **File Allocation Table (FAT)**.

La FAT es una matriz de entradas de tamaño fijo (12, 16 o 32 bits según la variante FAT12, FAT16 o FAT32). Cada entrada en la tabla corresponde directamente a un bloque o *cluster* físico en la región de datos.

```
       DIRECTORIO
+-----------------------+
| Nombre  | Cluster Ini |
+---------+-------------+
| file12  |    12       |
| file9   |     9       |
+---------+-------------+

          TABLA FAT                   REGIÓN DE DATOS
      +---------------+             +-----------------+
  0   |       0       | (Libre)     |  Cluster 0      |
 ...  |      ...      |            ...                |
  9   |      10       | --------->  |  Cluster 9      | (Datos file9)
 10   |      11       | --------->  |  Cluster 10     | (Datos file9)
 11   |      13       | --------->  |  Cluster 11     | (Datos file9)
 12   |      16       | --+         |  Cluster 12     | (Datos file12)
 13   |     EOF       |  |          |  Cluster 13     | (Datos file12)
 ...  |      ...      |  |         ...                |
 16   |     EOF       | <-+-------> |  Cluster 16     | (Datos file12)
      +---------------+             +-----------------+
```

#### Mecánica de Asignación y Rastreo
1. **Representación de Archivos:** La entrada de un archivo en el directorio almacena únicamente su nombre, atributos y el **número del primer cluster**.
2. **Encadenamiento:** Para leer un archivo, el sistema operativo consulta `FAT[cluster_actual]`. El valor almacenado en dicha entrada indica el número del siguiente cluster del archivo. Un valor especial de fin de archivo (`EOF` o `0xFFFFFFFF`) señala el último bloque.
3. **Gestión de Espacio Libre:** Si un cluster $i$ está libre, la entrada `FAT[i]` contiene el valor `0`. El sistema operativo encuentra espacio libre escaneando la tabla en busca de entradas con cero.

#### Desventajas, Fragmentación y Desfragmentación
* **Fragmentación Extrema:** Al asignar bloques según el algoritmo de primer ajuste (*first-fit*) o siguiente ajuste (*next-fit*), los clusters de un archivo terminan esparcidos aleatoriamente por todo el volumen. En discos mecánicos, esto provoca severas penalizaciones de tiempo de búsqueda (*seek time*).
* **Desfragmentación:** Requiere herramientas utilitarias que leen los bloques dispersos de cada archivo y los reescriben de forma contigua en nuevas ubicaciones secuenciales.
* **Punto Único de Falla:** Toda la estructura de índices del volumen se concentra en la tabla FAT. Si los sectores del disco que alojan la FAT se corrompen, el sistema de archivos completo queda inoperable.

---

### 3.2 Unix FFS (Fast File System)

#### Estructura de Árbol Multinivel (Fixed Tree)
El sistema de archivos **FFS** (*Fast File System*) introdujo la estructura de índice multinivel mediante inodos de tamaño fijo, diseñada para optimizar el acceso tanto a archivos pequeños como a archivos de escala de terabytes.

Un inodo tradicional en FFS contiene un arreglo de **15 punteros a bloques**:

```
 INODO DE FFS
+---------------------------+
| Metadatos del Archivo     |
+---------------------------+
| Puntero Directo 0         | ---> Bloque Datos 0
| ...                       |
| Puntero Directo 11        | ---> Bloque Datos 11
+---------------------------+
| Puntero Indirecto Simple  | ---> [Bloque Indirecto: 1024 Punteros Directos]
+---------------------------+
| Puntero Doble Indirecto   | ---> [Bloque Doble Ind.: 1024 Ptr. Ind. Simples]
+---------------------------+
| Puntero Triple Indirecto  | ---> [Bloque Triple Ind.: 1024 Ptr. Doble Ind.]
+---------------------------+
```

1. **12 Punteros Directos (Punteros 0 a 11):** Apuntan directamente a los primeros 12 bloques de datos del archivo. Permiten acceder a archivos pequeños (de hasta $48	ext{ KB}$ con bloques de $4	ext{ KB}$) con cero niveles de indirección.
2. **1 Puntero Indirecto Simple (Puntero 12):** Apunta a un bloque de datos interno que contiene un arreglo de punteros directos. Con bloques de $4	ext{ KB}$ y punteros de $4	ext{ bytes}$ ($32	ext{ bits}$), un bloque indirecto aloja $1024$ punteros, direccionando hasta $4	ext{ MB}$ adicionales.
3. **1 Puntero Doble Indirecto (Puntero 13):** Apunta a un bloque que contiene $1024$ punteros a bloques indirectos simples. Direcciona $1024 	imes 1024 = 1.048.576$ bloques ($4	ext{ GB}$ adicionales).
4. **1 Puntero Triple Indirecto (Puntero 14):** Apunta a un bloque con $1024$ punteros a bloques dobles indirectos, direccionando $1024^3 = 1.073.741.824$ bloques ($4	ext{ TB}$ adicionales).

#### Heurísticas de Localidad y Grupos de Bloques (Block Groups)
Para resolver la lentitud mecánica de las cabezas lectoras de los discos rígidos (los tiempos de búsqueda o *seek times* de $\sim 10	ext{ ms}$), FFS divide el disco físico en **Grupos de Bloques** (*Block Groups* o *Cylinder Groups*).

```
         DISCO FÍSICO DIVIDIDO EN GRUPOS DE BLOQUES
+-----------------------+-----------------------+-----------------------+
|    Block Group 0      |    Block Group 1      |    Block Group 2      |
| +-------------------+ | +-------------------+ | +-------------------+ |
| | Superblock Copia  | | | Superblock Copia  | | | Superblock Copia  | |
| | Bitmaps Locales   | | | Bitmaps Locales   | | | Bitmaps Locales   | |
| | Tabla Inodos Loc. | | | Tabla Inodos Loc. | | | Tabla Inodos Loc. | |
| | Bloques de Datos  | | | Bloques de Datos  | | | Bloques de Datos  | |
| +-------------------+ | +-------------------+ | +-------------------+ |
+-----------------------+-----------------------+-----------------------+
```

Cada Grupo de Bloques mantiene una copia local del superbloque, sus propios bitmaps de inodos y datos, su sección de tabla de inodos y su región de datos. FFS aplica dos heurísticas clave:
1. **Ubicación de Directorios:** Coloca los inodos y bloques de datos de un nuevo directorio en un grupo de bloques con baja ocupación para permitir su expansión local.
2. **Ubicación de Archivos:** Asigna el inodo y los bloques de datos de un archivo dentro del mismo grupo de bloques que aloja a su directorio padre, garantizando que el acceso al directorio y a sus archivos no requiera desplazamientos largos de la cabeza del disco.

---

## 4. Gestión de Performance y Persistencia: Page Cache y Primitivas de Sincronización

### 4.1 Jerarquía de Tiempos de Latencia
El acceso a dispositivos de almacenamiento secundario representa uno de los cuellos de botella más severos en la arquitectura de computadoras.

| Operación | Tiempo típico (ns) | Tiempo equivalente en escala humana |
| :--- | :--- | :--- |
| **Referencia a caché L1** | $0.5	ext{ ns}$ | 1 segundo |
| **Referencia a caché L2** | $7	ext{ ns}$ | 14 segundos |
| **Referencia a Memoria Principal (RAM)** | $100	ext{ ns}$ | 3.3 minutos |
| **Lectura aleatoria 4K en SSD** | $150.000	ext{ ns}$ ($150\ \mu	ext{s}$) | 3.5 días |
| **Búsqueda mecánica en Disco (`Seek`)** | $10.000.000	ext{ ns}$ ($10	ext{ ms}$) | 7.5 meses |
| **Lectura secuencial 1MB desde Disco** | $20.000.000	ext{ ns}$ ($20	ext{ ms}$) | 1.2 años |

Dado que acceder a un disco mecánico es 100.000 veces más lento que acceder a la memoria RAM, los sistemas operativos deben implementar mecanismos agresivos de almacenamiento en caché para mantener la fluidez del sistema.

---

### 4.2 El Buffer Cache / Page Cache del Kernel

El **Page Cache** de Linux es una capa unificada del subsistema de administración de memoria que intercepta todas las operaciones de I/O, manteniendo en RAM páginas de $4	ext{ KB}$ correspondientes a datos e inodos del disco.

Sus dos funciones principales son:
1. **Aceleración de Accesos (Caching):** Servir solicitudes de lectura directamente desde RAM (*Cache Hit*), evitando operaciones físicas de disco.
2. **Sincronización y Exclusión Mutua:** Garantizar que exista una única copia canónica de cada bloque de disco en memoria, coordinando los accesos concurrentes de múltiples hilos mediante cerrojos de página.

```
                  Petición de Lectura / Escritura
                                |
                                v
                     +---------------------+
                     | ¿Está en PageCache? |
                     +---------------------+
                        /                               SÍ     /                 \    NO
                      v                   v
              +---------------+   +-------------------+
              |  CACHE HIT    |   |    CACHE MISS     |
              | (Retorna RAM) |   | (Lectura desde    |
              +---------------+   |  Disco a RAM)     |
                                  +-------------------+
```

#### Aplicación en Bases de Datos (Estructuras B-Tree)
Las bases de datos relacionales (como PostgreSQL o MySQL) organizan sus archivos en disco utilizando árboles **B-Tree** formateados en páginas del tamaño de bloque del sistema (por ejemplo, $8	ext{ KB}$ en PostgreSQL). 

Debido a la alta localidad temporal, los nodos superiores de la jerarquía del árbol B-Tree son accedidos constantemente. El Page Cache los conserva indefinidamente en memoria RAM, permitiendo que la inmensa mayoría de las consultas se resuelvan a velocidad de memoria sin tocar el disco físico.

---

### 4.3 Buffering de I/O y Primitivas de Sincronización

#### Mecánica de Buffering y el Riesgo de Pérdida de Datos
Por defecto, la llamada al sistema `write(fd, buf, count)` aplica **Buffered I/O**: copia los datos desde el espacio de usuario hacia el Page Cache del kernel y retorna inmediatamente al programa **sin esperar a que el hardware escriba en el disco**.

```
 Espacio Usuario   | Espacio Kernel                      | Disco Físico
+----------------+ | +---------------------------------+ | +--------------+
| stdio buffer   | | | Kernel Page Cache               | | |              |
| (printf/fwrite)| | | (páginas "sucias" / dirty pages)| | |              |
+----------------+ | +---------------------------------+ | +--------------+
        |          |                 |                   |        |
   fflush() ------>| sys_write() --->|                   |        |
                   |                 | -- flusher thread |------->|
                   |                 |    (asincrónico)  |        |
                   |                 |                   |        |
                   | sys_fsync() ----|-------------------|------->|
```

Si ocurre un fallo de energía o un cuelgue del sistema (*crash*) antes de que el hilo de escritura asincrónico del kernel (*flusher thread*) vuelque las páginas "sucias" (*dirty pages*) al medio físico, **los datos se pierden permanentemente**.

#### Primitivas de Sincronización Explícita

##### 1. `fsync(int fd)`
Fuerza al kernel a volcar al disco no solo las páginas de datos modificadas del archivo referenciado por `fd`, sino también todos sus metadatos asociados almacenados en el inodo (tamaño, marcas de tiempo de modificación, permisos). Bloquea la ejecución hasta que el controlador de almacenamiento confirma la persistencia física.

##### 2. `fdatasync(int fd)`
Similar a `fsync()`, pero optimiza el rendimiento al volcar **únicamente las páginas de datos del archivo**, omitiendo la sincronización de metadatos del inodo salvo que cambios en el tamaño del archivo sean indispensables para localizar los datos.

##### 3. Mapeo de Memoria con `mmap()` y `msync()`
La llamada al sistema `mmap()` mapea un archivo directamente en el espacio de direcciones virtuales del proceso. El proceso modifica el archivo mediante punteros directos en memoria. Para garantizar la persistencia de los cambios en disco, se debe invocar explícitamente la llamada `msync()`:

```c
#include <sys/mman.h>

// Sincroniza la región mapeada desde 'addr' con el disco
int msync(void *addr, size_t length, int flags);
```
* **`MS_SYNC`:** Sincronización bloqueante. Retorna cuando los datos han sido escritos físicamente en el disco.
* **`MS_ASYNC`:** Solicita la escritura al kernel y retorna inmediatamente.

---

## 5. Consistencia ante Fallos y Recuperación (Crash Consistency & Journaling)

### 5.1 El Problema de la Inconsistencia por Cierres Inesperados
Una operación lógica simple a nivel de usuario (como adjuntar un bloque de datos a un archivo existente) requiere que el sistema de archivos ejecute múltiples escrituras independientes en el disco:
1. **Escritura $A$:** Actualizar el Inodo del archivo (nuevo tamaño y puntero a bloque).
2. **Escritura $B$:** Actualizar el Bitmap de Datos (marcar el nuevo bloque como ocupado).
3. **Escritura $C$:** Escribir los datos del usuario en el bloque asignado.

Si el sistema sufre una interrupción brusca de energía entre la ejecución de estas escrituras, el disco queda en un estado inconsistente. Por ejemplo, si se ejecuta la Escritura $B$ (bitmap actualizado) pero falla la Escritura $A$ (inodo sin actualizar), se produce una fuga de espacio (*space leak*): el bloque figura ocupado en el bitmap pero ningún inodo lo referencia.

---

### 5.2 Comprobación Tradicional (`fsck`) y sus Limitaciones
La solución clásica en sistemas de archivos antiguos consistía en ejecutar la utilidad **`fsck`** (*File System Check*) durante el arranque tras un apagado no correcto.

`fsck` realiza un escaneo exhaustivo de todo el disco:
* Recorre la tabla de inodos y reconstruye los bitmaps de datos e inodos desde cero para detectar discrepancias.
* Verifica que el número de referencias en directorios coincida con el `link_count` de cada inodo.
* Mueve inodos huérfanos sin dentry al directorio `/lost+found`.

**Limitación:** La complejidad de `fsck` escala de forma lineal con el tamaño del volumen. En discos modernos de terabytes o petabytes, una verificación con `fsck` puede demorar horas o días, resultando inaceptable para servidores de producción.

---

### 5.3 Journaling (Write-Ahead Logging - WAL)

#### Estructura y Funcionamiento General
Para evitar escaneos masivos tras un choque, los sistemas de archivos modernos (como ext3, ext4 o NTFS) incorporan **Journaling** (basado en el principio de *Write-Ahead Logging*). Reservan una región contigua del disco dedicada al diario o **Log**.

Antes de modificar las estructuras definitivas del sistema de archivos, el kernel registra la transacción completa en el Journal.

```
       ESTRUCTURA DE UNA TRANSIÓN EN EL JOURNAL
+---------------+-------------------+-------------------+---------------+
| Tx Header     | Metadata / Data   | Metadata / Data   | Tx Commit     |
| (Descriptor)  | Block A           | Block B           | Block         |
+---------------+-------------------+-------------------+---------------+
```

#### Flujo de Operación de una Transacción en Journaling
1. **Journal Write:** El kernel escribe el bloque descriptor de transacción (`Tx Header`) y el contenido de los bloques modificados dentro de la región del diario.
2. **Journal Commit:** Una vez escritas exitosamente las modificaciones en el diario, se escribe el **Bloque de Commit** (`Tx Commit`). Este bloque contiene un número de secuencia y marca el punto donde la transacción se considera atómica e irreversible.
3. **Checkpointing:** El kernel escribe de forma asincrónica las modificaciones desde el diario hacia sus ubicaciones definitivas en el sistema de archivos principal.
4. **Freeing:** Una vez completado el *checkpointing*, la transacción en el diario se marca como libre para ser sobrescrita.

En caso de un choque durante el arranque, el sistema operativo no escanea el disco completo: simplemente lee el diario. Si encuentra una transacción con `Tx Commit` válido pero no reflejada en el filesystem, aplica los bloques (*replay*). Si encuentra una transacción incompleta (sin `Tx Commit`), la descarta por completo.

---

#### Modos de Operación de Journaling

```
MODO PHYSICAL / DATA JOURNALING:
 Write Tx (Data + Metadata) ---> Write Commit ---> Checkpoint (Data + Metadata)

MODO ORDERED METADATA JOURNALING (Por Defecto en ext4):
 Write Data to FS -------------> Write Tx (Metadata) ---> Write Commit ---> Checkpoint (Metadata)
```

| Modo de Journaling | ¿Qué se registra en el Diario? | Garantía de Consistencia y Performance |
| :--- | :--- | :--- |
| **Data Journaling (Physical)** | **Metadatos y Datos** del usuario. | **Máxima seguridad.** Todo pasa dos veces por el disco (primero al diario, luego a ubicación definitiva). **Penalización severa en velocidad de escritura.** |
| **Ordered Journaling (Metadata)** | **Únicamente Metadatos.** | **Modo Estándar.** Garantiza que los **bloques de datos del usuario se escriben en el disco ANTES** de hacer el commit de los metadatos en el diario. Evita la exposición de datos de otros archivos. |
| **Writeback Journaling** | **Únicamente Metadatos.** | **Máxima velocidad.** No impone orden de escritura entre los datos de usuario y el diario. Riesgo de que un archivo apunte a datos antiguos si ocurre un choque durante la operación. |
