# Patrones de Diseño Creacionales

## 1. Introducción a los Patrones Creacionales

### Concepto y Propósito
Los patrones de diseño creacionales responden a una pregunta fundamental en el diseño de sistemas orientados a objetos: **¿Cómo creamos nuestros objetos?**

En aplicaciones complejas, instanciar objetos directamente mediante el operador `new` repartido por todo el código fuente genera un acoplamiento rígido con clases concretas. Esto dificulta la extensibilidad del sistema, violando principios de diseño como el de Abierto/Cerrado (OCP) y el de Inversión de Dependencias (DIP). 

El propósito de los patrones creacionales es abstraer y encapsular el proceso de instanciación. Ocultan la forma exacta en que los objetos se crean, componen y representan, permitiendo que el sistema sea independiente de las clases concretas utilizadas.

### Clasificación según Ámbito (Scope)
Los patrones creacionales se dividen técnicamente según su alcance o ámbito de aplicación:
- **Ámbito de Clase (Class Scope):** Utilizan la herencia para delegar el proceso de instanciación a las subclases. El ejemplo principal es **Factory Method**.
- **Ámbito de Objeto (Object Scope):** Utilizan la delegación y la composición entre objetos para manejar la creación. Entre ellos se encuentran **Abstract Factory**, **Builder**, **Singleton**, **Prototype** y **Object Pool**.

---

## 2. Factory Method

### Explicación Conceptual
El patrón **Factory Method** define una interfaz o método abstracto en una superclase para crear un objeto, pero delega a las subclases la decisión de qué clase concreta instanciar. 

#### Problema
Cuando una clase necesita instanciar productos pertenecientes a una misma jerarquía pero con variaciones según el contexto, el enfoque ingenuo recurre a estructuras condicionales (`if`/`else` o `switch`) comprobando cadenas de texto o códigos de tipo. Esto genera dos grandes inconvenientes:
1. La clase instanciadora queda acoplada a todas las clases concretas de productos.
2. Cada vez que se agrega un nuevo producto a la familia, se debe modificar el código existente alterando el condicional, rompiendo OCP.

#### Solución
Factory Method traslada la responsabilidad de instanciación a un método abstracto (`createProduct()`) dentro de una clase creadora abstracta. El método principal de la clase abstracta maneja la lógica general de negocio (el proceso uniforme para todos los productos), mientras que cada subclase concreta sobrescribe el método fábrica para retornar su producto específico.

#### Estructura y Participantes
- **Product (Interfaz/Clase Abstracta):** Define el contrato común para todos los objetos producidos.
- **ConcreteProduct:** Implementación concreta del producto.
- **Creator (Clase Abstracta):** Declara el método fábrica abstracto (`factoryMethod()`) y contiene la lógica de negocio general (`someOperation()`) que opera sobre los productos devueltos por la fábrica.
- **ConcreteCreator:** Sobrescribe `factoryMethod()` para instanciar y retornar un `ConcreteProduct`.

---

### Metáfora
Imaginen una cadena de franquicias gastronómicas como una pizzería. El proceso general de preparar una pizza (amasar la masa, agregar los ingredientes básicos, hornear, cortar y empaquetar en la caja) es exactamente idéntico en todas las sucursales. Sin embargo, la sucursal de Nueva York prepara una pizza con masa fina y queso Reggiano, mientras que la sucursal de Chicago prepara una pizza de masa gruesa con abundante salsa de tomate. 

En lugar de tener una cocina central con un condicional gigante evaluando la ciudad donde está el cliente, la marca fija el protocolo estándar de preparación en la casa matriz (superclase), pero exige que cada sucursal local (subclase) implemente la receta específica del armado inicial de la pizza.

---

### Ejemplo Práctico y Técnico en Java

#### Código Problemático (Sin el patrón)
```java
public class PizzaStore {

    public Pizza orderPizza(String type) {
        Pizza pizza;

        // Violación de OCP y acoplamiento directo a clases concretas
        if (type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (type.equals("greek")) {
            pizza = new GreekPizza();
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();
        } else {
            throw new IllegalArgumentException("Tipo de pizza desconocido");
        }

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }
}
```

#### Código Refactorizado con Factory Method

