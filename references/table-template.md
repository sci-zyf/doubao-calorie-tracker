# 建表模板（首次使用引导专用）

首次使用本技能时，若 `config.json` 不存在或 `initialized=false`，按本模板自动创建四张表。

## 建表顺序（必须按此顺序，formula 依赖前表存在）

1. `+base-create` 创建 Base，初始表为「饮食记录」（schema 见下）
2. `+table-create` 在 Base 内创建「每日汇总」（schema 见下，依赖饮食记录表）
3. `+table-create` 在 Base 内创建「身体档案」（schema 见下）
4. `+table-create` 在 Base 内创建「活动记录」（schema 见下）
5. 从各命令返回值中提取 base_token 和四个 table_id，写入 config.json
6. 询问用户身高、出生日期、性别、基础代谢、每日预期摄入，写入 config.json 的 profile 并置 `initialized: true`
7. 提示初始化完成

所有 `lark-cli base` 命令必须加 `--as user`。建表时通过 `--fields` 一次性传入所有字段（含 formula），**无需** `--i-have-read-guide`；该参数仅在后续单独用 `+field-create` 创建 formula/lookup 字段时才需要。

## 1. 饮食记录表（主字段：食物描述）

```json
[
  {"type": "text", "name": "食物描述"},
  {"type": "datetime", "name": "日期", "style": {"format": "yyyy-MM-dd"}},
  {"type": "select", "name": "餐次", "multiple": false, "options": [
    {"name": "早餐", "hue": "Yellow"},
    {"name": "午餐", "hue": "Orange"},
    {"name": "晚餐", "hue": "Blue"},
    {"name": "加餐", "hue": "Green"}
  ]},
  {"type": "number", "name": "热量(kcal)", "style": {"type": "plain", "precision": 0}},
  {"type": "number", "name": "碳水(g)", "style": {"type": "plain", "precision": 1}},
  {"type": "number", "name": "蛋白质(g)", "style": {"type": "plain", "precision": 1}},
  {"type": "number", "name": "脂肪(g)", "style": {"type": "plain", "precision": 1}},
  {"type": "text", "name": "备注"}
]
```

## 2. 每日汇总表（主字段：日期，含热量/营养素计算字段）

```json
[
  {"type": "datetime", "name": "日期", "style": {"format": "yyyy-MM-dd"}},
  {"type": "number", "name": "基础代谢", "style": {"type": "plain", "precision": 0}},
  {"type": "number", "name": "每日预期摄入", "style": {"type": "plain", "precision": 0}},
  {"type": "formula", "name": "活动能量", "expression": "IF([活动记录].COUNTIF(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"))=0,0,[活动记录].FILTER(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\")).[消耗热量].LISTCOMBINE().SUM())"},
  {"type": "formula", "name": "早餐热量", "expression": "IF([饮食记录].COUNTIF(AND(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"),CurrentValue.[餐次]=\"早餐\"))=0,0,[饮食记录].FILTER(AND(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"),CurrentValue.[餐次]=\"早餐\")).[热量(kcal)].LISTCOMBINE().SUM())"},
  {"type": "formula", "name": "午餐热量", "expression": "IF([饮食记录].COUNTIF(AND(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"),CurrentValue.[餐次]=\"午餐\"))=0,0,[饮食记录].FILTER(AND(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"),CurrentValue.[餐次]=\"午餐\")).[热量(kcal)].LISTCOMBINE().SUM())"},
  {"type": "formula", "name": "晚餐热量", "expression": "IF([饮食记录].COUNTIF(AND(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"),CurrentValue.[餐次]=\"晚餐\"))=0,0,[饮食记录].FILTER(AND(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"),CurrentValue.[餐次]=\"晚餐\")).[热量(kcal)].LISTCOMBINE().SUM())"},
  {"type": "formula", "name": "加餐热量", "expression": "IF([饮食记录].COUNTIF(AND(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"),CurrentValue.[餐次]=\"加餐\"))=0,0,[饮食记录].FILTER(AND(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"),CurrentValue.[餐次]=\"加餐\")).[热量(kcal)].LISTCOMBINE().SUM())"},
  {"type": "formula", "name": "总热量", "expression": "[早餐热量]+[午餐热量]+[晚餐热量]+[加餐热量]"},
  {"type": "formula", "name": "摄入余量", "expression": "[每日预期摄入]-[总热量]"},
  {"type": "formula", "name": "基础代谢缺口", "expression": "[基础代谢]-[总热量]"},
  {"type": "formula", "name": "真实热量缺口", "expression": "[基础代谢]+[活动能量]-[总热量]"},
  {"type": "formula", "name": "每日碳水", "expression": "IF([饮食记录].COUNTIF(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"))=0,0,[饮食记录].FILTER(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\")).[碳水(g)].LISTCOMBINE().SUM())"},
  {"type": "formula", "name": "每日蛋白质", "expression": "IF([饮食记录].COUNTIF(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"))=0,0,[饮食记录].FILTER(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\")).[蛋白质(g)].LISTCOMBINE().SUM())"},
  {"type": "formula", "name": "每日脂肪", "expression": "IF([饮食记录].COUNTIF(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\"))=0,0,[饮食记录].FILTER(TEXT(CurrentValue.[日期],\"yyyy-MM-dd\")=TEXT([日期],\"yyyy-MM-dd\")).[脂肪(g)].LISTCOMBINE().SUM())"},
  {"type": "formula", "name": "总质量", "expression": "[每日碳水]+[每日蛋白质]+[每日脂肪]"},
  {"type": "formula", "name": "碳水占比", "expression": "IF([总质量]=0,\"\",[每日碳水]/[总质量])"},
  {"type": "formula", "name": "蛋白质占比", "expression": "IF([总质量]=0,\"\",[每日蛋白质]/[总质量])"},
  {"type": "formula", "name": "脂肪占比", "expression": "IF([总质量]=0,\"\",[每日脂肪]/[总质量])"}
]
```

