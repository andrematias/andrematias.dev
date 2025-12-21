---
tags:
  - monitoring 
  - prometheus
author: André Matias 
date: 2025-12-21
draft: true
title: Entendendo PromQL queries do prometheus 
description:
    Entendendo um pouco os conceitos sobre as queries do prometheus
---

# Tipos de métricas 

## Counter
Faz somente incrementos e reset dos dados coletados. Ao serem criadas é incluso um sufixo `_total` no nome da metrica.

## Gauge
Pode fazer incremento e decrementos

## Histogram
Ideal para monitorar a duração de uma operação. Os dados são coletados de 3 formas que são utilizadas muitas vezes em conjuntos.

### Buckets
Armazena os dados em "acumuladores" por limites de tempo, chamados de buckets, por exemplo:
> 10 requests duraram 3s
> 70 requests duraram 5s

### Count
Faz o incremento de todos os dados coletados para ter o número total de tempo

### Sum


## Escrevendo queries
Uma query é feita usando o nome da métrica criada, o prometheus vai incluir sufixos em algumas queries.
Para fazer uma query com uma métrica do tipo Counter que faz a contagem de requisições http nas rotas da aplicação, podemos utilizar:

```promql
http_request_total
```

Esta é a forma original da métrica, sem filtros por `labels`, isso retorna todas as métricas coletadas, no formato: `metrica(labes) instant_vector`.

```
http_request_total(instance="fastapi-app:8000", job="fastapi-app", method="GET", path="/", status="200") 5
http_request_total(instance="fastapi-app:8000", job="fastapi-app", method="GET", path="/metrics", status="200") 323
```

### Filtrando por labels
Muitas métricas quando são planejadas, são criadas com etiquetas (`labels`) que facilitam e permitem filtrar os dados coletados de acordo com a label associada.

Seguindo o exemplo da metrica de contagem de requisições, se quisermos filtrar somente requisições que o status for igual a 200, fariamos:

```
http_request_total{status="200"}
```
É permitido utilizar uma combinação de labels para aprimorar o filtro.

```
http_request_total{status="200", path="/metris"}
```

## Time Series e dados recentes
O prometheus vai fazer scraping dos dados no endpoint configurado na aplicação, por exemplo, `/metrics`, no intervalo configurado na aplicação.
A cada scraping ele vai pegar o valor mais recente e salvar em sua base de dados time series.
Quando fazemos uma query sem especificar uma janela de coleta, ele retorna um vetor com os dados da time serie mais recentes. Se informarmos uma janela de tempo ele retorna o `range_vetor` com todos os dados coletados no intervalo informado.

```
http_request_total{status="200"}[5m]
```

# Funções mais comuns 

## rate(v range-vector)
Coleta dados por segundos e é mais suave
> rate = média de algo por segundos

## irate(v range-vector)
Coleta dados por segundos e não é suave

## increase(v renage-vector)
Coleta valores absolutos e é suave, bem parecido com o rate


## Referências
- [Prometheus Query Functions](https://prometheus.io/docs/prometheus/latest/querying/functions/)
- [PromQL Tutotial: A COMPLETE Guide to Prometheus Queries](https://www.youtube.com/watch?v=RC1ivt-ZN_U)
- [Understanding Counter Rates and Increases in PromQL | Reset Handling, Extrapolation, Edge Cases](https://www.youtube.com/watch?v=7uy_yovtyqw)