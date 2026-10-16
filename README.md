# Monitoreo Proactivo de PostgreSQL: Incidencias, Saturación y Tolerancia a Fallos en N8N

## 📌 Descripción del Proyecto

Este repositorio documenta el diseño e implementación de un sistema automatizado de **Observabilidad, Monitoreo de Rendimiento y Manejo Global de Errores** para bases de datos **PostgreSQL**, desarrollado sobre la plataforma de orquestación **N8N**.

El flujo principal ejecuta evaluaciones periódicas automatizadas para reportar **incidencias** y niveles de **saturación** en la base de datos. Además, la arquitectura está diseñada con **tolerancia a fallos desacoplada**: en lugar de saturar cada nodo de consulta con salidas de error dedicadas, los fallos de conexión o caídas críticas del motor de base de datos son capturados dinámicamente por un **Error Handler global externo**, el cual notifica de forma inmediata la caída del servicio a través de **Telegram**.

---

## 🛠️ Arquitectura y Tecnologías Utilizadas

* **Orquestador de Flujos:** N8N (Workflow Automation).
* **Motor de Base de Datos:** PostgreSQL.
* **Canal de Notificación y Alerta:** Telegram Bot API.
* **Mecanismo de Ejecución:** Schedule Trigger / Cron Job.
* **Gestión de Errores Críticos:** Error Workflow / Centralized Error Handler.

---

## 📐 Lógica del Workflow y Arquitectura de Resiliencia

El sistema opera bajo un esquema estructurado en cuatro fases de monitoreo más un gestor global de caídas:

1. **Trigger de Ejecución (`Schedule Trigger`):**
   * Desencadena la evaluación periódica del estado del motor de base de datos según los intervalos programados.

2. **Extracción y Evaluación de Métricas de Rendimiento:**
   * **Consulta de Incidencias:** Analiza registros de error, fallos en transacciones o eventos anómalos guardados en tablas de auditoría.
   * **Consulta de Saturación:** Evalúa métricas operativas clave como conexiones activas, cuellos de botella y tiempos de respuesta.
   * **Nodos Condicionales (`If Incidencias` / `If Saturaciones`):** Filtran los resultados y derivan las alertas hacia las salidas de Telegram únicamente cuando se superan los umbrales definidos.

3. **Manejo Desacoplado de Caídas de Base de Datos (Error Handler Externo):**
   * Para optimizar la legibilidad y mantenimiento del flujo, **no se implementaron salidas individuales de error en cada nodo de PostgreSQL**.
   * En su lugar, el flujo está vinculado externamente a un **Workflow de Manejo de Errores (`Error Trigger`)**. Si PostgreSQL sufre una caída repentina o rechaza las conexiones durante la ejecución de las consultas, el evento de error dispara automáticamente esta automatización centralizada, notificando de inmediato a Telegram sobre la **caída total o indisponibilidad del servicio de base de datos**.

## 💡 Impacto Operativo y Beneficios de Arquitectura

* **Doble Nivel de Cobertura:** Monitoreo activo de métricas internas (incidencias/saturación) y monitoreo pasivo de disponibilidad (caídas del servicio).
* **Diseño Limpio y Mantenible:** Evita la duplicación de nodos de manejo de errores dentro del flujo principal al delegar la detección de caídas a un ejecutor externo.
* **Reducción del MTTR (Mean Time to Repair):** Notificaciones inmediatas en Telegram clasificadas por tipo de alerta: Incidencia, Saturación o Caída Crítica.

---

## 📂 Estructura del Repositorio

```text
n8n-postgresql-observability-automation/
│
├── README.md
└── img/
    └── workflow_n8n_postgresql.png
