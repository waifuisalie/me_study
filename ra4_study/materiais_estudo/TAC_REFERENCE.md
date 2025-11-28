# Referência de Instruções TAC

## Resumo Executivo

TAC (Three Address Code) = Código intermediário com **no máximo 3 endereços** por instrução.

**Formato Geral**: `resultado = operando1 operador operando2`

---

## Tipos de Instruções (12 tipos)

### 1. Atribuição Simples

```
dest = source
```

**Exemplo**:
```
COUNTER = 1
X = 5
```

**Classe Python**:
```python
TACAssignment(dest='COUNTER', source='1')
```

**Gerada por**: Expressões `(valor variavel)`

---

### 2. Cópia

```
dest = source
```

**Exemplo**:
```
Y = X
t1 = t0
```

**Classe Python**:
```python
TACCopy(dest='Y', source='X')
```

**Gerada por**: Propagação de valores

---

### 3. Operação Binária

```
result = operand1 operator operand2
```

**Exemplo**:
```
t0 = 5 + 3
t1 = COUNTER <= LIMIT
t2 = X * Y
```

**Operadores suportados**:
- Aritméticos: `+`, `-`, `*`, `/`, `%`, `^`, `|`
- Relacionais: `<`, `>`, `<=`, `>=`, `==`, `!=`
- Lógicos: `&&`, `||`

**Classe Python**:
```python
TACBinaryOp(
    result='t0',
    operand1='5',
    operator='+',
    operand2='3',
    data_type='int'
)
```

**Gerada por**: Todas operações binárias

---

### 4. Operação Unária

```
result = operator operand
```

**Exemplo**:
```
t0 = -X
t1 = !condition
```

**Operadores**: `-` (negação), `!` (NOT lógico)

**Classe Python**:
```python
TACUnaryOp(
    result='t0',
    operator='-',
    operand='X'
)
```

**Gerada por**: Operações unárias

---

### 5. Label

```
Label:
```

**Exemplo**:
```
L0:
L1:
LOOP_START:
```

**Classe Python**:
```python
TACLabel(label='L0')
```

**Gerada por**: Estruturas de controle (WHILE, FOR, IF)

---

### 6. Goto Incondicional

```
goto target
```

**Exemplo**:
```
goto L0
goto LOOP_START
```

**Classe Python**:
```python
TACGoto(target='L0')
```

**Gerada por**: Final de loop, fim de bloco IF

---

### 7. Goto Condicional (If)

```
if condition goto target
```

**Exemplo**:
```
if t0 goto L1
```

**Significado**: Se `condition` é **verdadeiro** (≠ 0), pula para `target`

**Classe Python**:
```python
TACIfGoto(
    condition='t0',
    target='L1'
)
```

**Gerada por**: Estruturas IF

---

### 8. Goto Condicional Negado (IfFalse)

```
ifFalse condition goto target
```

**Exemplo**:
```
ifFalse t0 goto L1
```

**Significado**: Se `condition` é **falso** (= 0), pula para `target`

**Classe Python**:
```python
TACIfFalseGoto(
    condition='t0',
    target='L1'
)
```

**Gerada por**: Loops WHILE (sai se condição falsa)

---

### 9. Chamada de Função

```
result = call function_name, param_count
```

**Exemplo**:
```
t0 = call print, 1
result = call factorial, 1
```

**Classe Python**:
```python
TACCall(
    result='t0',
    function_name='print',
    param_count=1
)
```

**Gerada por**: Chamadas de função (futuro)

---

### 10. Return

```
return value
```

**Exemplo**:
```
return t0
return 42
```

**Classe Python**:
```python
TACReturn(value='t0')
```

**Gerada por**: Fim de função (futuro)

---

### 11. Leitura de Memória

```
result = MEM[address]
```

**Exemplo**:
```
t0 = MEM[X]
value = MEM[0x1000]
```

**Classe Python**:
```python
TACMemoryRead(
    result='t0',
    address='X'
)
```

**Gerada por**: Comando `(MEM)`

---

### 12. Escrita em Memória

```
MEM[address] = value
```

