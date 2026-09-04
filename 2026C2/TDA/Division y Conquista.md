# División y Conquista (D&C)

## 1. Fundamentos Matemáticos y el Teorema Maestro General

El paradigma de **División y Conquista (D&C)** constituye una de las estrategias más potentes en el diseño de algoritmos, fundamentada en un ciclo de tres etapas:

1. **Dividir:** descomponer el problema en subproblemas de menor escala.
2. **Resolver:** resolver cada subproblema de forma recursiva.
3. **Combinar:** combinar dichas soluciones para obtener el resultado global.

Su importancia estratégica radica en la reducción de la complejidad asintótica; al descomponer estructuras masivas en unidades manejables, es posible transitar de costos cuadráticos o cúbicos a eficiencias logarítmicas o cuasilineales que los enfoques iterativos ingenuos no pueden alcanzar.

### Teorema Maestro

Para evaluar formalmente estos algoritmos recursivos, se utiliza el **Teorema Maestro**, aplicado a recurrencias de la forma:

$$
T(n) = aT\left(\frac{n}{b}\right) + f(n)
$$

Este marco analítico exige condiciones rígidas:

- El coeficiente $a$ (número de llamados recursivos) debe ser un número natural $a \geq 1$.
- La base $b$ (proporción de reducción del tamaño de entrada) debe ser una constante real $b > 1$.
- El caso base debe ser constante:

$$
T(1) = \Theta(1)
$$

### Versión Generalizada

En escenarios avanzados donde la función de costo de combinación $f(n)$ no es una potencia polinómica simple, se requiere la versión generalizada del teorema.

Consideremos la recurrencia:

$$
T(n) = 2T\left(\frac{n}{2}\right) + \log n
$$

Aquí:

$$
\log_b a = \log_2 2 = 1
$$

Analizando la relación entre $f(n)$ y $n^{\log_b a}$, observamos que:

$$
f(n) = \log n
$$

Mediante el método de acotación/sándwich, demostramos que para cualquier $\epsilon > 0$ —por ejemplo, $\epsilon = 0.1$— se cumple que:

$$
\log n = O\left(n^{1-0.1}\right)
$$

dado que el crecimiento logarítmico es estrictamente menor que cualquier potencia polinomial positiva.

Al cumplirse que:

$$
f(n) = O\left(n^{\log_b a-\epsilon}\right)
$$

se aplica el **Caso 1** del teorema, permitiendo concluir con rigor que:

$$
T(n) = \Theta(n)
$$

Esta precisión analítica es el cimiento necesario para evaluar la optimización aritmética antes de abordar estructuras espaciales.

---

## 2. Optimización Aritmética: Multiplicación de Enteros Grandes (Algoritmo de Karatsuba-Offman)

La eficiencia en la multiplicación de grandes volúmenes de datos numéricos es un pilar fundamental para la criptografía y el cálculo científico. El algoritmo tradicional de la escuela primaria posee un costo de:

$$
O(n^2)
$$

resultando prohibitivo para operandos de miles de dígitos.

Una alternativa superior es la descomposición algebraica en base 2.

### Descomposición

El número se divide en una **"mitad alta"** ($x_1$) y una **"mitad baja"** ($x_0$).

Operativamente:

- Obtener $x_0$ se logra mediante una máscara `AND` contra:

$$
2^{n/2}-1
$$

- Obtener $x_1$ se logra mediante un desplazamiento de bits a la derecha (`shift right`) de $n/2$ posiciones.

Una división directa bajo D&C resultaría en **4 multiplicaciones recursivas**, manteniendo la complejidad en:

$$
O(n^2)
$$

### Algoritmo de Karatsuba-Offman

El algoritmo de **Karatsuba-Offman** reduce los llamados recursivos de 4 a 3 mediante un "truco" algebraico.

Definiendo:

$$
P = (x_1+x_0)(y_1+y_0)
$$

se observa que el término intermedio necesario para la combinación:

$$
x_1y_0+x_0y_1
$$

puede obtenerse como:

$$
P-x_1y_1-x_0y_0
$$

Dado que las restas operan en tiempo lineal, la recurrencia resulta en:

$$
T(n)=3T\left(\frac{n}{2}\right)+O(n)
$$

cuya resolución por Teorema Maestro deriva en una complejidad de:

$$
O(n^{\log_2 3}) \approx O(n^{1.58})
$$

### Traza con Caso Práctico

Para:

$$
x=1234,\qquad y=5678
$$

