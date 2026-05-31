# CLAUDE.md

## 项目概述

此文件为 Claude Code（claude.ai/code）在处理本代码库中的代码时提供了指导。

TestHub是一个由Django 4.2（后端）+ Vue 3（前端）构建的ai驱动的测试管理平台。它提供了测试用例管理、API测试、UI自动化测试、人工智能驱动的需求分析和测试用例生成功能。

## 常用命令

# 激活虚拟环境（Windows）

d:\TestHub\venv\Scripts\Activate.ps1

# 激活虚拟环境（MacOS）

source .venv/bin/activate

### 后端 (Django)

```bash
# 启动开发服务器
python manage.py runserver

# 数据库迁移
python manage.py makemigrations
python manage.py migrate

# 创建超级用户
python manage.py createsuperuser

# 运行所有计划任务（API测试+ UI自动化）
python manage.py run_all_scheduled_tasks

# 初始化UI自动化定位器策略
python manage.py init_locator_strategies

# 下载用于UI自动化的web驱动程序
python manage.py download_webdrivers
```

### 前端 (Vue 3 + Vite)

```bash
cd frontend

# 安装依赖关系
npm install

# 启动开发服务器
npm run dev

# 为生产而构建
npm run build

# 线头代码
npm run lint
```

## 体系结构

### 后端结构 (`apps/`)

Django项目使用模块化的应用程序结构 `apps/`:

- **users**: 用户身份验证和配置文件管理（自定义用户模型）
- **projects**: 项目和团队管理
- **testcases**: 手动测试用例管理，包括步骤、附件、注释
- **testsuites**: 测试套件组织
- **executions**: 测试计划执行和结果跟踪
- **reports**: 测试报告生成
- **reviews**: 带有模板和分配的测试用例审查工作流
- **versions**: 版本/发布管理
- **requirement_analysis**: ai驱动的需求文档解析（PDF/Word/TXT）和测试用例生成
- **assistant**: Dify AI聊天机器人集成
- **api_testing**: API测试模块（HTTP/WebSocket、环境、计划任务、Allure报告）
- **ui_automation**: UI自动化与硒/剧作家，元素管理，页面对象，AI智能模式

### 前端结构 (`frontend/src/`)

- **views/**: 按特性模块组织的页面组件
- **api/**: API服务层
- **stores/**: Pinia 状态管理
- **router/**: Vue路由器配置
- **components/**: 共享组件
- **layout/**: 布局组件

### 关键配置文件

- `backend/settings.py`: Django的设置 (database, REST framework, CORS, Celery, email)
- `frontend/vite.config.js`: 虚拟构建配置
- `.env`: 环境变量（DB凭据、API密钥、电子邮件配置）

## API结构

所有API端点的前缀都是 `/api/`:
- `/api/auth/` and `/api/users/`: 用户身份验证
- `/api/projects/`: 项目管理
- `/api/testcases/`: 测试用例CRUD
- `/api/testsuites/`: 测试套件管理
- `/api/executions/`: 测试执行
- `/api/reports/`: 报告生成
- `/api/reviews/`: 评审工作流程
- `/api/versions/`: 版本管理
- `/api/assistant/`: 人工智能助手聊天
- `/api/requirement-analysis/`: 人工智能需求分析
- `/api/` (api_testing): API测试端点
- `/api/ui-automation/`: UI自动化端点

API文档可在‘ / API /docs/ ’ （Swagger）和‘ / API /redoc/ ’ （redoc）处获得。

## 数据库

MySQL 8.0+ with `utf8mb4` charset.通过环境变量进行配置:
- `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`

## 人工智能集成

该平台支持配置的多个AI提供商 `requirement_analysis.AIModelConfig`:
- DeepSeek, Qwen (通义千问), SiliconFlow (硅基流动), OpenAI-compatible APIs
- AI roles: `testcase_writer`, `testcase_reviewer`, `browser_use_text`, `browser_use_vision`

AI模式使用‘ browser-use ’库和LangChain来实现智能浏览器自动化（' apps/ui_automation/ai_agent.py '）。

## 测试提示模板

中定义了生成AI测试用例的自定义提示:
- `tester.md`: 测试用例作者角色和输出格式
- `tester_pro.md`: 测试用例审稿人角色

## 关键依赖关系

后端（service）: Django REST Framework, drf-spectacular, django-filter, celery, httpx, selenium, playwright, browser-use, langchain-openai

前端（Frontend）: Vue 3, Element Plus, Pinia, Vue Router, Axios, ECharts, Monaco Editor, xlsx

## Commit 规范
- 默认不自动提交代码
- 多个相关修改应合并为一个 commit
- commit message 格式：`<type>: <简短描述>`
- 提交前必须运行 lint 和测试