**Exemplo**:
```
MEM[X] = t0
MEM[0x1000] = 42
```

**Classe Python**:
```python
TACMemoryWrite(
    address='X',
    value='t0'
)
```

**Gerada por**: Comando `(valor MEM)`

---

## Convenções de Nomenclatura

### Temporários
```
t0, t1, t2, t3, ...
```
- Gerados automaticamente
- Escopo local
- Podem ser otimizados

### Labels
```
L0, L1, L2, L3, ...
```
- Gerados automaticamente
- Marcam pontos de salto

### Variáveis do Usuário
```
COUNTER, RESULT, LIMIT, FIB_NEXT, ...
```
- Maiúsculas
- Definidas pelo código fonte
- **Nunca** são removidas por otimização

---

## Exemplos de Código Completos

### Exemplo 1: Expressão Aritmética

**Código fonte**: `(5 3 +)`

**TAC gerado**:
```
t0 = 5 + 3
```

---

### Exemplo 2: Atribuição

**Código fonte**: `(5 COUNTER)`

**TAC gerado**:
```
COUNTER = 5
```

---

### Exemplo 3: Loop WHILE

**Código fonte**:
```
((COUNTER LIMIT <=)
  (((RESULT COUNTER *) RESULT))
  ((COUNTER 1 +) COUNTER)
  WHILE)
```

**TAC gerado**:
```
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

**Estrutura**:
1. `L0`: Início do loop
2. Avalia condição
3. `ifFalse`: Sai se condição falsa
4. Executa corpo
5. `goto L0`: Volta ao início
6. `L1`: Fim do loop

---

## TAC → Assembly (Mapeamento)

| TAC | Assembly AVR | Descrição |
|-----|--------------|-----------|
| `t0 = 5` | `ldi r16, 5`<br/>`sts t0, r16` | Carrega constante |
| `t0 = X + Y` | `lds r16, X`<br/>`lds r17, Y`<br/>`add r16, r17`<br/>`sts t0, r16` | Soma |
| `X = Y` | `lds r16, Y`<br/>`sts X, r16` | Cópia |
| `goto L1` | `rjmp L1` | Salto relativo |
| `ifFalse t0 goto L1` | `lds r16, t0`<br/>`cpi r16, 0`<br/>`breq L1` | Salto condicional |
| `L0:` | `L0:` | Label |

---

## Fase de Geração

| Instrução | Gerada em | Por quê |
|-----------|-----------|---------|
| Atribuição | RA4a | Expressões `(valor variavel)` |
| Binária | RA4a | Operadores `+`, `-`, `*`, etc. |
| Unária | RA4a | Operadores `-`, `!` |
| Label | RA4a | Estruturas WHILE, FOR, IF |
| Goto | RA4a | Controle de fluxo |
| IfGoto/IfFalseGoto | RA4a | Condicionais |
| Call | RA4a | Funções (futuro) |
| Return | RA4a | Retorno de função (futuro) |
| Memory | RA4a | Comandos MEM |

---

## Otimizações Aplicadas

| Instrução | Pode ser otimizada? | Como |
|-----------|---------------------|------|
| `t0 = 2 + 3` | ✅ Sim | Constant folding → `t0 = 5` |
| `t0 = X + Y` onde X=5 | ✅ Sim | Propagation → `t0 = 5 + Y` |
| `t0 = 5` (nunca usado) | ✅ Sim | Dead code → Removido |
| `RESULT = 5` | ❌ Não | Variável do usuário |
| `goto L1`<br/>`L1:` | ✅ Sim | Jump elimination |
| `L0:` (nunca usado) | ✅ Sim | Dead label |

---

## Checklist de Validação

Ao revisar TAC, verifique:

- [ ] Cada instrução tem no máximo 3 endereços
- [ ] Temporários começam com `t`
- [ ] Labels terminam com `:`
- [ ] Todos os labels usados são definidos
- [ ] Variáveis são inicializadas antes do uso
- [ ] Tipos são consistentes (int, real, boolean)
- [ ] Estruturas de controle têm labels corretamente

---

**Use esta referência durante implementação e debugging!**
