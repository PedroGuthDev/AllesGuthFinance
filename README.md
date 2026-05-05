# AllesGuth Finance

Uma SaaS full-stack de finanças pessoais construída com Next.js. Abra o painel e entenda em segundos para onde foi cada real (ou dólar) — e se você está no caminho certo.

> **Status:** v1.0 MVP — 100% concluído.

---

## Funcionalidades

### Núcleo
- **Gestão de múltiplas contas** — conta corrente, poupança, cartões de crédito, carteira e investimentos com atualização de saldo em tempo real
- **Lançamentos manuais** — receitas, despesas e transferências entre contas com exclusão suave e trilha de auditoria
- **Importação em massa** — extratos bancários CSV e OFX/QFX com detecção automática de colunas, etapa de revisão editável e deduplicação por hash de conteúdo + FITID
- **Categorização inteligente** — categorias personalizáveis com regras automáticas (padrão → categoria), aplicadas na importação; 201 regras integradas para bancos brasileiros e americanos.**As categorias preenchidas manualmente são salvas para um futuro import vir preenchido.**
- **Exportação** — exportação em CSV com filtros por conta, período e natureza da transação

### Painel & Insights
- **Filtro de período** — semana atual, mês, mês anterior, ano ou intervalo de datas personalizado
- **Métricas principais** — fluxo de caixa do período, total de receitas, total de despesas, patrimônio líquido
- **Distribuição de gastos** — gráfico de rosca por categoria (top 6 + Outros)
- **Comparativo mês a mês** — mês atual vs. mês anterior por categoria com coloração de variação Δ
- **Projeção de burn rate** — taxa diária de gastos e “nesse ritmo você vai atingir seu limite em X dias”
- **Linha do tempo de gastos** — gráfico de linha por conta (diário ≤40 dias, mensal >40 dias)
- **Evolução mensal** — gráfico de barras agrupado de receitas vs. despesas
- **Visão por categoria** — alternância entre despesas e receitas em um único cartão
- **Vencimento de cartão** — contas de cartão de crédito a vencer por ciclo de fechamento / vencimento
- **Filtros por cartão** — cada cartão de gráfico tem seu próprio filtro via ícone de popover

### Recursos Inteligentes
- **Detecção de recorrência** — identifica assinaturas e cobranças recorrentes automaticamente, mostra próxima data prevista e envia alerta com 3 dias de antecedência
- **Simulador de “e se”** — modele uma redução de gasto mensal em qualquer categoria e veja a economia projetada em 6, 12 e 24 meses com impacto nas metas
- **Pontuação de saúde financeira** — score de 0–100 com base em aderência ao orçamento, taxa de poupança, relação dívida/renda e progresso de metas com delta semanal
- **Relatório de personalidade de gastos** — perfil comportamental mensal (dias de pico, maiores estabelecimentos, arquétipo de gastos) exportável como cartão de imagem compartilhável

### Orçamentos & Metas
- **Orçamentos mensais** — limites de gasto por categoria com barras de progresso e alertas personalizados.
- **Metas de economia** — valor alvo, prazo, descrição e conta vinculada opcional para acompanhamento automático do progresso.

### SaaS & Cobrança
- **Assinaturas Stripe** — plano pessoal com preços em BRL e USD e teste gratuito de 14 dias
- **Complemento familiar** — cobrança adicional para compartilhamento de workspace com controle de permissão (visualizador / editor)
- **Webhook-driven** — atualizações de status de assinatura em tempo real via webhooks do Stripe

### Autenticação & Segurança
- Cadastro e login por e-mail/senha
- Google OAuth
- Redefinição de senha por e-mail
- Row-Level Security (RLS) aplicada no banco de dados — usuários só veem seus próprios dados
- Verificações de autenticação em todos os handlers (não apenas no middleware)

### UX
- **i18n** — alternância PT-BR / EN-US com tradução completa da interface, troca de símbolo de moeda (R$ / $) e persistência de localidade
- **Design dark-first** — tema Cockpit × Glow com Radix UI e Tailwind CSS
- **Sidebar recolhível** — somente ícone (88 px) ou expandida (200 px), estado persistido no localStorage
- **Vercel Analytics + Speed Insights** — observabilidade em produção. Score 95+ em Real User Experience.

### Suporte a CSV/OFX e PDF bancário
Bradesco, Banco do Brasil, Itaú, Inter, XP, C6 Bank, Nubank, Santander, Bank of America, US Bank, Truist, TD Bank, Sicredi
---

## Stack Tecnológica

| Camada | Tecnologia |
|---|---|
| Framework | Next.js 15 (App Router) |
| UI | React, Radix UI, Tailwind CSS, Recharts |
| Estado | Zustand, TanStack Query |
| Formulários | React Hook Form + Zod |
| Auth | Auth.js v5 (NextAuth) |
| Banco de dados | Neon PostgreSQL (serverless) |
| ORM | Drizzle ORM |
| E-mail | Resend + React Email |
| Cobrança | Stripe |
| Testes | Vitest, Playwright |
| Deploy | Vercel |

---

## Visão Geral da Arquitetura

```
src/
├── app/
│   ├── (auth)/          # Login, cadastro, redefinição de senha
│   ├── (app)/           # Shell da aplicação autenticada
│   ├── (dashboard)/     # Painel e insights
│   ├── (billing)/       # Páginas de cobrança e assinatura
│   └── api/             # Webhooks (Stripe, NFe.io) e endpoints de importação
├── components/          # Componentes React reutilizáveis
├── db/                  # Schema Drizzle, migrations, scripts de RLS, seed
├── lib/
│   ├── services/        # Acesso a dados server-side (sempre via withUserContext)
│   └── parsers/         # Parsers PDF, CSV e OFX/SGML
└── types/               # Tipos TypeScript compartilhados
```

Decisões-chave de design:
- **`NUMERIC(15,2)` em todas as colunas monetárias** — evita erros de arredondamento de ponto flutuante
- **`withUserContext(userId, fn)` para cada chamada ao banco** — isolamento de dados via RLS, evita IDOR
- **`currentBalance` em cache na tabela de contas** — leituras de saldo em O(1) críticas para desempenho do painel
- **Transações de crédito não alteram `currentBalance`** — saldo do cartão de crédito é calculado em tempo de execução a partir do ciclo de faturamento

---

## Roteiro

| Fase | Funcionalidade | Status |
|---|---|---|
| 1 | Fundação (auth, RLS, workspace) | ✅ Concluído |
| 2 | Entrada de dados principal (contas, categorias, transações) | ✅ Concluído |
| 3 | Painel & insights | ✅ Concluído |
| 4 | Pipeline de importação (CSV/OFX) | ✅ Concluído |
| 5 | Orçamentos & metas | ✅ Concluído |
| 6 | SaaS & monetização (cobrança Stripe) | ✅ Concluído |
| 7 | Detecção de recorrência inteligente | ✅ Concluído |
| 8 | Simulador de “e se” | ✅ Concluído |
| 9 | Pontuação de saúde financeira | ✅ Concluído |
| 10 | Relatório de personalidade de gastos | ✅ Concluído |
| 11 | Desafios colaborativos (plano familiar) | ✅ Concluído |
| 12 | i18n & moeda (PT/EN, R$/$) | ✅ Concluído|

---

## Licença

MIT
