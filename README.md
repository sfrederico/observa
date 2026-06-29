# PoC Observabilidade: Logs

PoC de uma stack mínima de monitoramento e observabilidade, neste primeiro estágio focada exclusivamente em **logs**.

A API Python com FastAPI existe apenas como aplicação de apoio para gerar logs de teste. O foco do projeto é validar o fluxo de geração, coleta, armazenamento, consulta e visualização de logs usando OpenTelemetry, Grafana Alloy, Loki e Grafana.

## Objetivo

Executar uma aplicação mínima que gera logs com o `logging` padrão do Python, capturar esses logs com OpenTelemetry auto-instrumentation, enviá-los via OTLP para o Grafana Alloy, armazená-los no Loki e visualizá-los no Grafana.

Métricas e traces estão fora do escopo desta fase da PoC.

## Stack

- OpenTelemetry com autoinstrumentação
- Grafana Alloy para receber logs via OTLP e encaminhá-los
- Grafana Loki para armazenamento e consulta dos logs
- Grafana para visualização dos logs
- API mínima em Python 3.12, FastAPI e Uvicorn para gerar logs de teste
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

A aplicação não envia logs diretamente para o Loki. Ela apenas emite logs com o `logging` padrão do Python; o OpenTelemetry captura e exporta os registros para o Alloy, que encaminha os logs para o Loki.

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

## Gerar logs

Use o Swagger em <http://localhost:8000/docs> e execute os endpoints disponíveis. Também é possível chamar a API com `curl`:

```bash
curl -i http://localhost:8000/health
curl -i http://localhost:8000/success
curl -i http://localhost:8000/warning
curl -i http://localhost:8000/latency
curl -i http://localhost:8000/error/unhandled
```

Depois de chamar os endpoints, aguarde alguns segundos para o OpenTelemetry exportar os logs e o Alloy encaminhá-los ao Loki.

## Visualizar logs

Abra <http://localhost:3000>. No primeiro acesso, use `admin` / `admin` e defina uma nova senha quando solicitado.

O datasource `Loki` e o dashboard `Logs` são provisionados automaticamente. Para visualizar os logs, acesse o dashboard `Logs` no Grafana.

Também é possível consultar os logs em `Explore`, selecionando o datasource `Loki` e executando uma consulta LogQL:

```logql
{service_name="python-logs-api"}
```

## Endpoints para gerar logs

| Endpoint | Status esperado | Finalidade |
| --- | ---: | --- |
| `GET /health` | 200 | Health check e log informativo |
| `GET /success` | 200 | Resposta de sucesso com log informativo |
| `GET /warning` | 400 | Warning controlado |
| `GET /latency` | 200 | Latência simulada entre 100 e 5000 ms |
| `GET /error/unhandled` | 500 | Exceção proposital para gerar log de erro |

## Configuração OpenTelemetry

A aplicação é executada com `opentelemetry-instrument`. No Compose, os sinais são configurados por variáveis de ambiente para exportar somente logs:

```env
OTEL_SERVICE_NAME=python-logs-api
OTEL_EXPORTER_OTLP_ENDPOINT=http://alloy:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_LOGS_EXPORTER=otlp
OTEL_TRACES_EXPORTER=none
OTEL_METRICS_EXPORTER=none
OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED=true
```

O Alloy expõe OTLP em `4317` e `4318`, recebe os logs e os encaminha para o Loki.

## Limitações da PoC

- Sem banco de dados.
- Sem autenticação.
- Sem persistência de negócio.
- Sem chamadas externas reais.
- Sem pipeline ativo de métricas ou traces.
- Sem garantias de retenção, segurança, disponibilidade ou escala.
- Configuração local com volumes Docker, inadequada para produção.

Este projeto é apenas uma PoC local de observabilidade para desenvolvimento e estudo.
