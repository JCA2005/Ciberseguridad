<h1 align="center">Hardening de router TP-Link Archer AX55</h1>

<img width="1024" height="511" alt="image" src="https://github.com/user-attachments/assets/aae7d301-2e4c-4352-90ce-021ef9ee5185" />

<br>

**Fecha:** 22 de abril de 2026
<br>
**Autor:** Javier Calvillo Acebedo

## Índice

1. [Introducción](#1-introducción)
2. [Gestión y acceso al sistema](#2-gestión-y-acceso-al-sistema)
3. [Mantenimiento y actualizaciones del firmware](#3-mantenimiento-y-actualizaciones-del-firmware)
4. [Seguridad de red WiFi](#4-seguridad-de-red-wifi)
5. [Aislamiento y red de invitados](#5-aislamiento-y-red-de-invitados)
6. [Servicios y protocolos innecesarios](#6-servicios-y-protocolos-innecesarios)
7. [Firewall y control perimetral](#7-firewall-y-control-perimetral)
8. [Conclusión](#8-conclusión)

---

### 1. Introducción

En este trabajo voy a explicar cómo configurar de forma segura un router TP-Link Archer AX55. Este tipo de dispositivos son muy frecuentes tanto en entornos domésticos como en pequeñas oficinas, pero si se dejan con la configuración estándar pueden ser vulnerables a bastantes ataques básicos.

El objetivo es partir del equipo tal y como viene de fábrica y aplicar una serie de cambios esenciales. A lo largo del documento detallaré paso a paso cómo robustecer las contraseñas, mejorar el cifrado de la red WiFi, separar el tráfico de los invitados, desactivar servicios automáticos peligrosos y afinar la configuración del cortafuegos.

En cada punto explicaré la configuración que trae por defecto, los riesgos que esto supone y los cambios específicos que he aplicado en el panel de administración, adjuntando las capturas de pantalla de todo el proceso.

### 2. Gestión y acceso al sistema

Proteger la entrada al panel de configuración es el primer paso vital. Si un usuario no autorizado accede a este menú, tendrá el control total sobre toda la red local.

#### 2.1. Configuración de fábrica

Por defecto, estos routers a veces traen contraseñas de administrador genéricas o piden establecer una muy básica durante el primer inicio. Además, cuentan con una opción que permite acceder a este mismo menú de administración de forma remota a través de internet.

#### 2.2. Principales riesgos

Mantener contraseñas débiles facilita enormemente los ataques de fuerza bruta. Si a esto le sumamos tener la gestión remota activada, estamos dejando la puerta del panel de control expuesta para que cualquiera en internet pueda localizarla e intentar autenticarse.

#### 2.3. Ajustes de bastionado

Para solucionar esto, dentro del panel principal fui a la pestaña **Advanced**, después abrí el menú lateral en **System** y seleccioné **Administration**.

Aquí se puede cambiar la contraseña y establecer una nueva clave de mayor longitud, usando combinaciones de números, letras mayúsculas y símbolos para evitar que pueda ser adivinada de forma sencilla.

<img width="1920" height="870" alt="image" src="https://github.com/user-attachments/assets/b5b301a9-24ba-479a-a63f-7dcbfd90dfde" />
<br>

En esa misma sección, bajando hacia la zona de "Remote Management", comprobé que la casilla de "Enable" estuviera completamente desmarcada. Con esto nos aseguramos de que el panel de control solo sea accesible estando ya físicamente dentro de la red local.

<img width="1920" height="865" alt="image" src="https://github.com/user-attachments/assets/3a3a84d4-3db5-4b17-9a1f-384854f357e8" />

### 3. Mantenimiento y actualizaciones del firmware

Mantener el sistema operativo del router actualizado es otro de los pilares de seguridad más críticos para evitar problemas a largo plazo.

#### 3.1. Configuración de fábrica

Lo más común al instalar estos dispositivos es dejarlos funcionando y olvidarse de ellos durante años. Esto provoca que el firmware se quede obsoleto rápidamente mientras se descubren continuamente fallos para esas versiones antiguas.

#### 3.2. Principales riesgos

Con el paso del tiempo se acaban haciendo públicas las vulnerabilidades de los firmwares en foros de seguridad. Si los atacantes detectan que nuestra versión está desactualizada, pueden explotar esos fallos conocidos y tomar el control del dispositivo, esquivando por completo cualquier contraseña que hayamos establecido antes.

#### 3.3. Ajustes de bastionado

Para no tener que estar pendiente de realizar parcheos manualmente de forma rutinaria, fui a la pestaña **Advanced**, panel lateral de **System** y entré en **Firmware Update**.

Allí activé directamente el botón de "Auto Update". En el campo "Update Time" programé que el proceso se realice siempre de madrugada, entre las 3:00 y las 5:00. De esta manera el router descargará e instalará automáticamente las versiones más seguras sin interrumpir los horarios de trabajo durante el día.

<img width="1920" height="870" alt="image" src="https://github.com/user-attachments/assets/5ff07294-f1aa-4f25-83bb-e6d1f82efdd9" />

### 4. Seguridad de red WiFi

La red inalámbrica es el vector de ataque más accesible ya que las ondas se disipan libremente hacia el exterior de las instalaciones.

#### 4.1. Configuración de fábrica

Los routers suelen venir preconfigurados con seguridad WPA2 y un sistema activo llamado WPS, pensado originalmente para facilitar la conexión rápida introduciendo un PIN numérico en vez de la clave completa.

#### 4.2. Principales riesgos

Actualmente, disponer solo de WPA2 hace que la red sea vulnerable a ciertas capturas de paquetes por el aire si la contraseña no es muy larga. Pero la mayor vulnerabilidad real reside en el protocolo WPS; los atacantes usan herramientas que lanzan intentos masivos para probar el PIN numérico y lo acaban averiguando en unos pocos minutos. Una vez adivinado, se saltan nuestra clave directamente.

#### 4.3. Ajustes de bastionado

Para blindar la red, pulsé la pestaña superior **Advanced**, abrí el desplegable izquierdo de **Wireless** y pinché en **Wireless Settings**.

En el apartado de seguridad, forcé la configuración al estándar mixto WPA2/WPA3-Personal. El protocolo WPA3 ofrece una encriptación mucho más robusta frente a los asaltos externos, y lo acompañé de una contraseña nueva, larga y con alta entropía.

<img width="1920" height="870" alt="image" src="https://github.com/user-attachments/assets/52032c56-2735-4560-9930-6fe97bf2b01f" />
<br>

_(Aviso de configuración guardada tras modificar las credenciales en ambas bandas de 2.4GHz y 5GHz)_

<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/ab333344-c5e8-4842-81d0-d302daa7c072" />
<br>

A continuación, en ese mismo menú izquierdo de Wireless, entré en la opción **WPS**. Modifiqué su interruptor principal dejándolo deshabilitado, eliminando del dispositivo esta antigua vía de acceso en segundo plano.

<img width="1920" height="867" alt="image" src="https://github.com/user-attachments/assets/40f7a100-8f87-44aa-b300-73b5dd78eab3" />

### 5. Aislamiento y red de invitados

Es una práctica común facilitar la contraseña principal de conexión a clientes o visitantes puntuales para que no gasten datos, lo cual rompe totalmente la barrera de seguridad interna.

#### 5.1. Configuración de fábrica

El dispositivo no obliga a usar redes secundarias. Si damos la contraseña normal, cualquier visitante externo que se conecte lo hará compartiendo el entorno local directo con los ordenadores de la plantilla, las impresoras y los discos duros de la compañía.

#### 5.2. Principales riesgos

Si el visitante viene usando un portátil o teléfono móvil infectado previamente de manera accidental, el malware presente en su dispositivo tendrá vía libre para rastrear ordenadores y propagarse por la red sin oposición alguna al ya encontrarse situado dentro del área de red de confianza.

#### 5.3. Ajustes de bastionado

La solución aplicada consistió en ir a la pestaña **Advanced**, desplegar la rama de **Wireless** y pinchar en **Guest Network** (Red de invitados).

Habilité el servicio para crear puntos de acceso paralelos ("Visitas_Empresa2026"). Pero lo realmente necesario para aislar la red fue ir a la sección inferior de permisos de invitados y dejar desmarcada en blanco sobre todo la casilla de "Allow guests to access your local network". Así es posible ofrecer libre acceso a internet mientras se bloquea del todo cualquier comunicación con el resto de terminales del trabajo.

<img width="1920" height="870" alt="image" src="https://github.com/user-attachments/assets/55b11145-62a1-46b7-a56a-6685674a8132" />

### 6. Servicios y protocolos innecesarios

Los aparatos intentan automatizar parte de sus procesos internos en favor de la usabilidad, diseñados para no dar dolores de cabeza de configuración.

#### 6.1. Configuración de fábrica

Estos modelos traen habitualmente activado de fábrica el protocolo UPnP (Universal Plug and Play), un servicio que permite a cualquier aplicación que esté en la red local ordenar sin trabas al router que se le abra una comunicación de puertos directos hacia el exterior.

#### 6.2. Principales riesgos

En un entorno ofimático, un equipo comprometido por algún ransomware o troyano podría aprovechar el automatismo del UPnP para solicitar al firewall que habilite inmediatamente descargas ocultas y conexiones salientes sin generar alertas limitantes que de otra manera saltarían para bloquear accesos remotos hacia él.

#### 6.3. Ajustes de bastionado

Para neutralizar este excesivo nivel de confianza, fui de nuevo por la parte superior a **Advanced**, localicé en el menú el listado **NAT Forwarding** y seleccioné **UPnP**.

En esa pantalla simplemente presioné su interruptor para desactivarlo permanentemente. Cualquier aplicación de la empresa que en el futuro requiera acceso exterior concreto tendrá que pasar por una apertura manual de puertos por requerimiento técnico.

<img width="1920" height="866" alt="image" src="https://github.com/user-attachments/assets/0db87c83-bb28-48b3-ad9c-88e4de13bfa5" />

### 7. Firewall y control perimetral

El cortafuegos no solo protege de asaltos dirigidos al local, sino que es el modo en el que la IP responde o no a quienes husmean constantemente.

#### 7.1. Configuración de fábrica

Todas las características de Firewall base bloquean invasiones estandarizadas. Sin embargo, permite generosamente que el router se comunique respondiendo a paquetes automatizados de ping llegados de IP externas ajenas intentando sondear si hay equipamiento activo en esa dirección.

#### 7.2. Principales riesgos

Hay scripts robóticos de rastreo en internet barriendo números probando ecos Ping todo el día. Si el router despacha su "Ping" respondiendo un mensaje de vida de vuelta, se le informará al exterior de nuestra actividad marcando el objetivo, y pasaríamos a ser víctimas de un empuje sostenido en intentos extraños en próximas horas solo por el hecho de haber llamado la atención.

#### 7.3. Ajustes de bastionado

Me centré en apaciguar nuestra respuesta frente a internet acudiendo a **Advanced**, pasando por **Security** y marcando la solapa de **Firewall**.

No toqué el valor primordial del "SPI Firewall", ni tampoco la conveniencia de que responda llamadas Ping internamente desde nuestros ordenadores encendidos a la hora de hacer mantenimiento. Pero eliminé de un plumazo marcando de color gris apagado la opción "Respond to pings from WAN". Como el cable WAN representa la salida exterior, el router ignorará con un silencio opaco cualquier sondeo masivo proveniente de internet pareciendo un dispositivo apagado.

<img width="1919" height="866" alt="image" src="https://github.com/user-attachments/assets/f92a0362-3f50-4aef-90eb-dac25e924477" />

### 8. Conclusión

Una vez repasadas las medidas, queda claro que utilizar modelos comerciales con la configuración que acompaña fuera de la caja suele apostar enteramente por una flexibilidad que atenta a la integridad general de su seguridad. Dar rienda suelta y dejar funcionando protocolos de asociación simple y agujereada como el WPS o la autoliquidación de puertos con el UPnP empaña todo el valor de cualquier contraseña por mucha fuerza en su formato que adquiera. 

Con un leve enfoque repasando el sistema sin mayor esfuerzo es muy viable rectificar a los estándares profesionales todo el sistema. Retirarnos de la red con desactivados perimetrales, asentar reglas a una red dual estancada para invitados impidiendo propagaciones de malware interno y reubicar las redes bajo la criptografía de WPA3 transforma eficientemente al TP-Link Archer AX55, limitándolo en vulnerabilidades ordinarias masivas resultando en un puesto defendido competente sin mayor problema si es que hay rigor configurativo por su parte.
