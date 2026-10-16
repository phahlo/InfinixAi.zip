# InfinixAi
# AI Gateway (.mcpb)

Gateway de comunicação entre IAs **embutido** na extensão: roda dentro do Claude Desktop, sem Omniroute, sem Docker e sem servidor externo. Zero dependências (só Node ≥ 18, que o Claude Desktop já traz).

## Instalar (Claude Desktop)

1. Dê dois cliques em `ai-gateway.mcpb` (ou *Settings → Extensions → Advanced → Install Extension*).
2. Preencha **ao menos uma** chave (ou a URL do Ollama / endpoint compatível). As chaves ficam no cofre do sistema.
3. Pergunte ao Claude: *"Use ai_providers"*.

## Ferramentas

| Ferramenta | O que faz |
|---|---|
| `ai_providers` | Provedores configurados, modelo padrão, estatísticas da sessão |
| `ai_models` | Lista modelos de um provedor (consulta a API dele) |
| `ai_chat` | Prompt para outra IA. `strategy`: `fallback` (padrão), `round_robin`, `race` |
| `ai_broadcast` | Mesmo prompt para até 8 modelos em paralelo |
| `ai_debate` | 2–6 modelos leem e revisam as respostas uns dos outros (1–3 rodadas) + juiz opcional |
| `bus_post` / `bus_read` / `bus_channels` / `bus_clear` | Barramento de mensagens por canais, salvo em disco |
| `bus_ask` | Pergunta a uma IA dentro de um canal, com o histórico como contexto |

Alvos: `"provider"` (modelo padrão) ou `"provider/model"`, ex.: `openai/gpt-4o-mini`, `anthropic`, `openrouter/meta-llama/llama-3.3-70b-instruct`.
Provedores: `anthropic`, `openai`, `gemini`, `openrouter`, `groq`, `deepseek`, `mistral`, `ollama`, `custom` (qualquer API compatível com OpenAI: LM Studio, vLLM, LiteLLM…).

Os modelos padrão são só sugestões e envelhecem: troque pelo campo **Modelos padrão (JSON)** ou passe o modelo no alvo. `ai_models` mostra o que cada provedor oferece hoje.

## Claude web (claude.ai)

O `.mcpb` só funciona no **Claude Desktop** (processo local via stdio). O claude.ai web só aceita conectores remotos (HTTPS). Para isso o mesmo arquivo tem um modo HTTP:

```bash
AIGW_HTTP_TOKEN=um-segredo-longo OPENAI_API_KEY=... node server/index.js --http --port 8787 --host 0.0.0.0
# endpoint: http://HOST:8787/mcp   (Authorization: Bearer <token>)  ou  /mcp/<token>
```

Para o web, hospede em algum lugar com **HTTPS público** e cadastre `https://seu-host/mcp/<token>` como conector personalizado. O token na URL é necessário porque o claude.ai não envia cabeçalho Bearer arbitrário. **Sem token, qualquer pessoa com a URL gasta suas chaves**: o servidor se recusa a escutar fora de localhost sem `AIGW_HTTP_TOKEN`. Este modo foi testado com cliente HTTP local, não contra o claude.ai.

## Segurança

- Chaves vão para o servidor só como variáveis de ambiente e são removidas de mensagens de erro.
- Respostas de outros modelos e mensagens do barramento são **dados não confiáveis** (podem conter prompt injection); as ferramentas avisam isso e o `ai_debate` as entrega aos modelos como dados.
- O barramento grava em `~/.ai-gateway/bus.json` (permissão 600), até 1000 mensagens por canal.
- Cada chamada gasta tokens da sua conta nos provedores. `ai_debate` custa `rodadas × modelos` chamadas (+ juiz).

## Avançado

Variáveis opcionais: `AIGW_ANTHROPIC_BASE`, `AIGW_GEMINI_BASE` (proxies/mocks), `AIGW_HTTP_HOST`, `AIGW_HTTP_PORT`, `AIGW_TIMEOUT_SECONDS`.
