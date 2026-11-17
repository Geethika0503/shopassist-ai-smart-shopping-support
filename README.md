# ShopAssist — Smart Shopping Support Assistant

ShopAssist is an AI-powered customer support assistant developed for **Northwind Store**, a fictional online shopping platform. It helps customers track orders, estimate delivery times, check return eligibility, and get answers to shopping-related questions.

Built using **FastAPI, LangChain, LiteLLM, and Groq**, the application supports tool calling, conversation memory, input validation, prompt-injection protection, and streaming responses.

## Features

- **Order Tracking:** Check order details and delivery status.
- **Delivery Estimation:** Estimate delivery times using pincodes and delivery speed.
- **Return Eligibility:** Verify returns based on delivery dates, product categories, and reasons.
- **Conversation Memory:** Maintain recent chat history for follow-up questions.
- **Input Validation:** Handle invalid order IDs, pincodes, and tool arguments.
- **Prompt Security:** Resist attempts to override system instructions.
- **Streaming Responses:** Support real-time responses using Server-Sent Events (SSE).
- **Docker Support:** Run the application and LLM gateway together.

## Tech Stack

| Component | Technology |
|---|---|
| Language | Python |
| API Framework | FastAPI |
| Agent Framework | LangChain |
| LLM Gateway | LiteLLM |
| Model Provider | Groq |
| Primary Model | `groq/openai/gpt-oss-20b` |
| Fallback Model | `groq/openai/gpt-oss-120b` |
| Containerization | Docker Compose |
| Dependency Management | uv |

## Architecture

```text
Customer / API Client
        |
        v
FastAPI Application (:8000)
        |
        +-- System Prompt
        +-- Conversation Memory
        +-- LangChain Tools
        |     |-- Order Status
        |     |-- Delivery Estimate
        |     |-- Return Eligibility
        |
        v
LiteLLM Gateway (:4000)
        |
        v
Groq (Primary / Fallback Model)
        |
        v
Customer Response
```

The FastAPI application handles customer requests and tool execution, while LiteLLM manages model access and fallback routing.

## Tools and Return Policy

| Tool | Purpose |
|---|---|
| `get_order_status` | Retrieves order information |
| `get_delivery_estimate` | Estimates delivery time |
| `check_return_eligibility` | Checks return eligibility |

**Return rules:**

- Standard returns: Within **10 days** of delivery.
- Damaged, defective, or wrong items: Within **30 days**.
- Innerwear, groceries, and gift cards: **Non-returnable**.
- Undelivered orders: Not eligible for return.

Tool inputs are validated, including order IDs (`ORD-12345`), six-digit pincodes, return reasons, and delivery speeds.

## Getting Started

### Prerequisites

- Python 3.12 and uv
- Docker Desktop (for Docker setup)
- Groq API key

### Environment Configuration

Configure the required API credentials, LiteLLM gateway settings, and application options in a local `.env` file.

Keep API keys and secrets out of version control.

### Option 1: Docker Compose (Recommended)

Build and start the application and gateway:

``` powershell
docker compose up --build -d
```

Verify the services:

```
docker compose ps
```

Access the application:

| Service | URL |
|---|---|
| API Documentation | http://localhost:8000/docs |
| Health Check | http://localhost:8000/health |
| LiteLLM Gateway | http://localhost:4000 |

Stop the services:

```bash
docker compose down
```

### Option 2: Run Locally

Install dependencies:

```bash
uv sync
```

Start the LiteLLM gateway in terminal 1:

```bash
uv run --with "litellm[proxy]==1.80.0" litellm --config gateway/config.yaml --port 4000
```

Start FastAPI in terminal 2:

```bash
uv run uvicorn app.main:app --port 8000
```

Ensure the required environment variables are available to both processes.

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Application health and configuration |
| POST | `/chat` | Process customer queries |
| POST | `/chat/stream` | Stream responses using SSE |
| DELETE | `/sessions/{id}` | Clear a conversation session |

**Example request — `POST /chat`:**

```json
{
  "session_id": "customer-001",
  "message": "Where is my order ORD-10233?"
}
```

## Memory and Security

**Conversation Memory**

ShopAssist uses window-based memory to retain recent conversation context while controlling token usage.

```env
MEMORY=window
KEEP_TURNS=4
```

Window memory used fewer tokens than the other strategies in the experiment, although it did not recall an older detail.

**Prompt Security**

The system prompt follows an instruction hierarchy, treats fake system messages as untrusted input, protects internal instructions, and relies on actual tool results for order information.

The assistant was evaluated against four prompt-injection attacks.

## Evaluation Results

Run the evaluation with the LiteLLM gateway available:

```bash
uv run python eval/run_eval.py
```

| Evaluation | Result |
|---|---|
| Golden test cases | **10/10** |
| Correct tool selection | **10/10** |
| Memory test | **Pass** |
| Prompt-injection attacks resisted | **4/4** |

All checks passed in the final evaluation run.

Detailed results are available in `experiments/mp1_results.md`, and memory experiments and design observations are documented in `experiments/mp1_report.md`.

## Limitations and Future Improvements

The application currently uses fictional order data and cannot process real refunds or cancellations. Conversation sessions are stored in application memory, so they are not persistent across restarts. Window memory may also lose older conversation details.

Future improvements include integrating a real order database, adding customer authentication, building a chat interface, supporting persistent sessions, and expanding automated security tests.

---
