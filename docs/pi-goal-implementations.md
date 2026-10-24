# Pi Goal现有实现与本项目适配

日期：2026-10-24。状态：已读发布者文档及相关源码；未安装、启动或测试插件。Pi有Goal扩展能力，生态中有多个同名插件，不存在仅凭“pi-goal”名字就能确定的统一实现。

当前决定已变更：用户同日改选Codex，不再推进本文插件集成。本文保留源码学习记录；当前接法见[Codex与Claude Code](codex-integration.md)。

## Michaelliv/pi-goal：简单的同会话续跑

读取[入口源码](https://github.com/Michaelliv/pi-goal/blob/main/.pi/extensions/pi-goal/index.ts)、[状态逻辑](https://github.com/Michaelliv/pi-goal/blob/main/.pi/extensions/pi-goal/goal-state.ts)和[package.json](https://github.com/Michaelliv/pi-goal/blob/main/package.json)。仓库声明版本为0.1.7。

| 部分 | 源码行为 |
| --- | --- |
| 目标 | 保存目标文字、状态、用量和可选token预算 |
| 状态保存 | persist通过Pi自定义会话条目写入；恢复时读取当前分支最近状态 |
| 自动续做 | agent_end检查目标仍active且没有待处理消息，再通过微任务和follow-up队列发续做消息 |
| 每轮统计 | turn_start/turn_end记录时间和token；达到预算转为budget_limited |
| 完成 | update_goal只接受complete；实际处理直接变更状态，核对完成证据主要依赖模型遵守提示词 |
| 暂停与恢复 | /goal命令改变状态；源码明确reload时把活动目标暂停 |

原版没有blocked状态或三轮阻塞计数，也没有调用Lean的完成门槛。其依赖仍使用旧的@mariozechner包名，不能据此认定和本项目将选的Pi版本已经兼容。

入口的goal和续跑标记处于模块顶层。同一加载模块若被多个会话工厂共用，需要验证实例隔离或改造作用域；这是一项源码可见的集成检查点，本轮未证实实际会话串扰。

## fractaal/pi-extensions中的@fractaal/pi-goal-x

读取[README](https://github.com/fractaal/pi-extensions/blob/main/packages/goal-x/README.md)、[架构说明](https://github.com/fractaal/pi-extensions/blob/main/packages/goal-x/docs/architecture.md)、[入口源码](https://github.com/fractaal/pi-extensions/blob/main/packages/goal-x/extensions/goal.ts)、[完成审查实现](https://github.com/fractaal/pi-extensions/blob/main/packages/goal-x/extensions/goal-auditor.ts)及package.json。读取时仓库声明版本为0.29.0；本轮未确认该版本的npm发布状态。

| 部分 | 源码行为 |
| --- | --- |
| 状态 | 每个扩展实例维护状态，通过Pi自定义会话条目保存和按分支回放 |
| 续做 | 正常agent_end后登记ctx.onIdle回调，空闲时重新检查状态，再通过follow-up队列发自动续做消息 |
| 等待 | wait_goal记录下次继续时间，等待期间Goal保持active |
| 阻塞 | set_goal_blocked要求阻塞事实、证据、为何没有自主路径及解除条件；一次成功调用就暂停并发goal_blocked |
| 完成 | complete_goal调用完成审查函数；拒绝时继续active，通过后允许最终回复，再于agent_settled收录完成记录 |
| GUI接入 | 提供状态、提案和结构化生命周期事件；晚订阅者可请求重放当前状态 |
| 资源 | Pi负责底层重试和压缩；插件记录用量，项目全局额度仍需单独控制 |

它的自动续做实际使用ctx.onIdle，不是直接在agent_settled通知里启动任务，也不是使用agent_before_settle。普通终止、错误、中止与最终收尾分别处理。

入口允许注入runCompletionAuditor。数学场景可考虑把这一边界接到项目受信的Lean验收和证据整理；这是适配建议，不代表默认审查已经能验证Lean。默认审查会另外建立只读模型会话。

这个实现有两处直接影响本项目：

- 创建和修改Goal默认要求交互确认；无UI时直接抛错。主管分配大量已授权子任务的路径需要适配，不能让用户逐个确认100个Goal。
- 架构说明明确没有阻塞次数计数。当前set_goal_blocked立即暂停，不满足用户的连续三轮后才blocked规则，需要在状态转换处修改。

源码中没有与轻量pi-goal相同的tokenBudget限额状态，因此不能把用量记录描述成已有项目总预算控制。具体Pi最低版本声明和Node要求以安装时锁定的包为准；本轮没有做兼容运行。

## 另一个需要排除的默认行为

[Henry的@henryqw/pi-goal文档](https://pi.henry.wang/extensions/pi-goal/)明确列出仅交互模式自动继续、20次自动续做上限，以及两轮相同工具活动就停止等规则。这些默认行为与本项目后台SDK和三轮阻塞规则不同，不能仅因为名字相同就原样采用。

## 当前建议

本项目要复用的是单会话目标、续跑、暂停恢复和记录能力。轻量pi-goal便于理解最小循环；fractaal版本的状态事件和可注入完成审查值得作为GUI适配参考。两者均需接入Lean完成门槛和三轮阻塞转换，具体插件尚未选定。

首版无需叠加多个Goal插件、每轮新增裁判Agent或另造一套工作流平台。只保留一个续跑所有者，按[持续证明方案](continuous-proof.md)验证正常续做、完成、暂停和blocked边界。

## 源码核验记录

以下是本轮通过GitHub文件接口读取的文件blob SHA，不是仓库commit，也不表示已发布版本：

- Michaelliv入口：148a79dcab7b212d22e305119fbef9a0eb5b15ea；状态逻辑：5213087ee2f61acba320a56081b93e6aa84aaefd。
- fractaal入口：a7fd4ca08538e095334cf9d66fed4fc8307e3e1d；架构说明：843b291ed568df785f27293c9f3047e9b1881b65；完成审查：229722c914d5cf6a36063919e16124f32680952b。

没有安装插件、运行测试、调用真实模型、验证Lean完成或修改第三方源码。其他同名fork本轮未逐一审计。
