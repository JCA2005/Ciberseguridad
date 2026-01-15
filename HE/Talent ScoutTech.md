# Informe de Auditoría de Seguridad: Talent ScoutTech

![image](https://github.com/user-attachments/assets/b4474799-f73e-4fa3-8efd-9e287fa14954)

**Proyecto:** Auditoría Web y Pentesting - Talent ScoutTech  
**Fecha:** 14/01/2026  
**Tecnologías:** Apache, PHP, SQLite3  
**Autor:** Javier Calvillo Acebedo

---

## 1. Introducción
El objetivo de este proyecto ha sido realizar una auditoría de seguridad (Pentesting) sobre la aplicación web *Talent ScoutTech*. Se han analizado vulnerabilidades críticas siguiendo la metodología OWASP, centrándose en la inyección SQL, Cross-Site Scripting (XSS), gestión de sesiones insegura y Cross-Site Request Forgery (CSRF).

A continuación se detallan los hallazgos, las evidencias gráficas obtenidas durante las pruebas y las propuestas de mitigación.

### Resumen de Vulnerabilidades Halladas

| Vulnerabilidad | CWE | Severidad (CVSS v3) | Estado |
| :--- | :--- | :--- | :--- |
| **SQL Injection (Login)** | CWE-89 | Crítica (9.8) | Explotada |
| **Broken Access Control (IDOR)** | CWE-639 | Alta (7.5) | Explotada |
| **Cross-Site Scripting (Stored)** | CWE-79 | Alta (7.2) | Explotada |
| **Cross-Site Request Forgery** | CWE-352 | Media (6.5) | Explotada |
| **Information Disclosure (.php~)** | CWE-530 | Media (5.3) | Identificada |

---

## Configuración Previa

Para empezar con esta actividad, ejecutaremos **XAMPP** y seguidamente iniciaremos **Apache**:

<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 083531" src="https://github.com/user-attachments/assets/8773916c-b6a1-4338-9e63-6529205cdf14" />

Posteriormente copiaremos el zip **Web_Talent-ScoutTech** extraido que nos han proporcionado y lo pegaremos en la ruta **C:\xampp\htdocs**:

<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 100952" src="https://github.com/user-attachments/assets/82de0f79-6de0-43ac-bc6f-ba001ae43ee7" />

Seguidamente abriremos el navegador y accederemos a la ruta **http://localhost/Web_Talent-ScoutTech/web/**, como podemos ver ya estamos dentro de la web:

<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 083559" src="https://github.com/user-attachments/assets/0a5b5ea1-8f8f-48bd-9468-7cae1fb5aa8f" />

## Parte 1: Inyección SQL (SQLi) y Autenticación

### a) Evasión del Login mediante SQL Injection
El formulario de acceso no sanitiza correctamente las entradas del usuario, permitiendo la manipulación de la consulta SQL subyacente.

* **Vulnerabilidad:** SQL Injection (SQLi) en el campo de usuario.
* **Vector de ataque:** Se introdujo la cadena `' OR '1'='1` en el campo `User`.
* **Análisis de la Consulta:**
    La consulta original en el código PHP tiene esta estructura:
    `SELECT userId, password FROM users WHERE username = '$username' AND password = '$password';`
    
    Al inyectar el payload, la consulta resultante es:
    `SELECT userId, password FROM users WHERE username = '' OR '1'='1' AND password = '...';`

    **Explicación Técnica:** La comilla simple cierra la cadena de texto en la consulta SQL original. La condición `OR '1'='1'` siempre se evalúa como verdadera. Esto provoca que la base de datos devuelva el primer registro de la tabla (generalmente el administrador) sin validar la contraseña.

**Evidencia:** Payload inyectado en el formulario de login.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 085800" src="https://github.com/user-attachments/assets/8a643b4a-4746-4fa8-ae9a-74a63bbc8aa7" />

### b) Ataque de Diccionario
En el caso supuesto de que la inyección SQL estuviera parcheada, el sistema carece de mecanismos contra fuerza bruta.
* **Metodología:** Para impersonar al usuario "luis", se utilizaría una herramienta automatizada como **Burp Suite Intruder** o **Hydra**. Se configuraría el ataque contra `insert_player.php` (POST), fijando el usuario `luis` e iterando sobre el diccionario de contraseñas comunes (`123456`, `password`, etc.) hasta obtener un acceso exitoso.

<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 093314" src="https://github.com/user-attachments/assets/081184fc-aa74-4587-bc1d-086944f43a49" />

### c) Análisis de código: `SQLite3::escapeString()`
El análisis del código fuente (`register.php`) revela el uso de `SQLite3::escapeString()`.
* **Problema:** Esta función es insuficiente para la seguridad moderna. Solo escapa comillas simples (`'`), pero no protege si la inyección ocurre en un contexto numérico (donde no se usan comillas) o si hay errores lógicos en la query. Además, no separa los datos de la estructura de la consulta.
* **Solución:** Es obligatorio implementar **Consultas Preparadas (Prepared Statements)**, donde los parámetros se envían por separado al motor de la base de datos.

### d) Suplantación de Identidad (IDOR y Archivos de Respaldo)
Durante la fase de reconocimiento (Fuzzing), detectamos un archivo de respaldo olvidado en el servidor: `add_comment.php~`. Al analizar su código fuente, descubrimos que la aplicación confía ciegamente en una cookie llamada `userId` para identificar al autor.

Esto nos permite explotar un **Insecure Direct Object Reference (IDOR)**.

**Prueba de Concepto:**
1.  Mediante las herramientas de desarrollador (F12), se inspeccionó el almacenamiento local.
2.  Se localizó la cookie `user` y se modificó manualmente su valor a `1` (correspondiente al administrador).

**Evidencia:** Modificación de la cookie en el navegador.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 093838" src="https://github.com/user-attachments/assets/17b8f8cb-202f-40a4-b01f-42b69504cfa9" />

3.  Tras modificar la cookie, se publicó un nuevo comentario con el texto "Hola soy el admin Javi".

**Evidencia:** Publicación del comentario tras la manipulación.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 093939" src="https://github.com/user-attachments/assets/b9b4256d-e788-4de5-8839-ae81572a2dfc" />

4.  **Resultado:** El sistema aceptó la cookie manipulada y publicó el comentario bajo el nombre de "luis" (Administrador).

**Evidencia:** Comentario publicado suplantando al usuario Luis.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 094031" src="https://github.com/user-attachments/assets/c0d764f6-d0be-4eff-a94d-412248d8d4a5" />

---

## Parte 2: Cross Site Scripting (XSS)

### a) XSS Almacenado (Stored XSS)
La sección de comentarios (`show_comments.php`) es vulnerable a XSS Almacenado. La aplicación imprime el contenido de los comentarios directamente sin sanitización.

* **Prueba:** Se inyectó el payload `<script>alert('Cuidado con el hacker')</script>`.

<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 094325" src="https://github.com/user-attachments/assets/07df95af-e5ab-4061-a6c9-2bbb6b3ebf65" />

  
* **Resultado:** Al cargar la página, el navegador ejecutó el código JavaScript malicioso.

**Evidencia:** Ejecución del script (Alert) en el navegador.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 094346" src="https://github.com/user-attachments/assets/df6b6818-9bf2-4dc2-b36a-9f16d63a2227" />

Y como podemos ver el comentario sale invisible porque no es un comentario como tal, esto significa que nos ha cogido correctamente el alert mostrado anteriormente.

<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 094359" src="https://github.com/user-attachments/assets/532c7b9d-d439-4525-a4ca-22d8f7e00452" />

### b) Uso de `&`
En HTML, el carácter `&` se utiliza para iniciar referencias a entidades. En los enlaces con parámetros GET, el `&` debe escaparse como `&` para evitar errores de interpretación del navegador y cumplir con el estándar W3C. Si no se hace, el navegador podría confundir los parámetros con entidades HTML inválidas.

### c) Corrección de `show_comments.php`
* **Problema:** Uso de `echo $row['body'];` directo.
* **Solución:** Envolver la salida con `htmlspecialchars($row['body'], ENT_QUOTES, 'UTF-8')`. Esto convierte los caracteres especiales (`<`, `>`, `"`) en entidades HTML seguras que se visualizan pero no se ejecutan.

### d) XSS Reflejado y Fuga de Información
El buscador (`buscador.php`) también es vulnerable. Además, se detectó una grave fuga de información (*Information Disclosure*). Al introducir caracteres especiales, la aplicación devuelve errores de PHP revelando la ruta del servidor (`C:\xampp\htdocs...`).

**Evidencia:** Error de PHP revelando rutas del servidor.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 093810" src="https://github.com/user-attachments/assets/d7ba05dd-6c87-447c-84a4-57588183ad55" />

**Prueba de XSS en Buscador:** Se inyectó `<script>alert(123)</script>` en la barra de búsqueda.

**Evidencia:** Inyección en el buscador.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 100952" src="https://github.com/user-attachments/assets/5a5e9d75-5140-4a77-a1a1-13900e87d608" />

**Evidencia:** Resultado de la búsqueda (Payload reflejado en el título).
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 101253" src="https://github.com/user-attachments/assets/cfcdffd8-13ec-4398-afc3-531c05ba49b8" />

---

## Parte 3: Control de Acceso y Sesiones (Propuestas e Implementación)

Se han identificado vulnerabilidades críticas en la gestión de usuarios. A continuación, se detallan las propuestas de corrección con ejemplos de código:

### 1. Registro Seguro
Las contraseñas se almacenaban en texto plano. Se debe implementar `password_hash()`:
```php
// register.php corregido
$hash = password_hash($_POST['password'], PASSWORD_DEFAULT);
$stmt = $db->prepare("INSERT INTO users (username, password) VALUES (:u, :p)");
$stmt->bindValue(':u', $user);
$stmt->bindValue(':p', $hash);
```

### 2. Login Seguro
Se debe verificar el hash y usar consultas preparadas:

```php
// auth.php corregido
$stmt = $db->prepare("SELECT * FROM users WHERE username = :u");
$stmt->bindValue(':u', $user);
// ... fetch row ...
if (password_verify($_POST['password'], $row['password'])) {
    // Login correcto
}
```

### 3. Acceso al Registro
Usuarios logueados no deberían poder registrarse de nuevo.

Medida: Añadir al inicio de register.php:

```php
if (isset($_SESSION['user_id'])) {
    header("Location: list_players.php");
    exit();
}
```

### 4. Protección Carpeta /private/
Es vital evitar el acceso directo a archivos de configuración.

Medida: Crear un archivo .htaccess dentro de la carpeta /private/ con el siguiente contenido:

```
Deny from all
```

### 5. Sesiones Seguras
Actualmente se usan cookies editables (user y password en el navegador).

Solución: Usar sesiones de servidor.

```php
session_start();
$_SESSION['user_id'] = $db_id; // Guardar ID en servidor
// Eliminar setcookie de usuario y contraseña
```

## Parte 4: Servidores Web
El servidor Apache tiene habilitado el listado de directorios, permitiendo ver la estructura de archivos, lo cual facilita la fase de reconocimiento al atacante (Information Disclosure).

**Evidencia:** Listado de directorios visible.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 102042" src="https://github.com/user-attachments/assets/b65b52f9-48e0-42e4-b1f5-aca52176d036" />

Medidas de Mitigación en el Servidor:

Desactivar Indexing: Añadir `Options -Indexes` en el archivo .htaccess o en la configuración de Apache (httpd.conf).

Ocultar Errores: Configurar `display_errors = Off` en php.ini para evitar fugar rutas del sistema (como se vio en la parte 2d).

Harding: Deshabilitar la firma del servidor (`ServerSignature Off`).

## Parte 5: CSRF (Cross-Site Request Forgery)
Se diseñaron ataques para forzar a un usuario logueado en la plataforma ficticia web.pagos a realizar una donación involuntaria.

### a) Ataque mediante Botón Trampa
Se inyectó código HTML en el campo "Team name" de un jugador para crear un botón falso.

Código: `<a href="http://web.pagos/donate.php?amount=100&receiver=attacker">Profile</a>`

**Evidencia:** Inyección del código en la edición del jugador.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 101905" src="https://github.com/user-attachments/assets/3356dbd4-ea18-4a67-962d-14c52cce7c20" />

**Resultado:** El botón aparece en el listado público.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 101943" src="https://github.com/user-attachments/assets/2c595817-5886-4bcd-b30a-f4322950e868" />

**(Prueba de redirección al sitio externo):**
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 101957" src="https://github.com/user-attachments/assets/8efd8f84-6e91-4071-883c-60ee682b2846" />

### b) Ataque Invisible (Zero-Click)
Se utilizó una etiqueta de imagen (`<img>`) dentro de un comentario para ejecutar la petición automáticamente.

Código: `¡Qué buen jugador! <img src="http://web.pagos/donate.php?amount=100&receiver=attacker" width="0" height="0">`

**Evidencia:** Inyección del payload en comentario.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 102042" src="https://github.com/user-attachments/assets/cf391653-0f76-4ff6-b1f4-8afef1e497c6" />

**Evidencia:** Comentario publicado con la trampa oculta.
<img width="1920" height="1020" alt="Captura de pantalla 2026-01-14 102109" src="https://github.com/user-attachments/assets/bce57e45-303d-4de3-aa25-3b5e3dcad955" />

### c) Condiciones para el éxito del ataque
Para que este ataque funcione, deben cumplirse dos condiciones fundamentales:

Sesión Activa: La víctima debe haber iniciado sesión previamente en web.pagos en el mismo navegador y la sesión no debe haber expirado.

Cookies Persistentes: El navegador debe enviar automáticamente las cookies de sesión de web.pagos al realizar la petición a donate.php, incluso si la petición se origina desde nuestro sitio atacante (Third-party cookies).

### d) Ataque vía POST
Si la web de pagos requiriera POST para mitigar el ataque, seguiría siendo vulnerable si no implementa Tokens Anti-CSRF. Se usaría JavaScript para enviar un formulario oculto automáticamente:

```html
<form action="http://web.pagos/donate.php" method="POST" id="csrf">
    <input type="hidden" name="amount" value="100">
    <input type="hidden" name="receiver" value="attacker">
</form>
<script>document.getElementById('csrf').submit();</script>
