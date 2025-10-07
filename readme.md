# README - Prompt de Auditoria de Contratos ProBem

## 📋 Visão Geral

Este repositório contém um prompt YAML otimizado para auditoria automatizada de contratos educacionais do **Programa Universitário do Bem (ProBem)** da OVG. O sistema utiliza IA para extrair, validar e analisar informações contratuais de forma padronizada e confiável.

## 🎯 Objetivo

O prompt orienta uma IA a atuar como um **auditor especialista** capaz de:

- ✅ Extrair dados críticos dos contratos (CPF, mensalidades, semestre)
- 🔍 Validar informações contra dados esperados do sistema
- 📊 Gerar relatórios JSON padronizados
- 🚨 Identificar inconsistências automaticamente
- ⚡ Aplicar regras de validação rigorosas

## 🚀 Evolução do Sistema: JSON → YAML

### 📑 Proposta de Aprimoramento do Sistema de Extração de Dados por IA

Esta seção apresenta um comparativo entre o **formato JSON original** e a **nova proposta em YAML** para configuração e orientação do sistema de extração de dados via Inteligência Artificial.

O objetivo das mudanças é aumentar a **clareza, rastreabilidade, robustez e imparcialidade** do sistema, reduzindo ambiguidades e melhorando a capacidade da IA de seguir instruções de forma consistente.

### 🔄 A Importância da Estrutura do Prompt

**O que é um prompt?**  
A interface de comunicação com a IA. É a ponte entre a nossa intenção e a ação da máquina.

**Por que a estrutura importa?**  
Impacto direto na precisão, eficiência e manutenibilidade. Um prompt bem estruturado leva a respostas precisas e confiáveis.

**JSON vs. YAML:**  
Uma comparação entre dois formatos para estruturar prompts: JSON, orientado para máquinas, e YAML, focado em legibilidade humana.

### 📊 Análise Comparativa Detalhada

#### 1. Estrutura e Organização

| Critério | JSON Original | YAML Proposto |
|----------|---------------|---------------|
| **Estrutura** | Configurações em objetos aninhados e densos | Hierarquia clara, indentação intuitiva |
| **Modularidade** | Regras concentradas em blocos extensos | Seções independentes e reutilizáveis |
| **Manutenção** | Complexidade alta em regras longas | Clareza em camadas (condição, passos, exemplos) |

**Exemplo - Early Exit por CPF:**

**JSON Original:**
```json
"bloqueio_por_cpf": {
  "regra": "Se o CPF no contrato for divergente do esperado ou não for localizado: 1) o campo 'inconsistencias.conteudo' deve conter apenas o motivo relacionado ao CPF; 2) o 'status.conteudo' deve ser 'invalido'; 3) o 'conteudo' de TODOS os outros campos deve ser uma string vazia; 4) Nenhuma outra análise deve ser realizada."
}
```

**YAML Proposto:**
```yaml
# ====================================================
# EARLY EXIT — BLOQUEIO POR CPF
# ====================================================
# REGRA DE PRIORIDADE MÁXIMA: se o CPF não bater, PARE TUDO.
# Pergunta humana: "O CPF do contrato confere com o esperado?"
bloqueio_por_cpf:
  condicao: 'Se CPF divergente do esperado OU CPF nao localizado no contrato'
  acao_passos:
    1: 'PARE imediatamente toda a análise (nenhuma outra regra dever ser aplicada).'
    2: 'inconsistencias.conteudo = ''CPF nao localizado no contrato'' OU ''CPF do contrato diverge do sistema'''
    3: 'status.conteudo = ''invalido'' ; status.valido = 0'
    4: 'TODOS os outros campos .conteudo = '' (string vazia) e .valido = 0'
  observacao: 'Este é um Early Exit — garante comportamento determinístico em casos críticos.'
```

✅ **Benefício**: O YAML transforma texto denso em uma sequência de passos rastreáveis, reduzindo ambiguidades.

#### 2. Clareza e Legibilidade

| Critério | JSON Original | YAML Proposto |
|----------|---------------|---------------|
| **Linguagem** | Técnica, compacta e densa | Mais próxima da linguagem natural |
| **Instruções** | Regras em lista linear | Passos numerados e comentados |
| **Documentação** | Pouca descrição inline | Perguntas humanas e notas explicativas |

**Exemplo - Validação de Semestre:**

**JSON Original:**
```json
"semestre": {
  "tipo": "obrigatorio",
  "esperado": "(semestre)",
  "regras": [
    "Extraia o semestre no formato AAAA/1 ou AAAA/2.",
    "**Se ausente, tente deduzir pela data de assinatura ou vigência do contrato. Se a data estiver em um período ambíguo (ex: final de um semestre), e não houver outra menção, considere o semestre como 'nao localizado' para evitar erros.**"
  ]
}
```

