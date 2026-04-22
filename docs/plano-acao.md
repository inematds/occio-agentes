# Plano de Ação Faseado

> Checklist executável do zero até o beta pago. Cada fase tem um critério de saída claro — se falhar, volta para a anterior ou revisa a recomendação.

---

## Fase 0 — Fundação (semana 1)

**Objetivo:** tudo que não é produto, mas destrava o produto.

- [ ] Decisão final: rota **D → C** confirmada? (ou escolher outra)
- [ ] Criar repositório Git (`accio-agents`) no GitHub, branch `main` protegida
- [ ] Monorepo com Turborepo
  - `apps/web` — Next.js 15 + TS strict
  - `apps/api` — FastAPI + Python 3.12 + uv como package manager
  - `packages/shared` — tipos TS gerados do Pydantic via `datamodel-code-generator`
- [ ] Provisionar contas
  - [ ] Supabase (projeto `accio-agents-dev`)
  - [ ] Vercel (projeto conectado ao repo)
  - [ ] Railway (API + Workers)
  - [ ] Langfuse (cloud free ou self-host)
  - [ ] Sentry
  - [ ] Anthropic Console (para chave Claude)
- [ ] CI com GitHub Actions: lint + typecheck + testes em PR
- [ ] Docker compose local com Postgres + Redis + API + Web
- [ ] README + docs iniciais (este conjunto) já versionados
- [ ] Domínio registrado (ex: `accio-agents.com.br`)

**Critério de saída:** `docker-compose up` sobe tudo; deploy em staging funciona; primeiro commit na `main` com "hello world" passando no CI.

---

## Fase 1 — Opção D: UX + orquestrador fake (semanas 2–4)

**Objetivo:** produto navegável com squad simulado. Validação de UX e geração de demo.

### Semana 2 — esqueleto do produto

- [ ] Auth via Supabase (e-mail + magic link)
- [ ] Modelo de dados inicial: `users`, `runs`, `messages`, `agent_events`, `approvals`
- [ ] Endpoint `POST /runs` que inicia uma run e retorna ID
- [ ] WebSocket `GET /runs/:id/stream` com eventos mockados hardcoded
- [ ] Tela de Chat básica com streaming de texto

### Semana 3 — LangGraph real com 3 agentes genéricos

- [ ] Grafo LangGraph: `Planner → fan-out → [Researcher, Writer, DataAnalyst] → Aggregator`
- [ ] Checkpointer Postgres configurado (runs sobrevivem a restart)
- [ ] Tool `web_search` (Tavily ou Serper)
- [ ] Tool `file_read` para PDFs/planilhas enviados
- [ ] 3 casos de uso end-to-end funcionando:
  - "Escreva um relatório sobre {tema} baseado em {arquivos}"
  - "Analise essa planilha e gere 3 insights com gráficos"
  - "Pesquise 5 concorrentes de {produto} e resuma posicionamento"

### Semana 4 — polimento e demo

- [ ] **Timeline do squad** com animação de agentes em paralelo (Framer Motion)
- [ ] **Approval inbox** com 1 caso que dispara aprovação (enviar e-mail)
- [ ] Landing page de waitlist
- [ ] Vídeo-demo de 2 minutos gravado (Loom/ScreenStudio)
- [ ] Deploy público em `app.accio-agents.com.br`

**Critério de saída:**
- 3 casos de uso funcionam end-to-end com streaming real
- Vídeo-demo pronto
- ≥100 inscritos na waitlist (divulgação em LinkedIn + comunidades de e-commerce BR)

---

## Fase 2 — Opção C: vertical e-commerce BR (semanas 5–12)

**Objetivo:** produto que um lojista paga. Dois agentes de alto ROI funcionando, cobrança recorrente ativa.

### Semanas 5–6 — integrações base

- [ ] OAuth Mercado Livre (cadastrar aplicativo, fluxo completo)
- [ ] OAuth Shopee
- [ ] Integração WhatsApp via Z-API ou Evolution API (mais barato no MVP)
- [ ] Adapter pattern implementado (`MarketplaceAdapter`) para ML e Shopee
- [ ] Sincronização bidirecional: produtos + perguntas + pedidos

### Semanas 7–8 — Agente Criador de Anúncios

- [ ] Prompt + tool set do agente Criador
- [ ] Fluxo: usuário envia foto + descrição curta → Criador gera título SEO, bullets, categoria, atributos
- [ ] Approval obrigatório antes de publicar
- [ ] Publicação real em ML e Shopee
- [ ] Edição assistida: usuário pede ajustes, agente revisa

