# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![版本](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![授權](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

Babtab Chrome 擴充功能的**本機中轉伺服器**：把你的 AI Agent（Cursor / Pi / Claude Code / …）接到你真正的 Chrome 上。

MV3 擴充功能自己不能 listen port，所以需要這支小程式當橋樑。跑在 `localhost`，流量不出你的電腦；Relay 會在記憶體中轉發請求與結果（包含頁面觀察及截圖），不會儲存頁面內容。

## 升級裝置驗證

請一起更新 Relay 與擴充功能，重新載入擴充功能、在 Side Panel 重新連線，
再為每個 Agent 執行一次步驟 2 配對。擴充功能會產生新的 `dev_v2_` 裝置 ID；
舊 ID 的 token 無法控制新連線。Relay 現在明確只監聽 `127.0.0.1`；
遠端部署需自行設定 `HOST`，並使用 TLS（`wss://`／`https://`）保護傳輸憑證。

## 連線方式（3 個角色）

```text
AI Agent（Cursor / Pi …） ←→ Relay（本機 :3000） ←→ Chrome 擴充功能（含 Side Panel）
     用 MCP 設定連過來              只做轉發與配對              真正在分頁上動手的人
```

## 使用者快速開始（不用 clone、不用 install）

### 步驟 1：安裝 Chrome 擴充功能

`chrome://extensions` → 開啟**開發者模式** → **載入未封裝項目** → 選 `dist` 資料夾。

> 上架 Chrome 商店後，這步會變成「去商店按安裝」。

### 步驟 2：啟動 Relay（二選一，效果一樣）

```bash
# A. 有 Node 20+ 的人：免安裝，直接跑
npx @babtab/relay

# B. 不想裝 Node 的人：去 GitHub Release 下載對應系統的執行檔
./babtab-relay-darwin-arm64   # macOS Apple Silicon 為例
```

看到以下訊息就是起來了（預設 port `3000`）：

```text
[babtab-relay] listening on http://127.0.0.1:3000
```

進階用法（換 port、指定 token 存檔位置）：

```bash
PORT=3001 npx @babtab/relay
BABTAB_TOKEN_FILE=~/.babtab/relay-tokens.json npx @babtab/relay
```

> `npx` 不是安裝，是「下載來跑一次」，關掉終端機就停了。

### 步驟 3：擴充功能連上 Relay

1. 在 Chrome 開任意網站，點工具列的 Babtab 圖示開啟 **Side Panel**
2. 步驟 1：確認 port（預設 `3000`，要跟 Relay 一致）即可，不用手打 URL
3. 按 **"Save & Connect"**

這一步是把你的 Chrome 註冊成 Relay 上的一台 device。
（Relay 在遠端 VPS？同一步有 *Advanced: custom Relay URL* 可填。）

### 步驟 4：AI Agent 連上 Relay（以 Cursor 為例）

1. 在 Side Panel 同一頁，步驟 2 → 按 **Cursor** → 複製那行指令：

```bash
npx -y @babtab/relay setup --target cursor --port 3000 --device <你的deviceId>
```

2. 貼到任意終端機執行，會顯示 6 位配對碼 → 回到 Side Panel 在頂部橫條按 **Approve**。
3. 完成：指令會自動把 `babtab` 寫進 `~/.cursor/mcp.json`
  （原有內容保留，舊檔留一份 `.bak`）。去 Cursor 重載 MCP（Settings → MCP），
   Panel 出現 `Controlled by: cursor` 就通了。

驗證一句話（在 Agent 裡下）：

> 用 browser_observe 看我現在 Chrome 開的分頁，然後告訴我標題和網址

支援的 `--target`：`cursor`、`claude-code`、`windsurf`、`copilot`、
`copilot-insiders`、`codex`、`pi`、`claude-desktop`、`antigravity`、`devin`、
`kimi`、`hermes`、`manual`。`npx -y @babtab/relay setup --help` 看完整說明
（`--list-targets` 一次印出每個 harness 的現成指令）。沒終端機的環境，
步驟 2 也有 *Pair & copy JSON manually* 可退回手動貼上。

## 常見問題

- **按 "Save & Connect"沒反應？** 先看 Relay 終端機有沒有 `listening`，再確認步驟 1 的 port 跟 Relay 一致。
- **配對碼過期？** 配對碼只有短暫有效，超時就重跑一次 setup 指令。
- **想換 token？** 重跑一次 setup 指令會產生新 token 並重寫設定（舊檔留 `.bak`）。
- **Agent 跟 Chrome 不在同一台電腦？**（進階）把 Relay 放到 VPS 上，Side Panel 步驟 1 用 *Advanced: custom Relay URL* 填 `wss://…`，setup 指令加 `--relay-url https://…` 即可，流程不變。

## 隱私說明

- 跑在 `localhost` 就是純本地連線，封包不出電腦。
- Relay 只轉發指令與結果，不解析、不儲存頁面內容。
- Side Panel 隨時可以 **Pause / Take Over / 斷線**，人類永遠有最終控制權。

## 開發者

本倉僅含發布產物（混淆後的單一 bundle + 執行檔），不含開發源碼。Issue 與討論請直接開在本倉。
