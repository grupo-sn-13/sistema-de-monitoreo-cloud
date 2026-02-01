# Sistema de Monitoreo de Recursos Cloud con Grafana

Proyecto académico ESIT (Grupo SN-13): **“Sistema de Monitoreo de Recursos Cloud con Grafana”**.
Created At: 2026-01-20
Created By: Ing C. Fletes 

Este repositorio documenta y estandariza el despliegue de un stack de monitoreo para una entidad receptora ficticia (**ANSD**) con el fin de visualizar métricas de **CPU, RAM, Disco, Red y Disponibilidad (uptime)** desde múltiples recursos cloud.

---

## Objetivo

Proveer un **panel centralizado** para supervisar el estado de servidores/servicios críticos en la nube (CPU, memoria, almacenamiento, red y disponibilidad), con una vista clara para personal técnico y niveles ejecutivos.

---

## Arquitectura (alto nivel)

**Diseño lógico**
- **Targets (servidores a monitorear)**: instancias (p. ej. en AWS) con **Node Exporter** exponiendo métricas por HTTP.
- **Servidor de métricas**: **Prometheus** (scrapea periódicamente a los exporters y almacena series temporales).
- **Visualización**: **Grafana** (consulta Prometheus y presenta dashboards/alertas).
- (Opcional) **Reverse Proxy + HTTPS**: Nginx + dominio (DuckDNS o equivalente) para exponer Grafana de forma segura.

**Puertos típicos**
- Grafana: `3000/tcp`
- Prometheus: `9090/tcp`
- Node Exporter: `9100/tcp`
- (Opcional) Nginx/HTTPS: `80/tcp` y `443/tcp`

---

## Implementación usada en el proyecto (referencia)

- **VM Monitor en GCP** (Compute Engine):
  - SO: Ubuntu Server 22.04 LTS
  - Tipo: e2-medium (2 vCPU / 4 GB RAM)
  - Disco: 30 GB
  - Hospeda: Grafana + Prometheus (+ Node Exporter local si aplica)
- **Instancias Target en AWS (EC2)**:
  - Tipo: t3.micro (free tier)
  - SO: Debian (o similar)
  - Exporter: Node Exporter como servicio systemd

> Nota: Los detalles exactos (IPs, dominios y reglas) pueden variar por integrante/ambiente. Mantén la información sensible fuera del repositorio.

---

## Seguridad mínima (baseline)

- **No subir credenciales** al repositorio (usar variables de entorno/archivos fuera de Git).
- Exponer **solo puertos necesarios** y restringir por IP (Security Groups/Firewall).
- Crear **usuarios dedicados** (p. ej. `node_exporter`) con permisos mínimos.
- Preferir **SSM / acceso administrado** cuando aplique, y restringir SSH.

---

## Estructura recomendada del repositorio

```
.
├─ docs/
│  ├─ fase-1-analisis-y-diseno.pdf
│  ├─ bitacora-fase-2.docx
│  └─ capturas/
├─ infra/
│  ├─ prometheus/
│  │  ├─ prometheus.yml
│  │  └─ rules/            # alert rules (opcional)
│  ├─ systemd/
│  │  ├─ prometheus.service
│  │  └─ node_exporter.service
│  └─ nginx/
│     └─ grafana.conf       # reverse proxy (opcional)
└─ dashboards/
   └─ dashboards.json       # export de dashboards (opcional)
```

---

## Quick Start (paso a paso)

### 1) Monitor VM (GCP): instalar stack

**A. Actualizar sistema**
```bash
sudo apt update && sudo apt -y upgrade
```

**B. Instalar Prometheus**
- Descargar binarios oficiales.
- Crear usuario/paths.
- Configurar `prometheus.yml`.
- Crear servicio `systemd` y arrancar.

**C. Instalar Grafana**
- Instalar desde repositorio oficial.
- Habilitar/arrancar servicio.

**D. (Opcional) Nginx + HTTPS**
- Instalar Nginx.
- Configurar reverse proxy para `/` → `localhost:3000`.
- Configurar certificado TLS (Let's Encrypt) y dominio (DuckDNS u otro).

**E. Abrir firewall/puertos necesarios**
- `3000`, `9090`, `9100` (y `80/443` si usas Nginx/HTTPS).
- Restringir a IPs autorizadas.

### 2) Target (AWS): instalar Node Exporter

**A. Conectarse por SSH con par de llaves**
```bash
ssh -i "/ruta/ParDeClaves.pem" admin@IP_PUBLICA
```

**B. Instalar y registrar como servicio**
- Descargar Node Exporter.
- Crear usuario `node_exporter` sin login.
- Mover binario a `/usr/local/bin/`.
- Crear `/etc/systemd/system/node_exporter.service`.
- `sudo systemctl daemon-reload`
- `sudo systemctl enable --now node_exporter`

**C. Seguridad (Security Group)**
- Permitir `9100/tcp` **solo** desde la IP de la VM Monitor (Prometheus).
- Restringir SSH a IPs confiables (temporalmente puede abrirse para trabajo en equipo, pero se recomienda cerrarlo).

### 3) Prometheus: configurar targets

En la VM Monitor, editar `prometheus.yml` y agregar los targets:

```yaml
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets:
        - "IP_TARGET_1:9100"
        - "IP_TARGET_2:9100"
```

Recargar Prometheus:
```bash
sudo systemctl restart prometheus
```

Verificar en Prometheus:
- `http://<MONITOR>:9090/targets`

### 4) Grafana: agregar Prometheus como datasource

- Entrar a Grafana: `http://<MONITOR>:3000`
- **Data sources** → **Prometheus**
- URL: `http://localhost:9090` (si Prometheus está en la misma VM)
- Guardar & Test

### 5) Dashboards

Crear/importar dashboards con paneles:
- CPU
- Memoria RAM
- Disco
- Red (entrada/salida)
- Uptime / disponibilidad

---

## Métricas y umbrales sugeridos

- CPU:
  - 🟢 < 60% (normal)
  - 🟡 60–80% (atención)
  - 🔴 > 80% (riesgo)
- RAM:
  - 🟢 < 70%
  - 🟡 70–85%
  - 🔴 > 85%
- Disco (espacio libre):
  - 🟢 > 30% libre
  - 🟡 15–30% libre
  - 🔴 < 15% libre

---

## Roadmap (siguiente fase)

- Simular carga ligera (stress test), validar cambios de métricas y tiempos de refresco.
- Mejorar visualización: tags, paneles por servidor/servicio, filtros e intervalo de actualización.
- (Opcional) agregar alertas y notificaciones.

---

## Equipo y roles

- Irvin Osiris Cantarero Moreno – Líder de Proyecto Jr.
- Carlos Alberto Sopón Trujillo – Ingeniero Cloud / Infra Jr.
- René Ismael Pinto Ávalos – DevOps / Observabilidad Jr.
- Will Alberto López Palacios – Analista de Métricas Jr.
- Carlos José Fletes Alduvin – Documentador Técnico

---

## Licencia

Uso académico / educativo (ESIT). Si vas a reutilizarlo fuera del proyecto, ajusta la licencia y remueve datos sensibles.

