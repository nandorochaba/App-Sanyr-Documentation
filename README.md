<h1 align="center">
  Sanyr - Sistema de Agendamentos WhiteLabel
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
  <strong>Uma plataforma SaaS multi-tenant focada em agendamentos de beleza e bem-estar, com uma interface premium e experiência impecável.</strong>
</p>

---

## 📌 O Projeto

O **Sanyr** é um Software as a Service (SaaS) desenvolvido para salões de beleza, barbearias, estúdios de estética e clínicas. A plataforma resolve o desafio da gestão de agendas e conflitos de horários, automatizando as marcações e fornecendo um painel administrativo poderoso para os gestores.

O grande diferencial do sistema é sua abordagem **Dual Theme**:
- **Visão do Cliente (WhiteLabel):** Adapta as cores e a identidade visual da interface pública de acordo com a marca do estabelecimento, proporcionando uma experiência de reserva contínua e imersiva.
- **Visão do Gestor (Premium Dark Theme):** O painel de administração (`/admin`) utiliza um tema escuro e sofisticado de alto contraste, ideal para gestores que passam horas no sistema sem fadiga visual.

---

## 📸 Screenshots e Telas

*(Crie a pasta `docs/screenshots` na raiz do seu repositório e adicione as imagens reais do projeto com os nomes abaixo para visualizá-las aqui)*

| Interface do Cliente (Funil de Agendamento) | Painel Administrativo (Premium Dark) |
| :---: | :---: |
| <img src="docs/screenshots/client_home.png" alt="Home Cliente" width="400"/> | <img src="docs/screenshots/admin_dashboard.png" alt="Admin Dashboard" width="400"/> |
| <img src="docs/screenshots/client_booking.png" alt="Fluxo de Reserva" width="400"/> | <img src="docs/screenshots/admin_agenda.png" alt="Admin Agenda" width="400"/> |

---

## ✨ Principais Funcionalidades

### 💄 Para o Cliente (App WhiteLabel)
- **Reserva Autônoma (3 Cliques):** Sem necessidade de criar contas e gravar senhas. O agendamento é rápido e fluido.
- **Identidade Visual Dinâmica:** As cores da marca do estabelecimento refletem instantaneamente nos botões, fundos e textos.
- **Visualização de Portfólio:** Acesso a fotos de trabalhos anteriores e avaliação dos profissionais.
- **Mobile-First:** Design otimizado para celulares, garantindo fluidez (zero Cumulative Layout Shift) e micro-animações requintadas.

### 💼 Para o Gestor (Admin Dashboard)
- **Agenda Inteligente:** Calendário diário e semanal e algoritmo automático anti-conflito de horários.
- **Gestão de Equipe e Serviços:** Cadastre profissionais, delegue serviços, configure a duração e a precificação de forma independente.
- **Login Seguro e Moderno:** Autenticação via Google OAuth 2.0 e recuperação de senhas segura por Token Criptografado.
- **Dashboard de Métricas:** Visualização clara da performance do negócio utilizando "Coloração Semântica" (Azul para volumes, Verde para lucro, Amarelo para alertas, Vermelho para perdas).

---

## 🏗 Arquitetura do Sistema

A plataforma Sanyr utiliza uma robusta arquitetura dividida em 3 camadas (Frontend SPA, Backend REST API, e Cloud Database):

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

## 🛠 Tecnologias Utilizadas

