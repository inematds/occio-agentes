# Arquitetura de Referência

> Arquitetura técnica da plataforma. Vale para todas as opções de MVP — o que muda entre elas é o conjunto de agentes e integrações, não o esqueleto.

---

## Visão geral em camadas

```
┌──────────────────────────────────────────────────────────────────┐
│  CAMADA DE APRESENTAÇÃO                                          │
│  ─────────────────────                                           │
│  Next.js 15 (App Router) + React 19 + Tailwind + shadcn/ui       │
│  Vercel AI SDK (streaming) + Zustand (estado cliente)            │
│                                                                  │
│  Páginas-chave:                                                  │
│  • /chat            — input de objetivo + streaming de resposta  │
│  • /run/:id         — timeline do squad em execução              │
│  • /approvals       — inbox de ações aguardando aprovação humana │
│  • /skills          — biblioteca + editor de skills              │
│  • /dashboard       — métricas (vendas, custos, etc)             │
└──────────────────────────────┬───────────────────────────────────┘
                               │ HTTPS + WebSocket/SSE
┌──────────────────────────────┴───────────────────────────────────┐
│  CAMADA DE API                                                   │
│  ─────────────                                                   │
│  FastAPI (Python 3.12) + Pydantic v2                             │
│                                                                  │
│  Responsabilidades:                                              │
│  • Auth (JWT via Supabase/Clerk)                                 │
│  • Billing (Stripe/Asaas webhooks)                               │
│  • Rate limiting + quotas                                        │
│  • Roteamento para orquestrador                                  │
│  • WebSocket gateway para streaming de estado                    │
└──────────────────────────────┬───────────────────────────────────┘
                               │
┌──────────────────────────────┴───────────────────────────────────┐
│  CAMADA DE ORQUESTRAÇÃO                                          │
│  ──────────────────────                                          │
│  LangGraph (Python) — grafo de estados explícito                 │
│                                                                  │
│  Nós principais:                                                 │
│  • Planner        — objetivo → plano → seleção de agentes        │
│  • AgentRegistry  — catálogo de agentes disponíveis              │
│  • SquadRunner    — executa agentes em paralelo                  │
│  • ApprovalGate   — pausa o grafo aguardando aprovação humana    │
│  • Aggregator     — consolida resultados de múltiplos agentes    │
│                                                                  │
│  Persistência do estado do grafo: Postgres (checkpointer LG)     │
└──┬───────────────────┬──────────────────────┬────────────────────┘
   │                   │                      │
┌──┴─────────┐   ┌─────┴────────┐     ┌───────┴──────────┐
│ WORKERS    │   │ SANDBOX      │     │ INTEGRAÇÕES      │
│ Arq/Celery │   │ E2B/Daytona  │     │ WhatsApp/ML/NFe  │
│ + Redis    │   │ (containers  │     │ (SDKs+REST)      │
│            │   │  efêmeros)   │     │                  │
└────────────┘   └──────────────┘     └──────────────────┘
```

---

## Componentes em detalhe

### 1. Front — Next.js 15

**Por que Next.js:**
- Server Components para reduzir JS no cliente
- Streaming nativo + Vercel AI SDK para renderizar tokens em tempo real
- Deploy Vercel = zero-config + edge + preview por PR
- Ecossistema React é o mais profundo para UI complexa (timeline, dnd, etc.)

**Bibliotecas-chave:**
- `shadcn/ui` + Radix — primitivas acessíveis, copiadas para o repo (sem vendor lock)
- `zustand` — estado cliente leve, sem provedor gigante
- `ai` (Vercel AI SDK) — hook `useChat` + streaming estruturado
- `@tanstack/react-query` — cache de dados do servidor
- `framer-motion` — animação da timeline do squad

**Telas não-triviais:**
- **Timeline do squad:** cada agente é um swim-lane; eventos (thinking/tool_call/waiting_approval/done) aparecem como cards encadeados com animação
- **Approval inbox:** lista priorizada, com diff do que vai acontecer + botão one-click
- **Skills editor:** YAML + prompt + lista de tools; preview à direita

### 2. API — FastAPI

**Por que FastAPI:**
- Async nativo (essencial para proxy de LLM streaming)
- Pydantic v2 gera OpenAPI → tipos TS no front via `datamodel-code-generator`
- Baixo boilerplate, fácil de manter solo

**Estrutura sugerida:**
```
apps/api/
├── main.py
├── routers/
│   ├── runs.py          # POST /runs, GET /runs/:id (SSE)
│   ├── approvals.py     # GET/POST /approvals
│   ├── skills.py        # CRUD de skills
│   └── webhooks.py      # billing, marketplace events
├── core/
│   ├── auth.py
│   ├── db.py
│   └── settings.py
└── schemas/              # Pydantic — fonte da verdade dos tipos
```

### 3. Orquestrador — LangGraph

**Por que LangGraph e não CrewAI/AutoGen:**
- **Estado persistente em Postgres** via checkpointer — grafo sobrevive a restart
- **HITL nativo** com `interrupt()` — pausa em um nó, UI aprova, retoma
- **Paralelismo explícito** — nós em branch paralelo é o padrão, não workaround
- **Debug visual** — LangGraph Studio mostra o grafo em execução
- Menos "mágico" que CrewAI — você vê e controla cada transição

