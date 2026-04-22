# Opções de MVP

> Quatro recortes possíveis para o primeiro release. Todos partem da mesma arquitetura base (ver [`arquitetura.md`](arquitetura.md)); o que muda é o vertical de dados, o conjunto de agentes e o nível de autonomia.

---

## Matriz comparativa

| Critério | A — Sourcing B2B | B — Plataforma genérica | C — Vertical e-commerce BR ⭐ | D — Só UX |
|---|---|---|---|---|
| **Premissa** | Importador acha fornecedor | "Faça X" genérico | Lojista BR vende online | UI com squad simulado |
| **Dados proprietários** | Alto (e não temos) | Nenhum | Médio (APIs públicas BR) | Nenhum |
| **Competição** | Alibaba direto | Manus, Lindy, Relay | Fraca no BR | — |
| **Ticket médio potencial** | Alto (comex) | Médio | Médio-alto (recorrência) | Zero |
| **Tempo até receita** | 6+ meses | 4+ meses | 3–4 meses | Não aplicável |
| **Esforço MVP** | Alto (3–5 meses) | Médio (2–3 meses) | Médio-alto (2–4 meses) | Baixo (2–4 sem) |
| **Reaproveita p/ outros?** | Baixo | Alto | Médio | 100% |
| **Risco principal** | Sem catálogo próprio | Comoditização | Mudança de API marketplace | Não monetiza sozinho |

---

## Opção A — Clone fiel B2B sourcing

### Premissa
Importador brasileiro descreve o produto desejado (ex: "500 garrafas térmicas personalizadas, 500ml, entrega em 60 dias") → agentes encontram fornecedores, disparam RFQ, conduzem negociação e comparam cotações.

### Squad de agentes

| Agente | Responsabilidade |
|---|---|
| Pesquisador | Buscar em Alibaba, Made-in-China, DHgate, parceiros BR (Simplo7, B2Brazil) |
| Analista de spec | Extrair especificações, normalizar unidades, validar viabilidade |
| Negociador | Conduzir conversa multi-turno por e-mail/WhatsApp, pedir amostras |
| Compliance aduaneiro | HS Code, tributação de importação, Siscomex, INMETRO |
| Financeiro | Câmbio, carta de crédito, condições de pagamento |

### Telas-chave
1. Briefing do produto (formulário + chat)
2. Shortlist de fornecedores com score explicável
3. Timeline de negociação por fornecedor
4. Comparador de cotações (preço FOB/CIF, MOQ, lead time)
5. Aprovação final do pedido

### Integrações
- E-mail (IMAP/SMTP) e WhatsApp Business
- APIs de HS Code / Receita Federal
- Conversão cambial (API do BCB)
- Plataformas B2B via scraping (sujeito a rate limit e ToS)

### Por que é difícil
- **Não temos o catálogo do Alibaba.** Dependência de dados públicos sujos ou scraping.
- Alibaba compete diretamente (e melhor) com dados próprios.
- Nicho de importador BR é pequeno e conservador.

### Quando faz sentido
Se você tem acesso a uma rede de importadores BR e quer atacar comex (Siscomex + câmbio + ICMS) como diferencial local. Não é o caso atual.

---

## Opção B — Plataforma multi-agente genérica

### Premissa
Usuário descreve qualquer objetivo de negócio e o sistema monta o squad. O produto é o **framework de orquestração + UX + skills**, não o vertical.

### Squad de agentes (base)

| Agente | Responsabilidade |
|---|---|
| Planner | Quebrar objetivo em tarefas, decidir quem convocar |
| Researcher | Web search + RAG sobre documentos do usuário |
| Writer | Produzir relatórios, e-mails, copy |
| Coder | Scripts, automações, dashboards via sandbox |
| Data Analyst | Query em planilhas/BD, geração de gráficos |
| Communicator | Envio via e-mail, Slack, WhatsApp |

### Telas-chave
1. Chat de objetivo com streaming
2. Timeline do squad com agentes em paralelo
3. Editor de Skills (prompt + tools + YAML)
4. Marketplace de skills
5. Approval inbox

### Integrações
- **MCP (Model Context Protocol)** como porta de entrada universal para tools
- Conectores populares: Gmail, Slack, Notion, Google Drive, GitHub

### Por que é viável tecnicamente
LangGraph + MCP resolvem 80% do trabalho de orquestração. O difícil é a **UX** (mostrar agentes em paralelo de forma legível) e o **marketplace** (distribuição e moderação).

