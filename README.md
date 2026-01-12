# tiny-boiler

Boilerplate de skills do Claude Code para projetos Next.js.

## O que é isso?

Uma coleção de skills e agents pré-configurados para o Claude Code, prontos para usar em qualquer projeto.

## Como Usar

### 1. Clone o repositório

```bash
git clone git@github.com:andrehfp/tiny-boiler.git
cd tiny-boiler
```

### 2. Copie a pasta `.claude/` para seu projeto

```bash
cp -r .claude/ /caminho/do/seu/projeto/
```

### 3. Abra o Claude Code no seu projeto

```bash
cd /caminho/do/seu/projeto
claude
```

### 4. Use as skills

```bash
/posthog        # Analytics & Feature Flags
/seo-technical  # SEO para Next.js
/marketing-copy # Copywriting Direct Response
/ux-design      # UX Design
/stripe         # Pagamentos Stripe
/abacatepay     # Pagamentos PIX
```

## Skills Incluídas

| Skill | Descrição |
|-------|-----------|
| `posthog` | Analytics, feature flags, session replay com reverse proxy |
| `seo-technical` | SEO técnico: sitemaps, meta tags, structured data |
| `marketing-copy` | Copy usando Elevated Direct Response framework |
| `ux-design` | UX com princípios Jobs-era Apple |
| `stripe` | Checkout, webhooks, subscriptions, customer portal |
| `abacatepay` | Pagamentos PIX para o mercado brasileiro |

## Agents Incluídos

| Agent | Descrição |
|-------|-----------|
| `security-auditor` | Auditoria de segurança para APIs, database, auth |

## Estrutura

```
.claude/
├── CLAUDE.md              # Instruções base
├── settings.local.json    # Configurações do Claude Code
├── skills/
│   ├── posthog/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── seo-technical/
│   ├── marketing-copy/
│   ├── ux-design/
│   ├── stripe/
│   └── abacatepay/
└── agents/
    └── security-auditor.md
```

## Personalização

### Adicionar novas skills

1. Crie uma pasta em `.claude/skills/nome-da-skill/`
2. Adicione um arquivo `SKILL.md` com as instruções
3. Opcionalmente, adicione uma pasta `references/` com documentação

### Modificar skills existentes

Edite o arquivo `SKILL.md` ou os arquivos em `references/` conforme necessário.

## Requisitos

- [Claude Code CLI](https://claude.ai/code)
- Conta Anthropic com acesso ao Claude Code

## Licença

MIT
