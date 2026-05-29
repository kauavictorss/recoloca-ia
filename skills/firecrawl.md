# Skill — Firecrawl CLI

## Objetivo

Referência de comandos e regras para usar Firecrawl via CLI (linha de comando terminal).

## Pré-requisitos

1. Firecrawl deve estar instalado globalmente: `npm install -g firecrawl` (ou via package manager local)
2. `FIRECRAWL_API_KEY` deve estar definida como variável de ambiente
3. Acesso a terminal/shell do Zed

## Verificação Inicial

Para verificar se Firecrawl está disponível:

```bash
firecrawl --version
```

Se não reconhecer o comando, baixe e instale: https://www.firecrawl.dev/docs/cli

## Autenticação

Firecrawl usa `FIRECRAWL_API_KEY` armazenada como variável de ambiente. Se não estiver definida, os comandos falharão com erro de autenticação.

## Comandos Principais

### 1. Search (Descoberta)

```bash
firecrawl search "<query>" --json
```

**Parâmetros**:
- `<query>`: texto de busca (obrigatório)
- `--json`: retorna resultado em JSON (recomendado para parsing)

**Exemplo**:
```bash
firecrawl search "vagas Backend São Paulo" --json
```

**Saída esperada**:
```json
[
  {
    "url": "https://example.com/job1",
    "title": "Senior Backend Engineer",
    "description": "Looking for a senior backend engineer..."
  },
  ...
]
```

**Tratamento de falha**:
- Sem resultados: retorna array vazio `[]`
- Query inválida: erro com mensagem descritiva
- API indisponível: timeout ou erro de conexão

### 2. Scrape (Extração Completa)

```bash
firecrawl scrape "<url>" --format markdown
```

**Parâmetros**:
- `<url>`: URL completa a extrair (obrigatório)
- `--format markdown`: retorna em Markdown (padrão é JSON)
- Sem `--format`: retorna JSON com metadados

**Exemplo**:
```bash
firecrawl scrape "https://example.com/job1" --format markdown
```

**Saída esperada**:
```markdown
# Senior Backend Engineer

**Company**: TechCorp

**Location**: São Paulo, Brazil

## About the Role

[descrição completa em markdown]

## Requirements

- Python
- SQL
- Docker
- ...
```

**Tratamento de falha**:
- URL inválida: erro 404 ou similar
- Site bloqueado anti-bot: erro de acesso
- Timeout: aguarde, depois tente novamente
- Página dinâmica (JS): mesma URL pode retornar conteúdo diferente

### 3. Crawl (Rastreamento em Lote)

```bash
firecrawl crawl "<url>" --max-pages <n>
```

**Parâmetros**:
- `<url>`: URL raiz a rastrear (obrigatório)
- `--max-pages <n>`: limite de páginas a rastrear (ex: `--max-pages 10`)
- `--format markdown`: retorna em Markdown

**Obs**: Para este projeto (Scout), não usamos `crawl` no momento. Use `search` + `scrape` sequencial.

## Regras de Uso

### Taxa e Limites

1. **Rate Limit**: Firecrawl tem limite de requisições por unidade de tempo. Se exceder, receberá erro `429 Too Many Requests`.
   - Solução: aguarde alguns segundos e tente novamente.

2. **Timeouts**: URLs podem demorar até 30 segundos para responder.
   - Solução: não cancele prematuro. Se persistir após 2 tentativas, reporte erro.

3. **Tamanho de página**: Páginas muito grandes podem ser truncadas.
   - Solução: Se a descrição parecer incompleta après scrape, use título+descrição do search como fallback.

### Tratamento de Erros Comuns

| Erro | Causa | Solução |
|------|-------|---------|
| `401 Unauthorized` | `FIRECRAWL_API_KEY` não definida ou inválida | Verificar variável de ambiente |
| `429 Too Many Requests` | Rate limit excedido | Aguardar e tentar novamente após 5s |
| `404 Not Found` | URL inválida ou página removida | Usar fallback, pular URL |
| `Timeout` | Conexão levou > 30s | Tentar novamente, se persistir reporte |
| `Empty response` | URL acessada, mas sem conteúdo útil | Usar fallback do search |

## Integração com Scout

**Fluxo típico para Scout:**

1. Executar `firecrawl search` com query [área] [localização]
2. Parsear JSON, extrair URLs
3. Para cada URL (até limite de vagas desejadas):
   - Executar `firecrawl scrape <url> --format markdown`
   - Extrair: título, empresa, localização, requisitos
4. Se scrape falhar, usar descrição do search como fallback

## Output Parsing

### JSON (de `search` ou `scrape` sem `--format markdown`)

```bash
firecrawl search "..." --json
```

Saída: array JSON ou objeto JSON.

**Parsing (pseudocódigo)**:
```
resultado = JSON.parse(saida_terminal)
para cada item em resultado:
    extrair: url, title, description
```

### Markdown (de `firecrawl scrape ... --format markdown`)

Saída: texto puro em Markdown.

**Parsing (pseudocódigo)**:
```
linhas = markdown.split('\n')
para cada linha:
    se "Requirements" ou "Requisitos": marcar início section
    se "#" novo nível heading: nova seção
    coletar habilidades/requisitos conforme reconhecimento de palavra-chave
```

## Variáveis de Ambiente

### Definir `FIRECRAWL_API_KEY`

**Windows PowerShell**:
```powershell
$env:FIRECRAWL_API_KEY = "seu-api-key-aqui"
```

**Windows CMD**:
```cmd
set FIRECRAWL_API_KEY=seu-api-key-aqui
```

**Linux/Mac Bash**:
```bash
export FIRECRAWL_API_KEY="seu-api-key-aqui"
```

### Verificar se está definida

```bash
echo $env:FIRECRAWL_API_KEY  # PowerShell
echo $FIRECRAWL_API_KEY       # Bash
```

## Limitações Conhecidas

1. **Páginas dinâmicas (JavaScript)**: Firecrawl renderiza JS, mas algumas SPAs complexas podem retornar incompletamente.
2. **Login obrigatório**: Se a página exigir login, Firecrawl não consegue acessar (sem cookies salvos).
3. **Anti-bot**: Alguns sites bloqueiam Firecrawl. Usar fallback se necessário.
4. **Encoding**: Caracteres especiais (acentos, símbolos) geralmente são preservados em Markdown.

## Suporte e Documentação

- Site oficial: https://www.firecrawl.dev/
- Docs CLI: https://www.firecrawl.dev/docs/cli
- Status API: https://www.firecrawl.dev/status

