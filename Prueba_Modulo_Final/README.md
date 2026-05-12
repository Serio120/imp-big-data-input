
# 📋 GUÍA COMPLETA: PROYECTO FINAL BIG DATA + POWER BI

## 🎯 Resumen ejecutivo

Tu proyecto es un **pipeline ETL end-to-end** que va desde datos crudos hasta un dashboard ejecutivo en Power BI. Tienes **3 opciones de entorno**, pero todos entregan lo mismo.

**Duración estimada:** 3-5 semanas (depende del entorno elegido)

---

# ⚠️ PASO 0: ELIGE TU ENTORNO

Antes de empezar, **declara uno de estos**:

| Vía | Entorno | Pros | Contras |
|-----|---------|------|---------|
| **A** | Azure Databricks | Profesional, escalable | Requiere cuenta Azure ($$) |
| **B** | Jupyter + Almond Kernel | Fácil de instalar localmente | Menos profesional |
| **C** | IntelliJ + spark-submit | Máximo control | Más setup, más complejo |

**Mi recomendación:** Si eres principiante → **Vía B (Jupyter)**

---

# 🚀 PASO 1: DEFINE EL DOMINIO DE NEGOCIO

### Qué debes hacer:

1. **Elige una industria** (no es prescriptiva, pero elige UNA):
   - 🛍️ Comercio electrónico
   - 📦 Logística
   - 🏥 Sanidad
   - 🎮 Videojuegos / Entretenimiento
   - 🚗 Movilidad / Transporte
   - 🎓 Educación
   - 💰 Finanzas
   - ⚽ Deportes

2. **Define tu empresa ficticia** (máximo 5 líneas):
   ```
   Ejemplo:
   "TechStore es un e-commerce de electrónica que vende en 5 países.
   Tienen 1000 productos activos, 50k clientes registrados.
   Problema: No saben qué productos tienen mejor margen, en qué regiones 
   concentrar inversión, ni cómo predecir demanda estacional."
   ```

3. **Plantea 3-5 preguntas de negocio concretas**:
   ```
   Q1: ¿Cuáles son los 10 productos con mayor ingresos en el último trimestre?
   Q2: ¿Cuál es el margen bruto promedio por categoría de producto?
   Q3: ¿Cómo evoluciona la venta por mes? ¿Hay estacionalidad?
   Q4: ¿Qué regiones generan más ingresos por cliente?
   Q5: ¿Cuál es la tasa de devolución por categoría?
   ```

4. **Dibuja el flujo de datos**:
   ```
   [Datos CSV] ──┐
                 │
   [Datos JSON] ─┼──→ [Spark Bronze] ──→ [Spark Silver] ──→ [Spark Gold]
                 │                                              ↓
   [Datos BD]   ─┘                                        [CSV exportado]
                                                               ↓
                                                         [Power BI Desktop]
                                                               ↓
                                                          [Dashboard PDF]
   ```

### 📝 Documentación requerida:
- Crea un archivo `informe/00_definicion_negocio.md` con esta info
- Será la base de tu informe final

---

# 📊 PASO 2: DISEÑA LAS FUENTES DE DATOS

### Qué debes hacer:

1. **Crea al menos 2 fuentes de datos diferentes**:

   **Opción A (Recomendado): Generar datos sintéticamente**
   ```
   Ejemplo con Scala/Spark:
   
   // Tabla 1: Transacciones de venta (CSV)
   - order_id, customer_id, product_id, quantity, price, order_date
   - 100,000 filas
   
   // Tabla 2: Información de productos (JSON)
   - product_id, name, category, cost, supplier_id
   - 1,000 productos
   
   // Tabla 3: Datos de clientes (Parquet)
   - customer_id, name, email, country, signup_date
   - 10,000 clientes
   ```

   **Opción B: Descargar datasets públicos**
   - Kaggle: https://www.kaggle.com/datasets
   - Google Dataset Search: https://datasetsearch.research.google.com/

