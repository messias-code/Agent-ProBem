# Agent-ProBem 🎓🤖

**Automação de Extração (ETL) e Auditoria de Contratos Educacionais**

Este repositório contém as definições de comportamento (System Prompts) para dois agentes de Inteligência Artificial especializados, desenhados para o ecossistema do programa **ProBem/OVG**. O sistema atua em duas frentes: digitalização de dados acadêmicos (RIAF) e validação jurídica/financeira de contratos.

---

## 📂 Estrutura do Projeto

```text
Agent-ProBem/
│
├── RIAF/
│   └── prompt.yml       # Agente Engenheiro de Dados (Extração & Normalização)
│
└── CONTRATOS/
    └── prompt.yml       # Agente Auditor Sênior (Validação Lógica & Matemática)

```

---

## 🤖 Agentes e Responsabilidades

### 1. Agente RIAF (Resumo de Informações Acadêmicas e Financeiras)

**Papel:** Engenheiro de Dados Sênior (OCR e ETL).
**Objetivo:** Transformar documentos RIAF não estruturados em dados JSON limpos e tipados.

* **Entrada:** Texto bruto (OCR) do documento RIAF.
* **Saída:** JSON estruturado (Schema: Instituição, Aluno, Acadêmico, Financeiro, Assinatura).
* **Principais Regras de Negócio:**
* **Sanitização Monetária:** Converte "R$ 1.200,50" para `1200.50` (Float).
* **Tratamento de Nulos:** Campos vazios retornam `null`, nunca dados inventados.
* **Identificação de Assinaturas:** Detecção booleana de presença de ranhuras/texto nos campos de assinatura.



### 2. Agente CONTRATOS (Auditoria)

**Papel:** Auditor Sênior de Contratos Educacionais.
**Objetivo:** Validar se o contrato assinado (PDF) corresponde matematicamente e logicamente aos dados esperados no sistema (Input).

* **Entrada:** Texto do Contrato + Dados do Sistema (YAML/JSON com CPF, Mensalidades, Semestre).
* **Saída:** JSON de validação com `status: valido/invalido` e diagnósticos do catálogo.
* **As 4 Mentalidades de Execução:**
1. **Identidade:** Validação crítica de CPF (Bloqueante).
2. **Temporal:** Dedução de semestre letivo baseada em datas de vigência/assinatura.
3. **Financeira:** Cálculo reverso de parcelas e aplicação de descontos.
4. **Diagnóstico:** Uso estrito de um **Catálogo de Erros** (sem alucinação de mensagens).



---

## ⚙️ Regras Críticas de Implementação

### Regra de Precisão Decimal (Agente Contratos)

Para evitar falsos negativos em validações financeiras, o agente de contratos opera com uma regra estrita de **Truncagem** (não arredondamento):

> *Regra:* Considere estritamente as DUAS primeiras casas decimais.
> *Exemplo:* `1200.559` torna-se `1200.55`. O sistema não arredonda para cima.

### Regra de Supressão de Erros

A validação ocorre em cascata. Se um erro de nível **CRÍTICO** (CPF) for detectado:

1. A análise é interrompida imediatamente.
2. Os campos de semestre e mensalidade **não** são avaliados.
3. O JSON retorna apenas o erro de CPF para economizar tokens e processamento.

---

## 🚀 Como Usar

### Exemplo de Input para o Agente de Contratos

Ao chamar o LLM com o prompt `CONTRATOS/prompt.yml`, você deve injetar os dados esperados no topo ou no corpo da mensagem do usuário:

```yaml
# DADOS DO SISTEMA (Variáveis a serem substituídas)
dados_esperados:
  cpf: '123.456.789-00'
  mensalidade_sem_desconto: '1500.00'
  mensalidade_com_desconto: '750.00' # Se houver bolsa
  semestre: '2026/1'

```

### Exemplo de Resposta (Output JSON)

O sistema retornará estritamente um JSON:

```json
{
  "cpf": { "conteudo": "12345678900", "valido": 1 },
  "mensalidade_sem_desconto": { "conteudo": "1500.00", "valido": 1 },
  "mensalidade_com_desconto": { "conteudo": "750.00", "valido": 1 },
  "semestre": { "conteudo": "2026/1", "valido": 1 },
  "inconsistencias": { "conteudo": "Sem inconsistências", "valido": 1 },
  "status": { "conteudo": "valido", "valido": 1 }
}

```

---

## 🛠 Manutenção do Catálogo de Erros

As mensagens de erro no `CONTRATOS/prompt.yml` são **hardcoded**. Se for necessário adicionar novos tipos de erro, eles devem ser inseridos na seção `catalogo_erros` do arquivo YAML. O Agente é proibido de inventar novas frases para garantir padronização nos relatórios de auditoria.
