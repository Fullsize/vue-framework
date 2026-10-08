# Vue Framework — Vue 3 + TypeScript 管理后台脚手架

[English](README.md) | **简体中文**

Vue Framework 是一个基于 Vue 3、TypeScript、Vite、Ant Design Vue、Pinia 和 Vue Router 的管理后台项目模板。它包含 Vue 单文件组件、TSX 示例、角色路由守卫、可复用的 Composition API Hooks，以及 Docker + Nginx 部署配置。

适合用作内部工具、管理后台的开发起点，也适合学习 Vue 3 项目的组织方式。登录流程和后台数据均为本地演示，实际项目需要接入自己的后端。

## 目录

- [功能特性](#功能特性)
- [快速开始](#快速开始)
- [演示登录与路由](#演示登录与路由)
- [项目结构](#项目结构)
- [扩展项目](#扩展项目)
- [构建与部署](#构建与部署)
- [参与贡献](#参与贡献)
- [许可证](#许可证)

## 功能特性

- **Vue 3 + TypeScript**：使用 Composition API 和 `<script setup>` 编写组件。
- **Vite + TSX**：配置 Vue SFC、JSX/TSX 和旧浏览器兼容插件，兼容目标排除 IE 11。
- **Ant Design Vue**：用于登录页面和 TSX 示例的 UI 组件。
- **Pinia**：管理登录状态，并从 `localStorage` 恢复 token 和角色。
- **Vue Router**：Hash 路由、登录守卫与 `admin` 角色限制。
- **后台页面示例**：控制面板、用户管理、系统设置和可搜索的日志列表。
- **React 风格的 Vue Hooks**：包含 `useState`、`useSetState`、`useRef`、`useEffect`、`useLayoutEffect`、`useEffectOnce`。
- **Axios 请求模块**：提供基础 URL、超时设置和拦截器骨架。
- **静态部署**：Docker 分阶段构建与 Nginx 静态托管、gzip 配置。

## 快速开始

使用 Node.js 20 时建议选择 **20.19+**，也可以使用兼容的更高版本。**pnpm 8** 可匹配仓库现有锁文件格式。Docker 和 CI 目前使用 Node 20。

```bash
git clone https://github.com/Fullsize/vue-framework.git
cd vue-framework
npm install --global pnpm@8
pnpm install
pnpm dev
```

打开 [http://localhost:3000](http://localhost:3000)。如果端口被占用，以 Vite 终端输出的地址为准。

| 命令 | 用途 |
| --- | --- |
| `pnpm dev` | 启动开发服务器 |
| `pnpm build` | 执行 `vue-tsc -b`，再构建到 `dist/` |
| `pnpm preview` | 在本地预览构建产物 |

当前 `lint` 脚本为空，尚未配置自动化测试脚本。

## 演示登录与路由

未登录访问受保护页面时，会跳转到 `/login`。浏览器地址采用 Hash 形式，例如 `http://localhost:3000/#/admin`。

登录表单默认填入用户名 `abc`、密码 `123`，登录后角色为 `user`。将用户名改为 **`abc123`** 可体验 `admin` 角色。密码不做校验：演示 token 由用户名和密码直接拼接而成，与角色一起存入 `localStorage`。

| 路由 | 页面 | 访问条件 |
| --- | --- | --- |
| `/login` | 演示登录页 | 公开 |
| `/` | 首页示例 | 已登录 |
| `/test` | TSX 与状态 Hook 示例 | 已登录 |
| `/proxy` | Proxy 示例 | 已登录 |
| `/admin` | 管理员控制面板 | `admin` |
| `/admin/users` | 用户管理示例 | `admin` |
| `/admin/settings` | 系统设置示例 | `admin` |
| `/admin/logs` | 日志搜索与筛选 | `admin` |
| `/forbidden` | 无权限页面 | 已登录 |

该登录流程仅演示客户端页面访问控制。实际使用时应替换为后端认证，并在服务端验证权限。后台页面使用本地示例数据，保存设置、导出日志等部分操作仍为占位实现。应用界面目前使用中文；README 语言切换仅切换文档。

## 项目结构

```text
vue-framework/
├── .github/workflows/     # 构建与 GitHub Pages 工作流
├── public/               # 公共静态资源
├── src/
│   ├── assets/images/    # 导入使用的图片
│   ├── components/       # 公共组件
│   ├── hooks/            # Vue 状态与生命周期工具
│   ├── layout/           # 应用布局与 RouterView
│   ├── pages/            # 登录、后台与示例页面
│   ├── routes/           # 路由与导航守卫
│   ├── service/          # Axios 实例与拦截器
│   ├── store/            # Pinia 状态管理
│   ├── main.ts           # 应用初始化
│   └── style.css         # 全局样式
├── Dockerfile            # Node 构建与 Nginx 运行环境
├── nginx.conf            # 静态托管与 gzip
└── vite.config.ts        # 插件、别名、端口与资源路径
```

## 扩展项目

### 添加页面

在 `src/pages/` 创建组件，然后在 [src/routes/base.ts](src/routes/base.ts) 导入并将路由加入导出的数组：

```ts
import Reports from '@/pages/Reports.vue'

// 加入路由数组：
{
  path: '/reports',
  component: Reports,
  meta: {
    requiresAuth: true,
    requireRole: ['admin'],
  },
}
```

所有已登录用户都可访问的页面可以省略 `requireRole`。路由守卫位于 [src/routes/index.ts](src/routes/index.ts)，登录状态位于 [src/store/auth.ts](src/store/auth.ts)。

### 使用 Vue Hooks 编写 TSX

项目同时支持 `.vue` 和 `.tsx` 组件。`@` 别名指向 `src/`，`@images` 指向 `src/assets/images/`。

```tsx
import { defineComponent } from 'vue'
import { useState } from '@/hooks'

export default defineComponent({
  setup() {
    const [count, setCount] = useState(0)
    return () => (
      <button onClick={() => setCount(previous => previous + 1)}>
        Count: {count.value}
      </button>
    )
  },
})
```

这些工具基于 Vue ref、watch 和生命周期实现。命名借鉴 React，具体行为以项目中的 Vue 实现为准。可参考 [src/hooks/](src/hooks/) 和 [src/pages/Test/index.tsx](src/pages/Test/index.tsx)。

### 接入后端

在 [src/service/request.ts](src/service/request.ts) 中配置 API 地址、超时、请求头和错误处理。当前 `https://some-domain.com/api/` 为占位地址。在 [src/pages/Login.vue](src/pages/Login.vue) 接入登录接口，并将后台页面的本地数据替换为 API 返回值。

## 构建与部署

### 静态托管

```bash
pnpm build
pnpm preview
```

将 `dist/` 目录内容部署到静态托管服务。Vite 使用 `base: './'` 生成相对资源路径；Hash 路由在浏览器中处理 `#` 后的页面导航。

仓库包含 GitHub Pages 工作流，通过推送到 `master` 或手动运行触发。使用时需要在仓库设置中启用 GitHub Pages，并选择 **GitHub Actions** 作为发布来源。

### Docker

安装 Docker 并启用 BuildKit 后，在仓库根目录执行：

```bash
docker build -t vue-framework .
docker run --rm -p 8080:80 vue-framework
```

打开 [http://localhost:8080](http://localhost:8080)。镜像通过 Node 20 构建，再由 Nginx 托管。Dockerfile 当前通过 `https://registry.npmmirror.com` 安装 pnpm，可按运行环境调整镜像源。

## 参与贡献

欢迎[提交 Issue](https://github.com/Fullsize/vue-framework/issues) 反馈问题或提出功能建议，也可以提交范围明确的 Pull Request。反馈问题时请提供复现步骤，提交代码变更前请运行 `pnpm build`。

## 许可证

本项目采用 [Apache License 2.0](LICENSE)。