2. **Introduce intencionalmente problemas de calidad**:
   ```
   ✗ 5% de valores nulos en columnas importantes
   ✗ 100 registros duplicados
   ✗ Fechas en formatos inconsistentes (YYYY-MM-DD vs DD/MM/YYYY)
   ✗ Precios negativos o letras en columnas numéricas
   ✗ Espacios extra en nombres
   ✗ Valores fuera de rango (edad = 999, cantidad = -50)
   ```

3. **Documenta qué problemas introdujiste**:
   ```
   Archivo: datos/PROBLEMAS_CALIDAD.txt
   
   Fuente: sales.csv
   - Línea 5234: price = "no aplica" (debería ser número)
   - Líneas 1000-1050: customer_id duplicado (100 registros iguales)
   - 5% de valores nulos en quantity
   
   Fuente: products.json
   - Fechas en formato DD/MM/YYYY (vs ISO 8601 en CSV)
   - 200 registros sin category
   ```

### 📁 Estructura de carpetas:
```
proyecto_final_[tu_nombre]/
├── datos/
│   ├── sales.csv                    (tu fuente 1)
│   ├── products.json                (tu fuente 2)
│   ├── customers.parquet            (opcional: fuente 3)
│   ├── PROBLEMAS_CALIDAD.txt        (documentar qué introdujiste)
│   └── generate_data.scala          (script que generó los datos)
```

### 📊 Volumen mínimo:
- Tabla principal: **≥ 10,000 filas**
- Datos deben ocupar **≥ 10 MB** (para que Spark tenga sentido)

---

# 🏗️ PASO 3: CONSTRUYE LA CAPA BRONZE (Ingesta Raw)

### Qué debes hacer:

1. **Lee las fuentes SIN transformación**:
   ```scala
   // Ejemplo en Scala/Spark
   
   import org.apache.spark.sql.SparkSession
   
   val spark = SparkSession.builder()
     .appName("ETL-Bronze")
     .getOrCreate()
   
   // Leer CSV
   val salesDF = spark.read
     .option("header", "true")
     .option("inferSchema", "true")
     .csv("datos/sales.csv")
   
   // Leer JSON
   val productsDF = spark.read
     .json("datos/products.json")
   
   // Leer Parquet
   val customersDF = spark.read
     .parquet("datos/customers.parquet")
   ```

2. **Añade columnas de auditoría** a cada tabla:
   ```scala
   import org.apache.spark.sql.functions._
   from_unixtime(unix_timestamp(), "yyyy-MM-dd HH:mm:ss")
   
   val salesBronze = salesDF
     .withColumn("ingestion_timestamp", current_timestamp())
     .withColumn("source_file", lit("sales.csv"))
   
   val productsBronze = productsDF
     .withColumn("ingestion_timestamp", current_timestamp())
     .withColumn("source_file", lit("products.json"))
   ```

3. **Escribe en Delta Lake** (capa Bronze):
   ```scala
   salesBronze.write
     .mode("overwrite")
     .format("delta")
     .save("delta/bronze/sales")
   
   productsBronze.write
     .mode("overwrite")
     .format("delta")
     .save("delta/bronze/products")
   ```

4. **Registra en logs**:
   ```
   Archivo: etl/00_bronze_log.txt
   
   ✓ Tabla: bronze/sales
     - Filas ingeridas: 100,234
     - Schema: order_id, customer_id, product_id, quantity, price, 
               order_date, ingestion_timestamp, source_file
   
   ✓ Tabla: bronze/products
     - Filas ingeridas: 1,000
     - Schema: product_id, name, category, cost, supplier_id,
               ingestion_timestamp, source_file
   ```

### 📁 Carpeta de código:
```
etl/
├── 01_bronze.scala     (o .ipynb si usas Jupyter)
└── config.scala        (paths, variables de config)
```

---

# 🧹 PASO 4: CONSTRUYE LA CAPA SILVER (Limpieza y Normalización)

### Qué debes hacer:

