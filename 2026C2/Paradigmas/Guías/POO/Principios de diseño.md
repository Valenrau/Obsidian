
Identificar y justificar todos los _principios de diseño_ que se violan. Proponer e implementar una posible solución que respete dichos principios.

1.

```
class Zoológico {
    private Jaula jaula;

    public Zoológico(Jaula jaula) {
        this.jaula = jaula;
    }

    public void hacerSonidoDelAnimalEnJaula() {
        jaula.getHabitante().hacerSonido();
    }
}
```

	RTA: Se violan los principios de TDA y de PoLK, ya que inicialmente pregunta 
	por el estado de hacer la accion, luego con PoLK se da que Zoologico no 
	estaba hablando unicamente con su atributo, sino que ademas habla con los 
	atributos de Jaula

```
class Animal {
	public void hacerSonido();
}
class Jaula {
	private Animal habitante; //cada objeto jaula va a tener una variable llamada habitante que puede guardar una referencia a un objeto de tipo animal
	
	public Jaula(Animal habitante) { //Constructor
		this.habitante = habitante; //Guarda el habitante pasado por param.
	}
	public void hacerSonidoDelHabitante() {
		habitante.hacerSonido()
	}
}
class Zoologico {
	private Jaula jaula;
	
	public Zoologico(Jaula jaula) {
		this.jaula = jaula;
	}
	public void hacerSonidoDelAnimalEnjaulado() {
		jaula.hacerSonidoDelHabitante();
	}
}
```



2.

```
class CuentaBancaria {
    private double saldo;

    public CuentaBancaria(double saldoInicial) { this.saldo = saldoInicial; }

    public double tieneSaldoSuficiente(double monto) { return monto <= saldo; }

    public void retirar(double monto) { saldo -= monto; }
}

public class Cliente {
    public static void main(String[] args) {
        CuentaBancaria cuenta = new CuentaBancaria(1000);

        double montoSolicitado = 500;
        if (!cuenta.tieneSaldoSuficiente(montoSolicitado)) {
            throws MontoInsuficienteException("No se puede retirar. Fondos 
            insuficientes.");
        }


        cuenta.retirar(montoSolicitado);
    }
}
```

	RTA: Inicialmente el primer Principio violado seria el de TDA, ya que el 
	cliente pregunta para poder resolver hago que tendria que resolver 
	internamente la Cuenta. Ademas PoLC tambien se ve infringido ya que
	la clase expone metodos de más como si tiene suficiente saldo, siendo
	que ese deberia ser privado. Finalmente tambien rompe el principio de 
	PoLA, ya que uno esperaria que si quiere retirar mas de lo que tiene, 
	deberia romper/tirar excepcion por algun lado, no ejecutarse igual

```
class CuentaBancaria {
    private double saldo;

    public CuentaBancaria(double saldoInicial) { this.saldo = saldoInicial; }

    private boolean tieneSaldoSuficiente(double monto) { return monto <= saldo; }

    public boolean retirar(double monto) { 
	    if (!tieneSaldoSuficient(monto)) {
		    return false;
	    }
	    saldo -= monto; 
		return true;
	}
}

public class Cliente {
    public static void main(String[] args) {
        CuentaBancaria cuenta = new CuentaBancaria(1000);

        double montoSolicitado = 500;
        cuenta.retirar(montoSolicitado);
    }
}
```

3.

```
public class Deposito {
    public static void main(String[] args) {
        Inventario inventario = new Inventario(2000);

        int requerido= 50;
        if (!inventario.hayStock(requerido)) {
            throws StockInsuficienteException("No se puede retirar. Stock 
            insuficiente.");
        }
        cuenta.retirar(requerido);
    }
}

class Inventario {
    private int stock;

    public Inventario(int stockInicial) { this.stoc = stockInicial; }

    public double hayStock(int requerido) { return requerido <= stock; }

    public void retirar(int requerido) { stock -= requerido; }
}
```

	RTA: Mismo caso que con el de arriba, se explica que rompe los principios de 
	TDA, PoLC y PoLA

4.

```
public class GeneradorDeReportes {
    public void generarReporteExcel(String texto, Metadata metadata) {
        // Crea el reporte en formato Excel en base al texto recibido
        Excel excel = new Excel(texto);
        // Agrega metadata al reporte
        excel.agregarMetadata(metadata);
        // Guarda el archivo en el sistema de archivos
        excel.guardar();
    }

    public void generarReporteGoogleSheet(String texto, Metadata metadata) {
        // Crea el reporte en formato GoogleSheets en base al texto recibido
        GoogleSheet sheet = new GoogleSheet(texto);
        // Agrega metadata al reporte
        sheet.agregarMetadata(metadata);
        // Guarda el archivo en Google Drive
        sheet.guardar();
    }
}
```

	RTA: En este caso se puede ver como se viola principalmete el principio OCP
	ya que se esta haciendo un generador de reportes para cada formato, por lo 
	que podria ser que el generador de reportes reciba un formato y que se pueda 
	generar el deporte independientemente del formato. Actualmente si se 
	agregara un nuevo formato habria que agregar manualmente cada funcion cuando 
	podria resolverse de otra manera dentro de una clase "Formatos" y que cada 
	formato se encargue de hacer sus cosas. Tambien se rompe un poco DRY 

