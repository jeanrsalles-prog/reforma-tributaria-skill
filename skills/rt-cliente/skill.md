# Ficha de Cliente — Reforma Tributária

Cria ou abre a ficha de cadastro de um cliente no projeto, registrando dados essenciais para a análise de impacto da Reforma Tributária.

## Trigger

Usuário executa `/rt-cliente [Nome]` ou pede para cadastrar / abrir a ficha de um cliente.

## Inputs

- **Nome do cliente** (obrigatório) — razão social ou nome fantasia
- **Ação** (opcional): `novo` | `abrir` | `atualizar` (padrão: detectar automaticamente)

---

## Fluxo de Trabalho

### PASSO 1 — Verificar se Cliente Já Existe

Verificar se existe arquivo em `clientes/` com nome compatível:
```bash
ls clientes/ | grep -i "[nome]"
```

- Se encontrado → perguntar: "Encontrei [nome_arquivo]. Deseja abrir ou criar uma nova ficha?"
- Se não encontrado → criar nova ficha

### PASSO 2 — Nova Ficha

Criar arquivo `clientes/AAAA-MM-DD_[Nome-sem-espaços].md` copiando o template base e preenchendo o que for informado:

```markdown
# Ficha de Cliente — [Nome]

**Data de cadastro:** [data]
**Responsável:** [usuário]

---

## Identificação

| Campo | Valor |
|-------|-------|
| Razão Social | |
| CNPJ | |
| Setor | |
| Regime Tributário | |
| UF Principal | |
| Perfil de Venda | B2B / B2C / Misto |
| Porte | Pequeno / Médio / Grande |

---

## Dados Financeiros (último exercício)

| Campo | Valor |
|-------|-------|
| Receita Bruta | R$ |
| CMV Total | R$ |
| Margem Bruta Atual | % |
| ICMS Saída | % |
| ICMS Entrada | % |
| % Fornecedores Simples | % |
| ICMS-ST | Sim / Não |
| Saldo Credor ICMS | R$ |
| Saldo Credor PIS/COFINS | R$ |

---

## Histórico de Análises

| Data | Arquivo | Nível | Destaque |
|------|---------|-------|----------|

---

## Qualificação Comercial

| Campo | Valor |
|-------|-------|
| Pontuação T8 (Governança) | /7 |
| Qualificação | QUENTE / MORNO / FRIO |
| Pacote recomendado | |
| Próximo passo | |
| Data do próximo contato | |

---

## Notas

```

### PASSO 3 — Listar Análises Anteriores (se ficha existente)

Se a ficha já existir, mostrar:
- Data e nome de cada relatório em `relatorios/` que corresponde ao cliente
- Última qualificação registrada
- Próximo passo pendente

### PASSO 4 — Pergunta de Continuidade

Ao final, perguntar:
> "Deseja:
> - Iniciar uma análise completa para este cliente? (`/rt-analise`)
> - Apenas salvar a ficha por agora?
> - Registrar uma observação específica?"

---

## Output

Confirmar: `✅ Ficha criada/atualizada: clientes/[arquivo].md`
