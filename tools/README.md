# dnd5e 快查工具（fast query layer · **2014 优先**）

dnd5e 是 dnd5r 快查工具的 **2014-first 复刻**——**同一份不全书资料**（`docs/extracted/`，6171 md 与 dnd5r 完全相同）、**`query.py` 与 dnd5r 逐字相同**（它只读记录里的 `priority` 字段，不关心版本）。区别全在 `build_index.py`：

1. **`PRIORITY` 表翻转**：PHB14 核心（玩家手册/城主指南/怪物图鉴）= 0，2024 修订版（玩家手册2024/…）降到 1 → 单查折叠到 PHB14 印次、枚举默认 PHB14。
2. **目录域基准换 PHB14**：职业/种族/专长/装备的基准从 `玩家手册2024/…` 改到 `玩家手册/…`，PHB24 反过来当「追加」。

```
tools/
├── build_index.py   构建期：读 docs/extracted frontmatter → 索引（用 PyYAML；2014-first 配置）
├── query.py         查询期：只读索引、只用 stdlib（与 dnd5r 同一份文件）
├── index/           ← 派生产物，勿手改
│   ├── spells.json  monsters.json  classes.json  races.json
│   └── feats.json   equipment.json  magic.json
└── README.md        本文件
```

## 用法（8 个查询域，全 2014 优先）

```bash
# 法术 / 怪物 / 魔法物品（内容与 dnd5r 相同，仅优先级翻转）
python query.py spell 火球术                  # 主显玩家手册(2014)，2024 版列「其他版本」
python query.py spell --class 法师 --level 2   # 枚举默认 PHB14（--all 全部、--source 玩家手册2024 指定 2024 版）
python query.py monster 哥布林                # 哥布林→地精 别名自动归一
python query.py magic 雷神之锤                # 类型·稀有度·调谐 + path:line

# 职业 / 子职（PHB14 基准 + PHB24/塔莎/珊娜萨/奇械师/铳士/血猎手/第三方 追加）
python query.py class                         # 列全部 15 职业（PHB14 在前）
python query.py subclass --class 法师          # 8 学派；class 牧师 = 7 领域；warlock 统一为「邪术师」

# 种族 / 专长 / 装备（PHB14 基准）
python query.py race 半精灵                    # PHB14 含半精灵/半兽人（PHB24 已并掉，只在这查得到）
python query.py feat 警觉                       # PHB14 核心专长（2014 结构）
python query.py equip 长剑                      # PHB14 武器/护甲（列序异于 2024、无精通）
```

重建索引（资料库 md 改动后）：`python build_index.py`（自动定位 docs/extracted，重建全部 7 个索引）。

## 维护契约（与 dnd5r 的差异处）

**这套工具 = dnd5r 工具 + 2014 配置。** 从 dnd5r 同步更新时：`query.py` 可直接覆盖；`build_index.py` 须保留下列 dnd5e 专属改动：

1. **`PRIORITY`**：`玩家手册/城主指南/怪物图鉴` = 0，`玩家手册2024/城主指南2024/怪物图鉴2025` = 1（与 dnd5r 相反）。
2. **职业/子职**（`build_classes`）：基准 = `玩家手册/职业/<职业>/`（PHB14 = 子目录 + sibling 主文件，同奇械师布局），PHB24 当子职追加。两个坑：
   - PHB14 子职文件 frontmatter `name` 是**分类名**（武术范型/奥术传承）→ `_class_records(…, sub_name_from_file=True)` 取**文件名**当子职名（无单独 en）。
   - warlock：PHB14 目录是 `邪术师`、主文件 `name` 却是「魔契师」→ 职业名一律取目录名、`CLASS_NAME_ALIAS = {魔契师: 邪术师}`、`SUBCLASS_NOISE` 含「祈唤」排除「魔能祈唤」。
3. **种族**（`build_races`）：基准 = `玩家手册/种族/`；PHB14 race `name` 是「半精灵Half-Elf」中英黏一起 → `_race_record` 正则拆分；PHB24/剑湾/费资本当 `CROSS_RACE_SOURCES` 追加。
4. **专长**（`build_feats`）：`玩家手册/自定义选项/专长.md`（2014 结构）加进 `CROSS_FEAT_FILES`，走 `_feat_name_2014` 检测「中文 English」名行。
5. **装备**（`build_equipment`）：`玩家手册/装备/武器.md`（列序 `cost/damage/weight/properties`，**无精通列**）、`护甲与盾牌.md`（`cost/ac/strength/stealth/weight`）——列序与 2024 不同。

## 覆盖 + caveat
- **法术 1179 / 怪物 738+85族(CR 回填) / 魔法物品 331**：与 dnd5r **同内容**，仅优先级翻转。
- **职业 15 + 子职 207 / 种族 28 / 专长 140 / 装备 37武+18护**：PHB14 基准 + 各扩展追加。
- 每条记录带 `path:line` 指针（回源 md 引用 / 读全文）；索引**不含正文**，要全文按 path:line Read。
- 专长是**启发式解析**（PHB14/塔莎/费资本/珊娜萨 2014 结构无统一标记），少数 en 截断、长尾按 path:line 核对。
- **枚举默认只列官方书**（自动隐藏第三方/UA/模组，`--all` 全含、`--source <书>` 指定）；单条名字查询不受影响。
- 待接入：装备的工具/法器（魔法物品另有 `magic` 域）、设定书单文件多种族、谦卑林等无父职业信息的第三方子职。
- **不索引散文**（5237 个 `type:document` 规则/模组正文，永远 grep/read）；homebrew 不在索引，仍走 glob（规则 10）。
