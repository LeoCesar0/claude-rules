# Proposta — Gates de investigação do Claude Cookbooks

> **Status:** rascunho/proposta, nada implementado. Documento de discussão para
> decidir se e como criar essa skill/gate.
> **Não é lido automaticamente pelo Claude** — vive em `_inactive/`, que está em
> `claudeMdExcludes` no `settings.json`.
> **Criado em:** 2026-07-15
> **Fonte mapeada:** `/home/leonardo/Desktop/Work/claude-cookbooks-main` (cópia
> local do repo `anthropics/claude-cookbook`, sem `.git` — congela no estado do
> download; o caminho e a frescura são pontos em aberto abaixo).

---

## Conceito

Ao trabalhar em um repo do usuário, certas tarefas tocam áreas que o Claude
Cookbooks cobre com receitas oficiais da Anthropic (padrões de agente, memória,
prompts, evals, RAG, etc.). A ideia: um mecanismo de **investigação por
demanda** — quando a tarefa atual cruza com uma área mapeada, o Claude oferece
investigar o cookbook antes de implementar, e só investiga com autorização.

O que ele **não** é:

- Não é um import automático de código do cookbook para o projeto.
- Não é uma leitura preventiva/sempre-ativa — o cookbook nunca entra no
  contexto sem o gate 1 aprovado.
- Não substitui "Orient Before Building" (docs do próprio projeto vêm
  primeiro); é um complemento externo, opcional por tarefa.

## Fluxo proposto (dois gates)

1. **Detecção** — durante planejamento ou início de implementação, a tarefa
   bate em um gatilho da tabela de mapeamento abaixo.
2. **Gate 1 — autorização de investigação.** O Claude pergunta, em uma linha,
   se pode investigar o cookbook, nomeando a(s) pasta(s) exata(s) e o que
   espera trazer de lá. Ex.: *"Isso toca memória de conversação — posso
   investigar `tool_use/memory_cookbook.ipynb` e
   `tool_use/context_engineering/` e trazer o que se aplicaria aqui?"*
   - Sem autorização → segue a tarefa normalmente, sem tocar o cookbook.
   - A oferta é feita **uma vez por área por tarefa** — negou, não insiste.
3. **Investigação** (autorizada) — leitura limitada às pastas nomeadas no
   gate 1. Preferencialmente via subagente (Explore) para não inflar o
   contexto principal; o subagente devolve síntese, não dumps de notebook.
4. **Relato — "o que mudaria"** — ao terminar, o Claude apresenta os
   aprendizados como propostas concretas: para cada item, o que é, o que
   mudaria no código/abordagem atual (antes vs. depois), e o custo de aplicar.
   Apresentação via decision protocol (`decision-protocol.md`), um ponto por
   vez quando houver 2+ itens.
5. **Gate 2 — aceite por item.** O usuário aceita, rejeita ou adapta cada
   proposta individualmente. Só o que for aceito é aplicado. Rejeitados podem
   virar observation (`enhancement`) se o usuário quiser registro.

## Restrições (herdam do sistema atual)

- Investigação é **read-only** no cookbook — nunca editar, executar notebooks
  ou instalar dependências dele.
- Receita que sugerir dependência nova no projeto → passa pelo
  Dependency Supply-Chain Gate de `security.md`, sem atalho.
- Código de receita nunca é colado como-está — é adaptado às convenções do
  projeto (DRY, tipos, padrões existentes).
- O relato do gate 2 distingue: **aplicável agora** vs. **aprendizado para
  depois** (registrável como observation/memória, sem mudança de código).

## Mapeamento — gatilho → onde investigar

Gatilhos são situações de trabalho no repo do usuário, não pedidos explícitos.
Caminhos relativos à raiz do cookbook.

