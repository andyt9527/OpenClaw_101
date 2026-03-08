# OpenClaw 完整配置指南

本指南提供 OpenClaw 的全方位配置说明，包括国内 API 接入、本地模型部署、局域网访问等场景。

---

## 目录

1. [快速开始](#一快速开始)
2. [前置条件与环境检查](#二前置条件与环境检查)
3. [国内 API 配置](#三国内-api-配置)
4. [本地模型配置（llama.cpp）](#四本地模型配置llamacpp)
5. [局域网访问配置](#五局域网访问配置)
6. [基础使用与测试](#六基础使用与测试)
7. [常见问题与故障排查](#七常见问题与故障排查)
8. [附录](#八附录)

---

## 一、快速开始

### 1.1 安装 OpenClaw

```bash
# 官方一键安装脚本
curl -fsSL https://molt.bot/install.sh | bash

# 安装完成后运行初始化向导
openclaw onboard --install-daemon
```

### 1.2 快速配置国内 API（以 MiniMax 为例）

```bash
# 1. 备份配置
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# 2. 设置环境变量
export MINIMAX_CN_API_KEY="your_api_key_here"

# 3. 编辑配置文件，添加 provider 配置
nano ~/.openclaw/openclaw.json
```

添加以下配置：

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "minimax-cn": {
        "baseUrl": "https://api.minimaxi.com/anthropic",
        "apiKey": "${MINIMAX_CN_API_KEY}",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "MiniMax-M2.1",
            "name": "MiniMax M2.1",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 204800,
            "maxTokens": 131072
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "minimax-cn/MiniMax-M2.1"
      }
    }
  }
}
```

```bash
# 4. 重启 Gateway
openclaw gateway restart

# 5. 验证配置
openclaw models list
openclaw agent --message "Hello, are you working?" --session-id main
```

---

## 二、前置条件与环境检查

### 2.1 检查 OpenClaw 安装

```bash
# 检查是否在 PATH 中
which openclaw

# 查看版本（要求 ≥ 2026.1.24-3）
openclaw --version

# 检查运行状态
ps aux | grep openclaw
```

### 2.2 检查 Node.js 版本

```bash
node --version
```

**要求**: Node.js ≥ 22

### 2.3 配置文件位置

| 文件 | 路径 | 说明 |
|------|------|------|
| 主配置 | `~/.openclaw/openclaw.json` | 全局配置 |
| Main Agent 私有配置 | `~/.openclaw/agents/main/agent/models.json` | 可能覆盖全局配置 |
| Gateway 日志 | `~/.openclaw/logs/gateway.log` | 运行日志 |
| 错误日志 | `~/.openclaw/logs/gateway.err.log` | 错误日志 |

### 2.4 备份与恢复

**备份配置**：
```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup
```

**恢复配置**：
```bash
cp ~/.openclaw/openclaw.json.backup ~/.openclaw/openclaw.json
openclaw gateway restart
```

---

## 三、国内 API 配置

### 3.1 获取 API Key

| 服务商 | 控制台地址 | API Key 位置 |
|--------|-----------|--------------|
| **MiniMax** | https://www.minimaxi.com | 控制台 → API Keys |
| **智谱 AI** | https://bigmodel.cn | 控制台 → API Keys |
| **阿里云百炼** | https://dashscope.aliyuncs.com | 百炼平台 → 密钥管理 |

### 3.2 MiniMax 配置

**环境变量**（推荐）：
```bash
export MINIMAX_CN_API_KEY="your_api_key_here"
```

**配置示例**：
```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "minimax-cn": {
        "baseUrl": "https://api.minimaxi.com/anthropic",
        "apiKey": "${MINIMAX_CN_API_KEY}",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "MiniMax-M2.1",
            "name": "MiniMax M2.1",
            "reasoning": true,
            "input": ["text"],
            "cost": {
              "input": 0.3,
              "output": 1.2,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 204800,
            "maxTokens": 131072
          }
        ]
      }
    }
  }
}
```

### 3.3 智谱（GLM）配置

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "zhipu-cn": {
        "baseUrl": "https://open.bigmodel.cn/api/anthropic",
        "apiKey": "YOUR_ZHIPU_API_KEY",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "glm-4.7",
            "name": "GLM-4.7",
            "reasoning": true,
            "input": ["text"],
            "cost": {
              "input": 0.1,
              "output": 0.1,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
    }
  }
}
```

### 3.4 阿里云百炼配置

**环境变量**：
```bash
export DASHSCOPE_API_KEY="your_api_key_here"
```

**通义千问 3 Max**：
```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "bailian": {
        "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "apiKey": "${DASHSCOPE_API_KEY}",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen3-max-2026-01-23",
            "name": "通义千问 3 Max",
            "reasoning": false,
            "input": ["text"],
            "cost": {
              "input": 0.0025,
              "output": 0.01,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 262144,
            "maxTokens": 65536
          }
        ]
      }
    }
  }
}
```

### 3.5 多 API 同时配置

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "minimax-cn": {
        "baseUrl": "https://api.minimaxi.com/anthropic",
        "apiKey": "${MINIMAX_CN_API_KEY}",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "MiniMax-M2.1",
            "name": "MiniMax M2.1",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 204800,
            "maxTokens": 131072
          }
        ]
      },
      "zhipu-cn": {
        "baseUrl": "https://open.bigmodel.cn/api/anthropic",
        "apiKey": "${ZHIPU_API_KEY}",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "glm-4.7",
            "name": "GLM-4.7",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      },
      "bailian": {
        "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "apiKey": "${DASHSCOPE_API_KEY}",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen3-max-2026-01-23",
            "name": "通义千问 3 Max",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 262144,
            "maxTokens": 65536
          }
        ]
      }
    }
  }
}
```

### 3.6 设置默认模型

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "minimax-cn/MiniMax-M2.1"
      },
      "models": {
        "minimax-cn/MiniMax-M2.1": {}
      }
    }
  }
}
```

