# NovaMart Customer Support — Enterprise Multi-Agent AI System

![Python 3.12](https://img.shields.io/badge/Python-3.12-blue) ![AWS Bedrock](https://img.shields.io/badge/AWS-Bedrock-orange) ![Strands Agents](https://img.shields.io/badge/Strands-Agents-green) ![Score](https://img.shields.io/badge/Score-120%2F120-brightgreen)

> Production-grade multi-agent customer support system built on Amazon Bedrock AgentCore, featuring parallel RAG retrieval, shared workflow state, enterprise guardrails, and full observability.

---

## Architecture

```
Customer Request
      │
 OrchestratorAgent (Claude Haiku 4.5)
      │
  ┌───┼───────────────────┬──────────────────┐
  │   │                   │                  │
Inventory  Policy      Refund       Communication
           │
  ┌────────┼────────┐
Returns  Shipping  Warranty
  KB       KB       KB
  └──── parallel ──┘

Shared State: DynamoDB WorkflowStateTable (optimistic locking)
```

---

## What It Does

NovaMart is an e-commerce platform that uses this system to automate customer support end-to-end:

- Intelligently routes requests to specialist agents
- Retrieves policy information via parallel multi-agent RAG (3 Bedrock Knowledge Bases)
- Enforces enterprise safety guardrails (PII, content, competitor topics)
- Maintains shared workflow state across concurrent agent calls
- Produces complete, grounded, empathetic customer responses
- All traced end-to-end with X-Ray and CloudWatch

---

## Tech Stack

| Technology | Purpose |
|---|---|
| Amazon Bedrock (Claude Haiku 4.5 / Sonnet 4.5) | LLM backbone for all agents |
| Amazon Bedrock AgentCore | Managed runtime deployment & memory |
| Strands Agents SDK | Agent orchestration framework |
| Amazon DynamoDB | Order data + shared workflow state |
| Amazon S3 Vectors | Vector store backing Bedrock Knowledge Bases |
| Bedrock Knowledge Bases | Parallel RAG policy retrieval |
| Bedrock Guardrails | Content safety, PII, topic blocking |
| AWS X-Ray | Distributed tracing across agent calls |
| Amazon CloudWatch | Centralized logging & observability |
| Python 3.12 | Runtime language |

---

## Agent Responsibilities

| Agent | Model | Role | Tools |
|---|---|---|---|
| OrchestratorAgent | Claude Haiku 4.5 | Routes requests, manages WorkflowState | initialize_session, route_to_* |
| InventoryAgent | Claude Sonnet 4.5 | Fetches order & customer data from DynamoDB | check_order_status, get_customer_tier, list_customer_orders |
| PolicyAgent | Claude Sonnet 4.5 | Parallel RAG across 3 Knowledge Bases | search_all_policies |
| RefundAgent | Claude Sonnet 4.5 | Evaluates return/refund eligibility | get_inventory_context, initiate_refund |
| CommunicationAgent | Claude Sonnet 4.5 | Drafts final customer-facing response | get_full_workflow_context |

---

## Key Features

- **Parallel Multi-Agent RAG**: PolicyAgent fans out to 3 retriever sub-agents (Returns, Shipping, Warranty KBs) using ThreadPoolExecutor, merges and deduplicates results by relevance
- **Shared Workflow State**: DynamoDB `WorkflowStateTable` with optimistic locking (version-based conditional writes) ensures consistency across concurrent agent updates
- **Enterprise Guardrails**: Bedrock Guardrails block harmful content, PII redaction, competitor topics, and legal threats
- **Full Observability**: Every request traced to AWS X-Ray (Orchestrator → Worker → Knowledge Base call chain) and logged to CloudWatch; works identically locally and in the deployed runtime
- **Managed Runtime**: Deployed to Amazon Bedrock AgentCore Runtime (arm64, Python 3.12) via AgentCore CLI
- **Conversation Memory**: AgentCore Memory (SESSION_SUMMARY strategy) maintains context across turns

---

## How to Run Locally

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Configure AWS credentials (standard AWS credential chain)
export AWS_REGION=us-east-1

# 3. Copy the environment template and fill in your values
cp .env.example .env
# Edit .env: add your Bedrock KB IDs, Runtime ARN, Guardrail ID/Version

# 4. Verify AWS resources are reachable
python config.py

# 5. Seed DynamoDB and S3 with test data
python infrastructure/seed_data.py

# 6. Run 3 end-to-end test scenarios (local, traced to X-Ray)
python src/agent_orchestrator.py test

# 7. Interactive chat mode
python src/agent_orchestrator.py chat

# 8. Deploy to AgentCore Runtime
python src/agent_orchestrator.py deploy
```

---

## Test Scenarios

| Scenario | Request | Expected Route |
|---|---|---|
| 1 | "I want to return my order ORD-27176" | Orchestrator → Inventory → Refund → Communication |
| 2 | "What is the return policy for premium customers?" | Orchestrator → Policy (3 parallel KBs) → Communication |
| 3 | "How much are 5 items at $29.99 with 10% off?" | Orchestrator → Communication (direct calculation) |

---

## Project Structure

```
starter/
├── config.py                    # Central configuration (CloudFormation exports + .env)
├── requirements.txt
├── .env.example                 # Environment variable template
├── src/
│   ├── agent_orchestrator.py   # ⭐ Core: 5-agent graph, guardrails, observability
│   ├── agent_observability.py  # X-Ray tracing + CloudWatch logging
│   ├── bedrock_kb_retrieval.py # Bedrock Knowledge Base retrieval helper
│   └── demo.py
├── infrastructure/
│   ├── starter_stack.yaml      # CloudFormation: DynamoDB, S3, S3 Vectors, IAM
│   └── seed_data.py
├── agentcore/
│   └── agentcore.json          # AgentCore CLI runtime definition
└── tests/
    └── test_agent.py           # Automated test suite
```

---

## Results

- **Test score: 120/120** — all automated tests passing
- Full X-Ray trace captured: Orchestrator → Inventory/Policy/Refund → Communication chain visible in AWS Console
- See `submission_screenshots/` for X-Ray Service Map and test output screenshots

---

## Skills Demonstrated

| Skill Area | Details |
|---|---|
| Multi-Agent Systems | Hierarchical orchestration with 5 specialized agents |
| Retrieval-Augmented Generation | Parallel RAG across 3 Bedrock Knowledge Bases with result merging |
| AWS Cloud Architecture | Bedrock, DynamoDB, S3 Vectors, CloudWatch, X-Ray, AgentCore |
| Python Engineering | Concurrent execution, optimistic locking, async-safe shared state |
| Observability | Distributed tracing with X-Ray, structured logging with CloudWatch |
| AI Safety | Bedrock Guardrails: content filtering, PII redaction, topic blocking |
| Infrastructure as Code | CloudFormation stack, CDK-based AgentCore CLI deployment |

---

*Built with the Strands Agents SDK and Amazon Bedrock AgentCore.*
