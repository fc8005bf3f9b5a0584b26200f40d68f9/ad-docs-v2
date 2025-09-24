# Documentación AD - Attack/Defense CTF

## Esquema de Infraestructura
```mermaid
flowchart TD
    A[Farm] --> Z[Submitea las flags]

    B[Neo-server] <-->|Intercambio de exploits| C[Neo-client 1] --> F[Lanzan todos los exploits]
    B[Neo-server] <-->|Intercambio de exploits| D[Neo-client 2] --> F[Lanzan todos los exploits]
    B[Neo-server] <--> |Intercambio de exploits|E[Neo-client X] --> F[Lanzan todos los exploits]
    
    F[Lanzan todos los exploits] --> |Mandan flags|A[Farm]
```

## Documentación
- [Despliegue de la Infraestructura](./infra_ad.md)
- [Configuración de la infra por CTF](ctf_setup.md)
- [Configuración de Cliente](./client_setup.md)
- [Desarrollo de exploits](exploit_guide.md)
> **Importante**: todos debeis leeros la configuracion del cliente, saber como trabajar con el neo, desarrollar exploits...

## Servicios
- **IP servidor en la VPN**: `10.0.0.1`
- **IP Servidor**: `10.0.0.1`
- **Clientes**: `10.0.1.X/32`
- **Haduhana**: `http://10.0.0.1:8000/`
- **Farm**: `http://10.0.0.1:5137/`
- **Grafana Dashboard**: `http://10.0.0.1:3000/`
- **Neo Exploits**: `http://10.0.0.1:5005/`
