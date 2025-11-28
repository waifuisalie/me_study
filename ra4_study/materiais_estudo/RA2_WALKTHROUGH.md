# RA2: Walkthrough de Análise Sintática

## Teoria Resumida

**Análise Sintática** = Verifica se tokens seguem a gramática e constrói AST.

- **Entrada**: Tokens (de RA1)
- **Saída**: Árvore Sintática Abstrata (AST)
- **Método**: Parser LL(1) descendente preditivo

---

## Processo Completo

### 1. Leitura de Tokens
**Arquivo**: `lerTokens.py`
```python
tokens = ler_tokens('tokens_gerados.txt')
# Retorna: ['(', '5', 'COUNTER', '+', ')']
```

### 2. Construção da Gramática
**Arquivo**: `configuracaoGramatica.py`
```python
GRAMATICA = {
    'PROGRAM': [['LINHA', 'PROGRAM_PRIME']],
    'LINHA': [['(', 'SEQUENCIA', ')']],
    ...
}
```

### 3. Cálculo de FIRST
**Arquivo**: `calcularFirst.py`
```python
FIRST = {
    'PROGRAM': {'('},
    'LINHA': {'('},
    'OPERANDO': {'numero_inteiro', 'numero_real', 'variavel', '('},
    ...
}
```

### 4. Cálculo de FOLLOW
**Arquivo**: `calcularFollow.py`
```python
FOLLOW = {
    'PROGRAM': {'$'},
    'SEQUENCIA': {')'},
    'OPERANDO': {'+', '-', ')', ...},
    ...
}
```

### 5. Construção da Tabela LL(1)
**Arquivo**: `construirTabelaLL1.py`
```python
M = {
    ('PROGRAM', '('): 'PROGRAM → LINHA PROGRAM_PRIME',
    ('LINHA', '('): 'LINHA → ( SEQUENCIA )',
    ('SEQUENCIA_PRIME', ')'): 'SEQUENCIA_PRIME → ε',
    ...
}
```

### 6. Parsing
**Arquivo**: `parsear.py`
```python
derivacoes = parsear(tokens, tabela_LL1)
# Retorna sequência de produções aplicadas
```

### 7. Geração da AST
**Arquivo**: `gerarArvore.py`
```python
arvore = gerarArvore(derivacoes)
# Retorna estrutura hierárquica
```

---

## Exemplo: `(1 COUNTER)`

### Passo 1: Tokens
```
['(', 'numero_inteiro', 'variavel', ')']
```

### Passo 2: Parsing (Pilha)

| Pilha | Entrada | Ação |
|-------|---------|------|
| `[$, PROGRAM]` | `[(, 1, COUNTER, )]` | Expande: PROGRAM → LINHA PROGRAM_PRIME |
| `[$, PROGRAM_PRIME, LINHA]` | `[(, 1, COUNTER, )]` | Expande: LINHA → ( SEQUENCIA ) |
| `[$, PROGRAM_PRIME, ), SEQUENCIA, (]` | `[(, 1, COUNTER, )]` | Casa `(` |
| `[$, PROGRAM_PRIME, ), SEQUENCIA]` | `[1, COUNTER, )]` | Expande: SEQUENCIA → OPERANDO SEQUENCIA_PRIME |
| `[$, PROGRAM_PRIME, ), SEQ_PRIME, OPERANDO]` | `[1, COUNTER, )]` | Expande: OPERANDO → numero_inteiro OPERANDO_OPCIONAL |
| ... | ... | ... |
| `[$]` | `[$]` | **ACEITO** ✓ |

### Passo 3: AST Gerada

```json
{
  "linhas": [
    {
      "numero_linha": 1,
      "arvore": {
        "label": "LINHA",
        "filhos": [
          {"label": "("},
          {
            "label": "SEQUENCIA",
            "filhos": [
              {
                "label": "OPERANDO",
                "filhos": [{"label": "1"}]
              },
              {
                "label": "SEQUENCIA_PRIME",
                "filhos": [
                  {
                    "label": "OPERANDO",
                    "filhos": [{"label": "COUNTER"}]
                  },
                  {"label": "ε"}
                ]
              }
            ]
          },
          {"label": ")"}
        ]
      }
    }
  ]
}
```

---

## Código Principal

### Parser Stack-Based

**Arquivo**: `parsear.py`

```python
def parsear(tokens, tabela_ll1):
    pilha = ['$', SIMBOLO_INICIAL]
    entrada = tokens + ['$']
    indice = 0
    derivacoes = []

    while pilha:
        topo = pilha.pop()
        token_atual = entrada[indice]

        if topo == token_atual:
            # Casamento de terminal
            indice += 1
        elif topo in nao_terminais:
            # Consulta tabela
            producao = tabela_ll1.get((topo, token_atual))

            if producao:
                derivacoes.append(producao)
                # Empilha símbolos em ordem reversa
                for simbolo in reversed(producao.rhs):
                    if simbolo != 'ε':
                        pilha.append(simbolo)
            else:
                raise ErroSintatico(f"Nenhuma produção para [{topo}, {token_atual}]")
        else:
            raise ErroSintatico(f"Esperava {topo}, recebeu {token_atual}")

    return derivacoes
```

---

## Interface com RA3

**Saída**: `outputs/RA2/arvore_sintatica.json`

**Formato**:
```json
{
  "linhas": [
    {
      "numero_linha": 1,
      "arvore": { ... },
      "tokens": ["(", "1", "COUNTER", ")"]
    }
  ]
}
```

**Contrato**:
- Estrutura hierárquica com `label` e `filhos`
- Cada linha mantém tokens originais
- RA3 adiciona tipos a cada nó

---

## Perguntas Típicas

### P1: "Por que LL(1) e não LR(1)?"

**R**: LL(1) é **mais simples** para fins didáticos:
- Parser descendente (top-down) mais intuitivo
- Tabela menor
- Erros mais claros
- Suficiente para nossa gramática RPN

### P2: "Como funciona a tabela LL(1)?"

**R**: É um **lookup table** `M[não-terminal, terminal]`:
```
M[SEQUENCIA, numero_inteiro] = SEQUENCIA → OPERANDO SEQUENCIA_PRIME
M[SEQUENCIA_PRIME, )] = SEQUENCIA_PRIME → ε
```

Quando vemos `SEQUENCIA` no topo da pilha e `numero_inteiro` na entrada, sabemos qual produção aplicar.

### P3: "O que acontece se a gramática não for LL(1)?"

**R**: Haverá **conflitos** na tabela:
- **FIRST-FIRST**: Duas produções para mesma entrada
- **FIRST-FOLLOW**: Produção normal vs epsilon

Nossa gramática é LL(1) válida (sem conflitos).

---

**Ver também**: `RA2_LL1_GUIA_COMPLETO.md` para detalhes de LL(1)
**Próximo**: `RA3_WALKTHROUGH.md` - Análise Semântica
