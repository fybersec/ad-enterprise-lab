# 06 - Reglas del FortiGate

> **Fase:** 6 (Reglas del FortiGate) **Alcance:** política de salida `INTtoEXT` con perfiles de antivirus e IPS, política DoS para proteger la LAN interna frente a tráfico originado en la red externa, e intento de filtrado web con su limitación de licencia. **No incluye:** las GPOs (`07-gpos.md`). **Estado:** `INTtoEXT` queda con antivirus, IPS y sin inspección SSL; la política DoS queda creada con registro activado. El filtrado web no se pudo lograr. Las pruebas de detección están pendientes.

## 1. Resultado de esta fase

| Elemento               | Resultado                                                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Objetivo principal     | Filtrado web (web filtering) del tráfico del cliente hacia Internet                                                      |
| Resultado del objetivo | **No logrado**: el filtrado por categorías depende del servicio FortiGuard y el equipo solo tiene licencia de evaluación |
| Política de salida     | `INTtoEXT` (port2 → port1) con antivirus `default`, IPS `high_security` y SSL `no-inspection`                            |
| Modo de inspección     | Proxy-based                                                                                                              |
| Política DoS           | `INTLAN` (port1 → `LAN`), habilitada, con registro activado en las anomalías                                             |

## 2. Papel del FortiGate en el laboratorio

El foco de este laboratorio es Active Directory y la gestión de usuarios. El FortiGate completa la topología: separa el segmento interno de Internet y controla lo que sale. El detalle de su configuración y de sus perfiles de seguridad se documenta en el proyecto previo `fortigate-ngfw-lab`, por lo que este documento recoge solo lo necesario para el laboratorio.

El reparto de controles del laboratorio es:

|Control|Qué cubre|
|---|---|
|FortiGate|Tráfico que sale del segmento y tráfico que entra desde el exterior|
|GPOs|Restricciones en el equipo y en el usuario (`07-gpos.md`)|

El tráfico entre el servidor y el cliente va por el switch virtual y no pasa por el FortiGate.

## 3. Punto de partida

La política mínima de `02-servidor-base.md` (`INTtoEXT`, port2 → port1, origen `LAN`, destino `all`, servicio `ALL`, NAT) era temporal y amplia. Para las pruebas de esta fase se creó además una política auxiliar `EXTtoINT` (port1 → port2, origen `all`, destino `LAN`) con perfiles de seguridad, para aplicarlos también en sentido inverso.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/1.png" width="800"></p>

La captura muestra que `INTtoEXT` acumulaba 2.99 GB y `EXTtoINT`, 0 B. El tráfico de retorno de las sesiones del cliente no pasaba por `EXTtoINT`, de modo que la política no intervenía y se eliminó por innecesaria.

## 4. Objetivo no logrado: filtrado web

### 4.1 Qué se quería hacer

El objetivo era aplicar un perfil de filtrado web a `INTtoEXT` para restringir las categorías de sitios accesibles desde el cliente. El perfil `default` ya trae bloqueos por categoría (en la captura se ven, por ejemplo, contenido adulto, armas, alcohol y tabaco).

### 4.2 Limitación por licencia

Al editar el perfil, FortiOS advierte que el dispositivo no tiene licencia para el servicio de filtrado web de FortiGuard y que el tráfico puede bloquearse si la opción está activada.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/2.png" width="800"></p>

La pantalla de suscripciones confirma que solo la máquina virtual tiene licencia, en modo de evaluación, y que no figura ningún servicio de FortiGuard.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/3.png" width="800"></p>

El filtrado por categorías necesita que el FortiGate consulte a FortiGuard para clasificar cada sitio. Según un artículo técnico de Fortinet Community, cuando falta la licencia y la función está activa, FortiGuard devuelve un error de clasificación y el tráfico se bloquea por defecto.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/4.png" width="800"></p>

## 5. Política de salida `INTtoEXT`

### 5.1 Parámetros finales

|Parámetro|Valor|
|---|---|
|Nombre|`INTtoEXT`|
|Interfaces|port2 (LAN) → port1 (WAN)|
|Origen|`LAN` (`192.168.100.0/24`)|
|Destino|`all`|
|Servicio|`ALL`|
|Programación|`always`|
|Acción|`ACCEPT`|
|NAT|Activado, con la dirección de la interfaz de salida|
|Modo de inspección|Proxy-based|
|Registro del tráfico permitido|Activado, solo eventos de seguridad|

