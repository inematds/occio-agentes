# Matriz de Riscos

> Riscos relevantes com probabilidade, impacto e mitigação. Revisar ao final de cada fase.

---

## Escala

- **Probabilidade:** Baixa / Média / Alta
- **Impacto:** 1 (ruído) → 5 (mata o projeto)
- **Prioridade = Prob × Impacto**

---

## Riscos técnicos

### R1 — Custo de LLM explode
- **Prob:** Alta • **Imp:** 4 • **Prio:** Alta
- **Cenário:** agentes ficam em loop, contexto cresce sem controle, ou volume supera previsão
- **Mitigação:**
  - Prompt caching (Claude) obrigatório desde dia 1
  - Orçamento por run com kill-switch (ex: R$ 2/run default)
  - Roteamento por complexidade — Haiku para triagem, Sonnet só quando precisa de raciocínio
  - Dashboard de custo por cliente desde o começo (não depois)
  - Compactação automática de contexto em runs longas

### R2 — API de marketplace muda sem aviso
- **Prob:** Média • **Imp:** 4 • **Prio:** Alta
- **Cenário:** ML ou Shopee altera schema, nossas publicações falham em produção
- **Mitigação:**
  - Adapter pattern — troca isolada por marketplace
  - Testes de contrato semanais contra sandbox
  - Dead letter queue para operações falhadas + retry com backoff
  - Canal no Slack com alertas do webhook oficial de cada plataforma

### R3 — WhatsApp não-oficial (Z-API) é derrubado
- **Prob:** Média • **Imp:** 3 • **Prio:** Média
- **Cenário:** Meta bloqueia conta usada via Z-API/Evolution
- **Mitigação:**
  - Migrar para WhatsApp Business API oficial assim que houver receita
  - Rodar os dois em paralelo na transição
  - Nunca usar WhatsApp pessoal do cliente — sempre número dedicado

### R4 — Checkpointer LangGraph corrompe estado
- **Prob:** Baixa • **Imp:** 4 • **Prio:** Média
- **Cenário:** run fica "presa" em estado inconsistente após deploy
- **Mitigação:**
  - Versionamento de schema de estado desde dia 1
  - Comando admin `kill_run(id)` para casos patológicos
  - Backup do Postgres automático (Supabase faz por padrão)

### R5 — Sandbox E2B fica lento/instável
- **Prob:** Baixa • **Imp:** 2 • **Prio:** Baixa
- **Cenário:** execuções de código demoram > 30s ou falham
- **Mitigação:**
  - Usar sandbox só para casos realmente necessários
  - Ter Daytona como alternativa pré-integrada
  - Timeout agressivo + fallback para resposta sem sandbox

---

## Riscos de produto

### R6 — UX de multi-agente é confusa
- **Prob:** Média • **Imp:** 5 • **Prio:** Crítica
- **Cenário:** usuário não entende o que cada agente faz, perde confiança
- **Mitigação:**
  - Fase 1 (Opção D) inteira dedicada a validar UX antes de integrar
  - 5 sessões de usabilidade por semana durante fase 1
  - Cada agente tem **nome + persona + uma frase do que faz** sempre visível
  - Ações sensíveis com diff claro do "antes/depois" na aprovação

### R7 — Aprovação humana vira gargalo
- **Prob:** Alta • **Imp:** 4 • **Prio:** Alta
- **Cenário:** lojista abandona porque tem 200 aprovações pendentes
- **Mitigação:**
  - Política de auto-aprovação configurável por categoria
  - Agrupamento de aprovações similares (bulk approve)
  - Score de confiança visível e calibrado com feedback
  - Meta: ≥ 50% de auto-aprovação em atendimento até fim da Fase 2

### R8 — Agente erra em produção (anúncio errado, resposta rude)
- **Prob:** Alta • **Imp:** 3 • **Prio:** Alta
- **Cenário:** anúncio publicado com preço errado, cliente recebe resposta inadequada
- **Mitigação:**
  - Todas as ações externas passam por approval no MVP (sem exceção)
  - Auto-aprovação só após histórico consistente do cliente
  - Undo de 1-clique em toda ação externa (ex: despublicar anúncio)
  - Log imutável para auditoria

### R9 — Feature creep — escopo vira Opção B sem querer
- **Prob:** Alta • **Imp:** 4 • **Prio:** Alta
- **Cenário:** cada cliente pede 1 feature; viramos plataforma genérica sem foco
- **Mitigação:**
  - "Se não serve lojista BR de e-commerce, vai para backlog de Fase 3"
  - Roadmap público para cliente saber o que esperar
  - Marketplace de Skills resolve demanda custom sem desviar o core

