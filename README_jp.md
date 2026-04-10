# AgenticService GitHub リリース

このディレクトリは、`AgenticService` の GitHub リリース資産と配布資料のルートです。

Language versions:

- [ENG](./README.md)
- [CHINESE](./README_cht.md)
- [JAPANESE](./README_jp.md)

`AgenticService` は、ローカル優先の Go ベース agentic orchestration service です。主な特徴は以下のとおりです。

- 構造化されたツール実行
- 複数ステップのタスクワークフロー
- ローカルの shell とファイル操作
- task-backed の長時間実行ジョブ
- フォルダ単位のローカル RAG 検索
- Skill によるワークフロー起動と制約
- セッション、プラン、デバッグ、Skills、RAG を扱う内蔵 Web コンソール

## リリース対象

このリリースは現時点で以下を対象としています。

- Linux x86_64

以下のプラットフォームは公開リリースノートの対象外です。

- `linux_arm32_sf`

## リリース資産

GitHub Release には以下を含めることを推奨します。

- `AgenticService_linux_x86_minimal.tar.gz`
- `deploy_files/`
- release notes

## アセット構成

### `deploy_files/`

これは、最小配布パッケージを生成するためのリリース用ディレクトリです。

典型的な内容は以下のとおりです。

```text
deploy_files/
├── AgenticService
├── agent.sample.properties
├── run.sh
├── run_bg.sh
├── stop.sh
├── history/
├── provider/
├── skill/
└── website/
```

### プラットフォーム別ビルド成果物

`dist/` には、以下のようなビルド成果物が含まれる場合があります。

- `linux_x86_64/`
- `linux_arm64/`
- `macos_arm64/`

ただし、GitHub へ公開する際のデプロイ入口は `deploy_files/` を使います。

## 主な機能

このリリースパッケージは、以下の機能を提供します。

- 対話ベースのタスク実行
- Plan と Tool Trace の可視化
- shell / file / search / task ツール
- Skill ベースの実行制約とワークフロー注入
- スコープ付きファイル検索によるローカル RAG 検索
- 内蔵 Web UI による運用

## 設定ルール

このリリースには意図的に以下を含めません。

- `agent.properties`
- ホスト依存の認証情報
- ホスト依存のログ
- 公開すべきでない runtime アーティファクト

付属テンプレートを使用してください。

- `agent.sample.properties`

推奨ルール:

- 対象ホストに `agent.properties` が既に存在する場合は、そのまま保持する
- 存在しない場合は `agent.sample.properties` を `agent.properties` にコピーする

例:

```bash
cp ./agent.sample.properties ./agent.properties
```

## SSL / HTTPS 設定

`agent.properties` に SSL 設定を入れると、`AgenticService` は HTTPS を有効化できます。

対応パターン:

- PEM 証明書 + 秘密鍵
- PKCS#12 / PFX バンドル

### PEM モード

使用する項目:

- `ssl_key` : 証明書ファイルのパス
- `ssl_key_file` : 秘密鍵ファイルのパス
- `ssl_key_password` : 空欄可

例:

```json
"https_port" : 443,
"ssl_key" : "./certs/server.crt",
"ssl_key_file" : "./certs/server.key",
"ssl_key_password" : ""
```

### P12 / PFX モード

使用する項目:

- `ssl_key` : `.p12` または `.pfx` バンドルのパス
- `ssl_key_password` : バンドルのパスワード
- `ssl_key_file` : このモードでは不要

例:

```json
"https_port" : 443,
"ssl_key" : "./certs/server.p12",
"ssl_key_file" : "",
"ssl_key_password" : "your-password"
```

### HTTPS の確認

HTTPS を有効にしたら、次のように確認できます。

```bash
curl -k https://127.0.0.1:PORT/talk/hello
curl -k https://127.0.0.1:PORT/api/hello
```

`PORT` は `https_port` の値に置き換えてください。

## クイックデプロイ

### 方法 1: `deploy_files/` から直接デプロイ

```bash
chmod +x ./AgenticService ./run.sh ./run_bg.sh ./stop.sh
cp ./agent.sample.properties ./agent.properties
./run_bg.sh
```

### 方法 2: tar.gz にしてデプロイ

パッケージ作成:

```bash
tar -czf AgenticService_linux_x86_minimal.tar.gz -C deploy_files .
```

デプロイ:

```bash
mkdir -p /opt/AgenticService
tar -xzf AgenticService_linux_x86_minimal.tar.gz -C /opt/AgenticService
cd /opt/AgenticService
cp ./agent.sample.properties ./agent.properties
chmod +x ./AgenticService ./run.sh ./run_bg.sh ./stop.sh
./run_bg.sh
```

### 方法 3: リポジトリ付属のデプロイヘルパーを使う

このリポジトリにはリモートデプロイスクリプトがあります。

- [deploy_linux_x86_remote.sh](/Users/vader/Codes/Go/AgenticService2/deploy/deploy_linux_x86_remote.sh)

このスクリプトは `deploy_files/` と同じレイアウトを前提にしています。

## デプロイ後の確認

起動後は次のように確認できます。

```bash
curl http://127.0.0.1:PORT/talk/hello
curl http://127.0.0.1:PORT/api/hello
```

`PORT` は `agent.properties` に設定した実際のポートに置き換えてください。

期待される結果:

- Web UI が正常に開く
- `/talk/hello` が応答する
- `/api/hello` が応答する
- セッションを作成できる
- ツールベースのワークフローが実行できる

## 公開前の注意

GitHub Release に載せる前に、次を確認してください。

- `skill/` に公開不適切な内容が含まれていないか
- 秘密のログインファイルや token を含めない
- `agent.properties` をリリース資産に含めない
- ホスト側で生成された runtime データは、可能な限り配布物から外す

## Release Notes の推奨構成

GitHub Release の説明には次を入れると分かりやすいです。

1. 本リリースに含まれる内容
2. 対応プラットフォーム
3. デプロイ手順
4. 設定の注意点
5. 既知の制限

## 既知の制限

- 現在の公開説明は Linux x86_64 を中心にしています。
- `skill/` 内の runtime データは、公開前に人手で確認することを推奨します。
- ホストローカルの実行状態は、移植可能な配布資産として扱わないでください。
