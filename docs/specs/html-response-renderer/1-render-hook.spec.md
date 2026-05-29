---
status: discarded
feature: HTML Render Hook
reproduction-test: none
created: 2026-05-21
updated: 2026-05-21
discard-reason: Hook approach abandoned in favor of user-invocable skill (ahead:visualize). See `_overview.md` "Why discarded".
---

# HTML Render Hook (DISCARDED)

## Goal

Pipeline completo que, a cada turno do Claude Code (evento `Stop`), extrai a última resposta do assistant, envia pra um agente Claude (Haiku 4.5 via API direta) que a transforma em HTML estilizado, injeta num template, escreve em arquivo por sessão, abre no browser (apenas na primeira vez) e dispara notificação de desktop.

## Why

Respostas longas no chat monoespaço do Claude Code são difíceis de ler: tabelas comprimidas, hierarquia visual fraca, exemplos misturados com prosa, estado/contexto difícil de localizar. Um HTML por sessão com tipografia decente, callouts estilizados e refresh automático na aba do browser dá um canal de leitura paralelo ao chat, sem alterar a conversação em si.

Por que agente LLM e não conversor de markdown: vide `_overview.md` Design Decision 1. Resumindo: a meta é reestruturação intencional pra clareza, não conversão literal.

## Requirements

### 1. Trigger via hook `Stop`

- Registrado em `~/.claude/settings.json` apontando pra `~/.claude/rules/hooks/render-last.mjs`
- O snippet de `settings.json` necessário pra ativação fica documentado no `README.md` do repo de regras, na seção "Hooks"
- O hook recebe via stdin um JSON com (no mínimo) `session_id` e `transcript_path` — campos exatos a confirmar via doc do Claude Code antes de implementar
- O script sempre exita 0, mesmo em erro, pra não travar o fluxo do Claude Code; erros vão pra `~/.claude/renders/<session-id>.log`

**Edge cases:**
- Payload JSON malformado ou faltando campos → loga e sai sem renderizar
- `ANTHROPIC_API_KEY` ausente → loga mensagem clara e sai
- Transcript path inexistente → loga e sai

### 2. Extração da última resposta do transcript

- Lê o arquivo apontado por `transcript_path` (JSONL)
- Localiza a última mensagem com `role: assistant`
- Filtra blocos da mensagem: descarta `thinking`, `tool_use`, `tool_result`; mantém apenas `text`
- **Estratégia inicial**: passa todos os blocos `text` da última mensagem do assistant pro agente. O próprio agente decide o que é "narração intermediária" e atenua, conforme o prompt.
- Se a mensagem não tiver nenhum bloco `text` (turno foi só tool calls), sai sem renderizar

**Edge cases:**
- Transcript vazio ou só com user messages → sai sem renderizar
- JSONL parcialmente corrompido → tenta parsear linha a linha, ignora as quebradas

### 3. Chamada ao agente conversor

- Canal: `claude -p` (Claude Code CLI não-interativo) via `child_process.spawn`
- Modelo: `claude-haiku-4-5-20251001` (forçado via `--model`)
- System prompt: lido de `~/.claude/rules/hooks/prompt.md` e passado via `--system-prompt`
- User message: o texto extraído no passo 2, enviado via stdin pro processo filho
- Settings override: `--settings '{"hooks":{}}'` (defesa contra loop, complementa a guarda de env var)
- Persistência: `--no-session-persistence` (evita poluir `~/.claude/projects/`)
- Output: `--output-format text` (default), stdout do filho é o HTML
- Timeout: 90s (cold-start do `claude -p` pode levar alguns segundos)
- cwd do spawn: `/tmp` (evita carregar CLAUDE.md do projeto do usuário)
- Env: `CLAUDE_RENDERER=1` no env do filho (guard rail anti-loop — ver §9)

**Resposta esperada do `claude -p`:**
- Exit code 0
- stdout: HTML body content (sem `<html>`/`<head>`/`<body>` wrapper)
- Pode vir com code fence ```` ```html ... ``` ```` se o modelo for verboso — script faz strip