1. **Lee las tablas Bronze**:
   ```scala
   val salesBronze = spark.read.format("delta")
     .load("delta/bronze/sales")
   ```

2. **Aplica limpieza documentada**:

   **A) Eliminar o imputar nulos**:
   ```scala
   // Estrategia: Eliminar filas donde order_id es nulo (PK)
   // Estrategia: Imputar quantity con 1 (venta mínima)
   // Estrategia: Eliminar filas donde customer_id es nulo (FK)
   
   val salesCleaned = salesBronze
     .filter(col("order_id").isNotNull)
     .fillna(Map("quantity" -> 1))
     .filter(col("customer_id").isNotNull)
   ```

   **B) Deduplicar**:
   ```scala
   val salesDeduplicated = salesCleaned
     .dropDuplicates(Array("order_id", "customer_id", "order_date"))
   ```

   **C) Corregir tipos de datos y formatos**:
   ```scala
   import org.apache.spark.sql.functions.{to_date, col}
   
   val salesTyped = salesDeduplicated
     .withColumn("price", col("price").cast("decimal(10,2)"))
     .withColumn("quantity", col("quantity").cast("int"))
     .withColumn("order_date", to_date(col("order_date"), "yyyy-MM-dd"))
     .filter(col("price") > 0)  // Eliminar precios negativos
     .filter(col("quantity") > 0)  // Eliminar cantidades negativas
   ```

   **D) Normalizar nombres de columnas**:
   ```scala
   val salesSilver = salesTyped
     .withColumnRenamed("product id", "product_id")
     .withColumnRenamed("Order ID", "order_id")
     .withColumnRenamed("Customer Name", "customer_name")
     // Eliminar espacios y convertir a minúsculas
     .select(salesTyped.columns.map(name =>
       col("`" + name + "`").as(name.toLowerCase.replace(" ", "_"))
     ): _*)
   ```

3. **Aplica al menos 1 JOIN**:
   ```scala
   val productsSilver = /* limpieza similar */
   
   val salesWithProducts = salesSilver
     .join(productsSilver, col("product_id") === col("product_id"), "inner")
     .select(
       col("sales.order_id"),
       col("sales.customer_id"),
       col("sales.quantity"),
       col("sales.price"),
       col("products.product_id"),
       col("products.category"),
       col("products.cost")
     )
   ```

4. **Escribe en Delta Silver**:
   ```scala
   salesWithProducts.write
     .mode("overwrite")
     .format("delta")
     .save("delta/silver/sales_enriched")
   ```

### 📝 Documentación:
```
Archivo: etl/01_silver_log.txt

Tabla: silver/sales_enriched

Limpieza aplicada:
  1. Nulos: 
     - Eliminadas 45 filas con order_id nulo (PK)
     - Imputada cantidad con 1 en 230 registros
     - Eliminadas 120 filas con customer_id nulo (FK)
  
  2. Duplicados:
     - Eliminados 100 registros duplicados (mismo order_id + date)
  
  3. Tipos y formatos:
     - Convertida price a decimal(10,2)
     - Convertida quantity a int
     - Convertida order_date a DATE (formato YYYY-MM-DD)
     - Eliminados 5 registros con price < 0
     - Eliminados 10 registros con quantity <= 0
  
  4. Nombres: Normalizados a snake_case, minúsculas
  
  5. Join: 
     - Joinada tabla sales con products
     - Método: INNER JOIN en product_id
     - Filas resultado: 99,100 (99,234 - 134 sin producto)

Filas finales: 99,100
Reducción: 234 → 99,100 (57.6% de calidad mejorada)
```

---

# 📈 PASO 5: CONSTRUYE LA CAPA GOLD (KPIs y Agregaciones)

### Qué debes hacer:

1. **Lee la tabla Silver**:
   ```scala
   val salesSilver = spark.read.format("delta")
     .load("delta/silver/sales_enriched")
   ```

