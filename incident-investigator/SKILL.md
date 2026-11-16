---
name: incident-investigator
description: Investiga y correlaciona registros CSV de autenticación o seguridad con evidencia trazable y secretos redactados; no se activa para análisis genérico de CSV, edición de logs ni creación de detectores.
---

# Incident Investigator

Investiga archivos CSV de autenticación y seguridad de forma prudente, reproducible y de solo lectura. Trabaja con los campos que realmente existan: no asumas un proveedor, esquema, zona horaria, umbral, incidente ni autoría.

## Activación

Activa este skill cuando la persona pida investigar, revisar, correlacionar o hacer triage de CSV de autenticación o seguridad: por ejemplo, inicios de sesión sospechosos, fallos repetidos, cambios de MFA, actividad de cuentas, accesos denegados o cambios de privilegios.

Ejemplos para comprobar el límite contextual:

- **Sí:** «Investiga este CSV de autenticación y señala patrones sospechosos con filas de evidencia».
- **Sí:** «Revisa los eventos de seguridad exportados en CSV y prepara una cronología».
- **No:** «Resume las respuestas de esta encuesta CSV» o «calcula el total de ventas».
- **No:** «Escribe un detector en Python para estos logs» cuando se pide únicamente desarrollar código, o «corrige/formatea este CSV».

Si una solicitud combina una investigación de logs con otra tarea, aplica este skill a la investigación y atiende la otra parte solo si está dentro del alcance y autorizada.

## Límites

- Lee los CSV sin modificarlos. No los normalices ni los guardes de vuelta; cualquier artefacto de informe, si se solicita, debe ser separado.
- No uses Internet, servicios externos ni subas los logs a otra herramienta. Analiza los datos disponibles localmente y los indicadores de referencia que aporte la persona.
- No inventes ataques, intenciones, identidades de atacantes ni atribuciones. Separa hechos observados de señales e hipótesis; explica qué datos faltan.
- Redacta todo secreto detectado antes de mostrar citas, extractos o conclusiones. Nunca reproduzcas filas crudas que puedan contener credenciales.
- Usa las herramientas existentes para inspeccionar y filtrar datos. No priorices programar un detector; considera un script auxiliar solo si aporta valor concreto y el usuario lo pide o autoriza.

## Flujo de investigación

1. **Delimita la fuente y la pregunta.** Usa los archivos indicados; si falta la ruta o el archivo no está disponible, pide ese dato. No inspecciones otros datos ajenos al alcance.
2. **Comprueba cómo se puede leer el CSV.** Identifica encabezados, delimitador, comillas, codificación, cantidad de registros, campos vacíos y filas malformadas. Puede haber CSV separados por coma, punto y coma o tabulador. No supongas nombres ni significado de columnas. Si hay documentación local del esquema, úsala; no adivines el proveedor.
3. **Mapea campos y límites.** Reconoce qué campos representan tiempo, resultado, usuario/actor, origen, recurso, autenticación, MFA y privilegios solo cuando los valores lo sustenten. Conserva la zona horaria original. No conviertas IP, user-agent, identificador de sesión o cuenta de servicio en identidad de una persona.
4. **Consulta las reglas detalladas.** Antes de etiquetar patrones o priorizar hallazgos, lee [references/detection-rules.md](references/detection-rules.md). Úsalo como guía condicional según las columnas y eventos disponibles; no como catálogo de ataques confirmados.
5. **Busca y correlaciona señales.** Usa filtros, tablas dinámicas, consultas o utilidades existentes disponibles. Anota criterios, periodo y conteos. Correlaciona solo por campos presentes y relaciones temporales defendibles. Conserva eventos repetidos hasta determinar si son duplicados; no descartes filas silenciosamente.
6. **Comprueba los hallazgos.** Para cada señal, vuelve a las filas de origen, valida los valores y las secuencias temporales, y considera explicaciones benignas como reintentos, NAT, escáneres autorizados, pruebas, cuentas de servicio, reloj desajustado o duplicación de ingestión cuando haya evidencia de ellas.
7. **Redacta una conclusión acotada.** Distingue hecho, señal e interpretación; expresa confianza y limitaciones; no conviertas ausencia de eventos en prueba de ausencia de incidente. Responde en el idioma de la persona.

## Formato de evidencias y respuesta

Incluye, según corresponda:

- **Alcance:** archivos revisados (nombre base), periodo registrado, número de registros y cualquier problema de lectura o cobertura.
- **Hallazgos:** señal concreta, evaluación/prioridad con su fundamento, nivel de confianza, interpretación limitada y explicaciones alternativas relevantes. Si no hay señal sustentada, dilo sin afirmar que no ocurrió un incidente.
- **Evidencia:** referencias estables como `archivo.csv, registro de datos 27` (número lógico después del encabezado), con los nombres de columnas y valores necesarios, redactados. Numera registros por separado para cada archivo. No cites solo un número de línea si no identifica inequívocamente el registro. Si una fila malformada no se puede interpretar como registro CSV, cita el nombre del archivo y la línea física o rango de líneas observado.
- **Cronología:** cuando la secuencia importe, incluye hora tal como aparece en el CSV y zona horaria conocida; marca explícitamente si la zona no está indicada.
- **Límites y siguientes pasos:** datos o validaciones adicionales que permitirían confirmar o descartar la interpretación.

Usa citas suficientes para que otra persona pueda localizar cada afirmación en el CSV original, sin copiar filas enteras. En todo valor que sea o pueda ser secreto (contraseña, token, clave, cookie, cabecera de autorización, secreto de MFA/recuperación, material de clave privada o credencial equivalente), sustituye el valor completo por `[REDACTADO: tipo]`. Si no puedes determinar si un valor opaco es secreto, redacta el valor completo. Minimiza también los datos personales no necesarios para verificar la señal.
