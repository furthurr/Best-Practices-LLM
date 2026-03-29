# Guía de Buenas Prácticas de Prompting (Actualizada 2026)

Referencia compilada de guías oficiales de los principales proveedores de LLMs.

**Fuentes:** Anthropic (Claude), OpenAI (GPT), Google (Gemini), xAI (Grok), Meta (LLaMA), DeepSeek

**Fecha:** 2026-03-28

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

### Modelos actuales (Marzo 2026)

- **Claude Opus 4.6** — Máxima capacidad, tareas complejas de largo alcance
- **Claude Sonnet 4.6** — Equilibrio entre velocidad, costo e inteligencia
- **Claude Haiku 4.5** — Rápido y económico para tareas simples

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
- Claude Opus 4.6 usa LaTeX por defecto para matemáticas — desactívalo con instrucciones explícitas si prefieres texto plano

### Verbosidad

- Claude 4.6 es más conciso por defecto que versiones anteriores
- Si quieres más detalle, pídelo: "proporciona un resumen después de cada tool call"
- Los modelos más nuevos omiten verbalizaciones innecesarias

### Tool Use

- **Instrucciones explícitas:** Si quieres que Claude tome acción, sé explícito: "Cambia esta función" en lugar de "¿Puedes sugerir cambios?"
- **Parallel tool calling:** Claude 4.6 ejecuta múltiples herramientas en paralelo automáticamente. Puedes guiar este comportamiento con prompts.
- **Proactividad:** Puedes hacer que Claude sea más proactivo o más conservador mediante system prompts.

### Thinking and Reasoning (NUEVO)

Claude 4.6 introduce **adaptive thinking** donde el modelo decide dinámicamente cuándo y cuánto pensar:

- **Configuración:** `thinking: {type: "adaptive"}`
- **Control de esfuerzo:** Usa el parámetro `effort` ("low", "medium", "high", "max") en lugar de `budget_tokens`
- **Cuándo usar:** Tareas complejas multi-paso, agentic coding, razonamiento extendido
- **Ejemplo:**
```python
client.messages.create(
    model="claude-opus-4-6",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "..."}],
)
```

### Agentic Systems

- **Long-horizon reasoning:** Claude mantiene orientación en sesiones extendidas
- **Context awareness:** Los modelos rastrean su ventana de contexto restante
- **Multi-context workflows:** Usa diferentes prompts para la primera ventana de contexto
- **State management:** Usa formatos estructurados (JSON) para datos de estado, git para tracking

### Migración desde versiones anteriores

1. **Sé específico sobre comportamiento deseado**
2. **Migra away de prefilled responses** — ya no soportadas en Claude 4.6
3. **Ajusta anti-laziness prompting** — Claude 4.6 es más proactivo
4. **Actualiza thinking configuration** — usa adaptive thinking con effort parameter

---

## OpenAI — GPT

