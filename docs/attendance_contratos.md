# Contratos do Domínio Chatbot

> DOD da tarefa **US04-T03** — Consolidar contratos Attendance.

Base path: `/api/chatbot`

---

## Modelo de Domínio

### Attendance (`beanie.Document` — collection `attendances`)

```jsonc
{
  "_id": "ObjectId",                     // usado como triage_id
  "status": "opened | in_progress | finished",
  "start_date": "datetime (UTC)",
  "end_date": "datetime (UTC) | null",
  "client": {
    "id": "UUID",
    "name": "string",
    "email": "string",
    "company": {                         // opcional
      "id": "UUID",
      "name": "string"
    }
  },
  "triage": [
    {
      "step": "string (TriageState)",
      "question": "string",
      "answer_value": "string | null",
      "answer_text": "string | null"
    }
  ],
  "result": {                            // preenchido ao finalizar
    "type": "Ticket | Resolved",
    "closure_message": "string"
  },
  "evaluation": {                        // preenchido via avaliação
    "rating": "int (1–5)"
  }
}
```

Registro obrigatório em `init_beanie()` (`app/main.py` e `tests/conftest.py`).

---

## Schemas

### Entrada

| Schema                       | Responsabilidade                            |
|------------------------------|---------------------------------------------|
| `TriageInputDTO`             | Payload do webhook (triage_id + resposta)   |
| `CreateAttendanceDTO`        | DTO interno para criação de attendance      |
| `AttendanceSearchFiltersDTO` | Query params de filtragem (GET /)           |
| `EvaluationRequest`          | Payload de avaliação (rating: 1–5)          |

### Saída

| Schema              | Responsabilidade                                   |
|---------------------|----------------------------------------------------|
| `TriageData`        | Bloco `data` da resposta de triagem (usado com `GenericSuccessContent[TriageData]`) |
| `AttendanceResponse`| Consulta completa com campo computado `needs_evaluation` |
| `EvaluationResponse`| Confirmação da avaliação com `evaluated_at`        |

### `AttendanceSearchFiltersDTO`

Todos os campos são opcionais. Filtros combinados com AND.

| Campo             | Tipo                       | Comportamento                                      |
|-------------------|----------------------------|----------------------------------------------------|
| `client_id`       | `UUID`                     | Igualdade exata no `client.id`                     |
| `client_name`     | `string`                   | Busca parcial, case-insensitive, no `client.name`  |
| `status`          | `AttendanceStatus`         | Igualdade exata: `opened`, `in_progress`, `finished` |
| `result_type`     | `string`                   | Igualdade exata no `result.type`: `Ticket` ou `Resolved` |
| `start_date_from` | `datetime`                 | `start_date >= valor` (inclusive, UTC)              |
| `start_date_to`   | `datetime`                 | `start_date <= valor` (inclusive, UTC)              |
| `has_evaluation`  | `bool`                     | `true` = `evaluation != null`, `false` = `evaluation == null` |
| `rating`          | `int` (1–5)                | Igualdade exata no `evaluation.rating`             |

### `AttendanceResponse`

| Campo              | Tipo                           | Observação                              |
|--------------------|--------------------------------|-----------------------------------------|
| `triage_id`        | `string`                       | ObjectId do atendimento                 |
| `status`           | `AttendanceStatus`             |                                         |
| `start_date`       | `datetime`                     | UTC                                     |
| `end_date`         | `datetime \| null`             | UTC, preenchido ao finalizar            |
| `client`           | `AttendanceClient`             |                                         |
| `triage`           | `list[TriageStepSchema]`       | Histórico de perguntas/respostas        |
| `result`           | `AttendanceResult \| null`     | Preenchido ao finalizar                 |
| `evaluation`       | `AttendanceEvaluation \| null` | Preenchido via endpoint de avaliação    |
| `needs_evaluation` | `bool` (computado)             | `true` sse `status == finished` e `evaluation == null` |

### `EvaluationRequest`

| Campo    | Tipo         | Regra         |
|----------|--------------|---------------|
| `rating` | `int`        | `1 <= n <= 5` |

### `EvaluationResponse`

| Campo          | Tipo       |
|----------------|------------|
| `triage_id`    | `string`   |
| `rating`       | `int`      |
| `evaluated_at` | `datetime` |

---

## Endpoints

### POST `/` — Criar atendimento e iniciar triagem

```
Authorization: Bearer <token>
```

Cria um attendance com `status = opened`, identidade do cliente derivada do token,
e já executa a primeira transição da FSM — retornando a pergunta inicial (MAIN_MENU).
Não recebe request body.

Este é o **único ponto de criação** de um attendance. O webhook não cria attendances.

Resposta envelopada via `ResponseFactory` (`GenericSuccessContent[TriageData]`).

**Response `201`**

