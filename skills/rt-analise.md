# Análise de Impacto — Reforma Tributária

Realiza o diagnóstico completo do impacto da Reforma Tributária (EC 132/2023 · LC 214/2025 · LC 227/2026 · Decreto 12.955/2026 · Res. CGIBS nº 6/2026) sobre o CMV e a precificação de uma empresa, gerando os 5 entregáveis obrigatórios.

## Trigger

Usuário executa `/rt-analise` ou pede uma análise de impacto da Reforma Tributária sobre uma empresa específica.

## Inputs

Identificar qual modalidade de entrada está disponível:

**Modalidade SPED (nível ALTO — preferencial):**
- Caminhos dos arquivos `sped_fiscal.txt` e `sped_contrib.txt` salvos localmente
- Setor, regime tributário, UF, perfil (B2B/B2C/Misto)

**Modalidade NCM — análise produto a produto (nível ALTO com SPED / MÉDIO manual):**
- Lista de produtos com NCM (8 dígitos), receita anual e CMV por produto
- OU SPED Fiscal para extração automática dos NCMs via C170
- Usar `motor/motor_cmv_v5_0.py` — veja fluxo NCM abaixo

**Modalidade DRE (nível MÉDIO):**
- Arquivo PDF ou planilha com DRE
- Setor, regime tributário, UF, perfil

**Modalidade Manual (nível MÉDIO):**
Se não houver SPED nem DRE, coletar obrigatoriamente:
```
Cliente: ___
Setor: ___  (ver Setores abaixo)
Regime: Lucro Real / Lucro Presumido / Simples Nacional
UF principal: ___
Perfil de venda: B2B / B2C / Misto
Margem desejada (%): ___

ICMS saída — alíquota nas vendas (%): ___
ICMS entrada — alíquota média das compras (%): ___
% CMV de fornecedores Simples Nacional: ___
Há ICMS-ST relevante? Sim / Não

Receita Bruta (R$): ___
CMV Total (R$): ___
  Matérias-primas: ___
  Frete CIF: ___
  Energia elétrica: ___
  Serviços contratados: ___
  Embalagens: ___
  Mão de obra direta: ___
  Outros: ___

Saldo credor ICMS acumulado (R$): ___      ← opcional
Saldo credor PIS/COFINS acumulado (R$): ___  ← opcional
```

Se algum campo crítico não for informado, **perguntar antes de prosseguir**. Nunca iniciar cálculo com inputs incompletos.

---

## Fluxo de Trabalho

### ETAPA 0 — Atualização Normativa
Antes de qualquer cálculo, buscar atualizações recentes:
- `site:gov.br alíquotas IBS CBS Comitê Gestor`
- `LC 214 regulamentação complementar 2026`
- `comitegestor.gov.br portaria resolução`

Se encontrar algo posterior à **Res. CGIBS nº 6/2026 (30/04/2026)**, informar ao usuário antes de prosseguir.

### ETAPA 1 — Extração e Estruturação dos Dados
- Identificar receita bruta, CMV total e composição
- Separar ICMS saída ≠ ICMS entrada (crítico — nunca usar a mesma alíquota para os dois)
- Mapear perfil da cadeia: fornecedores Simples, ST, importação
- Calcular carga tributária efetiva atual

**Se SPED disponível**, rodar o motor Python:
```bash
cd [pasta_do_projeto]
python3 motor/motor_cmv_v4_1.py   # análise empresa (v4.4)
# OU, para análise por produto:
python3 motor/motor_cmv_v5_0.py   # análise NCM (v5.0)
```

O motor extrai automaticamente do SPED: alíquotas ICMS entrada/saída, flag ST, % Simples, saldo credor PIS/COFINS.

Exibir avisos extraídos pelo motor **antes de calcular** e confirmar com o usuário.

**Se análise por NCM (motor v5.0):**
```python
from motor_cmv_v5_0 import ItemNCM, PortfolioNCM, calcular_portfolio, ler_sped_c170_ncm
from motor_cmv_v4_1 import ClienteInput

# Com SPED:
itens = ler_sped_c170_ncm("sped_fiscal.txt")
portfolio = PortfolioNCM(cliente=inp, itens=itens, fonte="sped", nivel_confianca="ALTO")

# Ou manual:
portfolio = PortfolioNCM(
    cliente=inp,
    itens=[
        ItemNCM(ncm="19021900", descricao="Massas", receita_anual=3_000_000, cmv_anual=1_800_000),
        ItemNCM(ncm="22030000", descricao="Cerveja", receita_anual=5_000_000, cmv_anual=2_800_000),
    ],
    fonte="manual",
)

resultado = calcular_portfolio(portfolio, ano=2033)
print(resultado["relatorio"])
```

