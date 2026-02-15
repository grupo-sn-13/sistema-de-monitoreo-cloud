# Pruebas de Monitoreo — Fase 3
Matriz de pruebas, comandos de ejecución y validación de dashboards, alertas y umbrales

**Entorno:** Instancia en GCP  
**Grafana:** `esitproyectounotres.duckdns.org`  
**Dashboard:** `Node Exporter Full`  
**Instancia monitoreada:** `10.138.0.4:9100`  

---

## 1) Objetivo

Ejecutar pruebas controladas para validar:

- Estado normal del monitoreo.
- Alta carga de CPU.
- Falta de disco.
- Caída de servicio.
- Funcionamiento de paneles.
- Disparo de alertas.
- Ajuste de umbrales para reducir ruido sin perder criticidad.

---

## 2) Precondiciones

- Acceso a Grafana en `esitproyectounotres.duckdns.org`.
- Datasource Prometheus configurado.
- Node Exporter activo en el host monitoreado.
- Permisos para ejecutar comandos con `sudo` en el host de prueba.

---

## 3) Variables útiles

Ajustar variables de entorno:

```bash
# En el servidor donde está Prometheus
export PROM_URL="http://localhost:9090"

# Instancia a validar
export INSTANCE="10.138.0.4:9100"

# Job usado en tu Prometheus/Grafana
export JOB="node_exporter"
```

Consultas rápidas a Prometheus:

```bash
# Ver si Prometheus está respondiendo
curl -s "$PROM_URL/-/ready" && echo

# Ver targets UP
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=up' | head -c 400 && echo

# Ver alertas firing (Prometheus)
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=ALERTS{alertstate="firing"}' | head -c 400 && echo

# Ver lista de alertas activas (Prometheus)
curl -s "$PROM_URL/api/v1/alerts" | head -c 600 && echo
```

---

## 4) Matriz de pruebas

| ID   | Caso / Objetivo | Precondiciones | Datos/Parámetros | Pasos | Métrica/Query PromQL o Panel | Resultado esperado | Evidencia |
|------|------------------|----------------|------------------|------|------------------------------|-------------------|----------|
| PM-01 | Escenario normal, todo OK | Grafana corriendo | N/A | Abrir Grafana y dashboard | Paneles del dashboard | Todo UP | Anexo 1 |
| PM-02 | Alta carga, CPU alta | Ejecutar prueba de estrés | Instalar `stress` | Instalar herramienta | Panel CPU | Grafana recibe datos de alta carga | Anexo 2 |
| PM-03 | Falta de disco | Ejecutar prueba de disco lleno | Llenado al 95% | Simular llenado de disco | `100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})` | Disco al 95% y alerta visible | Anexo 3 |
| PM-04 | Caída de servicio | Simular caída | Detener node_exporter | Configurar alerta | `up{job="node_exporter"} == 0` | Al detener el servicio se detecta caída | Anexo 4 |
| PM-05 | Simulación de carga | Cargar servicio | Instalar `stress` | Ejecutar carga | Panel CPU | Carga mayor a 90% | Anexo 5 |
| PM-06 | Modificación de parámetros de pruebas | Ajustar alertas | Evaluación 1m a 10s | Modificar regla | Alert rules | Estado cambia a firing al detener servicio | Anexo 6 |
| PM-07 | Verificar paneles | Paneles visibles | N/A | Revisar dashboards | Paneles del dashboard | Paneles normales | Anexo 7 |
| PM-08 | Verificar alertas | Alertas configuradas | N/A | Probar caída | `up{job="node_exporter"} == 0` | Se disparó la alerta | Anexo 8 |
| PM-09 | Ajustar umbrales | Alertas ajustadas | N/A | Ajustar reglas | Alert rules | Alertas listas sin ruido excesivo | Anexo 9 |

---

## 5) Pruebas detalladas con comandos por PM

Recomendación: ejecutar cada prueba 2 a 5 minutos y validar en Grafana con rango de tiempo `Last 15 minutes`.

---

### PM-01 — Escenario normal, todo OK

