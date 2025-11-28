# Materiais de Estudo - Compilador RA4

## 📚 Visão Geral

Este diretório contém **12 documentos educacionais** criados para preparar a equipe RA4_1 para a defesa do projeto de compiladores.

**Total**: ~4.750 linhas de documentação em português brasileiro
**Objetivo**: Compreensão profunda de teoria + implementação para responder questões cross-cutting da defesa

---

## 📖 Guia de Leitura Recomendado

### 🎯 Fase 1: Fundação (Leia PRIMEIRO)

1. **ARQUITETURA_COMPILADOR.md** (598 linhas)
   - Visão geral do compilador completo
   - Pipeline de compilação
   - Mapeamento teoria → código
   - Fluxo de dados entre fases

2. **RA2_LL1_GUIA_COMPLETO.md** (763 linhas)
   - Teoria completa de LL(1)
   - Algoritmos FIRST/FOLLOW
   - Construção da tabela de parsing
   - Exemplo passo-a-passo

3. **RA4_OTIMIZACAO_TAC.md** (447 linhas)
   - 4 técnicas de otimização
   - Algoritmo multi-pass
   - Convergência e terminação
   - Exemplos com código real

### 📝 Fase 2: Walkthroughs (Leia em ORDEM)

4. **RA1_WALKTHROUGH.md** (234 linhas)
   - Análise léxica
   - Máquina de estados
   - Tipos de token
   - Exemplo passo-a-passo

5. **RA2_WALKTHROUGH.md** (235 linhas)
   - Análise sintática
   - Parser LL(1) em ação
   - Geração de AST
   - Interface RA2→RA3

6. **RA3_WALKTHROUGH.md** (372 linhas)
   - Análise semântica
   - Sistema de tipos
   - Tabela de símbolos
   - 3 fases de análise

7. **RA4_WALKTHROUGH.md** (419 linhas)
   - Geração de TAC
   - Otimização
   - Assembly AVR
   - Pipeline completo

### 🎓 Fase 3: Preparação para Defesa (CRÍTICO)

8. **PERGUNTAS_DEFESA.md** (450 linhas) ⭐⭐⭐
   - Perguntas cross-cutting esperadas
   - Perguntas sobre algoritmos
   - Perguntas de implementação
   - Respostas detalhadas com código

9. **DEMO_SCENARIOS.md** (315 linhas) ⭐⭐⭐
   - 4 cenários de demonstração
   - Trace completo de fatorial.txt
   - Demonstração de otimização
   - Demonstração de erros

10. **DIAGRAMAS.md** (274 linhas)
    - 10 diagramas Mermaid
    - Fluxogramas de algoritmos
    - Visualizações de estruturas
    - Evolução de dados

### 📑 Fase 4: Referências Rápidas (Durante a Defesa)

11. **THEORY_TO_CODE.md** (199 linhas)
    - Tabela conceito → arquivo
    - Funções principais
    - Números de linha
    - Comandos úteis

12. **TAC_REFERENCE.md** (442 linhas)
    - 12 tipos de instruções TAC
    - Sintaxe e semântica
    - Exemplos de uso
    - Mapeamento TAC → Assembly

---

## 🎯 Estratégia de Estudo

### Semana 1: Fundação
- [ ] Ler ARQUITETURA_COMPILADOR.md
- [ ] Ler RA2_LL1_GUIA_COMPLETO.md
- [ ] Ler RA4_OTIMIZACAO_TAC.md
- [ ] Fazer anotações próprias

### Semana 2: Detalhes
- [ ] Ler todos os 4 walkthroughs (RA1-RA4)
- [ ] Executar compilador com os 3 testes
- [ ] Examinar saídas de cada fase
- [ ] Comparar com descrições nos docs

### Semana 3: Preparação Final
- [ ] Estudar PERGUNTAS_DEFESA.md
- [ ] Praticar DEMO_SCENARIOS.md
- [ ] Revisar DIAGRAMAS.md
- [ ] Simular defesa com colegas

### Dia da Defesa
- [ ] Ter THEORY_TO_CODE.md impresso
- [ ] Ter TAC_REFERENCE.md aberto
- [ ] Ter outputs dos 3 testes prontos
- [ ] Ter código-fonte aberto em VS Code

---

## 📊 Estatísticas dos Documentos