### Semanas 9–10 — Agente Atendimento

- [ ] Inbox unificado: ML + Shopee + WhatsApp
- [ ] Agente responde com base em: descrição do produto + histórico do cliente + FAQ definido pelo lojista
- [ ] Política de auto-aprovação configurável por confiança
- [ ] Métrica de taxa de aprovação automática visível para o lojista

### Semanas 11–12 — Agente Precificador + monetização

- [ ] Agente Precificador: monitora concorrência diariamente, sugere ajuste com rationale
- [ ] Approval para mudança de preço acima de X%
- [ ] Billing via Stripe (ou Asaas para PIX recorrente)
- [ ] 3 planos: Starter (até 50 SKUs), Pro (até 500), Scale (ilimitado)
- [ ] Dashboard de custo de LLM por cliente (para monitorar unit economics)
- [ ] Recrutamento de 10 beta pagantes (mesmo que preço promocional)

**Critério de saída:**
- Agente Criador publicando ≥20 anúncios/dia em produção
- Agente Atendimento com ≥50% de taxa de auto-aprovação
- 10 clientes beta pagantes (MRR > R$ 0)
- Custo de LLM < 20% da receita por cliente

---

## Fase 3 — Escala e diferenciais (semanas 13+)

**Objetivo:** ir de beta para produto sério; abrir frentes de crescimento.

### Expansão de agentes

- [ ] Agente **Fiscal**: emissão de NF-e via Focus NFe, regime tributário correto
- [ ] Agente **Logística**: Melhor Envio — etiqueta, tracking, devolução
- [ ] Agente **Ads**: campanhas Meta/Google com budget e público otimizados
- [ ] Agente **Pesquisa de Mercado**: Nubimetrics + tendências de busca
- [ ] Agente **Finanças**: conciliação (vendas × taxas × impostos) e fluxo de caixa

### Marketplace de Skills (feature da Opção B dentro de C)

- [ ] Editor de skills (YAML + prompt + tools)
- [ ] Compartilhamento entre usuários do mesmo tenant
- [ ] Marketplace público com skills monetizáveis (split de receita com autor)
- [ ] Moderação de skills (review manual inicial)

### Infra e confiabilidade

- [ ] Migrar API para ambiente mais robusto (AWS/GCP com IaC)
- [ ] Testes de contrato semanais contra sandboxes de ML/Shopee
- [ ] WhatsApp oficial (Meta) em paralelo com Z-API
- [ ] SLA 99.5% publicado
- [ ] SOC 2 Type I (quando houver demanda enterprise)

### Crescimento

- [ ] Programa de parceiros (agências de e-commerce, contadores, hubs de marketplace)
- [ ] Integração com Bling/Tiny (ERPs existentes) como fonte de dados
- [ ] Mobile app (PWA primeiro, React Native se justificar)
- [ ] Internacionalização para AR/MX (mesmos marketplaces, fiscal diferente)

---

## Riscos que podem matar o plano

| Risco | Sinal de alerta | Ação |
|---|---|---|
| ML ou Shopee bloqueia nossa app | Rate limit ou suspensão | Ter contas de backup + modo read-only até resolver |
| Custo de LLM > receita | Unit economics negativo 2 meses seguidos | Reduzir contexto, mover mais coisa para Haiku, aumentar preço |
| WhatsApp Z-API instável | Mensagens perdidas > 1% | Acelerar migração para Meta oficial |
| Churn alto no beta | < 70% retenção mês 2 | Parar expansão, fazer 20 entrevistas de churn |
| Concorrente lança equivalente | Produto similar em PT-BR com funding | Dobrar em diferenciação local (fiscal + idioma + parcerias) |

---

## Resumo em uma imagem

```
Semana:  1    2────4         5────────────12           13──────►
         ├────┼───────────────┼─────────────────────────────────►
Fase:    0    1 (Opção D)     2 (Opção C)              3 (Escala)
         │    │ UX + squad    │ ML + Shopee + WA        │ Fiscal + Logística
         │    │ genérico      │ Criador + Atendimento   │ Ads + Skills market
         │    │ Demo + wait   │ Precificador + billing  │ IaC + parcerias
         │    │               │ 10 beta pagantes        │
         ▼    ▼               ▼                         ▼
       Setup Demo ao vivo    Receita primeira         Produto sério
```
