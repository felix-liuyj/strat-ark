# 策舟 StratArk

[![React][react-shield]][frontend-url]
[![FastAPI][fastapi-shield]][backend-url]
[![Python][python-shield]][python-url]
[![PostgreSQL][postgresql-shield]][postgresql-url]
[![MIT License][license-shield]][license-url]

<p align="center">
  <strong>AI 加密资产量化交易平台</strong>
  <br />
  聚合 React Web 前端与 FastAPI 后端，通过 Git submodule 固定双仓版本，统一承载品牌资源、全栈启动入口和协作文档。
</p>

## 目录

- [快速开始](#快速开始)
- [项目结构](#项目结构)
- [子仓库](#子仓库)
- [核心功能](#核心功能)
- [技术栈](#技术栈)
- [环境配置](#环境配置)
- [本地开发](#本地开发)
- [生产部署](#生产部署)
- [子仓库维护](#子仓库维护)
- [文档链接](#文档链接)
- [贡献](#贡献)
- [许可证](#许可证)
- [联系方式](#联系方式)

## 快速开始

### 环境要求

- Git，需支持 submodule
- Docker Engine 与 Docker Compose 插件
- 可用端口：前端 `3000`、后端 `8000`
- 本地基础设施端口：PostgreSQL `5432`、Redis `6379`

如本机已有 PostgreSQL 或 Redis 服务，先确认是否需要复用以及端口是否冲突，不要直接启动重复实例或重置已有数据。

### 克隆完整仓库

```sh
git clone --recurse-submodules https://github.com/felix-liuyj/strat-ark.git
cd strat-ark
```

如果已经克隆根仓库但尚未拉取子仓库：

```sh
git submodule update --init --recursive
```

### 启动全栈服务

全栈编排文件由后端子仓库维护：

```sh
cd strat-ark-backend
cp .env.compose.example .env
docker compose up -d --build
```

按需写入演示账号和内置策略：

```sh
docker compose exec backend python -m scripts.seed
```

默认入口：

- 前端：<http://localhost:3000>
- Swagger UI：<http://localhost:8000/docs>
- ReDoc：<http://localhost:8000/redoc>
- PostgreSQL：仅本机 `127.0.0.1:5432`
- Redis：仅本机 `127.0.0.1:6379`

后端镜像会在应用启动前执行 `alembic upgrade head`，迁移失败时中止启动。停止服务使用 `docker compose down`；除非明确需要删除本地数据库数据，不要追加 `-v`。

## 项目结构

```text
strat-ark/
├── strat-ark-frontend/   # React Web 前端子仓库
├── strat-ark-backend/    # FastAPI 后端子仓库及全栈编排
├── assets/               # 共享品牌与设计资源
├── AGENTS.md             # 项目级 AI 协作规范
├── .gitmodules           # 子仓库地址与挂载路径
├── README.md             # 根仓入口与协作说明
└── LICENSE               # 根仓 MIT License
```

根仓库只保存共享文件和子仓库提交指针，不复制前端或后端源码。克隆根仓后，两个子仓库会定位到根仓提交所记录的精确版本。

## 子仓库

| 子仓库 | 职责 | 默认分支 | 独立文档 |
| --- | --- | --- | --- |
| [`strat-ark-frontend`](strat-ark-frontend) | Web 界面、会话鉴权、交易及平台管理交互 | `main` | [Frontend README](strat-ark-frontend/README.md) |
| [`strat-ark-backend`](strat-ark-backend) | API、数据、权限、审计、订阅和引擎编排 | `main` | [Backend README](strat-ark-backend/README.md) |

子仓库拥有独立 Git 历史、远端、提交和发布边界。修改任一子仓库后，先在该子仓完成提交与推送，再提交根仓中的 gitlink 更新。

## 核心功能

- AI 投研：TradingAgents 多智能体市场分析、信号复核、回测复盘和结构化报告。
- 量化执行：策略、回测、交易信号、交易机器人、持仓和订单管理。
- 实盘编排：一机器人一 Freqtrade 实例，统一管理实例生命周期、连接和日志。
- 风险与审计：风险规则、触发记录、管理员操作审计和审计链校验。
- 账户与安全：邮箱认证、OAuth Public PKCE、双 token 刷新、权限守卫和 2FA。
- 订阅计费：Stripe Checkout、Customer Portal、Webhook 驱动的订阅和发票同步。
- 平台能力：行情、交易所连接、通知渠道、OSS 直传、国际化和深浅主题。

完整业务模块、路由和边界以两个子仓库的当前实现与独立 README 为准。

## 技术栈

### Web 前端

- React 18
- TypeScript strict
- Vite 6
- React Router 6
- GSAP 与 `@gsap/react`
- Nginx 静态托管

### 后端

- Python 3.13+
- FastAPI、Uvicorn、Gunicorn
- Pydantic v2 与 pydantic-settings
- SQLAlchemy 2.0 Async ORM、psycopg 3、Alembic
- PostgreSQL、Redis
- PyJWT、bcrypt、PyOTP

### 外部服务与基础设施

- Freqtrade 执行引擎
- TradingAgents 投研服务
- Stripe Billing
- 阿里云 OSS
- Docker Compose

## 环境配置

| 范围 | 配置入口 | 用途 |
| --- | --- | --- |
| 根仓库 | 无 `.env` | 根仓不保存运行时密钥或服务配置 |
| 全栈 Compose | `strat-ark-backend/.env.compose.example` | PostgreSQL、Redis、后端、前端构建及生产编排 |
| 后端本地开发 | `strat-ark-backend/.env.example` | 数据库、Redis、JWT、OAuth、Stripe、OSS、SMTP 等运行配置 |
| 前端本地开发 | `strat-ark-frontend/.env.example` | `VITE_API_BASE_URL`、标题、语言、主题和 OAuth public client ID |
| 引擎连接 | 管理端引擎配置 | Freqtrade、TradingAgents 和模型网关的地址、凭据与模型参数 |

配置约束：

- `.env` 和 `.env.local` 不得提交到根仓或子仓库。
- 前端只允许持有 `VITE_` 前缀的公开配置，不得保存服务端密钥。
- Freqtrade、TradingAgents 和模型网关以管理端配置与后端数据库为单一事实源，不在根仓重复维护。
- Stripe 只有在 API 密钥与 Webhook 密钥完整配置后才启用付费流程；未配置时不得本地伪造订阅成功。
- 生产环境必须替换默认数据库密码、JWT 密钥并固化 `ENCRYPT_KEY`，同时按实际启用的外部服务补齐凭据。

详细字段见 [前端环境变量说明](strat-ark-frontend/README.md#前端环境变量) 和 [后端环境变量说明](strat-ark-backend/README.md#环境变量)。

## 本地开发

本地开发使用各子项目原生命令。PostgreSQL、Redis 可以复用已有服务，也可以只通过 Compose 启动必要基础设施。

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

数据库连接必须使用 `postgresql+psycopg://`。执行迁移和 seed 前，按实际环境填写 `.env`，不要猜测已有基础设施的账号、密码、库名或端口。

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

默认开发地址为 <http://localhost:3000>，默认后端基址为 `http://localhost:8000`。

前端校验：

```sh
npm run typecheck
npm run i18n:check
npm run build
```

## 生产部署

### 全栈容器

```sh
cd strat-ark-backend
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build
```

生产启动前应从 `.env.compose.example` 建立部署配置，并为数据库、JWT、`ENCRYPT_KEY`、支付及已启用的外部服务注入真实值。

### Web 前端静态部署与 SPA fallback

生产前端先执行 `npm run build`，Web 服务器根目录必须指向 `strat-ark-frontend/dist/`，不能指向仓库根目录或 `src/`。项目使用 React Router `BrowserRouter`，深层路由需要回退到 `index.html`；真实静态资源缺失时必须返回 404。

Nginx 或宝塔可使用以下规则。前端默认通过 `VITE_API_BASE_URL` 跨域访问 API，因此示例不添加 `/api/` 反向代理：

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

生产验收：

1. 访问首页并确认登录入口正常加载。
2. 直接访问并刷新 `/dashboard`、`/pricing` 等深层路由。
3. 如启用 OAuth，刷新 `/callback` 并确认授权状态可以继续处理。
4. 直接访问一条构建后的 JS 或 CSS 资源并确认缓存头正确。
5. 访问不存在的 `/assets/*.js`，确认返回 404 而不是 `index.html`。
6. 验证浏览器实际请求的 API 基址、CORS 和 HTTPS 配置符合目标环境。

## 子仓库维护

查看根仓记录的版本和两个子仓库的工作区状态：

```sh
git submodule status
git -C strat-ark-frontend status --short --branch
git -C strat-ark-backend status --short --branch
```

拉取根仓库记录的精确版本：

```sh
git pull
git submodule update --init --recursive
```

显式更新到两个子仓库远端 `main` 的最新提交：

```sh
git submodule update --remote --merge
git status --short
```

不要在 detached HEAD 状态下提交子仓库代码。进入子仓库后先切换到协作分支，完成实现、校验、提交和推送，再回到根仓库提交更新后的子仓库指针。

## 文档链接

- [Frontend README](strat-ark-frontend/README.md)
- [Backend README](strat-ark-backend/README.md)
- [前后端接口契约](strat-ark-backend/docs/前后端接口契约.md)
- [Stripe 订阅计费架构](strat-ark-backend/docs/Stripe订阅计费架构.md)
- [后端主机部署](strat-ark-backend/deploy/README.md)
- [Swagger UI](http://localhost:8000/docs)
- [ReDoc](http://localhost:8000/redoc)

## 贡献

1. 在目标子仓库基于现有协作分支开发，不在根仓库混入前后端源码提交。
2. 遵循根目录 `AGENTS.md` 和子仓库当前代码约定。
3. 提交前运行对应类型检查、测试或构建命令。
4. 子仓库提交推送后，再在根仓库提交更新后的 submodule 指针。
5. 提交标题使用 `type: 中文描述`，一次提交只包含一类明确变更。

## 许可证

本根仓库采用 [MIT License](LICENSE)。前端、后端是独立仓库，根仓许可证不会自动改变两个子仓库的授权状态。

## 联系方式

Felix Liu - felixliuyj@gmail.com

[react-shield]: https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=white
[frontend-url]: https://github.com/felix-liuyj/strat-ark-frontend
[fastapi-shield]: https://img.shields.io/badge/FastAPI-ready-009688?logo=fastapi&logoColor=white
[backend-url]: https://github.com/felix-liuyj/strat-ark-backend
[python-shield]: https://img.shields.io/badge/Python-3.13%2B-3776AB?logo=python&logoColor=white
[python-url]: https://www.python.org/
[postgresql-shield]: https://img.shields.io/badge/PostgreSQL-async-4169E1?logo=postgresql&logoColor=white
[postgresql-url]: https://www.postgresql.org/
[license-shield]: https://img.shields.io/badge/License-MIT-yellow.svg
[license-url]: LICENSE
