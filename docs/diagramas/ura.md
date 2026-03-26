```mermaid
flowchart TD
    A['Olá! Bem vindo ao SyncDesk! Para começarmos, verifiquei no seu cadastro e você possui os seguintes produtos disponíveis para manutenção. Selecione a opção que indica sobre o que você quer falar hoje:<br> 1-Produto A<br> 2-Produto B<br> 3-Produto C<br> 4-Desejo apenas tirar uma dúvida.<br> 5-Desejo uma liberação de acesso no Sync Desk.]
    A -->|1| B
    A -->|2| B
    A -->|3| B[Entendi. Como posso te ajudar hoje em relação ao Produto >produto escolhido< ? <br> 1-O sistema apresenta falhas.<br> 2-Quero solicitar uma nova função.]
    A -->|4| C[Entendi. Selecione, por favor, qual a sua dúvida: <br> 1-Qual o período restante para manutenção dos sistemas que eu já adquiri?<br> 2-Estou com dúvidas sobre como utilizar um dos meus sistemas.<br> 3-Como faço para solicitar um novo sistema?]
    A -->|5| D[Entendi. Por favor, envie uma mensagem respondendo as seguintes perguntas: 1-Essa liberação se refere à um novo perfil ou à edição de um perfil já existente? 2-Qual o email e empresa da pessoa que deve ser cadastrada? 3-Qual o motivo da solicitação? 4-Quais produtos essa pessoa deve ter vinculados à sua conta?]
    B -->|1| F[Por favor, explique da maneira mais detalhada possível o seu problema. Lembre-se: Se a descrição do problema não for clara e/ou faltarem informações, seu chamado poderá ser cancelado pelo time de suporte. Seja específico e detalhista.]
    B -->|2| G[Por favor, explique da maneira mais detalhada possível a nova funcionalidade que deseja. Lembre-se: Se a descrição da função não for clara e/ou faltarem informações, sua solicitação poderá ser cancelada pelo time de analistas. Seja específico e detalhista.]
    C -->|1| X[Verifiquei e esses são os seguintes prazos:<br> Produto A - Até dd/mm/aaaa<br> Produto B - Até dd/mm/aaa<br> Produto C - Até dd/mm/aaaa]
    C -->|2| J[Todos os nossos produtos possuem manual do usuário, onde você pode logar e acessa todas as informações neceesárias para a navegação. Verifique no sistema e tire todas as suas dúvidas por lá.]
    C -->|3| L[Você pode enviar um pedido através do nosso email xxxxx.com]
    D -->|Liberação solicitada| E[Aguarde, sua solicitação foi criada e será atribuída à um de nossos analistas. Você já pode acompanhar o tema pela tela 'Minhas demandas'. Obrigada!]
    L --> H
    F -->|Chamado criado| E
    G -->|Requisição solicitada| E
    X --> H[Ajudo em algo mais?<br> 1-Sim<br> 2-Não]
    J --> H
    H -->|Sim| A
    H -->|Não| I[Atendimento finalizado! Momento de avaliação do atendimento.]
````
