# Pesquisa: Risco de ban com automação no Meta Ads Standard Access

**Gerado:** 2026-04-25
**Contexto:** Internal tool first-party (Escola de Automação) recém-aprovado em Standard Access (9.000 pts/h). Pergunta: rodar as automações sugeridas (auto-pause, auto-scale, day-parting, polling, AI agent 24/7) pode causar ban?
**Confiança:** ALTA

---

## Resumo Executivo

**NÃO vai ser banido se seguir 5 regras.** O ban-wave de 2025-2026 atinge 3 perfis específicos — Escola de Automação não se encaixa em nenhum, dado o setup atual (App Review aprovado, System User token + appsecret_proof, HITL flow, rate limiter com circuit breaker em 60% BUC, audit log).

---

## 5 regras anti-ban

1. **Polling ≥ 5 minutos.** Nunca tight loop.
2. **Writes só em horário comercial** (8h-20h BRT).
3. **HITL obrigatório em:** criativos novos, +20%+ budget, cross-account rebalance, criar campanha.
4. **Pausar polling em 60% BUC.** Já implementado.
5. **Auditar tudo** (já implementado).

---

## Análise do caso "Claude Code permanently banned"

Multi-source (Reddit r/FacebookAds 1sbsw6c + dev.to + scand.ai + 3 LinkedIn posts + Zentric) confirmam:

**Triggers no caso banido:**
- Token mal configurado / app sem App Review
- Polling tight (queries em segundos)
- Budget changes 3am sem revisão
- Criativos AI publicados sem human review
- MCP unofficial / token de browser session

**Verificação Escola de Automação:** falha em **TODOS** os triggers (✅ App Review, ✅ System User, ✅ HITL, ✅ rate limiter, ✅ horário comercial).

---

## Envelope oficial Meta (Ad Rules Engine)

Direto da doc `developers.facebook.com/docs/marketing-api/ad-rules`:

| Automação | Status oficial Meta |
|-----------|---------------------|
| Day-parting (schedule-based, ≥1h windows) | ✅ Endossado, exemplo direto |
| Auto-pause CPA > threshold | ✅ Endossado, exemplo direto |
| Auto-scale ROAS > threshold | ⚠️ Suportado mas SEM cap recomendado (use +15% max 1x/48h) |
| Slack alerts triggered | ✅ Notifications são parte do framework |
| Daily reports | ✅ Use case canônico |
| Cross-account rebalance | ⚠️ Edge — sem regra nativa, manter HITL |
| Bulk creative upload | ⚠️ Suportado, conteúdo é o risco |

**Cadência oficial:** ad-rules engine avalia a cada 30-60min, com janelas mín de 1 hora. Copiar essa cadência = max safety.

---

## Status dos Fatos-Chave

| Fato | Fontes | Verificado? |
|------|--------|-------------|
| Meta ad-rules avalia 30-60min, mín hourly windows | Meta official + Ads Manager help | ✅ |
| Caso Claude ban = combinação (não AI puro) | Reddit + Zentric + dev.to + scand.ai + 3 LinkedIn (5+ fontes convergem) | ✅ |
| Zentric "18 meses 0 bans" | Apenas Zentric Digital | ⚠️ fonte única |
| BUC throttle threshold ≠ ban threshold | Meta docs + AdAmigo | ✅ — throttle documentado, ban threshold não existe oficialmente |

---

## Recomendações por automação

### Confiança ALTA — rodar tranquilo

- Auto-refresh dashboards (5-15min)
- Day-parting automático
- Auto-pause underperformers (cooldown 6h + min spend)
- Slack alerts
- Relatórios diários
- AI agent 24/7 propondo via Slack (HITL no write)

### Confiança MÉDIA — implementar com cuidado

- Auto-scale +X% — use cap +15% max 1x/dia, horário comercial
- Bulk creative — HITL obrigatório no conteúdo
- Cross-account rebalance — HITL obrigatório

### NÃO fazer

- Polling sub-minuto
- Auto-publish criativo sem human review
- Mudanças fora horário comercial em alta velocidade
- Retry storm em erro 613/429

---

## Contradições identificadas

- **Madgicx**: alega "Madgicx MCP é o único 2026-compliant" → biased, Purpose=vender produto deles
- **Algumas LinkedIn posts**: atribuem ban a "AI puro" → refutado pela maioria

---

## Padrão Zentric (referência operacional)

5 client accounts, 1.279 operações em 28 dias, 18 meses 0 bans (auto-reportado):

- Polling máx hourly em horário comercial → 8-10 checks/account/dia
- Sequential per-account reads (não paralelo)
- Pausar account ao chegar em 60% BUC
- Todos writes via human approval em Slack

**Caveat:** claim auto-reportado, sem audit independente. Mas metodologia é detalhada e consistente com policy oficial Meta.

---

## Fontes Avaliadas (CRAAP)

### Tier A (80-100)
- Meta Ad Rules Engine docs — 92 — `developers.facebook.com/docs/marketing-api/ad-rules`
- Meta Marketing API Rate Limiting docs — 90 — `developers.facebook.com/documentation/ads-commerce/marketing-api/overview/rate-limiting`
- Meta Ads Manager Automated Rules help — 88 — `facebook.com/business/help/542231668359704`

### Tier B (60-79)
- Reddit r/FacebookAds 1sbsw6c "Claude Code banned" — 75
- Zentric Digital "Will Claude Code Get My Meta Ads Account Banned?" — 72 — `zentric.digital/insights/claude-code-meta-ads-api-ban-risk`
- dev.to "Claude Code Got Someone's Meta Ads Account Banned" — 70
- scand.ai analysis — 68
- LinkedIn Daniel Marcas (Openclaw case) — 65

### Tier C/D (com ressalva)
- Madgicx blog — 50 — Purpose=vender, descartado pra fact
- LinkedIn posts genéricos sobre "AI ban" — 55 — anecdotal