---

## 四、本地模型配置（llama.cpp）

### 4.1 启动 llama.cpp

**推荐启动命令**：
```bash
./llama-server \
  -m ~/workspace/qwen3.5-4b/qwen3.5-4B-Q4_0.gguf \
  --host 127.0.0.1 \
  --port 8080 \
  -c 16384 \
  --jinja \
  --reasoning-format none \
  --chat-template-kwargs '{"enable_thinking":false}'
```

**关键参数说明**：
| 参数 | 说明 |
|------|------|
| `--port 8080` | 与 OpenClaw 配置保持一致 |
| `-c 16384` | 上下文窗口，必须 ≥ 16000（OpenClaw 最低要求） |
| `--jinja` | 启用现代模板处理 |
| `--reasoning-format none` | 避免输出进入 `reasoning_content` |
| `--chat-template-kwargs '{"enable_thinking":false}'` | 关闭 thinking 模式 |

### 4.2 验证 llama.cpp 服务

**检查模型列表**：
```bash
curl http://127.0.0.1:8080/v1/models
```

**测试聊天接口**：
```bash
curl http://127.0.0.1:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model":"qwen3.5-4B-Q4_0.gguf",
    "messages":[{"role":"user","content":"Reply with exactly: OK"}],
    "max_tokens":64,
    "stream":false
  }'
```

**期望返回**：
```json
{"choices":[{"message":{"content":"OK"}}]}
```

如果 `content` 为空而内容在 `reasoning_content` 中，说明启动参数需要调整。

### 4.3 OpenClaw 全局配置

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "llamacpp": {
        "baseUrl": "http://127.0.0.1:8080/v1",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen3.5-4B-Q4_0.gguf",
            "name": "Qwen 3.5 4B Local",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 16384,
            "maxTokens": 4096
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "llamacpp/qwen3.5-4B-Q4_0.gguf"
      }
    }
  },
  "timeoutSeconds": 600
}
```

### 4.4 重要：检查 Main Agent 私有配置

**关键文件**：`~/.openclaw/agents/main/agent/models.json`

⚠️ **此文件会覆盖全局配置的 provider 设置！**

**检查内容**：
```bash
cat ~/.openclaw/agents/main/agent/models.json
```

**确保 baseUrl 正确**：
```json
{
  "llamacpp": {
    "baseUrl": "http://127.0.0.1:8080/v1"
  }
}
```

**常见错误**：私有配置仍指向旧端口 `8000`，而实际服务在 `8080`。

### 4.5 推荐的 Main Agent 配置

**精简版（仅本地模型）**：
```json
{
  "llamacpp": {
    "baseUrl": "http://127.0.0.1:8080/v1",
    "api": "openai-completions"
  },
  "models": [
    {
      "id": "qwen3.5-4B-Q4_0.gguf",
      "provider": "llamacpp",
      "contextWindow": 16384
    }
  ]
}
```

---

## 五、局域网访问配置

### 5.1 配置 Gateway

修改 `~/.openclaw/openclaw.json` 中的 `gateway` 配置：

```json
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "auth": {
      "mode": "token",
      "token": "your_secure_token_here"
    },
    "controlUi": {
      "allowInsecureAuth": true
    },
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    }
  }
}
```

**关键修改**：
- `bind`: 从 `loopback` 改为 `lan`
- `controlUi.allowInsecureAuth`: 添加并设为 `true`

### 5.2 获取访问信息

**局域网 IP**：
```bash
ip addr show | grep -E "inet " | awk '{print $2}' | cut -d'/' -f1 | grep -v "^127"
```

**Token**：
```bash
grep -A2 '"token"' ~/.openclaw/openclaw.json
```

### 5.3 访问 Web UI

访问地址：`http://<局域网IP>:18789/`

