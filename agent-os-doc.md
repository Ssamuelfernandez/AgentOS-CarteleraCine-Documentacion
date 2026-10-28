# Agent OS: documentación explicada paso a paso

Spec-Driven Development (SDD) asistido por IA · Versión analizada: Agent OS 3.x (octubre de 2026\)

## Índice

1. ¿Qué es Agent OS?  
2. Contexto: el SDD y la evolución de Agent OS  
3. ¿Cómo se usa? Comandos en el chat y plan mode  
4. Conceptos clave: product, standards, specs, skills y perfiles  
5. Flujo de trabajo: cada comando, sus preguntas y sus archivos  
6. Instalación y estructura de carpetas  
7. Ejemplo completo de principio a fin  
8. ¿Se puede afinar más?  
9. Ventajas, inconvenientes y comparación con otras alternativas  
10. Bibliografía

---

## 1\. ¿Qué es Agent OS?

Agent OS es un **paquete de archivos de texto (markdown) y de comandos** que se instala dentro de tu proyecto para que tu asistente de programación con IA (Claude Code, Cursor, etc.) trabaje de forma más ordenada.

Lo creó **Brian Casel (Builder Methods)**. Es gratuito y de código abierto (licencia MIT). Repositorio: `github.com/buildermethods/agent-os`.

**Qué NO es:** no es un programa con ventanas, ni un modelo de IA, ni un agente que programa por sí mismo. Es un conjunto de "instrucciones guardadas" que le dices a tu agente de IA que ejecute.

**Qué hace, en dos ideas:**

1. Guarda en archivos las **reglas de cómo se programa en tu proyecto** (los *estándares*) y se las pasa a la IA cuando las necesita.  
2. Ayuda a **planificar bien cada funcionalidad antes de programarla**, y guarda ese plan en el repositorio (la *spec*).

**El problema que resuelve:** cada vez que le pides algo a una IA, tienes que volver a explicarle cómo quieres las cosas ("las respuestas de la API van así", "los nombres de las tablas se escriben asá", "este proyecto usa tal librería"). La IA lee tu código, pero no sabe *por qué* haces las cosas de cierta manera. Agent OS escribe todo eso una vez en archivos, y la IA los lee cuando hace falta.

---

## 2\. Contexto: el SDD y la evolución de Agent OS

### 2.1 Qué es el SDD (Spec-Driven Development)

El desarrollo guiado por especificaciones consiste en **escribir y revisar primero una especificación** (qué se va a construir y cómo) y **solo después implementarla**, en lugar de pedirle código a la IA directamente. Agent OS es una de las metodologías que proponen este enfoque.

Agent OS organiza el contexto que recibe la IA en **tres capas**:

| Capa | Responde a... | Dónde vive |
| :---- | :---- | :---- |
| **Standards** | *Cómo* se construye (convenciones de código) | `agent-os/standards/` |
| **Product** | *Qué* se construye y *por qué* (misión, hoja de ruta, stack) | `agent-os/product/` |
| **Specs** | *Qué* se construye ahora (una funcionalidad concreta) | `agent-os/specs/` |

### 2.2 Historia de versiones (por qué cambian los tutoriales)

- **v1/v2 (mediados de 2025):** Agent OS escribía la spec, creaba la lista de tareas, implementaba y orquestaba subagentes (con comandos como `/write-spec`, `/create-tasks`, `/implement-tasks`).  
- **v3 (enero de 2026):** se eliminó todo eso porque las herramientas de IA ya lo hacen bien (plan mode, listas de tareas automáticas, modelos que delegan solos). Se quedó solo con lo que aporta valor propio: **estándares \+ product \+ `/shape-spec`**.  
- **Qué pasó con las specs:** en la v2 la spec era un único archivo `spec.md`. En la v3 la estructura de carpetas se mantiene, pero cambia cómo se crea (con plan mode \+ `/shape-spec`).  
- Tus contenidos (estándares, specs, docs de producto) son compatibles entre versiones; los comandos cambian. **Muchos tutoriales y artículos describen v1/v2**, así que ojo al citarlos.

