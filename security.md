# Política de seguridad

## 1. Alcance

Este repositorio documenta un laboratorio educativo de Active Directory, GPOs y firewall. Contiene documentación y capturas, no software que se despliegue en producción. Por eso no hay versiones con soporte de seguridad: la política cubre la documentación y los datos que se publican en ella.

## 2. Qué no se publica

- Contraseñas, ni siquiera enmascaradas, ni claves ni tokens.
- Datos personales.
- Identificadores de equipos o de licencias que permitan reconocerlos.

Las capturas se revisan y se recortan antes de subirlas. Si encuentras alguna con un dato de este tipo, repórtala como se indica en la sección 4.

## 3. Decisiones inseguras a propósito

Estas decisiones son deliberadas, están documentadas y se limitan a un laboratorio aislado. No son una recomendación para un entorno real.

|Decisión|Documento|Motivo y riesgo|
|---|---|---|
|IIS con ASP instalado en el controlador de dominio|`02-servidor-base.md`, sección 7|Crea una superficie de ataque controlada. En un entorno real amplía la superficie de la máquina más crítica del dominio.|
|Acceso administrativo al firewall por HTTPS, HTTP, SSH y PING en `port2`|`02-servidor-base.md`, sección 5.2|Facilita administrar el firewall desde la red interna. HTTP no cifra el tráfico.|
|Política de salida amplia (`LAN` → `all`, servicio `ALL`)|`06-reglas-fortigate.md`|Da conectividad completa al cliente. No restringe destinos ni servicios.|
|Inspección SSL en `no-inspection`|`06-reglas-fortigate.md`|No se inspecciona el contenido HTTPS. La inspección profunda exigiría distribuir la CA del firewall a los clientes.|
|Usuario `pmartinez` sin poder cambiar la contraseña y con contraseña que nunca expira|`03-active-directory.md`|Evita bloquear las pruebas de GPOs. Es una mala práctica fuera del laboratorio.|
|OU creadas sin protección contra eliminación accidental|`03-active-directory.md`|Simplifica el laboratorio. Una OU podría borrarse por error.|
|Sitio web de prueba por HTTP, sin certificado|`02-servidor-base.md`|Es una página de simulación. El navegador la marca como **No seguro**.|

Además, el FortiGate usa una licencia de evaluación sin servicios de FortiGuard, por lo que parte de sus protecciones no es efectiva. Se explica en `06-reglas-fortigate.md`.

## 4. Cómo reportar un problema

Si encuentras un dato sensible publicado por error, una captura que debería recortarse o un error de seguridad en la documentación:

1. No abras un issue público con los datos sensibles.
2. Usa **Security → Report a vulnerability** en la página del repositorio de GitHub. [PENDIENTE: confirmar que el reporte privado de vulnerabilidades está habilitado en el repositorio]
3. Si no está disponible, escribe a [PENDIENTE: contacto].
4. Indica el archivo, la sección y una breve descripción.

Plazo de respuesta: [PENDIENTE: definir].

## 5. Uso responsable

Los pasos y pruebas de este repositorio son para reproducirse en un entorno propio y aislado. No los uses contra sistemas que no sean tuyos ni sin autorización.
