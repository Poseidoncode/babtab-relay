# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![Versión](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![Licencia](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

El **servidor relay local** para la extensión Babtab de Chrome: conecta tu agente de IA (Cursor / Pi / Claude Code / …) con tu Chrome real.

Las extensiones MV3 no pueden escuchar en un puerto, así que este pequeño programa actúa como puente. Se ejecuta en `localhost`, por lo que el tráfico nunca sale de tu máquina. El relay reenvía solicitudes y resultados solo en memoria (incluidas observaciones de página y capturas); no almacena el contenido de las páginas.

## Instalación recomendada (0.3.0)

Requiere Node.js 20+. Instala la extensión desde Chrome Web Store y pulsa **Add to Cursor** / **Add to VS Code** / **Add to Hermes** (botones de un clic) en Babtab; **Add to Codex** / **Add to Antigravity** copian un fragmento de instalación. Confirma y activa Babtab en Cursor, vuelve a Chrome y pulsa **Approve**. Tu herramienta de IA inicia el relay automáticamente; no necesitas mantener una terminal abierta. Para otras herramientas, usa **Other AI tools / install with a command**.

Detén el relay manual anterior antes de cambiar. Consulta la [guía actualizada](README.md). Las instrucciones siguientes corresponden a la conexión HTTP manual.

## Actualización a conexiones de dispositivo autenticadas

Actualiza el relay y la extensión juntos, recarga la extensión, reconecta en el Side Panel y repite el paso 2 para cada agente. La extensión genera un nuevo ID de dispositivo `dev_v2_`; los tokens vinculados a IDs anteriores ya no controlan la nueva conexión. El relay ahora se vincula explícitamente a `127.0.0.1`. En despliegues remotos debes configurar `HOST` y usar TLS (`wss://` / `https://`) para proteger las credenciales en tránsito.

## Cómo se conecta (3 roles)

```text
Agente de IA (Cursor / Pi …) ←→ Relay (local :3000) ←→ Extensión de Chrome (con Side Panel)
   se conecta vía config MCP      solo reenvío + emparejamiento   hace el trabajo real en tus pestañas
```

## Inicio rápido (sin clonar, sin instalar)

### Paso 1: Instala la extensión de Chrome

`chrome://extensions` → activa el **modo de desarrollador** → **Cargar descomprimida** → selecciona la carpeta `dist`.

> Cuando se publique en la Chrome Web Store, este paso será "instalar desde la tienda".

### Paso 2: Inicia el relay (elige una, mismo resultado)

```bash
# A. ¿Tienes Node 20+? Sin instalar nada, solo ejecuta:
npx @babtab/relay

# B. ¿Sin Node? Descarga el binario de tu SO desde GitHub Releases:
./babtab-relay-darwin-arm64   # p. ej. macOS Apple Silicon
```

Sabrás que funciona cuando veas (puerto por defecto `3000`):

```text
[babtab-relay] listening on http://127.0.0.1:3000
```

Avanzado (puerto personalizado / ubicación del archivo de tokens):

```bash
PORT=3001 npx @babtab/relay
BABTAB_TOKEN_FILE=~/.babtab/relay-tokens.json npx @babtab/relay
```

> `npx` no instala nada — descarga y ejecuta una sola vez. Cerrar la terminal detiene el relay.

### Paso 3: Conecta la extensión al relay

1. Abre cualquier sitio web en Chrome, haz clic en el icono de Babtab para abrir el **Side Panel**
2. Paso 1: confirma el puerto (por defecto `3000`, debe coincidir con el relay) — sin escribir URLs
3. Haz clic en **"Save & Connect" (Guardar y conectar)**

Esto registra tu Chrome como un dispositivo en el relay.
(¿Relay remoto en un VPS? Usa *Advanced: custom Relay URL* en el mismo paso.)

### Paso 4: Conecta tu agente de IA al relay (ejemplo con Cursor)

1. En la misma página del Side Panel, paso 2 → haz clic en **Cursor** → copia el comando de una línea:

```bash
npx -y @babtab/relay setup --target cursor --relay-url http://127.0.0.1:3000 --device <tu-device-id>
```

2. Pégalo en cualquier terminal y ejecútalo. Mostrará un código de emparejamiento
   de 6 dígitos — apruébalo en el banner superior del Side Panel.
3. Listo: el comando escribe `babtab` en `~/.cursor/mcp.json` por ti
   (conserva las entradas existentes y guarda el archivo anterior como `.bak`).
   Recarga MCP en Cursor (Settings → MCP) y el panel mostrará `Controlled by: cursor`.

Valores `--target` soportados: `cursor`, `claude-code`, `windsurf`, `copilot`,
`copilot-insiders`, `codex`, `pi`, `claude-desktop`, `antigravity`, `devin`,
`kimi`, `hermes`, `manual`. `npx -y @babtab/relay setup --help` muestra todos los
comandos (`--list-targets` imprime una línea lista por harness). ¿Sin terminal a mano?
El paso 2 también ofrece *Pair & copy JSON manually* como alternativa.

3. Cuando el panel muestre `Controlled by: cursor`, ya estás conectado.

Una frase para verificar (pídesela a tu agente):

> Usa browser_observe para ver las pestañas abiertas en mi Chrome y dime sus títulos y URLs.

## Preguntas frecuentes

- **¿"Guardar y conectar" no hace nada?** Revisa primero que la terminal del relay muestre `listening`, y confirma que el puerto del paso 1 coincida con el del relay.
- **¿Caducó el código de emparejamiento?** Los códigos duran poco — vuelve a ejecutar el comando setup.
- **¿Quieres un token nuevo?** Repetir el comando setup genera un token nuevo y reescribe la config (se conserva el `.bak` anterior).
- **¿Agente y Chrome en máquinas distintas?** (Avanzado) Pon el relay en un VPS, usa *Advanced: custom Relay URL* en el paso 1 del Side Panel con tu `wss://…` y pasa `--relay-url https://…` al comando setup. El flujo es el mismo.

## Privacidad

- En `localhost` es una conexión puramente local — los paquetes nunca salen de tu computadora.
- El relay solo reenvía comandos y resultados. Nunca analiza ni almacena el contenido de las páginas.
- El Side Panel puede **pausar / tomar el control / desconectar** en cualquier momento — el humano siempre tiene el control final.

## Desarrolladores

Este repo solo contiene artefactos de release (un bundle ofuscado + binarios), no el código fuente de desarrollo. Abre issues y discusiones aquí mismo.