Esta documentación se basa en la **v3**.

---

## 3\. ¿Cómo se usa? Comandos en el chat y plan mode

### 3.1 ¿Línea de comandos o agente que pregunta?

**Es lo segundo: un asistente que te va haciendo preguntas, dentro del chat de tu herramienta de IA.**

- Los comandos que verás (`/discover-standards`, `/shape-spec`…) **no son comandos de terminal** (no se escriben en bash ni PowerShell). Son **comandos "slash"** que se escriben en el chat de Claude Code, igual que otros como `/help`.  
- Cada uno es en realidad un **archivo markdown con instrucciones** (en `.claude/commands/agent-os/`). Al escribir el comando, Claude lee esas instrucciones y las sigue: te hace preguntas, lee tu código y crea archivos.  
- Lo único que sí se ejecuta en terminal es la **instalación** (un script `project-install.sh`).  
- En herramientas que no son Claude Code (Cursor, etc.), no hay slash commands de Agent OS tal cual: se le dice al agente que lea el archivo del comando (por ejemplo, referenciándolo con `@`) y lo ejecute.

**Ejemplo de cómo se siente, escribes en el chat:**

```
/shape-spec
```

y el agente responde con algo como: *"¿Qué vamos a construir?"*, tú contestas, te pregunta si tienes mockups, y así sucesivamente.

### 3.2 ¿Qué es el "plan mode"?

El **plan mode** (modo plan) **no es de Agent OS: es una función de herramientas como Claude Code o Cursor.**

- En modo normal, el agente puede editar archivos y ejecutar cosas según le pides.  
- En **plan mode**, el agente **solo investiga y propone un plan**: lee el código, te hace preguntas y te presenta un plan paso a paso, **sin tocar nada** hasta que tú lo apruebes.  
- En Claude Code se activa, por ejemplo, con el comando `/plan`.

**Por qué importa:** es exactamente la idea del SDD. Primero se decide *qué* y *cómo* se va a hacer (la especificación) y se revisa; después se implementa. Evita que la IA se ponga a programar sin haber entendido bien lo que quieres.

**Relación con Agent OS (clave de la v3):**

- En las versiones antiguas (v1/v2), Agent OS tenía sus propios comandos para escribir la especificación.  
- Desde la **v3 (enero 2026\)** ya no: usa el plan mode de tu herramienta, porque hoy ya lo hace muy bien. Agent OS aporta `/shape-spec`, que **se ejecuta dentro del plan mode** y lo mejora: añade preguntas sobre tus estándares y tu producto, y **guarda el plan en archivos** del proyecto para que no se pierda al cerrar el chat.

---

## 4\. Conceptos clave: product, standards, specs, skills y perfiles

Aquí se definen los conceptos. Los comandos que los crean, con sus preguntas y archivos, se detallan en el apartado 5\.

### 4.1 Product (contexto del producto)

Además de estándares y specs, Agent OS guarda **el contexto del producto**: qué es la aplicación y hacia dónde va. Son tres archivos en `agent-os/product/`: `mission.md` (misión), `roadmap.md` (hoja de ruta) y `tech-stack.md` (tecnologías). Se crean con `/plan-product`.

Cuando ejecutas `/shape-spec`, la IA **lee estos archivos** para comprobar que la funcionalidad encaja con la misión y el roadmap y que las decisiones técnicas coinciden con el stack. Se hace una vez al comienzo y se puede editar a mano cuando cambie algo.

### 4.2 Standards (estándares)

Un **estándar** es un archivo markdown corto que explica **una convención de tu proyecto**: una regla que la IA debe seguir. Se guardan en `agent-os/standards/`, agrupados por carpetas según el tema.

**Ejemplo real de cómo sería un estándar** (`agent-os/standards/api/response-format.md`):

```markdown
# Formato de respuesta de la API

- Toda respuesta devuelve { "data": ..., "error": null } o { "data": null, "error": "mensaje" }.
- Códigos HTTP: 200 OK, 201 creado, 400 datos inválidos, 404 no encontrado.

Ejemplo:
  res.status(201).json({ data: newTask, error: null });
```

