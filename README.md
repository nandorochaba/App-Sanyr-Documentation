<h1 align="center">
  Sanyr - Sistema de Agendamentos
</h1>

<p align="center">
  <img src="https://img.shields.io/badge/React-18-blue.svg" alt="React" />
  <img src="https://img.shields.io/badge/Vite-5-646CFF.svg" alt="Vite" />
  <img src="https://img.shields.io/badge/Node.js-Express-339933.svg" alt="Node.js" />
  <img src="https://img.shields.io/badge/MongoDB-Atlas-47A248.svg" alt="MongoDB" />
  <img src="https://img.shields.io/badge/TailwindCSS-v4-38B2AC.svg" alt="Tailwind CSS" />
  <img src="https://img.shields.io/badge/TypeScript-Ready-3178C6.svg" alt="TypeScript" />
  <img src="https://img.shields.io/badge/License-All_Rights_Reserved-red.svg" alt="License" />
</p>

<p align="center">
  <strong>Plataforma SaaS multi-tenant voltada para agendamentos de serviços, com ênfase em alto desempenho, estabilidade e arquitetura WhiteLabel.</strong>
</p>

---

## 1. Visão Geral do Projeto

O **Sanyr** é um Software as a Service (SaaS) corporativo desenvolvido para otimizar a gestão de agendas e o relacionamento com o cliente em estabelecimentos focados em serviços com hora marcada. A plataforma mitiga conflitos de horários, automatiza o processo de reservas e fornece um painel administrativo robusto para controle operacional.

A arquitetura do sistema baseia-se no conceito de **Dual Theme**:
- **Interface do Cliente (WhiteLabel):** Adapta-se dinamicamente à identidade visual da marca (tenant) atendida, garantindo consistência visual e imersão durante o fluxo de agendamento.
- **Interface Administrativa (Premium Dark Theme):** O painel de gestão (`/admin`) adota um tema escuro de alto contraste, projetado ergonomicamente para reduzir a fadiga visual de operadores e gestores durante jornadas prolongadas de uso.

---

## 2. Interface e Telas

*(Nota estrutural: Adicione as evidências visuais no diretório `docs/screenshots` na raiz do projeto)*

| Visão do Cliente (Fluxo de Agendamento) | Painel Administrativo (Gestão) |
| :---: | :---: |
| <img src="docs/screenshots/client_home.png" alt="Home Cliente" width="400"/> | <img src="docs/screenshots/admin_dashboard.png" alt="Admin Dashboard" width="400"/> |
| <img src="docs/screenshots/client_booking.png" alt="Fluxo de Reserva" width="400"/> | <img src="docs/screenshots/admin_agenda.png" alt="Admin Agenda" width="400"/> |

---

## 3. Principais Funcionalidades

### Funcionalidades do Cliente (WhiteLabel)
- **Reserva Autônoma Otimizada:** Fluxo de conversão reduzido a três etapas, dispensando a exigência de criação de contas ou gerenciamento de senhas pelo usuário final.
- **Injeção de Identidade Visual:** As variáveis de design do lojista (tenant) refletem-se em tempo real em componentes estruturais (botões, superfícies e tipografia).
- **Apresentação de Portfólio:** Módulo integrado para exibição de trabalhos e currículo dos profissionais associados.
- **Arquitetura Mobile-First:** Design estruturado para dispositivos móveis com métricas controladas de Cumulative Layout Shift (CLS) e carregamento otimizado.

### Funcionalidades do Gestor (Dashboard Administrativo)
- **Motor de Agendamento Inteligente:** Algoritmo proprietário para alocação de tempo, cálculo de intervalos e prevenção de sobreposição de horários.
- **Gestão Operacional:** Controle independente de equipe, catálogo de serviços, precificação e regras de expediente por profissional.
- **Segurança de Acesso:** Autenticação consolidada via Google OAuth 2.0 e sistema de recuperação de credenciais baseado em tokens criptografados com tempo de expiração.
- **Painel Analítico:** Dashboards operacionais empregando semântica de cores padrão (Azul para volumetria, Verde para receita, Amarelo para alertas, Vermelho para churn/perdas).

---

## 4. Arquitetura do Sistema

A solução foi concebida sob uma arquitetura de três camadas escaláveis (Frontend SPA, Backend REST API e Cloud Database):

```mermaid
graph TD
    subgraph Cliente Navegador (Vercel Production)
        RootApp["Redirect Raiz (/) ➔ /admin/login"]
        ClientApp["Public Client Funnel (/:tenantSlug)"]
        AdminApp["Admin Dashboard (/admin)"]
        GoogleOAuth["Google Sign-In (@react-oauth/google)"]
    end

    subgraph Server Layer (Express API)
        ExpressAPI["Node.js + Express REST API (Porta 3000)"]
        GoogleVerify["Google OAuth 2.0 Verifier"]
        PasswordResetEngine["Crypto Password Reset Token"]
        ConflictEngine["Prevenção de Conflitos & Validações"]
    end

    subgraph Database Layer (Cloud)
        MongoDB[(MongoDB Atlas Cloud)]
    end

    RootApp --> AdminApp
    ClientApp <-->|HTTP / JSON| ExpressAPI
    AdminApp <-->|HTTP / JSON| ExpressAPI
    GoogleOAuth <-->|ID Token| GoogleVerify
    ExpressAPI <--> PasswordResetEngine
    ExpressAPI <--> ConflictEngine
    ExpressAPI <-->|Mongoose ODM| MongoDB
```

