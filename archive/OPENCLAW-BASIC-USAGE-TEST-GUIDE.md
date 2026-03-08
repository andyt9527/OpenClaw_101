# OpenClaw 基础使用测试指南

本指南帮助用户快速验证 OpenClaw 的基础可用性，包括安装状态、模型列表、对话响应与网关状态等。

## 目录

- [一、前置条件检查](#一前置条件检查)
- [二、基础可用性测试](#二基础可用性测试)
- [三、对话与模型测试](#三对话与模型测试)
- [四、常见问题](#四常见问题)

---

## 一、前置条件检查

### 1.1 检查 OpenClaw 是否已安装

```bash
which openclaw
openclaw --version
```

### 1.2 检查配置文件位置

OpenClaw 的主配置文件位于：

```bash
~/.openclaw/openclaw.json
```

查看当前配置：

```bash
cat ~/.openclaw/openclaw.json
```

### 1.3 检查 Node.js 版本

```bash
node --version
```

**要求**: Node.js ≥ 22

---

## 二、基础可用性测试

### 2.1 检查 Gateway 状态

```bash
openclaw gateway status
```

确认 Gateway 正常运行（状态为 running 或 active）。

### 2.2 列出可用模型

```bash
openclaw models list
```

**预期**: 输出中包含你已配置的模型，例如：

```
Model                                      Input      Ctx      Local Auth  Tags
minimax-cn/MiniMax-M2.1                    text       200k     no    yes   configured
```

---

## 三、对话与模型测试

### 3.1 基础对话测试

```bash
openclaw agent --message "Hello, are you working?" --session-id main
```

**预期**: 返回正常回复，说明模型可用。

### 3.2 指定模型测试

```bash
openclaw agent --message "Summarize this in one sentence." \
  --model minimax-cn/MiniMax-M2.1 \
  --session-id main
```

如果使用其他 Provider，请替换为对应模型：

```bash
openclaw agent --message "你好，请简短自我介绍" \
  --model zhipu-cn/glm-4.7 \
  --session-id main
```

### 3.3 切换模型测试

在对话中使用 `/model` 切换：

```bash
/model minimax-cn/MiniMax-M2.1
```

**预期**: 返回模型切换成功的提示。

---

## 四、常见问题

### Q1: `openclaw` 命令找不到

**解决方案**:

1. 确认安装是否完成：
   ```bash
   which openclaw
   ```

2. 重新加载 shell 配置：
   ```bash
   source ~/.bashrc
   # 或
   source ~/.zshrc
   ```

3. 如仍不可用，重新运行安装：
   ```bash
   curl -fsSL https://molt.bot/install.sh | bash
   ```

### Q2: 模型列表为空或缺失

**解决方案**:

1. 检查配置文件是否为有效 JSON：
   ```bash
   cat ~/.openclaw/openclaw.json | jq
   ```

2. 确认 Provider 名称与模型引用一致
3. 确认 API Key 已设置且有效
4. 重启 Gateway：
   ```bash
   openclaw gateway restart
   ```

### Q3: 对话无响应或报错

**解决方案**:

1. 查看 Gateway 日志：
   ```bash
   tail -f ~/.openclaw/logs/gateway.log
   cat ~/.openclaw/logs/gateway.err.log
   ```

2. 检查网络与 API Key 权限
3. 重新启动 Gateway

---

## 版本信息

- 文档版本: 1.0
- 创建日期: 2026-02-22
