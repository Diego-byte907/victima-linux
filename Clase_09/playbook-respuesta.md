# Playbook de Respuesta a Incidentes
## Caso: Fuerza Bruta contra Portal de Autenticación

### 1. Identificación

**Evento:** múltiples intentos de autenticación fallidos.

**Regla:** 100102

**Nivel:** 12 - Alta

**Técnica MITRE ATT&CK:** T1110.001 - Password Guessing

**Fuente:** portal-auth.log

**Decoder:** portal-auth

**Condición:** 5 intentos de autenticación fallidos desde una misma dirección IP en 60 segundos.

---

### 2. Triage inicial

Ante una alerta 100102:

1. Identificar la dirección IP origen.
2. Identificar el usuario afectado.
3. Revisar cantidad y frecuencia de los intentos.
4. Verificar si la alerta corresponde a un agente Windows o Linux.
5. Revisar si existen otras alertas relacionadas con la misma IP.
6. Determinar si existe evidencia de compromiso adicional.

---

### 3. Árbol de decisión

**¿Se detectaron 5 o más fallos en 60 segundos desde la misma IP?**

- NO → mantener monitoreo y registrar el evento.
- SÍ → continuar con la evaluación.

**¿La IP corresponde a una dirección autorizada o incluida en whitelist?**

- SÍ → no realizar contención automática y documentar la excepción.
- NO → continuar con la contención.

**¿Existe evidencia adicional de compromiso?**

- NO → mantener bloqueo temporal y continuar monitoreo.
- SÍ → escalar el incidente y preservar evidencias para análisis.

---

### 4. Acciones según severidad

**Baja**

- Registrar el evento.
- No realizar contención.
- Continuar monitoreo.

**Media**

- Realizar triage.
- Correlacionar eventos relacionados.
- Escalar a nivel superior si aparecen nuevos indicadores.

**Alta**

- Aplicar contención temporal.
- Preservar evidencias.
- Registrar IP, usuario, hora, regla y acción ejecutada.
- Escalar para análisis.

**Crítica**

- Aplicar las medidas autorizadas de contención.
- Preservar evidencias.
- Escalar inmediatamente a responsable de seguridad/gestión.
- Toda acción de mayor alcance requiere autorización explícita.


---

### 5. Contención automática

Para la regla 100102 se utiliza Active Response.

**Comando:** netsh

**Agente destino:** 002 - victima-windows

**Acción:** bloqueo temporal de la dirección IP origen.

**Duración:** 300 segundos.

**Regla activadora:** 100102.

La acción es reversible mediante el mecanismo de timeout configurado.

---

### 6. Preservación de evidencia

Registrar como mínimo:

- Fecha y hora de la alerta.
- Dirección IP origen.
- Usuario afectado.
- Regla Wazuh.
- Nivel de severidad.
- Técnica MITRE.
- Agente afectado.
- Acción de contención.
- Resultado de la contención.
- Eventos relacionados.

Conservar los registros de Wazuh y Windows necesarios para reconstruir la línea temporal.

---

### 7. Cierre y mejora

El incidente puede cerrarse cuando:

1. La actividad maliciosa haya cesado.
2. La contención haya sido verificada.
3. Las evidencias hayan sido preservadas.
4. Se haya registrado la línea temporal.
5. Se hayan calculado las métricas MTTD y MTTR.
6. Se hayan identificado mejoras para las reglas, playbook o automatizaciones.

Registrar cualquier falso positivo, excepción o ajuste realizado.

