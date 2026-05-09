# ai-agent

这是一个正在建设中的 TypeScript monorepo 项目，包含两个 Next.js 前端应用、一个基于 Hono 的 Cloudflare Workers API、共享 UI 组件、类型共享与 RPC 调用，以及通用的 lint/type 配置。

当前仓库主要完成了项目基础架构、共享组件体系、环境变量校验、API 路由拆分，以及前后端之间的类型共享与 RPC 调用。现在已经包含 system、catalog、user、order 等示例接口、共享 HTTP 请求模块和对应的 Web 验证页面，后续会在这个基础上继续补充真实业务页面、后台管理能力和后端业务接口。

## 架构

```text
ai-agent
├── apps
│   ├── web      面向用户侧的 Next.js 应用，本地端口 3005
│   ├── admin    管理后台 Next.js 应用，本地端口 3006
│   └── api      基于 Hono 的 Cloudflare Workers API，本地 Wrangler 端口 8787
├── packages
│   ├── contracts            共享请求/响应类型、zod schema、业务错误码
│   ├── ui                   共享 Tailwind 主题和 React UI 基础组件
│   ├── eslint-config        共享 ESLint 配置
│   └── typescript-config    共享 TypeScript 配置
├── pnpm-workspace.yaml      workspace 包声明和依赖 catalog
└── turbo.json               Turbo 任务图和环境变量透传配置
```

目前的请求流程：

```text
apps/web 或 apps/admin
  -> 共享 HTTP 模块 / 环境变量 API_BASE_URL 或 NEXT_PUBLIC_API_BASE_URL
  -> apps/api Hono routes
  -> packages/contracts schema 和响应结构
```

`packages/contracts` 负责类型共享，放公共的请求/响应类型、zod schema 和业务错误码。前端通过 `apps/web/src/http` 里的共享 HTTP 模块调用 API，接口入参和响应类型复用这里的定义；`apps/api` 也从这里引入同一套 schema 来校验接口入参，并通过统一的成功/失败响应结构返回数据。

## 应用和包

- `apps/web`：用户侧 Next.js 主应用，当前已接入基础页面、共享 UI、共享 HTTP 模块，以及 system/catalog/user/order 的服务端和客户端接口验证页面。
- `apps/admin`：管理后台 Next.js 应用，当前已接入共享 UI 和环境变量校验，后续承载后台管理功能。
- `apps/api`：Hono API 应用，当前已提供分模块路由、统一响应结构、请求校验和 `AppType` 导出，后续扩展业务接口。
- `packages/contracts`：负责类型共享，提供 `ApiResponse`、`BizCode`、system/catalog/user/order 等接口 schema 和类型定义。
- `packages/ui`：共享 Tailwind CSS 主题和 `Button`、`Card`、`Input`、`Label`、`Separator`、`TailwindDemo` 等组件。
- `packages/eslint-config`：供各应用和包复用的 ESLint 配置。
- `packages/typescript-config`：供各应用和包复用的 TypeScript 配置。

## 当前状态

- 已完成 monorepo workspace、Turbo 任务、共享依赖 catalog。
- 已完成 web/admin 两个前端应用的基础 Next.js 配置。
- 已完成 api 应用的 Hono/Wrangler 基础配置和分模块路由组织。
- 已完成 `@repo/contracts` 里的统一响应结构、业务错误码，以及 system/catalog/user/order 的类型定义。
- 已完成 `@repo/ui` 的基础主题和组件接入。
- 已跑通 web 到 api 的 RPC 调用，并复用了同一套请求/响应类型。
- 已新增 `apps/web/src/http` 作为统一请求入口，会根据运行环境自动选择服务端 `API_BASE_URL` 或浏览器端 `NEXT_PUBLIC_API_BASE_URL`。
- 已提供 Web 侧服务端和客户端接口验证页面，用于快速确认前后端类型、请求和响应是否一致。

## API 示例

当前 API 暴露以下示例接口：

```text
GET  /health
POST /rpc/system/ping
GET  /rpc/catalog/list
GET  /rpc/user/profile
POST /rpc/order/detail
```

对应的 Web 验证页面：

```text
/verify/system/health
/verify/system/ping
/verify/catalog/list
/verify/user/profile
/verify/order/detail
/verify/client/system/ping
```

## 后续方向

- 补充用户侧真实业务页面和交互流程。
- 补充管理后台的业务模块、表单、列表和权限相关能力。
- 扩展 API 的业务路由、数据访问层和错误处理策略。
- 持续沉淀共享 UI 组件、类型共享、RPC 调用方式和工程规范。

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

提交或推送前执行这组检查：

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
