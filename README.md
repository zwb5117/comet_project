# Comet｜个人 AI 知识库与记忆助手

## 项目介绍

Comet 将文档、网页和图片整理成可检索的个人知识库，支持带引用的问答、跨会话记忆和深度研究报告。

- **知识库检索**：支持 PDF 文本、Word（`.docx`）、Markdown、TXT、HTML、网页正文和图片描述。结合向量与 BM25 检索，命中后补回上下文，便于回答时查阅原始资料。
- **对话与深度研究**：支持流式问答。研究模式将复杂问题拆成子问题，检索和整理证据，生成带来源的报告，并对初稿进行评分与定向修订。
- **长期记忆**：从用户表达中提取陈述和实体关系，在后续会话按需召回。用户可确认、修正或删除抽取出的实体。

前端使用 React、TypeScript 与 Vite；后端使用 FastAPI 与 LangChain。PostgreSQL 存储业务记录，Elasticsearch 提供资料检索，Neo4j 存储记忆图谱，Redis 与 Celery 支撑后台任务。

## 本地启动

需要 Docker Desktop（含 Docker Compose）、Python 3.12+、`uv` 和 Node.js 22。下面以 macOS 命令为例，Windows 的差异标在对应步骤。

**1. 启动依赖服务**

在项目根目录运行：

```bash
cp .env.example .env
docker compose up -d --build postgres elasticsearch neo4j redis
docker compose ps
```

Windows PowerShell 用 `Copy-Item .env.example .env` 替代 `cp`。

**2. 启动后端**

```bash
cd api
uv sync
cp .env.example .env
```

编辑 `api/.env`，设置随机的 `JWT_SECRET` 和有效的 `FERNET_KEY`，并使数据库连接配置与容器一致。`FERNET_KEY` 可用 `uv run python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"` 生成。

```bash
uv run alembic upgrade head
uv run python run.py
```

启动后可访问 [后端健康检查](http://localhost:8000/api/health)。

**3. 启动后台 Worker**

另开终端，在 `api/` 目录运行：

```bash
uv run celery -A app.celery_app.celery_app worker -l info -Q default,parse,memory,beat,research --pool=threads --concurrency=4
```

Windows 将 `--pool=threads --concurrency=4` 替换为 `--pool=solo`。文档解析、图片入库和记忆提取依赖 Worker。

**4. 启动前端并配置模型**

另开终端，在 `web/` 目录运行：

```bash
npm ci
npm run dev
```

打开 [Comet](http://localhost:5173)，注册后配置对话模型和 Embedding 模型。Embedding 输出维度须与索引一致，默认 **1024**；使用图片入库或联网研究时，再配置相应的多模态模型或联网搜索服务。