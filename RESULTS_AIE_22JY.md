# Resultados — AIE-22JY

## Paso 11 · Chat contra Foundry

| Caso | Pregunta | Intención | Fuente observada | ¿Fuente correcta? | Primer texto en | Respuesta completa en |
| --- | --- | --- | --- | --- | ---: | ---: |
| Regla del corpus | ¿Cuál es el plazo para presentar un reclamo? | `product_question` | `claims-regulation.pdf` — 3.1 Plazo para presentar | Sí | 4959 ms | 5081 ms |
| Seguimiento | ¿y si es por fraude? | `product_question` | `claims-regulation.pdf` — 3.1 Plazo para presentar; `fraud-and-claims-faq.md` — ¿Cuánto tiempo tengo para reclamar un cargo? | Sí | 9791 ms | 9922 ms |
| Término exacto | ¿Qué es un contracargo? | `product_question` | `credit-card-chargebacks.html` — Qué es un contracargo; `fraud-glossary.md` — Contracargo | Sí | 4615 ms | 4756 ms |
| Documento vencido | ¿Qué decía la política de bloqueo de 2022? | `product_question` | `card-blocking-policy.docx` (política vigente); no citó `card-blocking-policy-2022.md` | Sí | 6188 ms | 6464 ms |
| Fuera del corpus | ¿A cuánto está el dólar hoy? | `out_of_scope` | Sin fuentes | Sí | 4263 ms | 4414 ms |

### Resultado

Los cinco casos cumplieron el criterio esperado de intención y procedencia de las fuentes. Cada respuesta sobre el producto incluyó una fuente válida, la consulta sobre el documento vencido no citó `card-blocking-policy-2022.md` y la pregunta fuera del corpus no incluyó fuentes.

