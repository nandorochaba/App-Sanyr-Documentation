# Sanyr - Arquitetura e Documentação Técnica (Multi-tenant SaaS)

O Sanyr é uma plataforma Software as a Service (SaaS) corporativa, de arquitetura multi-tenant, projetada para otimizar a gestão e o agendamento em estabelecimentos de serviços (salões de beleza, clínicas, barbearias). O sistema atua como um núcleo operacional, orquestrando agendas, profissionais e métricas financeiras.

Este repositório tem como objetivo servir como **documentação técnica e portfólio**, evidenciando as decisões arquiteturais, os padrões de projeto (Design Patterns) e as boas práticas de engenharia de software aplicadas durante todo o ciclo de desenvolvimento.

---

## 1. Visão Geral da Arquitetura

A aplicação foi concebida sob uma arquitetura desacoplada em três camadas (Three-tier Architecture):
- **Camada de Apresentação (Frontend)**: Single Page Application (SPA) desenvolvida com React e Vite.
- **Camada de Aplicação (Backend)**: API RESTful construída com Node.js e Express.
- **Camada de Dados**: Banco de dados não-relacional (NoSQL) gerenciado via MongoDB Atlas.

### 1.1. Estratégia de Interface Dupla (Dual Theme)
O frontend é lógica e visualmente segmentado para atender a dois perfis de usuários com necessidades distintas:
- **Interface do Cliente Final (White-label)**: Uma interface *Mobile-First* que recebe injeção dinâmica de propriedades de estilo (cores, tipografia e logotipos) com base nas configurações do *tenant* (lojista). O objetivo é garantir uma imersão contínua, onde o cliente não percebe a transição para um sistema de terceiros.
- **Painel Administrativo (Gestão)**: Uma interface densa em dados, projetada para operadores e gestores. Adota um *Dark Theme* customizado e de alto contraste. Esta decisão de UI/UX visa reduzir a fadiga visual (Eye Strain) durante jornadas prolongadas de uso e facilitar a leitura de dashboards analíticos.

---

## 2. Stack Tecnológico e Decisões de Engenharia

A escolha das tecnologias priorizou a experiência de desenvolvimento (DX), a performance da aplicação e a manutenibilidade a longo prazo.

### 2.1. Ecossistema Frontend
- **React 18 & Vite**: Optou-se por React em conjunto com Vite em detrimento de frameworks Server-Side Rendering (SSR) como Next.js. O sistema exige uma arquitetura estrita de SPA, onde a velocidade de renderização no lado do cliente (Client-side Rendering) e transições fluidas no painel administrativo são prioritárias. O Vite substitui bundlers tradicionais (como Webpack/Create React App), reduzindo drasticamente o tempo de *cold start* e otimizando o *Hot Module Replacement* (HMR) através de módulos ES nativos.
- **TypeScript**: Adotado rigorosamente em toda a base de código para garantir tipagem estática (Type Safety). A detecção de erros em tempo de compilação, interfaces auto-documentadas e a segurança em refatorações complexas justificam a curva de aprendizado e o overhead inicial.
- **Tailwind CSS v4**: Utilizado para a estilização baseada em utilitários (*Utility-first*). A escolha do Tailwind frente a bibliotecas de CSS-in-JS (ex: Styled Components) baseia-se na performance (ausência de processamento CSS em tempo de execução) e na facilidade de implementação do conceito White-label. Os temas dos *tenants* são aplicados injetando variáveis CSS na raiz do DOM, que o Tailwind mapeia dinamicamente.

### 2.2. Ecossistema Backend
- **Node.js & Express**: Fornece um framework leve e não-opinativo para a API REST. Embora frameworks opinativos como NestJS ofereçam padrões arquiteturais rígidos (úteis em times massivos), o Express permite uma orquestração altamente customizada de middlewares de roteamento, essencial para lidar com os fluxos específicos de autenticação e recepção de webhooks deste projeto.
- **MongoDB & Mongoose (ODM)**: A abordagem NoSQL foi selecionada devido à variabilidade das estruturas de dados exigidas por diferentes *tenants* (ex: atributos customizados de serviços, regras flexíveis de expediente). O Mongoose atua como camada de validação e modelagem na aplicação, garantindo integridade sem comprometer a flexibilidade do schema.

