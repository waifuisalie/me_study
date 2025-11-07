# RA3 Defense Quick Reference Guide

**Last Updated**: 2025-11-06
**Purpose**: Quick-reference summary for defense preparation
**Full Details**: See `DEFENSE_QUESTIONS.md` for complete answers

---

## Critical Points to Remember

### 1. **Your RA3 Implementation is Complete**

You have **7 working modules**:
- `tipos.py` (560 lines) - Type system with 14 compatibility functions
- `tabela_simbolos.py` (529 lines) - Symbol table with initialization tracking
- `gramatica_atributos.py` (649 lines) - 23 semantic rules
- `analisador_tipos.py` (474 lines) - Type checker with post-order traversal
- `analisador_memoria_controle.py` (247 lines) - Memory/control validation
- `analisador_semantico.py` (511 lines) - Orchestrator (3-pass architecture)
- `gerador_arvore_atribuida.py` (1242 lines) - Report generator

**Total**: ~4,200 lines of implementation + 73 passing tests

---

## Key Design Decisions You Made

### Type System Asymmetry (Q1)

**Question**: Why does `/` require integers but `+` allows mixed types?

**Answer**: Hardware semantics
- `/` → Integer division (RISC-V `div` instruction, integers only)
- `+` → FPU or ALU (supports both int and float)
- Trade-off: Uniformity vs. hardware reality

**Code**: `tipos.py:177-201` (division) vs. `tipos.py:152-175` (general arithmetic)

---

### Permissive Truthiness (Q2)

**Question**: Why allow `int`/`real` as boolean in conditions?

**Answer**: Design choice for expressiveness
- C-style truthiness (0 = false, else = true)
- Allows `(X body WHILE)` instead of `(X 0 != body WHILE)`
- Trade-off: Conciseness vs. type safety

**Code**: `tipos.py:98-144` (`para_booleano`)

---

### Boolean Storage Prohibition (Q3)

**Question**: Why can't boolean be stored in memory via MEM?

**Answer**: Hardware-inspired memory model
- Variables = numeric registers (RISC-V x0-x31)
- Boolean = condition codes (flags, not registers)
- RES can reference booleans (temporary), MEM cannot (persistent)

**Code**: `tipos.py:329-352` (storage check), `tabela_simbolos.py:68` (validation)

---

### Type Mutation (Q6)

**Question**: Why allow variable type changes?

**Answer**: RPN is value-centric, not variable-centric
- Variables are **memory locations**, not **typed bindings**
- `(5 X MEM)` → X:int, then `(3.14 X MEM)` → X:real (allowed)
- Trade-off: Flexibility vs. type safety

**Code**: `tabela_simbolos.py:186-194` (allows type change)

---

### Initialization Tracking (Q7)

**Question**: Why `inicializada` boolean instead of Option type?

**Answer**: Pragmatic simplicity
- Python lacks native Option types
- Boolean separates concerns (status vs. value)
- Symbol table stores **status**, not **values** (values in `historico_tipos`)

**Code**: `tabela_simbolos.py:60` (field), `tabela_simbolos.py:281-306` (verification)

---

### Flat Scope (Q8)

**Question**: Why flat dictionary when `escopo` field exists?

**Answer**: Future-proofing
- RPN has no nested scopes (no functions/blocks)
- `escopo` field ready for language extension
- Would need stack of dicts to implement nested scopes

**Code**: `tabela_simbolos.py:126` (flat dict), `tabela_simbolos.py:61` (unused field)

---

### Lambda Semantic Actions (Q11)

**Question**: Why lambdas instead of named functions?

**Answer**: Co-location and simplicity
- Semantic action **right next to** rule definition
- Most actions are simple (3-5 lines)
- Trade-off: Readability vs. debuggability

**Code**: `gramatica_atributos.py:67-88` (lambda examples)

---

### 'ERROR' Sentinel (Q12)

**Question**: Why return 'ERROR' string instead of raising exception?

**Answer**: Separation of concerns + fault tolerance
- Rules are **declarative** (what is valid)
- Analyzer is **imperative** (enforces and handles)
- Allows collecting **all errors** in one pass

**Code**: `gramatica_atributos.py:316` (returns 'ERROR'), `analisador_tipos.py:324` (detects)

---

## Theory-to-Code Mappings

### Attribute Grammars

| Theoretical Concept | Your Implementation |
|---------------------|---------------------|
| Γ (type environment) | `TabelaSimbolos` class |
| Γ ⊢ e : T (type judgment) | `'regra_formal'` strings in rules |
| Synthesized attribute | `'tipo'` computed bottom-up |
| Inherited attribute | `tabela` passed top-down |
| Semantic action | `'acao_semantica'` lambda |

**Code**: `gramatica_atributos.py` (all rules), `analisador_tipos.py:245-298` (application)

---

### Type Inference vs. Checking

| Operation | Direction | Your Code |
|-----------|-----------|-----------|
| **Inference** | Bottom-up (leaves → root) | `_avaliar_operando()` synthesizes types |
| **Checking** | Top-down (expected → actual) | `_aplicar_regra_semantica()` validates |

**Not Hindley-Milner**: No type variables, no unification, monomorphic only

