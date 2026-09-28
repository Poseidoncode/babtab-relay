# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![バージョン](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![ライセンス](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

Babtab Chrome 拡張機能のための**ローカル中継サーバー**です。あなたの AI エージェント（Cursor / Pi / Claude Code / …）を、実際の Chrome につなぎます。

MV3 の拡張機能は自分でポートを listen できないため、この小さなプログラムが橋渡しをします。`localhost` 上で動作するので、通信が PC の外に出ることはありません。Relay は転送だけを行い、**ページの内容を見ることはできません**。

## 接続方法（3つの役割）

```text
AI エージェント（Cursor / Pi …） ←→ Relay（ローカル :3000） ←→ Chrome 拡張機能（Side Panel 付き）
     MCP 設定で接続                    転送とペアリングのみ              タブ上で実際に操作する
```

## クイックスタート（clone も install も不要）

### ステップ 1：Chrome 拡張機能をインストール

`chrome://extensions` → **デベロッパーモード**を有効化 → **パッケージ化されていない拡張機能を読み込む** → `dist` フォルダを選択。

> Chrome ウェブストア公開後は、この手順は「ストアからインストール」に変わります。

### ステップ 2：Relay を起動する（どちらか一方、結果は同じ）

```bash
# A. Node 20+ がある場合：インストール不要、そのまま実行
npx @babtab/relay

# B. Node を入れたくない場合：GitHub Releases から OS に対応する実行ファイルをダウンロード
./babtab-relay-darwin-arm64   # 例：macOS Apple Silicon
```

次のメッセージが表示されたら起動成功です（デフォルトポート `3000`）：

```text
[babtab-relay] listening on http://127.0.0.1:3000
```

応用（ポート変更・トークン保存先の指定）：

```bash
PORT=3001 npx @babtab/relay
BABTAB_TOKEN_FILE=~/.babtab/relay-tokens.json npx @babtab/relay
```

> `npx` はインストールしません。「ダウンロードして一度だけ実行」するもので、ターミナルを閉じれば Relay は停止します。

### ステップ 3：拡張機能を Relay に接続する

1. Chrome で任意のサイトを開き、ツールバーの Babtab アイコンをクリックして **Side Panel** を開く
2. Relay URL にはデフォルトで `ws://127.0.0.1:3000` が入力済みなので、そのまま
3. **"Save & Connect"（保存して接続）**をクリック

この手順で、あなたの Chrome が Relay 上のデバイスとして登録されます。

### ステップ 4：AI エージェントを Relay に接続する（Cursor の例）

1. Side Panel の同じページでエージェントを選択 → **"Copy config"（設定をコピー）**をクリック
2. `~/.cursor/mcp.json` の `mcpServers` に貼り付け（単一プロジェクト専用にする場合は当該プロジェクトの `.cursor/mcp.json`）：

```json
{
  "mcpServers": {
    "babtab": {
      "url": "http://127.0.0.1:3000/mcp",
      "headers": { "Authorization": "Bearer 取得したトークン" }
    }
  }
}
```

3. パネルに `Controlled by: cursor` と表示されたら接続完了です。

動作確認の一言（エージェントに依頼）：

> browser_observe で今 Chrome で開いているタブを確認して、タイトルと URL を教えて

## よくある質問

- **"Save & Connect"を押しても反応がない？** まず Relay のターミナルに `listening` と表示されているか確認し、URL が `ws://127.0.0.1:3000` でポートが一致しているか確認してください。
- **ペアリングコードの有効期限が切れた？** コードの有効期間は短いので、"Copy config"をもう一度押してください。
- **トークンを変えたい？** ステップ 4 を再実行すると新しいトークンが発行されます。`mcp.json` の更新をお忘れなく。
- **エージェントと Chrome が別マシン？**（応用）Relay を VPS 上に置き、Side Panel の Relay URL を `wss://…` に変更するだけです。手順は同じです。

## プライバシー

- `localhost` 上では完全なローカル接続であり、パケットが PC の外に出ることはありません。
- Relay はコマンドと結果を転送するだけで、ページ内容の解析・保存は一切行いません。
- Side Panel からいつでも **Pause（一時停止）/ Take Over（操作の引き継ぎ）/ 断線**できます。最終的な制御権は常に人間にあります。

## 開発者向け

このリポジトリにはリリース成果物（難読化済みの単一バンドル＋実行ファイル）のみが含まれており、開発ソースは含まれません。Issue やディスカッションはこのリポジトリに直接お願いします。
