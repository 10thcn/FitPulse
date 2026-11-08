# FitPulse API 接口设计说明书（OpenAPI / Swagger 草稿）

| 项目 | 内容 |
|---|---|
| 项目名称 | FitPulse 健身动作矫正 & 居家锻炼智能 Agent |
| 团队名称 | FitPulse |
| 文档版本 | v0.1（草稿） |
| 编制日期 | 2026-11-08 |
| 编制人 | 徐心铭（102401531，后端 / API 集成） |
| 文档状态 | 草稿（待接口评审） |
| 参考规范 | GB/T 8567-2006《计算机软件文档编制规范》 |
| 契约文件 | [`docs/api/openapi.yaml`](api/openapi.yaml)（OpenAPI 3.0.3，29 个操作） |

---

## 文档修订记录

| 版本 | 日期 | 修改人 | 修改内容 |
|---|---|---|---|
| v0.1 | 2026-11-08 | 徐心铭 | 初稿：依据系统设计说明书 §4 / §5 输出完整 OpenAPI 3.0.3 契约，补齐认证、上传、分页等集成缺口，整理待确认事项 |

---

## 1. 编写目的与范围

### 1.1 编写目的

《FitPulse 系统设计说明书》§4 已按模块给出接口清单，但接口的请求体、响应结构、错误语义仍停留在表格描述层面，无法直接支撑前后端并行开发。本文档配合 `openapi.yaml` 契约文件，目标是：

1. 把 24 条路径、29 个操作固化为 **机器可读、可校验、可生成 Mock 与客户端代码** 的 OpenAPI 3.0.3 描述；
2. 统一认证、响应信封、分页、时间、错误码、SSE 流式协议等横切规范，避免前后端各自约定；
3. 明确系统设计说明书中未覆盖或存在歧义的部分，形成待评审问题清单（§9），供接口评审会一次性拍板；
4. 为后续 FastAPI 实现、测试工程师的契约测试（连向薇）、Swagger UI 演示提供唯一事实来源（Single Source of Truth）。

### 1.2 范围

**包含**：`/api` 前缀下的全部内部 REST 接口，以及 3 个 SSE 流式接口（对话、计划生成、动作纠错）。

**不包含**（属外部依赖或内部实现，仅在此说明）：

- Claude Messages / Embeddings API（见系统设计说明书 §4.1）；
- Chroma 向量库、MinIO 对象存储的内部调用；
- 前端页面路由（见系统设计说明书 §3.2.5）。

### 1.3 读者

前端工程师、后端工程师、测试工程师、队长（PR Review）、指导教师 / 助教（验收检查）。

### 1.4 交付物清单

| 文件 | 说明 |
|---|---|
| [`docs/api/openapi.yaml`](api/openapi.yaml) | OpenAPI 3.0.3 契约，唯一事实来源 |
| [`docs/api/swagger-ui.html`](api/swagger-ui.html) | 独立 Swagger UI 预览页（CDN 加载，需通过 HTTP 访问） |
| [`docs/api/swagger-ui-preview.png`](api/swagger-ui-preview.png) | Swagger UI 渲染截图（草稿 v0.1） |
| [`docs/api/README.md`](api/README.md) | 预览、校验、代码生成的使用说明 |
| `docs/FitPulse_API接口设计说明书.md` | 本文档（设计依据、协议约定、差异说明） |

---

## 2. 依据与术语

### 2.1 依据文档

| 文档 | 版本 | 引用内容 |
|---|---|---|
| 《FitPulse 需求分析文档》 | v0.1 | 三大核心能力、验收任务 T1–T3 |
| 《FitPulse 系统设计说明书》 | v1.0 | §3 模块设计、§4 接口设计、§5 数据结构、§7 出错处理 |
| 《FitPulse 数据库设计说明书》 | v1.0 | 实体与字段（users / exercises / knowledge_chunks / plans / sessions / messages / corrections / feedbacks） |

### 2.2 术语

