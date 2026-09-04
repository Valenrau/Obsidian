## Qué son las memorias?

Las memorias en el ámbito informático, son componentes físicos que almacenan datos temporales o permanentes para el procesamiento y ejecución de tareas por parte de una computadora. Las memorias permiten que el procesador acceda rápidamente a información necesaria durante el funcionamiento del sistema

### Jerarquías de memoria

Las memorias se suelen representar en un esquema piramidal de jerarquías, siendo de la base las mas lentas pero de mayor capacidad, como los discos rígidos, hasta las memorias mas rápidas pero a su vez de menor capacidad como lo son los registros del procesador.- A su vez dentro de este orden se encuentran dos conceptos clave, la **latencia** (tiempo necesario para que el cpu obtenga un dato de la memoria) y el **ancho de banda** (máxima cantidad de datos que puede ser transferida durante un acceso a memoria)

![[Pasted image 20260725161841.png]]

## RAM y ROM

La memoria principal del sistema se divide, en dos grandes categorías: **RAM** y **ROM**.

La memoria **ROM** (Read Only Memory) es una memoria no volátil, es decir conserva su contenido incluso cuando el sistema se apaga. Se utiliza principalmente para almacenar firmware y rutinas básicas de inicializacion del sistema.

Por otro lado esta la memoria **RAM** (Random Acces Memory) es volátil y se utiliza para almacenar los datos y programas que el procesador esta ejecutando en un momento dado. Dentro de la RAM se distinguen dos tipos principales. La *SRAM* (Static RAM) utiliza biestables para almacenar la información, no requiere refresco y es muy rápida, pero tiene un costo elevado y baja densidad. En cambio la *DRAM* (Dynamic RAM) utiliza capacitares que deben ser refrescados periódicamente, lo que la hace mas lenta, pero permite una mayor densidad de almacenamiento y un costo significativamente menor. Por esta razón, la SRAM se emplea tipicamente en memorias cache mientras que la DRAM se usa como memoria principal.

## Organización Interna de la Memoria

A nivel físico, las memorias se organizan en forma de matrices de celdas, utilizando decodificadores de direcciones para seleccionar filas y columnas. una de las organizaciones mas utilizadas es la llamada *estructura 2-1/2D*, que permite reducir el numero de pines necesarios y optimizar los tiempos de acceso, ya que separa la selección de filas y columnas y utiliza multiplexacion en el tiempo.

Ademas, para mejorar el rendimiento, la memoria principal puede dividirse en varios bancos independientes, dando lugar a la llamada arquitectura multibanco. En este esquema, es posible acceder a distintos bancos en paralelo o de forma entrelazada, de modo que mientras un banco esta ocupado, otro puede estar preparando el siguiente acceso. Esta técnica, conocida como memoria entrelazada, resulta especialmente eficiente en acceso secuenciales y permite aumentar el ancho de banda efectivo del sistema.

## Memoria Caché

La memoria caché es un tipo de memoria de alta velocidad ubicada entre el procesador y la memoria principal. Su función es almacenar copias de los datos e instrucciones que se utilizan con mayor frecuencia, con el objetivo de reducir el tiempo medio de acceso a memoria y, en consecuencia mejorar el rendimiento del sistema.

En los sistemas modernos, la caché se organiza en 3 niveles. La caché **L1** es la mas pequeña pero al estar en el núcleo del procesador es extremadamente rápida. La caché **L2** es un poco mas grande y en consecuencia levemente mas lenta. Finalmente la Caché **L3** es el nivel mas grande de estos, siendo proporcional con el aumento de tamaño y la reducción de velocidad.

El funcionamiento de la caché se basa en distintos esquemas de mapeo entre la memoria principal y las lineas de caché. En la asignación directa, cada bloque de memoria principal solo puede ubicarse en una única linea de caché, lo que simplifica el hardware pero puede generar conflictos frecuentes. En la asignación totalmente asociativa, cualquier bloque puede ubicarse en cualquier linea, lo que ofrece mayor flexibilidad a costa de una mayor complejidad. Entre ambos extremos se encuentra la asignación asociativa por conjuntos, que divide la caché en conjuntos y permite que cada bloque se ubique en cualquiera de las lineas de su conjunto correspondiente, logrando un equilibrio entre simplicidad y rendimiento.

Además del mapeo, la caché utiliza políticas de escritura. En la política *write-through*, cada escritura se realiza tanto en la cache como en la memoria principal, lo que simplifica la coherencia de datos pero incrementa el trafico de memoria. En la política *write-back*, las modificaciones se realizan primero en la cache y recién se escriben en la memoria principal cuando la memoria es reemplazada, reduciendo el trafico pero aumentando la complejidad del control.

