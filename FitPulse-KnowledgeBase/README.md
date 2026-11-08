# FitPulse 标准动作知识库（后台）

> 版本：`1.0.0` · 更新日期：`2026-11-08` · 语言：简体中文  
> 用途：为 FitPulse 的「动作查询」「计划生成」「动作纠错」提供结构化、可检索、可追溯的标准动作知识。

## 1. 本批次交付内容

本批次覆盖居家新手最常用的 **12 个动作、7 大动作类别（9 种编排模式）**，每个动作包含：

- 1 份 RAG 友好的 Markdown 条目（YAML frontmatter + 固定章节）；
- 1 张原创双栏示意图（SVG，起始位 + 标准位，含关键对齐提示）；
- 可批量入库的 [`manifest.json`](manifest.json)（它是快速加载的摘要索引；完整字段以 Markdown frontmatter 为准）；
- 集中式来源说明 [`SOURCES.md`](SOURCES.md)。

| ID | 动作 | 模式 | 难度 | 器械 | 条目 | 示意图 |
|---|---|---|---|---|---|---|
| EX-001 | 自重深蹲 | 蹲 | 入门 | 无 | [打开](exercises/01-bodyweight-squat.md) | [查看](assets/exercises/01-bodyweight-squat.svg) |
| EX-002 | 臀桥 | 髋主导 | 入门 | 无 | [打开](exercises/02-glute-bridge.md) | [查看](assets/exercises/02-glute-bridge.svg) |
| EX-003 | 后撤弓步蹲 | 单腿 | 进阶 | 无 | [打开](exercises/03-reverse-lunge.md) | [查看](assets/exercises/03-reverse-lunge.svg) |
| EX-004 | 俯卧撑（含跪姿退阶） | 水平推 | 进阶 | 无 | [打开](exercises/04-push-up.md) | [查看](assets/exercises/04-push-up.svg) |
| EX-005 | 前臂平板支撑 | 核心抗伸展 | 入门 | 无 | [打开](exercises/05-forearm-plank.md) | [查看](assets/exercises/05-forearm-plank.svg) |
| EX-006 | 死虫式 | 核心抗伸展 | 入门 | 无 | [打开](exercises/06-dead-bug.md) | [查看](assets/exercises/06-dead-bug.svg) |
| EX-007 | 鸟狗式 | 核心抗旋转 | 入门 | 无 | [打开](exercises/07-bird-dog.md) | [查看](assets/exercises/07-bird-dog.svg) |
| EX-008 | 哑铃单臂俯身划船 | 水平拉 | 进阶 | 哑铃 + 稳固椅子 | [打开](exercises/08-dumbbell-row.md) | [查看](assets/exercises/08-dumbbell-row.svg) |
| EX-009 | 哑铃罗马尼亚硬拉 | 髋主导 | 进阶 | 哑铃 | [打开](exercises/09-dumbbell-rdl.md) | [查看](assets/exercises/09-dumbbell-rdl.svg) |
| EX-010 | 哑铃肩上推举 | 垂直推 | 进阶 | 哑铃 | [打开](exercises/10-dumbbell-shoulder-press.md) | [查看](assets/exercises/10-dumbbell-shoulder-press.svg) |
| EX-011 | 靠墙静蹲 | 蹲（等长） | 入门 | 无 + 平整墙面 | [打开](exercises/11-wall-sit.md) | [查看](assets/exercises/11-wall-sit.svg) |
| EX-012 | 开合跳 | 心肺 / 弹跳 | 入门 | 无 | [打开](exercises/12-jumping-jack.md) | [查看](assets/exercises/12-jumping-jack.svg) |

## 2. 目录结构

```text
FitPulse-KnowledgeBase/
├── README.md                 # 本文件：规范、入库方式、验收标准
├── SOURCES.md                # 集中式知识来源与引用说明
├── manifest.json             # 12 个条目的机器可读索引
├── exercises/                # 12 份动作 Markdown
│   ├── 01-bodyweight-squat.md
│   └── ...
├── assets/exercises/         # 12 张原创 SVG 示意图
│   ├── 01-bodyweight-squat.svg
│   └── ...
└── templates/
    └── exercise-template.md  # 新增动作时复制的模板
```

