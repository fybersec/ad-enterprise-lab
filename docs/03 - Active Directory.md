# 03 - Active Directory

> **Fase:** 3 (Active Directory) **Alcance:** promoción del servidor a controlador de dominio con un bosque nuevo, estructura mínima del dominio (unidades organizativas y un usuario) y el paso posterior de autorización del DHCP en AD DS. **No incluye:** el ámbito DHCP, la configuración definitiva del DNS ni la unión del Windows 10 al dominio, que van en `04-dhcp-dns.md`. **Estado:** el servidor es controlador de dominio. Todas las pruebas son locales, desde el propio servidor; el cliente todavía no tiene IP.

## 1. Resultado de esta fase

|Parámetro|Valor|
|---|---|
|Operación|Nuevo bosque|
|Nombre del dominio raíz|`adlab.lan`|
|Nombre NetBIOS|`ADLAB`|
|Nivel funcional (bosque y dominio)|Windows Server 2016 (valor que muestra el asistente)|
|Funciones del controlador|Servidor DNS y catálogo global|
|Unidades organizativas|`ADLAB` y, dentro de ella, `Clientes`|
|Usuario creado|`pmartinez` (Pablo Martinez), en la OU `Clientes`|
|DHCP|Autorizado en AD DS (paso posterior a la instalación)|

## 2. Promoción del servidor a controlador de dominio

El rol de AD DS ya está instalado desde `02-servidor-base.md`. En el Administrador del servidor, la bandera con el icono amarillo indica que los roles necesitan configuración posterior. Desde esa notificación se selecciona **Promover este servidor a controlador de dominio**.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/1.png" width="800"></p>

### 2.1 Nuevo bosque y nombre del dominio

En **Configuración de implementación** se elige **Agregar un nuevo bosque** y se escribe el nombre del dominio raíz: `adlab.lan`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/2.png" width="800"></p>

### 2.2 Opciones del controlador de dominio

| Opción                                    | Valor                       |
| ----------------------------------------- | --------------------------- |
| Nivel funcional del bosque                | Windows Server 2016         |
| Nivel funcional del dominio               | Windows Server 2016         |
| Servidor DNS                              | Activado                    |
| Catálogo global (GC)                      | Activado                    |
| Controlador de dominio de solo lectura    | Desactivado                 |

La contraseña del modo de restauración de servicios de directorio (DSRM) se utiliza para recuperar el dominio si falla.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/3.png" width="800"></p>

### 2.3 Nombre NetBIOS

El asistente propone `ADLAB`, que se deja sin cambios.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/4.png" width="800"></p>

### 2.4 Rutas de acceso

Se dejan las rutas por defecto:

|Elemento|Ruta|
|---|---|
|Base de datos de AD DS|`C:\Windows\NTDS`|
|Archivos de registro|`C:\Windows\NTDS`|
|SYSVOL|`C:\Windows\SYSVOL`|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/5.png" width="800"></p>

### 2.5 Requisitos previos e instalación

La comprobación de requisitos previos termina correctamente, con dos advertencias:

- La configuración de seguridad que permite algoritmos de criptografía compatibles con Windows NT 4.0 viene desactivada por defecto en los controladores de dominio de Windows Server 2022.
- No se puede crear una delegación para este servidor DNS porque no se encuentra la zona principal autoritativa. 

El asistente avisa además de que el servidor se reiniciará automáticamente al terminar la promoción. Se pulsa **Instalar**.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/6.png" width="800"></p>

### 2.6 Reinicio e inicio de sesión en el dominio

El servidor se reinicia y aplica la configuración.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/7.png" width="800"></p>

Al volver a cargar, el inicio de sesión ya es con la cuenta del dominio: `ADLAB\Administrador`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/8.png" width="800"></p>

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)

## 3. Estructura del dominio

La estructura se crea desde **Herramientas → Usuarios y equipos de Active Directory**.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/9.png" width="800"></p>

### 3.1 Unidades organizativas (OUs)

|OU|Ubicación|Contenido|
|---|---|---|
|`ADLAB`|Raíz del dominio|OU principal del proyecto|
|`Clientes`|Dentro de `ADLAB`|Usuario del cliente|

