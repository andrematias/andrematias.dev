---
date: 2024-09-02
tags:
  - akka
  - scala
title: Comunicação entre atores com Akka
---

## Configuração das dependências

Para este exemplo funcionar precisamos instalar alguns pacotes utilizando o gerenciador de pacotes `sbt`.

```scala
ThisBuild / version := "0.1.0"

ThisBuild / scalaVersion := "3.3.3"

lazy val root = (project in file("."))
  .settings(
    name := "rest"
  )

resolvers += "Akka library repository".at("https://repo.akka.io/maven")
val akkaVersion = "2.9.5"
val sl4jVersion = "2.0.12"
val logBakVersion = "1.5.6"
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor-typed" % akkaVersion,
  "org.slf4j" % "slf4j-api" % sl4jVersion,
  "ch.qos.logback" % "logback-core" % logBakVersion,
  "ch.qos.logback" % "logback-classic" % logBakVersion
)
```

## Criando atores e enviando

No código abaixo vamos criar dois atores que enviarão mensagens entre eles, com o simples objetivo de dizer olá.

```scala
import akka.actor.typed.ActorRef
import akka.actor.typed.ActorSystem
import akka.actor.typed.Behavior
import akka.actor.typed.scaladsl.Behaviors


object HelloWorld {
    final case class Greet(whom: String, replyTo: ActorRef[Greeted])
    final case class Greeted(whom: String, replyTo: ActorRef[Greet])

    def apply(): Behavior[Greet] = Behaviors.receive { (context, message) =>
        context.log.info("Hello {}!", message.whom)
        message.replyTo ! Greeted(message.whom, context.self)
        Behaviors.same
    }
}
```

No objeto criado temos as case Classes `Greet` e `Greeted` que recebem em seu construtor os parâmetros `whom` e `replyTo` que informam quem são e para quem responder respectivamente. Estes serão nossas mensagens.

Também definimos a implementação do método `apply` que define qual é o comportamento que será executado.

Este objeto compõe nosso ator `HelloWorld`

### Descrevendo o comportamento do ator HelloWorld

No método `apply` utilizamos uma fabrica de `Behavior` chamada `Behaviors` e criamos um Behavior do tipo `receive` que recebe como implementação uma função anonima que tem como parâmetro `context` e `message`.

Este comportamento utiliza a propriedade `replyTo` que é do tipo `Greet` que esta no parâmetro `message` para enviar uma mensagem do tipo `Greeted` como resposta. Conseguimos acessar estas propriedades porque definimos que o tipo de mensagem seria um `ActorRef` do tipo `Greeted`.

Neste exemplo simples, não queremos modificar o comportamento para receber a próxima mensagem então utilizamos a fabrica `Behaviors.same` para retornar o mesmo que criamos.

## Criando outro ator para interação

Como diz o criador do modelo:

> "One Actor is no Actor - It would be quite lonely with nobody to talk to" -- Carl Hewitt

Portanto, um ator só será útil se houver outros atores para trocar mensagens.

Para utilizar o nosso ator `HelloWorld` precisamos de outro ator, então vamos criar:

```scala
object HelloWorldBot {
    def apply(max: Int): Behavior[HelloWorld.Greeted] = {
        bot(0, max)
    }

    private def bot(greetingCounter: Int, max: Int): Behavior[HelloWorld.Greeted] = Behaviors.receive { (context, message) =>
        val n = greetingCounter + 1
        context.log.info("Greeting {} for {}", n, message.whom)
        if (n == max) {
            Behaviors.stopped
        } else {
            message.replyTo ! HelloWorld.Greet(message.whom, context.self)
            bot(n, max)
        }
    }
}
```

Neste ator `HelloWorldBot` definimos 2 métodos, um método publico que é exigido pela interface `ActorRef` chamado `apply` e outro privado chamado `bot` que será recursivo e tem como objetivo ser chamado até que seu limite `max` seja atingido.
O método `apply` faz a inicialização utilizando o método `bot` com o valor inicial igual a 0 e `max` igual ao que será definido no objeto que o chamará.
Em toda a chamada de `bot` será enviado uma mensagem `Greet` para o ator `Greeted` que foi definido na declaração do retorno do `Behavior`.
Quando `max` chegar ao valor definido o método modifica o comportamento do ator para `stopped` criado a partir da fabrica `Behaviors`, caso contrário fará uma nova chamada recursiva.

## Utilizando os atores para troca de mensagens

Agora que já temos nossos dois atores vamos criar nosso ator principal para que ele possa dar o pontapé inicial no sistema.

```scala
object HelloWorldMain {
    final case class SayHello(name: String)

    def apply(): Behavior[SayHello] = Behaviors.setup { (context) =>
        val greeter = context.spawn(HelloWorld(), "greeter")

        Behaviors.receiveMessage { (message) =>
            val replyTo = context.spawn(HelloWorldBot(max = 3), message.name)
            greeter ! HelloWorld.Greet(message.name, replyTo)
            Behaviors.same
        }
    }

    def main(args: Array[String]): Unit = {
        val system = ActorSystem[HelloWorldMain.SayHello] = ActorSystem(HelloWorldMain(), "hello")

        system ! HelloWorldMain.SayHello("World")
        system ! HelloWorldMain.SayHello("Akka")
    }
}
```

Este ator `HelloWorldMain` implementa a interface de `ActorRef`, ou seja também tem seu método `apply` definindo um comportamento quando for executado.

O ator tem uma mensagem `SayHello` que será utilizada para enviar para ele mesmo via ator do sistema criado no método `main`.

O comportamento deste ator, diferente dos demais é um comportamento de configuração que gera um ator `greeter` que é utilizado para enviar a mensagem `HelloWorld.Greet` com o `name` definido na sua mensagem recebida `SayHello`.

No método de entrada `main` é criado uma instancia de `ActorSystem` do tipo `HelloWorldMain` que utiliza mensagens do tipo `SayHello`.


## Veja também
- [Fundamentos de atores](/posts/20240902_Actor_Fundamentals)

## Referencias
- [Typed actors - doc.akka.io](https://doc.akka.io/docs/akka/current/typed/actors.html?_gl=1*1raoif1*_gcl_au*ODA1OTAyNjcuMTcyNDg1NTM5Ng..#first-example)
