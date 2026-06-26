# Question Card — Perguntas de Múltipla Escolha no Chat

Skill que permite ao ULTRON apresentar um card interativo de perguntas de múltipla escolha no chat, igual ao design escuro com opções numeradas, paginação, "Outra opção" e "Pular".

## Quando usar

A IA deve emitir um card `<questions>` quando:

- Precisa de uma resposta estruturada antes de continuar
- Há 2–6 opções mutuamente exclusivas (ou seleção múltipla)
- Está num fluxo de onboarding, configuração ou clarificação de requisitos
- O utilizador pediu escolha entre opções concretas

**Não usar** para perguntas abertas — nesse caso, perguntar normalmente no texto.

## Formato XML

```xml
<questions>
  <question id="tempo-livre" prompt="Como preferes passar o teu tempo livre?">
    <option id="em-casa">Em casa a relaxar</option>
    <option id="cidade">A explorar a cidade</option>
    <option id="natureza">Na natureza</option>
    <option id="amigos-familia">Com amigos/família</option>
  </question>
  <question id="estilo-trabalho" prompt="Qual é o teu estilo de trabalho preferido?">
    <option id="focado">Blocos longos de foco</option>
    <option id="pomodoro">Sessões curtas com pausas</option>
    <option id="flexivel">Horário flexível ao longo do dia</option>
  </question>
</questions>
```

### Regras

| Campo | Regra |
|-------|-------|
| `question id` | Único, kebab-case |
| `prompt` | Pergunta completa terminada em `?` |
| `option id` | kebab-case descritivo (não usar `1`, `2`, `3`) |
| Opções | 2–6 por pergunta; labels curtas |
| Bloco | Até 5 `<question>` por `<questions>` (paginação automática: 1 de N) |
| `allow_multiple="true"` | Só quando várias opções são válidas |

### Onde colocar no XML

- **Modo chat**: fora do markdown, normalmente no final da mensagem (antes de `<suggestions>`)
- **Modo agente**: fora de `<response>`, quando precisar de input do utilizador antes de continuar o plano

## Comportamento no UI

O componente `question-card` renderiza:

- Cabeçalho com pergunta + navegação `‹` `N de M` `›` + fechar
- Opções numeradas; selecionada fica destacada com seta `→`
- **Outra opção** — campo de texto livre
- **Pular** — salta a pergunta atual

Quando o utilizador termina, o chat envia automaticamente:

```
[Respostas às perguntas]
- Como preferes passar o teu tempo livre?: Em casa a relaxar
- Qual é o teu estilo de trabalho preferido?: (pulado)
```

A IA deve **aguardar** esta mensagem antes de continuar.

## Exemplo completo (modo chat)

Texto introdutório + card + sugestões:

```
Antes de começarmos, preciso de saber umas preferências tuas.

<questions>
  <question id="tempo-livre" prompt="Como preferes passar o teu tempo livre?">
    <option id="em-casa">Em casa a relaxar</option>
    <option id="cidade">A explorar a cidade</option>
    <option id="natureza">Na natureza</option>
    <option id="amigos-familia">Com amigos/família</option>
  </question>
</questions>

<suggestions>
  <suggestion>Explica para que serves estas perguntas</suggestion>
  <suggestion>Pular tudo e ir direto ao assunto</suggestion>
  <suggestion>Mostra outro exemplo de card</suggestion>
</suggestions>
```

## Anti-padrões

- Listar opções A/B/C em markdown em vez do card
- Uma tag `<questions>` por opção (agrupar num bloco)
- Continuar a tarefa sem esperar `[Respostas às perguntas]`
- Ids genéricos como `opt1`, `opt2`

## Componente

- Ficheiro: `src/components/question-card.ts`
- Tag: `<question-card>`
- Eventos: `questions-complete`, `questions-dismissed`
