
# RUNBOOK — Sistema de Monitoreo (Grafana + Prometheus + Node Exporter)

Este runbook describe cómo **operar**, **verificar** y **solucionar** problemas del stack de monitoreo implementado para el proyecto.
Created At: 2026-01-30
Created By: Ing C. Fletes
---

## 1) Alcance

**Monitor VM (GCP)**
- Grafana (UI dashboards)
- Prometheus (scraping y almacenamiento)
- (Opcional) Nginx + HTTPS

**Targets (AWS u otros)**
- Node Exporter (métricas del host)

Puertos:
- Grafana `3000/tcp`
- Prometheus `9090/tcp`
- Node Exporter `9100/tcp`
- (Opcional) Nginx/HTTPS `80/443`

---

## 2) Accesos y seguridad

### Acceso a servidores
- **SSH** (con par de llaves `.pem` / `.ppk`).
- Recomendado: restringir SSH a IPs confiables.
- Si existe un rol/servicio de gestión (ej. SSM), preferirlo para evitar exposición pública.

### Principios de seguridad operativa
- No almacenar contraseñas en el repositorio.
- Rotar credenciales (Grafana, llaves SSH) si se compartieron durante prácticas.
- Permitir `9100/tcp` en los targets **solo** desde la IP del Prometheus/Monitor.

---

## 3) Checklist de salud (Health Checks)

### A. Grafana (Monitor VM)
1. UI accesible:
   - `http://<MONITOR>:3000`
2. Servicio:
```bash
sudo systemctl status grafana-server
```
3. Logs:
```bash
sudo journalctl -u grafana-server -f
```

### B. Prometheus (Monitor VM)
1. UI accesible:
   - `http://<MONITOR>:9090`
2. Targets:
   - `http://<MONITOR>:9090/targets`
3. Servicio:
```bash
sudo systemctl status prometheus
```
4. Logs:
```bash
sudo journalctl -u prometheus -f
```

### C. Node Exporter (Target / o local)
1. Endpoint:
   - `http://<TARGET>:9100/metrics`
2. Servicio:
```bash
sudo systemctl status node_exporter
```
3. Logs:
```bash
sudo journalctl -u node_exporter -f
```

---

## 4) Operaciones estándar (Day-2 Ops)

### Reiniciar servicios
```bash
sudo systemctl restart grafana-server
sudo systemctl restart prometheus
sudo systemctl restart node_exporter
```

### Habilitar servicios al arranque
```bash
sudo systemctl enable grafana-server
sudo systemctl enable prometheus
sudo systemctl enable node_exporter
```

### Ver puertos escuchando
```bash
sudo ss -lntp | egrep '(:3000|:9090|:9100)'
```

### Ver conectividad desde Monitor VM hacia un Target
```bash
curl -sS http://<TARGET>:9100/metrics | head
```

---

## 5) Umbrales operativos (para alertas y triage)

- CPU:
  - 🟢 < 60%
  - 🟡 60–80%
  - 🔴 > 80%
- RAM:
  - 🟢 < 70%
  - 🟡 70–85%
  - 🔴 > 85%
- Disco (libre):
  - 🟢 > 30%
  - 🟡 15–30%
  - 🔴 < 15%

Triage rápido:
- 🔴 → revisar procesos, logs, escalamiento o limpieza de disco.
- 🟡 → observar tendencia y planificar ajuste.

---

## 6) Troubleshooting (incidentes comunes)

### Caso 1: Grafana abre pero no muestra datos (paneles en “No data”)
**Síntomas**
- UI funciona, pero paneles no muestran métricas.

**Diagnóstico**
1. Verificar datasource Prometheus en Grafana (URL correcta).
2. Verificar Prometheus arriba:
   - `http://<MONITOR>:9090`
3. Verificar targets:
   - `http://<MONITOR>:9090/targets`

**Acciones**
- Corregir URL del datasource (si Prometheus está en la misma VM: `http://localhost:9090`)
- Reiniciar Prometheus si cambió config:
  ```bash
  sudo systemctl restart prometheus
  ```

