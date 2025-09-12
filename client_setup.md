# Cliente AD 101, de script kidde a master

## VPN
Para poder conectarte al servidor hay que utilizar wireguard, lo primero es instalarlo
```
sudo apt install wireguard
```

Ahora creamos la configuracion
```
sudo nano /etc/wireguard/wg0.conf

[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.0.1.X/32

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = SERVER_IP:25412
AllowedIPs = 10.0.0.0/24
```

La ip de la interfaz, clave publica de cada usuario y la clave privada y la ip del servidor estaran en el discord.

Con esto ya tenemos la VPN configurada y podemos levantar el tunel.
```
sudo wg-quick up wg0
```

Hacemos una prueba de conexion
```
ping 10.0.0.1
```

Si todo va bien ya tienes acceso ha toda la infraestructura

## NEO
Hay que descargarse la ultima release del `neo_client_...` de [aqui](https://github.com/C4T-BuT-S4D/neo/releases).
La descomprimes y te metes dentro de la carpeta

```
gzip -d neo_client_2.5.0-rc11_linux_amd64.tar.gz
tar -vxf neo_client_2.5.0-rc11_linux_amd64.tar
cd neo_client_2.5.0-rc11_linux_amd64
```

Actualizamos `client_config.yml`
```
host: "10.0.0.1:5005"
exploit_dir: "exploits"
grpc_auth_key: "s3cret_t0ken_pls_d0nt_leak"

metrics:
  url: "10.0.0.1:8428/api/v1/import/prometheus"
  user: "admin"
  password: "1234"
```

Tanto el token como las credenciales de cada uno estaran en el discord.

Ahora probamos la conexion con el servidor.
```
./neo run

INFO[2025-09-12T15:03:37.942+02:00] root.go:61 Using config file:client_config.yml          
DEBU[2025-09-12T15:03:37.942+02:00] root.go:70 Got configuration: map[config:client_config.yml exploit_dir:exploits grpc_auth_key:s3cret_t0ken_pls_d0nt_leak host:10.0.0.1:5005 metrics:map[password:1234 url:10.0.0.1:8428/api/v1/import/prometheus user:admin] verbose:true] 
DEBU[2025-09-12T15:03:37.942+02:00] config.go:13 Unmarshalled config &{Host:10.0.0.1:5005 ExploitDir:exploits GrpcAuthKey:s3cret_t0ken_pls_d0nt_leak UseTLS:false Metrics:0xc0001f78f0} 
INFO[2025-09-12T15:03:37.943+02:00] cli.go:72 Detected client id: 91f7fe3f4f414b4a97073e13fd0da00b 
       _
   ___| |__  ___   _ __   ___  ___    _ __ _   _ _ __  _ __   ___ _ __
  / __| '_ \/ __| | '_ \ / _ \/ _ \  | '__| | | | '_ \| '_ \ / _ \ '__|
 | (__| |_) \__ \ | | | |  __/ (_) | | |  | |_| | | | | | | |  __/ |
  \___|_.__/|___/ |_| |_|\___|\___/  |_|   \__,_|_| |_|_| |_|\___|_|
                                                                       
DEBU[2025-09-12T15:03:38.943+02:00] sender.go:78 Sending 0 logs
```

Si no da errores ya esta todo listo

## Exploits


TODO: Si falta algo de config, uso de neo