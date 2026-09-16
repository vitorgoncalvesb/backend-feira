# Feira Tecnológica - Backend de Votação

Este projeto implementa um backend em FastAPI conectado diretamente ao MariaDB com SQL puro, sem ORM, sem migrations e sem SQLAlchemy.

## Requisitos

- Python 3.11+
- MariaDB/MySQL
- Driver `mysql-connector-python`

## Instalação

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

No Windows PowerShell:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## Configuração do ambiente

Copie o arquivo `.env.example` para `.env` e ajuste as credenciais:

```env
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=sua_senha
DB_NAME=banco_manha
CORS_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
```

## Execução

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

A documentação Swagger estará disponível em:

- http://localhost:8000/docs
- http://localhost:8000/redoc

## Estrutura do projeto

```text
backend/
├── app/
│   ├── database/
│   │   ├── connection.py
│   │   └── queries/
│   ├── routes/
│   ├── schemas/
│   ├── services/
│   ├── config.py
│   ├── main.py
│   └── __init__.py
├── tests/
├── .env
├── .env.example
├── requirements.txt
├── README.md
└── ...
```

## Banco e regras

O backend usa as tabelas existentes do banco `banco_manha`:

- `visitante`
- `curtidas`
- `periodo_votacao`
- `projetos`
- `resultado`

O sistema trabalha com a tabela `curtidas` como registro de voto. O `cookie` funciona como identidade do visitante em navegação; o IP e o user-agent complementam o controle, mas não são uma garantia absoluta de que cada registro representa uma pessoa diferente. Isso fica documentado no código da identificação do visitante.

## Endpoints principais

### Visitantes

- `POST /visitantes/identificar`

### Votos

- `GET /votos/meu-voto`
- `PUT /votos`
- `DELETE /votos`

### Período

- `GET /votacao/status`
- `PATCH /votacao/periodo`
- `POST /votacao/encerrar`

### Ranking

- `GET /ranking`
- `GET /ranking/podio`
- `GET /resultado-final`

## Regras de votação

- Sempre validar o período no backend antes de criar, trocar ou remover um voto.
- O código do frontend pode esconder botões, mas o backend deve rejeitar requisições fora do período.
- Apenas um voto ativo por visitante é permitido.
- A tabela `curtidas` usa o campo `ativa` para indicar se o voto está ativo.
- O ranking é calculado diretamente a partir dos votos ativos.

## Funcionamento do cookie

Ao acessar a API pela primeira vez, o backend cria o visitante e envia o cookie `visitante_id` para o navegador. Em requisições futuras, o cookie identifica o visitante sem depender somente do IP. IPs podem ser compartilhados, cookies podem desaparecer e vários usuários podem usar o mesmo IP.

## Funcionamento do ranking

O ranking considera apenas votos ativos e inclui projetos com zero votos. O cálculo é feito diretamente via SQL e ordenado pela quantidade de votos em ordem decrescente e, em caso de empate, pelo `id_projeto`.

## Encerramento da votação

Ao encerrar, o sistema:

1. verifica o período atual;
2. calcula o ranking final;
3. salva os resultados definitivos na tabela `resultado`;
4. marca o período como encerrado (`andamento = 0`).

## Exemplos de requisição

### Identificar visitante

```bash
curl -X POST http://localhost:8000/visitantes/identificar \
  -H "User-Agent: Mozilla/5.0"
```

### Registrar voto

```bash
curl -X PUT http://localhost:8000/votos \
  -H "Cookie: visitante_id=1" \
  -H "Content-Type: application/json" \
  -d '{"id_projeto": 2}'
```

### Consultar meu voto

```bash
curl http://localhost:8000/votos/meu-voto \
  -H "Cookie: visitante_id=1"
```

### Ranking

```bash
curl http://localhost:8000/ranking
```

## Observações

- Não foi criado nenhum ORM ou migration.
- Todas as consultas utilizam parâmetros em SQL.
- A estrutura do banco foi respeitada sem criar tabelas paralelas.
