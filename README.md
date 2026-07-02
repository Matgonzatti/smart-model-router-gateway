# Smart Model Router Gateway

Gateway HTTP simples para rotear perguntas para modelos LLM via OpenRouter, com foco em uma interface unica (`POST /chat`) e estrategia de roteamento configuravel.

## Visao Geral

Este projeto expoe uma API Fastify com um endpoint de chat:

- Recebe uma pergunta em JSON
- Encaminha para OpenRouter usando o SDK oficial
- Retorna o modelo escolhido e o texto gerado

Hoje, a configuracao padrao usa o modelo `openai/gpt-oss-20b:free` e ordenacao de provedor por preco.

## Stack

- Node.js (com suporte nativo a TypeScript no runtime)
- Fastify
- OpenRouter SDK
- Node Test Runner (`node:test`)

## Estrutura

```text
smart-model-router-gateway/
	src/
		config.ts              # Configuracoes de modelo/provedor
		openrouterService.ts   # Integracao com OpenRouter
		server.ts              # Definicao do endpoint /chat
		index.ts               # Bootstrap da aplicacao
	tests/
		router.e2e.test.ts     # Testes de integracao via app.inject
```

## Pre-requisitos

- Node.js 22+
- Chave de API do OpenRouter

## Instalacao

```bash
npm install
```

## Configuracao de Ambiente

Crie um arquivo `.env` na raiz:

```env
OPENROUTER_API_KEY=seu_token_aqui
```

Sem essa variavel, a aplicacao e os testes falham com `console.assert`.

## Executando em Desenvolvimento

```bash
npm run dev
```

Servidor sobe em:

```text
http://0.0.0.0:3000
```

## Endpoint

### POST /chat

Body JSON:

```json
{
  "question": "Qual a capital da Franca?"
}
```

Regras de validacao:

- `question` e obrigatoria
- `question` deve ter no minimo 5 caracteres

Resposta de sucesso:

```json
{
  "model": "openai/gpt-oss-20b:free",
  "content": "Paris e a capital da Franca."
}
```

Resposta de erro interno:

```json
{
  "error": "Internal Server Error"
}
```

## Exemplo com cURL

```bash
curl -X POST http://localhost:3000/chat \
	-H "Content-Type: application/json" \
	-d '{"question":"Explique o que e roteamento de modelos em uma frase."}'
```

## Scripts

```bash
npm run dev       # inicia API em watch mode com inspector
npm run test      # executa testes
npm run test:dev  # executa testes em watch mode com inspector
```

## Testes

Os testes em `tests/router.e2e.test.ts` exercitam o endpoint `/chat` com duas estrategias de ordenacao do provider:

- `price`
- `throughput`

Em ambos os casos, validam status `200` e o modelo retornado.

Observacao: como o fluxo chama OpenRouter, e necessario ter `OPENROUTER_API_KEY` valida para os testes passarem.

## Configuracoes Atuais

Valores padrao definidos em `src/config.ts`:

- `models`: `openai/gpt-oss-20b:free`
- `temperature`: `0.2`
- `maxTokens`: `50`
- `systemPrompt`: `You are a helpful assistant`
- `provider.sort.by`: `price`
- `provider.sort.partition`: `none`
