# 重构 Claude Code 方案对齐审计

本审计基于本地文件进行，不依赖线上资料：

- 方案文档：`D:\GitHub\dev\hellox\docs`
- Claude Code 源码：`C:\Users\hellowind\Downloads\cc-recovered-main`
- 当前实现：`D:\GitHub\dev\hello2cc`

## 结论

- `hello2cc` 不是 `hellox` Rust workspace，不能把 `hellox-cli`、`hellox-gateway`、`hellox-tui` 等 crate 的完整落地进度计入本仓库。
- 按完整 `Hellox Rust 重构` 产品目标看，本仓库只承担插件适配层，整体产品级完成度不应按本仓库计算。
- 按 `hello2cc` 可承担的 Claude Code 插件适配职责看，当前已覆盖主要高风险面：hook lifecycle、Agent / Team / Task 参数归一、session 状态提取、subagent 上下文注入、失败防抖。
- 当前插件层实施清单已完成；剩余边界主要是宿主架构限制，而不是本仓库遗漏的适配层功能。

## 对照范围

| 方案能力域 | Claude Code 源码锚点 | hello2cc 当前承接方式 | 状态 |
|---|---|---|---|
| Hooks / lifecycle | `src/utils/hooks.ts`、`src/tools/TaskCreateTool`、`src/query/stopHooks.ts` | `hooks/hooks.json` + `scripts/*` | 基本完成 |
| Agent / Team / Task 语义 | `src/tools/AgentTool`、`src/utils/swarm/*`、`src/tools/Task*` | `orchestrator` pre/post tool 归一与状态记忆 | 基本完成 |
| Session / transcript 状态 | `src/utils/sessionStorage.ts`、`src/utils/messages.ts` | transcript 解析、host-state snapshot、team continuity | 基本完成 |
| Gateway / provider adapter | `docs/HELLOX_ANTHROPIC_GATEWAY_SPEC.md` | 第三方接入层不在 hello2cc；仅保留行为提示 | 不由本仓库实现 |
| Full TUI / CLI / Rust crates | `docs/HELLOX_RUST_REFACTOR_PLAN.md` | 不属于 Claude Code 插件仓库职责 | 不由本仓库实现 |
| Hosted cloud / remote hub | `docs/HELLOX_LOCAL_FIRST_BOUNDARIES.md` | 明确不做托管云端服务 | 不做 |

## 已完成项

- Hook 覆盖：`SessionStart`、`UserPromptSubmit`、`SubagentStart`、`SubagentStop`、`TeammateIdle`、`TaskCreated`、`TaskCompleted`、`ConfigChange`、`PreToolUse`、`PostToolUse`、`PostToolUseFailure` 均已接线。
- Task lifecycle：`TaskCreated` 不再要求完成态证据；`TaskCompleted` 保留交付物与验收证据校验，当前回归测试覆盖 issue #15 形态。
- Agent / Team：普通 subagent 默认避免伪 `name` / `team_name`，仅在宿主状态证明真实 active team 后才补齐 team 语义。
- Model 注入：`inherit` 表示不注入；其他配置值按原文传给 Claude Code，不再由 hello2cc 做 `opus` / `sonnet` / `haiku` 白名单归一。
- 同类兼容排查：`subagent-context` 身份读取器已同步兼容 `agent_id`、`agentId` 与 `agent.id`，避免相邻脚本再次出现同类 snake_case / camelCase 漏洞。
- Local-first 边界：不实现 hosted auth、remote hub、web console、托管 session sync；如需远端能力，只保留用户自定义接入层之外的接口边界说明。

## 仍需对齐的缺口

- 在本仓库边界内，已无额外待补齐的插件层实施项；后续只需随 Claude Code 宿主接口变化做版本跟踪回归。

## 分组实施清单

### 第一组：审计与边界收敛（已完成）

- 新增本文件，记录文档 / 源码 / 当前仓库的真实对照关系。
- 明确 `hello2cc` 只承担 Claude Code 插件适配层，不承接 `hellox` Rust workspace 或云端服务。
- 将后续待做项限定为本仓库可测试、可发布、可回归的插件层缺口。
- 验收：`npm run validate` 通过。


## 不纳入本轮的事项

- 不在本仓库创建 `hellox` Rust workspace。
- 不实现 `hellox-gateway` 或 provider adapter。
- 不修改 Claude Code 第三方源码。
- 不开发 hosted auth、remote hub、web console、托管云端服务。