2. **Calcula 5+ KPIs** que responden tus preguntas del Paso 1:

   **KPI 1: Ingresos totales** (Métrica de volumen)
   ```scala
   val totalRevenue = salesSilver
     .withColumn("revenue", col("quantity") * col("price"))
     .select(sum("revenue").as("total_revenue"))
   
   // Resultado: $5,234,567
   ```

   **KPI 2: Margen bruto por categoría** (Métrica de ratio)
   ```scala
   val marginByCategory = salesSilver
     .withColumn("revenue", col("quantity") * col("price"))
     .withColumn("cogs", col("quantity") * col("cost"))
     .withColumn("margin", col("revenue") - col("cogs"))
     .groupBy("category")
     .agg(
       sum("revenue").as("total_revenue"),
       sum("cogs").as("total_cogs"),
       (sum("margin") / sum("revenue")).as("gross_margin_pct")
     )
     .orderBy(desc("gross_margin_pct"))
   
   // Resultado:
   // category | gross_margin_pct
   // Electrónica | 45.3%
   // Accesorios | 38.2%
   ```

   **KPI 3: Evolución de ventas por mes** (Métrica temporal)
   ```scala
   val monthlySales = salesSilver
     .withColumn("year_month", trunc(col("order_date"), "month"))
     .groupBy("year_month")
     .agg(
       count("order_id").as("num_orders"),
       sum(col("quantity") * col("price")).as("revenue")
     )
     .orderBy("year_month")
   
   // Resultado:
   // year_month | num_orders | revenue
   // 2023-01-01 | 2,340 | $543,210
   // 2023-02-01 | 2,156 | $456,789
   ```

   **KPI 4: Top 10 productos** (Métrica de ranking)
   ```scala
   val top10Products = salesSilver
     .groupBy("product_id")
     .agg(
       sum(col("quantity") * col("price")).as("revenue"),
       count("order_id").as("num_orders")
     )
     .orderBy(desc("revenue"))
     .limit(10)
   
   // Resultado:
   // product_id | revenue | num_orders
   // P001 | $345,678 | 1,234
   // P002 | $298,456 | 987
   ```

   **KPI 5: Ingresos por cliente y región** (Métrica de dimensión)
   ```scala
   val revenueByCustomerRegion = salesSilver
     .groupBy("customer_id", "country")
     .agg(
       sum(col("quantity") * col("price")).as("revenue"),
       count("order_id").as("num_orders"),
       avg(col("quantity") * col("price")).as("avg_order_value")
     )
   ```

3. **Organiza en 2+ tablas Gold con granularidades diferentes**:

   **Gold Tabla 1: KPIs Diarios** (granularidad: día)
   ```scala
   val dailyKPIs = salesSilver
     .withColumn("revenue", col("quantity") * col("price"))
     .withColumn("cogs", col("quantity") * col("cost"))
     .withColumn("margin", col("revenue") - col("cogs"))
     .groupBy(to_date(col("order_date")).as("date"))
     .agg(
       count("order_id").as("num_orders"),
       sum("revenue").as("total_revenue"),
       sum("cogs").as("total_cogs"),
       (sum("margin") / sum("revenue")).as("gross_margin_pct")
     )
     .orderBy("date")
   
   dailyKPIs.write
     .mode("overwrite")
     .format("delta")
     .save("delta/gold/daily_kpis")
   ```

   **Gold Tabla 2: KPIs por Categoría** (granularidad: categoría)
   ```scala
   val categoryKPIs = salesSilver
     .withColumn("revenue", col("quantity") * col("price"))
     .withColumn("cogs", col("quantity") * col("cost"))
     .withColumn("margin", col("revenue") - col("cogs"))
     .groupBy("category")
     .agg(
       count("order_id").as("num_orders"),
       sum("revenue").as("total_revenue"),
       sum("cogs").as("total_cogs"),
       (sum("margin") / sum("revenue")).as("gross_margin_pct"),
       avg("revenue").as("avg_order_value")
     )
     .orderBy(desc("total_revenue"))
   
   categoryKPIs.write
     .mode("overwrite")
     .format("delta")
     .save("delta/gold/category_kpis")
   ```

