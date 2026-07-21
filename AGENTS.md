# Strat Ark AI 协作规范

适用范围：本文件位于 `strat-ark` 根目录，覆盖 `strat-ark-backend`、`strat-ark-frontend` 以及根目录配置文件。当前根目录不是 Git 仓库，后端与前端是独立 Git 仓库；提交时必须分别进入对应子仓库。

## 一、通用开发与 AI 协作规范

核心条款：

- 思考、分析、答复使用简体中文；代码标识符、第三方 API 保持英文。
- 严禁任何 Emoji，包括回复、文档、注释、文案、占位符、提交示例、按钮、状态标记、菜单、空态、提示。
- 前端 icon 统一使用内嵌 SVG。
- 禁止 `import *`。
- 方法短小，有效代码原则上不超过 20 行；复杂逻辑拆成清晰私有函数。
- 优先从具体子模块导入，而不是从模糊顶层聚合导入。
- 命名按子项目既有约定：后端 Python 使用 `snake_case`；React 组件使用 `PascalCase`；类使用 `PascalCase`；常量使用 `UPPER_SNAKE_CASE`。
- 临时调试脚本不入 Git；长期保留的工具脚本放 `tests/` 或明确工具目录。
- 修改代码时默认保留原有注释；注释已失效或与代码不符时才修正。
- 输出优先可执行内容：命令、代码、路径、校验步骤。
- 当前代码现状优先于抽象模板规则。

例外：

- 代码标识符使用英文。
- API 调用和第三方库接口保留英文。
- 技术术语和专有名词可保留英文原词。
- 引用外部文档或代码可保持原文。

## 二、Git 提交规范

提交标题：

- 格式：`type: 中文描述`。
- 必须使用 Conventional Commits 前缀。
- `type` 不允许使用括号携带补充说明，例如禁止 `feat(main)`。
- 中文描述必须直接说明结果，禁止 `update code`、`fix issue` 等模糊标题。
- 严禁 Emoji。
- 禁止纯英文标题，除非用户明确要求。

常用类型：

- `feat`：新功能、页面、接口、业务能力。
- `fix`：Bug、字段对齐、兼容、流程修复。
- `refactor`：结构优化且不改变业务行为。
- `chore`：配置、忽略、脚本、工程杂项。
- `style`：纯样式或排版；存在行为变化时优先使用 `feat` 或 `fix`。
- `docs`：只有文档改动时使用。

提交粒度：

- 一次提交只做一类明确变更。
- 不混入无关文件、临时文件、顺手格式化、调试输出。
- 多端同时改动时，在各自仓库独立提交。
- 根目录不是 Git 仓库，禁止在根目录提交。

自动提交与推送：

- 当前子项目存在 Git 仓库时，修改完成后自动在当前分支 commit 并 push，无需用户再次要求。
- 不为自动提交新建分支。
- 首次 push 使用 `-u`。
- 改写已推送历史才使用 `--force-with-lease`。
- 连接外部服务、部署等其它外向操作不在自动范围内，仍需授权。
- 不允许增加 `Co-Authored-By: Claude`、`Co-Authored-By: Codex` 等 AI 署名。

历史改写：

- 修改已推送的 commit message 属于改写历史。
- 推送使用 `--force-with-lease`。
- 改 message 时不改提交内容。
- 改写前如果存在未提交修改，先 stash。

## 三、后端框架与开发规范

当前后端：`strat-ark-backend`，Python 3.13+、Poetry、FastAPI、SQLAlchemy async、Redis、JWT、审计、阿里云 OSS。

核心条款：

