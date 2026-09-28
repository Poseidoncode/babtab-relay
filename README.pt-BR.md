# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![Versão](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![Licença](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

O **servidor relay local** para a extensão Babtab do Chrome: conecta seu agente de IA (Cursor / Pi / Claude Code / …) ao seu Chrome de verdade.

Extensões MV3 não conseguem escutar em uma porta, então este programinha age como ponte. Roda em `localhost`, por isso o tráfego nunca sai da sua máquina. O relay só encaminha mensagens — **não consegue ver o conteúdo das suas páginas**.

## Como se conecta (3 papéis)

```text
Agente de IA (Cursor / Pi …) ←→ Relay (local :3000) ←→ Extensão do Chrome (com Side Panel)
   conecta via config MCP         só encaminhamento + pareamento   faz o trabalho real nas suas abas
```

## Início rápido (sem clonar, sem instalar)

### Passo 1: Instale a extensão do Chrome

`chrome://extensions` → ative o **modo do desenvolvedor** → **Carregar sem compactação** → selecione a pasta `dist`.

> Quando for publicada na Chrome Web Store, este passo vira "instalar pela loja".

### Passo 2: Inicie o relay (escolha um, mesmo resultado)

```bash
# A. Tem Node 20+? Sem instalar nada, só rode:
npx @babtab/relay

# B. Sem Node? Baixe o binário do seu SO em GitHub Releases:
./babtab-relay-darwin-arm64   # ex. macOS Apple Silicon
```

Está no ar quando você vir (porta padrão `3000`):

```text
[babtab-relay] listening on http://127.0.0.1:3000
```

Avançado (porta personalizada / local do arquivo de tokens):

```bash
PORT=3001 npx @babtab/relay
BABTAB_TOKEN_FILE=~/.babtab/relay-tokens.json npx @babtab/relay
```

> `npx` não instala nada — baixa e executa uma única vez. Fechar o terminal para o relay.

### Passo 3: Conecte a extensão ao relay

1. Abra qualquer site no Chrome, clique no ícone do Babtab para abrir o **Side Panel**
2. A Relay URL já vem preenchida com `ws://127.0.0.1:3000` — deixe assim
3. Clique em **"Save & Connect" (Salvar e conectar)**

Isso registra seu Chrome como um dispositivo no relay.

### Passo 4: Conecte seu agente de IA ao relay (exemplo com Cursor)

1. Na mesma página do Side Panel, escolha seu agente → clique em **"Copy config" (Copiar configuração)**
2. Cole em `mcpServers` dentro de `~/.cursor/mcp.json` (ou no `.cursor/mcp.json` do projeto para uso em um só projeto):

```json
{
  "mcpServers": {
    "babtab": {
      "url": "http://127.0.0.1:3000/mcp",
      "headers": { "Authorization": "Bearer o-token-que-voce-acabou-de-obter" }
    }
  }
}
```

3. Quando o painel mostrar `Controlled by: cursor`, está conectado.

Uma frase para verificar (peça ao seu agente):

> Use o browser_observe para ver as abas abertas no meu Chrome e me diga os títulos e URLs delas.

## Perguntas frequentes

- **"Salvar e conectar" não faz nada?** Confira primeiro se o terminal do relay mostra `listening`, depois confirme que a URL é `ws://127.0.0.1:3000` com a porta certa.
- **Código de pareamento expirou?** Os códigos duram pouco — clique em "Copiar configuração" de novo.
- **Quer um token novo?** Repetir o passo 4 gera um token novo; lembre-se de atualizar o `mcp.json`.
- **Agente e Chrome em máquinas diferentes?** (Avançado) Coloque o relay num VPS e troque a Relay URL do Side Panel para o seu `wss://…`. O fluxo é o mesmo.

## Privacidade

- Em `localhost` é uma conexão puramente local — os pacotes nunca saem do seu computador.
- O relay só encaminha comandos e resultados. Nunca analisa nem armazena o conteúdo das páginas.
- O Side Panel pode **pausar / assumir o controle / desconectar** a qualquer momento — o humano sempre tem o controle final.

## Desenvolvedores

Este repo contém apenas artefatos de release (um bundle ofuscado + binários), não o código-fonte de desenvolvimento. Abra issues e discussões aqui mesmo.
