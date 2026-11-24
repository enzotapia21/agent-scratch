# Resultados de experimentos — AIE-22JY

## Tamaño de chunk

| `INGEST_CHUNK_MAX_CHARS` | Chunks generados | Observación |
| ---: | ---: | --- |
| 1500 | 51 | Más contexto dentro de cada fragmento. |
| 250 | 66 | Fragmentos más enfocados y precisos, pero con menos contexto compartido. |

### Resultado y decisión

Con un máximo de **250 caracteres** se generaron **66 chunks**, frente a los **51 chunks** de la configuración base de **1500 caracteres**.

- **Qué se gana:** precisión en la recuperación, porque cada chunk tiende a tratar una sola regla o tema.
- **Qué se pierde:** contexto, porque una regla, su condición y su excepción pueden quedar en fragmentos distintos.
- **Observación del corpus:** al revisar `dev/show_chunks.py`, se vio que algunas secciones de `claims-regulation.pdf`, como “Abono provisional” y distintas reglas con sus excepciones, quedaron partidas en más de un chunk. También `response-times.csv` pasó a varios fragmentos pequeños.
- **Configuración restaurada:** se vuelve a construir y publicar el índice con el valor base del equipo, **1500 caracteres**, hasta medir el efecto con evaluaciones de preguntas reales.


## 1. Línea Base (S5)

### Métricas generales
| Métrica | Resultado |
| :--- | :---: |
| **`recall`** | 1.00 |
| **`intención`** | 1.00 |
| **`fuentes`** | 0.93 |
| **`hechos`** | 0.89 |
| **`apoyo`** | 5.0 / 5 |
| **`relevancia`** | 5.0 / 5 |
| **`cita correcta`** | 1.00 |
| **`fugas de datos personales`** | 0 |
| **`citas de fuentes prohibidas`** | 1 |

### Análisis de caso del Goldenset (`fraud-AIE_22JY`)
* **Pregunta:** *"¿Qué debo hacer si mi tarjeta queda retenida o bloqueada en un cajero?"*
* **Motivo de selección:** Es un incidente físico frecuente en atención al cliente que exige combinar protocolos de bloqueo de emergencia y canales de atención oficiales sin salirse del corpus vigente.
* **Desempeño del agente:**
  * **Aciertos:** Recuperó los documentos exactos (`recall_at_k: 1.0`), identificó correctamente la intención (`1.0`) y obtuvo la calificación máxima en relevancia y fundamentación (`5.0/5.0`), sin fugas de datos personales ni citas de fuentes prohibidas.
  * **Observación:** El evaluador marcó `facts_ok: false` debido a una validación estricta que esperaba una coincidencia literal del hecho clave; sin embargo, en la práctica, la respuesta generada fue completa, precisa y segura para el usuario.

---

## 2. Paso 10 · Antes y Después (S5)

### Comparativa de experimentos (31 casos)

| Experimento | Configuración | Recall | Intención | Fuentes | Hechos | Apoyo / Relevancia | Fuentes prohibidas | Tokens (Entrada / Salida) |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Chunks más chicos** | `INGEST_CHUNK_MAX_CHARS="800"` | 1.00 (`@5`) | 1.00 | 0.97 | 0.86 | 5.0 / 5.0 | 1 | 28,374 / 16,425 |
| **Menos fragmentos** | `SEARCH_TOP_K="3"` | 1.00 (`@3`) | 1.00 | 0.97 | 0.86 | 5.0 / 5.0 | **0** | **23,528 / 14,655** |
| **Más fragmentos** | `SEARCH_TOP_K="8"` | 1.00 (`@8`) | 1.00 | 0.94 | 0.82 | 5.0 / 5.0 | 1 | 35,394 / 17,475 |
| **Base local (sin reranker)** | `KNOWLEDGE_BASE="local"` | 0.96 (`@8`) | 1.00 | 0.94 | 0.93 | 5.0 / 5.0 | 1 | 35,095 / 16,651 |