- 类型提示为最高优先级，所有函数、方法、`__init__`、`@property` 必须完整标注。
- 使用 `X | None`，不要使用 `Optional[X]`。
- 使用内置泛型 `list`、`dict`、`tuple`、`set`。
- 三层架构：API 路由层只做参数映射和依赖注入，ViewModel 承载业务逻辑，响应层只输出 Pydantic response。
- ViewModel 使用 `async with` 生命周期；`before()` 必须先 `await super().before()`。
- Forms 命名：`{Resource}CreateForm`、`{Resource}UpdateForm`；继承 `ApiFormModel`，字段直接使用 camelCase，`ConfigDict(extra="forbid")`。
- Forms 使用 `Body(..., embed=True)`；路由中作为单一参数注入；ViewModel 通过 `self.form.xxx` 访问；禁止在路由或 ViewModel 内散落内联 `Body(...)`。
- Response 数据模型继承 `ApiResponseModel`，字段直接使用 camelCase，`ConfigDict(extra="forbid")`。
- Response 使用 `pydantic.Field`；禁止直接返回 ORM 对象，必须逐字段手动映射。
- 审计通过 `audit_action`、`audit_resource` 类属性或 `@audit_log` 装饰器配置；辅助方法使用 `set_audit_resource_id`、`set_audit_changes`、`set_audit_metadata`。
- RESTful 路由文件内 `router = APIRouter()` 不设置 prefix。
- 禁止 `/search`、`/list`、`/create` 这类非 RESTful 路径。
- 资源级动作使用 `/{resource}/{id}/{action}`。
- Query、Path 使用 `snake_case`；Body、Form 使用 `camelCase`。
- 每个端点必须设置 `tags`、`summary`、`description`。
- `forms/`、`responses/`、`view_models/`、`models/` 必须从具体子模块导入。
- 序列解包使用 `first, *_ = data`。
- 字典合并优先使用 `|=`，不要连续写多个 `d["k"] = v`。

数据库与持久层：

- 本项目持久层使用 SQLAlchemy 2.x 异步 + PostgreSQL；按该栈编写规则和实现。
- API 路由层不碰查询；数据库读写放在 ViewModel、服务层或明确的基础设施层。
- 当前项目允许 ViewModel 直接持有 `AsyncSession` 并执行 `select` / `commit`；重复查询、跨模块拼接或复杂校验应抽成短小 helper 或服务函数。
- 持久层深处可抛 `AppServiceException` 子类，由全局 handler 映射响应；ViewModel 浅层也可直接调用 `self.not_found()`、`self.forbidden()`、`self.illegal_parameters()` 等响应方法。
- Form/Response 与 ORM model 分离；snake_case ORM 属性映射到 camelCase response 字段时必须显式逐字段转换。

SQLAlchemy + PostgreSQL：

- `DATABASE_URL` 必须使用 `postgresql+psycopg://` 前缀；驱动不匹配时应在启动阶段直接失败。
- 引擎使用 `create_async_engine(url, pool_pre_ping=True, pool_recycle=1800)`，不要无理由移除断线检测和连接回收。
- 会话工厂使用 `async_sessionmaker(autocommit=False, autoflush=False, expire_on_commit=False)`。
- 请求级会话走 `get_db()` 依赖注入，响应结束即关闭。
- FastAPI `BackgroundTasks` 或请求外后台逻辑必须使用 `new_async_session()` 自建会话并显式 `commit()`；禁止复用请求级会话。
- `DATABASE_SCHEMA` 为空时使用默认 schema；非空时通过 `Base.metadata = MetaData(schema=...)` 绑定。
- 非 `public` schema 初始化前必须先创建 schema。
- 应用启动建表使用 `init_db()`；多 worker 并发启动时必须通过 PostgreSQL advisory lock 串行化 DDL。
- `init_db()` 只做空库 `create_all` 兜底和必要 seed；不要在普通查询接口中隐式建表、改 schema、seed 大量数据。
- schema 演进不要塞进应用启动流程；当前未配置 Alembic 时，改表必须通过明确迁移脚本或部署期操作执行。
- ORM model 放在 `models/` 具体业务模块，并确保 `models/__init__.py` 导入该 model，使 `Base.metadata.create_all` 能注册表。
- ORM model 继承 `Base`，需要时间字段时同时继承 `TimestampMixin`。
- 每个 ORM model 必须显式声明 `__tablename__`，命名使用复数 `snake_case`。
- 字段统一使用 SQLAlchemy 2 风格：`Mapped[T] = mapped_column(...)`。
- 主键默认使用 `id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)`；业务标识符使用独立字段和唯一索引，不占用主键。
- 列类型优先使用原生 `String(N)`、`Integer`、`Boolean`、`Float`、`Text`、`JSON`、`DateTime`。
- 枚举优先用 `StrEnum` 表达业务值；数据库列按当前项目模式使用 `String(...)` 存储枚举值，不使用 PostgreSQL 原生 ENUM。
- `server_default` 必须与 Python `default` 同向对齐，避免 DDL 与 ORM 视角不一致。
- JSON 字段使用 SQLAlchemy `JSON`，可变默认值必须用 `default=list` 或 `default=dict`。
- 时间字段复用 `TimestampMixin` 的 `created_at`、`updated_at`，不要在各 model 重复定义。
- 外键使用 `ForeignKey("table.column", ondelete="CASCADE")` 并按查询需要设置 `index=True`。
- 当前项目跨模块绑定默认不用 ORM relationship；跨模块名称、详情或权限校验通过显式 `select` 查询拼接。
- 查询优先使用 `select(Model)`、`db.scalar(...)`、`db.scalars(...)`、`db.get(Model, id)`；多行结果用 `(await db.scalars(stmt)).all()`。
- 可复用 `scalars_all(db, stmt)`、`scalars_first(db, stmt)` 简化查询结果读取。
- 新增记录使用 `db.add(...)`，需要主键参与后续插入时先 `await db.flush()`，完整事务成功后再 `await db.commit()`。
- 写入后需要返回最新字段时执行 `await db.refresh(model)`，或按业务需要重新查询并映射 response。
- 更新记录优先先查询 ORM 对象再逐字段赋值，最后 `commit`；部分更新必须区分未传与显式置空，不能简单用 `if value is not None` 覆盖所有场景。
- 删除记录使用 `await db.delete(model)` 后提交。
- 批量更新或删除可使用 SQLAlchemy `update(...)` / `delete(...)`，但必须明确过滤条件、影响范围和事务提交。

