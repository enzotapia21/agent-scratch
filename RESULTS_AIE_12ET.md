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

## Sesión 7 — Skills, memoria de trabajo y Agent Framework

Dominio: `fraud`. La misma conversación con los dos runtimes (paso 7).

| | `react` | `agent_framework` |
|---|---|---|
| Tools pedidas en la consulta | get_transactions(customer_id=CUS-1001, limit=10) | get_transactions(customer_id=CUS-1001, limit=10) |
| La acción sensible quedó **NO SE EJECUTA** | block_card(card_id=CRD-2001, reason=fraude) NO SE EJECUTA, pendiente de aprobación | block_card(card_id=CRD-2001, reason=fraude) NO SE EJECUTA, pendiente de aprobación |
| Pregunta de memoria respondida sin `→` |  Sí. Agente [data_query]: Los movimientos duplicados son TRX-3001 y TRX-3002: ambos del 2026-10-20, SUPERMERCADO DEMO SAN ISIDRO, S/186.40, con la tarjeta CRD-2001. | Sí. Agente [data_query]: Los movimientos duplicados son TRX-3001 y TRX-3002: SUPERMERCADO DEMO SAN ISIDRO, fecha 2026-10-20, -186.40 PEN cada uno, con la tarjeta CRD-2001. |
| Primer texto (ms) | 16159 ms | 19858 ms |
| Respuesta completa (ms) | 17616 ms | 21259 ms |
| Tokens (entrada / salida) | 3032 / 1116 | 3440 / 1051 |

La conversación utilizada fue:
1. Soy CUS-1001, me cobraron dos veces en el supermercado
2. Sí, registra el reclamo por el cargo duplicado
3. ¿Cuál es el saldo de mi cuenta de ahorros?
4. ¿Qué movimientos estaban duplicados?

### ¿La skill frenó la tool de otro dominio?

Mensaje: Consulta la línea de crédito disponible de la tarjeta CRD-2001. Respuesta: Agente [out_of_scope]: Eso está fuera del alcance de este servicio. Para consultar la línea de crédito disponible usa la aplicación móvil o llama a la línea de atención al cliente, o acude a una agencia. Si necesitas ayuda con bloqueo, movimientos no reconocidos o reclamos, con gusto te atiendo.
Sí frenó la tool de otro dominio y esto se evidencia en la respuesta del agente como `out_of_scope`. Para este caso se uso el dominio de `cards`

### Diferencias entre los dos runtimes

Runtime: react (Razonamiento y repregunta)
El agente no ejecuta ninguna herramienta, sino que se detiene únicamente a responder con texto pidiendo más confirmaciones. 
El enfoque react prefiere volver a conversar, dudar y pedirle al usuario que le reconfirme los detalles antes de dar el siguiente paso.

Runtime: agent_framework (Acción directa)
El agente ejecuta las herramientas de inmediato en cuanto recibe la orden.
Intenta procesar la petición de golpe llamando a las herramientas (el sistema luego las frena por ser acciones sensibles, pero el agente fue directamente a la acción).

### Conclusión

agent_framework: es mucho más directo y pragmático: ante la instrucción de registrar el reclamo, intenta invocar las herramientas de inmediato de forma agrupada (file_claim para ambos cargos a la vez)
react:	 es más conservador y repetitivo: prefiere detenerse en la capa de razonamiento para volver a preguntar y pedir confirmaciones detalladas antes de decidirse a ejecutar la herramienta.

Adicionalmente, en ambos casos, la seguridad funciona de la misma manera: cuando se intenta una acción sensible (como bloquear una tarjeta o registrar un reclamo), el sistema la frena y devuelve el estado de pendiente_aprobacion
La diferencia está en que agent_framework llega a la barrera de seguridad de forma más rápida, mientras que react a veces alarga la conversación con preguntas antes de llegar al punto de ejecución.

