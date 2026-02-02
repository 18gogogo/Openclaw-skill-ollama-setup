# OpenClaw Qwen3 VL 與 NVIDIA MiniMax 模型配置模板

[English](#english) | 繁體中文

本模板提供 OpenClaw 整合 Qwen3 VL 遠端模型與 NVIDIA API 的完整配置，解決常見的模型錯誤問題。

## 目錄

- [功能特色](#功能特色)
- [快速開始](#快速開始)
- [環境變數設定](#環境變數設定)
- [常見錯誤與解決方案](#常見錯誤與解決方案)
- [模型列表](#模型列表)
- [故障排除](#故障排除)
- [進階設定](#進階設定)
- [English Documentation](#english)

---

## 功能特色

- ✅ 支援 Qwen3 VL 遠端模型（Ollama）
- ✅ 整合 NVIDIA MiniMax M2.1 模型
- ✅ 包含常見錯誤的完整解決方案
- ✅ 環境變數管理，安全保護敏感資料
- ✅ 符合 OpenClaw 標準格式

---

## 快速開始

### 1. 創建 Skill 檔案

將以下檔案放置到 OpenClaw 的 skills 目錄：

```
~/.openclaw/skill/
├── qwen-nvidia-model-config.json          ← 主設定檔
└── qwen-nvidia-model-config-readme.md     ← 說明文件
```

### 2. 設定環境變數

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

### 3. 重啟 OpenClaw

```bash
openclaw restart
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

## 常見錯誤與解決方案

### 錯誤 1：模型不被允許

**錯誤訊息：**

```
Model "ollama-remote/qwen3-vl:8b-thinking-bf16" is not allowed.
Use /models to list providers, or /models <provider> to list models.
```

**錯誤原因：**

該模型未在配置文件的允許清單中註冊。當你嘗試使用一個未被明確允許的模型時，OpenClaw 會拒絕此請求，這是系統的安全機制。

**解決步驟：**

1. 執行 `/models` 查詢可用模型列表
   
   在 OpenClaw 中輸入指令，系統會回傳所有已註冊的模型清單。這個步驟可以讓你確認哪些模型是被允許使用的。

2. 確認模型名稱格式正確
   
   模型名稱必須完全符合配置檔案中的 ID。檢查是否有拼寫錯誤、大小寫是否正確、前後是否有空格。

3. 在配置檔案中加入模型設定
   
   在 `models.providers.ollama-remote.models` 區塊中新增以下內容：

   ```json
   {
     "id": "qwen3-vl:8b-thinking-bf16",
     "name": "Qwen3 VL (Remote Ollama)",
     "reasoning": true,
     "input": ["text"],
     "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
     "contextWindow": 163840,
     "maxTokens": 65536
   }
   ```

4. 同步更新 agents 配置
   
   在 `agents.defaults.models` 中註冊模型 ID：

   ```json
   {
     "ollama-remote/qwen3-vl:8b-thinking-bf16": {}
   }
   ```

5. 儲存設定並重啟 OpenClaw

   儲存所有修改後，必須重啟 OpenClaw 服務使設定生效。

**預防措施：**

- 修改模型前先執行 `/models` 確認可用選項
- 確保模型名稱符合系統規範
- 新增模型時必須同步更新 config 的多個區塊
- 建立模型清單文件，記錄已成功使用的模型

---

### 錯誤 2：/restart 指令被禁用

**錯誤訊息：**

```
⚠️ /restart is disabled. Set commands.restart=true to enable.
```

**錯誤原因：**

重啟指令在預設情況下是禁用的，這是為了防止意外重啟導致服務中斷。你需要手動在配置中啟用此功能。

**解決步驟：**

在配置文件根層級加入以下設定：

```json
{
  "commands": {
    "restart": true
  }
}
```

儲存後重啟 OpenClaw 即可使用 `/restart` 指令。

**使用情境：**

啟用後，你可以隨時輸入 `/restart` 來重新啟動 OpenClaw 服務，這在修改配置後需要使設定生效時非常有用。

---

### 錯誤 3：Ollama 連線失敗

**錯誤訊息：**

```
Connection refused to http://192.168.x.x:11434/v1
```

或

```
ECONNREFUSED 192.168.x.x:11434
```

**錯誤原因：**

1. Ollama 伺服器未啟動
2. IP 位址錯誤
3. 防火牆阻擋
4. 網路連線問題

**解決步驟：**

1. 確認 Ollama 伺服器正在運行

   在 Ollama 伺服器上執行以下指令：

   ```bash
   # 檢查 Ollama 版本
   curl http://localhost:11434/api/version

   # 測試 API 是否正常
   curl http://localhost:11434/api/tags
   ```

2. 檢查 IP 位址設定是否正確

   確認配置檔案中的 `OLLAMA_SERVER_IP` 與實際伺服器 IP 一致。

3. 確認防火牆允許連線

   ```bash
   # 在 Ollama 伺服器上執行
   # 開放 11434 連接埠
   sudo ufw allow 11434

   # 或者使用 iptables
   sudo iptables -A INPUT -p tcp --dport 11434 -j ACCEPT
   ```

4. 在客戶端測試連線

   ```bash
   curl http://192.168.x.x:11434/api/tags
   ```

   如果連線成功，你會看到已下載的模型清單。

**疑難排解：**

- 檢查 Ollama 服務狀態：`systemctl status ollama`
- 查看 Ollama 日誌：`journalctl -u ollama`
- 確認伺服器IP是否可 Ping 通：`ping 192.168.x.x`

---

### 錯誤 4：NVIDIA API 驗證失敗

**錯誤訊息：**

```
401 Unauthorized - Invalid API Key
```

或

```
Error: Authentication failed
```

**錯誤原因：**

1. API Key 過期或無效
2. API Key 權限不足
3. API Key 已達到使用限制

**解決步驟：**

1. 前往 [NVIDIA API Keys](https://build.nvidia.com/account/keys) 檢查 API Key 狀態

   登入後在帳戶頁面查看 API Keys 列表，確認 Key 是否處於啟用狀態。

2. 如有必要，建立新的 API Key

   如果舊的 Key 已經過期或被撤銷，點擊「Create New Key」按鈕建立新的 API Key。

3. 更新環境變數

   將新的 API Key 填入環境變數檔案：

   ```bash
   NVIDIA_API_KEY=nvapi-your-new-api-key-here
   ```

4. 重啟 OpenClaw

   讓新的 API Key 生效。

---

## 模型列表

### Qwen3 VL 遠端模型（Ollama）

| 模型 ID | 名稱 | 用途 | Context Window |
|---------|------|------|----------------|
| `qwen3-vl:8b-thinking-bf16` | Qwen3 VL | 視覺理解、多模態 | 163,840 tokens |

### NVIDIA 模型

| 模型 ID | 名稱 | 用途 | Context Window |
|---------|------|------|----------------|
| `minimaxai/minimax-m2.1` | NVIDIA MiniMax M2.1 | 一般對話、推理 | 200,000 tokens |

---

## 故障排除

### Q1：如何查詢可用的模型？

在 OpenClaw 中輸入：

```bash
/models              # 列出所有 providers
/models ollama-remote # 列出 ollama-remote provider 的模型
/models nvidia-minimax # 列出 nvidia-minimax provider 的模型
```

系統會回傳已註冊的模型清單，包含模型 ID、名稱和其他資訊。

### Q2：如何切換預設模型？

修改 `agents.defaults.model.primary` 設定：

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama-remote/qwen3-vl:8b-thinking-bf16"
      }
    }
  }
}
```

切換後重啟 OpenClaw 即可使用新的預設模型。

### Q3：如何新增自定義模型？

1. 在 `models.providers.<provider>.models` 中加入模型設定
   
   根據你要新增的模型類型，選擇對應的 provider 區塊。

2. 在 `agents.defaults.models` 中註冊模型 ID
   
   這一步是必须的，否則模型雖然可以查詢，但無法實際使用。

3. 儲存並重啟

### Q4：重啟後設定消失？

確保使用正確的環境變數語法：

```json
{
  "apiKey": "${NVIDIA_API_KEY}",
  "baseUrl": "http://${OLLAMA_SERVER_IP}:11434/v1"
}
```

環境變數會在啟動時自動替換為實際值，這樣可以避免敏感資料被硬編碼在配置檔案中。

### Q5：如何備份配置？

```bash
# 複製配置檔案
cp /path/to/openclaw.json /path/to/openclaw.json.backup

# 或者使用時間戳記備份
cp openclaw.json openclaw.json.backup.$(date +%Y%m%d)
```

### Q6：如何查看目前使用的模型？

執行以下指令：

```bash
/status
```

系統會顯示目前的工作階段狀態，包括使用的模型名稱。

### Q7：修改配置後需要做什麼？

1. 儲存配置檔案
2. 執行 `/restart` 或重啟 OpenClaw 服務
3. 驗證設定是否生效

---

## 進階設定

### 自定義模型參數

你可以為模型添加額外的參數設定：

```json
{
  "models": {
    "providers": {
      "ollama-remote": {
        "models": [
          {
            "id": "qwen3-vl:8b-thinking-bf16",
            "name": "Qwen3 VL (Remote Ollama)",
            "reasoning": true,
            "input": ["text"],
            "temperature": 0.7,
            "top_p": 0.9,
            "maxTokens": 65536,
            "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
            "contextWindow": 163840
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
      "workspace": "/path/to/your/workspace",
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      },
      "compaction": {
        "mode": "safeguard"
      }
    }
  }
}
```

### 啟用詳細日誌

```json
{
  "hooks": {
    "internal": {
      "entries": {
        "command-logger": {
          "enabled": true
        }
      }
    }
  }
}
```

### 多伺服器配置

如果你有多個 Ollama 伺服器，可以配置多個 provider：

```json
{
  "models": {
    "providers": {
      "ollama-remote-home": {
        "baseUrl": "http://192.168.1.100:11434/v1",
        "apiKey": "ollama-local",
        "api": "openai-completions",
        "models": [...]
      },
      "ollama-remote-office": {
        "baseUrl": "http://192.168.1.200:11434/v1",
        "apiKey": "ollama-local",
        "api": "openai-completions",
        "models": [...]
      }
    }
  }
}
```

### 設定預設模型為 NVIDIA

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "nvidia-minimax/minimaxai/minimax-m2.1"
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

4. **使用**環境變數管理敏感資料，確保資訊安全

5. **限制**API Key 的權限範圍，只給予必要的權限

6. **監控**API 使用情況，及時發現異常使用

### 環境變數檔案保護

在專案根目錄建立 `.gitignore`：

```bash
# 忽略環境變數檔案
.env
.env.local
.env.*.local

# 忽略備份檔案
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

# OpenClaw Qwen3 VL and NVIDIA MiniMax Model Configuration Template

This template provides a complete configuration for integrating Qwen3 VL remote models and NVIDIA API with OpenClaw, including solutions for common model errors.

### Quick Start

1. Place skill files in `~/.openclaw/skill/`
2. Set environment variables
3. Restart OpenClaw

### Environment Variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `GATEWAY_TOKEN` | Yes | OpenClaw Gateway Auth Token | `f632860762fd9879...` |
| `TELEGRAM_BOT_TOKEN` | Yes | Telegram Bot Token | `8548351054:AAEp...` |
| `NVIDIA_API_KEY` | Yes | NVIDIA API Key | `nvapi-xxxxx-xxxxx` |
| `OLLAMA_SERVER_IP` | Yes | Ollama Server IP | `192.168.0.98` |

### Supported Models

- **Ollama**: Qwen3 VL
- **NVIDIA**: MiniMax M2.1

### Common Error Solutions

1. **Model not allowed**: Register model in `models.providers.ollama-remote.models`
2. **/restart disabled**: Set `"commands": { "restart": true }`
3. **Ollama connection failed**: Check server IP and firewall settings
4. **NVIDIA API auth failed**: Update API Key in environment variables

---

**Last Updated**: 2026-02-02
**Version**: 1.0.0
