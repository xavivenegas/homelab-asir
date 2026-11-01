# Fase 2 – DNS, hora y DHCP

Estado: **configurada** (falta probar el DHCP con un cliente real).

Continuación de la [Fase 1](../01-windows-server-ad/). Objetivo: que DC01 resuelva nombres de internet, tenga la hora correcta y reparta direcciones en la red interna.

## Red de DC01

| Adaptador | Conexión | IP | DNS | Notas |
|---|---|---|---|---|
| Ethernet | Red interna `lab-asir` | `10.10.10.10/24` (fija) | `127.0.0.1` | Sin puerta de enlace. Es la red que atiende el DHCP |
| Ethernet 2 | NAT de VirtualBox | DHCP (`10.0.3.15`) | `10.10.10.10` | Solo para internet. No registra su IP en el DNS del dominio |

## Pasos

### 1. Segundo adaptador (NAT) para internet
Con la máquina apagada desde Windows, en VirtualBox: Configuración → Red → Adaptador 2 → NAT.

Para que el NAT no ensucie el DNS del dominio:
```powershell
Set-DnsClient -InterfaceAlias "Ethernet 2" -RegisterThisConnectionsAddress $false
Set-DnsClientServerAddress -InterfaceAlias "Ethernet 2" -ServerAddresses 10.10.10.10
```

### 2. Reenviadores DNS
```powershell
Add-DnsServerForwarder -IPAddress 8.8.8.8
Add-DnsServerForwarder -IPAddress 1.1.1.1
```
Comprobación: `Resolve-DnsName google.com` resuelve y `Test-Connection 8.8.8.8` responde. En la zona `lab.local`, DC01 solo tiene el registro A `10.10.10.10`.

### 3. Hora (NTP)
DC01 es el emulador PDC y debe tener un origen de hora externo, porque Kerberos falla si los equipos se desfasan:
```powershell
w32tm /config /manualpeerlist:"0.pool.ntp.org,0x8 1.pool.ntp.org,0x8" /syncfromflags:manual /reliable:yes /update
Restart-Service w32time
w32tm /resync
w32tm /query /status
```

### 4. DHCP
```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName "dc01.lab.local" -IPAddress 10.10.10.10
Add-DhcpServerv4Scope -Name "LAB" -StartRange 10.10.10.100 -EndRange 10.10.10.200 -SubnetMask 255.255.255.0 -State Active
Set-DhcpServerv4OptionValue -ScopeId 10.10.10.0 -DnsServer 10.10.10.10 -DnsDomain "lab.local"
```
Ámbito `LAB`: 10.10.10.100 a 10.10.10.200, concesión de 8 días, sin puerta de enlace (la red interna no tiene router). Después, en el Administrador del servidor: «Completar la configuración de DHCP».

### 5. Verificación
```powershell
Get-DhcpServerv4Scope
Get-DhcpServerInDC
Get-DhcpServerv4Binding
dcdiag /test:dns
```
Resultado: ámbito `LAB` activo, `dc01.lab.local` autorizado, el DHCP solo escucha en `Ethernet` (la red interna) y `dcdiag /test:dns` pasa.

## Problemas y soluciones

| Problema | Causa | Solución |
|---|---|---|
| `dcdiag` sigue marcando DFSREvent y SystemLog | `dcdiag` revisa los eventos de las últimas 24 horas y arrastra los de justo después de promover | Repetir pasadas 24 horas. Los avisos de `_ldap._tcp` y de arranque son normales al iniciar |
| Errores DCOM con 8.8.8.8 y 1.1.1.1 | `dcdiag` intenta tratar los reenviadores como controladores de dominio | Inofensivo, lo provoca el propio `dcdiag` |
| Aviso en `dcdiag /test:dns` por IP dinámica en un adaptador | El NAT usa DHCP a propósito | Ignorar |
| Cierres inesperados del sistema | Apagué la máquina sin cerrar Windows | Apagar siempre desde Windows (`Stop-Computer`): en un controlador de dominio el apagado brusco puede dañar la base de datos de AD |
| El NAT usaba como DNS el router de casa | DHCP automático del NAT | Fijar su DNS a `10.10.10.10` y desactivar el registro de su IP en el DNS |

## Próxima fase
Conectar un servidor Ubuntu físico al laboratorio: que DC01 sea visible desde la red real, unir Ubuntu al dominio `lab.local` y monitorizar DC01 con Zabbix.