&nbsp;

Regla primero, ejemplo después, y breve (se mete en el contexto de la IA, así que cuanto más corto, mejor).

### 4.3 El índice de estándares: `index.yml` ("describir cada estándar")

Junto a los estándares hay un archivo llamado **`index.yml`**: un **índice** donde cada estándar tiene **una línea de descripción** de qué trata.

```yaml
api:
  response-format:
    description: Estructura de las respuestas de la API y códigos de estado
  error-handling:
    description: Convención de códigos de error y manejo de excepciones
database:
  migrations:
    description: Nombres de migraciones y convenciones de columnas
```

&nbsp;

**¿Para qué?** Si tuvieras 30 estándares y la IA tuviera que leerlos todos cada vez, **llenaría su memoria de trabajo** (el "contexto", que es limitado) con cosas irrelevantes. Con el índice, la IA lee solo esas descripciones (unas pocas líneas), decide cuáles encajan con la tarea y **carga únicamente esos**.

Es como el índice de un libro o un listado de carpetas con etiqueta: sirve para encontrar rápido lo que hace falta sin abrir todo. Si añades o borras estándares a mano, el comando `/index-standards` actualiza el índice.

### 4.4 Specs (especificaciones)

**Spec \= especificación de una funcionalidad.** Es el documento (en realidad, una carpeta con varios documentos) que describe **qué se va a construir, por qué y cómo**, antes de programarlo.

**Cada funcionalidad que planificas con `/shape-spec` genera una carpeta propia** dentro de `agent-os/specs/`, con la fecha y el nombre de la feature:

```
agent-os/specs/2026-01-15-1430-comentarios/
├── plan.md        → el plan completo, paso a paso
├── shape.md       → el alcance: qué entra, qué no, decisiones tomadas y contexto
├── references.md  → código existente del proyecto que sirve de modelo
├── standards.md   → copia del contenido completo de los estándares que aplican a esta feature
└── visuals/       → mockups o capturas (opcional)
```

**Qué contiene cada archivo, con un ejemplo (app de tareas, feature "etiquetas"):**

- `shape.md`: *"Se podrán crear etiquetas de color y asignarlas a tareas. Fuera de alcance: etiquetas compartidas entre usuarios."*  
- `references.md`: *"Seguir el patrón de `routes/tasks.js`."*  
- `standards.md`: el texto de las reglas que aplican (por ejemplo, el formato de respuesta de la API).  
- `plan.md`: la lista de pasos a implementar. **La Tarea 1 es siempre "guardar la documentación de la spec"**, y luego vienen las tareas de implementación.  
- `visuals/`: mockups, capturas o ejemplos, si los hay.

**Utilidad:** meses después (o si cambian los requisitos) puedes abrir esa carpeta y ver qué se hizo, por qué, y con qué reglas. Es el "historial de decisiones" del proyecto.

### 4.5 Standards vs. Skills

Esta comparación aparece porque en Claude Code existen las **skills** (una función propia de Claude Code, no de Agent OS), y hay que saber diferenciarlas de los estándares.

|  | Standard (estándar) | Skill |
| :---- | :---- | :---- |
| Qué es | Una **regla o convención** ("cómo debe ser") | Un **procedimiento** ("cómo se hace una tarea") |
| Ejemplo | *"Las respuestas de la API tienen este formato"* | *"Para crear un endpoint nuevo: 1\) crea la ruta, 2\) valida los datos, 3\) escribe el test, 4\) documenta"* |
| Cuándo se usa | **Tú lo pides** (`/inject-standards`) | **Claude lo detecta solo** cuando la tarea encaja |
| Dónde vive | `agent-os/standards/` | `.claude/skills/` |

**Ejemplo para entenderlo:** el estándar es el **reglamento** ("los informes llevan esta cabecera"); la skill es la **receta** ("para hacer un informe, sigue estos pasos").

