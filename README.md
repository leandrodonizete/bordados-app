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