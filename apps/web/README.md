# web

用户侧 Next.js 主应用，本地默认运行在 `http://localhost:3005`。当前还处于项目建设阶段，已经接入基础页面、共享 UI、环境变量校验，以及前后端类型共享与 RPC 调用，后续会继续补充真实业务页面和交互。

## 职责

- 承载用户侧主应用页面和后续业务交互。
- 当前已完成基础首页和设计提示 UI。
- 引入 `@repo/ui` 的共享组件和 Tailwind 主题。
- 使用 `hono/client` 和 `@repo/api` 导出的 `AppType` 调用后端 RPC 接口。
- 使用 `@repo/contracts` 做请求/响应类型共享。
- 分离读取服务端环境变量和浏览器公开环境变量。

## 当前状态

- 已完成用户侧基础页面结构。
- 已完成共享 UI 组件和主题接入。
- 已跑通 `POST /rpc/system/ping` RPC 调用。
- 已完成服务端和客户端环境变量校验。
- 后续继续扩展真实业务页面、数据展示和用户操作流程。

## RPC 调用流程

当前首页会调用：

```text
POST /rpc/system/ping
```

调用流程：

```text
apps/web/app/page.tsx
  -> getWebServerEnv()
  -> hc<AppType>(API_BASE_URL)
  -> apps/api /rpc/system/ping
  -> packages/contracts 里的共享 schema
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

- `app/page.tsx`：当前首页 UI、接口调用和响应展示，后续会继续拆分和承载业务模块。
- `src/env.server.ts`：服务端环境变量校验。
- `src/env.client.ts`：浏览器公开环境变量校验。
- `src/web-env-badge.tsx`：客户端环境变量展示组件。
- `app/globals.css`：Tailwind 和共享 UI theme 入口。