| 术语 | 说明 |
|---|---|
| OpenAPI | REST 接口的机器可读描述规范，本文使用 3.0.3 |
| Swagger UI | 基于 OpenAPI 文档渲染的交互式接口调试页面 |
| SSE | Server-Sent Events，服务器单向流式推送，`Content-Type: text/event-stream` |
| JWT | JSON Web Token，本文作为无状态访问令牌 |
| RAG | Retrieval-Augmented Generation，知识检索增强生成 |
| 契约测试 | 以 OpenAPI 为准校验后端实现与前端调用的测试方式 |

---

## 3. 接口总体规范

### 3.1 基础信息

| 项 | 约定 |
|---|---|
| 路径前缀 | `/api` |
| API 版本 | `0.1.0`（契约草稿，冻结后升 `1.0.0`） |
| 开发环境 Base URL | `http://localhost:8000`（uvicorn 直连） |
| 联调 / 生产 Base URL | `http://localhost`（Docker Compose，Nginx `/api` 反代） |
| 请求 / 响应编码 | UTF-8；JSON 为 `application/json` |
| 文件上传 | `multipart/form-data`，单文件 ≤ 10MB，JPG / PNG / WebP |
| 流式响应 | `text/event-stream`（SSE），仅 `POST /api/chat`、`POST /api/plans/generate`、`POST /api/correct` |
| 标识符 | 业务主键统一 UUID 字符串（`format: uuid`） |
| 时间 | ISO 8601 带时区，如 `2026-11-08T17:30:00+08:00`；纯日期 `YYYY-MM-DD` |

### 3.2 认证与鉴权

系统设计说明书 §2 明确「API 网关（Nginx + JWT）」，但 §4 未定义登录接口。本草稿补齐认证闭环（标注为草稿提案）：

```
登录 POST /api/auth/login ──→ { access_token(2h), refresh_token(7d) }
                                        │
        业务请求 Authorization: Bearer <access_token>
                                        │
                令牌过期(40101) → POST /api/auth/refresh → 新 access_token
```

| 项 | 约定 |
|---|---|
| 令牌类型 | JWT，`Authorization: Bearer <access_token>` |
| access_token 有效期 | 2 小时（7200s） |
| refresh_token 有效期 | 7 天，仅用于刷新，不可访问业务接口 |
| 管理员接口 | `/api/admin/**` 要求 JWT 携带 `role=admin`，否则 403 / 40300 |
| 免鉴权接口 | `GET /api/health`、`POST /api/auth/login`、`POST /api/auth/refresh` |
| 令牌失效 | 401 / 40100（未登录）或 40101（过期），前端清除本地令牌并跳转登录 |

> **待确认（Q1）**：账号体系用学号、邮箱还是手机号？注册入口是否由产品经理定义？本草稿先用 `username`（学号示例）+ `password` 占位。

### 3.3 统一响应格式

除健康检查与 SSE 外，所有 JSON 响应使用统一信封（与系统设计说明书 §4.3 一致）：

**成功**

```json
{
  "code": 0,
  "message": "success",
  "data": { "user_id": "…", "nickname": "小林" }
}
```

