
## Instrucciones

| Instrucción        | Función                                                                |
| ------------------ | ---------------------------------------------------------------------- |
| add $1, $2, $3     | Almacena en $1 la suma de los otros dos                                |
| la $t0, mem        | Carga en $t0 la dir. de memoria donde está almacenado *mem*            |
| lw  $1, offset($2) | Carga en $1 la palabra almacenada en la dir. de memoria de $2 + offset |
| addi $t1, $t2, 4   | Almacena en $1 la suma de los otros dos                                |
| sw $1, offset($2)  | Almacena en memoria en la posición de $2 + offset.                     |
| bne $t1, $t2, Loop | Si $t1 != $t2, Salta al inicio de Loop.                                |

## Ciclos, Tiempo Ejecución

#### CPI
$$CPI = \frac{\text{Ciclos Totales}}{\text{Instrucciones Totales}}$$
>[!Tip]
>En los procesadores **monociclo**, el CPI sera 1.

#### Tiempo Ejecución

$$\text{Tiempo de Ejecución = } \text{Ciclos totales }\cdot \frac{1}{Frec}ns$$

## Optimización de Código

Hay varias formas de optimizar el código.

#### Salto retardado desde antes
El hueco de retardo se rellena con una instrucción independiente al salto. Esta instrucción es una que debía haber antes del salto.

| Instrucción           | Optimizada            |
| --------------------- | --------------------- |
| add $0, $t1, $t2      | add $0, $t1, $t2      |
| **sub $t3, $t0, $t4** | beq $t5, $t6, Salto   |
| beq $t5, $t6, Salto   | **sub $t3, $t0, $t4** |
| ???                   | sw $t3, 0($s0)        |
| sw $t3, 0($s0)        |                       |
| Salto:                | Salto                 |
#### Salto retardado desde destino
El hueco de retardo se rellena con una instrucción que se tiene que ejecutar si este es tomado. Aumenta el CPI si el salto no es efectivo.

| Instrucción         | Optimizada          |
| ------------------- | ------------------- |
| Salto:              | add $t1, $t2, $t2   |
| add $t1, $t2, $t2   | Salto:              |
| add $t0, $t1, $t2   | add $t0, $t1, $t2   |
| beq $t5, $t0, Salto | beq $t5, $t0, Salto |
| ???                 | add $t1, $t2, $t2   |
| lw $t1, 0($s0)      | lw $t1, 0($s0)      |
#### Salto retardado desde instrucciones siguientes
El hueco de retardo se rellena con una instrucción que se tiene que ejecutar si este no es tomado.

| Instrucción         | Optimizada          |
| ------------------- | ------------------- |
| add $t0, $t1, $t2   | add $t0, $t1, $t2   |
| beq $t5, $t0, Salto | beq $t5, $t0, Salto |
| ???                 | slt $t3, $t4, $t5   |
| slt $t3, $t4, $t5   | sw $t3, 0($s0)      |
| sw $t3, 0($s0)      |                     |
| Salto:              | Salto               |
| and ...             | and ...             |
#### NOP
Cuando no se puede ninguna de las anteriores.

| Instrucción         | Optimizada          |
| ------------------- | ------------------- |
| add $t0, $t1, $t2   | add $t0, $t1, $t2   |
| beq $t5, $t0, Salto | beq $t5, $t0, Salto |
| ???                 | nop                 |
# Guía de Valgrind Cachegrind

## 1. Visión general de Valgrind y Cachegrind

Valgrind es un framework de instrumentación para construir herramientas de análisis dinámico. Cachegrind es una de estas herramientas, diseñada específicamente para el perfilado de caché y predicción de saltos. Ayuda a hacer que tus programas se ejecuten más rápido identificando fallos de caché y predicciones de salto erróneas.

## 2. Uso de Cachegrind

### Ejecutando Cachegrind

Para ejecutar Cachegrind en un programa `prog`, usa:

```
valgrind --tool=cachegrind prog
```

### Interpretando la salida

Cachegrind proporciona estadísticas resumidas y datos de perfilado detallados. El resumen incluye:

- Referencias a caché de instrucciones (Ir)
- Lecturas (Dr) y escrituras (Dw) de caché de datos
- Fallos de caché de nivel 1 y último nivel para instrucciones y datos

### Usando cg_annotate

Para obtener un análisis detallado, usa:

```
cg_annotate <nombre_archivo>
```

Esto proporciona desgloses archivo por archivo y función por función del comportamiento de la caché.

## 3. Simulación de caché y saltos

Cachegrind simula una máquina con:

- Cachés de primer nivel de instrucciones y datos (I1 y D1)
- Caché unificada de último nivel (LL)
- Predictor de saltos

## 4. Opciones de línea de comandos

Las opciones de Cachegrind incluyen:

- `--cachegrind-out-file=<archivo>`: Especifica el archivo de salida
- `--cache-sim=yes|no`: Habilita/deshabilita la simulación de caché
- `--branch-sim=yes|no`: Habilita/deshabilita la simulación de predicción de saltos

## 5. Detalles de implementación

Cachegrind funciona instrumentando cada acceso a memoria e instrucción de salto en el programa que se está perfilando. Simula el comportamiento de la caché y el predictor de saltos basándose en esta instrumentación.

## 6. Precisión y limitaciones

- No tiene en cuenta la actividad del kernel u otros procesos
- Utiliza direcciones virtuales en lugar de direcciones físicas
- Los resultados pueden verse afectados por cambios en el tamaño del ejecutable o las ubicaciones de las bibliotecas compartidas

## Optimización de código

### Intercambio de Bucles

En algunos programas, los bucles anidados acceden a los datos en memoria de manera desordenada, lo que reduce el rendimiento. Cambiar el orden de los bucles puede hacer que el acceso sea secuencial y más rápido.

• **FORTRAN** almacena datos por columnas, por lo que conviene acceder a ellos columna a columna.

• **C/C++** almacena datos por filas, así que **es mejor acceder fila a fila**.

Si los bucles acceden a una matriz de forma no consecutiva (por ejemplo, columna a columna en C/C++), cambiar el orden puede acelerar el programa. Este cambio debe respetar la lógica del código para que los resultados sigan siendo correctos.

#### Sin Intercambio
```c 
for(j=0; j<N; j++)
	for(i=0; i<M; i++)
		x[i][j] = 2*x[i][j];
```
#### Con Intercambio
```c 
for(i=0; i<M; i++)
	for(j=0; j<N; j++)
		x[i][j] = 2*x[i][j];
```

> En el primer caso, estamos accediendo primero a la lectura de columnas, pero en C los datos se almacenan por filas. Por eso, intercambiar los bucles es beneficioso.

### Fusión de Arrays
Algunos programas usan varios arrays con los mismos índices al mismo tiempo. Si estos arrays ocupan las mismas posiciones en la caché, pueden ocurrir fallos repetidos. **Esto pasa especialmente si los arrays tienen tamaño potencia de dos y están declarados uno tras otro en el código**.

Una solución es fusionar los arrays en uno solo. En C, esto se logra usando un array de estructuras en lugar de varios arrays separados. También es útil fusionar arrays cuando se acceden juntos de forma no secuencial, como al usar indirectamente un array de índices (por ejemplo, a[b[i]]) o accesos separados por distancias mayores al tamaño de la línea de caché.

#### Sin optimización
```c
int val[SIZE];
int key[SIZE];
```
#### Con optimización
```c
struct reg{
	int val;
	int key;
}
struct reg union_array[SIZE];
```

### Fusión de Bucles
En programas numéricos, es común realizar varias operaciones sobre los mismos datos usando múltiples bucles. Cuando los arrays son más grandes que las cachés, los datos cargados durante un bucle suelen desaparecer antes de que se vuelvan a usar en otro.

Una solución es **fusionar bucles consecutivos** siempre que sea posible y no afecte el resultado del programa. Esto permite reutilizar los datos en la caché, mejorando el rendimiento.

#### Sin optimización
```c
for( i=0; i<N; i++)
	for( j=0; j<N; j++)
		a[i][j] = 1/b[i][j] * c[i][j];

for( i=0; i<N; i++)
	for( j=0; j<N; j++)
		d[i][j] = a[i][j] + c[i][j];
```
#### Con optimización
```c
for( i=0; i<N; i++)
	for( j=0; j<N; j++) {
		a[i][j] = 1/b[i][j] * c[i][j];
		d[i][j] = a[i][j] + c[i][j];
	}
```

### Rellenado de Arrays
Cuando no es posible acceder a un array de forma secuencial, pueden ocurrir fallos de conflicto en la caché si las dimensiones del array y su disposición en memoria no están bien ajustadas.

