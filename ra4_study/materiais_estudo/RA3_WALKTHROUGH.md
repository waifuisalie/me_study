# RA3: Walkthrough de Análise Semântica

## Teoria Resumida

**Análise Semântica** = Verifica correção de tipos e valida uso de variáveis.

- **Entrada**: AST (de RA2)
- **Saída**: AST Atribuída (com tipos)
- **Método**: 3 fases de análise

---

## Sistema de Tipos

### Tipos Primitivos

```python
TYPE_INT = 'int'        # Números inteiros
TYPE_REAL = 'real'      # Ponto flutuante
TYPE_BOOLEAN = 'boolean' # Resultado de comparações
```

### Hierarquia e Promoção

```
int < real  (int promovido para real em operações mistas)
boolean (sem conversão automática)
```

**Regras de Promoção**:
```
int    + int    = int
int    + real   = real   (int → real)
real   + int    = real   (int → real)
real   + real   = real

int    < int    = boolean
real   < real   = boolean

boolean && boolean = boolean
```

---

## 3 Fases da Análise Semântica

### Fase 1: Verificação de Tipos

**Arquivo**: `analisador_tipos.py`

**O que faz**:
- Infere tipos de expressões (bottom-up)
- Valida compatibilidade de operadores
- Promove tipos quando necessário

**Algoritmo**:
```python
def inferir_tipo(no):
    # Caso base: literais
    if no.label == 'numero_inteiro':
        return 'int'
    if no.label == 'numero_real':
        return 'real'

    # Caso recursivo: operadores
    if no.label == 'ARITH_OP':
        tipo_esq = inferir_tipo(filho_esquerdo)
        tipo_dir = inferir_tipo(filho_direito)
        return promover_tipo(tipo_esq, tipo_dir)

    # Operadores relacionais sempre retornam boolean
    if no.label == 'COMP_OP':
        return 'boolean'
```

### Fase 2: Análise de Memória

**Arquivo**: `analisador_memoria_controle.py`

**O que faz**:
- Verifica se variáveis foram inicializadas antes do uso
- Detecta variáveis não utilizadas
- Valida comandos MEM

**Exemplo de erro**:
```
(X 5 +)   ❌ Erro: X não foi inicializado

Correto:
(5 X)     ✓ Inicializa X
(X 3 +)   ✓ Agora X pode ser usado
```

### Fase 3: Análise de Controle de Fluxo

**Arquivo**: `analisador_memoria_controle.py`

**O que faz**:
- Valida estruturas WHILE/FOR/IFELSE
- Verifica se condições são booleanas
- Valida comando RES (referência a resultado anterior)

**Exemplo**:
```
((COUNTER LIMIT <=) ... WHILE)
     ↑
  Deve ser boolean ✓
```

---

## Exemplo: `(5 3.0 +)`

### Passo 1: AST de Entrada (RA2)

```json
{
  "label": "LINHA",
  "filhos": [
    {"label": "("},
    {
      "label": "ARITH_OP",
      "operador": "+",
      "filhos": [
        {"label": "5"},
        {"label": "3.0"}
      ]
    },
    {"label": ")"}
  ]
}
```

### Passo 2: Inferência de Tipos (Bottom-Up)

```
1. "5" → int
2. "3.0" → real
3. "+" com (int, real):
   - Promove int → real
   - Resultado: real
```

### Passo 3: AST Atribuída (Saída)

```json
{
  "tipo_vertice": "LINHA",
  "tipo": "real",  ← ADICIONADO
  "filhos": [
    {"label": "("},
    {
      "tipo_vertice": "ARITH_OP",
      "operador": "+",
      "tipo": "real",  ← ADICIONADO
      "filhos": [
        {
          "subtipo": "numero_inteiro",
          "valor": "5",
          "tipo": "int"  ← ADICIONADO
        },
        {
          "subtipo": "numero_real",
          "valor": "3.0",
          "tipo": "real"  ← ADICIONADO
        }
      ]
    },
    {"label": ")"}
  ]
}
```

---

## Tabela de Símbolos

### Estrutura

```python
@dataclass
class SimboloInfo:
    nome: str                 # "COUNTER", "RESULT"
    tipo: str                 # "int", "real"
    inicializada: bool        # Foi atribuída?
    linha_declaracao: int     # Onde foi definida
    linha_ultimo_uso: int     # Última referência
```

### Operações