**Lo recomendado:** combinarlos. Una skill puede decir *"antes de implementar, lee `agent-os/standards/api/response-format.md`"*. Así tienes automatización (la skill) con reglas consistentes (los estándares). Y si ejecutas `/inject-standards`, además de los estándares te avisa de qué skills existen que podrían servir.

> Para tu práctica **no necesitas usar skills**; es solo para entender por qué Agent OS lo menciona.

### 4.6 Perfiles (profiles)

Un **perfil** es una **colección de estándares guardada fuera de tus proyectos** (en `~/agent-os/profiles/`), para **reutilizarla en varios proyectos del mismo tipo**.

**Ejemplo:** has creado muchos estándares para proyectos de Rails (`rails`) y otros para proyectos web genéricos (`default`). Cuando instalas Agent OS en un proyecto nuevo, eliges el perfil y **recibe esos estándares ya hechos**, sin tener que descubrirlos de nuevo.

- **Herencia:** un perfil puede basarse en otro (por ejemplo, `rails` hereda de `default`). Se configura en `~/agent-os/config.yml`.  
- **Sincronización:** hay un script (`sync-to-profile.sh`) para llevar los estándares nuevos de un proyecto de vuelta al perfil.

**¿Lo necesitas en la práctica de clase?** Probablemente no: con el perfil `default` y creando tus propios estándares en el proyecto es suficiente. Puedes decir en la exposición que existe para reutilizar estándares entre proyectos.

---

## 5\. Flujo de trabajo: cada comando, sus preguntas y sus archivos

Cada comando funciona como una **entrevista**: te hace unas preguntas, tú contestas en el chat y con tus respuestas rellena los archivos. Las preguntas están sacadas de la documentación oficial; la redacción exacta puede variar de una ejecución a otra. Los ejemplos de contenido son ilustrativos (la documentación describe qué recoge cada archivo, pero no publica su plantilla exacta).

**Bucle central:** descubrir → inyectar → construir → refinar.

| Comando | Función | Resultado |
| :---- | :---- | :---- |
| `/plan-product` | Define misión, roadmap y stack | `agent-os/product/*.md` |
| `/discover-standards` | Extrae el conocimiento tribal del código | Estándares en `agent-os/standards/` \+ `index.yml` |
| `/index-standards` | Reconstruye el índice de estándares | `index.yml` actualizado |
| `/inject-standards` | Carga los estándares relevantes en el contexto actual | Estándares en conversación, plan o skill |
| `/shape-spec` | Da forma a una feature en plan mode y la guarda | `agent-os/specs/<fecha>-<feature>/` |

### 5.1 `/plan-product` → carpeta `agent-os/product/`

**Antes de empezar:** si ya existen los archivos de producto, te pregunta si quieres *reemplazarlos todos*, *actualizar solo algunos* o *cancelar*.

| Archivo | Preguntas que te hace | Qué contiene | Ejemplo (app de gastos) |
| :---- | :---- | :---- | :---- |
| `mission.md` | ¿Qué problema resuelve el producto? ¿Para quién es? ¿Qué hace única tu solución? | Visión del producto, usuarios objetivo y problemas principales que resuelve | *"Ayudar a estudiantes a controlar sus gastos mensuales de forma simple."* |
| `roadmap.md` | ¿Qué funcionalidades son imprescindibles para el lanzamiento (MVP)? ¿Cuáles se planean para después? | Plan de desarrollo por fases, con las funcionalidades priorizadas | *MVP: registrar gastos, categorías, resumen mensual. Después: exportar CSV.* |
| `tech-stack.md` | Si ya existe un estándar de stack: ¿este proyecto usa el mismo? Si no: describe frontend, backend, base de datos y otras tecnologías | Las tecnologías elegidas para el proyecto | *Frontend React, backend Node, base de datos SQLite.* |

**Ejemplo ilustrativo** de `mission.md`:

```markdown
# Misión
Ayudar a estudiantes a controlar sus gastos mensuales de forma sencilla.

## Usuarios
Estudiantes universitarios con presupuesto ajustado.

## Qué la diferencia
Registrar un gasto en menos de 10 segundos, sin cuentas bancarias.
```

