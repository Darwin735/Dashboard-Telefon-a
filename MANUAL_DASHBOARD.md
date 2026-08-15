# Manual operativo del dashboard v16

## Acceso

En PowerShell abra el tunel con la IP actual de la PBX principal:

```powershell
ssh -N -L 3000:127.0.0.1:3000 root@IP_ACTUAL_PRINCIPAL
```

Mantenga esa ventana abierta y visite `http://localhost:3000`.

## Lectura de la pantalla

- **KPIs:** total, contestadas, no contestadas y duracion promedio.
- **Llamadas en curso:** origen, destino, departamento, tecnologia, sentido,
  duracion y alerta cuando una llamada tarda mas de 20 segundos en contestarse.
- **Diagnostico:** separa fallos de API, MariaDB, CDR, IVR, snapshots, troncal y
  monitor de seguridad. Un peer `DOWN` no significa que el dashboard este caido.
- **Comparativo:** contrasta el periodo elegido con el periodo anterior de igual duracion.
- **IAX2:** muestra peer, IP usada actualmente por Asterisk, estado, latencia,
  duplicados, caidas, disponibilidad y cambios de IP.
- **Grupos:** Ventas y Soporte incluyen total, contestadas, no contestadas,
  voicemail, tasa de respuesta, espera y conversacion promedio.
- **Explorador:** filtra CDR por estado, departamento, sentido, duracion y texto.
  **Ver detalle** muestra todas las piernas relacionadas por `linkedid`.
- **Correo de voz:** es la ultima seccion. Resume los buzones 2901 (Ventas) y
  2902 (Soporte), identifica mensajes nuevos/escuchados/urgentes y permite
  reproducir los audios WAV o MP3 sin moverlos ni borrarlos.

## Cambio de red o IP

La v16 usa `PBX_LOCAL_IP=auto`. La direccion local se obtiene de la interfaz
preferida (`PBX_INTERFACE`) o de la primera IPv4 privada activa. La direccion
remota se obtiene del peer llamado `BODEGA-IAX` mediante Asterisk.

Por tanto, si cambia la red:

1. actualice la troncal `BODEGA-IAX` en Issabel con la IP actual de Bodega;
2. aplique la configuracion PBX;
3. confirme `asterisk -rx "iax2 show peer BODEGA-IAX"`;
4. el dashboard mostrara las nuevas IP sin editar su codigo.

El dashboard detecta y reporta la IP; no reconfigura automaticamente la troncal,
porque cambiar Asterisk desde un panel de reportes seria inseguro.

## Reportes y evidencias

Use **Generar evidencia** para abrir un resumen con fecha, periodo, KPIs, grupos,
troncal, llamadas activas y seguridad. Pulse **Imprimir / guardar PDF** y elija
“Guardar como PDF” en el navegador. Los botones CSV exportan llamadas, IVR o el
resumen de evidencia con datos reales del periodo.

## Comprobacion tecnica

```bash
systemctl is-active voip-api.service voip-frontend.service voip-live-monitor.service voip-system-monitor.service
curl -s http://127.0.0.1:4000/api/system-status
curl -s http://127.0.0.1:4000/api/security
curl -s http://127.0.0.1:4000/api/trunks
curl -s http://127.0.0.1:4000/api/live-calls
curl -s http://127.0.0.1:4000/api/voicemail
node /opt/corporacion-litoral-v16/scripts/verify_dashboard.mjs
```

El verificador ejecuta 24 controles. Puede dejar pendiente la limpieza de peers
si encuentra una troncal heredada como `IAX2-trunk1` o `IAX2-trunk2`; elimine el
duplicado desde Issabel y conserve solamente el nombre formal de cada sede.

## Correo de voz y reproduccion

La API lee solamente estos directorios de Issabel:

```text
/var/spool/asterisk/voicemail/default/2901/{INBOX,Urgent,Old}
/var/spool/asterisk/voicemail/default/2902/{INBOX,Urgent,Old}
```

Los buzones se pueden ajustar en `dashboard/backend/.env`:

```dotenv
VOICEMAIL_ROOT=/var/spool/asterisk/voicemail
VOICEMAIL_CONTEXTS=default,device
VOICEMAIL_MAILBOXES=2901:Ventas,2902:Soporte
VOICEMAIL_MAX_MESSAGES=100
```

El servicio pertenece de forma suplementaria al grupo `asterisk` y recibe acceso
de solo lectura al directorio. No cambie recursivamente los permisos del spool.
WAV y MP3 se reproducen en el navegador; un mensaje que exista solamente en GSM
se lista, pero muestra **Audio no disponible en WAV/MP3**. Reproducirlo en el
dashboard no lo marca como escuchado en el telefono.

## Autenticacion opcional

Con Nginx HTTPS configurado:

```bash
chmod +x /opt/corporacion-litoral-v16/scripts/enable_dashboard_auth.sh
/opt/corporacion-litoral-v16/scripts/enable_dashboard_auth.sh operador
```

La clave minima es de 12 caracteres. El instalador deja este control desactivado
para no interrumpir el acceso actual. Al habilitarlo, use la direccion HTTPS de
Nginx; el acceso directo por el tunel al puerto 3000 no incluye el usuario del
proxy y la API lo rechazara de forma intencional.

Para volver al acceso sin autenticacion:

```bash
/opt/corporacion-litoral-v16/scripts/disable_dashboard_auth.sh
```

## Solucion de problemas

| Sintoma | Revision |
|---|---|
| No abre `localhost:3000` | Tunel SSH, IP actual y `voip-frontend.service` |
| API sin conexion | `/api/health`, `.env`, MariaDB y `voip-api.service` |
| Diagnostico desactualizado | `voip-system-monitor.service` y archivos en `/run/voip-dashboard` |
| Troncal DOWN pero hay ping | `iax2 show peer BODEGA-IAX`, UDP 4569, `qualify=yes` y firewall |
| Peer duplicado | Eliminar `IAX2-trunk*` antiguo desde Issabel y aplicar configuracion |
| IVR vacio | AGI `ivr_event.php`, tabla `ivr_events` y llamada real por ambos menus |
| IVR visible pero no aparece en detalle | Reinstalar telemetria v16 para pasar `${CHANNEL(linkedid)}` al AGI y efectuar una llamada nueva |
| Llamadas activas vacias | Mantener una llamada conectada y revisar `channels.concise` |
| Calidad RTP pendiente | CDR no contiene RTCP; se declara la limitacion y no se simulan valores |
| Correo de voz vacio | Confirmar que 2901/2902 tienen voicemail habilitado y dejar un mensaje real |
| Mensaje sin reproductor | Revisar que junto a `msgNNNN.txt` exista `msgNNNN.wav` o `msgNNNN.mp3` |
| Audio responde 401 | Abrir el dashboard por la misma URL HTTPS/autenticada, no mezclar accesos directos |

La API es de solo lectura. `dashboard_reader` conserva privilegios `SELECT`; los
escritores auxiliares solo insertan eventos IVR y estados historicos.