```java
// Interfaz Producto
public interface Pizza {
    void prepare();
    void bake();
    void cut();
    void box();
}

// Productos Concretos
public class NYStyleCheesePizza implements Pizza {
    @Override
    public void prepare() { System.out.println("Preparando pizza estilo Nueva York (masa fina)..."); }
    @Override
    public void bake() { System.out.println("Horneando a 350°C..."); }
    @Override
    public void cut() { System.out.println("Cortando en rebanadas triangulares..."); }
    @Override
    public void box() { System.out.println("Colocando en caja de la franquicia NY..."); }
}

public class ChicagoStyleCheesePizza implements Pizza {
    @Override
    public void prepare() { System.out.println("Preparando pizza estilo Chicago (masa gruesa)..."); }
    @Override
    public void bake() { System.out.println("Horneando a 400°C..."); }
    @Override
    public void cut() { System.out.println("Cortando en porciones cuadradas..."); }
    @Override
    public void box() { System.out.println("Colocando en caja profunda de Chicago..."); }
}

// Creador Abstracto
public abstract class PizzaStore {

    public Pizza orderPizza(String type) {
        // Se llama al método fábrica abstracto
        Pizza pizza = createPizza(type);

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }

    // Factory Method
    protected abstract Pizza createPizza(String type);
}

// Creadores Concretos
public class NYPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String type) {
        if (type.equals("cheese")) {
            return new NYStyleCheesePizza();
        }
        return null;
    }
}

public class ChicagoPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String type) {
        if (type.equals("cheese")) {
            return new ChicagoStyleCheesePizza();
        }
        return null;
    }
}
```

---

## 3. Abstract Factory

### Explicación Conceptual
El patrón **Abstract Factory** (Fábrica Absoluta) proporciona una interfaz para crear familias de objetos relacionados o dependientes sin especificar sus clases concretas.

#### Problema
A menudo se requiere que una aplicación funcione con familias de productos coherentes entre sí. Por ejemplo, en un entorno de cocina o restaurante, los ingredientes (masa, queso, salsa, vegetales) varían según el proveedor o región geográfica. Si un sistema mezcla ingredientes de un proveedor gourmet de alta calidad con ingredientes de baja calidad de forma aleatoria, se producen inconsistencias. Asimismo, el cliente no debería conocer las clases concretas de cada ingrediente individual.

#### Solución
Abstract Factory define una interfaz emisora con múltiples métodos fábrica, uno por cada tipo de producto abstracto que conforma la familia. Cada fábrica concreta implementa esa interfaz y garantiza la producción de variantes compatibles de cada producto. El cliente solo interactúa con la interfaz de la fábrica abstracta y con las interfaces abstractas de los productos, desacoplándose completamente de las implementaciones concretas.

#### Estructura y Participantes
- **AbstractFactory:** Interfaz que declara los métodos de creación para cada producto abstracto de la familia (`createDough()`, `createCheese()`, `createSauce()`).
- **ConcreteFactory1 / ConcreteFactory2:** Fábricas concretas que crean productos pertenecientes a una misma variante o familia.
- **AbstractProductA / AbstractProductB:** Interfaces de los distintos tipos de productos que componen la familia.
- **ConcreteProductA1, ConcreteProductB1:** Implementaciones específicas producidas por la `ConcreteFactory1`.
- **Client:** Clase cliente que utiliza únicamente las interfaces de `AbstractFactory` y de los productos abstractos.

---

### Metáfora
Piensen en dos cadenas gastronómicas de distintas categorías o franquicias, por ejemplo, una pizzería gourmet y una pizzería de barrio barata. Ambas necesitan comprar una familia completa de ingredientes: masa, queso y salsa. 

El dueño de la pizzería interactúa con una abstracción llamada "Proveedor de Ingredientes". La fábrica gourmet provee masa crujiente de fermentación lenta, queso Reggiano de origen controlado y salsa de tomates italianos importados. La fábrica económica provee masa al molde común, queso mozzarella abundante estándar y salsa de tomate en lata básica. El cocinero simplemente pide "dame la masa, el queso y la salsa a mi proveedor configurado" y prepara la pizza sin preocuparse por los detalles específicos de fabricación de cada insumo individual.

---

### Ejemplo Práctico y Técnico en Java

