# Documentación AD - Attack/Defense CTF

## Esquema de Infraestructura
```mermaid
flowchart LR
    n1["Game VPN"] <--> n2(["Yampa"]) & n4["Team Server"]
    n2 <--> n3["Vulnbox"]
    n4 --> n5(["Tulip"]) & n6(["Ataka"])
    n3 <--> n5

    n1@{ shape: rounded}
    n4@{ shape: rounded}
    n3@{ shape: rounded}
```

## Documentación
- [Despliegue de la infraestructura](./infra_ad.md)
- [Configuración por CTF](ctf_setup.md)
- [Configuración de cliente](./client_setup.md)
- [Desarrollo de exploits](exploit_guide.md)

- **IP servidor en la VPN**: `172.16.10.1`
- **Clientes**: `172.16.10.X/32`
- **Haduhana**: `http://172.16.10.1:8000/`
- **Farm**: `http://172.16.10.1:5137/`
- **Grafana Dashboard**: `http://172.16.10.1:3000/`
- **Neo Exploits**: `http://172.16.10.1:5005/`