建议在仓库中的落点：

```text
backend/rag/knowledge_base/       # 本目录整体复制到这里
```

如果后端约定是 `backend/rag/data/`，只改目录名即可；Markdown 内图片使用相对路径，不依赖绝对路径。

## 3. Markdown 条目规范

每个条目由两段组成：

1. **YAML frontmatter**：供数据库、Chroma metadata、计划生成器使用；
2. **固定 H2 章节**：供 RAG 切块、动作查询和引用展示使用。

### 3.1 frontmatter 字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `id` | string | 是 | 稳定主键，格式 `EX-001`，不可随文件名变化 |
| `slug` | string | 是 | URL / 文件名用英文短横线标识 |
| `name` | string | 是 | 中文标准动作名 |
| `aliases` | string[] | 是 | 检索别名、俗称、英文名 |
| `category` | string | 是 | 展示分类，如“下肢力量” |
| `movement_pattern` | string | 是 | 计划编排用模式：`squat` / `hinge` / `lunge` / `horizontal_push` / `horizontal_pull` / `vertical_push` / `core_anti_extension` / `core_anti_rotation` / `cardio` |
| `difficulty` | string | 是 | `入门` / `进阶` / `挑战` |
| `equipment` | string[] | 是 | 所需器械；无器械写 `["无"]` |
| `primary_muscles` | string[] | 是 | 主要目标肌群 |
| `secondary_muscles` | string[] | 是 | 协同肌群 |
| `impact_level` | string | 是 | `低` / `中` / `中高`，供计划筛选用 |
| `noise_level` | string | 是 | `低` / `中`，供宿舍场景筛选用 |
| `space_requirement` | string | 是 | `小` / `中` |
| `default_sets` | integer | 是 | 新手默认组数 |
| `default_reps` | string | 是 | 新手默认次数或时长 |
| `default_rest_seconds` | integer | 是 | 组间休息秒数 |
| `contraindications` | string[] | 是 | 需要先评估或避免的情况，不是诊断标准 |
| `tags` | string[] | 是 | 检索标签 |
| `image` | string | 是 | 相对 Markdown 文件的图片路径 |
| `image_alt` | string | 是 | 图片替代文本，前端无障碍展示用 |
| `source_ids` | string[] | 是 | 对应 `SOURCES.md` 的来源编号 |
| `version` | string | 是 | 条目版本 |
| `last_reviewed` | string | 是 | 最近复核日期，`YYYY-MM-DD` |

### 3.2 固定章节

```text
## 一句话要点
## 目标肌群与发力感
## 器械、场地与适用人群
## 标准动作
### 起始位
### 动作过程
### 结束与节奏
## 呼吸
## 关键对齐要点
## 常见错误与纠正
## 新手组次建议
## 降阶与进阶
## 安全提示
## 常见问题
## 知识出处
```

固定章节的好处：

- 按 H2 切块后，每一块都能独立回答一类问题；
- “常见错误与纠正”可直接作为动作纠错工具的检索上下文；
- “新手组次建议”和 frontmatter 可共同喂给计划生成工具；
- “知识出处”保证回答可追溯，避免模型编造来源。

## 4. 建议的入库方式（Chroma）

### 4.1 解析与切块

建议按 H2 切块，而不是按固定字符数硬切。每个 chunk 前拼接动作名和别名，避免“起始位”“呼吸”这类短标题失去上下文。

示例 chunk 文本：

```text
动作：自重深蹲（别名：深蹲、徒手深蹲、Bodyweight Squat）
章节：常见错误与纠正
内容：膝盖内扣——下蹲或站起时膝盖向中间靠拢……
```

### 4.2 推荐 metadata

Chroma 的 metadata 只接受标量值。数组字段请用中文逗号拼接为字符串，例如 `"股四头肌,臀大肌"`。

```json
{
  "doc_type": "exercise",
  "exercise_id": "EX-001",
  "slug": "bodyweight-squat",
  "name": "自重深蹲",
  "aliases": "深蹲,徒手深蹲,Bodyweight Squat",
  "category": "下肢力量",
  "movement_pattern": "squat",
  "difficulty": "入门",
  "equipment": "无",
  "primary_muscles": "股四头肌,臀大肌",
  "impact_level": "低",
  "noise_level": "低",
  "section": "常见错误与纠正",
  "source_ids": "S1,S3,S4,S5",
  "image_path": "assets/exercises/01-bodyweight-squat.svg",
  "version": "1.0",
  "last_reviewed": "2026-11-08"
}
```

