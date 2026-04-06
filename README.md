# SyncDesk

**Sistema Estruturado para Atendimento, Triagem e Escalonamento de Demandas**

> **Status do Projeto: Em Desenvolvimento**

## Desafio

Desenvolver uma aplicação mobile que centralize o atendimento ao cliente por meio de um sistema de chat estruturado que registre o
histórico completo das interações e permita o acompanhamento do fluxo de atendimento, incluindo abertura, escalonamento e encerramento de chamados. A jornada deve iniciar com uma triagem automatizada. O objetivo geral é estruturar e otimizar o processo de atendimento, promovendo eficiência
operacional, melhor experiência ao cliente e maior controle gerencial. 

## ⚡ Desenvolvimento Ágil

O projeto está sendo seguindo o método Ágil SCRUM, dividindo o trabalho em sprints de 21 dias, com reuniões diáras, revisões e retrospectivas ao final.

### 📋 Backlog do Produto

| Sprint | User Story | Status | Prioridade |
| :---  | :--- | :--- | :--- |
| **1** | Como atendente, quero poder trocar mensagens com clientes em uma tela de chat, para conseguir compreender suas necessidades e auxiliar rapidamente.  | CONCLUÍDA | ▲ Alta |
| **1** | Como solicitante, quero começar uma conversa rapidamente e ser guiado com algumas perguntas iniciais, para que meu problema seja identificado de forma ágil e descomplicada. | CONCLUÍDA | ▲ Alta |
| **1** | Como atendente, quero gerenciar o ciclo de vida dos chamados (atualizar status, encaminhar, atribuir e encerrar), para acompanhar e controlar as solicitações dos clientes. | CONCLUÍDA | ▲ Alta |
| **2** | Como gestor da aplicação, quero que o meu acesso possua ferramentas administrativas (como gestão de permissões, auditoria e relatórios), para ter controle sobre a operação do suporte. | PENDENTE | **=** Média |
| **2** | Como usuário do sistema, quero buscar conversas no histórico de atendimentos com base em critérios como nome do destinatário e data, para localizar atendimentos específicos de forma mais rápida. | PENDENTE | ▼ Baixa |
| **3** | Como solicitante do atendimento, quero receber notificações quando houver atualizações nos meus chamados, para acompanhar o andamento dos atendimentos sem precisar acessar constantemente o sistema. | PENDENTE | **=** Média |
| **3** | Como solicitante do atendimento, quero poder deixar uma avaliação após o encerramento, para registrar meu nível de satisfação com o suporte prestado. | PENDENTE | ▼ Baixa |

### 📅 Cronograma