La OU `ADLAB` toma el nombre del dominio. En la creación no se marcó la protección contra eliminación accidental.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/10.png" width="800"></p>

### 3.2 Usuario del cliente

Se crea un usuario en la OU `Clientes`:

|Campo|Valor|
|---|---|
|Nombre completo|Pablo Martinez|
|Inicio de sesión de usuario|`pmartinez@adlab.lan`|
|Inicio de sesión (anterior a Windows 2000)|`ADLAB\pmartinez`|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/11.png" width="800"></p>

Opciones de contraseña al crear el usuario (la contraseña no se documenta):

|Opción|Estado|
|---|---|
|El usuario debe cambiar la contraseña en el siguiente inicio de sesión|Desmarcada|
|El usuario no puede cambiar la contraseña|**Marcada**|
|La contraseña nunca expira|Desmarcada|
|La cuenta está deshabilitada|Desmarcada|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/12.png" width="800"></p>

**Ajuste posterior.** Como el usuario no puede cambiar su contraseña, si esta caducara durante el laboratorio solo un administrador podría restablecerla, y las pruebas de GPOs y auditoría (que se hacen iniciando sesión como `pmartinez`) quedarían bloqueadas. Por eso se abren las **Propiedades** del usuario, pestaña **Cuenta**, y se marca **La contraseña nunca expira**.

Estado final de la cuenta:

|Opción|Estado|
|---|---|
|El usuario debe cambiar la contraseña en el siguiente inicio de sesión|Desmarcada|
|El usuario no puede cambiar la contraseña|Marcada|
|La contraseña nunca expira|**Marcada**|
|Almacenar contraseña utilizando cifrado reversible|Desmarcada|
|Desbloquear cuenta|Desmarcada|
|La cuenta expira|Nunca|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/13.png" width="800"></p>

En la pestaña **Sesiones**, las sesiones no finalizan por sí solas (sin captura).

El usuario no se agrega a ningún grupo adicional: pertenece solo al grupo por defecto de los usuarios nuevos del dominio.

### 3.3 Cuenta del equipo cliente

No se crea manualmente la cuenta del equipo Windows 10. Al unirlo al dominio, la cuenta se genera automáticamente. 

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)

## 4. Paso posterior de DHCP

Con el servidor ya en el dominio, el Administrador del servidor avisa de que el rol DHCP requiere configuración posterior.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/14.png" width="800"></p>

El asistente de **posinstalación de DHCP** muestra lo que hará: crear los grupos de seguridad **Administradores de DHCP** y **Usuarios de DHCP** para delegar la administración, y autorizar el servidor DHCP en el dominio.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/15.png" width="800"></p>

En **Autorización** se usan las credenciales del administrador del dominio (`ADLAB\Administrador`) y se confirma.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/03%20-%20active-directory/16.png" width="800"></p>

> Este paso solo autoriza al servidor DHCP en Active Directory. El ámbito, las reservas y las opciones que entrega el DHCP se configuran en `04-dhcp-dns.md`.

**Recurso complementario:** [enlace](https://www.youtube.com/watch?v=joIubWzQ6P8&t=1375s)
## 5. Estado y alcance de esta fase

| Elemento                                          | Estado     |
| ------------------------------------------------- | ---------- |
| Promoción a controlador de dominio (nuevo bosque) | Hecho      |
| Inicio de sesión con la cuenta del dominio        | Hecho      |
| OUs `ADLAB` y `Clientes`                          | Hecho      |
| Usuario `pmartinez`                               | Hecho      |
| Contraseña de `pmartinez` sin expiración          | Hecho      |
| Grupos propios del proyecto                       | No creados |
| DHCP autorizado en AD DS                          | Hecho      |
| Ámbito DHCP                                       | Pendiente  |
| DNS (configuración definitiva y forwarder)        | Pendiente  |
| Verificación del dominio y de la zona DNS         | Pendiente  |
| Windows 10 en el dominio                          | Pendiente  |

**Siguiente:** `04-dhcp-dns.md`, donde se configura el ámbito DHCP y el DNS, el cliente obtiene su IP y se une al dominio.
