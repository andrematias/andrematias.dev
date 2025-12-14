---
tags:
  - scala
  - akka
date: 2024-09-03
title: Básico de Akka com Scala
description:
Explorando um pouco o framework Akka com a linguagem Scala na prática
---

## Creating Actors - Object Oriented style

Criar atores seguindo a abordagem de orientação a objetos nos fornece meios de ter um estado mutável diretamente pelos seus atributos definidos na criação. Podemos, também modelar o comportamento do ator sobrescrevendo métodos que serão utilizados pelo sistema de atores para gerenciar as mensagens.
É preferível modelar o ator utilizando orientação a objetos quando pretendemos que o estado do ator seja modificado.

```scala
import akka.actor.typed.Behavior
import akka.actor.typed.scaladsl.AbstractBehavior
import akka.actor.typed.scaladsl.ActorContext
import akka.actor.typed.scaladsl.Behavior

object Greeter {
	final case class Greet(whom: String)

	class GreeterBehavior(context: ActorContext[Greet]) extends AbstractBehavior[Greet](context) {
		override def onMessage(msg: Greet): Behavior[Greet] = {
		  context.log.info(s"Hello ${msg.whom}!")
		  this
		}
	}

	def apply(): Behavior[Greet] = Behaviors.setup(context => new GreeterBehavior(context))
}
```

## Creating Actors - Functional style

No estilo funcional podemos usufruir da imutabilidade nativa que o estilo frequentemente utiliza para modelagem. A mudança do estado do ator é dado pela mudança do comportamento, onde é passado um estado imutável por parâmetro para o novo comportamento.

```scala
import akka.actor.typed.Behavior
import akka.actor.typed.scaladsl.Behaviors

object Greeter {
  sealed trait GreetCommand
  final case class Greet(whom: String) extends GreetCommand
  final case class Goodbye(whom: String) extends GreetCommand

  def apply(): Behavior[GreetCommand] =
	Behaviors.receive { (context, message) =>
	    message match {
	      case Greet(whom) =>
	        context.log.info(s"Hello $whom")
	        Behaviors.same
	      case Goodbye(whom) =>
	        context.log.info(s"Goodbye $whom")
	        Behaviors.same
	    }
	}
}

```

Ambos os estilos compartilharam da mesma definição externa, por exemplo: os dois tem a mesma interação com `ActorRef[Protocol]` e são instanciados da mesma maneira.

## Instantiating actors using ActorSystem

O `ActorSystem` cria um ator de alto nível, também chamado de ator guardião. Este é responsável por gerenciar e gerar atores filhos, dando o ponta pé inicial no modelo de atores. Parando o ator guardião toda a arvore de atores abaixo dele será finalizada consequentemente.

```scala
import akka.actor.typed.ActorSystem

object Main {
	def main(args: String) {
		val greeter: ActorSystem[Greeter.Greet] = ActorSystem(Greeter(), "greeter")

		greeter ! Greeter.Greet("Matias")

		println(">>> Press ENTER to exit <<<")
		readLine()
		Exception.ignoring(classOf[IOException])

		greeter.terminate
	}
}
```

## Veja também

- [Actor Fundamentals](/posts/20240902_Actor_Fundamentals)
