# Archivo 1: CodeLab_Bastionado_BIOS_UEFI.md

---

# CodeLab: Bastionar la BIOS/UEFI (Lenovo Legion 7-16ACHg6 - simulador)

**Autor:** JCA

**Fecha:** 2025-11-11

**Resumen:** Guía paso a paso para hardening (bastionado) de la BIOS/UEFI de un equipo Lenovo Legion 7-16ACHg6 usando el simulador oficial de Lenovo. Está pensada como un tutorial amigable que puedas ejecutar en el simulador y luego aplicar en hardware real con pequeños cambios según el fabricante.

---

## Objetivos

- Incrementar la seguridad del proceso de arranque (BIOS/UEFI).
- Configurar contraseñas adecuadas (administrador y usuario / Power-On).
- Evitar arranques desde dispositivos externos.
- Establecer un orden de arranque seguro.
- Activar Secure Boot y TPM (si procede).
- Documentar la configuración y explicar mitigaciones contra Clear CMOS.

---

## Requisitos previos

1. Cuenta en GitHub (para publicar la guía en GitHub Pages).
2. Acceso al simulador de Lenovo: https://download.lenovo.com/bsco/index.html#/ (navegador con JavaScript habilitado).
3. Un navegador moderno (Chrome/Edge/Firefox).
4. Opcional pero recomendado: copiar/pegar estas instrucciones en un editor Markdown y probarlas en el simulador.

---

## Nota importante sobre el simulador

El simulador de Lenovo es una herramienta web interactiva. Si al abrir la página ves el mensaje "Please enable JavaScript to use the website", habilita JavaScript y recarga la página. En el simulador deberás buscar el modelo **Legion 7-16ACHg6 (82N6)** o un modelo equivalente si no aparece exactamente.

Si el simulador no carga correctamente en tu navegador por restriccones, prueba otro navegador o limpia caché.

---

## Estructura de la guía (qué harás)

1. Abrir el simulador y seleccionar el modelo.
2. Navegar por los menús de UEFI/BIOS (Security / Boot / Authentication / Startup / Exit) del simulador.
3. Configurar contraseñas: Supervisor (administrador) y User / Power-On.
4. Deshabilitar arranque desde USB/externos.
5. Establecer orden de arranque: disco interno primero.
6. Habilitar Secure Boot y comprobar TPM (Security Chip).
7. Guardar/Documentar la configuración.
8. Medidas físicas y recomendaciones adicionales.

---

## Paso a paso (usar en el simulador)

> **Atención:** las etiquetas de menú pueden variar ligeramente entre modelos y versiones de firmware. Si un nombre de opción no coincide, busca una opción con significado similar (por ejemplo "Supervisor Password" puede aparecer como "Admin Password").

### 1) Abrir el simulador y seleccionar modelo

1. Abre: `https://download.lenovo.com/bsco/index.html#/`
2. En la página del simulador, usa el buscador de modelos (si existe) y escribe: **Legion 7-16ACHg6** o el código **82N6**.
3. Selecciona el equipo en la lista. El simulador cargará una interfaz que emula la UEFI/BIOS del equipo.

> Si el simulador no muestra exactamente tu modelo, selecciona un modelo "Legion" o "Gaming" similar: los menús de seguridad suelen ser equivalentes.

### 2) Entrar al menú principal de la UEFI/BIOS

1. En la interfaz del simulador, haz clic en el área que representa la pantalla del BIOS/UEFI.
2. Localiza las pestañas: **Main**, **Security**, **Boot**, **Exit** (o **Startup** / **Authentication** según firmware).

### 3) Establecer contraseña de administrador (Supervisor / Admin Password)

1. Ve a la pestaña **Security** o **Authentication**.
2. Busca la opción **Set Supervisor Password**, **Set Admin Password** o similar.
3. Haz clic y escribe una contraseña robusta. Recomendaciones de contraseña:
   - Mínimo 12 caracteres.
   - Mezcla de mayúsculas, minúsculas, números y símbolos.
   - No uses contraseñas vinculadas a ti (fecha de nacimiento, nombre de mascota).
4. Confirma la contraseña cuando te lo pida.

**Efecto:** Con esta contraseña se solicita autenticación para cambiar parámetros de la BIOS.

### 4) Establecer contraseña de arranque (Power-On / HDD / User Password)

