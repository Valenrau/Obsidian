## ¿Qué son los principios de diseño?

Son reglas y recomendaciones que ayudan a tomar decisiones durante el desarrollo de software para lograr código más:

- limpio;
    
- comprensible;
    
- mantenible.
    

No son reglas absolutas: funcionan como una **guía** y deben evaluarse según el contexto.---

# Principios de diseño

## TDA — Tell, Don't Ask

**Decí, no preguntes.**

En programación orientada a objetos, se recomienda **pedirle directamente a un objeto que realice una acción**, en lugar de consultar su estado para luego decidir externamente qué hacer.

### Idea

En vez de:

```
if (cuenta.getSaldo() >= monto) {
    cuenta.setSaldo(cuenta.getSaldo() - monto);
}
```

preferir:

```
cuenta.retirar(monto);
```

### ¿Por qué?

Porque la lógica relacionada con el estado de `Cuenta` queda dentro de `Cuenta`.

**Idea clave:** interactuar con los objetos mediante su **comportamiento**, no manipulando directamente su estado.

---

## PoLC — Principle of Least Commitment

**Principio de mínimo compromiso.**

Una interfaz debería exponer solamente el **comportamiento esencial** que necesitan sus usuarios.

### Ejemplo

Si una `Cartulina` puede:

- pintar;
    
- doblar;
    
- reciclar;
    

una interfaz `Reciclable` debería exponer solamente:

```
reciclar();
```

Entonces un reciclador puede trabajar con:

```
Reciclable
```

en lugar de depender directamente de:

```
Cartulina
```

### ¿Por qué?

Reduce dependencias innecesarias y aumenta:

- modularidad;
    
- flexibilidad;
    
- cohesión.
    

---

## PoLA — Principle of Least Astonishment

**Principio de mínimo asombro.**

Un componente debería comportarse de la manera que el usuario del código **razonablemente espera**.

No debería producir efectos secundarios sorprendentes ni hacer cosas que no correspondan a su abstracción.

### Ejemplo

Si existe:

```
lista.ordenar();
```

no debería, inesperadamente, también guardar la lista en una base de datos.

### Idea clave

> El comportamiento debe ser coherente, intuitivo y predecible.

---

## PoLK — Principle of Least Knowledge

**Principio de mínimo conocimiento.**

Un objeto debería conocer lo menos posible sobre la estructura interna de otros objetos.

Según la presentación, un método debería invocar métodos solamente de:

- su propia clase;
    
- sus atributos;
    
- los objetos recibidos como parámetros;
    
- los objetos creados dentro del propio método.
    

### Ejemplo problemático

```
pedido.getCliente().getDireccion().getCodigoPostal();
```

El código necesita conocer demasiada estructura interna de `Pedido`, `Cliente` y `Direccion`.

### Idea clave

Evitar navegar innecesariamente por cadenas de objetos.

Esto reduce el **acoplamiento**.

---

## DRY — Don't Repeat Yourself

**No te repitas.**

Evitar duplicar información.

La duplicación puede aparecer en:

- código;
    
- datos;
    
- documentación.
    

### Ejemplo

Si el mismo cálculo aparece en varios métodos:

```
precio * 1.21
```

podría centralizarse en una abstracción apropiada.

### Idea clave

> Una misma idea debería tener, idealmente, una única fuente de verdad.

Esto mejora la mantenibilidad: si la lógica cambia, se modifica en un solo lugar.

---

## YAGNI — You Aren't Gonna Need It

**No lo vas a necesitar.**

No agregar funcionalidades, métodos, módulos o abstracciones **anticipadamente**.

### Ejemplo

No crear:

```
exportarExcel();
exportarPDF();
exportarCSV();
exportarXML();
```

si actualmente el sistema solamente necesita exportar CSV.

### Idea clave

No significa eliminar flexibilidad o escalabilidad.

Significa:

> No resolver problemas que todavía no existen.

El código innecesario aumenta:

- complejidad;
    
