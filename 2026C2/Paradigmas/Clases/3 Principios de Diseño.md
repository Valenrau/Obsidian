# Principios de Diseño — ejemplos mejorados

## Cómo leer estos principios

Los principios de diseño no son reglas absolutas: son criterios para tomar decisiones de diseño. El objetivo es que el código sea más comprensible, mantenible y fácil de modificar.

Los ejemplos de abajo están pensados para **distinguir principios que pueden parecerse entre sí**. En cada caso se muestra:

- el problema;
- una versión problemática;
- una versión mejorada;
- qué principio se está aplicando;
- cómo diferenciarlo de otros principios.

> **Importante:** este resumen se centra en TDA, PoLC, PoLA, PoLK, EDP, KOP y SoC. No incluye KISS, YAGNI ni DRY.

---

# 1. TDA — Tell, Don't Ask

## Idea

**No le preguntes a un objeto por sus datos para tomar vos una decisión que conceptualmente le corresponde al objeto. Decile directamente qué querés que haga.**

La diferencia fundamental es:

> **Ask:** "Dame tus datos y yo decido qué hacer."  
> **Tell:** "Hacé esto vos, porque vos conocés tu propio estado."

## Ejemplo

Tenemos una `Cuenta`.

### Problema

```java
class Banco {
    public void retirar(Cuenta cuenta, double monto) {
        if (cuenta.getSaldo() >= monto) {
            cuenta.setSaldo(cuenta.getSaldo() - monto);
        }
    }
}
```

`Banco` está preguntándole a `Cuenta` cuál es su saldo y después está manipulando su estado.

### Mejor

```java
class Cuenta {
    private double saldo;

    public void retirar(double monto) {
        if (saldo >= monto) {
            saldo -= monto;
        }
    }
}
```

Y desde afuera:

```java
cuenta.retirar(500);
```

Ahora `Cuenta` decide cómo se realiza un retiro.

## ¿Qué ganó el diseño?

La lógica relacionada con el saldo está donde corresponde: dentro de `Cuenta`.

Si mañana retirar dinero requiere, por ejemplo:

- verificar que el monto sea positivo;
- registrar la operación;
- cobrar una comisión;
- verificar un límite;

el código que usa `Cuenta` no necesita conocer esos detalles.

## Cómo diferenciarlo de PoLK

Ambos pueden aparecer juntos, pero miran problemas distintos:

- **TDA:** "¿Por qué estoy preguntando por el estado para decidir qué hacer?"
- **PoLK:** "¿Por qué estoy conociendo la estructura interna de tantos objetos?"

Por ejemplo:

```java
pedido.getCliente().getCuenta().getSaldo()
```

es claramente un problema de **PoLK** porque estás navegando por varios objetos.

Si después hacés:

```java
if (cuenta.getSaldo() >= monto) {
    cuenta.setSaldo(cuenta.getSaldo() - monto);
}
```

aparece además **TDA**, porque estás preguntando y manipulando el estado en lugar de decirle a `Cuenta` que retire.

---

# 2. PoLC — Principle of Least Commitment

## Idea

**Una clase debería depender del mínimo comportamiento que realmente necesita.**

No significa simplemente "tener pocos métodos". Significa que un usuario de una abstracción no debería quedar comprometido con detalles que no necesita.

## Ejemplo

Supongamos:

```java
class Cartulina {
    public void pintar() { }
    public void doblar() { }
    public void reciclar() { }
}
```

Un reciclador solamente necesita reciclar cosas.

### Problema

```java
class Reciclador {
    public void reciclar(Cartulina cartulina) {
        cartulina.reciclar();
    }
}
```

El `Reciclador` quedó comprometido con `Cartulina`, aunque solamente necesita una capacidad: `reciclar()`.

### Mejor

```java
interface Reciclable {
    void reciclar();
}
```

```java
class Cartulina implements Reciclable {
    public void pintar() { }
    public void doblar() { }

    @Override
    public void reciclar() { }
}
```

