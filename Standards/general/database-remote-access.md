# Database Remote Access — bastion VM en GCP + IAP

Cómo conectarte a una DB gestionada (DigitalOcean, o cualquier motor con firewall por IP —
"Trusted Sources") desde una máquina/red cuya IP no está autorizada, sin abrir el firewall de la
DB a internet ni exponer un servidor público nuevo.

---

## 1. Cuándo aplica

Cualquier DB con una lista de IPs permitidas (DigitalOcean Managed DB, Cloud SQL con IP
autorizada, RDS con security group por IP, etc.) donde ya existe un recurso de GCP —típicamente
un Cloud Run o una Cloud Function— cuya IP de salida SÍ está en esa lista (vía Cloud NAT). El
patrón resuelve "quiero esa misma IP de salida pero desde mi laptop/otra red", sin tocar la lista
de IPs autorizadas de la DB cada vez que cambias de lugar.

## 2. Por qué un bastion y no otra cosa

Cloud NAT solo traduce la salida de recursos que ya están *dentro* de la VPC que administra — no
se le puede "pedir prestada" la IP directamente desde una máquina externa. La solución es meter un
recurso barato de GCP dentro de esa misma VPC/subred (así su salida usa el mismo NAT) y saltar a
través de él:

1. VM `e2-micro` **sin IP pública**, en la subred con el Cloud NAT ya configurado.
2. Acceso solo por **IAP TCP tunneling** (`gcloud compute ssh --tunnel-through-iap`) — nunca abrir
   el puerto 22 a `0.0.0.0/0`. IAP usa la identidad de IAM de quien se conecta, no llaves SSH
   públicas ni un firewall abierto.
3. `-L <puerto-local>:<host-real-de-la-db>:<puerto-real>` en el mismo comando SSH hace el forward
   — el cliente local (Workbench, `mysql`, Alembic, `psql`, etc.) se conecta a
   `127.0.0.1:<puerto-local>` como si la DB estuviera ahí mismo.

## 3. Requisitos antes de crear la VM

- La VPC/subred con el Cloud NAT ya debe existir (normalmente ya está, si algún servicio de Cloud
  Run/Cloud Functions del mismo proyecto ya llega a esa DB).
- Confirmar la IP de salida del NAT y que **ya esté** en la lista de IPs autorizadas de la DB —
  si no, agregarla ahí primero (fuera de GCP, en el panel del proveedor de la DB).
- Habilitar la API de IAP: `gcloud services enable iap.googleapis.com --project=<proyecto>`.
- El usuario que se conecta necesita `roles/iap.tunnelResourceAccessor` (incluido en `roles/owner`).

## 4. Comandos — creación

```bash
# Regla de firewall: solo el rango de IAP puede llegar al puerto 22, solo a VMs con este tag
gcloud compute firewall-rules create allow-iap-ssh-<nombre-vm> \
  --project=<proyecto> \
  --network=<vpc-con-el-nat> \
  --direction=INGRESS --action=ALLOW --rules=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --target-tags=<nombre-vm>

# VM sin IP pública, en la subred del NAT
gcloud compute instances create <nombre-vm> \
  --project=<proyecto> --zone=<zona> \
  --machine-type=e2-micro \
  --image-family=debian-12 --image-project=debian-cloud \
  --boot-disk-size=30GB --boot-disk-type=pd-standard \
  --network=<vpc-con-el-nat> --subnet=<subred-con-el-nat> \
  --no-address --tags=<nombre-vm> \
  --shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring
```

`e2-micro` en `us-west1`/`us-central1`/`us-east1` entra en el Always Free tier de GCP (1 instancia
por cuenta de facturación, 30GB de disco estándar incluidos).

## 5. Conectarte

```bash
gcloud compute ssh <nombre-vm> --zone=<zona> --project=<proyecto> --tunnel-through-iap \
  -- -L <puerto-local>:<host-real-db>:<puerto-real-db> -N
```

