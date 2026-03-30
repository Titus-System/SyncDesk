# T04-US02 — Contrato de payload entre backend e frontend para triagem interativa

## Objetivo

Definir o payload JSON trocado entre backend e frontend no fluxo de triagem interativa.

Esta especificação existe para atender a necessidade de o backend informar ao frontend se a próxima resposta esperada do usuário deve ser:

- texto livre;
- clique em uma opção.

## Contexto de modelagem adotado

Com base no alinhamento do projeto:

- `live_chat` trata somente conversa em tempo real entre participantes;
- mensagens do chat com atendente não possuem interatividade;
- a interatividade pertence ao domínio de `triageSession`.

Por isso, este documento **não** descreve o payload do live chat.

## Escopo

Este contrato cobre:

- payload de saída do backend durante a triagem;
- metadados que orientam o frontend sobre o tipo de entrada esperado;
- estrutura de opções clicáveis;
- resposta do usuário para a etapa atual.

Este contrato não cobre:

- mensagens do `live_chat`;
- broadcast WebSocket de conversa com atendente;
- payload de arquivo do chat;
- regras de ticket.

---

## 1. Estrutura base da mensagem de triagem

### Server → Frontend

```json
{
  "data": {
    "triage_id": "67f1b8d2e1c24a0c9a8f9d12",
    "step_id": "step_03",
    "message": "Selecione o produto relacionado ao seu problema.",
    "input": {
      "mode": "quick_replies",
      "quick_replies": [
        {
          "label": "Portal do Cliente",
          "value": "portal_cliente"
        },
        {
          "label": "Aplicativo",
          "value": "mobile_app"
        },
        {
          "label": "Outro",
          "value": "other"
        }
      ]
    }
  },
  "meta": {
    "timestamp": "2026-03-29T13:00:00Z",
    "success": true,
    "request_id": "a1b6e4f0-36a2-47e7-bc18-3155c18e9931"
  }
}
```

### Campos

| Campo | Tipo | Obrigatório | Regra |
|---|---|---:|---|
| `triage_id` | `objectId` | sim | Identificador da sessão de triagem |
| `step_id` | `string` | sim | Identificador lógico da etapa atual |
| `message` | `string` | sim | Texto exibido ao usuário |
| `input.mode` | `free_text \| quick_replies` | sim | Tipo de resposta esperada |
| `input.quick_replies` | `array` | condicional | Obrigatório quando `mode = quick_replies` |

---

## 2. Modos de entrada

### 2.1 Texto livre

Usado quando o backend espera que o usuário digite uma resposta aberta.

#### Exemplo

```json
{
  "data": {
    "triage_id": "67f1b8d2e1c24a0c9a8f9d12",
    "step_id": "step_04",
    "message": "Descreva com mais detalhes o problema encontrado.",
    "input": {
      "mode": "free_text"
    }
  },
  "meta": {
    "timestamp": "2026-03-29T13:02:00Z",
    "success": true,
    "request_id": "6a0b8835-4f28-4773-a7ab-f68c306df102"
  }
}
```

### 2.2 Quick replies

Usado quando o backend apresenta opções fechadas para clique.

#### Exemplo

```json
{
  "data": {
    "triage_id": "67f1b8d2e1c24a0c9a8f9d12",
    "step_id": "step_02",
    "message": "Qual é o tipo da sua solicitação?",
    "input": {
      "mode": "quick_replies",
      "quick_replies": [
        {
          "label": "Incidente",
          "value": "incident"
        },
        {
          "label": "Dúvida",
          "value": "question"
        },
        {
          "label": "Solicitação",
          "value": "request"
        }
      ]
    }
  },
  "meta": {
    "timestamp": "2026-03-29T13:01:00Z",
    "success": true,
    "request_id": "ee2d5c48-0bb0-4d89-af8f-885ba4f4d103"
  }
}
```

### Estrutura de cada quick reply

| Campo | Tipo | Obrigatório | Regra |
|---|---|---:|---|
| `label` | `string` | sim | Texto visível para o usuário |
| `value` | `string` | sim | Valor técnico enviado de volta ao backend |

---

## 3. Payload de resposta do frontend

O frontend deve sempre enviar o identificador da triagem (`triage_id`) junto com a resposta da etapa. Esse campo é obrigatório para que o backend saiba a qual sessão de triagem a resposta pertence.

### 3.1 Resposta para etapa de texto livre

```json
{
  "triage_id": "67f1b8d2e1c24a0c9a8f9d12",
  "step_id": "step_04",
  "answer_text": "Não consigo concluir o login mesmo após redefinir a senha."
}
```

### 3.2 Resposta para etapa de quick replies

```json
{
  "triage_id": "67f1b8d2e1c24a0c9a8f9d12",
  "step_id": "step_02",
  "answer_value": "incident"
}
```

### Campos

| Campo | Tipo | Obrigatório | Regra |
|---|---|---:|---|
| `triage_id` | `objectId` | sim | Identificador da sessão de triagem que está sendo respondida |
| `step_id` | `string` | sim | Etapa que está sendo respondida |
| `answer_text` | `string` | condicional | Obrigatório quando o modo for `free_text` |
| `answer_value` | `string` | condicional | Obrigatório quando o modo for `quick_replies` |

### Regras mínimas

- `triage_id` é sempre obrigatório;
- `step_id` é sempre obrigatório;
- `answer_text` e `answer_value` não devem ser enviados juntos para a mesma etapa;
- o backend valida se o `triage_id` existe e corresponde a uma triagem ativa;
- o backend valida se a etapa informada em `step_id` pertence à triagem informada em `triage_id`;
- o backend valida se o tipo de resposta está compatível com o modo esperado na etapa atual.

---

## 4. Resposta do backend após processar a etapa

Depois de processar uma resposta, o backend pode:

- devolver a próxima etapa da triagem;
- informar conclusão da triagem;
- indicar geração de ticket.

### Exemplo — próxima etapa

```json
{
  "data": {
    "triage_id": "67f1b8d2e1c24a0c9a8f9d12",
    "step_id": "step_05",
    "message": "Informe o e-mail usado no acesso.",
    "input": {
      "mode": "free_text"
    }
  },
  "meta": {
    "timestamp": "2026-03-29T13:03:00Z",
    "success": true,
    "request_id": "f4c1b06a-f50d-4a9d-8d96-0ef5414ed104"
  }
}
```

### Exemplo — triagem concluída com geração de ticket

```json
{
  "data": {
    "triage_id": "67f1b8d2e1c24a0c9a8f9d12",
    "finished": true,
    "closure_message": "Triagem concluída. Seu ticket foi criado com sucesso.",
    "result": {
      "type": "Ticket",
      "id": "67f30a0ce1c24a0c9a8f9e10"
    }
  },
  "meta": {
    "timestamp": "2026-03-29T13:05:00Z",
    "success": true,
    "request_id": "25c9a80f-59d1-41a8-b90d-a04ce6e65105"
  }
}
```

---

## 5. Payload de erro

### Exemplo

```json
{
  "type": "about:blank",
  "title": "Validation Error",
  "status": 400,
  "detail": "Invalid answer for current step.",
  "errors": {
    "answer_value": ["Value is not allowed for this step"]
  },
  "meta": {
    "timestamp": "2026-03-29T13:06:00Z",
    "success": false,
    "request_id": "6b062635-c1a7-4ca7-9654-c5cb9f1a1106"
  }
}
```