Finalmente, ambos recuerdan perfectamente el contexto de la conversación.

## Sesión 8 — Aprobación humana, validación y reintentos

Dominio: `fraud` (paso 6).

| Caso | Runtime | Mensaje | Qué pasó | Resultado o ID |
|------|---------|---------|----------|----------------|
| Aprobar | `agent_framework` | Sí, registra el reclamo por el cargo duplicado | Se solicitó al asesor la aprobación del registro. Asesor indico `s` | Agente informo que el reclamo quedo registrado con código CLM-9001 |
| Rechazar | `react` | Sí, registra el reclamo por el cargo duplicado | Se solicitó al asesor la aprobación del registro. Asesor indico `n` | Agente informo que el asesor rechazó la solicitud de reclamo por TRX-3002; la acción no se ejecutó. |
| Error de negocio (`CUS-9999`) | `react` | Soy CUS-9999, me cobraron dos veces en el supermercado | Se generó el error El cliente CUS-9999 no existe | Agente informo No encuentro el cliente CUS-9999 en nuestro sistema. Confirma por favor tu código de cliente (ej. CUS-1001) |
| Input inválido | `react` | Mi número de cliente es 1001, ¿qué comisiones me cobraron? | Se generó un error de formato | Agente informo Para ver las comisiones debo consultar tus movimientos. Confirma tu código de cliente con el formato CUS-1001 (no envíes números de tarjeta ni claves) |

### La trayectoria en el log

| Trayectoria |
|-------------|
| INFO agent.application.chat_service: trayectoria 9ffc841d-22bd-4ca4-898e-c97fd5457c8a: get_transactions ok → respuesta sensitive_action |
| INFO agent.application.chat_service: trayectoria 9ffc841d-22bd-4ca4-898e-c97fd5457c8a: file_claim pending_approval → respuesta sensitive_action |
| INFO agent.application.chat_service: trayectoria 9ffc841d-22bd-4ca4-898e-c97fd5457c8a: decisión del asesor → file_claim ok |
| INFO agent.application.chat_service: trayectoria 1d06b9a5-6e33-4486-a73c-fe35304de411: get_transactions ok → respuesta data_query |
| INFO agent.application.chat_service: trayectoria 1d06b9a5-6e33-4486-a73c-fe35304de411: sin tools → respuesta sensitive_action |
| INFO agent.application.chat_service: trayectoria 1d06b9a5-6e33-4486-a73c-fe35304de411: sin tools → respuesta sensitive_action |
| INFO agent.application.chat_service: trayectoria 1d06b9a5-6e33-4486-a73c-fe35304de411: file_claim pending_approval → respuesta sensitive_action |
| INFO agent.application.chat_service: trayectoria 1d06b9a5-6e33-4486-a73c-fe35304de411: decisión del asesor → file_claim rejected |
| INFO agent.application.chat_service: trayectoria fd9e0d39-1bff-46fc-8cc8-690d8fda3c2a: get_transactions error → respuesta data_query |
| INFO agent.application.chat_service: trayectoria de999a58-b655-4581-9b39-f71e50310765: sin tools → respuesta data_query |

### ¿Algo se ejecutó sin que lo aprobaras?

No. En todos los escenarios con acciones sensibles (file_claim), el sistema solicitó confirmación y dejó la operación pendiente de aprobación humana. La acción solo se ejecutó cuando fue aprobada explícitamente por el asesor y fue rechazada cuando el asesor la denegó.

### Conclusión

El comportamiento fue correcto y seguro. Tanto Agent Framework como ReAct respetaron el flujo de aprobación para acciones sensibles, gestionaron adecuadamente errores de negocio (cliente inexistente) y validaciones de entrada (formato de cliente inválido). No se observaron alucinaciones ni ejecuciones no autorizadas; todas las respuestas estuvieron basadas en los resultados de las herramientas y en las decisiones de aprobación humana.


## Sesión 9 — Despliegue en Azure

