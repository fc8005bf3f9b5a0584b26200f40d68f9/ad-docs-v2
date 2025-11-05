# Infra 101, desde cero hasta hero

## Setup servidor
La guía esta hecha para un servidor con Debian 13

### Instalar herramientas

```
apt update && apt upgrade -y
apt install ufw wireguard git -y
```

Instalar [`docker`](https://docs.docker.com/engine/install/debian/#install-using-the-repository), siguiendo la documentación oficial.

### Configurar VPN Servidor

Primer paso es generar las claves del servidor.

```
cd /etc/wireguard
wg genkey | tee privatekey | wg pubkey > publickey
```

A continuación es necesario copiar la clave privada. Luego crear un archivo `wg0.conf` en la misma carpeta de wireguard.

```
[Interface]
PrivateKey = SERVER_PRIVATE_KEY
Address = 10.0.0.1/24
ListenPort = 25412

```

### Configurar VPN Cliente
Primero agregamos los clientes a la configuración:
`./add_clientes.sh CLIENTS_NUM`

```bash
#!/bin/bash

NUM_CLIENTES=$1
IP_INICIAL=2

WG_DIR="/etc/wireguard"
CONFIG_FILE="$WG_DIR/wg0.conf"
KEYS_DIR="$WG_DIR/clients_keys"
IP_BASE="10.0.1."

if [[ -z "$1" ]]; then
    echo "Error: No se especificó el número de clientes."
    echo "Uso: $0 <numero_de_clientes_a_generar>"
    exit 1
fi

mkdir -p "$KEYS_DIR"
chmod 700 "$KEYS_DIR"
echo "Directorio de claves asegurado en $WG_DIR/$KEYS_DIR"

declare -a client_privkeys_output

echo -e "\n# Clientes" >> "$CONFIG_FILE"

CURRENT_OCTET=$IP_INICIAL-1

echo "Generando $NUM_CLIENTES clientes nuevos..."

for (( i=1; i<=$NUM_CLIENTES; i++ )); do
    
    # Incrementamos el octeto para el nuevo cliente
    CURRENT_OCTET=$((CURRENT_OCTET + 1))
    
    CLIENT_IP="${IP_BASE}${CURRENT_OCTET}/32"
    
    PRIV_KEY_FILE="${KEYS_DIR}/${CURRENT_OCTET}_privatekey"
    PUB_KEY_FILE="${KEYS_DIR}/${CURRENT_OCTET}_publickey"
    
    echo "-------------------------------------"
    echo "Generando Cliente (IP: $CLIENT_IP)"

    CLIENT_PRIV_KEY=$(wg genkey)
    CLIENT_PUB_KEY=$(echo "$CLIENT_PRIV_KEY" | wg pubkey)

    echo "$CLIENT_PRIV_KEY" > "$PRIV_KEY_FILE"
    echo "$CLIENT_PUB_KEY" > "$PUB_KEY_FILE"
    
    chmod 600 "$PRIV_KEY_FILE"

    echo "Claves generadas y guardadas en $KEYS_DIR/"

    cat << EOF >> "$CONFIG_FILE"

[Peer]
AllowedIPs = $CLIENT_IP
PublicKey = $CLIENT_PUB_KEY
EOF

    echo "Cliente añadido a $CONFIG_FILE"
    client_privkeys_output+=("$CLIENT_IP - $CLIENT_PRIV_KEY")

done

# --- Salida Final ---
echo "======================================================"
echo " Proceso completado."
echo " Claves privadas de los clientes generados:"

for key_info in "${client_privkeys_output[@]}"; do
    echo "$key_info"
done

echo "======================================================"

exit 0
```

Luego se tendrá que configurar cada cliente.

```
[Interface]
PrivateKey = CLIENT_PRIVATE
Address = 10.0.1.X/32

[Peer]
PublicKey = SERVER_PUBLIC
Endpoint = SERVER_IP:25412
AllowedIPs = 10.0.0.0/24
```

### Iniciar VPN servidor

```
systemctl enable wg-quick@wg0
systemctl start wg-quick@wg0
```

### Iniciar VPN cliente 

Tiene que guardarse el archivo wg0.conf del cliente dentro de ``/etc/wireguard/``. A continuación, iniciar el tunel.

```
sudo wg-quick up wg0
```

Podemos comprobar que el tunel funciona haciendo un ping desde el cliente al servidor.

También podmeos comprobar que el servidor se conecta al cliente usando `wg`

Ejemplo de cuando funciona

```
root@spain-ecsc:/etc/wireguard# wg
interface: wg0
  public key: MVjmbLh3iZQHqtSHoVTXJlsDlypTqKGskWrczyL4gRs=
  private key: (hidden)
  listening port: 25412

peer: JqPWkpWaJ2b3+yTY22DZ3yf914Q09WgfNL16FzZMOzo=
  endpoint: 192.168.122.1:33519
  allowed ips: 10.0.1.2/32
  latest handshake: 2 minutes, 14 seconds ago 
  transfer: 436 B received, 348 B sent  
```

Ejemplo de cuando NO funciona

```
interface: wg0
  public key: MVjmbLh3iZQHqtSHoVTXJlsDlypTqKGskWrczyL4gRs=
  private key: (hidden)
  listening port: 25412

peer: JqPWkpWaJ2b3+yTY22DZ3yf914Q09WgfNL16FzZMOzo=
  allowed ips: 10.0.1.2/32
```

### Configurar firewall
```
ufw allow 22/tcp
ufw allow 25412/tcp
ufw allow in on wg0
ufw allow out on wg0
ufw enable
```

Se configura y se activa el firewall.

### Quitar SSH expuesto

Modificar el fichero ``/etc/systemd/system/sshd.service``. En la linea `After`, añadir `wg-quick@wg0.service`

```
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target nss-user-lookup.target auditd.service wg-quick@wg0.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service]
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
Alias=sshd.service
```

Reiniciar el servidor

## Instalación herramientas AD
### Instalación Tulip (Haduhana)

```
git clone https://github.com/OpenAttackDefenseTools/tulip
cd tulip
```

Editamos el `docker-compose-suricata.yml` para cambiar el endpoint del frontend:
```
frontend:
  build:
    context: frontend
    dockerfile: Dockerfile-frontend
  image: tulip-frontend:latest
  restart: unless-stopped
  ports:
    - "10.0.0.1:3000:3000"
```

Copiamos el env y lo configuramos `cp .env.example .env`:
```
TRAFFIC_DIR_HOST="./traffic"
FLAGID_SCRAPE=1
FLAGID_SCAN=1
SURICATA_DIR_HOST="./suricata"
```

Creamos los directorios necesarios:
```
. .env
mkdir -p {${SURICATA_DIR_HOST}/{etc,lib/rules,log},${TRAFFIC_DIR_HOST}}
```

Copy [rules](suricata.rules) to `./suricata/lib/rules/suricata.rules`

### Instalación ataka
```
git clone https://github.com/OpenAttackDefenseTools/ataka
cd ataka
```

Editamos el `docker-compose.yml` para cambiar el endpoint de `postgres`, `adminer` y `api` (`10.0.0.1:PORT:PORT`)

En el .env hay que cambiar USERID a `0:0`
