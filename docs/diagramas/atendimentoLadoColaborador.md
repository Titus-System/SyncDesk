```mermaid
flowchart TD
    A[Tela de Dashboard] -->B[Chamados não atribuídos]
    B -->C[Opção 'Atribuir a mim']
    B -->D[Opção 'Escalar o tema']
    D -->|Escalado?| E[Chamado é encaminhado à próxima fila de suporte e some da fila atual.]
    C -->|Atribuído?| F[Estão ativas as opções de conversa com o solicitante e edição de informações - Status e Comentários]
    F -->|Tratativa em andamento| G{Chamado resolvido ou cancelado?}
    G -->|Sim| H[Atendente muda o status para 'Encerrado']
    H --> J[Pop-Up - Você tem certeza que deseja encerrar/cancelar este chamado?]
    J -->|Sim| K[Chamado vai para a fila de 'Finalizados' e não tem mais a possibilidade de ser editado.]
    G -->|Não| I{Deseja atribuir este chamado a outro time?}
    I -->|Não| F
    I -->|Sim?| D
``````
