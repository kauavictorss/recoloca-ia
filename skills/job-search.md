# Skill — Job Search

## Objetivo

Buscar vagas de emprego via Firecrawl, analisar aderência às habilidades do usuário e retornar resultados estruturados.

## Fluxo Geral

1. **Descoberta**: usar `firecrawl search` para encontrar vagas com base em área e localização
2. **Extração**: usar `firecrawl scrape` em cada URL para obter descrição e requisitos completos
3. **Análise**: comparar habilidades do perfil com requisitos da vaga
4. **Formatação**: retornar em Response Envelope com até 5 vagas
5. **Erro**: reportar falhas sem inventar dados

## Ferramentas Zed Disponíveis

1. `terminal`
   - Executar comandos CLI do `firecrawl`
   - Interpretar saída JSON de busca
   - Tratamento de timeouts e conexões recusadas

## Comandos Firecrawl

### Descoberta de Vagas

```bash
firecrawl search "vagas [area_de_interesse] [localizacao]" --json
```

**Parametrização**:
- `[area_de_interesse]`: valor de `Área de interesse` do perfil (Frontend, Backend, Ciência de Dados, etc)
- `[localizacao]`: valor de `Localização` do perfil (cidade/estado ou "Remoto")

**Saída esperada**: JSON com campos para cada resultado:
- `url`: link direto para a vaga
- `title`: título da vaga
- `description`: resumo ou descrição breve

### Extração Completa da Vaga

```bash
firecrawl scrape <url> --format markdown
```

**Saída**: Markdown contendo título, empresa, localização, descrição, requisitos.

## Análise de Aderência

### 1. Extração de Habilidades Requeridas

A partir do resultado do `firecrawl scrape`, procure por seções de "Requisitos", "Qualificações", "Habilidades", "Skills requeridos" ou similares.

Extraia termos técnicos: linguagens de programação, frameworks, ferramentas, metodologias, etc.

**Exemplo**:
- Vaga menciona: "Python, SQL, Git, AWS, Docker, Scrum"
- Habilidades extraídas: `python`, `sql`, `git`, `aws`, `docker`, `scrum`

### 2. Comparação com Perfil do Usuário

Do campo `Habilidades atuais` do perfil, normalize para minúsculas e divida por vírgula:
- Entrada: "Python, SQL, Excel, Figma, Git"
- Normalizado: `["python", "sql", "excel", "figma", "git"]`

Para cada habilidade requerida pela vaga:
- Se existe na lista normalizada do perfil → **correspondência**.
- Caso contrário → **falta**.

**Regra de correspondência**: comparação case-insensitive. `Python` == `python`.

### 3. Contagem e Classificação

Contar o número de correspondências e faltas:
- `contagem_correspondencia`: "X de Y habilidades correspondem"
- Exemplo: "3 de 6 habilidades correspondem" (3 correspondências, 3 faltantes)

### 4. Filtragem por Nível

Se a vaga menciona um nível (Júnior, Junior, Pleno, Sênior, Senior):
- Preferir vagas que correspondam ao `Nível de experiência` do perfil.
- Se não houver vagas do nível correspondente, expandir para nível adjacente (ex: Pleno procura vagas de Pleno e Sênior juntas).
- Anotar no resultado se a vaga é de nível adjacente.

Se nenhuma vaga mencionou nível, proceder com o padrão do perfil.

## Tratamento de Erros

### Erro: firecrawl search falha

- Reportar o erro exato ao usuário: "Falha ao buscar vagas: [mensagem de erro]"
- Retornar Response Envelope com `estado: erro` e `erros: [mensagem]`
- **NÃO** inventar dados.

### Erro: firecrawl scrape falha em URL específica

- Usar o título e descrição do resultado da busca como fallback.
- Anotar no resultado: "Extração incompleta (detalhos não disponíveis)"
- Continuar com as outras vagas.

### Erro: Sem resultados na busca

- Reportar: "Nenhuma vaga encontrada para [área] em [localização]. Sugira ao usuário ampliar os termos de busca."
- Retornar Response Envelope com `estado: erro` e dados vazios.

## Formato de Retorno

**Response Envelope** conforme `skills/dispatch.md`:

```
## RESPOSTA: SCOUT
### estado
sucesso

### resumo
Encontrou X vagas para [área] em [localização]. Aqui estão as melhores correspondências com suas habilidades atuais.

### dados
1. titulo: [título da vaga]
   empresa: [nome da empresa]
   localizacao: [cidade ou Remoto]
   link: [URL]
   habilidades_correspondentes: [habilidade1, habilidade2]
   habilidades_faltantes: [habilidade3, habilidade4]
   contagem_correspondencia: [X de Y habilidades correspondem]

2. titulo: [próximo título]
   ...

(até 5 vagas)

### erros
[nenhum se sucesso]
```

## Regras Finais

1. **Até 5 vagas**: limitar resultado a no máximo 5 vagas.
2. **Nunca inventar dados**: se não conseguir extrair algo, não complete manualmente. Use fallback ou deixe vago.
3. **Sem tabelas markdown**: usar listas numeradas com pares chave-valor como mostrado acima.
4. **Preservar formato**: seguir exatamente o Response Envelope do `skills/dispatch.md`.
5. **Timeouts**: se o terminal demorar muito, reportar timeout e sugerir ao usuário tentar novamente.

## Exemplo Passo a Passo

**Entradas do Contexto:**
- Área: Backend
- Localização: São Paulo
- Habilidades atuais: Python, SQL, Docker, Git, AWS

**Passo 1: Busca**
```bash
firecrawl search "vagas Backend São Paulo" --json
```

**Passo 2: Para cada resultado, scrape**
```bash
firecrawl scrape https://job-url.com --format markdown
```

**Passo 3: Análise**
- Vaga requer: Python, SQL, Docker, Kubernetes, AWS, Jenkins
- Correspondências: Python, SQL, Docker, AWS (4 de 6)
- Faltantes: Kubernetes, Jenkins

**Passo 4: Retorno**
```
1. titulo: Senior Backend Engineer
   empresa: TechCorp
   localizacao: São Paulo
   link: https://...
   habilidades_correspondentes: Python, SQL, Docker, AWS
   habilidades_faltantes: Kubernetes, Jenkins
   contagem_correspondencia: 4 de 6 habilidades correspondem
```

## Regras de Estado

- Nunca modifique `data/user-profile.md` ou `data/personality-quiz.md`.
- Apenas leia dados do perfil para análise.
- Não crie novos arquivos em `data/`.
- O Maestro salva `data/job-search-results.md` após receber a resposta do Scout.

