# Codex + MiMo：Goal 与自动权限审核实测

日期：2026-10-25。用户授权一次有界接入测试；没有实现产品或运行真实数学证明。

## 结果

| 项目 | 实测结果 |
| --- | --- |
| 自定义模型接入 | Codex CLI 0.153.4，模型 mimo-v2.6-pro，MiMo 官方 Responses 接口，真实调用成功 |
| 自定义 Tool | app-server dynamicTools 注册一个合成阶段工具，宿主收到并执行三次调用 |
| 原生 Goal 自动继续 | 宿主只发送一次 turn/start；原生运行时启动三个不同 turn，各完成一个阶段，Goal 到达 complete |
| 自动权限审核 | 收到审核 started/completed 事件，但审核失败并拒绝命令；测试文件未创建 |

测试通过独立 CODEX_HOME 和专用工作目录运行，不改个人 Codex 配置。凭据只传给子进程环境，不写入配置或公开资料。测试已结束，app-server 退出码为 0，无留存后台任务。收尾时将测试 Goal 暂停；上表 complete 是收尾前实际读取的状态。

## Goal 如何验证

宿主工具每个 turn 最多接受一个阶段，返回已完成阶段数和回执。前两轮收到回执后，模型结束本轮；目标仍 active，Codex 原生调度器自动启动下一轮。第三轮取得最后一个回执后，模型通过原生生命周期工具完成 Goal。

实测计数：一次宿主 turn/start、三个原生 turn、三个不同 turn 的三次成功 Tool 调用。三轮最终均 completed。测试没有由宿主循环发送“继续”，也没有把同一轮的多次 Tool 调用当作自动续做。

这证明当前组合能自动跨轮执行并调用宿主 Tool。它没有证明模型能解决数学题，也不保证不偏航或长期稳定。Lean 验收、连续三轮业务阻塞规则、20分钟 fork 汇报和100会话容量均未测试。

## Auto-review 的具体阻塞

另开只读沙盒线程，配置 on-request 与 auto_review，仅授权创建一个无害的固定内容回执文件。模型申请标准权限升级；宿主没有手工批准。

审核后端实际返回：

> responses_feature_not_supported: text.format type 'json_schema' is not supported, only 'text' and 'json_object' are allowed.

Codex 因审核失败拒绝此次写入，没有创建回执。此结果说明当前 MiMo 直连组合不能完成这条原生自动审核路径，不能将 Goal 成功扩展成“Auto-review 全部可用”。日志中的 high/unknown 是失败事件字段，不能据此认定该无害写入被模型判断为高风险。

后续应单独验证支持该结构化输出要求的审核后端及其配置方式。未实现协议转换，未降低审核要求，也未绕过拒绝重试。使用受限自定义 Tool 时，文件范围仍由宿主强制检查；权限审核不替代该检查或 Lean 正确性验收。

## 接入要点与边界

- MiMo 官方 base_url 为 https://api.xiaomimimo.com/v1，模型 ID 为 mimo-v2.6-pro，Codex provider 使用 responses。
- app-server 通过 stdio 连接；实验 API 注册 dynamicTools；thread/goal/set 设置目标，turn/start 启动首轮，通知流观察后续执行，thread/goal/get 读取结果。
- Goal 阶段关闭 shell 和内置多 Agent，并使用单独的测试模型目录；审核阶段仅在另一测试线程开启命令工具。没有据此认定内置工具已被完全替换。
- 本轮只用合成任务。未发送用户项目文件或完整个人对话；未安装依赖、变更个人模型配置或运行并发压测。

## 来源

- [MiMo 官方 Responses API](https://mimo.mi.com/docs/en-US/api/chat/responses)
- [Codex app-server](https://learn.chatgpt.com/docs/app-server)
- [Codex Goals 示例](https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex)
- [Codex Auto-review](https://learn.chatgpt.com/docs/sandboxing/auto-review)

另外核对本机已安装版本生成的实验 JSON Schema，并以真实 RPC、宿主阶段回执、Goal 状态和审核通知判定结果。原始会话和运行日志仅保存在本机实验目录，不提交公开仓库。
