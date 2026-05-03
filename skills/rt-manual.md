# Manual do Simulador — Reforma Tributária

Assistente de onboarding e dúvidas operacionais do projeto de análise de impacto da Reforma Tributária. Use quando o estagiário ou usuário perguntar "como faço X?", "o que é Y?", "qual o próximo passo?" ou chamar `/rt-manual`.

---

## Como Começar uma Análise

Sempre abra o projeto pelo alias configurado:
```bash
rt
```
Ou manualmente:
```bash
cd '[pasta do projeto]'
claude
```

Depois use o comando `/rt-analise` e escolha uma das 4 modalidades abaixo.

---

## As 4 Modalidades de Entrada

### A) SPED — nível ALTO (preferencial, variação ±5%)
Use quando o cliente enviou os arquivos de escrituração fiscal.

1. Salve os arquivos na pasta do projeto:
   - `sped_fiscal.txt` (EFD-ICMS/IPI)
   - `sped_contrib.txt` (EFD-Contribuições)
2. Digite no chat:
```
Tenho os SPEDs da [Nome da empresa].
Setor: [setor]
Regime: [Lucro Real / Lucro Presumido / Simples Nacional]
UF: [estado]
Perfil: [B2B / B2C / Misto]

Arquivos salvos como sped_fiscal.txt e sped_contrib.txt. Inicia a análise.
```

### B) DRE em PDF ou planilha — nível MÉDIO (variação ±15%)
Use quando o cliente enviou a Demonstração de Resultado.

1. Salve o arquivo na pasta do projeto
2. Digite no chat:
```
Segue DRE da [Nome da empresa].
Arquivo: [nome_do_arquivo.pdf]
Setor: [setor]
Regime: [regime]
UF: [estado]
Perfil: [B2B / B2C / Misto]

Extrai os dados e inicia a análise.
```

### C) Dados manuais — nível MÉDIO (variação ±15%)
Use quando os dados chegaram por WhatsApp, e-mail ou ligação.

Preencha e cole no chat:
```
Cliente: ___
Setor: ___
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

Saldo credor ICMS (R$): ___       ← opcional
Saldo credor PIS/COFINS (R$): ___ ← opcional

Roda a análise.
```

### D) NCM — análise por produto (nível ALTO com SPED / MÉDIO manual)
Use quando quiser ver o impacto separado por produto/SKU, não só no nível empresa.

**Com SPED (extração automática):**
```python
from motor_cmv_v5_0 import ler_sped_c170_ncm, PortfolioNCM, calcular_portfolio
from motor_cmv_v4_1 import ClienteInput

inp = ClienteInput(nome="...", setor="...", regime="...", ...)
itens = ler_sped_c170_ncm("sped_fiscal.txt")
portfolio = PortfolioNCM(cliente=inp, itens=itens, fonte="sped", nivel_confianca="ALTO")
resultado = calcular_portfolio(portfolio, ano=2033)
print(resultado["relatorio"])
```

**Manual (lista de produtos):**
```python
from motor_cmv_v5_0 import ItemNCM, PortfolioNCM, calcular_portfolio

portfolio = PortfolioNCM(
    cliente=inp,
    itens=[
        ItemNCM(ncm="19021900", descricao="Massas alimentícias",
                receita_anual=3_000_000, cmv_anual=1_800_000),
        ItemNCM(ncm="22030000", descricao="Cerveja de malte",
                receita_anual=5_000_000, cmv_anual=2_800_000),
    ],
    fonte="manual",
)
resultado = calcular_portfolio(portfolio, ano=2033)
```

> **Atenção cClassTrib:** o motor avisa se a base NCM estiver desatualizada.
> - 15–30 dias sem verificar: alerta + pedido de confirmação
> - \> 30 dias: bloqueio — atualizar `motor/data/cclasstrib_status.json` antes de rodar
> - URL para verificar: https://dfe-portal.svrs.rs.gov.br/DFE/TabelaClassificacaoTributaria

---

## Os 5 Entregáveis Obrigatórios

Toda análise termina com estes 5 itens — nesta ordem:

| # | Entregável | Onde aparece |
|---|-----------|--------------|
| 1 | Relatório nos 9 tópicos | no chat |
| 2 | CRM + e-mail de follow-up + qualificação (QUENTE/MORNO/FRIO) | no chat |
| 3 | Script Python para gerar PDF | arquivo `.py` salvo |
| 4 | Deck `.md` para importar no Gamma.app | arquivo `.md` salvo |
| 5 | Plano de ação por horizonte + proposta de upsell | no chat |

Ao final, sempre salvar o relatório:
```
Salva o relatório como: relatorios/AAAA-MM-DD_Nome-Cliente_Setor.md
```

---

## Os 9 Tópicos do Relatório