```jsonc
{
  "data": {
    "triage_id": "682d3f...",
    "step_id": "step_a",
    "message": "Olá! Bem vindo ao SyncDesk! Para começarmos, selecione a opção...",
    "input": {
      "mode": "quick_replies",
      "quick_replies": [
        { "label": "Produto A", "value": "1" },
        { "label": "Produto B", "value": "2" },
        { "label": "Produto C", "value": "3" },
        { "label": "Desejo apenas tirar uma dúvida.", "value": "4" },
        { "label": "Desejo uma liberação de acesso no Sync Desk.", "value": "5" }
      ]
    },
    "finished": null,
    "closure_message": null,
    "result": null
  },
  "meta": { "timestamp": "...", "success": true, "request_id": "..." }
}
```

| Erro  | Causa                     |
|-------|---------------------------|
| `401` | Token ausente ou inválido |

---

### GET `/` — Listar atendimentos

```
Authorization: Bearer <token>
```

Retorna atendimentos com filtros opcionais via query params (`AttendanceSearchFiltersDTO`).

**Exemplos de uso:**

```http
GET /api/chatbot/
GET /api/chatbot/?status=finished
GET /api/chatbot/?client_name=pedro
GET /api/chatbot/?status=finished&has_evaluation=false
GET /api/chatbot/?start_date_from=2026-01-01T00:00:00Z&start_date_to=2026-03-31T23:59:59Z
GET /api/chatbot/?result_type=Ticket&rating=5
GET /api/chatbot/?client_id=0f7d7c4f-7b5b-45cb-9d85-6f3c69f0b5d2&status=finished
```

**Response `200`**

```jsonc
{
  "data": [
    {
      "triage_id": "682d3f...",
      "status": "finished",
      "start_date": "2026-04-14T12:00:00Z",
      "end_date": "2026-04-14T12:10:00Z",
      "client": { "id": "uuid", "name": "Pedro", "email": "pedro@syncdesk.pro" },
      "triage": [ { "step": "A", "question": "...", "answer_value": "1", "answer_text": null } ],
      "result": { "type": "Ticket", "closure_message": "..." },
      "evaluation": null,
      "needs_evaluation": true
    }
  ],
  "meta": { "timestamp": "...", "success": true, "request_id": "..." }
}
```

| Erro  | Causa                     |
|-------|---------------------------|
| `401` | Token ausente ou inválido |

---

### POST `/webhook` — Interagir com a triagem

```
Authorization: Bearer <token>
```

Recebe a resposta do usuário a uma pergunta existente e retorna o próximo passo da FSM.
O attendance já deve existir (criado via `POST /`).
Toda chamada ao webhook é uma resposta — ao menos um dos campos de resposta deve estar preenchido.

Resposta envelopada via `ResponseFactory` (`GenericSuccessContent[TriageData]`).

**Request body (`TriageInputDTO`)**

| Campo          | Tipo          | Obrigatório | Regra                                  |
|----------------|---------------|-------------|----------------------------------------|
| `triage_id`    | `string`      | sim         | Deve referenciar um attendance existente |
| `answer_text`  | `string/null` | condicional | Mutuamente exclusivo com `answer_value`|
| `answer_value` | `string/null` | condicional | Mutuamente exclusivo com `answer_text` |

> Exatamente um dos campos `answer_text` ou `answer_value` deve estar preenchido.
> Ambos preenchidos ou ambos `null` resultam em `422`.

**Response `200` — em andamento**

```jsonc
{
  "data": {
    "triage_id": "682d3f...",
    "step_id": "step_b",
    "message": "Entendi. Como posso te ajudar hoje em relação ao Produto escolhido?",
    "input": {
      "mode": "quick_replies",
      "quick_replies": [
        { "label": "O sistema apresenta falhas.", "value": "1" },
        { "label": "Quero solicitar uma nova função.", "value": "2" }
      ]
    },
    "finished": null,
    "closure_message": null,
    "result": null
  },
  "meta": { "timestamp": "...", "success": true, "request_id": "..." }
}
```

**Response `200` — finalizada (com ticket)**

```jsonc
{
  "data": {
    "triage_id": "682d3f...",
    "finished": true,
    "closure_message": "Aguarde, sua solicitação foi criada...",
    "result": { "type": "Ticket", "id": "682d40..." }
  },
  "meta": { "..." }
}
```

**Response `200` — finalizada (sem ticket)**

```jsonc
{
  "data": {
    "triage_id": "682d3f...",
    "finished": true,
    "closure_message": "Atendimento finalizado! Momento de avaliação...",
    "result": null
  },
  "meta": { "..." }
}
```

| Erro  | Causa                                              |
|-------|----------------------------------------------------|
| `401` | Token ausente ou inválido                          |
| `403` | Permissão `chatbot:interact` ausente               |
| `404` | Attendance não encontrado para o `triage_id`       |
| `422` | `answer_text` e `answer_value` enviados juntos     |
| `422` | `answer_text` e `answer_value` ambos `null`        |

---

### GET `/{triage_id}` — Consultar atendimento

```
Authorization: Bearer <token>
```

**Response `200` (`AttendanceResponse`)**