### 2.3. Integrações e Infraestrutura
- **Evolution API (WhatsApp)**: O serviço de mensageria assíncrona (lembretes automáticos) foi integrado via Evolution API. Esta decisão técnica prioriza a flexibilidade de gerenciamento de sessões multi-device e a viabilidade econômica, em contraste com a API Oficial da Meta (Cloud API), que impõe custos por conversa e dependência de provedores (BSPs).
- **Cloudflare Tunnels**: Ferramenta essencial na infraestrutura de desenvolvimento e produção. Permite expor endpoints locais de forma segura à internet, mecanismo obrigatório para o recebimento de webhooks (confirmações de leitura, recebimento de mensagens) advindos da Evolution API em tempo real.

---

## 3. Implementações Técnicas Essenciais

### 3.1. Autenticação e Segurança (Auth)
O sistema implementa uma estratégia de segurança híbrida:
- **Google OAuth 2.0 (Federated Identity)**: Implementado no cliente através da biblioteca `@react-oauth/google`. O frontend obtém um *ID Token* que é interceptado pelo backend. O servidor valida criptograficamente a assinatura deste token utilizando a biblioteca oficial `google-auth-library` antes de emitir um JWT (*JSON Web Token*) interno. Este fluxo previne vetores de ataque de personificação (Spoofing).
- **Gerenciamento Criptográfico de Credenciais**: Para autenticação padrão, as senhas recebem hash via `bcrypt`. Fluxos de recuperação de senha utilizam tokens randômicos gerados pelo módulo nativo `crypto` do Node, armazenados no MongoDB com índices de expiração rigorosos (Time-To-Live - TTL).

### 3.2. Motor de Agendamento e Prevenção de Conflitos
A lógica de negócio central (Core Domain) reside no motor de alocação de horários. Para mitigar *race conditions* (condições de corrida) e *double-bookings* (duplo agendamento):
- O backend calcula matrizes de disponibilidade em tempo real, cruzando regras de expediente, pausas (ex: horário de almoço) e agendamentos consolidados.
- Transações críticas no banco de dados utilizam operações atômicas (quando aplicável) para assegurar a consistência dos dados sob alta concorrência.

### 3.3. Gerenciamento de Estado (State Management)
A API de Contexto do React (Context API) é utilizada para o gerenciamento de estados globais estáticos ou de baixa volatilidade (ex: Sessão do Usuário, Configurações do Tenant). Bibliotecas complexas como Redux ou Zustand foram omitidas intencionalmente; a aplicação delega a complexidade ao *Server State*, gerenciado através de hooks customizados e instâncias abstraídas de requisições HTTP, mantendo o *bundle* do cliente otimizado.

---

## 4. Estrutura do Repositório (Monorepo)

O projeto é estruturado separando os domínios de forma lógica e escalar:

```text
Beauty Service Booking App/
├── frontend/                 # Client-side SPA (React + Vite)
│   ├── src/
│   │   ├── components/       # Primitivas de UI reutilizáveis e stateless
│   │   ├── contexts/         # Provedores de estado global (Auth, Tenant)
│   │   ├── layouts/          # Componentes estruturais (Admin Layout, Client Layout)
│   │   ├── pages/            # View components (Páginas do roteador)
│   │   ├── services/         # Camada de comunicação (Axios interceptors, API Client)
│   │   └── utils/            # Funções puras e formatadores (Helpers)
│   └── vite.config.ts        # Configurações do bundler
│
├── backend/                  # Server-side REST API (Node.js)
│   ├── src/
│   │   ├── controllers/      # Handlers de requisição e formatação de resposta HTTP
│   │   ├── middlewares/      # Interceptadores (Validação de JWT, Tratamento de Erros)
│   │   ├── models/           # Schemas do Mongoose (Data Access Objects)
│   │   ├── routes/           # Mapeamento de endpoints da API
│   │   ├── services/         # Regras de negócio (Integração WhatsApp, Lógica de Agenda)
│   │   └── server.ts         # Ponto de entrada (Entrypoint) da aplicação
│
└── evolution-api/            # Instância do microserviço de mensageria
```

