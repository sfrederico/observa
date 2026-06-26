# PRD — API Python mínima para PoC de Observabilidade

## 1. Visão geral

Este documento define os requisitos para criação de uma API Python mínima usada como aplicação de teste em uma PoC de monitoramento e observabilidade.

A aplicação não tem finalidade produtiva. Ela existe exclusivamente para gerar tráfego, logs, métricas automáticas, erros e traces que permitam validar uma stack de observabilidade baseada em:

* OpenTelemetry;
* Grafana Alloy;
* Prometheus;
* Loki;
* Tempo;
* Grafana.

A API deve ser simples, pequena e fácil de executar. O objetivo principal é validar o pipeline de observabilidade, não construir uma aplicação de negócio completa.

---

## 2. Objetivo do produto

Criar uma API Python mínima, instrumentada com OpenTelemetry por auto-instrumentation, capaz de simular comportamentos comuns de uma aplicação real:

* requisições bem-sucedidas;
* criação simulada de recurso;
* latência variável;
* erro tratado;
* erro não tratado;
* simulação de chamada externa;
* geração de logs;
* geração de traces automáticos;
* geração de métricas HTTP automáticas.

A aplicação deve enviar os sinais de observabilidade para o Grafana Alloy usando OTLP.

---

## 3. Contexto da PoC

A PoC tem como objetivo validar uma stack mínima de monitoramento e observabilidade antes de aplicar o conceito em uma API .NET real.

Como primeiro passo, será criada uma API Python por ser mais simples e familiar neste momento. Após a validação da stack, a mesma abordagem poderá ser replicada ou adaptada para uma API ASP.NET.

Esta aplicação deve funcionar como uma “aplicação de verdade”, mas sem complexidade produtiva. Ela deve ter endpoints claros, comportamentos previsíveis e capacidade de gerar cenários úteis para análise no Grafana.

---

## 4. Decisões técnicas principais

### 4.1 Framework

A aplicação deve utilizar **FastAPI**.

Motivos:

* código simples;
* baixo atrito para criação de endpoints;
* Swagger/OpenAPI automático;
* boa experiência para testes manuais;
* compatibilidade com OpenTelemetry auto-instrumentation;
* estrutura suficiente para parecer uma API real sem exigir arquitetura complexa.

### 4.2 Instrumentação

A aplicação deve utilizar **OpenTelemetry auto-instrumentation**.

A primeira versão não deve configurar OpenTelemetry manualmente no código da aplicação.

O objetivo é seguir uma abordagem semelhante à ideia de ferramentas como Sentry no onboarding inicial: instalar as dependências, configurar variáveis de ambiente e executar a aplicação instrumentada.

A execução deve ser feita com o comando `opentelemetry-instrument`, ou equivalente, dentro do Docker/container.

### 4.3 Exportação de dados

A aplicação deve exportar sinais para o Grafana Alloy via OTLP.

Fluxo esperado:

```text
FastAPI
  -> OpenTelemetry auto-instrumentation
    -> OTLP
      -> Grafana Alloy
        -> Prometheus / Loki / Tempo
          -> Grafana
```

### 4.4 Persistência

Não deve haver banco de dados nesta fase.

Dados simulados podem ser:

* fixos em memória;
* gerados dinamicamente no momento da requisição;
* descartados após a resposta.

### 4.5 Escopo produtivo

A aplicação não será usada em produção.

Portanto, não é necessário implementar:

* autenticação;
* autorização;
* banco de dados;
* migrations;
* cache;
* filas;
* mensageria;
* workers;
* arquitetura em camadas complexa;
* regras reais de negócio;
* testes extensos de domínio;
* CI/CD;
* Kubernetes;
* configuração produtiva de segurança.

---

## 5. Objetivos da aplicação

A API deve permitir responder às seguintes perguntas:

1. A aplicação consegue gerar traces automáticos via OpenTelemetry?
2. A aplicação consegue enviar telemetria via OTLP para o Alloy?
3. O Alloy consegue encaminhar os sinais para os backends corretos?
4. O Grafana consegue visualizar métricas, logs e traces dessa aplicação?
5. É possível identificar erro 500 nos traces e dashboards?
6. É possível analisar latência por endpoint?
7. É possível gerar logs suficientes para investigar uma requisição?
8. A experiência de instrumentação automática é simples o suficiente para ser demonstrada e comparada com uma futura API .NET?

