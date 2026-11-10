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