**Frontend:**
- [React 18](https://reactjs.org/) + [Vite](https://vitejs.dev/)
- [Tailwind CSS v4](https://tailwindcss.com/) (Com suporte nativo a Design System)
- [TypeScript](https://www.typescriptlang.org/)
- Google OAuth `@react-oauth/google`

**Backend:**
- [Node.js](https://nodejs.org/) + [Express](https://expressjs.com/)
- [Mongoose](https://mongoosejs.com/) (ODM para MongoDB)
- Autenticação e Segurança (Tokens Criptografados + `google-auth-library`)

**Infraestrutura & Deploy:**
- [Vercel](https://vercel.com/) (Frontend estático e Serverless Functions no Backend)
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) (Banco de dados de documentos na nuvem)

---

## 📂 Estrutura do Repositório

```text
Beauty Service Booking App/
├── api/                   # Arquivo de entrada para Vercel Serverless Functions
├── frontend/              # Código fonte do Frontend (SPA React + Vite)
│   └── src/
│       ├── components/    # Primitivas (UI) e Elementos Reutilizáveis
│       ├── contexts/      # Hooks de Estado Global (AuthContext, TenantContext)
│       ├── layouts/       # Shell (Admin Layout e Client Layout)
│       ├── pages/
│       │   ├── admin/     # Telas do Painel Administrativo
│       │   └── client/    # Telas do Funil de Agendamento (WhiteLabel)
│       └── services/      # Integração com a API Backend (Axios/Fetch)
│
├── backend/               # Código fonte do Backend (Express + Node.js)
│   └── src/
│       ├── db/            # Script Seeders e Conexão BD
│       ├── models/        # Schemas do MongoDB (Empresa, Usuario, Servico...)
│       ├── routes/        # Controladores e Endpoints REST
│       └── server.ts      # Instância Central do Express.js
```

---

## 🚀 Como Executar o Projeto Localmente

### Pré-requisitos
- **Node.js** (v18+)
- Conta no **MongoDB Atlas** (ou MongoDB local em execução)
- Credenciais OAuth do **Google Cloud Console**

### 1. Clonando o repositório
```bash
git clone https://github.com/nandorochaba/Sanyr-Agendamentos-App.git
cd Sanyr-Agendamentos-App
```

### 2. Configurando o Backend
```bash
cd backend
npm install
```
Crie um arquivo `.env` na raiz da pasta `backend` com as variáveis:
```env
PORT=3000
MONGODB_URI=mongodb+srv://<usuario>:<senha>@<cluster>.mongodb.net/sanyr_booking
GOOGLE_CLIENT_ID=seu_client_id_do_google
JWT_SECRET=sua_chave_secreta_aqui
```
Inicie o servidor de desenvolvimento:
```bash
npm run dev
```

### 3. Configurando o Frontend
Abra um novo terminal e navegue para a pasta `frontend`:
```bash
cd frontend
npm install
```
Inicie o servidor Vite:
```bash
npm run dev
```
A aplicação estará disponível por padrão em `http://localhost:8443`.

---

## 🔒 Mecanismos de Autenticação e Segurança

- **Senha Tradicional:** Suporte nativo offline/online. Senhas trafegadas são validadas de forma estrita.
- **Google OAuth 2.0:** O Backend verifica rigorosamente o *ID Token* emitido pelo front para impedir falsificações e automatiza o *Onboarding* criando tenants, categorias e horários base no primeiro login.
- **Recuperação de Acesso:** O sistema gera tokens hexadecimais aleatórios válidos por 1h (enviados por e-mail) para redefinições de segurança seguras.

---

## 🌐 Deploy em Produção (Vercel)

O projeto suporta hospedagem **Monorepo** diretamente na Vercel:
1. Conecte o repositório Github à Vercel.
2. Nas configurações do Vercel, não sobrescreva as pastas padrão (O `vercel.json` na raiz gerencia os diretórios e `rewrites`).
3. Adicione as variáveis de ambiente necessárias (MONGODB_URI, GOOGLE_CLIENT_ID).
4. O Vercel efetuará o build do Frontend estaticamente e hospedará o Backend como uma Serverless Function (`api/index.ts`).

---

## ⚖️ Licença e Uso (All Rights Reserved)

Este repositório é um **Showcase (Portfólio)**. Todo o código-fonte, design, logotipos e arquitetura deste projeto são propriedade exclusiva da **Sanyr Tecnologia** e estão protegidos pelas leis de Direitos Autorais. 

**O que você PODE fazer:**
- Ler, estudar e avaliar a estrutura do código (para fins de recrutamento ou avaliação técnica).

**O que você NÃO PODE fazer:**
- Clonar, copiar, hospedar, distribuir ou lucrar comercialmente com este software.
- Utilizar trechos de código estruturais deste SaaS para criar um produto concorrente.

Qualquer uso não autorizado em ambiente de produção está sujeito a medidas legais cabíveis.

---

**Desenvolvido com sofisticação pela Sanyr Tecnologia.**