### Configuración elegida y sustento
* **Configuración recomendada:** `SEARCH_TOP_K = "3"` (manteniendo el tamaño de chunk original y Azure AI Search con *reranker*).
* **Justificación técnica:**
  1. **Cero fuentes prohibidas:** Fue la única configuración que eliminó por completo las citas a documentos desactualizados o irrelevantes (`0`), al impedir que ingresen fragmentos de relleno al contexto del modelo.
  2. **Máxima precisión en búsqueda:** Mantuvo el `recall` en **1.00** (superando el `0.96` de la base local sin *reranker*), logrando `0.97` en fuentes y `5.0/5` en apoyo y relevancia.
  3. **Eficiencia en costos y latencia:** Redujo el consumo de tokens de entrada a **23,528** (hasta un 33% menos frente a `TOP_K=8`), haciendo al agente más rápido y económico sin comprometer la calidad de respuesta.

---

## 3. Comparativa de Jueces: Local vs. Azure Evaluation

| Métrica | Juez Local (30 casos) | Juez `azure_evaluation` (31 casos) | Diferencia / Observación |
| :--- | :---: | :---: | :--- |
| **`recall`** | 1.00 | 1.00 (`@8`) | Se mantiene perfecto en ambos |
| **`intención`** | 1.00 | 1.00 | Sin variaciones |
| **`fuentes`** | 0.93 | 0.94 | Ligera mejora (`+0.01`) |
| **`hechos`** | 0.89 | 0.86 | Leve caída (`-0.03`) por el caso adicional o mayor exigencia |
| **`apoyo`** | 5.0 / 5 | 5.0 / 5 | Consistente en puntuación máxima |
| **`relevancia`** | 5.0 / 5 | 4.9 / 5 | Azure Evaluation es ligeramente más estricto (`-0.1`) |
| **`cita correcta`** | 1.00 | 1.00 | Consistente en puntuación máxima |
| **`fugas de datos personales`** | 0 | 0 | Seguridad intacta |
| **`citas de fuentes prohibidas`** | 1 | 1 | Persiste 1 caso con fuente no permitida |
| **Tokens (Entrada / Salida)** | 27,567 / 15,636 | 35,053 / 16,160 | Mayor consumo por el caso extra y `recall@8` |

### Conclusión del análisis de jueces
* **¿Las notas de `apoyo` y de `cita correcta` van en la misma dirección?**
  **Sí, van completamente en la misma dirección.** En ambas evaluaciones tanto `apoyo` (`5.0/5`) como `cita correcta` (`1.00`) alcanzan el puntaje máximo. Esto confirma que ambos jueces coinciden en que el agente fundamenta sus respuestas estrictamente en los fragmentos recuperados y atribuye correctamente las fuentes sin alucinar información.


## Sesión 6 — Tools del MCP en el chat

Dominio: `fraud`. MCP: el mock (`http://127.0.0.1:8200/mcp`) y el compartido (`bank-mcp`). Ambos expusieron el mismo contrato para el dominio. Las conversaciones se probaron contra el mock para no modificar datos del servicio compartido; el agente utilizó la base de conocimiento de Azure AI Search (`idx-team-08`).

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock | `get_transactions` (consulta), `get_claim_status` (consulta), `block_card` (acción sensible), `file_claim` (acción sensible) |
| Compartido | `get_transactions` (consulta), `get_claim_status` (consulta), `block_card` (acción sensible), `file_claim` (acción sensible) |

### Conversación en `dev/chat.py` (paso 12)

| Prueba | Mensaje | Tool pedida (`→`) | Qué pasó | Primer texto / respuesta completa (ms) |
|---|---|---|---|---|
| Consulta de tu dominio | “Muéstrame mis últimos 2 movimientos. Mi identificador de cliente es CUS-1001.” | `get_transactions(customer_id=CUS-1001, limit=2)` | La tool se ejecutó correctamente y devolvió los dos movimientos más recientes. El agente respondió con los datos recibidos del MCP, sin completar valores por su cuenta. | 14 654 / 14 924 |
| Acción confirmada | “Confirmo que quiero bloquear la tarjeta CRD-2001 porque me la robaron.” | `block_card(card_id=CRD-2001, reason=robo)` | La acción no se ejecutó: quedó con estado `pending_approval`. El agente informó que la solicitud seguía pendiente de revisión por un asesor. | 23 729 / 23 995 |
| Sin tool que la resuelva | “¿Cuál es el saldo actual de mi cuenta CUS-1001?” | Ninguna | El agente indicó que no podía consultar saldos y no inventó un monto. Como alternativa, ofreció consultar los últimos movimientos. | 13 860 / 14 012 |
| Tool de otro dominio | “Consulta la línea de crédito disponible de mi tarjeta CRD-2001.” | Ninguna; `get_credit_line` no está expuesta a `fraud` | Clasificó la solicitud como `out_of_scope`, no intentó llamar una tool inexistente y derivó al cliente a canales de atención respaldados por el corpus. | 18 651 / 18 808 |

### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

No. Ante la consulta de saldo, el modelo reconoció que no disponía de una tool para obtenerlo y evitó proporcionar un monto. En la consulta de línea de crédito tampoco simuló el resultado de `get_credit_line`: respetó el límite del dominio y recomendó canales de atención citando el corpus. Los importes, fechas e identificadores mostrados en la prueba de movimientos provinieron de `get_transactions`.

### Conclusión

El contrato del mock coincide con el MCP compartido para `fraud`: ambos publican cuatro tools y aplican el filtro por dominio. Las tools de consulta se ejecutan y alimentan la respuesta con datos verificables, mientras que las acciones sensibles se interceptan antes de ejecutarse y quedan pendientes de aprobación humana. Cuando no existe una tool adecuada, o la tool pertenece a otro dominio, el agente no la invoca ni inventa datos. En estas pruebas, el primer texto llegó entre 13,9 y 23,7 segundos; la diferencia entre el primer texto y la respuesta completa fue pequeña (entre 152 y 270 ms), por lo que la mayor parte de la espera ocurrió antes de iniciar el streaming.


## Sesión 7 — Skills, memoria de trabajo y Agent Framework

Dominio: `fraud`. Se ejecutó la misma conversación de tres turnos con los runtimes `react` y `agent_framework`, usando el MCP mock para no modificar datos compartidos. Los tiempos se muestran en el orden **consulta / acción / memoria** y los tokens son el total acumulado de esos tres turnos.

| | `react` | `agent_framework` |
|---|---|---|
| Tools pedidas en la consulta | `get_transactions(customer_id=CUS-1001, limit=2)` | `get_transactions(customer_id=CUS-1001, limit=2)` |
| La acción sensible quedó **NO SE EJECUTA** | Sí. `block_card(card_id=CRD-2001, reason=robo)` quedó en `pending_approval`. | Sí. `block_card(card_id=CRD-2001, reason=robo)` quedó en `pending_approval`. |
| Pregunta de memoria respondida sin `→` | Sí. Respondió `-186.40 PEN` sin volver a consultar el MCP. | Sí. Respondió `-186.40 PEN` y `TRX-3001` sin volver a consultar el MCP. |
| Primer texto (ms) | 12 061 / 25 333 / 14 086 | 11 875 / 22 848 / 13 560 |
| Respuesta completa (ms) | 12 335 / 25 499 / 14 098 | 12 204 / 24 394 / 13 570 |
| Tokens (entrada / salida) | 10 387 / 4 931 | 10 517 / 3 931 |

La conversación utilizada fue:

1. “Muéstrame mis últimos 2 movimientos. Mi identificador de cliente es CUS-1001.”
2. “Confirmo que quiero bloquear la tarjeta asociada al último movimiento porque me la robaron.”
3. “¿Cuál fue el monto del último movimiento que consultaste?”

### ¿La skill frenó la tool de otro dominio?

Mensaje: “Consulta la línea de crédito disponible de la tarjeta CRD-2001.” La tool correspondiente, `get_credit_line`, pertenece a `cards` y no está en la lista blanca de la skill `fraud`. Ninguno de los runtimes mostró una llamada `→`: `react` clasificó el mensaje como `out_of_scope` y respondió que no podía consultar la línea; `agent_framework` lo clasificó como `data_query`, pero también indicó que no disponía de una herramienta para obtenerla y recomendó canales de atención. El filtro de dominio del MCP ya limita las tools visibles y la skill aplica una segunda restricción en el agente.

### Diferencias entre los dos runtimes

