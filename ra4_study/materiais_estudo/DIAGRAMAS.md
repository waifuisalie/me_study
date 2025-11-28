# Diagramas Visuais do Compilador

Todos os diagramas em formato **Mermaid** (renderiza em GitHub/VS Code).

---

## 1. Pipeline Completo

```mermaid
graph TB
    A[Código Fonte .txt] --> B[RA1: Análise Léxica]
    B -->|tokens| C[RA2: Análise Sintática]
    C -->|AST| D[RA3: Análise Semântica]
    D -->|AST Atribuída| E[RA4a: Geração TAC]
    E -->|TAC Original| F[RA4b: Otimização]
    F -->|TAC Otimizado| G[RA4c: Assembly]
    G -->|.s| H[RA4d: Compilação AVR]
    H -->|.hex| I[Arduino Uno]

    style B fill:#e1f5ff
    style C fill:#e1f5ff
    style D fill:#e1f5ff
    style E fill:#fff4e1
    style F fill:#fff4e1
    style G fill:#fff4e1
    style H fill:#fff4e1
```

---

## 2. Algoritmo LL(1) Parsing

```mermaid
flowchart TD
    START([Início]) --> INIT[Inicializa Pilha = $, PROGRAM<br/>Entrada = tokens + $]
    INIT --> LOOP{Pilha vazia?}
    LOOP -->|Sim| SUCCESS([Aceito ✓])
    LOOP -->|Não| POP[Remove topo da pilha]
    POP --> CHECK{Topo é<br/>terminal?}

    CHECK -->|Sim| MATCH{topo == entrada?}
    MATCH -->|Sim| ADVANCE[Avança entrada]
    ADVANCE --> LOOP
    MATCH -->|Não| ERROR1([Erro Sintático])

    CHECK -->|Não| TABLE[Consulta M[topo, entrada]]
    TABLE --> FOUND{Produção<br/>encontrada?}
    FOUND -->|Não| ERROR2([Erro Sintático])
    FOUND -->|Sim| EXPAND[Empilha símbolos<br/>da produção]
    EXPAND --> LOOP

    style SUCCESS fill:#90EE90
    style ERROR1 fill:#FFB6C1
    style ERROR2 fill:#FFB6C1
```

---

## 3. Verificação de Tipos (Bottom-Up)

```mermaid
graph BT
    A["5 [tipo: ?]"] --> B["+ [tipo: ?]"]
    C["3 [tipo: ?]"] --> B

    B --> D["LINHA [tipo: ?]"]

    A2["5 [tipo: int]"] -.Passo 1.-> B2["+ [tipo: ?]"]
    C2["3 [tipo: int]"] -.Passo 1.-> B2

    B2 -.Passo 2.-> D2["+ [tipo: int]"]

    D2 -.Passo 3.-> E["LINHA [tipo: int]"]

    style A2 fill:#90EE90
    style C2 fill:#90EE90
    style D2 fill:#90EE90
    style E fill:#90EE90
```

---

## 4. Otimização Multi-Pass

```mermaid
flowchart TD
    START([TAC Original]) --> INIT[iteracao = 0<br/>mudou = true]
    INIT --> LOOP{mudou &&<br/>iteracao < 100?}

    LOOP -->|Não| END([TAC Otimizado])

    LOOP -->|Sim| INC[iteracao++<br/>mudou = false]
    INC --> FOLD[Constant Folding]
    FOLD --> CHECK1{Houve mudança?}
    CHECK1 -->|Sim| SET1[mudou = true]
    CHECK1 -->|Não| PROP
    SET1 --> PROP

    PROP[Constant Propagation] --> CHECK2{Houve mudança?}
    CHECK2 -->|Sim| SET2[mudou = true]
    CHECK2 -->|Não| DEAD
    SET2 --> DEAD

    DEAD[Dead Code Elimination] --> CHECK3{Houve mudança?}
    CHECK3 -->|Sim| SET3[mudou = true]
    CHECK3 -->|Não| JUMP
    SET3 --> JUMP

    JUMP[Jump Elimination] --> CHECK4{Houve mudança?}
    CHECK4 -->|Sim| SET4[mudou = true]
    CHECK4 -->|Não| LOOP
    SET4 --> LOOP

    style START fill:#FFE4B5
    style END fill:#90EE90
```

---

## 5. Evolução das Estruturas de Dados