---

## 5. Setup do Ambiente de Desenvolvimento

Instruções para provisionamento e execução da stack em ambiente local:

### Pré-requisitos
- Node.js (v18+)
- Instância do MongoDB (Local ou Cluster Atlas)
- Cloudflared CLI
- Projeto configurado no Google Cloud Console (Credenciais OAuth)

### Inicialização da Camada de Aplicação (Backend)
1. Acesse o diretório `backend` e instale as dependências: `npm install`
2. Crie o arquivo `.env` (baseado no `.env.example`) e configure as variáveis obrigatórias (`MONGODB_URI`, `JWT_SECRET`, `GOOGLE_CLIENT_ID`).
3. Inicie o servidor de desenvolvimento: `npm run dev`

### Inicialização do Serviço de Mensageria (Evolution API)
1. Acesse o diretório `evolution-api` e configure as variáveis de ambiente necessárias.
2. Inicialize o serviço: `npm run start`
3. Exponha a porta local via túnel seguro para recebimento de webhooks: 
   `cloudflared tunnel run --url http://127.0.0.1:3000 <nome-do-tunel>`

### Inicialização da Camada de Apresentação (Frontend)
1. Acesse o diretório `frontend` e instale as dependências: `npm install`
2. Inicie o servidor Vite: `npm run dev`

---

## 6. Referências de Interface (UI/UX)

*Nota: As imagens ilustrativas abaixo demonstram a implementação técnica de layouts responsivos e injeção de variáveis CSS.*

| Módulo | Interface | Descrição Técnica |
| :--- | :---: | :--- |
| **Fluxo do Cliente (White-label)** | `<img src="docs/screenshots/client_home.png" width="400" alt="Interface Cliente"/>` | Interface adaptável. As classes do Tailwind herdam propriedades de variáveis CSS injetadas na renderização do *tenant*. |
| **Motor de Agendamento** | `<img src="docs/screenshots/client_booking.png" width="400" alt="Fluxo de Agendamento"/>` | Funil de conversão em 3 etapas. Otimizado para reduzir a carga cognitiva, dispensando criação de credenciais. |
| **Painel Administrativo** | `<img src="docs/screenshots/admin_dashboard.png" width="400" alt="Dashboard Gestão"/>` | Aplicação de *Dark Theme* nativo para consoles de gestão, priorizando o contraste em visualização de métricas (Charts). |
| **Gestão de Agenda** | `<img src="docs/screenshots/admin_agenda.png" width="400" alt="Calendário Administrativo"/>` | Renderização complexa de grid de calendário para lidar com validação de choques de horário em tempo real. |

---

## 7. Propriedade Intelectual e Licença

Este repositório é estritamente designado como **Documentação Técnica e Portfólio (Showcase)**.

A totalidade do código-fonte, decisões de arquitetura, lógicas de negócio, interfaces de usuário (UI) e ativos de marca contidos neste projeto constituem propriedade intelectual exclusiva da **Sanyr Tecnologia**.

**Permissões Concedidas:**
- Leitura, visualização e análise do código-fonte com o propósito exclusivo de avaliação técnica, processos seletivos e revisão educacional.

**Restrições:**
- É terminantemente proibida a cópia, clonagem (fork), modificação, distribuição, hospedagem ou qualquer forma de exploração comercial ou não-comercial deste software sem autorização prévia por escrito.
- É vedada a utilização de componentes estruturais, arquitetônicos ou visuais para o desenvolvimento de soluções concorrentes.

**Copyright © Sanyr Tecnologia. Todos os direitos reservados (All Rights Reserved).**
