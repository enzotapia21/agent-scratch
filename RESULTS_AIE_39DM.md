## Semana 5 - Sesión 10 — Trazas por turno y dashboard

Equipo: `team-08`. App Insights: `appi-team-08`. Los cuatro casos contra el agente desplegado (paso 8).

| Caso | operation_Id | Latencia (ms) | Tokens de entrada | Tokens de salida | Tools | Error | Observaciones |
|---|---|---:|---:|---:|---|---|---|
| Saludo | `0ee50fddcb383e75b3124d9aa08cfef4` | 3286 | 1479 | 176 | — | No | Latencia baja; todo el tiempo corresponde al procesamiento del agente, sin dependencias externas. |
| Consulta con retrieval | `153853b7d1b6940e00ce6d3b4a748762` | 8916 | 1656 | 963 | — | No | La consulta de conocimiento y las fuentes citadas explican el incremento de latencia frente al saludo. |
| Llamada a tool | `c1f8b4bebb1f113c3fc955494f4f6ed1` | 20245 | 3962 | 2224 | `get_transactions:ok` | No | La tool respondió correctamente en 87 ms; la mayor parte del tiempo se consumió en la orquestación y elaboración de la respuesta. |
| Error controlado | `c7baf7dea7729f446b376d6702bcce86` | 160 | *(no visible)* | *(no visible)* | — | Sí (`not_found`) | El error está claramente identificado en la traza por `success=False` y `error=not_found`; falló antes de invocar cualquier tool. |

En **Observaciones**, una línea: lo que llamó la atención de la traza (qué se llevó el tiempo, la dependencia que falló, si el
error permite ubicar el turno). En **Tools**, `—` si el turno no llamó ninguna.

### Lo que dice el workbook

| Panel | Lo que viste |
|---|---|
| Latencia p95 por intención | **sensitive_action** fue la intención más lenta (p95 = **30.128 ms**), seguida por **out_of_scope** (11.934 ms), **product_question** (8.469 ms) y **greeting** (3.094 ms). |
| Tokens por turno | Se procesaron **16 turnos**, con **38,4 mil tokens de entrada** y **18,7 mil tokens de salida**; los flujos con retrieval y acciones sensibles concentraron el mayor consumo. |
| Dónde se va el tiempo de un turno | La mayor parte del tiempo se consume en **Foundry - chat** (144.990 ms acumulados, p95 = 11.691 ms). Embeddings, AI Search y Cosmos DB aportan una fracción pequeña de la latencia total. |
| Tools con error | Se observó **1 error en `get_transactions`** (estado `error`, p95 = 10.075 ms). Además, hubo un **chat.decision** con `error=not_found` (`operation_Id = c7baf7dea7729f446b376d6702bcce86`). |

### ¿Qué optimizarías primero y por qué?

Optimizaría primero los flujos sensitive_action, porque concentran la mayor latencia (p95 de 30 s) y representan los casos más críticos para el usuario. También revisaría los fallos de get_transactions, ya que cuando la dependencia falla añade más de 10 s de espera.

### Conclusión

El agente mostró un comportamiento estable: 16 turnos (acciones de compañeros), 1 error registrado y trazabilidad completa de tools y aprobaciones HITL. El principal consumo de tiempo se concentra en Foundry Chat y en las intenciones sensitive_action, mientras que las tools tienen tiempos de ejecución bajos cuando responden correctamente.


## Semana 5 -Sesión 9 — Despliegue en Azure

Equipo: `team-08`. Dominio: `fraud`.

