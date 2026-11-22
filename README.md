Universal Multi-Agent Platform

A full-stack platform for building, orchestrating, monitoring and evolving teams of AI agents. It combines hierarchical teams, swarm intelligence, shared memory, a knowledge graph, a tool marketplace, reputation scoring, human feedback loops, cost tracking and full decision explainability in one system.

Backend: Python, FastAPI, async workers
Frontend: React, Vite, Tailwind CSS, Redux slices, WebSockets
Database: SQL (schema and migrations included)
Infra: Docker, Docker Compose, Kubernetes manifests, GitHub Actions
Table of Contents
Overview
Key Features
Architecture
Tech Stack
Project Structure
Prerequisites
Quick Start
Environment Variables
Backend Guide
Frontend Guide
Database Guide
Agent System
Orchestration
Memory System
Knowledge Graph
LLM Layer and Routing
Tools and Marketplace
Learning and Self-Improvement
Reputation System
Explainability
Cost Tracking
API Overview
Real-Time Events
Security
Testing
Docker Deployment
Kubernetes Deployment
CI/CD
Helper Scripts
Troubleshooting
Roadmap
Contributing
License
Overview

Most agent frameworks solve one slice of the problem: a single agent loop, a crew of role-based agents, or a conversation between a few agents. This platform treats agents as long-lived, observable, improvable workers:

Agents can be created by hand or generated dynamically from a task description.
Agents can be grouped into hierarchical teams or released as swarms.
Every agent has short-term memory, long-term memory and access to a shared memory pool.
Every decision is traced so a human can ask "why did the agent do that?".
Agents are scored, ranked and evolved over time using feedback and performance data.
Costs, tokens and latency are tracked per agent, per task and per model.

The goal is a single control plane where you can design agent systems, run them, watch them think, correct them, and let them get better.

Key Features
Agent Management
Create, update, pause, resume and retire agents through the UI or API.
Built-in specialist agents: research, coding, math, finance, healthcare, robotics and enterprise.
Dynamic agent creation: describe a need and the platform assembles an agent with a fitting prompt, tools and memory configuration.
Per-agent configuration for model, temperature, tools, memory scope and permissions.
Teams and Swarms
Hierarchical teams: a manager agent decomposes work and delegates to worker agents, then merges the results.
Swarm controller: many lightweight agents work in parallel on a shared goal with configurable convergence rules.
Live swarm visualizer and activity feed in the dashboard.
Orchestration
LangGraph-style graph flows for deterministic, resumable pipelines.
CrewAI-style role and task management.
AutoGen-style conversational multi-agent loops.
Distributed planner and task scheduler for splitting large goals into subtasks.
MCP (Model Context Protocol) client for connecting to external tool servers.
Memory
Short-term (conversation and working memory).
Long-term (persistent, searchable).
Shared memory (team-wide and platform-wide).
Vector store with embeddings for semantic recall.
Knowledge Graph
Entity extraction from agent outputs and documents.
Graph builder and query layer.
Interactive graph canvas in the frontend.
Self-Improvement
Reflection engine: agents critique their own outputs.
Self-correction: automatic retry with a revised plan when a check fails.
Self-healing: workers detect failing agents and restart or replace them.
Evolution engine: promote strong prompts and configurations, retire weak ones.
Autonomous learning, continuous improvement and human feedback loops.
Governance and Observability
Decision tracer and reasoning logger for full explainability.
Event log of everything that happens in the system.
Telemetry hooks for metrics and tracing.
Sandbox for running untrusted code from tools safely.
Cost analytics dashboard.
Architecture
                    +----------------------------------+
                    |        React Frontend (Vite)     |
                    |  Dashboard | Agents | Swarms ... |
                    +----------------+-----------------+
                                     |
                         REST (axios) + WebSocket
                                     |
                    +----------------v-----------------+
                    |          FastAPI Backend         |
                    |   api/v1  ->  services  ->  core |
                    +----+--------+--------+-----------+
                         |        |        |
        +----------------+   +----+----+   +-----------------+
        |                    |         |                     |
+-------v-------+   +--------v--+  +---v--------+   +--------v-------+
|  Agent Layer  |   | LLM Layer |  |   Memory   |   |  Knowledge     |
| base, teams,  |   | providers |  | short/long |   |  Graph         |
| swarm, evolve |   | router    |  | shared/vec |   |  build/query   |
+-------+-------+   | cost      |  +---+--------+   +--------+-------+
        |           +-----------+      |                     |