### 📝 Catálogo de KPIs (para tu informe):
```
KPI #1: Ingresos Totales
  Definición: Suma de (cantidad × precio) para todos los pedidos
  Fórmula: SUM(quantity * price)
  Valor: $5,234,567
  Negocio: Métrica de salud general del negocio
  
KPI #2: Margen Bruto por Categoría
  Definición: (Ingresos - Costo) / Ingresos, agrupado por categoría
  Fórmula: SUM(revenue - cogs) / SUM(revenue)
  Valor: 42.3% (promedio)
  Negocio: Identifica categorías más rentables

KPI #3: Evolución Mensual de Ventas
  Definición: Ingresos por mes
  Fórmula: SUM(quantity * price) GROUP BY MONTH
  Valor: Rango $234k - $567k según mes
  Negocio: Detecta estacionalidad
  
KPI #4: Top 10 Productos
  Definición: Productos ordenados por ingresos
  Fórmula: SUM(revenue) ORDER BY DESC
  Valor: Top 1 = P001 con $345,678
  Negocio: Enfoca stock y marketing en líderes
  
KPI #5: Ingresos por País
  Definición: Ingresos agrupados por país
  Fórmula: SUM(revenue) GROUP BY country
  Valor: USA $2.1M, UK $1.3M, etc.
  Negocio: Identifica mercados clave
```

---

# 💾 PASO 6: EXPORTA DATOS PARA POWER BI

### Qué debes hacer:

1. **Lee las tablas Gold** y **exporta a CSV**:
   ```scala
   // Tabla 1: Daily KPIs
   val dailyKPIs = spark.read.format("delta")
     .load("delta/gold/daily_kpis")
   
   dailyKPIs
     .coalesce(1)  // Consolida en 1 archivo
     .write
     .mode("overwrite")
     .option("header", "true")
     .csv("dashboard/daily_kpis")
   
   // Tabla 2: Category KPIs
   val categoryKPIs = spark.read.format("delta")
     .load("delta/gold/category_kpis")
   
   categoryKPIs
     .coalesce(1)
     .write
     .mode("overwrite")
     .option("header", "true")
     .csv("dashboard/category_kpis")
   ```

2. **Verifica que los CSVs se leen bien**:
   ```
   dashboard/
   ├── daily_kpis.csv
   ├── category_kpis.csv
   └── top10_products.csv
   ```

3. **Documenta la ubicación**:
   ```
   Archivo: informe/rutas_datos.txt
   
   Archivos exportados para Power BI:
   
   1. dashboard/daily_kpis.csv
      - Columnas: date, num_orders, total_revenue, gross_margin_pct
      - Filas: 365 (un año de datos)
      - Tamaño: 15 KB
   
   2. dashboard/category_kpis.csv
      - Columnas: category, num_orders, total_revenue, gross_margin_pct
      - Filas: 5 (5 categorías)
      - Tamaño: 2 KB
   ```

---

# 📊 PASO 7: CONSTRUYE EL DASHBOARD EN POWER BI

### Qué debes hacer:

1. **Abre Power BI Desktop**:
   - Descarga de: https://powerbi.microsoft.com/es-es/desktop/

2. **Importa los CSVs**:
   ```
   Inicio → Obtener datos → CSV
   Selecciona: dashboard/daily_kpis.csv
   Haz clic en Cargar
   
   Repite para category_kpis.csv
   ```

3. **Crea relaciones** (si tienes múltiples tablas):
   ```
   Vista Modelo → Nueva relación
   Tabla 1: daily_kpis | Columna: month (derivada de date)
   Tabla 2: category_kpis | Columna: category
   ```