| Sprint            | Prazo      | Status       | Documentação           | Entrega |
| ----------------- | ---------- | ------------ | ---------------------- | ------- |
| Kick Off          | 03/03/2026 | Concluído    | -                      | -       |
| Sprint 1          | 05/04/2026 | Concluído    | [sprint1](sprint_1.md) | [video](https://youtu.be/jFSbepQdjow)       |
| Sprint 2          | 03/05/2026 | Não iniciado | [sprint2](sprint_2.md) | Não entregue     |
| Sprint 3          | 31/05/2026 | Não iniciado     | [sprint3](sprint_3.md) | Não entregue       |
| Feira de Soluções | 11/06/2026 | Não iniciado     | [feira](feira_sol.md)  | -       |

### Roadmap

![roadmap](docs/roadmap.png)

### 👥 Fatec São José dos Campos - Prof. Jessen Vidal

| Cliente          | Período/Curso                                  | Professor M2      | Professora P2     | Contato Cliente                    |
| ---------------- | ---------------------------------------------- | ----------------- | ---------------- | ---------------------------------- |
| Larissa Souza e Rafael Monteiro - Empresa Pro4Tech | 5º Análise e Desenvolvimento de Sistemas | Ronaldo Emerick  | Gerson Penha | <creonice@tecsysbrasil.com.br> |

## Documentação Técnica

Arquitetura orientada a eventos e altamente desacoplada, mantendo foco em experiência do usuário, confiabilidade e integração transparente com IA. . Cada componente possui responsabilidades bem definidas e se comunica através de APIs REST, WebSockets e um broker de mensagens via Redis, conforme detalhado nos documentos de arquitetura.

## Arquitetura

![arquitetura](docs/arquitetura_sprint_2.svg)
Para detalhes da implementação: [Documento da Arquitetura](architecture.md)

Abaixo você encontra os links para a documentação específica de cada serviço.

-----

### 🔹 [Nexa API](https://github.com/Titus-System/Nexa-api)

O `Nexa-api` é o **orquestrador central** da aplicação. Construído em Python com Flask, ele atua como o API Gateway, gerenciando todas as requisições do cliente, a lógica de negócio principal e a comunicação assíncrona com os outros serviços.

**Principais Responsabilidades:**

- **Endpoints REST:** Expõe os endpoints para o frontend, incluindo `/upload-pdf` para o envio de documentos e `/classify-partnumber` para classificações individuais.
- **Orquestração Assíncrona:** Utiliza **Celery** e **Redis** para enfileirar tarefas pesadas (como o parsing de PDFs e as chamadas para a IA), mantendo a API sempre responsiva.
- **Comunicação em Tempo Real:** Gerencia a comunicação via **WebSocket (Socket.IO)** com o frontend para enviar atualizações de progresso em tempo real.
- **Persistência de Dados:** É o único serviço com responsabilidade de escrita no **banco de dados relacional (PostgreSQL)**, onde armazena os resultados finais das classificações.
- **Validação e Segurança:** Valida os dados de entrada (usando Pydantic) e lida com a lógica de autenticação e autorização de usuários.

**Tecnologias-chave:** `Python`, `Flask`, `Celery`, `Redis`, `Socket.IO`, `SQLAlchemy`, `Docker`.

-----

### 🧠 [Nexa AI Agents](https://github.com/Titus-System/Nexa-AI-Agents/)

O `Nexa-AI-Agents` é o **cérebro de IA** do sistema. Este serviço especializado, também em Python, é totalmente focado em executar as tarefas de inteligência artificial. Ele opera de forma independente, recebendo solicitações do `Nexa-api` e retornando resultados sem conhecer a lógica de negócio principal.

**Principais Responsabilidades:**

- **Processamento de IA:** Executa os modelos de linguagem para gerar descrições técnicas e classificar NCMs.
- **Retrieval-Augmented Generation (RAG):** Utiliza um **banco de dados vetorial (ChromaDB)** para buscar informações contextuais e semanticamente similares, aumentando a precisão e a qualidade das respostas geradas pela IA.
- **Publicação de Progresso:** Comunica-se de forma assíncrona com o `Nexa-api`, publicando atualizações de progresso em um canal **Redis (Pub/Sub)**.
- **Serviço Agnóstico:** Não possui estado e não se conecta diretamente a outros componentes, exceto o Redis e o ChromaDB, garantindo seu total desacoplamento.

**Tecnologias-chave:** `Python`, `Flask`, `Ollama`, `ChromaDB`, `Redis`, `smol-agents`, `Docker`.

-----

### 🖥️ [Nexa Frontend](https://github.com/Titus-System/Nexa-Frontend)

O `Nexa-frontend` é a **interface do cliente** da aplicação. Desenvolvida com React e TypeScript, esta Single-Page Application (SPA) foi projetada para oferecer uma experiência de usuário moderna, reativa e em tempo real.

**Principais Responsabilidades:**

- **Interação com o Usuário:** Fornece as telas para upload de documentos, entrada manual de Part Numbers e visualização de resultados.
- **Comunicação com a API:** Realiza chamadas para a `Nexa-api` via HTTP REST para iniciar os processos de classificação.
- **Atualizações em Tempo Real:** Estabelece uma conexão **WebSocket** com a API para receber e exibir o progresso das tarefas sem a necessidade de recarregar a página.
- **Gerenciamento de Estado:** Controla o estado da interface, garantindo que os dados exibidos sejam consistentes e atualizados.

**Tecnologias-chave:** `React`, `TypeScript`, `Vite`, `Socket.IO-client`, `CSS/Sass`, `Tailwind`.

## 🛠️ Tecnologias Utilizadas

<p align="center">
  <img alt="Python" height="30" width="40" src="https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg">
  <img alt="Flask" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/flask/flask-original.svg">
  <img alt="Pytest" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/pytest/pytest-original.svg" />
  <img alt="PostgreSQL" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/postgresql/postgresql-original.svg">
  <img alt="SQLAlchemy" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/sqlalchemy/sqlalchemy-original.svg" />
  <img alt="redis" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/redis/redis-original.svg" />
  <img alt="Smolagents" height="30" width="30" src="https://cdn-avatars.huggingface.co/v1/production/uploads/63d10d4e8eaa4831005e92b5/a3R8vs2eGE578q4LEpaHB.png">
  <img alt="Ollama" height="30" width="30" src="https://ollama.com/public/ollama.png">
  <img alt="ChromaDB" height="30" width="30" src="https://www.trychroma.com/img/favicon.ico">

</p>

<p align="center">
  <img alt="socketio" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/socketio/socketio-original.svg" />
  <img alt="Node" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/nodejs/nodejs-original.svg">
  <img alt="Typescript" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/typescript/typescript-original.svg" />
  <img alt="React" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/react/react-original-wordmark.svg" />
  <img alt="Vite" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/vitejs/vitejs-original.svg" />
  <img alt="HTML" height="30" width="40" src="https://raw.githubusercontent.com/devicons/devicon/master/icons/html5/html5-original.svg">
  <img alt="Tailwind" height="30" width="40"src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/tailwindcss/tailwindcss-original.svg">
</p>

<p align="center">
  <img alt="docker" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/docker/docker-original.svg" />
  <img alt="Git" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/git/git-original.svg">
  <img alt="Git" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/github/github-original.svg">
  <img alt="Figma" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/figma/figma-original.svg">
  <img alt="Jira" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/jira/jira-original.svg">
</p>

## Manual de Instalação e Execução

### 1. Pré-requisitos

- Python 3.11+ (para backend e IA)
- Servidor Ollama rodando o modelo `qwen2.5:14b`
- Node.js 18+ e npm (para frontend)
- Redis (pode ser local ou via Docker)
- Docker e Docker Compose (opcional, para facilitar a execução)

---

### 2. Clonando o Repositório

```bash
git clone https://github.com/Titus-System/Nexa.git
cd Nexa
```

---

### 3. Backend (Nexa-api)

```bash
cd Nexa-api
python -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
cp .env.example .env
# Edite o .env conforme necessário
python run.py
```

**Ou Usando Docker Compose (opcional)**

```bash
cd Nexa-api
docker compose up -d --build
```

O backend estará disponível em [http://localhost:5000](http://localhost:5000).

---

### 4. IA/Agentes (Nexa-AI-Agents)

```bash
cd ../Nexa-AI-Agents
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# Edite o .env conforme necessário

docker compose up --build -d
# Rodar o banco de dados vetorial

python database/create.py
# Inicia o banco de dados vetorial

python run.py
```

O serviço estará disponível em [http://localhost:5001](http://localhost:5000).

---

### 5. Frontend (Nexa-frontend)

```bash
cd ../Nexa-frontend
npm install
npm run dev
```

O frontend estará disponível em [http://localhost:5173](http://localhost:5173).

---

### 7. Observações

- Certifique-se de que o Redis está rodando (`localhost:6379` por padrão).
- Ajuste as variáveis de ambiente nos arquivos `.env` de cada módulo conforme necessário.
- Para produção, utilize Gunicorn no backend e configure variáveis de ambiente seguras.

## 🎓 Equipe <a id="equipe"></a>

<div>
  <table>
    <tr>
      <th>Membro</th>
      <th>Função</th>
      <th>Github</th>
      <th>Linkedin</th>
    </tr>
    <tr>
      <td>Julia Pereira</td>
      <td>Product Owner</td>
      <td>
        <a href="https://github.com/juliasoares17">
          <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white">
        </a>
      </td>
      <td>
        <a href="https://www.linkedin.com/in/julia-pereira-dev/">
            <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white">
        </a>
      </td>
    </tr>
    <tr>
      <td>Wesley Gonçalves</td>
      <td>Scrum Master</td>
      <td>
        <a href="https://github.com/WesleyGoncalves">
          <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white">
        </a>
      </td>
      <td>
        <a href="https://www.linkedin.com/in/wesley-d-goncalves/">
            <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white">
        </a>
      </td>
    </tr>
    <tr>
      <td>Pedro Garcia</td>
      <td>Dev Team</td>
      <td>
        <a href="https://github.com/pedro-fs-garcia">
          <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white">
        </a>
      </td>
      <td>
        <a href="https://www.linkedin.com/in/pedro-fs-garcia">
            <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white">
        </a>
      </td>
    </tr>
    <tr>
      <td>Eduardo Ribeiro</td>
      <td>Dev Team</td>
      <td>
        <a href="https://github.com/eduardo-Rib">
          <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white">
        </a>
      </td>
      <td>
        <a href="https://www.linkedin.com/in/eduardo-ribeiro-4b78002b2">
            <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white">
        </a>
      </td>
    </tr>
    <tr>
      <td>Maria Fernanda Diniz</td>
      <td>Dev Team</td>
      <td>
        <a href="https://github.com/Madhs31">
          <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white">
        </a>
      </td>
      <td>
        <a href="https://www.linkedin.com/in/mariafernandadiniz/">
            <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white">
        </a>
      </td>
    </tr>
    <tr>
      <td>Angelina Borroni</td>
      <td>Dev Team</td>
      <td>
        <a href="https://github.com/borroniff">
          <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white">
        </a>
      </td>
      <td>
        <a href="https://www.linkedin.com/in/angelinaferreiradev/">
            <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white">
        </a>
      </td>
    </tr>
    <tr>
      <td>Pablo Escobar</td>
      <td>Dev Team</td>
      <td>
        <a href="https://github.com/PabloEscobar1993">
          <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white">
        </a>
      </td>
      <td>
        <a href="https://www.linkedin.com/in/pablo-rafael-silva-9ab4771ba/">
            <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white">
        </a>
      </td>
    </tr>
    <tr>
      <td>Matheus Germano</td>
      <td>Dev Team</td>
      <td>
        <a href="https://github.com/m-germano">
          <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white">
        </a>
      </td>
      <td>
        <a href="https://www.linkedin.com/in/math-germano/">
            <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white">
        </a>
      </td>
    </tr>
  </table>
</div>