例如：`http://192.168.0.241:18789/`

### 5.4 安全建议

⚠️ **风险提示**：启用局域网访问后，同一网络内的任何设备都可以访问 OpenClaw。

**最佳实践**：
- ✅ 仅在受信任的网络使用（家庭/办公室）
- ❌ 避免在公共 WiFi 使用

**生成强 Token**：
```bash
openssl rand -hex 32
```

**恢复本地访问**：
```json
{
  "gateway": {
    "bind": "loopback",
    "controlUi": {
      "allowInsecureAuth": false
    }
  }
}
```

---

## 六、基础使用与测试

### 6.1 Gateway 管理

```bash
# 查看状态
openclaw gateway status

# 重启
openclaw gateway restart

# 停止
openclaw gateway stop

# 启动
openclaw gateway start
```

### 6.2 模型操作

```bash
# 列出可用模型
openclaw models list

# 切换模型（在对话中）
/model minimax-cn/MiniMax-M2.1
```

### 6.3 对话测试

**基础测试**：
```bash
openclaw agent --message "Hello, are you working?" --session-id main
```

**指定模型测试**：
```bash
openclaw agent --message "Summarize this." \
  --model minimax-cn/MiniMax-M2.1 \
  --session-id main
```

**本地模型测试**：
```bash
openclaw agent --agent main --message "Reply with exactly: OK" --local
```

### 6.4 查看日志

```bash
# 实时日志
tail -f ~/.openclaw/logs/gateway.log

# 错误日志
cat ~/.openclaw/logs/gateway.err.log
```

---

## 七、常见问题与故障排查

### 7.1 安装与启动问题

#### Q: `openclaw` 命令找不到

**解决步骤**：
```bash
# 1. 确认安装
which openclaw

# 2. 重新加载 shell 配置
source ~/.bashrc
# 或
source ~/.zshrc

# 3. 重新安装
 curl -fsSL https://molt.bot/install.sh | bash
```

#### Q: Gateway 启动失败

**排查步骤**：
```bash
# 1. 检查 JSON 语法
cat ~/.openclaw/openclaw.json | jq

# 2. 查看错误日志
cat ~/.openclaw/logs/gateway.err.log

# 3. 使用备份恢复
cp ~/.openclaw/openclaw.json.backup ~/.openclaw/openclaw.json
openclaw gateway restart
```

### 7.2 API 配置问题

#### Q: MissingEnvVarError - 缺少环境变量

**错误**：`MissingEnvVarError: Missing env var "MINIMAX_CN_API_KEY"`

**解决**：
```bash
# 检查是否设置
echo $MINIMAX_CN_API_KEY

# 添加到 shell 配置
echo 'export MINIMAX_CN_API_KEY="your_key"' >> ~/.bashrc
source ~/.bashrc
```

#### Q: HTTP 401 认证失败

**原因**：API Key 无效或过期

**解决**：
1. 验证 API Key 格式
2. 检查 API Key 是否有访问权限
3. 在控制台重新生成

#### Q: 模型未显示在列表中

**排查**：
```bash
# 1. 检查 JSON 语法
cat ~/.openclaw/openclaw.json | jq

# 2. 确认 provider 名称与模型引用一致
# 3. 重启 Gateway
openclaw gateway restart
```

### 7.3 本地模型问题

#### Q: `Connection error`

**原因**：
- `main/agent/models.json` 中的端口错误
- llama.cpp 未运行
- 配置未生效

