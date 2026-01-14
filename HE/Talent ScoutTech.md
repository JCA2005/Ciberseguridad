# Informe de Auditoría de Seguridad: Talent ScoutTech

**Proyecto:** Auditoría Web y Pentesting - Talent ScoutTech
**Fecha:** 14/01/2026
**Tecnologías:** Apache, PHP, SQLite3

---

## 1. Introducción
El objetivo de este proyecto ha sido realizar una auditoría de seguridad (Pentesting) sobre la aplicación web *Talent ScoutTech*. Se han analizado vulnerabilidades críticas siguiendo la metodología OWASP, centrándose en la inyección SQL, Cross-Site Scripting (XSS), gestión de sesiones insegura y Cross-Site Request Forgery (CSRF).

A continuación se detallan los hallazgos, las evidencias gráficas obtenidas durante las pruebas y las propuestas de mitigación.

---

## Parte 1: Inyección SQL (SQLi) y Autenticación

### a) Evasión del Login mediante SQL Injection
El formulario de acceso no sanitiza correctamente las entradas del usuario, permitiendo la manipulación de la consulta SQL subyacente.

* **Vulnerabilidad:** SQL Injection (SQLi) en el campo de usuario.
* **Vector de ataque:** Se introdujo la cadena `' OR '1'='1` en el campo `User`.
* **Explicación Técnica:** La comilla simple cierra la cadena de texto en la consulta SQL original. La condición `OR '1'='1'` siempre se evalúa como verdadera. Esto provoca que la base de datos devuelva el primer registro de la tabla (generalmente el administrador) sin validar la contraseña.

**Evidencia:** Payload inyectado en el formulario de login.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 085800" src="https://github.com/user-attachments/assets/5e2d7484-382a-4f03-b200-bf69a603d596" />



### b) Ataque de Diccionario
En el caso supuesto de que la inyección SQL estuviera parcheada, el sistema carece de mecanismos contra fuerza bruta.
* **Metodología:** Para impersonar al usuario "luis", se utilizaría una herramienta automatizada como **Burp Suite Intruder** o **Hydra**. Se configuraría el ataque contra `insert_player.php` (POST), fijando el usuario `luis` e iterando sobre el diccionario de contraseñas comunes (`123456`, `password`, etc.) hasta obtener un acceso exitoso.

### c) Análisis de código: `SQLite3::escapeString()`
El análisis del código fuente (`register.php`) revela el uso de `SQLite3::escapeString()`.
* **Problema:** Esta función es obsoleta e insegura para la prevención moderna de SQLi. Solo escapa comillas simples, pero no separa la lógica de la consulta de los datos.
* **Solución:** Es obligatorio implementar **Consultas Preparadas (Prepared Statements)**.

### d) Suplantación de Identidad (IDOR en Cookies)
Se descubrió una vulnerabilidad crítica de **Insecure Direct Object Reference (IDOR)**. La aplicación confía en una cookie llamada `userId` para identificar al autor de los comentarios.

**Prueba de Concepto:**
1.  Mediante las herramientas de desarrollador (F12), se inspeccionó el almacenamiento local.
2.  Se localizó la cookie `userId` y se modificó manualmente su valor a `1` (correspondiente al administrador).

**Evidencia:** Modificación de la cookie en el navegador.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 093838" src="https://github.com/user-attachments/assets/3f284d81-a506-4fd2-a2ae-67eb339a2564" />



3.  Tras modificar la cookie, se publicó un nuevo comentario con el texto "Hola soy el admin Javi".

**Evidencia:** Publicación del comentario tras la manipulación.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 093939" src="https://github.com/user-attachments/assets/acaebbed-04a6-4bef-9c7b-bd8820cf69e9" />



4.  **Resultado:** El sistema aceptó la cookie manipulada y publicó el comentario bajo el nombre de "luis" (Administrador).

**Evidencia:** Comentario publicado suplantando al usuario Luis.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 094031" src="https://github.com/user-attachments/assets/221097f9-eaee-4f1a-bd43-aa3b0b8267ad" />



---

## Parte 2: Cross Site Scripting (XSS)

### a) XSS Almacenado (Stored XSS)
La sección de comentarios (`show_comments.php`) es vulnerable a XSS Almacenado. La aplicación imprime el contenido de los comentarios directamente sin sanitización.

* **Prueba:** Se inyectó el payload `<script>alert('Cuidado con el hacker')</script>`.
* **Resultado:** Al cargar la página, el navegador ejecutó el código JavaScript malicioso.

