# 02 - Servidor base

> **Fase:** 2 (Servidor base) **Alcance:** preparación del Windows Server 2022 (nombre del equipo, zona horaria, IP estática y DNS), conectividad mínima hacia Internet a través del FortiGate e instalación de los roles. **No se configura ningún rol**: Active Directory se configura en `03-active-directory.md` y DHCP/DNS en `04-dhcp-dns.md`. **Estado:** el servidor queda preparado, con salida a Internet y con los roles instalados. El Windows 10 no participa en esta fase.

## 1. Resultado de esta fase

|Parámetro|Valor|
|---|---|
|Nombre del equipo (NetBIOS)|`SRV-AD01`|
|Nombre completo (FQDN)|`SRV-AD01.lan`|
|Descripción del equipo|`ad-enterprise-lab`|
|Sufijo DNS principal|`lan`|
|IP estática|`192.168.100.100/24`|
|Puerta de enlace|`192.168.100.1` (FortiGate, port2)|
|DNS (provisional)|Preferido `8.8.8.8`, alternativo `192.168.100.100`|
|Roles instalados|AD DS, DNS, DHCP, Servidor web (IIS) con ASP|

## 2. Nombre del equipo y sufijo DNS

El servidor se instaló con un nombre generado por Windows (`WIN-Q4017AOOD6E`) y pertenece a un grupo de trabajo.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/1.png" width="600"></p>

Desde **Servidor local → Nombre de equipo → Cambiar** se establecen tres valores:

- **Descripción del equipo:** `ad-enterprise-lab`.
- **Nombre del equipo:** `SRV-AD01`.
- **Sufijo DNS principal** (botón **Más...**): `lan`. Se escribe sin punto inicial, porque el campo rechaza `.lan`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/2.png" width="600"></p>

Windows avisa de que hay que reiniciar para aplicar los cambios.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/3.png" width="600"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/4.png" width="600"></p>

Tras el reinicio, el nombre queda aplicado:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/5.png" width="600"></p>

> El sufijo `lan` es el del equipo antes de promoverlo. El nombre definitivo del dominio se define en `03-active-directory.md`. En la captura aparece marcada la opción de cambiar el sufijo DNS principal cuando cambie la pertenencia al dominio. 

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=PXpupA3ap9M&t=41s)

## 3. Zona horaria

Desde el reloj de la barra de tareas se abre **Cambiar fecha y hora → Cambiar zona horaria** y se selecciona la que corresponde a la región de quien realiza el laboratorio.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/6.png" width="600"></p>

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=BQxZ4Bpwfro&t=2s)

## 4. IP estática y DNS

Desde **Conexiones de red → Ethernet0 → Propiedades → Protocolo de Internet versión 4 (TCP/IPv4)** se configura la IP estática:

|Campo|Valor|
|---|---|
|Dirección IP|`192.168.100.100`|
|Máscara de subred|`255.255.255.0`|
|Puerta de enlace|`192.168.100.1`|
|Servidor DNS preferido|`8.8.8.8`|
|Servidor DNS alternativo|`192.168.100.100`|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/7.png" width="600"></p>

Después se comprueba desde **Todos los servidores** que el administrador del servidor detecta la IP configurada:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/8.png" width="600"></p>

### DNS: configuración provisional

El DNS se deja así por ahora para poder resolver nombres de Internet mientras no existe el DNS del dominio. La configuración definitiva se decidirá más adelante. La opción que se evalúa es dejar como único DNS la IP del propio servidor, para tener control de la resolución de nombres, y que el DHCP entregue esa misma IP a los clientes, para que consulten primero al DC y luego salgan a Internet a través del FortiGate.

## 5. Conectividad hacia el exterior

