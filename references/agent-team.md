# 紧箍咒 （四层架构）

紧箍咒 v3 支持四层 Agent Team 架构，严格对应如来佛祖 P10→P9→P8→P7 管理层级：

```
P10 (CTO)              → 定战略、造土壤、断事用人
  ├─ 战略输入
  ▼
P9 (Tech Lead)         → 懂战略、搭班子、做导演
  ├─ Task Prompt (六要素)
  ▼
P8 (独当一面)           → 既能自己干，也能带 P7
  ├─ 简单任务自己做 / 复杂任务拆解后委派
  ▼
P7 (Senior Engineer)   → 方案驱动，在 P8 指导下执行子任务
  ├─ 方案 + 代码 + 审查三问
  ▼
交付物
```

## 角色与 紧箍咒 行为

| 角色 | 识别方式 | 紧箍咒 行为 | 详细协议 |
|------|---------|---------|---------|
| **P10 CTO** | 用户说"用 P10 模式" | 定义战略方向，P9 间仲裁 | `references/methodology-xuanzang-pro.md` |
| **P9 Tech Lead** | 用户说"用 P9 模式" | 编写 Task Prompt，管理 P8 团队 | `references/methodology-guanyin-pro.md` |
| **P8 独当一面** | 默认角色 / P9 派发 | 执行任务 + 可派发 P7 子任务 | SKILL.md |
| **P7 Senior Engineer** | P8 派发 P7 模式 sub-agent | 方案先行，审查三问 | `references/methodology-juanlian-pro.md` |

## P8 失败汇报格式（L2+ 时发送给 P9）

```
[紧箍咒-REPORT]
from: <P8 标识>
task: <当前任务>
failure_count: <本任务失败次数>
failure_mode: <卡住原地打转|直接放弃推锅|完成但质量烂|没搜索就猜|被动等待|差不多就行|空口完成>
attempts: <已尝试方案列表>
excluded: <已排除的可能性>
next_hypothesis: <下一个假设>
```

P8 升级请求（L3+ 时向 P9 请求支援）：使用 `[紧箍咒-ESCALATION]` 格式（详见 `references/methodology-guanyin-pro.md`）。

## 并行执行协议

**P9 创建并行 P8 团队**（详见 `references/methodology-guanyin-pro.md` 阶段三）：

```
P9 拆解任务后
  ├─ 2-3 个无依赖 P8 任务 → 同轮并行 dispatch_task
  ├─ 超过 3 个 → 分批顺序派发
  └─ 有依赖链 → 按依赖序逐批派发
```

**P8 管理并行 P7 的决策树**：

```
P8 收到任务
  ├─ 单文件 / <30 行改动 → 自己做
  ├─ 跨 2-3 模块紧耦合 → 派发 1 个 P7，自己做另一部分
  └─ 跨 3+ 模块可解耦 → 并行派发多个 P7
       ├─ 划分文件域（P7 之间绝不编辑同一文件）
       ├─ 收齐 [P7-COMPLETION] 后整合验证
```

**P8→P7 轻量任务模板**（四要素）：

```
## [子任务标题]
### WHAT — 交付物
[精确的修改项 + 验收标准]
### WHERE — 文件域
[只动哪些文件，不动哪些]
### DONE — 完成标准
[验证命令 + 预期输出]
### DON'T — 禁区
[不要碰的文件/不要引入的依赖]

开工前先用 Read 工具读取 references/methodology-juanlian-pro.md（进入 P7 方案驱动模式）。
```

**重要**：subagent 不能用 `/pua`（skill 只在主会话加载）。必须通过 Read 工具读取 SKILL.md 或对应 protocol 文件。

**多角色派发标准**（Marvis 环境适配）：

| 场景 | 工具 | 隔离方式 |
|------|------|---------|
| 2-3 个 P8 并行实施 | 同轮 dispatch_task | 文件域隔离 |
| P8 派发 P7 子任务 | dispatch_task（P7 模式） | 文件域隔离 |
| 调研/搜索 | dispatch_task 给对应 Sub Agent | 上下文隔离 |

## 四层协作规则

1. **P10→P9**：下发战略输入模板，不写 Task Prompt
2. **P9→P8**：下发 Task Prompt 六要素（WHY/WHAT/WHERE/HOW MUCH/DONE/DON'T），只和 P8 对话
3. **P8→P7**：自行决定是否拆解子任务给 P7，负责验收后整合
4. **P7→P8**：完成后发 [P7-COMPLETION]（方案+代码+审查三问）
5. **P8→P9**：交付结果 + 验证输出；失败时发 [紧箍咒-REPORT]，L3+ 发 [紧箍咒-ESCALATION]
6. **P9→P10**：汇报 Sprint 进展 + 需要决断的事项
7. **紧箍咒 流向**：P10→P9→P8→P7，不越级
8. **P8 内部 P7 文件域**：由 P8 负责划分；多个 P8 的文件域由 P9 负责划分
9. **任务不重置**：重新分配时附带 `前任已失败 N 次，压力等级 LX，已排除: [...]`
10. **验收≠释放**：P8 交付通过后，P9 必须显式发 `[TEARDOWN]` 或 `[REASSIGN]`，不允许静默挂起
11. **派发生命周期闭合**：每个 dispatch_task 派发的子 agent 完成后由主 agent 验收回收
12. **子 agent 不递归派发**：P8 被派发后可 dispatch_task 给 P7，但不能再往下分（防无限嵌套）

> 生命周期释放、清理、孤儿回收的完整协议见 `references/teardown-protocol.md`。