| Documento | Linhas | Tamanho | Prioridade | Tempo Leitura |
|-----------|--------|---------|------------|---------------|
| ARQUITETURA_COMPILADOR.md | 598 | 18KB | Alta | 30 min |
| RA2_LL1_GUIA_COMPLETO.md | 763 | 21KB | Alta | 40 min |
| RA4_OTIMIZACAO_TAC.md | 447 | 11KB | Alta | 25 min |
| RA1_WALKTHROUGH.md | 234 | 5.4KB | Média | 15 min |
| RA2_WALKTHROUGH.md | 235 | 5.3KB | Média | 15 min |
| RA3_WALKTHROUGH.md | 372 | 7.5KB | Média | 20 min |
| RA4_WALKTHROUGH.md | 419 | 7.4KB | Média | 20 min |
| PERGUNTAS_DEFESA.md | 450 | 11KB | **CRÍTICA** | 30 min |
| DEMO_SCENARIOS.md | 315 | 6.3KB | **CRÍTICA** | 20 min |
| DIAGRAMAS.md | 274 | 6.2KB | Alta | 15 min |
| THEORY_TO_CODE.md | 199 | 5.6KB | Média | 10 min |
| TAC_REFERENCE.md | 442 | 6.4KB | Média | 20 min |
| **TOTAL** | **4.748** | **111KB** | - | **~4h 20min** |

---

## 🔑 Conceitos-Chave Cobertos

### Teoria de Compiladores
- ✅ Análise léxica (autômatos finitos)
- ✅ Análise sintática (gramáticas CFG, LL(1))
- ✅ Análise semântica (sistemas de tipos)
- ✅ Código intermediário (TAC)
- ✅ Otimização (4 técnicas clássicas)
- ✅ Geração de código (Assembly AVR)

### Algoritmos Implementados
- ✅ Tokenização por máquina de estados
- ✅ Cálculo de FIRST/FOLLOW
- ✅ Parser LL(1) stack-based
- ✅ Inferência de tipos bottom-up
- ✅ Travessia pós-ordem de AST
- ✅ Constant folding/propagation
- ✅ Dead code elimination
- ✅ Jump elimination
- ✅ Multi-pass optimization

### Estruturas de Dados
- ✅ Tokens
- ✅ AST (sintática e atribuída)
- ✅ Tabela de símbolos
- ✅ Instruções TAC
- ✅ Tabela LL(1)

---

## 💡 Dicas para Usar os Materiais

### Durante o Estudo
1. **Leia com código aberto** - Compare descrições com implementação
2. **Execute exemplos** - Rode o compilador enquanto lê
3. **Faça anotações** - Escreva suas próprias notas
4. **Desenhe diagramas** - Reproduza os diagramas à mão
5. **Pratique explicações** - Explique em voz alta

### Durante a Defesa
1. **Tenha referências impressas** - THEORY_TO_CODE.md e TAC_REFERENCE.md
2. **Navegue rapidamente** - Saiba onde está cada conceito
3. **Use diagramas** - Desenhe no quadro quando necessário
4. **Demonstre código** - Mostre implementação real
5. **Seja preciso** - Use números de linha exatos

---

## 🎓 Perguntas Frequentes

### P: "Quanto tempo leva para estudar tudo?"
**R**: ~4-5 horas de leitura + 2-3 horas de prática = **7-8 horas total**

### P: "Qual documento é MAIS importante?"
**R**: **PERGUNTAS_DEFESA.md** - Contém respostas para perguntas típicas do professor

### P: "Preciso memorizar código?"
**R**: **NÃO**. Entenda a lógica. Use THEORY_TO_CODE.md para localizar código rapidamente.

### P: "E se o professor perguntar algo não coberto?"
**R**: Use os **princípios** aprendidos para deduzir. Mostre raciocínio, não apenas respostas memorizadas.

---

## 📞 Suporte

**Dúvidas?** Discuta com a equipe:
- Breno Rossi Duarte
- Francisco Bley Ruthes
- Rafael Olivare Piveta
- Stefan Benjamim Seixas Lourenço Rodrigues

---

## ✅ Checklist Final

Antes da defesa, certifique-se:

- [ ] Li todos os 12 documentos
- [ ] Executei os 3 testes (fatorial, fibonacci, taylor)
- [ ] Examinei saídas de todas as fases
- [ ] Pratiquei DEMO_SCENARIOS.md
- [ ] Revisei PERGUNTAS_DEFESA.md
- [ ] Tenho referências impressas/abertas
- [ ] Sei localizar conceitos rapidamente
- [ ] Pratiquei explicações verbais
- [ ] Simulei defesa com colegas

---

**Boa sorte na defesa! 🚀**

**Última atualização**: 2025-01-27
