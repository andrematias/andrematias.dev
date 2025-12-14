---
tags:
  - akka
  - scala

date: 2024-09-05
title: Benefícios do Akka
description: 
  Queria entender um pouco mais sobre o porque o framework Akka é tão disseminado por quem utiliza a linguagem
  Scala, nestas notas tentei pegar o essencial para absorver os principais benefícios dele.
---

Akka é uma implementação do [Modelo de ator](/posts/20240902_Actor_Fundamentals) utilizando a JVM para construir um mecanismo de alta concorrência, distribuída, tolerante a falhas baseado em entrega de mensagens.

Este toolkit segue os princípios do manifesto reativo ([Reactive Manifesto](http://www.reactivemanifesto.org))
- Responsivo: O sistema sempre response com rapidez
- Resiliente: O sistema permanece responsivo durante falhas
- Elástico:  O sistema permanece responsivo quando a carga de trabalho varia
- Baseado em mensagens: A fundação do sistema para escalabilidade, resilience e responsividade. Não bloqueante e independente

![Objetivos do akka](/images/objetivos_do_akka.png)

Os objetivos do Akka é unificar a modelagem de programas para:
- Simplificar a concorrência
- Simplificar a distribuição
- Simplificar a tolerância a falhas
- Simplificar a escalabilidade
- Melhorar a performance

### Simplificar a concorrência
O modelo de atores nos permite escrever código em uma ilusão de thread unica, ou seja, nenhum estado é compartilhado entre threads. Por consequência, podemos descartar os problemas comuns de sincronização entre threads como: locks e semáforos.
Isto simplifica o design de software e elimina as dificuldades de debugs e diagnostico de bugs.

### Simplificar a distribuição
Akka pode simplificar a distribuição já que todo seu eco-sistema é distribuído por padrão, uma vez que segue o modelo de atores que interagem assincronamente sem bloquear através de mensagens. No ponto de vista do programa, não importa se as mensagens recebidas estão local ou remoto.
A implementação pode utilizar de uma a diversas JVM quando necessário escalar o processamento do trabalho recebido para os atores.

### Simplificar a tolerância a falhas
Akka desacopla a comunicação da captura de erros.

> Quando vamos a uma cafeteria e pedimos um café ao barista e o barista vai prepara-lo mas a maquina quebra ele não pode voltar e pedir para que você corrija o problema. Isso é o que ocorre quando é lançada uma exceção em um método que chamamos.

O mecanismo de troca de mensagens entre atores, permite que seja implementado compartimentos de falhas como [Bulkhead Pattern](https://stackoverflow.com/a/47256351) que resumidamente podemos entende-lo como um isolamento de pontos de falhas para que não se espalhe por outros pontos do sistema fazendo com que ele pare de funcionar.

Uma implementação de bulkhead no Akka é o [Circuit Breaker Pattern](https://doc.akka.io/docs/akka/2.5/common/circuitbreaker.html?language=scala) para minimizar a sobrecarga de pares de serviços cooperativos.

### Simplificar a escalabilidade
Com o Akka o recebimento de mensagens é transparente, não importa se elas estão vindo localmente ou através da rede global.
O sistema de atores pode ser particionado entre múltiplos nodes em um cluster que pode ser escalado dinamicamente para cima ou para baixo dependendo da carga a ser processada.

### Simplificar a melhoria de performance
A simplificação ao codificar programa concorrentes gera mais eficiência na utilização de CPUs, pois utilizam poucas threads para que o programa principal seja executado e realize assincronamente o envio de mensagens não bloqueantes ou I/O que estão desacoplados do programa principal.
O dispatcher do Akka pode suportar milhões de mensagens por segundo e faz isso de uma forma que fique fácil codificar processos paralelos simplesmente enviando mensagens entre múltiplos atores.

## Veja também
- [Modelo de ator](/posts/20240902_Actor_Fundamentals)
- [Conceitos do framework Akka](/posts/20240904_akka_concepts)