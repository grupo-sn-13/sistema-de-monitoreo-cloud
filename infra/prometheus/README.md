### 📁 Prometheus (`infra/prometheus/`)

Contiene los archivos de configuración principales de Prometheus, encargados de definir los *targets* de monitoreo, intervalos de scraping y reglas adicionales.

#### Archivos

- **`prometheus.yml`**  
  Archivo principal de configuración de Prometheus.  
  Define:
  - Jobs de scraping (Node Exporter, Prometheus local, etc.).
  - Direcciones IP/DNS de las instancias monitoreadas.
  - Intervalos de recolección de métricas.

- **`rules/`** *(opcional)*  
  Carpeta destinada a reglas de alertas (*Alerting Rules*), en caso de implementar alertas con Prometheus Alertmanager.

#### Buenas prácticas
- No almacenar credenciales directamente en este archivo.
- Restringir el acceso al puerto de Prometheus mediante firewall o Security Groups.

