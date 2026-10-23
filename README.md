# ParrelSolveInstantProblem

面向数学、编程、物理与基因组分析的多 Agent 研究工作台，先完成 Lean 4 数学证明闭环。

**当前状态：规划与调研阶段。仓库尚无可运行产品，未验证百级真实 Agent 并发。**

## 目标

- 以研究方向和任务依赖图管理约 100 个独立 Agent 会话。
- 用受控工具与系统沙盒隔离任务，独立验收证明和产物。
- 持续汇总可验证进展，并在图上显示任务状态、证据和持续时间。
- 导出研究成果与续接状态，在其他机器重建环境后继续执行。

第一阶段题库已选定为 [The Last Math Competition](https://github.com/The-Last-Math-Competition/The-Last-Math-Competition)，尚未导入或运行求解。

## 文档

研究、调研与设计统一放在 [docs/](docs/README.md)。

- [整体规划](docs/plan.md)：证明过程、最小工具、Mac 原生隔离、监督与进度图、迁移和首个核心边界。
- [同类项目与资料调研](docs/research.md)：开源项目对比、题库来源、模型接入和待验证项。

首个核心拟实现：导入一题 → 分配方向 → 受限 Worker 求证 → 沙盒 Lean 检查 → 独立验收 → 图节点更新 → 导出复验。
