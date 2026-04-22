# Recomendação Estratégica: Caminho D → C

> Justificativa da rota recomendada e descarte explícito das alternativas.

---

## Decisão

**Fase 1:** executar a **Opção D** (camada UX/chat com squad simulado) — semanas 1–4
**Fase 2:** migrar para a **Opção C** (vertical e-commerce BR) — semanas 5–12+

Abandonar as Opções A e B como produto principal. A Opção B pode reaparecer como **feature** dentro de C (marketplace de skills).

---

## Por que D antes de C

1. **Risco técnico concentrado no início.** O desafio mais difícil não é integrar com Mercado Livre — é construir uma UX de multi-agente que seja legível e confiável. Resolver isso primeiro, com dados simulados, impede que bugs de integração mascarem problemas de UX.

2. **Demo comercial em 4 semanas.** Mesmo sem integrações reais, um squad resolvendo tarefas abertas (relatórios, análises, geração de conteúdo) é demonstrável e impressiona. Serve para captar waitlist, feedback de potenciais clientes e conversas com investidores.

3. **Reaproveitamento total.** Cada linha de D vai para C. Não é protótipo jogado fora — é o esqueleto do produto final.

4. **Learning curve do LangGraph.** Construir com 3 agentes genéricos ensina o framework sem pressão de integração. Quando os agentes verticais entrarem, a equipe já domina o padrão.

5. **Custos baixos na fase exploratória.** Sem integrações pagas (WhatsApp oficial, Focus NFe, Meta Ads), o custo mensal fica quase só em LLM — viável em modo solo.

---

## Por que C (não A, não B)

### Opção C vence A (sourcing B2B)

| Critério | A | C |
|---|---|---|
| Dados proprietários necessários | Catálogo de fornecedores (não temos) | APIs públicas BR (todas disponíveis) |
| Concorrente direto | Alibaba (impossível bater) | ERPs locais (Bling, Tiny) — sem IA |
| Tamanho do nicho | Importadores BR (~50k) | Lojistas online BR (~1M no ML) |
| Tempo até receita | 6+ meses | 3–4 meses |
| Fosso defensável | Difícil | Fiscal BR + PIX + idioma |

### Opção C vence B (plataforma genérica)

| Critério | B | C |
|---|---|---|
| Competidores financiados | Manus, Lindy, Relay, Dust | Nenhum com esse recorte |
| Proposta de valor | "Faz qualquer coisa" (vago) | "Seu e-commerce no automático" (tangível) |
| Dor do cliente | Difusa | Aguda e paga (lojista perde venda sem atendimento) |
| Monetização | Freemium + upsell (longo) | SaaS por SKU ou por volume (direto) |

---

## O que descartamos e por quê

### Opção A — Clone fiel B2B sourcing
**Motivo do descarte:** sem o catálogo do Alibaba, viramos scraper de fontes públicas competindo com o dono da fonte. Nicho de importadores BR é menor e mais conservador que o de lojistas. ROI do tempo investido é ruim.

**Quando reconsiderar:** se aparecer parceria com uma plataforma B2B BR já consolidada que ceda catálogo.

### Opção B — Plataforma genérica pura
**Motivo do descarte:** mercado já com 4–5 competidores bem-financiados (Manus com rumor de $500M, Lindy com $50M+). Entrar sem diferenciação forte é queimar runway. Genéricas também têm churn alto — cliente não sabe o que fazer com "faça qualquer coisa".

**Quando reaparece:** como **feature** — o marketplace de skills dentro da Opção C dá aos usuários poder de criar automações genéricas, sem o produto inteiro depender disso.

---

## Premissas que, se mudarem, mudam a recomendação

- **"Solo ou time pequeno, budget modesto"** — se virar time de 10+ com US$ 2M, Opção B volta a fazer sentido
- **"Prazo flexível mas não infinito"** — se o prazo for 18+ meses, Opção A com parceria B2B vira opção
- **"Stack TS + Python"** — mantida
- **"Público SMB BR"** — mantida; se mirar enterprise, rota muda para local-first desktop

---

## Critérios de sucesso por fase

### Fim da Fase 1 (semana 4) — saída da D
- [ ] Chat + timeline + approvals funcionando com streaming real
- [ ] Squad de 3 agentes resolvendo 5 casos de uso abertos end-to-end
- [ ] Landing pública + waitlist com ≥100 inscritos
- [ ] Vídeo-demo de 2 minutos pronto para pitch

### Fim da Fase 2 (semana 12) — entrada em C
- [ ] OAuth ML + Shopee + WhatsApp funcionando
- [ ] Agente Criador publicando anúncios aprovados pelo usuário
- [ ] Agente Atendimento respondendo com taxa de auto-aprovação ≥ 50%
- [ ] 10 clientes beta pagantes (mesmo que preço promocional)
- [ ] Custo unitário de LLM < 20% da receita por cliente

Se qualquer um desses falhar ao final da fase, rever recomendação.