```python
# Adicionar símbolo
tabela.adicionar_simbolo('COUNTER', 'int')

# Verificar inicialização
if not tabela.eh_inicializado('COUNTER'):
    raise ErroSemantico("COUNTER não inicializado")

# Obter tipo
tipo = tabela.obter_tipo('COUNTER')  # 'int'
```

### Exemplo de Tabela

```
| Variável | Tipo | Inicializada | Linha Decl | Último Uso |
|----------|------|--------------|------------|------------|
| COUNTER  | int  | Sim          | 1          | 15         |
| RESULT   | int  | Sim          | 2          | 20         |
| LIMIT    | int  | Sim          | 3          | 10         |
```

---

## Validações Semânticas

### 1. Tipos Compatíveis

```
✓ (5 3 +)              int + int = int
✓ (5.0 3.0 +)          real + real = real
✓ (5 3.0 +)            int + real = real (promoção)
❌ (5 true +)           int + boolean (incompatível)
```

### 2. Operadores Relacionais

```
✓ (5 3 <)              int < int = boolean
✓ (5.0 3.0 >=)         real >= real = boolean
❌ (5 3 <) (2 +)        boolean + int (incompatível)
```

### 3. Inicialização de Variáveis

```
❌ (X 5 +)              X não definido
✓ (5 X) (X 3 +)        X definido na linha anterior
```

---

## Relatórios Gerados

### 1. arvore_atribuida.md

Árvore com tipos anotados (visual)

```
LINHA [tipo: int]
├── (
├── ARITH_OP(+) [tipo: int]
│   ├── 5 [tipo: int]
│   └── 3 [tipo: int]
└── )
```

### 2. julgamento_tipos.md

Decisões de tipagem

```
Linha 1: Expressão (5 3 +)
  - Operando esquerdo: int
  - Operando direito: int
  - Operador: +
  - Tipo resultante: int
  - Promoção: Nenhuma
```

### 3. tabela_simbolos.md

Variáveis declaradas

```
| Nome    | Tipo | Linha Decl | Inicializada |
|---------|------|------------|--------------|
| COUNTER | int  | 1          | Sim          |
| RESULT  | int  | 2          | Sim          |
```

### 4. erros_semanticos.md

Erros encontrados (se houver)

```
Erro na linha 5:
  Tipo: Variável não inicializada
  Mensagem: Variável 'X' usada antes de ser definida
```

---

## Código Principal

**Arquivo**: `analisador_semantico.py`

```python
def analisarSemantica(arvore_sintatica):
    # Fase 1: Tipos
    arvore_com_tipos = analisador_tipos.processar(arvore_sintatica)

    # Fase 2: Memória
    validar_memoria(arvore_com_tipos)

    # Fase 3: Controle
    validar_controle_fluxo(arvore_com_tipos)

    # Gera relatórios
    gerar_relatorios(arvore_com_tipos, tabela_simbolos)

    return arvore_com_tipos
```

---

## Interface com RA4

**Saída**: `outputs/RA3/arvore_atribuida.json`

**Adições importantes**:
- Campo `"tipo"` em cada nó da AST
- Informação de tipo é usada em RA4 para:
  - Gerar instruções TAC corretas
  - Escolher registradores apropriados
  - Gerar código Assembly adequado

---

## Perguntas Típicas

### P1: "Onde acontece a promoção de tipos?"

**R**: Em `analisador_tipos.py`, função `promover_tipo()`:

```python
def promover_tipo(tipo1, tipo2):
    if tipo1 == 'real' or tipo2 == 'real':
        return 'real'
    elif tipo1 == 'int' and tipo2 == 'int':
        return 'int'
    else:
        raise ErroSemantico("Tipos incompatíveis")
```

### P2: "Como a tabela de símbolos detecta variável não inicializada?"

**R**: Mantém flag `inicializada` para cada variável:

```python
# Ao atribuir (X = 5):
tabela.adicionar_simbolo('X', 'int')
tabela.marcar_inicializada('X')

# Ao usar (X + 3):
if not tabela.eh_inicializado('X'):
    raise ErroSemantico("X não inicializado")
```

### P3: "Qual a diferença entre tipo estático e dinâmico?"

**R**:
- **Estático** (nosso compilador): Tipos checados em **tempo de compilação**
- **Dinâmico** (Python, JavaScript): Tipos checados em **tempo de execução**

Nossa análise semântica faz **verificação estática** completa.

---

**Ver também**: `ARQUITETURA_COMPILADOR.md` - Fluxo de tipos
**Próximo**: `RA4_WALKTHROUGH.md` - Geração de Código