**Esboço do grafo principal:**
```
START
  │
  ▼
Planner ──► decide squad
  │
  ▼
Fan-out ──► [Agent A] [Agent B] [Agent C]  (paralelo)
                │         │         │
                ▼         ▼         ▼
              (cada um pode pedir approval → ApprovalGate)
                │         │         │
                └────┬────┴─────────┘
                     ▼
                 Aggregator
                     │
                     ▼
                 Responder (stream ao user)
                     │
                     ▼
                    END
```

### 4. Workers e filas

**Arq** (baseado em asyncio+Redis) é mais leve que Celery e integra melhor com FastAPI. Celery entra se houver necessidade de tarefas longas (>30min) com garantias fortes.

**Uso típico:**
- Emissão de NF-e (lento, tem retry obrigatório)
- Chamadas a marketplaces com rate limit
- Batch de re-precificação diária
- Webhooks recebidos que disparam agentes

### 5. Sandbox de execução

**E2B** ou **Daytona** para quando um agente precisar:
- Rodar código Python/Node gerado pelo LLM
- Processar arquivos enviados pelo usuário (planilhas, PDFs, imagens)
- Testar scripts antes de aplicar em produção

Container efêmero por sessão, sem acesso à rede interna, com limite de CPU/RAM/tempo.

### 6. Dados

| Store | Uso |
|---|---|
| Postgres (Supabase) | Fonte da verdade: usuários, runs, approvals, skills, produtos vinculados |
| pgvector | Embeddings para RAG (docs do usuário, histórico de conversas com clientes) |
| Redis | Cache + filas Arq + session locks |
| S3 (Supabase Storage ou R2) | Uploads de imagens, PDFs, artefatos gerados |

### 7. LLMs e roteamento

| Modelo | Uso | Custo relativo |
|---|---|---|
| **Claude Sonnet 4.6** | Padrão para raciocínio, tool use, approval-sensitive | 1x |
| **Claude Haiku 4.5** | Triagem, classificação, roteamento, respostas simples | ~0.1x |
| **Claude Opus 4.7 (1M)** | Casos especiais: long context, agentes críticos | ~5x |
| GPT-4.1 / Gemini 2.5 | Fallback + validação cruzada | — |

**Prompt caching obrigatório** em todos os agentes — reduz custo de system prompt em 90%.

### 8. Observabilidade

- **Langfuse** — traces de cada run, custo por run, latência por agente, debugging de prompts
- **Sentry** — erros de aplicação (front + API)
- **Grafana + Prometheus** (fase 3) — métricas de negócio (GMV processado, taxa de aprovação automática, etc.)

### 9. Deploy e infra

**MVP (baixo custo):**
- Front: Vercel
- API + Workers: Railway ou Fly.io
- Postgres/Redis/Storage: Supabase
- Sandbox: E2B (pay-per-use)
- DNS/CDN: Cloudflare

**Pós-product-market-fit:**
- Migrar API para AWS/GCP com IaC (Terraform)
- Considerar self-host do Supabase se volumes crescerem

---

## Padrões-chave

### HITL (Human-in-the-Loop)

Toda ação que entra em uma dessas categorias **pausa** o grafo e espera aprovação:

- Publicar conteúdo em canal externo (anúncio, post, e-mail enviado)
- Responder cliente (em modo supervisionado)
- Gastar dinheiro (Ads, frete, compra)
- Emitir documento fiscal
- Compartilhar dados do usuário com terceiros

Cada categoria tem política configurável:
- **Sempre pedir** (padrão conservador)
- **Auto-aprovar se score ≥ X** (agente dá confiança + justificativa)
- **Auto-aprovar em horário comercial, pedir fora** (ex: atendimento)
- **Nunca auto-aprovar** (ex: transferências financeiras acima de R$ X)

### Streaming de estado

Front abre um **WebSocket** (ou SSE) por run ativa. API publica eventos estruturados:

```json
{"type": "agent_started", "agent": "criador", "run_id": "..."}
{"type": "tool_call", "agent": "criador", "tool": "ml_create_listing", "args": {...}}
{"type": "approval_required", "id": "...", "summary": "...", "diff": {...}}
{"type": "agent_completed", "agent": "criador", "result": {...}}
```

Front traduz em atualizações na timeline sem refetch.

### Abstração de marketplaces

Cada marketplace tem um adapter com interface comum:

```python
class MarketplaceAdapter(Protocol):
    async def create_listing(self, product: Product) -> ListingId: ...
    async def fetch_questions(self, since: datetime) -> list[Question]: ...
    async def reply_question(self, qid: QuestionId, text: str) -> None: ...
    async def fetch_orders(self, since: datetime) -> list[Order]: ...
```

Implementações: `MercadoLivreAdapter`, `ShopeeAdapter`, `AmazonBRAdapter`. Testes de contrato rodam semanalmente contra sandbox de cada plataforma para pegar breaking changes cedo.

---

## Requisitos não-funcionais alvo

| Requisito | Alvo MVP | Alvo beta |
|---|---|---|
| Latência de streaming (primeiro token) | < 2s | < 1s |
| Tempo para squad iniciar | < 5s | < 3s |
| Uptime | 99% | 99.5% |
| Custo de LLM / cliente / mês | < R$ 50 | < R$ 30 |
| Taxa de aprovação automática (atendimento) | 30% | 60% |
| Recovery após restart da API | < 30s (via checkpointer) | < 10s |