- tiempo de desarrollo;
    
- mantenimiento;
    
- posibilidades de errores.
    

---

## KISS — Keep It Simple, Stupid

**Mantenelo simple.**

Los sistemas suelen ser más mantenibles y comprensibles cuando se evita la complejidad innecesaria.

### Evitar

- _spaghetti code_;
    
- funciones enormes;
    
- demasiadas clases;
    
- demasiados módulos o librerías;
    
- soluciones "elegantes" pero difíciles de entender.
    

### Ejemplo

Una solución de 5 líneas claras suele ser preferible a una solución extremadamente abstracta que requiere entender 8 clases para realizar una tarea simple.

### Idea clave

> La complejidad debe justificarse.

---

## EDP — Explicit Dependencies Principle

**Principio de dependencias explícitas.**

Una clase o método debería declarar explícitamente los objetos que necesita para funcionar.

Las dependencias pueden expresarse mediante:

- parámetros;
    
- constructor.
    

### Ejemplo

```
public ServicioUsuario(RepositorioUsuarios repositorio) {
    this.repositorio = repositorio;
}
```

es preferible a que `ServicioUsuario` cree internamente un repositorio ocultando su dependencia.

### ¿Por qué?

Las dependencias explícitas hacen que sea más fácil:

- comprender la clase;
    
- modificarla;
    
- probarla.
    

---

## KOP — Knuth's Optimization Principle

**Principio de optimización de Knuth.**

No optimizar prematuramente.

Primero se debe conseguir un programa que:

1. funcione correctamente;
    
2. luego, si es necesario, identificar qué partes realmente necesitan optimización.
    

### Ejemplo

No reemplazar código claro por código extremadamente complejo solamente porque _podría_ ser más rápido, sin haber comprobado que exista un problema real de rendimiento.

### Idea clave

> Optimizar en el momento adecuado y en los lugares que realmente lo requieren.

---

## SoC — Separation of Concerns

**Separación de incumbencias.**

Separar el sistema en módulos donde cada uno tenga una responsabilidad o incumbencia diferente.

### Ejemplos de incumbencias

- lógica de negocio;
    
- interfaz visual;
    
- acceso a datos;
    
- conexión con servicios externos.
    

### Ejemplo

Una clase que calcula una compra no debería también:

- mostrar botones;
    
- conectarse directamente a la base de datos;
    
- enviar emails.
    

### Idea clave

Cambiar una parte del sistema debería afectar lo menos posible a las demás.

Esto mejora:

- modularidad;
    
- mantenibilidad;
    
- separación del trabajo.
    

---

# Buenas prácticas

## Legibilidad

El código se escribe una vez, pero se lee muchas veces.

Por eso hay que escribir pensando en la persona que tendrá que leerlo después.

> Código que funciona pero nadie entiende también es difícil de mantener.

---

## Estandarizar

Dentro de un proyecto se deberían utilizar convenciones consistentes para:

- nombres;
    
- formato;
    
- comentarios;
    
- documentación.
    

Siempre que sea posible, conviene seguir los estándares del lenguaje utilizado.

### Ejemplo

No mezclar:

```
obtenerUsuario()
```

con:

```
Get_User()
```

dentro del mismo proyecto.

---

## "Nadie es el Messi de la programación"

No escribir código para demostrar que uno es inteligente.

Evitar código excesivamente ingenioso o comprimido si perjudica la comprensión.

### Ejemplo

Esto puede funcionar:

```
return x > 0 ? x * 2 : x == 0 ? 0 : -x;
```

pero dependiendo del contexto podría ser más legible expresar la lógica de forma más clara.

También hay que evitar comentar lo obvio.

---

# Nombres

Nombrar correctamente es importante porque los nombres ayudan a que el código explique qué está haciendo.

Si es difícil encontrar un buen nombre para una entidad, puede ser una señal de que esa entidad no está bien definida.

## Recomendaciones

Los nombres deberían ser:

- intuitivos;
    