| # | Gatilho (trabalhando em...) | Investigar | O que trazer |
|---|---|---|---|
| 1 | UI, landing pages, design visual de frontend | `coding/prompting_for_frontend_aesthetics.ipynb` | Técnicas de direção estética ao gerar UI (complementa a skill `frontend-design`) |
| 2 | Chatbot / feature conversacional com LLM | `tool_use/customer_service_agent.ipynb` · `patterns/agents/basic_workflows.ipynb` | Esqueleto do loop de conversa com tools; routing/chaining de fluxos |
| 3 | Memória e contexto de conversas longas (persistência, compactação) | `tool_use/context_engineering/` · `tool_use/memory_cookbook.ipynb` · `tool_use/automatic-context-compaction.ipynb` · `misc/session_memory_compaction.ipynb` | Comparativo de estratégias (memory tool vs. compaction vs. tool clearing), custo de cada uma, implementação de referência |
| 4 | Escrita/iteração de prompts de uma feature LLM | `misc/metaprompt.ipynb` · `misc/generate_test_cases.ipynb` · `managed_agents/CMA_prompt_versioning_and_rollback.ipynb` | Geração de prompt inicial; casos de teste sintéticos; versionamento de prompt com gate de eval |
| 5 | Saída estruturada (JSON, schemas) de LLM | `tool_use/extracting_structured_json.ipynb` · `tool_use/tool_use_with_pydantic.ipynb` · `misc/how_to_enable_json_mode.ipynb` | Técnicas de confiabilidade de output estruturado; validação com Pydantic |
| 6 | Definição de tools para um LLM (design, escolha, paralelismo) | `tool_use/tool_choice.ipynb` · `tool_use/parallel_tools.ipynb` · `tool_use/programmatic_tool_calling_ptc.ipynb` · `tool_use/tool_search_with_embeddings.ipynb` · `tool_evaluation/` | Padrões de tool_choice; PTC; tool search em catálogos grandes; como avaliar qualidade de tools |
| 7 | RAG, busca semântica, embeddings | `capabilities/retrieval_augmented_generation/` · `capabilities/contextual-embeddings/` · `third_party/{Pinecone,MongoDB,LlamaIndex}/` | Pipeline RAG com harness de avaliação incluso; contextual retrieval; integrações por vector store |
| 8 | Classificação, moderação de conteúdo, sumarização | `capabilities/classification/` · `misc/building_moderation_filter.ipynb` · `capabilities/summarization/` | Receitas com dataset + avaliação prontos por capacidade |
| 9 | Consulta em linguagem natural a banco (text-to-SQL) | `capabilities/text_to_sql/` · `misc/how_to_make_sql_queries.ipynb` | Guia com dados e avaliação; padrões de segurança de query |
| 10 | Processamento de imagens, PDFs, documentos, gráficos | `multimodal/` · `misc/pdf_upload_summarization.ipynb` | Boas práticas de visão; transcrição de formulários; leitura de charts; crop tool |
| 11 | Evals / medição de qualidade de feature LLM | `misc/building_evals.ipynb` · `misc/generate_test_cases.ipynb` · `tool_evaluation/` · `evals/agentic_search/` | Como montar eval harness; grading automatizado; reprodução de benchmarks (conecta com o gate de "Eval tests" em `think-ahead.md`) |
| 12 | Orquestração multi-agente / subagentes | `patterns/agents/` (orchestrator-workers, evaluator-optimizer, async) · `multimodal/using_sub_agents.ipynb` · `managed_agents/CMA_coordinate_specialist_team.ipynb` · `managed_agents/CMA_plan_big_execute_small.ipynb` | Padrões canônicos de orquestração com prompts reais; modelo grande planeja / pequeno executa |
| 13 | Construção de agente standalone ou harness próprio (fora do Claude Code) | `claude_agent_sdk/` (8 notebooks) · `evals/agentic_search/reproduce_agentic_search_benchmarks.ipynb` | Agentes completos sobre o Agent SDK; harness sobre a Messages API com PTC, compaction server-side e budgets |
| 14 | Custo e latência de chamadas LLM | `misc/prompt_caching.ipynb` · `misc/speculative_prompt_caching.ipynb` · `misc/batch_processing.ipynb` · `observability/usage_cost_api.ipynb` · `fable_5_fallback_billing/guide.ipynb` | Caching (incl. especulativo); batches; monitoramento de custo via Admin API; fallback de modelo |
| 15 | Autoria de skills (Claude Skills / sistema `ahead:*`) | `skills/notebooks/` · `.claude/` do próprio cookbook | Progressive disclosure; skills com scripts executáveis embutidos; exemplo real de setup `.claude/` da Anthropic |
| 16 | Agentes com memória de usuário / preferências persistentes | `managed_agents/CMA_remember_user_preferences.ipynb` · `tool_use/memory_cookbook.ipynb` | Memory store entre execuções; aprendizado de preferências |
| 17 | Verificação/auto-correção de trabalho de agente | `managed_agents/CMA_verify_with_outcome_grader.ipynb` · `patterns/agents/evaluator_optimizer.ipynb` | Outcome graders; loop avaliador-otimizador |
| 18 | Raciocínio pesado / extended thinking | `extended_thinking/` | Uso de thinking com e sem tools |
| 19 | Citações e grounding de respostas | `misc/using_citations.ipynb` | API de citations |
| 20 | Voz/áudio (STT/TTS) com LLM | `third_party/ElevenLabs/` · `third_party/Deepgram/` | Pipeline de voz de baixa latência; transcrição |

