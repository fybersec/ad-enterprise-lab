# 01 - Topología

> **Fase:** 1 (Topología) **Alcance:** diseño de la red del laboratorio, segmentación en VMware y rol de cada máquina. **Estado:** solo se documenta la **segmentación virtual**. Todavía no hay direcciones IP aplicadas ni conectividad verificada entre las máquinas. El Windows Server se configura en `02-active-directory.md` y el DHCP/DNS en `03-dhcp-dns.md`.

## 1. Resumen del diseño

El laboratorio usa **un único segmento interno** (`192.168.100.0/24`) donde viven el controlador de dominio y el cliente. El FortiGate es el default gateway de ese segmento y la única salida hacia afuera: Internet.

Resumen de decisiones:

- Un solo segmento interno, sin separar servidor y cliente en redes distintas.
- El FortiGate controla únicamente el tráfico que sale del segmento.
- El DHCP y el DNS los presta el controlador de dominio, no el FortiGate.

## 2. Diagrama

<p align="center"> <img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/Diagrama_topologia.png"600"> </p>

> Las direcciones de la LAN interna son el diseño previsto. Se aplican y verifican en las fases siguientes.

## 3. Segmentos

|Segmento|Tipo en VMware|Subred|Quién la usa|
|---|---|---|---|
|WAN (externa)|Bridged (`VMnet0`, Automatic)|La de la LAN del GL.iNet|FortiGate (port1)|
|LAN interna|Custom `VMnet10`, `ad-enterprise-lab`|`192.168.100.0/24`|FortiGate (port2), Windows Server y Windows 10|

## 4. Direccionamiento (planificado)

|Dispositivo|Interfaz|Dirección|Asignación|
|---|---|---|---|
|FortiGate-VM|port1 (WAN)|La que entregue el GL.iNet|DHCP del GL.iNet|
|FortiGate-VM|port2 (LAN)|`192.168.100.1/24`|Estática, default gateway de la LAN|
|Windows Server|Adaptador único|`192.168.100.100/24`|Estática|
|Windows 10 Pro|Adaptador único|`192.168.100.50` a `.60` (ámbito)|DHCP del Windows Server|

**Ámbito DHCP previsto:** `192.168.100.50` a `192.168.100.60` (11 direcciones), sin solaparse con el gateway (`.1`) ni con el servidor (`.100`).

**Gateway previsto para los equipos de la LAN:** `192.168.100.1` (FortiGate).

## 5. Configuración de red en VMware

### 5.1 Red virtual `ad-enterprise-lab` (VMnet10)

La red se crea desde el **Virtual Network Editor**: **Add Network**, se elige `VMnet10`, se define la subred y se renombra.

<p align="center"> <img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/1.png" width="600"> </p> 

<p align="center"> <img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/2.png" width="600"> </p>

Parámetros resultantes:

|Parámetro|Valor|
|---|---|
|Red virtual|`VMnet10`|
|Nombre|`ad-enterprise-lab`|
|Subred|`192.168.100.0`|
|Máscara|`255.255.255.0`|
|Tipo seleccionado en el editor|Host-only|
|Servicio DHCP local de VMware|Desactivado|
|Adaptador virtual del host conectado a la red|Desactivado|

<p align="center"> <img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/3.png" width="600"> </p>

El servicio DHCP de VMware se desactiva para que el único DHCP de la red sea el del controlador de dominio.

Por último, se renombra la red:

<p align="center"> <img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/4.png" width="600"> </p>

### 5.2 Adaptadores de red por máquina

|Máquina|Adaptador|Modo en VMware|Red|Función prevista|
|---|---|---|---|---|
|FortiGate-VM|Network Adapter|Bridged (Automatic)|LAN física (GL.iNet)|WAN (port1)|
|FortiGate-VM|Network Adapter 2|Custom|`ad-enterprise-lab`|LAN (port2)|
|Windows Server|Network Adapter|Custom|`ad-enterprise-lab`|Red interna|
|Windows 10 Pro|Network Adapter|Custom|`ad-enterprise-lab`|Red interna|

**FortiGate-VM, adaptador 1 (WAN):** en modo Bridged con puenteo automático.

<p align="center"> <img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/7.png" width="600"> </p>

**FortiGate-VM, adaptador 2 (LAN):**

<p align="center"> <img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/6.png" width="600"> </p>

**Windows Server 2022:**

<p align="center"> <img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/5.png" width="600"> </p>

**Windows 10:**

<p align="center"> <img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/8.png" width="600"> </p>

> La plantilla del FortiGate trae 10 adaptadores por defecto. En este laboratorio ya se habían eliminado en un proyecto anterior, y solo se usan los dos anteriores. La correspondencia adaptador ↔ puerto (adaptador 1 = port1, adaptador 2 = port2) se verifica con la vista de interfaces del FortiGate en `04-reglas-fortigate.md`.

## 6. Rol de cada máquina

|Máquina|Rol en la topología|
|---|---|
|FortiGate-VM|Default gateway de la LAN interna y único punto de salida hacia Internet.|
|Windows Server|Controlador de dominio. Presta Active Directory, DNS y DHCP a la LAN interna.|
|Windows 10 Pro|Cliente que obtiene su IP por DHCP y se une al dominio.|

## 7. Decisiones de diseño

### 7.1 Un solo segmento interno

El servidor y el cliente comparten el segmento `192.168.100.0/24`. Esto implica que el tráfico entre ambos (DHCP, DNS, unión al dominio, autenticación y GPOs) circula por el switch virtual de VMware y **no pasa por el FortiGate**. El default gateway solo se usa para destinos fuera del segmento.

Motivos de la decisión:

- El foco del proyecto es Active Directory, DHCP/DNS, GPOs y auditoría, que funcionan de forma natural con servidor y cliente en la misma red.
- La segmentación entre dos redes con el FortiGate en medio ya se demostró en proyectos anteriores (ver [fortigate-ngfw-lab](https://github.com/fybersec/fortigate-ngfw-lab)).
- Con servidor y cliente en la misma red no se necesita DHCP relay.
- El FortiGate sigue teniendo función real: controla todo lo que sale hacia Internet.
## 8. Estado y alcance de esta fase

|Elemento|Estado|
|---|---|
|Red virtual `ad-enterprise-lab` creada|Hecho|
|Adaptadores de red asignados a las tres VMs|Hecho|
|IPs estáticas (FortiGate LAN, Windows Server)|Pendiente|
|Unión del Windows 10 al dominio|Pendiente|
|Conectividad entre servidor y cliente|**No verificada**|
|Conectividad hacia Internet|**No verificada**|

Esta fase solo define la segmentación. Que las máquinas estén en la misma red virtual no implica que tengan conectividad entre sí: aún no hay direcciones configuradas.

## 9. Fuentes

Las fuentes de esta fase se registran en la sección de referencias del `README.md`:

- Documentación de VMware: Virtual Network Editor (redes personalizadas, DHCP y adaptador del host). [enlace](https://medium.com/@oguzhan_ozdemir8/how-to-use-the-virtual-network-editor-in-vmware-workstation-e958f40465cd)
- Documentación de Fortinet: interfaces del FortiGate. [enlace](https://docs.fortinet.com/document/fortigate/8.0.1/administration-guide/574723/interface-settings#media-type-for-interfaces-that-support-transceiver-modules)
