# Meta Ads + Claude Code Starter

> Boilerplate testado pra passar pelo Meta App Review na primeira tentativa e ganhar Standard Access (9.000 pts/h em vez de 60 pts/h).

[![v1.0.1](https://img.shields.io/badge/version-1.0.1-blue)](https://github.com/thaleslaray/meta-ads-claude-starter/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

---

## 📖 Índice

1. [O que é isso](#-o-que-é-isso)
2. [Pra quem serve](#-pra-quem-serve)
3. [Pré-requisitos](#-pré-requisitos)
4. [Como funciona (visão geral)](#-como-funciona-visão-geral)
5. [Quickstart 30 minutos](#-quickstart-30-minutos)
6. [O que vem dentro](#-o-que-vem-dentro)
7. [FAQ](#-faq)
8. [Suporte](#-suporte)

---

## 🎯 O que é isso

A Meta Marketing API tem uma armadilha cruel: quando você cria um app novo, ele fica no tier **Development Access**, que dá **60 pontos por hora**. Cada leitura custa 1 ponto, cada escrita custa 3. Em poucos minutos de uso real, você esgota a quota e a API começa a retornar erro 613 (throttle).

Pra ganhar **Standard Access** (9.000 pts/h, ou seja, **150x mais quota**), você precisa passar pelo processo de **Meta App Review** — o time da Meta avalia seu app, valida que você tá usando a API de forma legítima, e libera a quota.

**O problema:** 90% dos apps são rejeitados na primeira tentativa porque o reviewer não consegue entender o que o app faz, ou porque a descrição/screencast/forms estão fora do padrão que a Meta espera. Você fica iterando 3-10 vezes, perdendo semanas.

**O que este repo entrega:** a fórmula testada que aprovou em **2 horas na primeira tentativa**, encapsulada em código + skills do Claude Code + documentação. Você clona, configura 5 variáveis de ambiente, deploya, segue o passo-a-passo e submete.

### Resultado típico

| Métrica | Sem este repo | Com este repo |
|---------|---------------|---------------|
| Tentativas até aprovação | 3-10 | 1 |
| Tempo total | 2-8 semanas | 2h-3 dias |
| Tempo de operador | 10-20h (gravando vídeos, refazendo descrição) | 30 min de setup + 15 min de form |
| Dor de cabeça | Alta | Baixa |

---

## 👤 Pra quem serve

✅ **Serve se você:**
- Tem 1+ contas próprias de Meta Ads (não está atendendo clientes externos)
- Quer automatizar gestão dessas contas via Claude Code, scripts, ou dashboards próprios
- Tá no tier Development Access (60 pts/h) e precisa subir pra Standard
- Tem stack técnica básica (consegue rodar `git clone` e `vercel deploy`)

❌ **Não serve se você:**
- Quer atender contas de clientes externos (precisa de Advanced Access, processo diferente)
- Não tem Business Manager verificado (faz a verificação primeiro)
- Não tem nenhum app Meta ainda (cria o app primeiro em developers.facebook.com)

---

## ✅ Pré-requisitos

Antes de clonar o repo, garanta que tem:

### Conta Meta

- [ ] Conta no [Meta Business Manager](https://business.facebook.com)
- [ ] **BM verificado** (selo verde nas Settings → Business Info). **Sem isso o App Review é rejeitado.** Pra verificar: Business Settings → Security Center → faça verificação de business (precisa de comprovante de empresa).
- [ ] App criado em [developers.facebook.com/apps](https://developers.facebook.com/apps) com use case "Other → Empresa"
- [ ] Marketing API adicionado como produto no app
- [ ] Privacy Policy URL pública configurada no app (pode ser do site marketing, ex: `https://www.empresa.com/privacy`)

### Tokens

Anote esses 3 valores antes de começar:

| Onde achar | O que copiar |
|------------|--------------|
| Business Settings → Users → System Users → Generate New Token | `META_ACCESS_TOKEN` (com permissions: `ads_management`, `ads_read`, `business_management`) |
| App Settings → Basic → App Secret (clique "Show") | `META_APP_SECRET` |
| Business Settings → Business Info → Business ID | `META_BM_ID` (numérico) |

### Stack local

- [ ] Node.js 20+ (`node --version`)
- [ ] Python 3.11+ (`python3 --version`)
- [ ] Git (`git --version`)
- [ ] [Claude Code](https://docs.anthropic.com/claude-code) instalado (`claude --version`)
- [ ] Conta Vercel (free tier funciona) + CLI: `npm install -g vercel`

---

## 🔄 Como funciona (visão geral)

```
┌─────────────────────────────────────────────────────────────┐
│  FASE 1: SETUP (15min)                                      │
│  Clone repo + tokens + .env                                 │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  FASE 2: DEPLOY (15min)                                     │
│  Vercel deploy → URL pública pra reviewer testar            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  FASE 3: APP REVIEW (2h-3 dias)                             │
│  Gravar screencast no dashboard + form + submit             │
│  Skill `meta-app-review-approval` te guia passo-a-passo     │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  FASE 4: APROVADO! Standard Access ativo (9.000 pts/h)      │
│  Operação contínua via Claude Code + MCP                    │
│  Dashboard pode ser desligado (cumpriu seu papel)           │
└─────────────────────────────────────────────────────────────┘
```

### ⚠️ IMPORTANTE: O dashboard é DEMO, não ferramenta

O dashboard incluso (Next.js + FastAPI) tem **um único propósito**: ser a prova visual de funcionamento que você grava no screencast pro Meta App Review. Após aprovado:

- ✅ Você pode **desligar o Vercel** (economia de recursos)
- ✅ Operação real do dia-a-dia é via **Claude Code + MCP `meta-ads-mcp`**, não pela UI
- ✅ Se quiser manter o dashboard pra acessar via celular sem terminal, pode — mas não é necessário

---

## 🚀 Quickstart 30 minutos

### Passo 1 — Clone o repo (1min)

```bash
git clone https://github.com/thaleslaray/meta-ads-claude-starter
cd meta-ads-claude-starter
```

### Passo 2 — Liste suas ad accounts (5min)

Antes de rodar o setup, você precisa dos IDs das contas que vai gerenciar.

1. Abra https://developers.facebook.com/tools/explorer/
2. Selecione seu app no dropdown
3. Cole no campo: `me/adaccounts?fields=id,name`
4. Use seu System User token (não o token do explorer)
5. Clique **Submit** — vai aparecer lista tipo:

```json
{
  "data": [
    {"id": "act_487731909607599", "name": "Conta Principal"},
    {"id": "act_708497467651098", "name": "Conta Teste"}
  ]
}
```

Anote os IDs (formato `act_xxxxxxxxx`).

### Passo 3 — Setup interativo (5min)

```bash
./scripts/setup.sh
```

O script vai te perguntar:
- `META_ACCESS_TOKEN` → cola o System User token
- `META_APP_SECRET` → cola o App Secret
- `META_BM_ID` → cola o Business ID
- `ORG_NAME` → nome do seu negócio (aparece no dashboard)
- Pra cada ad account: ID, nome, label curto

Ele gera um arquivo `.env` na raiz. **Esse arquivo é secreto, NUNCA commita.**

### Passo 4 — Teste local (5min)

```bash
cd dashboard
npm install                              # ~30s
pip install -r requirements.txt          # ~10s

# Em uma aba, sobe o backend Python:
cd api && uvicorn index:app --reload --port 8000

# Em outra aba, sobe o frontend Next:
npm run dev
```

Abra http://localhost:3000. **Você deve ver:**
- Suas contas no dropdown do header
- Insights (se tiver dados das contas)
- Lista de campanhas

Se não funcionar, veja [docs/05-troubleshooting.md](./docs/05-troubleshooting.md).

### Passo 5 — Deploy Vercel (5min)

```bash
cd ..  # volta pra raiz do dashboard/
vercel login
vercel link        # primeira vez: aceita criar projeto novo

# Adiciona env vars (uma por uma):
vercel env add META_ACCESS_TOKEN production
vercel env add META_APP_SECRET production
vercel env add META_BM_ID production
vercel env add META_AD_ACCOUNTS production
vercel env add ORG_NAME production

# Deploy:
vercel --prod
```

No final, pega a URL `https://meta-ads-dashboard-xxx.vercel.app`. Abra e confirme que funciona em produção.

**Opcional:** configure custom domain em Vercel → Settings → Domains. Recomendado: subdomínio do seu site marketing (ex: `meta.empresa.com`).

### Passo 6 — App Review (15min de trabalho + 2h-3 dias de espera)

```bash
cd ..  # volta pra raiz do repo
claude  # abre Claude Code dentro do projeto
```

Claude já carrega:
- ✅ MCP `meta-ads-mcp` conectado
- ✅ Skills `meta-ads-compliance`, `meta-ads-warmup`, `meta-app-review-approval`
- ✅ Contexto via `CLAUDE.md`

Diga ao Claude:

> "vou submeter pro Meta App Review pedindo Ads Management Standard Access"

Ele vai te guiar:
1. Pré-flight check (verificar se BM tá verificado, app tá Live, Privacy Policy responde 200)
2. Gravar screencast (split-screen dashboard + Ads Manager nativo)
3. Adicionar captions Netflix-style com `ffmpeg`
4. Preencher o form em developers.facebook.com (descrição em 5 seções, instruções analista, etc.)
5. Submeter

**Tempo de resposta da Meta:** 2 horas a 3 dias. Email vai chegar no email da conta dev.

### Passo 7 — Verificar aprovação

Quando receber o email "Your App Review results are ready":

```bash
./scripts/verify-tier.sh act_seu_id https://seu-dominio.vercel.app
```

Resposta esperada:

```
✓ APPROVED — tier is standard_access (9000 pts/h, was 60)
```

🎉 **Pronto. Você tá no Standard Access.**

---

## 📦 O que vem dentro

```
meta-ads-claude-starter/
├── README.md                          ← este arquivo
├── CLAUDE.md                          ← instruções pro Claude Code (auto-carregado)
├── LICENSE                            ← MIT
├── .gitignore
│
├── .env.example                       ← template das env vars
├── .mcp.json                          ← config do MCP meta-ads-mcp
│
├── .claude/skills/                    ← 3 skills auto-carregadas pelo Claude
│   ├── meta-ads-compliance/           ← regras anti-ban (sempre ativa)
│   ├── meta-ads-warmup/               ← acumula calls enquanto espera review
│   └── meta-app-review-approval/      ← workflow completo de aprovação
│
├── dashboard/                         ← Next.js + FastAPI (DEMO pra App Review)
│   ├── api/
│   │   ├── index.py                   ← endpoints FastAPI
│   │   └── meta_ads_mcp/              ← package vendored com client + audit + rate limiter
│   ├── app/                           ← Next.js App Router pages
│   ├── components/                    ← componentes UI
│   ├── lib/                           ← hooks SWR + helpers
│   └── vercel.json                    ← config Vercel Python Functions
│
├── docs/                              ← documentação detalhada
│   ├── 01-setup.md                    ← criar app Meta + tokens
│   ├── 02-deploy.md                   ← deploy Vercel
│   ├── 03-app-review.md               ← submeter App Review
│   ├── 04-operacao.md                 ← uso pós-aprovação
│   └── 05-troubleshooting.md          ← erros comuns
│
├── scripts/
│   ├── setup.sh                       ← wizard interativo de setup
│   ├── verify-tier.sh                 ← checa tier após aprovação
│   └── warmup.py                      ← acumula API calls
│
└── examples/
    ├── descricao-app-review.txt       ← template em pt-BR pra colar no form
    └── instrucoes-analista.txt        ← template em pt-BR
```

---

## ❓ FAQ

### O dashboard precisa ficar rodando pra sempre?

**Não.** Ele é só demo pra App Review. Após aprovado, pode desligar o Vercel.

### Posso usar o dashboard pra operação contínua?

Pode, mas não é o uso recomendado. Operação real funciona melhor via Claude Code + MCP. Dashboard é frágil (precisa manter Vercel + redeploys), enquanto Claude + MCP é só rodar `claude` na pasta.

### O que acontece se eu não passar no primeiro App Review?

Meta manda nota específica do que faltou. A skill `meta-app-review-approval` tem `references/myths.md` e `references/form-fields.md` cobrindo as causas mais comuns. Re-submit é rápido (3-5 dias).

### Como faço pra Advanced Access (acima do Standard)?

Advanced Access serve pra **third-party SaaS** atendendo contas de clientes externos. Pra uso first-party (suas próprias contas), Standard Access é o final da linha. Se você virar SaaS, a skill `meta-app-review-approval` tem uma seção "Quando precisar do próximo tier".

### Posso usar com Python? Node? Outras stacks?

A stack do dashboard incluso é Next.js + FastAPI, mas o **conceito vale pra qualquer stack**. A skill `meta-app-review-approval` tem o blueprint do que o demo precisa mostrar — você pode reimplementar em Django, Rails, Vue, qualquer coisa.

### Quanto custa rodar isso?

- **Vercel free tier:** R$ 0 (suficiente pro demo + uso interno baixo)
- **Meta API:** R$ 0 (Standard Access é grátis)
- **Claude Code:** depende do plano que você tem

### Funciona fora do Brasil?

Sim. O repo tá em pt-BR mas o código é universal. Templates em `examples/` estão em pt-BR mas você pode traduzir. App Review form pode ser em pt-BR ou inglês.

### Eu sou aluno da Escola de Automação. Posso usar?

Pode. O repo é MIT — qualquer um pode usar. Mas se travar, você pode pedir suporte direto pelo seu canal de aluno (em vez de Issues GitHub).

---

## 🆘 Suporte

### Travou? Antes de pedir ajuda:

1. Leia [docs/05-troubleshooting.md](./docs/05-troubleshooting.md) — cobre 80% dos erros
2. Procure em [GitHub Issues](https://github.com/thaleslaray/meta-ads-claude-starter/issues) (talvez já foi reportado)
3. Tente reproduzir com env limpa (às vezes é cache local)

### Ainda travou? Abra um Issue

Vá em https://github.com/thaleslaray/meta-ads-claude-starter/issues/new

**Inclua:**
- Comando exato que rodou
- Erro completo (não corte)
- Output de `vercel logs --since 30m` se for problema de deploy
- Versão dos pré-requisitos: `node --version`, `python3 --version`, `vercel --version`

**NÃO inclua:**
- ❌ `META_ACCESS_TOKEN`
- ❌ `META_APP_SECRET`
- ❌ Qualquer secret

---

## 📜 Licença

MIT — qualquer um pode usar, modificar, distribuir, vender. Ver [LICENSE](./LICENSE).

## 👤 Autor

Maintido por [Thales Laray](https://github.com/thaleslaray).

---

## ⭐ Gostou?

Se este repo te economizou tempo:
- ⭐ Star no GitHub (ajuda outros a encontrar)
- 🔄 Compartilhe com quem tá penando no App Review
- 💬 Conta no Issues como foi sua experiência (positiva ou negativa, ambas ajudam a melhorar)