---

## 6. Não objetivos

Esta PoC não deve tentar resolver:

* observabilidade completa de produção;
* modelagem de negócio real;
* métricas customizadas de negócio;
* spans manuais detalhados;
* rastreamento distribuído entre múltiplos serviços reais;
* persistência de dados;
* autenticação;
* autorização;
* deploy em nuvem;
* alta disponibilidade;
* escalabilidade horizontal;
* segurança produtiva.

---

## 7. Requisitos funcionais

### RF01 — Health check

A API deve expor um endpoint de health check.

Endpoint:

```http
GET /health
```

Comportamento esperado:

* retornar status HTTP 200;
* retornar uma resposta JSON simples;
* servir para validar se a aplicação está ativa.

Exemplo de resposta:

```json
{
  "status": "healthy",
  "service": "python-observability-api"
}
```

---

### RF02 — Consulta simulada de usuários

A API deve expor um endpoint para listar usuários fictícios.

Endpoint:

```http
GET /users
```

Comportamento esperado:

* retornar uma lista fixa ou simulada de usuários;
* não exigir parâmetros;
* gerar uma requisição HTTP observável;
* retornar status HTTP 200;
* registrar log informativo usando o `logging` padrão do Python.

Exemplo de resposta:

```json
{
  "items": [
    {
      "id": 1,
      "name": "Alice"
    },
    {
      "id": 2,
      "name": "Bruno"
    }
  ]
}
```

---

### RF03 — Criação simulada de usuário

A API deve expor um endpoint que simule a criação de um usuário.

Endpoint:

```http
POST /users/simulate-create
```

Comportamento esperado:

* não exigir body;
* gerar internamente os dados do usuário;
* retornar status HTTP 201;
* registrar log de início da operação;
* registrar log de sucesso da operação;
* permitir observar uma operação de escrita simulada.

Exemplo de resposta:

```json
{
  "id": 3,
  "name": "Generated User",
  "status": "created"
}
```

Observação:

Este endpoint não precisa persistir o usuário criado. A resposta pode ser gerada dinamicamente.

---

### RF04 — Simulação de latência

A API deve expor um endpoint para simular lentidão variável.

Endpoint:

```http
GET /latency/random
```

Comportamento esperado:

* gerar um atraso artificial antes de responder;
* o atraso pode ser aleatório, por exemplo entre 100 ms e 5000 ms;
* retornar status HTTP 200;
* registrar log informando o tempo de atraso simulado;
* permitir análise de latência no Grafana/Prometheus/Tempo.

Exemplo de resposta:

```json
{
  "status": "completed",
  "delay_ms": 2400
}
```

---

### RF05 — Erro tratado

A API deve expor um endpoint que simule um erro conhecido e tratado pela aplicação.

Endpoint:

```http
GET /errors/handled
```

Comportamento esperado:

* simular uma falha esperada;
* registrar log de warning ou error;
* retornar resposta controlada;
* retornar status HTTP 400 ou 422;
* não gerar exceção não tratada.

Exemplo de resposta:

```json
{
  "error": "simulated_validation_error",
  "message": "This is a controlled error generated by the API"
}
```

---

### RF06 — Erro não tratado

A API deve expor um endpoint que gere uma exceção real não tratada.

Endpoint:

```http
GET /errors/unhandled
```

Comportamento esperado:

* lançar uma exception real;
* retornar status HTTP 500;
* permitir que a auto-instrumentação capture o erro no trace;
* permitir testar dashboards, logs e alertas baseados em erro 5xx.

Observação:

Este endpoint existe propositalmente para gerar falha. Não deve ser tratado com resposta customizada.

---

### RF07 — Simulação de chamada externa

A API deve expor um endpoint que simule uma dependência externa.

Endpoint:

```http
GET /external/simulate
```

Comportamento esperado:

* simular uma chamada externa;
* inicialmente, a simulação pode ocorrer dentro da própria aplicação;
* pode gerar sucesso, erro ou latência simulada;
* registrar logs informando o resultado da simulação;
* retornar status HTTP 200 em caso de sucesso;
* permitir futura evolução para uma chamada HTTP real para outro container fake.

