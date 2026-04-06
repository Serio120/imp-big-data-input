# Primer dia Big Data


## ¿Que es Big DATA?


Computación tradicional vs computación distribuidas


Las tegnolgias de big data estan desarrolladas en apache.


Volumen, Velocidad, Variedad


Ejemplo un Excel CSV se cuelga eso es Big Data


Si solo se utiliza Excel no es Big Data


Videos, Json, etc procesa el Big Data


Procesamiento en tiempo real tambien es Big Data


Ecosistema: Hadoop, Spark, Kafka, Bases NoSQL

Lenguajes: Scala, Python, Java – en este curso: Scala + Spark


Volumen

- Cantidad masiva de datos generados

- Desde GB hasta Petabytes y Zettabytes

- Ejemplo: Facebook genera 4 PB de datos al día.


Velocidad

- Ritmo al que se genera

- Datos

- Ejemplo: Twitter genera 500.000 tweets por minuto.


Variedad

- Tipos y fuentes

- Estructurados

- Ejemplo: historial médico+ imagenes + conversaciones


Veracidad

- Calidad y fiabilidad de datos

- Los datos pueden estar incompletos o incorrectos.

- Ejemplo: Sensores con fallos, datos duplicados




## El Ecosistema Big Data


Almacenamiento (CAPA 1)

- HDFS

- Amazon S3, Azure Data, Lake

- HBase, Cassandra (NoSQL)



Procesamiento (CAPA 2)

- Apache Hadoop (MapReduce)

- Apache Spark (nuestro protagonista)

- Apache Flink (streamin)



Análisis e Ingesta (CAPA 3)

- Apache Kafka

## Datos Tradicionales vs Big Data


Datos Tradicionales

- Tamaño:

- Estructura:

- Herramientas: SQL, Excel, Oracle, MySQL


Big Data

- Tamaño : TB, PB, ZB – necesita clusters de servidores.

- Estructura heterogenea

- Herramientas Hadoop, Spark, Kafka

## Escalado Horizontal vs Vertical


### Escalado Horizontal en Big Data (Horizontal Scalling)


Se escala ampliando los nodos del cluster


(Cada nodo en un cluster tiene unas tareas especificas)


Al principio se crea el cluster en la NUBE y luego se migra a uno propio


### Comprendiendo el escalado vertical


Se escala en procesamiento y en memoria.


## Ecosistema de Roles en Big Data Y IA


  ## - Responsabilidades clave y herramientas tecnológicas


  ### I. Data Engineer (Constructor de infraestructuras de Datos) Spark, Kafka

  ### II. Data Scientist (Explorador de Patrones de Datos) Python, R, Spark

  ### III. Data Analyst (Interprete de datos Empresariales) SQL. Power

  ### IV. Big Data Architect (Estratega y Diseño de Sistemas) Hadoop , Google Cloud, AWS

  ### V. AI Engineer (Creador de Sistemas de IA Avanzados) Docker, PyTorch, Kubernetes


## Big Data en la Industria


El impacto del Big Data en los sectores más importantes de la economía:


Sanidad

- gg

- dd


Finanzas


Retail y Logistica

- gg

- dd


## Herramientas del Stack de Big Data


……




01 ¿Que es Big Data?


Datos Tradicionales VS Big Data


Tamaño MB a pocos GB



## Big Data en Finanzas y Banca


Detección de fraude en tiempo real

- Se analizan más de 150 variables por transacción

- Decisión en menos de 100 milisegundos

- Visa bloquea 2.500 millones de dolares en fraude al año con ML.


Scoring crediticio alternetivo

- Antes: solo se usaban nómina, historial bancario y DNI

- Ahora: comportamiento en redes, pagos de servicios, geolocalización.

- BBVA en España usa Big Data para incluir a más clientes sin historial.


Trading algoritmico

- …

- …


## Big Data en Sanidad


Dianóstico asistido por IA

- Google DeepMind detecta 50+ enfermedades en retinografias con precisión de oftalmólogo

- IBM Watson oncología

- En España: el hospital La FE (Valencia) usa IA para detectar sepsis 6 horas antes



Predicción de epidemias y salud pública


- Durante COVID-19 modelos de Big Data predi

- …

- …



## Big Data en Telecomunicaciones y otros sectores


Telecomunicaciones

- …

- …

- …


Transporte y Logística

- UPS optimiza

- ….

- ….


Energía y Utilities

- Smart grids:

- Iberdrola analiza millones de lecturas de contadores inteligentes

- ….


Agricultura de precisión

- Sensores en el campo

- Predicción de cosechas

- John Deere: tractores con Big Data integrado de fabrica



## El Stack Tecnologico Big Data


01 Apache Hadoop

- Framework de almacenamiento distribuido (HDFS)

- …

- …

- …

- …


02 Apache Spark

- Procesamiento en memoria

- ….

- ….

- …

- ...


03 Apache Kafka + Hive

- Kafka: Sisteme de Mensajería distribuida

- Ingesta millones de eventos/



¿Porqué Spark es el motor  central?



Problema de Hadoop MapReduce

- ….

- …


La solución de Spark: procesamiento en nemoria

- ….

- …


Spark es una navaja suiza

- ….

- …



Casos de Éxito Reales




Netflix


Tesla


Mercadona

- Analiza el ticket de compra de 5 millones de clientes diarios





Los Datos como Activo Estratégico


El nuevo petroleo del SIGLO XXI





Resumen del Bloque Teoríco
