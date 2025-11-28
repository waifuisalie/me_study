# Cenários de Demonstração para Defesa

## Cenário 1: Trace Completo de fatorial.txt

### Preparação

Antes da demonstração, tenha aberto:
- `inputs/RA4/fatorial.txt` (código fonte)
- `outputs/RA1/tokens/tokens_gerados.txt` (tokens)
- `outputs/RA2/arvore_sintatica.json` (AST)
- `outputs/RA3/arvore_atribuida.json` (AST com tipos)
- `outputs/RA4/tac_instructions.json` (TAC original)
- `outputs/RA4/tac_otimizado.json` (TAC otimizado)

### Passo 1: Mostrar Código Fonte

```
(1 COUNTER)                   # Linha 1
(1 RESULT)                    # Linha 2
(8 LIMIT)                     # Linha 3
((COUNTER LIMIT <=)           # Linha 4 (início)
  (((RESULT COUNTER *) RESULT))
  ((COUNTER 1 +) COUNTER)
  WHILE)
```

**Explicar**:
- Inicialização: COUNTER=1, RESULT=1, LIMIT=8
- Loop WHILE com condição `COUNTER <= LIMIT`
- Corpo: multiplica RESULT por COUNTER, incrementa COUNTER

### Passo 2: Tokens (RA1)

Mostrar `tokens_gerados.txt`:
```
Linha 1: (  1  COUNTER  )
Linha 2: (  1  RESULT  )
Linha 3: (  8  LIMIT  )
Linha 4: (  (  COUNTER  LIMIT  <=  )  ...
```

**Perguntas esperadas**:
- "Como o tokenizador identifica variáveis?" → Maiúsculas
- "E números reais?" → Presença de ponto decimal

### Passo 3: AST (RA2)

Mostrar estrutura JSON:
```json
{
  "linhas": [
    {
      "numero_linha": 1,
      "arvore": {
        "label": "LINHA",
        "filhos": [ ... ]
      }
    }
  ]
}
```

**Perguntas esperadas**:
- "Como constrói a árvore?" → Parser LL(1) stack-based
- "Mostre a tabela LL(1)" → Abrir `construirTabelaLL1.py` output

### Passo 4: Tipos (RA3)

Mostrar `arvore_atribuida.json`:
```json
{
  "tipo_vertice": "ARITH_OP",
  "operador": "*",
  "tipo": "int",  ← CAMPO ADICIONADO
  "filhos": [...]
}
```

**Perguntas esperadas**:
- "Como infere tipo de `*`?" → Verifica tipos dos filhos
- "E se fosse `(5 3.0 *)`?" → Promoveria para real

### Passo 5: TAC Original (RA4a)

Mostrar `tac_instructions.json`:
```
COUNTER = 1
RESULT = 1
LIMIT = 8
L0:
    t3 = COUNTER <= LIMIT
    ifFalse t3 goto L1
    t4 = RESULT * COUNTER
    RESULT = t4
    t6 = COUNTER + 1
    COUNTER = t6
    goto L0
L1:
```

**Contar**: 17 instruções

**Perguntas esperadas**:
- "Por que gera temporários?" → Formato TAC (3 endereços)
- "Como gera labels?" → TACManager incrementa contador

### Passo 6: TAC Otimizado (RA4b)

Mostrar `tac_otimizado.json`:
```
COUNTER = 1
RESULT = 1
LIMIT = 8
L0:
    t3 = COUNTER <= LIMIT
    ifFalse t3 goto L1
    RESULT = RESULT * COUNTER   ← t4 eliminado
    COUNTER = COUNTER + 1       ← t6 eliminado
    goto L0
L1:
```

**Contar**: 12 instruções (redução de 29.4%)

**Perguntas esperadas**:
- "Qual otimização removeu t4?" → Dead code elimination
- "Por que não remove t3?" → É usado no ifFalse

### Passo 7: Relatório de Otimização

Abrir `outputs/RA4/relatorios/otimizacao_tac.md`:

```markdown
## Estatísticas Gerais
- Instruções originais: 17
- Instruções otimizadas: 12
- Redução: 29.4%
- Iterações: 2
```