1. **Perfil da empresa** — setor, regime, UF, faturamento, nível de confiança
2. **Diagnóstico atual** — DRE simplificada, carga tributária efetiva
3. **Impactos no CMV** — créditos por componente, antes vs. depois
4. **Impactos no preço de venda** — carga saída atual vs. 2033
5. **Cenários 2026–2033** — tabela ano a ano com CMV e margem
6. **Alertas e oportunidades** — ICMS entrada/saída, Simples, saldo credor
7. **Próximos passos** — ações concretas por horizonte
8. **Governança** — checklist G.1 a G.7, pontuação, pacote recomendado
9. **Ressalva técnica** — disclaimer obrigatório

---

## Regras Que Nunca Podem Ser Quebradas

1. **Nunca** afirmar alíquotas definitivas sem citar a fonte normativa
2. **Nunca** prometer economia — usar sempre "estimativa" ou "potencial impacto"
3. **Nunca** iniciar análise sem: setor + regime + referência financeira
4. **Nunca** usar a mesma alíquota ICMS para entrada e saída
5. **Nunca** assumir `% Simples Nacional = 0%` sem perguntar
6. Sempre exibir os avisos do motor antes de calcular (quando usar SPED)
7. Sempre incluir a ressalva técnica em todos os entregáveis

---

## Tabela de Transição 2026–2033 (versão v4.4 — corrigida)

> ⚠️ IBS 2027 e 2028 = **0,10% efetivo** (não 6,10% ou 12,20%). Correto per Art. 597 Res. CGIBS nº 6/2026.

| Ano | CBS | IBS | Red. ICMS | Observação |
|-----|-----|-----|-----------|------------|
| 2026 | 0,90% | 0,10% | 0% | Declaratório — não paga |
| 2027 | ~2,20% | **0,10%** | 10% | CBS começa valer de verdade |
| 2028 | ~4,40% | **0,10%** | 20% | Transição |
| 2029 | 8,80% | ~13,30%* | 30% | Transição |
| 2030 | 8,80% | ~15,50%* | 60% | Virada grande |
| 2031 | 8,80% | ~16,60%* | 80% | Quase pleno |
| 2032 | 8,80% | ~17,20%* | 90% | Pré-pleno |
| 2033 | 8,80% | ~17,70%* | 100% | **Regime pleno** |

*Estimativas — serão fixadas pelo Senado Federal

---

## Fatores Setoriais

| Setor | Fator | O que significa |
|-------|-------|----------------|
| Indústria / Comércio / Serviços | 1,00 | Alíquota plena |
| Profissionais liberais (advocacia, contabilidade, engenharia...) | 0,70 | 30% de redução |
| Saúde, Educação, Alimentos, Cultural, Esporte | 0,40 | 60% de redução |
| Agronegócio | 0,40 | 60% de redução |
| Cesta Básica | 0,00 | Alíquota zero |

---

## Perguntas Frequentes

**Q: Posso usar a mesma alíquota de ICMS para compras e vendas?**
Não. ICMS saída (débito na venda) ≠ ICMS entrada (crédito nas compras). Sempre perguntar os dois separado ao cliente.

**Q: O cliente é Simples Nacional — como analiso?**
Com cuidado. O Simples ainda não tem regulamentação definitiva na Reforma (Arts. 162–180 LC 214/2025). Nunca aplicar premissas padrão. Informar ao cliente que a análise tem limitações.

**Q: Em 2026 o cliente já paga IBS e CBS?**
Não. Em 2026 é tudo declaratório — os valores aparecem na NF mas não são pagos. A carga real em 2026 ainda é PIS/COFINS + ICMS integrais.

**Q: O que é Split Payment?**
Mecanismo obrigatório a partir de 2027 em que o IBS e CBS são recolhidos automaticamente no momento do pagamento (via Pix, boleto, cartão). O cliente não precisa recolher — mas impacta o fluxo de caixa.

**Q: O que fazer com o saldo credor de ICMS/PIS-COFINS acumulado?**
É uma oportunidade crítica. Levantar e escriturar antes de 31/12/2026. Saldos de PIS/COFINS podem ser compensados com CBS após 2027 (Art. 602 Decreto 12.955/2026).

**Q: O que é ICMS-ST?**
Substituição Tributária — o fornecedor já recolhe o ICMS de toda a cadeia. Se o cliente compra com ST, ele não tem crédito de ICMS na entrada. Impacta diretamente o cálculo do CMV.

**Q: Qual é o prazo do monitor quinzenal?**
Todo dia 1º e 15 de cada mês. Use `/rt-monitor` para rodar o protocolo de atualização normativa.

---

## Estrutura de Pastas do Projeto

```
reforma-tributaria-cmv/
├── motor/motor_cmv_v4_1.py   ← motor de cálculo Python (v4.4)
├── knowledge/                ← base normativa e changelogs
├── templates/                ← templates de relatório e deck
├── training/                 ← 6 casos didáticos para treinamento
├── comercial/                ← playbook de upsell e propostas
├── clientes/                 ← fichas de clientes (local, não sobe pro GitHub)
├── relatorios/               ← relatórios gerados (local, não sobe pro GitHub)
└── docs/manual_usuario.html  ← manual completo em HTML
```

---

## Dúvida não respondida aqui?

Digite sua dúvida diretamente no chat. Se for sobre normativo (nova lei, nova alíquota), rode `/rt-monitor` primeiro.
