# dnd5e-skill

> **Claude Code skill**：让 Claude 成为你的中文 **DnD 5e 2014 经典版（旧版 / PHB14）** 助手——查证规则、车卡、查法术、**当 DM 带你跑模组**、**按需定制改写模组（单人 / 双人 / 克苏鲁风味 / 治愈向 / 主线设定调整 …）**，全部基于"DND 五版不全书"中文资料库，**绝不靠训练记忆瞎编**。

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> 想要 **2024 修订版（5r）**？用姊妹项目 **[dnd5r-skill](https://github.com/ZGP0072600/dnd5r-skill)**。两者**同一份资料、同一套工具**，区别只是默认规则版本：本项目默认 **2014 经典规则**，dnd5r 默认 **2024 修订规则**。

---

## 这是什么

一个针对 [Claude Code](https://docs.claude.com/en/docs/claude-code) 的 [skill](https://docs.claude.com/en/docs/claude-code/skills)。装上之后，你可以这样和 Claude 对话：

- "帮我用邪术师车一张 5 级卡" → 自动从 PHB14 查职业、子职、专长，给你一张可点可投骰的 HTML 电子卡
- "经典 5e 法师 1 环法术推荐哪个" → 查证后给推荐
- "火球术升 4 环加多少伤害" → 翻 `玩家手册/魔法/法术详述/3环.md` 引用原文回答
- "战士的奥法骑士子职怎么样" → 读 `玩家手册/职业/战士/奥法骑士.md` 给评价
- "成年红龙的喷吐多少伤害" → 翻怪物图鉴
- **"你来当 DM 带我跑团"** → 进 DM 模式，按章节读叙事、判定检定、管理战斗、记 session 日志
- **"做个双人克苏鲁版的 XX 模组"** → 自动改写模组为定制副本（怪物缩放、NPC 风味调整、剧情骨架不动）

**关键约束：所有规则数值都先查资料库再回答**——SKILL.md 第一条规则就是「绝不能用训练记忆答」，因为你训练数据里的"5e"可能混入 2024 内容、也可能与"不全书"中译术语不符。**2014 优先**：默认认 PHB14（玩家手册 / 城主指南 / 怪物图鉴），没有的再回退扩展书、最后才是 2024 修订版，并明示出处。

---

## 已实现的能力

### 1. 资料查证（2014 优先）

`docs/extracted/` 下是完整的"DND 五版不全书"，已转 UTF-8 + 解开锚点，包含：

- **玩家手册（PHB14）/ 城主指南（DMG14）/ 怪物图鉴（MM14）** — 默认优先来源
- 玩家手册 2024 / 城主指南 2024 / 怪物图鉴 2025（2024 修订版，作回退）
- 塔莎、珊娜萨、贤者谏言、万象无常等扩展；剑湾、费资本等设定与种族
- 多个第三方书（瓦尔达的秘密尖塔=铳士、血猎手 等整职业）
- 法术速查、怪物速查、种族速查等索引

### 2. ⚡ 快查 CLI（亮点）

`tools/` 下有一层**派生索引 + stdlib Python CLI**，把"grep 大文件 + 多次 Read"压成一次命令出结构化记录——秒级、确定、自带 `path:line` 引用。**任何带 Python 的 Agent 都能零安装直接调**（只用标准库），clone 即用（索引已随仓库附带）：

```bash
python tools/query.py spell 火球术                  # 主显玩家手册(2014)，2024 版列「其他版本」
python tools/query.py spell --class 法师 --level 2   # 枚举默认 PHB14（--all 全部、--source 玩家手册2024 指定 2024）
python tools/query.py monster 哥布林                # 哥布林→地精 别名自动归一
python tools/query.py class                         # 列全部职业；subclass --class 法师 = 8 学派
python tools/query.py race 半精灵                    # PHB14 含半精灵/半兽人（PHB24 已并掉，只在这查得到）
python tools/query.py feat 警觉                       # PHB14 核心专长
python tools/query.py equip 长剑                      # PHB14 武器/护甲
python tools/query.py magic 雷神之锤                  # 魔法物品
```

8 个查询域：`spell / monster / class / subclass / race / feat / equip / magic`，全 **2014 优先**。详见 [tools/README.md](tools/README.md)。资料库更新后跑 `python tools/build_index.py` 重建索引。

### 3. HTML 电子角色卡

`templates/character-sheet.html` 是一个**完全自包含**的单文件交互卡：

- 单文件、零外链：CSS/JS/字体/数据全部内联，离线可用——复制到任何位置都能用（发微信、放手机/iPad/电脑、浏览器打开即是）
- 内置投骰对话框（属性、技能、攻击、伤害、法术）支持优势/劣势
- HP / 法术位 / 职业资源（pip 池）/ 死豁 / 状态 / 抗性 / 装备负重 / 历史骰史
- 短休 / 长休一键、自动恢复对应资源；主题切换；LocalStorage 自动存档
- **兼职支持**：混合 HD 显示，短休按骰型分组依次询问消耗
- **特殊投骰面板**：如混沌施法术士的 d100 法力狂潮表，自动投骰/查表/续投/写历史

### 4. 通过 build-sheet.py 出卡

```bash
# 输入 JSON（schema 见 SKILL.md / CHARACTER_SCHEMA.md），输出自包含 HTML
python templates/build-sheet.py character.json output.html
```

### 5. DM 跑团（带模组）

SKILL.md **工作流 G** 让 Claude 切换为 DM 模式带你跑一本团（专为单人 / 双人小团设计）。**Step 1 文件持久化战役系统**——所有跑团状态（战斗 / 玩家 / NPC / 世界 / DM 风格 / 房规）落到 `campaigns/<战役名>/` 目录，不再仅靠对话上下文（完整设计见 [STEP1_DESIGN.md](STEP1_DESIGN.md)）。

- **G1 七步初始化**：确定模组 → 选 DM 风格 → 定房规 → 阵容 → 车卡 → 建战役目录 → 开场叙事
- **跑团循环 / 文件持久化战斗 / 桌末同步 / 跨 session 恢复**（说「继续上次跑团」即按固定顺序读全套战役文件续接）
- **🔒 秘密物理隔离**：DM 秘密放 `dm-only/` 子目录，AI 默认不读
- **DM 风格预设**（[profiles/dm-styles/](profiles/dm-styles/README.md)）：本期收录「标准」「粉红恋爱向」两个示范，战役引用 + override 微调

> 📦 **本仓库不附带任何模组内容**（版权原因），也未附示范战役。要跑团需把模组 PDF / 文本放进 `docs/modules/<模组名>/`，第一次开桌前让 Claude 抽取组织。查证类功能（规则 / 法术 / 怪物 / 车卡）**开箱即用**，无需准备。

### 6. 模组定制改写

SKILL.md **工作流 H**：原版只读、所有定制走**全量副本** `docs/modules/<原模组>__<标签>/`。用自然语言描述需求（"单人 + 克苏鲁 + 主角性转 + 加同伴 NPC" 等），Claude 解析成改写计划，确认后执行。**分层护栏**：🔒 数据原子不动 / 🟡 数值层按人数缩放 / 🟢 风味层改叙事描写 / ⚪ 剧情骨架默认不动。改完先给预览确认基调再批量推完。

### 7. 沙盒模式（开放世界生活模拟）

SKILL.md **工作流 I**（细则见 [WORKFLOW_I_SANDBOX.md](WORKFLOW_I_SANDBOX.md)）：玩家自驱的沙盒（"不跑模组，我想自由玩 / 定居 / 接委托 / 谈恋爱 / 经商 / 建要塞"）。含日常四段循环、时间推进、委托系统、关系演进、资产经营、家族系统等，可与模组（工作流 G）互相嵌套。

### 8. 自定义内容（Homebrew）

SKILL.md **工作流 J**：把自创的种族 / 职业 / 子职 / 专长 / 法宝 / 怪物 / 法术做成和官方同格式的 `.md` 存进库，之后车卡 / 跑团 / 枚举全程可查可用。

- **作用域**：全局 `homebrew/<类型>/` 或战役专属 `campaigns/<战役名>/homebrew/<类型>/`
- **查询优先级**：战役专属 → 全局 → 官方书
- **平衡校验**（不可跳过）：Grep 同类官方内容对标强度，主动指出偏强 / 偏弱

### 9. 存档系统（/save · /load）

SKILL.md **工作流 S**：跑团 / 沙盒中随时打存档点、任意回溯。`/save <名称>` 快照、`/load <名称>` 恢复、`/saves` 列出。每个存档独立目录互不覆盖；流水账（sessions / 战斗历史）不纳入存档、恢复后仍保留。

---

## 安装

```bash
# 用户级（所有项目可用）
git clone https://github.com/ZGP0072600/dnd5e-skill.git ~/.claude/skills/dnd5e

# 或项目级
git clone https://github.com/ZGP0072600/dnd5e-skill.git .claude/skills/dnd5e
```

资料库 `docs/extracted/` 与预建索引 `tools/index/` 已随仓库附带，clone 即用。启动 Claude Code 后提"帮我车一张 3 级武僧"或"经典 5e 火球术升 4 环加多少伤害"，skill 会自动触发（提到 D&D / 5e / 经典版 / 旧版 / 不全书 等关键词时）。

## 用法示例

详细工作流见 [SKILL.md](SKILL.md)，简要：

| 你说 | Claude 做的 |
|---|---|
| X 法术怎么样 / X 怪物 CR 多少 | 工作流 A/D：**首选快查 CLI**（`query.py spell/monster`，2014 优先），需正文细节再按 path:line 读原文 |
| X 职业有哪些子职 / 有哪些种族·专长 | 工作流 E：`query.py subclass --class X` / `class` / `race` / `feat`（PHB14 基准 + 各扩展追加） |
| 帮我用 X 车一张 N 级卡 | 工作流 B：种族 → 职业/子职 → 背景 → 属性 → 专长 → 装备 → 法术 → 结算 → 出 HTML 卡 |
| 某规则怎么裁定 | 工作流 C：grep `进行游戏` / `术语汇编`，2014 与 2024 有差异时先答 2014 |
| 你来当 DM 带我跑 X 模组 / 继续上次跑团 | 工作流 G：七步初始化 → 叙事循环 → 文件持久化战斗 → 桌末同步 → 跨 session 恢复 |
| 做单人 / 双人 / 克苏鲁版 / 治愈向 / 性转版 的 X 模组 | 工作流 H：解析需求 → 改写计划确认 → 全量副本 → 分层改写 |
| 不跑模组，我想自由玩 / 定居 / 接委托经商 | 工作流 I：沙盒五问 → 建目录骨架 → 日常四段循环 → 委托 / 关系 / 资产推进 |
| 帮我做一个自定义种族 / 子职 / 法宝 / 怪物 | 工作流 J：确定类型 → 采集设计 → 平衡校验 → 写 `.md` → 更新索引 |
| `/save` / `/load` / `/saves` | 工作流 S：打存档 / 恢复 / 列出（G 和 I 均支持） |

---

## 数据来源 / 版权说明

`docs/` 下的中文规则内容来自社区合作翻译项目「**DND 五版不全书**」。

- **原版权**：D&D 5e 系列规则归 Wizards of the Coast 所有
- **翻译版权**：中文翻译归不全书翻译团队及参与者所有
- **本仓库的角色**：仅作为 Claude Code skill 的数据源，让 Claude 从原文查证而非凭训练数据瞎编；不主张任何权利、不收费
- **如有侵权**：请提 [Issue](https://github.com/ZGP0072600/dnd5e-skill/issues) 或联系仓主，会立即下架。模组正文等版权敏感内容不随仓库发布

请尊重原创：**支持购买 WotC 官方书籍**，并在能力范围内**支持不全书翻译团队**的工作。

---

## 已知限制

- **目录索引覆盖**：职业/子职/种族/专长/装备已接入 CLI（PHB14 基准 + PHB24/塔莎/珊娜萨/第三方追加）；装备暂只武器+护甲、专长为启发式解析（少数 en 截断，长尾按 path:line 核对）
- **法术位池单字段** / **兼职 HD 共享一个 hdCur** / **豁免熟练**：兼职多施法者时 DC 用单一 castingAbility、HD 玩家自记、豁免熟练只继承 1 级职业（同 dnd5r 模板）
- **不全书更新**：上游有新内容时需手动更新 `docs/extracted/` 并重建索引

---

## License

代码（SKILL.md、templates/、tools/）使用 [MIT License](LICENSE)。`docs/` 下的规则文本不在本仓库 license 涵盖范围内——见上文「数据来源/版权说明」。
