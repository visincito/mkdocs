---
title: "opencode: el agente de IA en tu terminal que te escribe el código (casi) solo"
description: "opencode es un agente de IA de código abierto que lee tu proyecto, edita archivos y ejecuta comandos en tu terminal. Guía de instalación y primeros pasos."
date: 2026-08-18
author: visin
categories:
  - Desarrollo de software
  - Inteligencia Artificial
tags:
  - opencode
  - ia
  - agente de código
  - terminal
  - desarrollo
  - llm
  - cli
  - agente
  - open source
draft: false
---

# opencode: el agente de IA en tu terminal que te escribe el código (casi) solo

Imagina esto: estás en tu terminal, en medio de un refactor de esos que dan miedo a las tres de la tarde, y en lugar de ir copiando y pegando fragmentos de un chat, el agente lee tu proyecto, entiende el contexto, edita los archivos él solo y hasta ejecuta los comandos para comprobar que no has roto nada. Tú solo le dices qué quieres, y él se encarga del resto.

Eso es opencode. Y no, no es otro chatbot que te escupe código para que luego lo pegues tú a mano: es un agente de IA de código abierto que trabaja directamente en tu repo, con acceso al sistema de archivos y con la capacidad de ejecutar comandos. Vamos a ver qué es exactamente, cómo lo instalas en cinco minutos y cómo sacarle partido desde el primer día sin que te destroce nada por el camino.

Y ojo, que "agente" aquí no es postureo de marketing. Significa que la herramienta tiene acceso real a tu máquina: lee tu código, lo modifica y ejecuta comandos en tu terminal. Eso es una pasada y a la vez un peligro, así que también vamos a hablar de permisos y de cómo no dejar que un modelo de lenguaje te haga un `rm -rf` a las dos de la mañana.

¿Para quién es esto? Para cualquiera que programe en una terminal y quiera delegar el trabajo sucio: refactors mecánicos, tests repetitivos, búsquedas de bugs, tareas de mantenimiento. Y también para el que quiera probar qué se siente al tener un par de juniors virtuales currando en paralelo. Todo sin salir de la terminal.

## ¿Qué es opencode?

La definición oficial, sin adornos: *"OpenCode is an open source AI coding agent. It's available as a terminal-based interface, desktop app, or IDE extension."*

O sea: un agente de IA para programar, de código abierto, que se maneja principalmente desde la terminal (aunque también tienes app de escritorio y extensión para el IDE). Pero la definición se queda corta si no te cuento lo que hace por debajo.

### Un agente, no un chat

Aquí está la diferencia clave con las herramientas que ya conoces. Un chat de IA te da respuestas; un agente te hace el trabajo. opencode lee tu proyecto, entiende qué lenguaje usas, qué dependencias tienes y cómo está montado todo, y a partir de ahí edita archivos, ejecuta comandos y verifica resultados. Es como tener a un junior muy aplicado (y muy rápido) al lado, con acceso de escritura a tu repo.

Y como es un agente, la conversación importa menos que la acción: tú le das una tarea, él la ejecuta y te cuenta qué ha hecho. Si algo falla, lo corrige. Si necesita permiso para algo delicado, te lo pide. Tú supervisas, él curra.

La comparación con un junior no es casual. Igual que un junior, necesita contexto para no meter la pata, y igual que un junior, a veces hace cosas que no le has pedido. La diferencia es que el junior tarda una tarde en equivocarse; el agente tarda tres segundos. Por eso la supervisión no es opcional: es parte del trabajo.

### Lo que trae de serie

