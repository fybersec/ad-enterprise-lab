# 00 - Entorno

> **Fase:** 0 (Entorno) **Alcance:** versiones, recursos de las VMs y método de instalación de cada máquina. La instalación paso a paso con capturas ya está documentada en proyectos anteriores; aquí solo se referencia. La red (segmentos, direccionamiento y conectividad) se documenta en `01-topologia.md`.

## 1. Hipervisor

|Parámetro|Valor|
|---|---|
|Hipervisor|VMware Workstation Pro|
|Host|Windows, i7 core, 16RAM|

## 2. Máquinas del laboratorio

|Máquina|Rol|SO / versión|RAM|vCPU|Disco|
|---|---|---|---|---|---|
|FortiGate-VM|Gateway / firewall|FortiOS `VM64`|`2GB`|`1`|`HD1(SCSI)-2GB HD2(SCSI)-30GB`|
|Windows Server|Controlador de dominio (AD DS, DNS, DHCP)|Windows Server 2022 `Desktop`|`5GB`|`2-4`|`HD(SCSI)-70`|
|Cliente|Estación de trabajo unida al dominio|Windows 10 Pro `Desktop`|`3-4`|`2-3`|`HD(SCSI)-30GB`|

> Direcciones IP, hostnames y adaptadores de red: ver `01-topologia.md`. Además de estas 3 VMs, el laboratorio usa un equipo físico externo.

## 3. Reutilización de proyectos anteriores

El FortiGate-VM y el Windows Server provienen de un laboratorio previo y se reutilizan sin repetir su montaje. Solo el cliente Windows 10 se instala en este proyecto.

|Máquina|Estado en este proyecto|Documentación del montaje|
|---|---|---|
|FortiGate-VM|Reutilizada|[Mi GitHub](https://github.com/fybersec/fortigate-ngfw-lab/blob/main/docs/00-entorno-fortigate.md)|
|Windows Server|Reutilizada|[Mi GitHub](https://github.com/fybersec/sqli-sqlserver-lab/blob/main/docs/00-entorno-vms.md)|
|Windows 10 Pro|Nueva instalación|Este documento (sección 6)|

## 4. FortiGate-VM

**Método de instalación:** importación de la plantilla OVF ya provista por Fortinet; no se crea la VM manualmente.

1. Se abre la plantilla desde el hipervisor (**File → Open**) y se despliega la VM.
2. Al primer arranque se inicia sesión por la consola con el usuario `admin`.
3. El sistema exige cambiar la contraseña en el primer acceso.

**Enlaces**

- Documentación oficial: [Enlace](https://docs.fortinet.com/document/fortigate-private-cloud/8.0.0/vmware-esxi-administration-guide/143812/deploying-the-fortigate-vm)
- Mi documentación paso a paso: [Enlace](https://github.com/fybersec/fortigate-ngfw-lab/blob/main/docs/00-entorno-fortigate.md)

## 5. Windows Server 2022

**Origen de la imagen:** ISO descargada directamente del sitio oficial de Microsoft.

**Método de instalación:** manual (sin instalación asistida/automática del hipervisor).

1. Se crea la VM sin medio de instalación.
2. Se elimina el disco virtual creado por defecto.
3. Se crea un nuevo disco con controlador `SCSI`.
4. Se agrega la ISO a la VM y se arranca desde ella.

**Motivo del cambio de controlador:** `El instalador no detectaba el disco.`

**Enlaces**

- Descarga oficial: [Enlace](https://www.microsoft.com/es-es/evalcenter/download-windows-server-2022)
- Documentación oficial de instalación: [Enlace](https://www.alitajran.com/install-windows-server/)
- Mi documentación paso a paso: [Enlace](https://github.com/fybersec/sqli-sqlserver-lab/blob/main/docs/00-entorno-vms.md)

## 6. Windows 10 Pro (cliente)

**Origen de la imagen:** la página oficial de Microsoft entrega una herramienta ejecutable (`MediaCreationTool_22H2`). Al ejecutarla genera la ISO.

**Método de instalación:** manual, igual que en Windows Server (sin instalación asistida del hipervisor).

1. Se crea la VM sin medio de instalación.
2. Se elimina el disco SATA creado por defecto.
3. Se crea un nuevo disco de 30 GB con controlador `SCSI`.
4. Se agrega la ISO a la VM y se arranca desde ella.

**Configuración durante la instalación**

|Parámetro|Valor|
|---|---|
|Idioma a instalar|Español (España, internacional)|
|Formato de hora y moneda|Español (país de origen)|
|Teclado|Inglés (Estados Unidos)|
|Edición|Windows 10 Pro|
|Licencia|Se aceptan los términos del software|
|Disco de destino|Disco de 30 GB agregado a la VM|
|Cuenta local|`Mario`|

**Configuración posterior a la instalación**

1. Se instalan las VMware Tools, para que la pantalla de la VM se ajuste a la ventana.
2. Se reinicia la VM.

**Edición:** Pro, necesaria para unir el equipo al dominio.

**Enlaces**

- Descarga oficial: [Enlace](https://www.microsoft.com/es-es/software-download/windows10)
- Documentación oficial de instalación: [Enlace](https://www.youtube.com/watch?v=_1cXyzN2jwQ)
