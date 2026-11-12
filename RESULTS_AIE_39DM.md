# Semana 3 - Sesión 05 
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

¿Las notas de apoyo y de cita correcta van en la misma dirección?
¿Cuánto tardó cada corrida?
Para los casos donde no coinciden, abre el detalle de G-Eval: su reasoning explica qué vio.

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