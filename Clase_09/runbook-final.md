# Runbook Final - SOC / Capstone

## 1. Objetivo

Este documento consolida la configuración, detecciones, respuesta a incidentes, inteligencia, métricas y evidencias desarrolladas durante el Capstone.

---

## 2. Arquitectura y despliegue

### Componentes principales

| Componente | Función |
|---|---|
| Wazuh Manager | Gestión centralizada, análisis y correlación |
| victima-linux | Endpoint Linux monitoreado |
| victima-windows | Endpoint Windows monitoreado |
| Wazuh Dashboard | Visualización y análisis |
| Docker | Plataforma utilizada para el despliegue del Manager |

### Agentes

| ID | Nombre | Estado |
|---:|---|---|
| 001 | victima-linux | Active |
| 002 | victima-windows | Active |

El Manager corresponde al agente local `000`.

---

## 3. Fuentes de telemetría

### Linux

- Auditd
- SSH
- Archivos de registro del sistema

### Windows

- Windows Event Channel
- Evento de creación de procesos 4688
- Telemetría del agente Wazuh
- Active Response

### Portal

- `/var/ossec/logs/portal-auth.log`
- Decoder `portal-auth`

---

## 4. Detección de autenticación

### Regla 100100

Detecta intentos de autenticación fallidos provenientes del decoder `portal-auth`.

**Nivel:** 5

### Regla 100101

Detecta múltiples intentos de autenticación fallidos.

**Nivel:** 8  
**MITRE:** T1110 - Brute Force

### Regla 100102

Detecta fuerza bruta cuando se producen 5 fallos desde una misma IP dentro de 60 segundos.

**Nivel:** 12  
**MITRE:** T1110.001 - Password Guessing

---

## 5. Respuesta automática

La regla `100102` activa Active Response.

**Comando:** `netsh`

**Destino:** agente `002 - victima-windows`

**Acción:** bloqueo temporal de la IP origen.

**Timeout:** 300 segundos.

La acción fue validada experimentalmente mediante la creación de una regla Windows Firewall para la IP `203.0.113.48`.

---


---

## 6. Detecciones y cobertura

### Detección 1 - Autenticación fallida

**Regla:** 100100  
**Decoder:** portal-auth  
**Descripción:** identifica intentos de autenticación fallidos.

**Posibles falsos positivos:** errores legítimos de contraseña.

**Excepción:** usuarios o fuentes autorizadas pueden ser tratados mediante whitelist.

---

### Detección 2 - Múltiples fallos de autenticación

**Regla:** 100101  
**Nivel:** 8  
**MITRE:** T1110

**Lógica:** identifica múltiples intentos de autenticación fallidos.

**Posibles falsos positivos:** usuario legítimo que ingresa repetidamente una contraseña incorrecta.

---

### Detección 3 - Fuerza bruta

**Regla:** 100102  
**Nivel:** 12  
**MITRE:** T1110.001

**Lógica:** 5 fallos de autenticación desde la misma IP dentro de 60 segundos.

**Respuesta:** Active Response mediante `netsh`.

---

### Detección 4 - Creación de procesos Windows

**Evento:** Windows Security Event ID 4688.

**Objetivo:** registrar la creación de nuevos procesos en el endpoint Windows.

**Uso:** permite investigar procesos ejecutados durante un incidente y reconstruir actividad.

---

### Detección 5 - Active Response

**Regla relacionada:** 100102

**Evento generado:** regla 657 de Active Response.

**Objetivo:** registrar la ejecución de la respuesta automática en el agente Windows.

---

## 7. Vulnerabilidades

El análisis de vulnerabilidades forma parte del Capstone y debe mantenerse documentado junto con los hallazgos priorizados.

Para cada vulnerabilidad se debe registrar:

- CVE
- Producto o componente afectado
- Severidad
- Evidencia
- Impacto
- Recomendación de mitigación
- Estado de corrección

Los hallazgos deben priorizarse considerando criticidad y exposición del activo.

---

## 8. Inteligencia MISP

