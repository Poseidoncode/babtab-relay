# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![Versión](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![Licencia](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

El **servidor relay local** para la extensión Babtab de Chrome: conecta tu agente de IA (Cursor / Pi / Claude Code / …) con tu Chrome real.

Las extensiones MV3 no pueden escuchar en un puerto, así que este pequeño programa actúa como puente. Se ejecuta en `localhost`, por lo que el tráfico nunca sale de tu máquina. El relay solo reenvía mensajes — **no puede ver el contenido de tus páginas**.

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
2. La Relay URL ya viene con `ws://127.0.0.1:3000` — déjala así
3. Haz clic en **"Save & Connect" (Guardar y conectar)**

Esto registra tu Chrome como un dispositivo en el relay.

### Paso 4: Conecta tu agente de IA al relay (ejemplo con Cursor)

1. En la misma página del Side Panel, elige tu agente → haz clic en **"Copy config" (Copiar configuración)**
2. Pégalo en `mcpServers` dentro de `~/.cursor/mcp.json` (o en el `.cursor/mcp.json` del proyecto para un solo proyecto):

```json
{
  "mcpServers": {
    "babtab": {
      "url": "http://127.0.0.1:3000/mcp",
      "headers": { "Authorization": "Bearer el-token-que-acabas-de-obtener" }
    }
  }
}
```

3. Cuando el panel muestre `Controlled by: cursor`, ya estás conectado.

Una frase para verificar (pídesela a tu agente):

> Usa browser_observe para ver las pestañas abiertas en mi Chrome y dime sus títulos y URLs.

## Preguntas frecuentes

- **¿"Guardar y conectar" no hace nada?** Revisa primero que la terminal del relay muestre `listening`, y confirma que la URL sea `ws://127.0.0.1:3000` con el puerto correcto.
- **¿Caducó el código de emparejamiento?** Los códigos duran poco — pulsa "Copiar configuración" de nuevo.
- **¿Quieres un token nuevo?** Repetir el paso 4 genera un token nuevo; recuerda actualizar `mcp.json`.
- **¿Agente y Chrome en máquinas distintas?** (Avanzado) Pon el relay en un VPS y cambia la Relay URL del Side Panel a tu `wss://…`. El flujo es el mismo.

## Privacidad

- En `localhost` es una conexión puramente local — los paquetes nunca salen de tu computadora.
- El relay solo reenvía comandos y resultados. Nunca analiza ni almacena el contenido de las páginas.
- El Side Panel puede **pausar / tomar el control / desconectar** en cualquier momento — el humano siempre tiene el control final.

## Desarrolladores

Este repo solo contiene artefactos de release (un bundle ofuscado + binarios), no el código fuente de desarrollo. Abre issues y discusiones aquí mismo.

## Licencia

Apache-2.0 — ver [LICENSE](LICENSE) y [NOTICE](NOTICE).

Copyright 2026 Poseidoncode.
