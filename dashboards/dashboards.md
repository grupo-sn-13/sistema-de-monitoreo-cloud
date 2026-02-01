
---

## 📁 `infra/dashboards/`

```md
### 📁 Dashboards (`dashboards/`)

Esta carpeta contiene los dashboards de Grafana exportados en formato JSON.

Permite:
- Versionar dashboards.
- Replicar visualizaciones entre entornos.
- Restaurar configuraciones rápidamente en nuevas instancias de Grafana.

#### Archivos

- **`dashboards.json`**  
  Exportación de dashboards de Grafana que incluyen:
  - Métricas de CPU, memoria, disco y red.
  - Visualización de múltiples instancias.
  - Paneles optimizados para Node Exporter.

#### Importación en Grafana
1. Acceder a Grafana.
2. Ir a **Dashboards → Import**.
3. Cargar el archivo JSON.
4. Asociar la fuente de datos Prometheus.

