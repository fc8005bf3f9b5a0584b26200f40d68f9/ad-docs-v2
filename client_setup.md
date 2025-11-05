# Cliente AD 101, de script kiddie a master

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

La ip de la interfaz, clave publica, la clave privada y la ip del servidor estaran en el discord.

Con esto ya tenemos la VPN configurada y podemos levantar el tunel.
```
sudo wg-quick up wg0
```

Hacemos una prueba de conexion
```
ping 10.0.0.1
```

Si todo va bien ya tienes acceso a toda la infraestructura

## Ataka
Para usar ataka tienes que descargarte el cliente
```
curl http://10.0.0.1:8000/ -o atk
mv atk ~/.local/bin/atk
```
> Este cliente es diferente para cada ctf y se debe actualizar durante la ctf si se cambia la configuración de ataka con `atk reload`

Para usar ataka ver [guía de exploits](./exploit_guide.md)