Exemplo de resposta:

```json
{
  "provider": "fake-external-provider",
  "status": "success",
  "duration_ms": 850
}
```

---

## 8. Requisitos de observabilidade

### RO01 — OpenTelemetry por auto-instrumentation

A aplicação deve ser executada com OpenTelemetry auto-instrumentation.

Não deve haver configuração manual extensa de OpenTelemetry dentro do código da aplicação.

A configuração deve ser feita principalmente por:

* dependências instaladas;
* comando de inicialização;
* variáveis de ambiente.

---

### RO02 — Exportação OTLP

A aplicação deve exportar telemetria via OTLP para o Grafana Alloy.

O endpoint OTLP deve ser configurável por variável de ambiente.

Exemplo esperado:

```env
OTEL_EXPORTER_OTLP_ENDPOINT=http://alloy:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
```

---

### RO03 — Nome do serviço

A aplicação deve configurar o nome do serviço via variável de ambiente.

Exemplo:

```env
OTEL_SERVICE_NAME=python-observability-api
```

---

### RO04 — Traces

A aplicação deve gerar traces automáticos para requisições HTTP.

Cenários mínimos que devem aparecer nos traces:

* `GET /health`;
* `GET /users`;
* `POST /users/simulate-create`;
* `GET /latency/random`;
* `GET /errors/handled`;
* `GET /errors/unhandled`;
* `GET /external/simulate`.

O endpoint `/errors/unhandled` deve permitir visualizar erro no trace.

O endpoint `/latency/random` deve permitir visualizar maior duração da requisição.

---

### RO05 — Métricas

Nesta fase, a aplicação deve depender de métricas automáticas geradas pela instrumentação HTTP.

Não é obrigatório implementar métricas customizadas.

Métricas esperadas, conforme suporte da instrumentação e pipeline:

* quantidade de requisições HTTP;
* duração das requisições;
* status codes;
* método HTTP;
* rota/endpoint.

Métricas customizadas como `users_created_total` ou `simulated_errors_total` ficam fora do escopo da primeira fase.

---

### RO06 — Logs

A aplicação deve usar o módulo padrão `logging` do Python para registrar eventos relevantes.

Logs mínimos esperados:

* início da listagem de usuários;
* criação simulada de usuário;
* latência simulada;
* erro tratado;
* simulação de chamada externa;
* exceção não tratada, quando aplicável.

A exportação dos logs deve ser feita via OpenTelemetry auto-instrumentation, quando configurada.

Caso a exportação automática de logs exija configuração adicional, o Codex deve manter a solução simples e documentar no README quais variáveis ou comandos são necessários.

---

### RO07 — Correlação

Quando possível, os logs devem conter correlação com a requisição ou trace.

Como esta fase prioriza auto-instrumentation e baixo código, não é obrigatório implementar correlação manual.

Caso a correlação automática esteja disponível via OpenTelemetry, ela deve ser preservada.

---

## 9. Requisitos técnicos

### RT01 — Linguagem

Usar Python em versão moderna.

Preferência:

```text
Python 3.12+
```

---

### RT02 — Framework

Usar FastAPI.

---

### RT03 — Servidor

Usar Uvicorn para execução da API.

---

### RT04 — Dependências esperadas

O projeto deve incluir dependências necessárias para:

* FastAPI;
* Uvicorn;
* OpenTelemetry distro;
* OpenTelemetry OTLP exporter;
* OpenTelemetry instrumentation para FastAPI;
* OpenTelemetry instrumentation para logging;
* OpenTelemetry instrumentation para bibliotecas HTTP, caso usadas.

Dependências prováveis:

```txt
fastapi
uvicorn
opentelemetry-distro
opentelemetry-exporter-otlp
opentelemetry-instrumentation-fastapi
opentelemetry-instrumentation-logging
opentelemetry-instrumentation-requests
requests
```

O Codex pode ajustar a lista se identificar dependências mais adequadas, desde que preserve o objetivo de auto-instrumentation.

---

### RT05 — Estrutura simples de projeto

A estrutura deve ser simples.

Sugestão:

