# ad-enterprise-lab

> Laboratorio de Active Directory empresarial sobre VMware Workstation Pro: un controlador de dominio Windows Server 2022 (AD DS, DNS, DHCP e IIS), un cliente Windows 10 Pro unido al dominio, restricciones por GPO y un firewall FortiGate que controla la salida a Internet.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/01%20-%20topologia/Diagrama_topologia.png" width="800"></p>

## 1. Resumen

El laboratorio reproduce una red pequeña de empresa con tres máquinas virtuales y un router físico. El controlador de dominio presta Active Directory, DNS y DHCP. El cliente obtiene su IP por DHCP, se une al dominio `adlab.lan` e inicia sesión con un usuario del dominio. Sobre ese usuario se aplica una GPO con restricciones, y el FortiGate gobierna lo que sale hacia Internet.

El foco del proyecto es Active Directory y la gestión de usuarios. El FortiGate completa la topología; su configuración detallada está en el proyecto previo [fortigate-ngfw-lab](https://github.com/fybersec/fortigate-ngfw-lab).

## 2. Arquitectura

### 2.1 Máquinas

|Máquina|Rol|Sistema|RAM|vCPU|Disco|
|---|---|---|---|---|---|
|FortiGate-VM|Gateway y firewall|FortiOS 8.0.1|2 GB|1|2 GB y 30 GB (SCSI)|
|SRV-AD01|Controlador de dominio (AD DS, DNS, DHCP, IIS)|Windows Server 2022 Standard Evaluation|5 GB|4|70 GB (SCSI)|
|Cliente|Estación de trabajo unida al dominio|Windows 10 Pro 22H2|2 GB|2|30 GB (SCSI)|
|Router GL.iNet|Router de la LAN física (equipo físico)|Basado en OpenWrt|n/a|n/a|n/a|

### 2.2 Red

|Segmento|Tipo en VMware|Subred|Quién la usa|
|---|---|---|---|
|WAN|Bridged (`VMnet0`, Automatic)|La de la LAN del GL.iNet|FortiGate (`port1`)|
|LAN interna|`VMnet10` (`ad-enterprise-lab`)|`192.168.100.0/24`|FortiGate (`port2`), Windows Server y Windows 10|

|Dispositivo|Dirección|Asignación|
|---|---|---|
|FortiGate (`port1`)|La que entrega el GL.iNet|DHCP del GL.iNet|
|FortiGate (`port2`)|`192.168.100.1/24`|Estática (gateway de la LAN)|
|SRV-AD01|`192.168.100.100/24`|Estática|
|Windows 10|`192.168.100.50` a `.60` (ámbito)|DHCP del controlador de dominio|

### 2.3 Decisiones de diseño

- **Un solo segmento interno.** Servidor y cliente comparten `192.168.100.0/24`, por lo que su tráfico (DHCP, DNS, unión al dominio, autenticación y GPOs) circula por el switch virtual y no pasa por el FortiGate.
- **El DHCP y el DNS los presta el controlador de dominio**, no el FortiGate.
- **Reparto de controles:**

|Control|Qué cubre|
|---|---|
|FortiGate|Tráfico que sale de la LAN interna hacia Internet|
|GPOs|Restricciones en el usuario dentro del equipo|

## 3. Alcance y limitaciones

| Área                                   | Estado                                                                                        |
| -------------------------------------- | --------------------------------------------------------------------------------------------- |
| Dominio `adlab.lan` (AD DS, DNS, DHCP) | Hecho                                                                                         |
| Cliente unido al dominio               | Hecho, con inicio de sesión del usuario `pmartinez`                                           |
| GPO `Politicas_Cliente`                | Aplicada al usuario; 5 pruebas de apertura bloqueadas (`auditoria/gpos.md`)                   |
| Política de salida del FortiGate       | Hecha, con NAT, antivirus (`default`), IPS (`high_security`) y SSL `no-inspection`            |
| Filtrado web del FortiGate             | **No logrado**: depende del servicio FortiGuard y el equipo solo tiene licencia de evaluación |
| Pruebas de detección del FortiGate     | Sin documentar (antivirus con archivo de prueba, IPS y DoS)                                   |
| Sitio web de prueba en IIS             | Hecho; accesible desde el servidor y desde el cliente                                         |

## 4. Documentación

| Documento                         | Contenido                                                                              |
| --------------------------------- | -------------------------------------------------------------------------------------- |
| 00 - Entorno                      | Versiones, recursos de las VMs, método de instalación y router físico                  |
| 01 - Topología                    | Diseño de la red, segmentación en VMware y rol de cada máquina                         |
| 02 - Servidor base                | Nombre, IP, salida a Internet por el FortiGate, roles instalados y sitio web de prueba |
| 03 - Active Directory             | Promoción a controlador de dominio, OUs, usuario y autorización del DHCP               |
| 04 - DHCP y DNS                   | Zona inversa, registros del servidor y ámbito DHCP                                     |
| 05 - Unión del cliente al dominio | IP por DHCP, unión al dominio, incidente de credenciales e inicio de sesión            |
| 06 - Reglas del FortiGate         | Política de salida con antivirus e IPS, política DoS y limitación del filtrado web     |
| 07 - GPOs                         | GPO de usuario vinculada a `Clientes`                                                  |
| Auditoría de GPOs                 | Aplicación de la GPO en el cliente y pruebas de bloqueo                                |

## 5. Estructura del repositorio

```
ad-enterprise-lab/
├── README.md
├── LICENSE
├── SECURITY.md
├── docs/
│   ├── 00-entorno.md
│   ├── 01-topologia.md
│   ├── 02-servidor-base.md
│   ├── 03-active-directory.md
│   ├── 04-dhcp-dns.md
│   ├── 05-union-cliente-dominio.md
│   ├── 06-reglas-fortigate.md
│   └── 07-gpos.md
├── auditoria/
│   └── gpos.md
└── evidencia/
```

Las capturas de cada documento están en la carpeta de `evidencia/` que lleva su tema.

## 6. Seguridad del laboratorio

Varias decisiones del laboratorio son inseguras a propósito (por ejemplo, IIS en el controlador de dominio o el acceso administrativo por HTTP en el firewall). Están listadas, con su motivo, en SECURITY.md. No son una guía para un entorno real.
## 7. Proyectos relacionados

- [fortigate-ngfw-lab](https://github.com/fybersec/fortigate-ngfw-lab): montaje y configuración detallada del FortiGate. De aquí se reutiliza la VM del firewall.
- [sqli-sqlserver-lab](https://github.com/fybersec/sqli-sqlserver-lab): montaje de la VM de Windows Server, reutilizada en este laboratorio.

## 8. Referencias

Las fuentes y los recursos complementarios de cada documento se registran aquí. Los recursos complementarios son alternativas, más o menos detalladas, para quien quiera repetir el paso; no son la única fuente de lo documentado.

### 9.1 Documentación oficial y fuentes técnicas

| Documento | Fuente                                                                                                                                                                                                              |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 00        | [Fortinet: despliegue del FortiGate-VM](https://docs.fortinet.com/document/fortigate-private-cloud/8.0.0/vmware-esxi-administration-guide/143812/deploying-the-fortigate-vm)                                        |
| 00        | [Microsoft: descarga de Windows Server 2022 (evaluación)](https://www.microsoft.com/es-es/evalcenter/download-windows-server-2022)                                                                                  |
| 00        | [Microsoft: descarga de Windows 10](https://www.microsoft.com/es-es/software-download/windows10)                                                                                                                    |
| 01        | [Fortinet: configuración de interfaces del FortiGate](https://docs.fortinet.com/document/fortigate/8.0.1/administration-guide/574723/interface-settings#media-type-for-interfaces-that-support-transceiver-modules) |

### 9.2 Recursos complementarios

|Documento|Recurso|
|---|---|
|00|[Guía de instalación de Windows Server](https://www.alitajran.com/install-windows-server/)|
|00|[Video: instalación de Windows 10](https://www.youtube.com/watch?v=_1cXyzN2jwQ)|
|01|[Guía: uso del Virtual Network Editor de VMware Workstation](https://medium.com/@oguzhan_ozdemir8/how-to-use-the-virtual-network-editor-in-vmware-workstation-e958f40465cd)|
|02|[Video: nombre del equipo y sufijo DNS](https://www.youtube.com/watch?v=PXpupA3ap9M&t=41s)|
|02|[Video: zona horaria](https://www.youtube.com/watch?v=BQxZ4Bpwfro&t=2s)|
|03, 04, 05|[Video: Active Directory, DHCP, DNS y unión al dominio](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)|

### 9.3 Material de terceros

- Plantilla web del sitio de prueba de IIS: [AHMAD-JX/Website-GameOver](https://github.com/AHMAD-JX/Website-GameOver), con licencia MIT según figura en GitHub.

## 10. Licencia

Consulta el archivo `LICENSE.` La plantilla web de terceros conserva su propia licencia.