- legibles;
    
- consistentes en un idioma;
    
- descriptivos;
    
- relacionados con lo que representan, no solamente con su tipo.
    

### Ejemplos

Preferir:

```
usuarios
```

antes que:

```
arregloUsuarios
```

Preferir:

```
pesoEnKg
```

antes que:

```
peso
```

Los nombres cortos son aceptables en casos claros y convencionales:

```
i
max
min
cant
```

---

## Nombres de funciones

El nombre debería describir **qué hace** la función, no cómo está implementada.

Debe ser:

- claro;
    
- completo;
    
- conciso.
    

### Ejemplo

En un módulo de acceso a usuarios:

```
obtener()
```

puede ser suficiente si el contexto ya deja claro qué se obtiene.

Evitar nombres confusos o abreviaturas innecesarias:

```
obtUsr()
getUsrDB()
```

---

# Métodos y legibilidad

## Evitar dependencias innecesarias del orden de ejecución

Idealmente, los métodos deberían poder utilizarse sin requerir recordar un orden especial.

Si existe un orden obligatorio, debería estar claramente documentado.

---

## Recursividad

Usarla cuando realmente mejora la legibilidad.

La presentación también distingue la recursividad directa de ciclos indirectos entre métodos.

Ejemplo de ciclo indirecto problemático:

```
A → B → A
```

---

# Parámetros

Se recomienda:

- tener pocos parámetros;
    
- mantener un criterio de orden consistente;
    
- validar los parámetros al comienzo del método;
    
- no modificar parámetros salvo que el método lo indique explícitamente.
    

### Ejemplo

Una función de ordenamiento _in-place_ sí puede modificar su argumento, porque forma parte explícita de su comportamiento.

---

# Comentarios

Los comentarios pueden servir para:

- aclarar código poco intuitivo;
    
- explicar una decisión controversial;
    
- documentar la fuente de un algoritmo.
    

Sin embargo, el objetivo debería ser que el código sea lo suficientemente claro como para no necesitar demasiados comentarios.

### Idea clave

> Un comentario debería aportar información que el código no pueda expresar claramente.

---

# Bad smells / malas prácticas

Un _bad smell_ es una señal de que puede existir un problema de diseño o una oportunidad de mejorar el código.

No siempre significa que el código esté necesariamente mal, pero merece atención.

---

## Romper el anonimato de los objetos

Si el modelo está bien diseñado, normalmente no debería ser necesario preguntar constantemente de qué clase es un objeto.

Evitar, cuando sea posible:

```
objeto instanceof Clase
objeto.getClass()
```

También puede ser una señal de mal diseño crear métodos como:

```
getTipoDeObjeto()
```

solamente para decidir externamente cómo tratar al objeto.

### Idea

Muchas veces el **polimorfismo** permite delegar el comportamiento correcto al propio objeto.

---

## Acceder al estado interno de otro objeto

Un objeto no debería necesitar conocer directamente los detalles internos de otro.

En lugar de manipular atributos ajenos, debería interactuar mediante comportamiento y mensajes.

### Ejemplo conceptual

En vez de:

```
cuenta.saldo -= monto;
```

preferir:

```
cuenta.retirar(monto);
```

---

## Números mágicos

Evitar números _hardcodeados_ cuyo significado no sea evidente.

### Ejemplo

En vez de:

```
if (edad >= 18)
```

dependiendo del contexto, puede ser más claro:

```
if (edad >= EDAD_MINIMA)
```

Esto hace explícito el significado del valor.

La presentación admite casos triviales como `0`, `1` o `null`.

---

# Comentarios problemáticos

## Información inapropiada

Los comentarios deberían referirse al código cercano.

Información sobre:

- bugs;
    
- deuda técnica;
    
- tareas futuras;
    

debería estar en herramientas o documentación apropiadas.

---

## Comentarios obsoletos

Un comentario viejo, incorrecto o irrelevante es peligroso porque puede llevar al lector a entender mal el código.