```java
// Productos Abstractos
public interface Dough { void displayDough(); }
public interface Cheese { void displayCheese(); }
public interface Sauce { void displaySauce(); }

// Productos Concretos - Familia Gourmet / NY
public class ThinCrustDough implements Dough {
    @Override
    public void displayDough() { System.out.println("Masa fina crujiente"); }
}

public class ReggianoCheese implements Cheese {
    @Override
    public void displayCheese() { System.out.println("Queso Reggiano rallado"); }
}

public class MarinaraSauce implements Sauce {
    @Override
    public void displaySauce() { System.out.println("Salsa Marinara fresca"); }
}

// Productos Concretos - Familia Barata / Chicago
public class ThickCrustDough implements Dough {
    @Override
    public void displayDough() { System.out.println("Masa gruesa al molde"); }
}

public class MozzarellaCheese implements Cheese {
    @Override
    public void displayCheese() { System.out.println("Queso Mozzarella abundante"); }
}

public class PlumTomatoSauce implements Sauce {
    @Override
    public void displaySauce() { System.out.println("Salsa de tomate entero Plum"); }
}

// Fábrica Abstracta
public interface PizzaIngredientFactory {
    Dough createDough();
    Cheese createCheese();
    Sauce createSauce();
}

// Fábricas Concretas
public class NYPizzaIngredientFactory implements PizzaIngredientFactory {
    @Override
    public Dough createDough() { return new ThinCrustDough(); }
    @Override
    public Cheese createCheese() { return new ReggianoCheese(); }
    @Override
    public Sauce createSauce() { return new MarinaraSauce(); }
}

public class ChicagoPizzaIngredientFactory implements PizzaIngredientFactory {
    @Override
    public Dough createDough() { return new ThickCrustDough(); }
    @Override
    public Cheese createCheese() { return new MozzarellaCheese(); }
    @Override
    public Sauce createSauce() { return new PlumTomatoSauce(); }
}

// Cliente que consume la fábrica
public class ClientStore {
    private final PizzaIngredientFactory ingredientFactory;

    public ClientStore(PizzaIngredientFactory factory) {
        this.ingredientFactory = factory;
    }

    public void makePizza() {
        Dough dough = ingredientFactory.createDough();
        Cheese cheese = ingredientFactory.createCheese();
        Sauce sauce = ingredientFactory.createSauce();

        dough.displayDough();
        cheese.displayCheese();
        sauce.displaySauce();
    }
}
```

---

## 4. Builder

### Explicación Conceptual
El patrón **Builder** (Constructor) separa la construcción de un objeto complejo de su representación, de modo que el mismo proceso de construcción pueda crear diferentes representaciones.

#### Problema
La creación de objetos complejos que contienen múltiples partes, componentes opcionales o configuraciones dinámicas suele derivar en dos problemas de diseño:
1. **Constructores Telescópicos:** Sobrecarga excesiva de constructores con combinaciones infinitas de parámetros.
2. **Uso Excesivo de Banderas (Flags):** Constructores que reciben múltiples valores booleanos para indicar si se incluye o no determinada característica, lo que ensucia la firma y dificulta el mantenimiento.

#### Solución
Builder abstrae los pasos de construcción en métodos individuales dentro de un objeto constructor (`buildWalls()`, `buildRoof()`, `buildGarage()`). Esto permite construir el objeto de forma incremental, paso a paso. El estado del objeto en construcción se almacena en el Builder y solo al finalizar la configuración se solicita el producto final mediante un método como `getResult()` o `build()`.

Se introduce frecuentemente el concepto de **Director**, una clase que orquesta el orden de invocación de los pasos del Builder para producir configuraciones predefinidas populares.

#### Construcción Iterativa por Múltiples Clientes
Una ventaja técnica destacada del Builder es que no requiere ser utilizado por un único cliente. Al ser una instancia encargada del ensamblaje incremental, un Builder puede pasar consecutivamente por distintos componentes o fases de la aplicación (por ejemplo, una etapa de validación de datos, una de transformación y una de personalización estética), donde cada actor invoca los métodos que le competen antes de generar el resultado definitivo.

#### Estructura y Participantes
- **Builder (Interfaz):** Declara los pasos abstractos de creación comunes a todos los constructores concretos.
- **ConcreteBuilder:** Implementa los pasos de construcción especificando la representación concreta y almacena el resultado final.
- **Director:** Clientes u orquestadores opcionales que controlan la secuencia de construcción utilizando la interfaz del Builder.
- **Product:** El objeto complejo final producido.

---

### Metáfora
Imaginen la planificación de un itinerario de vacaciones de varios días o la construcción de una casa. 

