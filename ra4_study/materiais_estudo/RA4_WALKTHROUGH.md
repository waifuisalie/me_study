# RA4: Walkthrough de Geração de Código

## Visão Geral

RA4 tem **4 sub-fases**:
1. **Geração de TAC** - AST → TAC
2. **Otimização de TAC** - TAC → TAC otimizado
3. **Geração de Assembly** - TAC → Assembly AVR
4. **Compilação/Upload** - Assembly → Arduino

---

## Fase 4a: Geração de TAC

### Three Address Code (TAC)

**Formato**: Cada instrução tem no máximo 3 endereços

```
t0 = 5              # Atribuição simples
t1 = t0 + 3         # Operação binária
t2 = -t1            # Operação unária
goto L1             # Salto incondicional
ifFalse t2 goto L1  # Salto condicional
L1:                 # Label
```

### Algoritmo: Travessia Pós-Ordem

```python
def gerar_tac(no):
    # Caso base: literais
    if eh_literal(no):
        return no.valor

    # Caso recursivo: operadores
    if eh_operador_binario(no):
        # Processa filhos primeiro (pós-ordem)
        esq = gerar_tac(filho_esquerdo)
        dir = gerar_tac(filho_direito)

        # Gera temporário
        temp = novo_temp()  # t0, t1, t2, ...

        # Emite instrução
        emitir(f"{temp} = {esq} {op} {dir}")

        return temp
```

### Exemplo: `(5 3 +)`

```
Processamento:
1. Processa filho esquerdo: 5 → retorna "5"
2. Processa filho direito: 3 → retorna "3"
3. Gera temporário: t0
4. Emite: t0 = 5 + 3
5. Retorna: t0

TAC gerado:
  t0 = 5 + 3
```

### Estruturas de Controle: WHILE

**Entrada**:
```
((COUNTER LIMIT <=) (corpo...) WHILE)
```

**TAC gerado**:
```
L0:                         # Início do loop
    t0 = COUNTER <= LIMIT   # Avalia condição
    ifFalse t0 goto L1      # Sai se falso
    ... (corpo do loop)
    goto L0                 # Volta ao início
L1:                         # Fim do loop
```

### Gerenciamento de Temporários e Labels

**Arquivo**: `tac_manager.py`

```python
class TACManager:
    temp_counter = 0
    label_counter = 0

    @staticmethod
    def novo_temp():
        temp = f"t{TACManager.temp_counter}"
        TACManager.temp_counter += 1
        return temp  # t0, t1, t2, ...

    @staticmethod
    def novo_label():
        label = f"L{TACManager.label_counter}"
        TACManager.label_counter += 1
        return label  # L0, L1, L2, ...
```

---

## Fase 4b: Otimização de TAC

**Ver**: `RA4_OTIMIZACAO_TAC.md` para detalhes completos

### 4 Técnicas Aplicadas

1. **Constant Folding**: `t0 = 2 + 3` → `t0 = 5`
2. **Constant Propagation**: `t1 = 5; t2 = t1 + 3` → `t2 = 5 + 3`
3. **Dead Code Elimination**: Remove temporários não usados
4. **Jump Elimination**: Remove saltos redundantes

### Exemplo de Otimização

```
ANTES (5 instruções):        DEPOIS (3 instruções):
t0 = 2                       t2 = 5
t1 = 3                       RESULT = t2
t2 = t0 + t1
RESULT = t2
```

---

## Fase 4c: Geração de Assembly

### Registradores AVR

```
R16-R23: Temporários
R24-R25: Parâmetros/retorno
R26-R27 (X): Ponteiro de memória
R28-R29 (Y): Frame pointer
R30-R31 (Z): Stack pointer
```

### TAC → Assembly: Exemplos

#### Atribuição Simples

```
TAC:  t0 = 5

Assembly:
  ldi r16, 5        ; Carrega 5 em R16
  sts t0, r16       ; Armazena em memória (t0)
```

#### Operação Binária

```
TAC:  t2 = t0 + t1

Assembly:
  lds r16, t0       ; Carrega t0
  lds r17, t1       ; Carrega t1
  add r16, r17      ; Soma
  sts t2, r16       ; Armazena resultado
```

#### Salto Condicional

```
TAC:  ifFalse t0 goto L1

Assembly:
  lds r16, t0       ; Carrega condição
  cpi r16, 0        ; Compara com 0
  breq L1           ; Pula se igual (falso)
```

#### Label

```
TAC:  L0:

Assembly:
  L0:               ; Label direta
```

### Alocação de Registradores

