# sistema-monitoreo-aws-gcp

![status](https://img.shields.io/badge/status-acad%C3%A9mico-blue)
![cloud](https://img.shields.io/badge/cloud-AWS%20%7C%20GCP-orange)
![monitoring](https://img.shields.io/badge/monitoring-Prometheus%20%7C%20Grafana-red)

---

## Resumen
Repositorio del proyecto **Sistema de Monitoreo Multi-Cloud**, cuyo
objetivo es implementar la recolección y visualización de métricas de
una instancia en **AWS**, centralizadas mediante **Prometheus** y
visualizadas con **Grafana** en **GCP**, aplicando controles de red
entre ambas plataformas.

---

## Alcance técnico
- Exposición de métricas del sistema mediante **Node Exporter**
- Recolección centralizada con **Prometheus**
- Visualización de métricas en **Grafana**
- Comunicación inter-cloud controlada por reglas de red
- Separación lógica entre infraestructura, configuración y evidencias

---

## Arquitectura

```mermaid
flowchart LR
    A[AWS<br/>Node Exporter :9100] -->|scrape| B[GCP<br/>Prometheus :9090]
    B --> C[Grafana :3000]
