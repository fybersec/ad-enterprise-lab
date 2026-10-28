# 05 - Unión del cliente al dominio

> **Fase:** 5 (Unión del cliente al dominio) **Alcance:** el Windows 10 recibe su configuración IP por DHCP, se une al dominio `adlab.lan`, inicia sesión con un usuario del dominio y se comprueba su salida a Internet a través del FortiGate. Incluye el incidente de credenciales durante la unión. **No incluye:** las GPOs que se aplican al cliente (`07-gpos.md`) ni las reglas del firewall (`06-reglas-fortigate.md`). **Estado:** el cliente está unido al dominio y la sesión con `pmartinez` funciona. La verificación del DNS, los arrendamientos DHCP y la ubicación de la cuenta de equipo en Active Directory quedan pendientes.

## 1. Resultado de esta fase

| Parámetro                     | Valor                                                                 |
| ----------------------------- | --------------------------------------------------------------------- |
| Equipo                        | Windows 10 Pro 22H2 (2 GB de RAM, 2 vCPU)                             |
| Nombre del equipo             | `DESKTOP-AH74NEN` (nombre por defecto, sin renombrar)                 |
| Nombre completo tras la unión | `DESKTOP-AH74NEN.adlab.lan`                                           |
| IP recibida por DHCP          | `192.168.100.50/24`                                                   |
| Puerta de enlace              | `192.168.100.1` (FortiGate, port2)                                    |
| Sufijo DNS de la conexión     | `adlab.lan`                                                           |
| Dominio                       | `adlab.lan` (NetBIOS `ADLAB`)                                         |
| Cuenta usada para la unión    | Administrador integrado del dominio (nombre en español)               |
| Usuario de prueba             | `pmartinez` (Pablo Martinez)                                          |

## 2. Estado del cliente antes de la unión

Antes de unir el equipo se revisó la configuración de red con la cuenta local `Mario`. El cliente ya había tomado una IP del ámbito `AdlabIPv4`: `192.168.100.50`, la primera dirección del intervalo, con la puerta de enlace `192.168.100.1` y el sufijo DNS `adlab.lan` entregado por las opciones del DHCP.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/1.png" width="700"></p>


En **Configuración → Sistema → Acerca de** se ven el nombre del dispositivo y los recursos de la VM.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/2.png" width="700"></p>

|Dato|Valor|
|---|---|
|Nombre del dispositivo|`DESKTOP-AH74NEN`|
|Procesador|12th Gen Intel Core i7 (2 procesadores asignados a la VM)|
|RAM instalada|2.00 GB|
|Tipo de sistema|64 bits, basado en x64|

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)

## 3. Unión al dominio

### 3.1 Acceso al cuadro de unión

Desde **Acerca de** se entra a **Configuración avanzada del sistema**. En la pestaña **Nombre de equipo** el equipo aparece todavía en el grupo de trabajo `WORKGROUP`. El botón **Cambiar** abre el cuadro de unión.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/3.png" width="700"></p>

### 3.2 Dominio de destino

En **Miembro del** se selecciona **Dominio** y se escribe `adlab.lan`. El nombre del equipo no se modifica.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/4.png" width="700"></p>

### 3.3 Credenciales

Windows pide el nombre y la contraseña de una cuenta con permiso para unir equipos al dominio. Se usa la cuenta administradora integrada del dominio, escrita con su nombre en español (en la captura, `administrador`; con el dominio por delante sería `ADLAB\Administrador`). La contraseña no se documenta.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/5.png" width="700"></p>

### 3.4 Resultado de la unión

Windows confirma que el equipo se unió correctamente al dominio `adlab.lan`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/6.png" width="700"></p>

En las propiedades del sistema el equipo ya figura con el nombre completo `DESKTOP-AH74NEN.adlab.lan` y el dominio `adlab.lan`. Windows avisa de que los cambios tendrán efecto después de reiniciar.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/7.png" width="700"></p>

### 3.5 Reinicio

Se elige **Reiniciar ahora**.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/8.png" width="700"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/9.png" width="700"></p>



**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)

## 4. Incidente: error de credenciales al unir el equipo

| Elemento    | Detalle                                                                                                                                                                       |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Síntoma     | Al intentar la unión a `adlab.lan`, Windows rechazó las credenciales con un mensaje de credenciales incorrectas.                                                              |
| Diagnóstico | La conectividad con el servidor y el DNS eran correctos, así que el fallo no era de red. El problema estaba en el nombre de usuario: se había escrito `Administrator`.        |
| Solución    | Usar el nombre de la cuenta integrada tal como existe en el dominio, en español: `Administrador` (forma completa `ADLAB\Administrador`).                                      |
| Lección     | El nombre de las cuentas integradas depende del idioma con el que se instaló el sistema. En este dominio, instalado en español, el administrador no se llama `Administrator`. |

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)

## 5. Inicio de sesión con un usuario del dominio

Tras el reinicio, la pantalla de inicio de sesión ofrece **Otro usuario** e indica **Iniciar sesión en ADLAB**, es decir, el dominio al que se unió el equipo. Abajo a la izquierda sigue disponible la cuenta local `Mario`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/10.png" width="700"></p>

Se inicia sesión con el usuario `pmartinez`, creado en la OU `Clientes` en `03-active-directory.md`. La contraseña no se documenta.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/11.png" width="700"></p>

El inicio de sesión es exitoso y el menú Inicio muestra el nombre del usuario tal como está definido en Active Directory: **Pablo Martinez**.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/12.png" width="700"></p>

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)

## 6. Verificación de conectividad

Con la sesión de `pmartinez`, se hace `ping` a `8.8.8.8`: 4 paquetes enviados, 4 recibidos, 0 % perdidos, tiempo medio de 42 ms.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/05-union-cliente-dominio/13.png" width="700"></p>

Este resultado confirma que el tráfico del cliente sale a Internet por el FortiGate: el cliente usa `192.168.100.1` como puerta de enlace y la política `INTtoEXT` (port2 → port1, con NAT) lo permite. **No confirma la resolución de nombres**, porque el ping se hizo a una IP y no a un nombre.

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)
## 7. Cuenta de equipo en Active Directory

Al unirse, el equipo genera una cuenta de equipo en el dominio. Por defecto se crea en el contenedor `Computers`, y las GPOs se vinculan a OUs, no a ese contenedor.

Esto tiene una consecuencia para `07-gpos.md`:

|Tipo de GPO|De qué OU depende|
|---|---|
|De equipo|OU donde esté la cuenta de equipo|
|De usuario|OU donde esté la cuenta del usuario (`pmartinez` está en `Clientes`)|

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)

## 8. Estado y alcance de esta fase

| Elemento                                                         | Estado                                                  |
| ---------------------------------------------------------------- | ------------------------------------------------------- |
| Cliente con IP, puerta de enlace y sufijo por DHCP               | Hecho                                                   |
| Unión a `adlab.lan` y reinicio                                   | Hecho                                                   |
| Incidente de credenciales                                        | Documentado; falta captura del error y respaldo oficial |
| Inicio de sesión con `pmartinez`                                 | Hecho                                                   |
| Salida a Internet por IP                                         | Hecho                                                   |
| Pings al servidor y al FortiGate                                 | Pendiente                                               |
| Cuenta de equipo en OU definitiva                                | Pendiente (decisión y captura)                          |
| Renombrar el equipo                                              | Sin decidir (conserva el nombre por defecto)            |

**Siguiente:** `06-reglas-fortigate.md`, donde se revisa y se restringe la política temporal `INTtoEXT` y se definen los controles hacia Internet.