&nbsp;

**Consejos de la documentación:** las respuestas breves valen (se pueden ampliar después); si omites una sección, se crea un marcador de posición. Se puede actualizar más tarde editando los archivos a mano o ejecutando de nuevo `/plan-product` y eligiendo "actualizar archivos concretos".

### 5.2 `/discover-standards` → carpeta `agent-os/standards/`

No hace falta escribir los estándares a mano (aunque se puede): este comando **los extrae de tu propio código**. Es especialmente útil cuando ya tienes un proyecto con código escrito.

**Pasos y preguntas:**

1. **¿Qué área analizamos?** Si no la indicas, el agente analiza el proyecto y te propone entre 3 y 5 áreas (API, base de datos, componentes, autenticación…). También puedes indicarla de antemano: `/discover-standards api`.  
2. **Análisis:** lee unos 5-10 archivos representativos de esa área y busca patrones inusuales, opinados (elecciones que podrían haber sido otras), "tribales" (que un recién llegado no sabría) y consistentes (repetidos en varios archivos).  
3. **Una o dos preguntas por cada patrón detectado**, para entender el porqué. Ejemplo: *"Todas tus rutas devuelven `{data, error}`: ¿por qué esa estructura?"*  
4. **Confirmar, editar u omitir** cada estándar que te redacta.  
5. **Actualiza el índice** (ejecuta `/index-standards` automáticamente al final).  
6. **¿Seguimos con otra área?** Puedes continuar o parar.

**Qué genera:**

- Un archivo `.md` por estándar, dentro de una carpeta por tema (`api/response-format.md`).  
- El archivo `index.yml`, con una línea de descripción por estándar (ver apartado 4.3).

### 5.3 `/index-standards` → actualiza `index.yml`

Se usa cuando has creado o borrado estándares a mano, o si las sugerencias de `/inject-standards` parecen desfasadas. El agente:

- detecta los archivos nuevos y **te pregunta qué descripción ponerles**,  
- elimina las entradas de archivos borrados,  
- ordena el índice alfabéticamente.

### 5.4 `/inject-standards` → no crea archivos nuevos

"**Inyectar**" significa **meter los estándares en la conversación** con la IA para que los tenga presentes. Se usa **antes de implementar**: *"inyecto las reglas de API y ahora te pido que programes el endpoint de etiquetas"*.

**Modos de uso:**

- `/inject-standards` (sin nada más): la IA mira de qué estás hablando, consulta el índice y te propone 2-5 estándares relevantes para que **los confirmes**.  
- `/inject-standards api/response-format`: cargas uno concreto.  
- `/inject-standards api`: cargas todos los de la carpeta `api`.

**Lo que te pregunta** (según el contexto):

- En una conversación normal: carga directamente el contenido completo de los estándares.  
- Si estás creando un plan o una skill: **¿quieres referencias a los archivos o copiar el contenido?** Las referencias se mantienen actualizadas si el estándar cambia; la copia es una instantánea autocontenida.

### 5.5 `/shape-spec` → carpeta `agent-os/specs/<fecha>-<feature>/`

Se ejecuta en plan mode. Úsalo para features significativas (si quieres que las decisiones persistan, tienes visuales o referencias, o necesitas alinearte con el producto); para tareas rápidas basta el plan mode directamente.

**Las preguntas y dónde acaba cada respuesta** (la correspondencia es orientativa):

| Pregunta del agente | Respuesta de ejemplo | Dónde queda |
| :---- | :---- | :---- |
| **¿Qué vamos a construir?** (con preguntas de seguimiento para acotar el alcance) | "Etiquetas de color para las tareas" | `shape.md` y `plan.md` |
| **¿Tienes visuales?** (mockups, wireframes, capturas) | Una captura de la pantalla actual | `visuals/` |
| **¿Hay código parecido que sirva de referencia?** | "Sigue el patrón de `routes/tasks.js`" | `references.md` |
| **Alineación con el producto** (solo si existen los docs de `product/`) | Confirmar que encaja con la misión y el roadmap | Se refleja en el plan |
| **¿Qué estándares aplican?** (te propone algunos del índice) | "Formato de respuesta de la API" | `standards.md` |