El runtime `react` implementa explícitamente el ciclo modelo → tool → resultado en el proyecto, mientras que `agent_framework` delega esa orquestación a Microsoft Agent Framework. La skill, la memoria de trabajo, la aprobación de acciones sensibles y el acceso al MCP permanecen en `ChatService`, por lo que las garantías funcionales fueron iguales en ambos. En esta ejecución, `agent_framework` produjo el primer texto antes en los tres turnos y consumió 1 000 tokens menos de salida, aunque utilizó 130 tokens más de entrada. También hubo una diferencia de clasificación para la consulta de línea de crédito (`out_of_scope` frente a `data_query`) sin cambiar el resultado seguro. Al tratarse de una sola ejecución por runtime, las diferencias de latencia no constituyen un benchmark concluyente.

### Conclusión

La skill de `fraud` limitó correctamente las tools disponibles y definió el criterio de terminación. La memoria de trabajo conservó el resultado de `get_transactions`, permitió inferir `CRD-2001` en el turno de bloqueo y contestar después el monto sin repetir la consulta. Ambos runtimes respetaron la barrera de aprobación: `block_card` se solicitó, pero no se ejecutó. Agent Framework puede sustituir el ciclo ReAct propio sin perder las reglas de seguridad ni el contexto conversacional; en esta prueba fue ligeramente más rápido y más breve en la salida.

## Semana 4 - Sesión 8 — Aprobación humana, validación y reintentos

Dominio: `fraud` (paso 6).

| Caso | Runtime | Mensaje | Qué pasó | Resultado o ID |
|---|---|---|---|---|
| Aprobar | `agent_framework` | "Sí, registra el reclamo por el cargo duplicado TRX-3002" | Se consultaron los movimientos, se pidió `file_claim` para `TRX-3002` y quedó pendiente de aprobación. El asesor aprobó con `s`; la tool se ejecutó y creó el reclamo. | Reclamo registrado **CLM-9001** para `TRX-3002` |
| Rechazar | `agent_framework` | "Sí, registra el reclamo por el cargo duplicado TRX-3002" | Se pidió `file_claim` para `TRX-3002` y quedó pendiente de aprobación. El asesor rechazó con `n`; la acción no se ejecutó. | **Solicitud rechazada**, sin ID de reclamo |
| Error de negocio (`CUS-9999`) | `agent_framework` | "Soy CUS-9999, me cobraron dos veces en el supermercado" | `get_transactions` devolvió `{"error": "El cliente CUS-9999 no existe"}`. El agente informó el error y pidió confirmar el código de cliente. | Error de negocio: cliente `CUS-9999` inexistente |
| Input inválido | `agent_framework` | `customer_id=ABCD-1001` en `get_transactions` | La validación del contrato rechazó el identificador antes de llamar al banco: no cumple el patrón `^CUS-\d{4}$`. | Validación fallida: `ABCD-1001` no es un `CUS-XXXX` válido |

### La trayectoria en el log

Durante el reintento del servicio compartido se observaron ambos estados:

```text
INFO agent.application.chat_service: trayectoria 1f6ea454-f0a5-466c-894c-350a1319c9f6: get_transactions error → respuesta data_query
INFO agent.application.chat_service: trayectoria 391e64a7-60b7-4d2b-910f-f9ab8ca16d57: get_transactions ok → respuesta data_query
```

En las pruebas de aprobación y rechazo no se registró ejecución de la acción mientras estaba pendiente:

```text
NO SE EJECUTA, pendiente de aprobación
resultado: {"id": "CLM-9001", "customer_id": "CUS-1001", "transaction_id": "TRX-3002", "status": "registered"}
rechazada por el asesor: {"status": "rechazada", "detail": "El asesor rechazó la acción: no se ejecutó."}
```

### ¿Algo se ejecutó sin que lo aprobaras?

No. `file_claim(CUS-1001, TRX-3002)` quedó en `pending_approval` y no se ejecutó de inmediato. Al aprobar con `s`, se creó `CLM-9001`; al rechazar con `n`, la tool devolvió `rechazada` y no creó ningún reclamo.

### Conclusión

El comportamiento de `agent_framework` fue correcto y seguro en esta ejecución. Las acciones sensibles quedaron pendientes hasta la decisión del asesor: la aprobación creó `CLM-9001` y el rechazo impidió la ejecución. `CUS-9999` produjo el error de negocio esperado y `ABCD-1001` fue rechazado por el patrón de validación `CUS-\d{4}` antes de contactar al banco. También se confirmó que un fallo temporal de `get_transactions` puede recuperarse al reintentarlo. No se observaron ejecuciones no autorizadas ni alucinaciones.
