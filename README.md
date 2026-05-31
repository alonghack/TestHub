# TestHub — AI 驱动的测试管理平台

> 基于 Django 4.2 + Vue 3 构建的全栈测试管理平台，覆盖手工测试、API 自动化、UI 自动化、APP 自动化及 AI 智能测试（此项目为已获取授权的二开项目）。
<img width="1986" height="924" alt="image" src="https://github.com/user-attachments/assets/cf801902-4df5-4d0a-b7e0-ba4b029907f0" />

---

## 目录

- [项目概述](#项目概述)
- [技术栈](#技术栈)
- [快速开始](#快速开始)
- [系统架构](#系统架构)
- [后端模块详解（apps/）](#后端模块详解apps)
- [前端模块详解（frontend/src/）](#前端模块详解frontendsrc)
- [数据模型关系](#数据模型关系)
- [API 接口总览](#api-接口总览)
- [AI 集成架构](#ai-集成架构)
- [自动化测试模块](#自动化测试模块)
- [通知与定时任务系统](#通知与定时任务系统)
- [国际化](#国际化)
- [二次开发指南](#二次开发指南)
- [目录索引](#目录索引)

---

## 项目概述

TestHub 是一个面向测试团队的一站式测试管理平台，主要功能包括：

| 功能域 | 说明 |
|--------|------|
| **手工测试管理** | 测试用例编写、测试计划、执行跟踪、报告生成、用例评审 |
| **API 自动化测试** | HTTP/WebSocket 接口测试、集合管理、环境变量、断言、定时任务 |
| **UI 自动化测试** | 元素管理、页面对象模型、脚本录制/编辑（Selenium + Playwright + AI 模式） |
| **APP 自动化测试** | Android 设备管理、ADB + Airtest、场景编排、WebSocket 实时推送 |
| **AI 智能测试** | 需求文档解析 → AI 用例生成/评审、AI 浏览器自动化、Dify 助手 |
| **数据工厂** | 基于 Faker 的测试数据生成（银行卡号、身份证、编码转换等） |
| **通知中心** | 飞书/企微/钉钉 Webhook 机器人 + 邮件通知，定时任务告警 |

---

## 技术栈

### 后端

| 技术 | 版本 | 用途 |
|------|------|------|
| Python | 3.13 | 运行时 |
| Django | 4.2.7 | Web 框架 |
| Django REST Framework | 3.14 | API 框架 |
| drf-spectacular | 0.27 | OpenAPI 文档（Swagger/ReDoc） |
| SimpleJWT | 5.5 | JWT 认证 |
| Celery | 5.3 | 异步任务队列 |
| Channels + Daphne | 4.3 + 4.2 | WebSocket 支持 |
| PyMySQL | 1.1 | MySQL 驱动 |
| httpx | 0.28 | AI API 异步 HTTP 客户端 |
| Selenium | 4.15 | UI 自动化 |
| Playwright | 1.57 | UI 自动化 |
| browser-use | 0.10 | AI 浏览器智能体 |
| Airtest | 1.4 | APP 自动化 |
| EasyOCR | 1.7 | 图片文字识别 |

### 前端

| 技术 | 版本 | 用途 |
|------|------|------|
| Vue | 3.x | 前端框架 |
| Vite | 7.x | 构建工具 |
| Element Plus | 2.x | UI 组件库 |
| Pinia | 2.x | 状态管理 |
| Vue Router | 4.x | 路由 |
| Axios | 1.x | HTTP 客户端 |
| vue-i18n | 9.x | 国际化 |
| ECharts | 5.x | 图表 |
| Monaco Editor | 0.53 | 代码编辑器 |

---

## 快速开始

### 环境要求

- Python 3.10+
- MySQL 8.0+
- Redis（Celery/Channels）
- Node.js 18+

### 后端启动

```bash
# 1. 克隆项目
git@github.com:alonghack/TestHub.git
cd TestHub

# 2. 创建并激活虚拟环境
python -m venv .venv
# Windows:
.venv\Scripts\Activate.ps1
# macOS / Linux:
source .venv/bin/activate

# 3. 安装依赖
pip install -r requirements.txt

# 4. 配置环境变量（复制并编辑）
cp .env.example .env
# 编辑 .env 中的数据库配置、Redis、密钥等

# 5. 数据库迁移
python manage.py migrate

# 6. 创建超级用户
python manage.py createsuperuser

# 7. 初始化 UI 定位策略
python manage.py init_locator_strategies

# 8. 下载 WebDriver
python manage.py download_webdrivers

# 9. 启动开发服务器
python manage.py runserver
```

### 前端启动

```bash
cd frontend

# 安装依赖
npm install

# 启动开发服务器（端口 3000）
npm run dev

# 构建生产包
npm run build
```

### 访问地址

| 服务 | 地址 |
|------|------|
| 前端 | http://localhost:3000 |
| 后端 API | http://localhost:8000/api/ |
| Swagger 文档 | http://localhost:8000/api/docs/ |
| ReDoc 文档 | http://localhost:8000/api/redoc/ |
| Django Admin | http://localhost:8000/admin/ |

### 常用管理命令

```bash
# 数据库迁移
python manage.py makemigrations
python manage.py migrate

# 运行所有定时任务
python manage.py run_all_scheduled_tasks

# 初始化 UI 定位策略
python manage.py init_locator_strategies

# 下载 WebDriver
python manage.py download_webdrivers

# 导入组件包
python manage.py load_component_pack
```

---

## 系统架构

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend (Vue 3)                       │
│                   Port 3000 / Vite Dev                    │
├─────────────────────────────────────────────────────────┤
│                    Axios Proxy                            │
│  /api/* → http://127.0.0.1:8000                          │
│  /ws/*  → ws://127.0.0.1:8000                            │
├─────────────────────────────────────────────────────────┤
│                    Backend (Django 4.2)                   │
│                   Port 8000                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │               URL Router (backend/urls.py)        │   │
│  │  /api/auth/  /api/projects/  /api/testcases/ ... │   │
│  └──────────┬───────────────────────────┬───────────┘   │
│             ▼                           ▼                │
│  ┌────────────────────┐    ┌────────────────────┐       │
│  │   WSGI (HTTP)      │    │  ASGI (WebSocket)   │       │
│  │   runserver        │    │  Daphne             │       │
│  └────────┬───────────┘    └─────────┬──────────┘       │
│           │                          │                   │
│           ▼                          ▼                   │
│  ┌──────────────────────────────────────────────────┐   │
│  │              15 Django Apps (apps/)               │   │
│  │  users │ projects │ testcases │ executions        │   │
│  │  api_testing │ ui_automation │ app_automation    │   │
│  │  requirement_analysis │ assistant │ ...          │   │
│  └──────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────┤
│                    Data Layer                             │
│  ┌──────────────┐  ┌──────────────┐                     │
│  │   MySQL 8.0   │  │    Redis     │                     │
│  │   (持久化)    │  │  (缓存/队列)  │                    │
│  └──────────────┘  └──────────────┘                     │
└─────────────────────────────────────────────────────────┘
```

### 项目目录结构

```
TestHub/
├── .env                    # 环境变量配置（密钥、DB、Redis）
├── .env.example            # 环境变量模板
├── manage.py               # Django 管理入口
├── requirements.txt        # Python 依赖（190+）
├── backend/                # Django 项目配置
│   ├── settings.py         # 全局配置（DB/DRF/JWT/CORS/Celery）
│   ├── urls.py             # 根路由
│   ├── celery.py           # Celery 配置
│   ├── asgi.py             # ASGI 入口（Channels）
│   └── middleware.py       # 自定义中间件
├── apps/                   # 所有 Django App
│   ├── users/              # 用户认证
│   ├── projects/           # 项目管理
│   ├── testcases/          # 手工测试用例
│   ├── testsuites/         # 测试套件
│   ├── executions/         # 测试执行
│   ├── reports/            # 测试报告
│   ├── reviews/            # 用例评审
│   ├── versions/           # 版本管理
│   ├── api_testing/        # API 自动化测试
│   ├── ui_automation/      # UI 自动化测试
│   ├── app_automation/     # APP 自动化测试
│   ├── requirement_analysis/ # AI 需求分析
│   ├── assistant/          # AI 助手（Dify）
│   ├── core/               # 核心工具
│   └── data_factory/       # 数据工厂
├── frontend/               # Vue 3 前端
│   └── src/
│       ├── main.js         # 入口文件
│       ├── router/         # 路由定义
│       ├── stores/         # Pinia 状态管理
│       ├── api/            # API 服务层
│       ├── views/          # 页面组件
│       ├── layout/         # 布局组件
│       ├── components/     # 公共组件
│       ├── utils/          # 工具函数
│       ├── locales/        # 国际化
│       └── assets/         # 静态资源
├── docs/                   # 项目文档
├── logs/                   # 日志输出
├── media/                  # 用户上传文件
└── allure/                 # Allure 测试报告
```

---

## 后端模块详解（apps/）

### 模块总览

| 模块 | 目录 | 核心模型 | 功能职责 |
|------|------|----------|----------|
| users | `apps/users/` | User, UserProfile | 用户认证（JWT）、注册登录、个人信息 |
| projects | `apps/projects/` | Project, ProjectMember, ProjectEnvironment | 项目管理、团队成员、环境配置 |
| testcases | `apps/testcases/` | TestCase, TestCaseStep, TestCaseAttachment, TestCaseComment | 手工测试用例 CRUD |
| testsuites | `apps/testsuites/` | TestSuite, TestSuiteCase | **测试套件（TODO：视图未实现）** |
| executions | `apps/executions/` | TestPlan, TestRun, TestRunCase, TestRunCaseHistory | 测试计划、执行跟踪 |
| reports | `apps/reports/` | TestReport, ReportTemplate | 测试报告生成 |
| reviews | `apps/reviews/` | TestCaseReview, ReviewAssignment, ReviewTemplate | 用例评审流程 |
| versions | `apps/versions/` | Version | 版本管理 |
| api_testing | `apps/api_testing/` | ApiProject, ApiRequest, Environment, TestSuite, ScheduledTask, AIServiceConfig... | **API 自动化测试（最大模块之一）** |
| ui_automation | `apps/ui_automation/` | UiProject, Element, PageObject, TestScript, AICase... | **UI 自动化测试（最大模块之一）** |
| app_automation | `apps/app_automation/` | AppProject, AppDevice, AppElement, AppTestCase... | **APP 自动化测试** |
| requirement_analysis | `apps/requirement_analysis/` | RequirementDocument, AIModelConfig, PromptConfig, TestCaseGenerationTask | AI 需求分析 + 用例生成 |
| assistant | `apps/assistant/` | DifyConfig, AssistantSession, ChatMessage | Dify AI 助手聊天 |
| core | `apps/core/` | UnifiedNotificationConfig | 统一通知配置、管理命令 |
| data_factory | `apps/data_factory/` | DataFactoryRecord | 测试数据生成 |

### 模块详解

---

#### 1. users — 用户认证与权限

**位置：** `apps/users/`

**文件清单：**
| 文件 | 说明 |
|------|------|
| `models.py` | User（继承 AbstractUser）、UserProfile |
| `serializers.py` | UserSerializer, LoginSerializer, UserCreateSerializer, UserProfileSerializer |
| `views.py` | login_view, logout_view, RegisterView, UserListView, UserDetailView |
| `test_views.py` | test_register（调试用注册接口） |
| `urls.py` | 认证路由 |
| `admin.py` | Admin 后台配置 |

**API 端点：**

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/auth/login/` | 登录（返回 JWT token 对） |
| POST | `/api/auth/register/` | 注册 |
| POST | `/api/auth/test-register/` | 调试注册 |
| POST | `/api/auth/logout/` | 登出（黑名单 refresh token） |
| GET | `/api/users/me/` | 获取当前用户 |
| GET/PATCH | `/api/auth/profile/` | 用户个人信息 |
| GET | `/api/users/` | 用户列表 |
| GET/PUT/DELETE | `/api/users/<id>/` | 用户详情 |
| POST | `/api/auth/token/refresh/` | JWT 刷新 |

**认证机制：**
- 主要认证：JWT（SimpleJWT），access_token 60分钟，refresh_token 7天
- 默认权限：全局 `IsAuthenticated`
- 自定义用户模型：`AUTH_USER_MODEL = 'users.User'`
- 开发环境通过 `DisableCSRFMiddleware` 禁用 CSRF

---

#### 2. projects — 项目管理

**位置：** `apps/projects/`

**数据模型：**
```
Project (projects)
├── owner → User
├── members → M2M through ProjectMember (project_members)
│   └── role: owner/admin/developer/tester/viewer
└── environments → ProjectEnvironment (project_environments)
    └── variables: JSONField
```

**API 端点：**

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST | `/api/projects/` | 项目列表/创建 |
| GET | `/api/projects/all/` | 全量列表 |
| GET/PUT/DELETE | `/api/projects/<id>/` | 项目详情 |
| GET | `/api/projects/list/` | 用户参与的项目 |
| GET/POST | `/api/projects/<id>/environments/` | 环境变量管理 |
| GET/POST | `/api/projects/<id>/members/` | 成员管理 |

---

#### 3. testcases — 手工测试用例

**位置：** `apps/testcases/`

**数据模型：**
```
TestCase (testcases)
├── project → Project
├── versions → M2M → Version
├── author → User
├── assignee → User
├── step_details → TestCaseStep (testcase_steps)
├── attachments → TestCaseAttachment (testcase_attachments)
└── comments → TestCaseComment (testcase_comments)
```

**字段说明：**
- `priority`: low/medium/high/critical
- `status`: draft/active/deprecated
- `test_type`: functional/integration/api/ui/performance/security
- `tags`: JSONField（标签数组）

**API 端点：**

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST | `/api/testcases/` | 用例列表/创建 |
| GET/PUT/DELETE | `/api/testcases/<id>/` | 用例详情 |

---

#### 4. testsuites — 测试套件

**位置：** `apps/testsuites/`

> **注意：** 此模块的视图（views.py）目前为空，仅定义了模型。需要通过 TestCase 关联手工测试用例组成套件。

**数据模型：**
```
TestSuite (testsuites)
├── project → Project
├── author → User
└── testcases → M2M through TestSuiteCase (testsuite_cases)
```

---

#### 5. executions — 测试执行

**位置：** `apps/executions/`

**数据模型：**
```
TestPlan (test_plans)
├── projects → M2M → Project
├── version → Version
├── creator → User
├── assignees → M2M → User
└── test_runs → TestRun (test_runs)
    ├── test_plan → TestPlan
    ├── project → Project
    ├── assignee → User
    ├── creator → User
    ├── testcases → M2M through TestRunCase (test_run_cases)
    │   ├── status: untested/passed/failed/blocked/retest
    │   ├── defects: JSONField
    │   └── history → TestRunCaseHistory
    └── progress_stats (属性: 自动计算进度)
```

**API 端点：**

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST | `/api/executions/plans/` | 测试计划 |
| GET/PUT/DELETE | `/api/executions/plans/<id>/` | 计划详情 |
| GET/POST | `/api/executions/runs/` | 测试执行 |
| GET/PUT/DELETE | `/api/executions/runs/<id>/` | 执行详情 |
| GET/POST | `/api/executions/run_cases/` | 执行用例 |
| GET/POST | `/api/executions/history/` | 执行历史 |

---

#### 6. reports — 测试报告

**位置：** `apps/reports/`

**数据模型：**
- `TestReport`: project, name, report_type (execution/summary/trend), execution (OneToOne→TestRun), summary/content (JSON)
- `ReportTemplate`: name, template_config (JSON), is_default

---

#### 7. reviews — 用例评审

**位置：** `apps/reviews/`

**数据模型：**
```
TestCaseReview (testcase_reviews)
├── projects → M2M → Project
├── testcases → M2M → TestCase
├── creator → User
├── template → ReviewTemplate
├── assignments → ReviewAssignment (review_assignments)
│   ├── reviewer → User
│   ├── status: pending/approved/rejected/abstained
│   └── checklist_results: JSONField
└── comments → TestCaseReviewComment (review_comments)
```

**API 端点：**

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST | `/api/reviews/reviews/` | 评审列表/创建 |
| GET/PUT/DELETE | `/api/reviews/reviews/<id>/` | 评审详情 |
| GET/POST | `/api/reviews/review-comments/` | 评审意见 |
| GET/POST | `/api/reviews/review-templates/` | 评审模板 |

---

#### 8. versions — 版本管理

**位置：** `apps/versions/`

**数据模型：**
- `Version`: projects (M2M), name, description, is_baseline

**API 端点：**

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST | `/api/versions/` | 版本列表/创建 |
| GET | `/api/versions/projects/<pid>/versions/` | 项目下的版本 |

---

#### 9. api_testing — API 自动化测试（核心模块）

**位置：** `apps/api_testing/`

**文件清单：**
| 文件 | 说明 |
|------|------|
| `models.py` | 13 个模型（API 测试全部数据） |
| `serializers.py` | 全套序列化器 |
| `views.py` | 视图集 |
| `urls.py` | DefaultRouter 注册路由 |
| `utils.py` | 通用工具 |
| `variable_resolver.py` | 变量解析引擎 |
| `operation_logger.py` | 操作日志记录器 |
| `custom_email_backend.py` | 邮件发送后端 |

**数据模型：**
```
ApiProject (api_projects)
├── collections → ApiCollection (api_collections, 树形 parent 自引用)
│   └── requests → ApiRequest (api_requests)
│       ├── headers/params/body/auth/assertions: JSONField
│       └── histories → RequestHistory (api_request_histories)
├── environments → Environment (api_environments, GLOBAL/LOCAL)
├── test_suites → TestSuite (api_test_suites)
│   └── requests → M2M through TestSuiteRequest
│       └── executions → TestExecution (api_test_executions)
│
ScheduledTask (api_scheduled_tasks)
├── test_suite / api_request → FK
├── environment → FK
├── trigger_type: CRON / INTERVAL / ONCE
├── notification_settings → TaskNotificationSetting
└── execution_logs → TaskExecutionLog

NotificationLog (api_notification_logs)
AIServiceConfig (api_ai_service_configs)
│   service_type: openai/azure/anthropic/deepseek/qwen/siliconflow
│   role: doc_extractor/naming/mock_data/description
OperationLog (api_operation_logs)
```

**API 端点（全部挂载在 `/api/api-testing/` 下）：**

| 路径 | 说明 |
|------|------|
| `dashboard/` | 仪表盘统计数据 |
| `projects/` | API 项目 CRUD |
| `collections/` | 接口集合（树形结构） |
| `requests/` | API 请求定义 |
| `environments/` | 环境变量（全局/局部） |
| `histories/` | 请求历史 |
| `test-suites/` | 测试套件 |
| `test-suite-requests/` | 套件-请求关联 |
| `test-executions/` | 测试执行记录 |
| `users/` | 项目成员查询 |
| `scheduled-tasks/` | 定时任务 |
| `task-execution-logs/` | 任务执行日志 |
| `notification-logs/` | 通知日志 |
| `task-notification-settings/` | 任务通知设置 |
| `operation-logs/` | 操作日志 |
| `ai-service-configs/` | AI 服务配置 |

---

#### 10. ui_automation — UI 自动化测试（核心模块）

**位置：** `apps/ui_automation/`

**文件清单：**
| 文件 | 说明 |
|------|------|
| `models.py` | 20+ 模型（UI 测试全部数据） |
| `serializers.py` | 全套序列化器 |
| `views.py` + `views_config.py` | 视图集 |
| `urls.py` | DefaultRouter 注册路由 |
| `selenium_engine.py` | Selenium 执行引擎 |
| `playwright_engine.py` | Playwright 执行引擎 |
| `ai_agent.py` | AI 智能模式代理（browser-use） |
| `ai_base.py` | AI 代理基类（LangChain） |
| `test_executor.py` | 测试执行调度器 |
| `variable_resolver.py` | 变量解析 |
| `pdf_generator.py` | PDF 报告生成 |
| `reports.py` | 报告逻辑 |
| `operation_logger.py` | 操作日志 |

**数据模型层级：**
```
UiProject (ui_projects)
├── element_groups → ElementGroup (ui_element_groups, 树形)
│   └── elements → Element (ui_elements)
│       ├── locator_strategy → LocatorStrategy (locator_strategies)
│       ├── backup_locators: JSONField
│       ├── parent_element → Element (自引用)
│       └── page / component_name
├── page_objects → PageObject (ui_page_objects)
│   └── elements → PageObjectElement (ui_page_object_elements)
├── test_scripts → TestScript (ui_test_scripts)
│   └── steps → ScriptStep (ui_script_steps)
├── test_suites → TestSuite (ui_test_suites)
├── environments → TestEnvironment (ui_test_environments)
├── screenshots → Screenshot (ui_screenshots)
├── test_cases → TestCase (ui_test_cases)
│   └── steps → TestCaseStep (ui_test_case_steps)
│       └── executions → TestCaseExecution (ui_test_case_executions)
├── test_executions → TestExecution (ui_test_executions)
├── ai_cases → AICase (ui_ai_cases)
│   └── records → AIExecutionRecord (ui_ai_execution_records)
├── scheduled_tasks → UiScheduledTask (ui_scheduled_tasks)
│   └── notification_settings → UiTaskNotificationSetting
└── notification_logs → UiNotificationLog (ui_notification_logs)
```

**AI 智能模式：**
- `ai_agent.py`: 基于 `browser-use` 库 + LangChain
- 支持文本模式（`execution_mode='text'`）
- 流程：分析任务 → 规划步骤 → 执行操作 → 生成报告
- GIF 录制支持

**API 端点（`/api/ui-automation/`）：**

| 路径 | 说明 |
|------|------|
| `dashboard/` | 仪表盘 |
| `projects/` | UI 项目 CRUD |
| `locator-strategies/` | 定位策略管理 |
| `element-groups/` | 元素分组（树形） |
| `elements/` | 元素管理 |
| `test-scripts/` | 测试脚本 |
| `page-objects/` | 页面对象 |
| `steps/` | 脚本步骤 |
| `test-suites/` | 测试套件 |
| `test-executions/` | 测试执行 |
| `screenshots/` | 截图管理 |
| `test-cases/` | UI 测试用例 |
| `test-case-steps/` | 用例步骤 |
| `test-case-executions/` | 用例执行记录 |
| `scheduled-tasks/` | 定时任务 |
| `ai-cases/` | AI 测试用例 |
| `ai-execution-records/` | AI 执行记录 |
| `ai-case-generation/` | AI 用例生成 |
| `notification-logs/` | 通知日志 |
| `operation-records/` | 操作记录 |
| `config/environment/` | 环境配置 |
| `config/ai-mode/` | AI 模式配置 |
| `ai-models/` | AI 模型配置 |

---

#### 11. app_automation — APP 自动化测试

**位置：** `apps/app_automation/`

**文件清单：**

| 文件/目录 | 说明 |
|-----------|------|
| `models.py` | APP 自动化全部模型 |
| `serializers.py` | 序列化器 |
| `views/` | 多文件视图（按资源分文件） |
| `consumers.py` | WebSocket 消费者（执行进度推送） |
| `routing.py` | WebSocket 路由 |
| `executors/test_executor.py` | 测试执行器 |
| `managers/device_manager.py` | 设备管理器（锁定/解锁） |
| `runners/ui_flow_runner.py` | UI 流程运行器 |
| `tasks.py` | Celery 任务 |
| `utils/airtest_base.py` | Airtest 基础工具 |
| `utils/image_helpers.py` | 图片辅助 |
| `utils/ocr_helper.py` | OCR 文字识别 |
| `constants.py` | 状态常量 |

**数据模型：**
```
AppProject (app_projects)
├── devices → AppDevice (app_devices, 含锁定机制)
├── elements → AppElement (app_elements, 坐标/图片/区域定位)
├── packages → AppPackage (app_packages)
├── components → AppComponent (app_components)
├── custom_components → AppCustomComponent (app_custom_components)
├── component_packages → AppComponentPackage (app_component_packages)
├── test_cases → AppTestCase (app_test_cases, 流程编排)
├── test_suites → AppTestSuite (app_test_suites)
├── executions → AppTestExecution (app_test_executions)
├── scheduled_tasks → AppScheduledTask (app_scheduled_tasks)
└── notification_logs → AppNotificationLog (app_notification_logs)
```

**WebSocket：**
- 路径：`ws://host/ws/app-automation/executions/<execution_id>/`
- 用途：实时推送执行进度

**API 端点（`/api/app-automation/`）：**

| 路径 | 说明 |
|------|------|
| `projects/` | APP 项目 CRUD |
| `config/` | 测试配置 |
| `dashboard/` | 仪表盘 |
| `devices/` | 设备管理 |
| `elements/` | 元素管理 |
| `components/` | 组件库 |
| `custom-components/` | 自定义组件 |
| `component-packages/` | 组件包 |
| `packages/` | APP 包管理 |
| `test-cases/` | 测试用例 |
| `test-suites/` | 测试套件 |
| `scheduled-tasks/` | 定时任务 |
| `notification-logs/` | 通知日志 |
| `executions/` | 执行记录 |

---

#### 12. requirement_analysis — AI 需求分析

**位置：** `apps/requirement_analysis/`

**文件清单：**
| 文件 | 说明 |
|------|------|
| `models.py` | 需求文档、业务需求、分析任务 |
| `ai_models.py` | AI 模型配置 + Prompt 配置 + 生成任务 + 服务类（核心） |
| `serializers.py` | 序列化器 |
| `views.py` | 视图集 |
| `services.py` | 业务服务层 |
| `advanced_analyzer.py` | 高级分析器 |
| `urls.py` | DefaultRouter 路由 |

**AI 双模型流水线：**
```
用户上传需求文档 / 粘贴文本
        │
        ▼
AIModelService.generate_test_cases()
  ┌─ writer_prompt + writer_model ──→ 初版测试用例
  │
  └─ reviewer_prompt + reviewer_model ──→ 评审反馈
        │
        ▼
AIModelService.revise_test_cases_based_on_review()
  ┌─ 合并初版用例 + 评审意见 ──→ 终版用例
  │
  └─ 支持自动续写（长文本分段生成）
        │
        ▼
  保存到 GeneratedTestCase 或 TestCaseGenerationTask
```

**支持的 AI 提供商：**
- DeepSeek / 通义千问 / 硅基流动 / 其他（兼容 OpenAI 格式）
- 角色：writer（用例编写）/ reviewer（用例评审）/ browser_use_text（浏览器智能体）

**API 端点（`/api/requirement-analysis/`）：**

| 路径 | 说明 |
|------|------|
| `documents/` | 需求文档上传/管理 |
| `analyses/` | 需求分析结果 |
| `requirements/` | 业务需求 |
| `test-cases/` | AI 生成的测试用例 |
| `tasks/` | 分析任务状态 |
| `ai-models/` | AI 模型配置 |
| `prompts/` | 提示词配置 |
| `generation-config/` | 生成行为配置 |
| `testcase-generation/` | 用例生成任务 |
| `config/` | 配置管理 |

---

#### 13. assistant — AI 助手（Dify）

**位置：** `apps/assistant/`

**数据模型：**
- `DifyConfig`: api_key, api_url, is_active
- `AssistantSession`: user, session_id, conversation_id, title
- `ChatMessage`: session, role (user/assistant), content, conversation_id
- `AssistantMessage`（旧版兼容）

**API 端点（`/api/assistant/`）：**

| 路径 | 说明 |
|------|------|
| `sessions/` | 会话管理 |
| `chat/` | 聊天接口 |
| `config/dify/` | Dify 配置 |

---

#### 14. core — 核心工具

**位置：** `apps/core/`

**数据模型：**
- `UnifiedNotificationConfig`: name, config_type (webhook_feishu/wechat/dingtalk), webhook_bots (JSON), is_default

**管理命令：**
| 命令 | 说明 |
|------|------|
| `download_webdrivers` | 下载 Selenium/Playwright WebDriver |
| `init_locator_strategies` | 初始化 UI 定位策略数据 |
| `load_component_pack` | 导入 APP 自动化组件包 |
| `run_all_scheduled_tasks` | 执行所有模块的定时任务 |

---

#### 15. data_factory — 数据工厂

**位置：** `apps/data_factory/`

**数据模型：**
- `DataFactoryRecord`: user, tool_name, tool_category, input/output (JSON), tags

**内置工具模块（`tools/` 目录）：**

| 模块 | 功能 |
|------|------|
| `random_tools.py` | 随机数据生成 |
| `string_tools.py` | 字符串处理 |
| `json_tools.py` | JSON 工具 |
| `china_bank_card.py` | 中国银行卡号生成 |
| `unified_credit_code.py` | 统一信用代码生成 |
| `encryption_tools.py` | 加密/解密 |
| `encoding_tools.py` | 编码转换 |
| `image_tools.py` | 图片处理 |
| `crontab_tools.py` | Cron 表达式工具 |
| `test_data_tools.py` | 测试数据工具 |

---

## 前端模块详解（frontend/src/）

### 目录结构

```
frontend/src/
├── main.js                       # 入口：初始化 Pinia → Auth → Router → i18n → ElementPlus
├── App.vue                       # 根组件（el-config-provider 动态语言）
├── router/index.js               # 路由定义 + 导航守卫（519行）
├── stores/
│   ├── user.js                   # 用户认证状态（JWT/token刷新/自动刷新/localStorage持久化）
│   └── app.js                    # 全局状态（语言切换）
├── api/                          # API 服务层（6个文件，按模块分）
│   ├── api-testing.js            # API 测试接口（最完整之一）
│   ├── ui_automation.js          # UI 自动化接口（最大文件，~1060行）
│   ├── app-automation.js         # APP 自动化接口（~700行）
│   ├── requirement-analysis.js   # 需求分析接口
│   ├── data-factory.js           # 数据工厂接口
│   └── core.js                   # 通知配置接口
├── utils/
│   ├── api.js                    # Axios 实例 + 请求/响应拦截器（核心）
│   ├── codeGenerator.ts          # 代码生成器
│   ├── requestModel.ts           # 请求模型
│   └── app-automation-helpers.js # APP 自动化辅助函数
├── layout/index.vue              # 主布局（侧边栏 + 顶栏 + 模块菜单）
├── components/
│   └── DataFactorySelector.vue   # 数据工厂选择器组件
├── views/                        # 页面组件（按模块分目录）
│   ├── auth/                     # Login.vue, Register.vue
│   ├── Home.vue                  # 首页
│   ├── projects/                 # ProjectList, ProjectDetail（项目 CRUD）
│   ├── testcases/                # TestCaseList, TestCaseDetail, TestCaseForm, TestCaseEdit
│   ├── testsuites/               # TestSuiteList
│   ├── executions/               # ExecutionList, ExecutionDetailView, ExecutionListView
│   ├── reports/                  # ReportList, AiTestReport
│   ├── reviews/                  # ReviewList, ReviewDetail, ReviewForm, ReviewTemplateList
│   ├── versions/                 # VersionList
│   ├── profile/                  # UserProfile
│   ├── api-testing/              # 11个页面（Dashboard, ProjectManagement, InterfaceManagement...）
│   ├── ui-automation/            # 17个页面（Dashboard, ElementManager, ScriptEditor...）
│   ├── ui-automation/ai/         # 4个页面（AITesting, AICaseList, AIExecutionRecords...）
│   ├── app-automation/           # 12个页面（Dashboard, DeviceList, SceneBuilder...）
│   ├── requirement-analysis/     # 6个页面（RequirementAnalysis, AIModelConfig...）
│   ├── assistant/                # AssistantView（Dify 聊天）
│   ├── configuration/            # 4个页面（ConfigurationCenter, AIIntelligentModeConfig...）
│   ├── data-factory/             # DataFactory
│   └── notification/             # NotificationConfigs, NotificationLogs
├── locales/                      # 国际化（zh-cn + en，17个领域文件 × 2语言）
│   ├── index.js, datetimeFormats.js, numberFormats.js
│   └── lang/{zh-cn,en}/
└── assets/                       # CSS / 图片
```

### 路由架构

前端使用 `vue-router` 的 `createWebHistory()`，主要分为以下模块：

| 模块 | 路径前缀 | 路由数量 | 说明 |
|------|----------|----------|------|
| AI 用例生成 | `/ai-generation/` | 19条 | 需求分析/项目管理/用例/版本/评审/执行/报告 |
| 接口测试 | `/api-testing/` | 10条 | 仪表盘/项目/接口/自动化/环境/报告/定时任务 |
| UI 自动化 | `/ui-automation/` | 13条 | 仪表盘/元素/脚本/套件/执行/报告/定时任务 |
| AI 智能模式 | `/ai-intelligent-mode/` | 3条 | AI 测试/用例列表/执行记录 |
| APP 自动化 | `/app-automation/` | 12条 | 仪表盘/设备/元素/场景/套件/执行/报告 |
| 配置中心 | `/configuration/` | 8条 | AI 模型/提示词/环境/Dify/通知 |
| 独立页面 | 根路径 | 5条 | 首页/登录/注册/数据工厂/AI 助手 |

### HTTP 客户端（utils/api.js）

核心功能：
- **请求拦截器：** 自动附加 `Authorization: Bearer <token>`，token 即将过期时自动刷新
- **响应拦截器：** 401 时尝试 refresh token → 重试原请求，失败则跳转登录
- **请求队列：** 刷新 token 期间，并发请求排队等待
- **错误提示：** 401/500 及其他错误自动弹出 Element Plus 消息

### 状态管理（stores/）

- **user.js：** 管理 JWT token 对、用户信息，支持自动刷新（每2分钟检查），持久化到 localStorage
- **app.js：** 管理语言设置

### 构建配置（vite.config.js）

- 端口 3000，host `0.0.0.0`
- 代理 `/api/` `/media/` `/app-automation-*/` 到 `http://127.0.0.1:8000`
- WebSocket 代理 `/ws/` 到 `ws://127.0.0.1:8000`

---

## 数据模型关系

### 手工测试核心数据流

```
Project
  ├── TestCase ──┐
  │              ├── Version (M2M)
  │              ├── TestCaseStep
  │              ├── TestCaseAttachment
  │              └── TestCaseComment
  ├── TestPlan ──┤
  │              └── TestRun ──┬── TestRunCase
  │                             │    └── TestRunCaseHistory
  │                             └── User (assignee/creator)
  └── User (owner/members)
```

### API 自动化数据流

```
ApiProject
  ├── ApiCollection (树形)
  │   └── ApiRequest ──┬── RequestHistory
  │                     └── TestSuiteRequest → TestSuite
  │                                           └── TestExecution
  ├── Environment (全局/局部)
  └── ScheduledTask ──┬── TaskExecutionLog
                       └── TaskNotificationSetting → UnifiedNotificationConfig
```

### UI 自动化数据流

```
UiProject
  ├── ElementGroup → Element (多定位策略)
  ├── PageObject → PageObjectElement
  ├── TestScript → ScriptStep
  ├── TestSuite → TestExecution
  ├── TestCase → TestCaseStep → TestCaseExecution
  ├── AICase → AIExecutionRecord
  └── UiScheduledTask
```

### APP 自动化数据流

```
AppProject
  ├── AppDevice (含锁定机制)
  ├── AppElement
  ├── AppComponent / AppCustomComponent
  ├── AppPackage
  ├── AppTestCase (流程编排) → AppTestExecution
  ├── AppTestSuite → AppTestExecution
  └── AppScheduledTask
```

---

## API 接口总览

所有 API 端点以 `/api/` 为前缀，完整接口文档可通过 Swagger 或 ReDoc 访问：

| 前缀 | 来源 | 说明 |
|------|------|------|
| `/api/auth/` | users | 登录/注册/登出 |
| `/api/users/` | users | 用户 CRUD |
| `/api/projects/` | projects | 项目 CRUD、成员、环境 |
| `/api/testcases/` | testcases | 测试用例 CRUD |
| `/api/testsuites/` | testsuites | 测试套件（TODO） |
| `/api/executions/` | executions | 测试计划/执行 |
| `/api/reports/` | reports | 测试报告 |
| `/api/reviews/` | reviews | 用例评审 |
| `/api/versions/` | versions | 版本管理 |
| `/api/assistant/` | assistant | Dify AI 助手 |
| `/api/requirement-analysis/` | requirement_analysis | AI 需求分析 |
| `/api/ui-automation/` | ui_automation | UI 自动化测试 |
| `/api/app-automation/` | app_automation | APP 自动化测试 |
| `/api/api-testing/` | api_testing | API 测试 |
| `/api/core/` | core | 统一通知配置 |
| `/api/data-factory/` | data_factory | 数据工厂 |
| `/api/schema/` | drf-spectacular | OpenAPI Schema |
| `/api/docs/` | drf-spectacular | Swagger UI |
| `/api/redoc/` | drf-spectacular | ReDoc |

---

## AI 集成架构

平台支持四条独立的 AI 集成路径：

### 1. 需求分析 + 用例生成（requirement_analysis）

**核心文件：** `apps/requirement_analysis/ai_models.py`

```
AIModelConfig (ai_model_config)
├── model_type: deepseek / qwen / siliconflow / other
├── role: writer / reviewer / browser_use_text
├── api_key / base_url / model_name / temperature
└── 唯一约束: (model_type, role) 每种角色只能一个活跃配置

PromptConfig (prompt_config)
├── prompt_type: writer / reviewer
└── 唯一约束: (prompt_type, is_active) 每种类型只能一个活跃提示词

TestCaseGenerationTask (testcase_generation_task)
├── writer_model + reviewer_model → 双模型流水线
├── 支持 stream / complete 两种输出模式
└── 自动续写 + 用例重编号
```

### 2. API 测试 AI 服务（api_testing）

**核心文件：** `apps/api_testing/models.py`（AIServiceConfig）

```
AIServiceConfig (api_ai_service_configs)
├── service_type: openai / azure / anthropic / deepseek / qwen / siliconflow
└── role: doc_extractor / naming / mock_data / description
```

### 3. UI 自动化 AI 智能模式（ui_automation）

**核心文件：** `apps/ui_automation/ai_agent.py` + `ai_base.py`

- 基于 `browser-use` 库 + LangChain
- 流程：分析任务描述 → 规划执行步骤 → 自动执行操作 → 生成执行报告
- 支持 GIF 录制

### 4. Dify 助手（assistant）

**核心文件：** `apps/assistant/models.py` + `views.py`

- 集成 Dify Chat API
- 会话管理 + 消息记录

---

## 自动化测试模块

### API 自动化执行流程

```
定时任务触发 / 手动执行
        │
        ▼
Celery Worker 调度
        │
        ▼
遍历 TestSuite 中的 ApiRequest
  ├─ 解析环境变量 (variable_resolver.py)
  ├─ 执行 HTTP/WebSocket 请求 (httpx)
  ├─ 执行断言 (status_code / jsonpath / header)
  ├─ 记录请求历史
  └─ 汇总 TestExecution 结果
        │
        ▼
通知发送（邮件 / 飞书 / 企微 / 钉钉）
```

### UI 自动化引擎

| 引擎 | 文件 | 说明 |
|------|------|------|
| Selenium | `selenium_engine.py` | 传统 WebDriver 模式 |
| Playwright | `playwright_engine.py` | 现代浏览器自动化 |
| AI Agent | `ai_agent.py` | browser-use AI 智能模式 |

### APP 自动化技术架构

```
WebSocket (实时推送)
    │
AppTestExecutor (test_executor.py)
    │
    ├── DeviceManager (device_manager.py)
    │   ├── 设备锁定/解锁
    │   └── ADB 连接管理
    │
    ├── UiFlowRunner (ui_flow_runner.py)
    │   └── Airtest 脚本执行
    │
    └── OCR Helper (ocr_helper.py)
        └── EasyOCR 图片文字识别
```

---

## 通知与定时任务系统

### 通知系统架构

```
UnifiedNotificationConfig (apps/core)
├── config_type: webhook_feishu / webhook_wechat / webhook_dingtalk
├── webhook_bots: JSON (多机器人配置)
├── is_default / is_active
└── 被各模块的 TaskNotificationSetting 引用
        │
        ├── api_testing → TaskNotificationSetting
        ├── ui_automation → UiTaskNotificationSetting
        └── app_automation → AppTaskNotificationSetting
```

### 定时任务系统

三个自动化模块各自独立实现定时任务，底层共享 Celery Worker：

| 模块 | 任务模型 | 触发方式 | Celery 任务 |
|------|----------|----------|-------------|
| API 测试 | `ScheduledTask` | CRON/INTERVAL/ONCE | `run_all_scheduled_tasks` |
| UI 自动化 | `UiScheduledTask` | CRON/INTERVAL/ONCE | 同上 |
| APP 自动化 | `AppScheduledTask` | CRON/INTERVAL/ONCE | 同上 |

**全局管理命令：** `python manage.py run_all_scheduled_tasks`

---

## 国际化

前端使用 `vue-i18n`，支持中文（zh-cn）和英文（en）。

### 翻译文件结构

```
frontend/src/locales/
├── index.js               # i18n 实例配置
├── datetimeFormats.js     # 日期格式
├── numberFormats.js       # 数字格式
├── pluralRules.js         # 复数规则
└── lang/
    ├── zh-cn/             # 中文（17个文件）
    │   ├── index.js       # 合并所有子模块
    │   ├── nav.js         # 导航菜单翻译
    │   ├── common.js      # 通用翻译
    │   ├── auth.js        # 认证相关
    │   ├── project.js     # 项目模块
    │   ├── testcase.js    # 测试用例
    │   ├── execution.js   # 执行
    │   ├── report.js      # 报告
    │   ├── review.js      # 评审
    │   ├── version.js     # 版本
    │   ├── api-testing.js  # API 测试
    │   ├── ui-automation.js # UI 自动化
    │   ├── requirement.js # 需求分析
    │   ├── assistant.js   # 助手
    │   ├── notification.js # 通知
    │   ├── configuration.js # 配置
    │   └── data-factory.js # 数据工厂
    └── en/                # 英文（同名文件）
```

Element Plus 组件库的国际化通过 `el-config-provider` 在 `App.vue` 中动态配置。

---

## 二次开发指南

### 新建后端模块

1. 在 `apps/` 下创建模块目录：
   ```
   apps/<module>/
   ├── __init__.py
   ├── models.py        # 数据模型
   ├── serializers.py   # DRF 序列化器
   ├── views.py         # 视图集 (ViewSet / APIView)
   ├── urls.py          # 路由 (DefaultRouter 或 path())
   ├── admin.py         # Admin 配置
   ├── apps.py          # AppConfig
   └── migrations/      # 数据库迁移
   ```

2. 在 `backend/settings.py` 的 `LOCAL_APPS` 中注册：
   ```python
   LOCAL_APPS = [
       ...
       'apps.<module>',
   ]
   ```

3. 在 `backend/urls.py` 添加路由：
   ```python
   path('api/<prefix>/', include('apps.<module>.urls')),
   ```

4. 运行迁移：
   ```bash
   python manage.py makemigrations <module>
   python manage.py migrate
   ```

### 新建前端页面

1. 在 `views/` 下创建 Vue 组件
2. 在 `api/` 下创建/扩展 API 服务文件，使用 `utils/api.js` 的 axios 实例
3. 在 `router/index.js` 注册路由
4. 在 `locales/lang/{zh-cn,en}/` 添加翻译

### 新增 AI 模型提供商

在 `apps/requirement_analysis/ai_models.py` 中：
1. 在 `AIModelConfig.MODEL_CHOICES` 添加新选项
2. 所有模型使用 OpenAI 兼容格式，通常无需额外适配
3. 如 API 格式特殊，在 `AIModelService` 中添加专用方法

### 新增定时任务类型

1. 在对应模块的 `ScheduledTask` 模型 `TASK_TYPE_CHOICES` 添加类型
2. 实现任务执行逻辑
3. 在 `core/management/commands/run_all_scheduled_tasks.py` 注册

### 新增自动化测试引擎

- **API 测试：** 在 `apps/api_testing/` 扩展请求执行逻辑
- **UI 测试：** 在 `apps/ui_automation/` 新增引擎文件，实现 `test_executor.py` 中的引擎接口
- **APP 测试：** 在 `apps/app_automation/executors/` 扩展执行器

### 开发规范

**后端规范：**
- 使用 `ModelViewSet` + `DefaultRouter` 实现标准 CRUD
- JSON 字段使用 `models.JSONField()`（MySQL 原生 JSON 支持）
- 数据库表名通过 `db_table` 显式指定为 `snake_case`
- 使用 `django-filter` 实现过滤/搜索/排序
- 序列化器 `read_only_fields` 明确只读字段

**前端规范：**
- Composition API + `<script setup>`
- API 调用通过 `utils/api.js` 的 axios 实例
- 文本国际化通过 `$t('key')` 访问
- Element Plus 图标通过全局注册直接使用

### 常用开发命令

```bash
# === 后端 ===
python manage.py runserver              # 启动开发服务器
python manage.py makemigrations         # 生成迁移
python manage.py migrate                # 执行迁移
python manage.py showmigrations         # 查看迁移状态
python manage.py test apps/<module>     # 运行测试
python manage.py shell                  # Django shell

# === 前端 ===
cd frontend && npm run dev              # 启动开发服务器
cd frontend && npm run build            # 生产构建
cd frontend && npm run lint             # 代码检查
```

---

## 目录索引

### 后端配置文件

| 文件 | 说明 |
|------|------|
| `manage.py` | Django 管理入口 |
| `backend/settings.py` | 全局配置（DB/DRF/JWT/CORS/Celery/日志/国际化） |
| `backend/urls.py` | 根路由配置 |
| `backend/celery.py` | Celery 异步任务配置 |
| `backend/asgi.py` | ASGI 入口（Channels WebSocket） |
| `backend/middleware.py` | 自定义中间件（DisableCSRF） |
| `.env` | 环境变量（密钥/数据库/Redis/邮件） |
| `requirements.txt` | Python 依赖列表 |

### 后端 App 索引

| 模块 | 目录 | 模型数量 | 主要文件 |
|------|------|----------|----------|
| users | `apps/users/` | 2 | models.py, views.py, serializers.py, urls.py |
| projects | `apps/projects/` | 3 | models.py, views.py, project_list_views.py |
| testcases | `apps/testcases/` | 4 | models.py, views.py, serializers.py |
| testsuites | `apps/testsuites/` | 2 | models.py（视图未实现） |
| executions | `apps/executions/` | 4 | models.py, views.py, serializers.py |
| reports | `apps/reports/` | 2 | models.py, views.py |
| reviews | `apps/reviews/` | 4 | models.py, views.py, serializers.py |
| versions | `apps/versions/` | 1 | models.py, serializers.py |
| api_testing | `apps/api_testing/` | 13 | models.py, views.py, urls.py, utils.py, variable_resolver.py |
| ui_automation | `apps/ui_automation/` | 20+ | models.py, views.py, selenium_engine.py, playwright_engine.py, ai_agent.py |
| app_automation | `apps/app_automation/` | 14+ | models.py, views/, consumers.py, executors/, managers/ |
| requirement_analysis | `apps/requirement_analysis/` | 9 | models.py, ai_models.py, views.py, services.py |
| assistant | `apps/assistant/` | 4 | models.py, views.py, views_config.py |
| core | `apps/core/` | 1 | models.py, management/commands/ |
| data_factory | `apps/data_factory/` | 1 | models.py, tools/, views.py |

### 前端文件索引

| 类别 | 文件/目录 | 说明 |
|------|-----------|------|
| 入口 | `main.js` | 应用初始化 |
| 路由 | `router/index.js` | 全部路由配置 |
| HTTP 客户端 | `utils/api.js` | Axios 封装（Token管理/拦截器） |
| 状态管理 | `stores/user.js` | 用户认证 |
| 状态管理 | `stores/app.js` | 语言设置 |
| 布局 | `layout/index.vue` | 主布局（侧边栏 + 顶栏） |
| API 服务 | `api/api-testing.js` | API 测试接口 |
| API 服务 | `api/ui_automation.js` | UI 自动化接口（最大文件） |
| API 服务 | `api/app-automation.js` | APP 自动化接口 |
| API 服务 | `api/requirement-analysis.js` | 需求分析接口 |
| API 服务 | `api/data-factory.js` | 数据工厂接口 |
| API 服务 | `api/core.js` | 通知配置接口 |
| 视图 | `views/api-testing/` | 11 个页面 + 3 个组件 |
| 视图 | `views/ui-automation/` | 17 个页面 |
| 视图 | `views/app-automation/` | 12 个页面 |
| 视图 | `views/requirement-analysis/` | 6 个页面 |
| 国际化 | `locales/lang/{zh-cn,en}/` | 34 个翻译文件 |
| 配置 | `vite.config.js` | 构建配置 |
| 配置 | `package.json` | 前端依赖 |
