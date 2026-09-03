# Qué son?

- Los principios de diseño son una serie de reglas y recomendaciones, que un programador puede elegir seguir durante el desarrollo, para lograr un código mas limpio, comprensible y fácil de mantener.

- No es necesario seguir todos los principios pero se recomienda hacerlo.

- Puede ser difícil generar herramientas que obliguen a seguir los principios

- Se tienen en cuenta desde la primera reunión del proyecto hasta el fin del proyecto


# PoLC: Principio de mínimo compromiso

- Principio de diseño orientado a objetos que recomienda que una interfaz exponga nada más que su comportamiento esencial.
- Nos da mayor modularidad y flexibilidad al sistema. Se reduce la complejidad y aumenta la cohesión al exponer nada mas que los métodos imprescindibles.

# PoLA: Principio de mínimo asombro

- Principio de diseño orientado a objetos que propone que una abstracción capture todo el comportamiento de algún objeto y no contenga sorpresas ni efectos secundarios que vayan más allá del alcance de la abstracción.
- En otras palabras, se busca que el componente se comporte tal como el usuario espera que lo haga.
- Se busca un diseño de software más intuitivo y fácil de comprender
- Evitar problemas de mantenimiento y escalabilidad al reducir la complejidad y garantizar que el comportamiento de una abstracción sea coherente y predecible.

# PoLK: Principio de mínimo conocimiento

- Principio de diseño orientado a objetos que dice que un método *f* de una clase *C* solo debe invocar métodos de:
	- La propia clase C
	- Los objetos son atributos de C
	- Los objetos recibidos por f como argumentos
	- Los objetos instanciados en f
- El objetivo es que el código de un objeto conozca muy poco del código del otro.

# DRY: No te repitas

- Principio de diseño que recomienda la no duplicación en su sentido mas amplio. Esto puede incluir:
	- Código dentro de una clase (o varias)
	- Datos de una base de datos
	- Documentación
- Esto busca un código mas mantenible y escalable