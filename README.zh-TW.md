# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![版本](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![授權](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

> **需要先安裝 Babtab 外掛：**[從 Chrome 線上應用程式商店安裝 Babtab](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb)

[Babtab Chrome 擴充功能](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb)的**本機中轉伺服器**：把你的 AI Agent（Cursor / Pi / Claude Code / …）接到你真正的 Chrome 上。

MV3 擴充功能自己不能 listen port，所以需要這支小程式當橋樑。跑在 `localhost`，流量不出你的電腦；Relay 會在記憶體中轉發請求與結果（包含頁面觀察及截圖），不會儲存頁面內容。

## 使用者快速開始（0.3.1）

需要 Node.js 20+ 與對應新版外掛。一般使用者從 **[Chrome Web Store 安裝 Babtab](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb)**；開發者模式只供本機測試。

1. 開啟 Babtab Side Panel，按 **Add to Cursor**（或 **Add to VS Code**／**VS Code Insiders**／**Add to Hermes**，都是一鍵安裝）。
2. 在 AI 工具確認加入／啟用 Babtab；它會自動啟動本機橋接程式。**Add to Codex**／**Add to Antigravity** 則是一鍵複製安裝片段（Codex：貼上終端機執行，桌機版共用設定；Antigravity：貼進 mcp_config.json 後重啟 IDE）。
3. 回到外掛，確認工具名稱後按 **Approve**。
4. 在一般網站分頁，對 AI 說 **Use babtab to observe the current tab**；需要網站授權時在外掛批准。

不用先啟動 Relay、不用設定 port、不用保持終端機開啟。使用時保留 Chrome。
首次 npm 下載需要網路；憑證固定保存在 `~/.babtab/`，更換工作目錄不影響連線。

### 其他 AI 工具

開啟 **Other AI tools / install with a command**，選工具並複製專屬指令：

```bash
npx -y @babtab/relay@0.3.1 setup --target windsurf --port 3000 --device <外掛提供的deviceId>
```

只需執行一次。指令保留其他 server、備份舊設定，寫入由 AI 工具自動啟動的 MCP 設定。
接著重載／啟用 MCP，再回 Chrome 批准配對。
自動啟動支援 Cursor、Claude Code、Windsurf、Copilot、Codex、Claude Desktop、Antigravity、Devin、Kimi、Hermes；Pi／Manual 沿用進階 HTTP 流程。

### 自動啟動與重連

AI 工具啟動 `babtab-relay stdio --device … --client … --port 3000`，橋接程式啟動或重用本機 Relay。
未配對時可列出工具，實際操作仍須人工批准。多個 AI 工具共用 Relay；持有 Relay 的工具結束後，剩餘工具會接手。
中斷中的動作會回報錯誤，不會自動重送。

自動模式的憑證保存在 `~/.babtab/`（可透過 `BABTAB_DATA_DIR` 覆寫）。
重新開啟 Chrome／AI 工具後會自動重連。明確按 **Disconnect** 後，自動重連會停止，需再按 Reconnect。

### 從 0.1.x 升級

先關掉舊的手動 Relay，更新外掛，再重新加入 AI 工具並批准一次。
自動模式使用新的固定憑證目錄。若 port 被其他服務或舊版 Relay 佔用，會顯示錯誤；
可在 **Advanced connection settings** 換 port，再重新加入 AI 工具。

### 進階：手動或遠端 Relay

仍可執行 `npx @babtab/relay` 啟動獨立 HTTP Relay。
外掛 **Advanced connection settings** 可設定 port／URL 並按 **Save & Connect**；也保留手動產生 HTTP 設定。

手動模式沿用 `BABTAB_TOKEN_FILE` 或啟動目錄的 `./data/relay-tokens.json`。
遠端部署須明確設定 `HOST` 並使用 TLS（`wss://`／`https://`）；先讓外掛連上 Relay，
再跑 `setup --target <tool> --relay-url https://… --device <id>`。

### 常見問題

- **Cursor 沒開啟？** 展開按鈕下方的 setup 指令，執行一次，再重載／啟用 MCP。
- **沒出現配對要求？** 確认 AI 工具的 PATH 找得到 Node 20+／`npx`，且 Babtab 已啟用；保持外掛面板開啟。MCP 日誌會顯示啟動錯誤。
- **配對碼過期？** 保持 AI 工具執行，橋接程式會自動取得新碼。
- **斷線？** 開啟 AI 工具和 Chrome；若先前主動 Disconnect，請在外掛按 Reconnect。
- **憑證遺失或撤銷？** 重新批准配對；橋接程式不會跳過人工批准。

## 隱私說明

- 跑在 `localhost` 就是純本地連線，封包不出電腦。
- Relay 只轉發指令與結果，不解析、不儲存頁面內容。
- Side Panel 隨時可以 **Pause / Take Over / 斷線**，人類永遠有最終控制權。

## 開發者

本倉僅含發布產物（混淆後的單一 bundle + 執行檔），不含開發源碼。Issue 與討論請直接開在本倉。