**分页**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "items": [],
    "total": 120,
    "page": 1,
    "page_size": 10
  }
}
```

**失败**

```json
{
  "code": 40001,
  "message": "参数校验失败：缺少必填字段 message",
  "data": null
}
```

说明：

- 系统设计说明书 §4.2 表格中的「响应 = TrainingPlan」等，均指信封中的 `data` 字段；
- `GET /api/health` 为探活接口，直接返回健康对象，不套信封（便于 Docker / 监控直接判断）；
- SSE 响应不走信封，事件负载 `StreamEvent` 见 §4。

### 3.4 分页约定

| 参数 | 类型 | 默认 | 约束 | 说明 |
|---|---|---|---|---|
| `page` | integer | 1 | ≥ 1 | 页码，从 1 开始 |
| `size` | integer | 10 | 1–50 | 每页条数 |

适用接口：`/api/plans/history`、`/api/correct/history`、`/api/exercises`、`/api/admin/knowledge`、`/api/admin/users`、`/api/admin/feedbacks`。

> 系统设计说明书仅在计划、纠错历史中写了分页；本草稿将分页推广到所有列表接口，避免动作库与后台列表无上限（Q5）。

### 3.5 幂等与并发

| 场景 | 约定 |
|---|---|
| 打卡 `PUT /api/plans/{plan_id}/days/{day_index}/exercises/{ex_idx}` | 幂等：重复提交相同 `done` 值不产生副作用，直接返回成功 |
| 删除会话 / 知识条目 | 幂等：重复删除返回 200（第二次起可返回 40400，由实现选择；建议统一 200 便于前端） |
| 计划生成 `POST /api/plans/generate` | 非幂等；同一用户 60 秒内重复触发生成返回 429 / 42900，防止并发写坏当前计划 |
| 消息发送 `POST /api/chat` | 按 `session_id` 串行处理；同一会话并发请求由后端队列化或拒绝（建议 409） |

### 3.6 限流

由 Nginx 或 FastAPI 中间件实现，超限返回 **429 + `Retry-After`** 响应头（系统设计说明书 §7.1）：

| 接口组 | 建议阈值 | 说明 |
|---|---|---|
| `POST /api/auth/login` | 5 次 / 分钟 / IP | 防暴力破解 |
| `POST /api/chat` | 20 次 / 分钟 / 用户 | 单次可能触发 LLM 调用 |
| `POST /api/plans/generate`、`POST /api/correct` | 10 次 / 分钟 / 用户 | 长耗时、成本高 |
| `POST /api/uploads` | 20 次 / 分钟 / 用户 | 文件存储成本 |
| 其他读接口 | 60 次 / 分钟 / 用户 | — |

> 阈值为建议值，压测后调整（Q15）。

---

## 4. SSE 流式协议

### 4.1 为什么用 SSE

对话、计划生成、纠错都依赖 LLM，首字延迟可达数秒。SSE 让前端边收边渲染（思考状态、工具调用进度、增量文本），显著改善体验；相比 WebSocket 更轻量，且与现有 HTTP 鉴权、Nginx 反代兼容。

### 4.2 报文格式

每个事件的 `data` 是一行 JSON（`StreamEvent`），事件之间以空行分隔；后端可每 15 秒发送 `: ping` 注释行保活。

```
data: {"type":"thinking","content":"正在分析你的需求..."}

data: {"type":"tool_call","tool":"generate_plan","status":"running"}

data: {"type":"text","content":"好的！根据你的情况，"}

data: {"type":"done"}
```

### 4.3 事件类型

| `type` | 关键字段 | 说明 |
|---|---|---|
| `thinking` | `content` | 思考 / 进度提示，前端展示为状态文案 |
| `tool_call` | `tool`、`status` | 工具调用状态；`tool` ∈ `query_exercise` / `generate_plan` / `correct_posture` |
| `text` | `content` | 增量文本片段，**按到达顺序拼接**，不可单独渲染为完整消息 |
| `plan` | `data` | 完整 `TrainingPlan`，仅在计划生成流中出现 |
| `correction` | `data` | 完整 `CorrectionReport`，仅在纠错流中出现 |
| `done` | — | 流正常结束，前端关闭连接并落库 |
| `error` | `error.code`、`error.message` | 流内错误（HTTP 已返回 200 后发生），前端展示错误并允许重试 |

### 4.4 各接口事件序列

| 接口 | 事件序列 |
|---|---|
| `POST /api/chat` | `thinking`* → `tool_call`* → `text`* → `done`（或 `error`） |
| `POST /api/plans/generate` | `thinking`* → `tool_call`(generate_plan) → `text`* → `plan` → `done` |
| `POST /api/correct` | `thinking`* → `tool_call`(correct_posture) → `text`* → `correction` → `done` |

> **注意**：SSE 一旦开始返回 200，后续 LLM 失败只能通过 `error` 事件表达，不能再改 HTTP 状态码。前端需同时处理「HTTP 错误」与「流内 error 事件」两条路径（Q12）。

### 4.5 前端对接要点

```ts
const res = await fetch('/api/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${accessToken}`,
  },
  body: JSON.stringify({ session_id, message, images }),
});

const reader = res.body!.getReader();
const decoder = new TextDecoder();
let buffer = '';

