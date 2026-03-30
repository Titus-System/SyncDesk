# US3 — Contrato REST para criação de ticket e alteração de status

## Objetivo

Definir o contrato REST para:

- criação de ticket;
- alteração de status de ticket.

## Contexto de modelagem adotado

Com base nas modelagens mais recentes do projeto:

- `ticket` é a entidade principal do chamado;
- `triageSession` é a sessão de triagem que pode gerar um ticket;
- `conversation` é o chat em tempo real associado ao atendimento;
- `service_session` não deve mais ser tratado como o próprio chamado dentro deste contrato.

Em outras palavras:

- triagem e chat não são o mesmo domínio;
- ticket e conversation também não são o mesmo recurso.

## Escopo

Esta especificação cobre apenas o contrato HTTP entre backend e frontend para:

- criar ticket;
- alterar status de ticket.

Não cobre:

- fluxo da triagem;
- mensagens do live chat;
- regras de distribuição de atendimento;
- persistência interna;
- side effects de negócio.

## Convenções

### Content-Type

`application/json`

### Envelope de resposta

Seguir o padrão já utilizado pelo backend: resposta de sucesso com `data` e `meta`; resposta de erro com estrutura padronizada de erro e `meta`.

### Exemplo de sucesso

```json
{
  "data": {},
  "meta": {
    "timestamp": "2026-03-29T12:00:00Z",
    "success": true,
    "request_id": "uuid-opcional"
  }
}
```

### Exemplo de erro

```json
{
  "type": "about:blank",
  "title": "Validation Error",
  "status": 400,
  "detail": "Invalid request body.",
  "errors": null,
  "meta": {
    "timestamp": "2026-03-29T12:00:00Z",
    "success": false,
    "request_id": "uuid-opcional"
  }
}
```

---

## 1. Criar ticket

### Endpoint

`POST /api/tickets`

### Objetivo

Criar um novo ticket a partir do contexto do atendimento/triagem.

### Request body

```json
{
  "triage_id": "67f1b8d2e1c24a0c9a8f9d12",
  "type": "incident",
  "criticality": "medium",
  "product": "portal_cliente",
  "description": "Cliente informa falha ao concluir login."
}
```

### Campos

| Campo | Tipo | Obrigatório | Regra |
|---|---|---:|---|
| `triage_id` | `objectId` | sim | Identificador da triagem que originou o ticket |
| `type` | `string` | sim | Tipo funcional do ticket |
| `criticality` | `string` | sim | Nível de criticidade |
| `product` | `string` | sim | Produto ou sistema relacionado |
| `description` | `string` | sim | Descrição inicial do ticket |

### Regras mínimas

- `triage_id` deve existir;
- `description` não pode ser vazia;
- o backend define o `status` inicial;
- o backend pode preencher automaticamente dados derivados da triagem, como cliente, histórico inicial e vínculo com chat, se aplicável.

### Status inicial esperado

```text
open
```

### Response — 201 Created

```json
{
  "data": {
    "id": "67f30a0ce1c24a0c9a8f9e10",
    "triage_id": "67f1b8d2e1c24a0c9a8f9d12",
    "type": "incident",
    "criticality": "medium",
    "product": "portal_cliente",
    "status": "open",
    "description": "Cliente informa falha ao concluir login.",
    "chat_id": null,
    "created_at": "2026-03-29T12:00:00Z",
    "updated_at": "2026-03-29T12:00:00Z"
  },
  "meta": {
    "timestamp": "2026-03-29T12:00:00Z",
    "success": true,
    "request_id": "8e4f6e9a-2f0c-4e28-a72d-8c8071a09e10"
  }
}
```

### Erros esperados

#### 400 Bad Request

Payload inválido.

```json
{
  "type": "about:blank",
  "title": "Validation Error",
  "status": 400,
  "detail": "Invalid request body.",
  "errors": {
    "description": ["Field required"]
  },
  "meta": {
    "timestamp": "2026-03-29T12:00:00Z",
    "success": false,
    "request_id": "uuid-opcional"
  }
}
```

#### 404 Not Found

Triagem inexistente.

```json
{
  "type": "about:blank",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Triage session not found.",
  "errors": null,
  "meta": {
    "timestamp": "2026-03-29T12:00:00Z",
    "success": false,
    "request_id": "uuid-opcional"
  }
}
```

#### 409 Conflict

Tentativa de criar ticket duplicado quando a regra de negócio impedir duplicidade.

```json
{
  "type": "about:blank",
  "title": "Conflict",
  "status": 409,
  "detail": "Ticket already exists for this triage session.",
  "errors": null,
  "meta": {
    "timestamp": "2026-03-29T12:00:00Z",
    "success": false,
    "request_id": "uuid-opcional"
  }
}
```

---

## 2. Alterar status do ticket

### Endpoint

`PATCH /api/tickets/{ticket_id}/status`

### Objetivo

Alterar o status de um ticket existente.

### Request body

```json
{
  "status": "finished",
  "reason": "Solicitação concluída com sucesso."
}
```

### Campos

| Campo | Tipo | Obrigatório | Regra |
|---|---|---:|---|
| `status` | `string` | sim | Novo status do ticket |
| `reason` | `string` | não | Motivo da mudança |

### Status sugeridos

Os nomes finais devem seguir o enum oficial do domínio de ticket. Considerando a modelagem enviada, o contrato deve trabalhar com status em inglês, por exemplo:

- `open`
- `in_progress`
- `waiting_customer`
- `finished`
- `cancelled`

### Regras mínimas

- o ticket deve existir;
- a transição de status deve ser válida;
- ticket `finished` ou `cancelled` não deve voltar para estado aberto sem regra explícita;
- ao mudar para `finished`, o backend pode registrar data de encerramento.

### Response — 200 OK

```json
{
  "data": {
    "id": "67f30a0ce1c24a0c9a8f9e10",
    "status": "finished",
    "reason": "Solicitação concluída com sucesso.",
    "updated_at": "2026-03-29T12:10:00Z",
    "finished_at": "2026-03-29T12:10:00Z"
  },
  "meta": {
    "timestamp": "2026-03-29T12:10:00Z",
    "success": true,
    "request_id": "a8a6f13e-6e84-45d0-bdbc-287b8ef2d110"
  }
}
```

### Erros esperados

#### 400 Bad Request

Status inválido ou transição inválida.

```json
{
  "type": "about:blank",
  "title": "Invalid Status Transition",
  "status": 400,
  "detail": "Cannot change status from finished to open.",
  "errors": null,
  "meta": {
    "timestamp": "2026-03-29T12:10:00Z",
    "success": false,
    "request_id": "uuid-opcional"
  }
}
```

#### 404 Not Found

Ticket inexistente.

```json
{
  "type": "about:blank",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Ticket not found.",
  "errors": null,
  "meta": {
    "timestamp": "2026-03-29T12:10:00Z",
    "success": false,
    "request_id": "uuid-opcional"
  }
}
```

#### 409 Conflict

Ticket já finalizado e sem permitir nova alteração.

```json
{
  "type": "about:blank",
  "title": "Conflict",
  "status": 409,
  "detail": "Ticket is already closed.",
  "errors": null,
  "meta": {
    "timestamp": "2026-03-29T12:10:00Z",
    "success": false,
    "request_id": "uuid-opcional"
  }
}
```

---

## 3. DTOs sugeridos

### CreateTicketDTO

```json
{
  "triage_id": "objectId",
  "type": "string",
  "criticality": "string",
  "product": "string",
  "description": "string"
}
```

### UpdateTicketStatusDTO

```json
{
  "status": "string",
  "reason": "string"
}
```