Equipo: `team-08`. Dominio: `fraud`.

| Comprobación | Resultado |
|---|---|
| Tests en el contenedor (paso 3) | docker run --rm agent:test -> 139 passed, 5 skipped in 12.60s |
| Acción aprobada después de reiniciar `uvicorn` (paso 2, Cosmos DB) | error: {"error": "El servicio del banco no responde; intenta de nuevo en unos minutos"} - Agente [sensitive_action]: La solicitud de reclamo por TRX-3002 quedó registrada para que el banco la procese; no se aplicó. No la volveré a pedir. |
| Imagen desplegada (`team-NN/agent:<sha>`) | team-08/agent:d1a882b7ed13df622a4c9859a8de6a4640fa8ab7 |
| Revisión activa y duración del job `deploy` | 2m 59s |
| `/health` del agente desplegado | https://ca-agent-chat-08.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/ |
| Respuesta con `Fuentes:` contra Azure (y su tiempo en ms) | ¿Cuál es el plazo para presentar un reclamo? / respuesta completa: 65109 ms - ¿y si es por fraude? / respuesta completa: 15189 ms |
| Acción aprobada contra Azure, con su `resultado:` |  Asesor: ¿apruebas file_claim(customer_id=CUS-1001, transaction_id=TRX-3002, reason=Cargo duplicado en supermercado; solicito reclamo por TRX-3002.)? [s/n]: s -> Agente [sensitive_action]: El reclamo por TRX-3002 fue registrado con código CLM-9001; se abrió el 2026-10-28 y la fecha límite de respuesta es el 2026-11-27. La solicitud queda registrada para que el banco la procese; no está aplicada. |

### Qué pone Azure y qué pone la imagen

Lo que pone la imagen / contenedor: El código de la aplicación del agente (por ejemplo FastAPI/Uvicorn y la lógica del agente), sus dependencias empaquetadas y los valores por defecto o configuraciones no sensibles definidos en el `Dockerfile` / código (como el dominio o parámetros base de la app).
Lo que pone Azure (Container App / entorno / identidad): Las variables de entorno y endpoints inyectados en la Container App (como los endpoints/configuración de Foundry, AI Search, MCP y Cosmos DB), además de la identidad administrada (*Managed Identity*) del agente y la autenticación/tokens de Entra ID (`az login` desde el cliente para enviar el token HTTP). *(No se requiere `.env` en local al consumir el agente desplegado por HTTP).

### Problemas que encontraste y cómo los resolviste

Remoto `upstream` no configurado: Al correr los comandos de Git para traer el pipeline y `devops`, falló porque solo existía `origin`. Se resolvió agregando el remoto `upstream` apuntando al repositorio base/template, haciendo `git fetch upstream` y extrayendo `.github/workflows/deploy-agent.yaml` y `devops`.
Motor de Docker apagado en Windows: Al intentar hacer `docker build`, apareció error al conectar con el daemon de Docker. Se resolvió iniciando Docker Desktop / verificando el servicio antes de construir la imagen.
Consulta de autenticación en Container App (`{}`):** Al ejecutar `az containerapp auth show` sobre `ca-agent-chat-08`, devolvió `{}` porque la autenticación de Entra ID aún no estaba activa en la Container App; se verificaron los registros de app en Entra ID (`az ad app list`) para ubicar el `appId` correspondiente (`app-workshop-agent-chat` y `app-workshop-ci-team-08`).

### Conclusión

Desplegar el agente en una Container App separa claramente el código empaquetado en la imagen de contenedor frente a la configuración de entorno, secretos/endpoints e identidad administrada que gestiona Azure. Para consumir el agente ya desplegado desde la laptop basta autenticarse con `az login` y comunicarse por HTTP, mientras el agente usa su identidad administrada para conectarse a Foundry, AI Search, MCP y Cosmos DB sin depender de un archivo `.env` local.