**Evidencia:** Ejecución del script (Alert) en el navegador.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 094346" src="https://github.com/user-attachments/assets/8f50aa2a-39d6-4ca6-8246-c9017a2d17ed" />



### b) Uso de `&amp;`
En HTML, el carácter `&` se utiliza para iniciar referencias a entidades. En los enlaces con parámetros GET, el `&` debe escaparse como `&amp;` para evitar errores de interpretación del navegador y cumplir con el estándar W3C.

### c) Corrección de `show_comments.php`
* **Problema:** Uso de `echo $row['body'];`.
* **Solución:** Envolver la salida con `htmlspecialchars($row['body'], ENT_QUOTES, 'UTF-8')`.

### d) XSS Reflejado y Fuga de Información
El buscador (`buscador.php`) también es vulnerable. Además, se detectó una grave fuga de información (*Information Disclosure*). Al introducir caracteres especiales, la aplicación devuelve errores de PHP revelando la ruta del servidor (`C:\xampp\htdocs...`).

**Evidencia:** Error de PHP revelando rutas del servidor.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 093810" src="https://github.com/user-attachments/assets/8a88de93-ecf8-4bd3-b678-c36f1887c9d5" />



**Prueba de XSS en Buscador:** Se inyectó `<script>alert(123)</script>` en la barra de búsqueda.

**Evidencia:** Inyección en el buscador.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 100952" src="https://github.com/user-attachments/assets/b4740d15-60f1-4370-8056-fdd77b8c8acb" />



**Evidencia:** Resultado de la búsqueda (Payload reflejado en el título).
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 101253" src="https://github.com/user-attachments/assets/a91a368d-5e4d-4feb-a5db-cc33bcf19056" />



---

## Parte 3: Control de Acceso y Sesiones (Propuestas)

1.  **Registro:** Implementar `password_hash()` y validar complejidad de contraseñas.
2.  **Login:** Usar `password_verify()` y Consultas Preparadas.
3.  **Acceso al Registro:** Restringir acceso por IP en `.htaccess` o eliminar el archivo.
4.  **Carpeta `/private/`:** Añadir `.htaccess` con `Deny from all`.
5.  **Sesiones:** Usar `session_start()` y `$_SESSION` en lugar de cookies editables.

---

## Parte 4: Servidores Web

El servidor Apache tiene habilitado el **listado de directorios**, permitiendo ver la estructura de archivos.

**Evidencia:** Listado de directorios visible.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 102042" src="https://github.com/user-attachments/assets/6be8558d-717e-4bee-98df-09df7b6a686f" />



**Medidas:**
1.  Configurar `Options -Indexes`.
2.  Configurar `display_errors = Off` en `php.ini`.
3.  Forzar HTTPS.

---

## Parte 5: CSRF (Cross-Site Request Forgery)

Se diseñaron ataques para forzar a un usuario logueado en la plataforma ficticia `web.pagos` a realizar una donación involuntaria.

### a) Ataque mediante Botón Trampa
Se inyectó código HTML en el campo "Team name" de un jugador para crear un botón falso.
* **Código:** `<a href="http://web.pagos/donate.php?amount=100&receiver=attacker">Profile</a>`

**Evidencia:** Inyección del código en la edición del jugador.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 101905" src="https://github.com/user-attachments/assets/da06c88a-e0a0-44f8-baa1-6bf4211d5862" />



**Resultado:** El botón aparece en el listado público.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 101943" src="https://github.com/user-attachments/assets/fce3e4e0-9d3b-49ad-ab2a-841256418aa3" />



*(Prueba de redirección al sitio externo):*
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 101957" src="https://github.com/user-attachments/assets/3f774409-9f26-400f-916f-cd885427d171" />



### b) Ataque Invisible (Zero-Click)
Se utilizó una etiqueta de imagen (`<img>`) dentro de un comentario para ejecutar la petición automáticamente.
* **Código:** `¡Qué buen jugador! <img src="http://web.pagos/..." width="0" height="0">`

**Evidencia:** Inyección del payload en comentario.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 102042" src="https://github.com/user-attachments/assets/19c45ea2-2142-4e58-8e08-ee236d6d4145" />



**Evidencia:** Comentario publicado con la trampa oculta.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 102109" src="https://github.com/user-attachments/assets/2b35bf5c-2702-4ecf-a927-abc9aed11e74" />



### d) Ataque vía POST
Si la web de pagos requiriera POST, se usaría JavaScript:
```html
<form action="[http://web.pagos/donate.php](http://web.pagos/donate.php)" method="POST" id="csrf">
    <input type="hidden" name="amount" value="100">
</form>
<script>document.getElementById('csrf').submit();</script>