- **LSP enabled**: carga automáticamente los Language Server Protocol correctos para cada lenguaje, de modo que el modelo tenga el contexto que necesita. No tienes que configurar nada: él detecta el proyecto y tira. Y esto importa más de lo que parece: un modelo con el contexto de tu lenguaje y tus dependencias responde mucho mejor que uno que va a ciegas.
- **Multi-session**: puedes lanzar varios agentes en paralelo, cada uno en su sesión, trabajando en cosas distintas a la vez. Como tener un equipo sin pagar nóminas. Eso sí, cada agente que corre es un agente que gasta tokens, así que no te pases con el número de sesiones abiertas.
- **Share links**: puedes compartir una sesión con quien quieras, para enseñar un problema o pedir ayuda sin andar pegando pantallazos.
- **Login con GitHub Copilot y ChatGPT Plus/Pro**: si ya pagas alguna de estas, puedes usarla como proveedor. Vaya, que no estás obligado a sacar una API key nueva si no quieres.
- **75+ proveedores LLM**: Anthropic, OpenAI, Azure, Bedrock, Groq, DeepSeek, Ollama, LM Studio, OpenRouter, xAI... la lista es larga. Si tu proveedor favorito existe, casi seguro que está en la lista.
- **Licencia MIT**: libre, sin letra pequeña. Y con repo público, lo que significa que puedes mirar el código, reportar bugs y hasta colaborar si te pica el gusanillo.

### De dónde sale y quién lo mantiene

opencode lo mantiene el equipo de **SST**, que en julio de 2026 pasó a llamarse **Anomaly**. Ojo, que el rebranding solo afecta al nombre del equipo: opencode conserva su nombre, su repo y sus estrellas. El repo vive ahora en `github.com/anomalyco/opencode` (la URL antigua, `github.com/sst/opencode`, redirige).

Que un equipo se renombre no es un capricho: suele significar que han ampliado el campo de tiro más allá del proyecto original. Pero para lo que nos importa aquí, lo único que cambia de cara al usuario es la URL del repo y el nombre del tap de Homebrew. El resto sigue igual.

Y no es un proyecto de tres amigos: hablamos de **~198.7k estrellas y 25.6k forks**, con la última versión estable en **v1.18.18** (agosto de 2026). El paquete npm, `opencode-ai`, ronda las **2.3 millones de descargas semanales**. Vamos, que no es un experimento de fin de semana: es una de las herramientas de IA para programar más usadas del momento.

## Cómo se instala

Aquí no hay drama: tienes muchas vías y todas son rápidas. Elige la que más te cuadre y a otra cosa.

El orden de abajo no es un ranking: es simplemente el que más sentido tiene según tu sistema. El resultado final es el mismo, así que no te comas la cabeza con cuál es "el mejor".

### El método universal (Linux y macOS)

Un script y a correr:

```bash
curl -fsSL https://opencode.ai/install | bash
```

El script decide dónde instalar el binario por este orden de prioridad: `$OPENCODE_INSTALL_DIR`, `$XDG_BIN_DIR`, `$HOME/bin` y, como último recurso, `$HOME/.opencode/bin`. Si no tienes `$HOME/bin` en el PATH, ya sabes lo que toca.

Si después de instalar el comando `opencode` no te responde, lo primero es mirar en cuál de esos cuatro sitios ha caído el binario y si ese directorio está en tu PATH. Nueve de cada diez veces, el problema es ese y no otro.

### Con gestores de paquetes

Si prefieres las cosas con gestor, también hay:

```bash
npm install -g opencode-ai
```

Y lo mismo con los otros: `bun install -g opencode-ai`, `pnpm install -g opencode-ai` o `yarn global add opencode-ai`.

**Ojo, que aquí hay una trampa para novatos**: el paquete se llama `opencode-ai`, no `opencode`. Si haces `npm install -g opencode`, te vas a llevar un paquete que no es este y te vas a quedar tan pancho. Que no te pase.

En macOS con Homebrew tienes dos opciones:

```bash
brew install anomalyco/tap/opencode   # el tap oficial, siempre al día
brew install opencode                 # la fórmula oficial, se actualiza menos
```

En Arch, lo mismo pero con la eterna disyuntiva:

```bash
sudo pacman -S opencode    # la estable, desde los repos oficiales
paru -S opencode-bin       # la última, desde el AUR
```

