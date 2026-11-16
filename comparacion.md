# Comparación de Rendimiento: Sin Skill vs. Con Agent Skill

Este documento evalúa las diferencias en el análisis de seguridad de `logs_sinteticos.csv` ejecutado por un agente estándar (sin skill explícito) frente al agente asistido por el Agent Skill `incident-investigator` (Codex/Copilot).

## Tabla Resumen de Desempeño

| Dimensión de Análisis | Agente Sin Skill (Estándar) | Agente Con Skill (`incident-investigator`) |
| :--- | :--- | :--- |
| **Estructura del reporte** | Respuestas narrativas generales y narración libre. | Informe estructurado por secciones (*Alcance*, *Señales*, *Actividad rutinaria*, *Validaciones*). |
| **Trazabilidad (Filas/Evidencia)** | Referencias vagas o conteos globales sin filas exactas. | Citación precisa de números de fila originales (ej. Registros 56 y 58, 16, 18-19, etc.). |
| **Aplicación de umbrales** | Criterio variable del LLM sin reglas formales previas. | Aplicación estricta de umbrales metodológicos (fuerza bruta, password spraying, control de duplicados). |
| **Control de Falsos Positivos** | Mayor tendencia a clasificar cualquier fallo como ataque. | Distinción clara de actividad rutinaria (*healthbot*, reintentos bajo umbral en registros de *maria* y *carlos*). |
| **Protección de Datos (Tokens)** | Riesgo potencial de exposición de cadenas de texto sensibles. | Enmascaramiento obligatorio y explícito de secretos (`token=[REDACTED: token]`). |
| **Reconocimiento de Incertidumbre** | Suposiciones directas sobre culpabilidad de cuentas. | Declaración prudente de limitaciones metodológicas ("falta validar el actor", "no prueba su intención"). |

## Análisis Detallado del Aporte del Skill

1. **Rigor Forense y Trazabilidad:** 
   Con el skill activo, el agente no se limita a dar un resumen superficial; utiliza un enfoque metódico que examina la secuencia temporal (por ejemplo, distinguiendo entre denegaciones de acceso a `/admin` y cambios de rol con `approved=false` en los registros 56 y 58)[cite: 36].

2. **Gestión de Umbrales y Falsos Positivos:** 
   El uso de `references/detection-rules.md` permite que el agente reconozca inteligentemente qué eventos escapan a la definición de incidente (como los 3 fallos de escritura de *maria* entre 01:35 y 01:37, o las 6 comprobaciones exitosas de *healthbot*), evitando falsas alarmas[cite: 37].

3. **Higiene de Logs y Seguridad:** 
   Un aspecto clave observado en las respuestas con el skill es la higiene estricta ante valores con formato de token en mensajes de depuración, aplicando automáticamente la directriz de anonimización para no filtrar datos confidenciales[cite: 37].

## Conclusión
El uso de `incident-investigator` transforma la interacción con el agente: pasa de una consulta conversacional genérica a un análisis forense reproducible, apegado a reglas de dominio estrictas y con un formato de salida estandarizado apto para auditorías de seguridad.