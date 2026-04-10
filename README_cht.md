# AgenticService GitHub 發布說明

此目錄為 `AgenticService` 的 GitHub 發布資產與部署資料根目錄。

語言版本：

- [README.md](./README.md)
- [README_cht.md](./README_cht.md)
- [README_jp.md](./README_jp.md)

`AgenticService` 是以 Go 撰寫的本地優先 Agentic orchestration service，整合了：

- 結構化工具執行
- 多步驟任務工作流
- 本地 shell 與檔案操作
- task-backed 長任務處理
- 以本地知識目錄為基礎的 RAG 檢索
- 以 Skill 驅動的流程啟動與限制
- 內建 Web Console，用於對話、規劃、除錯、Skills 與 RAG 管理

## 發布範圍

目前這版發布主要針對：

- Linux x86_64

以下平台**不列入**本次公開發布說明：

- `linux_arm32_sf`

## 發布資產

建議 GitHub Release 內容包含：

- `AgenticService_linux_x86_minimal.tar.gz`
- `deploy_files/`
- Release Notes

## 資產結構

### `deploy_files/`

這是目前用來產生最小部署包的發布目錄。

典型內容如下：

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

### 平台編譯產物

`dist/` 下也可能包含平台編譯輸出，例如：

- `linux_x86_64/`
- `linux_arm64/`
- `macos_arm64/`

但 GitHub 對外發布時，應以 `deploy_files/` 作為部署入口。

## 服務主要特性

本次發布的服務能力包含：

- 對話式任務執行
- Plan 與 Tool Trace 視覺化
- shell、檔案、搜尋與 task 類工具
- Skill 驅動的流程注入與工具限制
- 本地 RAG 與 scoped retrieval
- 內建 Web UI 作為操作入口

## 設定檔原則

本發布包**刻意不包含**：

- `agent.properties`
- 主機專屬憑證
- 主機專屬 log
- 不適合公開的 runtime 狀態檔

請使用內附的：

- `agent.sample.properties`

建議規則：

- 若目標主機已存在 `agent.properties`，保留原檔
- 若不存在，再由 `agent.sample.properties` 複製建立

範例：

```bash
cp ./agent.sample.properties ./agent.properties
```

## SSL / HTTPS 設定

`AgenticService` 在 `agent.properties` 完成 SSL 設定後即可啟用 HTTPS。

目前支援兩種方式：

- PEM 憑證 + 私鑰
- PKCS#12 / PFX 憑證包

### PEM 模式

請設定：

- `ssl_key`：憑證檔路徑
- `ssl_key_file`：私鑰檔路徑
- `ssl_key_password`：可留空

範例：

```json
"https_port" : 443,
"ssl_key" : "./certs/server.crt",
"ssl_key_file" : "./certs/server.key",
"ssl_key_password" : ""
```

### P12 / PFX 模式

請設定：

- `ssl_key`：`.p12` 或 `.pfx` 憑證包路徑
- `ssl_key_password`：憑證包密碼
- `ssl_key_file`：此模式可留空

範例：

```json
"https_port" : 443,
"ssl_key" : "./certs/server.p12",
"ssl_key_file" : "",
"ssl_key_password" : "your-password"
```

### HTTPS 驗證

啟用 HTTPS 後，可用以下方式驗證：

```bash
curl -k https://127.0.0.1:PORT/talk/hello
curl -k https://127.0.0.1:PORT/api/hello
```

請將 `PORT` 替換成實際設定的 `https_port`。

## 快速部署

### 方式一：直接使用 `deploy_files/`

```bash
chmod +x ./AgenticService ./run.sh ./run_bg.sh ./stop.sh
cp ./agent.sample.properties ./agent.properties
./run_bg.sh
```

### 方式二：先打包成 tar.gz 再部署

打包：

```bash
tar -czf AgenticService_linux_x86_minimal.tar.gz -C deploy_files .
```

部署：

```bash
mkdir -p /opt/AgenticService
tar -xzf AgenticService_linux_x86_minimal.tar.gz -C /opt/AgenticService
cd /opt/AgenticService
cp ./agent.sample.properties ./agent.properties
chmod +x ./AgenticService ./run.sh ./run_bg.sh ./stop.sh
./run_bg.sh
```

### 方式三：使用專案內建遠端部署腳本

本專案提供遠端部署腳本：

- [deploy_linux_x86_remote.sh](/Users/vader/Codes/Go/AgenticService2/deploy/deploy_linux_x86_remote.sh)

此腳本目前就是以 `deploy_files/` 的結構為基礎進行部署。

## 部署後驗證

啟動完成後，可用以下方式檢查：

```bash
curl http://127.0.0.1:PORT/talk/hello
curl http://127.0.0.1:PORT/api/hello
```

請將 `PORT` 替換成 `agent.properties` 內設定的實際通訊埠。

正常情況下，應可確認：

- Web UI 可正常開啟
- `/talk/hello` 可回應
- `/api/hello` 可回應
- 可建立 Session
- 可執行工具型工作流

## 發布前注意事項

在上傳到 GitHub Release 前，建議先確認：

- `skill/` 內是否含有不適合公開的內容
- 不要把私密登入檔或 token 一起發布
- `agent.properties` 不要放進發布資產
- 主機執行後產生的 runtime 資料盡量不要直接納入發布包

## 建議的 Release Notes 結構

建議 GitHub Release 說明包含：

1. 本版包含內容
2. 支援平台
3. 部署步驟
4. 設定說明
5. 已知限制

## 已知限制

- 目前發布說明以 Linux x86_64 為主
- `skill/` 內的 runtime 資料在公開發布前應人工檢查
- 主機本地執行狀態不應視為可攜式發布資產