## 3. 身体档案表（主字段：日期）

```json
[
  {"type": "datetime", "name": "日期", "style": {"format": "yyyy-MM-dd"}},
  {"type": "number", "name": "身高", "style": {"type": "plain", "precision": 1}},
  {"type": "number", "name": "体重", "style": {"type": "plain", "precision": 1}},
  {"type": "formula", "name": "BMI", "expression": "ROUND([体重]/POWER([身高]/100,2),1)"},
  {"type": "formula", "name": "BMI分类", "expression": "IFERROR(IF(ISBLANK([体重]),\"\",IF(ISBLANK([身高]),\"\",IF([BMI]<18.5,\"偏瘦\",IF([BMI]<24,\"正常\",IF([BMI]<28,\"超重\",\"肥胖\"))))),\"\")"},
  {"type": "number", "name": "胸围", "style": {"type": "plain", "precision": 1}},
  {"type": "number", "name": "腰围", "style": {"type": "plain", "precision": 1}},
  {"type": "number", "name": "臀围", "style": {"type": "plain", "precision": 1}},
  {"type": "number", "name": "大腿围", "style": {"type": "plain", "precision": 1}},
  {"type": "number", "name": "手臂围", "style": {"type": "plain", "precision": 1}},
  {"type": "number", "name": "小腿围", "style": {"type": "plain", "precision": 1}},
  {"type": "text", "name": "备注"}
]
```

## 4. 活动记录表（主字段：运动项目，存每次运动明细）

```json
[
  {"type": "text", "name": "运动项目"},
  {"type": "datetime", "name": "日期", "style": {"format": "yyyy-MM-dd"}},
  {"type": "number", "name": "消耗热量", "style": {"type": "plain", "precision": 0}},
  {"type": "text", "name": "备注"}
]
```

## 说明

- 主字段必须是每个 JSON 数组的第一项（`+base-create`/`+table-create` 强制）。
- 每日汇总表的餐次热量、每日营养素、活动能量均为 formula 字段：用 `COUNTIF` 判断当天是否有对应记录，无记录时返回 `0`（而非空字符串，确保后续 `总热量`/`总质量`/`真实热量缺口` 等求和字段计算正确），有记录时用 `FILTER+SUM` 聚合；餐次热量额外带餐次条件。日期格式统一使用小写 `yyyy-MM-dd`（大写 `YYYY` 为 ISO 周年，跨年周会导致匹配错误）。
- 活动能量由「活动记录」表按日期自动聚合，**不要在每日汇总表手动写活动能量**；记录运动时写入「活动记录」表，每次一条，支持一天多次累加。
- 公式字段由表内自动计算，写入记录时只写存储字段（日期、基础代谢、每日预期摄入等），不写 formula。
- 基础代谢和每日预期摄入无默认值，初始化时必须由用户填写（附男女参考值）。
- 热量缺口分两个：基础代谢缺口 = 基础代谢 - 总热量（不含运动）；真实热量缺口 = 基础代谢 + 活动能量 - 总热量（含运动）。摄入余量 = 每日预期摄入 - 总热量。
- 初始化完成后，删除本引导过程中可能产生的任何测试数据（删除前向用户确认）。
- **性能注意**：每日汇总表的 formula 字段会对饮食记录表和活动记录表做全表 `FILTER`/`COUNTIF`，每条汇总记录独立计算。当饮食记录超过 3000 条时，打开汇总表或写入新记录后的公式重算可能明显变慢。若遇到性能问题，可考虑：① 将 formula 聚合改为 `lookup + rollup` 字段（需先在饮食记录表/活动记录表建立关联到每日汇总的 link 字段）；② 或在写入记录时由 agent 增量更新汇总表的存储字段，移除 formula 字段。
