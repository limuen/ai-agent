# api

基于 Hono 的 API 应用，通过 Wrangler 本地运行和部署到 Cloudflare Workers。当前还处于项目建设阶段，已经完成基础路由、统一响应结构、环境变量校验，以及类型共享与 RPC 调用基础，后续会继续扩展真实业务接口。

## 职责

- 承载后续业务 API 和前端 RPC 调用。
- 当前提供 `GET /health` 健康检查接口。
- 当前提供 `POST /rpc/system/ping` 基础 RPC 接口。
- 使用 `@repo/contracts` 做请求/响应类型共享，并复用其中的 zod schema 校验请求体。
- 使用统一的 `ApiResponse` 响应结构返回成功和失败结果。
- 导出 `AppType`，供前端通过 `hono/client` 调用 RPC 接口时复用后端类型。

## 当前状态

- 已完成 Hono 应用入口和 Worker 默认导出。
- 已完成基础健康检查接口和 ping RPC 接口。
- 已完成统一成功/失败响应结构。
- 已完成 zod 入参校验、环境变量校验和基础类型共享。
- 后续继续扩展业务路由、数据访问层、鉴权和错误处理策略。

## 路由

```text
GET  /health
POST /rpc/system/ping
```

`POST /rpc/system/ping` 请求体：

```json
{
  "name": "web"
}
```

成功响应会使用统一结构：

```json
{
  "ok": true,
  "data": {
    "service": "api",
    "message": "pong, web",
    "env": "development"
  },
  "meta": {
    "requestId": "...",
    "timestamp": "..."
  }
}
```

## 环境变量

Wrangler 本地开发默认读取 `.dev.vars`：

```env
APP_ENV=development
```

可以从示例文件复制：

```sh
cp apps/api/.dev.vars.example apps/api/.dev.vars
```

`APP_ENV` 只允许：

```text
development
test
production
```

## 常用命令

在仓库根目录执行：

```sh
pnpm dev:api
pnpm dev:api:test
pnpm --filter @repo/api check-types
```

也可以进入 `apps/api` 后执行：

```sh
pnpm dev
pnpm dev:test
pnpm check-types
```

## 部署

```sh
pnpm --filter @repo/api deploy:test
pnpm --filter @repo/api deploy:production
```

生成 Cloudflare Worker 类型：

```sh
pnpm --filter @repo/api cf-typegen
```

## 关键文件

- `src/app.ts`：Hono 应用、当前基础路由、错误处理和 `AppType` 导出。
- `src/index.ts`：Worker 默认导出入口。
- `src/env.ts`：Worker bindings 环境变量校验。
- `wrangler.jsonc`：Wrangler 应用名、入口、环境配置。