---

## Cenário 2: Demonstração de Otimização

### Código de Teste Simples

Criar temporariamente:
```
(2 X)
(3 Y)
(X Y +) Z)
```

### TAC Original
```
X = 2
Y = 3
t0 = X + Y
Z = t0
```

### Após Propagação
```
X = 2
Y = 3
t0 = 2 + 3    ← X e Y substituídos
Z = t0
```

### Após Folding
```
X = 2
Y = 3
t0 = 5        ← Avaliado
Z = t0
```

### Após Dead Code
```
t0 = 5        ← X, Y removidos
Z = t0
```

**Demonstrar**:
1. Executar compilador com esse input
2. Mostrar TAC original vs otimizado
3. Explicar cada passada

---

## Cenário 3: Demonstração de Erro Semântico

### Erro de Tipo

**Código**:
```
(5 X)
("texto" Y)   ← String não suportada
(X Y +)       ← Erro: int + string
```

**Executar compilador**:
- RA1: ✓ Aceita
- RA2: ✓ Aceita (sintaxe correta)
- RA3: ❌ **REJEITA**

**Mostrar**: `outputs/RA3/relatorios/erros_semanticos.md`

```markdown
## Erros Encontrados

Erro na linha 3:
  Tipo: Incompatibilidade de tipos
  Mensagem: Tipos incompatíveis na operação +: int e string
```

### Variável Não Inicializada

**Código**:
```
(X 5 +)   ← X não definido
```

**Executar**:
- RA3: ❌ Erro: "Variável X usada antes de inicialização"

**Mostrar código**: `analisador_memoria_controle.py:100-150`

---

## Cenário 4: Demonstração de Parsing LL(1)

### Mostrar Tabela LL(1)

Executar com verbose mode (se houver) ou mostrar output de:
```python
python -c "from src.RA2.functions.python.construirTabelaLL1 import *;
           tabela = construirTabelaLL1(); print(tabela)"
```

### Trace Manual

**Entrada**: `(5 3 +)`

**Preparar slides ou papel**:
```
Pilha: [$, PROGRAM]
Entrada: [(, 5, 3, +, ), $]

Ação: M[PROGRAM, (] = PROGRAM → LINHA PROGRAM_PRIME
Nova pilha: [$, PROGRAM_PRIME, LINHA]
...
(continuar até fim)
```

**Usar**: Quadro ou slides com animação

---

## Questões Que Podem Surgir

### Durante Cenário 1

**P**: "Mostre onde no código a AST é gerada"
**R**: Abrir `src/RA2/functions/python/gerarArvore.py:30-150`

**P**: "Como sabe que COUNTER é int?"
**R**: RA3 infere de `(1 COUNTER)` → 1 é int, logo COUNTER é int

### Durante Cenário 2

**P**: "E se tivesse loop?"
**R**: Variáveis mudam no loop, não dá para propagar

**P**: "Por que não inline t0 em Z = t0?"
**R**: Poderia! Mais uma otimização possível (copy propagation)

### Durante Cenário 3

**P**: "Dá para adicionar suporte a strings?"
**R**: Sim, requer:
  - Novo tipo em RA3
  - Novas instruções TAC
  - Tratamento em Assembly

### Durante Cenário 4

**P**: "E se houver ambiguidade na gramática?"
**R**: Tabela teria conflito → gramática não seria LL(1)

---

## Dicas para Apresentação

1. **Tenha todos arquivos abertos antes**
2. **Use split screen** (código + saída)
3. **Prepare dados em variáveis** para mostrar rapidamente
4. **Tenha calculadora** para verificar resultados
5. **Saiba números de linha** dos códigos importantes
6. **Pratique transições** entre arquivos

---

## Checklist Pré-Defesa

- [ ] Compilar os 3 testes (fatorial, fibonacci, taylor)
- [ ] Verificar todos outputs existem
- [ ] Testar com erro semântico intencional
- [ ] Revisar números de linha dos códigos
- [ ] Preparar exemplos extras (otimização)
- [ ] Testar explicação verbal de cada fase

---

**Pratique cada cenário pelo menos 3 vezes antes da defesa!**