**Después de las preguntas:**

1. El agente construye un plan con **"Guardar la documentación de la spec" como Tarea 1** y te lo presenta completo.  
2. Tú lo revisas y lo **apruebas**.  
3. Se ejecuta la Tarea 1 (se guardan los archivos descritos en el apartado 4.4) y se sigue con la implementación.

**Tips de la documentación:** mantén el shaping rápido (captura lo justo para empezar y refina mientras construyes), los visuales son opcionales y los estándares guían, no mandan siempre.

### 5.6 Resumen: qué comando uso según lo que quiero

| Quiero... | Hago... |
| :---- | :---- |
| Definir qué es mi app | `/plan-product` |
| Que la IA conozca las reglas de mi código | `/discover-standards` |
| Que la IA use esas reglas ahora | `/inject-standards` |
| Planificar una funcionalidad y guardar el plan | `/plan` y luego `/shape-spec` |
| Actualizar el índice tras tocar estándares a mano | `/index-standards` |

**Según la situación:**

- **Proyecto nuevo:** `/plan-product` y luego `/discover-standards` según vayan apareciendo patrones.  
- **Proyecto existente:** `/discover-standards` y después `/inject-standards` antes de implementar.  
- **Feature nueva:** plan mode → `/shape-spec` → aprobar y ejecutar.  
- **Implementación rápida:** `/inject-standards` y empezar.

---

## 6\. Instalación y estructura de carpetas

La documentación oficial muestra las instrucciones de instalación tras un registro gratuito con email. Lo que se puede verificar en fuentes públicas es el esquema siguiente. Hay dos sitios:

### 6.1 Instalación base (una sola vez, en tu usuario): `~/agent-os/`

Es el "almacén" general. Se instala clonando el repositorio oficial:

```
git clone https://github.com/buildermethods/agent-os.git ~/agent-os && rm -rf ~/agent-os/.git
```

Contiene los comandos originales, los perfiles de estándares, los scripts de instalación y `config.yml` (perfil por defecto y herencia):

```
~/agent-os/
├── commands/agent-os/       # comandos que se copian a los proyectos
├── profiles/
│   ├── default/standards/
│   └── rails/standards/
├── scripts/
│   ├── project-install.sh
│   ├── sync-to-profile.sh
│   └── common-functions.sh
└── config.yml
```

### 6.2 Instalación en tu proyecto (por cada proyecto)

Se ejecuta en la terminal desde la raíz del proyecto:

```
~/agent-os/scripts/project-install.sh
```

Esto prepara en tu proyecto la carpeta **`agent-os/`** y copia los comandos a **`.claude/commands/agent-os/`**. Después, **cada comando va rellenando las carpetas**:

| Carpeta / archivo | Qué guarda | ¿Quién lo rellena? |
| :---- | :---- | :---- |
| `.claude/commands/agent-os/` | Los comandos slash (`discover-standards.md`, `inject-standards.md`, `plan-product.md`, `shape-spec.md`, `index-standards.md`) | El script de instalación |
| `agent-os/product/` | `mission.md`, `roadmap.md`, `tech-stack.md` | `/plan-product` |
| `agent-os/standards/` | Estándares por carpetas (`api/`, `database/`…) \+ `index.yml` | `/discover-standards` (o tú a mano) \+ `/index-standards` |
| `agent-os/specs/` | Una carpeta por feature, con fecha y nombre | `/shape-spec` |

**Árbol completo cuando ya se ha usado todo:**

