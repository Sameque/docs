# Laya Serve - Imagem Docker all-in-one (CPU)

Imagem CPU com todas as dependências instaladas e os 3 checkpoints do
Hugging Face (`english`, `multilingual`, `typed-decisions`) já baixados
dentro da imagem. O container sobe em segundos, sem download em runtime.

Requisitos: Docker com Compose v2+, ~8 GB de RAM disponíveis e ~10 GB
de disco.

## 1. Build

```bash
docker build -t laya-serve:0.3.20 .
```

(O download dos pesos acontece DURANTE o build, com cache do BuildKit;
o snapshot do repo `convaiinnovations/laya` ~2,4 GB já contém os 3
checkpoints, nas subpastas em que o Router os procura.)

## 2. Variantes de build

Versão específica do Laya:

```bash
docker build --build-arg LAYA_VERSION=0.3.20 -t laya-serve:0.3.20 .
```

(Novas versões do pacote saem no [PyPI](https://pypi.org/project/laya/);
para usar a mais recente, ajuste `LAYA_VERSION` e a tag.)

## 3. Execução

### Com docker compose (recomendado):

Crie um arquivo `.env` ao lado do `docker-compose.yml` com a chave da API
(o compose falha imediatamente se `LAYA_API_KEY` não estiver definida):

```bash
echo "LAYA_API_KEY=$(openssl rand -hex 32)" > .env
```

Depois:

```bash
docker compose up -d --build
```

### Com docker run:

```bash
export LAYA_API_KEY=$(openssl rand -hex 32)
echo "Chave gerada: $LAYA_API_KEY"

docker run -d --name laya \
  -p 8000:8000 \
  -e LAYA_API_KEY=$LAYA_API_KEY \
  laya-serve:0.3.20
```

## 4. Teste rápido

```bash
curl -s http://localhost:8000/health

curl -s http://localhost:8000/v1/systemone \
  -H "Authorization: Bearer SUA_CHAVE" \
  -H "content-type: application/json" \
  -d '{
    "state": {"body": "billed twice, refund please"},
    "questions": {
      "dept": {"type": "choice",
               "instructions": "which team?",
               "criteria": {"billing": "refunds", "tech": "bugs"}},
      "refund": {"type": "noul",
                 "instructions": "Does the user request a refund?"}
    }
  }'
```

`/health` não exige autenticação; com `LAYA_PRELOAD=1` um container
saudável já tem os checkpoints carregados.

## 5. Variáveis de ambiente

| Variável         | Padrão (imagem)                        | Descrição                              |
|------------------|----------------------------------------|----------------------------------------|
| LAYA_DEVICE      | cpu                                    | cpu (imagem é CPU-only)                |
| LAYA_PRELOAD     | 1                                      | pré-carrega checkpoints no startup     |
| LAYA_HOST        | 0.0.0.0                                | bind address                           |
| LAYA_PORT        | 8000                                   | porta                                  |
| LAYA_MODELS      | english,multilingual,typed-decisions   | checkpoints pré-carregados             |
| LAYA_API_KEY     | (vazio = SEM auth)                     | exige `Authorization: Bearer <chave>`  |
| LAYA_AUTO_TASK   | 0                                      | roteia workflows do typed-decisions    |
| LAYA_THREADS     | OMP_NUM_THREADS (4)                    | threads de inferência CPU              |

## 6. Notas

- Sem `LAYA_API_KEY`, o endpoint fica SEM autenticação. Sempre defina
  a chave em produção.
- `TRANSFORMERS_OFFLINE=1` e `HF_HUB_OFFLINE=1` garantem que o servidor
  nunca tenta baixar nada em runtime — se faltar peso, falha rápido em
  vez de travar.
- `LAYA_AUTO_TASK=0` faz o Router alternar apenas entre `english` e
  `multilingual`; o `typed-decisions` é usado quando a requisição o
  escolhe explicitamente. Em host com pouca RAM, pré-carregue menos:
  `LAYA_MODELS=english,multilingual`.
- A imagem é CPU-only (base `python:3.12-slim` + torch de
  `download.pytorch.org/whl/cpu`). Para GPU seria preciso outra base e
  outro wheel de torch — não é o alvo deste repositório.
- TLS não é nativo. Em produção, coloque Caddy/nginx na frente (`/health`
  não exige auth, então o healthcheck continua funcionando atrás do proxy).
