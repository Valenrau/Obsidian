# Diagramas

### Qué es un diagrama?

Son herramientas graficas que ayudan a visualizar procesos, sistemas y estructuras que se dan en un contexto

Su objetivo es el de identificar, clarificar y reducir la ambiguedad

## Diagramas en el desarrollo de software

En el desarollo de software es basto el uso de diversos diagramas para representar distintas cosas, pero en general hay diagramas para cosas en especifico, no suele haber diagramas que abarquen exactamente todos porque cada uno tiene su complejidad y manera de ser explicado

# UML: Unified Modeling Language

Simplifica el complejo proceso de diseñar software, actuando como lenguaje intermediario entre los distintos contribuyentes del proyecto 
(Basicamente es un conjunto de diagramas)

### Diagramas Comunes

- **Diagramas de [[Programación Orientada a Objetos#Objeto|clase]]**:
- **Diagramas de secuencia**:

# Diagramas de Clase

Sirve para representar distintas entidades y sus relaciones estáticas (no cambian a traves del tiempo).

Su simbologia no hace referencia a ningun lenguaje en específico, por lo que su sintaxis deberia serlo tambien.

- No es necesario representar todas las claes con todos sus detalles
- Son herramientas, por lo que el grado de detalle hay que manejarlo desde el punto de vista de la practicidad

## Simbolos

Clases:
- Se representan con cajas rectangulares
- Constan de 3 secciones:
	- Nombre de la clase (en CammelCase)
	- Atributos
	- Métodos
- Los atributos y metodos pueden tener distintas visibilidad
	- Públicos (+)
	- Privados (-)
	- Protegidos (#)
- Pueden especificarse metodos y atributros de Clase en lugar de instancia (con subrayado)
- Las clases y métodos abstractos llevan nombres en itálicas

Interfaces:
- Se representan con cajas rectangulares
- Constan de 2 secciones:
	- Nombre de la interfaz (y la aclaracion de lo que es)
	- Métodos

Relaciones:
- Se representan con flechas de distintas caracteristicas
- Los significados son:
	- **Asociacion**: representa una relacion de conocimiento o uso entre un objeto y otro (User -> Utility)
	- **[[Programación Orientada a Objetos#Herencia|Herencia]]**: una clase hereda de a quien apunta
	- **Implementación**: una entidad implementa una interfaz a la que apunta
	- **Agregación**: se compone de objetos con otros ciclos de vida
	- **Composición**: se compone de objetos con mismo ciclo de vida
	- **Dependencia**: la clase usa ub objeto que si cambia, altera el comportamiento de ella

### Cardinalidad

Define el número de instancias de una clase que se pueden asociar con una instancia de otra clase (se leen al reves)
Facultad (1)------------(1...\*) Aula, se lee como que aula puede tener una facultad y una facultad puede tener 1  muchas aulas
- 1: Exactamente 1
- 0..1: Cero o uno
- \* (o 0..\*): Cero o muchos
- 1..\*: Uno o muchos

## Relaciones entre Objetos:

### Asociación

- Representa una relacion de conocimiento o uso entre ub objeto y otro
- Cada objeto tiene su propio ciclo de vida
- Es un vinculo débil

### Agregación 

- Representa una relación de "tiene un"
- Cada objeto tiene su propio ciclo de vida
- Es un vínculo un poco mas fuerte que el de Asociación
- Ej: Curso "tiene una" lista de alumnos y el alumno en cuestión
- El alumno no está atado al curso, el curso puede "morir" pero alumno puede estar en otros cursos tambien

### Composición

- Representa una relacion de "tiene un" pero donde la "parte" depende enteramente del "todo"
- Basicamente B está  en A, si A muere B tambien

### Herencia

- Representa una relacion de "es un"
- Permite derivar de una clase mas específica a una más general
- Ej: Un perro hereda de Mamifero

### Implementación

- Una Clase "implementa" una interfaz cuando cuenta con todos los métodos de la interfaz
- Permite agrupar y exponer comportamiento común entre distintas clases
- Ej: un auto implementa la interfaz de vehiculo

# Diagramas de secuencia

Herramienta gráfica que nos permite diseccionar un proceso y los objetos relacionados, viendo que accion realiza cada actor y como es la comunicación netre ellos.

Es util para detallar procedimientos difici de relatar con palabras

## Elementos

- Objetos: cada paricipante del proceso se rpresentara con un rectangulo en la parte superior del diagrama.
	- El rectanguo lleva el nombre del objeto
	- Todos los objetos estan alineados
- Linea de vida: Los objetos cuentan con una línea temporal, sobre la que se iran acumulando las acciones que realizará en orden en que aparecen sobre el eje vertical
- Acción: cuando un objeto realiza una accion, se diagrama con un rectángulo sobre su linea de vida.
	- El rectangulo tendra comienzo desde que se invoa la accion hasta que terminasu procesamiento y todas las acciones que esta invoque (mediante mensajes)
	- Pueden haber varias acciones de un objeto en ejecucion en un mismo momento.
- Mensajes/Eventos: Se representan con **flechas** que van de la linea de vida de un objeto a otro, dando inicio a una acción en el receptor
	- Las flechas contienen el nombre del mensaje (el metodo junto a sus parametros)
	- Los mensajes pueden ser dirigidos al mismo objeto
- Fragments: Loops, breaks o condicionales