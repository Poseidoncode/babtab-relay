# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![版本](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![授權](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

Babtab Chrome 擴充功能的**本機中轉伺服器**：把你的 AI Agent（Cursor / Pi / Claude Code / …）接到你真正的 Chrome 上。

MV3 擴充功能自己不能 listen port，所以需要這支小程式當橋樑。跑在 `localhost`，流量不出你的電腦；Relay 只負責轉發，**看不到你的頁面內容**。

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
2. Relay URL 預設已填好 `ws://127.0.0.1:3000`，不用改
3. 按 **"Save & Connect"**

這一步是把你的 Chrome 註冊成 Relay 上的一台 device。

### 步驟 4：AI Agent 連上 Relay（以 Cursor 為例）

1. 在 Side Panel 同一頁選你的 Agent → 按 **"Copy config"**
2. 貼到 `~/.cursor/mcp.json` 的 `mcpServers`（只想給單一專案用，就放該專案的 `.cursor/mcp.json`）：

```json
{
  "mcpServers": {
    "babtab": {
      "url": "http://127.0.0.1:3000/mcp",
      "headers": { "Authorization": "Bearer 剛拿到的token" }
    }
  }
}
```

3. Panel 出現 `Controlled by: cursor` 就通了。

驗證一句話（在 Agent 裡下）：

> 用 browser_observe 看我現在 Chrome 開的分頁，然後告訴我標題和網址

## 常見問題

- **按 "Save & Connect"沒反應？** 先看 Relay 終端機有沒有 `listening`，再確認 URL 是 `ws://127.0.0.1:3000` 且 port 跟 Relay 一致。
- **配對碼過期？** 配對碼只有短暫有效，超時就重新按一次 "Copy config"。
- **想換 token？** 重跑一次步驟 4 會產生新 token，記得同步更新 `mcp.json`。
- **Agent 跟 Chrome 不在同一台電腦？**（進階）把 Relay 放到 VPS 上，Side Panel 的 Relay URL 改成你的 `wss://…` 即可，流程不變。

## 隱私說明

- 跑在 `localhost` 就是純本地連線，封包不出電腦。
- Relay 只轉發指令與結果，不解析、不儲存頁面內容。
- Side Panel 隨時可以 **Pause / Take Over / 斷線**，人類永遠有最終控制權。

## 開發者

本倉僅含發布產物（混淆後的單一 bundle + 執行檔），不含開發源碼。Issue 與討論請直接開在本倉。
