# OpenClaw 国内 API 配置指南

本仓库提供 OpenClaw 配置国内版 LLM API 的完整指南，帮助用户快速上手 OpenClaw，并配置国内 API 服务商。

## 支持的 API 服务商

| 服务商 | 官网 | 特点 |
|--------|------|------|
| **MiniMax** | https://www.minimaxi.com | 高质量中文模型 M2.1 |
| **智谱 AI (GLM)** | https://bigmodel.cn | GLM-4.7 编程与对话 |
| **阿里云百炼** | https://dashscope.aliyuncs.com | 通义千问系列，OpenAI 兼容 |

## 快速开始

```bash
# 1. 安装 OpenClaw
curl -fsSL https://molt.bot/install.sh | bash

# 2. 备份配置
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# 3. 设置 API Key（以 MiniMax 为例）
export MINIMAX_CN_API_KEY="your_api_key_here"

# 4. 编辑配置文件，添加 provider
nano ~/.openclaw/openclaw.json

# 5. 重启 Gateway
openclaw gateway restart

# 6. 验证
openclaw models list
openclaw agent --message "Hello" --session-id main
```

详细配置示例请参考 [完整指南](./OPENCLAW-COMPLETE-GUIDE.md)。

## 文档索引

| 文档 | 内容 |
|------|------|
| [📖 完整指南](./OPENCLAW-COMPLETE-GUIDE.md) | **一站式参考文档**，涵盖所有配置场景 |

### 完整指南章节

1. **快速开始** - 5 分钟上手
2. **前置条件** - 环境检查与备份
3. **国内 API 配置** - MiniMax、智谱、阿里云百炼详细配置
4. **本地模型配置** - llama.cpp 集成与排障
5. **局域网访问** - Web UI 多设备访问
6. **基础使用** - 常用命令与测试
7. **故障排查** - 常见问题解决方案
8. **附录** - 参数说明与 API 测试

## 系统要求

- OpenClaw ≥ 2026.1.24-3
- Node.js ≥ 22
- 国内 API Key（MiniMax / 智谱 / 阿里云百炼）

## 许可证

MIT License - 详见 [LICENSE](./LICENSE) 文件

## 安全提醒

- 🔐 请妥善保管 API Key 和 Token
- 🚫 不要将敏感信息提交到版本控制系统
- 📊 定期检查 API 使用情况以控制成本
- 🌐 局域网访问请注意网络安全
