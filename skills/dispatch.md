# Skills — Dispatch e Handoff

## Objetivo

Este documento define o protocolo de despacho e de resposta entre o Maestro e os agentes especializados.

## Roteamento

1. A = Scout
   - Função: busca de vagas.
   - Fontes-alvo: Indeed, Catho, LinkedIn, Glassdoor, Infojobs.

2. B = Curator
   - Função: busca de cursos e lacunas de habilidades.
   - Fonte-alvo: Alura.

3. C = Coach
   - Função: simulação de entrevistas.
   - Regra: o Coach deve ser despachado 6 vezes em sequência para compor a entrevista completa.

4. D = Maestro
   - Função: o Maestro conduz o quiz, salva o estado e gera `data/user-profile.md`.

## Envelope de Despacho

O Maestro deve construir exatamente este envelope ao usar `spawn_agent`:

## DESPACHO: [NOME_DO_AGENTE]
### referencia_persona
[Conteúdo completo de `personas/<nome_do_agente_minusculo>.md`]

### tarefa
[Uma frase descrevendo o que o agente deve fazer]

### perfil_usuario
[Conteúdo de `data/user-profile.md`]

### contexto
[Contexto específico: ex: qual vaga para entrevistar, quais habilidades buscar cursos]

### saida_esperada
[Exatamente em que formato o agente deve retornar]

## Envelope de Resposta

O agente despachado deve retornar exatamente este envelope:

## RESPOSTA: [NOME_DO_AGENTE]
### estado
[sucesso | erro]

### resumo
[Resumo legível de 2-3 frases para o usuário]

### dados
[Resultados como listas numeradas com pares chave-valor. Sem tabelas markdown.]

### erros
[Apenas se estado for erro: o que deu errado]

## Handoff por agente

1. Scout
   - Entrada: `data/user-profile.md` e o contexto da busca.
   - Saída: lista de vagas, requisitos e observações de aderência.
   - Foco: retornar resultados úteis ao usuário em linguagem clara.

2. Curator
   - Entrada: `data/user-profile.md` e o contexto das lacunas técnicas.
   - Saída: cursos, trilhas e justificativas para cada lacuna.
   - Foco: priorizar o que reduz a distância entre perfil e vaga.

3. Coach
   - Entrada: `data/user-profile.md` e o contexto da etapa da entrevista.
   - Saída: uma única rodada de entrevista por despacho.
   - Foco: usar linguagem natural, objetiva e progressiva.

4. Maestro
   - Entrada: estado do quiz e ações do usuário.
   - Saída: atualização do quiz, geração de perfil e menu de navegação.
   - Foco: orquestrar o fluxo sem perder o estado em `data/`.

## Despacho Sequencial do Coach

Quando o usuário escolher a opção C, o Maestro deve despachar o Coach 6 vezes, sempre em sequência e sem paralelizar.

1. Despacho 1
   - Objetivo: abertura da entrevista e contexto geral.

2. Despacho 2
   - Objetivo: aprofundar experiência, responsabilidades e impacto.

3. Despacho 3
   - Objetivo: avaliar competências técnicas e decisões práticas.

4. Despacho 4
   - Objetivo: explorar soft skills, colaboração e comunicação.

5. Despacho 5
   - Objetivo: simular perguntas difíceis e sinais de maturidade.

6. Despacho 6
   - Objetivo: encerramento, feedback final e próximos passos.

## Regras de Tratamento de Erros

1. Se o `spawn_agent` falhar, o Maestro deve interromper o fluxo da ação atual.
2. O Maestro deve relatar a falha no campo `erros` do envelope ou na resposta conversacional ao usuário.
3. O Maestro não deve inventar resultados para compensar uma falha.
4. Se o conteúdo de `data/user-profile.md` estiver ausente, incompleto ou inconsistente, o Maestro deve voltar ao quiz.
5. Se um agente responder com `estado: erro`, o Maestro deve repassar o erro ao usuário sem mascarar a falha.
6. O Maestro não deve continuar silenciosamente após qualquer falha de despacho, leitura ou gravação de arquivo.