Ahora:

```java
class Reciclador {
    public void reciclar(Reciclable objeto) {
        objeto.reciclar();
    }
}
```

`Reciclador` sabe solamente lo que necesita saber.

## Una forma fácil de pensarlo

Preguntate:

> **"¿Estoy dependiendo de algo más específico de lo necesario?"**

Si la respuesta es sí, probablemente haya un problema de mínimo compromiso.

## Cómo diferenciarlo de EDP

- **PoLC:** se preocupa por **cuánto de una abstracción necesita conocer/dependender** el cliente.
- **EDP:** se preocupa por **cómo se obtienen las dependencias que sí necesita**.

Ejemplo:

```java
class Reciclador {
    private Reciclable objeto;

    public Reciclador(Reciclable objeto) {
        this.objeto = objeto;
    }
}
```

Acá:

- usar `Reciclable` en lugar de `Cartulina` → **PoLC**;
- recibir `Reciclable` por constructor en lugar de crearlo internamente → **EDP**.

Pueden aplicarse simultáneamente.

---

# 3. PoLA — Principle of Least Astonishment

## Idea

**El código debería comportarse de la manera que razonablemente espera quien lo utiliza.**

El problema no es necesariamente que el código esté "mal". El problema es que hace algo inesperado para su abstracción o nombre.

## Ejemplo

Supongamos:

```java
class Lista {
    public void ordenar() {
        // ordena los elementos
    }
}
```

Un usuario razonablemente espera que:

```java
lista.ordenar();
```

ordene la lista.

### Sorprendente

```java
class Lista {
    public void ordenar() {
        ordenarElementos();
        guardarEnBaseDeDatos();
        enviarEmailAlAdministrador();
    }
}
```

El método hace cosas que no se desprenden de su nombre ni de la responsabilidad esperada de `ordenar()`.

## Otro ejemplo más sutil

```java
class Cuenta {
    public double getSaldo() {
        cerrarCuenta();
        return saldo;
    }
}
```

Un `getSaldo()` normalmente se interpreta como una consulta.

Que consultar el saldo cierre la cuenta sería un comportamiento inesperado.

## Cómo diferenciarlo de SoC

- **PoLA:** "¿Este comportamiento sorprende al usuario de esta clase/método?"
- **SoC:** "¿Estoy mezclando incumbencias diferentes en el mismo componente?"

Por ejemplo, que `ordenar()` además envíe un email puede violar **SoC** porque mezcla lógica de ordenamiento con comunicación externa.

Pero también viola **PoLA** porque quien llama a `ordenar()` no espera que se envíe un email.

Una misma situación puede violar ambos, pero por motivos diferentes.

---

# 4. PoLK — Principle of Least Knowledge

## Idea

**Un objeto debería conocer lo menos posible sobre la estructura interna de otros objetos.**

Una forma clásica de detectarlo es buscar cadenas de llamadas:

```java
a.getB().getC().getD()
```

Cuanto más necesitás conocer la estructura interna de otros objetos, mayor es el acoplamiento.

## Ejemplo

Supongamos:

```java
class Pedido {
    private Cliente cliente;

    public Cliente getCliente() {
        return cliente;
    }
}
```

```java
class Cliente {
    private Direccion direccion;

    public Direccion getDireccion() {
        return direccion;
    }
}
```

```java
class Direccion {
    public String getCodigoPostal() {
        return "1824";
    }
}
```

### Problema

```java
pedido.getCliente()
      .getDireccion()
      .getCodigoPostal();
```

El código que trabaja con `Pedido` necesita conocer que:

```text
Pedido
  └── tiene Cliente
        └── tiene Direccion
              └── tiene CodigoPostal
```

Si cambia la estructura interna, puede romperse código externo.

### Mejor

```java
class Pedido {
    public String getCodigoPostalEntrega() {
        return cliente.getDireccion().getCodigoPostal();
    }
}
```

Desde afuera:

```java
pedido.getCodigoPostalEntrega();
```