**YAML Proposto:**
```yaml
semestre:
# Pergunta humana: "Qual semestre este contrato vale? (AAAA/1 ou AAAA/2)"
  tipo: obrigatorio
  esperado: '(semestre)'
  passos:
    1: 'Extraia semestre no formato AAAA/1 ou AAAA/2.'
    2: 'Se ausente, tente deduzir pela data de assinatura/vigência (Jan-Jun => /1, Jul-Dez => /2).'
    3: 'Se a dedução for ambígua (ex.: final de período) e não houver outra menção clara → conteudo = ''nao localizado''.'
```

✅ **Benefício**: O YAML explicita a intenção com perguntas humanas e deixa claro como agir em cenários ambíguos.

#### 3. Novas Funcionalidades Introduzidas

| Funcionalidade | Existência no JSON | Implementação no YAML |
|----------------|-------------------|----------------------|
| **Sanity Checks** | ❌ Inexistente | ✅ Validações cruzadas automáticas |
| **Exemplos de Saída** | ❌ Inexistente | ✅ Few-shot learning com 8 casos reais |
| **Comentários Humanos** | ❌ Inexistente | ✅ Perguntas e observações inline |
| **Mapeamento de Sinônimos** | ❌ Inexistente | ✅ Facilita identificação de termos |

**Exemplo - Checks de Sanidade:**
```yaml
# ====================================================
# CHECKS DE SANIDADE (validações cruzadas de coerência)
# ====================================================
sanity_checks:
  - id: 'COERENCIA_MONETARIA'
    descricao: 'O valor com desconto NUNCA pode ser maior que o valor integral. (caso seja é importante sinalizar)'
    regra: 'O valor final em `mensalidade_com_desconto.conteudo` deve ser sempre menor ou igual ao valor em `mensalidade_sem_desconto.conteudo`.'
  
  - id: 'COERENCIA_TEMPORAL'
    descricao: 'A data de assinatura/vigência do contrato deve ser plausível para o semestre letivo.'
    regra: 'Analise a data do contrato (se disponível). Um contrato assinado em Dezembro para o semestre AAAA/1, por exemplo, é atípico.'
```

### 🎯 A IA como Intérprete: O Papel da Linguagem Natural

**O Desafio:**  
A IA não "entende" como um humano, ela interpreta padrões. A ambiguidade é sua maior inimiga.

**A Solução:**  
Um prompt eficaz atua como um "guia de interpretação". Ele não apenas diz o que fazer, mas também como pensar sobre a tarefa.

**YAML como Facilitador:**  
O YAML permite "perguntas humanas" e comentários que criam um contexto mais rico para a IA, alinhando a interpretação da máquina com a intenção humana.

### 📈 Benefícios Mensuráveis da Migração

#### Redução de Ambiguidades
- **JSON**: Regras em texto corrido, sujeitas a interpretação múltipla
- **YAML**: Passos numerados, condições explícitas e exemplos práticos

#### Melhoria na Rastreabilidade
- **JSON**: Difícil identificar onde uma regra falhou
- **YAML**: Cada passo é auditável e referenciável

#### Aumento da Robustez
- **JSON**: Sem validações cruzadas
- **YAML**: Sanity checks automáticos e early exit determinístico

#### Facilidade de Manutenção
- **JSON**: Modificações arriscadas devido à densidade
- **YAML**: Seções modulares, comentários e documentação inline

## 🏗️ Estrutura do Prompt

### Componentes Principais

1. **Papel e Contexto**: Define a IA como auditor especializado
2. **Regras Gerais**: Base fundamental para todo o processo
3. **Early Exit**: Sistema de bloqueio por CPF inválido
4. **Regras Específicas**: Validações detalhadas por campo
5. **Catálogo de Inconsistências**: Lista oficial de problemas
6. **Exemplos**: Casos práticos para orientar a IA

### Campos Auditados

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `cpf` | Obrigatório | CPF do contratante |
| `mensalidade_sem_desconto` | Obrigatório | Valor integral da mensalidade |
| `mensalidade_com_desconto` | Desejável | Valor com descontos aplicados |
| `semestre` | Obrigatório | Período letivo (AAAA/1 ou AAAA/2) |

## 🚀 Como Usar

### Pré-requisitos
- IA compatível com prompts YAML estruturados
- Contrato educacional em formato de texto
- Dados esperados do sistema ProBem

### Exemplo de Uso

```yaml
# Dados de entrada esperados
uni_cpf: "12345678901"
valor_mensalidade_sem_desconto: "1200.00"
valor_mensalidade_com_desconto: "1100.00"
semestre: "2025/1"
```

### Saída Esperada