**解决**：
```bash
# 检查 llama.cpp 是否运行
curl http://127.0.0.1:8080/v1/models

# 检查 main agent 配置
cat ~/.openclaw/agents/main/agent/models.json
```

#### Q: `Model context window too small (8192). Minimum is 16000.`

**解决**：将 llama.cpp 启动参数 `-c` 和模型配置中的 `contextWindow` 都提升到 `16384` 或更高。

#### Q: `LLM request timed out`

**原因**：
- 本地模型响应慢
- 配置残留过重
- 端口错误

**解决**：
- 增加 `timeoutSeconds` 配置
- 清理残留插件配置
- 验证端口配置一致性

#### Q: 内容返回为空（`reasoning_content` 问题）

**原因**：Qwen 模型默认将输出放入 `reasoning_content`

**解决**：在 llama.cpp 启动参数中添加：
```bash
--reasoning-format none \
--chat-template-kwargs '{"enable_thinking":false}'
```

### 7.4 局域网访问问题

#### Q: `disconnected (1008)` 错误

**错误**：`control ui requires HTTPS or localhost`

**解决**：确保配置中包含：
```json
{
  "gateway": {
    "controlUi": {
      "allowInsecureAuth": true
    }
  }
}
```

#### Q: 局域网无法访问

**排查**：
```bash
# 1. 检查防火墙
sudo ufw status
sudo ufw allow 18789/tcp

# 2. 确认 bind 配置
grep '"bind"' ~/.openclaw/openclaw.json
# 应输出: "bind": "lan"
```

### 7.5 其他问题

#### Q: `plugin not found: openclaw-web-search`

**解决**：清理配置文件中的插件条目，或显式配置 `plugins.allow`。

#### Q: 如何彻底重启 OpenClaw

```bash
pkill -f openclaw
openclaw
```

---

## 八、附录

### 8.1 配置文件参数说明

| 参数 | 说明 | 示例值 |
|------|------|--------|
| `models.mode` | provider 合并模式 | `merge` / `replace` |
| `providers.*.baseUrl` | API 基础 URL | `https://api.minimaxi.com/anthropic` |
| `providers.*.apiKey` | API 密钥 | `sk-xxx` / `${ENV_VAR}` |
| `providers.*.api` | API 模式 | `anthropic-messages` / `openai-completions` |
| `models[].id` | 模型唯一标识 | `MiniMax-M2.1` |
| `models[].contextWindow` | 上下文窗口大小 | `204800` |
| `models[].maxTokens` | 最大输出 tokens | `131072` |
| `gateway.bind` | 网络绑定模式 | `loopback` / `lan` |
| `gateway.port` | 服务端口 | `18789` |
| `controlUi.allowInsecureAuth` | 允许 HTTP 认证 | `true` / `false` |

### 8.2 支持的模型列表

| 服务商 | 模型 ID | 上下文窗口 | 特性 |
|--------|---------|-----------|------|
| MiniMax | MiniMax-M2.1 | 200k | 支持推理 |
| 智谱 | glm-4.7 | 200k | 支持推理 |
| 阿里云百炼 | qwen3-max-2026-01-23 | 262k | OpenAI 兼容 |
| 阿里云百炼 | qwen-plus | 1029k | OpenAI 兼容 |

### 8.3 API 直接测试命令

**MiniMax**：
```bash
curl -X POST "https://api.minimaxi.com/anthropic/v1/messages" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "MiniMax-M2.1", "max_tokens": 1024, "messages": [{"role": "user", "content": "Hello"}]}'
```

**智谱**：
```bash
curl -X POST "https://open.bigmodel.cn/api/anthropic/v1/messages" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "glm-4.7", "messages": [{"role": "user", "content": "Hello"}]}'
```

### 8.4 相关链接

- [OpenClaw 官方文档](https://docs.clawd.bot)
- [MiniMax 官网](https://www.minimaxi.com)
- [智谱 AI 官网](https://bigmodel.cn)
- [阿里云百炼](https://dashscope.aliyuncs.com)

---

## 版本信息

- 文档版本: 2.0
- 最后更新: 2026-03-08
- 支持的 OpenClaw 版本: 2026.1.24-3+
- 支持的平台: Linux, macOS

---

**安全提醒**：请妥善保管您的 API Key 和 Token，不要将其提交到版本控制系统或公开分享。
