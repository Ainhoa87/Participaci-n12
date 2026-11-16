## Alcance y método

Se revisó únicamente `logs_sinteticos.csv`, sin modificarlo ni consultar las direcciones IP. El CSV tiene 93 registros de datos (94 filas físicas incluido el encabezado), los seis campos esperados (`timestamp`, `ip`, `user`, `event`, `result`, `detail`) y ningún campo vacío. Sus marcas de tiempo ya están ordenadas y usan el offset `-05:00`; el archivo no especifica una zona horaria nominal. El periodo cubierto es `2026-11-05T01:01:00-05:00` a `2026-11-05T02:29:00-05:00`.

Hay 84 eventos `login` (56 `success`, 28 `failed`), además de 6 `healthcheck/success`, un `debug/warning`, un `access/denied` y un `role_change/success`. Se aplicaron los criterios indicados para el ejercicio: coincidencia exacta de `event` y `result`, ventanas inclusivas de hasta 5 minutos y números de fila del CSV original (incluido el encabezado como fila 1). Son alertas basadas en registros sintéticos, no conclusiones forenses.

## Alertas justificadas

| Alerta | Prioridad orientativa | Regla y observación | Tiempo, IP y usuario | Evidencia y limitaciones |
|---|---|---|---|---|
| Fallos repetidos seguidos de inicio exitoso | Alta | 8 filas `login/failed` para `admin` desde la misma IP entre `01:14` y `01:18`, superan el umbral de 6 en 5 minutos. Un `login/success` desde esa IP para `admin` ocurre a `01:19`, un minuto después del último fallo y dentro de los 5 minutos siguientes. | `2026-11-05T01:14:00-05:00`–`01:19:00-05:00`; `198.51.100.77`; `admin`. | Fallos: filas 17, 19–20, 22–23 y 25–27; éxito: fila 28. Hay 8 filas de fallos, pero 3 pares comparten todos los valores y marca de tiempo: 19–20, 22–23 y 25–26. No se puede determinar si son reintentos o duplicados de ingesta; si se colapsaran por esos campos, quedarían 5 combinaciones únicas y no alcanzarían el umbral. El patrón bruto cumple la regla, pero la interpretación de adivinación de credenciales requiere validar duplicación, autorización y auditoría de la cuenta. |
| Fallos dirigidos a varias cuentas desde un origen | Media | 6 filas `login/failed` desde la misma IP contra 6 usuarios distintos dentro de 2 minutos, cumpliendo el criterio de password spraying probable. No hay `login/success` de ese origen en el archivo. | `2026-11-05T02:01:00-05:00`–`02:03:00-05:00`; `203.0.113.89`; usuarios distintos. | Filas 67–68, 70–71 y 73–74. Se observa el patrón requerido; no prueba un ataque ni identifica a una persona. Pruebas autorizadas o un origen compartido son explicaciones que habría que descartar con contexto adicional. |
| Cambio de rol marcado sin aprobación | Alta | `role_change/success` con `approved=false`, exactamente el criterio de escalada no autorizada según el registro. | `2026-11-05T01:50:00-05:00`; `203.0.113.42`; `guest`. | Fila 59. Un acceso a `/admin` fue denegado desde la misma IP y usuario un minuto antes (fila 57); la denegación no demuestra acceso. El CSV no aclara quién es el actor ni a qué cuenta se aplicó el cambio, ni corrobora el estado efectivo del rol. No se atribuye culpabilidad. |
| Posible token expuesto en depuración | Media | El campo `detail` contiene `token=`; se activa la regla de token potencialmente expuesto. | `2026-11-05T01:27:00-05:00`; `192.0.2.44`; `api_service`. | Fila 33: `event=debug`, `result=warning`, `detail=token=[REDACTADO: token]; debug output`. El valor se omite; el registro no permite saber si era una credencial válida o utilizable. |

## Actividad normal o que no alcanza los criterios

- **Tres fallos y luego éxito:** para `maria` desde `192.0.2.25` hay tres `login/failed` entre `01:35` y `01:37` y un `login/success` a `01:38` (filas 38–39, 41–42). Aunque los fallos comparten usuario y origen y el éxito llega un minuto después del último, tres no alcanzan el umbral de seis; no se clasifica como fuerza bruta. El detalle dice `mistyped password` y `legitimate user`, pero esos textos por sí solos no verifican la legitimidad.
- **Monitoreo:** los seis `healthcheck/success` de `healthbot` entre `02:10` y `02:15` (filas 78, 80–84) son eventos de salud programados. No son `login/failed` y no cuentan para los umbrales de autenticación.
- **Acceso denegado:** la solicitud de `guest` a `/admin` dio `access/denied` a `01:49` (fila 57). Se registra un intento bloqueado, no acceso exitoso; se informa por separado del cambio de rol registrado al minuto siguiente.
- **Otros fallos aislados:** los fallos individuales o con menor cantidad de intentos no satisfacen por sí solos los criterios de fuerza bruta o spraying definidos para este análisis.

## Validaciones recomendadas

1. Revisar la fuente de los registros de `admin` (filas 17–28) para establecer si las filas repetidas son duplicados y confirmar la legitimidad del éxito posterior.
2. Correlacionar el cambio de rol (fila 59) con una auditoría que identifique actor, cuenta afectada, aprobación y estado efectivo.
3. Confirmar si los seis fallos contra cuentas distintas (filas 67–74) proceden de pruebas autorizadas o de un origen compartido.
4. Verificar en la fuente de depuración si el valor omitido de la fila 33 era un secreto válido; si lo era, evaluar su revocación o rotación.
