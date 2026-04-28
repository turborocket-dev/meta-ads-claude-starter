# Pesquisa Adversarial: Auditoria de segurança meta-ads-claude-starter

**Gerado:** 2026-04-28
**Repo auditado:** github.com/thaleslaray/meta-ads-claude-starter (commit f08ba5a)
**Contexto:** Boilerplate pra alunos leigos passarem pelo Meta App Review
**Confiança:** ALTA — todos fatos-chave verificados em ≥2 fontes Tier A

---

## Resumo Executivo

7 issues confirmados. 3 falsos alarms eliminados via Nível 3. Top 5 fixes implementáveis em ~20min.

**Falsos alarms eliminados:**
- ❌ MontySandbox NÃO lê `.env` (filesystem isolation default-deny + $5k bug bounty)
- ❌ `prefab-ui` é legítima (Jeremiah Lowin, Apache 2.0)
- ❌ CVE-2025-59536 só via repo malicioso, não prompt injection (nosso repo oficial OK)

---

## Status dos Fatos-Chave (4/4 verificados)

| Fato | Status | Fontes |
|------|--------|--------|
| FastMCP no uv.lock = 3.1.0 (vulnerável a CVE-2026-32871) | ✅ confirmado local | uv.lock direto |
| MontySandbox tem filesystem isolation completa | ✅ confirmado | Pydantic docs + Mintlify + Simon Willison blog |
| prefab-ui é dep legítima | ✅ confirmado | PyPI metadata (author Jeremiah Lowin) |
| CVE-2025-59536 requer attacker-controlled repo | ✅ confirmado | CheckPoint + Tenable + NVD |

---

## Riscos confirmados (em ordem de prioridade)

### P0 — `setup.sh` ecoa token no terminal

**Atual:**
```bash
read -r -p "META_ACCESS_TOKEN: " META_ACCESS_TOKEN
```

**Problema:** Token aparece em `~/.bash_history`, scrollback do terminal, GitHub Codespaces logs.

**Fix:**
```bash
read -r -s -p "META_ACCESS_TOKEN: " META_ACCESS_TOKEN
echo ""  # newline depois do silent input
```

**Severidade:** CRITICAL. **Probabilidade:** HIGH (passive leak, todo aluno).

---

### P1 — `pyproject.toml` permite FastMCP vulnerável

**Atual:** `fastmcp[code-mode]>=3.1.0`
**uv.lock resolve:** `3.1.0` (vulnerável a CVE-2026-32871)

**Fix:** atualizar pra `fastmcp[code-mode]>=3.2.4`

**Severidade:** HIGH. **Aplicabilidade:** baixa (não usamos OpenAPIProvider), mas defense-in-depth.

---

### P1 — Doc: alunos precisam Claude Code ≥ 2.0.65

**Problema:** CVE-2025-59536 (RCE via project files) afeta < 1.0.111. Recomendado 2.0.65+ pra cobrir CVE-2026-21852 também.

**Fix:** Adicionar pré-requisito explícito no README + check no `setup.sh`.

---

### P2 — Bug funcional em `audit.py:get_write_count_today()`

**Atual:** conta TODAS as linhas do JSONL (não filtra `kind`).
**Impacto:** rate limit de writes pode ser computado errado quando audit log enche de reads.

**Fix:** Adicionar `if entry.get("kind") == "write"` no loop.

---

### P2 — Cada aluno deve criar SEU próprio app Meta

**Problema:** Caso Cas Smith — agência banida porque MCP third-party usava app SHARED. Se vários alunos compartilham mesmo App ID/Secret, banimento de 1 = banimento de todos.

**Fix:** Warning explícito no README + `.env.example`.

---

### P2 — Sem CI audit de deps (TeamPCP risk)

**Contexto:** 5 supply chain attacks em 12 dias (mar/2026): Trivy, LiteLLM, Telnyx, Axios, Checkmarx.

**Fix:** GitHub Action com `pip-audit` rodando weekly + on PR.

---

### P2 — Deps desnecessárias (`streamlit`, +100MB)

**Atual:** `streamlit>=1.56.0`, `prefab-ui>=0.10.0` no pyproject. MCP server core não usa nenhuma das 2.

**Impacto:** ~100-150MB de download extra pra cada aluno na primeira instalação.

**Fix:** Remover do `pyproject.toml` (verificar primeiro que não quebra).

---

## Riscos eliminados pelo Nível 3

| Risco do Nível 2 | Status final |
|------------------|--------------|
| MontySandbox lê `.env` via `open("../.env")` | ❌ Falso — sandbox isolation default-deny |
| `prefab-ui` é supply chain risk | ❌ Falso — author legítimo |
| CVE-2025-59536 explora MCP responses | ❌ Falso — só via repo files attacker-controlled |

---

## Aceitáveis (não dão pra fixar sem quebrar simplicidade)

| Risco | Por quê | Mitigação documental |
|-------|---------|---------------------|
| `CLAUDE.md` é writable system prompt | Mecânica core do Claude Code | "Nunca rode `claude` em fork sem auditar" |
| `.env` no mesmo tree do MCP | Padrão da indústria | Restringir filesystem sandbox a `./output/` |
| System User token tem `ads_management` (write scope) | Necessário pra warmup skill | Recomendar conta de teste com cap $0 |

---

## Fontes Avaliadas (CRAAP)

### Tier A (80-100)
- CheckPoint Research CVE blog — 92
- Tenable CVE-2025-59536 detail — 90
- gofastmcp.com docs — 88
- Pydantic Monty docs (Mintlify) — 88
- pypistats.org — 95

### Tier B (60-79)
- MintMCP CVE summary — 75
- TheHackerNews CVE article — 72
- ThreatLandscape analysis — 70
- AdAmigo automation/ban article — 68

### Tier C (descartados)
- LinkedIn posts genéricos sobre AI ad tools

---

## Próximos passos (fixes em ordem)

1. `setup.sh` → `read -s -p`
2. `pyproject.toml` → pin `fastmcp[code-mode]>=3.2.4`
3. `pyproject.toml` → remover `streamlit`, `prefab-ui` (se não quebrar)
4. `audit.py` → fix counter bug
5. README + `.env.example` → warnings
6. `.github/workflows/audit.yml` → pip-audit
7. `SECURITY.md` → documentar achados + recomendações

Tempo estimado total: ~25min.
