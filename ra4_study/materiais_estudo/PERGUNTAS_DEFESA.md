# Perguntas Antecipadas para Defesa

## Perguntas Cross-Cutting (Mais Prováveis)

### P1: "Como não-terminais de RA2 se relacionam com verificação de tipos em RA3?"

**Resposta**:

**RA2** produz AST com **não-terminais** que definem a estrutura sintática:
- `ARITH_OP`, `COMP_OP`, `LOGIC_OP` → Indicam tipo de operação
- `OPERANDO` → Pode ser número ou variável
- `SEQUENCIA` → Lista de operandos

**RA3** usa essa estrutura para **inferir tipos**:
- `ARITH_OP` → Verifica tipos dos filhos e promove se necessário
- `COMP_OP` → Retorna sempre `boolean`
- Literais (`numero_inteiro`, `numero_real`) → Determinam tipo base

**Exemplo**:
```
AST (RA2):
  ARITH_OP(+)
  ├── numero_inteiro(5)
  └── numero_real(3.0)

RA3 processa:
  1. numero_inteiro → tipo = int
  2. numero_real → tipo = real
  3. ARITH_OP vê (int, real) → promove para real
  4. Anota ARITH_OP com tipo = real
```

**Código**: `analisador_tipos.py:150-200`

---

### P2: "Trace constant folding desde AST até TAC otimizado"

**Resposta**:

**Entrada**: `(2 3 +)`

**RA2 → AST**:
```json
{
  "label": "ARITH_OP",
  "operador": "+",
  "filhos": [
    {"label": "2"},
    {"label": "3"}
  ]
}
```

**RA3 → AST Atribuída**:
```json
{
  "tipo": "int",
  "operador": "+",
  "filhos": [
    {"valor": "2", "tipo": "int"},
    {"valor": "3", "tipo": "int"}
  ]
}
```

**RA4a → TAC Original**:
```
t0 = 2 + 3
```

**RA4b → Otimização (Constant Folding)**:
```python
# otimizador_tac.py detecta:
if _is_constant("2") and _is_constant("3"):
    resultado = _eval_binary_op(2, '+', 3)  # = 5
    substituir instrução
```

**TAC Otimizado**:
```
t0 = 5
```

**Arquivos envolvidos**:
- `gerador_tac.py:200` - Gera TAC original
- `otimizador_tac.py:250` - Aplica folding

---

### P3: "O que acontece se RA2 produz AST inválida? Como RA3 detecta?"

**Resposta**:

**RA2 não pode produzir AST inválida** porque:
1. Parser LL(1) é **determinístico**
2. Só aceita entrada que segue a gramática
3. Se entrada é inválida, RA2 **falha antes** de gerar AST

**MAS**, RA3 ainda detecta erros **semânticos**:

**Exemplo - Erro de Tipo**:
```
Entrada: (5 "texto" +)

RA2: Aceita (sintaxe correta)
  → AST: ARITH_OP com filhos (5, "texto")

RA3: REJEITA
  → Erro: "Tipos incompatíveis: int + string"
```

**Exemplo - Variável Não Inicializada**:
```
Entrada: (X 5 +)

RA2: Aceita (sintaxe correta)
  → AST: ARITH_OP com filhos (X, 5)

RA3: REJEITA
  → Erro: "Variável X usada antes de inicialização"
```

**Código**: `analisador_semantico.py:80-120`

---

## Perguntas Algoritmos Profundas

### P4: "Explique FOLLOW para produções epsilon"

**Resposta**:

**FOLLOW** é usado quando um não-terminal pode derivar **epsilon** (ε).

**Regra**: Se `A → ε`, precisamos saber quais terminais podem vir **após** A para decidir se aplicamos a produção epsilon.

**Exemplo na nossa gramática**:
```
SEQUENCIA_PRIME → OPERANDO SEQUENCIA_PRIME
                | OPERADOR_FINAL
                | ε

FOLLOW(SEQUENCIA_PRIME) = { ) }
```

**Por quê?**:
```
LINHA → ( SEQUENCIA )
SEQUENCIA → OPERANDO SEQUENCIA_PRIME

Logo, SEQUENCIA_PRIME sempre vem antes de )
Portanto, ) ∈ FOLLOW(SEQUENCIA_PRIME)
```

**Uso na tabela LL(1)**:
```
M[SEQUENCIA_PRIME, )] = SEQUENCIA_PRIME → ε
```

Quando vemos `)` e temos `SEQUENCIA_PRIME` no topo da pilha, aplicamos epsilon.