4. **Crea 4 visualizaciones obligatorias**:

   **Viz 1: Tarjeta - Ingresos Totales** (KPI clave)
   ```
   Valores: SUM(total_revenue)
   Resultado: $5,234,567
   ```

   **Viz 2: Gráfico de líneas - Ingresos por mes** (Temporal)
   ```
   Eje X: date (group by MONTH)
   Eje Y: SUM(total_revenue)
   Línea: Azul oscuro
   ```

   **Viz 3: Gráfico de barras - Top 5 categorías** (Comparación)
   ```
   Eje X: category
   Eje Y: SUM(total_revenue)
   Barras: Verdes degradadas
   Ordenar: Descendente por revenue
   ```

   **Viz 4: Tabla - Detalles categorías** (Adicional)
   ```
   Columnas: category, num_orders, total_revenue, gross_margin_pct
   Formato: Moneda para revenue, porcentaje para margin
   ```

5. **Añade 2 Segmentadores (Slicers)**:

   **Segmentador 1: Rango de fechas**
   ```
   Campo: date
   Tipo: Entre fechas
   Conectar a: Viz 1, 2, 4
   ```

   **Segmentador 2: Categoría**
   ```
   Campo: category
   Tipo: Lista (checkboxes)
   Conectar a: Viz 3, 4
   ```

6. **Diseña visual**:
   ```
   - Título: "Dashboard de Ventas Mensual"
   - Color corporativo: Azul #0078D4
   - Fondo: Blanco
   - Fuente: Segoe UI, 11pt
   - Alineación: Centrada
   ```

7. **Guarda y exporta**:
   ```
   Archivo → Guardar
   Nombre: dashboard_proyecto.pbix
   Ubicación: dashboard/
   
   Archivo → Exportar → Exportar a PDF
   Nombre: dashboard_proyecto.pdf
   Ubicación: informe/
   ```

---

# 📄 PASO 8: REDACTA EL INFORME TÉCNICO

Crea un PDF de **4-8 páginas** con estas secciones:

### Estructura del informe:

```
INFORME TÉCNICO - PROYECTO FINAL BIG DATA + POWER BI

1. RESUMEN EJECUTIVO (0.5 páginas)
   - Qué es el proyecto
   - Qué problema resuelve
   - 3 hallazgos clave
   
   Ejemplo:
   "Este proyecto analiza datos de venta de TechStore (2023-2024).
   Problema: No tenían visibilidad de productos rentables ni 
   estacionalidad regional. Hallazgos:
   1. Electrónica genera 45% de margen (vs 38% en accesorios)
   2. Demanda pico en nov-dic (estacionalidad clara)
   3. USA es 40% de ingresos pero UK tiene cliente LTV mayor"

2. DESCRIPCIÓN DEL DOMINIO (1 página)
   - Empresa ficticia (TechStore)
   - Problemática (¿qué no sabían?)
   - Preguntas de negocio (3-5)
   
3. ARQUITECTURA DEL PIPELINE (0.5 páginas)
   - Diagrama: Datos → Bronze → Silver → Gold → Power BI
   - Breve descripción de cada capa
   
4. DESCRIPCIÓN DE FUENTES (0.5 páginas)
   - Tabla 1: sales.csv (100k filas, formato, problemas)
   - Tabla 2: products.json (1k filas, formato, problemas)
   - Problemas de calidad encontrados
   
5. DECISIONES DE LIMPIEZA (1 página)
   - Qué problemas se encontraron
   - Cómo se resolvieron (eliminar vs imputar)
   - Ejemplos: "Eliminar 100 duplicados porque no tenían valor"
   
6. CATÁLOGO DE KPIs (1 página)
   - Tabla con: Nombre | Definición | Fórmula | Valor | Interpretación
   
   Ejemplo:
   ┌─────────────────┬──────────────────┬──────────────────┬─────────────┐
   │ KPI             │ Definición       │ Fórmula          │ Valor       │
   ├─────────────────┼──────────────────┼──────────────────┼─────────────┤
   │ Ingresos        │ Total anual      │ SUM(qty * price) │ $5.2M       │
   │ Margen Bruto    │ % ganancia       │ (Rev-Cogs)/Rev   │ 42.3%       │
   │ Top Producto    │ Producto #1      │ ORDER BY revenue │ P001: $345k │
   └─────────────────┴──────────────────┴──────────────────┴─────────────┘

7. CAPTURAS DEL DASHBOARD (1.5 páginas)
   - Captura 1: Tarjeta ingresos
     "El dashboard muestra ingresos totales de $5.2M, indicador de salud"
   - Captura 2: Gráfico líneas
     "Evolución mensual revela pico en noviembre (+35% vs promedio)"
   - Captura 3: Gráfico barras
     "Electrónica lidera con 55% del volumen de ordenes"
   - Captura 4: Tabla detalles
     "Margen por categoría: Electrónica 45%, Accesorios 38%"

8. CONCLUSIONES (0.5 páginas)
   - Qué aprendiste
   - Próximos pasos (si tuvieras más tiempo)
   - Ejemplo: "Con ML podríamos predecir demanda. Con Streaming,
     KPIs en tiempo real"
```

