# 07 - GPOs

> **Fase:** 7 (GPOs) **Alcance:** creación de una GPO de usuario, vinculada a la OU `Clientes`, con restricciones sobre el panel de control, la personalización del escritorio y la opción de agregar o quitar programas. **No incluye:** las pruebas que demuestran si las restricciones funcionan (`auditoria/gpos.md`). **Estado:** la GPO está creada, vinculada y con las directivas configuradas en el servidor. Falta aplicarla en el cliente con `gpupdate /force` y probarla.

## 1. Resultado de esta fase

| Parámetro                  | Valor                                                                         |
| -------------------------- | ----------------------------------------------------------------------------- |
| GPO                        | `Politicas_Cliente`                                                           |
| Ámbito                     | Configuración de usuario (la configuración del equipo no se toca)             |
| OU de vinculación          | `ADLAB\Clientes`                                                              |
| Usuario afectado           | `pmartinez` (está en `Clientes`)                                              |
| Orden de vínculo           | 1                                                                             |
| Estado del vínculo         | Habilitado, no exigido, sin filtro WMI                                        |
| Directivas habilitadas     | 7 (ver sección 5)                                                             |

## 2. Decisiones de diseño

| Decisión                                  | Motivo                                                                                                                      |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Una sola GPO para todas las restricciones | El laboratorio tiene un solo usuario y pocas directivas. Todas son de usuario y van a la misma OU, así que una GPO basta.   |
| GPO propia, sin tocar las predeterminadas | No se modifican `Default Domain Policy` ni `Default Domain Controllers Policy`.                                             |
| Vinculación a `Clientes` y no al dominio  | Vincularla al dominio alcanzaría también al administrador y al servidor. En `Clientes` solo afecta a `pmartinez`.           |
| Solo directivas de usuario                | El alcance lo decide la OU del usuario, no la del equipo. Por eso la ubicación de la cuenta de equipo no afecta a esta GPO. |

## 3. Qué entra y qué queda fuera

Solo se aplican las restricciones de este documento.

|Restricción|Estado|
|---|---|
|Panel de control y Configuración de PC|Incluida|
|Personalización del escritorio|Incluida|
|Agregar o quitar programas|Incluida|

> Fundamental no bloquear CMD, PowerShell ni el cuadro Ejecutar, `gpupdate /force` y `gpresult /r` se pueden ejecutar desde la sesión de `pmartinez`.

## 4. Creación y vinculación de la GPO

### 4.1 Consola de administración de directivas de grupo

Desde el Administrador del servidor: **Herramientas → Administración de directivas de grupo**.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/1.png" width="800"></p>

### 4.2 Crear la GPO directamente en la OU

En el árbol `adlab.lan → ADLAB`, clic derecho sobre la OU `Clientes` y **Crear un GPO en este dominio y vincularlo aquí...**. Con esta opción la GPO se crea y se vincula en un solo paso.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/2.png" width="800"></p>

### 4.3 Nombre

|Campo|Valor|
|---|---|
|Nombre|`Politicas_Cliente`|
|GPO de inicio de origen|(ninguno)|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/3.png" width="800"></p>

### 4.4 Vínculo resultante

En la pestaña **Objetos de directiva de grupo vinculados** de `Clientes` aparece la GPO:

|Columna|Valor|
|---|---|
|Orden de vínculos|1|
|GPO|`Politicas_Cliente`|
|Exigido|No|
|Vínculo habilitado|Sí|
|Estado de GPO|Habilitado|
|Filtro WMI|Ninguno|

Con clic derecho sobre la GPO se elige **Editar...** para abrir el editor.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/4.png" width="800"></p>

## 5. Directivas configuradas

Todas están en **Configuración de usuario → Directivas → Plantillas administrativas → Panel de control**. En el editor, la GPO se identifica como `Directiva Politicas_Cliente [SRV-AD01...]`.