+-------v--------+                     |                     |
| Orchestration  |                     |                     |
| graph / crew / |                     |                     |
| autogen / MCP  |                     |                     |
+-------+--------+                     |                     |
        |                              |                     |
+-------v------------------------------v---------------------v-------+
|                     Tools (registry, marketplace, sandbox)          |
+-------------------------------+-------------------------------------+
                                |
                    +-----------v-----------+
                    |   SQL Database        |
                    |   + Vector Store      |
                    +-----------------------+

        Background workers: task, swarm, learning, healing
Request Flow (Task Execution)
The user creates a task in the UI.
POST /api/v1/tasks validates the request and stores it.
The routing layer picks an agent, team or swarm based on task type, agent reputation and cost.
The task worker pulls the task and hands it to the chosen orchestration strategy.
Agents call the LLM router, tools and memory as needed.
Each step is written to the decision tracer and event log.
Results are validated by the reflection engine and, if required, corrected.
The final result, cost and trace are stored and pushed to the UI over WebSocket.
Feedback and reputation updates feed the learning workers.
Tech Stack
Layer	Technology
Backend framework	FastAPI
Language	Python 3.11+
Validation	Pydantic schemas
ORM / DB access	SQL models in app/models
LLM provider	Gemini (pluggable provider interface)
Orchestration	LangGraph-style flows, CrewAI-style crews, AutoGen-style chats
Tool protocol	MCP client
Frontend	React with Vite
Styling	Tailwind CSS and PostCSS
State	Redux-style slices
Realtime	WebSocket
Containers	Docker and Docker Compose
Orchestration (infra)	Kubernetes
CI/CD	GitHub Actions
Project Structure
universal-multi-agent-platform/
|
|-- README.md
|-- .gitignore
|-- .env.example
|-- docker-compose.yml
|-- LICENSE
|
|-- docs/                     Architecture, API, schema and lifecycle docs
|   `-- diagrams/             System, agent-flow and ER diagrams
|
|-- backend/
|   |-- main.py               FastAPI entry point
|   |-- requirements.txt
|   |-- pyproject.toml
|   |-- Dockerfile
|   |-- app/
|   |   |-- config.py         Settings loaded from environment
|   |   |-- database.py       DB engine and session handling
|   |   |-- dependencies.py   Shared FastAPI dependencies
|   |   |-- api/              Routers (v1 endpoints)
|   |   |-- core/             Security, logging, events, sandbox, telemetry
|   |   |-- agents/           Base agent, teams, swarm, evolution, specialists
|   |   |-- orchestration/    Graph, crew, autogen, planner, scheduler, MCP
|   |   |-- memory/           Short-term, long-term, shared, vector, embeddings
|   |   |-- knowledge/        Graph builder, query, entity extractor
|   |   |-- llm/              Providers, router, cost tracker, prompts
|   |   |-- tools/            Registry, marketplace, builtin and custom tools
|   |   |-- learning/         Autonomous learning, feedback, improvement
|   |   |-- explainability/   Decision tracer and reasoning logger
|   |   |-- models/           Database models
|   |   |-- schemas/          Request and response schemas
|   |   |-- services/         Business logic layer
|   |   |-- workers/          Background workers
|   |   `-- utils/            ID, time, validators, helpers
|   |-- tests/
|   `-- scripts/
|
|-- frontend/
|   |-- package.json
|   |-- vite.config.js
|   |-- tailwind.config.js
|   |-- postcss.config.js
|   |-- index.html
|   |-- Dockerfile
|   |-- public/
|   `-- src/
|       |-- api/              Axios client and per-resource API modules
|       |-- store/            State slices
|       |-- routes/           App routes and protected routes
|       |-- layouts/          Main, dashboard and auth layouts
|       |-- pages/            One file per screen
|       |-- components/       common, dashboard, agents, teams, swarms,
|       |                     tasks, knowledge, marketplace,
|       |                     explainability, charts
|       |-- hooks/            Data and WebSocket hooks
|       |-- contexts/         Auth, theme and WebSocket contexts
|       |-- utils/            Constants, formatters, validators, helpers
|       `-- styles/           Global CSS, animations, themes
|
|-- database/
|   |-- schema.sql            Full schema
|   |-- seed.sql              Sample data
|   |-- migrations/           001 to 011, applied in order
|   `-- backups/
|
|-- infra/
|   |-- docker/               Per-service Dockerfiles
|   |-- k8s/                  Deployments, StatefulSet, Ingress
|   `-- scripts/              setup.sh, start.sh, stop.sh
|
`-- .github/workflows/        ci-backend.yml, ci-frontend.yml, deploy.yml
Prerequisites
Tool	Version	Notes
Python	3.11 or newer	Backend
Node.js	18 or newer	Frontend
npm	9 or newer	Comes with Node
Docker	24 or newer	Optional, for containers
Docker Compose	v2	Optional
Git	any recent	Source control
LLM API key		Gemini key for the default provider
Quick Start
Option A: Docker Compose (recommended)
bash
git clone <your-repo-url> universal-multi-agent-platform
cd universal-multi-agent-platform

