##  Línea base (S5)

| Task | Resultado 
|---|---
| `recall` | 1.00
| `intención` | 1.00
| `fuentes` | 0.93
| `hechos` | 0.89
| `apoyo` | 5.0/5
| `relevancia` | 5.0/5
| `cita correcta` | 1.0
| `fugas de datos personales` | 0la
| `citas de fuentes prohibidas` | 1

### Caso del Goldenset (`fraud-AIE12ET`)

* **Pregunta:** `¿Qué debo hacer si mi tarjeta queda retenida o bloqueada en un cajero?`
* **¿Por qué lo elegí?** Es un incidente físico muy común en la atención al cliente que requiere combinar procedimientos de bloqueo de emergencia y canales de atención sin salir del corpus actual.
* **¿Cómo le fue al agente?** 
  * **Aciertos:** Recuperó los documentos exactos (`recall_at_k: 1.0`), identificó bien la intención (`1.0`), obtuvo la nota máxima en relevancia y fundamentación (`5.0/5.0`), sin fugas de datos ni fuentes prohibidas.
  * **Observación:** Marcó `facts_ok: false` debido a que el juez estricto esperaba una coincidencia literal del hecho clave, aunque en la práctica la respuesta generada fue completa, precisa y segura para el usuario.

## Paso 10 · Antes y después (S5)

### Comparativa de experimentos (31 casos)

| Experimento | Configuración | Recall | Intención | Fuentes | Hechos | Apoyo / Relevancia | Fuentes prohibidas | Tokens (Entrada / Salida) |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Chunks más chicos** | `INGEST_CHUNK_MAX_CHARS="800"` | 1.00 (`@5`) | 1.00 | 0.97 | 0.86 | 5.0 / 5.0 | 1 | 28,374 / 16,425 |
| **Menos fragmentos** | `SEARCH_TOP_K="3"` | 1.00 (`@3`) | 1.00 | 0.97 | 0.86 | 5.0 / 5.0 | **0** | **23,528 / 14,655** |
| **Más fragmentos** | `SEARCH_TOP_K="8"` | 1.00 (`@8`) | 1.00 | 0.94 | 0.82 | 5.0 / 5.0 | 1 | 35,394 / 17,475 |
| **Base local (sin reranker)** | `KNOWLEDGE_BASE="local"` | 0.96 (`@8`) | 1.00 | 0.94 | 0.93 | 5.0 / 5.0 | 1 | 35,095 / 16,651 |

### Configuración elegida y sustento

* **Configuración recomendada:** `SEARCH_TOP_K = "3"` (manteniendo el tamaño de chunk original y Azure AI Search con reranker).
* **¿Por qué la elegimos?**
  1. **Cero fuentes prohibidas:** Fue la única configuración que eliminó por completo las citas de documentos desactualizados o irrelevantes (`0`), al evitar que entren fragmentos de relleno en el contexto.
  2. **Máxima precisión en búsqueda:** Mantuvo el `recall` en **1.00** (superando al 0.96 de la base local sin reranker), logrando `0.97` en fuentes y `5.0/5` en apoyo y relevancia.
  3. **Eficiencia en costos y latencia:** Redujo el consumo de tokens de entrada a **23,528** (hasta un 33% menos frente a `TOP_K=8`), haciendo al agente más rápido y económico sin perder calidad de respuesta.

  ## Comparativa de Jueces: Local vs. Azure Evaluation

