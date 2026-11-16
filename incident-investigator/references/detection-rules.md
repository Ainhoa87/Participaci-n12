# Reglas de revisión para CSV de autenticación y seguridad

Lee esta referencia **después de reconocer el esquema disponible y antes de clasificar hallazgos**. Aplica solo reglas respaldadas por columnas/eventos presentes; omite las demás. Estas reglas generan preguntas y señales de revisión, no confirman por sí solas un ataque. No uses Internet para enriquecer direcciones, dominios o indicadores.

## Lectura de los eventos

- Primero identifica los valores reales de tipo de evento y resultado. Determina si `success`, `failure`, `denied`, `challenged` u otros códigos tienen ese sentido a partir del esquema local o los propios datos. No infieras el significado de una columna desconocida.
- Determina si los tiempos incluyen zona horaria, si el reloj parece consistente y si las filas están ordenadas. Conserva siempre el valor de tiempo original. No combines eventos de zonas distintas sin una conversión justificada.
- Para secuencias, usa claves observadas (por ejemplo, cuenta, origen, dispositivo o recurso) solo si esas columnas existen y su significado es claro. Una dirección de origen compartida puede representar NAT, proxy, VPN, proveedor o varios usuarios.
- No impongas un umbral universal de conteo o ventana temporal. Prefiere umbrales definidos por la organización, un baseline del mismo conjunto comparable o una petición explícita. Si haces un corte exploratorio ad hoc, decláralo como criterio de análisis, explica por qué y no lo presentes como estándar de ataque.

## Señales a revisar

### Fallos y éxitos de autenticación

- Busca fallos repetidos agrupados por cuenta, origen, dispositivo y periodo, si existen esos campos. Comprueba la concentración, distribución, secuencia y resultados posteriores.
- Revisa si una secuencia de fallos es seguida por un éxito para la misma cuenta o desde un origen relacionado. Cita los registros de fallo **y** de éxito; limita la conclusión a la secuencia observada. Esto puede ser compatible con adivinación de credenciales, pero también con errores del usuario, reintentos o credenciales guardadas.
- Revisa una fuente con fallos en muchas cuentas y una cuenta con fallos desde muchas fuentes. Describe el patrón y los recuentos exactos antes de usar etiquetas como “spraying” o “fuerza bruta”. No llames ataque a un solo fallo ni atribuyas acciones a la dirección IP.
- No interpretes un éxito como sospechoso solo porque sea éxito, ni una ubicación distinta como “viaje imposible” sin datos de geolocalización fiables, tiempos comparables y contexto/base legítimos.

### MFA, recuperación y credenciales

- Revisa secuencias de retos MFA rechazados o expirados, éxitos posteriores, cambios de factor, desactivación, restablecimiento y uso de recuperación cuando dichos eventos estén registrados.
- Un cambio de factor o recuperación cerca de un inicio de sesión es una relación temporal, no prueba de toma de cuenta. Verifica actor, cuenta objetivo, resultado, tiempos y registros correlativos antes de describir el riesgo.
- No califiques intentos como “MFA fatigue” salvo que los eventos disponibles acrediten solicitudes repetidas y contexto suficiente. Si solo hay fallos MFA, informa exactamente eso.
- Trata contraseñas, secretos de MFA, códigos de recuperación, cookies y tokens como secretos. No los uses como citas visibles ni como claves de correlación que se muestren en el informe.

### Cuenta, acceso y privilegios

- Busca creación, activación, desactivación, bloqueo, desbloqueo, cambio de contraseña, restablecimiento, alta/baja de roles o permisos y cambios en políticas. Distingue actor, cuenta afectada, recurso, resultado y hora si están disponibles.
- Revisa la secuencia entre un cambio de acceso/privilegio y eventos de autenticación o acceso posteriores, solo cuando puedan vincularse con campos observados. Un evento administrativo no demuestra por sí mismo uso indebido ni escalada maliciosa.
- Los accesos denegados, bloqueos y revocaciones pueden ser controles que funcionaron. Describe impacto observado y cualquier éxito posterior; no infieras acceso efectivo a partir de una denegación.

### Sesiones, tokens y otros eventos de seguridad

- Si los registros incluyen creación, validación, renovación o revocación de sesión/token, revisa errores repetidos, revocación seguida de actividad, uso simultáneo o secuencias incompatibles solo cuando el esquema permita esa comparación.
- Nunca imprimas, cites ni incluyas en tablas el valor de un token, cookie, identificador de sesión reutilizable o secreto. Si se necesita correlación, hazla internamente y refiere las filas y los campos sin revelar el valor.
- Revisa eventos de política, acceso a recursos sensibles, cambios de configuración o alertas registradas por el sistema. Distingue alerta emitida, intento bloqueado y acción exitosa; no eleves un evento de “alerta” a compromiso confirmado sin evidencia adicional.

## Correlación, confianza y prioridad

- Una correlación requiere campos y relación observables: misma cuenta, actor/recurso, origen, dispositivo, identificador sensible correlacionado internamente o proximidad temporal explicada. Indica qué relación usaste y las filas incluidas.
- Considera como explicaciones alternativas solo posibilidades pertinentes al patrón: NAT/proxy/VPN, servicios y trabajos automatizados, escáneres autorizados, pruebas, reintentos de cliente, desfase de reloj y duplicados de ingestión. No las afirmes como causa sin evidencia.
- **Confianza alta:** varios campos consistentes o eventos correlativos sustentan el patrón y las filas fuente lo muestran claramente. **Media:** el patrón está en los datos, pero falta contexto importante o una clave de correlación. **Baja:** señal aislada o ambigua. La confianza mide el soporte de la interpretación, no la gravedad.
- Si el usuario u organización define severidad, úsala y explica la correspondencia. Si no la define, puedes expresar prioridad descriptiva (informativa/baja/media/alta) según el acceso o cambio observado, el resultado y la solidez de la evidencia. No uses “crítica” ni declares incidente confirmado sin criterios organizacionales o corroboración clara.
- Los conteos deben poder reproducirse: informa el periodo y las dimensiones de agrupación, cita evidencia representativa y, cuando el número total sea importante, cuenta todos los eventos relevantes. No ocultes excepciones ni descartes duplicados sin indicarlo.

## Redacción y evidencia

Por cada hallazgo, separa:

1. **Hecho:** qué evento/campo indica el CSV, con conteo y periodo si aplica.
2. **Señal o interpretación:** qué podría sugerir, en lenguaje condicional y con confianza.
3. **Límite:** qué no puede establecerse a partir de estos registros y qué explicaciones siguen abiertas.
4. **Evidencia:** nombre del archivo, número lógico del registro después del encabezado y campos/valores mínimos necesarios, con todos los secretos redactados.

Usa la forma `[archivo.csv, registro de datos 27]`; cuenta registros lógicos por archivo desde 1 después de la fila de encabezado. Si el CSV contiene campos multilínea, no confundas número de registro con línea física. Para un registro malformado que el parser no pueda delimitar, cita la línea física/rango, señala el problema y no inventes un número de registro lógico. Una señal sin filas que la sostengan no es un hallazgo reportable.