while (true) {
  const { value, done } = await reader.read();
  if (done) break;
  buffer += decoder.decode(value, { stream: true });

  const events = buffer.split('\n\n');
  buffer = events.pop()!;            // 保留未完整的事件
  for (const raw of events) {
    const line = raw.trim();
    if (!line.startsWith('data:')) continue;   // 忽略 ": ping"
    const event = JSON.parse(line.slice(5).trim()) as StreamEvent;
    handleEvent(event);                        // thinking/tool_call/text/plan/correction/done/error
  }
}
```

- 原生 `EventSource` **不支持 POST / 自定义 Header / FormData**，必须用 `fetch` + `ReadableStream`；
- 纠错接口是 `multipart/form-data`，同样用 `fetch` + `FormData`，**不要**手动设置 `Content-Type`（浏览器会自动带 boundary）。

### 4.6 Nginx 配置要求

反代时必须关闭缓冲，否则 SSE 会被整段缓存、失去流式效果：

```nginx
location /api/ {
    proxy_pass http://backend:8000;
    proxy_http_version 1.1;
    proxy_set_header Connection '';
    proxy_buffering off;
    proxy_cache off;
    proxy_read_timeout 300s;
    add_header X-Accel-Buffering no;
}
```

---

## 5. 接口清单

图例：🔒 需登录 JWT；👑 需管理员；🌊 SSE；🆕 本草稿新增（系统设计说明书未定义）。

| # | 模块 | 方法 | 路径 | 说明 | 来源 | 鉴权 |
|---:|---|---|---|---|---|---|
| 1 | 健康检查 | GET | `/api/health` | 探活 | §6.3 | 免 |
| 2 | 认证 🆕 | POST | `/api/auth/login` | 登录换取 JWT | 草稿提案 | 免 |
| 3 | 认证 🆕 | POST | `/api/auth/refresh` | 刷新访问令牌 | 草稿提案 | 免 |
| 4 | 文件 🆕 | POST | `/api/uploads` | 上传聊天图片 / 头像 | 草稿提案 | 🔒 |
| 5 | 对话 | POST | `/api/chat` | 发送消息 🌊 | §4.2.1 | 🔒 |
| 6 | 对话 | POST | `/api/sessions` | 创建会话 | §4.2.1 | 🔒 |
| 7 | 对话 | GET | `/api/sessions/{session_id}` | 会话历史 | §4.2.1 | 🔒 |
| 8 | 对话 | DELETE | `/api/sessions/{session_id}` | 删除会话 | §4.2.1 | 🔒 |
| 9 | 计划 | GET | `/api/plans/current` | 当前周计划 | §4.2.2 | 🔒 |
| 10 | 计划 | POST | `/api/plans/generate` | 生成计划 🌊 | §4.2.2 | 🔒 |
| 11 | 计划 | PUT | `/api/plans/{plan_id}/days/{day_index}/exercises/{ex_idx}` | 动作打卡 | §4.2.2 | 🔒 |
| 12 | 计划 | GET | `/api/plans/history` | 历史计划 | §4.2.2 | 🔒 |
| 13 | 纠错 | POST | `/api/correct` | 图片纠错 🌊 | §4.2.3 | 🔒 |
| 14 | 纠错 | GET | `/api/correct/history` | 纠错历史 | §4.2.3 | 🔒 |
| 15 | 纠错 | GET | `/api/correct/{correction_id}` | 纠错详情 | §4.2.3 | 🔒 |
| 16 | 用户 | GET | `/api/user/profile` | 获取档案 | §4.2.4 | 🔒 |
| 17 | 用户 | PUT | `/api/user/profile` | 更新档案 | §4.2.4 | 🔒 |
| 18 | 用户 | GET | `/api/user/stats` | 训练统计 | §4.2.4 | 🔒 |
| 19 | 动作库 | GET | `/api/exercises` | 动作列表 | §4.2.5 | 🔒 |
| 20 | 动作库 | GET | `/api/exercises/{exercise_id}` | 动作详情 | §4.2.5 | 🔒 |
| 21 | 后台 👑 | GET | `/api/admin/knowledge` | 知识条目列表 | §4.2.6 | 👑 |
| 22 | 后台 👑 | POST | `/api/admin/knowledge` | 新增条目 | §4.2.6 | 👑 |
| 23 | 后台 👑 | GET | `/api/admin/knowledge/{chunk_id}` | 条目详情 🆕 | §4.2.6 细化 | 👑 |
| 24 | 后台 👑 | PUT | `/api/admin/knowledge/{chunk_id}` | 更新条目 🆕 | §4.2.6 细化 | 👑 |
| 25 | 后台 👑 | DELETE | `/api/admin/knowledge/{chunk_id}` | 删除条目 🆕 | §4.2.6 细化 | 👑 |
| 26 | 后台 👑 | GET | `/api/admin/users` | 用户列表 | §4.2.6 | 👑 |
| 27 | 后台 👑 | GET | `/api/admin/dashboard` | 系统概览 | §4.2.6 | 👑 |
| 28 | 后台 👑 | GET | `/api/admin/feedbacks` | 反馈列表 | §4.2.6 | 👑 |
| 29 | 后台 👑 | PUT | `/api/admin/feedbacks/{feedback_id}` | 处理反馈 🆕 | §4.2.6 细化 | 👑 |

统计：24 条路径 / 29 个操作；其中 5 条路径为草稿新增或 REST 化细化。

### 5.1 关键接口示例

**对话（请求）**

```http
POST /api/chat
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "session_id": "6f1c2a8e-6b1a-4e8e-9d2f-1a2b3c4d5e6f",
  "message": "我想练胸，给我一个 20 分钟的居家计划",
  "images": []
}
```

**计划生成（SSE 关键帧）**

```
data: {"type":"thinking","content":"正在结合你的档案与知识库编排计划..."}