¿Y Windows? La recomendación oficial es usar **WSL** y listo. Si no, también tienes `choco install opencode` y `scoop install opencode`, pero vamos, que si ya estás en Windows, la vía WSL es la que mejor te va a funcionar con una herramienta hecha para terminales de verdad.

### ¿Y cuál elijo?

Si estás en Linux o macOS y quieres la vía rápida, el script. Si ya vives dentro de un gestor de paquetes, el gestor. Si estás en Arch, `opencode-bin` del AUR te da la última versión sin esperar a que llegue a los repos oficiales. Y si eres de los que odian reinstalar cosas, el tap de Homebrew se mantiene siempre al día, que es lo que toca con una herramienta de estas.

### Requisitos previos (pocos, pero importantes)

Antes de lanzarte, dos cosas:

1. **Un emulador de terminal moderno**. Esto no corre en la terminal fea de serie del sistema. Hablamos de WezTerm, Alacritty, Ghostty (en Linux/macOS) o Kitty. Si tu terminal no pinta bien, la TUI de opencode no va a estar fina.
2. **API keys de los proveedores LLM**. Sin un proveedor conectado, el agente no tiene cerebro. Esto lo vemos en un momento.

Y un aviso del propio README, de esos que conviene leer: **"Remove versions older than 0.1.x before installing."** Si tienes una versión antigua (anterior a 0.1.x), desinstálala antes de instalar la nueva. No es un consejo, es una advertencia.

Y una cosa más antes de seguir: esto no es un plugin que se instala en tu editor y listo. opencode es una herramienta de terminal, con su TUI, sus atajos y su forma de hacer las cosas. Si no estás cómodo en una terminal, el primer día va a ser un poco raro. Pero se te pasa rápido, y el salto merece la pena.

Cuando lo tengas instalado, la prueba de fuego es sencilla: `opencode` a secas en un directorio cualquiera. Si se abre la TUI, ya tienes la herramienta. Si te escupe un error, revisa el PATH y los requisitos de arriba. Y recuerda lo del README: fuera versiones viejas antes de instalar.

## Primeros pasos

Vale, ya está instalado. Ahora la parte divertida.

### Arranca la TUI

Nada de magia:

```bash
cd /ruta/a/tu/proyecto
opencode
```

Con eso se lanza la interfaz de terminal (TUI). Ya estás dentro. A partir de aquí, todo se maneja con comandos de barra, estilo slash commands, que verás según escribes `/`.

Y sí, el detalle del directorio importa: opencode se apoya en el proyecto en el que estás para dar contexto al agente (y para los snapshots de `/undo`, que requieren git). Así que no lo lances desde cualquier sitio; entra en el proyecto y desde ahí.

### La TUI por dentro

La interfaz es de esas que se aprenden solas: abajo escribes, arriba se desarrolla la conversación con el agente, y los comandos de barra (`/`) te dan acceso a todo. ¿Dudas de qué puedes hacer? Escribe `/` y mira la lista; todo está a un slash de distancia. `/models` para el modelo, `/init` para el contexto del proyecto, `/undo` y `/redo` para deshacer, `/share` para compartir... No hace falta memorizar nada, la lista está siempre ahí.

### `/init`: que el agente conozca tu proyecto

Lo primero que deberías hacer en un proyecto es lanzar:

```
/init
```

opencode analiza el proyecto y te genera un archivo `AGENTS.md` en la raíz, con las instrucciones y el contexto que el agente va a usar para trabajar en ese repo. Es su "manual de instrucciones" particular: cómo está estructurado el proyecto, qué convenciones sigues, qué comandos se usan para buildear y testear...

Y aquí va el tip oficial del equipo: **haz commit del `AGENTS.md`**. Es un archivo de tu repo, no un artefacto temporal. Versionarlo significa que cualquier agente (y cualquier compañero) que abra el proyecto tiene el mismo contexto. El usuario ni se entera, pero el siguiente agente te lo agradece.