Hay que actualizarlo o eliminarlo.

---

## Comentarios redundantes

Evitar comentarios que simplemente repiten lo que ya dice el código.

Ejemplo:

```
// Incrementa contador
contador++;
```

El comentario no aporta información nueva.

---

## Código comentado

No dejar código viejo comentado "por si acaso".

Problemas:

- dificulta la lectura;
    
- puede quedar desactualizado;
    
- nadie sabe si todavía es relevante.
    

### Idea clave

> Para conservar versiones anteriores existe Git.

---

# Funciones

## Demasiados argumentos

Cuantos menos parámetros, mejor.

Según la presentación, más de tres parámetros ya es cuestionable.

Muchos argumentos pueden indicar que:

- la función hace demasiado;
    
- existe una abstracción faltante.
    

---

## Parámetros Flag

Un parámetro booleano puede indicar que una función realiza más de una cosa.

Ejemplo:

```
procesar(usuario, true);
```

El comportamiento depende de una bandera y puede resultar ambiguo.

Puede ser preferible separar comportamientos:

```
procesarNormal(usuario);
procesarUrgente(usuario);
```

cuando conceptualmente son operaciones diferentes.

---

## Funciones muertas

Un método que nunca es llamado probablemente debería eliminarse.

Mantener código sin uso:

- aumenta la complejidad;
    
- confunde;
    
- genera mantenimiento innecesario.
    

---

# Problemas generales

## Múltiples lenguajes en un archivo

Los lenguajes modernos permiten mezclar distintos formatos, por ejemplo:

- Java;
    
- XML;
    
- JSON;
    
- YAML.
    

La presentación recomienda evitarlo cuando sea posible y mantener una separación clara.

---

## Comportamiento obvio no implementado

Una clase o función debería hacer lo que su nombre y abstracción hacen esperar.

Cuando el comportamiento es inesperado, el lector deja de confiar en el código y necesita inspeccionar toda la implementación.

Está relacionado con **PoLA**.

---

## Saltear medidas de seguridad

No ignorar mecanismos de seguridad o advertencias sin una razón muy justificada.

Ejemplos:

- _warnings_ del compilador;
    
- pruebas.
    

---

## Duplicación

Código duplicado puede indicar una oportunidad de abstracción.

Está directamente relacionado con **DRY**.

---

## Demasiada información

Un módulo bien definido debería tener una interfaz relativamente pequeña que permita hacer mucho.

Evitar exponer innecesariamente:

- datos internos;
    
- funciones auxiliares;
    
- constantes;
    
- métodos que no forman parte del comportamiento esencial.
    

Está relacionado con:

- PoLC;
    
- encapsulamiento;
    
- bajo acoplamiento.
    

---

## Código muerto

Además de métodos sin uso, puede incluir:

- `if` con condiciones que nunca ocurren;
    
- `catch` que nunca se ejecutan;
    
- casos de un `switch` imposibles.
    

El código muerto suele quedar desactualizado con el tiempo.

---

## Separación vertical

Las variables y funciones deberían estar definidas cerca de donde se utilizan, en términos de la estructura del código.

Esto facilita seguir el flujo y encontrar la información necesaria.

---

## Inconsistencia

Elegir convenciones y mantenerlas.

La inconsistencia obliga al lector a aprender nuevas reglas constantemente.

Está relacionada con el principio de mínimo asombro.

---

## Acoplamiento artificial

No relacionar artificialmente conceptos que no pertenecen juntos.

Hay que pensar cuidadosamente dónde colocar:

- funciones;
    
- constantes;
    
- enumeraciones;
    
- otras entidades.
    

Ejemplo: un `enum` general no debería estar dentro de una clase específica solamente por comodidad.

---

## Feature Envy — Envidia de la funcionalidad

Un método debería interesarse principalmente por los datos y comportamiento de su propia clase.

Hay _feature envy_ cuando un método utiliza constantemente `getters` y `setters` de otro objeto para realizar una lógica que probablemente debería estar en ese otro objeto.

