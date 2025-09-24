# Guía para conectar la infra al CTF

La configuración tiene sus particularidades dependiendo de la competición, pero hay varios
pasos que son casi siempre iguales.

## Vulnbox
Puede estar alojada por la organización o ser una máquina virtual que hay que desplegar como
parte de la infra. El acceso se hace mediante VPN, la de la CTF o una interna si la desplegamos
en nuestros servidores.

### Alojada por la organización
No requiere hacer mucho, leventar los dockers y poco más.

### Alojada por el equipo

## Haduhana
Tiene 3 modos, siempre que se tenga acceso como root a la máquina se usa el modo B, que hay que
configurarlo en el `docker-compose.yml`.

En el `.env` hay que completar los diferentes campos, los ticks, servicios, puertos...

Ahora, hay que replicar el tráfico de la interfaz de la vulnbox a una interfaz del servidor donde esta la Haduhana. Para ello hay que seguir los siguiente pasos:

 - Habilitar ebashl tunneling por SSH en la vulnbox:
    ```
    echo -e 'PermitTunnel yes' | sudo tee -a /etc/ssh/sshd_config
    systemctl restart ssh
    ```

- Crear tun5 y establecer el túnel SSH (desde la máquina de la Haduhana):
    ```bash
    sudo ip tuntap add tun5 mode tun user "$USER" 2>/dev/null || true
    sudo ip link set tun5 upo
    ssh -w 5:5 root@VULNBOX_IP "sudo ip link set tun5 up; ip addr show dev tun5"
    ```

- Verificar tun5 en ambos lados:
    - Local:
        ```bash
        ip -brief link show tun5
        ip addr show dev tun5
        ```
    - Vulnbox:
        ```bash
        ssh root@10.20.9.6 "ip -brief link show tun5; ip addr show dev tun5"
        ```

- Configurar nftables en la vulnbox para duplicar TODO el tráfico de la interfaz de la CTF hacia tun5:
    ```bash
    # Crear tabla netdev + cadenas atadas a la interfaz 'game'
    sudo nft add table netdev mirror
    sudo nft 'add chain netdev mirror ingress { type filter hook ingress device game priority 0; }'
    sudo nft 'add chain netdev mirror egress  { type filter hook egress  device game priority 0; }'

    # Duplicar TODO el tráfico de/para 'game' hacia tun5
    sudo nft add rule netdev mirror ingress  dup to tun5
    sudo nft add rule netdev mirror egress   dup to tun5

    # Listar para comprobar
    sudo nft list table netdev mirror
    ```

- Arrancar captura pcaps, dentro del directorio `/root/shovel/input-pcaps`:
    ```bash
    sudo tcpdump -n -i tun5 -G 30 -Z root -w trace-%Y-%m-%d_%H-%M-%S.pcap
    ```
    > **Importante**: Hay un bug conocido para la version 4.99 de tcpdump que provoca segfault al usar los parametros -Z y -w. 

- Comandos de debug:
    ```bash
    # comprobar contadores de reglas (muestra cuántos paquetes pasan por cada regla)
    sudo nft list table netdev mirror -a

    # ver interfaces
    ip -brief link show game tun5

    # ver tráfico en vivo en tun5
    sudo tcpdump -n -i tun5 -c 20
    ```

- Comandos de limpieza:
    - vulnbox:
        ```bash
        # borrar tabla nft (elimina reglas)
        sudo nft delete table netdev mirror
        ```
    - local:
        ```
        # en local
        sudo ip link set tun5 down
        sudo ip tuntap del tun5 mode tun 2>/dev/null || true

        ssh root@VULNBOX_IP "sudo ip link set tun5 down; sudo ip tuntap del tun5 mode tun 2>/dev/null || true"
        ```

## Farm
Para la farm solo hay que actualizar el archivo `server/app/config.py` con las opciones específicas de la CTF. Para sacar el array de equipos hay un script en el directorio utils.

## Neo
El neo no necesita configuración adicional si esta debidamente conectado al farm.

> **Importante**: si se reinicia el neo hay que reconectar todos los clientes, aunque no se vea ningun error. Las estadísticas de clientes se pueden ver en la dashboard de grafana `10.0.0.1:3000`