| Comprobación | Resultado |
|---|---|
| Tests en el contenedor (paso 3) | 139 passed, 5 skipped in 5.92s |
| Acción aprobada después de reiniciar `uvicorn` (paso 2, Cosmos DB) |   resultado: {"id": "CLM-9002", "customer_id": "CUS-1001", "transaction_id": "TRX-3002", "reason": "Cargo duplicado en supermercado - solicito reclamo por TRX-3002", "status  Agente [sensitive_action]: Su reclamo quedó registrado con código CLM-9002 por TRX-3002; se abrió el 2026-10-28 y la fecha límite de respuesta es 2026-11-27. La solicitud está registrada para que el banco la procese, no aplicada. |
| Imagen desplegada (`team-NN/agent:<sha>`) | Run az acr login --name "${ACR_LOGIN_SERVER%%. Login Succeeded The push refers to repository [acrworkshopjsu3.azurecr.io/team-08/agent]881fc932939e: Waiting d1a882b7ed13df622a4c9859a8de6a4640fa8ab7: digest: sha256:c9b6a27c5ebae8052316bae59e259d93b7aa6496ba4be2b235529e3532912132 size: 856|
| Revisión activa y duración del job `deploy` | deploy succeeded 4 hours ago in 2m 59s |
| `/health` del agente desplegado | (agent) PS E:\IA\git\agent-scratch> uv run python dev/chat.py Agente en https://ca-agent-chat-08.icybush-2ea2dfc1.eastus2.azurecontainerapps.io: ok Conversación con el agente. Escribe "nueva" para otra sesión o "salir" para terminar. |
| Respuesta con `Fuentes:` contra Azure (y su tiempo en ms) | Tú: Soy CUS-1001, me cobraron dos veces en el supermercado Agente [sensitive_action]: Intenté consultar tus movimientos pero el servicio del banco no respondió. Para abrir el reclamo necesito que confirmes cuál es el cargo: envía el código TRX-... que aparece en la app o, si no lo tienes, dime la fecha y el monto exacto del doble cobro en el supermercado. No compartas números completos de tarjeta ni claves. Cuando confirmes el movimiento, procedo a registrar el reclamo; la acción quedará pendiente de aprobación por un asesor. Nota: el reclamo por doble cobro (contracargo) se puede presentar hasta 60 días desde la operación. Fuentes: [1] service-channels.html · Canales de atención para fraude y reclamos > Aplicación móvil [2] unrecognized-charges-procedure.md · Procedimiento ante cargos no reconocidos > Paso 2: revisar los movimientos [3] unrecognized-charges-procedure.md · Procedimiento ante cargos no reconocidos > Paso 4: seguimiento (primer texto en 14138 ms, respuesta completa en 14280 ms, 2081 tokens de entrada, 1611 de salida) Tú: Sí, registra el reclamo por el cargo duplicado TRX-3002 → file_claim(customer_id=CUS-1001, transaction_id=TRX-3002, reason=Cobro duplicado en supermercado: me cobraron dos veces.) NO SE EJECUTA, pendiente de aprobación: {"status": "pendiente_aprobacion", "action_id": "call-26dd80b7", "detail": "Acción sensible: no se ejecutó ni se registró todavía. Dile al cliente que la pedist Agente [sensitive_action]: He solicitado registrar el reclamo por TRX-3002. La solicitud quedó pendiente de aprobación por un asesor (no está resuelta). Puedes consultar el estado en la app: Reclamos > Mis reclamos, o por teléfono con el código del reclamo. Estados posibles: registrado, en evaluación, ampliado, resuelto. Te avisaré si hay novedades. Fuentes: [1] service-channels.html · Canales de atención para fraude y reclamos > Aplicación móvil [2] unrecognized-charges-procedure.md · Procedimiento ante cargos no reconocidos > Paso 4: seguimiento (primer texto en 15773 ms, respuesta completa en 16503 ms, 4418 tokens de entrada, 1991 de salida)|
| Acción aprobada contra Azure, con su `resultado:` | Asesor: ¿apruebas file_claim(customer_id=CUS-1001, transaction_id=TRX-3002, reason=Cobro duplicado en supermercado: me cobraron dos veces.)? [s/n]: s resultado: {"id": "CLM-9001", "customer_id": "CUS-1001", "transaction_id": "TRX-3002", "reason": "Cobro duplicado en supermercado: me cobraron dos veces.", "status": "regi Agente [sensitive_action]: He registrado tu reclamo por TRX-3002: ID CLM-9001, estado "registered", abierto el 2026-10-28 y con fecha límite de respuesta 2026-11-27. La solicitud queda registrada para que el banco la procese.|

### Qué pone Azure y qué pone la imagen

APPLICATIONINSIGHTS_CONNECTION_STRING
AZURE_CLIENT_ID
AZURE_OPENAI_ENDPOINT
COSMOS_CONTAINER
COSMOS_DATABASE
COSMOS_ENDPOINT
FOUNDRY_PROJECT_ENDPOINT
KEY_VAULT_URI
MCP_SCOPE
MCP_URL
SEARCH_ENDPOINT
SEARCH_INDEX
TEAM_ID

### Problemas que encontraste y cómo los resolviste

El problema recurrente en el que la api falla al primer intento porque no esta caliente.

### Conclusión

Asimismo, se comprobó el comportamiento esperado para acciones sensibles: inicialmente la creación del reclamo quedó en estado pendiente de aprobación y no fue ejecutada automáticamente, respetando el mecanismo de control y autorización. Posteriormente, al aprobarse la acción, el agente registró correctamente el reclamo en Azure/Cosmos DB, generando un identificador válido (CLM-9001/CLM-9002) y devolviendo la información correspondiente al usuario. El agente está desplegado, saludable, trazable, respeta el flujo de aprobación para acciones sensibles y ejecuta correctamente las operaciones una vez autorizadas.

## Semana 4 -Sesión 8 — Aprobación humana, validación y reintentos

Dominio: `frauds` (paso 6).

| Caso | Runtime | Mensaje | Qué pasó | Resultado o ID |
|---|---|---|---|---|
| Aprobar | `agent_framework` | "Sí, registra el reclamo por el cargo duplicado TRX-3002" | Se solicitó la creación del reclamo, quedó pendiente de aprobación del asesor y posteriormente fue aprobada y ejecutada. | Reclamo registrado **CLM-9002** para **TRX-3002** |
| Rechazar | `react` | "Sí, registra el reclamo por el cargo duplicado TRX-3002" | Se solicitó el reclamo, quedó pendiente de aprobación, pero el asesor rechazó la acción, por lo que no se ejecutó. | **Solicitud rechazada**, sin ID de reclamo |
| Error de negocio (`CUS-9999`) | `react` | "Soy CUS-9999, me cobraron dos veces en el supermercado" | La herramienta devolvió que el cliente no existe. El agente informó el error y pidió validar el código de cliente. | Error: **"El cliente CUS-9999 no existe"** |
| Input inválido | `react` | "Mi número de cliente es ABCD-1001, me cobraron dos veces en el supermercado" | El identificador no cumple el formato esperado (`CUS-XXXX`). El agente solicitó un código válido antes de continuar. | Validación fallida: formato de cliente inválido (**ABCD-1001**) |

### La trayectoria en el log

INFO:     Will watch for changes in these directories: ['E:\\IA\\git\\agent-scratch']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [35132] using WatchFiles
INFO agent.infrastructure.bootstrap: Foundry auth provider: entra_id
INFO azure.identity._credentials.environment: No environment configuration found.
INFO azure.identity._credentials.managed_identity: ManagedIdentityCredential will use IMDS
INFO agent.infrastructure.bootstrap: Foundry auth provider: entra_id
INFO azure.identity._credentials.environment: No environment configuration found.
INFO azure.identity._credentials.managed_identity: ManagedIdentityCredential will use IMDS
INFO azure.identity._credentials.environment: No environment configuration found.
INFO azure.identity._credentials.managed_identity: ManagedIdentityCredential will use IMDS
INFO azure.core.pipeline.policies.http_logging_policy: Request URL: 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=REDACTED'
Request method: 'GET'
Request headers:
    'x-ms-client-request-id': '2a729d9d-e459-11f1-af71-0093376f1a7a'
    'User-Agent': 'azsdk-python-identity/1.26.0 Python/3.12.10 (Windows-11-10.0.26200-SP0)'
No body was attached to the request
INFO azure.identity._credentials.chained: DefaultAzureCredential acquired a token from AzureCliCredential
INFO azure.core.pipeline.policies.http_logging_policy: Request URL: 'https://srch-workshop-jsu3.search.windows.net/indexes('idx-team-08')/docs/$count?api-version=2026-04-01'
Request method: 'GET'
Request headers:
    'Accept': 'application/json;odata.metadata=none'
    'x-ms-client-request-id': '2a729d9c-e459-11f1-b020-0093376f1a7a'
    'User-Agent': 'azsdk-python-search-documents/12.0.0 Python/3.12.10 (Windows-11-10.0.26200-SP0)'
    'Authorization': 'REDACTED'
No body was attached to the request
INFO azure.core.pipeline.policies.http_logging_policy: Response status: 200
Response headers:
    'Transfer-Encoding': 'chunked'
    'Content-Type': 'text/plain; charset=utf-8'
    'Content-Encoding': 'REDACTED'
    'Vary': 'REDACTED'
    'Strict-Transport-Security': 'REDACTED'
    'Preference-Applied': 'REDACTED'
    'OData-Version': 'REDACTED'
    'request-id': '2a729d9c-e459-11f1-b020-0093376f1a7a'
    'elapsed-time': 'REDACTED'
    'Date': 'Thu, 19 Nov 2026 20:38:26 GMT'
INFO azure.core.pipeline.policies.http_logging_policy: Request URL: 'https://srch-workshop-jsu3.search.windows.net/indexes('idx-team-08')/docs/$count?api-version=2026-04-01'
Request method: 'GET'
Request headers:
    'Accept': 'application/json;odata.metadata=none'
    'x-ms-client-request-id': '2c2cf84f-e459-11f1-8630-0093376f1a7a'
    'User-Agent': 'azsdk-python-search-documents/12.0.0 Python/3.12.10 (Windows-11-10.0.26200-SP0)'
    'Authorization': 'REDACTED'
No body was attached to the request
INFO azure.core.pipeline.policies.http_logging_policy: Response status: 200
Response headers:
    'Transfer-Encoding': 'chunked'
    'Content-Type': 'text/plain; charset=utf-8'
    'Content-Encoding': 'REDACTED'
    'Vary': 'REDACTED'
    'Strict-Transport-Security': 'REDACTED'
    'Preference-Applied': 'REDACTED'
    'OData-Version': 'REDACTED'
    'request-id': '2c2cf84f-e459-11f1-8630-0093376f1a7a'
    'elapsed-time': 'REDACTED'
    'Date': 'Thu, 19 Nov 2026 20:38:26 GMT'
INFO agent.infrastructure.bootstrap: Knowledge base: azure_search · https://srch-workshop-jsu3.search.windows.net (índice idx-team-08) · 56 chunks
INFO azure.identity._credentials.environment: No environment configuration found.
INFO azure.identity._credentials.managed_identity: ManagedIdentityCredential will use IMDS
INFO azure.core.pipeline.policies.http_logging_policy: Request URL: 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=REDACTED'
Request method: 'GET'
Request headers:
    'x-ms-client-request-id': '2c40e63f-e459-11f1-9442-0093376f1a7a'
    'User-Agent': 'azsdk-python-identity/1.26.0 Python/3.12.10 (Windows-11-10.0.26200-SP0)'
No body was attached to the request
INFO azure.identity._credentials.chained: DefaultAzureCredential acquired a token from AzureCliCredential
INFO httpx2: HTTP Request: POST https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp "HTTP/1.1 200 OK"
INFO httpx2: HTTP Request: POST https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp "HTTP/1.1 200 OK"
INFO agent.infrastructure.bootstrap: MCP: https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp · 4 tools: get_transactions, get_claim_status, block_card, file_claim
INFO agent.infrastructure.bootstrap: Agent runtime: agent_framework
INFO agent.infrastructure.bootstrap: Foundry auth provider: entra_id
INFO azure.identity._credentials.environment: No environment configuration found.
INFO azure.identity._credentials.managed_identity: ManagedIdentityCredential will use IMDS
INFO:     Started server process [17564]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO azure.core.pipeline.policies.http_logging_policy: Request URL: 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=REDACTED'
Request method: 'GET'
Request headers:
    'x-ms-client-request-id': '5f4e0bc7-e459-11f1-931a-0093376f1a7a'
    'User-Agent': 'azsdk-python-identity/1.26.0 Python/3.12.10 (Windows-11-10.0.26200-SP0)'
No body was attached to the request
INFO azure.identity._credentials.chained: DefaultAzureCredential acquired a token from AzureCliCredential
INFO httpx2: HTTP Request: POST https://aif-workshop-jsu3.openai.azure.com/openai/v1/embeddings "HTTP/1.1 200 OK"
INFO azure.core.pipeline.policies.http_logging_policy: Request URL: 'https://srch-workshop-jsu3.search.windows.net/indexes('idx-team-08')/docs/search.post.search?api-version=2026-04-01'
Request method: 'POST'
Request headers:
    'Content-Type': 'application/json'
    'Content-Length': '31432'
    'Accept': 'application/json;odata.metadata=none'
    'x-ms-client-request-id': '60f7cf05-e459-11f1-8feb-0093376f1a7a'
    'User-Agent': 'azsdk-python-search-documents/12.0.0 Python/3.12.10 (Windows-11-10.0.26200-SP0)'
    'Authorization': 'REDACTED'
A body is sent with the request
INFO azure.core.pipeline.policies.http_logging_policy: Response status: 200
Response headers:
    'Transfer-Encoding': 'chunked'
    'Content-Type': 'application/json; odata.metadata=none; odata.streaming=true; charset=utf-8'
    'Content-Encoding': 'REDACTED'
    'Vary': 'REDACTED'
    'Strict-Transport-Security': 'REDACTED'
    'Preference-Applied': 'REDACTED'
    'OData-Version': 'REDACTED'
    'request-id': '60f7cf05-e459-11f1-8feb-0093376f1a7a'
    'elapsed-time': 'REDACTED'
    'Date': 'Thu, 19 Nov 2026 20:39:55 GMT'
INFO:     127.0.0.1:61789 - "POST /chat/stream HTTP/1.1" 200 OK
INFO azure.core.pipeline.policies.http_logging_policy: Request URL: 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=REDACTED'
Request method: 'GET'
Request headers:
    'x-ms-client-request-id': '613d9a2a-e459-11f1-97ff-0093376f1a7a'
    'User-Agent': 'azsdk-python-identity/1.26.0 Python/3.12.10 (Windows-11-10.0.26200-SP0)'
No body was attached to the request
INFO azure.identity._credentials.chained: DefaultAzureCredential acquired a token from AzureCliCredential
INFO httpx2: HTTP Request: POST https://aif-workshop-jsu3.openai.azure.com/openai/v1/chat/completions "HTTP/1.1 200 OK"
INFO agent_framework: Function name: get_transactions
INFO httpx2: HTTP Request: POST https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp "HTTP/1.1 200 OK"
INFO httpx2: HTTP Request: POST https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp "HTTP/1.1 200 OK"
INFO agent_framework: Function get_transactions succeeded.
INFO agent_framework: Function duration: 0.559432s
INFO httpx2: HTTP Request: POST https://aif-workshop-jsu3.openai.azure.com/openai/v1/chat/completions "HTTP/1.1 200 OK"
INFO agent.infrastructure.llm.agent_framework_runtime: llm AgentFramework {'models': ['gpt-5-mini'], 'llm_calls': 2, 'latency_ms': 14221, 'prompt_tokens': 3140, 'completion_tokens': 1105}
INFO agent.application.chat_service: trayectoria 1f6ea454-f0a5-466c-894c-350a1319c9f6: get_transactions error → respuesta data_query
INFO httpx2: HTTP Request: POST https://aif-workshop-jsu3.openai.azure.com/openai/v1/embeddings "HTTP/1.1 200 OK"
INFO azure.core.pipeline.policies.http_logging_policy: Request URL: 'https://srch-workshop-jsu3.search.windows.net/indexes('idx-team-08')/docs/search.post.search?api-version=2026-04-01'
Request method: 'POST'
Request headers:
    'Content-Type': 'application/json'
    'Content-Length': '31394'
    'Accept': 'application/json;odata.metadata=none'
    'x-ms-client-request-id': 'ac639bec-e459-11f1-9a99-0093376f1a7a'
    'User-Agent': 'azsdk-python-search-documents/12.0.0 Python/3.12.10 (Windows-11-10.0.26200-SP0)'
    'Authorization': 'REDACTED'
A body is sent with the request
INFO azure.core.pipeline.policies.http_logging_policy: Response status: 200
Response headers:
    'Transfer-Encoding': 'chunked'
    'Content-Type': 'application/json; odata.metadata=none; odata.streaming=true; charset=utf-8'
    'Content-Encoding': 'REDACTED'
    'Vary': 'REDACTED'
    'Strict-Transport-Security': 'REDACTED'
    'Preference-Applied': 'REDACTED'
    'OData-Version': 'REDACTED'
    'request-id': 'ac639bec-e459-11f1-9a99-0093376f1a7a'
    'elapsed-time': 'REDACTED'
    'Date': 'Thu, 19 Nov 2026 20:42:01 GMT'
INFO:     127.0.0.1:57105 - "POST /chat/stream HTTP/1.1" 200 OK
INFO httpx2: HTTP Request: POST https://aif-workshop-jsu3.openai.azure.com/openai/v1/chat/completions "HTTP/1.1 200 OK"
INFO agent_framework: Function name: get_transactions
INFO httpx2: HTTP Request: POST https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp "HTTP/1.1 200 OK"
INFO httpx2: HTTP Request: POST https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp "HTTP/1.1 200 OK"
INFO httpx2: HTTP Request: POST https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp "HTTP/1.1 200 OK"
INFO agent_framework: Function get_transactions succeeded.
INFO agent_framework: Function duration: 0.673200s
INFO httpx2: HTTP Request: POST https://aif-workshop-jsu3.openai.azure.com/openai/v1/chat/completions "HTTP/1.1 200 OK"
INFO agent.infrastructure.llm.agent_framework_runtime: llm AgentFramework {'models': ['gpt-5-mini'], 'llm_calls': 2, 'latency_ms': 16961, 'prompt_tokens': 3331, 'completion_tokens': 1896}
INFO agent.application.chat_service: trayectoria 391e64a7-60b7-4d2b-910f-f9ab8ca16d57: get_transactions ok → respuesta data_query
INFO:     Shutting down
INFO:     Waiting for application shutdown.
INFO:     Application shutdown complete.
INFO:     Finished server process [17564]
INFO:     Stopping reloader process [35132]

### ¿Algo se ejecutó sin que lo aprobaras?

No. En todos los escenarios con acciones sensibles (file_claim), el sistema solicitó confirmación y dejó la operación pendiente de aprobación humana. La acción solo se ejecutó cuando fue aprobada explícitamente por el asesor y fue rechazada cuando el asesor la denegó.

### Conclusión

El comportamiento fue correcto y seguro. Tanto Agent Framework como ReAct respetaron el flujo de aprobación para acciones sensibles, gestionaron adecuadamente errores de negocio (cliente inexistente) y validaciones de entrada (formato de cliente inválido). No se observaron alucinaciones ni ejecuciones no autorizadas; todas las respuestas estuvieron basadas en los resultados de las herramientas y en las decisiones de aprobación humana.

# Semana 4 - Sesion 7 — Skills, memoria de trabajo y Agent Framework

Dominio: `fraud`. La misma conversación con los dos runtimes (paso 7).

| | `react` | `agent_framework` |
|---|---|---|
| Tools pedidas en la consulta | `get_transactions` | `get_transactions`, `file_claim` |
| La acción sensible quedó **NO SE EJECUTA** | No | Sí: `file_claim(TRX-3001)` y `file_claim(TRX-3002)` quedaron pendientes de aprobación |
| Pregunta de memoria respondida sin `→` | Sí: «¿Cuál es el saldo de mi cuenta de ahorros?» | Sí: «¿Cuál es el saldo de mi cuenta de ahorros?» |
| Primer texto (ms) | 17252<br>16849<br>17521 | 31673 ("error": "El servicio del banco no responde)<br>28830<br>21343<br>18481 |
| Respuesta completa (ms) | 18561 <br>17006<br>17661 | 32600 ("error": "El servicio del banco no responde)<br>29089<br>21357<br>18728 |
| Tokens (entrada / salida) | 3425 / 1336<br>2565 / 1849<br>2777 / 1894 | 3180 / 1899 ("error": "El servicio del banco no responde)<br>4106 / 2690<br>5516 / 2162<br>3685 / 1527 |

### ¿La skill frenó la tool de otro dominio?

Mensaje: Sí, registra el reclamo por el cargo duplicado. 
Respuesta: Sí, en react la skill de fraude obligó a completar primero el flujo de bloqueo de la tarjeta antes de permitir el registro del reclamo. Aunque el usuario pidió registrar el reclamo, el runtime no ejecutó ninguna herramienta adicional y volvió a solicitar confirmación para bloquear la tarjeta y elegir una transacción específica. En cambio, en agent_framework sí se permitió continuar con la intención principal, invocando file_claim para ambos cargos y dejando la acción sensible en estado pendiente de aprobación.



### Diferencias entre los dos runtimes

| Aspecto | `react` | `agent_framework` |
|---|---|---|
| Aplicación de la skill | Aplica una restricción más rígida del procedimiento. Considera el bloqueo de tarjeta como requisito previo para registrar un reclamo por cargo duplicado. | Separa la consulta de datos de las acciones sensibles. Identifica los cargos duplicados y luego permite solicitar el reclamo con confirmación del usuario. |
| Uso de herramientas | Solo ejecutó `get_transactions`. | Ejecutó `get_transactions` y posteriormente `file_claim`. |
| Registro del reclamo | No llegó a intentar ninguna herramienta de reclamo. | Intentó registrar reclamos para ambos cargos duplicados mediante `file_claim`. |
| Encadenamiento de tools | Limitado al flujo de consulta inicial. | Capaz de encadenar la consulta de movimientos y la solicitud de reclamo. |
| Manejo de la intención del usuario | Redirige la conversación a confirmar el bloqueo y seleccionar una transacción. | Avanza hacia la acción solicitada (reclamo) manteniendo las validaciones requeridas. |
| Interacciones requeridas | Necesita pasos adicionales antes de continuar. | Requiere menos interacción y progresa directamente hacia la acción pedida. |
| Acciones sensibles | No intenta ejecutar la acción de reclamo. | Solicita la acción sensible, pero la deja **pendiente de aprobación** y no la ejecuta. |
| Tiempo de respuesta | Más rápido (≈17–18 s). | Más lento (≈18–32 s). |
| Consumo de tokens | Menor consumo de entrada y salida. | Mayor consumo debido al razonamiento adicional y a la gestión de múltiples tools. |
| Alineación con la intención del usuario | Media. Prioriza el procedimiento de la skill sobre la acción solicitada. | Alta. Mantiene las restricciones de seguridad sin impedir el avance hacia la solicitud del usuario. |

### Conclusión

agent_framework mostró un comportamiento más alineado con la intención del usuario. Aunque respetó la política de acciones sensibles y no ejecutó realmente los reclamos sin aprobación, sí fue capaz de solicitar la operación mediante file_claim y dejarla pendiente de validación. Por el contrario, react quedó bloqueado y nunca llegó a intentar el registro del reclamo, redirigiendo repetidamente la conversación hacia el bloqueo de la tarjeta. En este escenario, agent_framework demuestra mejor capacidad para coordinar herramientas, mantener el flujo conversacional y evitar que una regla procedimental impida atender la solicitud principal del usuario.

## react
Tú: Soy CUS-1001, me cobraron dos veces en el supermercado
  → get_transactions(customer_id=CUS-1001, limit=20)
    resultado: {"result": [{"id": "TRX-3001", "date": "2026-10-20", "description": "SUPERMERCADO DEMO SAN ISIDRO", "amount": -186.4, "currency": "PEN", "card_id": "CRD-2001", 
  Agente [sensitive_action]: He localizado dos cargos iguales en SUPERMERCADO DEMO SAN ISIDRO: TRX-3001 (2026-10-20) -186.4 PEN y TRX-3002 (2026-10-20) -186.4 PEN. La tarjeta asociada es CRD-2001. Al reportar un cobro duplicado lo primero es bloquear la tarjeta (el bloqueo es definitivo). ¿Confirmas que proceda a bloquear la tarjeta CRD-2001? Después, dime cuál movimiento quieres reclamar (TRX-3001 o TRX-3002) para registrar el reclamo.
  (primer texto en 17252 ms, respuesta completa en 18561 ms, 3425 tokens de entrada, 1336 de salida)
Tú: Sí, registra el reclamo por el cargo duplicado
  Agente [sensitive_action]: Para proteger su cuenta primero debo bloquear la tarjeta CRD-2001 (el bloqueo es definitivo). ¿Confirma que proceda a bloquearla? Además, ¿qué movimiento quiere reclamar: TRX-3001 o TRX-3002? Tras su confirmación ejecutaré el bloqueo y luego registraré el reclamo; quedará pendiente de aprobación por un asesor.
  Fuentes:
   [1] unrecognized-charges-procedure.md · Procedimiento ante cargos no reconocidos > Paso 1: bloquear la tarjeta
   [2] unrecognized-charges-procedure.md · Procedimiento ante cargos no reconocidos > Paso 3: registrar el reclamo
   [3] card-blocking-policy.docx · Política de bloqueo de tarjetas > Confirmación obligatoria
  (primer texto en 16849 ms, respuesta completa en 17006 ms, 2565 tokens de entrada, 1849 de salida)
Tú: ¿Cuál es el saldo de mi cuenta de ahorros?
  Agente [out_of_scope]: Eso está fuera de mi área (fraude, bloqueo y reclamos); no puedo consultar saldos. Para ver su saldo, use la aplicación móvil o la banca por internet, o llame al (01) 555-0200. Si necesita ayuda con un cargo no reconocido, bloqueo de tarjeta o registrar un reclamo, puedo asistirle aquí.
  Fuentes:
   [1] service-channels.html · Canales de atención para fraude y reclamos > Aplicación móvil
   [2] service-channels.html · Canales de atención para fraude y reclamos > Banca por teléfono
  (primer texto en 17521 ms, respuesta completa en 17661 ms, 2777 tokens de entrada, 1894 de salida)

## Agent framework
Tú: nueva
  (sesión nueva)
Tú: Soy CUS-1001, me cobraron dos veces en el supermercado
  → get_transactions(customer_id=CUS-1001, limit=10)
    error: {"error": "El servicio del banco no responde; intenta de nuevo en unos minutos"}
  Agente [sensitive_action]: Entiendo. Intenté consultar sus movimientos (CUS-1001) pero el servicio no respondió. ¿Desea que lo intente de nuevo ahora para identificar el/los cargos duplicados y, con su confirmación sobre el movimiento, registrar el reclamo? Si sospecha fraude y prefiere bloquear la tarjeta, tenga en cuenta que el bloqueo es definitivo; si confirma, le pediré qué tarjeta (CRD-...) bloquear y solicitaré la acción, que quedará pendiente de aprobación del asesor. No comparta números completos de tarjeta, DNI ni claves.
  (primer texto en 31673 ms, respuesta completa en 32600 ms, 3180 tokens de entrada, 1899 de salida)
Tú: Soy CUS-1001, me cobraron dos veces en el supermercado
  → get_transactions(customer_id=CUS-1001, limit=10)
    resultado: {"result": [{"id": "TRX-3001", "date": "2026-10-20", "description": "SUPERMERCADO DEMO SAN ISIDRO", "amount": -186.4, "currency": "PEN", "card_id": "CRD-2001", 
  Agente [data_query]: He revisado sus movimientos: aparecen dos cargos idénticos —TRX-3001 y TRX-3002— ambos el 2026-10-20 en SUPERMERCADO DEMO SAN ISIDRO por −186.40 PEN con la tarjeta CRD-2001. ¿Confirma que desea que registre un reclamo por ambos cargos duplicados? Si confirma, procederé a registrar el reclamo (la acción requiere su confirmación y quedará pendiente de aprobación del asesor). Si prefiere, también puedo solicitar el bloqueo definitivo de la tarjeta CRD-2001 (el bloqueo es irreversible y, si lo solicita, quedará pendiente de aprobación). No comparta números completos de tarjeta, DNI ni claves.
  (primer texto en 28830 ms, respuesta completa en 29089 ms, 4106 tokens de entrada, 2690 de salida)
Tú: Sí, registra el reclamo por el cargo duplicado
  → file_claim(customer_id=CUS-1001, transaction_id=TRX-3001, reason=Cobro duplicado en SUPERMERCADO DEMO SAN ISIDRO (cargo TRX-3001). Solicito reclamo por cargo duplicado.)
  → file_claim(customer_id=CUS-1001, transaction_id=TRX-3002, reason=Cobro duplicado en SUPERMERCADO DEMO SAN ISIDRO (cargo TRX-3002). Solicito reclamo por cargo duplicado.)
    NO SE EJECUTA, pendiente de aprobación: {"status": "pendiente_aprobacion", "detail": "Acción sensible: no se ejecutó ni se registró todavía. Dile al cliente que la pediste y que un asesor la va a revi
    NO SE EJECUTA, pendiente de aprobación: {"status": "pendiente_aprobacion", "detail": "Acción sensible: no se ejecutó ni se registró todavía. Dile al cliente que la pediste y que un asesor la va a revi
  Agente [sensitive_action]: He solicitado el registro del reclamo por los cargos TRX-3001 y TRX-3002 (cargo duplicado). La solicitud está pendiente de aprobación por un asesor; le avisaré en cuanto haya novedades. No comparta números completos de tarjeta, DNI ni claves.
  (primer texto en 21343 ms, respuesta completa en 21357 ms, 5516 tokens de entrada, 2162 de salida)
Tú: ¿Cuál es el saldo de mi cuenta de ahorros?
  Agente [out_of_scope]: Eso está fuera del área de fraude, bloqueo y reclamos. Para consultar su saldo use la aplicación móvil del banco o llame a la banca por teléfono al (01) 555-0200 (lun–sáb 8:00–20:00). Si necesita revisar cargos no reconocidos o registrar un reclamo, con gusto le ayudo.
  Fuentes:
   [1] service-channels.html · Canales de atención para fraude y reclamos > Aplicación móvil
   [2] service-channels.html · Canales de atención para fraude y reclamos > Banca por teléfono
  (primer texto en 18481 ms, respuesta completa en 18728 ms, 3685 tokens de entrada, 1527 de salida)

# Semana 3 - Sesión 6 — Tools del MCP en el chat

Dominio: `fraud`. MCP: el mock (`http://127.0.0.1:8200/mcp`) y, si ya tienes `MCP_URL`, el compartido (`bank-mcp`).

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock | http://127.0.0.1:8200/mcp · fraud · 4 tools|
| Compartido | https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp · fraud · 4 tools |

### Conversación en `dev/chat.py` (paso 12)

| Prueba | Mensaje | Tool pedida (`→`) | Qué pasó | Primer texto / respuesta completa (ms) |
|---|---|---|---|---|
| Consulta de tu dominio | "Soy CUS-1001, me cobraron dos veces en el supermercado" | Revisar movimientos → / Registrar reclamo por cobro duplicado | El agente identificó correctamente una consulta de fraude/reclamos y ofreció acciones válidas dentro de su dominio. | 15274 / 15677 |
| Acción confirmada | "Sí, registra el reclamo por el cargo duplicado" | Registrar reclamo por cobro duplicado | El agente activó el flujo de acción sensible, pidió datos mínimos (ID, fecha y monto) y solicitó confirmación explícita antes de continuar. | 18842 / 19049 |
| Sin tool que la resuelva | "¿Cuál es el saldo de mi cuenta de ahorros?" | Consulta de saldo | El agente indicó que está fuera de alcance, no intentó inventar una respuesta y derivó al usuario a los canales oficiales. | 12803 / 13305 |
| Tool de otro dominio | "Soy CUS-1001, actualiza mi correo a ana.demo@example.com" | Actualización de datos personales | El agente rechazó la solicitud porque pertenece a otro dominio funcional (gestión de datos de cliente) y redirigió al canal oficial de atención. | 12035 / 12250 |

### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

No se observó invención de datos. En los casos fuera de alcance, el agente rechazó la solicitud y derivó al usuario a canales oficiales en lugar de generar información no verificada. Por ejemplo:

### Conclusión

El agente mostró un comportamiento correcto y seguro durante las pruebas:

- Reconoció consultas dentro de su dominio (fraude).
- Solicitó confirmación explícita antes de iniciar acciones sensibles.
- Rechazó peticiones fuera de su ámbito funcional.
- No intentó utilizar tools de otros dominios.
- No inventó información cuando no disponía de una herramienta o capacidad para resolver la solicitud.

# Semana 3 - Sesión 05 
# Comparacion de AI Search vs Local

| Pregunta | Fuentes en Local | Fuentes en AI Search | ¿Mejoró con el reranker? | Respuesta completa en (Local / AI Search) |
|-----------|------------------|----------------------|--------------------------|-------------------------------------------|
| ¿Cuál es el plazo para presentar un reclamo? | 1 fuente (`claims-regulation.pdf`) | 2 fuentes (`claims-regulation.pdf`, `fraud-and-claims-faq.md`) | No. La fuente adicional no aporta información nueva relevante. | **Empate**. Ambas respuestas son correctas y completas. |
| ¿Y si es por fraude? | 3 fuentes relevantes | 4 fuentes relevantes (incluye `response-times.csv`) | Leve mejora. AI Search incorpora una fuente adicional que refuerza los tiempos de respuesta. | **Empate técnico**. AI Search es ligeramente más fundamentado, pero la respuesta Local ya era completa. |
| ¿Qué es un contracargo? | No recupera la definición. No muestra fuentes útiles. | Recupera `credit-card-chargebacks.html` (definición y casos de uso). | Sí. Recupera el documento correcto y aporta la definición solicitada. | **AI Search gana claramente**. Local entrega una respuesta incompleta. |
| ¿Qué decía la política de bloqueo de 2022? | Recupera 5 fragmentos correctos de `card-blocking-policy.docx`. | No recupera la política. Devuelve documentos de reclamos y fraude. | No. El reranker empeora el resultado porque prioriza documentos incorrectos. | **Local gana claramente**. AI Search produce un falso negativo. |
| ¿A cuánto está el dólar hoy? | Sin fuentes (out of scope). | Sin fuentes útiles (out of scope). | No aplica. | **Empate**. Ambas respuestas son adecuadas. |

# Línea base (S5)
Resumen de 30 casos (juez local):
  recall@5: 1.00   intención: 0.97   fuentes: 0.90   hechos: 0.89
  apoyo: 5.0/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 1
  tokens: 27601 de entrada, 15021 de salida
Detalle de cada caso: E:\IA\git\agent-scratch\data\eval\fraud-20261112-145631.json

Resumen de 31 casos (juez local):
  recall@5: 1.00   intención: 1.00   fuentes: 0.94   hechos: 0.86
  apoyo: 4.9/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 1
  tokens: 28409 de entrada, 16650 de salida
Detalle de cada caso: E:\IA\git\agent-scratch\data\eval\fraud-20261112-150901.json

esumen de 31 casos (juez azure_evaluation):
  recall@5: 1.00   intención: 1.00   fuentes: 0.97   hechos: 0.93
  apoyo: 4.9/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 1
  tokens: 28226 de entrada, 16501 de salida
Detalle de cada caso: E:\IA\git\agent-scratch\data\eval\fraud-20261112-151356.json

# Paso 11 - Comparar los 2 jueces

1. ¿Las notas de apoyo y de cita correcta van en la misma dirección?
Sí, pero Azure es más sensible que G-Eval local.
Las métricas sí van en la misma dirección.

De hecho, los peores casos de:

-groundedness
-citation_correct

son prácticamente los mismos.

La diferencia es que:
el juez local es casi binario (1.0 o nada),
Azure es más gradual y castiga pequeñas inferencias o contradicciones.

2. ¿Cuánto tardó cada corrida?

| Corrida | Tiempo total | Promedio |
|---------|--------------|----------|
| Local | 179 s | 5.78 s |
| Azure | 168 s | 5.41 s |

Azure fue aproximadamente:

11,5 segundos más rápida (≈ 6% menos tiempo total)

3. Para los casos donde no coinciden, abre el detalle de G-Eval: su reasoning explica qué vio.

El reasoning útil solo existe en el juez local. En Azure todos los campos reasoning están vacíos, así que no es posible inspeccionar qué vio exactamente el evaluador. Para analizar discrepancias, el archivo local es mucho más valioso.

Las mayores discrepancias aparecen entre los checkers binarios (facts_ok, sources_ok) y los jueces LLM. Casos como fraud-10, fraud-12, fraud-15 y fraud-16 sugieren varios falsos negativos de los evaluadores heurísticos.

# Paso 12 - Cerrar el tema 2
| Métrica | Resultado | Umbral | Cumple |
|---|---:|---:|---:|
| recall@k | 1.00 | ≥ 0.80 | ✅ |
| hechos | 0.9286 | ≥ 0.75 | ✅ |
| apoyo | 4.91 | ≥ 4.0 | ✅ |
| fugas de datos personales | 0 | = 0 | ✅ |
| citas de fuentes prohibidas | 1 | = 0 | ❌ |

1. Si alguna no, escribe qué cambiarías.

El paso fraud-25 usa la cito internal-notice-cafeteria.html, se podria eliminar o bloquear la citacion

# Semana #2 Sesion 4 - Retrieval

# Local con SEARCH_TOP_K=5
Conversación con el agente. Escribe "nueva" para otra sesión o "salir" para terminar.
Tú: ¿Cuál es el plazo para presentar un reclamo?
  Agente [product_question]: Depende del tipo de reclamo: para una operación no reconocida tiene hasta 60 días calendario desde la fecha de la operación; para reclamos por cobros de comisiones o intereses tiene 1 año desde el cobro.
  Fuentes:
   [1] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
  (primer texto en 9447 ms, respuesta completa en 9589 ms, 932 tokens de entrada, 456 de salida)
Tú: ¿y si es por fraude?
  Agente [product_question]: Si es por fraude (operación no reconocida con tarjeta): tienes hasta 60 días calendario desde la fecha de la operación para presentar el reclamo. El banco responde esas operaciones en un plazo máximo de 10 días hábiles; para reclamos en general el plazo máximo es 15 días hábiles. Si el caso depende de un comercio del exterior, el plazo puede ampliarse una vez por 15 días hábiles.
  Fuentes:
   [1] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
   [2] fraud-and-claims-faq.md · Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿En cuánto tiempo me responden?
   [3] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta
  (primer texto en 12214 ms, respuesta completa en 12370 ms, 1217 tokens de entrada, 1044 de salida)
Tú: ¿Qué es un contracargo?
  Agente [product_question]: En los fragmentos proporcionados no aparece una definición de “contracargo”.
  (primer texto en 13112 ms, respuesta completa en 13121 ms, 1434 tokens de entrada, 1314 de salida)
Tú: ¿Qué decía la política de bloqueo de 2022?
  Agente [product_question]: La política indicaba, en resumen: 
- Antes de ejecutar un bloqueo definitivo se requiere la confirmación explícita del titular. 
- Canales: aplicación móvil (Tarjetas > Bloquear tarjeta, disponible 24 h), línea de emergencias (01) 555-0100 disponible 24 h, y agencias en horario de atención. 
- El bloqueo no cancela cargos recurrentes; el titular debe actualizar sus suscripciones. 
- El titular no asume operaciones no reconocidas realizadas después del reporte; las operaciones previas al reporte se evalúan según el reglamento de reclamos. 
- Tipos: bloqueo temporal (se puede activar y desactivar desde la app) y bloqueo definitivo (irreversible, aplicado por robo, clonación o fraude confirmado, y requiere reponer la tarjeta). 
- La tarjeta se bloquea de inmediato al reportar robo, pérdida, clonación u operación no reconocida; el banco puede además bloquear preventivamente si detecta patrones de fraude y contacta al titular.
  Fuentes:
   [1] card-blocking-policy.docx · Política de bloqueo de tarjetas > Confirmación obligatoria
   [2] card-blocking-policy.docx · Política de bloqueo de tarjetas > Canales de bloqueo
   [3] card-blocking-policy.docx · Política de bloqueo de tarjetas > Responsabilidad por operaciones
   [4] card-blocking-policy.docx · Política de bloqueo de tarjetas > Tipos de bloqueo
   [5] card-blocking-policy.docx · Política de bloqueo de tarjetas > Cuándo se bloquea una tarjeta
  (primer texto en 14131 ms, respuesta completa en 14514 ms, 1460 tokens de entrada, 1507 de salida)
Tú: ¿A cuánto está el dólar hoy?
  Agente [out_of_scope]: Eso está fuera del alcance de este servicio. No tengo información de tipo de cambio. Consulta el dólar actualizado en la web o la app del banco, en una agencia o en la página del Banco Central de Reserva del Perú (BCRP).
  (primer texto en 8363 ms, respuesta completa en 8493 ms, 1893 tokens de entrada, 682 de salida)

  # AI Search con SEARCH_TOP_K=5

  Conversación con el agente. Escribe "nueva" para otra sesión o "salir" para terminar.
Tú: ¿Cuál es el plazo para presentar un reclamo?
  Agente [product_question]: Tiene hasta 60 días calendario desde la fecha de la operación para reclamar una operación no reconocida. Para reclamos sobre cobros de comisiones o intereses, el plazo es de 1 año desde el cobro.
  Fuentes:
   [1] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
   [2] fraud-and-claims-faq.md · Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿Cuánto tiempo tengo para reclamar un cargo?
  (primer texto en 9752 ms, respuesta completa en 9894 ms, 915 tokens de entrada, 391 de salida)
Tú: ¿y si es por fraude?
  Agente [product_question]: Si se trata de fraude (operación no reconocida con tarjeta): tienes hasta 60 días calendario desde la fecha de la operación para presentar el reclamo. El banco debe responder esa investigación en un plazo máximo de 10 días hábiles; si el caso depende de la marca o del comercio (p. ej. comercio del exterior), el plazo puede ampliarse una vez por 15 días hábiles. En general, el banco responde todo reclamo en un plazo máximo de 15 días hábiles.
  Fuentes:
   [1] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
   [2] fraud-and-claims-faq.md · Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿En cuánto tiempo me responden?
   [3] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta
   [4] response-times.csv · response-times
  (primer texto en 13444 ms, respuesta completa en 13704 ms, 1271 tokens de entrada, 1306 de salida)
Tú: ¿Qué es un contracargo?
  Agente [product_question]: Un contracargo es la devolución de una compra con tarjeta de crédito que el banco gestiona ante la marca de la tarjeta, cuando el comercio no entregó el producto, cobró un monto distinto o la operación no fue autorizada por el titular. Aplica en casos como: producto o servicio no recibido; cobro duplicado o monto distinto; suscripción cancelada que se siguió cobrando; y operación no reconocida por el titular.
  Fuentes:
   [1] credit-card-chargebacks.html · Contracargos con tarjeta de crédito > Qué es un contracargo
   [2] credit-card-chargebacks.html · Contracargos con tarjeta de crédito > Casos que aplican
  (primer texto en 10816 ms, respuesta completa en 10974 ms, 1319 tokens de entrada, 864 de salida)
Tú: ¿Qué decía la política de bloqueo de 2022?
  Agente [product_question]: En los fragmentos proporcionados no aparece la "política de bloqueo de 2022". Los documentos disponibles tratan plazos y tiempos de respuesta para reclamos, pero no contienen esa política. ¿Deseas que busque más información o te indique cómo solicitarla formalmente?
  Fuentes:
   [1] fraud-and-claims-faq.md · Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿En cuánto tiempo me responden?
   [2] fraud-and-claims-faq.md · Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿Cuánto tiempo tengo para reclamar un cargo?
   [3] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
   [4] response-times.csv · response-times
   [5] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta
  (primer texto en 13184 ms, respuesta completa en 13325 ms, 1755 tokens de entrada, 1277 de salida)
Tú: ¿A cuánto está el dólar hoy?
  Agente [out_of_scope]: Lo siento, esa consulta está fuera del alcance del servicio de fraude, bloqueo y reclamos. Para la cotización del dólar revisa la web o la app del banco, la página del Banco Central de Reserva del Perú o una casa de cambio.
  (primer texto en 8177 ms, respuesta completa en 8319 ms, 1778 tokens de entrada, 676 de salida)

  # Comparacion de  SEARCH_TOP_K = 2 vs SEARCH_TOP_K = 10

- Con 2: ¿falta información en alguna respuesta?;
En 2 preguntas se detecta falta de informacion ¿Qué es un contracargo? y ¿Qué decía la política de bloqueo de 2022?" que estan en SEARCH_TOP_K = 10, pero no en SEARCH_TOP_K = 2

- Con 10: ¿cuántos tokens de entrada más cuesta cada turno?, ¿aparecen fuentes que no vienen al caso?;

Pregunta	    TOP_K=2	TOP_K=10	Diferencia
Reclamo	      657	    1422	    +765
Fraude	      927	    1704	    +777
Contracargo	  1046	  1780	    +734
Política 2022 1280	  2022	    +742
Dólar	        1446	  2173	    +727
749 tokens adicionales por turno
Respecto a si aparecen fuente qu no vienen al caso, no se ha detectado. Sin embargo, en K=10 se observa informacion adicional que no solicita la pregunta

- El valor que elige el equipo y por qué.

Como el experimento muestra que:

TOP_K=2 tiene problemas de recall.

Pierde la definición de contracargo.
Pierde completamente la política de bloqueo 2022.

TOP_K=10 recupera toda la información relevante.

Pero aumenta ~70% el coste de contexto.
Introduce información secundaria no solicitada.

Se opta por una opcion intermedia que seria el TOP_K = 5
  # SEARCH_TOP_K = 2
PS E:\IA\git\agent-scratch> (Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned) ; (& e:\IA\git\agent-scratch\.venv\Scripts\Activate.ps1)
(agent) PS E:\IA\git\agent-scratch> uv run python dev/chat.py
Conversación con el agente. Escribe "nueva" para otra sesión o "salir" para terminar.
Tú: ¿Cuál es el plazo para presentar un reclamo?
  Agente [product_question]: Tiene hasta 60 días calendario, contados desde la fecha de la operación, para reclamar una operación no reconocida. Para reclamos por cobros de comisiones o intereses, el plazo es de 1 año desde el cobro.
  Fuentes:
   [1] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
  (primer texto en 10509 ms, respuesta completa en 10644 ms, 657 tokens de entrada, 329 de salida)
Tú: ¿y si es por fraude?
  Agente [product_question]: Si es por fraude, tienes hasta 60 días calendario, contados desde la fecha de la operación, para presentar el reclamo.
  Fuentes:
   [1] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
   [2] fraud-and-claims-faq.md · Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿Cuánto tiempo tengo para reclamar un cargo?
  (primer texto en 11141 ms, respuesta completa en 11315 ms, 927 tokens de entrada, 1036 de salida)
Tú: ¿Qué es un contracargo?
  Agente [product_question]: En los fragmentos proporcionados no hay una definición de “contracargo”. Solo indican plazos: el titular debe solicitarlo dentro de 60 días calendario desde la fecha del cargo; las compras en comercios del exterior pueden tardarhasta 45 días hábiles en resolverse; y que, en general, para operaciones no reconocidas el plazo es 60 días (para comisiones o intereses, 1 año).
  Fuentes:
   [1] credit-card-chargebacks.html · Contracargos con tarjeta de crédito > Plazos del contracargo
   [2] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
  (primer texto en 12281 ms, respuesta completa en 13159 ms, 1046 tokens de entrada, 1416 de salida)
Tú: ¿Qué decía la política de bloqueo de 2022?
  Agente [product_question]: En los fragmentos proporcionados no hay información sobre la política de bloqueo de 2022.
  (primer texto en 10379 ms, respuesta completa en 10394 ms, 1280 tokens de entrada, 1012 de salida)
Tú: ¿A cuánto está el dólar hoy?
  Agente [out_of_scope]: Lo siento, no puedo proporcionar cotizaciones en tiempo real. Consulta la app o la web del Banco Demo, una casa de cambio o un portal financiero para el tipo de cambio del día.
  (primer texto en 9959 ms, respuesta completa en 10092 ms, 1446 tokens de entrada, 1023 de salida)
Tú: salir
(agent) PS E:\IA\git\agent-scratch> 

  # SEARCH_TOP_K = 10
(agent) PS E:\IA\git\agent-scratch> uv run python dev/chat.py
Conversación con el agente. Escribe "nueva" para otra sesión o "salir" para terminar.
Tú: ¿Cuál es el plazo para presentar un reclamo?
  Agente [product_question]: Tienes hasta 60 días calendario desde la fecha de la operación para reclamar una operación no reconocida. Para reclamos por cobros de comisiones o intereses, el plazo es de 1 año desde el cobro.
  Fuentes:
   [1] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
   [2] fraud-and-claims-faq.md · Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿Cuánto tiempo tengo para reclamar un cargo?
  (primer texto en 12066 ms, respuesta completa en 12202 ms, 1422 tokens de entrada, 520 de salida)
Tú: ¿y si es por fraude?
  Agente [product_question]: Si es por fraude u operación no reconocida: tienes hasta 60 días calendario desde la fecha de la operación para presentar el reclamo. El banco responde en un plazo máximo de 10 días hábiles para operaciones no reconocidas con tarjeta; ese plazo puede ampliarse una sola vez por 15 días hábiles si se requiere información de terceros (marca o comercio en el exterior).
  Fuentes:
   [1] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
   [2] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta
   [3] claims-regulation.pdf · Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta > 4.1 Ampliación del plazo
  (primer texto en 12609 ms, respuesta completa en 12762 ms, 1704 tokens de entrada, 1273 de salida)
Tú: ¿Qué es un contracargo?
  Agente [product_question]: Un contracargo es la devolución de una compra con tarjeta de crédito que el banco gestiona ante la marca de la tarjeta cuando, por ejemplo, el comercio no entregó el producto, cobró un monto distinto o la operación no fue autorizada por el titular.
  Fuentes:
   [1] credit-card-chargebacks.html · Contracargos con tarjeta de crédito > Qué es un contracargo
   [2] credit-card-chargebacks.html · Contracargos con tarjeta de crédito > Casos que aplican
  (primer texto en 12233 ms, respuesta completa en 12401 ms, 1780 tokens de entrada, 1023 de salida)
Tú: ¿Qué decía la política de bloqueo de 2022?
  Agente [product_question]: La política de bloqueo (2022) establecía: la tarjeta se bloquea de inmediato cuando el titular reporta robo, pérdida, clonación o una operación no reconocida; el banco también puede bloquearla de forma preventiva si su sistema detecta un patrón de fraude y en ese caso contacta al titular. Además, el titular no asume las operaciones realizadas después del reporte de bloqueo; las operaciones previas al reporte se evalúan según el reglamento de reclamos.
  Fuentes:
   [1] card-blocking-policy.docx · Política de bloqueo de tarjetas > Cuándo se bloquea una tarjeta
   [2] card-blocking-policy.docx · Política de bloqueo de tarjetas > Responsabilidad por operaciones
  (primer texto en 11835 ms, respuesta completa en 13107 ms, 2022 tokens de entrada, 1245 de salida)
Tú: ¿A cuánto está el dólar hoy?
  Agente [out_of_scope]: Lo siento, eso está fuera de mi ámbito. No puedo proporcionar la cotización del dólar. Consulta la web o la app del Banco Demo, o una casa de cambio para la tasa del día.
  (primer texto en 9405 ms, respuesta completa en 9543 ms, 2173 tokens de entrada, 603 de salida)
Tú: salir

# Semana 2 - Sesión 03 - Paso 11·Experimentar con el tamaño de chunk#

C:\labs\agent-scratch> uv run python dev/show_chunks.py
card-blocking-policy-2022.md               203  Política de bloqueo de tarjetas (versión 2022) > Canales de bloqueo
card-blocking-policy-2022.md               155  Política de bloqueo de tarjetas (versión 2022) > Tipos de bloqueo
card-blocking-policy.docx                  181  Política de bloqueo de tarjetas > Cuándo se bloquea una tarjeta
card-blocking-policy.docx                  239  Política de bloqueo de tarjetas > Cuándo se bloquea una tarjeta
card-blocking-policy.docx                  245  Política de bloqueo de tarjetas > Canales de bloqueo
card-blocking-policy.docx                  291  Política de bloqueo de tarjetas > Tipos de bloqueo
card-blocking-policy.docx                  266  Política de bloqueo de tarjetas > Confirmación obligatoria
card-blocking-policy.docx                  188  Política de bloqueo de tarjetas > Confirmación obligatoria
card-blocking-policy.docx                  262  Política de bloqueo de tarjetas > Responsabilidad por operaciones
card-replacement.docx                      160  Reposición de tarjeta > Cuándo se repone
card-replacement.docx                      205  Reposición de tarjeta > Costo
card-replacement.docx                      239  Reposición de tarjeta > Plazo de entrega
claims-regulation.pdf                      304  Reglamento de atención de reclamos - Banco Demo > 1. Objeto y alcance
claims-regulation.pdf                      139  Reglamento de atención de reclamos - Banco Demo > 1. Objeto y alcance
claims-regulation.pdf                      309  Reglamento de atención de reclamos - Banco Demo > 2. Definiciones
claims-regulation.pdf                      179  Reglamento de atención de reclamos - Banco Demo > 2. Definiciones
claims-regulation.pdf                      285  Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo
claims-regulation.pdf                      323  Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.1 Plazo para presentar
claims-regulation.pdf                      347  Reglamento de atención de reclamos - Banco Demo > 3. Presentación del reclamo > 3.2 Información mínima
claims-regulation.pdf                      256  Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta
claims-regulation.pdf                      290  Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta > 4.1 Ampliación del plazo
claims-regulation.pdf                      196  Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta > 4.1 Ampliación del plazo
claims-regulation.pdf                      306  Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta > 4.2 Excepciones
claims-regulation.pdf                      174  Reglamento de atención de reclamos - Banco Demo > 4. Plazos de respuesta > 4.2 Excepciones
claims-regulation.pdf                      279  Reglamento de atención de reclamos - Banco Demo > 5. Abono provisional
claims-regulation.pdf                      204  Reglamento de atención de reclamos - Banco Demo > 5. Abono provisional
claims-regulation.pdf                      268  Reglamento de atención de reclamos - Banco Demo > 6. Resultado del reclamo
claims-regulation.pdf                      267  Reglamento de atención de reclamos - Banco Demo > 6. Resultado del reclamo
credit-card-chargebacks.html               296  Contracargos con tarjeta de crédito > Qué es un contracargo
credit-card-chargebacks.html               231  Contracargos con tarjeta de crédito > Casos que aplican
credit-card-chargebacks.html               148  Contracargos con tarjeta de crédito > Plazos del contracargo
credit-card-chargebacks.html               231  Contracargos con tarjeta de crédito > Plazos del contracargo
credit-card-chargebacks.html               290  Contracargos con tarjeta de crédito > Qué documentos se piden
digital-security-and-phishing.md           255  Guía de seguridad digital > Cómo reconocer un intento de phishing
digital-security-and-phishing.md           218  Guía de seguridad digital > Qué hacer si compartiste tus datos
digital-security-and-phishing.md           236  Guía de seguridad digital > Buenas prácticas
fraud-and-claims-faq.md                    262  Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero?
fraud-and-claims-faq.md                    195  Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿Cuánto tiempo tengo para reclamar un cargo?
fraud-and-claims-faq.md                    333  Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿En cuánto tiempo me responden?
fraud-and-claims-faq.md                    251  Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿Me devuelven el dinero mientras revisan?
fraud-and-claims-faq.md                    308  Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿El banco me va a pedir mi clave por teléfono?
fraud-and-claims-faq.md                    218  Preguntas frecuentes: fraude y reclamos > ¿Me robaron la tarjeta, qué hago primero? > ¿Puedo desbloquear mi tarjeta?
fraud-glossary.md                          155  Glosario de fraude y reclamos > Clonación
fraud-glossary.md                          133  Glosario de fraude y reclamos > Clonación > Phishing
fraud-glossary.md                          126  Glosario de fraude y reclamos > Clonación > Vishing
fraud-glossary.md                          144  Glosario de fraude y reclamos > Clonación > Contracargo
fraud-glossary.md                          139  Glosario de fraude y reclamos > Clonación > Abono provisional
internal-notice-cafeteria.html             178  Comunicado interno: horario de la cafetería > Nuevo horario
response-times.csv                         248  response-times
response-times.csv                         221  response-times
response-times.csv                         227  response-times
response-times.csv                         235  response-times
response-times.csv                         225  response-times
sample-cases.pdf                           231  Casos de ejemplo para capacitación (datos ficticios) > 1. Caso de clonación en cajero
sample-cases.pdf                           198  Casos de ejemplo para capacitación (datos ficticios) > 1. Caso de clonación en cajero
sample-cases.pdf                           315  Casos de ejemplo para capacitación (datos ficticios) > 2. Caso de compra en el exterior
sample-cases.pdf                           282  Casos de ejemplo para capacitación (datos ficticios) > 3. Caso de vishing
service-channels.html                      187  Canales de atención para fraude y reclamos > Línea de emergencias
service-channels.html                      169  Canales de atención para fraude y reclamos > Banca por teléfono
service-channels.html                      176  Canales de atención para fraude y reclamos > Aplicación móvil
service-channels.html                      170  Canales de atención para fraude y reclamos > Agencias
unrecognized-charges-procedure.md          264  Procedimiento ante cargos no reconocidos > Paso 1: bloquear la tarjeta
unrecognized-charges-procedure.md          175  Procedimiento ante cargos no reconocidos > Paso 2: revisar los movimientos
unrecognized-charges-procedure.md          273  Procedimiento ante cargos no reconocidos > Paso 2: revisar los movimientos
unrecognized-charges-procedure.md          284  Procedimiento ante cargos no reconocidos > Paso 3: registrar el reclamo
unrecognized-charges-procedure.md          244  Procedimiento ante cargos no reconocidos > Paso 4: seguimiento