**Code**: `analisador_tipos.py:107-204` (inference), `analisador_tipos.py:245-298` (checking)

---

## Quick Answers to Expected Questions

### "Walk me through type checking for `(5 3.5 +)`"

1. **Inference**: `5 → int`, `3.5 → real` (bottom-up from literals)
2. **Checking**: Both in `TIPOS_NUMERICOS`? Yes (top-down validation)
3. **Inference**: `promover_tipo(int, real) → real` (result type)

**Code path**: `analisador_tipos.py:124` → `tipos.py:55` → `gramatica_atributos.py:74`

---

### "How does RES reference work?"

1. User writes: `(5 3 +)` on line 1 → result 8 (type int)
2. User writes: `(1 RES)` on line 2 → references line 1
3. Analyzer: `historico_tipos[1] = {'tipo': 'int', 'valor': 8}`
4. RES: Load `historico_tipos[line_atual - N]`

**Difference from MEM**:
- RES: References **computation results** (any type, including boolean)
- MEM: Stores **persistent values** (int/real only)

**Code**: `gramatica_atributos.py:465-503` (RES rule), `analisador_memoria_controle.py:63-103` (validation)

---

### "What happens if IFELSE branches have different types?"

1. Rule: `true_branch['tipo'] == false_branch['tipo']` → False
2. Rule: Returns `{'tipo': 'ERROR', 'validacao': False}`
3. Analyzer: Detects `tipo == 'ERROR'`
4. Analyzer: Raises `ErroSemantico` with details
5. Exception caught, error collected, analysis continues

**Why not raise in rule?** Separation of concerns + fault tolerance

**Code**: `gramatica_atributos.py:316` → `analisador_tipos.py:324`

---

### "Why three separate analyzers instead of one?"

**Answer**: Separation of concerns (sequential architecture)
- `analisador_tipos.py`: Type checking only
- `analisador_memoria_controle.py`: Memory/control validation
- `gerador_arvore_atribuida.py`: Report generation

**Benefits**:
- Modular (can run type check independently)
- Testable (each analyzer has focused responsibility)
- Extensible (add new analyzer for optimization)

**Code**: `analisador_semantico.py:470-508` (orchestrates 3 passes)

---

## Common Pitfalls to Avoid in Defense

### ❌ DON'T SAY:

1. "I don't know why we did it that way"
   - ✅ SAY: "That's a design trade-off between X and Y"

2. "It's just how the professor wanted it"
   - ✅ SAY: "We chose this approach because..."

3. "The code is buggy/incomplete"
   - ✅ SAY: "The implementation is complete with 73 passing tests"

4. "I can't remember the difference between X and Y"
   - ✅ SAY: "Let me trace through the code to show you exactly"

---

## If You Get Stuck

### Strategy 1: Trace Through Code

Professor: *"How does your system handle X?"*

You: *"Let me trace through the actual code. In `file.py:line`, we first... then in `other_file.py:line2`, we..."*

### Strategy 2: Reference Tests

You: *"We have a test for that specific case in `test_tabela_simbolos.py:215` which validates..."*

### Strategy 3: Show the Theory Connection

You: *"That corresponds to the formal rule `Γ ⊢ e₁ + e₂ : T` where T is computed by..."*

---

## File Quick Reference

| Concept | File | Key Lines |
|---------|------|-----------|
| Type promotion | `tipos.py` | 55-96 |
| Division restriction | `tipos.py` | 177-201 |
| Power asymmetry | `tipos.py` | 203-228 |
| Storage prohibition | `tipos.py` | 329-352 |
| Symbol table | `tabela_simbolos.py` | 126 (flat dict) |
| Initialization | `tabela_simbolos.py` | 281-306 |
| Type mutation | `tabela_simbolos.py` | 186-194 |
| Semantic rules | `gramatica_atributos.py` | All |
| IFELSE rule | `gramatica_atributos.py` | 306-332 |
| RES rule | `gramatica_atributos.py` | 465-503 |
| Type inference | `analisador_tipos.py` | 107-204 |
| Type checking | `analisador_tipos.py` | 245-298 |
| ERROR detection | `analisador_tipos.py` | 324-332 |

---

## Key Numbers to Remember

- **3 types**: int, real, boolean
- **14 compatibility functions** in `tipos.py`
- **23 semantic rules** in `gramatica_atributos.py`
- **7 arithmetic operators**: +, -, *, |, /, %, ^
- **6 comparison operators**: >, <, >=, <=, ==, !=
- **3 logical operators**: &&, ||, !
- **3 control structures**: FOR, WHILE, IFELSE
- **4 command operations**: MEM_STORE, MEM_LOAD, RES, EPSILON
- **73 passing tests** across all modules
- **~4,200 lines** of implementation

---

## Final Advice

1. **Be confident**: You have a complete, working implementation
2. **Show code**: Don't just explain—trace through actual files
3. **Connect theory**: Every design decision has theoretical justification
4. **Admit trade-offs**: No design is perfect; show you understand pros/cons
5. **Use examples**: Walk through concrete inputs like `(5 3.5 +)`

**Good luck with your defense tomorrow!** 🍀

---

For complete, detailed answers with full code traces and theory explanations, see **DEFENSE_QUESTIONS.md**.