```mermaid
graph LR
    A["TEXTO<br/>'( 5 3 + )'"] -->|RA1| B["TOKENS<br/>[abre_paren, 5, 3, +, fecha_paren]"]
    B -->|RA2| C["AST<br/>LINHA<br/>├─ (<br/>├─ +<br/>│  ├─ 5<br/>│  └─ 3<br/>└─ )"]
    C -->|RA3| D["AST ATRIBUÍDA<br/>LINHA [int]<br/>├─ (<br/>├─ + [int]<br/>│  ├─ 5 [int]<br/>│  └─ 3 [int]<br/>└─ )"]
    D -->|RA4a| E["TAC<br/>t0 = 5 + 3"]
    E -->|RA4b| F["TAC OTIM<br/>t0 = 8"]
    F -->|RA4c| G["ASSEMBLY<br/>ldi r16, 8<br/>sts t0, r16"]

    style A fill:#FFE4B5
    style G fill:#90EE90
```

---

## 6. Tabela de Símbolos - Ciclo de Vida

```mermaid
sequenceDiagram
    participant Código
    participant RA3
    participant Tabela

    Código->>RA3: (5 X)
    RA3->>Tabela: adicionar_simbolo('X', 'int')
    Tabela-->>RA3: OK
    RA3->>Tabela: marcar_inicializada('X')

    Código->>RA3: (X 3 +)
    RA3->>Tabela: eh_inicializado('X')?
    Tabela-->>RA3: Sim ✓
    RA3->>Tabela: obter_tipo('X')
    Tabela-->>RA3: 'int'
    RA3->>RA3: Processa operação

    Note over Código,Tabela: Caso de Erro
    Código->>RA3: (Y 2 +)
    RA3->>Tabela: eh_inicializado('Y')?
    Tabela-->>RA3: Não ❌
    RA3-->>Código: ErroSemantico
```

---

## 7. Alocação de Registradores AVR

```mermaid
graph TD
    subgraph "Registradores AVR (ATmega328P)"
        R0["R0-R1<br/>Multiplicação"]
        R2["R2-R15<br/>Salvos"]
        R16["R16-R23<br/>TEMPORÁRIOS"]
        R24["R24-R25<br/>Params/Retorno"]
        R26["R26-R27 (X)<br/>Ponteiro"]
        R28["R28-R29 (Y)<br/>Frame Pointer"]
        R30["R30-R31 (Z)<br/>Stack Pointer"]
    end

    subgraph "Mapeamento TAC → AVR"
        T1["t0, t1, t2"] -->|Alocados| R16
        V1["COUNTER, RESULT"] -->|Fixos| R2
        P1["Spill"] -->|Memória| MEM["SRAM"]
    end

    style R16 fill:#90EE90
    style R24 fill:#FFE4B5
```

---

## 8. Fluxo de Controle: WHILE

```mermaid
graph TD
    START([Início]) --> L0[L0: Label Início Loop]
    L0 --> COND{Avalia<br/>Condição}
    COND -->|Falso| L1[L1: Label Fim Loop]
    COND -->|Verdadeiro| CORPO[Executa Corpo]
    CORPO --> GOTO[goto L0]
    GOTO --> L0
    L1 --> END([Fim])

    style L0 fill:#FFE4B5
    style L1 fill:#90EE90
```

**TAC correspondente**:
```
L0:
    t0 = COUNTER <= LIMIT
    ifFalse t0 goto L1
    ... (corpo)
    goto L0
L1:
```

---

## 9. Hierarquia de Tipos

```mermaid
graph TD
    A[Tipos Primitivos] --> B[int]
    A --> C[real]
    A --> D[boolean]

    B -.promoção.-> C

    style B fill:#E3F2FD
    style C fill:#E8F5E9
    style D fill:#FFF9C4
```

**Regras**:
- `int` pode ser promovido para `real`
- `boolean` não tem conversão automática
- Operadores relacionais sempre retornam `boolean`

---

## 10. Constant Folding - Fluxo

```mermaid
flowchart LR
    A["t0 = 2 + 3"] --> CHECK{Ambos<br/>constantes?}
    CHECK -->|Não| B["Mantém instrução"]
    CHECK -->|Sim| EVAL["Avalia: 2 + 3 = 5"]
    EVAL --> SUBST["t0 = 5"]

    style A fill:#FFE4B5
    style SUBST fill:#90EE90
```

---

## Como Usar os Diagramas

### No GitHub/VS Code
Os diagramas Mermaid renderizam automaticamente.

### Em Apresentações
1. Copie código Mermaid
2. Use https://mermaid.live para gerar PNG
3. Insira em slides

### Para Estudo
- Trace fluxos manualmente
- Adicione exemplos concretos
- Compare com código real

---

**Dica**: Desenhe esses diagramas à mão durante a defesa para mostrar compreensão!
