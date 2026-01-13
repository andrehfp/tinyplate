# CLAUDE.md

Este arquivo fornece instruções ao Claude Code quando trabalhando neste repositório.

## Instruções Gerais

- Sempre use `bun` em vez de `npm`
- Execute testes antes de commit: `bun run test && bun run lint`
- Prefira TypeScript sobre JavaScript

## Skills Disponíveis

| Skill | Comando | Descrição |
|-------|---------|-----------|
| PostHog | `/posthog` | Analytics, feature flags, session replay |
| SEO Technical | `/seo-technical` | SEO técnico para Next.js |
| Marketing Copy | `/marketing-copy` | Copywriting Direct Response |
| UX Design | `/ux-design` | UX Design (princípios Apple) |
| Stripe | `/stripe` | Pagamentos internacionais |
| AbacatePay | `/abacatepay` | Pagamentos PIX (Brasil) |
| Cloudflare | `/cloudflare` | DNS, domínios, email routing, R2 storage |
| Favicon | `/favicon` | Geração de favicons e app icons |

## Commands Disponíveis

| Command | Descrição |
|---------|-----------|
| `/commit` | Stage all changes e cria commit com mensagem AI |
| `/push` | Push da branch atual para o remote |
| `/pr` | Cria Pull Request no GitHub |
| `/ship` | Commit + Push + PR em um só comando |

## Agents Disponíveis

| Agent | Descrição |
|-------|-----------|
| security-auditor | Auditoria de segurança para APIs, database, auth |

## Como Usar as Skills

### PostHog - Analytics & Feature Flags
```bash
# Invoke skill
/posthog
```
Implementa analytics com:
- Event tracking
- Feature flags
- Session replay
- Reverse proxy para evitar ad blockers

### SEO Technical
```bash
/seo-technical
```
Configura SEO completo:
- Sitemaps e robots.txt
- Meta tags e OpenGraph
- Structured data (JSON-LD)
- Performance (Core Web Vitals)

### Marketing Copy
```bash
/marketing-copy
```
Escreve copy usando:
- Framework Elevated Direct Response
- Tom contrarian educator
- Hooks e CTAs otimizados

### UX Design
```bash
/ux-design
```
Design de UX com:
- Princípios da era Jobs (Apple)
- Progressive disclosure
- Anticipatory design

### Stripe
```bash
/stripe
```
Integração de pagamentos:
- Checkout sessions
- Webhooks
- Subscriptions
- Customer portal

### AbacatePay
```bash
/abacatepay
```
Pagamentos PIX para Brasil:
- Billing sessions
- QR Code PIX
- Webhooks
- Subscriptions

### Cloudflare
```bash
/cloudflare
```
Setup de domínios e storage:
- DNS records (Clerk, Vercel)
- Email routing e redirects
- R2 buckets (CORS, public access, custom domains)
- Verificação de propagação

### Favicon
```bash
/favicon
```
Geração de favicons:
- Todos os tamanhos (16, 32, 180, 192, 512)
- Apple touch icon e Android chrome
- Web manifest para PWA
- Configuração de metadata Next.js

## Segurança

Após implementar features, execute o agent de segurança:
```
O agent security-auditor pode ser invocado para auditar:
- APIs e endpoints
- Database e RLS
- Autenticação e autorização
- Exposição de dados
```
