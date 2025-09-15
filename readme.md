# Configuração de Ambiente de Desenvolvimento com Docker Compose

Este guia detalha a configuração de um ambiente de desenvolvimento padronizado e reprodutível para uma aplicação web moderna. O projeto utiliza Docker Compose para orquestrar diversos serviços, garantindo que toda a equipe de desenvolvimento trabalhe em um ambiente idêntico.

-----

### Visão Geral do Ecossistema

O ambiente é composto pelos seguintes serviços, todos interconectados através de uma rede Docker:

  * **Frontend**: Aplicação React com Vite.
  * **Backend**: Aplicação NestJS.
  * **Banco de Dados**: PostgreSQL para persistência de dados.
  * **Admin do Banco de Dados**: pgAdmin, uma interface gráfica para gerenciar o PostgreSQL.
  * **Proxy Reverso**: Nginx, responsável por rotear o tráfego para os serviços corretos.

-----

### Estrutura de Diretórios

A estrutura do projeto foi organizada para facilitar a gestão dos serviços e suas configurações.

```
.
├── .env.example
├── docker-compose.yml
├── volumes
│   ├── db-data
│   └── pgadmin-data
├── frontend
│   ├── Dockerfile
│   └── (seu projeto React/Vite)
├── backend
│   ├── Dockerfile
│   └── (seu projeto NestJS)
└── proxy
    ├── Dockerfile
    └── nginx.conf
```

-----

### Passo a Passo para Configuração

#### Passo 1: Configurar Variáveis de Ambiente

Crie um arquivo chamado `.env` na raiz do seu projeto, copiando o conteúdo de `.env.example`. Este arquivo centraliza todas as variáveis configuráveis do ambiente, como portas e credenciais.

**`.env`**

```env
# Variáveis do Projeto
PROJECT_NAME=minha-app

# Portas
APP_PORT=3000
FRONTEND_PORT=3001
PROXY_PORT=80
DB_PORT=5432
PGADMIN_PORT=8080

# Credenciais do Banco de Dados
POSTGRES_USER=appuser
POSTGRES_PASSWORD=apppassword
POSTGRES_DB=appdb

# Credenciais do pgAdmin
PGADMIN_DEFAULT_EMAIL=admin@admin.com
PGADMIN_DEFAULT_PASSWORD=adminpassword
```

#### Passo 2: Criar os Projetos Frontend e Backend

Para que o ambiente funcione, as pastas `frontend` e `backend` devem conter projetos válidos.

1.  **Crie o projeto React/Vite**:
    Acesse o diretório `frontend` e execute:

    ```bash
    npm create vite@latest . -- --template react-ts
    ```

2.  **Crie o projeto NestJS**:
    Acesse o diretório `backend` e execute:

    ```bash
    npx @nestjs/cli new . --skip-install
    ```

    Após a criação, instale as dependências com `npm install` em ambos os diretórios.

#### Passo 3: Inicializar o Ambiente Docker

Com os arquivos e projetos no lugar, execute o comando abaixo na raiz do seu projeto. A flag `--build` garante que as imagens customizadas do Dockerfile sejam construídas antes da inicialização dos contêineres.

```bash
docker-compose up --build
```

O Docker Compose fará o seguinte:

  * Construirá as imagens do `frontend`, `backend` e `proxy`.
  * Criará e iniciará todos os cinco serviços.
  * Criará a rede `web-network` para a comunicação entre os serviços.
  * Criará os volumes `db-data` e `pgadmin-data` para persistência dos dados.

-----

### Rotas e Funcionamento

O serviço de **Nginx** atua como um proxy reverso, gerenciando todo o tráfego de entrada e redirecionando as requisições para os serviços apropriados com base nas regras de roteamento definidas em `nginx.conf`.

#### Mapeamento de Rotas

| URL de Acesso       | Serviço Destino       | Descrição                                                                            |
| ------------------- | --------------------- | ------------------------------------------------------------------------------------ |
| `http://localhost/` | **Frontend (React)** | Rota principal. O Nginx direciona todas as requisições para a aplicação frontend.    |
| `http://localhost/api/` | **Backend (NestJS)** | O Nginx redireciona todas as requisições que começam com `/api/` para o serviço backend. O prefixo `/api/` é removido. |

#### Acesso aos Serviços

  * **Aplicação Frontend**: Acesse através de `http://localhost`.
  * **API Backend**: Acesse as rotas da sua API via `http://localhost/api/`. Por exemplo, se você tem uma rota `/users` no seu backend, ela será acessível em `http://localhost/api/users`.
  * **pgAdmin**: Acesse a interface de administração em `http://localhost:8080` com as credenciais definidas no `.env`.

-----

### Persistência de Dados e Hot-Reload

  * **Persistência de Dados**: Os volumes `db-data` e `pgadmin-data` garantem que os dados do banco de dados e as configurações salvas no pgAdmin não sejam perdidos ao parar e reiniciar os contêineres.
  * **Hot-Reload**: O mapeamento de volumes (`./frontend:/app` e `./backend:/app`) nos contêineres de desenvolvimento sincroniza o código do seu sistema de arquivos local com o contêiner em tempo real. Isso permite que qualquer alteração nos arquivos do frontend ou backend seja refletida imediatamente, sem a necessidade de reiniciar o contêiner.