|#|Directiva|Carpeta|Estado|
|---|---|---|---|
|1|Prohibir el acceso a Configuración de PC y a Panel de control|Panel de control|Habilitada|
|2|Impedir cambiar el tema|Personalización|Habilitada|
|3|Impedir cambiar el color y la apariencia|Personalización|Habilitada|
|4|Impedir cambiar el fondo de pantalla|Personalización|Habilitada|
|5|Impedir cambiar iconos del escritorio|Personalización|Habilitada|
|6|Ocultar la opción "Agregar un programa desde un CD-ROM ..." (el nombre se corta en la interfaz)|Agregar o quitar programas|Habilitada|
|7|Quitar Agregar o quitar programas|Agregar o quitar programas|Habilitada|

### 5.1 Panel de control

La carpeta **Panel de control** contiene cinco directivas, todas en **No configurada** antes de empezar.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/5.png" width="800"></p>

Se abre **Prohibir el acceso a Configuración de PC y a Panel de control** y se marca **Habilitada**. Se confirma con **Aceptar**.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/6.png" width="800"></p>

Según la ayuda de la propia directiva, impide que se ejecuten `Control.exe` y `SystemSettings.exe`, por lo que el usuario no puede abrir el Panel de control ni Configuración de PC. Además quita ambos de la pantalla Inicio y, en el caso del Panel de control, del Explorador de archivos.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/7.png" width="800"></p>

### 5.2 Personalización

En la carpeta **Personalización** (16 directivas) quedan habilitadas cuatro:

|Directiva|Estado|
|---|---|
|Impedir cambiar el tema|Habilitada|
|Impedir cambiar el color y la apariencia|Habilitada|
|Impedir cambiar el fondo de pantalla|Habilitada|
|Impedir cambiar iconos del escritorio|Habilitada|

Se habilitan las cuatro juntas porque la ayuda de **Impedir cambiar el tema** indica que, aunque el usuario no pueda cambiar ni guardar un tema, aún podría cambiar sus elementos (color, sonidos, protector de pantalla y fondo del escritorio) salvo que otras directivas los desactiven.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/8.png" width="800"></p>

> **Alcance real del fondo:** la directiva impide que el usuario **cambie** el fondo; no establece uno concreto. Para fijar una imagen haría falta otra directiva, que no se configuró en esta fase.

### 5.3 Agregar o quitar programas

En la carpeta **Agregar o quitar programas** (11 directivas) quedan habilitadas dos:

|Directiva|Estado|
|---|---|
|Ocultar la opción "Agregar un programa desde un CD-ROM ..."|Habilitada|
|Quitar Agregar o quitar programas|Habilitada|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/9.png" width="800"></p>

> **Limitación:** estas directivas ocultan opciones de la interfaz. No impiden por sí solas que el usuario ejecute un instalador. Para restringir la ejecución de programas haría falta otra herramienta (por ejemplo AppLocker o directivas de restricción de software), que no se usa aquí.

## 6. Verificación

La GPO se configuró en el servidor. Para que llegue al cliente se actualiza la directiva con `gpupdate /force` en la sesión de `pmartinez`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/10.png" width="800"></p>
<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/07%20-%20GPOs/11.png" width="800"></p>

Las pruebas funcionales (intentar abrir el Panel de control, cambiar el fondo, abrir Agregar o quitar programas) se documentan en `auditoria/gpos.md`.

|Comprobación|Resultado|
|---|---|
|GPO creada y vinculada a `Clientes`|Hecho|
|Directivas habilitadas en el editor|Hecho|

## 7. Estado y alcance de esta fase

| Elemento                                     | Estado                          |
| -------------------------------------------- | ------------------------------- |
| GPO `Politicas_Cliente` creada               | Hecho                           |
| Vinculada a la OU `Clientes`                 | Hecho                           |
| Panel de control y Configuración de PC       | Configurado                     |
| Personalización (tema, color, fondo, iconos) | Configurado                     |
| Agregar o quitar programas                   | Configurado                     |
| Aplicación en el cliente (`gpupdate /force`) | Pendiente                       |
| Pruebas funcionales                          | Pendiente (`auditoria/gpos.md`) |

**Siguiente:** `auditoria/gpos.md`, donde se actualiza la directiva en el cliente y se prueba cada restricción.
