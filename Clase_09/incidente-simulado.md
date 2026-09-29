# Incidente Simulado - Fuerza Bruta

## 1. Identificación

**Incidente:** Credential Storm - Fuerza Bruta contra Portal  
**Fecha:** 29/09/2026  
**IP origen:** 203.0.113.48  
**Usuario afectado:** testuser  
**Agente de respuesta:** 002 - victima-windows  
**Regla:** 100102  
**Nivel:** 12  
**MITRE ATT&CK:** T1110.001 - Password Guessing  

## 2. Línea temporal

| Hora | Evento |
|---|---|
| 18:11:03 | Se registra actividad bajo regla 100101. |
| 18:11:05.814 | Se activa regla 100102 por 5 fallos en 60 segundos. |
| 18:11:06.478 | Wazuh ejecuta Active Response mediante netsh en Windows 002. |
| 18:11:06 | Windows crea la regla de bloqueo para 203.0.113.48/32. |
| 18:16:07 | Wazuh ejecuta la eliminación de la regla de firewall. |

## 3. Clasificación

El evento se clasifica como **Alta**, debido a la detección confirmada de múltiples intentos de autenticación fallidos y al nivel 12 de la regla 100102.

La técnica MITRE asociada es **T1110.001 - Password Guessing**.

## 4. Triage

Se identificó:

- IP origen: 203.0.113.48
- Usuario: testuser
- Cinco o más intentos dentro de 60 segundos.
- Regla 100102 activada.
- No se utilizó una IP incluida en la whitelist.
- Se aplicó contención temporal.

## 5. Decisión de contención

De acuerdo con el playbook, al tratarse de una alerta de nivel alto se aplicó contención temporal.

La acción configurada fue:

- Active Response: netsh
- Agente: 002 - victima-windows
- Acción: bloquear IP origen
- Duración: 300 segundos

La creación del bloqueo fue verificada mediante los registros de Wazuh y Windows.

## 6. Resultado

La dirección 203.0.113.48 fue bloqueada mediante una regla de Windows Firewall.

Posteriormente, transcurridos aproximadamente 300 segundos, Wazuh ejecutó la acción de eliminación de la regla.

## 7. Métricas

**MTTD:** no se calcula desde el primer intento porque no existe en la evidencia disponible una marca temporal independiente del inicio del ataque.

**Detección de la condición de fuerza bruta:** 18:11:05.814.

**Tiempo de respuesta automática:** aproximadamente 0,664 segundos.

**Tiempo de contención:** aproximadamente 5 minutos.

**MTTR:** no se presenta como métrica completa porque no se dispone de una marca de inicio y fin de recuperación del incidente. Se registra en su lugar la duración de la contención automática.

## 8. Evidencias

- Regla Wazuh 100101.
- Regla Wazuh 100102.
- Active Response mediante netsh.
- Agente Windows 002.
- Evento Windows 4688.
- Regla de Windows Firewall.
- Eliminación automática del bloqueo.

## 9. Cierre y mejora

El incidente se considera contenido después de verificar la creación del bloqueo y su posterior eliminación automática.

Como mejora se mantiene la detección 100102 y el Active Response con alcance limitado al agente Windows 002, con timeout de 300 segundos.

