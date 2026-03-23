```mermaid
flowchart TD
    A[Tela de Cadastro] --> B{Qual tipo de perfil?}
    B -->|Novo Solicitante| C[****Tela solicitantes****<br> - Nome<br> - Email<br> - Produto<br> - Empresa]
    B -->|Novo Atendente| D[****Tela atendentes****<br> - Nome<br> - Nível de suporte<br> - Email<br> - Tipo de acesso]
    C --> |Gerar senha aleatória| E[Usuário cadastro!]
    D --> |Gerar senha aleatória| E
    F[Obs: Um cadastro só pode ser realizado por um usuário administrador.] 
`````
