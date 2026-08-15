# VMware e IAX2 con DHCP en la misma red

## Modelo utilizado

Las dos VMs usan adaptador **Bridged/Puente** y reciben DHCP de la red donde se
encuentran las computadoras. No se usa VPN. La troncal funciona cuando ambas PBX
tienen direcciones alcanzables dentro de la misma LAN.

Las direcciones pueden cambiar de `192.168.x.x` a `10.x.x.x`. El dashboard v16
lo soporta: detecta la IP local y lee la IP remota que Asterisk tiene aplicada en
el peer `BODEGA-IAX`.

## Cuando cambie la red

En cada PBX:

```bash
ip -br -4 addr
ip route
```

Anote una IP de Principal y una de Bodega que pertenezcan a la misma subred.
Compruebe en ambos sentidos:

```bash
ping -c 4 IP_DE_LA_OTRA_PBX
```

En Issabel Principal, edite `BODEGA-IAX`:

```ini
host=IP_ACTUAL_BODEGA
qualify=yes
port=4569
```

En Issabel Bodega, edite su peer hacia Principal usando:

```ini
host=IP_ACTUAL_PRINCIPAL
qualify=yes
port=4569
```

Si utiliza `permit=` en los detalles entrantes, actualicelo tambien con la IP
actual de la otra PBX. Pulse **Enviar cambios** y **Aplicar configuracion**.

## Validacion

```bash
asterisk -rx "iax2 show peers"
asterisk -rx "iax2 show peer BODEGA-IAX"   # en Principal
```

El estado correcto es `OK (n ms)`. El dashboard actualizara las direcciones sin
editar `.env` porque usa:

```dotenv
PBX_LOCAL_IP=auto
IAX_TRUNK_NAME=BODEGA-IAX
IAX_CONNECTION_MODE=internal
IAX_REGISTRATION_REQUIRED=false
```

`IAX_PEER_IP` queda como valor de respaldo; la lectura en vivo por nombre tiene
prioridad. El dashboard no cambia `host=` dentro de Issabel automaticamente.

## Recomendacion practica

DHCP funciona para la demostracion, pero exige actualizar ambas troncales cada
vez que cambien las direcciones. Si el router lo permite, una reserva DHCP por
MAC evita esa tarea sin convertir las IP de las VMs en manuales.
