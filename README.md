# Python Logs API

PoC mínima para gerar, exportar e visualizar logs de uma aplicação Python com FastAPI. O projeto é focado somente em logs nesta fase e não deve ser usado como base de produção.

## Objetivo

Executar uma aplicação Python simples que gera logs com o `logging` padrão, usar OpenTelemetry auto-instrumentation para exportar esses logs via OTLP, receber os logs no Grafana Alloy, armazená-los no Loki e consultá-los no Grafana.

Métricas e traces estão fora do escopo desta fase. A escolha por OpenTelemetry mantém a instrumentação desacoplada da aplicação e facilita adicionar novos sinais futuramente.

## Stack

- Python 3.12, FastAPI e Uvicorn
- OpenTelemetry Python auto-instrumentation
- Grafana Alloy como receptor OTLP e encaminhador de logs
- Loki para armazenamento e consulta de logs
- Grafana para visualização
- Docker Compose para execução local

## Arquitetura

```text
Aplicação Python
  -> OpenTelemetry auto-instrumentation
    -> OTLP logs
      -> Grafana Alloy
        -> Loki
          -> Grafana
```

A aplicação não usa bibliotecas específicas de Loki e não envia logs diretamente para o Loki. Ela apenas emite logs com o `logging` padrão do Python; o OpenTelemetry captura e exporta os registros para o Alloy.

## Executar com Docker Compose

Pré-requisito: Docker com Docker Compose.

```bash
docker compose up --build
```

Serviços disponíveis:

- Aplicação: <http://localhost:8000>
- Swagger da API: <http://localhost:8000/docs>
- Grafana: <http://localhost:3000>
- Alloy: <http://localhost:12345>
- Loki: <http://localhost:3100/ready>

Para encerrar mantendo os volumes:

```bash
docker compose down
```

Para encerrar e remover os volumes da PoC:

```bash
docker compose down -v
```

## Configuração OpenTelemetry

A aplicação é executada com `opentelemetry-instrument`. No Compose, os sinais são configurados por variáveis de ambiente:

```env
OTEL_SERVICE_NAME=python-logs-api
OTEL_EXPORTER_OTLP_ENDPOINT=http://alloy:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_LOGS_EXPORTER=otlp
OTEL_TRACES_EXPORTER=none
OTEL_METRICS_EXPORTER=none
OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED=true
```

O Alloy expõe OTLP em `4317` e `4318`, recebe somente logs e os encaminha para o Loki.

## Acessar a aplicação

Use o Swagger em <http://localhost:8000/docs> ou execute chamadas com `curl`.

```bash
curl -i http://localhost:8000/health
curl -i http://localhost:8000/success
curl -i http://localhost:8000/warning
curl -i http://localhost:8000/latency
curl -i http://localhost:8000/error/unhandled
```

## Acessar o Grafana

Abra <http://localhost:3000>. No primeiro acesso, use `admin` / `admin` e defina uma nova senha quando solicitado.

O datasource `Loki` é provisionado automaticamente e definido como padrão.

## Consultar logs no Grafana

1. Acesse `Explore`.
2. Selecione o datasource `Loki`.
3. Execute uma consulta LogQL:

```logql
{service_name="python-logs-api"}
```

Consultas úteis:

```logql
{service_name="python-logs-api"} |= "duration_ms"
{service_name="python-logs-api"} |= "WARNING"
{service_name="python-logs-api"} |= "ERROR"
```

Depois de chamar os endpoints, aguarde alguns segundos para o OpenTelemetry exportar os logs e o Alloy encaminhá-los ao Loki.

## Endpoints para gerar logs

| Endpoint | Status esperado | Finalidade |
| --- | ---: | --- |
| `GET /health` | 200 | Health check e log informativo |
| `GET /success` | 200 | Resposta de sucesso com log informativo |
| `GET /warning` | 400 | Warning controlado |
| `GET /latency` | 200 | Latência simulada entre 100 e 5000 ms |
| `GET /error/unhandled` | 500 | Exceção proposital para gerar log de erro |

## Limitações da PoC

- Sem banco de dados.
- Sem autenticação.
- Sem persistência de negócio.
- Sem chamadas externas reais.
- Sem pipeline ativo de métricas ou traces.
- Sem garantias de retenção, segurança, disponibilidade ou escala.
- Configuração local com volumes Docker, inadequada para produção.

Este projeto é apenas uma PoC de logs para desenvolvimento e estudo.