```text
.
├── app/
│   ├── Dockerfile
│   ├── main.py
│   ├── PRD.md
│   └── requirements.txt 
├── docker-compose.yml
└── README.md
```

Se necessário, o Codex pode adicionar arquivos auxiliares, mas deve evitar arquitetura excessiva.

Não criar camadas como:

* repository;
* service;
* usecase;
* domain;
* infrastructure.

A menos que isso seja estritamente necessário, manter a aplicação pequena.

---

### RT06 — Docker

A aplicação deve ser executável via Docker.

O Dockerfile deve:

* instalar dependências;
* copiar o código da aplicação;
* expor a porta da API;
* executar a API com `opentelemetry-instrument`;
* permitir configuração por variáveis de ambiente.

---

### RT07 — Docker Compose

O projeto deve conter um `docker-compose.yml` simples para executar a API.

O compose pode considerar que a stack de observabilidade já existe em outro compose/rede, mas deve deixar o endpoint do Alloy configurável.

Exemplo de intenção:

```env
OTEL_EXPORTER_OTLP_ENDPOINT=http://alloy:4318
```

Caso o serviço `alloy` não esteja no mesmo compose, o README deve explicar como ajustar o endpoint.

---

### RT08 — Swagger

Como a aplicação usa FastAPI, a documentação Swagger deve estar disponível automaticamente em:

```http
/docs
```

A documentação OpenAPI deve estar disponível automaticamente em:

```http
/openapi.json
```

---

## 10. Variáveis de ambiente esperadas

O projeto deve suportar configuração por variáveis de ambiente.

Variáveis mínimas:

```env
OTEL_SERVICE_NAME=python-observability-api
OTEL_EXPORTER_OTLP_ENDPOINT=http://alloy:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_TRACES_EXPORTER=otlp
OTEL_METRICS_EXPORTER=otlp
OTEL_LOGS_EXPORTER=otlp
OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED=true
```

Variáveis adicionais podem ser usadas conforme necessidade.

---

## 11. Critérios de aceite

A implementação será considerada concluída quando:

1. A API subir localmente via Docker ou Docker Compose.
2. O Swagger estiver acessível em `/docs`.
3. Todos os endpoints definidos estiverem disponíveis.
4. `GET /health` retornar HTTP 200.
5. `GET /users` retornar usuários simulados.
6. `POST /users/simulate-create` criar resposta simulada sem exigir body.
7. `GET /latency/random` gerar atraso artificial.
8. `GET /errors/handled` retornar erro controlado.
9. `GET /errors/unhandled` gerar HTTP 500 propositalmente.
10. `GET /external/simulate` simular dependência externa.
11. A aplicação executar com `opentelemetry-instrument`.
12. O endpoint OTLP ser configurável por variável de ambiente.
13. A aplicação gerar traces automáticos para requisições HTTP.
14. A aplicação gerar logs usando o logging padrão do Python.
15. O README explicar como executar e testar a API.
16. O código permanecer simples e compatível com o objetivo de PoC.

---

## 12. Cenários de teste manual

Após subir a aplicação, testar os seguintes cenários:

### Cenário 1 — API viva

Requisição:

```http
GET /health
```

Resultado esperado:

* HTTP 200;
* resposta JSON simples;
* trace de requisição no backend de traces.

---

### Cenário 2 — Consulta bem-sucedida

Requisição:

```http
GET /users
```

Resultado esperado:

* HTTP 200;
* lista de usuários;
* log informativo;
* métrica HTTP;
* trace automático.

---

### Cenário 3 — Criação simulada

Requisição:

```http
POST /users/simulate-create
```

Resultado esperado:

* HTTP 201;
* usuário gerado;
* logs de operação;
* trace automático.

---

### Cenário 4 — Latência

Requisição:

```http
GET /latency/random
```

Resultado esperado:

* HTTP 200;
* atraso artificial;
* duração visível nas métricas/traces;
* log com tempo simulado.

---

### Cenário 5 — Erro tratado

Requisição:

```http
GET /errors/handled
```

Resultado esperado:

* HTTP 400 ou 422;
* resposta controlada;
* log de warning/error;
* trace sem exception não tratada.

---

### Cenário 6 — Erro 500

Requisição:

```http
GET /errors/unhandled
```