```
public interface Formato {
	void agregarMetadata(Metadata metadata);
	void guardar();
}
public class GeneradorDeReportes {
	public void generadorDeReportes(String texto, Metadata metadata, Formato 
	formato) {
		formato.agregarMetadata(metadata);
		formato.guardar();
	}
}
```

5.

```
public class Juego {
    public void saludar() {
        System.out.println("Iniciando Juego...");
        // logica de juego
    }
}
```

	RTA: El principio que no se cumple aca es el de PoLA, ya que se espera que
	haga algo y despues hace otra cosa que no tiene nada que ver. 
	El comportamiento debe ser coherente e intuitivo

```
public class Juego {
    public void iniciarJuego() {
        System.out.println("Iniciando Juego...");
        // logica de juego
    }
}
```

6.

```
public class Usuario {
    private String nombre;

    public void guardarEnBaseDeDatos() {
        System.out.println("Guardando usuario...");
    }

    public void enviarCorreoBienvenida() {
        System.out.println("Enviando correo de bienvenida...");
    }

    public void validarEmail() {
        System.out.println("Validando email...");
    }
}
```

	RTA: No se cumple con el principip de SRP, ya que la clase Usuario esta 
	teniendo responsabilidades de más que no van al caso, los otros metodos
	deberian ser clases sueltas que ejecuten esas funciones

```
public class Usuario {
	private String nombre;
}
class guardarEnBaseDeDatos {
	public void guardar() {
		System.out.println("Guardando usuario...");
	}
}
class enviarCorreoBienvenida {
	public void enviar() {
		System.out.println("Enviando correo de bienvenida...");
	}
}
class validarEmail() {
	public void validar() {
		System.out.println("Validando email...");
	}
}
```

7.

```
public class Calculadora {
    public int operar(String operacion, int a, int b) {
        if (operacion.equals("suma")) {
            return a + b;
        } else if (operacion.equals("resta")) {
            return a - b;
        }
        return 0; 
    }
}
```

	RTA: No se cumple el principio OCP, ya que no esta abierto a extensin, para 
	cada operacion hay un if en vez de una clase por cada operacion, siendo que 
	si hay operaciones nuevas tenga que modificar la clase calculadora

```
public interface Operacion {
    int calcular(int a, int b);
}
public class Suma implements Operacion {
    @Override
    public int calcular(int a, int b) {
        return a + b;
    }
}

public class Resta implements Operacion {
    @Override
    public int calcular(int a, int b) {
        return a - b;
    }
}
public class Calculadora {
    public int operar(Operacion operacion, int a, int b) {
        return operacion.calcular(a, b);
    }
}
```

8.

```
public class Animal {
    public void caminar() {
        System.out.println("El animal corre");
    }

    public void volar() {
        System.out.println("El animal vuela");
    }
}

public class Perro extends Animal {
    @Override
    public void volar() {
        throw new Exception("El perro no puede volar");
    }
}

public class Pajaro extends Animal {

}
```

	RTA: Se rommpe el principio LSP, ya que no cumple con que si un programa
	funciona utilizando una clase, debe poder utilizar cualquier subclase
	sin que el comportamiento afecte. En este caso lanza una especie de
	promesa "vacia" de que cada subclase de Animal va a poder volar 
	cuando realmente no es asi.

```
// Abstracción base para lo común a todo animal (si hace falta)
public abstract class Animal {
    // Atributos y comportamientos generales a todos los animales
}

// Interfaces específicas (capacidades segregadas)
public interface Caminante {
    void caminar();
}

public interface Volador {
    void volar();
}

public interface Nadador {
    void nadar();
}
public class Perro extends Animal implements Caminante, Nadador {
}
```

9.

```
public interface Estacionamiento {
	void aparcarCoche(); 
	void sacarCoche(); 
	void getCapacidad();
	double calcularTarifa(Coche coche); 
	void hacerPago(Coche coche);
}

public class EstacionamientoGratis implements Estacionamiento {
    @Override
    public void aparcarCoche() {
        System.out.println("Aparcando coche");
    }
    @Override
    public void sacarCoche() {
        System.out.println("Sacando coche");
    }
    @Override
    public void getCapacidad() {
        System.out.println("Capacidad del estacionamiento...");
    }
    @Override
    public double calcularTarifa(Coche coche) {
        return 0;
    }
    @Override
    public void hacerPago(Coche coche) {
        throw new Exception("El estacionamiento es gratis");
    }
}
```