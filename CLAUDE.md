# CRM-Lite / Captador de Leads — CLAUDE.md

## O que é este projeto

Inbox unificado de leads + funil de vendas simples. O n8n captura leads de múltiplas fontes (formulário web, e-mail, WhatsApp) e os envia para este app via webhook. O app organiza, exibe e permite gerenciar o ciclo de vida de cada lead num quadro visual (kanban de funil).

Projeto de estudo (ritmo ~1h/dia) com potencial de monetização futura. Stack: Better-T-Stack (Next.js + Express + tRPC + Prisma + PostgreSQL).

## Arquitetura

```
n8n (ingestão)  →  POST /api/leads  →  PostgreSQL
                                           ↓
                                    Next.js (kanban/funil)
```

- **n8n** resolve integrações (WhatsApp, e-mail, formulários, scraping). Este app não integra diretamente com essas fontes.
- **Express + tRPC** expõe a API consumida pelo frontend e recebe webhooks do n8n.
- **Next.js** exibe o funil e permite mover leads entre etapas.
- **Supabase** (local via Docker) como banco PostgreSQL em desenvolvimento.

## Estrutura do monorepo

```
my-crm/
├── apps/
│   ├── web/       # Frontend Next.js — funil kanban, views de leads
│   └── server/    # Backend Express + tRPC — API REST + webhook endpoint
├── packages/
│   ├── ui/        # Componentes shadcn/ui compartilhados
│   ├── api/       # Definições tRPC (routers, procedures)
│   ├── auth/      # Better-Auth config
│   ├── db/        # Schema Prisma + client
│   ├── config/    # Configurações compartilhadas
│   └── env/       # Variáveis de ambiente tipadas (Zod)
```

## Modelo de dados principal

Um **Lead** tem:
- Fonte (whatsapp | email | form | manual)
- Dados de contato (nome, telefone, e-mail)
- Etapa do funil (ex.: novo → qualificado → proposta → fechado | perdido)
- Metadados do n8n (payload bruto, timestamp de ingestão)
- Notas/histórico de interações

## Comandos essenciais

```bash
pnpm install          # instalar dependências
pnpm run dev          # rodar tudo (web + server)
pnpm run dev:web      # só frontend (porta 3001)
pnpm run dev:server   # só backend (porta 3000)
pnpm run db:push      # aplicar schema ao banco
pnpm run db:studio    # Prisma Studio (UI do banco)
pnpm run check-types  # verificar tipos TS em todo o monorepo
```

## Convenções

- Usar tRPC para comunicação web → server; REST simples (`POST /api/leads`) para o webhook do n8n.
- Variáveis de ambiente validadas com Zod em `packages/env`.
- Componentes de UI em `packages/ui` (compartilhados); componentes de página em `apps/web/app` e `apps/web/components`.
- Sem comentários desnecessários — nomes descritivos bastam.

## Fluxo de desenvolvimento típico

1. Definir/alterar schema em `packages/db/prisma/schema/`
2. `pnpm run db:push` para sincronizar
3. Adicionar/ajustar procedure tRPC em `packages/api/`
4. Consumir no componente Next.js via hook tRPC

## O que este app NÃO faz

- Não integra diretamente com WhatsApp/e-mail/formulários — isso é responsabilidade do n8n.
- Não tem automação de envio de mensagens (por enquanto).
- Não é um CRM completo — foco no funil de leads, não em gestão de clientes.