**System prompt deve instruir o agente a:**
- Produzir HTML body válido (sem `<html>`, `<head>`, `<body>` — só o conteúdo interno)
- Usar classes CSS que o template define (`callout`, `code-block`, `verdict good/warn/bad`, etc.)
- Nível **moderado** de reestruturação: pode reorganizar hierarquia, lift exemplos pra callouts, mesclar parágrafos redundantes, atenuar narração intermediária. Não pode inventar conteúdo, mudar conclusões ou omitir informação substantiva.
- Preservar a língua do original (PT ou EN)
- Se houver tool calls relevantes pro contexto (ex.: "li o arquivo X e encontrei Y"), mencionar de forma resumida — não listar burocraticamente
- Output deve começar direto com a primeira tag HTML, sem preâmbulo nem code fence

**Edge cases:**
- `claude` CLI ausente do PATH → spawn error → loga, sai
- `claude -p` exit ≠ 0 (auth expired, rate limit, etc.) → loga exit code + stderr tail (últimas 5 linhas) → sai
- Timeout (90s estourado) → SIGTERM no filho, loga e sai
- stdout vazio → loga, sai sem escrever HTML
- Resposta começa com ```` ```html ```` ou similar → script faz strip do code fence

### 4. Template HTML + injeção

- Template em `~/.claude/rules/hooks/template.html` com placeholders `{{BODY}}` e `{{TITLE}}` e `{{POLLER_SCRIPT}}`
- CSS embutido no `<head>` — render standalone, abre direto no browser, sem dependências externas
- Mesma design language do `~/.claude/rules/observation-template.html` (paleta, tipografia, badges, callouts)
- `{{TITLE}}` = nome do projeto extraído do CWD do transcript (parsing simples: último segmento do path antes do CWD do Claude Code; fallback = session-id truncado)
- `{{POLLER_SCRIPT}}` = bloco `<script>` que faz o auto-refresh (requirement 6)

### 5. Escrita do arquivo por sessão

- Output: `~/.claude/renders/<session-id>.html`
- Diretório `~/.claude/renders/` criado pelo script se não existir (`mkdir -p`)
- Escrita atômica: escreve em `<session-id>.html.tmp` e renomeia — evita o JS poller pegar arquivo meio escrito

### 6. Auto-refresh na aba do browser (JS poller)

- Embutido no template via `{{POLLER_SCRIPT}}`
- Comportamento:
  - A cada 1.5s, `fetch(window.location.href + '?_=' + Date.now())` (cache-busting)
  - Compara o `<body>` da resposta com o atual; se diferente, substitui `document.body.innerHTML`
  - Preserva posição de scroll usando `window.scrollY` antes/depois da troca
- Não usa `<meta http-equiv="refresh">` (reseta scroll)
- Se o fetch falhar, ignora silenciosamente e tenta de novo no próximo tick

**Edge cases:**
- Arquivo deletado enquanto a aba está aberta → fetch 404 → poller continua tentando (eventual readback se sessão revivida)
- Resposta gigante (>1MB) → poller ainda funciona, mas troca pode flickar; aceito

### 7. Abertura do browser (1x por sessão)

- Sentinel file: `~/.claude/renders/<session-id>.opened`
- Antes de abrir, script verifica se o sentinel existe
  - Não existe → `xdg-open <session-id>.html` e cria o sentinel
  - Existe → pula a abertura
- Sentinel é criado **depois** do `xdg-open` retornar 0, pra evitar estado órfão se o open falhar
- Sentinel persiste enquanto a sessão existir; não é limpo automaticamente (limpeza é responsabilidade de cleanup futuro)

**Edge cases:**
- `xdg-open` indisponível → loga, segue (notificação ainda dispara)
- Usuário fechou a aba manualmente → sentinel ainda existe, script não reabre. **Limitação conhecida da v1**. Workaround: deletar manualmente o sentinel. Evolução futura: detectar se a aba ainda está aberta (não trivial sem servidor).

### 8. Notificação de desktop

- `notify-send "<TITULO>" "<CORPO>"` chamada após o HTML ser escrito
- **Título**: `Claude — <nome do projeto>` (nome extraído como no template)
- **Corpo v1**: `Resposta pronta` (string fixa, sem custo extra de tokens)
- Evolução futura (não nesta spec): corpo gerado pelo próprio agente (primeira frase ou resumo de uma linha)
- Se `notify-send` não estiver disponível, loga e segue

### 9. Prevenção de loop

Crítico: quando o script chama `claude -p`, o filho ao terminar também dispara seu próprio `Stop` hook, que invocaria esse mesmo script de novo → loop infinito se não houver guarda.

**Defesa principal (load-bearing)**: env var `CLAUDE_RENDERER=1` setado no env do filho. Primeira linha do script verifica `if (process.env.CLAUDE_RENDERER === '1') process.exit(0)`. Quando o hook do filho disparar, o script entra, vê o env var, e sai imediatamente — sem chamar `claude -p` de novo. Recursão para na 2ª camada.

**Defesa secundária (belt-and-suspenders)**: `--settings '{"hooks":{}}'` passado no `claude -p`. A intenção é zerar hooks pro filho — efetividade depende de como o CC mergeia settings (arrays podem mergear em vez de substituir), por isso não é a defesa principal.

**Smoke check de loop**: rodar manualmente o smoke test e confirmar que `~/.claude/renders/` ganha 1 (um) HTML, não 2 ou ∞.

### 10. Tratamento de erro robusto

- Todo erro é capturado em try/catch top-level
- Stack trace + contexto vão pra `~/.claude/renders/<session-id>.log` (append mode)
- Script sempre exita 0 (não derrubar o turno do Claude Code)
- Se qualquer um dos passos críticos (API, escrita) falhar, o turno fica sem HTML mas o usuário não fica sem chat

## Out of Scope (v1)

- Limpeza automática de renders/sentinels antigos — usuário limpa manualmente
- Corpo da notificação gerado pelo agente (custo extra de tokens) — fica como string fixa
- Servidor local com SSE pra push de atualizações — JS poller é suficiente
- Histórico navegável (lista de renders, índice) — escopo de outra feature
- Re-abrir aba se usuário fechou manualmente — limitação aceita
- Suporte cross-platform além de Linux — Linux-only v1 (`xdg-open`, `notify-send` são Linux)
- Customização per-projeto do template ou prompt — global only

## Open Questions (resolvidas durante implementação)

1. **Formato exato do payload stdin do hook `Stop`** — ✅ Resolvido. Confirmado via validação de `settings.json` schema: campos disponíveis incluem `session_id`, `transcript_path`, `cwd`, `permission_mode`, `hook_event_name`.
2. **Caminho do transcript** — ✅ Resolvido. Caminho absoluto, transcript já está escrito quando o hook dispara.
3. **CWD do Claude Code** — ✅ Resolvido. Campo `cwd` está no payload.
4. **Schema de `settings.json` para hooks** — ✅ Resolvido durante implementação (a doc original encontrada estava incompleta). O formato correto é aninhado:
   ```json
   "hooks": {
     "Stop": [
       { "matcher": "", "hooks": [ { "type": "command", "command": "...", "async": true, "timeout": 60 } ] }
     ]
   }
   ```
   Cada entrada em `Stop[]` é um grupo `{ matcher, hooks[] }` — `matcher` é string (vazia pra Stop, que não tem tool pra filtrar), e `hooks[]` contém as definições reais.

## Validation

Validação manual (sem suite automatizada):

1. Configurar o hook + script + template + prompt + `ANTHROPIC_API_KEY`
2. Abrir uma sessão nova do Claude Code num projeto
3. Fazer uma pergunta que gere resposta razoavelmente longa
4. Verificar:
   - HTML criado em `~/.claude/renders/<session-id>.html`
   - Browser abriu automaticamente na primeira resposta
   - Notificação de desktop apareceu
   - HTML está estilizado, legível, reestruturado adequadamente (não conversão literal)
5. Fazer segunda pergunta na mesma sessão
6. Verificar:
   - HTML foi atualizado (mesma aba, sem nova aba aberta)
   - Notificação disparou de novo
   - Scroll position preservado se você scrollou no HTML antes
7. Abrir uma segunda janela do Claude Code em outro projeto
8. Verificar:
   - Cada janela tem seu próprio HTML (session-id diferente)
   - Notificações distinguem os projetos
9. Forçar erros (revogar API key, etc.) e verificar:
   - Turno do Claude Code não trava
   - Log de erro escrito em `<session-id>.log`
