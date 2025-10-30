# Uso de RAG para Consultas sobre Normativa en Protección de Datos

## Índice
1. **Investigación sobre sanciones aplicables a la intrusión en servidores**
   - 1.1. Artículos relevantes (RGPD y LOPDGDD)
2. **Caso práctico**
   - 2.1. Descripción del delito
   - 2.2. Consecuencias penales
   - 2.3. Consecuencias administrativas
   - 2.4. Consecuencias civiles
   - 2.5. Resumen final

---

## 1. Investigación sobre sanciones aplicables a la intrusión en servidores que contienen datos personales

A continuación, se presenta una recopilación normativa obtenida mediante el uso de **NotebookLM** para consultas basadas en documentos legales:

| Artículo | Nombre | Descripción | Enlace |
|---|--------|-------------|--------|
| **Artículo 5 RGPD** | Principios relativos al tratamiento | Establece que los datos personales deben tratarse de forma que se garantice la **confidencialidad e integridad**, evitando accesos no autorizados. | https://www.privacy-regulation.eu/es/5.htm |
| **Artículo 5 LOPDGDD** | Deber de confidencialidad | Obliga a todos los intervinientes en el tratamiento a garantizar la confidencialidad de los datos personales. | https://www.iberley.es/legislacion/articulo-5-ley-organica-proteccion-datos-personales-garantia-derechos-digitales-lopdgdd |
| **Artículo 32 RGPD** | Seguridad del tratamiento | Establece la obligación de implementar medidas técnicas y organizativas adecuadas que garanticen la seguridad de los datos. | https://www.privacy-regulation.eu/es/32.htm |
| **Artículo 33 RGPD** | Notificación de violaciones de seguridad | Obliga a notificar a la autoridad competente en un máximo de **72 horas** tras detectar una brecha. | https://www.privacy-regulation.eu/es/33.htm |
| **Artículo 34 RGPD** | Comunicación de violaciones a los interesados | Obliga a comunicar la brecha a los afectados si existe riesgo significativo para sus derechos. | https://www.privacy-regulation.eu/es/34.htm |
| **Artículo 83 RGPD** | Régimen sancionador | Establece multas de hasta **20 millones €** o el **4 % del volumen de negocio global** según gravedad. | https://gdpr-text.com/es/read/article-83/ |

---

## 2. Caso Práctico

### 2.1. Situación del Delito
La empresa **ClínicaSalud S.L.** gestiona historiales médicos de pacientes en un servidor protegido.  
Un técnico informático, **Carlos**, accede de forma **no autorizada** utilizando credenciales internas.  
Copia **450 historiales médicos** y los entrega a una empresa competidora a cambio de dinero.

#### Conductas cometidas
- Acceso no autorizado a datos personales.
- Vulneración del deber de confidencialidad.
- Divulgación ilícita de datos personales especialmente protegidos (salud).
- La empresa no notifica la violación de seguridad a la AEPD dentro de 72 horas.

---

### 2.2. Consecuencias Penales (Código Penal Español)

| Artículo | Delito | Pena |
|---------|--------|------|
| **Art. 197.2 CP** | Acceso sin autorización a datos personales | **1 a 4 años de prisión** + **multa de 12 a 24 meses** |
| **Art. 197.3 CP** | Cesión o revelación de datos a terceros | **2 a 5 años de prisión** |
| **Art. 197.5 CP** | Tratamiento de datos especialmente protegidos (salud) | Pena en su **mitad superior** |

**Resultado para Carlos:** Riesgo de **hasta 7 años de prisión**.

---

### 2.3. Consecuencias Administrativas (RGPD y LOPDGDD)

| Normativa | Artículo | Infracción | Posible Sanción |
|----------|----------|------------|----------------|
| **RGPD** | Art. 32 | Falta de medidas de seguridad adecuadas | Hasta **10 millones €** o **2 %** del negocio anual |
| **RGPD** | Art. 33 | No notificar brecha en 72 horas | Hasta **2 %** del negocio anual |
| **LOPDGDD** | Art. 5 | Incumplimiento del deber de confidencialidad | **40.001 € – 300.000 €** |

**Resultado para la empresa:** Multas significativas + pérdida de confianza pública.

---

### 2.4. Consecuencias Civiles
Los pacientes afectados pueden exigir:
- **Indemnización económica por daños y perjuicios**.
- Responsabilidad patrimonial por mala gestión de sus datos personales.

---

### 2.5. Resumen Final

- **Carlos** puede recibir hasta **7 años de prisión** por acceso y filtración de datos.
- **La empresa** puede enfrentar **multas elevadas** por no garantizar la seguridad y no notificar la brecha.
- **Los pacientes** pueden reclamar **compensaciones económicas**.

---