En la construcción de una casa, no se entrega la estructura terminada de un solo golpe tirando todos los materiales en un camión. El cliente o el contratista (Director) le da instrucciones al constructor (Builder): primero construye las paredes, luego instala el techo, después agrega las puertas y las ventanas, y opcionalmente añade un garaje. Cada paso se ejecuta por separado. Al final, el contratista le pide al constructor "entrégame la casa terminada".

---

### Ejemplo Práctico y Técnico en Java

```java
// Producto Complejo
public class House {
    private String walls;
    private String roof;
    private int windows;
    private boolean hasGarage;

    public void setWalls(String walls) { this.walls = walls; }
    public void setRoof(String roof) { this.roof = roof; }
    public void setWindows(int windows) { this.windows = windows; }
    public void setHasGarage(boolean hasGarage) { this.hasGarage = hasGarage; }

    @Override
    public String toString() {
        return "Casa [Paredes: " + walls + ", Techo: " + roof + 
               ", Ventanas: " + windows + ", Garaje: " + hasGarage + "]";
    }
}

// Interfaz Builder
public interface HouseBuilder {
    void reset();
    HouseBuilder buildWalls();
    HouseBuilder buildRoof();
    HouseBuilder buildWindows(int count);
    HouseBuilder buildGarage();
    House getResult();
}

// Builder Concreto
public class ConcreteHouseBuilder implements HouseBuilder {
    private House house;

    public ConcreteHouseBuilder() {
        this.reset();
    }

    @Override
    public void reset() {
        this.house = new House();
    }

    @Override
    public HouseBuilder buildWalls() {
        house.setWalls("Paredes de hormigón");
        return this;
    }

    @Override
    public HouseBuilder buildRoof() {
        house.setRoof("Techo teja española");
        return this;
    }

    @Override
    public HouseBuilder buildWindows(int count) {
        house.setWindows(count);
        return this;
    }

    @Override
    public HouseBuilder buildGarage() {
        house.setHasGarage(true);
        return this;
    }

    @Override
    public House getResult() {
        House product = this.house;
        this.reset(); // Listo para construir otra casa
        return product;
    }
}

// Director (Opcional)
public class CivilEngineerDirector {
    private HouseBuilder builder;

    public CivilEngineerDirector(HouseBuilder builder) {
        this.builder = builder;
    }

    public House constructLuxuryHouse() {
        return builder.buildWalls()
                      .buildRoof()
                      .buildWindows(6)
                      .buildGarage()
                      .getResult();
    }
}
```

---

## 5. Singleton

### Explicación Conceptual
El patrón **Singleton** (Instancia Única) garantiza que una clase tenga una y solo una instancia en todo el ciclo de vida de la aplicación, y proporciona un punto de acceso global a dicha instancia.

#### Problema
Existen situaciones funcionales y técnicas donde permitir múltiples instancias de un objeto genera conflictos graves de recursos, condiciones de carrera o inconsistencias en el estado del sistema. Esto ocurre típicamente con controladores de entrada/salida (I/O), gestores de archivos de registro (loggers), administradores de conexiones a bases de datos, o clientes de servicios web externos expuestos a límites de peticiones (rate limiting).

Si múltiples componentes crean su propia instancia individual de un cliente web con *rate limiting*, la cuota de peticiones permitidas no se compartirá y el servidor bloqueará la aplicación.

#### Solución
El patrón Singleton restringe la instanciación directa aplicando las siguientes reglas en el diseño de la clase:
1. **Constructor Privado:** Se declara el constructor con visibilidad privada para impedir que código externo utilice el operador `new`.
2. **Atributo Estático Privado:** La clase guarda internamente su única instancia en una variable de clase privada.
3. **Método de Acceso Público Estático (`getInstance()`):** Proporciona un método público de clase que evalúa si la instancia ya fue creada. Si no existe (`null`), la instancias de manera perezosa (*lazy initialization*); si ya existe, devuelve la referencia existente.

#### Consideraciones de Implementación y Variantes
- **Thread Safety:** En entornos multihilo o concurrentes, el método `getInstance()` debe protegerse contra condiciones de carrera para evitar que dos hilos instancien el objeto simultáneamente.
- **Método Reset:** En ciertos contextos (como pruebas unitarias), se suele incluir un método `reset()` para volver a colocar la instancia en `null`.
- **Multi-Singleton:** Es una variante que limita la creación de instancias a un número fijo prefijado (por ejemplo, un pool de un máximo de dos instancias identificadas por clave).