ViewModel 响应：

- 常用响应方法包括 `operating_successfully`、`empty_content`、`nothing_changed`、`operating_failed`、`unauthorized`、`forbidden`、`not_found`、`illegal_parameters`、`system_error`。
- `handled=True` 表示已处理，不再抛异常。

部署脚本：

- `deploy/deploy.sh` 必须幂等。
- 支持 `--uninstall`。
- 破坏性操作默认二次确认，`--force` 显式跳过确认。
- 路径、端口、worker 数走环境变量。
- 只管理本项目资源。
- 敏感信息不写入脚本。

## 四、Web 前端开发规范

当前前端：`strat-ark-frontend`，React 18、TypeScript、Vite 6、React Router、GSAP。

目录与分层：

- 代码优先保持现有目录结构。
- 页面、API、类型、工具函数不混放。
- 新功能必须同步检查路由注册、权限控制、API 封装、类型定义、页面实现。
- 前端环境变量使用 `VITE_API_BASE_URL`、`VITE_APP_TITLE` 等 `VITE_` 前缀变量。
- API 统一走项目现有 API client；页面和业务模块禁止散落 `fetch` 或 `axios` 初始化。

路由与权限：

- 路由入口按现有项目文件维护。
- 页面采用 lazy import。
- 访问控制集中在 `ProtectedRoute`、`AuthContext` 和权限工具中。
- 前端角色名不能直接使用后端返回值，必须先经过映射层。
- 菜单、页面、按钮权限使用同一来源。

TypeScript 与 React：

- strict 风格，避免隐式 `any`。
- 保持 `strictNullChecks` 与 `noImplicitAny` 开启。
- 函数使用 `camelCase`。
- 组件使用 `PascalCase`。
- Hook 必须使用 `use` 前缀。
- 页面局部状态用 React Hook；全局状态走 Context 或项目既有状态方案。
- 不在页面写大段匿名类型；请求、响应、列表项、表单类型放类型文件。

UI 与文案：

- 优先复用现有页面模式、表格、弹窗、筛选结构。
- 禁止 Emoji。
- icon 必须使用内嵌 SVG，组件内 `<svg>` 或 URL 编码后的 `data:image/svg+xml,<...>`，禁止 base64、icon font、外链 URL、字符图标。
- 公共基础类负责尺寸、对齐、`currentColor`、`mask`。
- 文案保持业务直接、清晰，避免解释型空话。
- 不做与当前页面风格不一致的营销式布局。
- UI 元素和文字不得重叠；移动端与桌面端都要检查文本是否溢出。

动画：

