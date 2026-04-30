# admin

管理后台 Next.js 应用，本地默认运行在 `http://localhost:3006`。

## 职责

- 验证管理后台可以消费 `@repo/ui` 的共享组件。
- 展示服务端环境变量和浏览器公开环境变量。
- 验证每个前端应用都能独立编译 Tailwind，同时扫描共享 UI 包里的 class。
- 作为第二个前端应用，保证 monorepo 内共享主题和基础组件能跨应用复用。

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
cp apps/admin/.env.example apps/admin/.env.development
```

## 常用命令

在仓库根目录执行：

```sh
pnpm dev:admin
pnpm --filter admin check-types
pnpm --filter admin lint
pnpm --filter admin build
```

也可以进入 `apps/admin` 后执行：

```sh
pnpm dev
pnpm check-types
pnpm lint
pnpm build
```

测试环境相关命令：

```sh
pnpm --filter admin dev:test
pnpm --filter admin build:test
pnpm --filter admin start:test
```

## 关键文件

- `app/page.tsx`：管理后台页面和共享 UI 验证区。
- `src/env.server.ts`：服务端环境变量校验。
- `src/env.client.ts`：浏览器公开环境变量校验。
- `src/admin-env-badge.tsx`：客户端环境变量展示组件。
- `app/globals.css`：Tailwind 和共享 UI theme 入口。