```json
{
  "cpf": {"conteudo": "12345678901", "valido": 1},
  "mensalidade_sem_desconto": {"conteudo": "1200.00", "valido": 1},
  "mensalidade_com_desconto": {"conteudo": "1100.00", "valido": 1},
  "semestre": {"conteudo": "2025/1", "valido": 1},
  "inconsistencias": {"conteudo": "Sem inconsistencias", "valido": 1},
  "status": {"conteudo": "valido", "valido": 1}
}
```

## 🔧 Funcionalidades Avançadas

### Sistema Early Exit
- **Bloqueio imediato** se CPF não conferir
- Evita processamento desnecessário
- Garante comportamento determinístico

### Validação de Descontos
- Suporte a descontos percentuais e fixos
- Validação cruzada entre texto e tabelas
- Detecção de múltiplos descontos ambíguos
- Sistema de fallback para contratos sem desconto

### Normalização Automática
- **CPF**: Remove pontos e traços
- **Valores**: Formato monetário (1234.56)
- **Semestre**: Padronização AAAA/S
- **Datas**: Conversão automática para semestre

## 📊 Catálogo de Inconsistências

### Níveis de Prioridade

**🔴 Crítico**
- CPF não localizado no contrato
- CPF do contrato diverge do sistema

**🟡 Alto**
- Semestre letivo não localizado
- Semestre letivo diverge do sistema

**🟠 Médio**
- Mensalidade integral não localizada
- Valores divergentes (maior/menor que esperado)

**🟢 Baixo**
- Problemas com descontos e tabelas
- Ambiguidades contratuais

## 🧪 Casos de Teste

O prompt inclui 8 exemplos práticos:

1. **Contrato válido** - Todos os dados conferem
2. **CPF não localizado** - Falha crítica
3. **CPF divergente** - Falha crítica
4. **Valor semestral válido** - Divisão por 6 parcelas
5. **Valor semestral inválido** - Sem especificação de parcelas
6. **Desconto inconsistente** - Texto vs tabela
7. **Múltiplos descontos** - Regras ambíguas
8. **Semestre ambíguo** - Data não conclusiva

## 🛡️ Checks de Sanidade

### Validações Cruzadas
- **Coerência Monetária**: Valor com desconto ≤ valor integral
- **Coerência Temporal**: Data vs semestre letivo
- **Consistência Interna**: Status vs inconsistências

## ⚙️ Configurações

### Formatos Padronizados
```yaml
numero_monetario: "1234.56"    # Ponto decimal, 2 casas
semestre: "AAAA/1 ou AAAA/2"   # Janeiro-Junho = /1, Julho-Dezembro = /2
status: "valido ou invalido"    # Minúsculas obrigatórias
```

### Strings Fixas Obrigatórias
- `"nao localizado"` - Para dados não encontrados
- `"Sem inconsistencias"` - Para contratos válidos

## 📈 Impacto da Migração JSON → YAML

### 🎯 Conclusão dos Aprimoramentos

As mudanças do **JSON original → YAML proposto** foram pensadas para:

- 📌 **Reforçar imparcialidade** → Regras objetivas, menos interpretação subjetiva
- 📌 **Aumentar rastreabilidade** → Condições e passos explícitos, auditáveis
- 📌 **Melhorar robustez** → Early exit e sanity checks evitam erros graves
- 📌 **Aproximar da linguagem natural** → Perguntas humanas e exemplos reduzem atrito de entendimento

### Métricas de Melhoria Esperadas

| Métrica | Antes (JSON) | Depois (YAML) |
|---------|-------------|---------------|
| Casos de teste cobertos | 0 | 8 cenários |
| Validações cruzadas | 0 | 3 checks |

## 📈 Versioning

- **Versão**: 1.0
- **Última Atualização**: 2025-10-06
- **Compatibilidade**: Programa ProBem/OVG
- **Migração**: JSON → YAML (Otimizado para IA)

## 📋 Checklist de Implementação

- [X] Configurar IA com prompt YAML
- [X] Preparar dados de entrada do sistema
- [X] Implementar tratamento de saída JSON
- [X] Validar com contratos reais
- [ ] Migrar base existente JSON → YAML
- [ ] Configurar logs de auditoria

## ⚠️ Limitações

- Depende da qualidade do texto de entrada
- Necessita calibração inicial por tipo de contrato

## 🔍 Troubleshooting

### Problemas Comuns

**IA não encontra CPF**
- Verificar formato no contrato
- Validar se não está em imagem

**Valores monetários incorretos**
- Confirmar separadores decimais
- Verificar se valor está por extenso

**Semestre não detectado**
- Buscar datas de vigência
- Verificar períodos acadêmicos

## 📞 Suporte GGCI

Para questões técnicas ou melhorias:
- Abra uma issue com detalhes do problema
- Inclua exemplo do contrato (anonimizado)
- Especifique comportamento esperado vs observado

---

**Desenvolvido para**: Programa Universitário do Bem (ProBem) - OVG   
**Linguagem**: Português (Brasil)  
**Formato**: YAML Otimizado para IA