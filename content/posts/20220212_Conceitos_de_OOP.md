---
title: Conceitos de OOP
date: 2022-02-12
author: André Matias
tags:
  - programming
  - oop
toc: true
---

## Pensando na perspectiva de orientação á objeto

Orientação a objeto pode ser encarado como um modo de pensar e abstrair o que esta a nossa volta. Podemos quebrar as grandes coisas em pequenas e analisa-las, segrega-las e definitivamente entende-las como peças individuais.

Isso nos permite definir cada parte e descrever seus atributos, mapeando-os em nossa mente e compondo um objeto.


## Conceito mental

Uma vez com o mapeamento dos objetos em nossa mente, podemos montar um guia de instruções que utilizam esses objetos para que eles possam se relacionar e produzir novos resultados ou ser alterados.

Sem um compreendimento do que cada objeto representa não temos como explicar nossos objetivos e muito menos informar ao computador quais são as etapas que compõem o programa que resolve-rá o problema proposto.

Quando temos um mapa mental criado, o processo de ilustrar a outras pessoas (ou computadores) o que deve ser produzido como resultado ou como um determinado objeto do programa deve se comportar e se relacionar com outros ficará facilitado. Caso contrário cada um pode criar concepções diferentes da sua e interpretar, ou pior, projetar errado seu software.

> Concepts are important to object-oriented thinking because if you cannot form and maintain an understanding of objects in your mind, you cannot extract details about what they represent and relate to, and you cannot understand their interrelations. [Amini, Kanran][1]


## Atributos do objeto

Um objeto pode carregar seus valores em campos, por exemplo, na frase:

"Um vazo tem a cor azul"

Podemos concluir que o objeto 'vazo' tem um atributo (campo) cor e seu valor é azul. Este objeto carrega um estado próprio, portanto ele é mutável `mutable`, porém há objetos que não contém atributos, somente comportamentos, sendo assim se torna um objeto sem estado `stateless`.


## Domínio

Todo software é escrito para resolver um determinado problema. Este problema deve ter um escopo bem definido para que programa e programadores não passem dos seus limites estabelecidos. Este conceito é chamado de `domain`.
Um bom domínio determina seus termos utilizados para definir seus limites e responsábilidades, com o benefício de que todos os envolvidos falarão a mesma linguagem e não misturarão os contextos de cada objeto.


## Relacionamento entre objetos

Um objeto pode se relacionar através de seus atributos e ou comportamentos mencionando o outro objeto.
Quando temos este relacionamento, em nossa mente formamos uma rede de objetos que podemos denomina-la de modelo de objetos.


## Operações de um objeto

Assim como a concepção de um objeto é construido e destruído em nossa mente, este também é construido e destruído no computador. A contrução pode ser realizada de duas formas, seguindo um protótipo ou com um conjunto pré-definido de atributos e comportamentos, que podemos chamar de classe.

Na primeira abordagem um objeto nasce sem atributos e valores. Ao decorrer das tomas de decisões dentro do software o objeto vai criando sua forma e determinando seus comportamentos, assim terá sua função dentro do programa.

Já na segunda forma é responsábilidade do programador determinar antecipadamente um template de atributos que o objeto terá, mas não seus valores.

Objetos tem o tempo de vida limitado, seja ele durante um determinado ponto do programa ou no fim deste.

Ao destruir um programa é necessário que aqueles objetos que fazem referência ao que foi eliminado seja modificado também para que não faça menção a um local na memória que seja nulo, ou pior, que conténha dados de outro objeto em uso.
Em algumas linguagens o `Garbage coletor` pode realizar a limpeza automaticamente para evitar falhas, mas sempre é bom que nós nos reponsabilizemos por este ato também.


## Comportamentos de um objeto

Como mencionado em alguns pontos anteriores, um objeto pode ter comportamentos que irão mudar seu estados atual. Estes comportamentos são realizados por meio de funcionalidades atribuídas a ele. Um exemplo classico que muitos professores de faculdade gostam de aplicar (e que geralmente alunos odeiam) é o exemplo de um objeto que representa um automóvel.

Um carro além de seus atributos que definem suas características também contém comportamentos como andar e parar. A funcionalidade andar e parar mudará certamento o atríbuto velocidade deste objeto.


## Referências

[1]: K. Amini, Extreme C: taking you to the limit in concurrency, OOP, and the most advanced capabilities of C. Birmingham Mumbai: Packt, 2019.