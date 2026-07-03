# Guía de Buenas Prácticas de Prompting (2 de Jul 26)

Referencia compilada de guías oficiales de los principales proveedores de LLMs.

**Fuentes:** Anthropic (Claude), OpenAI (GPT), Google (Gemini), xAI (Grok), Meta (LLaMA), DeepSeek

**Fecha:** 2026-07-02

---

## Índice

1. [Principios universales](#principios-universales)
2. [Anthropic — Claude](#anthropic--claude)
3. [OpenAI — GPT](#openai--gpt)
4. [Google — Gemini](#google--gemini)
5. [xAI — Grok](#xai--grok)
6. [Meta — LLaMA](#meta--llama)
7. [DeepSeek](#deepseek)
8. [Comparativa rápida](#comparativa-rápida)
9. [Reglas prácticas transversales](#reglas-prácticas-transversales)

---

## Principios universales

Estos principios aplican a TODOS los modelos:

1. **Sé claro y específico** — no asumas que el modelo infiere tu intención
2. **Usa ejemplos (few-shot)** — 3-5 ejemplos mejoran la consistencia dramáticamente
3. **Estructura tu prompt** — usa secciones, headers, XML tags o markdown para separar instrucciones, contexto y datos
4. **Da contexto/motivo** — explica POR QUÉ una instrucción importa, no solo QUÉ hacer
5. **Especifica el formato de salida** — JSON, tabla, markdown, longitud, idioma
6. **Asigna un rol** — "Eres un experto en..." mejora el enfoque del modelo
7. **Itera** — prompting es un proceso, no un evento único. Refina basándote en outputs

---

## Anthropic — Claude

Fuente: [docs.anthropic.com](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/claude-prompting-best-practices)

### Modelos actuales (Julio 2026)

- **Claude Fable 5** (`claude-fable-5`) — El modelo más capaz de disponibilidad general. Para el razonamiento más exigente y trabajo agentic de largo alcance. Adaptive thinking siempre activo. 1M contexto, 128K output. ($10 / $50 por MTok)
- **Claude Mythos 5** (`claude-mythos-5`) — Sucesor de Claude Mythos Preview; disponibilidad limitada vía Project Glasswing (acceso por invitación)
- **Claude Opus 4.8** (`claude-opus-4-8`) — El modelo Opus más capaz para razonamiento complejo y coding agentic. Flagship recomendado para empezar. 1M contexto, 128K output, cutoff de conocimiento Ene 2026. ($5 / $25 por MTok)
- **Claude Sonnet 5** (`claude-sonnet-5`) — El mejor equilibrio entre velocidad e inteligencia. Adaptive thinking activo por defecto. 1M contexto, 128K output, cutoff Ene 2026. ($3 / $15 por MTok; **precio introductorio $2 / $10 hasta 2026-08-31**)
- **Claude Haiku 4.5** (`claude-haiku-4-5`) — El modelo más rápido con inteligencia casi-frontier. 200K contexto. ($1 / $5 por MTok)

> **Legacy (aún disponibles):** Opus 4.7, Opus 4.6, **Sonnet 4.6**, Sonnet 4.5, Opus 4.5. Opus 4.1 está deprecado (retiro: 2026-08-05).
>
> **Nota de naming:** desde la generación 4.6 los IDs usan formato sin fecha (`claude-opus-4-8`) pero siguen siendo snapshots fijos, no punteros evergreen.

### Principios clave

- **Regla de oro:** Muestra tu prompt a un colega con mínimo contexto. Si se confunde, Claude también.
- **Sé directo** — da instrucciones explícitas en lugar de insinuaciones
- **"Ve más allá" cuando quieras excelencia** — si no lo pides explícitamente, Claude da respuestas estándar

### Estructura de prompts largos

1. **Identity** — propósito y estilo del asistente
2. **Instructions** — reglas y comportamiento
3. **Examples** — ejemplos de entrada/salida (3-5, diversos)
4. **Context** — datos adicionales, documentos de referencia

### Técnica específica: XML tags

Claude responde muy bien a XML para estructurar prompts complejos:

```xml
<instructions>
Tu tarea es analizar...
</instructions>

<context>
{{DATOS}}
</context>

<examples>
<example>
<input>...</input>
<output>...</output>
</example>
</examples>
```

### Contexto largo (>20k tokens)

- **Pon los documentos AL INICIO** del prompt, arriba de las instrucciones
- **La query va AL FINAL** — mejora la respuesta hasta 30%
- **Usa `<document>` tags** con metadata (`<source>`, `<index>`)
- **Pide citas antes de análisis** — "Primero cita las partes relevantes, luego analiza"

### Formato de salida

- Para reducir markdown: describe el estilo deseado en positivo ("escribe en prosa fluida con párrafos completos")
- Para formatos específicos: usa XML format indicators
- La forma del prompt influencia la forma de la respuesta
- Claude Opus 4.8 usa LaTeX por defecto para matemáticas — desactívalo con instrucciones explícitas si prefieres texto plano

### Verbosidad

- Los modelos Claude 4.x recientes son más concisos por defecto que versiones anteriores
- Si quieres más detalle, pídelo: "proporciona un resumen después de cada tool call"
- Los modelos más nuevos omiten verbalizaciones innecesarias

### Tool Use

- **Instrucciones explícitas:** Si quieres que Claude tome acción, sé explícito: "Cambia esta función" en lugar de "¿Puedes sugerir cambios?"
- **Parallel tool calling:** Claude (4.6+) ejecuta múltiples herramientas en paralelo automáticamente. Puedes guiar este comportamiento con prompts.
- **Proactividad:** Puedes hacer que Claude sea más proactivo o más conservador mediante system prompts.

### Thinking, Reasoning y parámetro `effort`

Los modelos recientes usan **adaptive thinking**: el modelo decide dinámicamente cuándo y cuánto pensar. Fable 5, Mythos 5 y Sonnet 5 lo tienen **activo por defecto** (en Fable 5/Mythos 5 `thinking: {type: "disabled"}` es rechazado); en Opus 4.8/4.7/4.6 y Sonnet 4.6 se activa con `thinking: {type: "adaptive"}`.

El control recomendado de profundidad es el parámetro **`effort`** (reemplaza a `budget_tokens`, que está deprecado). Afecta a **TODOS** los tokens de la respuesta: texto, llamadas a herramientas y thinking.

**Niveles de `effort`** (de mayor a menor gasto):

| Nivel | Uso típico | Disponibilidad de los niveles tope |
|---|---|---|
| `max` | Razonamiento más profundo, sin límite de tokens | Fable 5, Mythos 5, Opus 4.8/4.7/4.6, Sonnet 5, Sonnet 4.6 |
| `xhigh` | Trabajo agentic/coding de largo alcance (>30 min, millones de tokens) | Fable 5, Mythos 5, Opus 4.8, Opus 4.7, Sonnet 5 |
| `high` | **Default.** Razonamiento complejo, coding difícil, tareas agenticas | Todos |
| `medium` | Equilibrio velocidad/costo/rendimiento | Todos |
| `low` | Máxima eficiencia (subagentes, tareas simples, latencia baja) | Todos |

- **Default:** el API usa `high` en todas las superficies (incluido Claude Code). `effort: "high"` ≡ omitir el parámetro.
- **Opus 4.8 (y 4.7):** empieza con `xhigh` para coding/agentic; usa `high` para el resto de cargas sensibles a inteligencia; baja a `medium`/`low` solo si tus evals confirman que mantienen calidad.
- **Fable 5 (y Mythos 5):** empieza con `high` (default); usa `xhigh` para lo más exigente; `medium`/`low` rinden bien en trabajo rutinario.
- **Sonnet 5:** default `high`; adaptive thinking activo por defecto. Sube a `xhigh` para coding/agentic; baja a `medium`/`low` para chat y cargas sensibles a latencia.
- **Sonnet 4.6 (legacy):** se recomienda fijar `medium` explícitamente como default para evitar latencia inesperada.
- En `xhigh`/`max`, fija un `max_tokens` grande (64K es un buen punto de partida) para dar espacio a thinking + tool calls.
- En Opus 4.8/4.7, el thinking manual (`thinking: {type:"enabled", budget_tokens:N}`) **ya no se soporta y devuelve error 400**; usa adaptive thinking + `effort`.

```python
client.messages.create(
    model="claude-opus-4-8",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "xhigh"},  # coding/agentic
    messages=[{"role": "user", "content": "..."}],
)
```

> **Tip Claude Code:** el modo `ultracode` = `effort: xhigh` + permiso para lanzar workflows multi-agente; no es un nivel de `effort` adicional del API.

### Agentic Systems

- **Long-horizon reasoning:** Claude mantiene orientación en sesiones extendidas
- **Context awareness:** Los modelos rastrean su ventana de contexto restante
- **Multi-context workflows:** Usa diferentes prompts para la primera ventana de contexto
- **State management:** Usa formatos estructurados (JSON) para datos de estado, git para tracking

### Migración a Opus 4.8 / Fable 5

1. **Sé específico sobre comportamiento deseado**
2. **Elimina los prefilled responses** — ya no se soportan desde Claude 4.6 (devuelven error 400)
3. **Ajusta el anti-laziness prompting** — los modelos recientes son más proactivos
4. **Migra de `budget_tokens` a `effort`** — usa adaptive thinking con el parámetro `effort`
5. **Sube el `effort` antes de "promptear alrededor"** — si ves razonamiento superficial en problemas complejos, sube el nivel en vez de añadir instrucciones

---

## OpenAI — GPT

Fuente: [platform.openai.com/docs/guides/prompt-engineering](https://platform.openai.com/docs/guides/prompt-engineering)

### Modelos actuales (Julio 2026)

- **GPT-5.5** — Modelo flagship. 1M contexto, 128K output, cutoff Dic 2025. Soporta `reasoning_effort`. ($5 / $30 por MTok)
- **GPT-5.5-pro** — Opción de mayor inteligencia vía API (recomendado en los docs de reasoning para lo más difícil). ($30 / $180 por MTok)
- **GPT-5.4** — Generación previa de alto rendimiento ($2.50 / $15 por MTok)
- **GPT-5.4-pro** — Variante pro de la generación 5.4 ($30 / $180 por MTok)
- **GPT-5.4-mini** — Equilibrio costo/capacidad, 400K contexto ($0.75 / $4.50 por MTok)
- **GPT-5.4-nano** — El más rápido y económico para tareas de alto volumen
- Todos los modelos GPT-5.x son razonadores: generan cadena de pensamiento interna y aceptan `reasoning_effort`.

> **En preview:** **GPT-5.6** está disponible en preview limitado (partners seleccionados); disponibilidad amplia "próximamente", sin precio público aún.

### Estructura de mensajes

OpenAI usa roles jerárquicos:

- **developer** — instrucciones del desarrollador (mayor prioridad, reemplaza a "system")
- **user** — input del usuario
- **assistant** — respuestas del modelo

Piensa en developer como la función y user como los argumentos.

### Componentes del prompt

1. **Identity** — propósito, estilo de comunicación, objetivos
2. **Instructions** — reglas, qué hacer y qué no hacer
3. **Examples** — inputs deseados con outputs esperados
4. **Context** — datos adicionales, documentos, código relevante

### Técnica: Markdown + XML

- **Markdown headers** para secciones lógicas y jerarquía
- **XML tags** para delimitar contenido de referencia (documentos, datos)
- **XML attributes** para metadata sobre el contenido

### Prompt caching

- Pon contenido que se repite al INICIO del prompt
- Entre los primeros parámetros del request JSON
- Esto ahorra costo y latencia en requests repetidas

### Prompts reutilizables (DEPRECADO — versiona en código)

⚠️ Los objetos de prompt reutilizables del dashboard fueron des-enfatizados (2026-06-03) y el endpoint `v1/prompts` se apaga el **2026-11-30**.

- **Recomendación actual:** versiona tus prompts **en tu propio código/repositorio**, no como objetos en el dashboard
- Trátalos como código: control de versiones, code review, evals automatizadas
- Usa plantillas/placeholders propios (ej: `{{customer_name}}`) e inyéctalos desde tu aplicación

### Control de razonamiento (`reasoning_effort`)

- Los modelos GPT-5.x generan cadena de pensamiento interna; mejor para tareas complejas multi-paso
- Controla la profundidad con el parámetro **`reasoning_effort`**: `none`, `minimal`, `low`, `medium`, `high`, `xhigh`
  - Los valores soportados son **dependientes del modelo** (algunos aceptan solo un subconjunto). `gpt-5.5` usa `medium` por defecto.
  - `none` / `minimal` / `low` → respuestas rápidas y económicas para tareas simples
  - `high` / `xhigh` → razonamiento profundo para problemas difíciles (más lento y caro)
- Benefician de instrucciones más explícitas sobre CÓMO lograr la tarea
- Sé explícito con el formato y los criterios de éxito; evita CoT manual redundante (ya razonan internamente)

### Agentic Tasks

Para tareas agenticas y de largo alcance con GPT-5.5:

- **Planificación y persistencia:** Instruye al modelo a resolver completamente la query antes de ceder control
- **Preambles para transparencia:** Pide al modelo que explique por qué llama a una herramienta
- **Progress tracking con TODOs:** Usa herramientas de lista TODO para tracking estructurado

### Frontend Engineering

GPT-5.5 es excelente construyendo frontends complejos:

- **Librerías recomendadas:** Tailwind CSS, shadcn/ui, Radix Themes (styling), Lucide/Material Symbols (icons), Motion (animación)
- **Zero-to-one web apps:** Puede generar apps completas desde un solo prompt
- **Integración con codebases grandes:** Instrucciones sobre principios, UI/UX, estructura, componentes

### Versionado

- Fija versiones con snapshot fechado en producción (ej: `gpt-5.5-YYYY-MM-DD`) en lugar del alias evergreen `gpt-5.5`
- Construye evals para medir comportamiento de tus prompts
- Monitorea performance al cambiar modelos

---

## Google — Gemini

Fuente: [Gemini 3 prompting guide — Google Cloud](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/start/gemini-3-prompting-guide) (oficial)

### Modelos actuales (Julio 2026)

- **Gemini 3.1 Pro** — Modelo flagship / máxima capacidad; razonamiento-first para workflows agenticos complejos y coding, adaptive thinking, 1M contexto, grounding integrado. (Actualmente en estado **Preview**)
- **Gemini 3.5 Flash** — GA y modelo **Featured**: inteligencia cercana a Pro a costo/velocidad de Flash (coding nivel Pro, ejecución agentic en paralelo, 1M contexto)
- **Gemini 3.1 Flash-Lite** — El más económico, optimizado para baja latencia y **tráfico de alto volumen sensible a costo**
- **Gemini Omni Flash** (Preview) — Modelo Flash para generar/editar video desde texto o assets de referencia
- También disponibles: Gemini 3 Flash, variantes de imagen (3.1 Flash Image, 3 Pro Image), y los previos Gemini 2.5 Pro / 2.5 Flash

> Gemini 3 supera significativamente a 2.5 en todas las tareas, prioriza la lógica sobre la verbosidad y es **menos verboso por defecto**. No existe (aún) "Gemini 3.5 Pro" ni "Gemini 4": la línea Pro llega hasta 3.1 Pro.

### Parámetros recomendados (oficial)

- **Temperature = 1.0 (default):** Google **recomienda fuertemente mantener `temperature` en su valor por defecto de `1.0`**. El razonamiento de Gemini 3 está optimizado para ese valor.
  - ⚠️ Bajarla (<1.0) puede causar **comportamiento inesperado, loops o degradación**, sobre todo en tareas matemáticas/razonamiento.
- **Latencia baja:** fija el thinking level en `LOW` y usa una system instruction como `think silently`.

### Verbosidad

- Por defecto Gemini 3 es **directo y poco verboso**.
- Si necesitas un tono conversacional, **debes pedirlo explícitamente**, p.ej.:
  > `Explain this as a friendly, talkative assistant.`

### Estructura del prompt y posición de instrucciones (oficial)

Gemini 3 puede **descartar restricciones negativas, de formato o cuantitativas** (conteos de palabras, etc.) si aparecen demasiado pronto. Coloca tu petición central y tus restricciones más críticas como **última línea**. Orden recomendado:

```
[Contexto y material fuente]
[Instrucciones de la tarea principal]
[Restricciones negativas, de formato y cuantitativas]   ← AL FINAL
```

- **Contexto largo (libros, codebases, videos):** coloca la pregunta/instrucción **AL FINAL, después de los datos**. Ancla el razonamiento abriendo con `Based on the entire document above...`
  > Evita que el modelo deje de procesar tras la primera coincidencia relevante.

### Deducción vs. información externa (oficial)

Negativos amplios como `do not infer` o `do not guess` **son contraproducentes**: el modelo puede sobre-indexar y fallar en lógica/aritmética básica.

- ❌ Poco efectivo: `What was the profit? Do not infer.`
- ✅ Efectivo:
  > `You are expected to perform calculations and logical deductions based strictly on the provided text. Do not introduce external information.`

### Grounding: el contexto como única fuente de verdad

Para hipótesis que contradicen el mundo real, el modelo puede revertir a sus datos de entrenamiento. Declara explícitamente que el contexto proporcionado es la **única** fuente de verdad de la sesión y que debe reportar solo lo que aparece, sin inferir.

### Verificación en dos pasos (anti-alucinación)

Para datos oscuros o capacidades que el modelo quizá no tenga (p.ej. acceder a una URL en vivo), divide el prompt: **1)** verifica que la info/capacidad existe; **2)** solo entonces genera.

> `Verify with high confidence if you're able to access [X]. If you cannot verify, state 'No Info' and STOP. If verified, proceed to generate a response.`

### Personas (toma el rol en serio)

Gemini 3 trata el rol asignado con seriedad y **a veces ignora instrucciones** para mantener la coherencia con la persona. Revisa el rol asignado y evita ambigüedades.

> `You are a data extractor. You are forbidden from clarifying, explaining, or expanding terms. Output text exactly as it appears. Do not explain why.`

### Estructura recomendada (los 4 elementos)

1. **Persona** — quién es Gemini
2. **Task** — qué hacer claramente
3. **Context** — background, audiencia, propósito
4. **Format** — cómo debe verse la salida

Usa tags XML-style o Markdown para delimitar secciones; mantén la estructura consistente y define términos ambiguos. Usa imperativos (`Crea una lista...`) en vez de preguntas.

---

## xAI — Grok

Fuente: [docs.x.ai/docs/models](https://docs.x.ai/docs/models) (oficial)

### Modelos actuales (Julio 2026)

- **Grok 4.3** — Modelo flagship unificado (chat + coding). 1M contexto. Soporta `reasoning_effort` (`none`/`low` (default)/`medium`/`high`). ($1.25 / $2.50 por MTok)
- **grok-4.20** (`grok-4.20-0309`) — Disponible en variantes razonadora y no-razonadora
- **grok-4.20-multi-agent** (`grok-4.20-multi-agent-0309`) — Multi-agente: aquí `reasoning.effort` controla **cuántos agentes colaboran** (4 o 16), no la profundidad; añade el nivel `xhigh`. 1M contexto ($1.25 / $2.50 por MTok)
- **grok-build-0.1** — Enfocado en coding agentic, 256K contexto ($1 / $2 por MTok)

> Los IDs aceptan aliases `-latest` y fechados `-<YYYYMMDD>`. Fija el ID fechado en producción.

### Para coding y tareas agenticas

1. **Proveer contexto necesario:** Selecciona código específico, especifica file paths, project structures
2. **Establecer metas explícitas:** Define objetivos y problemas específicos
3. **Refinar continuamente:** Itera rápido aprovechando velocidad y costo
4. **Usa native tool calling** — diseñado para esto, mejor que XML-based
5. **System prompt detallado:** Describe tarea, expectativas, casos edge
6. **Optimiza para cache hits** — un prefix estable se recupera de cache automáticamente

### Fórmula de 4 partes

1. **Goal** — objetivo exacto en una oración
2. **Context** — audiencia, inputs, restricciones, paths de archivos
3. **Output Format** — estructura, tabla, JSON, longitud, estilo de código
4. **Quality Bar** — qué incluir, qué evitar, cómo manejar incertidumbre

### Características y notas de API

- **Sin restricción en el orden de roles:** `system`, `user` y `assistant` pueden ir en cualquier orden
- **Datos en tiempo real:** requiere habilitar **Web Search / X Search** explícitamente (no es automático)
- **Razonamiento (grok-4.3):** se controla con `reasoning_effort` (`none`/`low`/`medium`/`high`); expone razonamiento **resumido** en `reasoning_content` y, opcionalmente, razonamiento **cifrado** vía `include: ["reasoning.encrypted_content"]` (no el CoT crudo)
- **Restricción en modelos razonadores:** `presencePenalty`, `frequencyPenalty` y `stop` **no se pueden usar** con modelos de razonamiento (la petición devuelve error)
- **`logprobs` / `top_logprobs` no están soportados** en `grok-4.20` y posteriores (se ignoran silenciosamente si se envían)
- **Herramientas nativas** sobre XML para mejor performance

### Errores comunes

| ❌ Mal prompt | ✅ Buen prompt | Por qué mejora |
|---|---|---|
| "Crea un food tracker" | "Crea un food tracker que muestre consumo calórico diario por nutriente, con overview y tendencias, output como dashboard interactivo" | Agrega contexto, formato y detalles |
| "Fix my code" | "Agrega error handling a @sql.ts usando códigos de @errors.ts; incluye bloques try-catch" | Especifica archivos y requisitos |

### Evitar vaguedad

Grok penaliza más que otros modelos los prompts vagos. Siempre especifica.

---

## Meta — LLaMA

Fuentes: [AWS — Llama 3 prompting](https://aws.amazon.com/blogs/machine-learning/best-prompting-practices-for-using-meta-llama-3-with-amazon-sagemaker-jumpstart/), disponibilidad de modelos vía Vertex AI Model Garden

*Nota: la guía oficial de prompting de llama.com no está accesible actualmente. Las técnicas siguientes son model-agnostic; el chat template mostrado corresponde a Llama 3.x.*

### Modelos actuales (Julio 2026)

- **Llama 4 Maverick** — Modelo grande multimodal (texto + imagen), el más capaz de la familia Llama 4
- **Llama 4 Scout** — Variante más eficiente, contexto largo
- **Llama 3.3** — Modelo de texto previo, ampliamente soportado

> Llama 4 (Maverick/Scout) es multimodal de forma nativa. Para detalles de plantilla de chat de Llama 4, consulta la documentación del proveedor (los tokens difieren de Llama 3.x).

### Formato de chat template (Llama 3.x)

```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>

You are a helpful AI assistant<|eot_id|><|start_header_id|>user<|end_header_id|>

What can you help me with?<|eot_id|><|start_header_id|>assistant<|end_header_id|>
```

### Técnicas principales

- **Zero-shot** para tareas simples
- **Few-shot** (1-3 ejemplos) para tareas que necesitan guía
- **Chain-of-thought (CoT)** — "Piensa paso a paso antes de responder"
- **Task decomposition** — divide tareas complejas en subtareas
- **Meta prompting** — estructuras lógicas abstractas (Step 1, Step 2, Step 3)

### Parámetros de inferencia

- **Temperature (0-1):** bajo (0.1) para factual, alto (0.7) para creativo
- **Top-k:** bajo para enfoque, alto para diversidad
- **Top-p:** Controla opciones de token basado en probabilidad
- **Stop sequences:** `<|start_header_id|>`, `<|end_header_id|>`, `<|eot_id|>`
- **Minimiza tokens:** itera prompts, divide tareas, acorta outputs

### Limitaciones

- Sin memoria persistente entre sesiones — gestiona contexto manualmente
- Para salidas estructuradas: pide JSON, tablas, markdown explícitamente
- Soporta imágenes vía uploads con instrucciones de análisis

### Chat de múltiples turnos

- Re-enuncia contexto clave en conversaciones largas
- Pide formatos específicos: tablas, bloques de código, markdown
- Separa lógicamente cada subtarea

---

## DeepSeek

Fuente: [api-docs.deepseek.com](https://api-docs.deepseek.com/) (oficial)

### DeepSeek V4 (Julio 2026)

DeepSeek V4 reemplaza al modelo unificado V3.2. Ahora hay **dos modelos** y el razonamiento se controla con un **toggle de thinking** sobre el mismo modelo (ya no son endpoints separados):

| | `deepseek-v4-flash` | `deepseek-v4-pro` |
|---|---|---|
| Uso | Alto volumen, rápido y económico | Máxima capacidad |
| Contexto | 1M tokens | 1M tokens |
| Max output | 384K tokens | 384K tokens |
| Precio cache-hit (in) | $0.0028 / 1M | $0.003625 / 1M |
| Precio cache-miss (in) | $0.14 / 1M | $0.435 / 1M |
| Precio output | $0.28 / 1M | $0.87 / 1M |

> ⚠️ **Deprecación:** `deepseek-chat` y `deepseek-reasoner` se retiran el **2026-07-24 15:59 UTC**. Migra a `deepseek-v4-flash` / `deepseek-v4-pro`.

### Thinking: toggle y `reasoning_effort`

- **Thinking activo por defecto** (`enabled`). Conmútalo con:
  ```json
  { "thinking": { "type": "enabled" } }   // o "disabled"
  ```
- **`reasoning_effort`** acepta `high` y `max` (los valores `low`/`medium` se mapean a `high`; `xhigh` se mapea a `max`).
- Los **agentes de coding (Claude Code, OpenCode, etc.) usan `max` automáticamente**.
- ⚠️ **Con thinking activo NO se soportan** `temperature`, `top_p` ni penalties (se ignoran).
- La cadena de pensamiento se expone en el campo **`reasoning_content`**.
- **Tool calls** y **JSON** funcionan en ambos modos; **FIM** (fill-in-the-middle) solo sin thinking.

### Reglas fundamentales

1. **Elige el modelo por costo/capacidad** (flash vs pro) y **activa/desactiva thinking** según la tarea (razonamiento complejo → thinking on; structured output simple/baja latencia → thinking off)
2. **Name the artifact** — especifica EXACTAMENTE qué quieres: diff, JSON, tabla, resumen de 200 palabras, archivo pytest. No pidas "ayuda" o "explica"
3. **Less is more con thinking activo** — elimina system prompts elaborados, few-shot y "think step by step"; el modelo razona internamente y el scaffolding externo interfiere

### Con thinking activo

- Prompt mínimo: solo el problema, sin instrucciones de razonamiento ni ejemplos de CoT
- No fijes `temperature`/`top_p` (se ignoran)
- Nombra el artefacto de salida claramente

### Sin thinking (modo directo)

- System prompts detallados y persona instructions sí funcionan
- `temperature`/`top_p` completamente ajustables
- Estructura con secciones explícitas; **los prompts con XML** funcionan excepcionalmente bien
- Óptimo para structured outputs, herramientas y FIM

### Context Caching (automático)

DeepSeek cachea prefixes repetidos a nivel de API. El cache-hit es ~50x más barato que cache-miss (p.ej. flash: $0.0028 vs $0.14 por 1M).

- Coloca system prompts y contenido estable **al inicio** para maximizar cache hits

---

## Comparativa rápida

| Aspecto | Claude (Opus 4.8/Fable 5) | GPT-5.5 | Gemini 3.1 | Grok 4.3 | LLaMA 4 | DeepSeek V4 |
|---|---|---|---|---|---|---|
| Estructura preferida | XML tags | Markdown + XML | 4 elementos + XML/MD | 4-part formula | Chat template | Minimal (thinking) / Estructurado |
| Ejemplos | 3-5, diversos | 3-5 | 1+ | Incluir explícitos | 1-3 (few-shot) | Evitar con thinking on |
| Contexto largo | Docs al inicio, query al final | Cacheable al inicio | Instrucciones al final | Context upfront | Re-enunciar en turnos | Prefix estable al inicio |
| Rol/Persona | System prompt | developer role | Persona explícita (la toma en serio) | Roles en cualquier orden | Header role | System prompt (thinking off) |
| Formato salida | Prosa o XML | Markdown + XML | Restricciones AL FINAL | Especificar explícitamente | Especificar explícitamente | Name the artifact |
| Thinking/Reasoning | Adaptive thinking + `effort` (low→max/xhigh) | `reasoning_effort` (none→xhigh, incl. `minimal`) | Adaptive; `LOW` + "think silently" | `reasoning_effort` (none→high); razonamiento resumido | Chain-of-thought | Toggle `thinking` + `reasoning_effort` (high/max) |
| Parámetros | `effort` afecta todos los tokens | Roles developer/user/assistant; valores effort model-dependent | **`temperature=1.0`** (no bajar) | `logprobs` no en 4.20+; penalties/`stop` dan error en razonadores | temperature/top-k/top-p | Sin `temperature` con thinking on |
| Errores comunes | Ser vago; usar prefill (400) | Saltar estructura | Bajar temperature; negativos amplios | Ser vago | No usar template | Usar thinking innecesariamente |

---

## Reglas prácticas transversales

### Para TODOS los modelos

1. **Sé explícito sobre formato de salida** — no dejes que el modelo decida
2. **Usa 3-5 ejemplos** cuando el output debe seguir un patrón específico
3. **Separa instrucciones de datos** con tags/secciones
4. **Especifica restricciones claras** — idioma, longitud, tono, audiencia
5. **Itera** — primer prompt rara vez es óptimo; refina según resultados

### Para agentes y automatización

1. **Asigna rol en system prompt** — no en el user message
2. **Define comportamiento de edge cases** — qué hacer cuando no hay información suficiente
3. **Usa chain-of-thought solo cuando sea necesario** — no todos los modelos lo necesitan (DeepSeek thinking mode lo hace internamente)
4. **Versiona tus prompts** — trátalos como código
5. **Construye evals** — mide si tus prompts funcionan consistentemente

### Para reducir costos

1. **Prompt caching** — contenido repetido al inicio del prompt (cache-hit hasta ~50x más barato en DeepSeek)
2. **Modelos pequeños para tareas simples** — no uses Opus/GPT-5.5/V4-pro para resúmenes básicos; usa Haiku, GPT-5.4-nano, Flash, v4-flash
3. **Baja el `effort`/`reasoning_effort`** para tareas simples o sensibles a latencia
4. **Salidas concisas** — pide "responde en máximo X líneas"
5. **Evita system prompts gigantes** — solo incluye lo necesario

### Errores universales

1. ❌ Ser vago y esperar que el modelo adivine
2. ❌ No dar ejemplos cuando el output debe ser consistente
3. ❌ Mezclar instrucciones con datos en el mismo bloque
4. ❌ No especificar formato de salida
5. ❌ Usar el modelo más caro para todo
6. ❌ Usar thinking mode innecesariamente (DeepSeek)
7. ❌ No aprovechar prompt caching

---

*Compilado: 2026-07-02 | Fuentes oficiales: Anthropic, OpenAI, Google, xAI, Meta, DeepSeek*
