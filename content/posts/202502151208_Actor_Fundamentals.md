---
date: 2024-09-02 17:00:00
title: Fundamentos do modelo de ator
toc: true
tags:
  - akka
  - scala
  - software-design
description: Um estudo basico sobre a arquitetura baseada em modelo de atores utilizando Scala e o framework Akka
---

O modelo de ator, é um modelo criado para suportar operações concorrentes e distribuídas de aplicações

## Influencias do modelo

O modelo foi desenvolvido com influencias da física, principalmente da teoria da relatividade de Einstein e também pela física quântica.

Além da física, no meio computacional, teve bastante influencia o lambda cálculos de [Alonzo Church](https://en.wikipedia.org/wiki/Alonzo_Church) com a formalização de passagem de mensagens ([Earliest Message passing formalism](https://en.wikipedia.org/wiki/Message_passing)), onde um programa pode invocar um comportamento por meio de uma mensagem.

Este modelo também foi baseado em conceitos chaves de objetos retirados de linguagens de programação como [Simula-67](https://pt.wikipedia.org/wiki/Simula_67) a primeira linguagem orientada a objetos, projetada par apoiar a simulação de eventos discretos e por Smalltack-72 que fazia a comunicação entre objetos enviando mensagens tipadas dinamicamente.

## Influencias do modelo de atores na computação

Com o advento da expansão da computação paralela, dos micro processadores multi core o modelo foi mais explorado para que fosse evitado os problemas mais comuns em programação paralela e multi-threads.

O modelo ganhou começou a ser utilizado em linguagens de programações já consolidadas no mercado, como: Erlang e Swift e foi implementadas em frameworks como Akka que utiliza a JVM do Java

## Conceitos fundamentais de um modelo de ator

Um ator é uma **entidade computacional** que tem um **endereço** que outros atores utilizam para enviar mensagens para ele, que também pode **receber** e **processar** mensagens **uma por vez** e pode se comunicar com outros atores **enviando mensagens** a eles.

Quando um ator processa uma mensagem recebida, ele pode:

1. Mudar seu próprio estado
2. Mudar seu comportamento para receber as próximas mensagens
3. Criar um novo ator
4. Enviar ou agendar mensagens para ele mesmo ou outro atores
5. Se finalizar

Quando um ator não esta processando uma mensagem ele esta no modo aguardando.

## Benefícios de um modelo de ator

- Ele é conceitualmente simples, onde ajuda a resolver problemas complexos.
- Permite uma alta eficiência e uma concorrência segura
- Permite alto paralelismo
- Facilita a criação de software resiliente
- Facilita a criação de sistemas distribuídos

## Anatomia de um ator

Um ator encapsula um estado e um comportamento, ele também tem uma [[Filas com lista dinamicamente encadeadas|fila]] (normalmente FIFO, mas pode ter outras implementações de filas) onde armazena as mensagens a serem processadas. Quando o ator processa uma mensagem ele pode alterar seu estado e/ou seu comportamento.

### O que é uma mensagem

Uma mensagem é uma entrada ou saídas de dados para o ator, esta deve ser imutável normalmente um record, struct ou um objeto de valor que é aceita pelo design da API do ator.

Mensagens são comumente implementadas no [[1725301003-UOXI|Domain Driven Design]] quando segue o design de uma entidade, por exemplo:

- Command
- Event
- Query
- Result

### Caixa de mensagens (Mailbox)

A caixa de mensagens utilizada pelo ator para enfileirar as mensagens a serem processadas tem algumas configurações para:

- Máximo de mensagens
- Priorização de mensagens
- Ordem de entrega
- Comportamento de enfileiramento (bloqueante / não bloqueante)

### Estado

O estado gerenciado pelo ator pode ser mutável, porém, de forma segura seguindo algumas politicas como:

- O próprio ator pode mudar seu estado somente durante o processamento da mensagem recebida.
- Outros atores só podem requisitar o estado do ator receptor enviando mensagens a ele.

O estado do ator normalmente é ephemeral (armazenado temporariamente em memória) ou opcionalmente persistido

### Comportamento

Um comportamento é como o ator processa suas mensagens. Um ator pode ter múltiplos comportamentos, porém somente um ativo por vez, onde o comportamento ativo processa a próxima mensagem.
O comportamento normalmente é implementado como uma função ou método type-safe, ou seja, que deve validar a mensagem.

No momento de processamento da mensagem o comportamento pode:

- Atualizar o estado do ator
- Criar um ou mais atores
- Enviar uma mensagem para qualquer ator que o ator atual tenha referencia via mensagem processada ou seu estado
- Mudar o comportamento ativo para processar a próxima mensagem

## Mutação de um ator

### Mutação do estado

Um ator pode ter um estado mutável, este estado pode ser diferente a cada comportamento ativo no mesmo ator, onde pode ser utilizado qualquer estrutura para mantê-lo, desde que respeite seu acesso e mutabilidade definidos pelo design do modelo de atores.
O estado nunca deve ser compartilhado entre atores diretamente, portanto o ator A não pode interagir por outros meios com o estado do ator B a não ser pelas mensagens.

### Mutação do comportamento

Quando o ator esta processando uma mensagem ele pode definir o comportamento ativo para processar as mensagens subsequentes. Esta mutação pode permitir implementações como:

- Representar uma máquina de estados finitos
- Responder diferentemente a mensagens recebidas
- Atender as mudanças baseada em um sistema de notificações
- Responder a queries diferentes em diferentes horas do dia

### Criação de novos atores

O ator pode criar atores filhos que formam uma hierarquia de pai e filho e se comunicar entre eles.
Isto nos permite:

- Distribuir a complexidade de processamento entre atores filhos
- Paralelizar o processamento das mensagens entre os atores filhos
- Distribuir a computação entre a rede para resiliência da aplicação

### Finalização do ator

Uma finalização é quando um ator elimina seu estado e comportamento e deixa o sistema, liberando memória e eliminando todos os resquícios que gerou durante seu tempo de vida.

- A finalização de um ator é esperada por seu design. O ator pode somente finalizar a si próprio ou ao seus filhos, tendo como princípio que ao se auto finalizar seus filhos também se finalizarão.
- Um ator pode sugerir através de mensagens que outros atores se finalizem, porém a decisão é do ator que esta recebendo a mensagem se atende a requisição ou não.
- Alguns sistemas de atores pode permitir que outros atores sejam notificados sobre a finalização de outros atores.

### Envio de mensagens

Este é uma das principais tarefas de um ator. Enviar uma mensagem para outro ator requer que este ator tenha um endereço da caixa de mensagens recebido via mensagem que esta em processamento ou armazenado no estado.
Esta mensagem pode ser enviada para o próprio ator, para seus filhos ou pode ser agendada para ser enviada posteriormente.

## Sistemas de atores (Actor Systems)

### O que é um sistema de atores

Este é uma entidade de gerenciamento de colaboração entre a hierarquia de atores, um pool de threads utilizadas para processar os trabalhos, as caixas de mensagens dos atores e configuração do ambiente.

> An ActorSystem is a heavyweight structure that will allocate 1...N Threads, so create one per logical application.
> --- Akka Documentation

Com um sistema de ator é possível dividir o trabalho em partes menores, distintas e gerenciáveis para executá-los em paralelo e implementar passos de um processo ou sagas.

Podemos definir múltiplos sistemas de atores desde que cada um cuide de diferentes categorias de trabalho, normalmente um ator por micro serviço.

O sistema precisa de um pool de threads para que as mensagens sejam entregues para os atores, normalmente a quantidade de threads no pool não deveria passar do dobro do número de cores do CPU

Este sistema fornece funcionalidades para os atores como:

- Enfileirar, Desenfileirar e entregar mensagens
- Agendar mensagens para serem entregues mais tarde
- Fornecer um serviço de Logging
- Acesso ao ambiente e configurações
- Comunicação transparente entre os sistemas de atores