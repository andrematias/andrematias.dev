---
tags:
  - akka
  - scala
date: 2024-09-04
title: Conceitos do framework Akka
description:
Neste artigo explorei um pouco os conceitos do framework Akka
---

## ActorSystem
Akka fornece uma implementação de ActorSystem como um objeto que implementa os conceitos de sistema de atores. Este representa uma aplicação ou serviço para colaboração entre os atores e gerenciar seus namespaces e endereços, assim como suas caixas de mensagens e facilitar a transmissão de mensagens entre atores.

Além de gerenciar as mensagens e recursos que os atores utilizarão, também faz o gerenciamento de threads e processos de disparos. Utiliza um serviço de scheduler para envio das mensagens.

O ActorSystem pode prover um publicador e assinante de eventos para log de mensagem, dead letters e outros pontos que podem ser de interesse de um ator ou processo interno da aplicação.

### ActorContext
Um sistema de ator não vive isolado, portanto é fornecido um contexto (ActorContext) que contém informações do contexto onde o sistema de ator esta inserido para que possa ser utilizado na execução.

O ActorContext fornece informações contextual ao ator como suas próprias informações de contexto, acesso a um sistema de ator que ele pertence e acesso ao sistema de log do Akka.

O contexto permite a um ator a completar ações como:
- Spawn (Gerar) um ator filho
- Watch (Observar) atores filhos
- Schedule (Agendar) a finalização de atores filhos ou sua própria finalização
- Schedule (Agendar) mensagens para outros atores

### Actor Supervision

Um ator parente pode supervisionar seus filhos através de um Supervisor fornecido pelo ActorSystem, podendo definir como será a resposta a exceções que ocorrem nos seus filhos.
Existem diversas estratégias de supervisão para ser escolhida. A estratégia padrão é finalizar o ator filho que esta falhando.
Outras são:
- Restart
- Resume
- Restart with limit
Um supervisor também contém sinais de processamento:
- PresStart
- PosStop
- PreRestart
- ChildFailed

### ActorRef
Um ator tem um **endereço único** utilizado para endereçar mensagens a ele criando um encapsulamento e habilitando uma comunicação sem conhecer a origem do ator, este é o ActorRef.

![Comunicação](/images/comunicacao_akka.png)


#### Enfileiramento e agendamento de mensagens para um ator

![Enfileiramento](/images/enfileiramento_akka.png)

Cada ator tem uma caixa de mensagens e um despachante de mensagens. O ActorRef enfileira mensagens que são recebidas dentro da caixa de mensagens e o despachante agenda uma thread para processar uma caixa de mensagens que contém mensagens. Porém a caixa de mensagens e o despachante não fazem parte do ator em si, mas sim gerenciado pelo ActorSystem.

#### Desenfileiramento e despacho de mensagens para um ator

![Desenfileiramento](/images/desenfileiramento_akka.png)

Uma mensagem é desenfileirada e despachada para o ator por processos que podem ocorrer em threads diferentes em momentos diferentes, mas o ator sempre processa-rá uma mensagem de cada vez, mantendo a ilusão de operar com somente uma thread.

### Typed vs Classic Actors
Akka faz a implementação de dois tipos de atores:
- Clássicos: Onde as mensagens esperadas pelo ator não tem um tipo definido, podendo ser qualquer coisa.
- Typed: São atores que tem pré-definido qual o tipo de mensagem aguardam, garantindo que não recebe nada além do que foi estipulado em sua concepção.



## Veja também
- [Fundamentos do modelo de ator](/posts/20240902_Actor_Fundamentals)
- [Benefícios do Akka](/posts/20240905_Beneficios_do_Akka)