# 两页GUI与模型接入

日期：2026-10-24。状态：界面范围与TypeScript后端为用户要求；桌面框架为助手建议。只规划，未实现、打包或调用模型。

## 用户确定的界面范围

先展示问题和参数配置页，启动后进入成果导图与主Agent对话框组成的工作页。第一版不增加多个管理面板。

| 配置项 | 当前含义 |
| --- | --- |
| 问题 | 用户输入猜想或选择题库问题 |
| Agent数量 | 本次可管理的证明Agent数量 |
| 模型地址和名字 | 证明模型的API地址与模型ID |
| 审查模型名字 | 默认与证明模型一致；建议默认同时继承其地址与凭据配置 |
| 并发 | 同时在途的模型请求上限，与Agent数量分开 |
| 汇报间隔 | 默认20分钟 |

API Key通过本机凭据配置提供，不写进证明目录或导出包。计算任务并发可留作高级参数，不把Agent数量直接当作远端计算并发。

工作页仅有导图与主Agent对话。点击节点可展开当前任务、进展、子模块及对应文件；用户的方向调整通过主Agent执行。无新证明不增加成果节点。

## 一体桌面GUI的建议

可以将前后端打进同一个桌面安装包，内部仍保留界面与执行模块的边界。按用户要求后端用TypeScript，助手建议优先采用：

| 部分 | 建议技术 | 用途 |
| --- | --- | --- |
| 桌面外壳 | Electron | 提供应用窗口并管理本机后台生命周期 |
| 前端 | React、TypeScript、Vite | 配置表单、对话框和节点详情 |
| 导图 | React Flow | 节点、连线、选择、缩放与自定义成果节点 |
| 后台 | Node.js、TypeScript与Codex app-server | 复用Codex会话、Goal和自动审核，项目负责业务流程；用户已改选Codex，接入未实测，桌面框架仍为建议 |

[Electron进程文档](https://www.electronjs.org/docs/latest/tutorial/process-model)说明主进程具有Node.js能力，界面运行于渲染进程；[React Flow文档](https://reactflow.dev/learn/concepts/terms-and-definitions)说明节点、连线和选择等交互。这里是选型依据，不是本项目已完成的性能或三端验收。

本项目只需一套GUI管理多个异步Agent会话，不为每个Agent新开一套桌面应用。Electron带有Chromium进程模型，实际内存仍需测量，不能据此保证Air的百Agent容量。

建议把TS执行模块与窗口代码分开，窗口关闭时可以保留后台运行；彻底退出应用时保存续接状态并停止本机调度，远端已提交任务的续接由计算工具处理。以后迁移后台到服务器时，沿用相同业务接口并补远端连接。

Tauri也能通过[sidecar机制](https://v2.tauri.app/develop/sidecar/)打包外部后台，但采用Node/TS后台时需处理额外运行时及其打包。当前为保持TS开发路径简单，先推荐Electron；用户尚未选定桌面框架。

## OpenCode Go的自建客户端接入

[Go官方文档](https://opencode.ai/v2/docs/console/go/)允许产生类似编码请求的其他Agent接入，并公开直接调用接口，不只限于Pi或Codex。要求客户端用自己的User-Agent，并逐对话发送稳定的x-opencode-session。

本次核验的DeepSeek模型ID为deepseek-v4.1-flash，完整请求地址为https://opencode.ai/inference/go/openai/v1/chat/completions。配置Base URL可填https://opencode.ai/inference/go/openai/v1，后端再按接口拼接请求路径。

建议自建客户端如实标识ParrelSolveInstantProblem及版本，稳定区分各对话。Lean源码编写与验证接近编码Agent用法；官方未单独确认本项目批量科研场景或承诺100路并发。接入后仍须遵守额度与服务限制；不以伪装客户端或轮换身份规避限制。

2026-10-24补充：用户改选Codex。Go有官方Codex配置，但Codex自定义provider要求Responses；上述DeepSeek当前列出的Chat Completions接口不能仅改地址就认定直连可用。详见[运行时比较与接入](codex-integration.md)，不自动更换用户指定模型。

当前只读官方说明，未验证用户密钥、发送推理请求、购买服务或安装客户端。模型地址与名字保留可配置，避免产品绑定单一套餐。
