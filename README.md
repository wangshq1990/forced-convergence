# Forced Convergence · 五要素强制收敛提示词引擎

> 把模糊、半成型的 AI 任务请求，收敛成一份可直接交付的提示词；并可选地在同会话内以该提示词为任务简报继续执行。
> A prompt-engineering skill that forces vague requests into five hard constraints before execution.

**版本**：v1.2.0 · **格式**：Agent Skill（`SKILL.md`） · **许可**：MIT

---

## 它解决什么问题

大模型的输出是发散性的概率分布。只说「帮我写个方案」，得到的是通用、冗长、不可执行的内容。本 Skill 用五个硬槽位把分布收敛到可交付：

| 槽位 | 回答的问题 | 缺它后果 |
|---|---|---|
| 角色 Role | 谁来做、用什么**可观察的评判标准** | 通用实习生式泛泛而谈 |
| 背景 Context | 在什么真实条件下做（筹码、限制、环境） | 给出超出能力或预算的方案 |
| 目标 Goal | 要什么**具体交付物**，拆成 1/2/3 个勾选项 | 「探讨一下」式软弱内容 |
| 约束 Constraint | 绝不能怎样（禁用词、资源上限、语气、篇幅） | 几万字正确的废话 |
| 格式 Format | 最终以什么形态交付（表格 / 大纲 / JSON） | 需二次排版 |

第 6 槽「验收 Acceptance」可选，用于写明合格线与自查方式，显著减少返工。

## 工作流

1. **识别领域与路由**：营销 / 软件工程 / 学术写作 / 法律 / 数据分析 / 产品，其它领域走移植契约。判断用户是否「只要提示词」（Q1）。
2. **逐槽补全**：缺哪个追问哪个，不一次性索要全部信息。角色写评判标准而非头衔；目标用祈使句并拆解勾选项。
3. **加载领域档案**：从 `references/domain-profiles/` 读取结构建议，不编造领域事实。
4. **组装提示词**：以中文方头括号标注各槽。
5. **确定性校验**：调用 `scripts/build_prompt.py` 检查五要素覆盖率、约束互斥、约束数量。
6. **Phase A**：在对话内展示最终提示词与适配说明（**仅展示、不落盘**）。
7. **Phase B**（默认）：以该提示词为 authoritative brief，在同会话内执行用户的真实任务；执行前留出纠偏窗口（Q2：调整 / 停止 / 继续）。

若用户在 Q1 明示「只要提示词 / 给别处 AI」，则停在 Phase A，不进入执行。

## 目录结构

```
forced-convergence/
├── SKILL.md                         # 主规则：五要素模型、7 步流程、边界
├── references/
│   ├── five-element-method.md       # 方法论深度说明
│   ├── domain-portability.md        # 跨领域移植契约
│   ├── quality-rubric.md            # 产出验收标准
│   └── domain-profiles/             # 6 个领域的默认值档案
│       ├── marketing.md
│       ├── software-engineering.md
│       ├── academic-writing.md
│       ├── legal.md
│       ├── data-analysis.md
│       └── product.md
├── scripts/
│   ├── build_prompt.py              # 确定性组装 + 覆盖率/冲突校验
│   └── selftest.py                  # 自检用例
├── LICENSE
└── .gitignore
```

## 安装

将本目录整体放入你的 Agent 的 skills 目录，保持目录名为 `forced-convergence`。例如 WorkBuddy：

```
~/.workbuddy/skills/forced-convergence/
```

无需安装依赖、无需联网、无需 API key。`build_prompt.py` 仅使用 Python 标准库。

## 触发示例

- 「帮我把这个需求变成给 AI 的提示词」
- 「怎么写 prompt 才不让 AI 说废话」
- 「把这个任务结构化一下再交给 AI」
- 「把这个想法变成成品」

## 校验脚本

```bash
# 默认只打印到屏幕，不写任何文件
python scripts/build_prompt.py --input example.json

# 运行自检
python scripts/selftest.py
```

`build_prompt.py` 以退出码 0 / 1 表示五要素齐全 / 缺失（INCOMPLETE），并给出约束互斥与数量的警告。设计上不联网、不启动子进程、不读取环境变量。

## 边界

本 Skill 明确不做以下事情：

- **B1** 不在对话外自动执行删除、覆盖、发送、发布、付费、外部 API 写入等副作用动作
- **B2** 不读写用户个人文件、配置或凭证
- **B3** 不索要 API key、密码、token
- **B4** 不编造领域事实或数据
- **B5** 用户未提供真实任务信息时不臆造、不空执行
- **B6** 不跨会话、不后台常驻

允许的是：同会话内执行用户真实任务（A1），以及对话内展示提示词供调整（A2）。提示词仅在对话内展示，不落盘。

## 许可

MIT — 见 [LICENSE](LICENSE)。
