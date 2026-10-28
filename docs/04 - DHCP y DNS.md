# 04 - DHCP y DNS

> **Fase:** 4 (DHCP y DNS) **Alcance:** configuración del DNS del dominio (zona de búsqueda inversa y registros del servidor) y del ámbito DHCP en el controlador de dominio. **No incluye:** el Windows 10. Que el cliente obtenga su IP, resuelva el dominio y se una a él se documenta en `05-union-cliente-dominio.md`. **Estado:** el servidor queda con el DNS y el ámbito DHCP configurados. El cliente todavía no participa.

## 1. Resultado de esta fase

| Parámetro                         | Valor                                                               |
| --------------------------------- | ------------------------------------------------------------------- |
| Zona directa                      | `adlab.lan` (creada con la promoción a DC)                          |
| Zona inversa                      | `100.168.192.in-addr.arpa` (red `192.168.100.0`)                    |
| Almacenamiento de la zona inversa | Integrada en Active Directory                                       |
| Replicación                       | Servidores DNS en controladores de dominio del dominio `adlab.lan`  |
| Actualizaciones dinámicas         | Solo seguras                                                        |
| Ámbito DHCP                       | `AdlabIPv4`                                                         |
| Intervalo                         | `192.168.100.50` a `192.168.100.60` (`/24`)                         |
| Exclusiones                       | Ninguna                                                             |
| Duración de la concesión          | 8 días                                                              |
| Opciones que entrega el DHCP      | Gateway `192.168.100.1`, DNS `192.168.100.100`, dominio `adlab.lan` |

## 2. DNS

La consola se abre desde **Herramientas → DNS** en el Administrador del servidor.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/1.png" width="800"></p>

### 2.1 Zona de búsqueda directa

La zona `adlab.lan` ya existe: se creó automáticamente al promover el servidor a controlador de dominio, junto con `_msdcs.adlab.lan`. En la captura se ven sus carpetas `_msdcs`, `_sites`, `_tcp`, `_udp`, `DomainDnsZones` y `ForestDnsZones`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/2.png" width="800"></p>

### 2.2 Zona de búsqueda inversa

Una zona inversa traduce direcciones IP en nombres DNS. Se crea desde **Zonas de búsqueda inversa → Zona nueva**, con estos pasos del asistente:

|Paso|Valor|
|---|---|
|Tipo de zona|Zona principal, con la zona almacenada en Active Directory|
|Ámbito de replicación|Todos los servidores DNS en controladores de dominio del dominio `adlab.lan`|
|Tipo de zona inversa|IPv4|
|Id. de red|`192.168.100` (nombre de la zona: `100.168.192.in-addr.arpa`)|
|Actualización dinámica|Permitir solo actualizaciones dinámicas seguras|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/3.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/4.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/5.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/6.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/.7png" width="800"></p>


### 2.3 Servidor de nombres de la zona inversa

En las propiedades de la zona inversa, pestaña **Servidores de nombres**, el servidor `srv-ad01.adlab.lan.` aparece con la IP "Desconocido". Se edita el registro y se pulsa **Resolver**: la IP `192.168.100.100` queda validada.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/8.png" width="800"></p>

### 2.4 Registro de host del servidor

En la zona `adlab.lan` figura el registro `Host (A)` de `srv-ad01` apuntando a `192.168.100.100`, de tipo estático. En sus propiedades está marcada la opción **Actualizar registro del puntero (PTR) asociado**, para que la zona inversa también conozca al servidor.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/9.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/10.png" width="800"></p>

### 2.5 Pendiente del DNS

- **DNS del propio servidor:** sigue con el valor provisional de `02-servidor-base.md`.
- **Reenviadores (forwarders):** el DC debe poder resolver nombres de Internet para los clientes. `

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)

## 3. DHCP

La consola se abre desde **Herramientas → DHCP**. El servidor ya fue autorizado en Active Directory en `03-active-directory.md`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/11.png" width="800"></p>

Sobre **IPv4** se elige **Ámbito nuevo**:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/12.png" width="800"></p>

### 3.1 Nombre del ámbito

Nombre `AdlabIPv4`, sin descripción.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/13.png" width="800"></p>

### 3.2 Intervalo de direcciones

|Campo|Valor|
|---|---|
|Dirección IP inicial|`192.168.100.50`|
|Dirección IP final|`192.168.100.60`|
|Longitud|`24`|
|Máscara de subred|`255.255.255.0`|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/14.png" width="800"></p>

### 3.3 Exclusiones

No se agregan exclusiones y el retraso queda en 0 ms. El intervalo no incluye el gateway (`.1`) ni el servidor (`.100`), así que no hace falta excluirlos.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/15.png" width="800"></p>

### 3.4 Duración de la concesión

8 días.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/16.png" width="800"></p>

### 3.5 Opciones DHCP

Se elige **Configurar estas opciones ahora**:

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/04%20-%20dhcp-dns/18.png" width="800"></p>

|Opción|Valor|Quién es|
|---|---|---|
|Enrutador (gateway)|`192.168.100.1`|FortiGate (port2)|
|Dominio primario|`adlab.lan`|Dominio de Active Directory|
|Servidor DNS|`192.168.100.100`|El propio controlador de dominio|

## 4. Estado y alcance de esta fase

| Elemento                                    | Estado                                    |
| ------------------------------------------- | ----------------------------------------- |
| Zona directa `adlab.lan`                    | Existente (creada con la promoción)       |
| Zona inversa `100.168.192.in-addr.arpa`     | Hecho                                     |
| Ámbito DHCP `AdlabIPv4` con sus opciones    | Hecho                                     |
| Reservas DHCP                               | No documentadas                           |
| DNS del servidor (configuración definitiva) | Pendiente                                 |
| Reenviadores DNS                            | Pendiente                                 |
| Verificación en el servidor                 | Pendiente                                 |
| Windows 10 con IP por DHCP y en el dominio  | Pendiente (`05-union-cliente-dominio.md`) |

**Siguiente:** `05-union-cliente-dominio.md`, donde el Windows 10 obtiene su IP del DHCP, resuelve el dominio y se une a él.
