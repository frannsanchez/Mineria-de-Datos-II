# Cloud Provider Analytics

**Integrantes:**\
- Franco Sanchez
- Rodrigo Cirera
- Juan Baliota

## Descripción del Proyecto

Este proyecto implementa un **pipeline de Ingeniería de Datos
End-to-End** usando la **Arquitectura Medallion (Bronze, Silver, Gold)**
con **PySpark** y una capa de servicio en **Cassandra / AstraDB**.

El objetivo es: - Procesar datos de uso de nube (Batch + Streaming) -
Aplicar reglas de calidad de datos - Calcular métricas de FinOps -
Disponibilizar datos para consultas en tiempo real

## Arquitectura del Pipeline

### Flujo Multi-hop (Medallion Architecture)

**Landing**
- Datos crudos (CSV maestros, JSONL de eventos streaming)

**Bronze (Raw)**
- Batch: Ingesta de maestros con deduplicación\
- Streaming: Ingesta de usage_events con Structured Streaming

**Silver (Enriched & Validated)**
- Limpieza, normalización y reglas de calidad\
- Separación en valid y quarantine

**Gold (Aggregated)**
- Data Mart FinOps\
- Agregaciones por org, fecha y servicio

**Serving**\
- Carga incremental a AstraDB

## 📁 Estructura del Proyecto

``` bash
/content/
├── landing/                   
├── bronze/
│   ├── customers_orgs/
│   ├── users/
│   ├── billing_monthly/
│   └── events/
├── silver/
│   ├── events_valid/
│   └── events_quarantine/
├── gold/
│   └── org_daily_usage_by_service/
├── checkpoints/
└── usage_events_stream/
```

## 🔧 Prerrequisitos

-   Google Colab o Spark local\
-   Python 3.10+\
-   pyspark, cassandra-driver, astrapy\
-   Cuenta en AstraDB

## 🚀 Quickstart

1.  Configuración inicial\
2.  Ingesta Batch (Bronze)\
3.  Ingesta Streaming\
4.  Procesamiento Silver\
5.  Generación Gold\
6.  Carga a Cassandra

## 🛡️ Calidad de Datos

-   `event_id` no nulo\
-   `cost_usd_increment >= -0.01`\
-   Registros inválidos → `events_quarantine`

## 📊 Consultas en Cassandra

``` sql
SELECT daily_date, service, total_cost_usd
FROM cloud_analytics.org_daily_usage_by_service
WHERE org_id = 'ORG_X' AND daily_date >= '2025-01-01';
```

``` sql
SELECT service, SUM(total_cost_usd) AS total
FROM cloud_analytics.org_daily_usage_by_service
GROUP BY service;
```

## 🔁 Idempotencia

-   dropDuplicates\
-   Auditoría de particiones\
-   Checkpoints

## 🧭 Decisiones Técnicas

-   Arquitectura Lambda + Medallion\
-   Particionamiento optimizado\
-   Modelado Cassandra query-first\
-   Watermark 2h\
-   Cuarentena activa
