# Pizzaria do Jacson — página do chatbot

Página estática que conversa com o fluxo do n8n. Sem build, sem dependências.

```
render.yaml
public/
  index.html
```

## Publicar no Render

1. Suba estes arquivos para um repositório no GitHub ou GitLab.
2. No Render, use **New → Blueprint** e aponte para o repositório. Ele lê o
   `render.yaml` e cria um site estático no plano gratuito.
   (Alternativa sem blueprint: **New → Static Site**, deixe o *Build Command*
   vazio e informe `public` em *Publish Directory*.)
3. Ao final você recebe uma URL como `https://pizzaria-do-jacson.onrender.com`.

## Configurar o webhook

O valor padrão já vem configurado em `public/index.html`:

```
https://jacsonbordin.app.n8n.cloud/webhook/pizzaria-jacson
```

O endereço fica salvo no `localStorage` do navegador assim que a página abre.
Quem quiser usar outro webhook pode clicar em **Conexão** e colar uma URL
diferente.

> Use `/webhook/` (produção), não `/webhook-test/`. A URL de teste só responde
> enquanto você mantém o botão *Execute Workflow* ativo no editor.

## Liberar o CORS no n8n (obrigatório)

O navegador bloqueia a chamada se o n8n não autorizar a origem do site.
No nó **Webhook** do fluxo, abra *Options* → **Allowed Origins (CORS)** e
informe o domínio do Render:

```
https://pizzaria-do-jacson.onrender.com
```

Para testar rapidamente, `*` também funciona, mas libera qualquer site a chamar
seu webhook. Prefira o domínio específico assim que a URL estiver definida.

## O que a página espera de volta

O nó *Responder* devolve este formato, já produzido pelo fluxo atual:

```json
{
  "resposta": "texto exibido ao cliente",
  "mostrar_cardapio": false,
  "pedido_completo": false,
  "pedido": null,
  "total": 0
}
```

- `resposta` é renderizada como ficha de pedido. Trechos que começam com
  `CARDÁPIO` ou `RESUMO DO PEDIDO` aparecem em bloco monoespaçado, preservando
  o alinhamento.
- `pedido_completo: true` estampa o selo **Pedido confirmado** e inicia uma
  sessão nova, espelhando a limpeza de memória que o fluxo faz do lado do n8n.

## Sessões

Cada navegador recebe um `sessionId` próprio, gravado localmente e enviado em
toda mensagem — é ele que separa a memória de cada cliente no n8n. O botão
**Novo pedido** gera um identificador novo e começa uma conversa do zero.
