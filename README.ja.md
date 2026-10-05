# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![バージョン](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![ライセンス](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

> **Babtab 拡張機能が必要です：**[Chrome ウェブストアから Babtab をインストール](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb)

[Babtab Chrome 拡張機能](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb)のための**ローカル中継サーバー**です。あなたの AI エージェント（Cursor / Pi / Claude Code / …）を、実際の Chrome につなぎます。

MV3 の拡張機能は自分でポートを listen できないため、この小さなプログラムが橋渡しをします。`localhost` 上で動作するので、通信が PC の外に出ることはありません。Relay はリクエストと結果をメモリ上でのみ転送します（ページ観測やスクリーンショットを含みます）。ページ内容を保存することはありません。

## 推奨セットアップ（0.3.1）

Node.js 20+ が必要です。[Chrome ウェブストアから拡張機能をインストール](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb)し、Babtab の **Add to Cursor**／**Add to VS Code**／**Add to Hermes** などのワンクリックボタンを押します（**Add to Codex**／**Add to Antigravity** はインストール用スニペットのコピーです）。Cursor で追加・有効化し、Chrome に戻って **Approve** を押してください。Relay は AI ツールが自動起動するため、ターミナルを開いたままにする必要はありません。ほかの AI ツールは **Other AI tools / install with a command** から設定できます。

旧版の手動 Relay を終了してから切り替えてください。詳細は [最新ガイド](README.md) を参照してください。以下は手動 HTTP 接続用の手順です。

## デバイス認証へのアップグレード

Relay と拡張機能を一緒に更新し、拡張機能を再読み込みして Side Panel で再接続したうえで、各エージェントについてステップ 2 を再実行してください。拡張機能は新しい `dev_v2_` デバイス ID を生成します。古い ID に紐づくトークンでは新しい接続を操作できません。Relay は明示的に `127.0.0.1` にバインドするようになりました。リモート配置では `HOST` を指定し、TLS（`wss://`／`https://`）で転送中の認証情報を保護してください。

## 接続方法（3つの役割）

```text
AI エージェント（Cursor / Pi …） ←→ Relay（ローカル :3000） ←→ Chrome 拡張機能（Side Panel 付き）
     MCP 設定で接続                    転送とペアリングのみ              タブ上で実際に操作する
```

## クイックスタート（clone も install も不要）

### ステップ 1：Chrome 拡張機能をインストール

[Chrome ウェブストアから Babtab をインストール](https://chromewebstore.google.com/detail/babtab/pnapkckkiphkbciihimdkhofdabnmjhb)してください。

> デベロッパーモード / パッケージ化されていない拡張機能の読み込みはローカル開発用です。

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
2. ステップ 1 でポートを確認するだけです（デフォルト `3000`、Relay と一致させること）。URL の入力は不要です
3. **"Save & Connect"（保存して接続）**をクリック

この手順で、あなたの Chrome が Relay 上のデバイスとして登録されます。
（VPS 上のリモート Relay を使う場合も、同じステップ内の *Advanced: custom Relay URL* で対応できます。）

### ステップ 4：AI エージェントを Relay に接続する（Cursor の例）

1. Side Panel の同じページでステップ 2 → **Cursor** をクリック → ワンライナーのコマンドをコピーします：

```bash
npx -y @babtab/relay setup --target cursor --relay-url http://127.0.0.1:3000 --device <あなたのデバイスID>
```

2. 任意のターミナルに貼り付けて実行します。6 桁のペアリングコードが表示されるので、Side Panel 上部のバナーで **Approve（承認）**します。
3. 完了です。コマンドが `~/.cursor/mcp.json` に `babtab` を自動で書き込みます
   （既存のエントリは保持され、変更前のファイルは `.bak` として残ります）。Cursor で MCP をリロード
   （Settings → MCP）すると、パネルに `Controlled by: cursor` と表示されます。

サポートされている `--target`：`cursor`、`claude-code`、`windsurf`、`copilot`、
`copilot-insiders`、`codex`、`pi`、`claude-desktop`、`antigravity`、`devin`、
`kimi`、`hermes`、`manual`。`npx -y @babtab/relay setup --help` ですべての
コマンドを確認できます（`--list-targets` で各ハーネス用の完成済みコマンドを一覧表示）。
ターミナルが使えない環境では、ステップ 2 の *Pair & copy JSON manually* で手動貼り付けに戻れます。

動作確認の一言（エージェントに依頼）：

> browser_observe で今 Chrome で開いているタブを確認して、タイトルと URL を教えて

## よくある質問

- **"Save & Connect"を押しても反応がない？** まず Relay のターミナルに `listening` と表示されているか確認し、ステップ 1 のポートが Relay のポートと一致しているか確認してください。
- **ペアリングコードの有効期限が切れた？** コードの有効期間は短いので、setup コマンドを再実行してください。
- **トークンを変えたい？** setup コマンドを再実行すると新しいトークンが発行され、設定も自動で書き換えられます（旧ファイルは `.bak` として残ります）。
- **エージェントと Chrome が別マシン？**（応用）Relay を VPS 上に置き、Side Panel ステップ 1 の *Advanced: custom Relay URL* に `wss://…` を設定し、setup コマンドに `--relay-url https://…` を付けるだけです。手順は同じです。

## プライバシー

- `localhost` 上では完全なローカル接続であり、パケットが PC の外に出ることはありません。
- Relay はコマンドと結果を転送するだけで、ページ内容の解析・保存は一切行いません。
- Side Panel からいつでも **Pause（一時停止）/ Take Over（操作の引き継ぎ）/ 断線**できます。最終的な制御権は常に人間にあります。

## 開発者向け

このリポジトリにはリリース成果物（難読化済みの単一バンドル＋実行ファイル）のみが含まれており、開発ソースは含まれません。Issue やディスカッションはこのリポジトリに直接お願いします。