| Métrica | Juez Local (Línea base - 30 casos) | Juez `azure_evaluation` (31 casos) | Diferencia / Observación |
| :--- | :---: | :---: | :--- |
| **`recall`** | 1.00 | 1.00 (`@8`) | Se mantiene perfecto en ambos |
| **`intención`** | 1.00 | 1.00 | Sin variaciones |
| **`fuentes`** | 0.93 | 0.94 | Ligera mejora (+0.01) |
| **`hechos`** | 0.89 | 0.86 | Leve caída (-0.03) por el caso adicional o mayor exigencia |
| **`apoyo`** | 5.0 / 5 | 5.0 / 5 | Consistente en puntuación máxima |
| **`relevancia`** | 5.0 / 5 | 4.9 / 5 | Azure Evaluation es ligeramente más estricto (-0.1) |
| **`cita correcta`** | 1.0 | 1.0 | Consistente en puntuación máxima |
| **`fugas de datos personales`** | 0 | 0 | Seguridad intacta |
| **`citas de fuentes prohibidas`** | 1 | 1 | Persiste 1 caso con fuente no permitida |
| **Tokens (Entrada / Salida)** | 27,567 / 15,636 | 35,053 / 16,160 | Mayor consumo por el caso extra y `recall@8` |

---

### Análisis y respuestas

* **¿Las notas de `apoyo` y de `cita correcta` van en la misma dirección?**
  **Sí, van completamente en la misma dirección.** En ambas corridas tanto `apoyo` (`5.0/5`) como `cita correcta` (`1.0`) alcanzan el puntaje máximo[cite: 7]. Esto confirma que ambos jueces coinciden en que el agente fundamenta sus respuestas estrictamente en los fragmentos recuperados y atribuye correctamente las fuentes sin inventar información[cite: 7].

## Sesión 6 — Tools del MCP en el chat

Dominio: `fraud`. MCP: el mock (`http://127.0.0.1:8200/mcp`) y, si ya tienes `MCP_URL`, el compartido (`bank-mcp`).

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock | block_card (acción sensible), file_claim (acción sensible), get_claim_status (consulta), get_transactions (consulta) |
| Compartido | block_card (acción sensible), file_claim (acción sensible), get_claim_status (consulta), get_transactions (consulta) |

### Conversación en `dev/chat.py` (paso 12)

| Prueba | Mensaje | Tool pedida (`→`) | Qué pasó | Primer texto / respuesta completa (ms) |
|---|---|---|---|---|
| Consulta de tu dominio | Soy CUS-1001, me cobraron dos veces en el supermercado. | get_transactions(customer_id=CUS-1001, limit=10) | Invocación correcta de la Tool MCP. El agente interpretó de forma precisa la intención y decidió llamar a la herramienta correspondiente proporcionada por el servidor MCP, pasando los argumentos correctos. Adicionalmente, brindo una respuesta conversacional clara y orientada al cliente. | 27720 ms / 28060 ms |
| Acción confirmada | Sí, registra el reclamo por el cargo duplicado | file_claim (customer_id=CUS-1001, transaction_id=TRX-3001, TRX-3002) | El agente intentó registrar el reclamo por cargos duplicados, pero por seguridad quedó pendiente de aprobación humana, informando pasos y fuentes documentales. | 38623 ms / 38915 ms |
| Sin tool que la resuelva | ¿Cuál es el saldo de mi cuenta de ahorros? | out_of_scope | El agente indica que la consulta esta fuera del alcance de este servicio y no invento un monto. Dio la opción de ver saldos en diferentes canales. | 7095 ms / 7269 ms |
| Tool de otro dominio | ¿Cuánto pagaría al mes por un préstamo de 20000 soles a 24 meses? | out_of_scope | El agente indica que la consulta esta fuera del alcance de este servicio. Dio la opción de simular cuotas de un prestamo en diferentes canales. | 16181 ms / 16573 ms |

### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

No. Respondió correctamente sin invención de datos. Respeto los limítes de sus dominio. Adicionalmente, ante preguntas fuera de su alcance o sensibles, dio alternativas de donde verlos.

### Conclusión

El agente respeta estrictamente los límites de su dominio, clasificando correctamente las consultas propias (fraudes) y externas (saldos y préstamos) como out_of_scope. Esto evita alucinaciones o cálculos no autorizados, manteniendo una buena experiencia al redirigir al usuario hacia los canales oficiales del banco. Adicionalmente, reconoció que ciertos procedimientos requiere de confirmación de aprobación humana. 