**Comandos de validación**
```bash
# Ver UP por job e instancia
curl -sG "$PROM_URL/api/v1/query" --data-urlencode "query=up{job=\"$JOB\",instance=\"$INSTANCE\"}" | head -c 400 && echo

# Ver si hay alertas firing
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=ALERTS{alertstate="firing"}' | head -c 400 && echo

# Ver CPU actual (consulta estándar)
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)' | head -c 400 && echo

# Ver uso de disco (consulta usada en tu matriz)
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})' | head -c 400 && echo
```

**Resultado esperado**
- `up{job="node_exporter",instance="10.138.0.4:9100"} = 1`
- Sin alertas firing.

---

### PM-02 — Alta carga, CPU alta

**Instalación**
```bash
sudo apt update && sudo apt install -y stress
```

**Ejecución**
```bash
stress --cpu 2 --timeout 120
```

**Validación (Prometheus)**
```bash
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)' | head -c 400 && echo
```

---

### PM-03 — Falta de disco (llenado al 95%)

**Métrica PromQL**
```promql
100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"} )
```

**Validación antes**
```bash
df -h /
```

**Ejecución**
```bash
sudo fallocate -l 2G /var/tmp/llenado_disco_1.test
df -h /

# Repetir si hace falta acercarse al 95%
# sudo fallocate -l 2G /var/tmp/llenado_disco_2.test
# df -h /
```

**Validación (Prometheus)**
```bash
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})' | head -c 400 && echo
```

**Limpieza**
```bash
sudo rm -f /var/tmp/llenado_disco_*.test
df -h /
```

---

### PM-04 — Caída de servicio

**Ejecución**
```bash
sudo systemctl stop node_exporter
```

**Validación (Prometheus)**
```bash
curl -sG "$PROM_URL/api/v1/query" --data-urlencode "query=up{job=\"$JOB\",instance=\"$INSTANCE\"}" | head -c 400 && echo
curl -sG "$PROM_URL/api/v1/query" --data-urlencode "query=up{job=\"$JOB\"} == 0" | head -c 400 && echo
curl -s "$PROM_URL/api/v1/alerts" | head -c 800 && echo
```

**Restauración**
```bash
sudo systemctl start node_exporter
```

---

### PM-05 — Simulación de carga

**Instalación**
```bash
sudo apt update && sudo apt install -y stress
```

**Ejecución**
```bash
stress --cpu 2 --timeout 120
```

---

### PM-06 — Modificación de parámetros de pruebas

**Acción en Grafana**
- En `Alerting` → `Alert rules`, cambiar evaluación de 1m a 10s.

**Comandos para documentar cambios en repo**
```bash
git status
git diff
```

**Prueba rápida**
```bash
sudo systemctl stop node_exporter
sleep 20
sudo systemctl start node_exporter
```

---

### PM-07 — Verificar paneles

**Validación con queries**
```bash
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)' | head -c 400 && echo
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100' | head -c 400 && echo
curl -sG "$PROM_URL/api/v1/query" --data-urlencode 'query=100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})' | head -c 400 && echo
```

---

### PM-08 — Verificar alertas

**Ejecución**
```bash
sudo systemctl stop node_exporter
sleep 20
```

**Validación**
```bash
curl -sG "$PROM_URL/api/v1/query" --data-urlencode "query=up{job=\"$JOB\"} == 0" | head -c 400 && echo
curl -s "$PROM_URL/api/v1/alerts" | head -c 1000 && echo
```

**Restauración**
```bash
sudo systemctl start node_exporter
```

---

### PM-09 — Ajuste de umbrales

**Valores recomendados**
- CPU: `> 80%`
- Disco: `> 90%`
- Caída de servicio: `up == 0`

**Ejemplo de reglas en Prometheus**
```yaml
groups:
  - name: node_alerts
    rules:
      - alert: HighCPU
        expr: (100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)) > 80
        for: 5m
        labels:
          severity: warning

      - alert: LowDiskSpace
        expr: (100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"} )) > 90
        for: 5m
        labels:
          severity: critical

      - alert: NodeExporterDown
        expr: up{job="node_exporter"} == 0
        for: 2m
        labels:
          severity: critical
```