1. En la misma pestaña **Security** o en **Boot**, busca **Set User Password**, **Power-On Password** o **HDD Password**.
2. Si tu objetivo es pedir contraseña al encender el equipo, configura la **Power-On Password** (a menudo aparece como "Power-On Password" o "HDD Password").
3. Escribe una contraseña diferente o igual según tu política (recomendado: diferente de la admin). Anota su ubicación en tu gestor de contraseñas.

**Consideraciones:**
- Si pones Power-On en un servidor/servicio que se reinicia automáticamente (o WoL), ten en cuenta que el equipo no arrancará hasta introducir dicha contraseña manualmente.

### 5) Deshabilitar arranque desde dispositivos externos (USB, PXE, CD)

1. Ve a la pestaña **Boot** o **Startup**.
2. Busca opciones tipo **Boot from USB**, **External Device Boot**, **Boot from Removable Devices**, **Network Boot (PXE)**.
3. Cambia el valor a **Disabled** para cada dispositivo externo que quieras bloquear (habitualmente: USB, Optical, Network/PXE).

**Si quieres permitir temporalmente un arranque externo:** deja la opción habilitada pero usa la contraseña de BIOS para evitar que cualquiera lo haga; o cambia el orden de arranque solo cuando necesites.

### 6) Establecer orden de arranque seguro (Boot Priority)

1. En **Boot** -> **Boot Priority** / **Boot Order**, selecciona el disco interno (NVMe/SSD/HDD) como **First Boot Device**.
2. Mueve unidades extraíbles (USB/Optical) y Network Boot al final o deshabilítalas por completo.
3. Si aparece **UEFI: <nombre disco>** y **Legacy/CSM**, selecciona la entrada UEFI del disco.

**Nota:** Si el sistema tiene arranque dual o particiones especiales, documenta cualquier cambio para poder revertirlo.

### 7) Habilitar Secure Boot

1. En la pestaña **Security** o **Boot**, busca **Secure Boot**.
2. Pon **Secure Boot = Enabled**.
3. Establece el modo a **Standard** o **Windows UEFI mode** si aparece.
4. Si la opción pide seleccionar una clave o plataforma, normalmente **Standard (Microsoft)** es correcto para Windows; para Linux puede requerir importación de claves o usar modo "Custom" y añadir claves de la distro.

**Verificación en Windows:** más tarde podrás comprobar desde msinfo32 que Secure Boot está "On".

### 8) Habilitar TPM / Security Chip

1. Busca en **Security** una opción llamada **Security Chip**, **TPM**, **PTT** (Intel) o **fTPM** (AMD) o similar.
2. Actívalo. En sistemas AMD modernos aparecen como **fTPM**; en Intel se llama **Intel PTT**.
3. Guarda los cambios.

**Importante:** Una vez activado el TPM y usado por un sistema operativo para cifrar discos (BitLocker, LUKS+clevis, etc.), desactivarlo puede ocasionar pérdida de acceso a las claves cifradas si no se hizo copia de seguridad.

### 9) Desactivar Legacy/CSM (Compatibility Support Module)

1. En **Boot** busca **CSM Support**, **Legacy Boot** o similar.
2. Establece **CSM = Disabled** para forzar UEFI puro. Esto mejora la seguridad y facilita Secure Boot.

**Precaución:** Si tu sistema depende de drivers/OS antiguos, prueba antes.

### 10) Otras opciones de seguridad recomendadas

- **Disable USB ports at boot**: Si existe la opción para inhabilitar puertos USB durante el POST, actívala.
- **Enable Intel AMT / Management Engine protections** (si aplica): Deshabilita o restringe acceso remoto de administración si no usas dicha función.
- **Disable Boot from Removable Device Without Supervisor**: Algunas BIOS permiten que solo el supervisor cambie la prioridad de arranque.
- **Fast Boot**: Activarlo acelera el arranque, pero puede saltarse comprobaciones — valora su impacto.

### 11) Guardar cambios y salir

1. Ve a **Exit** y selecciona **Save Changes and Exit**.
2. El simulador te mostrará un resumen — acepta.

### 12) Documentar y hacer copia de seguridad de la configuración

1. Anota todas las opciones modificadas en un fichero de texto (o en este CodeLab) con fecha.
2. Si el firmware soporta exportar configuración a USB (poco común), exporta la configuración y guarda el archivo en un lugar seguro.
3. Crea una copia de seguridad de tus contraseñas en un gestor (ej. Bitwarden, KeePassXC).

### 13) Mitigación frente a Clear CMOS / reset físico

