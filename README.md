# dnd5e-skill

> **Claude Code skill**：让 Claude 成为你的中文 **DnD 5e 2014 经典版（旧版 / PHB14）** 助手——查证规则、车卡、查法术、**当 DM 带你跑模组**，全部基于"DND 五版不全书"中文资料库，**绝不靠训练记忆瞎编**。

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> 想要 **2024 修订版（5r）**？用姊妹项目 [dnd5r-skill](https://github.com/ZGP0072600/dnd5r-skill)。两者同一份资料、同一套工具，区别只是**默认 2014 经典规则 vs 2024 修订规则**。

---

## 这是什么

一个针对 [Claude Code](https://docs.claude.com/en/docs/claude-code) 的 [skill](https://docs.claude.com/en/docs/claude-code/skills)。装上之后，Claude 处理 D&D 5e **2014 经典版**请求时会：

- **查证再答**：所有规则数值 / 法术 / 怪物 / 子职 / 专长 / 装备，都先从 `docs/extracted/` 的中文资料库查到原文再回答，并标注来源 `path:line`。
- **2014 优先**：默认认 PHB14（玩家手册 / 城主指南 / 怪物图鉴），2014 没有的再回退扩展书、最后才是 2024 修订版，并明示出处。
- **会车卡、会带团**：按步骤建角色卡（产出自包含 HTML），也能扮演 DM 带你跑模组。

## ⚡ 快查 CLI（亮点）

`tools/` 下有一层**派生索引 + stdlib Python CLI**，把"grep 大文件 + 多次 Read"压成一次命令出结构化记录——秒级、确定、自带引用。**任何带 Python 的 Agent 都能零安装直接调**（只用标准库）：

```bash
python tools/query.py spell 火球术                  # 主显玩家手册(2014)，2024 版列「其他版本」
python tools/query.py monster 哥布林                # 哥布林→地精 别名自动归一
python tools/query.py subclass --class 法师          # 8 学派（PHB14）
python tools/query.py race 半精灵                    # PHB14 含半精灵/半兽人
python tools/query.py feat 警觉                       # PHB14 核心专长
python tools/query.py equip 长剑                      # PHB14 武器/护甲
python tools/query.py magic 雷神之锤                  # 魔法物品
```

8 个查询域：`spell / monster / class / subclass / race / feat / equip / magic`，全 **2014 优先**。详见 [tools/README.md](tools/README.md)。

## 安装

```bash
git clone https://github.com/ZGP0072600/dnd5e-skill.git
```

把整个仓库内容放到 Claude Code 的 skill 目录（如 `~/.claude/skills/dnd5e/` 或项目级 `.claude/skills/dnd5e/`），让 Claude 能加载 `SKILL.md`。资料库 `docs/extracted/` 已随仓库附带，clone 即用。

提到 D&D / 5e / 经典版 / 旧版 / 不全书 等关键词时，Claude 会自动启用本 skill。

## 内容

- `SKILL.md` — skill 主文件（工作流、规则、约束）
- `tools/` — 快查 CLI（`build_index.py` 构建索引、`query.py` 查询、`index/` 预建索引）
- `docs/extracted/` — "DND 五版不全书"中文资料库（2014 优先 + 各扩展 + 2024 修订版）
- `templates/` `ui/` `profiles/` — 角色卡生成、面板、DM 风格预设
- `homebrew/` — 自定义内容库

## 致谢

规则资料来自中文社区"DND 五版不全书"。本项目仅做查询/工具封装，版权归原作者与 Wizards of the Coast。模组正文等版权敏感内容不随仓库发布。

## License

[MIT](LICENSE)（指本仓库的工具代码与封装；资料库内容版权归原权利人）
