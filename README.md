# Guará Pizzaria — página do chatbot

Página estática que conversa com o **agente** do n8n
(`guara-pizzaria-agente-web.json`). Sem build, sem dependências.

```
render.yaml
public/
  index.html
```

## Antes de publicar: o fluxo do n8n

Importe `guara-pizzaria-agente-web.json` e deixe pronto:

1. **Credenciais** — Google Gemini(PaLM) API no nó do modelo, e Google Sheets
   OAuth2 nos nós *Consultar cardapio* e *Registrar pedido*.
2. **Planilha** — precisa estar convertida para Planilhas Google. Selecione o
   documento pela lista nos dois nós de Sheets.
3. **Ative o fluxo.** Sem isso a URL de produção responde 404.

> Este fluxo usa **Webhook**, não Chat Trigger. Os dois não são intercambiáveis:
> o Chat Trigger fala `chatInput` e responde `output`, enquanto esta página fala
> `mensagem` e lê `resposta`. Guarde a versão com Chat Trigger para testar
> dentro do editor do n8n.

## Publicar no Render

1. Suba estes arquivos para um repositório no GitHub ou GitLab.
2. No Render, **New → Blueprint** apontando para o repositório. Ele lê o
   `render.yaml` e cria um site estático no plano gratuito.
   (Alternativa: **New → Static Site**, *Build Command* vazio e
   `public` em *Publish Directory*.)
3. Você recebe uma URL como `https://guara-pizzaria.onrender.com`.

## Configurar o webhook

Abra o site, clique em **Conexão** e cole a URL de produção:

```
https://SEU-ESPACO.app.n8n.cloud/webhook/guara-pizzaria
```

Use `/webhook/`, não `/webhook-test/` — a de teste só responde enquanto você
mantém o *Execute Workflow* ativo no editor.

O endereço fica no `localStorage`, ou seja, por navegador. Para fixar de vez,
edite `public/index.html` e substitua o valor inicial de `webhookUrl`.

## Liberar o CORS

O nó *Webhook* já vem com **Allowed Origins** em `*`, então funciona de
imediato. Assim que a URL do Render existir, troque `*` pelo domínio:

```
https://guara-pizzaria.onrender.com
```

Com `*`, qualquer site pode chamar seu webhook e gastar sua cota do Gemini.

## Contrato entre a página e o fluxo

Envia:

```json
{ "sessionId": "web-abc123", "mensagem": "quero uma pizza" }
```

Recebe:

```json
{ "resposta": "texto exibido ao cliente", "pedido_completo": false }
```

- O `sessionId` é gerado por navegador e é o que separa a memória de cada
  cliente. O nó *Memoria da conversa* usa esse valor como chave da sessão.
- Trechos que começam com `CARDÁPIO` ou `RESUMO DO PEDIDO` aparecem em bloco
  monoespaçado, preservando o alinhamento.
- Quando a resposta contém `RESUMO DO PEDIDO`, o nó *Formatar resposta* marca
  `pedido_completo`, a página estampa o selo **Pedido confirmado** e começa uma
  sessão nova.
- O botão **Novo pedido** também gera uma sessão do zero.

## Aviso de pedido no Discord

Quando um pedido é fechado, o fluxo publica o resumo no canal da cozinha.
O aviso sai **depois** da resposta ao cliente, então não atrasa a conversa.

1. No Discord: **Editar canal → Integrações → Webhooks → Novo webhook**.
   Copie a URL.
2. No n8n, abra o nó *Avisar a cozinha* e crie a credencial **Discord Webhook**
   com essa URL.

O nó *Pedido fechado?* garante que o aviso só dispare no turno em que o pedido
foi realmente registrado — não a cada mensagem da conversa.

> O resumo inclui nome, endereço e telefone do cliente. Use um canal restrito à
> equipe, não um canal aberto do servidor.
