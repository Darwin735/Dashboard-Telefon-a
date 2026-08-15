# Instalacion segura del dashboard v16

## 1. Copiar desde Windows

```powershell
scp ".\Instalador PBX\Actualizacion_Dashboard_Correo_Voz_v16.zip" root@IP_ACTUAL_PRINCIPAL:/root/
ssh root@IP_ACTUAL_PRINCIPAL
```

`IP_ACTUAL_PRINCIPAL` es la IP que muestra `ip -br -4 addr` en ese momento.

## 2. Extraer e instalar en una carpeta nueva

```bash
mkdir -p /opt/corporacion-litoral-v16
unzip -o /root/Actualizacion_Dashboard_Correo_Voz_v16.zip -d /opt/corporacion-litoral-v16
chmod +x /opt/corporacion-litoral-v16/scripts/install_v16.sh
/opt/corporacion-litoral-v16/scripts/install_v16.sh
```

El instalador copia `.env` primero desde v15, fuerza `PBX_LOCAL_IP=auto` e
`IAX_PEER_IP=auto`, respalda la v15 y las configuraciones activas, ejecuta
pruebas, compila Next.js e instala
los cuatro servicios. No elimina la version anterior.

## 3. Comprobar

```bash
systemctl is-active voip-api.service voip-frontend.service voip-live-monitor.service voip-system-monitor.service
curl -s http://127.0.0.1:4000/api/health
curl -s http://127.0.0.1:4000/api/system-status
curl -s http://127.0.0.1:4000/api/security
curl -s http://127.0.0.1:4000/api/trunk
curl -s http://127.0.0.1:4000/api/voicemail
node /opt/corporacion-litoral-v16/scripts/verify_dashboard.mjs
```

La troncal puede aparecer `DOWN` si la PBX bodega no esta encendida. Eso no es
un fallo del dashboard: el diagnostico debe indicar que el peer esta configurado
pero no responde.

## 4. Acceso

```powershell
ssh -N -L 3000:127.0.0.1:3000 root@IP_ACTUAL_PRINCIPAL
```

Abra `http://localhost:3000` y mantenga la ventana del tunel abierta.

## 5. Autenticacion web opcional

Si ya usa el proxy HTTPS incluido:

```bash
chmod +x /opt/corporacion-litoral-v16/scripts/enable_dashboard_auth.sh
/opt/corporacion-litoral-v16/scripts/enable_dashboard_auth.sh operador
```

La clave debe tener al menos 12 caracteres. Esta accion es opcional y no se
activa automaticamente para evitar bloquear el acceso existente.

## 6. Restaurar la version anterior

```bash
BACKUP_DIR=$(cat /root/ULTIMO_RESPALDO_DASHBOARD)
bash /opt/corporacion-litoral-v16/scripts/restore_previous_version.sh "$BACKUP_DIR"
```

La restauracion repone servicios y configuraciones respaldadas. La carpeta
`/opt/corporacion-litoral-v15` tambien permanece intacta.
