<h1 align=center>Spark: DataFrame vs. Dataset Visual</h1>

¡Claro! El esquema de la imagen muestra conceptos fundamentales de **Apache Spark**, específicamente la relación entre los datos en bruto, los **DataFrames** y los **Datasets** fuertemente tipados usando `case classes`.

Aquí tienes una representación mucho más limpia y visual de esos conceptos para que los entiendas de un vistazo:

---

## 🏗️ Estructura de Datos en Spark

### 1. El Esquema (Schema)
Es el "molde" que define qué tipo de datos contiene cada columna.
*   **ID:** Entero (`Int`)
*   **Nombre:** Cadena (`String`)
*   **Sueldo:** Decimal (`Double`)



---

### 2. De DataFrame a Dataset
El esquema visualiza la diferencia jerárquica: un **DataFrame** es simplemente un **Dataset de objetos tipo "Row"** (genérico), mientras que un **Dataset** utiliza una clase específica.

| Concepto | Representación Visual | Características |
| :--- | :--- | :--- |
| **Row** | `[ 1, "Ana", 2000.0 ]` | Una fila genérica sin tipos definidos explícitamente. |
| **DataFrame** | **Dataset[Row]** | Una tabla organizada con esquema, pero "ciega" al tipo de objeto. |
| **Dataset** | **Dataset[Venta]** | Una tabla donde cada fila es una instancia de una `case class`. |

---

### 3. Case Class: El ADN del Dataset
En Scala/Spark, usamos la `case class` para darle identidad a los datos. Según tu ejemplo:

```scala
// Definición de la estructura
case class Venta(id: Int, nombre: String, sueldo: Double)

// Creación de objetos (Instanciación)
val venta1 = Venta(1, "Ana", 2000.0)
val venta2 = Venta(2, "Pedro", 1500.0)
```

---

### 🔄 Resumen del Flujo
1.  **Datos brutos:** Entran como filas genéricas (`Row`).
2.  **Schema:** Se aplica para dar orden (Columnas: ID, Nombre, Sueldo).
3.  **DataFrame:** Los datos ya tienen forma de tabla.
4.  **Dataset (Case Class):** Los datos se convierten en objetos programables (`Venta`), lo que permite detectar errores de escritura antes de ejecutar el código.

<img src="/assets/modelos_datos_spark.png" height=40% width=40%>


---
---

Para profundizar en la distinción entre **DataFrame** y **Dataset** en Apache Spark, es fundamental entender que, aunque ambos forman parte de la misma API, su comportamiento varía drásticamente en cuanto a la seguridad del código y el rendimiento.

A continuación, presento un análisis detallado basado en la lógica del esquema de la imagen.

---

### 1. La Diferencia Fundamental: Tipado de Datos

| Característica | DataFrame | Dataset |
| :--- | :--- | :--- |
| **Definición** | Es un Dataset organizado en columnas con nombre: `Dataset[Row]`. | Es una colección de objetos de dominio fuertemente tipados. |
| **Seguridad de Tipo** | **No tipado.** Los errores se detectan en **tiempo de ejecución**. | **Tipado fuerte.** Los errores se detectan en **tiempo de compilación**. |
| **Estructura** | Basado en objetos `Row` genéricos. | Basado en una `case class` (en Scala) o clases Java. |
| **Análisis de Errores** | Si escribes mal el nombre de una columna, el programa fallará cuando ya esté corriendo. | Si intentas acceder a un campo que no existe en la clase, el código no compilará. |


---

### 2. Ejemplos de Uso y Comparativa de Código

Imagina que queremos calcular el sueldo total del archivo mencionado en la imagen **20260504_115609.jpg**.

#### A. Usando DataFrame (Enfoque Genérico)
Es ideal para análisis rápidos o cuando los datos no tienen una estructura fija de antemano.
```scala
// El DataFrame no sabe qué hay dentro, solo que hay columnas
val df = spark.read.json("ventas.json")

// Error común: Si escribo "Sueldos" en vez de "Sueldo", Spark falla al ejecutar
df.select("Sueldo").where($"Sueldo" > 1800).show()
```

#### B. Usando Dataset (Enfoque Seguro)
Es el preferido para aplicaciones robustas donde quieres evitar errores tontos de sintaxis.
```scala
// Definimos el contrato (como se ve en watermarked_img_11880710886772610449.png)
case class Venta(id: Int, nombre: String, sueldo: Double)

// Transformamos el DataFrame en un Dataset tipado
val ds = df.as[Venta]

// Aquí el compilador sabe que existe .sueldo. 
// Si escribes .salary, el IDE te avisará antes de lanzar el proceso.
ds.filter(v => v.sueldo > 1800).map(v => v.nombre).show()
```

---

### 3. ¿Cuándo usar cada uno?

1.  **Usa DataFrame cuando:**
    *   Estés usando **Python (PySpark)** o **R**, ya que el concepto de "Dataset" tipado es exclusivo de Scala y Java.
    *   Necesites el máximo rendimiento de serialización (Spark optimiza muy bien los objetos `Row`).
    *   Realices transformaciones sencillas como `select`, `groupBy` o `filter`.

2.  **Usa Dataset cuando:**
    *   Trabajes en **Scala** y quieras aprovechar la potencia de los tipos.
    *   Tu lógica de negocio sea compleja y requiera funciones personalizadas (`map`, `flatMap`) que operen sobre objetos.
    *   Busques que el código sea más legible y fácil de mantener a largo plazo.

### 4. El motor bajo el capó: Catalyst y Tungsten
Independientemente de cuál elijas, ambos utilizan el optimizador **Catalyst** para generar el plan de ejecución más eficiente. Sin embargo, los DataFrames suelen ser ligeramente más rápidos porque Spark entiende mejor la estructura interna de un `Row` que la de un objeto arbitrario de una clase de usuario.