# 推行经验 (Lessons Learned)

这份文档记录在实际项目里跑 PM + Dev + QA 三角色协作时积累的教训，和 README 里的流程文档互补——README 讲"应该怎么做"，这里讲"为什么这么做以及踩过什么坑"。

## 1. Dev 用 Opus，PM / QA 用 Sonnet

**规则**：`开发工程师/agent.md` frontmatter 里建议 `model: opus`；PM 和 QA 保留 `model: sonnet`。

**为什么**：
- Dev 的工作是写/改源码、处理跨文件重构、保证测试通过。代码质量直接决定 PR 能不能合，这里不能省 token。
- PM 和 QA 主要做分析、对照、写报告，sonnet 足够且更便宜。
- 实测 Dev 用 sonnet 容易出现"补丁式修复"——把问题绕过去而不是解决，review 阶段会被反复打回，整体反而更贵。

**怎么落**：把 `开发工程师/agent.md` 顶部的 `model: sonnet` 改成 `model: opus` 即可；其他两个不动。

## 2. 任何改源码的任务都必须开 Team，不能单干

**规则**：Dev agent 不在主会话里直接改代码。只要涉及改项目源文件，就开一个 Team（PM + Dev + QA），走完 Issue → PR → Review → QA 流程。

**为什么**：
- 单人直改容易跳过 Issue 拆分，改着改着范围失控。
- 没有 PM review 就没人对照验收标准，也没人挡住"顺手优化"。
- Team 流程强制留下 Issue/PR 痕迹，后面排查时能看懂"为什么这么改"。

**怎么落**：
- 用户说"改一下 XX"时，PM 先判断是否需要动源码。要动 → 开 Team；只是咨询/分析 → 主会话回答即可。
- 开 Team 的标准配置：PM (sonnet) + Dev (opus) + QA (sonnet)。

## 3. Team 模式下必须显式触发 skill，不能只发自然语言

**规则**：给 subagent 发消息时，要明确写出要调用的 skill 名字（比如 `e2e-browser-test`、`simplify`、`review-pr`），不能只描述任务。

**为什么**：
- subagent 看到自然语言任务时会按自己的默认流程走，经常绕开已配好的 skill。
- Skill 里沉淀的是"踩过坑的流程"（比如 e2e-browser-test 里对 session 注入、select / dialog / 异步等待的处理），不走 skill = 重新踩一遍。

**怎么落**：
- PM 派活给 Dev："请使用 `submit-pr` skill 完成 Issue #12"
- PM 自审 PR："请使用 `review-pr` skill 对 PR #34 做 review"
- QA 验证："请使用 `e2e-browser-test` skill 完成 PR #34 的验收"
- Dev 简化代码："请使用 `simplify` skill 处理 XXX"

## 4. 严格串行门禁：每个 Task 必须 PM Review PASS 才开下一个

**规则**：一次拆成多个 Task / Issue 时，不能让 Dev 并行全部做掉再一次性交付。必须：Issue 1 完成 → PM review → PASS → 才启动 Issue 2。

**为什么**：
- 一次性交付十个 PR 时 PM review 会疲劳漏问题，返工成本极高。
- 前一个 Issue 的 review 结论往往会改变后面 Issue 的做法（接口调整、拆分方式等），提前并行做等于白做。
- 串行做时每个 PR 小、聚焦，review 质量高，bug 更容易在当轮就被抓出来。

**怎么落**：
- PM 在拆 Issue 时标出依赖顺序，Dev 一次只接一个。
- 遇到"能不能一起做"的冲动就停下——等上一个合并了再开下一个。

## 5. 协作通信：GitHub 只留结论，拉齐沟通走 Team 内部消息

**规则**：Issue / PR 评论区只写正式结论（验收标准、review 结果、测试报告、LGTM / 返工）。三角色之间的讨论、澄清、快速拉齐，走 Claude Code Team 的内部消息。

**为什么**：
- GitHub 评论区是对外可见的需求 / 验收证据，灌太多"我这边在想……"的讨论会淹没关键结论。
- 别的 agent 或未来回顾时只想看"这个 PR 到底过没过、为什么过"，讨论过程是噪声。
- Team 内部消息本来就是为了让 PM 和 Dev、Dev 和 QA 实时对齐，用对工具才能高效。

**怎么落**：
- 拿不准需求 → Team 里问 PM，PM 确认后 Dev 再去 Issue 下留言沉淀结论。
- QA 发现疑似 bug → 先 Team 里和 Dev 对一下是否预期 → 确认是 bug 再在 PR 写正式失败报告。

## 6. PRD / 路线图处理边界（项目相关）

**规则**：对照 PRD 时，只把"已有后端能力但前端未实现"的模块补入路线图。完全未实现的新模块不自动纳入，要用户明确要做才加。

**为什么**：
- 路线图是做承诺，承诺了就必须排期交付。把所有 PRD 项塞进来等于给未来的自己挖坑。
- 已有后端能力说明成本低、价值被验证过；纯新模块需要独立立项评估。

**怎么落**：PM 梳理 PRD 时分两栏——"后端已就绪待前端"（入路线图）与"全新模块"（列清单不排期），让用户勾选后者是否进入路线图。

---

这些经验都是在 StepClaw Console 项目推这套流程时踩出来的，未来新坑会继续补进来。
