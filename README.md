# Enterprise Market Intelligence Platform

## Motor de Procesamiento y Análisis Semántico a Gran Escala con Arquitectura RAG
*Pipeline de Big Data y Motor Generativo Asimétrico para la Auditoría Automatizada de la "Voz del Cliente" (Voice of Customer - VoC).*

---

## Descripción General
Este proyecto consiste en el diseño, desarrollo e implementación de una plataforma de datos corporativa de extremo a extremo (End-to-End) que unifica la ingeniería de datos a gran escala (Big Data) con la Inteligencia Artificial Generativa y Procesamiento de Lenguaje Natural (NLP). 

El sistema ingesta de forma masiva millones de registros no estructurados de retroalimentación de clientes (reseñas, quejas y opiniones), realiza transformaciones analíticas avanzadas, e indexa la información semántica. Finalmente, expone una consola inteligente basada en una arquitectura RAG (Retrieval-Augmented Generation) que permite a directores de negocio e investigadores de mercado interrogar analíticamente el repositorio mediante lenguaje natural, obteniendo reportes consolidados, trazables y libres de alucinaciones.

## Problemática y Necesidad de Negocio
Las corporaciones globales y las cadenas de E-commerce se enfrentan al reto de procesar diariamente volúmenes masivos de datos textuales provenientes de sus canales de atención y venta. Analizar manualmente estos textos para extraer insights accionables es operativamente inviable y costoso.

Las soluciones tradicionales de Business Intelligence (BI) fallan en este entorno porque solo miden métricas estructuradas cuantitativas (como calificaciones de estrellas), perdiendo por completo el contexto cualitativo (el porqué de la insatisfacción, fallas repetitivas en productos específicos o alertas tempranas de fraude). Por otro lado, implementar Inteligencia Artificial Generativa directamente sobre estos hilos de datos sin control provoca altos costos de procesamiento en la nube, riesgos de filtración de datos y respuestas inexactas (alucinaciones del modelo).

## Arquitectura del Sistema (Enterprise Architecture)
Para solucionar este cuello de botella, se diseñó una infraestructura híbrida de alto rendimiento, desacoplada y escalable:

* **Fase de Big Data (Databricks + PySpark):** Ingesta masiva del dataset original para realizar tareas de normalización, limpieza y transformación mediante procesamiento distribuido en memoria.
* **Fase Relacional y Semántica (PostgreSQL + pgvector):** Modelado de datos estructurados en tablas relacionales y transformación de opiniones textuales en embeddings vectoriales de alta dimensionalidad bajo una misma base unificada.
* **Fase de Inteligencia y Orquestación (Python, FastAPI & LangChain):** Microservicio interno asíncrono y de baja latencia que segmenta la información (Semantic Chunking), gestiona índices avanzados (HNSW) y orquesta llamadas optimizadas hacia LLMs Open Source de alta velocidad (vía Groq/Ollama).
* **Fase de Negocio y Gobernanza (PHP Backend):** Núcleo central seguro encargado del control de accesos, lógica transaccional clásica y almacenamiento de bitácoras de auditoría inmutables para monitorear el uso de la IA.
* **Fase de Entrega e Interfaz (React.js + Tailwind CSS):** Dashboard web altamente reactivo dividido en un panel analítico tradicional (KPIs cuantitativos) y una terminal interactiva RAG con flujo de datos en tiempo real (streaming/Server-Sent Events).

---

## Fuente de Datos (Data Source)
El proyecto se alimenta del corpus histórico de código abierto de **Amazon Customer Reviews Dataset (McAuley Lab - UCSD)**. La estrategia de adquisición se divide en dos fases:

1. **Entorno de Experimentación (Laboratorio Local):** Subconjunto de la categoría *Electronics* optimizado bajo la regla *5-core*. Se trabaja con un universo de **1,689,188 registros** del cual se extrajo una muestra probabilística controlada de **30,000 filas** en JSON para perfilado y validación de código.
2. **Entorno de Producción (Fase Cloud):** Repositorio masivo centralizado Amazon Reviews 2023 con decenas de millones de filas procesadas concurrentemente sobre los clústers de Databricks.

### Estructura de Metadatos:
* `asin`: Identificador único global del producto (Key de catálogo).
* `reviewer_id`: Identificador único del cliente.
* `overall`: Calificación cuantitativa (1 a 5 estrellas).
* `review_text`: Opinión explícita extendida (Entrada principal RAG).
* `summary`: Título/resumen ejecutivo de la reseña (Ancla semántica).
* `unix_review_time`: Marca de tiempo UNIX para análisis cronológico rápido.

---

## Plan Global de Desarrollo y Ejecución Cronológica

[Fase 0: Lab Local] -> [Fase 1: Big Data] -> [Fase 2: Modelado SQL] -> [Fase 3: RAG & NLP] -> [Fase 4: FastAPI] -> [Fase 5: PHP] -> [Fase 6: UI React]

