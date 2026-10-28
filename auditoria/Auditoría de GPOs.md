# Auditoría de GPOs

> **Fase:** auditoría de la fase 7 (`07-gpos.md`) **Alcance:** comprobar en el cliente que la GPO `Politicas_Cliente` se aplica a `pmartinez` y que las restricciones del Panel de control, de la personalización y de agregar o quitar programas impiden abrir esas herramientas. **No incluye:** la verificación de eventos de Windows. **Estado:** la GPO se aplica al usuario y las 5 pruebas de apertura quedaron bloqueadas. No se hicieron pruebas con un usuario de control, por lo que la causa se atribuye a la GPO con reservas (sección 5).

## 1. Resultado

| Elemento                | Resultado                                       |
| ----------------------- | ----------------------------------------------- |
| GPO aplicada al usuario | Sí: `Politicas_Cliente` figura en `gpresult /r` |
| Pruebas bloqueadas      | 5 (ninguna herramienta llegó a abrirse)         |

## 2. Entorno de prueba

|Dato|Valor|
|---|---|
|Equipo|`DESKTOP-AH74NEN` (Windows 10 Pro)|
|Usuario|`pmartinez` (Pablo Martinez), en la OU `Clientes`|
|GPO|`Politicas_Cliente`, vinculada a `Clientes` (ver `07-gpos.md`)|
|Controlador de dominio|`SRV-AD01.adlab.lan`|

## 3. Aplicación de la GPO en el cliente

### 3.1 `gpupdate /force`

Desde la sesión de `pmartinez` se ejecuta `gpupdate /force`. La captura muestra el mensaje de que la actualización de la directiva de equipo se completó correctamente.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/1.png" width="800"></p>

> En la captura solo se ve la línea de la directiva de **equipo**. La línea de la directiva de usuario no aparece. Que la GPO de usuario sí se aplicó lo confirma el `gpresult` siguiente.

### 3.2 `gpresult /r`

En la sección **Configuración de usuario**:

|Campo|Valor|
|---|---|
|Usuario|`CN=Pablo Martinez,OU=Clientes,OU=ADLAB,DC=adlab,DC=lan`|
|Última aplicación de la directiva|27/10/2026, 12:17:44 p. m.|
|Directivas aplicadas desde|`SRV-AD01.adlab.lan`|
|Dominio|`ADLAB`|
|Objetos de directiva de grupo aplicados|`Politicas_Cliente`|
|GPO filtrados|Directiva de grupo local (filtro: no aplicado, vacío)|

Esto confirma que la GPO llega al usuario por estar este en la OU `Clientes`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/2.png" width="800"></p>

## 4. Pruebas de bloqueo

Todas se hacen con la sesión de `pmartinez`. El mensaje de bloqueo es el mismo en las pruebas 1, 2, 4 y 5:

> Esta operación ha sido cancelada debido a las restricciones especificadas para este equipo. Póngase en contacto con el administrador del sistema.

|#|Acción|Directiva relacionada|Resultado|
|---|---|---|---|
|1|Abrir **Panel de control** desde la búsqueda de Inicio|Prohibir el acceso a Configuración de PC y a Panel de control|Bloqueado: ventana "Restricciones"|
|2|`Win+R` → `control`|Ídem|Bloqueado: ventana "Restricciones"|
|3|Clic derecho en el escritorio → **Personalizar**|Ídem y las de Personalización|No se abre Configuración; aparece un error distinto (ver 4.3)|
|4|`Win+R` → `appwiz.cpl`|Ídem y las de Agregar o quitar programas|Bloqueado: ventana "Restricciones"|
|5|`Win+R` → `desk.cpl`|Ídem y las de Personalización|Bloqueado: ventana "Restricciones"|

### 4.1 Panel de control desde la búsqueda de Inicio

El Panel de control sigue apareciendo como **Mejor coincidencia** en la búsqueda de Inicio: la directiva no lo oculta de los resultados. Al abrirlo, se muestra el mensaje de restricción.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/3.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/4.png" width="800"></p>

### 4.2 Panel de control desde Ejecutar

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/5.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/6.png" width="800"></p>

### 4.3 Personalizar desde el escritorio

El menú contextual del escritorio sigue mostrando la opción **Personalizar**.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/7.png" width="800"></p>

Al elegirla no se abre Configuración. Aparece un cuadro titulado `ms-settings:personalization-background` que dice que el archivo no tiene una aplicación asociada para llevar a cabo la acción.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/8.png" width="800"></p>

> Este mensaje es distinto al de "Restricciones". El resultado (no se abre la personalización) es el esperado, pero el texto del error no menciona la GPO.

### 4.4 Agregar o quitar programas (`appwiz.cpl`)

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/9.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/10.png" width="800"></p>

### 4.5 Propiedades de pantalla (`desk.cpl`)

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/11.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/auditoria/GPOs/12.png" width="800"></p>

## 5. Qué demuestran estas pruebas y qué no

**Lo que sí muestran**

- La GPO se aplica al usuario (`gpresult /r`).
- Con ese usuario, las cinco formas de abrir el Panel de control, Configuración, Agregar o quitar programas y las propiedades de pantalla quedaron bloqueadas.
- En cuatro de las cinco, el mensaje habla de restricciones especificadas para el equipo.
## 7. Estado y alcance

| Elemento                                           | Estado                     |
| -------------------------------------------------- | -------------------------- |
| `gpupdate /force` en el cliente                    | Hecho                      |
| `gpresult /r` con la GPO aplicada                  | Hecho                      |
| Bloqueo del Panel de control (búsqueda y Ejecutar) | Hecho                      |
| Bloqueo de Personalizar                            | Hecho (con error distinto) |
| Bloqueo de `appwiz.cpl` y `desk.cpl`               | Hecho                      |