**Estratégia**:
1. Variáveis frequentes → Registradores fixos
2. Temporários → R24-R25 (reutilizados)
3. Spilling → Memória (se necessário)

**Arquivo**: `gerador_assembly.py`

```python
def alocar_registrador(variavel):
    if variavel in registro_fixo:
        return registro_fixo[variavel]
    else:
        return 'r24'  # Temporário padrão
```

---

## Fase 4d: Compilação e Upload

### Ferramentas AVR

**avr-gcc**: Compilador Assembly → ELF
```bash
avr-gcc -mmcu=atmega328p -o programa.elf programa.s
```

**avr-objcopy**: ELF → HEX
```bash
avr-objcopy -O ihex programa.elf programa.hex
```

**avrdude**: Upload HEX → Arduino
```bash
avrdude -c arduino -p atmega328p -P COM3 -U flash:w:programa.hex
```

### Detecção Automática

**Arquivo**: `arduino_tools.py`

```python
def check_avr_toolchain():
    # Procura avr-gcc em:
    # - PATH
    # - Arduino IDE
    # - MSYS2
    # - WinAVR

def detect_arduino_port():
    # Enumera portas seriais
    # Detecta Arduino Uno (VID:PID)
    return 'COM3'  # ou /dev/ttyUSB0
```

---

## Exemplo Completo: fatorial.txt

### Código Fonte

```
(1 COUNTER)
(1 RESULT)
(8 LIMIT)
((COUNTER LIMIT <=)
  (((RESULT COUNTER *) RESULT))
  ((COUNTER 1 +) COUNTER)
  WHILE)
```

### TAC Original (17 instruções)

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

### TAC Otimizado (12 instruções)

```
COUNTER = 1
RESULT = 1
LIMIT = 8
L0:
    t3 = COUNTER <= LIMIT
    ifFalse t3 goto L1
    RESULT = RESULT * COUNTER    ← Inline (t4 removido)
    COUNTER = COUNTER + 1        ← Inline (t6 removido)
    goto L0
L1:
```

### Assembly AVR (exemplo simplificado)

```asm
; Inicialização
    ldi r16, 1
    sts COUNTER, r16
    sts RESULT, r16
    ldi r16, 8
    sts LIMIT, r16

; Loop
L0:
    lds r16, COUNTER
    lds r17, LIMIT
    cp r16, r17          ; Compara
    brsh L1              ; Pula se maior

    lds r16, RESULT
    lds r17, COUNTER
    mul r16, r17         ; Multiplica
    sts RESULT, r0       ; Armazena

    lds r16, COUNTER
    inc r16              ; Incrementa
    sts COUNTER, r16

    rjmp L0              ; Volta

L1:
    ; Fim
```

---

## Instruções TAC Completas

### 1. Atribuição

```python
TACAssignment(dest, source)
# Exemplo: COUNTER = 1
```

### 2. Operação Binária

```python
TACBinaryOp(result, operand1, operator, operand2)
# Exemplo: t0 = 5 + 3
```

### 3. Operação Unária

```python
TACUnaryOp(result, operator, operand)
# Exemplo: t1 = -t0
```

### 4. Label

```python
TACLabel(label)
# Exemplo: L0:
```

### 5. Goto

```python
TACGoto(target)
# Exemplo: goto L0
```

### 6. Salto Condicional

```python
TACIfGoto(condition, target)
# Exemplo: if t0 goto L1

TACIfFalseGoto(condition, target)
# Exemplo: ifFalse t0 goto L1
```

---

## Perguntas Típicas

### P1: "Por que pós-ordem na travessia?"

**R**: Para processar **operandos antes do operador**:

```
Árvore: +
       / \
      5   3

Pré-ordem: +, 5, 3   ❌ (operador antes de operandos)
Pós-ordem: 5, 3, +   ✓ (operandos primeiro)
```

### P2: "Como funciona o gerenciamento de temporários?"

**R**: Contador global incrementado:

```python
temp_counter = 0

def novo_temp():
    global temp_counter
    temp = f"t{temp_counter}"
    temp_counter += 1
    return temp

# Uso:
t0 = novo_temp()  # "t0"
t1 = novo_temp()  # "t1"
t2 = novo_temp()  # "t2"
```

### P3: "Como TAC facilita otimização?"

**R**: TAC é **linear** e **simples**:
- Cada instrução faz 1 coisa
- Fácil de analisar padrões
- Fácil de transformar
- Independente da arquitetura alvo

---

**Ver também**:
- `RA4_OTIMIZACAO_TAC.md` - Otimizações detalhadas
- `TAC_REFERENCE.md` - Referência de instruções
- `ARQUITETURA_COMPILADOR.md` - Pipeline completo
