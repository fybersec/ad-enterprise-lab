## 08 - Sitio web de prueba en IIS

> Es una simulación: el sitio solo da algo que visitar y que deje registros en IIS. No es una aplicación real ni se le hace ningún ajuste.

### 8.1 Plantilla elegida

| Dato        | Valor                                                                                         |
| ----------- | --------------------------------------------------------------------------------------------- |
| Repositorio | [AHMAD-JX/Website-GameOver](https://github.com/AHMAD-JX/Website-GameOver)                     |
| Descripción | Plantilla web de una tienda de videojuegos                                                    |
| Tecnología  | HTML, CSS y JavaScript estáticos (`index.html`, `Home.html`, `Login.html` y carpeta `assets`) |
| Licencia    | MIT (según figura en GitHub)                                                                  |

Se eligió porque es estática: no necesita compilación ni base de datos y basta con copiarla a la carpeta del sitio.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/1.png" width="800"></p>

### 8.2 Descarga y copia al servidor

La plantilla se descarga desde el propio servidor con **Code → Download ZIP** (el servidor tiene salida a Internet por el FortiGate, ver sección 5).

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/2.png" width="800"></p>

El ZIP (`Website-GameOver-main`, 77.2 MB) queda en la carpeta raíz del sitio de IIS, `C:\inetpub\wwwroot`, donde ya estaban los archivos `iisstart` de la página por defecto.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/3.png" width="800"></p>

Se extrae en esa misma carpeta (110 elementos copiados).

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/4.png" width="800"></p>

Una vez extraído, el ZIP se elimina.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/5.png" width="800"></p>

El contenido de la plantilla queda en `C:\inetpub\wwwroot\lab`: la carpeta `assets` y los archivos `Home`, `index`, `Login`, `LICENSE` y `README.md`.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/6.png" width="800"></p>

> La página por defecto de IIS no se sustituye: la plantilla vive en la subcarpeta `lab`, por lo que se abre en `/lab/` y no en la raíz.

### 8.3 Prueba desde el servidor

Con `localhost` se ve la página por defecto de IIS, y con `localhost/lab/`, la plantilla.

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/7.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/8.png" width="800"></p>

### 8.4 Prueba desde el cliente

Desde el Windows 10, con la IP del servidor, se abren las mismas dos páginas. El navegador marca **No seguro** porque el sitio usa HTTP, sin certificado.

|URL|Resultado|
|---|---|
|`http://192.168.100.100`|Página por defecto de IIS|
|`http://192.168.100.100/lab/`|Plantilla GameOver|

<p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/9.png" width="800"></p> <p align="center"><img src="https://github.com/fybersec/ad-enterprise-lab/blob/main/evidencias/docs/08%20-%20Webapp/10.png" width="800"></p>

Este tráfico circula dentro del mismo segmento (`192.168.100.0/24`) y no pasa por el FortiGate, como se explica en `01-topologia.md`.

### 8.5 Limitaciones

- Es una plantilla estática (archivos `.html`). No usa ASP, aunque el rol esté instalado.
- La página `Login.html` es solo diseño. No se ha comprobado que valide credenciales.
- Los accesos del cliente deberían quedar en los registros HTTP de IIS.

### Cambio en la tabla de estado (sección 9)

|Elemento|Estado|
|---|---|
|Sitio web de prueba en IIS (`/lab/`)|Hecho; accesible desde el servidor y desde el cliente|
|Registros de acceso en IIS|Sin verificar|
