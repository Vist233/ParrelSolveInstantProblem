# 开源项目与基础能力调研

调研日期：2026-10-23。核验方式：读取官方仓库、项目文档和论文页面；未在本项目部署、运行或压测。并发配置、角色数量、累计任务数量与真实同时运行的 Agent 数量分别记录，不能相互替代。

## 最接近的开源项目

| 项目 | 与本项目相关的能力 | 适用边界与待核验项 |
| --- | --- | --- |
| [unsorry](https://github.com/agenticsnz/unsorry) | 分布式 Lean 4 求证、任务认领、验证队列、共享证明库与形式化声明绑定 | 数学闭环最接近；一条命令启动求证节点不等于一次启动百个节点，未核实 Air 百并发实测 |
| [unsorry-guild](https://github.com/agenticsnz/unsorry-guild) | 独立只读站点，显示目标、贡献和证明图，以 Git 为事实源 | 展示层可参考；不是完整可交互的研究调度工作台 |
| [CORAL](https://github.com/Human-Agent-Society/CORAL) | 多领域持续探索、独立工作区、共享经验、会话管理与评分 | 管理框架接近长期方向；数学验收与严格原生隔离不能直接假定具备，百并发实测待核验 |
| [OpenGauss](https://github.com/math-inc/OpenGauss) | Lean 工作流、子 Agent 会话、swarm 跟踪与恢复 | 数学工作台参考；未找到百并发保证 |
| [OpenProver](https://github.com/kripner/OpenProver) | 规划与并行 Worker、时间/token 预算和恢复 | 可配置并行不代表百并发已经验证 |
| [ShinkaEvolve](https://github.com/SakanaAI/ShinkaEvolve) | 候选程序进化、异步生成与并行评估、本地或 Slurm 执行 | 适合有评价器的科学计算；候选评估并发不同于持续 Agent 协作 |
| [OpenEvolve](https://github.com/algorithmicsuperintelligence/openevolve) | 程序生成、评价器筛选、多岛进化 | 可参考搜索循环；需要领域求解器与独立有效性检查 |
| [DiscoverPhysics](https://github.com/SampsonML/DiscoverPhysics) | 在模拟世界设计实验、推断物理规律，批量运行模型×世界×种子 | 主要是研究基准；批量运行不能直接当作百 Agent 调度能力 |
| [ColliderAgent](https://github.com/HET-AGI/ColliderAgent) | 高能物理模型与事件生成、模拟、分析和绘图 | 领域范围明确，依赖 Magnus 后端与领域软件 |
| [dsh-swarm-orchestrator](https://github.com/linkbag/dsh-swarm-orchestrator) | DAG 调度、审查重试、暂停恢复和看板；文档默认单次50并发、全局100 | 属于容量配置，不是科研百并发实测；数学/物理专用验证仍需接入 |

尚未核实到一个同时满足本项目全部目标、且有 MacBook Air 百级真实 Agent 公开实测的完整项目。优先参考模块和协议，避免把多个完整系统直接拼接。

## 题库与研究任务来源

| 来源 | 计划用途 | 注意事项 |
| --- | --- | --- |
| [Formal Conjectures](https://github.com/google-deepmind/formal-conjectures) | 开放研究题库主候选，已有 Lean/mathlib 声明、来源和稳定 benchmark 标签，包含 Erdős 问题 | 固定版本并核对解题状态；声明忠实程度、新颖性和证明正确性分别检查 |
| [miniF2F Lean 4](https://github.com/yangky11/miniF2F-lean4) | 少量已知可解题的工程校准 | 公开题可能受训练记忆影响；不作为新数学发现 |
| [PutnamBench](https://github.com/trishullab/PutnamBench) | 检查更难竞赛问题的求证能力 | 固定工具链与题目版本，不能与开放猜想直接混算 |

题目入库拟保留来源 URL、上游 commit、原文、形式化声明、状态核查日期、版本和许可。来源仓库许可不能自动覆盖其所有第三方材料。

## 模型与主管

- [OpenCode Go 官方文档](https://opencode.ai/v2/docs/console/go/)列有 DeepSeek V4.1 Flash。支持符合要求的编码 Agent 客户端，要求诚实的 User-Agent 和稳定会话标识。未找到百并发承诺；额度、限流、缓存命中与求证质量均需真实验证。
- [Codex Goals 官方说明](https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex)描述持续目标、完成条件和生命周期。可作为主管持续工作的机制；不保证不偏航，也不代替权限、调度或独立验证。自有后端如何驱动该机制需在接入时核实接口。
- KV 缓存属于模型服务计算优化，不是任务状态存储；跨会话或跨机器续接应依赖可导出的消息、摘要和检查点。

## 工具、隔离与 GUI

- [lean-lsp-mcp](https://github.com/oOo0oOo/lean-lsp-mcp)提供诊断、当前目标与搜索等接口，可参考工具划分；其现成实现依赖 Python。
- [Lean 官方 IO 说明](https://lean-lang.org/functional_programming_in_lean/Hello___-World___/Additional-Conveniences/)确认 #eval 能执行 IO，故不给 Python 不能代替隔离 Lean 进程。
- [Anthropic sandbox-runtime](https://github.com/anthropics/sandbox-runtime)可对本机进程施加文件与网络限制。macOS 使用 sandbox-exec；默认广泛读取，需要显式收紧并做逃逸负向验证。参考 [Apple 仓库中的弃用讨论](https://github.com/apple/containerization/issues/737)，隔离后端应可替换。
- [React Flow 官方思维导图教程](https://reactflow.dev/learn/tutorials/mind-map-app-with-react-flow)可参考节点编辑、依赖连接和交互；项目拟用其构建研究图，尚未实现。

## 下一步要取得的实测证据

1. 单题真实证明与独立验收，包含未通过、超时和错误声明的情况。
2. 任务间文件隔离、凭据不可读、网络拒绝和子进程限制。
3. 主管分工、Worker 恢复、审查摘要与图上证据的一致性。
4. 模型请求并发与 Lean 执行并发分别测量，记录内存、交换空间、限流和完成率。
5. 导出到新目录或机器后，重建依赖、复验证明并继续任务。