O motor v5.0 verifica automaticamente a atualização da base cClassTrib:
- ≤ 15 dias: silencioso
- 16–30 dias: alerta + confirmação
- > 30 dias: bloqueio (use `forcar_cclasstrib=True` apenas em emergência)

### ETAPA 2 — Cálculo de Impactos
O motor calcula para cada componente do CMV:

**Créditos atuais:**
- PIS/COFINS: 9,25% (Lucro Real) | 0% (Lucro Presumido — regime cumulativo, Lei 9.718/98) | 0% (Simples)
- ICMS entrada: usar `aliq_icms_entrada` (não saída). Zero se ICMS-ST. Zero em energia (LC 87/96 art. 33).

**Créditos futuros IVA:**
- Conservador: `65% × fatorSetor × fatorPerfil × (1 − pct_simples)`
- Otimista: `90% × fatorSetor × fatorPerfil × (1 − pct_simples)`

**Preço de venda** — IVA "por dentro" (art. 11 LC 214/2025):
- Carga saída atual: PIS/COFINS + ICMS saída
- Carga futura 2033: IVA × fatorSetor × fatorPerfil
- Novo preço = custo / (1 − carga_saída_nova)

### ETAPA 3 — Tabela de Transição 2026–2033

> ⚠️ IBS 2027 e 2028 = **0,10% efetivo** (não 6,10%/12,20%). Correto per Art. 597 Res. CGIBS nº 6/2026.
> IBS 2029–2033 = estimativas pendentes de fixação pelo Senado Federal.

| Ano | CBS | IBS (efetivo) | Red. ICMS | Red. ISS | Status |
|-----|-----|---------------|-----------|----------|--------|
| 2026 | 0,90% | 0,10% | 0% | 0% | Declaratório — sem recolhimento |
| 2027 | ~2,20% | **0,10%** | 10% | 10% | Início da transição |
| 2028 | ~4,40% | **0,10%** | 20% | 20% | Transição |
| 2029 | 8,80% | ~13,30%* | 30% | 30% | Transição |
| 2030 | 8,80% | ~15,50%* | 60% | 60% | Transição acelerada |
| 2031 | 8,80% | ~16,60%* | 80% | 80% | Quase pleno |
| 2032 | 8,80% | ~17,20%* | 90% | 90% | Pré-pleno |
| 2033 | 8,80% | ~17,70%* | 100% | 100% | **Regime pleno** |

*Estimativas — Senado Federal fixa anualmente (CGIBS submete ao TCU até 31/jul)

### ETAPA 4 — Síntese nos 9 Tópicos Obrigatórios

Gerar o relatório completo seguindo esta estrutura:

**T1. PERFIL DA EMPRESA** — setor, regime, UF, perfil, faturamento, nível de confiança

**T2. DIAGNÓSTICO ATUAL** — DRE simplificada, carga tributária efetiva, margem bruta atual

**T3. IMPACTOS NO CMV** — créditos por componente (atual vs. conservador vs. otimista), tabela com R$ e %

**T4. IMPACTOS NO PREÇO DE VENDA** — carga saída atual vs. 2033, novo preço necessário, variação %

**T5. CENÁRIOS 2026–2033** — tabela ano a ano com CMV efetivo e margem projetada

**T6. ALERTAS E OPORTUNIDADES** — diferencial ICMS entrada/saída, fornecedores Simples, ST, saldo credor, IS se aplicável

**T7. PRÓXIMOS PASSOS** — ações concretas por horizonte (imediato / 6 meses / 2027)

**T8. GOVERNANÇA — MATRIZ DE MATURIDADE**
Checklist G.1 a G.7 (Sim/Parcial/Não):
- G.1 Mapeamento de créditos acumulados (ICMS + PIS/COFINS)
- G.2 ERP habilitado para CBS/IBS
- G.3 Contratos com cláusula de reequilíbrio tributário
- G.4 Política de reprecificação definida
- G.5 Cadeia de fornecedores mapeada (Simples vs. regime regular)
- G.6 Time treinado na nova sistemática
- G.7 Comitê ou responsável interno para monitoramento

