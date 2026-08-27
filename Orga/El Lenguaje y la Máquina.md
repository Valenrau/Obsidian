
### Niveles de abstracción en una computadora:

1. **Operación:** Lo que el usuario ve/hace.
2. **Lenguaje de Alto Nivel:** Código legible por humanos (C, C++, etc.).
3. **Assembler:** Representación simbólica del código de máquina.
4. **Código de Máquina:** Ceros y unos.
5. **Procesador:** Hardware que interpreta el código.
6. **Compuertas / ALU:** Lógica digital.
7. **Transistores / Física:** Componentes electrónicos básicos.

### Del diseño a la Ejecución

Se describe el flujo completo ("Toolchain") para transformar código fuente en un programa en ejecución:

1. **Compilación:** Transforma el Lenguaje de Alto Nivel a Assembler
2. **Ensamblado:** Transforma el Assembler a Código de Máquina (creando un archivo objeto)
3. **Linking (Enlazado):** Combina varios módulos objeto (Módulo 1, ..., Módulo n) en un solo código ejecutable
4. **Loading (Carga)**: Carga el ejecutable desde el disco a la memoria RAM para convertirse en un "Programa en ejecución".

![[Pasted image 20260720214254.png]]

## Compiladores

El compilador es el encargado de traducir el código fuente. 

### Tipos de compiladores

- **De una sola pasada o múltiples pasadas:** Según cuantas veces recorra el código fuente
- **Incremental:** Genera código instrucción por instrucción 
- **Cross-compilador:** Genera código para una arquitectura de máquina distinta a la que se esta usando para compilar

### Proceso de compilación 
#### Front End (Independiente del procesador):

1. **Análisis Léxico:** Distingue palabras clave e identificadores
2. **Análisis Sintáctico:** Detecta la estructura y verifica que cumpla las reglas del lenguaje
3. **Análisis Semántico:** Verifica la lógica, como la congruencia de tipos de expresiones y asocia atributos (tipo, ámbito) a las variables en la **Tabla de Símbolos**
4. **Manejo de Errores:** Reporta fallos en cualquiera de las etapas anteriores

#### Back End (Dependiente del procesador)

1. **Mapeo de Acciones:** Asocia sentencias del programa con estructuras lógicas equivalentes en bajo nivel
2. **Generación de Código:** Crea el código Assembler final, decidiendo la localización de las variables y el uso de registros

### Mapeo de Acciones

El compilador debe traducir construcciones de alto nivel a instrucciones simples (aritméticas, control de flujo, movimiento de datos, etc.)

