# 策舟 StratArk

策舟 StratArk 是面向加密资产量化交易的全栈平台。根仓库负责聚合前端、后端和共享品牌资源；前端与后端保留独立 Git 历史，并通过 Git submodule 固定到可复现的提交。

## 目录

- [仓库结构](#仓库结构)
- [快速开始](#快速开始)
- [本地开发](#本地开发)
- [配置边界](#配置边界)
- [生产部署](#生产部署)
- [子仓库维护](#子仓库维护)
- [文档](#文档)
- [贡献](#贡献)
- [许可证](#许可证)

## 仓库结构

```text
strat-ark/
├── strat-ark-frontend/   # React 18 + TypeScript + Vite 6 前端子仓库
├── strat-ark-backend/    # FastAPI + SQLAlchemy async 后端子仓库及全栈编排
├── assets/               # 共享品牌与设计资源
├── AGENTS.md             # 项目级 AI 协作规范
├── .gitmodules           # 子仓库地址与挂载路径
└── LICENSE               # MIT License
```

| 子仓库 | 职责 | 技术栈 | 独立文档 |
| --- | --- | --- | --- |
| `strat-ark-frontend` | Web 界面、鉴权会话、交易与平台管理交互 | React 18、TypeScript、Vite 6、React Router、GSAP | [Frontend README](strat-ark-frontend/README.md) |
| `strat-ark-backend` | API、数据、权限、审计、订阅及引擎编排 | Python 3.13+、FastAPI、SQLAlchemy async、PostgreSQL、Redis | [Backend README](strat-ark-backend/README.md) |

## 快速开始

### 1. 克隆完整仓库

首次克隆时同时拉取子仓库：

```sh
git clone --recurse-submodules <repository-url> strat-ark
cd strat-ark
```

已经克隆根仓库但目录为空时执行：

```sh
git submodule update --init --recursive
```

### 2. 使用 Docker Compose 启动全栈

全栈编排文件由后端子仓库维护。先确认本机没有需要复用的 PostgreSQL 或 Redis 实例占用 `5432`、`6379`，再启动服务：

```sh
cd strat-ark-backend
cp .env.compose.example .env
docker compose up -d --build
docker compose exec backend python -m scripts.seed
```

默认入口：

- 前端：<http://localhost:3000>
- 后端 API 文档：<http://localhost:8000/docs>
- PostgreSQL：仅本机 `127.0.0.1:5432`
- Redis：仅本机 `127.0.0.1:6379`

`scripts.seed` 用于写入演示账号和内置策略，可按需执行。后端镜像会在启动应用前运行 `alembic upgrade head`，迁移失败会中止后端启动。

停止应用容器时使用 `docker compose down`。除非明确要删除本地数据库数据，不要追加 `-v`。

## 本地开发

本地开发优先使用各子项目原生命令；PostgreSQL、Redis 可以复用现有服务，也可以只用 Compose 启动必要基础设施。

### 后端

环境要求：Python 3.13+、Poetry、PostgreSQL、Redis。

```sh
cd strat-ark-backend
poetry install
cp .env.example .env
poetry run alembic upgrade head
poetry run python -m scripts.seed
poetry run uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

数据库连接必须使用 `postgresql+psycopg://`。执行迁移和 seed 前，按实际环境填写 `.env` 中的数据库、Redis 与鉴权配置，不要猜测已有服务凭据。

后端校验：

```sh
poetry run ruff check .
poetry run python -m unittest discover -s tests -v
```

### 前端

环境要求：Node.js 20+、npm。

```sh
cd strat-ark-frontend
npm ci
cp .env.example .env.local
npm run dev
```

默认开发地址为 <http://localhost:3000>。前端通过 `VITE_API_BASE_URL` 访问后端，默认值为 `http://localhost:8000`。

前端校验：

```sh
npm run typecheck
npm run i18n:check
npm run build
```

## 配置边界

- 根仓库不保存 `.env`；前端和后端分别维护自己的示例文件与运行配置。
- 后端密钥、数据库地址、第三方凭据和跨环境部署参数通过后端 `.env` 或部署环境注入。
- 前端只使用 `VITE_` 前缀的公开构建配置，不得存放服务端密钥。
- Freqtrade、TradingAgents 和模型网关的连接信息以管理端配置和后端数据库为单一事实源，不在根仓重复维护。
- Stripe 只有在 API 密钥与 Webhook 密钥完整配置后才启用付费流程；未配置时不会本地伪造订阅成功。

完整配置项分别见 [前端环境变量说明](strat-ark-frontend/README.md#前端环境变量) 和 [后端环境变量说明](strat-ark-backend/README.md#环境变量)。

## 生产部署

### 全栈容器

生产编排位于后端子仓库：

```sh
cd strat-ark-backend
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build
```

生产启动前必须从 `.env.compose.example` 建立部署环境配置，并为数据库、JWT、`ENCRYPT_KEY`、支付及其它已启用的外部服务注入真实值。不要把 `.env` 提交到任一仓库。

### Web 前端静态部署与 SPA fallback

生产前端应先执行 `npm run build`，Web 服务器根目录必须指向 `strat-ark-frontend/dist/`，不能指向仓库根目录或 `src/`。项目使用 React Router `BrowserRouter`，深层路由需要回退到 `index.html`；真实静态资源缺失时必须返回 404。

Nginx 或宝塔可使用以下规则。当前前端 API 默认跨域访问 `VITE_API_BASE_URL`，因此这里不添加 `/api/` 反向代理：

```nginx
root /path/to/strat-ark-frontend/dist;
index index.html;

location = /index.html {
    add_header Cache-Control "no-cache";
    try_files /index.html =404;
}

location = /runtime-config.js {
    add_header Cache-Control "no-store";
    try_files $uri =404;
}

location /assets/ {
    expires 1y;
    add_header Cache-Control "public, max-age=31536000, immutable";
    try_files $uri =404;
}

location / {
    try_files $uri $uri/ /index.html;
}
```

生产验收至少包括：

1. 访问首页并完成登录入口加载。
2. 直接访问并刷新 `/dashboard`、`/pricing` 等深层路由。
3. 如启用 OAuth，刷新 `/callback` 并确认授权状态能继续处理。
4. 直接访问一条构建后的 JS 或 CSS 资源并确认缓存头正确。
5. 访问不存在的 `/assets/*.js`，确认返回 404 而不是 `index.html`。
6. 验证浏览器实际请求的 API 基址、CORS 与 HTTPS 配置符合目标环境。

## 子仓库维护

根仓库只记录子仓库提交指针。修改代码时先在对应子仓库完成提交和推送，再更新根仓库指针：

```sh
git -C strat-ark-frontend status
git -C strat-ark-backend status
git submodule status
```

拉取根仓库记录的精确版本：

```sh
git pull
git submodule update --init --recursive
```

更新到两个子仓库远端分支的最新提交需要显式执行，完成验证后再提交根仓库中的 gitlink 变更：

```sh
git submodule update --remote --merge
git status --short
```

不要直接在 detached HEAD 状态下提交子仓库代码；进入子仓库后先切换到其协作分支。

## 文档

- [前后端接口契约](strat-ark-backend/docs/前后端接口契约.md)
- [Stripe 订阅计费架构](strat-ark-backend/docs/Stripe订阅计费架构.md)
- [后端主机部署](strat-ark-backend/deploy/README.md)
- [前端部署说明](strat-ark-frontend/README.md#文档与部署)

## 贡献

1. 在目标子仓库基于现有协作分支开发，不在根仓库混入前后端源码提交。
2. 遵循根目录 `AGENTS.md` 和子仓库当前代码约定。
3. 提交前运行对应类型检查、测试或构建命令。
4. 子仓库提交推送后，再在根仓库提交更新后的 submodule 指针。
5. 提交标题使用 `type: 中文描述`，一次提交只包含一类明确变更。

## 许可证

本根仓库采用 [MIT License](LICENSE)。前端、后端是独立仓库，根仓许可证不会自动改变两个子仓库的授权状态。
