# Web Search — Pesquisa na Web (Skill Independente)

Skill **totalmente independente** do question-card, integrações OAuth e computer-use. Ativa o motor de pesquisa do ULTRON e o card visual "Pesquisou na web" no chat.

## Motor de pesquisa

| Modo | Configuração |
|------|----------------|
| **Padrão** | DuckDuckGo HTML com User-Agent Chromium (sem API key) |
| **Opcional** | Google Custom Search: `GOOGLE_CSE_API_KEY` + `GOOGLE_CSE_CX` no `.env` |

Endpoint: `POST /api/web-search` — body `{ "query": "...", "limit": 8 }`

## Quando a IA deve pesquisar

- Informação atual (notícias, versões, preços, eventos)
- O utilizador pede pesquisa na web / Deep Research
- Factos que não podes confirmar sem fontes

## Formato XML (ordem = ordem no chat)

### Bloco com várias pesquisas

```xml
<web_research>
  <thought>O utilizador pediu informação sobre X. Vou pesquisar termos relevantes.</thought>
  <search query="primeiro termo" />
  <search query="segundo termo mais específico 2026" />
</web_research>
```

### Pesquisa avulsa inline

```xml
<web_search query="termo exato" thought="opcional: uma frase" />
```

**Regras:**
- Não inventar URLs, títulos nem domínios
- O sistema executa a pesquisa real e mostra o card
- Coloca os blocos XML **na ordem** em que queres que apareçam na resposta (antes ou entre parágrafos Markdown)

## Resposta automática após pesquisa

1. A IA emite `<web_research>` (só pesquisas, sem inventar links)
2. O servidor executa as pesquisas e mostra o **card**
3. O sistema chama a IA **outra vez** com os resultados reais e **acrescenta** o Markdown abaixo do card

O utilizador vê: card → texto de resposta. Indicador: *"A redigir resposta com base na pesquisa..."*

## Card no chat

O componente `web-search-card` renderiza **inline na mensagem**, por ordem:

1. Ícone relógio + raciocínio (`<thought>`)
2. Por cada `<search>`: globo + query + "N resultados" + lista com favicon
3. "Concluído" quando terminar

## Ficheiros

| Ficheiro | Função |
|----------|--------|
| `src/search/chromium-search.ts` | Motor de pesquisa |
| `src/utils/web-search-parser.ts` | Parser XML + blocos ordenados |
| `src/components/web-search-card.ts` | UI do card |
| `Skills/web-search/SKILL.md` | Esta skill |

## Troubleshooting

- **0 resultados**: DDG pode bloquear IP; configurar Google CSE no `.env`
- **Card não aparece**: verificar consola `[WebSearch]` — a IA tem de emitir `<web_research>` ou `<web_search>`
- **Só texto**: a IA listou links em Markdown em vez de XML — repetir instrução da skill
