# 📊 GEO Analytics Service

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.110-green?logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue?logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-ready-blue?logo=docker&logoColor=white)
![Tests](https://img.shields.io/badge/Tests-107%20passing-brightgreen)
![Coverage](https://img.shields.io/badge/Coverage-80%25+-brightgreen)

> Serviço Python de análise GEO — detecta AI Overviews, calcula GEO Index e Share of Voice integrando com Google Search Console via OAuth 2.0.

Desenvolvido como parte do SaaS B2B **GEO Intelligence**, em colaboração com [@marcosalvesdev](https://github.com/marcosalvesdev).

---

## 🧠 O que é GEO?

**GEO (Generative Engine Optimization)** é a prática de otimizar conteúdo para aparecer em respostas geradas por IA como ChatGPT, Gemini e Copilot.

Quando um motor generativo exibe uma resposta direta (**AI Overview**), os links orgânicos abaixo recebem menos cliques do que o esperado para aquela posição. Este serviço detecta esse padrão e mede o impacto real no tráfego orgânico.

---

## ⚙️ Stack de Tecnologias

| Camada | Tecnologia |
|---|---|
| API | FastAPI + Uvicorn |
| Banco de dados | PostgreSQL 16 + SQLAlchemy |
| Migrations | Alembic |
| Criptografia | Fernet (AES-256) |
| Testes | Pytest + SQLite in-memory |
| Infraestrutura | Docker Compose |
| Integração | Google Search Console API v3 |
| Auth | OAuth 2.0 (Google) |

---

## 🔍 O que este serviço faz

- ✅ Conecta ao **Google Search Console** via OAuth 2.0 e sincroniza dados de performance (cliques, impressões, CTR, posição)
- ✅ **Detecta AI Overviews** por anomalia estatística de CTR — sem depender de sinais diretos do Google
- ✅ Calcula o **GEO Index** (0–100): métrica proprietária de saúde da visibilidade orgânica frente à IA
- ✅ Calcula **Share of Voice** por marca de IA (ChatGPT, Gemini, Copilot)
- ✅ Analisa **séries temporais** de cliques para identificar tendências
- ✅ Armazena tokens OAuth **criptografados** com Fernet (AES-256)
- ✅ **Multi-tenant**: isolamento total de dados por `organization_id`

---

## 🧮 Algoritmos desenvolvidos

### Detecção de AI Overview (AIO)
O Google não informa quando exibe um AI Overview. O serviço infere isso comparando o CTR real com o CTR esperado por posição:

```
Posição 1 → CTR esperado: ~28%
Posição 2 → CTR esperado: ~15%
Posição 3 → CTR esperado: ~11%
```

Se uma página no 1º lugar recebe apenas 8% de cliques, o algoritmo sinaliza presença de AIO.

### GEO Index
Número de 0 a 100 representando a saúde da visibilidade GEO:
- **100** = nenhum impacto detectado de AI Overviews
- **0** = CTR completamente suprimido em todas as queries monitoradas

```
GEO Index = média(CTR_real / CTR_esperado) × 100
```

### Share of Voice
Percentual de cliques que cada IA (ChatGPT, Gemini, Copilot) captura nas buscas monitoradas.

---

## 🗂️ Arquitetura

```
geo-analytics/
├── src/
│   ├── api/          # Camada HTTP (routers FastAPI)
│   ├── analytics/    # Algoritmos: AIO detector, GEO Index, Share of Voice
│   ├── cleaning/     # Limpeza e normalização de dados
│   ├── collection/   # Integração Google Search Console
│   ├── core/         # Config, segurança, logs, exceções
│   ├── middleware/   # CORS e tratamento de erros
│   ├── models/       # ORM (SQLAlchemy), repositório, schemas Pydantic
│   ├── services/     # Orquestração da lógica de negócio
│   └── utils/        # Validadores, formatadores, criptografia
├── migrations/       # Histórico Alembic do banco de dados
├── tests/            # 107 testes automatizados
└── docker-compose.yml
```

**Fluxo de dados:**
```
POST /gsc/sync
  → GSCService → coleta Google API → limpa dados → salva no banco

GET /analytics/*
  → busca do banco → detecta AIO → calcula GEO Index → retorna JSON
```

---

## 🛣️ Endpoints disponíveis

| Método | Rota | Descrição |
|---|---|---|
| GET | `/` | Health check |
| POST | `/api/v1/gsc/sync` | Dispara coleta de dados do GSC |
| POST | `/api/v1/oauth/token` | Salva token OAuth (criptografado) |
| GET | `/api/v1/oauth/status` | Verifica se token existe |
| DELETE | `/api/v1/oauth/token` | Remove token |
| GET | `/api/v1/analytics/geo-metrics` | GEO Index e impacto de AIO |
| GET | `/api/v1/analytics/share-of-voice` | Share of Voice por marca de IA |

Documentação interativa disponível em `http://localhost:8000/docs` com a API rodando.

---

## 🚀 Como rodar localmente

```bash
# 1. Clone o repositório
git clone https://github.com/Guilherme-fonte/geo-analytics-service.git
cd geo-analytics-service

# 2. Crie e ative o ambiente virtual
python -m venv venv
venv\Scripts\activate  # Windows
# source venv/bin/activate  # Linux/Mac

# 3. Instale as dependências
pip install -r requirements.txt

# 4. Configure o ambiente
copy .env.example .env

# 5. Suba o banco com Docker
docker-compose up -d

# 6. Aplique as migrations
python -m alembic upgrade head

# 7. Inicie a API
uvicorn src.main:app --reload --port 8000
```

---

## 🧪 Testes

```bash
# Rodar todos os 107 testes
pytest tests/ -v

# Com relatório de cobertura
pytest tests/ --cov=src --cov-report=term-missing
```

Os testes usam **SQLite em memória** — sem necessidade de Docker ou banco externo rodando. Integrações externas são mockadas.

---

## 🔐 Variáveis de Ambiente

| Variável | Obrigatória | Descrição |
|---|---|---|
| `DATABASE_URL` | ✅ | String de conexão PostgreSQL |
| `ENCRYPTION_KEY` | ✅ | Chave Fernet para criptografia de tokens |
| `API_KEY` | ❌ | Autenticação por API Key (vazio = sem auth) |
| `LOG_LEVEL` | ❌ | DEBUG, INFO, WARNING, ERROR |

---

## 👨‍💻 Sobre o Desenvolvedor

Desenvolvido por **Guilherme Andrade** como parte do estágio em desenvolvimento de software.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Guilherme%20Andrade-blue?logo=linkedin)](https://linkedin.com/in/guilherme-andrade-5a9504200)
[![GitHub](https://img.shields.io/badge/GitHub-Guilherme--fonte-black?logo=github)](https://github.com/Guilherme-fonte)