### 4.3 图片处理

- 源文件是 SVG，适合网页直接展示，也方便后续导出 PNG。
- 入库时建议上传到 MinIO，例如对象键 `knowledge/exercises/01-bodyweight-squat.svg`，再把可访问 URL 写入业务表；不要把本地相对路径直接暴露给前端。
- 若前端或图片审核链路只接受 PNG，可用 `rsvg-convert`、Inkscape 或浏览器截图将 SVG 转为 2x PNG；保留 SVG 作为源文件。
- 图片是**动作示意**，不用于医学诊断，也不替代真人教练的现场评估。

### 4.4 建议的检索策略

- 动作查询：先按 `name` / `aliases` 精确匹配，再做向量检索；
- 计划生成：先按 `equipment`、`difficulty`、`movement_pattern`、`impact_level` 过滤，再取“新手组次建议”段落；
- 动作纠错：检索“关键对齐要点”+“常见错误与纠正”+ 参考图，不直接把整篇条目塞进上下文；
- 回答中必须带 `source_ids`，前端展示为“知识出处”。

## 5. 内容口径与安全边界

- 面向**健康成人 / 大学生新手**的居家健身教学，不构成医疗诊断或个体化康复处方。
- 出现胸痛、晕厥、明显呼吸困难、急性关节肿胀、夜间痛、进行性麻木或无力等情况，应停止运动并就医。
- 条目中的“禁忌”是提醒先评估，不是让 Agent 给用户下诊断。Agent 应使用“建议先咨询医生/康复师”这类措辞。
- 不建议用“膝盖绝不能超过脚尖”“平板支撑越久越好”“练臀必须夹紧到腰痛”等绝对化说法。
- 训练剂量是新手起点，不是必须完成的目标；动作质量优先于次数和时长。
- 疼痛感与肌肉酸胀要区分：尖锐痛、关节痛、放射痛应停止；延迟性肌肉酸痛可观察。

## 6. 入库前验收清单

- [ ] 12 个 `id` 唯一，且与 `manifest.json` 一致；
- [ ] frontmatter 可被 YAML 解析器读取；
- [ ] 每个条目都包含固定 H2 章节；
- [ ] Markdown 内图片相对路径存在；
- [ ] SVG 包含 `<title>` 与 `<desc>`，可被读屏软件识别；
- [ ] 每个条目的 `source_ids` 都能在 `SOURCES.md` 找到；
- [ ] “常见错误与纠正”至少 4 条，“关键对齐要点”至少 4 条；
- [ ] 计划生成所需的 `equipment`、`movement_pattern`、`default_*` 字段完整；
- [ ] 不含绝对化医疗断言，不含未标注来源的“研究证明”；
- [ ] 图片为原创示意图，未使用未授权网图。

## 7. 如何新增动作

1. 复制 [`templates/exercise-template.md`](templates/exercise-template.md)；
2. 分配下一个 `EX-0xx`，更新 `slug`、文件名、图片名；
3. 按固定章节补全内容，至少给出 4 条对齐要点和 4 条常见错误；
4. 在 `SOURCES.md` 中确认来源编号存在；
5. 在 `manifest.json` 的 `entries` 数组追加一条；
6. 更新 `README.md` 的目录表与版本号；
7. 按第 6 节清单验收。

## 8. 版权与引用

- 文字条目由 FitPulse 团队依据公开教材、专业机构动作库与教学共识整理，**不是原文复制**；引用来源见 [`SOURCES.md`](SOURCES.md)。
- 12 张 SVG 示意图为本项目原创绘制，仅用于课程项目展示与教学示意；如需对外商用，请另行确认字体、图形素材的许可。
- 建议在应用内展示：“内容用于健身教学参考，不能替代医生或线下教练的个体化评估。”

---

维护人：FitPulse 领域知识组 · 复核人：待团队指定 · 下一批复核：动作纠错上线前
