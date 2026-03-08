# OpenClaw + llama.cpp 本地配置教程

本文整理了一套经过实际排障验证的 OpenClaw + llama.cpp 本地可用方案，并总结本次 debug 过程中遇到的关键问题与最终修复方式。

## 一、最终结论

这次问题不是单一原因，而是多个问题叠加：

1. **Qwen 在 llama.cpp 中默认可能把输出放到 `reasoning_content`**，导致 OpenClaw 读取不到正常 `content`。
2. **OpenClaw embedded agent 要求最小上下文窗口至少 16000**，`8192` 会被直接拦截。
3. **`~/.openclaw/agents/main/agent/models.json` 会覆盖全局 `openclaw.json` 的 provider 配置**。
4. 最后真正导致 `main` agent 仍然失败的关键点，是 `main` agent 私有 `models.json` 中的 `llamacpp.baseUrl` 还指向旧端口 `8000`，而实际运行的 `llama-server` 在 `8080`。

## 二、推荐的 llama.cpp 启动命令

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

### 参数说明

- `--port 8080`：与 OpenClaw 配置保持一致。
- `-c 16384`：满足 OpenClaw embedded agent 最低上下文要求。
- `--jinja`：启用较新的模板处理方式。
- `--reasoning-format none`：避免输出进入 `reasoning_content`。
- `--chat-template-kwargs '{"enable_thinking":false}'`：关闭 thinking 模式，确保回复进入标准 `content` 字段。

## 三、先验证 llama.cpp 是否正常

### 1）检查模型列表

```bash
curl http://127.0.0.1:8080/v1/models
```

### 2）检查聊天接口

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

期望返回：

```json
{"choices":[{"message":{"content":"OK"}}]}
```

如果 `content` 为空，而内容跑到 `reasoning_content`，说明 llama.cpp 启动参数还不对。

## 四、全局配置模板

全局配置文件通常是：

```text
~/.openclaw/openclaw.json
```

可以参考 `openclaw_openclaw_json_global.template.json`。

关键点：

- `providers.llamacpp.baseUrl` 必须指向 `http://127.0.0.1:8080/v1`
- `agents.defaults.model.primary` 要设成 `llamacpp/qwen3.5-4B-Q4_0.gguf`
- `timeoutSeconds` 建议保留较大的默认值，例如 `600`
- `plugins.allow` 建议显式配置，避免自动加载残留插件

## 五、main agent 私有配置比全局更关键

这次排障中最容易忽略的一点是：

```text
~/.openclaw/agents/main/agent/models.json
```

这个文件会覆盖 `main` agent 使用的 provider 配置。

也就是说：

- 即使全局 `openclaw.json` 已经改成 `8080`
- 只要 `main/agent/models.json` 还写着 `8000`
- `openclaw agent --agent main --local` 实际还是会连错地址

### 本次真实问题

错误配置示例：

```json
"llamacpp": {
  "baseUrl": "http://127.0.0.1:8000/v1"
}
```

正确配置示例：

```json
"llamacpp": {
  "baseUrl": "http://127.0.0.1:8080/v1"
}
```

## 六、推荐的 main agent 配置方式

### 方案 A：保留原有多 provider，但修正 llama.cpp

可参考：

- `openclaw_llamacpp_main_agent_models.template.json`

适合你仍想保留 MiniMax / Ollama / llama.cpp 的多 provider 结构。

### 方案 B：使用精简版本地配置

可参考：

- `openclaw_main_agent_models.clean.json`

这份配置只保留本地 `llamacpp`，更适合排障完成后的稳定使用。

## 七、替换配置后如何生效

修改完配置后，建议彻底重启 OpenClaw，而不是只依赖热重载：

```bash
pkill -f openclaw
openclaw
```

然后重新验证：

```bash
openclaw agent --agent main --message "Reply with exactly: OK" --local
```

成功时通常会看到类似输出：

```text
OK
```

## 八、常见报错与对应原因

### 1）`Connection error`
可能原因：

- `main/agent/models.json` 仍然连错端口
- provider 配置未真正生效
- 旧插件/旧路由干扰

### 2）`Model context window too small (8192 tokens). Minimum is 16000.`
原因：

- `contextWindow` 太小
- `llama-server -c` 太小

修复：

- 将两边都提升到 `16384` 或更高

### 3）`LLM request timed out`
可能原因：

- 实际请求 prompt 比手工 curl 重很多
- 本地模型响应偏慢
- agent 配置残留过重
- 端口错误导致内部请求行为异常

### 4）`plugin not found: openclaw-web-search`
原因：

- 配置文件中有陈旧插件条目

修复：

- 清理 `plugins.entries.openclaw-web-search`
- 显式配置 `plugins.allow`

## 九、推荐的最终实践

1. 先确认 llama.cpp 单独可用。
2. 再确认全局 `openclaw.json` 配置正确。
3. 再重点检查 `~/.openclaw/agents/main/agent/models.json`。
4. 如果 `main` agent 历史太复杂，建议改用精简版 `models.json`。
5. 配置修改后务必彻底重启 OpenClaw。

## 十、建议额外注意

- 如果曾经贴出过真实 token / access key，建议立即轮换。
- 如果 `main` agent 中仍有旧的 MiniMax / Ollama / Anthropic 配置残留，后续建议逐步清理。
- 本地小模型可以跑通 OpenClaw，但 embedded agent 对 prompt 规模、上下文和响应时间更敏感，建议优先使用更干净的 agent 配置。