El usuario de `Pedido` no necesita conocer cómo está organizado internamente.

## Cómo diferenciarlo de TDA

Este ejemplo también puede verse desde TDA, pero la pregunta que te conviene hacer en el examen es:

- **TDA:** "¿Estoy preguntando por datos para hacer una operación que debería hacer el objeto?"
- **PoLK:** "¿Estoy conociendo/navegando la estructura interna de otros objetos?"

Un caso puede violar ambos.

### Relación con Feature Envy

Si una clase hace constantemente:

```java
empleado.getHoras();
empleado.getTarifa();
empleado.getHorasExtra();
```

para calcular algo que conceptualmente pertenece a `Empleado`, además de conocimiento excesivo puede haber **Feature Envy**.

Pero para distinguir los principios permitidos, quedate con:

> **PoLK = reducir el conocimiento estructural entre objetos.**

---

# 5. EDP — Explicit Dependencies Principle

## Idea

**Una clase debería dejar explícito qué objetos necesita para funcionar.**

Una dependencia es explícita cuando puede verse en la interfaz de la clase, por ejemplo en un parámetro o en el constructor.

## Ejemplo

Supongamos un servicio que necesita un repositorio.

### Dependencia oculta

```java
class ServicioUsuario {
    private RepositorioUsuarios repositorio;

    public ServicioUsuario() {
        repositorio = new RepositorioUsuarios();
    }
}
```

Para entender qué necesita `ServicioUsuario`, hay que mirar el interior del constructor.

Además, la clase decidió por sí misma qué implementación concreta utilizar.

### Dependencia explícita

```java
class ServicioUsuario {
    private RepositorioUsuarios repositorio;

    public ServicioUsuario(RepositorioUsuarios repositorio) {
        this.repositorio = repositorio;
    }
}
```

Ahora la dependencia aparece directamente en la declaración:

```java
new ServicioUsuario(repositorio)
```

Es evidente que `ServicioUsuario` necesita un `RepositorioUsuarios`.

## ¿Por qué es útil?

Facilita:

- comprender la clase;
- reemplazar dependencias;
- probar la clase;
- reutilizarla en otros contextos.

Por ejemplo, en una prueba podemos pasar:

```java
RepositorioUsuariosFalso
```

en lugar del repositorio real.

## Cómo diferenciarlo de PoLC

Esta diferencia es especialmente importante:

### PoLC

Pregunta:

> **"¿De qué cosa estoy dependiendo?"**

```java
void procesar(Cartulina cartulina)
```

si solamente necesito reciclarla es demasiado específico.

### EDP

Pregunta:

> **"¿Cómo llega a mí la cosa que necesito?"**

```java
ServicioUsuario() {
    repositorio = new RepositorioUsuarios();
}
```

oculta una dependencia que podría recibirse externamente.

Por eso, PoLC y EDP pueden aparecer juntos:

```java
class ServicioUsuario {
    public ServicioUsuario(RepositorioUsuarios repositorio) {
        ...
    }
}
```

El constructor hace explícita la dependencia (**EDP**) y, si `RepositorioUsuarios` es una abstracción suficientemente general para lo que el servicio necesita, también puede aplicar **PoLC**.

---

# 6. KOP — Knuth's Optimization Principle

## Idea

**No optimizar prematuramente.**

Primero hay que conseguir una solución correcta y razonablemente clara. Después, si existe un problema real de rendimiento, se identifica dónde está y se optimiza esa parte.

La idea no es "el rendimiento no importa".

La idea es:

> **No sacrificar claridad por una optimización que todavía no sabemos si necesitamos.**

## Ejemplo

Supongamos que necesitamos buscar un usuario.

Una solución simple:

```java
public Usuario buscar(String nombre, List<Usuario> usuarios) {
    for (Usuario usuario : usuarios) {
        if (usuario.getNombre().equals(nombre)) {
            return usuario;
        }
    }

    return null;
}
```

Alguien podría reemplazar inmediatamente todo por una estructura mucho más compleja "porque es más rápida".