El modo de inspección cambió de flow-based a proxy-based. Los documentos anteriores usaban flow-based porque la política mínima no aplicaba perfiles; al aplicar antivirus e IPS se pasó a proxy-based.

Estado de la política antes de aplicar los perfiles:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/5.png" width="800"></p>

Edición de la política con los perfiles aplicados:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/6.png" width="800"></p>

Estado final:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/7.png" width="800"></p>

### 5.2 Perfiles de seguridad

Se usan perfiles predefinidos de FortiOS, sin crear perfiles propios.

|Perfil|Estado|
|---|---|
|Antivirus|Activado, `default`|
|Filtro web|Desactivado|
|Filtro de vídeo|Desactivado|
|Filtro DNS|Desactivado|
|Control de aplicaciones|Desactivado|
|IPS|Activado, `high_security`|
|Filtro de archivos|Desactivado|
|Inspección SSL|`no-inspection`|

**Antivirus (`default`).** Escanea archivos y bloquea virus. En el perfil, el modo es Block, el conjunto de funciones es proxy-based y los protocolos inspeccionados son HTTP, SMTP, POP3, IMAP y FTP. CIFS, MAPI, SSH y WebSocket están desactivados. Está activa la opción de tratar como virus los ejecutables de Windows en adjuntos de correo y el envío a FortiSandbox está desactivado.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/8.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/9.png" width="800"></p>

**IPS (`high_security`).** Perfil predefinido que ya existía en el equipo. Según su comentario en la interfaz, bloquea las vulnerabilidades de severidad crítica, alta y media, y algunas de severidad baja.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/10.png" width="800"></p>
## 6. Política DoS

Una política DoS se configura aparte de las políticas de firewall: define umbrales para ciertos tipos de tráfico anómalo y registra los excesos. Aquí protege la LAN interna frente a un posible ataque originado en la red externa, por lo que evalúa el tráfico que entra por port1 con destino `LAN`. Se abre desde **Policy & Objects → DoS Policy**, que estaba vacía.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/11.png" width="800"></p>

### 6.1 Parámetros

|Parámetro|Valor|
|---|---|
|Nombre|`INTLAN`|
|Interfaz de entrada|port1|
|Origen|`all`|
|Destino|`LAN`|
|Servicio|`ALL`|
|Política habilitada|Sí|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/06%20-%20fortigate/12.png" width="800"></p>

### 6.2 Anomalías y umbrales

| Capa | Anomalía           | Umbral |
| ---- | ------------------ | ------ |
| L3   | `ip_src_session`   | 3000   |
| L3   | `ip_dst_session`   | 3000   |
| L4   | `tcp_syn_flood`    | 1000   |
| L4   | `tcp_port_scan`    | 500    |
| L4   | `tcp_src_session`  | 3000   |
| L4   | `tcp_dst_session`  | 3000   |
| L4   | `udp_flood`        | 1000   |
| L4   | `udp_scan`         | 1000   |
| L4   | `udp_src_session`  | 3000   |
| L4   | `udp_dst_session`  | 3000   |
| L4   | `icmp_flood`       | 250    |
| L4   | `icmp_sweep`       | 100    |
| L4   | `icmp_src_session` | 300    |
| L4   | `icmp_dst_session` | 1000   |
| L4   | `sctp_flood`       | 1000   |
| L4   | `sctp_scan`        | 1000   |
| L4   | `sctp_src_session` | 3000   |
| L4   | `sctp_dst_session` | 3000   |

## 9. Estado y alcance de esta fase

| Elemento                                | Estado                   |
| --------------------------------------- | ------------------------ |
| Filtrado web                            | No logrado (licencia)    |
| Política `INTtoEXT` con antivirus e IPS | Hecho                    |
| Política auxiliar `EXTtoINT`            | Eliminada (sin tráfico)  |
| Política DoS `INTLAN`                   | Creada; acción pendiente |

**Siguiente:** `07-gpos.md`, donde se aplican restricciones al usuario en el equipo (CMD, panel de control y otras herramientas).
