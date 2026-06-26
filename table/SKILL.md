# Data Table — Tabelas Estruturadas no Chat (Skill Independente)

Skill **totalmente independente** que permite à IA emitir tabelas estruturadas no chat em vez de as escrever em Markdown. A tabela aparece renderizada como cartão entre o texto, com cabeçalho sticky, ordenação por coluna, scroll horizontal e formatação de números/datas/moedas.

## Quando a IA deve usar

- **Comparações** (X vs Y, prós vs contras, frameworks, linguagens, ferramentas)
- **Rankings** (top N, leaderboards, melhores/seguintes)
- **Listas tabulares** (2+ linhas × 2+ colunas de dados comparáveis)
- **Dados numéricos estruturados** (preços, datas, métricas, specs, KPIs)
- **Configurações** (variáveis de ambiente, opções com chave+valor+descrição)

**Não usar** para: texto corrido, listas simples de uma coluna, prosa, snippets de código.

## Formato XML (JSON dentro do atributo `data`)

A IA emite UM bloco `<table>` por tabela, com a estrutura toda em JSON no atributo `data`. **A posição do `<table>` no texto = posição da tabela na resposta.**

```xml
<table data='{
  "caption": "Comparação de frameworks JavaScript",
  "columns": [
    { "key": "name",    "label": "Framework" },
    { "key": "year",    "label": "Ano",          "align": "right", "type": "number" },
    { "key": "company", "label": "Criador" }
  ],
  "rows": [
    { "name": "React",  "year": 2013, "company": "Meta" },
    { "name": "Vue",    "year": 2014, "company": "Evan You" },
    { "name": "Svelte", "year": 2016, "company": "Rich Harris" }
  ]
}' />
```

### Regras do JSON

| Campo | Obrigatório | Descrição |
|------|-------------|-----------|
| `caption` | opcional | Título exibido em cima da tabela |
| `columns` | sim | Array de `{key, label, align?, type?, width?}` |
| `rows` | sim | Array de objetos `{colKey: valor}` ou arrays `[v1, v2]` |

### Tipos de coluna

| `type` | Formatação |
|--------|------------|
| `text` (default) | Texto simples |
| `number` | `1 234 567` (separador de milhares) |
| `currency` | `1 234,56 €` (usa `Intl.NumberFormat`) |
| `percent` | `12,5%` |
| `date` | Tenta ISO `YYYY-MM-DD`; mostra como está se falhar |

### Alinhamento (`align`)

- `left` (default) — texto, labels
- `right` — números, valores
- `center` — sim/não, status, badges

## Regras para a IA

1. **NÃO inventar dados** — se não tens a certeza, diz "não tenho essa informação" em texto normal.
2. **Posicionar entre parágrafos** — o `<table>` aparece onde for emitido.
3. **Aspas:** delimita `data=` com `'...'` no XML; dentro do JSON usa `"..."`.
4. **Limite de tamanho:** ~30 linhas × 10 colunas é o sweet spot. Para mais, divide em várias tabelas.
5. **Uma tabela por bloco** — se tens 2 datasets, emite 2 `<table>` separados.

## Card no chat

O componente `<data-table-card>` renderiza inline na mensagem, na posição onde a IA emitiu a tag. Inclui:

- **Caption** (se existir) em cima
- **Cabeçalho sticky** que não rola com o body
- **Linhas zebra** (fundo alternado) para legibilidade
- **Hover** em cada linha (highlight suave)
- **Ordenação** clicando no header (asc/desc)
- **Overflow-x** automático em ecrãs estreitos
- **Botão "Copiar como Markdown"** no canto superior direito
- **Collapse/expand** para tabelas grandes
- **Estado de erro** se o JSON for inválido (mostra mensagem útil, não crasha)
- **Estado "A gerar tabela..."** durante streaming enquanto a tag `<table>` ainda não fechou

## Ficheiros

| Ficheiro | Função |
|----------|--------|
| `src/table/types.ts` | Tipos `DataTable`, `DataTableColumn`, `DataTableRow` |
| `src/utils/table-parser.ts` | Parser XML+JSON + `TABLE_INSTRUCTION` + helpers |
| `src/components/data-table-card.ts` | UI do cartão (Lit element) |
| `Skills/table/SKILL.md` | Esta skill |

## Integração

A skill é activada automaticamente — basta estar presente nos imports e na system prompt de `src/app-root.ts` (já feito). A IA recebe a `TABLE_INSTRUCTION` no system prompt e sabe quando a invocar.

## Troubleshooting

- **Tabela não aparece:** a IA pode estar a usar Markdown (`| col | col |`) em vez de XML. Repetir a instrução.
- **JSON inválido:** a IA pode ter metido `"` dentro do `data='...'` (usou aspas duplas dentro de aspas simples). Devia usar `'` no atributo XML e `"` no JSON.
- **Tabela distorcida:** alinhamento automático — usa `align: 'right'` em colunas de números e `type: 'number'` para formatar.
- **Muitas colunas:** define `width` em cada coluna (ex.: `width: '120px'`) para forçar larguras.