data: {"type":"tool_call","tool":"generate_plan","status":"running"}

data: {"type":"plan","data":{"plan_id":"9b2f...","week_number":1,"start_date":"2026-11-09","days":[]}}

data: {"type":"done"}
```

**动作纠错（multipart + SSE）**

```http
POST /api/correct
Authorization: Bearer <JWT>
Content-Type: multipart/form-data; boundary=----fitpulse

------fitpulse
Content-Disposition: form-data; name="image"; filename="pushup.jpg"
Content-Type: image/jpeg

<二进制>
------fitpulse
Content-Disposition: form-data; name="description"

做俯卧撑感觉腰酸
------fitpulse--
```

响应为 `text/event-stream`，最终 `correction` 事件的 `data` 即完整 `CorrectionReport`。

---

## 6. 分模块设计说明

### 6.1 认证（🆕 草稿提案）

| 设计点 | 说明 |
|---|---|
| 登录 | 校验账号密码 → 签发 access_token + refresh_token；`user_id` 一并返回，前端免二次请求 |
| 刷新 | 仅接受 refresh_token；刷新后旧 access_token 自然过期（不主动黑名单，降低实现复杂度） |
| 登出 | 本期不做服务端登出（无状态 JWT）；前端删除本地令牌即可。如需强制下线，后续加 Redis 黑名单 |
| 安全 | 密码 bcrypt 存储；登录失败统一提示「账号或密码错误」，不区分账号是否存在 |

### 6.2 对话

- `POST /api/sessions` 创建空会话，返回 `session_id`；前端进入首页时先建会话；
- `POST /api/chat` 由编排层完成意图识别 → RAG / 工具 → LLM → SSE；消息落库由后端负责；
- `GET /api/sessions/{id}` 返回完整历史（含 `tool_calls`），供刷新恢复；
- `DELETE /api/sessions/{id}` 逻辑删除，保留统计数据。

### 6.3 训练计划

- `GET /api/plans/current`：仅返回 `status=active` 且日期覆盖当天的计划；无计划返回 404 / 40400，前端引导「生成计划」；
- `POST /api/plans/generate`：`user_profile` 可选，不传用当前档案；生成成功自动置为当前计划，旧计划置 `abandoned`；
- 打卡：`day_index` 为 1–7，`ex_idx` 为当日动作数组下标（从 0 开始），二者共同定位 `PlanExercise`；
- 历史：按 `start_date` 倒序分页。

### 6.4 动作纠错

- 上传走 `multipart/form-data`（字段 `image` 必填、`description` 可选）；
- 后端保存原图到 MinIO，`image_url` 写库；
- SSE 最终返回 `CorrectionReport`，包含 `issues[]` 与 `safety_note`；
- 命中疼痛 / 受伤信号时，`safety_note` 必须给出「停止训练并就医」提示（需求文档 §6.3 安全边界）；
- **路由顺序**：`GET /api/correct/history` 必须在 `GET /api/correct/{correction_id}` 之前注册。

### 6.5 用户

- 档案字段与数据库 `users` 表一一对应（`equipment` 为 JSON 数组）；
- `PUT` 为全量更新（Q2），前端表单一次提交全部字段；
- 统计口径：`total_workouts` 按完成的训练日计数；`weekly` 返回最近 7 天。

### 6.6 动作库

- 列表支持 `category`（胸/背/腿/肩/核心/全身）、`difficulty`、`search` 组合筛选；
- 详情可选 `include_knowledge=true` 附带关联知识片段，供「动作查询」回答标注出处；
- 分类枚举沿用系统设计说明书的中文值（Q14），若后续国际化再改英文键。

### 6.7 后台管理（👑）

- 统一要求 `role=admin`；非管理员 403 / 40300；
- 知识库 CRUD 由设计文档的「集合级 /api/admin/knowledge」细化为资源级（Q4）：列表 / 新增在集合上，详情 / 更新 / 删除在 `/{chunk_id}` 上；
- 知识条目新增 / 更新后，由 RAG 流水线异步向量化；向量库异常返回 503 / 50002，条目本身仍可落库（最终一致，Q16）；
- 反馈管理由 `GET/PUT /api/admin/feedbacks` 细化为列表 + `PUT /{feedback_id}`（Q11）。

### 6.8 健康检查

- `GET /api/health` 返回 `status`、`version`、`timestamp`；依赖不可用时返回 503，供 Docker healthcheck 与 Nginx 摘流。

---

## 7. 错误码与 HTTP 状态映射

系统设计说明书 §4.3 定义了业务错误码；本草稿补充其与 HTTP 状态码的映射，避免前端需要同时解析两层状态。

| 业务码 | 含义 | HTTP 状态 | 前端处理建议 |
|---|---|---:|---|
| 0 | 成功 | 200 / 201 | — |
| 40001 | 参数校验失败 | 400 | 展示 `message`，定位到表单字段 |
| 40100 | 未登录 | 401 | 跳转登录 |
| 40101 | Token 过期 | 401 | 调 `/api/auth/refresh`，失败再跳登录 |
| 40300 | 无权限 | 403 | 提示无权限 / 隐藏入口 |
| 40400 | 资源不存在 | 404 | 空态页 / 返回上一页 |
| 42900 | 请求过于频繁 | 429 | 按 `Retry-After` 倒计时 |
| 50000 | 服务器内部错误 | 500 | 通用错误提示 + 重试 |
| 50001 | LLM API 调用失败 | 503 | 提示「智能教练暂时繁忙」 |
| 50002 | 向量数据库异常 | 503 | 提示稍后重试 |
| 50003 | 文件上传失败 | 400 / 413 | 提示格式或大小限制 |

> HTTP 语义与业务码并存：HTTP 状态用于网关、监控、重试策略；业务码用于前端精细化提示。SSE 场景下，流开始后的错误只能通过 `error` 事件携带上述业务码（Q12）。

---

## 8. 数据模型字典

`openapi.yaml` 共定义 34 个 schema。与系统设计说明书 §5.1 的对应关系：

| Schema | 对应设计文档结构 | 用途 |
|---|---|---|
| `ApiResponse` / `ErrorResponse` | §4.3 通用响应 | 统一信封 |
| `PageMeta` / `SuccessData` | §4.3 分页 | 分页元信息、操作结果 |
| `HealthResponse` | §6.3 健康检查 | 探活 |
| `LoginRequest` / `TokenData` / `RefreshRequest` | 🆕 | 认证 |
| `UploadedFile` | 🆕（MinIO） | 上传结果 |
| `ChatRequest` | §3.2.1 输入 | 发送消息 |
| `StreamEvent` | §3.2.1 SSE 输出 | 流式事件（7 种类型） |
| `ChatMessage` / `ToolCall` | 数据库 messages / tool_calls | 消息与工具调用 |
| `Session` / `SessionDetail` | 数据库 sessions | 会话 |
| `UserProfile` / `UserProfileInput` | §5.1.1 | 用户档案 |
| `UserStats` / `WeeklyStat` | §4.2.4 训练统计 | 个人中心 |
| `Exercise` | §5.1.2 | 动作 |
| `TrainingPlan` / `PlanDay` / `PlanExercise` | §5.1.3 | 训练计划 |
| `CorrectionReport` / `CorrectionIssue` | §5.1.4 | 纠错 |
| `KnowledgeChunk` / `KnowledgeChunkInput` | §3.2.3 知识条目 | 后台知识库 |
| `AdminUser` / `DashboardStats` / `KnowledgeCountByModule` | 后台 | 用户管理、系统概览 |
| `Feedback` / `FeedbackUpdate` | 数据库 feedbacks | 反馈管理 |
| `MarkDoneRequest` / `PlanGenerateRequest` | §4.2.2 | 计划请求体 |

字段级定义（类型、必填、枚举、示例）以 `openapi.yaml` 的 `components.schemas` 为准；本表只做索引，避免文档与契约两处维护导致漂移。

---

## 9. 与系统设计说明书的差异及待确认事项

以下问题需在接口评审会拍板；未确认前后端按「本草稿建议」实现，确认后同步修订契约与本文档。

| 编号 | 问题 | 本草稿建议 | 影响 |
|---|---|---|---|
| Q1 | 设计文档未定义登录 / 注册接口，账号体系未定 | 补 `POST /api/auth/login`、`POST /api/auth/refresh`；账号暂用学号，密码 8–64 位 | 前端登录页、网关 JWT 校验、用户表新增 `password_hash` 字段 |
| Q2 | `PUT /api/user/profile` 是全量还是部分更新 | 全量更新（除 `avatar_url` 外必填），语义清晰、避免 PATCH 复杂度 | 前端表单需一次提交完整档案 |
| Q3 | `/api/plans/current` 无计划时 404 还是 200 + `data:null` | 404 / 40400，前端展示「去生成计划」空态 | 前端需处理 404 分支 |
| Q4 | 后台知识库原设计为集合级 `GET/POST/PUT/DELETE /api/admin/knowledge` | 细化为资源级 CRUD（`/{chunk_id}`） | 后台页面接口调用方式 |
| Q5 | 列表接口是否统一分页 | 统一 `page` / `size`，默认 10、最大 50 | 动作库、后台列表前端需接分页组件 |
| Q6 | 聊天图片上传接口缺失（`images[]` 只有 URL） | 补 `POST /api/uploads`（MinIO） | 前端拍照 / 选图流程 |
| Q7 | SSE 走 POST，原生 `EventSource` 不支持 | 前端统一 `fetch` + `ReadableStream`；Nginx 关闭 `proxy_buffering` | 前端 SSE 封装、部署配置 |
| Q8 | 打卡路径的 `day_index`、`ex_idx` 语义 | `day_index` 1–7；`ex_idx` 为动作下标（0 起，对应 `sort_order`） | 前后端定位一致，避免 off-by-one |
| Q9 | 头像更新流程 | 先 `POST /api/uploads?purpose=avatar` 得 URL，再 `PUT /api/user/profile` | 个人中心交互 |
| Q10 | `/api/correct/history` 与 `/{correction_id}` 路由冲突 | 后端先注册 `history`；契约已注明 | FastAPI 路由顺序 |
| Q11 | 反馈管理 `PUT /api/admin/feedbacks` 集合级更新 | 细化为 `PUT /api/admin/feedbacks/{feedback_id}` | 后台反馈页 |
| Q12 | SSE 流内错误语义 | 流内失败发 `error` 事件（业务码 + 消息），HTTP 已 200 不再改 | 前端双路径错误处理 |
| Q13 | 新用户档案何时创建 | 注册时创建默认档案（`beginner` / `general` / `[]` / 3 天） | 注册接口、`users` 表默认值 |
| Q14 | 动作分类枚举用中文还是英文键 | 沿用设计文档中文值（胸/背/腿/肩/核心/全身） | 知识条目、前端筛选 |
| Q15 | 限流阈值 | 按 §3.6 建议值，压测后调整 | 网关配置、测试用例 |
| Q16 | 知识条目与向量库的一致性 | MySQL 先落库，异步向量化；失败返回 503 / 50002 并可重试 | RAG 初始化脚本、后台提示 |
| Q17 | API 版本策略 | URL 不带版本号（`/api`），契约升到 1.0.0 后冻结；破坏性变更另起 `/api/v2` | 部署与文档 |

---

## 10. 预览、校验与后续计划

### 10.1 本地预览

```bash
# 在仓库根目录启动静态服务（Swagger UI 需 HTTP 访问，file:// 会被 CORS 拦截）
python -m http.server 8090