El problema es que todavía no sabemos:

- cuántos usuarios habrá;
- cuánto tiempo tarda realmente;
- si esa operación es un cuello de botella;
- si la optimización complica innecesariamente el código.

### Aplicación de KOP

Primero implementamos una solución correcta y clara.

Después medimos.

Si descubrimos que `buscar()` consume una parte importante del tiempo de ejecución, recién entonces analizamos alternativas como un `HashMap`.

## Importante

KOP **no dice**:

> "Siempre usá la solución más lenta y simple."

Dice:

> "No optimices sin una razón basada en un problema real."

Si desde el comienzo existe un requisito explícito de rendimiento, entonces sí hay una razón para diseñar teniendo ese rendimiento en cuenta.

---

# 7. SoC — Separation of Concerns

## Idea

**Separar diferentes incumbencias del sistema para que cada parte se encargue de una preocupación distinta.**

Una incumbencia puede ser, por ejemplo:

- lógica de negocio;
- persistencia;
- interfaz;
- comunicación externa;
- generación de reportes.

## Ejemplo

Supongamos una compra.

### Problema

```java
class Compra {
    public void procesar() {
        // calcula total
        // valida stock
        // guarda en base de datos
        // genera PDF
        // envía email
    }
}
```

Esta clase está mezclando muchas incumbencias.

Si cambia la base de datos, la generación del PDF o el sistema de emails, `Compra` puede necesitar modificaciones.

### Mejor separación

```java
class CalculadorDeCompra {
    public double calcularTotal(Compra compra) {
        // lógica de negocio
    }
}
```

```java
class RepositorioCompras {
    public void guardar(Compra compra) {
        // persistencia
    }
}
```

```java
class GeneradorDeFactura {
    public void generar(Compra compra) {
        // generación de factura
    }
}
```

```java
class ServicioEmail {
    public void enviarFactura(Compra compra) {
        // comunicación externa
    }
}
```

Cada componente se ocupa de una incumbencia distinta.

## Cómo diferenciarlo de TDA

Esto suele confundirse mucho.

### TDA

Se pregunta:

> **"¿Quién debería realizar esta operación?"**

Si `Compra` conoce la lógica de calcular su propio total, podría tener sentido:

```java
compra.calcularTotal();
```

en vez de sacar todos sus datos y calcularlos desde afuera.

### SoC

Se pregunta:

> **"¿Qué incumbencias diferentes estoy mezclando?"**

Que una clase calcule el total y además envíe emails mezcla dos preocupaciones diferentes.

Por eso:

```text
TDA → dónde colocar un comportamiento relacionado con un objeto
SoC → cómo separar preocupaciones diferentes del sistema
```

---

# Comparaciones que suelen confundirse

## TDA vs PoLK

### TDA

```java
if (cuenta.getSaldo() >= monto) {
    cuenta.setSaldo(cuenta.getSaldo() - monto);
}
```

Problema principal:

> Pregunto el estado de `Cuenta` y decido desde afuera qué hacer.

Solución:

```java
cuenta.retirar(monto);
```

### PoLK

```java
pedido.getCliente().getDireccion().getCodigoPostal();
```

Problema principal:

> Conozco demasiado sobre la estructura interna de otros objetos.

Solución:

```java
pedido.getCodigoPostalEntrega();
```

### Regla mental

> **TDA = "hacelo vos".**  
> **PoLK = "no necesito saber cómo estás armado".**

---

## PoLC vs EDP

### PoLC

```java
void reciclar(Cartulina cartulina)
```

si solamente se necesita:

```java
cartulina.reciclar();
```

Problema:

> Estoy dependiendo de una abstracción más específica de lo necesario.

### EDP

```java
class Servicio {
    Servicio() {
        repositorio = new Repositorio();
    }
}
```

Problema:

> La dependencia existe, pero está escondida.

Solución:

```java
Servicio(Repositorio repositorio)
```

### Regla mental