> **Esta sección no es la configuración del FortiGate como tal.** Se documenta aquí únicamente el mínimo necesario para que el servidor tenga salida a Internet y se pueda continuar. La configuración del firewall (interfaces, políticas y NAT) se documenta en `05-reglas-fortigate.md`. Los pasos de esta sección son los mismos que ya se realizaron en proyectos anteriores (ver [fortigate-ngfw-lab](https://github.com/fybersec/fortigate-ngfw-lab)).

La red `ad-enterprise-lab` es una red virtual aislada y no tiene salida propia. Toda salida hacia Internet depende del FortiGate, que es la puerta de enlace del servidor. Por eso su interfaz interna se configura antes de seguir.

### 5.1 IP de la interfaz interna (port2)

`port1` (WAN) ya toma su IP por DHCP del router GL.iNet, así que no se modifica. Por consola (CLI) se asigna la IP estática de `port2`:

```
config system interface
    edit port2
        set mode static
        set ip 192.168.100.1 255.255.255.0
    next
end
```

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/9.png" width="600"></p>

Resultado en la vista de interfaces: `port1` con la IP del GL.iNet y `port2` con `192.168.100.1/24`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/10.png" width="600"></p>

Esto confirma que el adaptador en Bridged corresponde a `port1` y el adaptador de `ad-enterprise-lab` a `port2`, como se planteó en `01-topologia.md`.

### 5.2 Acceso administrativo en port2

La interfaz `port2` no tenía ningún servicio de acceso administrativo habilitado. Se habilitaron HTTPS, HTTP, SSH y PING para poder comunicarse con el firewall desde la red interna.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/11.png" width="600"></p>

> Decisión de laboratorio. En un entorno real se limitarían estos servicios a los estrictamente necesarios, y HTTP no cifra el tráfico. 

### 5.3 Prueba servidor → FortiGate

Con `port2` configurada, el servidor hace ping al FortiGate y responde:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/12.png" width="600"></p>

Esto confirma que la IP, la máscara, la puerta de enlace y la asignación de adaptadores son correctas.

**Observación: el FortiGate no puede hacer ping al servidor.** En el proyecto anterior (FortiGate con Debian) el ping desde el firewall hacia el equipo interno sí respondía, pero aquí no. La causa identificada es el firewall de Microsoft Defender del propio Windows Server, que por defecto no acepta solicitudes de eco ICMP entrantes. Por eso la prueba válida de conectividad en este laboratorio es siempre del servidor hacia el FortiGate.

### 5.4 Salida a Internet

Con la IP y la puerta de enlace correctas, el servidor todavía no sale a Internet. El ping a `8.8.8.8` agota el tiempo de espera:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/13.png" width="600"></p>

Falta una política en el FortiGate que permita ese tráfico. Se crea una política básica de salida:

|Parámetro|Valor|
|---|---|
|Nombre|`INTtoEXT`|
|Interfaz de entrada|`port2` (LAN interna)|
|Interfaz de salida|`port1` (WAN)|
|Origen|`LAN` (objeto de dirección con la subred `192.168.100.0/24`)|
|Destino|`all`|
|Servicio|`ALL`|
|Programación|`always`|
|Acción|`ACCEPT`|
|Modo de inspección|Flow-based|
|NAT|Activado, usando la dirección de la interfaz de salida|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/14.png" width="600"></p>

El objeto `LAN` se creó previamente como un objeto de dirección con la subred `192.168.100.0/24`.

> Es una política deliberadamente amplia, solo para dar conectividad en esta fase. Se revisa y se restringe en `05-reglas-fortigate.md`, junto con los perfiles de seguridad.

Con la política aplicada, el ping a `8.8.8.8` responde:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/15.png" width="600"></p>

> Solo se verificó la salida hacia una IP pública. La resolución de nombres no se ha probado todavía, porque depende de la configuración definitiva del DNS.

## 6. Instalación de roles y características

La instalación sigue el flujo básico del **Administrador del servidor**, ya utilizado en proyectos anteriores. Solo se instalan los roles, sin configurarlos.

Desde el **Panel** se abre **Agregar roles y características**:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/16.png" width="600"></p>

Tipo de instalación: **basada en características o en roles**.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/17.png" width="600"></p>

Roles seleccionados:

|Rol|Para qué se instala|
|---|---|
|Servicios de dominio de Active Directory|Controlador de dominio|
|Servidor DNS|Resolución de nombres del dominio|
|Servidor DHCP|Asignación de IPs al cliente|
|Servidor web (IIS)|Servicio web de prueba (ver sección 7)|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/18.png" width="600"></p>

En las páginas de características y de los roles DNS, DHCP e IIS no se seleccionó nada adicional.

**Servicios de rol de IIS:** se dejan los valores por defecto y se añade **ASP** (en Desarrollo de aplicaciones). **Registro HTTP** queda marcado, ya que de ahí salen los logs del servidor web. Las **Extensiones ISAPI** aparecen marcadas junto con ASP. 

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/19.png" width="600"></p>

En la confirmación, el asistente añade por sí solo las herramientas de administración de cada rol (administración de directivas de grupo, herramientas de AD DS, DHCP, DNS e IIS). Se deja marcada la opción de reiniciar automáticamente si es necesario; finalmente no fue necesario reiniciar.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/20.png" width="600"></p>

Durante la instalación:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/21.png" width="600"></p>

Al terminar, el Panel del administrador del servidor muestra los grupos de roles instalados:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/02%20-%20servidor-base/22.png" width="600"></p>

> En la parte superior del administrador aparece una bandera con un icono amarillo. Indica que los roles recién instalados necesitan configuración posterior para funcionar. Esa configuración se hace en los siguientes documentos: la promoción a controlador de dominio en `03-active-directory.md` y el DHCP y el DNS en `04-dhcp-dns.md`.

## 7. Decisión de diseño: IIS y ASP en el controlador de dominio

IIS con ASP se instala en el mismo servidor que será el controlador de dominio. Es una decisión deliberada de laboratorio:

- **Motivo:** crear una superficie de ataque controlada, que permita generar actividad en la auditoría. Los accesos y errores del sitio web quedan registrados en los logs de IIS (Registro HTTP).
- **Riesgo asumido:** un servidor web en un controlador de dominio amplía la superficie de ataque de la máquina más crítica del dominio. En un entorno real no se haría. 
- **Dónde se detecta:** el tráfico entre equipos del mismo segmento no pasa por el FortiGate, por lo que esta actividad se analiza en la auditoría y no en el firewall.

## 8. Estado y alcance de esta fase

|Elemento|Estado|
|---|---|
|Nombre del equipo y sufijo DNS|Hecho|
|Zona horaria|Hecho|
|IP estática del servidor|Hecho|
|DNS|Provisional (decisión final pendiente)|
|IP de la interfaz interna del FortiGate (port2)|Hecho|
|Política mínima de salida en el FortiGate|Hecho (se revisa en `05-reglas-fortigate.md`)|
|Salida del servidor a Internet (IP pública)|Verificada|
|Resolución de nombres|No verificada|
|Roles instalados|Hecho (sin configurar)|
|Sitio web de prueba en IIS|Pendiente|
|Windows 10|No participa en esta fase|

**Siguiente:** `03-active-directory.md`, donde se promueve el servidor a controlador de dominio y se crea la estructura del dominio.
