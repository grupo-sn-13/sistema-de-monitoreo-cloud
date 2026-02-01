### 📁 Servicios del sistema (`infra/systemd/`)

Incluye los archivos de servicio **systemd** necesarios para ejecutar Prometheus y Node Exporter como servicios del sistema en Linux.

Estos archivos permiten:
- Arranque automático al iniciar el sistema.
- Gestión estandarizada de procesos.
- Mayor estabilidad y control operativo.

#### Archivos

- **`prometheus.service`**  
  Define el servicio systemd para Prometheus:
  - Usuario dedicado.
  - Ruta del binario.
  - Archivo de configuración.
  - Reinicio automático ante fallos.

- **`node_exporter.service`**  
  Define el servicio systemd para Node Exporter:
  - Ejecución con privilegios mínimos.
  - Exposición del endpoint de métricas en el puerto correspondiente.

#### Uso
```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus

