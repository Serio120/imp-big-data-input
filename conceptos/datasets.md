<h1 align=center>🚀 ¿Qué es un Dataset en Big Data?</h1>
  
Vamos a ver **qué son los *Datasets* en Big Data**, especialmente en el contexto de **Apache Spark**, que es donde este término tiene un significado muy concreto.

---

# ¿Qué es un Dataset en Big Data?
Un **Dataset** es una **colección distribuida de datos**, similar a un DataFrame, pero con una diferencia clave:

> Un **Dataset** combina la **facilidad de uso de un DataFrame** con la **seguridad de tipos del lenguaje (type safety)**.

Es decir, es como un DataFrame **más estricto y más seguro**, pensado sobre todo para Scala y Java.

---

# 🧩 Idea principal
Un **Dataset** es una estructura de datos tabular o semiestructurada que Spark puede **distribuir**, **optimizar** y **procesar en paralelo**, pero con **tipos definidos** (clases, objetos, estructuras).

---

# 🧱 ¿En qué se diferencia de un DataFrame?

| Característica | DataFrame | Dataset |
|----------------|-----------|---------|
| Tipo de datos | Genérico (como una tabla) | Tipado (clases, objetos) |
| Lenguajes | Python, Scala, SQL | Scala y Java |
| Seguridad de tipos | No | Sí |
| Rendimiento | Muy alto | Muy alto |
| Flexibilidad | Alta | Muy alta para datos complejos |

**En resumen:**  
- **DataFrame** → fácil, rápido, flexible  
- **Dataset** → seguro, estructurado, ideal para programadores de Scala/Java  

---

# 🧠 ¿Por qué existen los Datasets?
Porque Spark quería ofrecer:
- La **simplicidad** de los DataFrames  
- La **seguridad de tipos** de los RDDs  
- El **rendimiento** del motor Catalyst  

Es una especie de “híbrido perfecto”.

---

# 🔧 ¿Qué se puede hacer con un Dataset?
Lo mismo que con un DataFrame, pero con más control:

- Filtrar  
- Mapear  
- Agrupar  
- Ordenar  
- Unir  
- Convertir a objetos  
- Validar tipos en tiempo de compilación  

Ejemplo en Scala:
```scala
case class Persona(nombre: String, edad: Int)
val ds = spark.read.json("personas.json").as[Persona]
```

Aquí Spark sabe que **edad es un Int** y **nombre es un String**, y te avisa si te equivocas.

---

# 🏗️ ¿Dónde se usan los Datasets?
Principalmente en:
- **Spark con Scala**  
- **Spark con Java**  
- Procesos donde necesitas **tipos estrictos**  
- Pipelines de Machine Learning avanzados  
- Transformaciones complejas con objetos  

En Python **no existen Datasets**, solo DataFrames.

---

# 🎯 Explicación en una frase
Un **Dataset** es una estructura de datos distribuida como un DataFrame, pero con **tipos definidos**, lo que permite trabajar con datos de forma más segura y estructurada en Spark.

---

<h1 align=center>🚀 Usos de DataSets</h1>

Vamos a desarrollar **4 puntos** de Datasets 

1. **Diferencia entre Dataset, DataFrame y RDD**  
2. **Cuándo usar cada uno**  
3. **Ejemplos prácticos en Spark**  
4. **Cómo se integran en un pipeline de Big Data**

---

# 1) 🧱 Diferencia entre Dataset, DataFrame y RDD

Aquí tienes la comparación más clara posible:

## **RDD (Resilient Distributed Dataset)**
- Es la **estructura más básica** de Spark.  
- Trabaja con **listas distribuidas** de objetos.  
- Muy flexible, pero **poco optimizada**.  
- No tiene esquema (no sabe qué columnas hay).  
- No aprovecha el optimizador Catalyst.

**Piensa en RDD como “cajas de datos sin etiqueta”.**

---

