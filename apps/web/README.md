# web

用户侧 Next.js 应用，本地默认运行在 `http://localhost:3005`。

## 职责

- 展示用户侧首页和设计提示 UI。
- 引入 `@repo/ui` 的共享组件和 Tailwind 主题。
- 使用 `hono/client` 和 `@repo/api` 导出的 `AppType` 调用 typed RPC。
- 使用 `@repo/contracts` 里的请求类型、响应类型和业务错误码。
- 分离读取服务端环境变量和浏览器公开环境变量。

## 请求链路

当前首页会调用：

```text
POST /rpc/system/ping
```

调用链路：

```text
apps/web/app/page.tsx
  -> getWebServerEnv()
  -> hc<AppType>(API_BASE_URL)
  -> apps/api /rpc/system/ping
  -> packages/contracts PingRequestSchema
```

如果 API 请求失败，页面会使用 `BizCode.SYSTEM_UPSTREAM_TIMEOUT` 渲染兜底错误响应。

## 环境变量

本地默认配置：

```env
APP_ENV=development
API_BASE_URL=http://127.0.0.1:8787
NEXT_PUBLIC_APP_ENV=development
NEXT_PUBLIC_API_BASE_URL=http://127.0.0.1:8787
```

可以从示例文件复制：

```sh
cp apps/web/.env.example apps/web/.env.development
```

## 常用命令

在仓库根目录执行：

```sh
pnpm dev:web
pnpm --filter web check-types
pnpm --filter web lint
pnpm --filter web build
```

也可以进入 `apps/web` 后执行：

```sh
pnpm dev
pnpm check-types
pnpm lint
pnpm build
```

## 关键文件

- `app/page.tsx`：首页 UI、RPC 调用和响应展示。
- `src/env.server.ts`：服务端环境变量校验。
- `src/env.client.ts`：浏览器公开环境变量校验。
- `src/web-env-badge.tsx`：客户端环境变量展示组件。
- `app/globals.css`：Tailwind 和共享 UI theme 入口。