### Cómo crear el PDF:

**Opción A: Usar Word/Google Docs**
1. Escribe el informe en Word
2. Exporta como PDF

**Opción B: Usar Markdown + Pandoc**
```bash
pandoc informe.md -o informe.pdf
```

**Opción C: Usar Notion**
1. Escribe en Notion
2. Exporta como PDF

---

# ✅ PASO 9: LISTA DE VERIFICACIÓN FINAL

Antes de entregar, **verifica todos estos puntos**:

```
CARPETA DEL PROYECTO:
☐ proyecto_final_[tu_nombre]/datos/ contiene al menos 2 fuentes
☐ proyecto_final_[tu_nombre]/etl/ contiene código Scala/Jupyter
☐ proyecto_final_[tu_nombre]/analisis/ contiene KPIs
☐ proyecto_final_[tu_nombre]/delta/ contiene tablas Bronze, Silver, Gold
☐ proyecto_final_[tu_nombre]/dashboard/ contiene CSVs y .pbix
☐ proyecto_final_[tu_nombre]/informe/ contiene PDF

CÓDIGO:
☐ El código ejecuta sin errores
☐ No hay rutas absolutas (o están parametrizadas)
☐ Cada función/bloque tiene comentarios
☐ Variables/columnas tienen nombres claros (no df1, x, temp)
☐ Documentado si usaste overwrite/append/merge en Delta

DATOS:
☐ Datos Bronze tienen columnas de auditoría (ingestion_timestamp, source_file)
☐ Datos Silver están limpios y normalizados
☐ Datos Gold responden las 3+ preguntas de negocio
☐ CSVs para Power BI tienen encabezados y tipos correctos

POWER BI:
☐ Dashboard importa los 2-3 CSVs sin errores
☐ Hay 4+ visualizaciones
☐ Hay 2+ segmentadores
☐ Archivo .pbix se guarda sin corrupción
☐ PDF exportado está en informe/

INFORME:
☐ Contiene 8 secciones (resumen, dominio, arquitectura, etc.)
☐ Tiene 4-8 páginas
☐ Incluye capturas del dashboard
☐ Catálogo de KPIs con fórmulas
☐ Está en PDF

DOCUMENTACIÓN:
☐ Archivo README.md en la raíz explica qué es el proyecto
☐ Ruta de cada tabla (Bronze, Silver, Gold) está documentada
☐ Decisiones de limpieza están justificadas
☐ Problemas de calidad de datos originales están listados
```

---

# 🗂️ ESTRUCTURA FINAL DE CARPETAS

