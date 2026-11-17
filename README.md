# Hologrow for DeepSeek Harness

[中文](#安装) | [English](#english)

DeepSeek Harness 插件。安装后，当前 profile 会连接 Hologrow 远程 MCP
`https://mcp.hologrow.ai/mcp`，模型通过 `mcp__hologrow__*` 调用该工作区已开通的只读数据工具。

npm 上的无作用域包名 `dsh-plugin` 属于社区插件市场，本包的名称是 `@hologrow/dsh-plugin`。

## 安装

需要 DeepSeek Harness `>=0.2.0` 和 Node.js `>=22.19.0`。

```sh
dsh plugin --profile web add github:hologrow/dsh-plugin
```

使用其他 profile 时，把 `web` 换成该 profile 的名字。这个包没有构建脚本，不需要 `allowBuilds`。

在启动 DSH 之前设置 API Key。Key 在
[Hologrow AI 平台](https://connector.hologrow.ai/ai-platforms) 创建，形如 `sk-hol-...`。

```sh
export HOLOGROW_API_KEY=sk-hol-...
dsh web
```

Key 只从进程环境读出，并作为 `Authorization: Bearer` 发给上面的 MCP 地址。未设置 Key 时 profile 仍能启动，但 Hologrow 工具不会注册；补上 Key 后重启 DSH。

## 模型看到的工具

DSH 把服务端工具名加上 `mcp__hologrow__` 前缀。已绑定工作区始终提供这五个只读工具：

| 服务端名称 | DSH 名称 | 作用 |
|---|---|---|
| `list_platforms` | `mcp__hologrow__list_platforms` | 查看已连接平台和 schema |
| `list_tables` | `mcp__hologrow__list_tables` | 按平台批量列出表，并带平台查询引导 |
| `get_data_dictionary` | `mcp__hologrow__get_data_dictionary` | 读取所选表的字段、指标和查询约束 |
| `get_freshness` | `mcp__hologrow__get_freshness` | 诊断同步覆盖和数据陈旧；不要每次查询都调用 |
| `query_db` | `mcp__hologrow__query_db` | 对已发现的表执行一条只读 SQL |

允许名单中的工作区还可能看到 `query_db_analytics`，以及 `query_db_table_view` 和 `query_db_table_view_sheet`。已开通 Signal 插件的工作区还会看到 `hologrow_` 前缀的插件工具。工具列表以该 Key 实际返回的为准。

`@deepseek-ai/dsh-mcp-client` 只桥接工具调用，不加载 MCP App 或资源。`query_db_table_view` 的交互式 Workbook 界面不会在 DSH 里打开；没有该界面时，沿用 `query_db` 已经返回的结果。

数据范围是该工作区已连接的 Amazon Seller/Vendor/Ads/DSP、Shopify、Shoplazza、TikTok Shop、Meta Ads、Google Ads、Google Analytics/GA4、Search Console 和领星 ERP。查询顺序是 `list_platforms`、`list_tables`、`get_data_dictionary`，然后 `query_db`。

## 改用预发布地址

默认地址是生产 MCP。要改用 staging，在 profile 自己的 `cordis.patch.yml` 里按相同 `id` 覆盖整行，并写全配置：

```yaml
- id: hologrow
  name: '@deepseek-ai/dsh-mcp-client'
  config:
    serverName: hologrow
    transport: streamable-http
    url: https://staging-mcp.hologrow.ai/mcp
    headers:
      Authorization: !!js 'process.env.HOLOGROW_API_KEY ? `Bearer ${process.env.HOLOGROW_API_KEY}` : ""'
    toolCallTimeoutMs: 200000
    failOnStartupError: false
```

## English

This plugin connects a DeepSeek Harness profile to the Hologrow MCP at
`https://mcp.hologrow.ai/mcp`.

```sh
dsh plugin --profile web add github:hologrow/dsh-plugin
export HOLOGROW_API_KEY=sk-hol-...
dsh web
```

Create the key at <https://connector.hologrow.ai/ai-platforms>. The key is read from the environment and sent only as a Bearer token to that endpoint. Without it, the profile still starts and the Hologrow tools stay unregistered.

A bound workspace always exposes `list_platforms`, `list_tables`, `get_data_dictionary`, `get_freshness`, and `query_db`. DSH registers them as `mcp__hologrow__<name>`. Allowlisted workspaces may also expose `query_db_analytics`, `query_db_table_view`, and `query_db_table_view_sheet`. Signal plugins appear under a `hologrow_` prefix only when that workspace has them. DSH does not render the workbook MCP App; use the `query_db` result when the interactive view is unavailable.
