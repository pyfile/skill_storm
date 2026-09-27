# storm-report

一个 [CodeBuddy Code](https://cnb.cool/codebuddy/codebuddy-code) skill：用 Stanford STORM 框架研究一个主题，产出一份给自己看的综合研究报告。

改编自 Shao et al., *Assisting in Writing Wikipedia-like Articles From Scratch* ([arXiv:2402.14207](https://arxiv.org/abs/2402.14207))，并叠加了两个原论文没有的环节：**显式冲突图谱（Contradiction Map）** 与 **隔离式同行评审（Peer Preview）**。保留论文中真正带来增益的机制，砍掉为论文评测服务的部分。

## 它产出什么

一份 `60_report.md`，固定五个模块，顺序不可变：

1. **60 秒摘要** — 只读这一段就该知道这个主题现在是什么情况
2. **五个最重要的发现** — 按**可靠性**而非重要性排序，每条必带反对视角与边界条件
3. **隐藏联系** — 不同视角之间最不容易发现的关联
4. **可执行建议** — 只针对 `references/user-profile.md` 里那个人，具体到能直接开工
5. **前沿问题** — 被解决就意味着该领域被重定义，不是「未来研究方向」清单

## 工作流

七个阶段，全部状态落盘到 `.storm/<topic-slug>/`，中断可续：

| 阶段 | 做什么 | 备注 |
|---|---|---|
| 0 · 准备 | 确认主题与范围，建工作目录 | 不确定边界就先问，不要猜 |
| 1 · 发现视角 | 扫相关主题的 TOC，套 5 个固定角色 | **硬闸门**，必须停下等批准 |
| 2a · 基础事实速查 | 3–5 条无可争议的基本事实 | 可跳过 |
| 2b · 并行模拟对话 | 每个视角一条多轮对话，默认 3 轮 | 核心环节，不可省略 |
| 3 · 冲突图谱 | 五问：冲突 / 证据强弱 / 关键问题 / 共识 / 集体盲区 | 主 agent 亲自做 |
| 4 · 证据池与提纲 | 抽证据条目，两段式生成提纲 | |
| 5 · 写报告 | 按 `references/report-spec.md` 产出 | |
| 6 · 同行评审 | 独立评审子代理 + 主 agent 逐条处置 | 只看报告 + 来源清单 |

## 关于工作目录

`.storm/<topic-slug>/` 是**相对路径**，解析到**你调用这个 skill 时所在的项目根目录**，不会写进 skill 自身的安装目录（skill 执行时不改变 cwd，只注入一个 `Base directory` 提示）。这是刻意的：研究报告跟着被研究的项目走，而不是塞进 `~/.codebuddy/skills/`。

所以如果这个 skill 已经装在 `~/.codebuddy/skills/storm-report/`，你在 `~/proj/A` 里说「研究一下 X」，产物就在 `~/proj/A/.storm/x/`。建议在你自己的项目里把 `.storm/` 加进 `.gitignore`。

## 三条来之不易的硬结论

这些是原论文消融实验给出的，不要凭直觉改动：

- **多轮对话式追问是主要增益来源** — 去掉 conversation，heading soft recall 从 86.26 掉到 77.97；去掉 perspective 只掉到 84.49。**宁可砍视角数，不要砍轮次。**
- **提问方看不到检索原文** — 检索只发生在回答侧。把未组织的搜索结果直接塞进上下文会让提纲质量**下降**。
- **提纲必须两段式** — 先纯参数知识出草稿搭骨架，再用全部对话修订。不要一步到位。

另外，评审 agent **绝不能**看到 `20_conv/` 与推理过程，否则评审退化为自评、分数虚高。

## harness 无关

不依赖任何框架专属 API，只要求宿主具备：联网搜索、抓取网页正文、并行子代理、上下文隔离、读写文件。缺哪样按 `SKILL.md` 的「能力槽」表降级处理 —— 比如没有并行子代理就串行执行五遍，流程与产物不变，只是慢。

## 安装

把内容放到你的 skills 目录下即可：

```bash
git clone git@github.com:pyfile/skill_storm.git ~/.codebuddy/skills/storm-report
```

项目级安装同理，放 `.codebuddy/skills/storm-report/`。目录名与 `SKILL.md` frontmatter 里的 `name: storm-report` 保持一致。skill 加载器只认文件名严格等于 `SKILL.md` 的条目，目录里的其他文件（含本 README）不会被解析，也不会被自动注入上下文。

然后直接说「帮我研究一下 X」「做个综述」「多视角分析」。

## 目录结构

```
SKILL.md              主流程与全部规则
references/
  personas.md         5 个固定角色的身份与关注点模板
  report-spec.md      报告五模块的结构契约
  peer-review.md      评审子代理的完整 prompt 骨架
  anti-patterns.md    原论文已知缺陷清单，写完后自查
  user-profile.md     用户画像 —— 模块 4 的唯一依据，同时约束全文表达风格
```

## 已知缺陷

源偏见转移、把无关事实硬连起来（red herring）、情绪化措辞、时效性表述（把 `现在` 写成 `截至 YYYY-MM`）。清单见 `references/anti-patterns.md`，这也是阶段 6 评审的输入来源。