---

### Metáfora
Imaginen una sala de reuniones con un único micrófono físico. Si dos o más participantes hablaran por micrófonos independientes al mismo tiempo, el sonido se acoplaría y la reunión se volvería ininteligible. 

Para coordinar la comunicación, existe una regla estricta: hay un único micrófono central. Quien desee hablar debe pedir "el micrófono". Si alguien más pide el micrófono mientras está en uso, el gestor le entrega el mismo micrófono advirtiéndole que debe esperar su turno. No se pueden fabricar micrófonos nuevos dentro de la sala.

---

### Ejemplo Práctico y Técnico en Java

```java
public class RateLimitedApiClient {
    // Atributo estático privado que almacena la única instancia
    private static RateLimitedApiClient instance;

    private int requestCount;
    private final int MAX_REQUESTS = 100;

    // Constructor PRIVADO para evitar instanciación externa
    private RateLimitedApiClient() {
        this.requestCount = 0;
    }

    // Método estático de acceso global (con thread-safety básico)
    public static synchronized RateLimitedApiClient getInstance() {
        if (instance == null) {
            instance = new RateLimitedApiClient();
        }
        return instance;
    }

    public void makeRequest(String endpoint) {
        if (requestCount < MAX_REQUESTS) {
            requestCount++;
            System.out.println("Petición enviada a " + endpoint + ". Peticiones realizadas: " + requestCount);
        } else {
            System.err.println("Límite de peticiones alcanzado (Rate limit de 100 excedido).");
        }
    }

    public int getRequestCount() {
        return requestCount;
    }

    // Método opcional para reiniciar estado (útil para pruebas)
    public static void resetInstance() {
        instance = null;
    }
}
```

---

## 6. Otros Patrones Creacionales

### Prototype (Prototipo)
- **Concepto:** Permite crear nuevos objetos clonando o copiando una instancia existente (prototipo) en lugar de instanciarlos desde cero.
- **Propósito:** Evitar el costo computacional elevado de crear un objeto complejo mediante constructores pesados o consultas pesadas a bases de datos.
- **Funcionamiento:** Se declara un método de clonación (`clone()`) en una interfaz común que los objetos concretos implementan para duplicar su estado interno.

### Object Pool (Grupo de Objetos)
- **Concepto:** Mantiene una colección de objetos inicializados y listos para ser usados (un "pool") en lugar de crearlos y destruirlos bajo demanda.
- **Propósito:** Optimizar el rendimiento del sistema cuando el costo de instanciación y recolección de basura de objetos es extremadamente alto o cuando los recursos son escasos (por ejemplo, conexiones a bases de datos o hilos de ejecución).
- **Funcionamiento:** Un cliente solicita un objeto al Object Pool, realiza el trabajo y luego devuelve el objeto al pool para que pueda ser reciclado por otros clientes.

---

## 7. Matriz Comparativa de Patrones Creacionales

| Patrón | Propósito Principal | Ámbito | Mecanismo de Creación | Caso de Uso Típico |
| :--- | :--- | :--- | :--- | :--- |
| **Factory Method** | Delegar la creación de un objeto a subclases | Clase | Herencia / Método Abstracto | Creación de productos con un proceso de negocio común pero variantes por subclase |
| **Abstract Factory** | Crear familias de objetos relacionados compatibles | Objeto | Composición / Interfaz | Selección de proveedores de insumos o variantes temáticas de componentes |
| **Builder** | Construcción paso a paso de objetos complejos | Objeto | Ensamblaje incremental de a partes | Objetos con partes opcionales, múltiples configuraciones o constructores complejos |
| **Singleton** | Garantizar una única instancia global en el sistema | Objeto | Constructor privado / `getInstance()` | Gestores de recursos críticos, loggers, clientes con *rate limiting* |
| **Prototype** | Clonar objetos existentes sin acoplarse a su clase | Objeto | Copia de estado (`clone()`) | Duplicación rápida de objetos costosos de instanciar |
| **Object Pool** | Reciclar objetos caros en lugar de crearlos/destruirlos | Objeto | Colección administrada reusable | Gestión de conexiones a BD, hilos de ejecución (*thread pools*) |
