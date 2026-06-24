# CRM-Lite — Captador de Leads

Inbox unificado de leads + funil de vendas simples. O **n8n** captura leads de formulários, e-mail e WhatsApp e os envia via webhook para este app. O app organiza tudo num quadro visual (kanban) onde você move o lead de etapa em etapa até fechar — ou perder.

> Projeto de estudo (Better-T-Stack, ~1h/dia) com potencial de monetização futura.

## Como funciona

```
n8n (formulário / e-mail / WhatsApp)
        │
        │  POST /api/leads  (webhook)
        ▼
   Express + tRPC  ──→  PostgreSQL (Supabase)
        │
        ▼
   Next.js  →  Funil kanban de leads
```

O n8n resolve a parte difícil (integrações, agendamento, scraping). Este app resolve a parte que ninguém entrega pronta: guardar, organizar, mostrar e reagir a cada lead.

## Stack

| Camada | Tecnologia |
|---|---|
| Frontend | Next.js + TailwindCSS + shadcn/ui |
| Backend | Express + tRPC |
| Banco | PostgreSQL via Prisma (Supabase local) |
| Auth | Better-Auth |
| Monorepo | Turborepo + pnpm workspaces |

## Estrutura

```
my-crm/
├── apps/
│   ├── web/       # Frontend Next.js (funil, views de leads)
│   └── server/    # Backend Express + tRPC + webhook endpoint
└── packages/
    ├── ui/        # Componentes shadcn/ui compartilhados
    ├── api/       # Routers tRPC
    ├── auth/      # Better-Auth
    ├── db/        # Schema Prisma + client
    ├── config/    # Config compartilhada
    └── env/       # Env vars tipadas com Zod
```

## Setup local

### 1. Instalar dependências

```bash
pnpm install
```

### 2. Banco de dados (Supabase local)

```bash
# Na pasta packages/db:
supabase init
supabase start
# Copie a DB URL do output e cole em apps/server/.env:
# DATABASE_URL="postgresql://..."
```

### 3. Aplicar schema e rodar

```bash
pnpm run db:push   # sincroniza o schema
pnpm run dev       # inicia web (3001) + server (3000)
```

- Web: [http://localhost:3001](http://localhost:3001)
- API: [http://localhost:3000](http://localhost:3000)

## Scripts disponíveis

| Script | O que faz |
|---|---|
| `pnpm run dev` | Inicia tudo em modo desenvolvimento |
| `pnpm run dev:web` | Só o frontend |
| `pnpm run dev:server` | Só o backend |
| `pnpm run build` | Build de produção |
| `pnpm run check-types` | Verifica tipos TS em todo o monorepo |
| `pnpm run db:push` | Aplica o schema ao banco |
| `pnpm run db:generate` | Gera o cliente Prisma |
| `pnpm run db:migrate` | Roda migrations |
| `pnpm run db:studio` | Abre o Prisma Studio |
| `pnpm run docker:build` | Build das imagens Docker |
| `pnpm run docker:up` | Sobe o stack Docker Compose |
| `pnpm run docker:down` | Para o stack |
| `pnpm run docker:logs` | Tail dos logs Docker |

## UI — componentes compartilhados

Primitivos shadcn/ui ficam em `packages/ui`. Para adicionar mais:

```bash
npx shadcn@latest add accordion dialog table -c packages/ui
```

Importe assim:

```tsx
import { Button } from "@my-crm/ui/components/button";
```

Tokens de design e estilos globais: `packages/ui/src/styles/globals.css`.

## Deploy

Docker Compose para o server (`docker-compose.yml`). Dockerfiles em `apps/*/Dockerfile`. Variáveis de ambiente lidas dos `.env` de cada app e sobrescritas no `docker-compose.yml` para networking entre containers.

## Contexto do projeto

Criado com [Better-T-Stack](https://github.com/AmanVarshney01/create-better-t-stack). Para detalhes de arquitetura e convenções de código, veja [CLAUDE.md](./CLAUDE.md).
