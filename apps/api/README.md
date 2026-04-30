# api

基于 Hono 的 API 应用，通过 Wrangler 本地运行和部署到 Cloudflare Workers。

## 职责

- 提供 `GET /health` 健康检查接口。
- 提供 `POST /rpc/system/ping` typed RPC 示例接口。
- 使用 `@repo/contracts` 里的 zod schema 校验请求体。
- 使用统一的 `ApiResponse` 响应结构返回成功和失败结果。
- 导出 `AppType`，供前端通过 `hono/client` 获得类型安全的 RPC 调用。

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

- `src/app.ts`：Hono 应用、路由、错误处理和 `AppType` 导出。
- `src/index.ts`：Worker 默认导出入口。
- `src/env.ts`：Worker bindings 环境变量校验。
- `wrangler.jsonc`：Wrangler 应用名、入口、环境配置。