```jsonc
{
  "data": {
    "triage_id": "682d3f...",
    "status": "finished",
    "start_date": "2026-04-14T12:00:00Z",
    "end_date": "2026-04-14T12:10:00Z",
    "client": {
      "id": "uuid",
      "name": "Pedro",
      "email": "pedro@syncdesk.pro",
      "company": { "id": "uuid", "name": "SyncDesk" }
    },
    "triage": [
      { "step": "A", "question": "Olá! ...", "answer_value": "1", "answer_text": null },
      { "step": "B", "question": "Como posso ajudar?", "answer_value": "1", "answer_text": null },
      { "step": "F", "question": "Descreva o problema...", "answer_value": null, "answer_text": "Erro ao gerar relatório" }
    ],
    "result": { "type": "Ticket", "closure_message": "Aguarde, sua solicitação foi criada..." },
    "evaluation": { "rating": 4 },
    "needs_evaluation": false
  },
  "meta": { "timestamp": "...", "success": true, "request_id": "..." }
}
```

| Erro  | Causa                      |
|-------|----------------------------|
| `401` | Token ausente ou inválido  |
| `404` | Atendimento não encontrado |

---

### POST `/{triage_id}/evaluation` — Avaliar atendimento

```
Authorization: Bearer <token>
```

**Request body (`EvaluationRequest`)**

```jsonc
{ "rating": 4 }
```

**Response `200` (`EvaluationResponse`)**

```jsonc
{
  "data": {
    "triage_id": "682d3f...",
    "rating": 4,
    "evaluated_at": "2026-04-14T12:15:00Z"
  },
  "meta": { "timestamp": "...", "success": true, "request_id": "..." }
}
```

| Erro  | Causa                                                   |
|-------|---------------------------------------------------------|
| `401` | Token ausente ou inválido                               |
| `404` | Atendimento não encontrado                              |
| `409` | Atendimento ainda não finalizado (`status != finished`) |
| `409` | Atendimento já avaliado (`evaluation != null`)          |
| `422` | `rating` fora do intervalo 1–5                          |

---

## Fluxo Completo de Integração

Exemplo de uma triagem que resulta em abertura de ticket e posterior avaliação.

```
┌─────────┐                                  ┌─────────┐
│ Frontend │                                  │ Backend │
└────┬────┘                                  └────┬────┘
     │                                            │
     │  1. POST /api/chatbot/                     │
     │  Authorization: Bearer <token>             │
     │ ──────────────────────────────────────────► │  Cria attendance + executa FSM(null)
     │                                            │  → MAIN_MENU
     │  201 { triage_id, step_id: "step_a",       │
     │        message: "Olá! Bem vindo...",       │
     │        input: { mode: "quick_replies",     │
     │          quick_replies: [...5 opções] } }  │
     │ ◄────────────────────────────────────────── │
     │                                            │
     │  2. POST /api/chatbot/webhook              │
     │  { triage_id, answer_value: "1" }          │  Produto A selecionado
     │ ──────────────────────────────────────────► │  FSM(A) → CHOOSING_PRODUCT_PROBLEM
     │                                            │
     │  200 { step_id: "step_b",                  │
     │        message: "Como posso ajudar?",      │
     │        input: { quick_replies: [           │
     │          "Falhas", "Nova função"] } }      │
     │ ◄────────────────────────────────────────── │
     │                                            │
     │  3. POST /api/chatbot/webhook              │
     │  { triage_id, answer_value: "1" }          │  "O sistema apresenta falhas"
     │ ──────────────────────────────────────────► │  FSM(B) → WAITING_FAILURE_TEXT
     │                                            │
     │  200 { step_id: "step_f",                  │
     │        message: "Explique o problema...",  │
     │        input: { mode: "free_text" } }      │
     │ ◄────────────────────────────────────────── │
     │                                            │
     │  4. POST /api/chatbot/webhook              │
     │  { triage_id,                              │
     │    answer_text: "Erro ao gerar relatório"} │  Texto livre
     │ ──────────────────────────────────────────► │  FSM(F) → TICKET_CREATED
     │                                            │  Cria ticket, finaliza attendance
     │  200 { finished: true,                     │
     │        closure_message: "Aguarde...",      │
     │        result: { type: "Ticket",           │
     │                  id: "682d40..." } }       │
     │ ◄────────────────────────────────────────── │
     │                                            │
     │  5. POST /api/chatbot/682d3f.../evaluation │
     │  { rating: 4 }                             │
     │ ──────────────────────────────────────────► │  Registra avaliação
     │                                            │
     │  200 { triage_id, rating: 4,               │
     │        evaluated_at: "2026-..." }          │
     │ ◄────────────────────────────────────────── │
     │                                            │
```

**Regras do fluxo:**
- `POST /` é sempre o primeiro passo — cria o attendance e retorna a primeira pergunta
- Cada chamada ao webhook é uma resposta a uma pergunta anterior (nunca vazia)
- `POST /` e `POST /webhook` retornam o mesmo formato (`GenericSuccessContent[TriageData]`) via `ResponseFactory`
- Após `finished: true`, o único endpoint válido é `POST /{triage_id}/evaluation`
- A avaliação só pode ser feita uma vez