Fuente: [platform.openai.com/docs/guides/prompt-engineering](https://platform.openai.com/docs/guides/prompt-engineering)

### Modelos actuales (Marzo 2026)

- **GPT-5.4** — Último modelo de la serie GPT-5
- **GPT-5** — Modelo principal de la serie GPT
- **Reasoning models (o-series)** — Generan cadena de pensamiento interna

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

### Reusable prompts

- Crea prompts reutilizables en el dashboard de OpenAI
- Usa placeholders: `{{customer_name}}`
- Referencia por ID en el API: `prompt: { id: "pmpt_abc123", variables: {...} }`

### Modelos razonadores (o-series)

- Generan cadena de pensamiento interna
- Mejor para tareas complejas multi-paso
- Más lentos y caros que GPT estándar
- Benefician de instrucciones más explícitas sobre CÓMO lograr la tarea
- **NUEVO:** Usan el parámetro `reasoning: {effort: "low"}` para controlar profundidad

### Agentic Tasks (NUEVO)

Para tareas agenticas y de largo alcance con GPT-5:

- **Planificación y persistencia:** Instruye al modelo a resolver completamente la query antes de ceder control
- **Preambles para transparencia:** Pide al modelo que explique por qué llama a una herramienta
- **Progress tracking con TODOs:** Usa herramientas de lista TODO para tracking estructurado

### Frontend Engineering (NUEVO)

GPT-5 es excelente construyendo frontends complejos:

- **Librerías recomendadas:** Tailwind CSS, shadcn/ui, Radix Themes (styling), Lucide/Material Symbols (icons), Motion (animación)
- **Zero-to-one web apps:** Puede generar apps completas desde un solo prompt
- **Integración con codebases grandes:** Instrucciones sobre principios, UI/UX, estructura, componentes

### Versionado

- Fija versiones específicas en producción (ej: `gpt-4.1-2025-04-14`)
- Construye evals para medir comportamiento de tus prompts
- Monitorea performance al cambiar modelos

---

## Google — Gemini

Fuentes: [workspace.google.com/blog](https://workspace.google.com/blog/productivity-collaboration/because-you-asked-take-prompting-to-the-next-level), [philschmid.de](https://www.philschmid.de/gemini-3-prompt-practices)

### Gemini 3 Pro (Noviembre 2025)

Gemini 3 Pro es significativamente mejor que 2.5 Pro en todas las tareas. Favorito sobre persuasión, lógica sobre verbosidad.

### Los 4 elementos esenciales

1. **Persona** — quién es Gemini (ej: "Eres un gerente de marketing senior")
2. **Task** — qué hacer claramente
3. **Context** — background, audiencia, propósito, objetivo
4. **Format** — cómo debe verse la salida

Si falta alguno, Gemini lo adivina → mayor riesgo de error.

### Principios clave de Gemini 3

- **Instrucciones precisas:** Sé conciso. Gemini 3 responde mejor a instrucciones directas y claras.
- **Consistencia y parámetros definidos:** Mantén estructura uniforme (ej: tags XML estandarizados) y define términos ambiguos explícitamente.
- **Output Verbosity:** Por defecto es menos verboso. Si quieres tono conversacional, pídelo explícitamente.
- **Multimodal Coherence:** Texto, imágenes, audio y video son inputs de igual clase. Instrucciones deben referenciar modalidades específicas.
- **Constraint Placement:** Coloca restricciones de comportamiento y roles en la System Instruction o al inicio del prompt.
- **Long Context Structure:** Con contextos largos (libros, codebases), coloca instrucciones AL FINAL después de los datos.
- **Context Anchoring:** Al transicionar de un bloque de datos a tu query, usa frases puente como "Basándote en la información anterior..."

### Reasoning and Planning (NUEVO)

- **Explicit Planning & Decomposition:** Antes de la respuesta final, parsea el objetivo en sub-tareas, verifica si la información está completa, crea un outline estructurado.
- **Self-updating TODO Tracker:** Crea una lista TODO para trackear progreso.
- **Critique its own output:** Revisa la salida generada contra las restricciones originales del usuario.

### Structured Prompting (NUEVO)

Usa XML-style tagging o Markdown para estructurar prompts. Proporciona límites inequívocos. No mezcles XML y Markdown; elige uno.

**Ejemplo XML:**
```xml
<rules>
    1. Be objective.
    2. Cite sources.
</rules>

<planning_process>
    1. Analyze the Request
    2. Decompose
    3. Strategize
    4. Verify
</planning_process>

<context>
    [Insert User Input Here]
</context>
```

### Agentic Tool Use (NUEVO)

- **The Persistence Directive:** Eres un agente autónomo. Continúa trabajando hasta que la query esté COMPLETAMENTE resuelta.
- **Pre-Computation Reflection:** Antes de llamar a cualquier herramienta, explica: por qué la llamas, qué datos esperas, cómo ayudan a resolver el problema.

### Formulación efectiva

- **Usa imperativos**, no preguntas: "Crea una lista..." en vez de "¿Puedes crear una lista?"
- **21 palabras promedio** es el sweet spot — la mayoría de usuarios da menos de 9
- **Incluye detalles de contexto:** timing, metas, audiencia, longitud de sesión

### Enfoque iterativo

- Trata prompting como conversación, no monólogo
- Cada respuesta es una oportunidad para refinar el siguiente prompt
- Pide a Gemini que genere preguntas de clarificación: "¿Qué 10 preguntas debo hacer para confirmar que esto alinea con mi estrategia?"

### Posición de instrucciones

- **Contextos largos (libros, codebases, videos):** pon instrucciones AL FINAL después de los datos
- **Prompts cortos:** pon restricciones de comportamiento y roles AL INICIO

### Referencias cruzadas

- Incluye referencias a archivos de Google Workspace: `@DocumentName en Docs`
- Crea status updates que referencien múltiples archivos de Drive
- Requiere habilitar Google Workspace extensions

---

## xAI — Grok

Fuentes: [docs.x.ai](https://docs.x.ai/developers/advanced-api-usage/grok-code-prompt-engineering), [promptbuilder.cc](https://promptbuilder.cc/blog/grok-prompt-guide-2026)

### Grok Code Fast 1 (NUEVO)

Modelo agentic ligero diseñado para pair-programming en herramientas de coding:

- **4x más rápido** que modelos comparables
- **1/10 del costo** de otros modelos agentic
- **Reasoning model** con tool-calling intercalado durante thinking
- **Thinking traces** accesibles via streaming: `chunk.choices[0].delta.reasoning_content`

### Para usuarios de Grok Code Fast 1

1. **Proveer contexto necesario:** Selecciona código específico, especifica file paths, project structures
2. **Establecer metas explícitas:** Define objetivos y problemas específicos
3. **Refinar continuamente:** Aprovecha la velocidad y costo para iterar rápidamente
4. **Asignar tareas agenticas:** Ideal para navegar grandes cantidades de código con herramientas

### Para desarrolladores construyendo coding agents

- **Usa native tool calling** — diseñado específicamente para esto, mejor que XML-based
- **System prompt detallado:** Describe tarea, expectativas, casos edge
- **Introduce contexto con XML tags o Markdown** para claridad
- **Optimiza para cache hits** — prefix estable se recupera de cache automáticamente

### Fórmula de 4 partes (Prompt Builder)

1. **Goal** — objetivo exacto en una oración
2. **Context** — audiencia, inputs, restricciones, paths de archivos
3. **Output Format** — estructura, tabla, JSON, longitud, estilo de código
4. **Quality Bar** — qué incluir, qué evitar, cómo manejar incertidumbre

### 30 Prompts por Caso de Uso (NUEVO)

**Marketing (10 prompts):**
- Landing page rewrite, SEO title/meta, email sequence, ad angle testing, competitor teardown, LinkedIn calendar, case study, ecommerce PDP, internal linking, weekly performance summary

**Coding (10 prompts):**
- Refactor request, bug triage, unit tests, API contract validator, PR review, SQL optimization, migration plan, error handling, feature spec, release notes

**Content (10 prompts):**
- Blog outline, article rewrite, FAQ generator, newsletter, YouTube script, content repurposing, editorial brief, fact-check checklist, content update, editorial QA

### Características únicas de Grok

- **Think Mode** — para razonamiento complejo
- **DeepSearch** — datos en tiempo real de X/Twitter
- **4x más rápido** que modelos comparables → soporta iteración rápida
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

Fuente: [aws.amazon.com](https://aws.amazon.com/blogs/machine-learning/best-prompting-practices-for-using-meta-llama-3-with-amazon-sagemaker-jumpstart/)

*Nota: La fuente oficial de llama.com no está disponible actualmente. Información basada en documentación de AWS para Llama 3.*

### Formato de chat template

LLaMA usa formato específico:

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

Fuentes: [chat-deep.ai](https://chat-deep.ai/guide/deepseek-prompts/), [yuv.ai](https://yuv.ai/learn/deepseek)

### DeepSeek V3.2 (Marzo 2026) — Modelo Unificado

DeepSeek ahora ejecuta un **modelo unificado V3.2** en dos modos:

| Característica | Thinking Mode (deepseek-reasoner) | Chat Mode (deepseek-chat) |
|---|---|---|
| Modelo subyacente | DeepSeek-V3.2 | DeepSeek-V3.2 |
| System prompt | Soportado pero mantener mínimo | Usar libremente para personas |
| Few-shot examples | Degrada performance | Funciona bien |
| "Think step by step" | Contraproducente (built in) | Útil cuando se necesita |
| Temperature | Ignorado por API | Completamente ajustable |
| JSON mode | Soportado | Excelente |
| Tool calls | Soportado (innovación V3.2) | Completamente soportado |
| Max output tokens | Default 32K, max 64K | Default 4K, max 8K |
| Mejor para | Razonamiento complejo, matemáticas, lógica | Todo lo demás |

### Decisión crítica: modo antes del prompt

Antes de escribir, elige modo:

| Modo | Cuándo usar | Cómo funciona |
|---|---|---|
| **Thinking (deepseek-reasoner)** | Matemáticas, debugging, estrategia, lógica compleja | Genera cadena de pensamiento interna. Más lento, más caro, más preciso en hard problems |
| **Chat (deepseek-chat)** | Todo lo demás, contenido, herramientas, propósito general | Rápido, versátil, optimizado para structured outputs |

⚠️ **Error común:** usar thinking mode por defecto pensando que es "más inteligente". Es más lento, más caro, y peor para structured outputs.

### 3 reglas fundamentales

1. **Match mode to task** — enruta al modo correcto según complejidad
2. **Name the artifact** — especifica EXACTAMENTE qué quieres: diff, JSON, tabla, resumen de 200 palabras, pytest file. No pidas "ayuda" o "explica"
3. **Less is more para thinking mode** — elimina system prompts elaborados, few-shot y CoT instructions. DeepSeek thinking mode maneja razonamiento internamente; agregar scaffolding externo causa interferencia

### Diferencia clave vs ChatGPT

- Los prompts de ChatGPT funcionan en DeepSeek pero producen resultados subóptimos
- Los system prompts detallados que mejoran ChatGPT EMPEORAN el thinking mode de DeepSeek
- Simplifica prompts para thinking mode
- Usa formatos estructurados para chat mode

### Para chat mode

- System prompts detallados y persona instructions sí funcionan
- Usa flags como "unsubstantiated claims" para aprovechar razonamiento analítico
- Estructura con secciones explícitas para forzar organización
- **XML-structured prompts** funcionan excepcionalmente bien

### Para thinking mode

- Prompt mínimo: solo el problema, sin instrucciones de razonamiento
- No agregues ejemplos de CoT — el modelo lo hace internamente
- Nombre el artefacto de salida claramente

### Context Caching (NUEVO)

DeepSeek automáticamente cachea prefixes repetidos a nivel de API:

- **Tokens cacheados:** $0.028 por millón
- **Tokens frescos:** $0.28 por millón
- **Descuento:** 90%
- **Diseña prompts** con system prompts y few-shot examples al inicio para maximizar cache hits

### Clarificación antes de grandes entregables

- Usa formato multiple-choice para prevenir suposiciones erradas
- Mantiene el intercambio ágil antes de escribir planes de negocio o specs técnicos

---

## Comparativa rápida

| Aspecto | Claude 4.6 | GPT-5 | Gemini 3 | Grok | LLaMA | DeepSeek V3.2 |
|---|---|---|---|---|---|---|
| Estructura preferida | XML tags | Markdown + XML | 4 elementos + XML | 4-part formula | Chat template | Minimal (think) / Estructurado (chat) |
| Ejemplos | 3-5, diversos | 3-5 | 1+ | Incluir explícitos | 1-3 (few-shot) | Evitar en think mode |
| Contexto largo | Docs al inicio, query al final | Cacheable al inicio | Instrucciones al final | Context upfront | Re-enunciar en turnos | Depende del modo |
| Rol/Persona | System prompt | developer role | Persona explícita | Parte de la formula | Header role | System prompt (solo chat) |
| Formato salida | Prosa o XML | Markdown + XML | Imperativo | Especificar explícitamente | Especificar explícitamente | Name the artifact |
| Thinking/Reasoning | Adaptive thinking con effort | Reasoning models (o-series) | Reasoning and Planning | Think Mode | Chain-of-thought | Thinking mode (V3.2) |
| Errores comunes | Ser vago | Saltar estructura | Falta de elementos | Ser vago | No usar template | Usar think mode innecesariamente |

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

1. **Prompt caching** — contenido repetido al inicio del prompt (90% descuento en DeepSeek)
2. **Modelos pequeños para tareas simples** — no uses opus/gpt-5 para resúmenes básicos
3. **Salidas concisas** — pide "responde en máximo X líneas"
4. **Evita system prompts gigantes** — solo incluye lo necesario

### Errores universales

1. ❌ Ser vago y esperar que el modelo adivine
2. ❌ No dar ejemplos cuando el output debe ser consistente
3. ❌ Mezclar instrucciones con datos en el mismo bloque
4. ❌ No especificar formato de salida
5. ❌ Usar el modelo más caro para todo
6. ❌ Usar thinking mode innecesariamente (DeepSeek)
7. ❌ No aprovechar prompt caching

---

*Compilado: 2026-03-28 | Fuentes: Anthropic, OpenAI, Google, xAI, Meta, DeepSeek*