¿Y qué pinta tiene ese archivo? Pues depende del proyecto, pero la idea es que sea legible tanto para humanos como para agentes: estructura, comandos habituales, convenciones, cosas que no se deben tocar. Es el mismo concepto que un README, pero orientado a quien va a trabajar con el código, sea una persona o un modelo.

### Conecta tus proveedores

El agente necesita un cerebro. Para conectar tus proveedores:

```bash
opencode auth login
```

Las claves se guardan en `~/.local/share/opencode/auth.json`, y también puedes pasarle las keys por variables de entorno o por un `.env`. Tú eliges cómo.

Si no tienes API keys de los grandes o no te apetece liarte, el equipo recomienda **OpenCode Zen**, su propio proveedor con modelos curados y verificados. Es opcional, ojo, pero es la vía más cómoda para empezar: en la TUI, `/connect` → eliges "opencode" → te manda a `https://opencode.ai/auth` → copias la API key → y con `/models` eliges el modelo con el que quieres trabajar. Cinco minutos y listo.

Un apunte sobre las claves: van a `auth.json` en tu home, no al repo. Y si prefieres no guardar nada en disco, las variables de entorno son tu amiga. Lo que no hagas es pegar una API key en un archivo del proyecto y hacer commit de ella. Eso no se hace ni en broma.

Y no tienes por qué quedarte con un solo proveedor: puedes tener varios conectados y cambiar entre ellos según el modelo que necesites. Para tareas gordas, un modelo grande; para cosas triviales, uno pequeño y barato. Ahí entra en juego la clave `small_model` de la config, que veremos más adelante.

### Tu primera sesión

Para probar sin abrir siquiera la TUI, tienes la ejecución programática:

```bash
opencode run "Explain how closures work in JavaScript"
```

Eso lanza una sesión, ejecuta la petición y te devuelve el resultado por stdout. Perfecto para scripts, para pruebas y para hacerte una idea de cómo responde sin comprometerte con la interfaz.

Y como `opencode run` es programático, se lleva bien con el resto de flags: continúa la última sesión con `-c`, elige modelo con `-m` y hasta aprueba permisos automáticamente con `--auto`. Para automatizar tareas repetitivas, es oro puro.

Dentro de la TUI, los modelos se eligen con `/models`, y el formato es `provider/model`. Por ejemplo:

```
anthropic/claude-sonnet-4-20250514
```

Fíjate en el formato: `provider/model`. Primero el proveedor, luego el modelo. Si algún día no sabes qué tienes disponible, `/models` te lista los modelos de tu proveedor y a partir de ahí es elegir y a currar.

Y si ya sabes con qué modelo quieres trabajar desde el principio, el flag `-m`:

```bash
opencode -m anthropic/claude-sonnet-4-20250514
```

### Flags que te van a salvar el pellejo

La CLI tiene unos cuantos flags que merecen la pena desde el día uno:

- `-c` / `--continue`: continúa la última sesión. El pan de cada día.
- `-s` / `--session`: abre una sesión concreta.
- `--fork`: bifurca una sesión, útil para probar un enfoque alternativo sin tocar la original.
- `--prompt`: pasa el prompt directamente.
- `-m` / `--model`: elige modelo, como hemos visto.
- `--agent`: elige el agente con el que trabajar.
- `--auto`: auto-aprueba los permisos. Hablamos de esto más abajo, que tiene miga.

Con `-c` y `-m` ya cubres el 80% del día a día; el resto lo vas descubriendo según lo necesitas.

### Deshacer, rehacer y compartir

Dentro de la TUI, `/undo` y `/redo` te permiten deshacer y rehacer los cambios del agente, gracias a snapshots. Ojo a un detalle: **requieren que estés en un repo git**. Con git detrás, tienes red de seguridad; sin git, a vivir peligrosamente.

Y cuando quieras enseñarle algo a alguien, `/share` te genera un enlace para compartir la sesión. Ideal para pedir ayuda sin tener que explicar el problema dos veces.