cp .env.example .env
# Edit .env and add your API keys and secrets

docker compose up --build

Then open:

Frontend: http://localhost:5173 (or the port set in docker-compose.yml)
Backend API: http://localhost:8000
Interactive API docs: http://localhost:8000/docs
Option B: Run locally

Windows (CMD):

bat
copy .env.example .env

cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000

In a second terminal:

bat
cd frontend
copy .env.example .env
npm install
npm run dev

macOS / Linux:

bash
cp .env.example .env

cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000

In a second terminal:

bash
cd frontend
cp .env.example .env
npm install
npm run dev
First Run Checklist
Start the database and apply database/schema.sql (or run migrations).
Optionally load database/seed.sql for demo data.
Start the backend and confirm GET /api/v1/health returns a healthy status.
Start the frontend and register the first user.
Create your first agent, then submit a task.
Environment Variables

Copy .env.example to .env at the project root. Suggested variables:

Backend
Variable	Description	Example
APP_ENV	Runtime environment	development
APP_NAME	Application name	universal-multi-agent-platform
APP_HOST	Bind host	0.0.0.0
APP_PORT	Bind port	8000
DEBUG	Debug mode	true
SECRET_KEY	Signing key for tokens	long random string
ACCESS_TOKEN_EXPIRE_MINUTES	Token lifetime	60
DATABASE_URL	Database connection string	postgresql://user:pass@db:5432/agents
GEMINI_API_KEY	Gemini API key	your-key
DEFAULT_MODEL	Default model name	set to your chosen model
EMBEDDING_MODEL	Embedding model name	set to your chosen model
VECTOR_STORE_PATH	Vector index location	./data/vectors
CORS_ORIGINS	Allowed frontend origins	http://localhost:5173
MAX_AGENT_STEPS	Safety cap on steps per task	25
SANDBOX_TIMEOUT_SECONDS	Code runner time limit	10
DAILY_COST_LIMIT_USD	Hard spending cap	20
LOG_LEVEL	Logging verbosity	INFO
Frontend
Variable	Description	Example
VITE_API_BASE_URL	Backend REST base URL	http://localhost:8000/api/v1
VITE_WS_URL	WebSocket URL	ws://localhost:8000/ws

Never commit .env. Only .env.example belongs in git.

Backend Guide
Layered Design
api/v1 (routers)  ->  services  ->  agents / orchestration / memory / llm / tools
                                 ->  models (database)