---

### Caso 2: Target DOWN en Prometheus (/targets muestra “DOWN”)
**Síntomas**
- Target aparece DOWN, errores de scrape.

**Diagnóstico**
1. Desde Monitor VM:
   ```bash
   curl -v http://<TARGET>:9100/metrics
   ```
2. Revisar SG/Firewall:
   - `9100/tcp` permitido **desde IP del Monitor**
3. Revisar servicio en el target:
   ```bash
   sudo systemctl status node_exporter
   sudo journalctl -u node_exporter -f
   ```

**Acciones**
- Abrir `9100` solo para el Monitor.
- Reiniciar Node Exporter:
  ```bash
  sudo systemctl restart node_exporter
  ```

---

### Caso 3: Node Exporter no levanta (service failed)
**Diagnóstico**
- Ver logs:
  ```bash
  sudo journalctl -u node_exporter -n 200 --no-pager
  ```

**Causas típicas**
- Ruta incorrecta del binario en el service.
- Permisos del usuario `node_exporter`.
- Puerto ocupado (raro).

**Acciones**
- Confirmar binario:
  ```bash
  which node_exporter || ls -l /usr/local/bin/node_exporter
  ```
- Revisar unit file:
  ```bash
  sudo cat /etc/systemd/system/node_exporter.service
  sudo systemctl daemon-reload
  sudo systemctl restart node_exporter
  ```

---

### Caso 4: Prometheus no arranca / config inválida
**Diagnóstico**
```bash
sudo journalctl -u prometheus -n 200 --no-pager
```

**Acciones**
- Revisar sintaxis YAML de `prometheus.yml`.
- Validar targets y puertos.
- Reiniciar.

---

### Caso 5: Problemas con HTTPS / dominio
**Diagnóstico**
- Ver estado Nginx:
```bash
sudo systemctl status nginx
sudo journalctl -u nginx -f
```
- Ver configuración:
```bash
sudo nginx -t
```

**Acciones**
- Corregir reverse proxy (host/puerto).
- Renovar certificado (si Let’s Encrypt).
- Verificar DNS apunta a la IP pública correcta.

---

### Caso 6: Disco casi lleno en Monitor VM
**Acciones**
1. Ver uso:
```bash
df -h
```
2. Revisar consumo:
```bash
sudo du -h /var/lib/prometheus | tail
sudo du -h /var/lib/grafana | tail
```
3. Limpieza (con cuidado):
- Rotación de logs, eliminar archivos temporales.
- Ajustar retención de Prometheus (flags de arranque).

---

## 7) Respuesta a incidentes (flujo recomendado)

1. **Detectar** (dashboard/targets/alertas).
2. **Clasificar** severidad (🟡/🔴).
3. **Aislar** (¿Grafana? ¿Prometheus? ¿Target? ¿Red?).
4. **Mitigar** (reinicio controlado, abrir regla específica, revertir config).
5. **Resolver** (cambio permanente y documentado).
6. **Post-mortem** (qué pasó, causa raíz, acción preventiva).

---

## 8) Backups y recuperación (recomendado)

Respaldar:
- Config de Prometheus: `/etc/prometheus/prometheus.yml`
- Datos Prometheus: `/var/lib/prometheus`
- Config Grafana: `/etc/grafana/grafana.ini`
- Datos Grafana: `/var/lib/grafana`
- Config Nginx: `/etc/nginx/sites-available/`

Restauración (alto nivel):
1. Detener servicios.
2. Restaurar configs y data.
3. Arrancar y validar /targets y dashboards.

---

## 9) Contactos (equipo)

- Líder de Proyecto Jr.: Irvin Osiris Cantarero Moreno
- DevOps/Observabilidad Jr.: René Ismael Pinto Ávalos
- Ingeniero Cloud/Infra Jr.: Carlos Alberto Sopón Trujillo
- Analista de Métricas Jr.: Will Alberto López Palacios
- Documentador Técnico: Carlos José Fletes Alduvin
