# ai-agent

这是一个 TypeScript monorepo 项目，包含两个 Next.js 前端应用、一个基于 Hono 的 Cloudflare Workers API、共享 UI 组件、共享 RPC 契约，以及统一的 lint/type 配置。

## 架构

```text
ai-agent
├── apps
│   ├── web      面向用户侧的 Next.js 应用，本地端口 3005
│   ├── admin    管理后台 Next.js 应用，本地端口 3006
│   └── api      基于 Hono 的 Cloudflare Workers API，本地 Wrangler 端口 8787
├── packages
│   ├── contracts            共享 zod schema、API 响应结构、业务错误码
│   ├── ui                   共享 Tailwind 主题和 React UI 基础组件
│   ├── eslint-config        共享 ESLint 配置
│   └── typescript-config    共享 TypeScript 配置
├── pnpm-workspace.yaml      workspace 包声明和依赖 catalog
└── turbo.json               Turbo 任务图和环境变量透传配置
```

核心请求链路：

```text
apps/web 或 apps/admin
  -> Hono typed client / 环境变量 API_BASE_URL
  -> apps/api Hono route
  -> packages/contracts schema 和响应结构
```

`packages/contracts` 是请求校验和响应结构的单一来源。前端从这里引入请求/响应类型和业务错误码，`apps/api` 也从这里引入同一套 zod schema 来校验接口入参。

## 应用和包

- `apps/web`：用户侧 Next.js 应用，展示共享 UI，并调用 `POST /rpc/system/ping`。
- `apps/admin`：管理后台 Next.js 应用，使用同样的环境变量校验方式。
- `apps/api`：Hono API 应用，导出 `AppType` 给 typed RPC client 使用，并通过 Wrangler 部署。
- `packages/contracts`：共享 `ApiResponse`、`BizCode`、`PingRequestSchema` 等契约类型。
- `packages/ui`：共享 Tailwind CSS 主题和 `Button`、`Card`、`Input`、`Label`、`Separator`、`TailwindDemo` 等组件。
- `packages/eslint-config`：供各应用和包复用的 ESLint 配置。
- `packages/typescript-config`：供各应用和包复用的 TypeScript 配置。

## 环境要求

- Node.js `>=18`
- pnpm `10.33.2`

在仓库根目录安装依赖：

```sh
pnpm install
```

## 环境变量

本地默认 API 地址是 `http://127.0.0.1:8787`。

前端环境变量：

```env
APP_ENV=development
API_BASE_URL=http://127.0.0.1:8787
NEXT_PUBLIC_APP_ENV=development
NEXT_PUBLIC_API_BASE_URL=http://127.0.0.1:8787
```

API 环境变量：

```env
APP_ENV=development
```

可以从示例文件复制一份本地配置：

```sh
cp apps/web/.env.example apps/web/.env.development
cp apps/admin/.env.example apps/admin/.env.development
cp apps/api/.dev.vars.example apps/api/.dev.vars
```

## 本地开发

分别启动各服务：

```sh
pnpm dev:api
pnpm dev:web
pnpm dev:admin
```

默认本地地址：

- Web：`http://localhost:3005`
- Admin：`http://localhost:3006`
- API：`http://127.0.0.1:8787`

通过 Turbo 启动全部 dev 任务：

```sh
pnpm dev
```

## 校验

提交或推送前建议跑这组检查：

```sh
pnpm install
pnpm --filter @repo/ui check-types
pnpm --filter web check-types
pnpm --filter admin check-types
pnpm lint
pnpm build
```

常用根目录脚本：

```sh
pnpm check-types
pnpm lint
pnpm build
pnpm format
```

## 构建

通过 Turbo 构建所有应用和包：

```sh
pnpm build
```

构建或启动指定环境的前端目标：

```sh
pnpm build:web:test
pnpm start:web:test
pnpm build:admin:test
pnpm start:admin:test
```

## 部署说明

`apps/api` 使用 Wrangler 配置部署：

```sh
pnpm --filter @repo/api deploy:test
pnpm --filter @repo/api deploy:production
```

前端的生产和测试 API 地址需要通过对应的 `.env.production`、`.env.test` 或平台环境变量配置。