### Fora do mapa (deliberadamente)

- `finetuning/` — fluxo restrito, pouco aplicável.
- `misc/read_web_pages_with_haiku.ipynb`, `third_party/Wikipedia`, `WolframAlpha` — casos estreitos; se surgirem, tratáveis ad hoc sem gate.

## Exemplos de interação

**Exemplo A — o caso do enunciado (UI/landing page):**

> Usuário: "cria a landing page do produto X"
> Claude: invoca `frontend-design` (regra existente) e, no planejamento:
> *"Gatilho 1 do mapa do cookbook: posso investigar
> `coding/prompting_for_frontend_aesthetics.ipynb` e trazer técnicas de
> direção estética aplicáveis? (leitura única, via subagente)"*
> Usuário: "pode" → investiga → relato: *"3 aprendizados; o nº 2 mudaria a
> tipografia proposta de A para B porque..."* → usuário aceita 2 de 3 →
> aplica só os aceitos.

**Exemplo B — memória de chatbot:**

> Tarefa: implementar histórico persistente num chatbot do projeto.
> Gatilho 3 → gate 1 nomeando `context_engineering/` + `memory_cookbook` →
> investigação compara memory tool vs. compaction → relato via decision
> protocol com antes/depois da arquitetura proposta → usuário decide.

## Forma de implementação (a decidir)

O mecanismo tem duas metades com custos diferentes:

1. **Detecção de gatilho** — precisa estar disponível quando a tarefa começa.
   Opções:
   - **(a) Regra enxuta sempre carregada** em `~/.claude/rules/` — só a tabela
     de gatilhos (colunas 1–2, sem a coluna "o que trazer") + a instrução do
     gate 1. Custo de contexto permanente, porém pequeno se a tabela for
     comprimida.
   - **(b) Skill com description rica** (`ahead:cookbook`?) — custo zero de
     contexto, mas detecção depende do matching da description; gatilhos
     situacionais ("estou implementando memória") disparam pior que pedidos
     explícitos.
   - **(c) Híbrido** — regra mínima de 3–5 linhas ("ao tocar áreas X..Y,
     ofereça investigação via skill `ahead:cookbook`") + skill com o mapa
     completo e o procedimento. Provável recomendação: espelha o padrão já
     usado em "Specs Framework (Opt-In)".
2. **Procedimento de investigação** — mora na skill: mapa completo, protocolo
   dos dois gates, formato do relato, restrições.

## Questões em aberto

- [ ] Caminho do cookbook: hardcoded (`~/Desktop/Work/claude-cookbooks-main`) ou
  configurável? A pasta já mudou de lugar uma vez nesta própria discussão.
- [ ] Frescura: a cópia é um snapshot sem `.git`. Vale trocar por clone git
  (`git pull` ocasional) ou aceitar o congelamento? O mapa referencia caminhos
  que podem drifar com updates do upstream.
- [ ] Forma (a/b/c acima) — decidir custo de contexto vs. confiabilidade de
  disparo.
- [ ] Granularidade dos gatilhos: 20 linhas é muito? Agrupar (ex.: 2+3+16 em
  "conversação e memória") reduziria a tabela sempre-carregada na opção (a)/(c).
- [ ] Interação com skills existentes: gatilho 1 sobrepõe `frontend-design`,
  gatilho 15 sobrepõe `ahead:instructions`, gatilho 11 toca o gate de eval
  tests — a skill deve declarar precedência ou só complementar?
- [ ] O relato do gate 2 usa o decision protocol completo ou um formato mais
  leve quando há 1 item só?
- [ ] Notebooks `.ipynb` são JSON verboso — definir se o subagente lê via
  `jupyter nbconvert`/`jq` para extrair markdown+código, ou Read direto
  (caro em tokens).

## Próximo passo sugerido

Decidir a forma (a/b/c) e a granularidade da tabela; com isso, redigir a skill
real via `ahead:instructions` e, se for o caso, a regra mínima de gatilho.
