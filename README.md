# accio-agents

> Plataforma de orquestração de agentes de IA inspirada no **Accio Work** (Alibaba International), adaptada para o mercado **SMB brasileiro**.

**Status:** fase de planejamento — abr/2026
**Stack alvo:** Next.js 15 (TypeScript) + FastAPI (Python) + LangGraph
**Público alvo:** pequenas e médias empresas brasileiras

---

## O que é este projeto

Uma plataforma onde o usuário descreve um **objetivo de negócio em linguagem natural** e o sistema monta dinamicamente um time ("squad") de agentes de IA especializados que trabalham em paralelo para executar o objetivo, pedindo aprovação humana nas etapas sensíveis.

**Exemplo concreto do vertical recomendado (e-commerce BR):**

> Usuário: "quero vender essa caneca personalizada no Mercado Livre e Shopee"
>
> Sistema monta um squad com:
> - **Pesquisador de mercado** — analisa concorrência e palavras-chave
> - **Criador de anúncios** — gera título SEO, descrição, fotos tratadas
> - **Precificador** — sugere preço com margem-alvo considerando frete
> - **Atendimento** — responde perguntas via WhatsApp + chat dos marketplaces
> - **Fiscal** — emite NF-e via Focus NFe quando vender
>
> Tudo com timeline visível e aprovação humana antes de publicar/responder/emitir.

---

## Inspiração: Accio Work

O [Accio Work](https://accio.works/) é o agent orchestrator enterprise da Alibaba International, lançado em março/2026. Em 4 meses atingiu 10M+ MAU globais. Leia a análise completa em [`docs/relatorio-accio-work.md`](docs/relatorio-accio-work.md).

**Conceitos centrais que estamos adaptando:**

| Conceito Accio | Adaptação BR |
|---|---|
| Goal-to-squad | Mesma ideia — usuário descreve, planner monta o time |
| Paralelismo + HITL | Mesmo — sandbox + aprovação humana p/ ações sensíveis |
| Skills reutilizáveis | Marketplace de skills BR em fase posterior |
| Grounding em dados Alibaba | Grounding em **APIs BR**: ML, Shopee, Focus NFe, Melhor Envio, Asaas |
| WhatsApp/Telegram | WhatsApp como canal principal (mercado BR) |

---

## Opções de MVP avaliadas

Foram estudadas 4 opções de recorte inicial. A recomendação é o caminho **D → C**.

| # | Opção | Resumo | Esforço MVP |
|---|---|---|---|
| A | [Clone fiel B2B sourcing](docs/opcoes-mvp.md#opção-a--clone-fiel-b2b-sourcing) | Importador BR encontra fornecedor e negocia | Alto (3–5 meses) |
| B | [Plataforma multi-agente genérica](docs/opcoes-mvp.md#opção-b--plataforma-multi-agente-genérica) | Framework horizontal tipo Manus/Lindy | Médio (2–3 meses) |
| C | [Vertical e-commerce BR](docs/opcoes-mvp.md#opção-c--vertical-próprio-e-commerce-br) ⭐ | Accio para lojista BR (ML+Shopee+WhatsApp+NF-e) | Médio-alto (2–4 meses) |
| D | [Só a camada UX/chat](docs/opcoes-mvp.md#opção-d--só-a-camada-uxchat) | UI completa + squad simulado | Baixo (2–4 sem) |

**Recomendação:** começar por **D** (valida UX, vira pitch, reaproveitado 100%) e evoluir para **C** (integrações reais com APIs BR). Justificativa completa em [`docs/recomendacao.md`](docs/recomendacao.md).

---

## Estrutura planejada do monorepo

```
accio-agents/
├── apps/
│   ├── web/              # Next.js 15 + React 19 + Tailwind + shadcn/ui
│   └── api/              # FastAPI + LangGraph
├── packages/
│   ├── shared/           # Tipos TS gerados a partir dos Pydantic schemas
│   └── ui/               # Design system compartilhado
├── workers/              # Celery/Arq p/ jobs assíncronos
├── docs/                 # Esta documentação
└── infra/                # IaC (Terraform / docker-compose local)
```

---

## Arquitetura em 1 tela

```
┌─────────────────────────────────────────────────────────┐
│  Front (Next.js) — Chat + Timeline + Approvals          │
└───────────────────────┬─────────────────────────────────┘
                        │ WebSocket / SSE
┌───────────────────────┴─────────────────────────────────┐
│  API Gateway (FastAPI) — Auth, Billing, Roteamento      │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────┴─────────────────────────────────┐
│  Orquestrador (LangGraph) — Planner, Registry, HITL     │
└────┬──────────────────┬──────────────────┬──────────────┘
     │                  │                  │
┌────┴────┐      ┌──────┴────┐      ┌──────┴──────┐
│ Workers │      │ Sandbox   │      │ Integrações │
│ (Arq)   │      │ (E2B)     │      │ WA, ML, NFe │
└─────────┘      └───────────┘      └─────────────┘

Dados: Postgres + pgvector + Redis + S3
LLM:   Claude Sonnet 4.6 (default) / Haiku 4.5 (triagem)
Obs:   Langfuse + Sentry
```

Detalhes em [`docs/arquitetura.md`](docs/arquitetura.md).

---

## Plano de ação resumido

- **Fase 0 (sem 1):** fundação — monorepo, CI, Supabase, observabilidade
- **Fase 1 (sem 2–4):** UX + orquestrador com squad genérico (Opção D)
- **Fase 2 (sem 5–12):** vertical e-commerce — agentes Criador, Atendimento, Precificador (Opção C)
- **Fase 3 (sem 13+):** Fiscal, Logística, Ads, marketplace de skills

Plano completo com tasks em [`docs/plano-acao.md`](docs/plano-acao.md).

---

## Principais riscos

| Risco | Mitigação |
|---|---|
| Custo de LLM explodir | Prompt caching + roteamento por complexidade (Haiku triagem → Sonnet raciocínio) |
| API WhatsApp oficial cara | Z-API/Evolution no MVP → Meta oficial quando houver receita |
| Mudança de API dos marketplaces | Camada de abstração + testes de contrato semanais |
| Diferenciação vs. Manus/Lindy | Fosso BR: fiscal, pagamento (PIX), canais e idioma locais |
| HITL virar gargalo | Auto-aprovação configurável por score de confiança do agente |

Detalhe em [`docs/riscos.md`](docs/riscos.md).

---

## Índice da documentação

- [`docs/relatorio-accio-work.md`](docs/relatorio-accio-work.md) — Análise do Accio Work (produto, arquitetura, métricas)
- [`docs/opcoes-mvp.md`](docs/opcoes-mvp.md) — Esboço detalhado das 4 opções de MVP
- [`docs/recomendacao.md`](docs/recomendacao.md) — Justificativa da rota D → C
- [`docs/arquitetura.md`](docs/arquitetura.md) — Arquitetura técnica de referência
- [`docs/plano-acao.md`](docs/plano-acao.md) — Plano faseado com checklist
- [`docs/riscos.md`](docs/riscos.md) — Matriz de riscos e mitigações
- [`docs/stack.md`](docs/stack.md) — Stack técnica recomendada

---

## Próximos passos

1. Confirmar rota **D → C** (ou escolher outra)
2. Inicializar monorepo + repositório git
3. Subir Supabase + Vercel + Railway
4. Escrever o grafo LangGraph inicial (Planner + 3 agentes genéricos)
5. Implementar tela de Chat + Timeline de squad com streaming