Y un detalle que se agradece: como los snapshots dependen de git, si trabajas en un repo versionado, el `/undo` te devuelve a un estado anterior sin dramas. Es casi como si el agente no hubiera tocado nada.

Para terminar con lo básico: usa **`@`** para hacer búsqueda fuzzy de archivos y referenciarlos en el prompt, y si necesitas enseñarle una imagen (un mockup, una captura de un error), **arrástrala al terminal** y se añade al prompt. Detalles tontos que marcan la diferencia.

### El flujo de trabajo diario

Resumiendo, una sesión típica con opencode tiene esta pinta: entras en el proyecto con `opencode`, lanzas `/init` si es la primera vez, eliges modelo con `/models`, le explicas la tarea con contexto (y con `@` si hace falta referenciar archivos), trabajas en modo `plan` hasta que el enfoque esté claro, pulsas **Tab** para pasar a `build`, supervisas lo que hace, y si algo se tuerce, `/undo` y a repetir. Cuando termines, `/share` si quieres enseñarlo, y `opencode stats` para ver cuánto ha costado.

Todo eso sin salir de la terminal. Vaya tela.

## Recomendaciones para empezar con buen pie

Vale, ya sabes lo básico. Ahora, lo que yo haría el primer día (y lo que evitaría).

### Plan vs Build: primero piensa, luego ejecuta

opencode tiene dos agentes primarios:

- **build**: el de por defecto, con acceso completo. Lee, escribe, ejecuta.
- **plan**: solo lectura. Analiza, propone y, antes de ejecutar nada con bash, pide permiso.

El flujo sano es: trabajas en **plan**, iteráis sobre el enfoque, y cuando el plan esté claro, pulsas **Tab** para cambiar a **build** y que ejecute. Es la diferencia entre "déjame pensar antes de disparar" y "dispara y luego vemos". Con agentes que ejecutan comandos en tu máquina, pensar antes de disparar no es un capricho.

Además tienes subagentes: `general` para tareas generales, `explore` (solo lectura) para explorar el código y `scout` (también solo lectura) especializado en documentación y dependencias.

El modo `plan` es tu mejor amigo el primer día: te deja ver cómo piensa el agente antes de darle poder de escritura. Úsalo sin miedo, que no rompe nada.

### Háblale como a un junior

Esto es lo que más cuesta al principio. No le sueltes "arregla el bug" y esperes magia. Dale contexto, como se lo darías a un junior que acaba de aterrizar en el proyecto: qué archivo es, qué comportamiento esperas, qué has probado ya, qué no debe tocar. Cuanto mejor le expliques el problema, mejor te lo resuelve. Y usa `@` para referenciar archivos concretos en lugar de hacerle adivinar cuál es.

Y no tengas miedo de iterar: la primera respuesta casi nunca es la definitiva. Pídele cambios, pregúntale por qué ha hecho algo, que te explique su razonamiento. Como con un junior, la conversación es parte del trabajo.

### Permisos: qué le dejas hacer (y qué no)

Aquí va lo serio. Por defecto, opencode tiene **casi todo en "allow"**: puede leer y escribir archivos, ejecutar comandos... Pero hay excepciones importantes: `external_directory` y `doom_loop` piden confirmación, y los archivos `.env` están **denegados por defecto**. Los secretos no se tocan, y eso está muy bien.

Los valores de permisos son `allow`, `ask` o `deny`, y la configuración es granular, con wildcards. Un ejemplo de la jugada:

```json
{
  "permission": {
    "bash": {
      "*": "ask",
      "git *": "allow",
      "rm *": "deny"
    }
  }
}
```

Traducción: los comandos bash piden permiso por defecto, los `git` pasan sin preguntar y los `rm` están prohibidos en seco. Vas a dormir mejor así.

