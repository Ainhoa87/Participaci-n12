# Respuestas: Plugin Detective

## 1. Skill frente a plugin
¿Por qué `incident-investigator` es un skill y qué otros componentes (comando, hook, MCP, manifiesto, según tu plataforma) podrían empaquetarse para distribuirlo como plugin?
- **Skill:** Es un componente especializado enfocado en proporcionar instrucciones metodológicas y recursos contextuales (`SKILL.md` y reglas) para guiar el razonamiento del agente en una tarea específica, sin conceder permisos autónomos por sí mismo.
- **Plugin:** Es una unidad de empaquetado y distribución más amplia que permite agrupar múltiples extensiones de manera conjunta, tales como comandos personalizados, hooks de automatización ante eventos del entorno, servidores MCP para estandarizar el acceso a capacidades externas y un manifiesto de configuración.

## 2. Ralph Loop
¿Cómo podría repetirse el análisis para corregir errores detectados por comprobaciones verificables, con máximo 3 iteraciones y un criterio de parada explícito, sin fabricar evidencia nueva?
- Se implementaría un bucle de ejecución controlada (patrón Ralph Loop) utilizando un comando y un hook de parada (*stop hook*) que intercepte los intentos de finalización para evaluar aserciones de éxito estrictamente sobre las filas originales del CSV.
- **Criterio de parada:** El ciclo se detiene de forma explícita al alcanzar un máximo de 3 iteraciones o cuando las validaciones se cumplan por completo (por ejemplo, comprobar que todas las alertas citen números de fila reales del registro y oculten tokens sensibles), sin inventar datos nuevos.s