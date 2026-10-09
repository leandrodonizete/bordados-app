# BordadosApp - Sistema de Gestão de Bordados

Sistema web para gestão de uma microempresa de bordados eletrónicos. O projeto combina uma API Django com um frontend React/TypeScript/Vite/Tailwind, otimizado para um ambiente de produção distribuído (Vercel + Render + Umbler) e ambiente de desenvolvimento local via Docker.

## 🏗️ Arquitetura do Projeto

*   **Frontend:** React (Vite) alojado na **Vercel** (com domínio personalizado).
*   **Backend API:** Django 4.1.13 alojado no **Render** (Web Service).
*   **Base de Dados:** MySQL 5.7 externo gerido pela **Umbler**.
*   **Armazenamento de Código/CI:** GitHub.

---

## 🚀 Guia de Deploy em Produção (Nuvem)

A aplicação está configurada para funcionar num ecossistema de nuvem. Siga estas diretrizes para manter o sistema online.

### 1. Base de Dados (Umbler)
*   **Motor:** MySQL 5.7 (Nota: O Django foi estabilizado na versão 4.1.13 para manter compatibilidade total com o MySQL 5.7 da Umbler).
*   Os dados de acesso (`DATABASE_URL`) devem ser configurados nas variáveis de ambiente do backend.

### 2. Backend (Render)
No painel do Render, o Web Service deve conter a seguinte configuração:
*   **Build Command:** `pip install -r requirements.txt`
*   **Start Command:** 
    cd api && python manage.py migrate --noinput && python manage.py createsuperuser --username admin --email admin@teste.com --noinput || true && gunicorn core.wsgi:application
