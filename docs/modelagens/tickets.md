```json
{
  "_id": "706",
  "triage_id": "651",
  "type": "issue", //issue, access, new_feature
  "criticality": "high", //issue - always high, access - always medium, new feature - always LOW
  "product": "Product a",
  "status": "finished", //Open, In progress, Waiting for provider, Waiting for validation, Finished
  "creation_date": "2024-03-26T10:00:00Z",
  "description": "free text provided by the requester",
  "chat_id": "052",

  "agent_history": [
    {
      "agent_id": "88",
      "name": "Ricardo Souza",
      "level": "N1",
      "assignment_date": "2024-03-26T10:05:00Z",
      "exit_date": "2024-03-26T11:00:00Z",
      "transfer_reason": "Escalated to N2"
    },
    {
      "agent_id": "99",
      "name": "João Silva",
      "level": "N2",
      "assignment_date": "2024-03-26T11:00:00Z",
      "exit_date": null, 
      "transfer_reason": null
    }
  ],

  "client": {
    "id": "99",
    "name": "João Silva",
    "company": {
      "name": "Tech Solutions",
      "id": "34"
    },
    "email": "joão@gmail"
  },

  "comments": [
    {
      "comment_id": "c1",
      "author": "Technical Support",
      "text": "We are analyzing the server log.",
      "date": "2024-03-26T10:10:00Z",
      "internal": true 
    },
    {
      "comment_id": "c2",
      "author": "João Silva",
      "text": "Thank you, I'll wait.",
      "date": "2024-03-26T10:15:00Z",
      "internal": false
    }
  ]
}
```
