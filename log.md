# Log

> 时间线 append-only。每条记录起一行新二级标题，格式：
> `## [YYYY-MM-DD] <action> [<slug>] — <一句话>`
> action 枚举：init / ingest / distill / log / lint / decision / pitfall / milestone / rename / promote / archive / delete
> 详细规则见 [[CLAUDE]] §5.2。

## [2026-05-01] init — 个人 wiki 初始化，骨架 + schema + 6 个 slash commands + hooks 落地

## [2026-05-02] rename 2026-05-02-155920-youtube→karpathy-vibe-coding-agentic-engineering — 将 YouTube 剪藏 source 改为语义化 slug

## [2026-05-02] distill karpathy-vibe-coding-agentic-engineering — 提炼 Karpathy 关于 agentic engineering 与 AI 自动化边界的访谈

## [2026-05-02] log — 衡量好 agent 工程师的标准：0→1 高效构建大型项目不牺牲质量；关键能力是深度投资 agent 工具链；质量度量可用"组织 agent 攻击系统看能否承受"

## [2026-05-02] ingest agent-engineer-quality — 把上述 log 心得 fan-out 成 3 个 concept 骨架（agent-engineer-quality / agent-toolchain-investment / agent-adversarial-quality-test）

## [2026-05-02] decision ingest-supports-log-input — /ingest 输入正式支持 log 心得引用（log:<date-or-slug>）；跳过 source 页创建直接 fan-out 到 entity/concept

## [2026-05-03] log — 评审 AI 技术方案时强制要求补 Done when 完成标准：业务结果、技术结果、边界情况、测试验证、监控指标、回滚方式；说不清完成标准的方案，多半是在写技术作文

## [2026-05-03] rename 2026-05-03-194746-article→hoard-things-you-know-how-to-do — 将 Simon Willison 文章剪藏 source 改为语义化 slug

## [2026-05-03] distill hoard-things-you-know-how-to-do — 提炼 Simon Willison 关于囤积可运行方案并复用为 agent 输入资产的文章

## [2026-05-03] rename 2026-05-03-200332-article→ai-should-help-us-produce-better-code — 将 Simon Willison 文章剪藏 source 改为语义化 slug

## [2026-05-03] distill ai-should-help-us-produce-better-code — 翻译并提炼 Simon Willison 关于用 AI agent 提升代码质量的文章

## [2026-05-30] rename 2026-05-30-154829-article→building-self-improving-tax-agents-with-codex — 将 OpenAI Codex 税务 agent 文章剪藏 source 改为语义化 slug

## [2026-05-30] distill building-self-improving-tax-agents-with-codex — 提炼 OpenAI/Thrive Tax AI 如何用生产 trace、专家反馈和 Codex eval 循环构建自我改进 agent

## [2026-05-31] rename 2026-05-31-184729-article→how-to-eval-ai-agents-2026-guide — 将 Ben Hylak agent eval 指南剪藏 source 改为语义化 slug

## [2026-05-31] distill how-to-eval-ai-agents-2026-guide — 提炼 Ben Hylak 关于 agent eval 应从真实失败、代码感知测试和生产监控中持续抬高下限的指南
