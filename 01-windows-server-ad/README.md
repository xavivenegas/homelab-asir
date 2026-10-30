# Fase 1 – Windows Server 2025: Active Directory

Laboratorio de ASIR en VirtualBox. Objetivo de la fase: dejar un controlador de dominio funcionando (`lab.local`) para después añadir DNS/DHCP y unir un servidor Ubuntu.

## Entorno

| Elemento | Valor |
|---|---|
| Hipervisor | Oracle VirtualBox |
| Sistema | Windows Server 2025 Standard Evaluation (licencia de evaluación de 180 días) |
| Nombre del servidor | `DC01` |
| Red | Red interna `lab-asir` |
| IP del servidor | `10.10.10.10` (fija) |
| DNS preferido | `10.10.10.10` (el propio servidor) |
| Dominio | `lab.local` (NetBIOS: `LAB`) |

## Pasos

### 1. Preparación de la máquina
1. Instalar Windows Server 2025 y renombrar el equipo a `DC01`.
2. Configurar IP fija `10.10.10.10` y DNS preferido `10.10.10.10`.
3. Hacer una instantánea de la máquina antes de tocar roles.

### 2. Instalar el rol AD DS
1. Administrador del servidor → **Agregar roles y características**.
2. Tipo de instalación: basada en características o roles. Servidor de destino: `DC01`.
3. Marcar solo **Servicios de dominio de Active Directory** y aceptar **Agregar características**.
4. Confirmar e **Instalar**. Esperar a «Instalación correcta».

### 3. Promover a controlador de dominio
1. Bandera de notificaciones → **Promover este servidor a controlador de dominio**.
2. **Agregar un nuevo bosque** con nombre raíz `lab.local`.
3. Servidor DNS marcado y contraseña DSRM (no se publica en el repositorio).
4. Ignorar el aviso de delegación DNS. NetBIOS: `LAB`. Instalar; el servidor se reinicia.

### 4. Verificación
```powershell
dcdiag
Get-ADDomain
```
Se espera que `dcdiag` pase las pruebas y que `Get-ADDomain` muestre `lab.local`.

Después de verificar: instantánea llamada `AD-listo`.

## Problemas y soluciones

| Problema | Causa | Solución |
|---|---|---|
| El asistente instalaba roles que no quería (AD FS, Servicios de Escritorio remoto) | Marqué roles de más en la lista | Desmarcarlos y dejar únicamente AD DS |
| Tras cerrar el asistente no aparecía AD DS en el panel | La instalación no llegó a ejecutarse | Repetir «Agregar roles» hasta pulsar **Instalar** y esperar al mensaje de éxito |
| El servidor no tenía internet | Solo tenía la red interna | Añadir un segundo adaptador en NAT (DHCP) y dejar el DNS de la red interna en 10.10.10.10 |

## Próximas fases
- Fase 2: DNS y DHCP.
- Fase 3: integrar un servidor Ubuntu en el dominio `lab.local`.