### Por que é arriscado comercialmente
Competidores bem-financiados já existem:
- [Manus](https://manus.im) — chinês, $500M+ funding rumor
- [Lindy](https://lindy.ai) — US, $50M+ Series A
- [Relay.app](https://relay.app) — US, focado em automação
- [Dust.tt](https://dust.tt) — francês, enterprise

Ganhar nesse mercado exige ou preço radicalmente menor, ou vertical muito específico, ou UX 10x melhor.

### Quando faz sentido
Como **feature** da plataforma (marketplace de skills dentro da Opção C), não como produto principal.

---

## Opção C — Vertical próprio (e-commerce BR) ⭐

### Premissa
Accio para o **lojista brasileiro**. Usuário diz "quero vender {produto} no Mercado Livre + Shopee + loja própria" e o squad cuida de tudo: pesquisa de mercado, criação de anúncios, precificação, atendimento multi-canal, fiscal, logística.

### Squad de agentes

| Agente | Responsabilidade | APIs/fontes |
|---|---|---|
| Pesquisa de mercado | Analisar concorrência, keywords, sazonalidade | Nubimetrics, ML Trends, Shopee Analytics |
| Criador de anúncios | Título SEO, descrição, fotos tratadas, categoria, atributos | ML Developers API, Shopee Open Platform |
| Precificador | Preço competitivo respeitando margem + frete | Dados dos agentes anteriores |
| Atendimento | Responder perguntas ML + chat Shopee + WhatsApp | APIs dos marketplaces + Meta WhatsApp |
| Fiscal | Emitir NF-e na venda, regime tributário correto | Focus NFe, NFe.io, Tecnospeed |
| Logística | Etiqueta, tracking, devoluções | Melhor Envio, Frete Rápido, Correios |
| Ads (fase 2) | Campanhas Meta/Google com budget otimizado | Meta Marketing API, Google Ads API |

### Telas-chave
1. **Onboarding:** "o que você quer vender" + conexão OAuth dos marketplaces
2. **Studio de produto:** squad monta listagens em minutos, usuário aprova e publica
3. **Calendário de campanhas:** visão de promoções e ads agendados
4. **Caixa unificada:** WhatsApp + ML + Shopee no mesmo inbox, respostas sugeridas
5. **Dashboard de vendas:** GMV, margem real (após taxas + frete + impostos), top SKUs
6. **Fila de aprovações:** publicações, respostas difíceis, emissões de NF, ajustes de preço

### Integrações (APIs públicas, todas documentadas)
- [Mercado Livre Developers](https://developers.mercadolivre.com.br/)
- [Shopee Open Platform](https://open.shopee.com/)
- [Meta WhatsApp Business API](https://developers.facebook.com/docs/whatsapp)
- [Focus NFe](https://focusnfe.com.br/) — emissão de NF-e
- [Melhor Envio](https://melhorenvio.com.br/) — etiquetas e rastreio
- [Asaas](https://www.asaas.com/) — PIX e cobrança recorrente

### Por que ganha
- **Dor real e massiva:** 1M+ lojistas ativos só no ML
- **APIs prontas e acessíveis** — não depende de scraping
- **Competição local fraca:** ferramentas atuais (Bling, Tiny, ENP) são ERPs, não agentes autônomos
- **Pricing em R$** adequado ao caixa de PME
- **Compliance fiscal BR** é barreira natural para concorrentes gringos

### Risco principal
Mudança de API dos marketplaces — mitigado com camada de abstração + testes de contrato semanais.

### Quando faz sentido
**Agora.** É a recomendação principal deste projeto.

---

## Opção D — Só a camada UX/chat

### Premissa
Construir a **interface completa** de orquestração (chat + timeline + approvals + skills) com um squad simulado por trás (LangGraph com 3 agentes genéricos respondendo sobre tarefas abertas), sem integrações reais ainda.

### O que é entregue
- UX completa e polida, com streaming real e paralelismo visível
- Squad mínimo funcionando (Planner + Researcher + Writer)
- Casos de uso que funcionam de verdade: "escreva um relatório sobre X", "analise esses PDFs", "gere uma apresentação"
- Zero integrações com marketplaces/WhatsApp/NF-e

### Por que faz sentido como fase 1
1. **Valida UX** — mostrar agentes em paralelo é o ponto mais difícil e diferenciador
2. **Gera deck de venda** — demo ao vivo é o melhor material comercial
3. **100% reaproveitado** — tudo vai ser usado na fase C
4. **Risco técnico isolado** — resolve orquestração antes de misturar com integrações instáveis
5. **Prazo curto** — 2–4 semanas solo

### Por que NÃO é produto final
UX sem integração real não monetiza. Usuário pagante precisa de automação que economiza tempo/dinheiro em processos reais — isso vem na Opção C.

### Quando faz sentido
**Como ponte obrigatória para a Opção C.** Rodar D antes de C reduz risco e acelera o tempo-até-demo.