*   **Variáveis de Ambiente (Environment):**
    *   `DATABASE_URL`: `mysql://utilizador:palavra-passe@host:porta/base_de_dados`
    *   `DJANGO_ALLOWED_HOSTS`: `*` *(ou os domínios exatos sem o prefixo https://)*
    *   `DJANGO_CORS_ALLOWED_ORIGINS`: `https://bordados-app-gamma.vercel.app,https://bordados.elidsystem.com.br` *(separados por vírgula, sem espaços e com https://)*
    *   `DJANGO_SUPERUSER_PASSWORD`: `[A_SUA_PALAVRA_PASSE]`

### 3. Frontend (Vercel)
*   O deploy é automático através da ligação ao repositório GitHub.
*   **Rotas SPA (Single Page Application):** Para evitar erros `404 Not Found` ao recarregar páginas ou aceder a rotas diretas (como `/login`), a raiz do frontend contém um ficheiro `vercel.json` com regras de reescrita para o `index.html`.
*   **DNS:** O domínio `bordados.elidsystem.com.br` está apontado para a Vercel na gestão de DNS da Umbler através de um registo tipo **A** a apontar para o IP `76.76.21.21`.

---

## 💻 Ambiente de Desenvolvimento Local

Para desenvolver novas funcionalidades no seu computador, a arquitetura local utiliza contentores para simular o ambiente.

### Pré-requisitos
- **Docker** (versão 20+)
- **Docker Compose** (versão 2+)
- **Git**

### Guia de Início Rápido Local

1. **Clonar o repositório**
   git clone https://github.com/seu-utilizador/bordados-app.git
   cd bordados-app

2. **Subir a aplicação com Docker Compose**
   # Iniciar banco, backend (API Django) e frontend (React)
   docker compose up -d
   
   # Acompanhar os registos (logs) da aplicação
   docker compose logs -f

   *O Docker Compose lê o ficheiro `.env` da raiz. Garanta que os ficheiros `api/.env` e `app/.env` estão preenchidos com base nos exemplos fornecidos.*

3. **Aceder à aplicação (Local)**
   - **Frontend:** `http://localhost:5173`
   - **Backend API (root):** `http://localhost:8000/api/`
   - **API Docs (Swagger UI):** `http://localhost:8000/api/docs/`

4. **Parar a aplicação**
   docker compose down

---

## 🛠️ Comandos de Desenvolvimento e Testes

**Usar Makefile (Recomendado):**
# Ver todos os comandos disponíveis
make help

# Rodar testes dentro do Docker Compose (Backend + Frontend)
make test

# Simular o pipeline de CI localmente
make ci

**Com Docker Direto:**
# Executar testes no Backend
docker compose exec api pytest --tb=short

# Executar testes no Frontend
docker compose exec frontend npm run test -- --run

### CI/CD Pipeline
A cada Push ou Pull Request, o GitHub Actions executa automaticamente:
- ✅ Testes do backend
- ✅ Testes do frontend
Se os testes falharem, o PR é bloqueado. Os *deploys* para a Vercel e Render disparam automaticamente na branch `main` após sucesso.

---

## 🐛 Resolução de Problemas (Troubleshooting)

### Erro 400 (Bad Request) ou Bloqueio de CORS no Login (Produção)
*   **Verificar `DJANGO_CORS_ALLOWED_ORIGINS` no Render:** Confirme se os URLs do frontend estão exatos, separados por vírgula, sem espaços e incluem `https://`.
*   **Verificar `DJANGO_ALLOWED_HOSTS` no Render:** Confirme se tem o valor `*` (asterisco). Se contiver URLs, eles **não** podem ter `https://` (exemplo: `bordados.elidsystem.com.br`).
*   **Formato dos Dados:** A API espera receber o *payload* de início de sessão com os campos `{"email": "...", "senha": "..."}`.

### Frontend devolve 404 ao recarregar a página (Vercel)
Certifique-se de que o ficheiro `vercel.json` na raiz da pasta `app/` (junto ao `package.json`) não foi apagado. Ele é responsável por redirecionar os caminhos para o router do React:
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}

### Dependências desatualizadas no ambiente local
# Recriar os contentores e renovar os volumes
docker compose down -v
docker compose up -d --build




# Histórico de Alterações e Configurações de Produção

Durante o processo de *deploy* e ligação entre os serviços (Vercel, Render e Umbler), foram feitas as seguintes alterações estruturais no projeto original:

## 1. Ficheiros Adicionados
*   **`app/vercel.json`**
    *   **Motivo:** Aplicações React (SPA) geradas com Vite perdem a referência das rotas quando a página é recarregada num servidor estático, gerando o erro `404 Not Found`.
    *   **Ação:** Criado na raiz do frontend para forçar a Vercel a reescrever todos os caminhos para o `index.html`.
    *   **Conteúdo adicionado:**
        {
          "rewrites": [
            { "source": "/(.*)", "destination": "/index.html" }
          ]
        }

## 2. Ficheiros Editados
*   **`api/requirements.txt`** (e/ou ficheiro equivalente de dependências do Python)
    *   **Motivo:** A versão mais recente do Django estava a gerar conflitos de compatibilidade com a base de dados MySQL 5.7 (hospedada na Umbler).
    *   **Ação:** O Django foi alvo de um *downgrade* (fixado na versão `4.1.13`) para garantir estabilidade total com o motor da base de dados externa.

*   **`README.md`**
    *   **Motivo:** Necessidade de documentar a nova arquitetura em nuvem.
    *   **Ação:** Atualizado com as instruções de deploy no Render, Vercel e Umbler, mantendo o guia do Docker para desenvolvimento local.

## 3. Configurações de Ambiente (Infraestrutura)
Apesar de não estarem no código-fonte do repositório, as seguintes variáveis foram obrigatoriamente ajustadas no painel do **Render** (Backend) para permitir a comunicação com o Frontend:

*   **`DJANGO_ALLOWED_HOSTS`**
    *   **Alteração:** Definido como `*` (asterisco).
    *   **Motivo:** O Render estava a bloquear os pedidos de início de sessão (Erro 400 Bad Request em formato HTML) porque o protocolo `https://` tinha sido incluído erradamente na lista de domínios permitidos. O asterisco removeu esta barreira.

*   **`DJANGO_CORS_ALLOWED_ORIGINS`**
    *   **Alteração:** Adicionados os links precisos do frontend: `https://bordados-app-gamma.vercel.app,https://bordados.elidsystem.com.br`
    *   **Motivo:** O navegador (Vercel) bloqueava o acesso à API (Render) por política de CORS. Foi necessário registar as duas origens com `https://` para autorizar a troca de dados (JSON).

## 4. Configuração de DNS (Umbler)
*   **Ação:** O registo DNS principal (`@`) do domínio `bordados.elidsystem.com.br` foi alterado do IP original para o IP oficial da Vercel (`76.76.21.21`) no formato de registo **A**.



## 🗺️ Organograma da Arquitetura

Abaixo encontra-se a representação visual de como os serviços em nuvem estão conectados em produção:

```text
                    👤 Utilizador Final (Navegador/Telemóvel)
                                  │
                                  ▼ (Acessa: bordados.elidsystem.com.br)
                                  │
[Gestão de Domínio]          🌐 DNS (Umbler)
                                  │
                                  ▼ (Apontamento Registo A para IP Vercel)
                                  │
[Hospedagem Frontend]        🖥️ Vercel (Frontend React/Vite) ──────────┐
                                  │                                    │ (Deploy Automático)
                                  ▼ (Requisições API via HTTPS)        │
                                  │                                    ▼
[Hospedagem Backend]         ⚙️ Render (API Django 4.1.13) ⮜═════ 🐙 GitHub (Repositório)
                                  │                                    ▲
                                  ▼ (Conexão via DATABASE_URL)         │
                                  │                                    │ (Deploy Automático)
[Serviço de Dados]           🗄️ MySQL 5.7 (Umbler) ────────────────────┘




graph TD
    A[👤 Utilizador Final] -->|Acessa: bordados.elidsystem.com.br| B{🌐 DNS - Umbler}
    B -->|Apontamento de Registo A| C(🖥️ Frontend React/Vite - Vercel)
    C -->|Requisições API via HTTPS| D(⚙️ API Django 4.1.13 - Render)
    D -->|Conexão via DATABASE_URL| E[(🗄️ MySQL 5.7 - Umbler)]
    
    F[[🐙 Repositório - GitHub]] -.->|Deploy Automático| C
    F -.->|Deploy Automático| D

    style A fill:#f9f9f9,stroke:#333,stroke-width:2px
    style B fill:#ffe6cc,stroke:#ff9900,stroke-width:2px
    style C fill:#e6f3ff,stroke:#0066cc,stroke-width:2px
    style D fill:#e6ffe6,stroke:#00cc00,stroke-width:2px
    style E fill:#f2e6ff,stroke:#6600cc,stroke-width:2px
    style F fill:#f2f2f2,stroke:#666,stroke-width:2px