- 默认使用 GSAP。
- 序列编排使用 `gsap.timeline()`。
- 滚动驱动使用 `ScrollTrigger`。
- 响应式与 `prefers-reduced-motion` 使用 `gsap.matchMedia()`。
- React 中使用 `@gsap/react` 的 `useGSAP` 管理生命周期与清理。
- 存量动画不强制重写；同一组件不要混用两套动画库。

运行与校验：

- 遵循当前项目 package manager；未确认前不要假设存在 `lint` 或 `test` 脚本。
- 当前前端已有脚本：`build`、`typecheck`、`i18n:check`、`preview`、`dev`。
- 修改前端后，优先运行 `npm run typecheck` 或项目实际可用的等价命令；高风险 UI 改动需要浏览器验证。

部署：

- 静态站点配置调整时同步检查 SPA rewrite、Vite 构建输出、路由刷新是否命中 `index.html`、API 基础地址是否受环境变量控制。

## 五、OSS 签名上传流程

模式：后端签发预签名 URL，前端 PUT 直传 OSS，前端调 confirm，业务接口保存 `publicUrl`。

共用接口：

- `POST /common/oss/presign`
- `POST /common/oss/confirm`

后端模块：

- 路由：`backend/api/common/oss.py`
- 规则：`backend/libs/upload_rules.py`
- 能力：`backend/libs/oss.py`

presign 流程：

- 鉴权，未登录或游客拒绝。
- `validate_upload_request()` 校验目录、扩展名、MIME。
- 空 MIME 或 `application/octet-stream` 按扩展名回填。
- `build_upload_object_key()` 生成 `{directory}/{uuid}.{ext}`。
- `sign_url("PUT", ...)` 生成 600 秒有效 URL。
- 返回 `objectKey`、`uploadUrl`、`publicUrl`、`contentType`。

前端直传：

- PUT 到 `uploadUrl`。
- Header 必须带后端返回的 `Content-Type`。
- Header 必须带 `x-oss-object-acl: public-read`。
- body 是文件二进制。

confirm 流程：

- 再次鉴权。
- `validate_object_key_directory()` 校验目录前缀。
- `bucket.put_object_acl(object_key, public-read)` 兜底设置 ACL。
- 返回成功。
- 不建议省略 confirm。

Web 前端：

- API 入口按当前 `strat-ark-frontend/src` 目录结构新增或复用。
- 流程为前置校验、presign、PUT、confirm、返回 `publicUrl`。

业务接口入库校验：

- ViewModel 调 `validate_oss_url()` 或 `validate_oss_url_list()` 做二次校验。
- URL 必须是 `https`。
- URL 必须属于当前 OSS Bucket Host。
- URL 必须在允许目录前缀下。
- 扩展名必须符合业务字段。
- 业务字段目录锁定，例如头像只能 `avatars`，Logo 只能 `school`。

服务端产物直传：

- PDF、裁图等使用 `upload_bytes_to_oss()`。
- 后端 AK/SK 构建 Bucket 后直接 `put_object`。
- 与前端签名上传是两条独立链路。

排查要点：

- `directory` 是否命中白名单。
- 是否使用后端返回的 `contentType`。
- 是否带 `x-oss-object-acl: public-read`。
- 是否调用 confirm。
- 业务字段存的是 `publicUrl`，不是 `uploadUrl` 或本地路径。
- 业务字段目录与上传目录是否一致。

## 六、OAuth 与登录鉴权规范

架构选型：

- Web SPA 普通业务默认 Public PKCE + 后端 exchange。
- 需要长期或离线访问 IdP API 时使用 Confidential BFF。
- 高安全场景可升级为 BFF + HttpOnly cookie + CSRF，但不要与 localStorage token 混搭。

通用约束：

- 自家 session 是已登录的唯一凭证。
- 第三方 id_token 只作为一次性出身证明，验签通过即丢弃。
- 第三方 id_token 必须 RS256 + JWKS 验签。
- 必须校验签名、`aud`、`iss`、`exp`、`iat`。
- Microsoft Entra 等 multi-tenant IdP 必须显式校验 `tid`。
- JWKS 客户端进程级缓存，TTL 建议 1 小时。
- 业务接口鉴权依赖只校验自家 session token。
- access token 短 TTL，建议 10 到 15 分钟。
- refresh token 长 TTL，建议 7 到 30 天。
- refresh token rotation + Redis 白名单。
- refresh、logout 接受失效 token 时不抛 500。
- 统一 `issue_session(identity)` 出口，邮箱密码、邮箱注册、OAuth 登录路径都使用同一会话签发逻辑。