Deja esa terminal abierta (es el túnel; no imprime nada mientras corre) y apunta el cliente a
`127.0.0.1:<puerto-local>`. Para MySQL en Workbench: Connection Method `Standard (TCP/IP)`, host
`127.0.0.1`, el puerto local elegido, SSL en modo `Require` (no hace falta CA salvo que el
proveedor exija `verify-ca`/`verify-full`).

Verificar antes de dar por buena la VM (una sola vez, al crearla):

```bash
# Confirma que la IP de salida es la esperada (debe ser la misma que Cloud NAT)
gcloud compute ssh <nombre-vm> --zone=<zona> --project=<proyecto> --tunnel-through-iap \
  --command="curl -s ifconfig.me"

# Confirma que el puerto de la DB responde desde la VM
gcloud compute ssh <nombre-vm> --zone=<zona> --project=<proyecto> --tunnel-through-iap \
  --command="timeout 5 bash -c '</dev/tcp/<host-real-db>/<puerto-real-db>' && echo OK || echo FALLO"
```

## 6. Encendido, apagado y costo

El cobro de Compute Engine es **solo por el tiempo que la VM está encendida** (por segundo,
mínimo 1 minuto) — prender y apagar no tiene costo aparte, así que conviene apagarla cuando no se
usa:

```bash
gcloud compute instances stop  <nombre-vm> --zone=<zona> --project=<proyecto>
gcloud compute instances start <nombre-vm> --zone=<zona> --project=<proyecto>
```

## 7. Auto-apagado por inactividad (recomendado)

Para no tener que acordarte de apagarla: un cron dentro de la VM revisa cada 5 minutos si hay una
conexión SSH activa (el túnel mismo) y, si no la hay durante N minutos seguidos, la apaga sola.
Un `shutdown` iniciado desde adentro de una VM de GCE la deja **detenida** (mismo estado que
`gcloud compute instances stop`) — no se reinicia sola, así que no sigue generando costo.

```bash
# Correr una vez, vía SSH/IAP, dentro de la VM ya creada:
sudo tee /usr/local/bin/auto-shutdown-idle.sh > /dev/null << 'SCRIPTEOF'
#!/bin/bash
IDLE_MINUTES=30
LAST_ACTIVE_FILE=/var/run/last-ssh-activity

active=$(ss -Ht state established "( dport = :22 or sport = :22 )" | wc -l)

if [ "$active" -gt 0 ]; then
  date +%s > "$LAST_ACTIVE_FILE"
else
  if [ -f "$LAST_ACTIVE_FILE" ]; then
    last=$(cat "$LAST_ACTIVE_FILE")
    now=$(date +%s)
    diff=$(( (now - last) / 60 ))
    if [ "$diff" -ge "$IDLE_MINUTES" ]; then
      logger "auto-shutdown-idle: sin conexiones SSH activas por $diff min, apagando"
      shutdown -h now
    fi
  else
    date +%s > "$LAST_ACTIVE_FILE"
  fi
fi
SCRIPTEOF
sudo chmod +x /usr/local/bin/auto-shutdown-idle.sh
echo "*/5 * * * * root /usr/local/bin/auto-shutdown-idle.sh" | sudo tee /etc/cron.d/auto-shutdown-idle > /dev/null
sudo chmod 644 /etc/cron.d/auto-shutdown-idle
```

Ajusta `IDLE_MINUTES` según qué tan seguido reprendes la VM — como el encendido no cuesta nada
aparte, conviene errar del lado de un umbral corto (ej. 30 min) en vez de dejarla encendida "por
si acaso".

## 8. Instancia real ya montada con este patrón

`buho-db-connection` (proyecto `prod-apps-y-computo`, zona `us-central1-a`) — bastion para la
MySQL de DigitalOcean del proyecto Nexus/Connect. Detalle completo (host real, credenciales, qué
proyecto la usa) en el `CLAUDE.md` de `Nexus - DB`, no aquí — este archivo es el patrón genérico,
reutilizable para cualquier proyecto futuro con el mismo problema.

---
**Version:** 1.0
**Last Updated:** 2026-08-07
