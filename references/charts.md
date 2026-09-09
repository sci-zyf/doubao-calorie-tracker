# 图表模板（当日三图 + 历史看板）

本文件定义所有图表的输出规范。当日三图默认输出（用户明确不要图时跳过）；历史看板只在用户声明查看历史时才输出。

## 颜色规范（全部图表统一）

| 颜色 | 用途 |
|---|---|
| `#8BC8EA` 蓝 | 早餐 / 碳水 / 基础代谢 |
| `#E8906A` 橙 | 午餐 / 脂肪 / 已摄入 |
| `#A8D8A8` 绿 | 晚餐 / 蛋白质 / 热量缺口 |
| `#B39DDB` 紫 | 加餐 |
| `#C8CDD3` 灰 | 每日预期摄入 |
| `#D8DEE4` 浅灰 | 剩余（仅总进度图用） |
| `#5B8FF9` 蓝 | 身体档案：体重 |
| `#F6BD16` 黄 | 身体档案：BMI |
| `#B39DDB` 紫 / `#79C0C0` 青 | 身体档案：围度补充色 |

## 通用渲染约束

- ECharts option：自包含、无注释、无外部变量、无 `const option=`/`setOption`/HTML 包裹；代码块语言小写 `echarts`。
- 禁止箭头函数、`let`/`const`、模板字符串、可选链等现代语法；callback 用 ES5，访问参数前判空。
- tooltip 必须含 `triggerOn:'click'`、`renderMode:'richText'`、`confine:true`。
- 移动端按约 351×351dp 卡片规划；title 14-16px，legend/axisLabel 10-12px；柱状图 grid 必须 `containLabel:true` 且 `left≥44`。
- 图前用一句话说明数据口径。
- 区间统计（本周/昨日等）把标题中的"今日"改为对应区间，subtext 写清日期范围。

## 布局约束（避免元素重叠，所有直角坐标图必须遵守）

### 输出形态选择