Pontuação → qualificação: 5+ Não = **Pacote Parceria Estratégica** | 3–4 Não = **Pacote Gestão** | 0–2 Não = **Monitoramento**

**T9. RESSALVA TÉCNICA**
> *"Esta análise tem caráter diagnóstico e estimativo, baseada nos dados fornecidos e na legislação vigente até [data]. Não substitui parecer jurídico formal. A validação jurídica e contábil deve ser realizada pelo time técnico responsável."*

---

## Entregáveis Obrigatórios (5)

Nenhuma análise está concluída sem os 5 passos abaixo, nesta ordem:

**PASSO 1 — Relatório nos 9 tópicos** *(no chat)*

**PASSO 2 — Output Comercial** *(no chat)*
- Registro CRM: nome, setor, regime, pontuação T8, potencial (R$), próximo passo
- E-mail de follow-up: assunto + corpo (3 parágrafos, tom direto)
- Qualificação: QUENTE / MORNO / FRIO com justificativa

**PASSO 3 — Script Python para PDF** *(arquivo .py)*
Script que gera PDF do relatório usando `reportlab` ou `fpdf2`. Salvar como `relatorios/[data]_[cliente]_pdf.py`.

**PASSO 4 — Deck Gamma** *(arquivo .md)*
Estrutura de 9 slides em markdown para importar no Gamma.app. Salvar como `relatorios/[data]_[cliente]_deck.md`.

**PASSO 5 — Plano de Ação + Proposta de Upsell** *(no chat)*
- Horizonte 1 (0–90 dias): ações imediatas
- Horizonte 2 (90 dias–2027): preparação
- Horizonte 3 (2027+): acompanhamento contínuo
- Pacote de upsell recomendado (baseado na pontuação T8)

Ao final: `Salva o relatório como: relatorios/AAAA-MM-DD_Nome-Cliente_Setor.md`

---

## Referência Rápida

### Fatores Setoriais (LC 214/2025 + Decreto 12.955/2026)

| Setor | Fator | Base legal |
|-------|-------|------------|
| Indústria / Comércio / Serviços gerais | 1,00 | — |
| **Profissionais liberais** (advocacia, contabilidade, engenharia...) | **0,70** | Art. 202 Decreto 12.955/2026 |
| Agronegócio | 0,40 | Art. 212 Decreto 12.955/2026 |
| Saúde | 0,40 | Art. 205–208 Decreto 12.955/2026 |
| Educação | 0,40 | Art. 204 Decreto 12.955/2026 |
| Alimentos (consumo humano) | 0,40 | Art. 210 Decreto 12.955/2026 |
| Cultural / Esporte | 0,40 | Art. 203 Decreto 12.955/2026 |
| Cesta Básica | 0,00 | Art. 199 Decreto 12.955/2026 |

### Fatores por Perfil de Venda

| Perfil | Fator |
|--------|-------|
| B2B | 1,000 |
| B2C | 0,850 |
| Misto | 0,925 |

### Premissas Regime Pleno 2033

| Tributo | Alíquota |
|---------|----------|
| CBS | 8,80% (definitiva) |
| IBS | ~17,70% (estimativa — Comitê Gestor) |
| IVA Total | ~26,50% |

---

## Regras de Conduta (Invioláveis)

1. **NUNCA** afirmar alíquotas definitivas sem indicar a fonte normativa
2. **NUNCA** prometer economia ou ganho — usar "estimativa" ou "potencial impacto"
3. **NUNCA** iniciar análise sem: setor + regime + referência financeira
4. **NUNCA** usar mesma alíquota ICMS para entrada e saída sem confirmar
5. **NUNCA** assumir `pct_simples = 0%` sem perguntar
6. Se pedirem parecer jurídico formal: *"Posso estruturar hipóteses; a análise formal cabe ao time tributário responsável."*
7. SPED fornecido → exibir avisos do motor **antes** de calcular e confirmar com o usuário

---

## Output

Ao concluir os 5 entregáveis, perguntar:

> "Deseja que eu:
> - Aprofunde algum tópico específico?
> - Ajuste os cenários com premissas diferentes?
> - Prepare a apresentação executiva para a diretoria?
> - Inicie a análise do próximo cliente?"
