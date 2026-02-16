# Pruebas de Estrés del Sistema de Monitoreo

Este apartado documenta los **tests de estrés** realizados sobre nuestro sistema de monitoreo. El objetivo principal fue evaluar el comportamiento y la estabilidad del sistema bajo condiciones de carga elevadas en CPU y almacenamiento.

---

## 🛠 Herramientas utilizadas

1. **Fallocate**  
   Se utilizó para **estresar el almacenamiento en disco**, permitiendo simular situaciones de alta ocupación de espacio.

2. **Herramientas de Linux para CPU**  
   Se crearon **scripts de prueba** para generar carga en el procesador y evaluar la respuesta del sistema bajo **uso intensivo de CPU**.  
   > Nota: Se utilizó una herramienta típica de Linux para pruebas de CPU, como `stress` o `stress-ng`.

3. **CRON**  
   Se empleó para **programar la ejecución de los scripts** en distintos horarios, simulando **horas pico** de actividad. Esto permitió verificar la estabilidad del sistema en momentos críticos.

---

##   Objetivos de las pruebas

- Validar la capacidad del sistema de monitoreo para **detectar y reportar cargas altas** en CPU y disco.
- Verificar la ejecución programada de scripts y su efecto en el rendimiento durante **simulaciones de tráfico intenso**.
- Simular cargas altas de trabajo en las instancias de GCP y AWS.

---

##   Resultados esperados

- Detección correcta de **uso elevado de CPU y disco** por parte del sistema.
- Generación de alertas oportunas en la interfaz de monitoreo.
- Confirmación de que los scripts ejecutados en horarios programados **no generan errores** ni bloqueos del sistema.

---

##   Notas

- Los scripts están diseñados para ser **reutilizables y ajustables**, permitiendo variar el tamaño de carga o la duración de la prueba.
- Este repositorio sirve como referencia para futuras pruebas de **estrés y rendimiento** del sistema.

---
