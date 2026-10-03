# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![Versão](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![Licença](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

O **servidor relay local** para a extensão Babtab do Chrome: conecta seu agente de IA (Cursor / Pi / Claude Code / …) ao seu Chrome de verdade.

Extensões MV3 não conseguem escutar em uma porta, então este programinha age como ponte. Roda em `localhost`, por isso o tráfego nunca sai da sua máquina. O relay encaminha solicitações e resultados apenas em memória (incluindo observações de página e capturas); não armazena o conteúdo das páginas.

## Instalação recomendada (0.3.0)

Requer Node.js 20+. Instale a extensão pela Chrome Web Store e clique em **Add to Cursor** / **Add to VS Code** / **Add to Hermes** (botões de um clique) no Babtab; **Add to Codex** / **Add to Antigravity** copiam um trecho de instalação. Confirme e ative no Cursor, volte ao Chrome e clique em **Approve**. A ferramenta de IA inicia o relay automaticamente; não é preciso manter um terminal aberto. Para outras ferramentas, use **Other AI tools / install with a command**.

Encerre o relay manual anterior antes de mudar. Consulte o [guia atualizado](README.md). As instruções abaixo são para conexão HTTP manual.

## Atualização para conexões de dispositivo autenticadas

Atualize o relay e a extensão juntos, recarregue a extensão, reconecte no Side Panel e repita o passo 2 para cada agente. A extensão gera um novo ID de dispositivo `dev_v2_`; tokens vinculados a IDs anteriores não controlam mais a nova conexão. O relay agora se vincula explicitamente a `127.0.0.1`. Em implantações remotas, configure `HOST` e use TLS (`wss://` / `https://`) para proteger as credenciais em trânsito.

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
2. Passo 1: confirme a porta (padrão `3000`, precisa ser igual à do relay) — sem digitar URLs
3. Clique em **"Save & Connect" (Salvar e conectar)**

Isso registra seu Chrome como um dispositivo no relay.
(Relay remoto num VPS? Use *Advanced: custom Relay URL* no mesmo passo.)

### Passo 4: Conecte seu agente de IA ao relay (exemplo com Cursor)

1. Na mesma página do Side Panel, passo 2 → clique em **Cursor** → copie o comando de uma linha:

```bash
npx -y @babtab/relay setup --target cursor --relay-url http://127.0.0.1:3000 --device <seu-device-id>
```

2. Cole em qualquer terminal e execute. Ele mostra um código de pareamento de
   6 dígitos — aprove no banner no topo do Side Panel.
3. Pronto: o comando escreve `babtab` em `~/.cursor/mcp.json` para você
   (preserva as entradas existentes e guarda o arquivo anterior como `.bak`).
   Recarregue o MCP no Cursor (Settings → MCP) e o painel mostra `Controlled by: cursor`.

Valores `--target` suportados: `cursor`, `claude-code`, `windsurf`, `copilot`,
`copilot-insiders`, `codex`, `pi`, `claude-desktop`, `antigravity`, `devin`,
`kimi`, `hermes`, `manual`. `npx -y @babtab/relay setup --help` mostra todos os
comandos (`--list-targets` imprime uma linha pronta por harness). Sem terminal à mão?
O passo 2 também oferece *Pair & copy JSON manually* como alternativa.

3. Quando o painel mostrar `Controlled by: cursor`, está conectado.

Uma frase para verificar (peça ao seu agente):

> Use o browser_observe para ver as abas abertas no meu Chrome e me diga os títulos e URLs delas.

## Perguntas frequentes

- **"Salvar e conectar" não faz nada?** Confira primeiro se o terminal do relay mostra `listening`, depois confirme que a porta do passo 1 é igual à do relay.
- **Código de pareamento expirou?** Os códigos duram pouco — rode o comando setup de novo.
- **Quer um token novo?** Repetir o comando setup gera um token novo e reescreve a config (mantém o `.bak` anterior).
- **Agente e Chrome em máquinas diferentes?** (Avançado) Coloque o relay num VPS, use *Advanced: custom Relay URL* no passo 1 do Side Panel com o seu `wss://…` e passe `--relay-url https://…` ao comando setup. O fluxo é o mesmo.

## Privacidade

- Em `localhost` é uma conexão puramente local — os pacotes nunca saem do seu computador.
- O relay só encaminha comandos e resultados. Nunca analisa nem armazena o conteúdo das páginas.
- O Side Panel pode **pausar / assumir o controle / desconectar** a qualquer momento — o humano sempre tem o controle final.

## Desenvolvedores

Este repo contém apenas artefatos de release (um bundle ofuscado + binários), não o código-fonte de desenvolvimento. Abra issues e discussões aqui mesmo.
