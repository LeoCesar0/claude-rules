---
status: discarded
created: 2026-05-21
updated: 2026-05-21
discard-reason: Approach abandoned — see "Why discarded" below. Replaced by user-invocable skill `ahead:visualize`.
---

# HTML Response Renderer (experimental, DISCARDED)

## Why discarded

A abordagem de hook `Stop` automático foi testada e abandonada em 2026-05-21 por duas razões fundamentais que apareceram em uso real:

1. **Race condition transcript ↔ hook**: o hook `Stop` dispara antes do CC terminar de gravar a última mensagem do assistant no transcript JSONL. O script lia o arquivo num momento em que ele estava incompleto (162 chars de 2128 no caso real observado). Tentativas de mitigar via poll de mtime ajudaram mas não eliminaram o problema — o transcript continua sendo escrito em pedaços, e múltiplas janelas do CC tornam o sinal de "estabilidade" ambíguo.

2. **Conversational drift do `claude -p`**: o child Claude (invocado pra renderizar HTML) ignora o system prompt e responde como assistente conversacional quando o input parece uma pergunta/tarefa. Caso real observado: input "Vou checar se há alguma falha no log" virou resposta "Não vejo nenhum log anexado em sua mensagem…" em vez de HTML. Mitigar isso exigiria desabilitar o CLAUDE.md global do child via `--bare`, que por sua vez exige `ANTHROPIC_API_KEY` separado — o que justamente queríamos evitar.

Tentativas de defesa em profundidade (`<content-to-render>` envelope, `claudeMdExcludes` no `--settings`, validação `looksLikeHtml`) funcionaram em testes isolados mas não dão garantia em produção sem `--bare`. O custo de manter heurísticas que falham silenciosamente é alto.

**Decisão**: virar pra skill manualmente invocável (`ahead:visualize`). A skill é executada pela MESMA sessão CC que produziu o conteúdo, o que elimina os dois problemas estruturais:
- Não há race condition porque a skill roda dentro da conversa, não depende de transcript flush
- Não há drift porque é a mesma instância do Claude que conhece o contexto e o intent
- Bônus: a skill pode **enriquecer** o conteúdo (explicar, adicionar contexto, definir termos), não só typesetar. Isso é o que o usuário queria de verdade.

Spec preservada como registro do que foi tentado e por que não funcionou — útil pra evitar repetir a abordagem no futuro.

## Goal (original)

Renderizar automaticamente cada resposta final do Claude Code em um arquivo HTML estilizado, por sessão (suporta múltiplas janelas em paralelo), abrindo no browser na primeira resposta da sessão e sinalizando atualizações subsequentes via notificação de desktop.

Motivação: respostas longas no chat monoespaço são difíceis de ler — tabelas, hierarquia, exemplos, estado atual. Um HTML bem formatado, gerado por um agente Claude dedicado (não conversão mecânica de markdown), reestrutura a resposta pra leitura, sem alterar o conteúdo da conversação em si.

## Specs

| # | Spec | Status | Notes |
|---|------|--------|-------|
| 1 | [render-hook](1-render-hook.spec.md) | 🗑️ discarded | Implementação foi feita e testada; abandonada por race condition + conversational drift. Substituída por skill `ahead:visualize` |

## Design Decisions

Opções discutidas e a escolha pra v1. Cada decisão lista alternativas pra registro — pra que a evolução futura saiba o que foi considerado.

### 1. Conversor: agente LLM vs. script de markdown

- **Opção A — Agente Claude dedicado** ← escolhida
- Opção B — Conversor mecânico (markdown-it, pandoc)

**Escolhida A**. Razão: o usuário quer reestruturação intencional pra clareza (lift de exemplos pra callouts, hierarquia repaginada, narração interna atenuada) — não conversão 1:1 de markdown→HTML. Um agente entende intenção; um conversor mecânico só faz transformação literal.

Trade-off aceito: custo de tokens (~$3-15/mês estimado) e latência adicional de 2-5s por turno.

### 2. Canal de chamada do agente

- Opção A — Anthropic API direta (escolha inicial, revertida)
- **Opção B — `claude -p` (Claude Code CLI em modo não-interativo)** ← escolhida na revisão pós-implementação

**Histórico**: a v0 do script implementou Opção A (API direta com `ANTHROPIC_API_KEY`). O usuário pediu pra evitar a dependência de API key separada e reusar a auth ativa do Claude Code, então a feature foi migrada pra Opção B.

**Razões pra Opção B**:
- Sem `ANTHROPIC_API_KEY` separada — reusa a auth já configurada do Claude Code (OAuth ou keychain)
- Refresh de token gerenciado pelo CC, sem código de auth no script
- Funciona com Claude Pro/Max (OAuth) ou API key dependendo do que o usuário tem logado

**Trade-offs aceitos**:
- Consome quota da assinatura do Claude Code (Opção A não consumia)
- Latência maior: ~5-15s vs. ~2-5s (overhead de boot do `claude -p`)
- Risco de loop existe mas é mitigado estruturalmente — ver Design Decision 9

**Comando concreto**:
```bash
claude -p \
  --model claude-haiku-4-5-20251001 \
  --system-prompt "<conteúdo de prompt.md>" \
  --settings '{"hooks":{}}' \
  --no-session-persistence \
  --output-format text
```
User text via stdin. Spawn com `cwd: /tmp` e `env.CLAUDE_RENDERER=1`.