#### 1. Descomposición ($n=4$)

$$
x_1=12,\qquad x_0=34
$$

$$
y_1=56,\qquad y_0=78
$$

#### 2. Multiplicaciones Recursivas

- $m_1=x_1\cdot y_1=12\cdot56=672$
- $m_2=x_0\cdot y_0=34\cdot78=2652$
- $m_3=(x_1+x_0)\cdot(y_1+y_0)=46\cdot134=6164$

#### 3. Cálculo del Término Intermedio

$$
6164-672-2652=2840
$$

#### 4. Recombinación Final

$$
672\cdot10^4+2840\cdot10^2+2652
$$

$$
=6\,720\,000+284\,000+2\,652
$$

$$
=7\,006\,652
$$

Esta eficiencia demuestra que D&C trasciende la aritmética simple para optimizar también estructuras espaciales complejas.

---

## 3. Geometría Computacional: Vértice Extremo de un Polígono Convexo

La geometría computacional es vital en la representación de imágenes y teoría de juegos, donde la convexidad facilita optimizaciones críticas.

Definimos un **polígono convexo** como una figura monotónica donde ninguna recta la corta en más de dos puntos y sus ángulos interiores son:

$$
\leq 180^\circ
$$

Para hallar el vértice extremo respecto a un eje $u$, utilizamos los vectores de cambio de proyección:

$$
e_{v_i}=v_{i+1}-v_i
$$

### Búsqueda Logarítmica

El algoritmo emplea una búsqueda logarítmica basada en una matriz de decisiones sobre tres puntos:

- Inicio ($A$)
- Fin ($B$)
- Medio ($C$)

El *casework* sobre las proyecciones de los vectores $e_A$ y $e_C$ permite descartar la mitad del espacio de búsqueda:

- **Si** la proyección de $e_C$ sobre $u$ es positiva y la de $e_A$ es coherente con un crecimiento hacia $C$, el extremo reside en el segmento $[C,B]$.
- **Si** la proyección de $e_C$ sobre $u$ es negativa, el extremo debe encontrarse en el segmento $[A,C]$.

Esta lógica de descarte unívoco define la recurrencia:

$$
T(n)=T\left(\frac{n}{2}\right)+O(1)
$$

y, por lo tanto:

$$
T(n)=O(\log n)
$$

### Ejemplo

En un pentágono:

$$
\{A,B,C,D,E\}
$$

en sentido antihorario, si buscamos el máximo vertical y el vector en $C$ apunta hacia arriba, el rango se reduce instantáneamente a:

$$
\{C,D,E\}
$$

convergiendo al extremo en pasos logarítmicos.

---

## 4. Resolución del Problema del Par de Puntos Más Cercanos (Closest Pair)

El enfoque de fuerza bruta:

$$
O(n^2)
$$

es ineficiente para grandes conjuntos de datos espaciales.

La estrategia de D&C planar optimiza esto a:

$$
O(n\log n)
$$

Se requiere un pre-ordenamiento en $P_x$ y $P_y$ para mantener la eficiencia de la etapa de combinación en tiempo lineal:

$$
O(n)
$$

sin re-ordenamientos internos.

### Franja Central

Tras dividir el plano y hallar la distancia mínima $d$ en las sub-regiones, se analiza la **"Franja Central"** de ancho $2d$.

La optimización reside en dividir esta franja en cajas de tamaño:

$$
\frac{d}{2}\times\frac{d}{2}
$$

### Demostración por Reducción al Absurdo

Si una caja contuviera dos o más puntos, la distancia máxima entre ellos sería:

$$
\frac{d}{\sqrt{2}}
$$

Dado que:

$$
\frac{d}{\sqrt{2}}<d
$$

esto contradice la premisa de que $d$ es la distancia mínima ya encontrada en las mitades.

Por lo tanto, cada caja contiene a lo sumo un punto.

Geométricamente, esto garantiza que basta con comparar cada punto contra los siguientes **15** en la lista ordenada por $Y$ dentro de la franja.

El algoritmo total se mantiene en:

$$
O(n\log n)
$$

vinculando la búsqueda planar con la optimización de matrices de alta dimensión.

---

## 5. Multiplicación de Matrices y el Algoritmo de Strassen

El costo cúbico:

$$
O(n^3)
$$

de la multiplicación de matrices es un cuello de botella en la computación moderna.

Una división directa en 8 submatrices de tamaño:

$$
\frac{n}{2}\times\frac{n}{2}
$$

no altera la complejidad:

$$
T(n)=8T\left(\frac{n}{2}\right)+O(n^2)
$$

por lo que:

$$
T(n)=O(n^3)
$$

### Algoritmo de Strassen

El **Algoritmo de Strassen** revoluciona este proceso al reducir los llamados recursivos de 8 a 7.

Esto se logra mediante la definición de siete matrices intermedias:

$$
M_1,M_2,\dots,M_7
$$

las cuales son combinaciones lineales de las submatrices originales.

La lógica transforma multiplicaciones costosas en operaciones de suma y resta más económicas.

La recurrencia resultante es:

$$
T(n)=7T\left(\frac{n}{2}\right)+O(n^2)
$$

derivando en una complejidad de:

$$
O(n^{\log_2 7})\approx O(n^{2.81})
$$

En el ámbito profesional, Strassen opera como una "caja negra" de alto rendimiento para el procesamiento de estructuras combinatorias y matriciales.

---

## 6. Métricas de Desemejanza: Conteo de Inversiones

El conteo de inversiones es una métrica de desemejanza fundamental en sistemas de recomendación.

Se adapta la etapa de fusión (*Merge*) de **MergeSort** para contabilizar inversiones simultáneamente al ordenamiento.

Si al fusionar un elemento de la derecha se posiciona antes que elementos restantes en la izquierda, se suma dicha cantidad de "saltos" al contador global, manteniendo la complejidad en:

$$
O(n\log n)
$$

### Resolución del Arreglo

Consideremos el arreglo:

$$
[6,8,7,2,1,5,3,4]
$$

#### 1. Nivel Base

- $[6,8]\rightarrow 0$ inv.
- $[7,2]\rightarrow 1$ inv.
- $[1,5]\rightarrow 0$ inv.
- $[3,4]\rightarrow 0$ inv.

#### 2. Fusiones Intermedias

**Izquierda:**

Fusión de $[6,8]$ y $[2,7]$.

- El 2 salta a 6 y 8: **+2**
- El 7 salta a 8: **+1**

Total del nivel:

$$
3
$$

Acumulado izquierdo:

$$
1+3=4
$$

**Derecha:**

Fusión de $[1,5]$ y $[3,4]$.

- El 3 salta a 5: **+1**
- El 4 salta a 5: **+1**

Total del nivel:

$$
2
$$

Acumulado derecho:

$$
0+2=2
$$

#### 3. Fusión Final

Al combinar:

$$
[2,6,7,8]
$$

y:

$$
[1,3,4,5]
$$

se obtiene:

- El 1 salta 4 elementos: **+4**
- El 3 salta 3 elementos: **+3**
- El 4 salta 3 elementos: **+3**
- El 5 salta 3 elementos: **+3**

Inversiones de fusión:

$$
4+3+3+3=13
$$

**Total general:**

$$
4+2+13=19
$$

Por lo tanto, el arreglo posee **19 inversiones**.

---

## 7. Caso de Aplicación: Caminos de Longitud $K$ en Grafos

Para determinar caminos de longitud exacta $K$ en un grafo, se recurre a la potencia $K$-ésima de la matriz de adyacencia $A$.

La técnica empleada es la **Potenciación Binaria** mediante D&C.

### Potenciación Binaria

Si $K$ es par:

$$
A^K=\left(A^{K/2}\right)^2
$$

Si $K$ es impar:

$$
A^K=A\cdot\left(A^{(K-1)/2}\right)^2
$$

### ¿Por qué no se aplica el Teorema Maestro?

Es imperativo señalar que el **Teorema Maestro no es aplicable** en este escenario por tres razones fundamentales:

1. **Invariabilidad de la dimensión:** El tamaño de la matriz $n$ permanece constante; lo que se divide es el exponente $K$.
2. **Multivariabilidad:** El costo depende de dos variables independientes ($n$ y $K$).
3. **Naturaleza del caso base:** El caso base no es de tamaño constante $O(1)$, sino que depende de la dimensión $n$ de la matriz.

### Complejidad

La complejidad total se deduce como el producto de:

- La profundidad del árbol recursivo:

$$
O(\log K)
$$

- El costo de la multiplicación de matrices en cada nivel.

Utilizando el algoritmo de Strassen:

$$
O(n^{\log_2 7})
$$

la complejidad final es:

$$
O(n^{2.81}\cdot\log K)
$$

Esta resolución subraya la versatilidad técnica de **División y Conquista** para optimizar problemas de alta complejidad computacional.