- **Medida física:** Cierra el chasis con tornillos y/o candado, usa sellos de garantía y coloca el equipo en un lugar con control de acceso.
- **Etiquetas y registro:** Lleva registro de quién tiene acceso físico al equipo.
- **Cifrado del disco:** Configura cifrado completo (BitLocker con TPM+PIN en Windows, LUKS con TPM/seal o passphrase para Linux). Si un atacante extrae el disco, estará cifrado.
- **Protección de la contraseña de BIOS:** Algunos portátiles guardan el Power-On en memoria no volátil (ejemplo histórico Sony Vaio). No dependas solo de la BIOS: combina con cifrado y control físico.

### 14) Validación final

1. Reinicia el equipo (o usa el simulador) y comprueba que al entrar en BIOS se solicita **Supervisor Password** para cambiar parámetros.
2. Apaga y vuelve a encender para comprobar si se solicita **Power-On Password**.
3. Comprueba que no es posible arrancar desde USB (o que prioridad está en el disco interno).
4. En caso de Windows, abre `msinfo32` y verifica: "Estado de arranque seguro: On" y "Compatibilidad con cifrado de dispositivo" (TPM).

---

## Checklist (entregable para evaluación)

- [ ] Explicación y pasos para configurar contraseña de arranque (Power-On) ✅
- [ ] Explicación y pasos para configurar contraseña de administrador (Supervisor) ✅
- [ ] Explicación para evitar arranques externos ✅
- [ ] Explicación para establecer orden de arranque seguro ✅
- [ ] Otras opciones de seguridad (Secure Boot, TPM, CSM) ✅
- [ ] Documentación de mitigaciones frente a Clear CMOS y copia de seguridad de la configuración ✅

> Usa esta checklist para rellenar el formulario de evaluación y para comprobar los puntos del criterio 6a y 6c.

---

## Anexos: ejemplo de texto que puedes pegar en el README.md del repositorio

> Este repositorio contiene la guía CodeLab para bastionar la BIOS/UEFI de un Lenovo Legion 7-16ACHg6. Incluye instrucciones paso a paso para configurar contraseñas, Secure Boot, TPM y deshabilitar arranque desde dispositivos externos. Publicado en GitHub Pages: (pegar enlace aquí).

---

## Buenas prácticas finales

- Documenta la política de contraseñas (rotación y custodio).
- Guarda la documentación del equipo (modelo, número de serie) en el README.
- Prueba la recuperación: comprueba el comportamiento cuando olvidas la contraseña y valida las herramientas de recuperación documentadas por el fabricante.
- Mantén el firmware actualizado — revisa periódicamente las actualizaciones de BIOS/UEFI en la web del fabricante.

---

# Archivo 2: README.md (contenido que debes incluir en el README de tu ZIP)

```
# README — Guía BIOS/UEFI para Lenovo Legion 7-16ACHg6

Esta entrega contiene:

- `CodeLab_Bastionado_BIOS_UEFI.md` — Guía paso a paso para hardening de la BIOS/UEFI usando el simulador de Lenovo.

Publicación en GitHub Pages:
- Enlace a la guía publicada: https://<tu-usuario>.github.io/<nombre-repo>/  <- sustituye por tu enlace definitivo.

Instrucciones rápidas para publicar en GitHub Pages:
1. Crea un nuevo repositorio en GitHub llamado `hardening-bios` (por ejemplo).
2. Añade `CodeLab_Bastionado_BIOS_UEFI.md` al repositorio en la rama `main`.
3. Ve a `Settings` -> `Pages` -> `Source` y selecciona la rama `main` y carpeta `/ (root)`.
4. Pulsa `Save`. GitHub generará la página en `https://<tu-usuario>.github.io/<nombre-repo>/`.
5. Comprueba que la página carga y comparte la URL en Moodle.

Contenido del .zip entregable:
- `CodeLab_Bastionado_BIOS_UEFI.md`
- `README.md` (este fichero)

Autor: JCA
Fecha: 2025-11-11
```

---

## Siguientes pasos para ti (rápido y práctico)

1. Abre el simulador Lenovo y practica cada paso del apartado "Paso a paso".
2. Copia este Markdown a un archivo llamado `CodeLab_Bastionado_BIOS_UEFI.md`.
3. Crea un repositorio en GitHub, sube ese archivo y sigue las instrucciones del README para publicar.
4. Comprime (`zip`) los dos archivos: `CodeLab_Bastionado_BIOS_UEFI.md` y `README.md` y súbelos a Moodle en la tarea Proyecto 1.1.

---

**Fin del CodeLab**

