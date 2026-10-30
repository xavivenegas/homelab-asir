# Fase 1 – Windows Server 2025: Active Directory

Estado: **completada** (dominio `lab.local` creado y verificado).

Laboratorio de ASIR en VirtualBox. Objetivo de la fase: dejar un controlador de dominio funcionando (`lab.local`) para después añadir DNS/DHCP y unir un servidor Ubuntu.

## Entorno

| Elemento | Valor |
|---|---|
| Hipervisor | Oracle VirtualBox |
| Sistema | Windows Server 2025 Standard Evaluation (licencia de evaluación de 180 días) |
| Nombre del servidor | `DC01` |
| Red | Red interna `lab-asir` |
| IP del servidor | `10.10.10.10/24` (fija, sin puerta de enlace en la red interna) |
| DNS preferido | `10.10.10.10` (el propio servidor) |
| Dominio / bosque | `lab.local` (NetBIOS: `LAB`), nivel funcional Windows Server 2025 |

## Pasos

### 1. Preparación de la máquina
1. Instalar Windows Server 2025 y renombrar el equipo a `DC01`.
2. Configurar IP fija `10.10.10.10` y DNS preferido `10.10.10.10`.
3. Quitar la puerta de enlace de la red interna: no hay ningún router en ella.

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.10.10
Get-NetIPConfiguration
```

### 2. Instalar el rol AD DS
```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Get-WindowsFeature AD-Domain-Services
```
Debe aparecer como `Installed`. Reiniciar si lo pide.

### 3. Promover a controlador de dominio
Comprobar que no hay reinicio pendiente y promover (la contraseña DSRM se pide por consola y no se publica):

```powershell
Test-Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending"
Install-ADDSForest -DomainName "lab.local" -DomainNetbiosName "LAB" -InstallDns
```
El aviso sobre la delegación DNS es normal en un dominio nuevo. El servidor se reinicia solo.

### 4. Verificación
```powershell
dcdiag
Get-ADDomain
```
Resultado: `Get-ADDomain` muestra `lab.local`, NetBIOS `LAB`, `DomainMode: Windows2025Domain`, y DC01 con los roles PDC, RID e Infrastructure Master. `dcdiag` supera las pruebas de conectividad, publicación, SYSVOL, replicación, servicios y particiones. Fallan `DFSREvent` y `SystemLog`, algo habitual justo después de promover (ver abajo).

## Problemas y soluciones

| Problema | Causa | Solución |
|---|---|---|
| El asistente instalaba roles que no quería (AD FS, Servicios de Escritorio remoto) | Marqué roles de más en la lista | Desmarcarlos y dejar únicamente AD DS |
| Tras cerrar el asistente no aparecía AD DS en el panel | La instalación no llegó a ejecutarse | Repetir «Agregar roles» hasta pulsar Instalar y esperar al mensaje de éxito |
| Error en la promoción: «El cambio de rol está en curso o este equipo debe reiniciarse» | Reinicio pendiente | Reiniciar, comprobar `RebootPending` = False y repetir la promoción |
| Aviso de delegación DNS | Dominio nuevo sin DNS superior | Ignorar; no es un error |
| DNS del adaptador apuntaba a 8.8.8.8 | Configuración inicial | Cambiar a 10.10.10.10 antes de promover |
| El servidor no tenía internet | Solo tenía la red interna | Añadir un segundo adaptador en NAT (DHCP) |
| `dcdiag`: DFSREvent y SystemLog con errores | Eventos de arranque tras la promoción, resolución de nombres externos sin reenviador DNS, sincronización de hora sin origen externo | Repetir `dcdiag` pasado un rato y configurar un reenviador DNS (Fase 2) |

## Próximas fases
- Fase 2: reenviador DNS, DHCP y revisión de `dcdiag`.
- Fase 3: integrar un servidor Ubuntu en el dominio `lab.local`.
