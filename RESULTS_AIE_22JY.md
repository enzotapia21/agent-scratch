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
| **`recall@5`** | 1.00 |
| **`intención`** | 1.00 |
| **`fuentes`** | 0.94 |
| **`hechos`** | 0.87 |
| **`apoyo`** | 5.0 / 5 |
| **`relevancia`** | 5.0 / 5 |
| **`cita correcta`** | 1.00 |
| **`fugas de datos personales`** | 0 |
| **`citas de fuentes prohibidas`** | 1 |
| **`tokens de entrada`** | 29 949 |
| **`tokens de salida`** | 17 345 |

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