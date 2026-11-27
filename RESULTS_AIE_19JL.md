Sesión 6 — Integración y uso de Tools MCP en el chat

Dominio: fraud.

Servidores MCP: Se trabajó con el servidor mock (http://127.0.0.1:8200/mcp) y el servidor compartido (bank-mcp), ambos configurados para proporcionar las herramientas correspondientes al dominio de fraude.

Las pruebas de conversación se realizaron utilizando el servidor mock, evitando ejecutar operaciones sobre el entorno compartido. Para complementar las respuestas del agente se utilizó la base de conocimiento de Azure AI Search (idx-team-08).

Tools disponibles para el dominio

Servidor MCP

Herramientas identificadas en dev/mcp/mcp_tools.py

Mock

get_transactions (lectura), get_claim_status (lectura), block_card (acción sensible), file_claim (acción sensible)

Compartido

get_transactions (lectura), get_claim_status (lectura), block_card (acción sensible), file_claim (acción sensible)

Se comprobó que ambos servidores ofrecen las mismas herramientas para atender las operaciones relacionadas con fraude.

Validación de herramientas mediante dev/chat.py (paso 12)

Escenario

Consulta realizada

Herramienta seleccionada

Resultado obtenido

Tiempo (primer texto / total ms)

Consulta de movimientos

“Necesito ver mis dos transacciones más recientes. Mi ID de cliente es CUS-1001.”

get_transactions(customer_id=CUS-1001, limit=2)

La herramienta respondió correctamente con los dos movimientos solicitados. El agente utilizó la información devuelta para construir su respuesta.

14 654 / 14 924

Bloqueo de tarjeta

“Quiero bloquear mi tarjeta CRD-2001 por motivo de robo. Confirmo la operación.”

block_card(card_id=CRD-2001, reason=robo)

La operación no se realizó inmediatamente. La solicitud quedó registrada como pending_approval, informando al usuario que necesitaba aprobación de un asesor.

23 729 / 23 995

Consulta no disponible

“Indícame cuánto dinero tengo actualmente en mi cuenta CUS-1001.”

No se seleccionó ninguna herramienta

El agente explicó que no contaba con acceso a la información del saldo. En su lugar, propuso consultar los movimientos de la cuenta.

13 860 / 14 012

Solicitud fuera del dominio

“¿Puedes indicarme cuál es mi línea de crédito disponible para la tarjeta CRD-2001?”

Ninguna (get_credit_line no pertenece al dominio fraud)

El agente identificó que la solicitud estaba fuera del alcance de sus herramientas y respondió con una alternativa basada en los canales de atención disponibles en la base de conocimiento.

18 651 / 18 808

Comportamiento del modelo ante consultas sin herramientas disponibles

Durante las validaciones no se observaron respuestas con información inventada.

Cuando se solicitó conocer el saldo de la cuenta, el agente reconoció que no tenía una herramienta habilitada para consultar ese dato y evitó indicar cantidades sin respaldo.

De igual manera, al preguntar por la línea de crédito, no intentó ejecutar una herramienta que no estaba disponible dentro del dominio fraud. En ambos casos proporcionó alternativas de atención o consultas que sí podía realizar.

En el escenario de movimientos, los datos presentados por el agente correspondieron a la información obtenida directamente mediante get_transactions.

Conclusiones de la sesión

Las pruebas permitieron verificar que la integración MCP funciona de acuerdo con las herramientas habilitadas para el dominio fraud. Tanto el servidor mock como el compartido presentan el mismo conjunto de cuatro herramientas.

Finalmente, los tiempos registrados muestran que la primera respuesta apareció entre 13,9 y 23,7 segundos, mientras que el tiempo adicional hasta completar la respuesta fue de entre 152 y 270 milisegundos. Esto indica que la mayor parte del tiempo de procesamiento se produjo antes de comenzar la entrega del contenido al usuario.