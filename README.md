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
> **Importante**: todos debeis leeros la configuracion del cliente, saber como trabajar con el neo, desarrollar exploits...

## Servicios
- **IP servidor en la VPN**: `10.0.0.1`
- **Clientes**: `10.0.1.X/32`
- **Haduhana**: `http://10.0.0.1:3000/`
- **Ataka API**: `http://10.0.0.1:8000/`
