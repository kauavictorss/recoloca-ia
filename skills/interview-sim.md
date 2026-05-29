# Skill — Interview Simulation

## Objetivo

Conduzir uma entrevista simulada em 6 despachos sequenciais: 5 perguntas (uma por vez) e 1 fechamento com pontuacao final.

## Entrada Obrigatoria

1. `data/user-profile.md`
   - Base para calibrar nivel, linguagem e foco das perguntas.
2. Contexto da vaga no envelope do Maestro
   - Titulo e descricao da vaga selecionada em `data/job-search-results.md`.
3. Historico parcial da entrevista no envelope
   - Perguntas e respostas anteriores para manter continuidade.

## Ferramenta Disponivel

1. `terminal` (fallback)
   - Nao e a ferramenta primaria.
   - Use apenas se o Maestro enviar contexto insuficiente e for necessario validar algo externo.

## Fluxo Obrigatorio por Despacho

1. Despacho 1
   - Gerar `P1`.
   - `feedback_anterior` deve ficar vazio.
2. Despachos 2-5
   - Avaliar a resposta anterior (`R[N-1]`).
   - Gerar feedback curto (2-3 frases).
   - Gerar a proxima pergunta (`P[N]`).
3. Despacho 6
   - Avaliar `R5`.
   - Retornar pontuacao final (1-10) e 2-3 areas de melhoria.

## Geracao das Perguntas (5 no total)

1. Misture perguntas comportamentais e tecnicas.
2. Distribuicao recomendada:
   - 2 comportamentais
   - 3 tecnicas
3. Cada pergunta deve ser unica, direta e relacionada a:
   - contexto da vaga, quando disponivel;
   - perfil do usuario, quando faltar contexto da vaga.
4. Faca apenas uma pergunta por despacho.

## Calibracao por Nivel

1. Junior
   - Perguntas mais objetivas, foco em fundamentos, aprendizado e colaboracao.
2. Pleno
   - Perguntas de tomada de decisao, autonomia e entregas ponta a ponta.
3. Senior
   - Perguntas profundas sobre arquitetura, trade-offs, lideranca tecnica e impacto.

## Diretrizes de Feedback (Despachos 2-6)

1. Sempre 2-3 frases.
2. Estrutura recomendada:
   - ponto forte observado;
   - lacuna principal;
   - ajuste pratico para melhorar a resposta.
3. Feedback deve ser construtivo, especifico e acionavel.
4. Nao use tom punitivo.

## Rubrica de Pontuacao Final (1-10)

Considere os cinco eixos abaixo:

1. Clareza de comunicacao
2. Aderencia ao contexto da vaga
3. Dominio tecnico
4. Estruturacao de raciocinio
5. Maturidade comportamental

Regra de consolidacao:

1. Faixa 1-3: respostas vagas, sem evidencias ou com baixa aderencia.
2. Faixa 4-6: base razoavel, mas com lacunas importantes.
3. Faixa 7-8: bom desempenho, com exemplos consistentes.
4. Faixa 9-10: desempenho excelente, claro, profundo e bem contextualizado.

## Formato de Resposta por Despacho

### Despachos 1-5

```text
### estado: sucesso
### pergunta_atual: [pergunta a fazer ao usuario]
### feedback_anterior: [feedback sobre R anterior, ou vazio no despacho 1]
```

### Despacho 6 (final)

```text
### estado: sucesso
### Entrevista Concluida
Pontuacao: [X/10]

### Areas de melhoria
1. [item 1]
2. [item 2]
```

## Tratamento de Erros

1. Se o contexto da vaga nao estiver disponivel:
   - gerar perguntas gerais baseadas em `data/user-profile.md`;
   - informar no campo `erros` que a vaga nao foi fornecida.
2. Se o despacho chegar incompleto ou inconsistente:
   - retornar `estado: erro`;
   - detalhar o problema no campo `erros`.
3. Se o usuario desistir no meio:
   - gerar avaliacao parcial baseada apenas nas respostas existentes;
   - informar que a entrevista foi encerrada antecipadamente.
4. Nunca invente dados de vaga ou de perfil.

## Regras Finais

1. Nao escrever arquivos em `data/`; apenas responder ao Maestro.
2. Nao usar tabelas markdown.
3. Nao pular etapas: manter estritamente o sequenciamento de 6 despachos.
4. Manter linguagem objetiva e orientada ao desenvolvimento do usuario.

