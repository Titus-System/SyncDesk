```json
{
  "_id": "651",
  "status": "finished",
  "start_date": "2024-03-26T10:00:00Z",
  "end_date": "2024-03-26T10:05:00Z",

  "customer": {
    "id": "99",
    "name": "João Silva",
    "company": {
      "name": "Tech Solutions",
      "id": "34"
    },
    "email": "joao@tech.com"
  },
  
  "triage": [
    {
      "step": "A",
      "question": "Select the product or query",
      "answer_value": "1", 
      "answer_text": "Product A"
    },
    {
      "step": "B",
      "question": "How can I help regarding Product A?",
      "answer_value": "1",
      "answer_text": "The system is showing failures."
    },
    {
      "step": "F",
      "question": "Explain the problem in detail",
      "answer_text": "The system freezes when generating the monthly sales report on the desktop version.",
      "type": "free_text"
    }
  ],
  
  "result": {
    "type": "Ticket",
    "closure_message": "Please wait, your request has been created..."
  },
  
  "evaluation": {
    "rating": 5
  }
}
````