---

## 5. Stack Tecnológico

**Camada de Apresentação (Frontend):**
- [React 18](https://reactjs.org/) + [Vite](https://vitejs.dev/)
- [Tailwind CSS v4](https://tailwindcss.com/)
- [TypeScript](https://www.typescriptlang.org/) rigorosamente tipado
- Autenticação Google (`@react-oauth/google`)

**Camada de Serviços (Backend):**
- [Node.js](https://nodejs.org/) + [Express](https://expressjs.com/)
- [Mongoose](https://mongoosejs.com/) (Object Data Modeling)
- `google-auth-library` para integridade de tokens

**Camada de Infraestrutura:**
- [Vercel](https://vercel.com/) (Hospedagem de recursos estáticos e Serverless Functions)
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) (Persistência de dados)

---

## 6. Estrutura do Repositório

```text
Beauty Service Booking App/
├── api/                   # Mapeamento para Vercel Serverless Functions
├── frontend/              # Source da aplicação SPA (React + Vite)
│   └── src/
│       ├── components/    # Primitivas de UI corporativa
│       ├── contexts/      # Hooks de Estado (AuthContext, TenantContext)
│       ├── layouts/       # Componentes de infraestrutura visual (Shell)
│       ├── pages/
│       │   ├── admin/     # Módulos do Painel Administrativo
│       │   └── client/    # Módulos do Fluxo de Agendamento (WhiteLabel)
│       └── services/      # Camada de comunicação de rede (API Client)
│
├── backend/               # Source da API RESTful (Express + Node.js)
│   └── src/
│       ├── db/            # Scripts de Seed e Inicialização
│       ├── models/        # Entidades do Domínio Mongoose
│       ├── routes/        # Mapeamento de Controladores e Endpoints
│       └── server.ts      # Configuração e Entrypoint do Express.js
```

---

## 7. Instruções para Implantação Local

### Pré-requisitos
- **Node.js** (v18 ou superior)
- Instância do **MongoDB** (Local ou Cluster Atlas)
- Credenciais provisionadas no **Google Cloud Console**

### Passos de Execução
1. Realize o clone do repositório:
```bash
git clone https://github.com/nandorochaba/Sanyr-Agendamentos-App.git
cd Sanyr-Agendamentos-App
```

2. Configure a camada de serviços (Backend):
```bash
cd backend
npm install
```
Configure o ambiente criando o arquivo `.env`:
```env
PORT=3000
MONGODB_URI=mongodb+srv://<usuario>:<senha>@<cluster>.mongodb.net/sanyr_booking
GOOGLE_CLIENT_ID=seu_client_id_do_google
JWT_SECRET=sua_chave_secreta
```
Inicie a aplicação:
```bash
npm run dev
```

3. Configure a camada de apresentação (Frontend):
Em um novo terminal:
```bash
cd frontend
npm install
npm run dev
```
O serviço estará acessível no endereço padrão `http://localhost:8443`.

---

## 8. Segurança e Conformidade

- **Autenticação Proprietária:** Suporte padrão com validação estrita de tráfego de senhas em plain-text.
- **Autenticação Federated (Google OAuth 2.0):** Validação de integridade do *ID Token* realizada exclusivamente na camada backend (Server-side) mitigando ataques de personificação.
- **Recuperação de Credenciais:** Emissão de tokens hexadecimais aleatorizados com Time-To-Live (TTL) de 60 minutos, garantindo aderência a práticas seguras de reset de senhas.

---

## 9. Implantação em Produção (Vercel)

A solução suporta integração direta (Monorepo) através da plataforma Vercel:
1. Sincronize o repositório na plataforma.
2. O arquivo `vercel.json` gerenciará os rewrites e rotas automaticamente (não sobrescreva as diretrizes de Build).
3. Insira as variáveis de ambiente necessárias nas configurações do projeto (`MONGODB_URI`, `GOOGLE_CLIENT_ID`).
4. A implantação construirá o frontend de forma estática e alocará a API RESTful em funções Serverless (`api/index.ts`).

---

## 10. Licença e Direitos de Uso (All Rights Reserved)

Este repositório é designado exclusivamente como **Showcase Técnico (Portfólio)**. A totalidade do código-fonte, arquitetura, lógicas de negócio, interfaces e logotipos contidos neste projeto constituem propriedade intelectual exclusiva da **Sanyr Tecnologia**, protegidos sob a legislação vigente de Direitos Autorais.

**Permissões Concedidas:**
- Visualização, leitura e análise do código-fonte para fins estritamente avaliativos, educacionais ou de processos seletivos.

**Restrições (Não Permitido):**
- É expressamente proibida a cópia, clonagem, modificação, distribuição, hospedagem ou qualquer exploração comercial deste software.
- É vedada a utilização de componentes estruturais, lógicas sistêmicas ou identidade visual para o desenvolvimento de soluções concorrentes.

Qualquer violação aos termos estabelecidos estará sujeita a providências legais.

---

**Sanyr Tecnologia.** Todos os direitos reservados.
