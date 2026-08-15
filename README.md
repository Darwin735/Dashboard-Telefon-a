# Dashboard de Telefonia Corporacion Litoral — v16

Version de observabilidad y evidencias para Issabel/Asterisk. Mantiene los
datos reales de CDR, IVR, grupos, troncal y llamadas activas de la v15 y agrega:

- centro de diagnostico de servicios, fuentes y snapshots;
- bandeja de correo de voz de Ventas (2901) y Soporte (2902), al final del panel;
- reproduccion protegida de mensajes WAV/MP3 desde el almacenamiento de Issabel;
- inventario de peers IAX2, duplicados, disponibilidad, caidas y cambios de IP;
- deteccion automatica de IP local y lectura de la IP remota desde `BODEGA-IAX`;
- llamadas en curso con departamento y alerta de espera;
- explorador de CDR, detalle por piernas, filtros y exportacion CSV;
- correlacion IVR/llamada mediante `CHANNEL(linkedid)` y paginacion del explorador;
- comparacion con el periodo anterior y metricas ampliadas de grupos;
- reporte imprimible/guardable como PDF y CSV de evidencia;
- autenticacion web opcional preparada para Nginx.

La v16 se instala en `/opt/corporacion-litoral-v16`. El instalador conserva la
v15 y crea un respaldo adicional en `/opt/backups` antes de cambiar servicios.

## Instalacion

```bash
mkdir -p /opt/corporacion-litoral-v16
unzip -o /root/Actualizacion_Dashboard_Correo_Voz_v16.zip -d /opt/corporacion-litoral-v16
chmod +x /opt/corporacion-litoral-v16/scripts/install_v16.sh
/opt/corporacion-litoral-v16/scripts/install_v16.sh
```

Despues valide:

```bash
node /opt/corporacion-litoral-v16/scripts/verify_dashboard.mjs
```

Consulte `INSTALAR_ACTUALIZACION_DASHBOARD.md`, `MANUAL_DASHBOARD.md`,
`CAMBIOS_DASHBOARD_V16.md` antes de la demostracion final.
