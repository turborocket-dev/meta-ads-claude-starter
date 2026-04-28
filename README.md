# Meta Ads + Claude Code Starter

> Quer falar com o Claude sobre suas campanhas do Facebook/Instagram em linguagem natural? Tipo "pausa essa campanha que tá queimando dinheiro" ou "quanto gastei essa semana?". Este repo te dá o caminho completo.

[![v1.0.1](https://img.shields.io/badge/version-1.0.1-blue)](https://github.com/thaleslaray/meta-ads-claude-starter/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

---

## 🔥 Por que este repo existe (a história)

**Em 2025-2026, uma onda de banimentos da Meta varreu a comunidade.** Pessoas que tentaram conectar AI agents (Claude, Cursor, OpenAI) nas suas contas de Meta Ads pra automatizar gestão **perderam contas inteiras**. Histórico de campanhas, audiências treinadas, pixel data, tudo apagado.

O caso mais famoso viralizou no Reddit: ["Claude Code got my Meta ads account permanently banned"](https://www.reddit.com/r/FacebookAds/comments/1sbsw6c/claude_code_got_my_meta_ads_account_permanently/). Um anunciante conectou Claude Code direto na Marketing API pra deixar o agente otimizando campanhas 24/7. Funcionou por 1 semana. Depois Meta baniu permanentemente.

### Por que isso aconteceu

Eu (Thales) pesquisei o caso a fundo (relatório completo em `docs/pesquisa-ban-risco.md`) e confirmei: **a Meta NÃO bane por usar AI**. Eles banem por **3 padrões específicos** que o caso viralizado fazia:

1. **Token mal configurado** — usar token de browser/sessão em vez de System User token assinado
2. **Tight polling** — fazer queries em loop apertado (cada poucos segundos), parecendo bot/scraper
3. **Mudanças sem revisão humana** — auto-publicar criativos, mudar budget às 3am, alterações em alta velocidade fora de horário comercial

A confirmação veio direto de um representante da Meta pra agência **Zentric Digital** (5 contas, 1.279 operações/mês, 18 meses zero bans):

> "A Meta não bane contas por usar AI. Bane contas cujo código viola rate limits, ignora sinais de integridade, ou quebra políticas de anúncios."

### O que este repo garante

Tudo neste repo foi desenhado pra **falhar em ZERO** dos 3 triggers de ban:

- ✅ **System User token** (não browser session) + assinatura `appsecret_proof` em toda chamada
- ✅ **Rate limiter integrado** que pausa polling em 60% da quota (você nunca chega perto do limite)
- ✅ **HITL (Human-In-The-Loop)** obrigatório pra mudanças sensíveis (criar criativo, +20% budget, etc.)
- ✅ **Audit log** append-only de toda chamada (se Meta perguntar, você prova uso legítimo)
- ✅ **Skill `meta-ads-compliance`** que aplica as 5 regras anti-ban automaticamente em cada conversa

**Resultado:** você pode rodar Claude Code conectado às suas contas de Meta Ads **com tranquilidade**, sabendo que tem cinto de segurança. Mesmo se você tentar pedir algo perigoso, o Claude (com a skill ativa) recusa e explica o porquê.

### Pra quem fez sentido criar isso

Eu administro 3 contas próprias da Escola de Automação que movem dezenas de milhares de reais/mês em Meta Ads. Eu queria usar Claude pra automatizar gestão sem virar mais um caso de banimento. Não achei nenhum boilerplate que cobrisse o problema completo (do setup até o App Review). Então construí.

Decidi liberar público porque:
1. **Toda pessoa que usa Claude com Meta Ads precisa disso** — sem isso, o risco é real
2. **A documentação da Meta é confusa** — leva semanas pra montar tudo do zero
3. **Mentorados meus precisavam** — em vez de explicar pra cada um, deixei aberto

Se este repo te economiza dor de cabeça (ou previne um banimento), ⭐ no GitHub me ajuda a saber se valeu a pena.

---

## 📖 Primeiro: do que estamos falando?

Se você nunca mexeu com automação de Meta Ads, leia esta seção primeiro. Vou explicar do absoluto zero, sem jargão.

### O que é "Meta Ads via API"

Quando você quer mexer nas suas campanhas do Facebook/Instagram, você normalmente:

1. Abre o **Meta Ads Manager** (https://business.facebook.com)
2. Clica nos botões pra ver campanhas, mudar budget, pausar, etc.

Mas você também pode mexer **sem clicar em nada** — usando uma "porta dos fundos" chamada **Marketing API**. Essa API permite que **um programa** (no nosso caso, o Claude) faça as mesmas ações: ver campanhas, pausar, mudar budget, criar anúncios.

**Por que isso é útil?**

Em vez de você abrir o Ads Manager e fazer 10 cliques, você pode falar:

> "Claude, pausa todas as campanhas com CPA acima de R$ 50"

E ele faz. Em 5 segundos. Sem você precisar clicar em nada.

### Por que existe um "limite" (e por que esse repo existe)

A Meta deixa qualquer um usar a Marketing API, **mas com limite**. Eles chamam esse limite de **pontos por hora**. Funciona assim:

- Cada **leitura** (tipo "ver lista de campanhas") custa **1 ponto**
- Cada **escrita** (tipo "pausa essa campanha") custa **3 pontos**

Quando você cria seu app na Meta, ele começa num tier chamado **Development Access** com **60 pontos por hora** — ou seja, dá pra fazer ~20 leituras OU ~6 escritas por hora antes do limite estourar.

**Pra ganhar mais quota** (9.000 pontos/h, chamado **Standard Access**), você precisa passar pelo **Meta App Review** — o time da Meta avalia seu app, vê se tá tudo certo, e libera. Esse processo costuma rejeitar 90% dos apps na primeira tentativa.

**Este repo é o caminho mais curto pra passar de 60 → 9.000 pts/h.** Aprovação testada em **2 horas** na primeira tentativa.

### Mas você pode usar HOJE com 60 pts/h?

**Pode.** 60 pontos/hora dá pra fazer:
- ~20 leituras de dados por hora ("quanto gastei?", "qual campanha tá pior?", "mostra meu CTR")
- ~6 escritas por hora (pausar/ativar/mudar budget de uma campanha por vez)

Isso cobre o uso de uma pessoa **fazendo 1-2 mexidas por hora**. Só não cobre **automação em volume** (rodar coisas em loop) ou **análises pesadas**.

**O que muda quando você passa pra Standard Access (9.000 pts/h):**
- Pode automatizar de verdade (rodar regras, alertas, relatórios diários)
- Pode fazer análises pesadas em segundos
- Pode ter o Claude rodando 24/7 monitorando suas contas

---

## 💡 A "barra de quota" automática

Quando você usa o Claude com este repo, ele **monitora sua quota em tempo real**. Se você tá perto do limite, Claude **avisa**:

```
⚠️ Quota Meta API: 38/60 pts usados nesta hora (63%)
   Recomendo pausar polling até reset (~22min).
```

E se você tentar fazer algo que vai estourar o limite, Claude **bloqueia**:

```
⛔ Operação bloqueada: faria você passar de 60 pts/h.
   Reset em 18 minutos, ou você pode submeter App Review
   pra ganhar Standard Access (9.000 pts/h, 150x mais).
```

Ou seja: você não corre risco de quebrar nada. Pode começar HOJE mesmo com 60 pts/h e só decide submeter App Review quando começar a apertar de verdade.

---

## ⚠️ Antes de continuar: você precisa MESMO de App Review?

Lembra que App Review serve pra ganhar **Standard Access (9.000 pts/h)**? Pois é. Se você só quer **ler** dados (não escrever), tem um atalho **bem mais fácil** que evita App Review completamente.

### A pergunta-chave: você quer LER ou ESCREVER?

| O que você quer fazer | Exemplos | Solução |
|----------------------|----------|---------|
| **Só LER** dados | "Quanto gastei essa semana?", "Qual campanha tem o melhor ROAS?", "Mostra meu CTR" | 🟢 **Use Windsor.ai** (5min, sem App Review) — [pula pra explicação ↓](#-caso-1--você-só-quer-ler-dados) |
| **ESCREVER** mudanças | "Pausa essa campanha", "Aumenta o budget pra R$ 500", "Cria uma campanha nova" | 🔴 **Use este repo** (precisa de App Review) — [pula pra explicação ↓](#-caso-2--você-quer-escrever-precisa-deste-repo) |
| **Ambos** | Algumas leituras + algumas escritas | 🔴 Use este repo (cobre os 2 casos) |

> **Por que Windsor.ai resolve só pra leitura?** Windsor.ai é uma empresa que **já passou pelo App Review da Meta** com o app deles. Quando você conecta sua conta lá, você usa a quota DELES (que é gigante, sem limite prático). Mas eles só liberam leitura. Pra escrever (mudar campanhas), você precisa do SEU próprio app aprovado pela Meta — que é exatamente o que este repo te ajuda a fazer.

---

## 🟢 CASO 1 — Você só quer LER dados

**Boa notícia:** você não precisa deste repo. Você não precisa criar app na Meta. Você não precisa passar por App Review. Tudo isso é dor de cabeça desnecessária pra você.

### O que usar: Windsor.ai + Claude

[Windsor.ai](https://windsor.ai) é uma empresa que conecta com Meta Ads (e Google Ads, TikTok, etc.) e te dá acesso aos dados via uma API que **já passou pelo App Review da Meta** — você herda a aprovação deles.

Eles têm um **MCP oficial integrado ao Claude.ai**, então é só:

1. **Cria conta gratuita** em https://windsor.ai (3 min)
2. **Conecta sua conta Meta Ads** clicando em "Add Connector → Facebook Ads" (1 min, faz o OAuth)
3. **Vai pro Claude.ai** (não Claude Code, é a versão web/app)
4. **Conecta o Windsor.ai** em Settings → Connectors → Windsor.ai
5. **Pergunta o que quiser:**
   - "Quanto gastei em Meta Ads essa semana?"
   - "Compara o CTR das minhas 3 contas"
   - "Quais campanhas têm CPA > R$ 50?"

**Pronto.** Você tá fazendo análise de Meta Ads via Claude sem ter mexido em nenhuma linha de código. Custo: zero (free tier do Windsor cobre uso pessoal).

> 💡 **Quando o Windsor não basta?** Quando você quer **mudar** alguma coisa (pausar campanha, mudar budget). Nesse caso volta aqui em cima e segue o **Caso 2**.

---

## 🔴 CASO 2 — Você quer ESCREVER (precisa deste repo)

Se você confirmou que precisa fazer mudanças nas campanhas via API (não só ler), então segue. Esta é a parte difícil — vou te guiar passo a passo.

### A história do problema

A Meta tem dois "tiers" de acesso pra Marketing API:

| Tier | Quota | Quem tem |
|------|-------|----------|
| **Development Access** (padrão) | 60 chamadas/h | Todo app novo começa aqui |
| **Standard Access** (precisa aprovação) | 9.000 chamadas/h | Apps que passaram pelo App Review |

60 chamadas por hora **esgota em segundos** quando você tá automatizando. É como se você tivesse um carro com tanque de 1 litro de combustível.

Pra ganhar Standard Access (150x mais quota), você precisa **convencer um humano da Meta** que seu app é legítimo e bem feito. Esse processo é o **Meta App Review**.

### Por que App Review é difícil

Meta recebe **milhares de submissões por dia**. O reviewer dá ~5 minutos pra cada app. Se em 5 minutos ele não entende o que você faz, **rejeita sem ler direito**.

Os 90% que são rejeitados na primeira tentativa caem porque:
- Descrição confusa ou genérica ("nosso app otimiza ads") — reviewer não entende o que é específico
- Vídeo demo mal feito (sem captions, fora de foco, mostrando coisa errada)
- Forms mal preenchidos ("none of the above" em campos que pedem maturidade de compliance)
- URL de privacidade quebrada (404)
- Tentativas óbvias de "burlar o sistema" (auto-publicar criativos sem revisão humana)

### O que este repo entrega

A **fórmula testada que aprovou em 2 horas na primeira tentativa** quando submetemos pra app `meta.escoladeautomacao.com.br` em abril/2026.

Encapsulada em:
1. **Dashboard pronto** — você não precisa construir UI nenhuma. Só configura 5 variáveis e deploya. Esse dashboard é o "objeto" que o reviewer vai analisar no vídeo.
2. **Skills do Claude Code** — quando você abre o projeto no Claude Code, ele já carrega 3 skills (`meta-ads-compliance`, `meta-ads-warmup`, `meta-app-review-approval`) que te guiam em cada passo.
3. **Templates** — descrição, instruções pro analista, captions Netflix-style do vídeo, todos prontos pra adaptar.
4. **Documentação detalhada** — 5 docs cobrindo setup → deploy → app review → operação → troubleshooting.

### Resultado típico (com este repo)

| Métrica | Sem este repo | Com este repo |
|---------|---------------|---------------|
| Tentativas até aprovação | 3-10 | 1 |
| Tempo total | 2-8 semanas | 2h-3 dias |
| Tempo seu (de trabalho ativo) | 10-20h | 30 min |

---

## ✅ Pré-requisitos (caso 2)

Antes de clonar o repo, garanta que tem TUDO isso. Se faltar 1, não adianta começar:

### Coisas da Meta

- [ ] **Conta no Business Manager** — https://business.facebook.com
- [ ] **Business Manager VERIFICADO** (selo verde nas Settings → Business Info)
  - **O que é:** Meta confirma que você é uma empresa real (CNPJ + endereço + telefone)
  - **Como fazer:** Business Settings → Security Center → Business Verification → segue o passo-a-passo (precisa de comprovante de empresa, ex: contrato social)
  - **Por que importa:** **App Review é AUTOMATICAMENTE rejeitado** se o BM não tá verificado, antes mesmo de um humano olhar
- [ ] **App criado** em https://developers.facebook.com/apps com use case "Other → Empresa"
- [ ] **Marketing API** adicionado como produto no app
- [ ] **Privacy Policy URL pública** configurada no app
  - **O que é:** uma página tipo `seusite.com/privacidade` explicando como você trata dados
  - **Onde por:** App Settings → Basic → Privacy Policy URL
  - **Pode usar:** se você já tem site institucional com política de privacidade, usa essa URL. Se não tem, gera uma rápida no Termly.io ou similar.

### Tokens (você vai colar isso no setup do repo)

Anote esses 3 valores:

| Onde achar | O que copiar | Parece com |
|------------|--------------|------------|
| Business Settings → Users → System Users → "Generate New Token" | `META_ACCESS_TOKEN` | `EAAXxxxxxxxxxxxxxxxxxxxxxx...` (200+ caracteres) |
| App Settings → Basic → App Secret (clica "Show", coloca senha do FB) | `META_APP_SECRET` | `abc123def456...` (32 caracteres) |
| Business Settings → Business Info → "ID da empresa" | `META_BM_ID` | `1234567890` (numérico) |

> ⚠️ **MUITO IMPORTANTE sobre o token:** ao gerar o System User token, marque essas 3 permissões: `ads_management`, `ads_read`, `business_management`. E define a expiração como "Never" (System User tokens não expiram, é normal).

### Stack local

- [ ] **Node.js 20+** — testa: `node --version` (se não tem, baixa em https://nodejs.org)
- [ ] **Python 3.11+** — testa: `python3 --version` (Mac já vem, Windows baixa em https://python.org)
- [ ] **Git** — testa: `git --version`
- [ ] **Claude Code** — testa: `claude --version` (instala em https://docs.anthropic.com/claude-code)
- [ ] **Conta Vercel** (gratuita) + CLI: `npm install -g vercel`

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
│  FASE 4: APROVADO! Standard Access ativo (9.000/h)          │
│  Operação contínua via Claude Code + MCP                    │
│  Dashboard pode ser desligado (cumpriu seu papel)           │
└─────────────────────────────────────────────────────────────┘
```

### ⚠️ MUITO IMPORTANTE: O dashboard é DEMO, não ferramenta de uso diário

O dashboard incluso (Next.js + FastAPI) tem **um único propósito**: ser a prova visual de funcionamento que você grava no vídeo pro Meta App Review.

**Após aprovado:**
- ✅ Você pode **desligar o Vercel** (economia de recursos)
- ✅ Sua operação real do dia-a-dia vira via **Claude Code + MCP `meta-ads-mcp`**, não pelo dashboard
- ✅ Se quiser manter o dashboard pra acessar via celular sem terminal, tudo bem — mas não é necessário

Pensa assim: o dashboard é o **portfolio que você mostra pro empregador na entrevista**. Depois de contratado, você não usa o portfolio mais — usa as ferramentas reais do trabalho.

---

## 🚀 Quickstart 30 minutos (passo-a-passo)

### Passo 1 — Clone o repo (1min)

Abre o terminal:

```bash
git clone https://github.com/thaleslaray/meta-ads-claude-starter
cd meta-ads-claude-starter
```

### Passo 2 — Liste suas ad accounts (5min)

Antes de rodar o setup, você precisa saber os IDs das contas que vai gerenciar.

1. Abre https://developers.facebook.com/tools/explorer/
2. Seleciona seu app no dropdown do canto direito
3. Cola no campo de query: `me/adaccounts?fields=id,name`
4. **Importante:** clica em "Get Token" → "Get User Access Token" e seleciona as permissions `ads_read` e `ads_management`. (Esse token é só pra fazer essa consulta agora, NÃO é o token do passo 3.)
5. Clica **Submit**

Você vai ver lista tipo:

```json
{
  "data": [
    {"id": "act_487731909607599", "name": "Conta Principal"},
    {"id": "act_708497467651098", "name": "Conta Teste"}
  ]
}
```

Anota os IDs (formato `act_xxxxxxxxx`) — você vai colar no próximo passo.

### Passo 3 — Setup interativo (5min)

```bash
./scripts/setup.sh
```

O script vai te perguntar uma coisa por vez. Cola cada valor:

```
META_ACCESS_TOKEN (System User token): EAAxxx...
META_APP_SECRET: abc123...
META_BM_ID (numeric Business Manager ID): 1234567890
ORG_NAME (your business name): Minha Empresa LTDA

Add ad account? (y/N) y
  Account ID (act_xxx): act_487731909607599
  Account name: Conta Principal
  Short label: Main

Add ad account? (y/N) y
  Account ID (act_xxx): act_708497467651098
  Account name: Conta Teste
  Short label: Test

Add ad account? (y/N) n

✓ .env written (2 accounts configured)
```

Ele cria o arquivo `.env` na raiz. **Esse arquivo tem segredos, NUNCA commita no git** (já tá no `.gitignore`).

### Passo 4 — Teste local (5min)

Pra confirmar que tudo funciona antes de subir pra Vercel:

```bash
cd dashboard
npm install              # baixa dependências do Next.js (~30s)
pip install -r requirements.txt  # baixa dependências do Python (~10s)
```

Abre **2 abas** do terminal:

**Aba 1 — backend Python:**
```bash
cd dashboard/api
uvicorn index:app --reload --port 8000
```

**Aba 2 — frontend Next.js:**
```bash
cd dashboard
npm run dev
```

Abre http://localhost:3000 no navegador. **Você deve ver:**
- ✅ Suas contas no dropdown do header
- ✅ Insights (se as contas têm gastos recentes)
- ✅ Lista de campanhas

Se NÃO funcionar, lê [docs/05-troubleshooting.md](./docs/05-troubleshooting.md). Erros mais comuns:
- "META_AD_ACCOUNTS env var is required" → o `.env` tá errado
- "Invalid OAuth access token" → token errado ou expirado
- "Invalid appsecret_proof" → App Secret tem espaço no final

### Passo 5 — Deploy Vercel (5min)

Vamos colocar isso no ar pra Meta poder acessar:

```bash
cd dashboard
vercel login    # primeira vez: vai abrir navegador pra fazer login
vercel link     # primeira vez: aceita criar projeto novo
```

Agora adiciona cada variável de ambiente (uma por uma):

```bash
vercel env add META_ACCESS_TOKEN production
# cola o token quando pedir, enter

vercel env add META_APP_SECRET production
# cola o secret, enter

vercel env add META_BM_ID production
# cola o BM ID, enter

vercel env add META_AD_ACCOUNTS production
# cola o JSON inteiro, enter
# (pega do seu .env, é o valor de META_AD_ACCOUNTS)

vercel env add ORG_NAME production
# cola o nome, enter
```

Agora deploy:

```bash
vercel --prod
```

No final aparece a URL: `https://meta-ads-dashboard-xxxx.vercel.app`. **Abre essa URL no navegador e confirma que funciona em produção.**

> 💡 **Opcional mas recomendado:** configurar custom domain pra ficar tipo `meta.suaempresa.com.br`. Vai em Vercel Dashboard → Settings → Domains → Add. **Vantagem:** parece mais profissional pro reviewer.

### Passo 6 — App Review (15min de trabalho + 2h-3 dias de espera)

Aqui é onde a mágica acontece. Volta pra raiz do repo e abre Claude Code:

```bash
cd ..  # volta pra raiz
claude
```

Quando o Claude abrir, ele já vem **pré-carregado** com:
- ✅ MCP `meta-ads-mcp` conectado (acesso direto à Marketing API)
- ✅ Skills `meta-ads-compliance`, `meta-ads-warmup`, `meta-app-review-approval`
- ✅ Contexto do projeto via `CLAUDE.md`

Então você simplesmente diz pra ele:

> "Vou submeter o Meta App Review pedindo Ads Management Standard Access. Me guia passo a passo."

E ele vai te guiar:
1. **Pré-flight check** — verifica se BM tá verificado, app tá Live, Privacy Policy responde 200
2. **Gravar screencast** — split-screen com seu dashboard à esquerda + Meta Ads Manager nativo à direita (mostra que muda em tempo real)
3. **Adicionar captions Netflix-style** com `ffmpeg` (script vem pronto)
4. **Preencher o form** em developers.facebook.com — descrição em 5 seções, instruções analista, todos os campos do "Tratamento de dados"
5. **Submeter**

Tempo de resposta da Meta: **2 horas a 3 dias**. Email vai chegar no email da conta dev.

### Passo 7 — Verificar aprovação 🎉

Quando receber o email "Your App Review results are ready":

```bash
./scripts/verify-tier.sh act_seu_id https://seu-dominio.vercel.app
```

Resposta esperada:

```
✓ APPROVED — tier is standard_access (9000 pts/h, was 60)
```

**Você acabou de ganhar 150x mais quota.** 🚀

---

## 🛠️ Detalhamento: o que cada peça faz

Quando você roda `claude` na pasta deste repo, **4 coisas carregam automaticamente** no Claude. Vou explicar cada uma com clareza.

### 1️⃣ MCP `meta-ads-mcp` — a "ponte" Claude ↔ Meta

**O que é:** MCP (Model Context Protocol) é o jeito que Claude conversa com APIs externas. O `meta-ads-mcp` é a ponte específica entre Claude e a Marketing API da Meta.

**O que faz por baixo dos panos:**
- Recebe seu pedido em linguagem natural ("lista campanhas ativas")
- Traduz pra chamada HTTP na Marketing API
- Assina cada chamada com `appsecret_proof` (assinatura HMAC-SHA256, exigência de segurança da Meta)
- Aplica rate limiter (não deixa estourar quota)
- Loga cada chamada num audit log JSONL
- Devolve a resposta em formato que Claude entende

**O que você vê na prática:**

Você fala:
> "Lista todas as minhas campanhas ativas com gasto > R$ 100"

Claude (via MCP) faz:
1. Chama `GET /act_xxx/campaigns?fields=id,name,status,spend`
2. Filtra status=ACTIVE e spend>100
3. Te mostra a lista formatada

Você não precisa entender nada de API, código, autenticação. Só fala em português.

**Configuração:** já vem pronta no `.mcp.json`. Lê `META_ACCESS_TOKEN` e `META_APP_SECRET` do seu `.env`. Zero esforço seu.

---

### 2️⃣ Skill `meta-ads-compliance` — o "freio de mão"

**Quando ativa:** **SEMPRE** que você mencionar "meta ads", "campanha", "anúncio" ou qualquer coisa relacionada. Skill auto-detecta o contexto e carrega.

**O que faz:**
- **Monitora quota** — lê o header `X-Business-Use-Case-Usage` em toda resposta da API e avisa quando chega em 60% de uso
- **Bloqueia padrões perigosos** — se você pedir algo tipo "faz refresh do dashboard a cada 30 segundos", ela recusa e explica o porquê (= ban-trigger)
- **Aplica 5 regras anti-ban automaticamente:**
  1. Polling ≥ 5 minutos (nunca tight loop)
  2. Writes só em horário comercial (8h-20h)
  3. HITL obrigatório em criar criativo, +20% budget, cross-account, criar campanha
  4. Pausar em 60% BUC (a famosa "barra")
  5. Audit log append-only
- **Sugere upgrade pra Standard** quando detecta tier dev sufocando o uso

**O que você vê na prática:**

Você fala:
> "Cria um auto-pause de campanha quando CPA > R$ 50"

Claude responde (com skill ativa):
> "Vou criar, mas com guard rails:
> - Cooldown de 6h entre pauses (evita flapping)
> - Mínimo 10 conversões antes de avaliar
> - Roda a cada 30min em horário comercial (não madrugada)
> - Log de cada pause no audit
>
> Confirma esses parâmetros antes de eu codar?"

Sem a skill, Claude faria o auto-pause "puro" e você correria risco de ban. Com a skill, vem com cinto de segurança.

**Cobre o ban-wave de 2025-2026** — apps que ignoraram essas regras foram banidos pela Meta nos últimos 6 meses.

---

### 3️⃣ Skill `meta-ads-warmup` — o "aquecedor"

**Quando ativa:** quando você quer aumentar o histórico de uso do app (útil enquanto espera App Review, ou pra mostrar atividade real).

**O que faz:** acumula chamadas legítimas à API (leituras inofensivas) pra "aquecer" o app pros sistemas da Meta. Útil em 2 cenários:

1. **Antes de submeter App Review** — Meta gosta de ver atividade real no app, não app dormente
2. **Caminho pra Advanced Access** — Advanced exige 1.500+ chamadas em 30 dias com <10% de erro

**O que você vê na prática:**

Você roda:
```bash
python scripts/warmup.py --depth standard --save
```

A skill (carregada quando você menciona warmup):
- Faz ~50-100 chamadas distribuídas em 30min (não tudo de uma vez)
- Respeita rate limit (pausa em 60% BUC)
- Salva log do que rodou em `audit_logs/`
- Te mostra um relatório:
  ```
  ✓ Warmup completo
  - 73 chamadas feitas (52 reads, 21 writes simulados)
  - 0 erros
  - Tier atual: development_access
  - Próximo passo recomendado: submeter App Review
  ```

**Quando descartar:** após ganhar Standard Access ou Advanced. Não precisa mais de warmup.

---

### 4️⃣ Skill `meta-app-review-approval` — o "GPS pro App Review"

**Quando ativa:** quando você fala "App Review", "Standard Access", "submeter pro Meta", ou termos similares.

**O que faz:** carrega o workflow completo de aprovação no contexto do Claude. Inclui:

- **Phase 0** — blueprint do dashboard demo (5 features mínimas)
- **Phase 1** — checklist pré-flight (BM verificado? App Live? Privacy Policy 200?)
- **Phase 2** — template de descrição em 5 seções (actor, auth, what app does, why standard, compliance)
- **Phase 3** — guia de gravação de screencast com captions Netflix-style
- **Phase 4** — preenchimento do form (todos os campos com resposta certa)
- **Phase 5** — submeter + verificar tier após aprovação

**O que você vê na prática:**

Você fala:
> "Quero submeter pro Meta App Review pedindo Standard Access"

Claude (com skill ativa) responde:
> "Beleza. Antes de qualquer coisa, vamos validar 6 pré-requisitos. Me responde:
>
> 1. Seu Business Manager está verificado (selo verde)?
> 2. App está em modo Live (não Development)?
> 3. App tem Privacy Policy URL configurada?
> 4. Você tem System User token com permissions ads_management + ads_read?
> 5. Token assina com appsecret_proof?
> 6. Dashboard está deployado e respondendo 200 numa URL pública?
>
> Se algum for 'não', a Meta rejeita automaticamente sem human review.
> Vamos validar um a um."

Depois ele te guia: gravar vídeo → captions → preencher form → submeter → esperar email → verificar tier.

**Templates inclusos** (em `examples/`):
- `descricao-app-review.txt` — colar no campo "Descrição" do form
- `instrucoes-analista.txt` — colar no campo "Instruções para o analista"

**Auto-suficiente:** mesmo se você usar essa skill em outro projeto sem clonar este repo, ela tem o blueprint completo. Você consegue replicar o processo "from scratch".

---

## 🎬 Resumo visual do fluxo

```
┌──────────────────────────────────────────────────────────────┐
│                   Você fala em português                     │
│              "lista minhas campanhas ativas"                 │
└──────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────┐
│                   Claude Code (no terminal)                  │
│  Carrega: CLAUDE.md + .mcp.json + 3 skills                   │
└──────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────┬───────────────────────┬────────────────┐
│  Skill compliance   │       MCP             │   Skill review │
│  • Checa quota      │  • Faz HTTP request   │  • Carrega só  │
│  • Aplica regras    │  • Assina HMAC        │    quando você │
│  • Bloqueia perigo  │  • Loga audit         │    pede        │
│  • Avisa "barra"    │  • Devolve dados      │                │
└─────────────────────┴───────────────────────┴────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────┐
│              Meta Marketing API (oficial)                    │
└──────────────────────────────────────────────────────────────┘
                              ↓
                    Resposta volta pra você
                em linguagem natural, formatada
```

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

### Eu sou leigo total, dá pra fazer mesmo?

Dá. Você precisa saber:
- Abrir terminal e copiar comandos (não precisa entender o que cada comando faz)
- Cadastrar coisas em sites (Meta, Vercel, GitHub)
- Ler instruções com calma

Você NÃO precisa saber:
- Programar
- Como funciona Next.js, FastAPI, Vercel internamente
- Inglês fluente (a doc é toda pt-BR)

Se travar, lê o [docs/05-troubleshooting.md](./docs/05-troubleshooting.md) ou abre Issue no GitHub.

### Quanto tempo dura todo o processo?

| Fase | Tempo seu | Tempo Meta |
|------|-----------|------------|
| Setup local | 30min | — |
| Submeter App Review | 15min | 2h-3 dias |
| **Total** | **~45min** | **2h-3 dias** |

### Quanto custa?

| Serviço | Custo |
|---------|-------|
| Meta API (Standard Access) | R$ 0 |
| Vercel (free tier) | R$ 0 |
| GitHub | R$ 0 |
| Claude Code | depende do seu plano (Pro a partir de $20/mês) |
| **Total** | **~R$ 100/mês (só Claude Code)** |

### O dashboard precisa ficar rodando pra sempre?

**Não.** Ele é só demo pra App Review. Após aprovado, pode desligar o Vercel sem problema. Operação real é via Claude Code + MCP.

### O que acontece se eu não passar no primeiro App Review?

Meta manda email com nota específica do que faltou. As mais comuns:
- **"Unable to verify use case experience"** → vídeo não mostrava bem o System User auth → re-grava com caption explícita
- **"Privacy policy URL not accessible"** → URL deu 404 → corrige no app dashboard
- **"Permissions don't match demonstrated use"** → você pediu ads_management mas só mostrou leitura no vídeo → re-grava mostrando uma escrita

Re-submit é rápido (3-5 dias). A skill `meta-app-review-approval` tem `references/myths.md` cobrindo todos os casos.

### Posso usar pra atender clientes externos?

Não com este repo. **Standard Access é só pra contas próprias.** Se você quer atender clientes externos, precisa de **Advanced Access** (processo bem mais complexo, precisa demonstrar OAuth de cliente, etc.).

A skill `meta-app-review-approval` tem uma seção "Quando precisar do próximo tier" explicando essa diferença.

### Posso usar com Python? Node? Outras stacks?

A stack do dashboard incluso é Next.js + FastAPI, mas o **conceito vale pra qualquer stack**. A skill `meta-app-review-approval` tem o blueprint do que o demo precisa mostrar — você pode reimplementar em Django, Rails, Vue, qualquer coisa.

### Funciona fora do Brasil?

Sim. O repo tá em pt-BR mas o código é universal. Templates em `examples/` estão em pt-BR mas você pode traduzir. App Review form pode ser preenchido em pt-BR ou inglês.

### E se eu só quiser ler dados, tem como evitar tudo isso?

**Sim, leia [a seção do topo deste README](#%EF%B8%8F-antes-de-começar-você-precisa-mesmo-disso).** Spoiler: usa Windsor.ai + Claude.ai e resolve em 5min.

---

## 🆘 Suporte

### Travou? Antes de pedir ajuda:

1. **Lê [docs/05-troubleshooting.md](./docs/05-troubleshooting.md)** — cobre 80% dos erros
2. **Procura em [GitHub Issues](https://github.com/thaleslaray/meta-ads-claude-starter/issues)** (talvez já foi reportado)
3. **Tenta reproduzir com env limpa** (às vezes é cache local)

### Ainda travou? Abre um Issue

Vai em https://github.com/thaleslaray/meta-ads-claude-starter/issues/new

**Inclui:**
- Comando exato que rodou
- Erro completo (não corta)
- Output de `vercel logs --since 30m` se for problema de deploy
- Versão dos pré-requisitos: `node --version`, `python3 --version`, `vercel --version`

**NÃO inclui:**
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
- 🔄 Compartilha com quem tá penando no App Review
- 💬 Conta no Issues como foi sua experiência (positiva ou negativa, ambas ajudam a melhorar)