```
tu-proyecto/
├── agent-os/
│   ├── product/
│   │   ├── mission.md
│   │   ├── roadmap.md
│   │   └── tech-stack.md
│   ├── standards/
│   │   ├── api/
│   │   │   ├── response-format.md
│   │   │   └── error-handling.md
│   │   ├── database/
│   │   │   └── migrations.md
│   │   └── index.yml
│   └── specs/
│       └── 2026-01-15-1430-comentarios/
│           ├── plan.md
│           ├── shape.md
│           ├── references.md
│           ├── standards.md
│           └── visuals/
└── .claude/
    └── commands/agent-os/
        ├── discover-standards.md
        ├── inject-standards.md
        ├── index-standards.md
        ├── plan-product.md
        └── shape-spec.md
```

**Qué subir a Git:** `standards/`, `product/` y `specs/` (así queda el historial del proyecto). `.claude/commands/` se puede subir o regenerar.

> Nota: no he podido comprobar en fuentes públicas qué carpetas vacías deja exactamente el script nada más instalar. Lo seguro es que `agent-os/` y los comandos se crean al instalar, y que `product/`, `standards/` y `specs/` se llenan al ejecutar cada comando.

---

## 7\. Ejemplo completo de principio a fin (app de tareas)

Ejemplo ilustrativo (los diálogos son una aproximación; las preguntas reales las define cada comando).

**Paso 0, instalación (terminal):**

```
~/agent-os/scripts/project-install.sh
```

**Paso 1, definir el producto (chat de Claude Code):**

```
/plan-product
```

La IA pregunta: *¿qué problema resuelve?, ¿para quién?, ¿qué entra en el MVP?, ¿qué tecnologías?* → crea `mission.md`, `roadmap.md` y `tech-stack.md`.

**Paso 2, capturar reglas (si ya hay código):**

```
/discover-standards api
```

La IA lee tus rutas, detecta que todas devuelven `{data, error}`, te pregunta por qué y escribe `api/response-format.md` \+ actualiza `index.yml`.

**Paso 3, planificar una funcionalidad ("etiquetas para tareas"):**

```
/plan
/shape-spec
```

La IA pregunta: *¿qué vamos a construir?, ¿tienes mockups?, ¿hay código parecido que seguir?, ¿encaja con la misión?, ¿qué estándares aplican?* → monta el plan. Tú lo revisas y lo **apruebas**.

**Paso 4, implementar:** al aprobar, la Tarea 1 guarda `agent-os/specs/2026-..-etiquetas/` con `plan.md`, `shape.md`, etc., y el agente programa aplicando los estándares.

**Paso 5, cambio de requisitos (lo que pide la práctica):** *"Ahora las etiquetas se podrán compartir entre usuarios."* → editas `roadmap.md` si procede y ejecutas otra vez `/plan` \+ `/shape-spec` para esa modificación. Queda una **nueva carpeta de spec** con la decisión documentada, y el historial de cambios se ve en Git.

---

## 8\. ¿Se puede afinar más? Sí, de tres formas

Nada de lo que genera Agent OS es definitivo ni está "cerrado": son archivos de texto normales.

**1\. Dar más indicaciones al agente (en el chat)**

- Contestar las preguntas con más detalle (cuanto más concreta la respuesta, más preciso el archivo).  
- Pedir cambios directamente: *"añade al estándar de la API que los errores incluyan un código numérico"*.  
- Volver a ejecutar el comando:  
  - `/plan-product` permite **actualizar solo algunos archivos** (por ejemplo, solo el `roadmap.md`).  
  - `/discover-standards` se puede repetir sobre otra área o más adelante, cuando hayan aparecido nuevos patrones.  
  - `/shape-spec` se ejecuta de nuevo para cada funcionalidad o cambio.

**2\. Editar los archivos a mano** Son markdown y YAML, así que se abren en cualquier editor. Los documentos de producto "son solo markdown" y se pueden editar directamente. Si tocas los estándares (creas, borras o renombras archivos), actualiza el índice con `/index-standards` o edita `index.yml` a mano.

**3\. Combinar ambas** Lo habitual es dejar que el agente genere un primer borrador y retocarlo tú después. Para estándares, la recomendación es que sean **cortos**: regla primero, porqué después, ejemplo de código y viñetas.

