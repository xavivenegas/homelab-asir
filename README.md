# homelab-asir

Laboratorio de prácticas del ciclo ASIR: Windows Server, Active Directory, DNS/DHCP e integración con Ubuntu Server, montado en VirtualBox.

## Fases

| Fase | Contenido | Estado |
|---|---|---|
| [01](01-windows-server-ad/) | Windows Server 2025 y Active Directory (`lab.local`) | Completada |
| [02](02-dns-dhcp/) | Reenviadores DNS, hora (NTP) y DHCP | Configurada (falta probar con un cliente) |
| 03 | Conexión de un servidor Ubuntu físico, unión al dominio y monitorización con Zabbix | Pendiente |

## Esquema de red

```
Portátil (VirtualBox)
  DC01 (Windows Server 2025)
   - Red interna "lab-asir": 10.10.10.10/24 (DHCP 10.10.10.100-200)
   - NAT: salida a internet

Servidor Ubuntu físico (por cable)  --->  por conectar a DC01 (Fase 3)
```

> Nota: no se publican contraseñas, claves ni datos personales en este repositorio.
