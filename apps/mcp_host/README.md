# pervault-mcp

把 Pervault 本地记忆 daemon 暴露给 MCP 客户端（Claude Desktop / Cursor 等）的薄桥接。

## 架构约束

本进程**不开数据库、不跑后台**（方案 D9）。一切经 loopback 转发给常驻 daemon：

```
Claude Desktop ──stdio──> server.py ──127.0.0.1:8000 + X-Pervault-Token──> Pervault daemon
```

token 自动从 `~/.pervault/core_token` 读取（daemon 首次收到请求时生成，0600）。

## 使用

> 注意：本 MCP server 只提供「记忆工具」，不提供模型本身。Claude Desktop / Codex 等客户端需要先配置好自己的账号或 API；OpenClaw / Hermes 这类本地模型运行方式，如果本地模型已经可用，客户端侧不需要额外云 API。后端 `.env` 里的 LLM / embedding key 只影响记忆富化、向量检索、画像和图谱等能力；基础写入和非向量检索可以先不配。

1. 先启动 daemon：双击项目根目录 `RUN.command`。
2. Claude Desktop 配置（`~/Library/Application Support/Claude/claude_desktop_config.json`）：

```json
"mcpServers": {
  "pervault-memory": {
    "command": "<uv 绝对路径，可用 `which uv` 查看，如 ~/.local/bin/uv>",
    "args": [
      "--directory", "<项目绝对路径>/apps/mcp_host",
      "run", "python", "server.py"
    ]
  }
}
```

3. 重启 Claude Desktop，对话里即可使用 8 个工具：
   `memory_store` / `memory_search` / `memory_graph` / `memory_update` / `persona_get` / `reflections_list` / `memory_why` / `memory_stats`。

## 测试

- 单测（工具映射，无需 daemon）：`uv run pytest`
- E2E（需 daemon 运行中）：`uv run python tests/e2e_roundtrip.py`

## 环境变量

- `PERVAULT_DAEMON_URL`：daemon 地址，默认 `http://127.0.0.1:8000`
- `PERVAULT_CORE_TOKEN`：直接指定 token（默认读 `~/.pervault/core_token`）
- `PERVAULT_HOME`：覆盖 `~/.pervault` 位置（测试用）
