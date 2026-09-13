
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
	private Animal habitante;
	
	public Jaula(Animal habitante) {
		this.habitante = habitante;
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
            throws MontoInsuficienteException("No se puede retirar. Fondos insuficientes.");
        }


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
            throws StockInsuficienteException("No se puede retirar. Stock insuficiente.");
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

5.

```
public class Juego {
    public void saludar() {
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