La inteligencia utilizada durante el Capstone debe registrarse indicando:

- Indicador observado.
- Tipo de indicador.
- Fuente.
- Relación con el incidente.
- Técnica ATT&CK asociada cuando corresponda.
- Acción tomada.

Los indicadores relevantes deben correlacionarse con las alertas de Wazuh cuando sea posible.

---

## 9. Playbook de respuesta

El procedimiento de respuesta se encuentra documentado en:

`/var/ossec/bitacora/clase09/playbook-respuesta.md`

El playbook contiene:

- Identificación.
- Triage.
- Árbol de decisión.
- Acciones por severidad.
- Contención automática.
- Preservación de evidencia.
- Cierre y mejora.

---

## 10. Incidente simulado

El incidente simulado se encuentra documentado en:

`/var/ossec/bitacora/clase09/incidente-simulado.md`

### Caso

**Credential Storm - Fuerza Bruta**

**IP:** 203.0.113.48  
**Usuario:** testuser  
**Regla:** 100102  
**MITRE:** T1110.001

La alerta fue detectada y posteriormente se ejecutó Active Response sobre el agente Windows 002.

La respuesta creó una regla de firewall temporal para bloquear la dirección IP origen.

---

## 11. Métricas SOC

Las métricas se encuentran documentadas en:

`/var/ossec/bitacora/clase09/metricas.md`

### Resumen

**Alertas registradas:** 1053

**Agentes de endpoint esperados:** 2

**Agentes activos:** 2

**Cobertura:** 100%

### Técnicas MITRE observadas

- T1110: 27
- T1484: 8
- T1078: 7
- T1531: 6
- T1021: 2
- T1110.001: 2
- T1562.001: 1

---

## 12. Métricas del incidente

**Detección de la condición de fuerza bruta:** 18:11:05.814

**Respuesta automática:** aproximadamente 0,664 segundos después.

**Duración de contención:** aproximadamente 5 minutos.

El MTTD y MTTR completos no se calculan cuando no existen marcas temporales suficientes para determinar el inicio y recuperación completa del incidente.

---


---

## 13. Evidencias de validación

### Active Response

La regla 100102 fue validada mediante una prueba real.

Se utilizaron cinco eventos de autenticación fallida desde:

203.0.113.48

La regla 100102 se activó y Wazuh ejecutó netsh.exe sobre el agente Windows 002.

### Windows Firewall

Se verificó en Windows la existencia de la regla:

WAZUH ACTIVE RESPONSE BLOCKED IP

con dirección remota:

203.0.113.48

### Reversibilidad

Después del timeout configurado de 300 segundos, Wazuh ejecutó la eliminación de la regla de firewall.

---

## 14. Procedimiento de reproducción

Para reproducir la prueba se generan cinco eventos de autenticación fallida desde una misma dirección IP.

Posteriormente se verifica:

- Activación de la regla 100102.
- Ejecución de Active Response.
- Creación de la regla de firewall en Windows.
- Eliminación automática después del timeout.

---

## 15. Archivos generados

Los archivos principales de la Clase 9 son:

- playbook-respuesta.md
- incidente-simulado.md
- metricas.md
- runbook-final.md

Ubicación:

/var/ossec/bitacora/clase09/

---

## 16. Cierre del Capstone

Se validó el flujo completo de detección y respuesta:

Evento -> Decoder -> Regla -> Severidad -> Active Response -> Contención -> Verificación -> Liberación automática

La detección de fuerza bruta utiliza la regla 100102 y la técnica MITRE T1110.001.

La respuesta automática se encuentra limitada al agente Windows 002 y utiliza un timeout de 300 segundos, permitiendo una contención temporal y reversible.

Las evidencias y métricas obtenidas durante las pruebas quedan registradas para su utilización en la documentación y defensa del Capstone.

---

## 17. Referencias y documentación

- Documentación oficial de Wazuh.
- MITRE ATT&CK.
- Documentación utilizada durante los laboratorios del Capstone.
- Evidencias generadas en el entorno de laboratorio.

