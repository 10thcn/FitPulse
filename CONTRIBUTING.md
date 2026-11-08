# 🤝 FitPulse 团队成员协作指南

> **目标**：每位团队成员 Fork 本仓库，完成自己的成果提交，形成可量化的团队协作证据。

## 队员操作步骤（每人约 5 分钟）

### 第 1 步：Fork 仓库

1. 打开 https://github.com/power819/FitPulse
2. 点击右上角 **Fork** 按钮
3. 保持仓库名 `FitPulse`，点击 **Create fork**

### 第 2 步：添加你的成果

在你的 Fork 仓库中，根据你的角色添加对应文件：

| 角色 | 成员 | 需要添加的文件/内容 |
|---|---|---|
| 产品经理 | 陈加豪 | 用户画像、PRD 补充、需求细化说明 |
| 领域知识研究员 A | 林复彬 | 标准动作知识条目（Markdown + 图片）（后台知识库部分） |
| 领域知识研究员 B | 陈辉腾 | 肌肉发力+错误姿势知识条目（Markdown + 图片）（后台知识库部分） |
| Agent 提示词工程师 | 罗堉楠 | 三大场景提示词设计文档 |
| RAG/知识库工程师 | 马锦城 | 知识库结构设计、Chroma 配置方案 |
| 后端/API 集成 | 徐心铭 | API 接口定义（OpenAPI/Swagger 草稿） |
| 前端工程师 | 张智轩 | 前端组件设计稿、页面结构说明 |
| 工具调用/流程编排 | 林祺皓 | 工具 Schema 定义、Agent 流程编排设计 |
| 测试工程师 | 连向薇 | 测试用例列表、三组典型案例对比方案 |
| UI/UX 设计+文档 | 李江炜 | 设计稿说明、原型设计思路补充 |
| 队长/统筹 | 罗炜 | 主仓库维护、PR Review、合并管理 |

### 第 3 步：发起 Pull Request

1. 进入你的 Fork 仓库页面
2. 点击 **Contribute** → **Open Pull Request**
3. 标题格式：`[角色名] 提交XX成果`（如：`[提示词工程师] 三大场景提示词设计`）
4. 描述你的成果内容，点击 **Create pull request**

---

## 协作流程示意

```
power819/FitPulse (主仓库)
    │
    ├── Fork → 陈加豪/FitPulse ──PR──→ 用户画像+PRD补充
    ├── Fork → 林复彬/FitPulse ──PR──→ 标准动作知识条目
    ├── Fork → 陈辉腾/FitPulse ──PR──→ 错误姿势知识条目
    ├── Fork → 罗堉楠/FitPulse ──PR──→ 提示词设计文档
    ├── Fork → 马锦城/FitPulse ──PR──→ 知识库结构设计
    ├── Fork → 徐心铭/FitPulse ──PR──→ API接口定义
    ├── Fork → 张智轩/FitPulse ──PR──→ 前端组件设计
    ├── Fork → 林祺皓/FitPulse ──PR──→ 工具Schema设计
    ├── Fork → 连向薇/FitPulse ──PR──→ 测试用例
    └── Fork → 李江炜/FitPulse ──PR──→ 设计稿说明
```

---

## 📸 协作截图取证清单

Fork+PR 流程完成后，以下页面可供截图作为「团队协作记录」证据：

| 截图内容 | 链接 | 证明力 |
|---|---|---|
| Fork Network 图 | https://github.com/power819/FitPulse/network/members | ⭐⭐⭐ 显示所有队员 Fork |
| Pull Requests 列表 | https://github.com/power819/FitPulse/pulls | ⭐⭐⭐ 显示所有人提交的 PR |
| Insights → Contributors | https://github.com/power819/FitPulse/graphs/contributors | ⭐⭐ 显示贡献者活跃度 |
| 单个 PR 详情页 | 点击任意 PR | ⭐⭐ 显示讨论记录 |
| Commits 历史 | https://github.com/power819/FitPulse/commits/main | ⭐⭐ 显示提交频率 |

---

## ⚡ 后续工作流（第9周起）

正式写代码后，团队协作流程升级为：

1. 每个 sprint 开始 → 从主仓库 **Sync fork** 拉取最新代码
2. 在自己的 fork 中开发 → commit → push
3. 发起 PR 到主仓库 → 队长 Code Review → Merge
4. 每周末统计 PR 数量作为协作量化指标

---

> 💡 **协作截图建议**：等至少 5+ 位队员完成 Fork 和 PR 后，截取 Fork Network 页面 + PR 列表页面，这就是最有力的「团队协作过程」证据。