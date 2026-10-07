# workbuddy-bridge

> 让腾讯 Marvis 把本机 WorkBuddy 当「子 Agent」调用——MCP 桥接层。
>
> Let Tencent Marvis drive WorkBuddy as a sub-agent through an MCP bridge.

基于 [LinHaiJ/workbuddy-mcp](https://github.com/LinHaiJ/workbuddy-mcp)（MIT）封装：Marvis 作为 MCP 客户端，通过 `workbuddy-mcp` 的 `run_workbuddy_task` 工具委托本机 WorkBuddy CLI（`codebuddy`）执行真实任务。

## 架构

```
Marvis (MCP 客户端)
      │ 调用 MCP 工具: run_workbuddy_task(prompt, cwd, model, ...)
      ▼
workbuddy-mcp   (stdio MCP Server, 本机 node 进程)
      │ shell 调用:
      ▼
codebuddy -p --dangerously-skip-permissions   # prompt 经 stdin 传入
      │
      ▼
WorkBuddy   (执行真实任务，返回结果文本)
```

## 前置条件

| 依赖 | 要求 | 检查 |
|---|---|---|
| Node.js | >= 18 | `node -v` |
| WorkBuddy CLI (`codebuddy`) | 已安装并登录一次 | `codebuddy --version` |
| workbuddy-mcp | npm 全局安装 | `npm ls -g --depth=0` |

```bash
# WorkBuddy CLI（首次运行会打开登录流程）
npm install -g @tencent-ai/codebuddy-code
codebuddy -p "hello" --dangerously-skip-permissions

# 桥接包
npm install -g workbuddy-mcp
```

## 在 Marvis 中接入

Marvis 的 MCP 管理入口添加以下配置（stdio 类型，指向本机已安装的 server.js）：

```json
{
  "mcpServers": {
    "workbuddy": {
      "command": "node",
      "args": ["C:/Users/<你的用户名>/AppData/Roaming/npm/node_modules/workbuddy-mcp/server.js"]
    }
  }
}
```

保存后信任并连接，Marvis 的可用工具中会出现 `run_workbuddy_task`（1 个工具）。

## 使用方法

直接对 Marvis 说「让 workbuddy 做 XX」即可，Marvis 会自动调用 `run_workbuddy_task`。

工具参数：

| 参数 | 必填 | 说明 |
|---|---|---|
| `prompt` | 是 | 交给 WorkBuddy 的任务描述（经 stdin 传递，支持多行） |
| `cwd` | 否 | codebuddy 工作目录（读/写文件的位置），默认 WB_CWD 或当前目录 |
| `model` | 否 | 模型别名，如 `hy3` / `hy3-x` / `deepseek-v4-flash` / `glm-5.3` / `kimi-k3-1` / `auto` |
| `fallbackModel` | 否 | 主模型过载/限流时自动回退的模型（仅 `--print` 生效） |
| `json` | 否 | true 时追加 `--output-format json` |
| `timeoutMs` | 否 | 本次调用超时（默认 900000ms，长调研建议 1800000） |

示例：

- 「让 workbuddy 读取 ./reports 下的 CSV，生成一份中文月度总结」
- 「让 workbuddy 在仓库根目录跑测试，把失败日志整理成 Markdown」

## 环境变量配置

在 MCP 配置的 `environment` 块中设置：

| 变量 | 默认 | 说明 |
|---|---|---|
| `WB_COMMAND` | `codebuddy` | CLI 命令；找不到时填绝对路径 |
| `WB_SKIP_PERMISSIONS` | `true` | true 跳过交互授权自动执行；false 保留人工确认 |
| `WB_TIMEOUT` | `900000` | 默认任务超时（ms） |
| `WB_CWD` | 未设置 | 默认工作目录 |
| `WB_MODEL` | 未设置 | 默认模型 |
| `WB_FALLBACK_MODEL` | 未设置 | 限流自动回退模型 |

## 安全提示

默认 `WB_SKIP_PERMISSIONS=true`，即 WorkBuddy 跳过交互授权、自动执行 Agent 请求的任何操作。个人可信自动化场景可用；如需保留人工确认，请设为 `false`。

## 模型切换

免费模型被限流时建议配置回退而非硬切换：

```jsonc
{
  "WB_MODEL": "hy3",
  "WB_FALLBACK_MODEL": "deepseek-v4-flash"
}
```

可用模型（`codebuddy --help` 查看）：`auto`、`hy3`、`hy3-x`、`glm-5.3`、`glm-5.2`、`glm-5.1`、`glm-5v-turbo`、`minimax-m3`、`kimi-k3-1`、`kimi-k2.7`、`kimi-k2.6`、`deepseek-v4-flash`、`deepseek-v4-pro`。

## 参考

- [LinHaiJ/workbuddy-mcp](https://github.com/LinHaiJ/workbuddy-mcp) —— 桥接核心（MIT）
- [WorkBuddy 官方文档](https://www.workbuddy.cn/docs)

## License

MIT
