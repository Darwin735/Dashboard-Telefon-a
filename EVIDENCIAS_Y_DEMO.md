# Evidencias y guion de demostracion

## Capturas obligatorias

Guarde las imagenes con el nombre indicado y agreguelas al documento final antes de entregar.

1. `01_ip_pbx_principal.png`: `ip addr` y puerta de enlace de VM 1.
2. `02_ip_pbx_bodega.png`: `ip addr` y puerta de enlace de VM 2.
3. `03_ping_entre_vms.png`: ping bidireccional exitoso.
4. `04_extensiones_principal.png`: lista 2000-2012 en Issabel.
5. `05_extensiones_bodega.png`: lista 3001-3002 en Issabel.
6. `06_extension_sip_detalle.png`: una extension con secreto oculto y codecs.
7. `07_troncal_hq.png` y `08_troncal_bodega.png`: parametros IAX2 sin exponer la clave.
8. `09_iax_peer_ok.png`: `asterisk -rx "iax2 show peers"` con estado OK.
9. `10_llamada_2000_3001.png`: softphones y CLI durante una llamada entre sedes.
10. `11_ivr_7000.png` y `12_ivr_7001.png`: ambos niveles del IVR.
11. `13_grupo_ventas.png`: ringall, 25 s y fallback al buzon 2901.
12. `14_grupo_soporte.png`: hunt/rrmemory, 25 s y fallback al buzon 2902.
13. `15_voicemail_fallback.png`: registro o mensaje dejado al vencer el grupo.
14. `16_fail2ban.png`: `fail2ban-client status asterisk`.
15. `17_firewall.png`: reglas activas y politica de denegacion por defecto.
16. `18_anonimas_rechazadas.png`: opcion deshabilitada y prueba fallida sin autenticacion.
17. `19_dashboard_resumen.png`: KPIs con datos reales y distintivo **Datos en vivo**.
18. `20_dashboard_departamentos_grupos.png`: volumen por departamento y tasas de Ventas/Soporte después de llamar a ambos grupos.
19. `21_dashboard_horas_dias.png`: distribución por hora y evolución diaria del mismo periodo.
20. `22_dashboard_ivr.png`: opciones reales registradas en los IVR 7000/7001.
21. `23_dashboard_troncal.png`: estado actualizado, IP, latencia y origen de la lectura de la troncal.
22. `24_consulta_bd.png`: filas de `cdr`, `ivr_events` y `trunk_status` que correspondan a la demo.
23. `25_dashboard_correo_voz.png`: bandeja final con buzones 2901/2902 y un audio reproducible.
24. `26_dashboard_peers_historial.png`: inventario IAX2 sin duplicados e historial de disponibilidad.
25. `27_dashboard_llamada_activa.png`: llamada en curso con departamento y tecnologias.
26. `28_evidencia_dashboard.pdf`: reporte generado con **Generar evidencia**.
27. `29_verificador_dashboard.png`: salida completa de `node /opt/corporacion-litoral-v16/scripts/verify_dashboard.mjs` con `24/24` controles `[OK]`.

Nunca muestre contrasenas, cookies, llaves privadas ni secretos de troncal en las capturas.

## Secuencia de demo (5-7 minutos)

1. Mostrar el diagrama de red y las IP actuales obtenidas por DHCP en las dos VMs.
2. Ejecutar `asterisk -rx "iax2 show peers"`.
3. Llamar de 2000 a 3001 y contestar en la bodega.
4. Simular una llamada entrante al IVR 7000; pulsar 1 y comprobar que 2001-2004 suenan al mismo tiempo.
5. Repetir y pulsar 2; comprobar la secuencia del grupo 6200.
6. No contestar una llamada y demostrar el fallback al buzon.
7. Abrir el dashboard, pulsar **Aplicar periodo** y relacionar las llamadas con los KPIs, departamentos, Ventas/Soporte, horas/días e IVR.
8. Mostrar la troncal en `UP`; apagar o desconectar brevemente la VM 2 y comprobar que cambia a `DOWN` sin conservar un falso positivo.
9. Mantener una llamada activa y ejecutar `node /opt/corporacion-litoral-v16/scripts/verify_dashboard.mjs`; mostrar `24/24` controles `[OK]`.
10. Generar la evidencia, guardarla como PDF y exportar el CSV de llamadas.
11. Mostrar fail2ban, firewall y rechazo de llamada anonima.

## Pruebas de aceptacion

| ID | Prueba | Resultado esperado |
|---|---|---|
| T01 | Registrar 2000 | Extension disponible y audio en ambos sentidos |
| T02 | 2000 llama a 2001 | Llamada interna exitosa |
| T03 | 2000 llama a 3001 | Cruza la troncal IAX2 y hay audio bidireccional |
| T04 | IVR opcion 1 | Suenan 2001-2004 simultaneamente |
| T05 | IVR opcion 2 | Suenan 2005-2007 en secuencia/rrmemory |
| T06 | No contestar grupo | A los 25 s llega al buzon correspondiente |
| T07 | IVR opcion 4, luego 1 | Segundo nivel envia a Gerencia 2010 |
| T08 | Llamada sin autenticar | Rechazada y registrada por Asterisk/fail2ban |
| T09 | Actualizar dashboard | Aparecen KPIs, departamentos, tasas de ambos grupos, horas/días e IVR desde la BD |
| T10 | Apagar VM 2 | El estado vigente de Asterisk/dashboard cambia a DOWN; no se presenta una lectura UP vencida |
| T11 | Ejecutar verificador | Todos los requisitos del dashboard aparecen como `[OK]` |

Cada prueba debe registrar fecha, responsable, evidencia y resultado real (Aprobada/Fallida).
