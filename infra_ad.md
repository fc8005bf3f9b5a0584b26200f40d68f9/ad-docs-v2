# Infra 101, desde cero hasta hero

## Herramientas que vamos a utilizar

- Docker (Docker compose se instala separado)
- UFW (contabo expone todo el trafico, firewall aisla)
- Wireguard VPN (mas facil que OpenVPN)

## Emepzar

### Instalar herramientas

``apt install ufw docker.io wireguard``

Instalar [`docker-compose-plugin`](https://docs.docker.com/compose/install/linux/#install-using-the-repository). Es necesario añadir el repositorio y luego

``apt install docker-compose-plugin``

### Configurar firewall

``ufw allow 22/tcp``
``ufw allow 25412/udp``
``ufw enable``

Se configura y se activa el firewall.

### Configurar VPN Servidor

Primer paso es generar las claves de la VPN.

```
cd /etc/wireguard
wg genkey > privatekey 
wg pubkey < privatekey > publickey
```

A continuación es necesario copiar la clave privada. Luego crear un archivo `wg0.conf` en la misma carpeta de wireguard.

```
[Interface]
PrivateKey = SERVER_PRIVATE (la que se ha generado anteriormente)
Address = 10.0.0.1/24
ListenPort = 25412

## Aqui abajo se ponen las claves publicas de cada cliente
[Peer]
AllowedIPs = 10.0.1.2/32
PublicKey = CLIENT PUBLIC KEY
```

### Configurar VPN Cliente

A continuación cada cliente deberá de hacer su propio par de claves privadas y publicas.

```
wg genkey > privatekey 
wg pubkey < privatekey > publickey
```

Luego se tendrá que configurar de la siguiente manera el cliente.

```
[Interface]
PrivateKey = CLIENT_PRIVATE (la que se ha generado anteriormente)
Address = 10.0.1.2/32

[Peer]
PublicKey = SERVER_PUBLIC
Endpoint = SERVER_IP:LISTENPORT
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

### Quitar SSH expuesto

Modificar el fichero ``/etc/systemd/system/sshd.service``. En la linea `After`, añadir wg-quick@wg0.service

```
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target nss-user-lookup.target wg-quick@wg0.service auditd.service
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

Finalmente, borrar SSH del firewall

```
# Permitir conexiones de wg0
sudo ufw allow in on wg0 to any port 22

# Borrar regla antigua de ALLOW IN Anywhere
ufw status numbered
ufw delete NUMERO_LINEA
```

Siendo el número de linea el indicador producido por el comando `ufw status numbered`

Resultado final 

```
root@spain-ecsc:/home/nacabaro# sudo ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 25412/udp                  ALLOW IN    Anywhere                  
[ 2] 22 on wg0                  ALLOW IN    Anywhere                  
[ 3] 25412/udp (v6)             ALLOW IN    Anywhere (v6)             
[ 4] 22 (v6) on wg0             ALLOW IN    Anywhere (v6)
```
 
Reiniciar

### Configurar Shovel (Haduhana)

```
sudo apt install git
git clone https://github.com/FCSC-FR/shovel
cd shovel
```

Es necesario configurar .env, sino no fufa.

A continuación, modificar `docker-compose.yml`. Modificar linea 44:

```yml  
    ...
    ports:
      - 127.0.0.1:8000:8000 
```

Por lo siguiente

```yml  
    ...
    ports:
      - 10.0.0.1:8000:8000 # O la IP del servidor de wireguard.
```

Y se levanta el docker.

```
docker compose up -d 
```

Una vez levantado, añadir a firewall

```
sudo ufw allow in on wg0 to any port 8000
```

### Configurar S4DFarm
```
git clone https://github.com/C4T-BuT-S4D/S4DFarm
```
Antes de iniciar neo hay que configurar la Farm. Para ello mirar el README.md de S4DFarm.

```
nano ./server/app/config.py
```

Modificar `server/docker/front/Dockerfile`. Añadir esto

```
FROM front-base AS front-build
RUN corepack use pnpm@8.x                    # Añadir esta linea en esta ubicacion
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile
RUN pnpm run build
```

Modificar las contraseñas es recomendado. Añadir a todos los puertos del `compose.yml` del S4DFarm el valor `10.0.0.1`, como se ha visto antes en el Shovel.

Finalmente, guardar y salir.

```
docker compose up -d
```

Una vez levantado, añadir a firewall

```
sudo ufw allow in on wg0 to any port 5137 ?????
```

### Configurar neo

Descargar la ultima [release](https://github.com/pomo-mondreganto/neo/releases) del servidor

Modificar `configs/server/config.yml`, actualizar `grpc_auth_key`, la password del farm y actualizar las urls a 10.0.0.1
Modificar `compose.yml` poniendo `10.0.0.1` en todos los ports

TODO: Terminar reglas firewall o no, no se si hay que bloquear algo mas
      Tambien hacer forks y dejar los compose.yml actualizados en nuestros repos
      