Resultado esperado:

* HTTP 500;
* exception visível;
* trace marcado com erro;
* logs de falha.

---

### Cenário 7 — Dependência externa simulada

Requisição:

```http
GET /external/simulate
```

Resultado esperado:

* HTTP 200 em caso de sucesso simulado;
* latência simulada;
* logs informando provider fake;
* trace automático da requisição.

---

## 13. README esperado

O projeto deve conter um README com, no mínimo:

1. Objetivo da aplicação.
2. Aviso de que é uma PoC, não produção.
3. Stack usada.
4. Como instalar localmente, se aplicável.
5. Como executar com Docker.
6. Como executar com Docker Compose.
7. Como acessar o Swagger.
8. Lista dos endpoints disponíveis.
9. Variáveis de ambiente de OpenTelemetry.
10. Como apontar a aplicação para o Grafana Alloy.
11. Como gerar cenários de erro e latência.
12. Limitações conhecidas.

---

## 14. Limitações conhecidas

A primeira versão aceita as seguintes limitações:

* sem banco de dados;
* sem autenticação;
* sem métricas customizadas;
* sem spans manuais;
* sem domínio real;
* sem chamadas externas reais;
* sem testes automatizados extensos;
* sem preocupação com performance;
* sem preocupação com segurança produtiva.

Essas limitações são intencionais para manter a PoC simples.

---

## 15. Possíveis evoluções futuras

Após validar a primeira fase, considerar:

### Fase 2 — Instrumentação manual mínima

Adicionar spans manuais em apenas um endpoint para comparar com a auto-instrumentation.

Exemplo:

```text
POST /users/simulate-create
  -> generate_fake_user
  -> validate_user
  -> save_user_fake
```

Objetivo:

* demonstrar diferença entre traces automáticos e traces enriquecidos.

---

### Fase 3 — Métricas customizadas

Adicionar métricas customizadas, como:

```text
users_created_total
simulated_errors_total
external_provider_failures_total
```

Objetivo:

* avaliar valor de métricas de negócio/operação.

---

### Fase 4 — Serviço externo fake

Adicionar um segundo container simulando uma dependência externa real.

Objetivo:

* validar trace distribuído;
* validar falhas entre serviços;
* validar latência externa.

---

### Fase 5 — API .NET mínima

Criar uma API ASP.NET mínima com comportamento semelhante.

Objetivo:

* comparar complexidade da instrumentação Python versus .NET;
* validar se a mesma stack atende o cenário real desejado.

---

## 16. Instruções específicas para o Codex

Ao implementar este PRD, siga as orientações abaixo:

1. Priorize simplicidade.
2. Não transforme a PoC em uma aplicação produtiva.
3. Não adicione banco de dados.
4. Não adicione autenticação.
5. Não crie arquitetura em camadas complexa.
6. Não configure OpenTelemetry manualmente no código, exceto se for estritamente necessário para viabilizar a execução.
7. Use auto-instrumentation com `opentelemetry-instrument`.
8. Configure OpenTelemetry por variáveis de ambiente.
9. Mantenha o código da aplicação pequeno.
10. Documente qualquer decisão técnica relevante no README.
11. Caso alguma dependência ou variável precise ser alterada para funcionamento correto, ajuste e explique.
12. Caso logs via OpenTelemetry exijam configuração extra, mantenha a solução simples e documentada.
13. Não implemente funcionalidades fora do escopo sem necessidade.
14. Não adicionar métricas customizadas nesta primeira fase.
15. Não adicionar spans manuais nesta primeira fase.
16. A aplicação deve ser fácil de executar e fácil de demonstrar.

---

## 17. Resultado esperado

Ao final da implementação, deve existir uma API Python mínima que possa ser usada para validar a stack de observabilidade.

A aplicação deve permitir que alguém acesse o Swagger, execute endpoints simples e observe no Grafana:

* requisições HTTP;
* latência;
* status codes;
* erro 500;
* logs;
* traces automáticos;
* comportamento básico do pipeline OpenTelemetry -> Alloy -> Grafana stack.

O sucesso da PoC não depende de uma aplicação complexa. Depende de uma aplicação simples, executável e capaz de gerar sinais úteis para validar a infraestrutura de monitoramento.
