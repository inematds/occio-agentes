# Stack Técnica

> Escolhas concretas de tecnologia com justificativa. Otimizado para solo/time pequeno com stack TS + Python.

---

## Front-end

| Escolha | Versão | Por quê |
|---|---|---|
| **Next.js** | 15 (App Router) | Server Components, streaming nativo, deploy Vercel zero-config |
| **React** | 19 | use() hook, Actions, compatível com streaming do AI SDK |
| **TypeScript** | 5.6+ strict | Tipos a partir do backend via OpenAPI |
| **Tailwind CSS** | 4.x | Padrão de mercado, funciona bem com shadcn |
| **shadcn/ui** | latest | Componentes acessíveis copiados no repo, sem vendor lock |
| **Zustand** | 5.x | Estado cliente leve; Context + useReducer seria ok para MVP |
| **Vercel AI SDK** | latest | Streaming estruturado, `useChat`, tool-call rendering |
| **TanStack Query** | 5.x | Cache de servidor + revalidação |
| **Framer Motion** | 11.x | Animação da timeline do squad |

**Evitamos:** Redux (excessivo), Jotai (sobreposição com Zustand), MUI (design genérico), CSS-in-JS runtime (overhead).

---

## Back-end (API + Orquestrador)

| Escolha | Versão | Por quê |
|---|---|---|
| **Python** | 3.12 | Performance melhor que 3.11, type hints maduras |
| **uv** | latest | 10-100x mais rápido que pip, resolve dependências determinísticas |
| **FastAPI** | 0.115+ | Async nativo, OpenAPI auto-gerado, ecossistema maduro |
| **Pydantic** | v2 | Fonte da verdade dos tipos, gera TS via `datamodel-code-generator` |
| **LangGraph** | 0.2+ | Orquestrador escolhido (ver abaixo) |
| **SQLAlchemy + asyncpg** | 2.x | ORM + driver async; alternativa: SQLModel |
| **Alembic** | latest | Migrations |
| **Arq** | latest | Fila + workers em asyncio + Redis — mais leve que Celery |

**Alternativas consideradas:**
- **CrewAI:** mais alto nível, mas menos controle. Usa multi-agente por dentro. Bom para protótipo; não para produção com HITL sério.
- **AutoGen:** Microsoft, maduro, mas API instável entre versões.
- **LangChain puro:** sem o Graph, não dá controle bom de estado para HITL.

**LangGraph ganha porque:**
1. Estado persistente em Postgres (`PostgresSaver`) — runs sobrevivem a deploy
2. `interrupt()` nativo para HITL
3. Parallelism como primitive, não hack
4. LangGraph Studio para debug visual
5. Integra com LangSmith/Langfuse para traces

---

## LLMs

| Modelo | Uso primário | ID |
|---|---|---|
| **Claude Sonnet 4.6** | Padrão para raciocínio e tool use | `claude-sonnet-4-6` |
| **Claude Haiku 4.5** | Triagem, classificação, respostas simples | `claude-haiku-4-5-20251001` |
| **Claude Opus 4.7 (1M)** | Casos especiais: contexto longo, decisões críticas | `claude-opus-4-7` |
| GPT-4.1 | Fallback + validação cruzada | — |
| Gemini 2.5 Pro | Fallback + vision barata | — |

**Regras de roteamento:**
- Sempre começar com Haiku para classificar
- Escalar para Sonnet se Haiku indicar complexidade ou tool use obrigatório
- Opus só em runs marcadas como "high stakes" pelo usuário ou pelo Planner
- **Prompt caching obrigatório** em todos os agentes

---

## Dados

| Componente | Escolha | Por quê |
|---|---|---|
| **Postgres** | via Supabase | Auth + DB + Storage + Realtime num lugar só |
| **pgvector** | extension | Embeddings de docs para RAG |
| **Redis** | via Railway | Fila Arq + cache + session locks |
| **Storage** | Supabase Storage | S3-compatible, barato, integrado com auth |

**Por que Supabase vs. construir stack do zero:**
- Auth funcional em 1 dia (magic link, OAuth, MFA)
- RLS (Row Level Security) elimina classe inteira de bugs
- Edge Functions para triggers pontuais
- Self-host disponível quando volume justificar