## **DataFrame**
- Es una **tabla distribuida** con columnas y tipos.  
- Muy rápido gracias al optimizador Catalyst.  
- Ideal para SQL, Python y análisis.  
- No tiene seguridad de tipos en tiempo de compilación.

**Piensa en DataFrame como “una tabla gigante tipo Excel, pero distribuida”.**

---

## **Dataset**
- Es como un DataFrame, pero con **tipos estrictos**.  
- Solo existe en **Scala y Java**.  
- Combina rendimiento + seguridad de tipos.  
- Ideal para datos complejos (objetos, clases).

**Piensa en Dataset como “una tabla gigante, pero con tipos estrictos y controlados”.**

---

# 2) 🎯 Cuándo usar cada uno

## ✔️ **Usa DataFrame cuando:**
- Trabajas en **Python**.  
- Haces análisis, agregaciones, joins, filtros.  
- Quieres máximo rendimiento sin complicarte.  
- Procesas datos tabulares (CSV, Parquet, SQL).

**Es la opción más común en Big Data.**

---

## ✔️ **Usa Dataset cuando:**
- Trabajas en **Scala o Java**.  
- Necesitas **tipos estrictos** (case classes, POJOs).  
- Procesas datos complejos (JSON profundo, estructuras anidadas).  
- Quieres evitar errores de tipo en tiempo de compilación.

---

## ✔️ **Usa RDD cuando:**
- Necesitas operaciones muy personalizadas.  
- Procesas datos no estructurados.  
- Trabajas con funciones de bajo nivel.  
- Necesitas control total del procesamiento.

**Hoy en día se usa poco, salvo casos muy específicos.**

---

# 3) 🔧 Ejemplos prácticos en Spark

## **Ejemplo DataFrame (Python)**
```python
df = spark.read.csv("ventas.csv", header=True, inferSchema=True)
df.filter(df.precio > 100).groupBy("categoria").count().show()
```

---

## **Ejemplo Dataset (Scala)**
```scala
case class Venta(id: Int, categoria: String, precio: Double)

val ds = spark.read.json("ventas.json").as[Venta]
ds.filter(_.precio > 100).groupBy("categoria").count().show()
```

Aquí Spark sabe que `precio` es un `Double`.

---

## **Ejemplo RDD**
```scala
val rdd = spark.sparkContext.textFile("datos.txt")
val palabras = rdd.flatMap(linea => linea.split(" "))
```

Muy flexible, pero menos eficiente.

---

# 4) 🔄 Cómo se integran en un pipeline de Big Data

Un pipeline típico en Spark podría verse así:

### **1. Ingesta**
- Leer datos desde S3, HDFS, bases de datos…
- Normalmente como **DataFrame**.

### **2. Limpieza**
- Quitar nulos  
- Corregir formatos  
- Normalizar columnas  
- Todo con **DataFrames** (rápido y fácil)

### **3. Transformación**
- Joins  
- Agregaciones  
- Enriquecimiento  
- También con **DataFrames** o **Datasets**

### **4. Validación**
- Comprobar tipos  
- Comprobar rangos  
- Aquí **Dataset** puede ser útil por su seguridad de tipos

### **5. Salida**
- Guardar en Parquet, Delta Lake, Data Warehouse  
- Normalmente como **DataFrame**

### **6. Machine Learning (opcional)**
- Spark ML usa **DataFrames** como formato estándar.

---

# 🎯 Resumen final en una frase
- **RDD** → bajo nivel, flexible, poco usado hoy  
- **DataFrame** → estándar, rápido, ideal para análisis  
- **Dataset** → DataFrame + tipos estrictos (solo Scala/Java)

---

Posibles añadidos:

- Un **diagrama visual** comparando los tres  
- Un ejemplo completo de **pipeline real en Spark**  
- O ayudarte a elegir cuál usar en un caso concreto que tengas en mente

## Introducción

Este documento explica cómo funcionan los ***Datasets***.

> “Tema…”  
> — Autor ...

**Fuente:** A.I.

---

Parte del contenido ha sido adaptado del trabajo de *Copilot*.