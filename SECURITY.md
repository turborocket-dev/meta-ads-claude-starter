# Security

## Reportando vulnerabilidades

Se encontrar uma vulnerabilidade de segurança, **NÃO abra Issue público**. Mande email pra `thales@laray.com.br` com:
- Descrição do problema
- Passos pra reproduzir
- Impacto estimado

Resposta em até 48h.

## Modelo de ameaças deste repo

Este é um starter educacional pra alunos leigos rodarem MCP + Claude Code com Meta Ads. As principais ameaças que mitigamos:

| Ameaça | Mitigação |
|--------|-----------|
| Token vaza no bash history | `setup.sh` usa `read -s -p` (silent) |
| Token commitado por engano | `.gitignore` cobre `.env` + `.env.local` |
| App secret compartilhado entre alunos | Warning explícito em `.env.example` e README |
| Supply chain via deps comprometidas | `uv.lock` pinado; CI roda `pip-audit` weekly |
| FastMCP CVE-2026-32871 (SSRF) | `pyproject.toml` pin `>=3.2.4` (patched) |
| Claude Code CVE-2025-59536 (RCE via project files) | Pré-req documentado: Claude Code ≥ 2.0.65 |

## Boas práticas pra quem usa este repo

### Antes de rodar `claude` na pasta

1. **NUNCA clone fork não-oficial** — só `github.com/thaleslaray/meta-ads-claude-starter`
2. **Verifique sua versão de Claude Code:** `claude --version` deve ser ≥ 2.0.65
3. **Audite os arquivos `.mcp.json`, `CLAUDE.md` e `.claude/skills/`** antes de cada `git pull` se vier de PR de terceiros

### Tokens

- **Nunca** cole `META_ACCESS_TOKEN` em chat público (Slack, Discord, Issues, prints)
- Se vazou, **revogue imediatamente** em business.facebook.com → System Users → seu user → Tokens
- Use **conta Meta de teste** com cap de gasto $0 nos primeiros experimentos

### Code Mode (FastMCP)

O MCP usa `MontySandboxProvider` (Pydantic Monty) pra executar código gerado pelo LLM. O sandbox tem:
- ✅ Filesystem isolation (default-deny — não lê fora do diretório)
- ✅ Network isolation (sem requests HTTP arbitrários)
- ✅ Memory + CPU caps configuráveis
- ✅ Bug bounty $5k pela Pydantic pra quem quebrar

## CVEs conhecidos no stack

| CVE | Componente | Versão patched | Como verificamos |
|-----|------------|----------------|------------------|
| CVE-2026-32871 | FastMCP < 3.2.0 | ≥ 3.2.4 | `pyproject.toml` pin |
| CVE-2025-59536 | Claude Code < 1.0.111 | ≥ 2.0.65 (recomendado) | Pré-req README |
| CVE-2026-21852 | Claude Code (env override) | ≥ 2.0.65 | Pré-req README |
| CVE-2025-69196 | FastMCP OAuth | ≥ 3.2.0 | Coberto pelo pin acima |

## Auditorias passadas

- **2026-04-28** — Auditoria adversarial completa (`docs/auditoria-adversarial-2026-04-28.md`). Achados: 7 issues, todos corrigidos no v1.1.0.

## Contato

- GitHub Issues (vulnerabilidades NÃO sensíveis)
- thales@laray.com.br (vulnerabilidades sensíveis)