> **PoLC = mínimo compromiso.**  
> **EDP = dependencia visible.**

---

## PoLA vs SoC

### PoLA

```java
lista.ordenar();
```

y que eso además mande un email.

Problema:

> El comportamiento sorprende.

### SoC

```java
class Compra {
    calcularTotal();
    guardarEnBD();
    enviarEmail();
}
```

Problema:

> Se mezclan incumbencias diferentes.

### Regla mental

> **PoLA = "¿me sorprende lo que hace?"**  
> **SoC = "¿está mezclando preocupaciones diferentes?"**

---

# Una misma situación puede violar varios principios

Los principios no son compartimentos estancos.

Por ejemplo:

```java
if (pedido.getCliente().getCuenta().getSaldo() >= monto) {
    pedido.getCliente().getCuenta()
          .setSaldo(pedido.getCliente().getCuenta().getSaldo() - monto);
}
```

Acá pueden aparecer varios problemas:

### TDA

Se pregunta por el saldo y se modifica desde afuera.

### PoLK

Se conoce la estructura:

```text
Pedido → Cliente → Cuenta
```

### PoLC

Si el código solamente necesita retirar dinero, no necesita conocer toda esa estructura ni depender de esos detalles.

Una posible solución sería:

```java
pedido.pagar(monto);
```

y que `Pedido` delegue internamente lo necesario:

```java
class Pedido {
    public void pagar(double monto) {
        cliente.pagar(monto);
    }
}
```

y:

```java
class Cliente {
    public void pagar(double monto) {
        cuenta.retirar(monto);
    }
}
```

Cada objeto conserva el conocimiento y comportamiento que le corresponde.

---

# Tabla mental para estudiar

| Principio | Pregunta que te ayuda a detectarlo |
|---|---|
| **TDA** | ¿Estoy preguntando datos para decidir desde afuera algo que debería hacer el objeto? |
| **PoLC** | ¿Estoy dependiendo de algo más específico de lo que realmente necesito? |
| **PoLA** | ¿El comportamiento de este componente sorprende razonablemente a quien lo usa? |
| **PoLK** | ¿Estoy conociendo demasiado la estructura interna de otros objetos? |
| **EDP** | ¿Las dependencias que necesita una clase están claramente declaradas? |
| **KOP** | ¿Estoy complicando/optimizando el código sin haber demostrado que exista un problema de rendimiento? |
| **SoC** | ¿Estoy mezclando incumbencias diferentes en el mismo componente? |

---

# Una forma rápida de reconocerlos en un ejercicio

Cuando te den código y te pregunten qué principio se viola, podés recorrer estas preguntas:

1. **¿Un objeto está preguntando datos y haciendo el trabajo que debería hacer otro?**
   → **TDA**

2. **¿Una clase depende de una abstracción más grande o específica de lo que necesita?**
   → **PoLC**

3. **¿Un método hace algo inesperado para su nombre o abstracción?**
   → **PoLA**

4. **¿Hay cadenas de `get...().get...().get...` o conocimiento de la estructura interna?**
   → **PoLK**

5. **¿La clase crea internamente objetos que necesita en lugar de recibirlos?**
   → **EDP**

6. **¿Se está optimizando o complicando el código sin evidencia de que haga falta?**
   → **KOP**

7. **¿Una clase o módulo mezcla responsabilidades/incumbencias diferentes?**
   → **SoC**

## Idea final

Los siete principios pueden resumirse así:

> **TDA:** cada objeto debería hacerse cargo de su comportamiento.  
> **PoLC:** depender de lo mínimo necesario.  
> **PoLA:** comportarse como razonablemente se espera.  
> **PoLK:** conocer lo mínimo sobre los demás.  
> **EDP:** hacer visibles las dependencias.  
> **KOP:** optimizar cuando exista una razón real.  
> **SoC:** separar incumbencias diferentes.

Una buena forma de estudiarlos no es memorizar solamente las definiciones, sino aprender a reconocer **qué pregunta responde cada uno**.
