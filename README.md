# SOS-JF API

API centralizadora de pedidos de ajuda, voluntários, doações e outros dados de emergência de Juiz de Fora/MG, agregando 20+ portais externos.

## Stack

- **FastAPI** + **SQLModel** (async) + **PostgreSQL**
- **APScheduler** — scraping automático a cada hora
- **httpx** + **BeautifulSoup4** — scrapers
- **Ruff** — linter e formatter
- **uv** — gerenciador de pacotes

---

## Quick start

### Local

```bash
cp .env.example .env          # configure suas credenciais
cd backend
uv sync                        # instala dependências
uv run alembic upgrade head    # roda migrations
uv run python app/initial_data.py  # cria superuser
uv run fastapi dev app/main.py     # sobe a API
```

API em **http://localhost:8000** | Docs em **http://localhost:8000/docs**

### Docker

```bash
cp .env.example .env           # mínimo: POSTGRES_PASSWORD, SECRET_KEY, FIRST_SUPERUSER_PASSWORD
docker compose up --build      # sobe tudo (banco + migrations + API)
```

```bash
docker compose down            # para
docker compose down -v         # para e remove dados do banco
```

---

## Variáveis de ambiente

| Variável | Descrição | Padrão |
|----------|-----------|--------|
| `POSTGRES_SERVER` | Host do banco | `localhost` |
| `POSTGRES_PORT` | Porta do banco | `5432` |
| `POSTGRES_DB` | Nome do banco | `app` |
| `POSTGRES_USER` | Usuário | `postgres` |
| `POSTGRES_PASSWORD` | Senha | — |
| `SECRET_KEY` | Chave JWT | — |
| `FIRST_SUPERUSER` | Email do admin | — |
| `FIRST_SUPERUSER_PASSWORD` | Senha do admin | — |
| `SCRAPER_INTERVAL_HOURS` | Intervalo do scraping | `1` |
| `SCRAPER_RUN_ON_STARTUP` | Rodar scraping ao iniciar | `false` |
| `KPIS_INTERVAL_HOURS` | Intervalo de atualização dos KPIs | mesmo do scraper |
| `KPIS_RUN_ON_STARTUP` | Calcular KPIs ao iniciar | `false` |
| `DB_POOL_SIZE` | Tamanho do pool de conexões | `10` |
| `DB_MAX_OVERFLOW` | Conexões extras além do pool | `20` |

---

## Usando a API

### 1. Registrar uma API Key (para o seu app)

```bash
curl -X POST http://localhost:8000/api/v1/api-keys \
  -H "Content-Type: application/json" \
  -d '{"name": "meu-app", "description": "descrição opcional"}'
```

Guarde o campo `key` — ele é exibido **apenas uma vez**.

### 2. Consultar dados

```bash
curl http://localhost:8000/api/v1/pedidos \
  -H "X-API-Key: sos_..."
```

### Endpoints disponíveis

#### Dados públicos (requerem API Key via header `X-API-Key`)

| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/api/v1/pedidos` | Lista pedidos de socorro |
| `POST` | `/api/v1/pedidos` | Cria pedido |
| `PUT/PATCH` | `/api/v1/pedidos/{id}` | Atualiza pedido (só o portal que criou) |
| `GET` | `/api/v1/voluntarios` | Lista voluntários |
| `POST` | `/api/v1/voluntarios` | Cadastra voluntário |
| `PUT/PATCH` | `/api/v1/voluntarios/{id}` | Atualiza voluntário (só o portal que criou) |
| `GET` | `/api/v1/pontos` | Lista pontos de ajuda |
| `POST` | `/api/v1/pontos` | Cria ponto de ajuda |
| `PUT/PATCH` | `/api/v1/pontos/{id}` | Atualiza ponto (só o portal que criou) |
| `GET` | `/api/v1/pets` | Lista pets (perdidos/encontrados/adoção) |
| `POST` | `/api/v1/pets` | Cadastra pet |
| `PUT/PATCH` | `/api/v1/pets/{id}` | Atualiza pet (só o portal que criou) |
| `GET` | `/api/v1/feed` | Lista alertas e notícias |
| `POST` | `/api/v1/feed` | Cria item no feed |
| `PUT/PATCH` | `/api/v1/feed/{id}` | Atualiza item do feed (só o portal que criou) |
| `GET` | `/api/v1/outros` | Lista contatos, links, vaquinhas |
| `POST` | `/api/v1/outros` | Cria item |
| `PUT/PATCH` | `/api/v1/outros/{id}` | Atualiza item (só o portal que criou) |
| `GET` | `/api/v1/eventos` | Lista eventos entre portais |
| `POST` | `/api/v1/eventos` | Cria evento (ex: indicação de doador) |
| `PUT/PATCH` | `/api/v1/eventos/{id}` | Atualiza evento (remetente ou destinatário) |
| `GET` | `/api/v1/kpis/` | Histórico de KPIs |
| `GET` | `/api/v1/kpis/ultimo` | Último snapshot de KPIs |

#### API Keys

| Método | Rota | Auth | Descrição |
|--------|------|------|-----------|
| `POST` | `/api/v1/api-keys` | — | Registra API Key |
| `GET` | `/api/v1/api-keys` | JWT | Lista API Keys do usuário |
| `GET` | `/api/v1/api-keys/me/{prefix}` | JWT | Detalhes de uma key |
| `DELETE` | `/api/v1/api-keys/{id}` | JWT | Remove API Key |

#### Auth e usuários

| Método | Rota | Auth | Descrição |
|--------|------|------|-----------|
| `POST` | `/api/v1/users/signup` | — | Cria conta de desenvolvedor |
| `POST` | `/api/v1/login/access-token` | — | Login (retorna JWT) |

#### Health

| Método | Rota | Descrição |
|--------|------|-----------|
| `GET` | `/api/v1/utils/health-check/` | Liveness check |
| `GET` | `/api/v1/utils/ready/` | Readiness check (banco + último scraping) |

Todos os GETs de dados aceitam `?skip=0&limit=100` e filtros específicos por endpoint (ex: `?cidade=juiz+de+fora&tipo=abrigo`).

Documentação interativa completa em `/docs`.

---

## Contribuindo

Veja o [CONTRIBUTING.md](CONTRIBUTING.md) para setup do ambiente, comandos de lint/format/testes e convenções do projeto.