**Código**: `calcularFollow.py:40-80`

---

### P5: "Por que constant propagation precisa múltiplas passadas?"

**Resposta**:

Porque uma propagação cria **oportunidades em cascata**:

**Exemplo**:
```
Código original:
  t1 = 5
  t2 = t1 + 3
  t3 = t2 * 2
  RESULT = t3

Iteração 1 - Propagation:
  t1 = 5
  t2 = 5 + 3      ← t1 substituído
  t3 = t2 * 2
  RESULT = t3

Iteração 1 - Folding:
  t1 = 5
  t2 = 8          ← avaliado
  t3 = t2 * 2
  RESULT = t3

Iteração 2 - Propagation:
  t1 = 5
  t2 = 8
  t3 = 8 * 2      ← t2 substituído
  RESULT = t3

Iteração 2 - Folding:
  t1 = 5
  t2 = 8
  t3 = 16         ← avaliado
  RESULT = t3

Iteração 3 - Dead Code Elimination:
  t3 = 16         ← t1, t2 removidos (não usados)
  RESULT = t3

Iteração 4 - Nenhuma mudança → CONVERGE
```

**Prova de terminação**:
- Cada passada reduz ou mantém número de instruções
- Nunca aumenta
- Limite inferior = 0 instruções
- Limite de iterações = 100
- **Logo, sempre termina**

**Código**: `otimizador_tac.py:134-165`

---

### P6: "Como prova que seu otimizador termina?"

**Resposta**:

**Teorema**: O algoritmo sempre termina.

**Prova**:

1. **Invariante**: Seja n(i) = número de instruções na iteração i
   - n(i+1) ≤ n(i) para todo i
   - (Nunca aumentamos instruções)

2. **Monotonia**: Cada otimização é monotônica
   - Folding: substitui instrução complexa por simples
   - Propagation: não muda número de instruções
   - Dead code: remove instruções
   - Jump: remove instruções
   - Logo: n(i+1) ≤ n(i)

3. **Limite inferior**: n ≥ 0
   - Número mínimo de instruções é 0

4. **Condição de parada**:
   - Se n(i+1) = n(i), não houve mudança → para
   - Ou iteração ≥ 100 → para

5. **Conclusão**:
   - Sequência n(0), n(1), n(2), ... é **decrescente** e **limitada inferiormente**
   - Logo, converge para ponto fixo
   - **QED** ∎

**Código**: `otimizador_tac.py:127-156`

---

## Perguntas de Implementação

### P7: "Mostre o código onde type promotion acontece"

**Resposta**:

**Arquivo**: `src/RA3/functions/python/analisador_tipos.py`

**Função** (linhas ~120-140):
```python
def promover_tipo(tipo1: str, tipo2: str) -> str:
    """
    Promove tipos seguindo hierarquia: int < real

    Args:
        tipo1: Primeiro tipo
        tipo2: Segundo tipo

    Returns:
        Tipo promovido

    Raises:
        ErroSemantico: Se tipos incompatíveis
    """
    # Se um dos tipos é real, promove para real
    if tipo1 == TYPE_REAL or tipo2 == TYPE_REAL:
        return TYPE_REAL

    # Se ambos são int, resultado é int
    elif tipo1 == TYPE_INT and tipo2 == TYPE_INT:
        return TYPE_INT

    # Tipos incompatíveis
    else:
        raise ErroSemantico(
            f"Tipos incompatíveis: {tipo1} e {tipo2}"
        )
```

**Uso** (linhas ~200-220):
```python
def inferir_tipo_operador_binario(no):
    tipo_esq = inferir_tipo(no.filhos[0])
    tipo_dir = inferir_tipo(no.filhos[1])

    # Aplica promoção
    tipo_resultado = promover_tipo(tipo_esq, tipo_dir)

    return tipo_resultado
```

**Exemplo de uso**:
```python
# Entrada: (5 3.0 +)
tipo_esq = 'int'    # de 5
tipo_dir = 'real'   # de 3.0
resultado = promover_tipo('int', 'real')  # retorna 'real'
```

---

### P8: "Walk through stack states para expressão aninhada"

**Resposta**:

**Entrada**: `((5 3 +) 2 *)`

**Parsing Stack-Based**:

| Passo | Pilha | Entrada | Ação |
|-------|-------|---------|------|
| 1 | `[$, PROGRAM]` | `[((, 5, 3, +, ), 2, *, ), $]` | Expande PROGRAM |
| 2 | `[$, PROGRAM_PRIME, LINHA]` | `[((, 5, 3, +, ), 2, *, ), $]` | Expande LINHA |
| 3 | `[$, PR_PRIME, ), SEQ, (]` | `[((, 5, 3, +, ), 2, *, ), $]` | Casa `(` externo |
| 4 | `[$, PR_PRIME, ), SEQ]` | `[(, 5, 3, +, ), 2, *, ), $]` | Expande SEQUENCIA |
| 5 | `[$, PR_PRIME, ), SEQ_PRIME, OPERANDO]` | `[(, 5, 3, +, ), 2, *, ), $]` | Expande OPERANDO |
| 6 | `[$, PR_PRIME, ), SEQ_PRIME, LINHA]` | `[(, 5, 3, +, ), 2, *, ), $]` | OPERANDO → LINHA (recursão!) |
| 7 | `[$, PR_PRIME, ), SEQ_PRIME, ), SEQ, (]` | `[(, 5, 3, +, ), 2, *, ), $]` | Expande LINHA interna |
| 8 | `[$, PR_PRIME, ), SEQ_PRIME, ), SEQ]` | `[5, 3, +, ), 2, *, ), $]` | Casa `(` interno |
| ... | ... | ... | (processa 5, 3, +) |
| 15 | `[$, PR_PRIME, ), SEQ_PRIME]` | `[2, *, ), $]` | Casa `)` interno |
| 16 | `[$, PR_PRIME, ), SEQ_PRIME]` | `[2, *, ), $]` | Expande SEQUENCIA_PRIME |
| ... | ... | ... | (processa 2, *) |
| 25 | `[$]` | `[$]` | **ACEITO** ✓ |

**Observação**: Pilha cresce quando encontra **expressão aninhada** (linha recursiva).

**Código**: `parsear.py:50-150`

---

## Perguntas de Design

### P9: "Por que TAC em vez de gerar Assembly direto?"

**Resposta**:

**Vantagens do TAC**:

1. **Independência de arquitetura**
   - TAC é genérico
   - Mesmo TAC pode gerar x86, ARM, AVR

2. **Facilita otimização**
   - TAC é linear e simples
   - Fácil de analisar padrões
   - Fácil de transformar

3. **Modularidade**
   - Geração TAC separada de otimização
   - Otimização separada de Assembly
   - Fases independentes

4. **Debugging**
   - TAC é legível para humanos
   - Fácil verificar correção
   - Relatórios claros

**Desvantagens** (aceitas):
- Fase extra de compilação
- Overhead de conversão

**Decisão**: Vantagens superam desvantagens para fins **didáticos**.

---

### P10: "Como escolheu representação de float (16-bit half precision)?"

**Resposta**:

**Limitações do Arduino Uno**:
- Memória limitada (2KB RAM)
- Processador 8-bit
- Sem FPU (hardware float)

**Opções consideradas**:
1. **32-bit IEEE 754** (float padrão)
   - ❌ Muito grande para 8-bit
   - ❌ Operações lentas via software

2. **16-bit half precision**
   - ✓ Compacto (2 bytes)
   - ✓ Suficiente para fins didáticos
   - ✓ Mais rápido que 32-bit

3. **Fixed-point**
   - ✓ Mais rápido
   - ❌ Range limitado
   - ❌ Mais complexo de implementar

**Decisão**: **16-bit half precision** balanceia precisão e performance.

**Referência**: `docs/RA4/convencoes_registradores.md`

---

## Perguntas Sobre Testes

### P11: "Por que fatorial, fibonacci e taylor?"

**Resposta**:

Cada teste demonstra aspectos diferentes:

**fatorial.txt**:
- ✓ Loop WHILE simples
- ✓ Multiplicação acumulada
- ✓ Contador incremental
- ✓ Teste de condição `<=`
- **Conceito**: Iteração básica

**fibonacci.txt**:
- ✓ Múltiplas atribuições no loop
- ✓ "Sliding window" (janela deslizante)
- ✓ Dependências de dados
- **Conceito**: Estado complexo

**taylor.txt**:
- ✓ Aritmética de ponto flutuante
- ✓ Muitas variáveis temporárias
- ✓ Divisão real
- ✓ Expressões matemáticas complexas
- **Conceito**: Computação científica

**Juntos, cobrem**:
- Tipos: int e real
- Operadores: aritméticos e relacionais
- Estruturas: WHILE
- Complexidade: baixa, média, alta

---

**Use este documento para praticar respostas antes da defesa!**