### Fase 0: Laboratorio de Validación Local (Muestra Controlada)
* **Aislamiento de Entorno:** Configuración de un entorno virtual independiente en Python 3.11 (`market-intelligence`) para garantizar paridad absoluta con los runtimes LTS de Databricks.
* **Ingeniería de Datos & 1FN:** Conversión de nomenclatura a `snake_case`. Desempaquetado del objeto complejo original `helpful` en columnas tabulares independientes (`helpful_votes`, `total_votes`, `helpful_ratio`), llevando el set a la **Primera Forma Normal (1FN)**.
* **Paradigma NLP Moderno:** Abandono de técnicas clásicas (como la remoción de *stop words* o minúsculas forzadas). Se preserva la puntuación y sintaxis exacta para que los mecanismos de **Atención (*Attention*)** de los Transformers capturen ironía, tecnicismos y emociones. Aplicación de `Regex` para eliminar basura HTML y saltos de línea redundantes.
* **Pre-vuelo de Big Data:** Configuración local de un nodo paralelo simulado en PySpark (`master("local[*]")`). Se validó la sintaxis utilizando transformaciones nativas vectorizadas de Spark (`pyspark.sql.functions`), prohibiendo funciones `.apply()` o lambdas de Pandas que degradan el rendimiento distribuido.
* **Estrategia de Chunking:** Prototipado del orquestador mediante `LangChain` aplicando un divisor recursivo (`RecursiveCharacterTextSplitter`) parametrizado a un `chunk_size` de 150 palabras y un `chunk_overlap` de 30 palabras para erradicar el efecto *"Lost in the Middle"*.

### Fase 1: Ingesta y Pipelines de Big Data (Databricks + PySpark)
* Migración de las transformaciones de la Fase 0 a la nube en **Databricks Community Edition**.
* Ingesta directa vía comandos de terminal (`%sh wget/curl`) puenteando la red local.
* Gobernanza del repositorio a gran escala mediante el uso de **Delta Lake** para asegurar transacciones ACID y desduplicación secuencial distribuida.

### Fase 2: Arquitectura Relacional y Modelado Semántico (PostgreSQL + pgvector)
* Diseño físico de tablas relacionales optimizadas para las entidades `productos`, `clientes` y `reseñas`.
* Inyección de la extensión **`pgvector`** para habilitar el tipo de dato `vector`.
* Estructuración de índices vectoriales avanzados (como **HNSW** o **IVFFlat**) para resolver con alta eficiencia las búsquedas de vecinos más cercanos mediante similitud de cosenos.

### Fase 3: Pipeline de NLP e Inteligencia Generativa (LangChain + Vector DB)
* Extracción automatizada de metadatos (Análisis de sentimiento multivariable y extracción de entidades sobre fallas de productos) integrando LLMs Open Source.
* Poblado masivo de embeddings textuales en PostgreSQL e inyección dinámica de contextos al prompt previniendo alucinaciones.

### Fase 4: Desacoplamiento y Construcción de la REST API (Python + FastAPI)
* Empaquetado del backend de IA como un microservicio independiente con **FastAPI**.
* Implementación de procesamiento asíncrono nativo para soportar alta concurrencia.
* Configuración de respuestas basadas en eventos para el **Streaming de Tokens** en tiempo real.

### Fase 5: Orquestación Empresarial y Gobernanza (PHP Backend)
* Desarrollo de la lógica tradicional de la plataforma en **PHP**.
* Consumo interno del microservicio de FastAPI para inyectar IA y ejecución de queries SQL nativas directas para desplegar reportes globales masivos.
* Manejo de sesiones seguras y almacenamiento de bitácoras de auditoría inmutables (registro obligatorio de preguntas de usuarios y latencia del LLM).

### Fase 6: Frontend Interactivo de Alta Fidelidad (React.js)
* Construcción de una SPA (Single Page Application) con **React.js y Tailwind CSS**.
* Módulo Cualitativo: Gráficas dinámicas de dispersión, tendencias de quejas e históricos cuantitativos de satisfacción.
* Módulo de IA: Terminal conversacional fluida alimentada por *Server-Sent Events (SSE)* para lectura inmediata de tokens.

---

## Estructura del Repositorio

```text
mi-proyecto-market-intelligence/
├── .gitignore                          # Exclusión de datos pesados y credenciales
├── README.md                           # Documentación principal de arquitectura
├── requirements.txt                    # Dependencias estrictas del entorno local
├── data/
│   ├── electronics_sample_30k.json     # Muestra controlada de laboratorio (Ignorado)
│   └── *.parquet/                      # Archivos columnares de salida local (Ignorado)
├── notebooks/
│   └── market_intelligence_platform.ipynb # Bitácora de experimentación y pre-vuelo
└── src/
    ├── __init__.py
    ├── pipeline_etl.py                 # Script refinado de Spark listo para Jobs
    └── motor_rag.py
```