El rendimiento de la cache se evalua mediante la tasa de aciertos y el tiempo efectivo de acceso (TEA), que combina los tiempos de acceso a cache y a memoria principal ponderados por la probabilidad de aciertos y fallos. Una caché eficiente debe minimizar la cantidad de fallos, ya que los accesos a memoria principal tienen un costo muy superior

## Memoria Virtual

La **memoria virtual** es una técnica que permite a los programas utilizar un espacio de direcciones mayor que la memoria física disponible, apoyándose en el almacenamiento secundario como respaldo. Para ello, el sistema divide la memoria en páginas virtuales y marcos físicos de igual tamaño, y mantiene una tabla de paginas que traduce las direcciones virtuales en direcciones físicas.

Cuando un programa intenta acceder a una pagina que no se encuentra en memoria principal, se produce una falla de pagina. En ese caso, el sistema operativo selecciona un marco de memoria, guarda su contenido en disco si fue modificado, carga la pagina requerida desde el almacenamiento secundario y actualiza la tabla de paginas antes de reanudar la ejecución del programa.

Otra técnica relacionada es las segmentación, que divide el espacio de direcciones en segmentos lógicos, como código, datos o pila. A diferencia de la paginación, los segmentos pueden tener tamaños variables y permiten una gestión mas flexible de la protección y los permisos de acceso, aunque requieran una administración mas compleja por parte del sistema operativo.

## TLB (Translation Lookaside Buffer)

En un sistema con memoria virtual, cada acceso a memoria implica, en principio, consultar la tabla de paginas y luego acceder a la memoria física, lo que introducir una sobrecarga significativa. Para reducir este costo, se utiliza el TLB, que es una pequeña memoria asociativa que almacena un subconjunto de las traducciones mas recientes de la tabla de paginas.

Cuando el procesador genera una dirección virtual, primero consulta el TLB si la traducción se encuentra allí (*TLB hit*), se obtiene directamente la dirección física. Si no la encuentra (*TLB miss*), se debe consultare la tabla de paginas en memoria principal, actualizar el TLB y luego continuar con el acceso. El diseño del TLB puede ser unificado o dividido entre instrucciones y datos, y puede implementarse como totalmente asociativo o asociativo por conjuntos, buscando siempre un equilibrio entre velocidad, complejidad y costo.

El impacto del TLB en el rendimiento es considerable, ya que una alta tasa de aciertos reduce de forma notable el tiempo efectivo de acceso a memoria.

## Buffering e Interrupciones

El buffering es una técnica que utiliza ares de memoria intermedia para desacoplar dispositivos o procesos que operan a distintas velocidades. Los buffers permiten evitar perdidas de datos y mejorar la eficiencia de sistemas de entrada/salida. Existen diversas variantes, como el uso de un solo buffer, doble buffering o buffers circulares, muy comunes en aplicaciones de streaming y procesamiento en tiempo real.

Por otro lado, las interrupciones y traps son mecánicos fundamentales para la interacción entre el hardware, el sistema operativo y los programas. Las interrupciones son generadas por dispositivos externos y ocurren de manera asíncrona, mientras que las traps suelen ser generadas por el propio procesador ante situaciones como divisiones por cero o accesos inválidos a memoria. En ambos casos, el sistema guarda el contexto del programa en ejecución y transfiere el control a una rutina de servicio apropiada.

## Organización de Memoria en Procesadores Multicore

En los sistemas multicore, la organización de la memoria adquiere una importancia aun mayor. Existen dos modelos principales: el modelo **UMA**, en el cual todos los núcleos acceden a la memoria principal con la misma latencia, y el modelo **NUMA**, donde la memoria se divide en regiones con distintas latencias según la cercana al núcleo que realiza el acceso.

Estos sistemas suelen combinar memoria compartida con esquemas de cache multinivel para reducir la latencia y mejorar la escalabilidad, aunque esto introduce nuevos desafíos relacionados con la coherencia de cache y la sincronización entre núcleos

## Taxonomía de Flynn

La taxonomía de Flynn clasifica las arquitecturas de computadores según la cantidad de flujos de instrucciones y de datos que se pueden manejar. En el modelo **SISD**, un único flujo de instrucciones opera sobre un único flujo de datos, como los procesadores secuenciales clásicos. En **SIMD**, una misma instrucción se aplica en paralelo sobre múltiples datos, como ocurre en procesadores vectoriales o GPUs. El modelo **MISD**, menos común, utiliza múltiples instrucciones sobre un único flujo de datos. Finalmente, el modelo **MIMD**, característico de los sistemas multiprocesador y multicore modernos, permite que múltiples procesadores ejecuten distintas instrucciones sobre distintos datos en paralelo