# 浏览器打开
# http://localhost:8090/docs/api/swagger-ui.html
```

### 10.2 契约校验

```bash
# 方式一：Python 校验（CI 可复用）
python -m pip install openapi-spec-validator pyyaml
python -c "from openapi_spec_validator.readers import read_from_filename as r; from openapi_spec_validator import validate; s,_=r('docs/api/openapi.yaml'); validate(s); print('OK')"

# 方式二：Redocly Lint（可检查规范与风格问题）
npx @redocly/cli lint docs/api/openapi.yaml
```

### 10.3 后续计划

| 阶段 | 工作 | 负责人 |
|---|---|---|
| 第 9 周 | 接口评审，关闭 §9 待确认项，契约升 v1.0 冻结 | 徐心铭 + 罗炜 + 张智轩 |
| 第 9–10 周 | 按契约实现 FastAPI 路由与 Pydantic 模型；Swagger UI 挂载 `/docs` | 徐心铭 |
| 第 10 周 | 前端按契约对接，Mock（Prism）先行 | 张智轩 |
| 第 11 周 | 契约测试（OpenAPI 校验 + 关键用例） | 连向薇 |
| 第 12 周 | SSE、限流、错误码联调与压测 | 全员 |

### 10.4 验收对照（需求文档 §7）

| 验收任务 | 相关接口 |
|---|---|
| T1 深蹲膝盖不舒服 → 原因 + 纠正 + 出处 | `POST /api/chat`、`GET /api/exercises/{id}?include_knowledge=true` |
| T2 一周训练计划 | `POST /api/plans/generate`、`GET /api/plans/current` |
| T3 上传俯卧撑照片纠错 | `POST /api/correct`、`GET /api/correct/{id}` |

---

## 附录 A：接口分组与设计文档章节对照

| 分组 | 设计文档章节 | 契约 tag |
|---|---|---|
| 健康检查 | §6.3 | 健康检查 |
| 认证 | 🆕 | 认证 |
| 文件上传 | 🆕 | 文件上传 |
| 对话 | §3.2.1、§4.2.1 | 对话 |
| 训练计划 | §4.2.2、§5.1.3 | 训练计划 |
| 动作纠错 | §4.2.3、§5.1.4 | 动作纠错 |
| 用户 | §4.2.4、§5.1.1 | 用户 |
| 动作库 | §4.2.5、§5.1.2 | 动作库 |
| 后台管理 | §4.2.6 | 后台管理 |

## 附录 B：OpenAPI 文件结构速览

```
docs/api/
├── openapi.yaml       # 契约：info / servers / tags / paths / components
├── swagger-ui.html    # 预览页
└── README.md          # 预览与校验说明

openapi.yaml
├── paths/             # 24 条路径、29 个操作
├── components/
│   ├── securitySchemes/   # bearerAuth (JWT)
│   ├── parameters/        # 分页、路径参数
│   ├── responses/         # 400/401/403/404/413/429/500/503
│   └── schemas/           # 34 个数据模型
```