Routers only parse input, call a service and return a schema.
Services hold business logic and are the only layer routers call.
Models map to tables. Schemas define request and response shapes.
Core provides cross-cutting concerns used everywhere.
Core Modules
Module	Purpose
core/security.py	Password hashing, token creation and verification
core/logging.py	Structured logging setup
core/exceptions.py	Custom exception classes and handlers
core/events.py	Internal event bus and event types
core/sandbox.py	Isolated execution for untrusted code
core/telemetry.py	Metrics and tracing hooks
Running the Backend
bash
cd backend
uvicorn main:app --reload --host 0.0.0.0 --port 8000
Adding a New Endpoint
Add request and response schemas in app/schemas/.
Add a model in app/models/ if new data is stored.
Add business logic in app/services/.
Add a router in app/api/v1/ and register it in app/api/router.py.
Add tests in tests/.
Adding a New Specialist Agent
Create app/agents/specialists/my_agent.py and extend BaseAgent.
Define its system prompt, default tools and memory scope.
Register it in app/agents/specialists/__init__.py.
Add it to the dynamic creator's catalogue so it can be picked automatically.
Add a test in tests/test_agents.py.
Frontend Guide
Pages
Page	Purpose
Login / Register	Authentication
Dashboard	Platform health, activity feed, cost chart, leaderboard
Agents / Agent Detail / Create Agent	Manage agents, view memory and live thinking
Teams / Team Detail	Build and inspect hierarchical teams
Swarm	Configure, launch and watch swarms
Tasks / Task Detail	Submit tasks, follow the timeline, view results
Memory Explorer	Browse and search agent memory
Knowledge Graph	Explore entities and relationships
Tool Marketplace	Browse and install tools
Reputation	Agent scores and rankings
Feedback	Submit and review human feedback
Cost Analytics	Spend by agent, model and time
Explainability	Inspect decision traces
Settings	Account and platform preferences
Component Groups
common/ shared UI: Navbar, Sidebar, Button, Card, Modal, Loader, Toast, Badge, Tooltip, EmptyState, SearchBar.
dashboard/ widgets: StatsCard, ActivityFeed, CostChart, AgentHealthWidget, SwarmStatusWidget, ReputationLeaderboard.
agents/, teams/, swarms/, tasks/ domain components.
knowledge/ graph canvas, entity node and controls.
marketplace/ tool cards, grid and install modal.
explainability/ decision trace, reasoning step and panel.
charts/ line, bar, pie and donut charts.
State Management

Each domain has a slice in src/store/: agentSlice, teamSlice, swarmSlice, taskSlice, uiSlice, authSlice. API calls live in src/api/ and are wrapped by hooks in src/hooks/.

Running the Frontend
bash
cd frontend
npm install
npm run dev       # development server
npm run build     # production build
npm run preview   # preview the production build
Styling

Tailwind utility classes are used throughout. Shared tokens, animations and theme variables live in src/styles/. Light and dark themes are handled by ThemeContext.

Database Guide
Tables (by migration)
Migration	Area
001_init.sql	Users and base setup
002_agents.sql	Agents and agent configuration
003_teams.sql	Teams and membership
004_swarms.sql	Swarms and swarm members
005_tasks.sql	Tasks, subtasks and results
006_memory.sql	Memory records
007_knowledge_graph.sql	Entities and relationships
008_tools.sql	Tools and installations
009_reputation.sql	Reputation scores and history
010_feedback.sql	Human and automated feedback
011_costs_logs.sql	Cost logs and event logs
Applying the Schema

Fresh database:

bash
psql "$DATABASE_URL" -f database/schema.sql
psql "$DATABASE_URL" -f database/seed.sql   # optional demo data

Incremental, in order:

bash
for f in database/migrations/*.sql; do psql "$DATABASE_URL" -f "$f"; done
Backups

Store dumps in database/backups/. The folder is tracked with .gitkeep, but add *.sql or *.dump backup patterns to .gitignore so real data never gets committed.

bash
pg_dump "$DATABASE_URL" > database/backups/backup_$(date +%Y%m%d).sql
Agent System
Base Agent

Every agent extends BaseAgent and provides:

Identity: id, name, role, description.
Configuration: model, temperature, token limits.
Tools it is allowed to call.
Memory handles: short-term, long-term and shared.
A run loop: plan, act, observe, reflect.
Hooks for tracing, cost recording and event emission.
Agent Lifecycle
CREATED -> READY -> RUNNING -> (WAITING | REFLECTING | CORRECTING)
                        |
                        +-> COMPLETED
                        +-> FAILED -> HEALING -> READY
                        +-> PAUSED -> READY
                        +-> RETIRED

See docs/AGENT_LIFECYCLE.md for transitions and the events emitted at each stage.

Specialist Agents
Agent	Focus
Research	Search, read and summarise sources
Coding	Write, review and run code in the sandbox
Math	Calculation and step-by-step reasoning
Finance	Financial analysis and reporting
Healthcare	Clinical information support with strict safety prompts
Robotics	Planning and control-style task decomposition
Enterprise	Business workflows and internal process automation

Healthcare and finance agents provide information support only. They are not a substitute for qualified professionals, and outputs should be reviewed by a human before any real-world decision.

Hierarchical Teams

A team has one manager and many workers. The manager:

Breaks the goal into subtasks.
Assigns each subtask to the best-fit worker.
Collects the outputs.
Resolves conflicts and merges the final answer.
Swarms

The swarm controller launches many agents against one goal. Configurable options include swarm size, communication pattern, voting or consensus rule, step budget and stop conditions.

Dynamic Creator

Given a task description, the creator selects a specialist template, writes a tailored prompt, picks tools and returns a ready agent. Created agents can be saved or discarded after the task.

Self-Healing

The healing worker watches for stalled, looping or repeatedly failing agents. It can restart the agent, reset its working memory, swap its model, or replace it with a fresh copy and replay the task.

Orchestration
Module	Role
langgraph_flow.py	Graph-based pipelines with nodes, edges and checkpoints
crew_manager.py	Role and task based crews
autogen_manager.py	Conversation-driven multi-agent loops
distributed_planner.py	Decomposes large goals into a dependency graph of subtasks
task_scheduler.py	Priorities, retries, concurrency limits and scheduling
mcp_client.py	Connects to MCP servers and exposes their tools to agents
Choosing a Strategy
Situation	Use
Fixed, repeatable pipeline	Graph flow
Clear roles, sequential or parallel tasks	Crew
Open-ended reasoning between agents	AutoGen-style chat
Huge goal with many independent parts	Distributed planner plus swarm
Memory System
Layer	Scope	Lifetime	Storage
Short-term	One agent, one task	Task duration	In process
Long-term	One agent	Persistent	Database plus vectors
Shared	Team or platform	Persistent	Database plus vectors
How Recall Works
Text is converted into vectors by embeddings.py.
Vectors are saved through vector_store.py.
On recall, the query is embedded and the nearest records are returned.
Results are ranked by similarity, recency and importance.
Memory Hygiene
Set an expiry or importance score on records.
Periodically summarise old memories into compact notes.
Restrict shared memory writes to trusted agents.
Knowledge Graph
entity_extractor.py pulls entities and relations from text.
graph_builder.py inserts and merges nodes and edges.
graph_query.py supports neighbourhood, path and filter queries.
The frontend renders the graph with KnowledgeGraphCanvas, with filters in GraphControls.

Typical uses: tracing how facts connect, grounding agent answers, and finding gaps in what the system knows.

LLM Layer and Routing
Provider Interface

base_provider.py defines the contract every provider must meet: generate, stream, embed and report token usage. gemini_provider.py is the default implementation. To add another provider, subclass the base provider and register it with the router.

Router

router.py chooses a model per request using:

Task complexity and type.
Agent configuration and reputation.
Cost budget remaining.
Provider health and latency.

It also handles retries and fallbacks when a provider fails.

Prompt Templates

Reusable system and task prompts live in prompt_templates.py. Keep prompts versioned so the evolution engine can compare variants.

Tools and Marketplace
Built-in Tools
Tool	Description
web_search	Search the web and return snippets
calculator	Safe arithmetic and expression evaluation
code_runner	Execute code inside the sandbox
file_reader	Read supported file types
Registry

registry.py keeps a catalogue of available tools with name, description, input schema and permissions. Agents only see tools they are allowed to use.

Marketplace

marketplace.py and the Tool Marketplace page let users browse, install and remove tools. Custom tools go in app/tools/custom/.

Writing a Custom Tool
Create a file in app/tools/custom/.
Define an input schema and a run function.
Register the tool in the registry.
Declare the permissions it needs.
Test it in isolation before assigning it to agents.
Sandbox Rules

Anything that executes code or touches the file system must go through core/sandbox.py with a timeout, memory limit and no unrestricted network or file access.

Learning and Self-Improvement
Module	Purpose
reflection_engine.py	Agent critiques its own output against the goal
self_correction.py	Re-plans and retries after a failed check
evolution_engine.py	Mutates and selects prompts and configurations
autonomous_learning.py	Learns patterns from past runs
feedback_loop.py	Turns feedback into updates
continuous_improvement.py	Scheduled improvement cycles
human_feedback.py	Collects ratings and corrections from people
Evolution Cycle
Collect performance and feedback data.
Generate candidate variants of prompts or settings.
Evaluate candidates on recent or held-out tasks.
Promote the best, retire the worst.
Log every change so it can be rolled back.

Keep a human in the loop for any change that affects safety-sensitive agents.

Reputation System

Each agent has a score built from:

Task success rate.
Human feedback ratings.
Reflection and validation pass rate.
Cost efficiency.
Latency and reliability.

Scores decay slowly so recent performance matters more. The router uses reputation when choosing an agent, and the leaderboard on the dashboard displays the top performers.

Explainability
decision_tracer.py records each decision: input, options considered, choice made and reason.
reasoning_logger.py stores step-by-step reasoning in a structured format.
The Explainability page renders a trace as an expandable timeline.

Use traces for debugging, audits, compliance reviews and improving prompts.

Cost Tracking
cost_tracker.py records tokens and cost for every LLM call.
Costs roll up by agent, task, team, model and day.
The Cost Analytics page and the dashboard cost chart read from the cost log.
A daily limit can pause new work once the budget is reached.
API Overview

Base URL: /api/v1. Full details are in docs/API_REFERENCE.md and live at /docs when the server is running.

Area	Example endpoints
Auth	POST /auth/register, POST /auth/login
Agents	GET /agents, POST /agents, GET /agents/{id}, PUT /agents/{id}, DELETE /agents/{id}
Teams	GET /teams, POST /teams, GET /teams/{id}
Swarms	POST /swarms, GET /swarms/{id}, POST /swarms/{id}/stop
Tasks	POST /tasks, GET /tasks, GET /tasks/{id}, POST /tasks/{id}/cancel
Memory	GET /memory, POST /memory, POST /memory/search
Knowledge Graph	GET /knowledge-graph, POST /knowledge-graph/query
Tools	GET /tools, POST /tools/install, DELETE /tools/{id}
Reputation	GET /reputation, GET /reputation/{agent_id}
Feedback	POST /feedback, GET /feedback
Routing	POST /routing/preview
Reflection	POST /reflection/run
Evolution	POST /evolution/run, GET /evolution/history
Cost	GET /cost/summary, GET /cost/breakdown
Explainability	GET /explainability/{task_id}
Health	GET /health

The paths above describe the intended design. Check /docs for the exact routes once implemented.

Example: Create an Agent
bash
curl -X POST http://localhost:8000/api/v1/agents \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Market Researcher",
    "role": "research",
    "description": "Finds and summarises market information",
    "model": "default",
    "tools": ["web_search", "file_reader"]
  }'
Example: Submit a Task
bash
curl -X POST http://localhost:8000/api/v1/tasks \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Competitor overview",
    "description": "Summarise the top five competitors in our market",
    "strategy": "team",
    "priority": "normal"
  }'
Real-Time Events

The frontend opens a WebSocket through WebSocketContext and useWebSocket. Typical event types:

Event	Meaning
agent.status	Agent state changed
agent.thinking	Streaming thought or step output
task.updated	Task progress or status change
task.completed	Task finished with result
swarm.update	Swarm membership or progress changed
cost.update	New cost record added
system.alert	Health or limit warning

Events share a common envelope: type, timestamp, resource_id and payload.

Security
Authentication: token-based, with hashed passwords.
Authorization: protect routes with dependencies and check ownership of resources.
Secrets: keep keys in environment variables, never in code.
CORS: allow only known frontend origins.
Input validation: enforce schemas on every request.
Sandboxing: run untrusted code with strict limits.
Prompt injection: treat tool output and web content as untrusted data, not instructions.
Rate limiting: apply per user and per agent.
Spending caps: enforce daily cost limits.
Audit trail: every action lands in the event log.
Sensitive domains: healthcare, finance and robotics agents should require human approval for consequential actions.

Report vulnerabilities privately to the maintainers rather than opening a public issue.

Testing
Backend
bash
cd backend
pip install pytest pytest-asyncio httpx
pytest -v
Test file	Covers
test_agents.py	Agent creation, run loop, specialists
test_swarm.py	Swarm controller behaviour
test_memory.py	Memory layers and recall
test_tools.py	Registry and built-in tools
test_llm_router.py	Routing, fallback and cost logic
test_api.py	Endpoint behaviour end to end

Use conftest.py for shared fixtures such as a test database and a mocked LLM provider. Mock the LLM in tests so they are fast, free and deterministic.

Frontend
bash
cd frontend
npm run lint
npm run build

Add a test runner such as Vitest with React Testing Library as the UI grows.

Docker Deployment
Compose Services
Service	Purpose
backend	FastAPI application
frontend	Built static app served by a web server
db	SQL database
workers	Optional background workers
Commands
bash
docker compose up --build -d      # start in the background
docker compose logs -f backend    # follow backend logs
docker compose ps                 # list services
docker compose down               # stop and remove containers
docker compose down -v            # also remove volumes (deletes data)

Per-service Dockerfiles also live in infra/docker/ for separate builds in CI.

Kubernetes Deployment

Manifests are in infra/k8s/:

File	Resource
backend-deployment.yaml	Backend Deployment and Service
frontend-deployment.yaml	Frontend Deployment and Service
db-statefulset.yaml	Database StatefulSet with persistent storage
ingress.yaml	Ingress routing to frontend and backend
bash
kubectl create namespace agents
kubectl -n agents create secret generic app-secrets --from-env-file=.env
kubectl -n agents apply -f infra/k8s/
kubectl -n agents get pods

Production tips:

Store secrets in a secret manager, not plain files.
Set resource requests and limits for every pod.
Add readiness and liveness probes using /api/v1/health.
Scale workers independently from the API.
Enable TLS on the ingress.

See docs/DEPLOYMENT_GUIDE.md for a step-by-step walkthrough.

CI/CD
Workflow	Trigger	Actions
ci-backend.yml	Changes under backend/	Install, lint, run tests
ci-frontend.yml	Changes under frontend/	Install, lint, build
deploy.yml	Push to main or tagged release	Build images, push, deploy

Store registry credentials and cluster access as GitHub repository secrets.

Helper Scripts
Backend scripts (backend/scripts/)
Script	Purpose
seed_db.py	Insert demo users, agents and tasks
create_agents.py	Create the standard specialist agents
run_swarm_demo.py	Launch a sample swarm and print progress
bash
cd backend
python scripts/seed_db.py
python scripts/create_agents.py
python scripts/run_swarm_demo.py
Infra scripts (infra/scripts/)
Script	Purpose
setup.sh	Install dependencies and prepare the environment
start.sh	Start all services
stop.sh	Stop all services
Troubleshooting
Problem	Likely cause	Fix
type nul does nothing in PowerShell	Wrong shell	Use Command Prompt (cmd)
Backend cannot connect to DB	Wrong DATABASE_URL or DB not started	Check the URL and start the database
401 errors in the UI	Expired or missing token	Log in again
CORS error in browser	Origin not allowed	Add the frontend URL to CORS_ORIGINS
LLM calls fail	Missing or invalid API key	Check GEMINI_API_KEY
WebSocket will not connect	Wrong VITE_WS_URL	Match backend host and port
Frontend shows blank page	Build or runtime error	Check the browser console and npm run dev output
Port already in use	Another process on 8000 or 5173	Stop it or change the port
Agents loop forever	No step cap	Lower MAX_AGENT_STEPS
Costs growing fast	Large swarms or long contexts	Set DAILY_COST_LIMIT_USD and smaller swarms
Empty .png or .ico shows as broken	Placeholder files are 0 bytes	Replace with real images
Roadmap
[ ] Complete core CRUD for agents, teams and tasks
[ ] Gemini provider with streaming
[ ] Short-term and long-term memory with vector search
[ ] Hierarchical team execution
[ ] Swarm controller and live visualizer
[ ] Tool registry, sandbox and built-in tools
[ ] Reflection and self-correction loops
[ ] Reputation scoring and leaderboard
[ ] Knowledge graph builder and canvas
[ ] Explainability traces in the UI
[ ] Cost analytics and spending caps
[ ] Evolution engine with rollback
[ ] MCP server integration
[ ] Additional LLM providers
[ ] Multi-tenant workspaces and roles
[ ] Plugin SDK for third-party tools
Contributing
Fork the repository and create a branch: git checkout -b feature/my-change.
Make your changes with clear, focused commits.
Add or update tests.
Run the backend tests and the frontend build locally.
Update docs if behaviour changes.
Open a pull request describing what changed and why.
Code Style
Python: follow PEP 8, add type hints, keep functions small.
JavaScript/React: functional components, hooks, one component per file.
Commits: short imperative subject line, for example Add swarm stop endpoint.
Branches: feature/, fix/, docs/ prefixes.
Pull Request Checklist
[ ] Tests pass
[ ] No secrets or large files committed
[ ] Docs updated
[ ] Linting passes
[ ] Screenshots attached for UI changes
License

This project is released under the license in the LICENSE file. Add your chosen license text there (for example MIT or Apache 2.0) before publishing.

Acknowledgements

This design draws on ideas from the wider agent ecosystem, including graph-based workflows, role-based crews, conversational multi-agent systems and the Model Context Protocol.