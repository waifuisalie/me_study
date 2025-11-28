# RA1: Walkthrough de Análise Léxica

## Teoria Resumida

**Análise Léxica** = Primeira fase do compilador que converte texto em tokens.

- **Entrada**: Código fonte (string)
- **Saída**: Sequência de tokens
- **Método**: Máquina de estados (autômato finito)

### O que é um Token?

**Token** = Unidade léxica com significado

```python
class Token:
    tipo: Tipo_de_Token    # NUMERO_INTEIRO, SOMA, VARIAVEL, etc.
    valor: str              # "5", "+", "COUNTER", etc.
```

### Tipos de Token (20+)

```
Números:
  - NUMERO_INTEIRO: 1, 42, 100
  - NUMERO_REAL: 3.14, 0.5, 2.0

Operadores:
  - SOMA (+), SUBTRACAO (-), MULTIPLICACAO (*)
  - DIVISAO_INTEIRA (/), DIVISAO_REAL (|)
  - RESTO (%), POTENCIA (^)

Relacionais:
  - MENOR (<), MAIOR (>), IGUAL (==)
  - MENOR_IGUAL (<=), MAIOR_IGUAL (>=), DIFERENTE (!=)

Lógicos:
  - AND (&&), OR (||), NOT (!)

Estruturas:
  - WHILE, FOR, IFELSE

Especiais:
  - RES, MEM
  - ABRE_PARENTESES ((), FECHA_PARENTESES ())
  - VARIAVEL (identificadores maiúsculos)
```

---

## Máquina de Estados

### Estados do Tokenizador

```
estado_zero() → Ponto de entrada
  ├─> '(' ou ')' → Token de parêntese
  ├─> Letra → estado_comando()
  ├─> Dígito → estado_numero()
  └─> Operador → estado_operador()

estado_numero() → Lê números
  ├─> Dígitos → continua
  ├─> '.' → muda para real
  └─> Espaço/fim → finaliza token

estado_comando() → Lê palavras
  ├─> Letras maiúsculas → VARIAVEL
  ├─> "while", "for", "ifelse" → Palavra-chave
  └─> "res", "mem" → Comando especial

estado_operador() → Lê operadores
  ├─> '+', '-', '*', etc → Operador simples
  └─> '==', '<=', '>=', '!=', '&&', '||' → Operador duplo
```

---

## Exemplo Passo-a-Passo

### Entrada: `(5 COUNTER +)`

```
Posição  Caractere  Estado          Ação
0        '('        estado_zero     → Token: ABRE_PARENTESES
1        '5'        estado_numero   → Lê '5'
2        ' '        estado_numero   → Finaliza: Token NUMERO_INTEIRO(5)
3        'C'        estado_comando  → Lê 'C'
4        'O'        estado_comando  → Lê 'O'
5        'U'        estado_comando  → Lê 'U'
6        'N'        estado_comando  → Lê 'N'
7        'T'        estado_comando  → Lê 'T'
8        'E'        estado_comando  → Lê 'E'
9        'R'        estado_comando  → Lê 'R'
10       ' '        estado_comando  → Finaliza: Token VARIAVEL(COUNTER)
11       '+'        estado_operador → Token: SOMA
12       ')'        estado_zero     → Token: FECHA_PARENTESES
```

### Saída

```
[
  Token(ABRE_PARENTESES, '('),
  Token(NUMERO_INTEIRO, '5'),
  Token(VARIAVEL, 'COUNTER'),
  Token(SOMA, '+'),
  Token(FECHA_PARENTESES, ')')
]
```

---

## Código Principal

**Arquivo**: `src/RA1/functions/python/analisador_lexico.py`

### Função `parseExpressao()`

```python
def parseExpressao(expressao):
    tokens = []
    i = 0

    while i < len(expressao):
        char = expressao[i]

        # Pula espaços
        if char.isspace():
            i += 1
            continue

        # Parênteses
        if char in '()':
            tokens.append(Token(tipo_parentese(char), char))
            i += 1

        # Números
        elif char.isdigit():
            token, i = estado_numero(expressao, i)
            tokens.append(token)

        # Letras (comandos/variáveis)
        elif char.isalpha():
            token, i = estado_comando(expressao, i)
            tokens.append(token)

        # Operadores
        else:
            token, i = estado_operador(expressao, i)
            tokens.append(token)

    return tokens
```

### Função `estado_numero()`

```python
def estado_numero(texto, pos):
    numero = ""
    eh_real = False

    while pos < len(texto):
        char = texto[pos]

        if char.isdigit():
            numero += char
            pos += 1
        elif char == '.' and not eh_real:
            numero += char
            eh_real = True
            pos += 1
        else:
            break

    tipo = NUMERO_REAL if eh_real else NUMERO_INTEIRO
    return Token(tipo, numero), pos
```

---

## Interface com RA2

**Saída**: `outputs/RA1/tokens/tokens_gerados.txt`

**Formato**:
```
Linha 1: (  5  COUNTER  +  )
Linha 2: (  8  LIMIT  )
```

**Contrato**:
- Um token por elemento separado por espaços
- Preserva ordem dos tokens
- RA2 lê esse arquivo e reconstrói os tokens

---

## Perguntas Típicas

### P1: "Qual a diferença entre lexema e token?"

**R**:
- **Lexema** = Sequência de caracteres no texto fonte (ex: "123", "COUNTER")
- **Token** = Classificação do lexema (ex: NUMERO_INTEIRO, VARIAVEL)

```
Lexema: "42"    → Token: NUMERO_INTEIRO
Lexema: "+"     → Token: SOMA
Lexema: "COUNT" → Token: VARIAVEL
```

### P2: "Por que usar máquina de estados?"

**R**: Porque permite reconhecer padrões complexos de forma **determinística** e **eficiente**:
- Números com ponto decimal
- Operadores de dois caracteres (`==`, `<=`)
- Palavras-chave vs variáveis

### P3: "Como diferencia variável de palavra-chave?"

**R**: Após ler toda a palavra, verifica contra lista de palavras-chave:

```python
if palavra in ['while', 'for', 'ifelse', 'res', 'mem']:
    return Token(PALAVRA_CHAVE, palavra)
else:
    return Token(VARIAVEL, palavra)
```

---

**Ver também**: `ARQUITETURA_COMPILADOR.md` para visão geral
**Próximo**: `RA2_WALKTHROUGH.md` - Análise Sintática
