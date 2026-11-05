# FitPulse —— 健身动作矫正 & 居家锻炼智能 Agent

> 🏋️ 面向学生健身新手的「随身智能教练」—— 用 AI 把专业教练的动作指导、训练规划和纠错反馈装进你的手机。

**2026秋软工实践团队项目** | 团队 FitPulse | 11 人

---

## 📌 项目简介

越来越多大学生开始居家健身，但普遍存在 **「不会练、怕练错、请不起私教」** 三大障碍。调研显示：88% 的人不知道动作是否标准，79% 担心动作错误导致受伤。

FitPulse 基于 **Claude 大模型 API**，通过提示词设计、知识检索（RAG）、工具调用与流程编排，打造一个能 **文字+图片指导、训练计划生成、动作纠错** 的领域智能体。

### 三大核心能力

| 能力 | 说明 |
|---|---|
| 🔍 **动作查询** | 查询居家常用动作的标准要领 + 配图 + 目标肌群 + 常见错误，标注知识出处 |
| 📋 **计划生成** | 根据用户水平/目标/器械/时间，生成结构化周训练计划（动作+组数+次数+休息） |
| 🩺 **动作纠错** | 上传动作照片，Agent 分析姿势偏差，给出定位+描述+纠正+参考图的反馈 |

---

## 🏗️ 技术架构

```
用户（响应式网页）
    │
    ▼
API 网关（Nginx + JWT）
    │
    ▼
业务编排层（FastAPI + LangChain）
  ├── 意图识别
  ├── 上下文管理
  ├── 工具调度
  └── 响应生成
    │
    ├── LLM API（Claude）
    ├── RAG 知识检索（Chroma + Claude Embedding）
    └── 工具集（动作查询/计划生成/动作纠错）
    │
    ▼
数据持久层
  ├── MySQL 8.0（业务数据）
  ├── Chroma（知识向量）
  └── MinIO（文件存储）
```

### 技术栈

| 层 | 技术 |
|---|---|
| 前端 | React 18 + TailwindCSS + TypeScript |
| 后端 | Python 3.11 + FastAPI |
| Agent 编排 | LangChain / LlamaIndex |
| LLM | Claude API (Anthropic) |
| 数据库 | MySQL 8.0 + Chroma（向量） |
| 对象存储 | MinIO (S3-compatible) |
| 部署 | Docker + Docker Compose |

---

## 🚀 快速开始

### 前置条件

- Docker & Docker Compose
- Claude API Key（[获取](https://console.anthropic.com/)）

### 一键部署

```bash
# 1. 克隆仓库
git clone https://github.com/power819/FitPulse.git
cd FitPulse

# 2. 配置环境变量
cp .env.example .env
# 编辑 .env，填入 ANTHROPIC_API_KEY 等

# 3. 启动所有服务
docker compose up -d

# 4. 初始化知识库
docker compose exec backend python scripts/init_knowledge_base.py

# 5. 访问
# 前端: http://localhost
# API 文档: http://localhost:8000/docs
```

---

## 👥 团队成员

| 学号 | 姓名 | 角色 |
|---|---|---|
| 102401317 | 罗炜 | 队长 / 项目统筹 |
| 102401529 | 陈加豪 | 产品经理 / 需求负责人 |
| 102401527 | 林复彬 | 领域知识研究员 A（标准动作） |
| 102401521 | 陈辉腾 | 领域知识研究员 B（肌肉发力+错误姿势） |
| 102401218 | 罗堉楠 | Agent 提示词工程师 |
| 102401328 | 马锦城 | RAG / 知识库工程师 |
| 102401531 | 徐心铭 | 后端 / API 集成工程师 |
| 102401121 | 张智轩 | 前端工程师 |
| 102401119 | 林祺皓 | 工具调用 / 流程编排工程师 |
| 102401203 | 连向薇 | 测试工程师 |
| 102401222 | 李江炜 | UI/UX 设计师 + 文档 |

---

## 📂 项目结构

```
FitPulse/
├── docker-compose.yml
├── .env.example
├── README.md
├── docs/
│   ├── FitPulse_系统设计说明书.md
│   ├── FitPulse_数据库设计说明书.md
│   └── FitPulse_需求分析文档.md
├── frontend/          # React 前端
├── backend/           # FastAPI 后端 + Agent 编排
│   ├── orchestrator/  # Agent 编排引擎
│   ├── rag/           # 知识检索模块
│   ├── tools/         # 工具集
│   ├── models/        # ORM 模型
│   └── services/      # 业务服务
├── admin/             # 后台管理
└── scripts/           # 初始化脚本
```

---

## 📖 文档

- [需求分析文档](docs/FitPulse_需求分析文档.md)
- [系统设计说明书](docs/FitPulse_系统设计说明书.md)
- [数据库设计说明书](docs/FitPulse_数据库设计说明书.md)

---

## 📅 开发计划

| 阶段 | 时间 | 里程碑 |
|---|---|---|
| 第9周 | 11.10–11.16 | 环境搭建与知识库整理 |
| 第10周 | 11.17–11.23 | Agent 核心开发 |
| 第11周 | 11.24–11.30 | 前端 + 后端联调 |
| 第12周 | 12.01–12.07 | 功能完善与测试 |
| 第13周 | 12.08–12.14 | 典型案例对比与上线 |
| 第14周 | 12.15–12.21 | 答辩准备 |
| 第15周 | 12.22–12.28 | 答辩与复盘 |

---

## 📄 License

本项目为软件工程课程团队作业，仅供学习交流使用。