# OpenClaw OLLAMA Qwen2.5 與 NVIDIA MiniMax 模型配置模板

[English](#english) | 繁體中文

本模板提供 OpenClaw 整合 Qwen2.5 遠端模型與 NVIDIA API 的完整配置，解決常見的模型錯誤問題，並提供 Modelfile 設定指南。

## 目錄

- [功能特色](#功能特色)
- [快速開始](#快速開始)
- [環境變數設定](#環境變數設定)
- [Modelfile 設定（重要！）](#modelfile-設定重要)
- [常見錯誤與解決方案](#常見錯誤與解決方案)
- [模型列表](#模型列表)
- [故障排除](#故障排除)
- [進階設定](#進階設定)
- [English Documentation](#english)

---

## 功能特色

- ✅ 支援 Qwen2.5 遠端模型（Ollama）
- ✅ 支援 Qwen2.5 32B（複雜推理）和 14B（快速回應）
- ✅ 整合 NVIDIA MiniMax M2.1 模型
- ✅ 包含 Modelfile 設定指南，正確設定 num_ctx
- ✅ 包含常見錯誤的完整解決方案
- ✅ 環境變數管理，安全保護敏感資料
- ✅ 符合 OpenClaw 標準格式

---

## 快速開始

### 1. 創建 Skill 檔案

將以下檔案放置到 OpenClaw 的 skills 目錄：

```
~/.openclaw/skill/
├── ollamasetup/
│   ├── skill.json              ← 主設定檔
│   └── readme.md               ← 說明文件
```

### 2. 在 Ollama 伺服器上創建 Modelfile（重要！）

**必須先完成此步驟！** OpenClaw 無法直接設定 num_ctx，需透過 Modelfile。

#### 創建 Qwen2.5 14B Modelfile（128K 上下文）

```powershell
# 在 5090 Windows 11 上執行 PowerShell
@"
FROM qwen2.5:14b-instruct-q8_0
PARAMETER num_ctx 131072
"@ | Out-File -Encoding UTF8 "$env:USERPROFILE\Ollama\Modelfiles\Qwen2.5-14B"

ollama create -f "$env:USERPROFILE\Ollama\Modelfiles\Qwen2.5-14B" qwen2.5:14b-instruct-q8_0-ctx131072
```

#### 創建 Qwen2.5 32B Modelfile（64K 上下文）

```powershell
@"
FROM qwen2.5:32b-instruct-q4_1
PARAMETER num_ctx 65536
"@ | Out-File -Encoding UTF8 "$env:USERPROFILE\Ollama\Modelfiles\Qwen2.5-32B"

ollama create -f "$env:USERPROFILE\Ollama\Modelfiles\Qwen2.5-32B" qwen2.5:32b-instruct-q4_1-ctx64k
```

#### 驗證模型

```powershell
ollama list
```

應該看到：
```
qwen2.5:14b-instruct-q8_0-ctx131072      abc123...   15 GB
qwen2.5:32b-instruct-q4_1-ctx64k         def456...   20 GB
```

### 3. 設定環境變數

複製範本並填入你的資料。建立 `.env` 檔案：

```bash
# OpenClaw Gateway 認證 Token
GATEWAY_TOKEN=your_gateway_token_here

# Telegram Bot Token
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here

# NVIDIA API Key
NVIDIA_API_KEY=nvapi-your-nvidia-api-key-here

# Ollama 伺服器 IP 位址
OLLAMA_SERVER_IP=192.168.x.x
```

### 4. 重啟 OpenClaw

```bash
openclaw doctor --fix
```

---

## 環境變數設定

| 變數名稱 | 必填 | 說明 | 範例 |
|----------|------|------|------|
| `GATEWAY_TOKEN` | 是 | OpenClaw Gateway 認證 Token | `f632860762fd9879...` |
| `TELEGRAM_BOT_TOKEN` | 是 | Telegram Bot Token | `8548351054:AAEp...` |
| `NVIDIA_API_KEY` | 是 | NVIDIA API Key | `nvapi-xxxxx-xxxxx` |
| `OLLAMA_SERVER_IP` | 是 | Ollama 伺服器 IP 位址 | `192.168.0.98` |

### 取得 NVIDIA API Key

1. 前往 [NVIDIA API Keys](https://build.nvidia.com/account/keys)
2. 建立新的 API Key
3. 複製並貼到環境變數

### 取得 Telegram Bot Token

1. 聯絡 @BotFather 在 Telegram
2. 輸入 `/newbot` 建立新機器人
3. 複製 Token 並貼到環境變數

---

## Modelfile 設定（重要！）

### 為什麼需要 Modelfile？

**OpenClaw 無法直接傳遞 num_ctx 參數給 Ollama！**

嘗試在 OpenClaw 配置中加入 `numCtx` 會被 `doctor --fix` 自動移除，產生錯誤：
```
Invalid config: models.providers.ollama-remote.models.0: Unrecognized key: "numCtx"
```

### 解決方案：使用 Modelfile

在 Ollama 伺服器上創建 Modelfile，固定 num_ctx 參數。

### Modelfile 範本

#### 範本 1：文字模型（128K 上下文）

```dockerfile
FROM qwen2.5:14b-instruct-q8_0
PARAMETER num_ctx 131072
```

#### 範本 2：文字模型（64K 上下文）

```dockerfile
FROM qwen2.5:32b-instruct-q4_1
PARAMETER num_ctx 65536
```

#### 範本 3：輕量模型（64K 上下文）

```dockerfile
FROM qwen2.5:7b-instruct-q8_0
PARAMETER num_ctx 65536
```

### PowerShell 自動化腳本

```powershell
# 創建 Modelfile 並生成新模型
$modelName = "qwen2.5:14b-instruct-q8_0"
$ctxSize = 131072

$modelfile = @"
FROM $modelName
PARAMETER num_ctx $ctxSize
"@

$modelfilePath = "$env:USERPROFILE\Ollama\Modelfiles\$($modelName.Replace(':', '-').Replace('/', '-'))"
New-Item -ItemType Directory -Force -Path (Split-Path $modelfilePath) | Out-Null
$modelfile | Out-File -FilePath $modelfilePath -Encoding UTF8

ollama create -f $modelfilePath "$modelName-ctx$ctxSize"

Write-Host "模型已創建：$modelName-ctx$ctxSize"
```

### 常見問題

**Q：Modelfile 放在哪裡？**

A：放在 Ollama 伺服器的 `Modelfiles` 目錄：
- Windows: `%USERPROFILE%\Ollama\Modelfiles\`
- Linux: `/etc/ollama/Modelfiles/`

**Q：如何刪除模型？**

```powershell
ollama rm qwen2.5:14b-instruct-q8_0-ctx131072
```

**Q：可以修改已創建的模型嗎？**

A：不行，需要先刪除再重新創建。

---

## 常見錯誤與解決方案

### 錯誤 1：模型不被允許

**錯誤訊息：**

```
Model "ollama-remote/qwen2.5:14b-instruct-q8_0-ctx131072" is not allowed.
Use /models to list providers, or /models <provider> to list models.
```

**錯誤原因：**

該模型未在配置文件的允許清單中註冊。

**解決步驟：**

1. 執行 `/models` 查詢可用模型列表

2. 在 `models.providers.ollama-remote.models` 中加入：

   ```json
   {
     "id": "qwen2.5:14b-instruct-q8_0-ctx131072",
     "name": "Qwen2.5 14B Instruct Q8_0 (128K context)",
     "reasoning": false,
     "input": ["text"],
     "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
     "contextWindow": 131072,
     "maxTokens": 131072
   }
   ```

3. 在 `agents.defaults.models` 中註冊：

   ```json
   "ollama-remote/qwen2.5:14b-instruct-q8_0-ctx131072": {}
   ```

4. 執行 `openclaw doctor --fix`

---

### 錯誤 2：OpenClaw 不支援 thinking 參數

**錯誤訊息：**

```
Invalid config: models.providers.ollama-remote.models.0: Unrecognized key: "thinking"
```

**錯誤原因：**

OpenClaw 配置中 `"thinking"` 不是有效的 key。

**解決方案：**

- 不要在 OpenClaw 配置中加入 "thinking" 參數
- 改用非 thinking 版本模型（如 `qwen2.5:14b-instruct-*` 而非 `qwen3-vl:8b-thinking-*`）

---

### 錯誤 3：OpenClaw 不支援 numCtx

**錯誤訊息：**

```
Invalid config: models.providers.ollama-remote.models.0: Unrecognized key: "numCtx"
```

**錯誤原因：**

`numCtx`/`num_ctx` 不是 OpenClaw 支援的 key。

**解決方案：**

使用 Modelfile 在 Ollama 端設定 num_ctx（詳見上方 Modelfile 設定章節）。

---

### 錯誤 4：Llama 3.2 Vision 不支援 tools

**錯誤訊息：**

```
400 ... does not support tools
```

**錯誤原因：**

Llama 3.2 Vision 模型本身不支援 tools/function calling。

**解決方案：**

- 刪除 Llama 3.2 Vision 模型
- 改用 Qwen2.5 系列（支援 tools）

---

### 錯誤 5：Ollama 連線失敗

**錯誤訊息：**

```
Connection refused to http://192.168.x.x:11434/v1
```

**解決步驟：**

1. 確認 Ollama 伺服器正在運行

   ```bash
   curl http://localhost:11434/api/tags
   ```

2. 檢查 IP 位址設定

3. 確認防火牆允許連線

   ```bash
   sudo ufw allow 11434
   ```

---

### 錯誤 6：Configuration key 不支援

**錯誤訊息：**

```
Warning: agents.entries is not supported
```

**解決步驟：**

1. 移除 `agents.entries` 包裝層
2. 移除 `hooks.internal.entries` 包裝層
3. 移除 `plugins.entries` 包裝層
4. 執行 `openclaw doctor --fix`

---

## 模型列表

### Qwen2.5 遠端模型（Ollama）

| 模型 ID | 名稱 | 上下文 | VRAM | 用途 | 狀態 |
|---------|------|--------|------|------|------|
| `qwen2.5:32b-instruct-q4_1-ctx64k` | Qwen2.5 32B Instruct Q4_1 | 64K | ~20 GB | 複雜推理、深度任務 | ✅ 正常 |
| `qwen2.5:14b-instruct-q8_0-ctx131072` | Qwen2.5 14B Instruct Q8_0 | 128K | 15 GB | 日常對話、快速回應 | ✅ 正常 |

### 舊版模型（不再使用）

| 模型 ID | 問題 |
|---------|------|
| `qwen3-vl:8b-thinking-bf16` | 不支援 OpenClaw 的 think 參數 |
| `llama3.2-vision:*` | 不支援 tools |

### NVIDIA 模型

| 模型 ID | 名稱 | 用途 | Context Window |
|---------|------|------|----------------|
| `minimaxai/minimax-m2.1` | NVIDIA MiniMax M2.1 | 一般對話、推理 | 200,000 tokens |

### 模型比較

| 任務類型 | 推薦模型 |
|----------|----------|
| 日常對話、快速問答 | **Qwen2.5 14B**（128K 更快） |
| 複雜代碼、深度分析 | **Qwen2.5 32B**（推理更強） |
| 長文件處理（>64K） | **Qwen2.5 14B**（128K 上下文） |
| 雲端備用 | **NVIDIA MiniMax**（200K 上下文） |

---

## 故障排除

### Q1：如何查詢可用的模型？

```bash
/models              # 列出所有 providers
/models ollama-remote # 列出 ollama-remote provider 的模型
```

### Q2：如何切換模型？

```bash
/model ollama-remote/qwen2.5:14b-instruct-q8_0-ctx131072  # 14B
/model ollama-remote/qwen2.5:32b-instruct-q4_1-ctx64k     # 32B
/model nvidia-minimax/minimaxai/minimax-m2.1             # NVIDIA
```

### Q3：如何新增自定義模型？

1. 在 Ollama 伺服器上創建 Modelfile
2. 執行 `ollama create` 創建新模型
3. 在 `models.providers.ollama-remote.models` 中加入設定
4. 在 `agents.defaults.models` 中註冊
5. 執行 `openclaw doctor --fix`

### Q4：VRAM 不足怎麼辦？

選擇較小的模型：

| 模型 | VRAM |
|------|------|
| Qwen2.5 7B | ~8 GB |
| Qwen2.5 14B | ~15 GB |
| Qwen2.5 32B | ~20 GB |

### Q5：修改配置後需要做什麼？

```bash
openclaw doctor --fix
```

### Q6：如何備份配置？

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup.$(date +%Y%m%d)
```

---

## 進階設定

### 自定義模型參數

```json
{
  "models": {
    "providers": {
      "ollama-remote": {
        "models": [
          {
            "id": "qwen2.5:14b-instruct-q8_0-ctx131072",
            "name": "Qwen2.5 14B Instruct Q8_0",
            "reasoning": false,
            "input": ["text"],
            "temperature": 0.7,
            "maxTokens": 131072,
            "contextWindow": 131072
          }
        ]
      }
    }
  }
}
```

### 調整工作區設定

```json
{
  "agents": {
    "defaults": {
      "workspace": "/home/ubuntu/.openclaw/workspace",
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      }
    }
  }
}
```

### 多伺服器配置

```json
{
  "models": {
    "providers": {
      "ollama-remote-home": {
        "baseUrl": "http://192.168.1.100:11434/v1",
        "models": [...]
      },
      "ollama-remote-office": {
        "baseUrl": "http://192.168.1.200:11434/v1",
        "models": [...]
      }
    }
  }
}
```

---

## 安全性建議

⚠️ **重要提醒：**

1. **不要**將包含真實 API Key 的配置檔案提交到 Git

2. **不要**在程式碼中硬編寫 API Keys，請使用環境變數

3. **定期**輪換 API Keys，建議每 90 天更換一次

4. **使用**環境變數管理敏感資料

### .gitignore 範本

```bash
.env
.env.local
.env.*.local
*.backup
*.bak
```

---

## 貢獻指南

歡迎提交 Issue 和 Pull Request！

1. Fork 此專案
2. 建立分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -am 'Add some feature'`
4. 推送分支：`git push origin feature/your-feature`
5. 建立 Pull Request

---

## 授權

MIT License

---

## 聯絡

- GitHub Issues: https://github.com/openclaw/skill-qwen-nvidia-model-config/issues
- OpenClaw Discord: https://discord.gg/openclaw

---

<a name="english"></a>

# OpenClaw OLLAMA Qwen2.5 and NVIDIA MiniMax Model Configuration Template

This template provides a complete configuration for integrating Qwen2.5 remote models and NVIDIA API with OpenClaw, including Modelfile setup guide and common error solutions.

### Quick Start

1. Create Modelfile on Ollama server (IMPORTANT!)
2. Place skill files in `~/.openclaw/skill/`
3. Set environment variables
4. Run `openclaw doctor --fix`

### Modelfile Setup (Critical!)

OpenClaw cannot pass num_ctx directly. Use Modelfile on Ollama server:

```powershell
# PowerShell
@"
FROM qwen2.5:14b-instruct-q8_0
PARAMETER num_ctx 131072
"@ | Out-File -Encoding UTF8 "$env:USERPROFILE\Ollama\Modelfiles\Qwen2.5-14B"

ollama create -f "$env:USERPROFILE\Ollama\Modelfiles\Qwen2.5-14B" qwen2.5:14b-instruct-q8_0-ctx131072
```

### Environment Variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `GATEWAY_TOKEN` | Yes | OpenClaw Gateway Auth Token | `f632860762fd9879...` |
| `TELEGRAM_BOT_TOKEN` | Yes | Telegram Bot Token | `8548351054:AAEp...` |
| `NVIDIA_API_KEY` | Yes | NVIDIA API Key | `nvapi-xxxxx-xxxxx` |
| `OLLAMA_SERVER_IP` | Yes | Ollama Server IP | `192.168.0.98` |

### Supported Models

| Model | Context | VRAM | Use Case |
|-------|---------|------|----------|
| `qwen2.5:32b-instruct-q4_1-ctx64k` | 64K | ~20GB | Complex reasoning |
| `qwen2.5:14b-instruct-q8_0-ctx131072` | 128K | 15GB | General purpose |
| `minimaxai/minimax-m2.1` (NVIDIA) | 200K | Cloud | Cloud backup |

### Common Error Solutions

1. **Model not allowed**: Register in `models.providers.ollama-remote.models`
2. **thinking parameter error**: Use non-thinking models (qwen2.5 instead of qwen3-vl)
3. **numCtx not supported**: Use Modelfile on Ollama server
4. **Llama Vision tools error**: Use Qwen2.5 (supports tools)
5. **Ollama connection failed**: Check server IP and firewall
6. **NVIDIA API auth failed**: Update API Key

---

**Last Updated**: 2026-02-03
**Version**: 1.1.0