### Ejemplo conceptual

Si `CalculadoraDePago` extrae constantemente datos de `Empleado`:

```
empleado.getHoras();
empleado.getTarifa();
empleado.getHorasExtra();
```

puede ser una señal de que parte de esa lógica debería pertenecer a `Empleado`.

---

## Obscured Intent — Intención oculta

El código debería expresar claramente qué está haciendo.

Evitar:

- nombres crípticos;
    
- números mágicos;
    
- líneas excesivamente complejas;
    
- formato difícil de leer.
    

### Ejemplo

Preferir:

```
horasExtra * tarifaHora
```

antes que:

```
h * r
```

cuando el contexto no hace evidente el significado.

---

## Static inapropiado

Los métodos estáticos son útiles cuando el comportamiento depende únicamente de sus parámetros y no de un estado.

Sin embargo, el abuso de `static` puede reducir la posibilidad de utilizar polimorfismo.

### Ejemplo conceptual

Una función estática fija una implementación.

Una abstracción con métodos de instancia puede permitir distintas implementaciones mediante polimorfismo.

---

## Preferir polimorfismo

Cuando existen distintos comportamientos según el tipo de objeto, muchas veces conviene utilizar polimorfismo en lugar de grandes cadenas de:

```
if / else
```

o:

```
switch / case
```

### Ejemplo

En vez de:

```
if (figura instanceof Circulo) {
    ...
} else if (figura instanceof Rectangulo) {
    ...
}
```

cada figura puede implementar:

```
figura.calcularArea();
```

---

## Seguir convenciones estándar

Usar convenciones conocidas del lenguaje reduce la necesidad de documentación adicional.

El lector puede aprovechar conocimientos que ya posee.

---

## Precisión

Elegir cuidadosamente:

- tipos;
    
- visibilidad;
    
- abstracciones.
    

### Ejemplos

- No usar `float` para representar precios.
    
- Si solamente se necesita una `List`, no depender innecesariamente de una implementación concreta.
    
- No hacer todos los atributos `protected` sin necesidad.
    

---

## Encapsular condicionales

La lógica booleana compleja puede ser difícil de leer.

Conviene extraerla a un método con un nombre expresivo.

### Ejemplo

En vez de:

```
if (timer.expiro() && !timer.esRecurrente())
```

usar:

```
if (debeBorrarse(timer))
```

El nombre explica la intención.

---

## Evitar condicionales negativos

Los condicionales positivos suelen ser más fáciles de comprender.

### Ejemplo

Preferir:

```
if (usuario.esMayor())
```

antes que:

```
if (!usuario.noEsMayor())
```

---

## Las funciones deben hacer una sola cosa

Una función debería tener una responsabilidad clara.

Si una función:

1. valida datos;
    
2. guarda información;
    
3. genera un reporte;
    
4. envía un email;
    

probablemente esté haciendo demasiadas cosas.

---

## Encapsular condiciones borde

Las condiciones especiales o de borde pueden ser difíciles de rastrear cuando están dispersas por todo el sistema.

Conviene concentrarlas en un lugar o abstraerlas adecuadamente.

---

# Resumen final

Los principios de diseño apuntan, en conjunto, a producir software:

- **simple** → KISS;
    
- **sin duplicación** → DRY;
    
- **sin funcionalidad innecesaria** → YAGNI;
    
- **con responsabilidades separadas** → SoC;
    
- **con pocas dependencias y poco conocimiento entre objetos** → PoLC y PoLK;
    
- **con comportamiento intuitivo** → PoLA;
    
- **orientado al comportamiento de los objetos** → Tell, Don't Ask;
    
- **con dependencias claras** → EDP;
    
- **optimizado solamente cuando corresponde** → KOP.
    

## Idea general para recordar

> **Un buen diseño busca que cada componente tenga una responsabilidad clara, conozca poco sobre los demás, exponga solamente lo necesario y se comporte de forma simple y predecible.**