---

## Riscos comerciais

### R10 — Competidor gringo lança versão PT-BR
- **Prob:** Média • **Imp:** 5 • **Prio:** Alta
- **Cenário:** Lindy ou Manus traduzem UI e entram no BR
- **Mitigação:**
  - Fosso: integração fiscal BR (NF-e, regime tributário) — barreira real de 6+ meses
  - Fosso: canais locais (PIX, WhatsApp como primary, Mercado Pago)
  - Parcerias com ERPs/contadores BR = distribuição exclusiva
  - Brand local + comunidade de lojistas

### R11 — Churn alto no beta
- **Prob:** Média • **Imp:** 4 • **Prio:** Alta
- **Cenário:** < 70% de retenção no mês 2
- **Sinais:** cliente para de aprovar ações, sessões caem, cancelamento
- **Mitigação:**
  - 20 entrevistas de churn antes de qualquer nova feature
  - Onboarding guiado com sucesso mensurável na primeira semana
  - CS ativo para os 10 primeiros clientes (ligar, não esperar ticket)

### R12 — Unit economics negativo
- **Prob:** Média • **Imp:** 5 • **Prio:** Alta
- **Cenário:** custo de LLM + infra > ARPU, empresa queima caixa
- **Mitigação:**
  - Alvo: custo variável < 30% da receita
  - Preço mínimo calibrado por cohort (não descontar MRP no MVP)
  - Limite de uso por plano com upsell transparente
  - Ver R1 (custo de LLM)

### R13 — Lei de IA brasileira / regulamentação
- **Prob:** Média • **Imp:** 3 • **Prio:** Média
- **Cenário:** PL 2338/2023 ou sucessor exige certificação, auditoria ou opt-in explícito
- **Mitigação:**
  - Log de decisões do agente + rastreabilidade desde dia 1
  - Termos de uso claros sobre agente de IA respondendo em nome do lojista
  - Opt-in explícito para cada canal (WhatsApp, e-mail)
  - Monitorar projetos em tramitação mensalmente

### R14 — LGPD: dados de cliente final expostos
- **Prob:** Baixa • **Imp:** 5 • **Prio:** Alta
- **Cenário:** agente loga PII de comprador do lojista em prompt, vaza por bug
- **Mitigação:**
  - PII nunca vai para system prompt; só nas mensagens do turno corrente
  - Mascaramento automático (CPF, e-mail, telefone) antes de persistir em logs
  - Data Processing Agreement com OpenAI/Anthropic ativos
  - DPO designado (mesmo que seja você) desde cedo

---

## Riscos de execução

### R15 — Solo/time pequeno não sustenta o ritmo
- **Prob:** Média • **Imp:** 5 • **Prio:** Crítica
- **Cenário:** dois lados pesados (front + backend + agentes) travam progresso
- **Mitigação:**
  - Fase 1 é desenhada para ser executável solo em 4 semanas
  - Contratação de 1 dev full-stack só após 10 beta pagantes (Fase 2)
  - Skills editor e marketplace vêm só na Fase 3 — não aceleram receita

### R16 — Dependência de um fornecedor LLM
- **Prob:** Baixa • **Imp:** 3 • **Prio:** Baixa
- **Cenário:** Anthropic muda preço drasticamente ou suspende conta
- **Mitigação:**
  - Abstração sobre provedores (Claude + OpenAI + Gemini) desde dia 1
  - Contratos com compromisso mínimo só após previsibilidade de volume
  - Prompt caching próprio dentro do nosso cache (independente de provedor)

---

## Matriz de prioridade (top 10)

| Prio | Risco | Fase de maior exposição |
|---|---|---|
| Crítica | R6 — UX multi-agente confusa | Fase 1 |
| Crítica | R15 — Time pequeno não sustenta | Todas |
| Alta | R1 — Custo LLM explode | Fase 2+ |
| Alta | R2 — Mudança de API marketplace | Fase 2+ |
| Alta | R7 — Aprovação vira gargalo | Fase 2 |
| Alta | R8 — Agente erra em produção | Fase 2 |
| Alta | R9 — Feature creep | Fase 2 |
| Alta | R10 — Competidor PT-BR | Fase 3 |
| Alta | R11 — Churn alto | Fase 2 |
| Alta | R12 — Unit economics negativo | Fase 2 |

---

## Ritual de revisão

Ao final de cada fase:
1. Reavaliar probabilidade e impacto de cada risco
2. Abrir risco novo descoberto no período
3. Fechar risco que virou não-risco
4. Confirmar se mitigações previstas foram implementadas