- **Asignación (`A=B+4`):** En una maquina [[Arquitectura de Programación y Set de Instrucciones#RISC vs CISC|RISC]], esto implica cargar `B` a un registro (`ld`), sumar la constante (`add`), y guardar el resultado en `A`(`st`). El compilador debe usar técnicas sofisticadas para gestionar la escasez de registros.
- **Control de Flujo:**
    - `Goto`→ `ba` (Branch Always)
    - `If/Else` → Se traduce usando restas de comparación (`subcc`) seguidas de saltos condicionales (`bne`, `be`) y etiquetas (`labels`) para saltar el código que no debe ejecutarse.
    - `While / Do-While` → Combinaciones de etiquetas, operaciones del cuerpo del ciclo, comparaciones y saltos condicionales para repetir o salir.
- **Almacenamiento de las Variables:** Donde almacenar variables en la RAM
	- *Globales* (Estáticas): Posición de memoria fija conocida al compilar.
	- *Locales*: Se guardan en el **Stack** (desaparecen al terminar la función)
	- *Registros*: Es el almacenamiento optimo, pero limitado.

## Ensambladores

Realiza una **transcodificación** (relación 1 a 1) entre el código Assembler y código de máquina. Le ofrece al programador:
- Representación simbólica para direcciones y constantes
- Definir la ubicación de las variables en memoria
- Variables inicializadas antes de ejecución
- Provee cierto grado de aritmética en tiempo de ensamblado
- Utilizar variables declaradas en otros módulos
- Utilizar macros

### Directivas

Son instrucciones para el propio programa ensamblador, no para el CPU, algunas generan información en memoria, otras no. [[Arquitectura de Programación y Set de Instrucciones#Instrucciones y Directivas de ARC|Ejemplos Clave]]

#### Proceso de Ensamblado de Dos Pasadas

Para resolver referencias a etiquetas que aún no han aparecido en el código (referencias hacia adelante), el ensamblador opera en dos vueltas:

- **Primera Pasada:**
	1. Expande macros.
	2. Detecta identificadores (etiquetas, variables) y les asigna un una dirección en memoria basada en un "contador de posición".
	3. Construye la **Tabla de Símbolos** (Nombre -> Valor/Dirección).
- **Segunda Pasada:**
	1. Convierte cada instrucción a su binario.
	2. Reemplaza los identificadores por sus direcciones reales consultando la tabla de Símbolos creada en la primera pasada.
	3. Genera el **Archivo de Listado** (texto) y el **Archivo de Código Objeto** (binario)

## Linking y Archivos Objeto

El **Linker** combina múltiples módulos objeto (de la aplicación, librerías o el SO).

- **Archivos Objeto:** Contienen código maquina, encabezados con la dirección de inicio (`main`) y tablas de símbolos con información sobre símbolos **Globales** (definidos aquí, visibles fuera) y **Externos** (usados aquí, definidos fuera)
- **Símbolos Relocalizables:** El ensamblador marca que direcciones son relativas (como un salto a una etiqueta local) y cuales son absolutas (como una dirección de E/S fija). El linker usa esta información para reasignar direcciones cuando combina módulos, ya que el modulo 2 no empezara en la dirección 0, sino donde termine el modulo 1.

### Loading (Carga)

El **Loader** lleva el programa de disco a RAM. En sistemas multitudinaria, la dirección final de carga no se conoce hasta el tiempo de ejecución. 

Tipos de carga/enlazado:

- **Link-editor:** Genera un archivo **`.exe`** estático en disco.
- **Relocating loader:** Carga y re localiza direcciones en tiempo de carga.
- **Linking loader:** Enlaza librerías y carga en un solo paso.
- **Dynamic Linking (DLL):** Enlaza en **tiempo de ejecución**. Carga rutinas en memoria solo cuando el programa las necesita

![[Pasted image 20260722013210.png]]


## Eficiencia del Código en Alto Nivel

Aunque el compilador optimiza gran parte del código, ciertas decisiones del programador influyen directamente en la calidad del código máquina generado. Elegir adecuadamente constantes, tipos de datos o la forma de acceder a la memoria puede reducir la cantidad de instrucciones ejecutadas y mejorar el rendimiento.

- **Constantes vs Variables:** Usar `#define` (constantes) permite al compilador usar instrucciones con "inmediatos" (`add ... 1000`), ahorrando accesos a memoria. Usar variables obliga a instrucciones de carga (`ld`) y aritmética de registros, lo cual es mas lento.
- **Tipos de Datos:**
	- En arquitecturas de 32 bits, usar tipos pequeños como `char` (byte) puede ser **menos eficiente** que usar `long` o `int` (palabra completa), porque el procesador debe agrear instrucciones extra para enmascarar bits o extender el signo (`ldsb` vs `ld` )
	- El punto flotante (`floats`, `double`) requiere coprocesadores o instrucciones complejas
- **Arreglos:** El acceso a arreglos requiere calcular el offset. El uso de punteros o indices eficientes es critico.
- **Llamadas a Funciones:** No son gratuitas. implican una sobrecarga ("overhead") significativa: backup de dirección de retorno, pasar parámetros (copia o registros), salvar registros actuales, saltar, ejecutar, restaurar registros y retornar.
- **Orden de Declaración:** Los compiladores suelen asignar registros (el recurso mas rápido) a las primeras variables declaradas. Se recomienda declarar primero las variables de mayor uso.