### 3. Modelo

- **Opção A — Haiku 4.5** ← escolhida
- Opção B — Sonnet 4.6

**Escolhida A**. Markdown→HTML estilizado com reestruturação leve está dentro da capacidade do Haiku. Mais rápido e ~10x mais barato. Se a qualidade ficar abaixo do desejado, migrar pra Sonnet é trocar uma string.

### 4. Estratégia de arquivo

- **Opção A — Per-session file (`<session-id>.html`)** ← escolhida
- Opção B — Arquivo aleatório novo a cada Stop
- Opção C — Arquivo único compartilhado

**Escolhida A**. Razões:
- Suporta múltiplas janelas do Claude Code em paralelo (cada janela escreve no próprio arquivo)
- Estável por sessão → mesma aba do browser pode atualizar via auto-refresh
- Sem acúmulo descontrolado de arquivos (1 por sessão, não 1 por turno)

Localização: `~/.claude/renders/<session-id>.html`.

### 5. Abertura do browser

- **Opção A — Abre 1x por sessão + notificação a cada Stop** ← escolhida
- Opção B — Abre sempre (várias abas piling up)
- Opção C — Servidor local com SSE (push)

**Escolhida A**. Razões:
- 1 aba por sessão (sem bagunça de N abas idênticas)
- Notificação (`notify-send`) dá sinal visual de "pronto" independente de onde a atenção esteja
- Sem servidor long-running

Mecanismo: sentinel file `~/.claude/renders/<session-id>.opened` indica que a aba já foi aberta. Primeiro Stop cria o sentinel + `xdg-open`. Subsequentes só reescrevem o HTML.

Opção C fica como evolução futura se a v1 mostrar limitações.

### 6. Auto-refresh dentro da aba

- **Opção A — JS poller (`fetch` + comparação + `innerHTML` replace)** ← escolhida
- Opção B — `<meta http-equiv="refresh">`

**Escolhida A**. Meta refresh perde a posição de scroll a cada 2s — irritante em respostas longas. JS poller preserva scroll (substitui só o `<body>`).

### 7. Nível de criatividade do agente conversor

- Conservador — preserva estrutura, só estiliza
- **Moderado — reestrutura pra clareza, não inventa conteúdo** ← escolhida
- Criativo — repagina livremente

**Escolhida moderado**. Pode lift exemplos pra callouts, mesclar parágrafos redundantes, atenuar narração intermediária. Não pode inventar dados, mudar conclusões ou omitir informação substantiva.

### 8. Limpeza de arquivos antigos

- **Não decidida**. Opções: cron diário apagando renders >7 dias, cleanup quando o script roda (apaga próprio sentinel + html ao detectar inatividade), ou manual.
- v1 começa sem cleanup automático; revisita se acumular incomodar.

### 9. Conteúdo da notificação

- **Não decidida**. Opções: só "pronto", ou resumo curto (custo extra de tokens pro agente gerar). v1: só "pronto" + nome do projeto extraído do CWD do transcript.

## Implementation Notes

Quando implementar qualquer spec deste grupo:

- O hook é executado pelo harness do Claude Code, não pelo Claude. Configuração vive em `~/.claude/settings.json`. Skill `update-config` confirma o padrão.
- Reuso: o `template.html` deve compartilhar a paleta/tipografia de `~/.claude/rules/observation-template.html` (mesma design language) — não duplicar CSS, considerar extrair tokens comuns se valer.
- Testes: implementação é experimental e local; não há CI/test suite. Validação é manual: rodar uma sessão real e ver se o HTML aparece + notifica.
- Atualizar o README do repo de regras (`~/.claude/rules/README.md`) com seção "Hooks" documentando o snippet de `settings.json` necessário pra ativar em outra máquina (regra global: README é fonte de verdade pra setup de nova máquina).
- A spec deste grupo será impactada caso o formato do payload do hook `Stop` no Claude Code não bata com a suposição (campos `session_id` e `transcript_path` via stdin JSON) — confirmar via doc oficial antes de codar.

### Group-specific anchors / constraints

- **Script principal**: `~/.claude/rules/hooks/render-last.mjs` (Node, ESM). Dependências: nativas se possível; se não, `package.json` mínimo no diretório do hook.
- **Template**: `~/.claude/rules/hooks/template.html` com `{{BODY}}` e `{{TITLE}}` como placeholders.
- **Prompt do agente**: `~/.claude/rules/hooks/prompt.md` — versionado, ajustável sem mexer no script.
- **Output**: `~/.claude/renders/<session-id>.html` + `~/.claude/renders/<session-id>.opened` (sentinel).
- **Env vars**: `ANTHROPIC_API_KEY` obrigatório; script falha cedo com mensagem clara se ausente.
- **Logs de erro**: stderr do script é capturado pelo harness; falhas não devem travar o turno do Claude Code (script deve sempre exitar 0, logar erros em `~/.claude/renders/<session-id>.log`).

## Dependencies

- Node (>= 18) — uso de `spawn` e `fetch` nativos
- `claude` CLI em PATH (already required to use Claude Code)
- Auth ativa do Claude Code (OAuth via claude.ai ou API key configurada no CC) — reusada pelo `claude -p` no spawn
- Ferramentas Linux: `xdg-open` (abrir browser), `notify-send` (notificação). Ambas vêm por padrão no Ubuntu.
- **Não requer** `ANTHROPIC_API_KEY` no env.
