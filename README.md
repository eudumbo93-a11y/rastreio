# SaaS de Rastreio

Plataforma multi-tenant de **pos-venda para e-commerce**: ingestao de
pedidos via API, notificacoes automaticas (e-mail/SMS), pagina publica
de rastreio white-label e integracao com transportadoras europeias
(CTT, DPD, GLS, Correos, etc.) atraves de um padrao plugavel
(`CarrierProvider`).

Mercado-alvo inicial: **Portugal e Espanha**. Idiomas: PT-PT, EN, ES.

---

## Visao rapida

```
[Loja do seller] → POST /v1/orders → API NestJS → Fila (Redis) → Worker
                                                                   ├─ Notification (e-mail/SMS)
                                                                   └─ CarrierProvider (Mock | CTT | DPD | ...)
                                                                          ↓
                                                              PostgreSQL + S3
                                                                          ↓
                                          rastreio.{seller}.com  ← Next.js SSR (apps/tracking)
                                          painel.dominio.com     ← Next.js     (apps/web)
```

## Documentacao para o agente (Cursor)

Todo o plano detalhado vive em [`.cursor/rules/`](./.cursor/rules/) e e
carregado automaticamente em toda conversa com o Cursor:

| Arquivo                          | Conteudo                                  |
|----------------------------------|-------------------------------------------|
| `00-project-overview.mdc`        | Visao, escopo, principios, escopo proibido |
| `01-tech-stack.mdc`              | Stack oficial e estrutura do monorepo     |
| `02-architecture.mdc`            | Modulos, fluxo end-to-end, multi-tenancy  |
| `03-data-model.mdc`              | Schema Prisma completo                    |
| `04-carrier-provider.mdc`        | Padrao Mock -> Real (peca critica)        |
| `05-notifications.mdc`           | E-mail/SMS, templates, i18n               |
| `06-compliance-gdpr.mdc`         | GDPR, seguranca, auditoria                |
| `07-roadmap.mdc`                 | Fases de implementacao detalhadas         |

> Regra fundamental: o `MockCarrierProvider` e exclusivamente para
> desenvolvimento e testes. Ele e **bloqueado por design** em producao.
> Em producao usamos sempre integracao real com transportadora ou
> agregador (17Track, AfterShip).

## Stack

TypeScript em todo o monorepo. NestJS (API + Worker), Next.js 15 (Web +
Tracking), Prisma + PostgreSQL, BullMQ + Redis, Resend (e-mail),
MessageBird (SMS), Stripe (billing), Tailwind + shadcn/ui, Turborepo
+ pnpm.

## Como comecar (quando o codigo existir)

```bash
pnpm install
docker compose up -d        # Postgres + Redis
pnpm db:migrate
pnpm db:seed
pnpm dev                    # turbo run dev em todos os apps
```

## Roadmap resumido

1. Fundacao do monorepo
2. Banco e contratos (Prisma + Zod)
3. API publica minima (POST /v1/orders)
4. Workers e fila (BullMQ)
5. Notification Engine (e-mail/SMS, templates, i18n)
6. CarrierProvider Mock
7. Pagina publica de rastreio
8. Painel do seller
9. Billing (Stripe)
10. GDPR e auditoria
11. Hardening e observabilidade
12. Carriers reais (CTT, DPD, 17Track...)

Detalhes de cada fase em [`.cursor/rules/07-roadmap.mdc`](./.cursor/rules/07-roadmap.mdc).

## Licenca e termos

Uso interno. Sellers que usarem a plataforma aceitam DPA + politica
anti-fraude que **proibe** uso do mock provider em comunicacoes a
clientes finais reais.
