# Referência Rápida: Teoria → Código

## Tabela Mestre

| Conceito Teórico | Fase | Arquivo | Função Principal | Linhas |
|------------------|------|---------|------------------|--------|
| **Análise Léxica** | | | | |
| Autômato Finito | RA1 | `analisador_lexico.py` | `parseExpressao()` | 50-200 |
| Estado Inicial | RA1 | `analisador_lexico.py` | `estado_zero()` | 100-130 |
| Estado Número | RA1 | `analisador_lexico.py` | `estado_numero()` | 140-170 |
| Tipos de Token | RA1 | `tokens.py` | `class Tipo_de_Token` | 10-60 |
| **Análise Sintática** | | | | |
| Gramática CFG | RA2 | `configuracaoGramatica.py` | `GRAMATICA_RPN` | 14-41 |
| Conjunto FIRST | RA2 | `calcularFirst.py` | `calcularFirst()` | 13-56 |
| Conjunto FOLLOW | RA2 | `calcularFollow.py` | `calcularFollow()` | 15-90 |
| Tabela LL(1) | RA2 | `construirTabelaLL1.py` | `construirTabelaLL1()` | 20-150 |
| Parser Descendente | RA2 | `parsear.py` | `parsear()` | 50-200 |
| Geração AST | RA2 | `gerarArvore.py` | `gerarArvore()` | 30-150 |
| **Análise Semântica** | | | | |
| Sistema de Tipos | RA3 | `tipos.py` | Constantes `TYPE_*` | 5-15 |
| Promoção de Tipos | RA3 | `analisador_tipos.py` | `promover_tipo()` | 120-145 |
| Inferência de Tipos | RA3 | `analisador_tipos.py` | `inferir_tipo()` | 150-300 |
| Tabela de Símbolos | RA3 | `tabela_simbolos.py` | `class TabelaSimbolos` | 20-150 |
| Verificação Semântica | RA3 | `analisador_semantico.py` | `analisarSemantica()` | 50-200 |
| Análise de Memória | RA3 | `analisador_memoria_controle.py` | `validar_memoria()` | 80-150 |
| **Geração de Código** | | | | |
| TAC Generation | RA4 | `gerador_tac.py` | `gerarTAC()` | 100-400 |
| Travessia Pós-Ordem | RA4 | `ast_traverser.py` | `traverse_postorder()` | 30-100 |
| Gerenciador TAC | RA4 | `tac_manager.py` | `class TACManager` | 10-80 |
| Instruções TAC | RA4 | `tac_instructions.py` | Classes de instruções | 1-300 |
| **Otimização** | | | | |
| Constant Folding | RA4 | `otimizador_tac.py` | `otimizar_constant_folding()` | 200-280 |
| Constant Propagation | RA4 | `otimizador_tac.py` | `otimizar_constant_propagation()` | 290-370 |
| Dead Code Elimination | RA4 | `otimizador_tac.py` | `otimizar_dead_code_elimination()` | 380-450 |
| Jump Elimination | RA4 | `otimizador_tac.py` | `otimizar_jump_elimination()` | 460-530 |
| Multi-Pass Loop | RA4 | `otimizador_tac.py` | `otimizarTAC()` | 103-170 |
| **Assembly** | | | | |
| Geração Assembly | RA4 | `gerador_assembly.py` | `gerar()` | 50-1400 |
| Alocação Registradores | RA4 | `gerador_assembly.py` | `_alocar_registrador()` | 200-350 |
| Ferramentas AVR | RA4 | `arduino_tools.py` | `check_avr_toolchain()` | 30-100 |

---

## Fluxo de Execução: Entrada → Saída

```
Entrada: (5 3 +)

RA1 (analisador_lexico.py:parseExpressao)
  → Tokens: ['(', '5', '3', '+', ')']
  → Arquivo: outputs/RA1/tokens/tokens_gerados.txt

RA2 (parsear.py:parsear)
  → AST hierárquica
  → Arquivo: outputs/RA2/arvore_sintatica.json

RA3 (analisador_semantico.py:analisarSemantica)
  → AST + tipos
  → Arquivo: outputs/RA3/arvore_atribuida.json

RA4a (gerador_tac.py:gerarTAC)
  → TAC: t0 = 5 + 3
  → Arquivo: outputs/RA4/tac_instructions.json

RA4b (otimizador_tac.py:otimizarTAC)
  → TAC: t0 = 8
  → Arquivo: outputs/RA4/tac_otimizado.json

RA4c (gerador_assembly.py:gerar)
  → Assembly AVR
  → Arquivo: outputs/RA4/*.s
```

---

## Perguntas Comuns → Respostas Rápidas

| Pergunta | Arquivo | Função/Linha |
|----------|---------|--------------|
| "Onde calcula FIRST?" | `calcularFirst.py` | Linha 13-56 |
| "Onde adiciona tipo ao nó?" | `analisador_tipos.py` | Linha 200-250 |
| "Como gera temporário?" | `tac_manager.py` | `novo_temp()` linha 25 |
| "Onde faz constant folding?" | `otimizador_tac.py` | Linha 200-280 |
| "Como aloca registrador?" | `gerador_assembly.py` | Linha 200-350 |
| "Onde detecta variável não inicializada?" | `analisador_memoria_controle.py` | Linha 100-150 |
| "Como constrói tabela LL(1)?" | `construirTabelaLL1.py` | Linha 20-150 |

---

## Estruturas de Dados Chave

### Token
```python
# tokens.py:20-35
class Token:
    tipo: Tipo_de_Token
    valor: str
```

### Nó AST
```python
# AST sintática (RA2)
{
    "label": "ARITH_OP",
    "filhos": [...]
}

# AST atribuída (RA3)
{
    "tipo_vertice": "ARITH_OP",
    "tipo": "int",  ← Adicionado
    "filhos": [...]
}
```

### Instrução TAC
```python
# tac_instructions.py:50-100
class TACBinaryOp:
    result: str
    operand1: str
    operator: str
    operand2: str
    line: int
    data_type: str
```

### Símbolo
```python
# tabela_simbolos.py:15-30
@dataclass
class SimboloInfo:
    nome: str
    tipo: str
    inicializada: bool
    linha_declaracao: int
```

---

## Comandos Úteis

### Executar Compilador
```bash
python compilador.py inputs/RA4/fatorial.txt
```

### Ver Tokens
```bash
cat outputs/RA1/tokens/tokens_gerados.txt
```

### Ver TAC Original
```bash
cat outputs/RA4/tac_output.md
```

### Ver TAC Otimizado
```bash
cat outputs/RA4/tac_otimizado.md
```

### Ver Relatório de Otimização
```bash
cat outputs/RA4/relatorios/otimizacao_tac.md
```

---

## Constantes Importantes

### Tipos
```python
# tipos.py
TYPE_INT = 'int'
TYPE_REAL = 'real'
TYPE_BOOLEAN = 'boolean'
```

### Gramática
```python
# configuracaoGramatica.py
SIMBOLO_INICIAL = 'PROGRAM'
```

### Otimização
```python
# otimizador_tac.py
MAX_ITERATIONS = 100
```

---

## Use Esta Tabela Durante a Defesa!

Imprima ou tenha em tablet para referência rápida quando professor perguntar:
- "Mostre o código de X"
- "Onde está implementado Y"
- "Como funciona Z"