La idea es que configures la política por comando, no un todo o nada. ¿Quieres que pueda hacer `git` a sus anchas pero que te pregunte antes de tocar el sistema? Lo escribes y ya está. ¿No quieres que borre nada bajo ningún concepto? `deny` y a otra cosa.

Y ojo con `--auto`: aprueba automáticamente todo lo que no esté denegado. Es cómodo, pero recuerda que estás dándole las llaves del coche a un agente. Revisa bien qué permisos le dejas en "allow" antes de soltarle el `--auto`, porque la ejecución de comandos es real y puede ser peligrosa. No es paranoia: un `rm -rf` mal dirigido no perdona.

### Configuración: `opencode.json`

Toda la configuración vive en un archivo `opencode.json` (acepta JSON y JSONC), que empieza siempre con el schema para que tu editor te autocomplete:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "plan",
  "share": "auto"
}
```

Se busca en varias ubicaciones y **los archivos se fusionan**: `~/.config/opencode/opencode.json` para la config global, `opencode.json` en la raíz del proyecto y los directorios `.opencode/`. La global va para ti; la del proyecto va para el equipo.

Y como se fusionan, puedes tener una config de casa (modelos, proveedores, tu tono) y otra por proyecto (permisos, agentes, convenciones del repo) sin pisarte los pies.

Entre las claves que puedes tocar: `small_model` (un modelo pequeño para tareas triviales, que te ahorra pasta), `default_agent` (el agente por defecto), `share` (`manual`, `auto` o `disabled`), `autoupdate`, `lsp`, `mcp` y `plugin`. Hay más, pero con estas ya controlas el 90% del día a día.

### Controla el gasto (sí, esto cuesta dinero)

Los tokens no son gratis, y un agente que itera solo puede quemar pasta si no le pones límites. Dos comandos para dormir tranquilo:

```bash
opencode stats
opencode models --verbose
```

`opencode stats` te dice los tokens y el coste por sesión. `opencode models --verbose` te da el detalle de los modelos disponibles. Y en la configuración puedes limitar el número de **`steps`** por agente, que es la forma directa de acotar el gasto: menos pasos, menos tokens, menos sorpresas en la factura.

El límite de `steps` por agente es tu mejor amigo cuando dejas al agente trabajando solo: le pones un tope al número de pasos y acotas el gasto antes de que se desboque. Mejor eso que el susto con la factura a final de mes.

### El ecosistema

La TUI es la estrella, pero no es lo único: hay **app de escritorio** (en beta, así que no la uses para misión crítica todavía), **extensión de IDE** para VS Code y Zed, y comandos como `opencode web` y `opencode serve` para otras vías de uso. Si la terminal te pilla grande, siempre puedes empezar por la extensión del IDE y dar el salto cuando te sientas cómodo.

## Conclusión

opencode es de esas herramientas que cambian la forma de trabajar: un agente de IA de código abierto, con licencia MIT, que vive en tu terminal, lee tu proyecto, ejecuta comandos y te escribe el código. No es un chatbot que te da fragmentos para pegar; es un agente que hace el trabajo, y tú supervisas.

La receta para empezar con buen pie es corta: instálalo con el script o el gestor que prefieras, lanza `/init` para que conozca tu proyecto, conecta un proveedor (OpenCode Zen es la vía fácil), trabaja en modo **plan** antes de pasar a **build**, revísale los permisos y controla el gasto con `opencode stats`. Con eso, el agente curra y tú duermes.

Y si te queda la duda de si esto va a sustituir a los programadores: no. Sustituye a las tareas mecánicas, a los copy-paste de Stack Overflow y a las horas buscando un bug tonto. El criterio, la arquitectura y las decisiones importantes siguen siendo tuyas. El agente es la pala; tú sigues siendo el que decide dónde cavar.

Y recuerda: los modelos cambian, los proveedores cambian, pero el flujo bueno no cambia. Primero piensa, luego ejecuta, y revisa siempre qué le has dejado hacer. Con eso, opencode te va a salvar más de una tarde — y a deshoras, que es cuando más se agradece.