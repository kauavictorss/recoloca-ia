# Skill — Course Analysis

## Objetivo

Buscar cursos na Alura para preencher lacunas de habilidades identificadas a partir de `data/job-search-results.md`, priorizando resultados úteis e retornando até 5 recomendações ordenadas por nível.

## Entrada Obrigatória

1. `data/job-search-results.md`
   - Usado para extrair a lista de habilidades faltantes.
2. `data/user-profile.md`
   - Usado para ler a área de interesse do usuário.
3. `personas/curator.md`
   - Define o comportamento do agente Curator.
4. `skills/firecrawl.md`
   - Define comandos e regras do Firecrawl CLI.

## Ferramentas do Zed

1. `find_path`
   - Verificar se `data/job-search-results.md` existe antes de iniciar a busca.
2. `terminal`
   - Executar comandos `firecrawl search` e `firecrawl scrape`.

## Validação de Pré-Requisitos

1. Verifique se `data/job-search-results.md` existe.
2. Se não existir, reporte erro e peça ao usuário para buscar vagas primeiro.
3. Leia `data/job-search-results.md` e extraia a lista de habilidades faltantes.
4. Se o arquivo existir mas não tiver resultados válidos ou não houver habilidades faltantes extraíveis, reporte erro e pare.
5. Leia `data/user-profile.md` para obter a área de interesse.
6. Se a área de interesse estiver ausente, reporte erro e pare.

## Fluxo de Busca de Cursos

Para cada habilidade faltante extraída do arquivo de vagas:

1. Execute a busca principal na Alura.

```bash
firecrawl search "alura [habilidade]" --json
```

2. Se a busca principal falhar para aquela habilidade, tente o fallback.

```bash
firecrawl search "site:alura.com.br [habilidade]" --json
```

3. Se o fallback também falhar, pule essa habilidade e continue com as demais.
4. Não pare o processamento inteiro por causa de uma única habilidade com falha.
5. Priorize resultados cujo link pertença a `alura.com.br/formacoes`.
6. Não navegue diretamente para uma URL antes de usar a busca para descobrir cursos relevantes.
7. Se o resultado da busca não trouxer duração ou nível de forma clara, use `firecrawl scrape` na página descoberta para confirmar os dados.

```bash
firecrawl scrape "[url-do-curso]" --format markdown
```

## Extração de Dados

Para cada curso selecionado, extraia:

1. nome_curso: título do curso ou formação.
2. duracao: duração informada na página ou no resultado da busca.
3. nivel: iniciante, intermediario ou avancado.
4. link: URL do curso.
5. aborda_habilidade: habilidade faltante associada ao curso.

## Classificação de Nível

Classifique o curso com base no título e no contexto da página:

1. `iniciante`
   - Título contém: "Introdução", "Primeiros Passos", "Fundamentos", "Básico" ou "Para Iniciantes".
2. `intermediario`
   - Título contém: "Intermediário" ou indica conhecimento prévio.
3. `avancado`
   - Título contém: "Avançado", "Profundo", "Expert", "Arquitetura" ou "Especialista".

Se o título não deixar o nível explícito, use a indicação mais conservadora que ainda faça sentido com o texto da página.

## Ordenação dos Resultados

1. Ordene primeiro os cursos `iniciante`.
2. Depois os cursos `intermediario`.
3. Depois os cursos `avancado`.
4. Dentro do mesmo nível, preserve a melhor correspondência entre curso e habilidade faltante.
5. Retorne no máximo 5 cursos no total.

## Formato de Resposta Esperado

Retorne exatamente no formato de Response Envelope do `skills/dispatch.md`.

```text
## RESPOSTA: CURATOR
### estado
[sucesso | erro]

### resumo
[Resumo legível de 2-3 frases para o usuário]

### dados
1. nome_curso: [título do curso]
   duracao: [ex: 20 horas]
   nivel: [iniciante | intermediario | avancado]
   aborda_habilidade: [nome da habilidade]
   link: [URL]

2. [próximo curso no mesmo formato]

Ordem sugerida:
1. [nome do curso]
2. [nome do curso]
3. [nome do curso]

### erros
[Apenas se estado for erro: o que deu errado]
```

## Tratamento de Erros

1. Se `firecrawl search` falhar para uma habilidade específica, tente o fallback.
2. Se o fallback também falhar, registre a falha e siga para a próxima habilidade.
3. Se uma `firecrawl scrape` falhar em uma URL específica, use o melhor dado disponível do resultado da busca e anote que a extração foi incompleta.
4. Se nenhuma habilidade puder ser atendida, retorne `estado: erro` com a lista de falhas.
5. Nunca invente nome, duração, nível ou link.
6. Nunca silencie falhas de busca ou extração.
7. Se `data/job-search-results.md` estiver ausente ou vazio, pare e informe que o usuário precisa buscar vagas primeiro.

## Regras Finais

1. Não escreva arquivos de estado; apenas retorne o Response Envelope ao Maestro.
2. Mantenha a resposta curta, estruturada e objetiva.
3. Use somente listas numeradas com pares chave-valor; não use tabelas markdown.
4. Preserve a ordem iniciando pelos cursos mais acessíveis ao usuário.

