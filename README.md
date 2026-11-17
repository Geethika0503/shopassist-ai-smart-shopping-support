# ShopAssist — Mini-Project 1

Order support for Northwind Store (a fictional online store): a FastAPI service with per-session memory, three
validated tools, a defended system prompt and streamed answers. The model is reached only through a LiteLLM
gateway, and both run together with Docker Compose.

**Author:** Geethika Dharmarao (GitHub: [Geethika0503](https://github.com/Geethika0503))
**Model tested:** `shopassist` → `groq/openai/gpt-oss-20b` (backup: `groq/openai/gpt-oss-120b`)

## Architecture

```
browser / client ──► app (FastAPI, :8000) ──► litellm gateway (:4000) ──► Groq
                     memory · prompt · tools    master key · fallback · cost      API_KEY lives only here
```

## API

| Method | Path | What it does |
|---|---|---|
| GET | `/health` | Liveness, plus the running model, gateway, memory strategy and tools |
| POST | `/chat` | `{"session_id", "message"}` → `{"answer", "tool_steps", "turns"}` |
| POST | `/chat/stream` | The same, streamed as Server-Sent Events: `tool`, `token`…, `done` |
| DELETE | `/sessions/{id}` | Forget one conversation |

## Run with Docker Compose (Path A)

```powershell
docker compose up --build        # gateway + app; Ctrl+C to stop, docker compose down to remove
```
Then open http://localhost:8000/docs to view the FastAPI Swagger documentation and available endpoints. To verify that the application is running, visit http://localhost:8000/health and confirm that the JSON response contains `"status": "ok"` along with the configured model, gateway, memory strategy and tools.

## Run without Docker (Path B)

```powershell
uv sync
# terminal 1 - the gateway
uv run --with "litellm[proxy]==1.80.0" litellm --config gateway/config.yaml --port 4000
# terminal 2 - the app
uv run uvicorn app.main:app --port 8000
```

## Evidence

```powershell
uv run python eval/run_eval.py      # needs the gateway running; writes experiments/mp1_results.md
```
Results:
- golden: 10/10
- golden_right_tools: 10/10
- memory: pass
- attacks_resisted: 4/4

Discussion: `experiments/mp1_report.md`.

## Design

ShopAssist uses a **window memory strategy** with `KEEP_TURNS=4` to remember recent conversation history while controlling token usage and cost. The application provides three tools: `get_order_status` to check order details, `get_delivery_estimate` to estimate delivery times based on pincodes, and `check_return_eligibility` to verify whether an item can be returned. Input validation checks order IDs, pincodes, return reasons and delivery speeds, while invalid arguments produce clear error messages instead of incorrect results. The system prompt follows an instruction hierarchy and protects against prompt-injection attacks by ignoring fake system messages, unauthorized instructions and fabricated tool responses. The LiteLLM gateway provides centralized model access, retry handling and a fallback model through Groq. One limitation is that ShopAssist uses fictional order data and cannot directly cancel orders, process refunds or make changes to customer accounts.