PKCE：

- PKCE S256 必选。
- `code_verifier` 存 sessionStorage，禁止存 localStorage 或 cookie。
- `state` 必选，校验后立即消费。
- 多 provider 共用同一 `/callback`。
- IdP 控制台 redirect URI 必须逐字符匹配。
- SPA 前端只持有 client_id，不配置 client_secret。

前端会话：

- 邮箱密码路径与 OAuth 路径必须返回相同 session 结构。
- 前端使用同一个 `applySession(session, provider, setUser)` 或项目等价工具落库。
- 业务请求遇 401 时触发一次 refresh 并重试原请求。
- 启动应用时主动 refresh 一次恢复登录态。
- 并发请求共享同一个 inflight refresh Promise。
- access 提前 60 秒视为快过期并主动 refresh。

反模式：

- 禁止把 id_token 当 Bearer 给业务接口。
- 禁止后端只 base64 解码 JWT payload 后信任。
- 禁止前端持有 client_secret。
- 禁止 refresh token 持久化无 jti 白名单。
- 禁止各登录路径自己拼 token。
- 禁止 GET 请求隐式写库。
- 禁止 scope 一次申请所有可能权限。
- 禁止 HttpOnly cookie 与 localStorage token 混搭。

## 七、缺省资源与占位资源

核心目标：缺省资源放远程 OSS 或 CDN，更换缺省只改远程文件，各端自动生效，无需改代码发版。

职责：

- 数据层：缺失字段留空，禁止把默认 URL 或占位符写入 DB。
- 资源源：缺省文件放固定远程路径，更换缺省即替换同名文件。
- 展示层：字段空或加载失败时显示远程缺省资源。

Web 端：

- 远程缺省 URL 放环境变量，常量读取。
- `<img src={字段 || 缺省常量} onError={...}>`。
- `onError` 防循环：如果当前 `img.src` 已经是缺省常量，直接 return。

反模式：

- 禁止 DB 写默认 URL。
- 禁止请求层或响应层做字符串占位符替换。
- 禁止缺省图使用本地内嵌资源导致换图必须发版。

## 八、README 模板

需要为 FastAPI + Poetry 技术栈后端生成 README 时，复用以下结构并替换占位符：

- shields。
- 项目标题与描述。
- 目录。
- 快速开始：环境要求、安装步骤。
- 项目结构。
- API 模块。
- 核心功能。
- 技术栈：核心框架、数据库缓存、认证安全、云服务。
- 文档链接。
- 贡献。
- 许可证。
- 联系方式。
- shields 变量定义。

常用占位符：

- `{PROJECT_NAME}`、`{PROJECT_NAME_ZH}`、`{PROJECT_DESCRIPTION}`、`{PROJECT_DIR}`。
- `{GITHUB_REPO_URL}`、`{GITHUB_USER}`、`{GITHUB_REPO}`。
- `{PYTHON_VERSION}`、`{PORT}`、`{DATABASE}`、`{DATABASE_DRIVER}`、`{CACHE}`。
- `{AUTH_METHOD}`、`{PERMISSION_SYSTEM}`、`{CLOUD_STORAGE}`。
- `{MODULE_1}`、`{MODULE_2}`、`{FEATURE_1_NAME}`、`{FEATURE_1_DESCRIPTION}`。
- `{AUTHOR_NAME}`、`{AUTHOR_EMAIL}`。

## 九、工作流要求

- 开始修改前先读取相关代码和配置，不凭空假设。
- 遇到用户给出明确业务语义或客户口径时，按原话落地。
- 用户说“查漏补缺”时，默认继续做死代码、旧注释、白名单遗漏、遗留模板槽位等收尾清理。
- 用户说“执行修复和优化”“逐个实现”“继续”时，默认连续推进到实现、验证、提交，不要每个子任务都停下来确认。
- 本地验证以真实可用为验收标准；涉及浏览器流程时实际打开和测试。
- 导出或下载类问题以能真正下载和打开为验收标准。
- 高风险变更必须说明已验证内容和未验证风险。
