# Python Observability API

PoC mínima de uma API FastAPI usada para gerar requisições, logs, métricas HTTP
automáticas, erros e traces. Não é uma aplicação de produção.

A aplicação não configura OpenTelemetry no código. O container inicia o Uvicorn
com `opentelemetry-instrument`, e toda a configuração do SDK/exportadores é feita
por variáveis de ambiente.

## Stack

- Python 3.12, FastAPI e Uvicorn
- OpenTelemetry Python auto-instrumentation
- Grafana Alloy como receptor OTLP
- Prometheus para métricas, Loki para logs e Tempo para traces
- Grafana com os três data sources provisionados

## Executar com Docker Compose

Pré-requisito: Docker com Docker Compose.

```bash
docker compose up --build
```

Serviços:

- API e Swagger: <http://localhost:8000/docs>
- OpenAPI: <http://localhost:8000/openapi.json>
- Grafana: <http://localhost:3000> (`admin` / `admin` no primeiro acesso)
- Alloy: <http://localhost:12345>
- Prometheus: <http://localhost:9090>
- Loki: <http://localhost:3100/ready>
- Tempo: <http://localhost:3200/ready>

Para encerrar e manter os dados:

```bash
docker compose down
```

Para também remover os volumes da PoC:

```bash
docker compose down -v
```

## Executar somente a API

Construa a imagem:

```bash
docker build -t python-observability-api .
```

Execute apontando para um Alloy acessível pelo container:

```bash
docker run --rm -p 8000:8000 \
  -e OTEL_SERVICE_NAME=python-observability-api \
  -e OTEL_EXPORTER_OTLP_ENDPOINT=http://host.docker.internal:4318 \
  -e OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf \
  -e OTEL_TRACES_EXPORTER=otlp \
  -e OTEL_METRICS_EXPORTER=otlp \
  -e OTEL_LOGS_EXPORTER=otlp \
  -e OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED=true \
  python-observability-api
```

No Linux, adicione `--add-host=host.docker.internal:host-gateway` se o Alloy
estiver no host. Se API e Alloy estiverem na mesma rede Docker, use o nome do
serviço, por exemplo `http://alloy:4318`.

## Variáveis OpenTelemetry

| Variável | Valor padrão no Compose |
| --- | --- |
| `OTEL_SERVICE_NAME` | `python-observability-api` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | `http://alloy:4318` |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | `http/protobuf` |
| `OTEL_TRACES_EXPORTER` | `otlp` |
| `OTEL_METRICS_EXPORTER` | `otlp` |
| `OTEL_LOGS_EXPORTER` | `otlp` |
| `OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED` | `true` |
| `OTEL_PYTHON_LOG_CORRELATION` | `true` |

Os valores podem ser sobrescritos no ambiente antes de executar o Compose.

## Endpoints e testes

```bash
curl -i http://localhost:8000/health
curl -i http://localhost:8000/users
curl -i -X POST http://localhost:8000/users/simulate-create
curl -i http://localhost:8000/latency/random
curl -i http://localhost:8000/errors/handled
curl -i http://localhost:8000/errors/unhandled
curl -i http://localhost:8000/external/simulate
```

Resultados esperados:

| Endpoint | Status | Finalidade |
| --- | ---: | --- |
| `GET /health` | 200 | Health check |
| `GET /users` | 200 | Listagem e log informativo |
| `POST /users/simulate-create` | 201 | Escrita simulada sem persistência |
| `GET /latency/random` | 200 | Latência aleatória de 100 a 5000 ms |
| `GET /errors/handled` | 400 | Erro controlado |
| `GET /errors/unhandled` | 500 | Exceção proposital capturada no trace |
| `GET /external/simulate` | 200 | Dependência fake com latência |

Após gerar tráfego, use o Explore do Grafana:

- Prometheus: procure métricas `http_server_*`;
- Loki: consulte `{exporter="OTLP"}` ou use o seletor disponível no label browser;
- Tempo: pesquise traces pelo serviço `python-observability-api`.

## Limitações intencionais

- sem banco de dados, autenticação ou persistência;
- sem chamadas externas reais;
- sem métricas customizadas ou spans manuais;
- sem arquitetura em camadas;
- configuração local e armazenamento em volumes Docker, inadequados para produção.
