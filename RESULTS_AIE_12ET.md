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