**Un detalle importante con las specs (valoración propia):** el archivo `standards.md` de cada spec guarda el contenido de los estándares *en el momento de crearla*. Si luego cambias un estándar, las specs antiguas no se actualizan, y tiene sentido que sea así, porque son el historial de lo que se decidió entonces. Para un cambio de requisitos, lo coherente es crear una **spec nueva** en lugar de reescribir la anterior.

---

## 9\. Ventajas, inconvenientes y comparación con otras alternativas

### 9.1 Ventajas

- Ligero: solo archivos markdown; sirve para cualquier lenguaje y framework.  
- Gratuito y de código abierto.  
- Las reglas del proyecto quedan escritas una vez y se reutilizan.  
- Útil en proyectos con código ya escrito (extrae el conocimiento implícito).  
- Las specs quedan guardadas en el repositorio como historial de decisiones.  
- No sustituye a tu herramienta de IA: se apoya en su plan mode.

### 9.2 Inconvenientes (parte es valoración propia)

- Los comandos están pensados sobre todo para Claude Code; en otras herramientas hay que referenciar archivos a mano.  
- Desde la v3 depende de lo bien que funcione el plan mode de tu herramienta.  
- Es menos estricto que otros enfoques SDD: no obliga a pasar por fases con aprobación formal.  
- Los estándares hay que mantenerlos; si quedan viejos, la IA se desalinea.  
- Mucha documentación en internet es de versiones antiguas.  
- La página oficial de instalación exige un registro con email para ver las instrucciones (el repositorio sí es público).

### 9.3 Comparación breve con otras alternativas SDD

Agent OS se centra en **estándares de código y calidad de la spec** y se integra con la herramienta que ya usas. Otras opciones del enunciado (GitHub Spec Kit, OpenSpec, BMad Method, Kiro) proponen flujos más completos o estructurados, o un entorno propio (Kiro es un IDE basado en Code OSS con el flujo SDD integrado). Para una comparación rigurosa conviene contrastarlo con la documentación de cada una.

---

## 10\. Bibliografía

- Documentación oficial de Agent OS: [https\://buildermethods.com/agent-os](https://buildermethods.com/agent-os)  
- Flujo de trabajo: [https\://buildermethods.com/agent-os/workflow](https://buildermethods.com/agent-os/workflow)  
- Discover standards: [https\://buildermethods.com/agent-os/discover-standards](https://buildermethods.com/agent-os/discover-standards)  
- Inject standards: [https\://buildermethods.com/agent-os/inject-standards](https://buildermethods.com/agent-os/inject-standards)  
- Product planning: [https\://buildermethods.com/agent-os/product-planning](https://buildermethods.com/agent-os/product-planning)  
- Shape spec: [https\://buildermethods.com/agent-os/shape-spec](https://buildermethods.com/agent-os/shape-spec)  
- Standards vs Skills: [https\://buildermethods.com/agent-os/standards-vs-skills](https://buildermethods.com/agent-os/standards-vs-skills)  
- Estructura de archivos: [https\://buildermethods.com/agent-os/file-structure](https://buildermethods.com/agent-os/file-structure)  
- Novedades y migración a v3: [https\://buildermethods.com/agent-os/migration](https://buildermethods.com/agent-os/migration)  
- Documentación v2 (histórica): [https\://buildermethods.com/agent-os/v2](https://buildermethods.com/agent-os/v2)  
- Repositorio: [https\://github.com/buildermethods/agent-os](https://github.com/buildermethods/agent-os)  
- Release 3.0: [https\://github.com/buildermethods/agent-os/releases](https://github.com/buildermethods/agent-os/releases)  
- Anuncio de v3: [https\://github.com/buildermethods/agent-os/discussions/310](https://github.com/buildermethods/agent-os/discussions/310)  
- Plan mode de Claude Code: [https\://code.claude.com/docs/en/common-workflows\#use-plan-mode-for-safe-code-analysis](https://code.claude.com/docs/en/common-workflows#use-plan-mode-for-safe-code-analysis)