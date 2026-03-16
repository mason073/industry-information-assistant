# AI Assistant

基于多 Agent 协作的行业深度研究助手，集成 Deep Research、RAG 知识库、实时资讯采集、Text2SQL 数据分析等能力，帮助用户快速获取行业洞察。

## 核心功能

**Deep Research** — 多 Agent 协作的深度研究引擎，基于 LangGraph 构建。由 Architect（规划）、Scout（搜索）、Data Analyst（分析）、Writer（撰写）、Critic（审核）等专家 Agent 协同工作，自动完成从信息收集到报告生成的全流程。

**AI 对话** — 基于 ReAct 框架的智能对话系统，根据用户意图自动调用工具链：Web 搜索、知识库检索、Text2SQL 数据查询、智能数据分析、图表生成。支持多轮对话和会话管理。

**知识库管理** — 支持 PDF、Word、Excel、Markdown 等多格式文档上传，通过阿里云 DocMind 或本地解析进行文档切片，使用 Milvus 向量数据库实现语义检索，DashScope Rerank 提升召回精度。

**行业资讯** — 自动采集行业新闻、政策动态和招投标信息，支持定时任务调度，内置智慧交通、金融科技、医疗健康、能源电力四个行业配置。

**数据分析** — Text2SQL 自然语言查询数据库，智能数据分析与可视化，支持 ECharts 图表自动生成。

## 技术架构

```
┌─────────────────────────────────────────────────┐
│                   Frontend                       │
│        React 19 + TypeScript + Ant Design        │
│              Vite 8 + ECharts 6                  │
├─────────────────────────────────────────────────┤
│                   Backend                        │
│           FastAPI + LangGraph + OpenAI           │
│         LlamaIndex + ReAct Framework             │
├──────────┬──────────┬──────────┬────────────────┤
│ PostgreSQL│  Redis   │  Milvus  │ Elasticsearch  │
│  业务数据  │   缓存   │ 向量检索  │   全文检索     │
└──────────┴──────────┴──────────┴────────────────┘
```

| 层级 | 技术栈                                                                |
| ---- | --------------------------------------------------------------------- |
| 前端 | React 19, TypeScript, Vite 8, Ant Design 5, ECharts 6, React Router 7 |
| 后端 | FastAPI, LangGraph, OpenAI SDK, LlamaIndex, DashScope                 |
| 存储 | PostgreSQL 15, Redis 7, Milvus 2.3, Elasticsearch 8                   |
| 工具 | uv (Python), Docker Compose                                           |

## 快速开始

### 环境要求