```
proyecto_final_[tu_nombre]/
│
├── README.md                              ← Guía rápida del proyecto
│
├── datos/
│   ├── sales.csv                          ← Fuente 1 (100k filas)
│   ├── products.json                      ← Fuente 2 (1k filas)
│   ├── customers.parquet                  ← Fuente 3 (10k filas)
│   ├── PROBLEMAS_CALIDAD.txt              ← Qué problemas introdujiste
│   └── generate_data.scala                ← Script que generó los datos
│
├── etl/
│   ├── 01_bronze.scala                    ← Ingesta raw (sin transformar)
│   ├── 02_silver.scala                    ← Limpieza y normalización
│   ├── 03_gold.scala                      ← KPIs y agregaciones
│   ├── config.scala                       ← Rutas y configuración
│   ├── 01_bronze_log.txt                  ← Log de filas/schema
│   ├── 02_silver_log.txt                  ← Log de limpieza
│   └── 03_gold_log.txt                    ← Log de KPIs
│
├── analisis/
│   └── kpis_calculo.scala                 ← Detalles de cálculos
│
├── delta/
│   ├── bronze/
│   │   ├── sales/
│   │   ├── products/
│   │   └── customers/
│   ├── silver/
│   │   ├── sales_enriched/
│   │   └── customers_enriched/
│   └── gold/
│       ├── daily_kpis/
│       ├── category_kpis/
│       └── top_products/
│
├── dashboard/
│   ├── daily_kpis.csv                     ← Exportado para Power BI
│   ├── category_kpis.csv                  ← Exportado para Power BI
│   ├── top_products.csv                   ← Exportado para Power BI
│   └── dashboard_proyecto.pbix            ← Archivo Power BI
│
└── informe/
    ├── informe_proyecto.pdf               ← Informe técnico (4-8 páginas)
    ├── dashboard_proyecto.pdf             ← Captura del dashboard
    ├── 00_definicion_negocio.md           ← Dominio + preguntas
    ├── rutas_datos.txt                    ← Ubicación de CSVs
    └── arquitectura_pipeline.png          ← Diagrama flujo datos
```

---

# ⏱️ TIMELINE ESTIMADO

| Paso | Tarea | Duración |
|------|-------|----------|
| 0 | Elegir entorno | 1 hora |
| 1 | Definir dominio + preguntas | 2-3 horas |
| 2 | Crear/descargar datos | 4-6 horas |
| 3 | Capa Bronze | 2-3 horas |
| 4 | Capa Silver | 4-6 horas |
| 5 | Capa Gold | 3-4 horas |
| 6 | Exportar a CSV | 1 hora |
| 7 | Dashboard Power BI | 4-6 horas |
| 8 | Informe técnico | 3-4 horas |
| 9 | Revisión final | 2 horas |
| | **TOTAL** | **26-36 horas** |

---

¿Por dónde quieres empezar? ¿Necesitas ayuda con algún paso específico?

# 🅿️🅿️🅿️🅿️Creo que por ahora ya tengo suficientes datos, y el tiempo que has incluido me viene muy bien. Te iré informando del desarrollo de este proyecto 😊.

¡Perfecto! 🎉 Me alegra haber podido ayudarte con la estructura completa del proyecto.

Tienes un **roadmap muy claro** ahora:
- ✅ 9 pasos bien documentados
- ✅ Ejemplos de código en Scala/Spark
- ✅ Timeline realista (26-36 horas)
- ✅ Estructura de carpetas lista
- ✅ Checklist de verificación final

## 📌 Cuando necesites ayuda:

Puedo ayudarte con:
- **Debugging de código Scala/Spark** (si hay errores)
- **Consultas sobre limpieza de datos** (¿cómo manejar X problema?)
- **Visualizaciones en Power BI** (¿cómo crear X gráfico?)
- **Redacción del informe técnico** (estructura, contenido)
- **Cualquier bloqueador** que encuentres en el camino

Solo **comparte:**
- El código que tengas hasta el momento
- Errores o logs si algo no funciona
- Preguntas específicas sobre pasos concretos

## 🚀 Mucho éxito con el proyecto

Es un ejercicio muy completo que te dará experiencia real en **ETL, Spark, Delta Lake y Business Intelligence**. 

**¡Adelante con TechStore! 💪**

Cuando tengas avances o preguntas, estaré aquí. 😊

> BY COPILOT GITHUB 
@Serio120

