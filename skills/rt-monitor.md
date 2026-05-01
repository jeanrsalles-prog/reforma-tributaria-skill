# Monitor Quinzenal — Reforma Tributária

Executa o protocolo de atualização normativa quinzenal: busca novidades nas fontes oficiais, compara com o CHANGELOG do projeto e propõe atualizações nos arquivos afetados.

## Trigger

Usuário executa `/rt-monitor` ou pede para rodar o monitoramento / atualização normativa da Reforma Tributária.

## Inputs

Nenhum input obrigatório. Opcionalmente o usuário pode informar:
- Data de referência (padrão: hoje)
- Foco específico (ex: "só IBS", "só Simples Nacional")

---

## Fluxo de Trabalho

### PASSO 1 — Busca nas Fontes Oficiais

Executar buscas web nas seguintes fontes, nesta ordem de prioridade:

**Fontes primárias (gov.br — confiança total):**
- `site:gov.br/receitafederal reforma tributária CBS IBS 2026`
- `site:comitegestor.gov.br resolução portaria`
- `site:fazenda.gov.br reforma tributária nota`
- `site:planalto.gov.br lei complementar 2026`
- `site:in.gov.br CBS IBS reforma tributária`

**Portal de análise (interpretação — referenciar com cautela):**
- `site:reformatributaria.com` — noticiário e análises

### PASSO 2 — Parâmetros Críticos a Verificar

Para cada parâmetro, indicar: **SEM ALTERAÇÃO** | **ATUALIZADO** | **PENDENTE**

| # | Parâmetro | Referência atual | Status |
|---|-----------|-----------------|--------|
| 1 | Alíquota CBS de referência 2027 | ~2,20% (estimativa) | Pendente — RFB envia ao TCU até 31/07/2026 |
| 2 | Alíquota IBS efetivo 2027–2028 | **0,10%** (Art. 597 Res. CGIBS nº 6) | ✅ Correto |
| 3 | Alíquotas IBS referência 2029–2033 | Estimativas 13,30%–17,70% | Pendente — CGIBS/Senado |
| 4 | Fator CBS regime pleno | 8,80% | ✅ Correto |
| 5 | Regulamento do Imposto Seletivo | Pendente (SRESP/RFB) | 🔄 Pendente |
| 6 | Portaria Split Payment operacional | Pendente (RFB + CGIBS) | 🔄 Pendente |
| 7 | Simples Nacional — regulamentação CG | Arts. 162–180 LC 214/2025 | 🔄 Pendente |
| 8 | Fator setorial profissionais liberais | 0,70 (Art. 202 Decreto 12.955) | ✅ Correto |
| 9 | Fator setorial saúde/educação | 0,40 (Art. 203/204 Decreto 12.955) | ✅ Correto |
| 10 | Novos atos CGIBS ou RFB | — | Verificar |

### PASSO 3 — Identificação de Novidades

Para cada novidade encontrada:
- Nome do ato normativo, data, fonte (DOU/portal oficial)
- Resumo do impacto (máx. 3 linhas)
- Classificação: **CRÍTICO** (altera motor/cálculo) | **RELEVANTE** (altera premissas) | **INFORMATIVO**
- SLA: CRÍTICO = atualizar em 48h | RELEVANTE = 72h | INFORMATIVO = próximo ciclo

### PASSO 4 — Proposta de Atualização

Para cada novidade CRÍTICA ou RELEVANTE, indicar:

| Arquivo a atualizar | Seção | O que mudar |
|--------------------|-------|-------------|
| `knowledge/CHANGELOG_NORMATIVO.md` | Entrada nova | Registrar o ato |
| `knowledge/02_cronograma_transicao.md` | Tabela de anos | Se alíquota mudar |
| `motor/motor_cmv_v4_1.py` | `TRANSICAO` dict | Se alíquota mudar |
| `CLAUDE.md` | Seção 7 ou 6 | Se fator/tabela mudar |

Perguntar ao usuário: *"Encontrei [N] novidade(s). Deseja que eu atualize os arquivos agora?"*

Só atualizar após confirmação explícita.

### PASSO 5 — Registro no CHANGELOG

Após confirmação, adicionar entrada em `knowledge/CHANGELOG_NORMATIVO.md`:

```markdown
### DD/MM/AAAA — [Nome do Ato] ⭐ NOVO
- **Fonte:** [DOU/portal] | [data de publicação]
- **Escopo:** [resumo]
- **Impacto no motor:** [o que muda, se mudar]
- **Knowledge:** [arquivo atualizado, se aplicável]
```

---

## Output

Ao final, exibir resumo:

```
📋 MONITOR QUINZENAL — [data]
──────────────────────────────
Fontes verificadas: [N]
Novidades encontradas: [N]
  CRÍTICO: [N] | RELEVANTE: [N] | INFORMATIVO: [N]

Próximos eventos:
  • [data]: [evento esperado]
  • [data]: [evento esperado]

Próxima rodada: [data + 15 dias]
```
