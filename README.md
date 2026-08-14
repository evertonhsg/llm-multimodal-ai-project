# Multimodal AI Content Moderation System

A multi-agent content moderation system that reviews text, images, video, and audio in real time before customer-service messages are sent — built around four specialized Gemini-powered agents, structured Pydantic outputs, and full observability tracing.

## The scenario

A trainee customer service agent at a fictional company, ACME Enterprise, handles a support chat with a simulated (LLM-played) angry customer whose product — the ACME Power Widget Pro — has stopped working. Every message and file the trainee sends is moderated in real time to make sure it meets company communication standards before it reaches the customer.

## What it detects

| Content type | Agent | Flags |
|---|---|---|
| Text | `text_agent.py` | PII, unfriendly tone, unprofessional tone |
| Image | `image_agent.py` | PII / people in frame, disturbing content, low image quality |
| Video | `video_agent.py` | PII / people in frame, disturbing content, low quality |
| Audio | `audio_agent.py` | Transcription, PII, unfriendly tone, unprofessional tone |

Each agent returns a strongly-typed Pydantic result (e.g. `contains_pii`, `is_unfriendly`, `is_disturbing`) plus a written rationale explaining the decision — never a bare pass/fail.

## Architecture

```
Gradio Chat UI (frontend)  ---\
                                >--  4 specialized agents (text, image, video, audio)
FastAPI REST API (backend) ---/          using Google Gemini + Pydantic AI
                                                    |
                                          Pydantic structured results
                                                    |
                                      Arize Phoenix tracing (agent behavior + spans)
```

- **Specialized agents** — four independent moderation agents, each a `pydantic-ai` `Agent` with a custom system prompt and a typed `output_type`, calling Google Gemini.
- **LLM-as-customer** — `customer_agent.py` plays the angry customer, giving the trainee a realistic, evolving conversation to practice on.
- **Structured results** — every agent returns a `ModerationResult` subclass (`types/moderation_result.py`) with explicit boolean flags and a `rationale` field, so downstream systems can act on results programmatically instead of parsing free text.
- **Frontend, Gradio Chat UI** (`gradio_app.py`) — interactive web interface for chatting and uploading files, with real-time moderation feedback.
- **Backend, FastAPI REST API** (`fastapi_app.py`) — HTTP endpoints (`/moderate/text`, `/moderate/image`, etc.) wrapping the same agent logic, so a future production frontend (React/Vue/Angular) could reuse the backend without touching the AI layer.
- **Observability** — Arize Phoenix integration (`tracing.py`) for tracing and monitoring agent behavior end to end.

## Tech stack

| Category | Tools |
|---|---|
| Language | Python 3.12+ |
| LLM | Google Gemini, via `pydantic-ai` |
| Structured outputs | Pydantic |
| Frontend | Gradio |
| Backend API | FastAPI + Uvicorn |
| Observability | Arize Phoenix, OpenInference instrumentation |
| Testing | pytest, pytest-asyncio |
| Dependency management | uv |

## Repository structure

```
llm-multimodal-ai-project/
├── multimodal_moderation/
│   ├── agents/
│   │   ├── text_agent.py
│   │   ├── image_agent.py
│   │   ├── video_agent.py
│   │   ├── audio_agent.py
│   │   └── customer_agent.py       # simulated angry customer
│   ├── types/
│   │   └── moderation_result.py    # Pydantic output schemas
│   ├── gradio_app.py                # chat frontend
│   ├── fastapi_app.py               # REST backend
│   ├── tracing.py                   # Phoenix observability setup
│   └── app.py                       # runs frontend + backend + tracing together
├── tests/                           # pytest suite covering each agent + app
├── evals/                           # per-modality evaluation harness (text/image/audio/video)
├── pyproject.toml
├── requirements.txt
└── env.example
```

## Getting started

```bash
git clone https://github.com/evertonhsg/llm-multimodal-ai-project.git
cd llm-multimodal-ai-project

uv sync --dev
uv pip install -e .

cp env.example .env
# then fill in GEMINI_API_KEY and USER_API_KEY in .env
```

Run everything (backend, Gradio frontend, and Phoenix tracing) with:

```bash
uv run multimodal-moderation
```

- Chat UI: [http://localhost:7860](http://localhost:7860)
- FastAPI docs: [http://localhost:8000/docs](http://localhost:8000/docs)
- Phoenix traces: [http://localhost:6006/projects](http://localhost:6006/projects)

Run the test suite:

```bash
uv run pytest tests/ -vv
```

Run the evaluation harness per modality:

```bash
uv run evals/text/test_cases.py
uv run evals/image/test_cases.py
uv run evals/audio/test_cases.py
uv run evals/video/test_cases.py
```

## Skills demonstrated

Multi-agent orchestration, structured output design with Pydantic, prompt engineering (role/context/instruction-based prompts), multimodal LLM integration (text, image, video, audio), REST API design with FastAPI, LLM observability with Arize Phoenix, automated evaluation harnesses, test-driven development with pytest.

## About

Built by Everton Gomes as a hands-on generative AI engineering project: the agent architecture, structured moderation schemas, Gradio/FastAPI wiring, and evaluation harness were implemented end to end against a scaffolded assignment (tests provided, all core logic written from scratch).