Un ejemplo común es el acceso por columnas a una matriz en C, donde los datos se almacenan por filas. **Si el número de elementos por fila es una potencia de dos**, los elementos de una columna pueden quedar mapeados al mismo conjunto de la caché. Esto provoca que, al procesar una columna, las líneas de caché asociadas sean expulsadas continuamente, impidiendo su reutilización para la siguiente columna.

**Solución:** agregar columnas de relleno al array. Por ejemplo, si una matriz es m[4][8], podemos declararla como m[4][10] pero usar solo las 8 primeras columnas. Este relleno distribuye las líneas de caché de las columnas entre distintos conjuntos, permitiendo reutilizar los datos al procesar la siguiente columna.

#### Sin optimización
```c
float m[4][8];

// recorre por columnas... 
for(j=0; j<8; j++)
	for(i=0; i<4; i++)
		q = q * 1.2 + m[i][j];
```
#### Con optimización
```c
float m[4][10]; // cambiamos para q no sea multiplo de 4

// recorre por columnas... 
for(j=0; j<8; j++)
	for(i=0; i<4; i++)
		q = q * 1.2 + m[i][j];
```
### Partición de Bloques (*Tiling*)
La partición en bloques es una técnica para mejorar el uso de la caché al dividir los datos en bloques más pequeños que se reutilizan antes de ser reemplazados.

Por ejemplo, en un programa que accede a varios arrays, algunos por filas y otros por columnas, ni almacenar los datos según filas o columnas ni cambiar el orden de los bucles es suficiente, ya que ambos tipos de acceso ocurren en cada iteración.

**Solución:** procesar los datos en bloques más pequeños, trabajando con una parte de filas y columnas a la vez. **Esto mantiene los datos necesarios en la caché durante más tiempo, reduciendo el número de accesos a la memoria principal y mejorando el rendimiento.**
#### Sin optimización
```c
for(x=0; x<N; x++)
	for(y=0; y<N; y++)
		acum = c[x][y];
		for(z=0; z<N; z++)
			acum += a[x][z] * b[z][y];
		c[x][y] = acum;
```
#### Con optimización
```c
for(yy=0; yy<N; yy+=T)
	for(zz=0; zz<N; z+=T)
		for(x=0; x<N; x++)
			for(y=yy; y<min(yy+T,N); y++)
				acum = c[x][y];
				for(z= zz; z<min(zz+T, N); z++)
					acum += a[x][z] * b[z][y];
				c[x][y] = acum;
```

> Se trabaja con datos de tamaño más pequeño, esto debido al uso del tamaño T y calculando el mínimo entre eso + variable y N.

### Distribución de datos en estructuras
Las estructuras y objetos permiten agrupar datos relacionados. Sin embargo, los compiladores colocan los campos en memoria según el orden en que se definen, lo que puede afectar el rendimiento si algunos campos se usan mucho y otros muy poco.

Para mejorar la eficiencia:

- Declara juntos los campos que se usan con frecuencia en las mismas operaciones.
- Esto aumenta la **localidad espacial**, haciendo que los datos necesarios estén en la misma línea de memoria y se carguen juntos en la caché.

Por ejemplo, si un bucle usa los campos *coef* y v repetidamente, es mejor colocarlos consecutivamente en la estructura. Así, ambos campos tienen más probabilidades de estar en la misma línea de caché, acelerando el acceso y mejorando el tiempo de ejecución.

#### Sin optimización
```c
struct datos{
	float coef, b, c;
	char id[20];
	float posicion[3];
	double v;
};

struct datos ar[N];

for(i=0; i<M; i++)
	r += ar[d[i]].coef * ar[d[i]].v;
```
#### Con optimización
```c
struct datos{
	double v;
	float coef, b, c;
	char id[20];
	float posicion[3];
};

struct datos ar[N];

for(i=0; i<M; i++)
	r += ar[d[i]].coef * ar[d[i]].v;
```
#### Con ++ optimización
```c
struct datos1{
	double v;
	float coef;
};

struct datos2{
	float b, c;
	char id[20];
	float posicion[3];
}

struct datos1 ar[N];
struct datos2 ar2[N];

for(i=0; i<M; i++)
	r += ar[d[i]].coef * ar[d[i]].v;
```