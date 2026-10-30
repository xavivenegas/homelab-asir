# homelab-asir

Laboratorio de prácticas del ciclo ASIR: Windows Server, Active Directory, DNS/DHCP e integración con Ubuntu Server, montado en VirtualBox.

## Fases

| Fase | Contenido | Estado |
|---|---|---|
| [01](01-windows-server-ad/) | Windows Server 2025 y Active Directory (`lab.local`) | Completada |
| 02 | Reenviador DNS, DHCP y revisión de `dcdiag` | Pendiente |
| 03 | Integración de Ubuntu Server en el dominio | Pendiente |

## Esquema de red

```
Red interna "lab-asir" (10.10.10.0/24)

DC01 (Windows Server)  <---->  Ubuntu Server
10.10.10.10                    (por definir)
AD DS + DNS
```

> Nota: no se publican contraseñas, claves ni datos personales en este repositorio.
