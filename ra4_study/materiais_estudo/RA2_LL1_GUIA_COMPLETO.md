# Guia Completo: LL(1) - Teoria e Implementação

## Índice

1. [O que é LL(1)?](#o-que-é-ll1)
2. [Gramática do Compilador](#gramática-do-compilador)
3. [Conjunto FIRST](#conjunto-first)
4. [Conjunto FOLLOW](#conjunto-follow)
5. [Tabela de Parsing LL(1)](#tabela-de-parsing-ll1)
6. [Algoritmo de Parsing](#algoritmo-de-parsing)
7. [Exemplo Completo Passo-a-Passo](#exemplo-completo-passo-a-passo)
8. [Implementação no Código](#implementação-no-código)
9. [Perguntas Típicas da Defesa](#perguntas-típicas-da-defesa)

---

## O que é LL(1)?

### Definição

**LL(1)** significa:
- **L** (Left-to-right): Lê a entrada da esquerda para a direita
- **L** (Leftmost derivation): Usa derivação mais à esquerda
- **1**: Usa apenas **1 token de lookahead** (olha 1 símbolo à frente)

### Por que LL(1)?

**Vantagens**:
✅ Parsing **eficiente** - O(n) onde n = tamanho da entrada
✅ Parsing **determinístico** - sem backtracking
✅ Fácil de **implementar** (stack-based)
✅ Fácil de **entender** e **ensinar**
✅ Mensagens de erro **claras**

**Desvantagens**:
❌ Não aceita todas as gramáticas (requer transformações)
❌ Não aceita recursão à esquerda
❌ Não aceita ambiguidade

### Gramáticas LL(1) vs Não-LL(1)

**LL(1) válida**:
```
E → T E'
E' → + T E' | ε
T → F T'
T' → * F T' | ε
F → ( E ) | id
```

**NÃO-LL(1) (recursão à esquerda)**:
```
E → E + T | T  ❌ (recursão à esquerda em E → E + T)
```

---

## Gramática do Compilador

### Gramática Completa (Notação BNF)

```
PROGRAM → LINHA PROGRAM_PRIME

PROGRAM_PRIME → LINHA PROGRAM_PRIME
              | ε

LINHA → ( SEQUENCIA )

SEQUENCIA → OPERANDO SEQUENCIA_PRIME

SEQUENCIA_PRIME → OPERANDO SEQUENCIA_PRIME
                | OPERADOR_FINAL
                | ε

OPERANDO → numero_inteiro OPERANDO_OPCIONAL
         | numero_real OPERANDO_OPCIONAL
         | variavel OPERANDO_OPCIONAL
         | LINHA

OPERANDO_OPCIONAL → res
                  | ε

OPERADOR_FINAL → ARITH_OP
               | COMP_OP
               | LOGIC_OP
               | CONTROL_OP

ARITH_OP → + | - | * | / | | | % | ^

COMP_OP → < | > | == | <= | >= | !=

LOGIC_OP → && | || | !

CONTROL_OP → for | while | ifelse
```

### Estrutura da Gramática

**Não-Terminais** (8):
- `PROGRAM`, `PROGRAM_PRIME`
- `LINHA`, `SEQUENCIA`, `SEQUENCIA_PRIME`
- `OPERANDO`, `OPERANDO_OPCIONAL`
- `OPERADOR_FINAL`
- `ARITH_OP`, `COMP_OP`, `LOGIC_OP`, `CONTROL_OP`

**Terminais** (mais de 30):
- Números: `numero_inteiro`, `numero_real`
- Variáveis: `variavel` (identificadores)
- Parênteses: `(`, `)`
- Operadores aritméticos: `+`, `-`, `*`, `/`, `|`, `%`, `^`
- Operadores relacionais: `<`, `>`, `==`, `<=`, `>=`, `!=`
- Operadores lógicos: `&&`, `||`, `!`
- Comandos de controle: `for`, `while`, `ifelse`
- Comando especial: `res`

**Símbolo Especial**:
- `ε` (epsilon): produção vazia

### Exemplo de Código Aceito

```
(1 COUNTER)               → atribuição
(8 LIMIT)                 → atribuição
((COUNTER LIMIT <=)       → condição de loop
  (((RESULT COUNTER *) RESULT))  → multiplicação
  ((COUNTER 1 +) COUNTER)        → incremento
  WHILE)                  → estrutura WHILE
```

---

## Conjunto FIRST

### Definição Teórica

**FIRST(α)** = {conjunto de terminais que podem aparecer como primeiro símbolo em alguma derivação de α}

Se α pode derivar ε, então `ε ∈ FIRST(α)`

### Regras para Calcular FIRST

#### Regra 1: Terminal
```
FIRST(a) = {a}    onde a é terminal
```

#### Regra 2: Epsilon
```
Se X → ε, então ε ∈ FIRST(X)
```

#### Regra 3: Não-Terminal
```
Se X → Y₁ Y₂ ... Yₖ:
  - Adicione FIRST(Y₁) - {ε} a FIRST(X)
  - Se ε ∈ FIRST(Y₁), adicione FIRST(Y₂) - {ε} a FIRST(X)
  - Se ε ∈ FIRST(Y₁) e ε ∈ FIRST(Y₂), adicione FIRST(Y₃) - {ε}
  - ...
  - Se todos Yᵢ podem derivar ε, então ε ∈ FIRST(X)
```

### Algoritmo FIRST (Ponto Fixo)

```python
def calcularFirst():
    # Inicialização
    FIRST = {nt: set() for nt in não_terminais}

    mudou = True
    while mudou:  # Itera até convergir
        mudou = False

        for cada produção X → α₁ α₂ ... αₖ:
            # Caso especial: epsilon
            if α == [ε]:
                if ε ∉ FIRST[X]:
                    FIRST[X].add(ε)
                    mudou = True
                continue

            # Processa cada símbolo αᵢ
            for αᵢ in α:
                if αᵢ é terminal:
                    if αᵢ ∉ FIRST[X]:
                        FIRST[X].add(αᵢ)
                        mudou = True
                    break  # Para aqui

                else:  # αᵢ é não-terminal
                    antes = len(FIRST[X])
                    FIRST[X] |= FIRST[αᵢ] - {ε}
                    if len(FIRST[X]) > antes:
                        mudou = True

                    if ε ∉ FIRST[αᵢ]:
                        break  # Para aqui

            # Se todos αᵢ podem derivar ε
            else:
                if ε ∉ FIRST[X]:
                    FIRST[X].add(ε)
                    mudou = True

    return FIRST
```

### FIRST da Nossa Gramática

```
FIRST(PROGRAM) = { ( }
FIRST(PROGRAM_PRIME) = { (, ε }
FIRST(LINHA) = { ( }
FIRST(SEQUENCIA) = { numero_inteiro, numero_real, variavel, ( }
FIRST(SEQUENCIA_PRIME) = { numero_inteiro, numero_real, variavel, (,
                            +, -, *, /, |, %, ^, <, >, ==, <=, >=, !=,
                            &&, ||, !, for, while, ifelse, ε }
FIRST(OPERANDO) = { numero_inteiro, numero_real, variavel, ( }
FIRST(OPERANDO_OPCIONAL) = { res, ε }
FIRST(OPERADOR_FINAL) = { +, -, *, /, |, %, ^, <, >, ==, <=, >=, !=,
                           &&, ||, !, for, while, ifelse }
FIRST(ARITH_OP) = { +, -, *, /, |, %, ^ }
FIRST(COMP_OP) = { <, >, ==, <=, >=, != }
FIRST(LOGIC_OP) = { &&, ||, ! }
FIRST(CONTROL_OP) = { for, while, ifelse }
```

### Implementação no Código

**Arquivo**: `src/RA2/functions/python/calcularFirst.py`

**Função principal** (linhas 13-56):
```python
def calcularFirst():
    gramatica = GRAMATICA_RPN
    nao_terminais = set(gramatica.keys())
    FIRST = {nt: set() for nt in nao_terminais}

    mudou = True
    while mudou:
        mudou = False
        for nt_head, producoes in gramatica.items():
            for producao in producoes:
                # Produção vazia
                if producao == ['epsilon']:
                    if 'epsilon' not in FIRST[nt_head]:
                        FIRST[nt_head].add('epsilon')
                        mudou = True
                    continue

                for simbolo in producao:
                    # Terminal
                    if simbolo not in nao_terminais:
                        if simbolo not in FIRST[nt_head]:
                            FIRST[nt_head].add(simbolo)
                            mudou = True
                        break

                    # Não-terminal
                    else:
                        tamanho_anterior = len(FIRST[nt_head])
                        FIRST[nt_head].update(FIRST[simbolo] - {'epsilon'})
                        if len(FIRST[nt_head]) > tamanho_anterior:
                            mudou = True

                        if 'epsilon' not in FIRST[simbolo]:
                            break

    return FIRST
```

---

## Conjunto FOLLOW

### Definição Teórica

**FOLLOW(A)** = {conjunto de terminais que podem aparecer imediatamente após A em alguma derivação}

Adicionamos `$` (fim de entrada) a FOLLOW do símbolo inicial.

### Regras para Calcular FOLLOW

#### Regra 1: Símbolo Inicial
```
$ ∈ FOLLOW(SIMBOLO_INICIAL)
```

#### Regra 2: Produção A → αBβ
```
Se existe A → αBβ:
  FOLLOW(B) ⊇ FIRST(β) - {ε}
```

#### Regra 3: Produção A → αB ou A → αBβ onde ε ∈ FIRST(β)
```
Se existe A → αB  OU  A → αBβ onde ε ∈ FIRST(β):
  FOLLOW(B) ⊇ FOLLOW(A)
```

### Algoritmo FOLLOW (Ponto Fixo)

```python
def calcularFollow(FIRST):
    FOLLOW = {nt: set() for nt in não_terminais}

    # Regra 1: $ em FOLLOW do símbolo inicial
    FOLLOW[SIMBOLO_INICIAL].add('$')

    mudou = True
    while mudou:
        mudou = False

        for cada produção A → β₁ β₂ ... βₙ:
            for cada não-terminal Bᵢ em β:
                # Resto da produção após Bᵢ
                resto = βᵢ₊₁ βᵢ₊₂ ... βₙ

                # Regra 2: FIRST(resto) - {ε}
                antes = len(FOLLOW[Bᵢ])
                FOLLOW[Bᵢ] |= FIRST(resto) - {ε}
                if len(FOLLOW[Bᵢ]) > antes:
                    mudou = True

                # Regra 3: Se ε ∈ FIRST(resto), adiciona FOLLOW(A)
                if ε ∈ FIRST(resto):
                    antes = len(FOLLOW[Bᵢ])
                    FOLLOW[Bᵢ] |= FOLLOW[A]
                    if len(FOLLOW[Bᵢ]) > antes:
                        mudou = True

    return FOLLOW
```

### FOLLOW da Nossa Gramática

```
FOLLOW(PROGRAM) = { $ }
FOLLOW(PROGRAM_PRIME) = { $ }
FOLLOW(LINHA) = { (, $ }
FOLLOW(SEQUENCIA) = { ) }
FOLLOW(SEQUENCIA_PRIME) = { ) }
FOLLOW(OPERANDO) = { numero_inteiro, numero_real, variavel, (,
                      +, -, *, /, |, %, ^, <, >, ==, <=, >=, !=,
                      &&, ||, !, for, while, ifelse, ) }
FOLLOW(OPERANDO_OPCIONAL) = { numero_inteiro, numero_real, variavel, (,
                               +, -, *, /, |, %, ^, <, >, ==, <=, >=, !=,
                               &&, ||, !, for, while, ifelse, ) }
FOLLOW(OPERADOR_FINAL) = { ) }
FOLLOW(ARITH_OP) = { ) }
FOLLOW(COMP_OP) = { ) }
FOLLOW(LOGIC_OP) = { ) }
FOLLOW(CONTROL_OP) = { ) }
```

### Por que FOLLOW é Importante?

FOLLOW é usado para preencher entradas da tabela LL(1) quando há **produções epsilon**:

```
Se A → ε  e  t ∈ FOLLOW(A):
  M[A, t] = A → ε
```

**Exemplo na nossa gramática**:
- `SEQUENCIA_PRIME → ε`
- `FOLLOW(SEQUENCIA_PRIME) = { ) }`
- Logo, `M[SEQUENCIA_PRIME, )] = SEQUENCIA_PRIME → ε`

---

## Tabela de Parsing LL(1)

### Definição

A **tabela LL(1)** é uma matriz `M[não-terminal, terminal]` que indica qual produção usar.

### Regras de Construção

Para cada produção `A → α`:

#### Regra 1: FIRST
```
Para cada terminal t ∈ FIRST(α):
  M[A, t] = A → α
```

#### Regra 2: FOLLOW (quando ε ∈ FIRST(α))
```
Se ε ∈ FIRST(α):
  Para cada terminal t ∈ FOLLOW(A):
    M[A, t] = A → α
```

### Algoritmo de Construção

```python
def construirTabelaLL1(FIRST, FOLLOW):
    M = {}  # Tabela vazia

    for cada produção A → α:
        # Regra 1: Terminais em FIRST(α)
        for t in FIRST(α) - {ε}:
            M[A, t] = A → α

        # Regra 2: Se ε ∈ FIRST(α), usa FOLLOW(A)
        if ε ∈ FIRST(α):
            for t in FOLLOW(A):
                M[A, t] = A → α

    return M
```

### Exemplo de Entradas da Tabela

| Não-Terminal | Terminal | Produção |
|--------------|----------|----------|
| PROGRAM | `(` | PROGRAM → LINHA PROGRAM_PRIME |
| PROGRAM_PRIME | `(` | PROGRAM_PRIME → LINHA PROGRAM_PRIME |
| PROGRAM_PRIME | `$` | PROGRAM_PRIME → ε |
| LINHA | `(` | LINHA → ( SEQUENCIA ) |
| SEQUENCIA | `numero_inteiro` | SEQUENCIA → OPERANDO SEQUENCIA_PRIME |
| SEQUENCIA | `variavel` | SEQUENCIA → OPERANDO SEQUENCIA_PRIME |
| SEQUENCIA_PRIME | `+` | SEQUENCIA_PRIME → OPERADOR_FINAL |
| SEQUENCIA_PRIME | `)` | SEQUENCIA_PRIME → ε |

### Conflitos LL(1)

Uma gramática **NÃO é LL(1)** se houver **conflitos**:

#### Conflito FIRST-FIRST:
```
M[A, t] já contém A → α₁
Tentamos adicionar A → α₂ para o mesmo [A, t]
```

#### Conflito FIRST-FOLLOW:
```
M[A, t] já contém A → α (de FIRST)
Tentamos adicionar A → ε (de FOLLOW) para o mesmo [A, t]
```

**Nossa gramática**: ✅ Sem conflitos (é LL(1) válida)

---

## Algoritmo de Parsing

### Parser Descendente Preditivo (Stack-Based)

**Estruturas**:
- **Pilha**: Armazena símbolos a processar
- **Entrada**: Sequência de tokens
- **Tabela LL(1)**: Guia de decisões

### Algoritmo

```python
def parsear(tokens, tabela_LL1):
    pilha = ['$', SIMBOLO_INICIAL]
    entrada = tokens + ['$']
    indice = 0

    while pilha:
        topo = pilha.pop()
        token_atual = entrada[indice]

        # Caso 1: Topo é terminal
        if topo é terminal:
            if topo == token_atual:
                indice += 1  # Avança entrada
            else:
                ERRO("Esperava {topo}, recebeu {token_atual}")

        # Caso 2: Topo é não-terminal
        else:
            producao = tabela_LL1[topo, token_atual]

            if producao existe:
                if producao != ε:
                    empilha símbolos de produção (em ordem reversa)
            else:
                ERRO("Nenhuma produção para [{topo}, {token_atual}]")

    SUCESSO("Parsing completo")
```

### Visualização do Algoritmo

```
Pilha:    [$, PROGRAM]
Entrada:  [(, numero_inteiro, variavel, ), $]
          ^

Ação: M[PROGRAM, (] = PROGRAM → LINHA PROGRAM_PRIME
Pilha:    [$, PROGRAM_PRIME, LINHA]
Entrada:  [(, numero_inteiro, variavel, ), $]
          ^

Ação: M[LINHA, (] = LINHA → ( SEQUENCIA )
Pilha:    [$, PROGRAM_PRIME, ), SEQUENCIA, (]
Entrada:  [(, numero_inteiro, variavel, ), $]
          ^

Ação: Casamento de (
Pilha:    [$, PROGRAM_PRIME, ), SEQUENCIA]
Entrada:  [(, numero_inteiro, variavel, ), $]
             ^

... (continua até pilha vazia)
```

---

## Exemplo Completo Passo-a-Passo

### Entrada: `(5 3 +)`

**Tokenização**:
```
[(, numero_inteiro(5), numero_inteiro(3), +, )]
```

### Parsing Detalhado

| Passo | Pilha | Entrada | Ação |
|-------|-------|---------|------|
| 1 | `[$, PROGRAM]` | `[(, 5, 3, +, ), $]` | M[PROGRAM, (] → PROGRAM → LINHA PROGRAM_PRIME |
| 2 | `[$, PROGRAM_PRIME, LINHA]` | `[(, 5, 3, +, ), $]` | M[LINHA, (] → LINHA → ( SEQUENCIA ) |
| 3 | `[$, PROGRAM_PRIME, ), SEQUENCIA, (]` | `[(, 5, 3, +, ), $]` | Casa ( |
| 4 | `[$, PROGRAM_PRIME, ), SEQUENCIA]` | `[5, 3, +, ), $]` | M[SEQUENCIA, 5] → SEQUENCIA → OPERANDO SEQUENCIA_PRIME |
| 5 | `[$, PROGRAM_PRIME, ), SEQUENCIA_PRIME, OPERANDO]` | `[5, 3, +, ), $]` | M[OPERANDO, 5] → OPERANDO → numero_inteiro OPERANDO_OPCIONAL |
| 6 | `[$, PROGRAM_PRIME, ), SEQUENCIA_PRIME, OPERANDO_OPCIONAL, numero_inteiro]` | `[5, 3, +, ), $]` | Casa 5 |
| 7 | `[$, PROGRAM_PRIME, ), SEQUENCIA_PRIME, OPERANDO_OPCIONAL]` | `[3, +, ), $]` | M[OPERANDO_OPCIONAL, 3] → OPERANDO_OPCIONAL → ε |
| 8 | `[$, PROGRAM_PRIME, ), SEQUENCIA_PRIME]` | `[3, +, ), $]` | M[SEQUENCIA_PRIME, 3] → SEQUENCIA_PRIME → OPERANDO SEQUENCIA_PRIME |
| 9 | `[$, PROGRAM_PRIME, ), SEQUENCIA_PRIME, OPERANDO]` | `[3, +, ), $]` | M[OPERANDO, 3] → OPERANDO → numero_inteiro OPERANDO_OPCIONAL |
| 10 | `[$, PROGRAM_PRIME, ), SEQUENCIA_PRIME, OPERANDO_OPCIONAL, numero_inteiro]` | `[3, +, ), $]` | Casa 3 |
| 11 | `[$, PROGRAM_PRIME, ), SEQUENCIA_PRIME, OPERANDO_OPCIONAL]` | `[+, ), $]` | M[OPERANDO_OPCIONAL, +] → OPERANDO_OPCIONAL → ε |
| 12 | `[$, PROGRAM_PRIME, ), SEQUENCIA_PRIME]` | `[+, ), $]` | M[SEQUENCIA_PRIME, +] → SEQUENCIA_PRIME → OPERADOR_FINAL |
| 13 | `[$, PROGRAM_PRIME, ), OPERADOR_FINAL]` | `[+, ), $]` | M[OPERADOR_FINAL, +] → OPERADOR_FINAL → ARITH_OP |
| 14 | `[$, PROGRAM_PRIME, ), ARITH_OP]` | `[+, ), $]` | M[ARITH_OP, +] → ARITH_OP → + |
| 15 | `[$, PROGRAM_PRIME, ), +]` | `[+, ), $]` | Casa + |
| 16 | `[$, PROGRAM_PRIME, )]` | `[), $]` | Casa ) |
| 17 | `[$, PROGRAM_PRIME]` | `[$]` | M[PROGRAM_PRIME, $] → PROGRAM_PRIME → ε |
| 18 | `[$]` | `[$]` | Casa $ - **SUCESSO!** |

### AST Gerada

```
PROGRAM
├── LINHA
│   ├── (
│   ├── SEQUENCIA
│   │   ├── OPERANDO
│   │   │   └── 5
│   │   ├── SEQUENCIA_PRIME
│   │   │   ├── OPERANDO
│   │   │   │   └── 3
│   │   │   └── SEQUENCIA_PRIME
│   │   │       └── OPERADOR_FINAL
│   │   │           └── +
│   └── )
└── PROGRAM_PRIME
    └── ε
```

---

## Implementação no Código

### Arquivos Principais

#### 1. configuracaoGramatica.py (linhas 14-41)
Define a gramática:
```python
GRAMATICA_RPN = {
    'PROGRAM': [['LINHA', 'PROGRAM_PRIME']],
    'PROGRAM_PRIME': [['LINHA', 'PROGRAM_PRIME'], ['epsilon']],
    'LINHA': [['abre_parenteses', 'SEQUENCIA', 'fecha_parenteses']],
    # ... (completa)
}
```

#### 2. calcularFirst.py (linhas 13-56)
Calcula conjunto FIRST (já mostrado acima)

#### 3. calcularFollow.py
Calcula conjunto FOLLOW usando FIRST

#### 4. construirTabelaLL1.py
Constrói a tabela usando FIRST e FOLLOW

#### 5. parsear.py
Implementa o parser stack-based

**Função principal**:
```python
def parsear(tokens, tabela_ll1):
    pilha = ['$', SIMBOLO_INICIAL]
    # ... (algoritmo completo)
```

#### 6. gerarArvore.py
Converte sequência de derivações em AST

---

## Perguntas Típicas da Defesa

### P1: "O que significa LL(1)?"

**R**:
- **L** = Left-to-right (lê entrada da esquerda → direita)
- **L** = Leftmost derivation (deriva sempre o não-terminal mais à esquerda)
- **1** = 1 token de lookahead (olha apenas 1 símbolo à frente para decidir)

É um método de parsing **top-down** e **determinístico**.

---

### P2: "Por que precisamos de FIRST e FOLLOW?"

**R**:
- **FIRST**: Diz quais terminais podem iniciar uma derivação de um não-terminal
  - Usado para decidir qual produção aplicar quando vemos um terminal

- **FOLLOW**: Diz quais terminais podem aparecer após um não-terminal
  - Usado quando há produção epsilon (ε) - decidimos se podemos "pular" o não-terminal

**Exemplo**:
```
SEQUENCIA_PRIME → ε  (pode ser vazia)

Quando vemos ), que está em FOLLOW(SEQUENCIA_PRIME),
sabemos que podemos aplicar SEQUENCIA_PRIME → ε
```

---

### P3: "Como calculamos FIRST de uma sequência?"

**R**: Algoritmo:
```python
def FIRST_sequencia(α₁ α₂ ... αₙ):
    resultado = {}

    for αᵢ in [α₁, α₂, ..., αₙ]:
        if αᵢ é terminal:
            return {αᵢ}

        # αᵢ é não-terminal
        resultado |= FIRST(αᵢ) - {ε}

        if ε ∉ FIRST(αᵢ):
            return resultado

    # Se chegou aqui, todos podem derivar ε
    resultado.add(ε)
    return resultado
```

**Exemplo**:
```
FIRST(OPERANDO SEQUENCIA_PRIME):
  FIRST(OPERANDO) = {numero_inteiro, numero_real, variavel, (}
  ε ∉ FIRST(OPERANDO)
  Logo: FIRST(OPERANDO SEQUENCIA_PRIME) = {numero_inteiro, numero_real, variavel, (}
```

---

### P4: "Como saber se uma gramática é LL(1)?"

**R**: A gramática é LL(1) se **não houver conflitos** na tabela:

**Teste 1** (FIRST-FIRST): Para cada não-terminal A com produções A → α | β:
```
FIRST(α) ∩ FIRST(β) = ∅  (conjuntos disjuntos)
```

**Teste 2** (FIRST-FOLLOW): Se A → α | β e ε ∈ FIRST(α):
```
FIRST(α) ∩ FOLLOW(A) = ∅
```

Nossa gramática passa em ambos os testes ✅

---

### P5: "Mostre o código onde FIRST é calculado"

**R**: Arquivo `src/RA2/functions/python/calcularFirst.py`, linhas 13-56:

**Lógica principal**:
1. Inicializa FIRST vazio para cada não-terminal (linha 21)
2. Loop até convergir (linhas 26-54)
3. Para cada produção:
   - Se é epsilon, adiciona ε (linhas 32-35)
   - Para cada símbolo:
     - Se terminal: adiciona e para (linhas 40-44)
     - Se não-terminal: adiciona FIRST do símbolo (linhas 48-51)
     - Para se não tem ε (linhas 53-54)

---

### P6: "Trace a pilha para `(1 COUNTER)`"

**R**:
```
Entrada: [(, numero_inteiro, variavel, ), $]

Passo 1:
  Pilha: [$, PROGRAM]
  Ação: Expande PROGRAM → LINHA PROGRAM_PRIME

Passo 2:
  Pilha: [$, PROGRAM_PRIME, LINHA]
  Ação: Expande LINHA → ( SEQUENCIA )

Passo 3:
  Pilha: [$, PROGRAM_PRIME, ), SEQUENCIA, (]
  Ação: Casa (
  Entrada avança

Passo 4:
  Pilha: [$, PROGRAM_PRIME, ), SEQUENCIA]
  Entrada: [numero_inteiro, variavel, ), $]
  Ação: Expande SEQUENCIA → OPERANDO SEQUENCIA_PRIME

... (continua até pilha vazia)
```

---

### P7: "Qual a diferença entre LL(1) e LR(1)?"

**R**:

| Aspecto | LL(1) | LR(1) |
|---------|-------|-------|
| Direção | Top-down | Bottom-up |
| Derivação | Leftmost | Rightmost reversa |
| Poder | Menos gramáticas | Mais gramáticas |
| Complexidade | Mais simples | Mais complexo |
| Tabela | Menor | Maior |
| Uso | Compiladores didáticos | Compiladores industriais |

**Nosso compilador usa LL(1)** porque é mais fácil de entender e implementar.

---

## Próximos Passos

Para dominar completamente LL(1):
1. ✅ Leia este guia
2. 📖 Estude `RA2_WALKTHROUGH.md` para ver exemplo completo
3. 💻 Trace manualmente `fatorial.txt` pelo parser
4. 🧪 Teste modificando a gramática e recalculando FIRST/FOLLOW
5. 📝 Pratique respondendo perguntas de `PERGUNTAS_DEFESA.md`

---

**Última atualização**: 2025-01-27
