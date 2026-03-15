# Architecture Overview

This document captures the current runtime architecture and the target split architecture.

## 1) Current Architecture (Single FastAPI Service)

```mermaid
flowchart TD
  subgraph Current[Current: Single FastAPI Service]
    U[User Browser]
    P["python main.py\n(Uvicorn + FastAPI)"]
    R1["GET /"]
    R2["GET /static/*"]
    WS["WebSocket /ws/{user_id}/{session_id}"]
    ADK["ADK Runner + Agent\n(liveagent/agent.py)"]

    U -->|HTTP| R1 --> P
    U -->|HTTP| R2 --> P
    U -->|Bidi stream| WS --> P
    P --> ADK
    ADK --> P
    P -->|events/audio/text| U
  end
```

### Notes
- One process serves frontend assets and backend websocket/API.
- Browser opens websocket to the same host/port.
- ADK runner and agent execute inside the same backend process.

---

## 2) Target Architecture (Split Frontend + Backend)

```mermaid
flowchart TD
  subgraph Future[Future: Split Frontend + Backend Service]
    U2[User Browser]
    FE["Frontend App\n(Static hosting / CDN)"]
    BE["Backend API Service\n(Cloud Run FastAPI)"]
    WS2["WebSocket /ws/*"]
    ADK2["ADK Runner + Agent"]
    STORE["Session Store\n(Redis/DB)"]

    U2 -->|Load UI| FE
    U2 -->|HTTP API + WS| BE
    WS2 --> BE
    BE --> ADK2
    ADK2 --> BE
    BE <--> STORE
    BE -->|streamed responses| U2
  end
```

### Notes
- Frontend is deployed independently from backend.
- Multiple frontends can reuse the same backend service.
- Shared session store is recommended for Cloud Run scaling.