- Docker 20.0+
- Python 3.12+
- Node.js 18+
- [uv](https://docs.astral.sh/uv/) (Python 包管理)

### 1. 克隆项目

```bash
git clone https://github.com/mason073/industry-information-assistant.git
cd industry-information-assistant
```

### 2. 启动基础服务

```bash
# 一键启动 PostgreSQL、Redis、Milvus、Elasticsearch
chmod +x start-services.sh
./start-services.sh start

# 或直接使用 Docker Compose
docker compose up -d
```

<details>
<summary>服务端口一览</summary>

| 服务          | 端口  | 默认账号                |
| ------------- | ----- | ----------------------- |
| PostgreSQL    | 5432  | postgres / postgres123  |
| Redis         | 6379  | -                       |
| Milvus        | 19530 | -                       |
| Elasticsearch | 1200  | -                       |
| MinIO Console | 9001  | minioadmin / minioadmin |

</details>

### 3. 配置环境变量

```bash
cd backend
cp .env.example .env
```

编辑 `.env`，填入以下必填项：

```env
# 阿里云百炼 - LLM & Embedding（必填）
# 申请地址: https://bailian.console.aliyun.com/
DASHSCOPE_API_KEY=sk-xxx

# 博查搜索 API（必填）
# 申请地址: https://open.bochaai.com/
BOCHA_API_KEY=xxx
```

<details>
<summary>可选配置项</summary>

| 变量                           | 说明                    | 申请地址                                                                         |
| ------------------------------ | ----------------------- | -------------------------------------------------------------------------------- |
| `OPENROUTER_API_KEY`           | OpenRouter 多模型网关   | [openrouter.ai](https://openrouter.ai/)                                          |
| `DOCMIND_ACCESS_KEY_ID/SECRET` | 阿里云 DocMind 文档解析 | [阿里云 RAM](https://help.aliyun.com/zh/ram/user-guide/create-an-accesskey-pair) |
| `BID_APP_KEY/SECRET/CODE`      | 招投标信息 API          | [阿里云市场](https://market.aliyun.com/detail/cmapi00063550)                     |
| `JUHE_STOCK_API_KEY`           | 聚合数据股票行情        | [juhe.cn](https://www.juhe.cn/docs/api/id/21)                                    |
| `SERPER_API_KEY`               | Serper 搜索 API         | [serper.dev](https://serper.dev/)                                                |

</details>

### 4. 启动后端

```bash
cd backend
uv sync
uv run python app/app_main.py
```

后端运行在 http://localhost:8000 ，API 文档: http://localhost:8000/docs

### 5. 启动前端

```bash
cd frontend
npm install --legacy-peer-deps
npm run dev
```

前端运行在 http://localhost:5173

## 项目结构

```
industry-information-assistant/
├── backend/
│   ├── app/
│   │   ├── config/                # 行业配置、LLM 配置
│   │   ├── core/                  # 数据库连接
│   │   ├── models/                # SQLAlchemy 数据模型
│   │   ├── router/                # API 路由
│   │   ├── schemas/               # Pydantic 数据结构
│   │   ├── service/
│   │   │   ├── deep_research_v2/  # Deep Research 多 Agent 引擎
│   │   │   │   ├── agents/        # Architect, Scout, Writer, Critic...
│   │   │   │   ├── graph.py       # LangGraph 工作流编排
│   │   │   │   └── state.py       # 全局工作记忆
│   │   │   ├── react_controller.py    # ReAct 对话控制器
│   │   │   ├── tool_executor.py       # 工具执行器
│   │   │   ├── chat_service.py        # 对话服务
│   │   │   ├── milvus_service.py      # 向量检索服务
│   │   │   ├── text2sql_service.py    # 自然语言转 SQL
│   │   │   ├── news_collection_service.py  # 资讯采集
│   │   │   └── ...
│   │   └── app_main.py            # 应用入口
│   ├── pyproject.toml             # Python 依赖 (uv)
│   ├── uv.lock
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── api/                   # API 接口层
│   │   ├── components/            # 通用组件
│   │   ├── pages/
│   │   │   ├── chat/              # AI 对话
│   │   │   ├── knowledge/         # 知识库管理
│   │   │   ├── news/              # 行业资讯
│   │   │   ├── bidding/           # 招投标信息
│   │   │   ├── database/          # 数据查询
│   │   │   └── memory/            # 记忆管理
│   │   ├── store/                 # 状态管理 (Valtio)
│   │   └── router/                # 路由配置
│   └── package.json
├── data/                          # 示例文档
├── docker/init-db/                # 数据库初始化脚本
├── docker-compose.yml             # 基础服务编排
└── start-services.sh              # 一键启动脚本
```

## Deep Research 架构

基于 LangGraph 实现的多 Agent 协作研究引擎，采用状态机驱动：

```
Init → Planning → Researching → Analyzing → Writing → Reviewing
                      ↑                                   │
                      └──── Re-Researching ←── (缺失信息) ─┘
                                                          │
                                              Revising ← (文字修改)
                                                  │
                                              Completed
```

| Agent            | 职责                                 |
| ---------------- | ------------------------------------ |
| **Architect**    | 分析研究课题，制定研究大纲和搜索策略 |
| **Scout**        | 执行多轮网络搜索，收集和整理信息     |
| **Data Analyst** | 数据分析、趋势预测、图表生成         |
| **Writer**       | 基于收集的信息撰写结构化研究报告     |
| **Critic**       | 对抗审核，发现信息缺失并驱动补充搜索 |
| **Wizard**       | 报告优化和润色                       |

## 常见问题

<details>
<summary>后端连接数据库失败</summary>

1. 确认基础服务已启动：`./start-services.sh status`
2. 检查 `.env` 中数据库配置
3. 如密码包含特殊字符（如 `@`），代码已做 URL 编码处理

</details>

<details>
<summary>前端 npm install 报错</summary>

```bash
rm -rf node_modules package-lock.json
npm install --legacy-peer-deps
```

</details>

<details>
<summary>研究历史无法恢复</summary>

执行数据库迁移后重启后端：

```sql
ALTER TABLE research_checkpoints ADD COLUMN IF NOT EXISTS ui_state_json JSONB;
ALTER TABLE research_checkpoints ADD COLUMN IF NOT EXISTS final_report TEXT;
```

</details>

## License

MIT
