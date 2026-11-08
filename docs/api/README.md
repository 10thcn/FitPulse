# FitPulse API 契约（OpenAPI / Swagger 草稿）

本目录存放 FitPulse 后端的 OpenAPI 3.0.3 契约，是前后端联调、Mock、契约测试的唯一事实来源。

| 文件 | 说明 |
|---|---|
| `openapi.yaml` | 契约：24 条路径 / 29 个操作 / 34 个数据模型 |
| `swagger-ui.html` | 独立 Swagger UI 预览页 |
| `swagger-ui-preview.png` | Swagger UI 渲染截图（草稿 v0.1） |
| `README.md` | 本文件 |

设计与待确认事项见 [`../FitPulse_API接口设计说明书.md`](../FitPulse_API接口设计说明书.md)。

## 本地预览

Swagger UI 通过 `fetch` 读取 `openapi.yaml`，必须经 HTTP 访问（直接双击 HTML 会因 `file://` 跨域限制而空白）。

```bash
# 在仓库根目录执行
python -m http.server 8090
```

浏览器打开：<http://localhost:8090/docs/api/swagger-ui.html>

## 校验契约

```bash
# Python（推荐，CI 可复用）
python -m pip install openapi-spec-validator pyyaml
python -c "from openapi_spec_validator.readers import read_from_filename as r; from openapi_spec_validator import validate; s,_=r('docs/api/openapi.yaml'); validate(s); print('OpenAPI OK')"
```

```bash
# Redocly Lint（额外检查风格与最佳实践）
npx @redocly/cli lint docs/api/openapi.yaml
```

## 生成客户端 / 服务端骨架

```bash
# 前端 TypeScript 客户端
npx @openapitools/openapi-generator-cli generate \
  -i docs/api/openapi.yaml -g typescript-fetch -o frontend/src/api

# 后端 FastAPI 路由骨架（仅参考，正式实现遵循现有目录结构）
npx @openapitools/openapi-generator-cli generate \
  -i docs/api/openapi.yaml -g python-fastapi -o backend_skeleton
```

## Mock 服务

```bash
# Prism：按契约返回示例数据，前端可在后端完成前先行开发
npx @stoplight/prism-cli mock docs/api/openapi.yaml -p 4010
```

## 约定速查

- 路径前缀 `/api`；除健康检查与认证外均需 `Authorization: Bearer <JWT>`
- JSON 响应统一信封 `{ code, message, data }`；列表分页 `data = { items, total, page, page_size }`
- 流式接口（SSE）：`POST /api/chat`、`POST /api/plans/generate`、`POST /api/correct`
- 业务错误码见设计说明书 §7
- 草稿新增/细化项（认证、上传、后台资源级 CRUD 等）见设计说明书 §9
