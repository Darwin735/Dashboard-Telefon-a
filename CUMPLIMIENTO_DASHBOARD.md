# Cumplimiento del dashboard v16

Fecha de revision local: **9 de agosto de 2026**.

## Implementacion

| Area | Resultado |
|---|---|
| MariaDB y CDR | Pool local de solo lectura; llamadas agrupadas por `linkedid` |
| KPIs y tendencias | Total, respuesta, duracion, hora/dia y periodo anterior |
| Departamentos | Recepcion, Ventas, Soporte, Finanzas, Gerencia, Logistica y Bodega |
| Grupos | Respuesta, perdidas, voicemail, espera y conversacion de 6100/6200 |
| IVR | Eventos reales de menus 7000 y 7001, sin datos simulados |
| Troncal | Peer `BODEGA-IAX`, IP dinamica leida de Asterisk, latencia e historial |
| Llamadas activas | Agrupacion por conversacion, departamento y alerta de espera |
| Diagnostico | Diez fuentes/servicios diferenciados |
| Seguridad | Fail2ban, firewall, SIP anonimo, MariaDB, HTTPS, auth y respaldo |
| Evidencias | Vista imprimible, CSV de llamadas, IVR y resumen |
| Segunda PBX | API remota opcional, sin publicar MariaDB |
| Reversibilidad | Instalacion v16 independiente y respaldo completo de v15 |

## Evidencia final

```bash
node /opt/corporacion-litoral-v16/scripts/verify_dashboard.mjs
```

El verificador contiene 24 controles. La evidencia final es completa cuando
indica `24/24`. Si detecta un peer heredado `IAX2-trunk*`, debe eliminarse el
duplicado desde Issabel antes de capturar la pantalla definitiva.

No se declaran jitter ni perdida RTP: requieren una fuente RTCP/CEL real que no
forma parte de los CDR actuales.