---

## Sandbox

| Escolha | Notas |
|---|---|
| **E2B** | Primeira escolha — SDK Python pronto, cold start < 2s |
| Daytona | Backup — self-hostable |

Usados para: execução de código gerado pelo LLM, processamento de arquivos do usuário em ambiente isolado.

---

## Integrações (Opção C — e-commerce BR)

| Serviço | Tipo | Doc |
|---|---|---|
| Mercado Livre | REST + OAuth 2.0 | [developers.mercadolivre.com.br](https://developers.mercadolivre.com.br/) |
| Shopee BR | REST + signature | [open.shopee.com](https://open.shopee.com/) |
| WhatsApp Business (Meta) | REST + webhooks | [developers.facebook.com/docs/whatsapp](https://developers.facebook.com/docs/whatsapp) |
| Z-API (não oficial) | REST | [z-api.io](https://z-api.io) — MVP barato |
| Evolution API | Self-host | Alternativa open source ao Z-API |
| Focus NFe | REST — NF-e, NFC-e, NFS-e | [focusnfe.com.br](https://focusnfe.com.br/) |
| Melhor Envio | REST + OAuth | [melhorenvio.com.br/docs](https://melhorenvio.com.br/docs) |
| Asaas | REST — PIX + cobrança recorrente | [asaas.com/docs](https://asaas.com/docs) |
| Stripe | REST — alternativa internacional | [stripe.com/docs](https://stripe.com/docs) |
| Meta Marketing API | REST — Ads | [developers.facebook.com/docs/marketing-apis](https://developers.facebook.com/docs/marketing-apis) |
| Google Ads API | REST | [developers.google.com/google-ads/api](https://developers.google.com/google-ads/api) |

---

## Observabilidade

| Ferramenta | Uso |
|---|---|
| **Langfuse** | Traces de LLM, custo por run, debug de prompts |
| **Sentry** | Erros de aplicação (front + API) |
| **OpenTelemetry** | Traces distribuídos fase 3 |
| **Grafana Cloud** | Dashboards de negócio fase 3 |

---

## Deploy

### MVP (R$ 500–1000/mês)
- **Vercel** — front (free tier suficiente no começo)
- **Railway** — API + workers (plano Hobby)
- **Supabase** — DB + Auth + Storage (free tier)
- **Cloudflare** — DNS + CDN + proteção
- **E2B** — pay-per-use

### Beta (R$ 3k–10k/mês)
- Upgrade Railway para Pro
- Supabase Pro
- Langfuse self-host em VPS simples
- Backups redundantes

### Produto sério (pós-PMF)
- Migrar API para AWS ECS ou GCP Cloud Run com IaC (Terraform)
- Multi-AZ Postgres
- Observabilidade enterprise (Datadog ou Grafana Cloud Pro)

---

## Ferramentas de desenvolvimento

| Ferramenta | Uso |
|---|---|
| **Turborepo** | Monorepo + cache de builds |
| **pnpm** | Gerenciador de pacotes Node (rápido, workspace-friendly) |
| **uv** | Gerenciador de pacotes Python |
| **Ruff** | Lint + format Python |
| **mypy** | Type check Python |
| **Biome** ou ESLint + Prettier | Lint + format TS |
| **Vitest** | Testes front |
| **pytest** | Testes back |
| **Playwright** | E2E |
| **GitHub Actions** | CI/CD |
| **Cursor** ou VSCode + Claude Code | IDE |

---

## O que explicitamente NÃO usamos (ainda)

| Tech | Por que não |
|---|---|
| Kubernetes | Excesso para MVP; Railway/Fly resolvem |
| Kafka | Redis + Arq bastam até milhares de msgs/min |
| GraphQL | OpenAPI do FastAPI gera tipos TS de graça |
| Monorepo com Nx | Turborepo é mais simples |
| Electron/Tauri (desktop) | Começamos web; desktop só se houver demanda enterprise local-first |
| React Native | Web responsivo + PWA atendem fase 1–3 |
| MongoDB | Postgres resolve — JSONB onde precisar |