| 场景 | 输出形态 | 理由 |
|---|---|---|
| 单张图、无需切换时间范围 | 内嵌 ```echarts 代码块 | 用户在对话中直接可见，无需点开链接 |
| 历史趋势（默认） | 内嵌 ```echarts（热量趋势 + 营养素趋势），图后提示可生成 HTML | 用户在对话中直接可见；HTML 仅在用户明确要求时生成 |
| 历史趋势（用户明确要求 HTML） | HTML 交互看板 | 控件和多图刷新需要 JS，echarts 代码块无法承载 |
| 当日三图（圆环+圆环+柱状） | 内嵌 ```echarts，逐张输出 | 单图无联动需求 |

> 判断口诀：**要切换就 HTML，不切换就内嵌。历史趋势默认内嵌，HTML 按需生成。**

### 四元素空间分配（title / legend / grid / dataZoom）

核心规则：**title 永远独占顶部；legend 和 dataZoom slider 不同侧。**

| 场景 | legend 位置 | dataZoom | grid.top | grid.bottom |
|---|---|---|---|---|
| 内嵌单图、无 slider | `bottom:2` | 仅 `inside` | 56~60 | 36 |
| HTML 看板、有 slider | `top:24` | `inside` + `slider(bottom:4,height:14)` | 52 | 30 |
| 双 y 轴图 | 同上 | 同上 | 同上 | 同上 |

- `grid.top` 必须 ≥ title 高度（约 36px，含 subtext）+ legend 高度（约 20px）+ 间距。
- `grid.bottom` 必须 ≥ 底部元素高度（legend 约 20px 或 slider 约 18px）+ xAxis 标签高度（约 16px）。
- 禁止 legend 和 dataZoom slider 同时放在底部或同时放在顶部。
- 饼图/圆环图无 grid，legend 默认 `bottom:0`，不受此表约束。

### 双 y 轴分隔线

- 凡双 y 轴图，**次轴（通常右轴）必须 `splitLine:{show:false}`**，只保留主轴一套水平分隔线。
- 主轴分隔线建议浅灰虚线：`splitLine:{lineStyle:{color:"#E5E7EB",type:"dashed"}}`。
- 原因：两轴刻度单位不同（如 % 和 g），默认各画一套横线会交错重叠。

### 缺失数据

- 无记录的日期填 `null`，柱不画、折线自动断段。
- **禁止 `connectNulls:true`**，不得用虚线伪连接缺失日。

---

## 一、当日三图（默认输出）

### 图1：各餐热量分布（圆环图）

早/中/晚/加餐四餐，单位 kcal；某餐为 0 时扇区不显示但 legend 保留，补记后自动更新。

```echarts
{
  backgroundColor: "transparent",
  title: {
    text: "今日各餐热量分布",
    subtext: "YYYY-MM-DD | 单位 kcal",
    left: "center",
    textStyle: { color: "#1A1B1C", fontSize: 15, fontWeight: 600 },
    subtextStyle: { color: "#6B7280", fontSize: 11 }
  },
  tooltip: {
    trigger: "item",
    triggerOn: "click",
    renderMode: "richText",
    confine: true,
    textStyle: { fontSize: 10, lineHeight: 14 },
    padding: [6, 8]
  },
  legend: {
    bottom: 0,
    itemWidth: 14,
    itemHeight: 8,
    textStyle: { color: "#6B7280", fontSize: 11 }
  },
  series: [
    {
      name: "热量",
      type: "pie",
      radius: ["42%", "68%"],
      center: ["50%", "52%"],
      avoidLabelOverlap: true,
      label: {
        formatter: "{b} {c} kcal",
        color: "#555",
        fontSize: 11
      },
      data: [
        { value: 早餐热量, name: "早餐", itemStyle: { color: "#8BC8EA" } },
        { value: 午餐热量, name: "午餐", itemStyle: { color: "#E8906A" } },
        { value: 晚餐热量, name: "晚餐", itemStyle: { color: "#A8D8A8" } },
        { value: 加餐热量, name: "加餐", itemStyle: { color: "#B39DDB" } }
      ]
    }
  ]
}
```

### 图2：三大营养素（圆环图）

碳水/蛋白质/脂肪，单位 g；营养素为估算值时在 subtext 标注"（估算）"，用户未提供时该图改为文字说明"营养素未提供"，不要编造。

```echarts
{
  backgroundColor: "transparent",
  title: {
    text: "今日三大营养素",
    subtext: "YYYY-MM-DD | 单位 g（估算）",
    left: "center",
    textStyle: { color: "#1A1B1C", fontSize: 15, fontWeight: 600 },
    subtextStyle: { color: "#6B7280", fontSize: 11 }
  },
  tooltip: {
    trigger: "item",
    triggerOn: "click",
    renderMode: "richText",
    confine: true,
    textStyle: { fontSize: 10, lineHeight: 14 },
    padding: [6, 8]
  },
  legend: {
    bottom: 0,
    itemWidth: 14,
    itemHeight: 8,
    textStyle: { color: "#6B7280", fontSize: 11 }
  },
  series: [
    {
      name: "营养素",
      type: "pie",
      radius: ["42%", "68%"],
      center: ["50%", "52%"],
      avoidLabelOverlap: true,
      label: {
        formatter: "{b} {c} g",
        color: "#555",
        fontSize: 11
      },
      data: [
        { value: 碳水合计g, name: "碳水", itemStyle: { color: "#8BC8EA" } },
        { value: 蛋白质合计g, name: "蛋白质", itemStyle: { color: "#A8D8A8" } },
        { value: 脂肪合计g, name: "脂肪", itemStyle: { color: "#E8906A" } }
      ]
    }
  ]
}
```

### 图3：热量缺口（柱状图 + 虚线参考线）

基础代谢 / 已摄入 / 基础代谢缺口 / 真实热量缺口，单位 kcal；每日预期摄入以灰色虚线作参考线。
- 基础代谢缺口 = 基础代谢 - 总热量（不含运动，为负表示已超基础代谢）
- 真实热量缺口 = 基础代谢 + 活动能量 - 总热量（含运动消耗，减脂期主要看这个）
- **活动能量为 0 时**：基础代谢缺口与真实热量缺口数值相等、两根柱子等高，属正常现象。此时在图下方加一句文字说明："今日未记录活动能量，真实热量缺口 = 基础代谢缺口。回复'跑步消耗XXX卡'可记录运动。"

```echarts
{
  backgroundColor: "transparent",
  title: {
    text: "今日热量缺口",
    subtext: "YYYY-MM-DD | 单位 kcal | 灰线=每日预期摄入",
    left: "center",
    textStyle: { color: "#1A1B1C", fontSize: 15, fontWeight: 600 },
    subtextStyle: { color: "#6B7280", fontSize: 11 }
  },
  tooltip: {
    trigger: "axis",
    triggerOn: "click",
    renderMode: "richText",
    confine: true,
    textStyle: { fontSize: 10, lineHeight: 14 },
    padding: [6, 8]
  },
  legend: {
    data: ["基础代谢", "已摄入", "基础代谢缺口", "真实热量缺口"],
    bottom: 2,
    itemWidth: 14,
    itemHeight: 8,
    textStyle: { color: "#6B7280", fontSize: 11 }
  },
  grid: {
    left: 44,
    right: 16,
    top: 60,
    bottom: 36,
    containLabel: true
  },
  xAxis: {
    type: "category",
    data: ["基础代谢", "已摄入", "基础代谢缺口", "真实热量缺口"],
    axisLabel: { color: "#555", fontSize: 11 }
  },
  yAxis: {
    type: "value",
    name: "kcal",
    axisLabel: { color: "#555", fontSize: 11 }
  },
  series: [
    {
      name: "热量",
      type: "bar",
      barWidth: "40%",
      label: {
        show: true,
        position: "top",
        color: "#555",
        fontSize: 11
      },
      markLine: {
        silent: true,
        symbol: "none",
        lineStyle: { color: "#C8CDD3", type: "dashed", width: 1.5 },
        label: { formatter: "预期摄入 {c} kcal", color: "#C8CDD3", fontSize: 10, position: "insideEndTop" },
        data: [{ yAxis: 每日预期摄入 }]
      },
      data: [
        { value: 基础代谢, itemStyle: { color: "#8BC8EA" } },
        { value: 总热量, itemStyle: { color: "#E8906A" } },
        { value: 基础代谢缺口, itemStyle: { color: "#A8D8A8" } },
        { value: 真实热量缺口, itemStyle: { color: "#B39DDB" } }
      ]
    }
  ]
}
```

---

## 二、历史看板（仅用户声明时输出）

### 触发

- 用户说"看历史 / 趋势 / 回顾 / 最近X周月年 / 体重趋势 / 营养素变化"等，输出本看板。
- 默认不输出；现有轻量区间查询（本周/昨日文字+三图）继续保留。

### 数据来源

一次查询拉取，均在「每日汇总」表按日期范围过滤（`+record-list --filter-json`，datetime 区间不支持 `>=`/`<=`，用 `>`/`<` + `ExactDate(边界前一天/后一天)`）：
- 热量趋势：总热量、每日预期摄入、基础代谢、活动能量、基础代谢缺口、真实热量缺口、日期
- 营养素趋势：每日碳水、每日蛋白质、每日脂肪（占比与总质量由表格公式已算好，直接读）
- 身体档案：从「身体档案」表按日期范围拉取 体重、BMI、BMI分类、六围度

### 输出形态

`html type="renderer"` 交互页，结构：

- 首块外层 `<html style="margin:0;padding:0;">` + 透明 div；不用 DOCTYPE/head/body；CSS 全部内联；根容器自然撑高，禁止 `100vh`/`height:100%`。
- 脚本用 IIFE + try/catch + DOM 判空，不监听 `DOMContentLoaded`。
- **顶部控件区**（横向弹性换行）：
  - 粒度 select：日 / 周 / 月 / 年
  - 范围 select（快捷）：本周 / 上周 / 本月 / 近30天 / 近90天 / 近1年 / 自定义
  - 自定义时显示起止日期两个 `<input type="date">`
  - 控件高度 ≥ 44px，支持 click/tap
- **粒度与范围的语义**：粒度与范围只决定"展示哪一段日期的逐日数据"，**不对数据做任何聚合**；范围越大图越密（年=365 个点）。
- 默认初始：粒度=月、范围=近30天，初始状态必须表达主结论。
- 三个区块自上而下：热量趋势图、营养素趋势图、身体档案图（图A+图B）。切换控件后全部图表联动刷新。
- 每次刷新即时更新图形，并显示当前选中的区间文字（如"2026-08-10 ~ 2026-09-08 · 逐日"）。
- 每张图容器设置明确局部高度（如 280px），直接父元素 `min-width:0`；`window` resize 时 `chart.resize()`。
- ECharts 内嵌于 renderer：tooltip `confine:true`；直角坐标 `grid.containLabel:true`；数值统一精度单位。
- 外部 ECharts 库加载失败时显示静态文字说明，不能空白。

### 区块1：热量趋势图（三条折线）

- x 轴：日期（category，按升序）
- 系列：每日总热量（#E8906A）、每日预期摄入（#C8CDD3）、基础代谢（#8BC8EA）
- 单 y 轴 kcal，`grid.containLabel:true`
- 必须开启 `dataZoom`（inside + slider 至少一种），初始完整展示所选范围
- 缺失日：该日三个系列都填 `null`（不连线、自动断段），**禁止 `connectNulls:true`**

### 区块2：营养素趋势图（双轴）

- 第一 y 轴（左，%）：100% 堆叠柱。每天一根柱子，三段 value 为当天 碳水占比/蛋白质占比/脂肪占比（表格公式已算，0~1，填图时乘 100），`stack: "ratio"`，yAxis max=100，三色：#8BC8EA / #A8D8A8 / #E8906A。柱高一致=100%。
- 第二 y 轴（右，g）：总质量折线（#5B8FF9）。
- tooltip 同时给出各营养素克数、占比和总质量。
- `dataZoom` 同步作用于两轴数据。
- **缺失规则**：当天完全无记录 → 柱与折线该日都 null（柱不画、线断段）；当天有热量但营养素记录不全（判定标准：当天总热量 > 0，但碳水、蛋白质、脂肪三项中任意一项为空或 0）→ 该日柱与总质量折线整日留空，图上方文字注明"X月X日 营养素记录不全"，热量图不受影响。禁止 `connectNulls:true`。

### 区块3：身体档案图

- **图A 体重+BMI 双轴折线**：第一轴 体重 kg（#5B8FF9），第二轴 BMI 无单位（#F6BD16，yAxis min 建议 15、max 25 便于观察波动）；tooltip 同时显示 BMI 分类；`dataZoom`。
- **图B 六项围度折线**：胸围 #8BC8EA、腰围 #E8906A、臀围 #A8D8A8、大腿围 #C8CDD3、手臂围 #B39DDB、小腿围 #79C0C0，单轴 cm；`legend.type:'scroll'`，点击图例可显隐某条线。
- 身体数据天然不连续：未测的日子该系列填 `null` 自动断段，禁止 `connectNulls:true`；允许部分填写（只测腰围就只画腰围）。
- 图上方文字显示当前区间内最新一条的 BMI 与分类（如"最新 BMI 21.8 · 正常"）。

### 历史看板数据口径提示

- 基础代谢缺口 = 基础代谢 - 总热量（负=已超基础代谢）；真实热量缺口 = 基础代谢 + 活动能量 - 总热量（含运动，减脂期主要看这个）；摄入余量 = 每日预期摄入 - 总热量（负=已超标）。
- 活动能量由活动记录表按日期聚合，无记录时为 0，此时真实热量缺口 = 基础代谢缺口。
- 营养素占比合计恒为 100%；当日三项之和为 0（无任何营养素记录）时占比留空。
- 粒度/范围变化不改变数据本身，只改变展示窗口；标题 subtext 写清日期范围与"逐日"。
