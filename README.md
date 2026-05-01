# Reforma Tributária — Skill para Claude Code

Skill instalável no Claude Code com 3 comandos para diagnóstico tributário da Reforma Tributária (EC 132/2023 · LC 214/2025 · Decreto 12.955/2026 · Res. CGIBS nº 6/2026).

**Motor:** v4.4 (01/05/2026) — inclui correção crítica IBS 2027–2028 = 0,10% efetivo.

---

## Instalação

```bash
claude skills install github:jeanrsalles-prog/reforma-tributaria-skill
```

---

## Comandos

| Comando | O que faz |
|---------|-----------|
| `/rt-analise` | Diagnóstico completo de impacto no CMV e preço — gera os 5 entregáveis |
| `/rt-monitor` | Protocolo quinzenal de atualização normativa |
| `/rt-cliente [Nome]` | Cria ou abre ficha de cliente |

---

## `/rt-analise` — Modos de entrada

```
/rt-analise sped    → usa arquivos SPED Fiscal + Contribuições (nível ALTO ±5%)
/rt-analise dre     → usa DRE em PDF ou planilha (nível MÉDIO ±15%)
/rt-analise manual  → entrada manual de dados (nível MÉDIO ±15%)
```

---

## Entregáveis gerados por análise

1. Relatório nos 9 tópicos (perfil → diagnóstico → CMV → preço → cenários → alertas → próximos passos → governança → ressalva)
2. Output comercial: CRM + e-mail de follow-up + qualificação do lead
3. Script Python para geração de PDF
4. Deck `.md` para importar no Gamma.app
5. Plano de ação por horizonte + proposta de upsell

---

## Base normativa embutida

- Tabela de transição 2026–2033 **corrigida** (IBS 2027–2028 = 0,10% per Art. 597 Res. CGIBS nº 6/2026)
- Fatores setoriais atualizados (profissionais liberais 0,70 · saúde/educação 0,40 · cesta básica 0,00)
- 7 regras de conduta invioláveis
- Protocolo de atualização normativa quinzenal

---

> Esta skill tem caráter diagnóstico e estimativo. Não substitui parecer jurídico ou contábil formal.
