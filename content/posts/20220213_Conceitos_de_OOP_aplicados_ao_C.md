---
title: Conceitos de OOP aplicados ao C
date: 2022-02-13
author:
  - André Matias
toc: true
tags:
  - c-cpp
  - oop 
description: Notas criadas durante a leitura do livro Extreme C de Kamran Amini
---

## Introdução

Estas são minhas notas criadas durante a leitura do capitulo de OOP do livro Extreme C de Kamran Amini.

Por padrão a linguagem C não oferece suporte explicito ao paradigma de programação de orientação á objetos.

Muitas linguagens de programação foram criadas com o propósito de substituir o C neste aspecto, a mais famosa e derivada diretamente foi a C++. O objetivo da linguagem C, quando criada, foi facilitar o desenvolvimento para as maquinas da época, estas que eram programada, nos melhores casos em Assembly ou Basic.

O C veio com a intenção de abstrair muitas das instruções que eram meticulosamente colocadas no programa para controlar diretamente o que o CPU deveria fazer.

O paradigma de programação estruturada já existia, porém era muito mais trabalhosas.

Como nós humanos pensamos em abstração, logo foi criado o conceito de Orientação á objetos para facilitar nossas vidas, mas para isso ainda precisaríamos transportar nossa abstração para procedimentos que a CPU possa executar, e para isso C cumpre esse papel com excelência. Por este motivo que C permanece atual, mesmo com tantas linguagens de alto nível surgindo a cada dia que passa.

Mas ainda assim é possível aplicar os conceitos de orientação á objetos usando os recursos da linguagem. Para isso realizamos a compreensão do que é um objeto e o que ele faz "implicitamente".


## Encapsulamento

Quando pensamos que um determinado objeto deve ter atributos e comportamentos únicos e exclusivos a ele estamos encapsulando-o.

Encapsulamento nada mais é do que definir limites e responsabilidades á determinado objeto.

Com linguagens próprias dedicadas à OOP temos em sua sintaxe facilidades de definir isso. Por exemplo em Java, C++, python e demais podemos criar uma classe onde é especificado os atributos e funcionalidades que darão ao objeto sua forma pré-definida antes de sua criação e construção.

Em C não temos esta sintaxe, mas podemos encapsular um objeto utilizando convenções de nomenclaturas nas variáveis e funções.

### Encapsulando atributos

Levando em consideração um objeto que representa uma linha, podemos definir utilizando variáveis seus atributos x e y, que indicarão o início e o fim desta linha.

```c
int line_l1_x = 10;
int line_l1_y = 20;
```

No exemplo temos uma linha `l1` que é um objeto do tipo `line` que contém um atributo `x` com o valor `10`, o mesmo ocorre para o atributo `y`.

Esta forma nos proporciona saber que o objeto existe e o que ele tem como atributo, tudo isso implicitamente. Porém, C nos oferece um tipo nativo que realiza um melhor encapsulamento, `struct`.

```c
struct line {
    int x;
    int y;
};

struct line l1;

l1.x = 10;
l1.y = 20;
```

Além de deixar o código mais legível, nos permite reutilizar um mesmo template de atributos que nós podemos pré-determinar a um objeto, isso é o que uma classe faz, a única diferença é que uma struct não encapsula funcionalidades.

Ainda podemos utilizar á palavra reservada `typedef` do C, que define um novo tipo de dado, baseado no que criarmos.

```c
typedef struct {
    int x;
    int y;
} line_t;

line_t l1;

l1.x = 10;
l1.y = 20;
```

### Encapsulando comportamentos

Como mencionado, `struct` não permite a inclusão de um comportamento entre seus valores. Este é um dos fatos que classes podem fazer e que structs não podem.

Para encapsular uma funcionalidade de um objeto podemos seguir também, convenções que nos remetem a um encapsulamento implícito.

Em um arquivo de cabeçalho `header`, incluímos as declarações do que nosso objeto é e do que ele pode fazer.

**Arquivo: car.h**

```c
#ifndef car_h
#define car_h

typedef struct {
    char name[32];
    double fuel;
    double speed;
} car_t;

void car_contruct(car_t*, const char*);
void car_destruct(car_t*);
void car_accelerate(car_t*);
void car_brake(car_t*);
void car_refuel(car_t*, double);

#endif
```

No arquivo de declarações temos um template do que nosso objeto do tipo `car_t` pode conter e fazer.

As funções que irão compor o comportamento deste objeto seguem a convenção onde o prefixo `car_` informa ao leitor do código e todos os envolvidos que aquela função representa um comportamento de um `car`. Outro ponto é que todas essas funções manipulam diretamente uma `instancia` do tipo struct `car_t` através de um ponteiro fornecido no primeiro parâmetro de cada função.

Assim como em linguagens que seguem protótipos, este exemplo é composto conforme o decorrer do programa. Temos somente as definições, para termos as `implementações` devemos, de fato, criar os comportamentos e preencher os valores do objeto.

**Arquivo: car.c**

```c
#include <string.h>
#include "car.h"

void car_contruct(car_t* car, const char* name) {
    strcpy(car->name, name);
    car->speed = 0.0;
    car->fuel = 0.0;
}

void car_destruct(car_t* car) { }

void car_accelerate(car_t* car) {
    car->speed += 0.05;
    car->fuel -= 1.0;
    if (car->fuel < 0.0) {
        car->fuel = 0.0;
    }
}

void car_brake(car_t* car) {
    car->speed -= 0.07;

    if (car->speed < 0.0) {
        car->speed = 0.0;
    }

    car->fuel -= 2.0;

    if (car->fuel < 0.0) {
        car->fuel = 0.0;
    }
}

void car_refuel(car_t* car, double amount) {
    car->fuel = amount;
}

```

No exemplo acima, implementamos os comportamentos do objeto, agora basta que alguma parte do programa os utilize. Em nosso caso básico usaremos a própria função `main` do C.

**Arquivo main.c**

```c
#include <stdio.h>
#include "car.h"

int main(int argc, char** argv) {
    car_t car;

    car_construct(&car, "Pegeout");

    car_refuel(&car, 100.0);

    printf("Carro abastecido com %f\n", car.fuel);

    while(car.fuel > 0) {
        printf("Combustível do carro em %f\n", car.fuel);
        if (car.speed < 80) {
            car_accelerate(&car);
            printf("Velocidade atual %f\n", car.speed);
        } else {
            car_brake(&car);
            printf("Diminuindo a velocidade, velocidade atual %f\n", car.speed);
        }
    }

    car_destruct(&car);
    return 0;
}
```

Nosso programa ficou responsável por brincar com nosso objeto carro, nele podemos ver que o carro foi contraído, abastecido, andou, parou e por fim destruído quando não ia mais ser utilizado.

Pontos importantes a serem levados em consideração é que, nosso carro não foi 'criado' já com todas os seus atributos preenchidos, este foi preenchido somente quando chamados o comportamento de construí-lo. Criar se dá ao ato de alocar memória suficiente para alocar uma estrutura já definida, isso é feito em tempo de compilação em nosso exemplo, já que somente usamos a região da memória denominada `stack`.

Ao finalizar o uso o objeto foi destruído através da função `car_destruct`, caso fosse utilizado a memória `heap` este seria um ponto extremamente relevante para o manuseio de memória, uma vez que a heap não tem liberação automática do espaço utilizado, já a stack tem por padrão esse comportamento.


### Escondendo informações

Um dos benefícios de encapsular um objeto é a possibilidade de esconder informações importantes ou que somente diz respeito à aquele objeto. No mundo da orientação à objetos temos o termo de `public` e `private`.

Em C podemos alcançar estes recursos declarando no arquivo header somente o que será publicamente exposto para outras partes do sistema. Mas como fazer isso?
Para atributos podemos declarar somente o nome da estrutura sem os campos. por exemplo:

 Ao invés de colocar no arquivo header:

```c
struct {
    size_t size;
    int* items;
} list_t;
```

Faça:

```c
struct list_t;
```

Assim todos as partes que fizer o import do cabeçalho verá somente o tipo que pode ser utilizado, mas não saberá quais seus campos que á compõem.


Para comportamentos declaramos no header somente aqueles que devemos deixar públicos e usamos o arquivo de implementação `.c` para criar nossas funções privadas.

Portanto, em um exemplo de lista onde queremos expor somente seus comportamentos podemos criar o cabeçalho assim:

**Arquivo list.h**

```c
#ifndef list_h
#define list_h

#include <unistd.h>

struct list_t; // Informa somente que é uma estrutura

struct list_t* list_create();

void list_construct(struct list_t*);
void list_destroy(struct list_t*);

int list_add(struct list_t*, int);
int list_get(struct list_t*, int, int*);
void list_clear(struct list_t*);
size_t list_size(struct list_t*);
void list_print(struct list_t*);

#endif
```

**Arquivo list.c**

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 10

typedef int bool_t;

// Define aqui o tipo e os campos da estrutura
typedef struct {
    size_t size;
    int* items;
} list_t;

//Convenção de nomenclatura para funcionalidade privada inicia com __
bool_t __list_is_full(list_t* list) {
    return (list->size == MAX_SIZE);
}

bool_t __check_index(list_t* list, const int index) {
    return (index >= 0 && index <= list->size);
}

list_t* list_create() {
    return (list_t*)malloc(sizeof(list_t));
}

void list_construct(list_t* list) {
    list->size = 0;
    list->items = (int*)malloc(MAX_SIZE * sizeof(int));
}

void list_destroy(list_t* list) {
    free(list->items);
}

int list_add(list_t* list, const int item) {
   if(__list_is_full(list)) {
       return -1;
   }
   list->items[list->size++] = item;
   return 0;
}

int list_get(list_t* list, const int indexm int* result) {
    if (__check_inde(list, index)) {
        *result = list->items[index];
        return 0;
    }

    return -1;
}

void list_clear(list_t* list) {
    list->size = 0;
}

int list_size(list_t* list) {
    return list->size;
}

void list_print(list_t* list) {
    printf("[");
    for (size_t i = 0; i < list->size; i++) {
        printf("%d ", list->items[i]);
    }

    printf("]\n");
}
```

Um detalhe que podemos ver no arquivo de implementação do objeto é que alocamos espaço na memória heap usando a função `malloc`, então se faz necessário realizar a limpeza após seu uso com a função `free`. Mas além disso isso se faz necessário por que para alocar memória para este tipo temos que informar seu tamanho, e na visão da função main isso não pode ser resgatado através da função `sizeof` porque main só sabe o tipo de list_t. Então para realizar essa criação e alocar memória na heap nós expomos um método de criação como público e na sua implementação esse terá as infamações necessárias para realizar o procedimento utilizando sizeof, uma vez que no arquivo de implementação privado teremos a declaração completa de `list_t`.

Em nossa estrutura completa list_t, temos um campo que é um ponteiro de inteiros que será utilizado para armazenar os elementos da lista. Para isso precisamos reservar espaço na heap para incluir os valores inteiros. Isso é feito através da funcionalidade `list_construct`.

**Arquivo main.c**

```c
#include <stdlib.h>
#include "list.h"

int reverse(struct list_t* source, struct list_t* dest) {
    list_clear(dest);
    for (size_t i = list_size(source) - 1; i >= 0; i--) {
        int item;
        if(list_get(source, i, &item)) {
            return -1;
        }
        list_add(dest, item);
    }
    return 0;
}

int main(int argc, char** argv) {
    struct list_t* list1 = list_create();
    struct list_t* list2 = list_create();

    list_construct(list1);
    list_construct(list2);

    list_add(list1, 4);
    list_add(list1, 6);
    list_add(list1, 1);
    list_add(list1, 5);


    list_add(list2, 9);

    reverse(list1, list2);
    list_print(list1);
    list_print(list2);

    list_destroy(list1);
    list_destroy(list2);

    free(list1);
    free(list2);
    return 0;
}
```

Como visto no arquivo main.c as funções escritas utilizaram somente o que foi exposto no arquivo list.h, ou seja, foi implementado somente o que foi fornecido na API do objeto list.

Para realizar a montagem do programa é necessário passar pela `linkagem` dos arquivos de objetos do C.
Para isso compilamos os arquivos separadamente com o comando:

```bash
gcc -c list.c -o private.o
gcc -c main.c -o main.o
```

Depois que temos os arquivos objetos podemos lincá-los e obter um programa:

```bash
gcc main.o private.o -o list
./list
```

Este processo nos oferece a possibilidade de modificar somente o arquivo de implementação de list porque sua API pública já é conhecida por main. Se essa api mudar, ai sim será necessário realizar todo o procedimento de compilação e lincagem.

Isso também nos habilita a exportar nossa implementação de list como uma biblioteca dinâmica como `list.so` e fazer o processo de importação em tempo de compilação do arquivo main.c.


## Composição de objetos

É muito comum em um software que partes dele se comuniquem um com os outros. Assim como objetos em nossa vida tem relações, objetos na OOP também podem ter.
Levando isso em consideração explicarei como um objeto fará menção a outro diretamente utilizando a abordagem de composições.

Composição é o ato de um objeto "ter" como sua parte um outro objeto e usufruir de seus atributos e comportamentos.

Em linguagens de programação de mais alto nível, vemos com mais facilidade um relacionamento deste tipo, por exemplo em C++ temos o seguinte:

```c++

typedef enum {
    ON, OFF
} state_t;

class Engine {
    public:
        Engine();
        ~Engine;
        void turn_on();
        void turn_off();
        void get_temperature();
    private:
        state_t state
        double temperature;
};

class Car {
    public:
        Car(Engine engine);
        ~Car;
        void turn_on();
        void turn_off();
        double get_speed();
    
    private:
        std::string name;
        double speed;
        Engine engine; // Composition
};

```

No código simples apresentado acima, podemos ver claramente ao ler as classes que um carro (Car) "tem" um motor (Engine). Isto implica que ao destruir o objeto Car a Engine não existirá mais, pois foi destruído junto ao carro.

Mas como será possível realizar essa composição utilizando C, já que não temos a syntax de class?

Lembre-se uma classe nada mais é que um template que será utilizado para criar objetos e esse template só disponibiliza os atributos (e comportamentos no caso de C++) que um objeto pode ter.

Em C podemos utilizar `struct` para definir esse template e utilizar os arquivos de cabeçalhos para disponibilizar os atributos e comportamentos **públicos** de nossos objetos e todos os dados privados ficará escondidos no arquivo de implementação. Veja a sessão de encapsulamento anterior.


**Implementando uma composição em C**
Para facilitar vamos utilizar o mesmo exemplo que no código de C++. Começaremos definindo as interfaces públicas.

**Arquivo car.h**

```c
#ifndef car_h
#define car_h

struct car_t;

struct cat_t* car_new();
void car_ctor(struct car_t*);
void car_dtor(struct car_t*);

void car_start(struct car_t*);
void car_stop(struct car_t*);
double car_get_enfine_temperature(struct car_t*);

#endif
```

**Arquivo engine.h**

```c
#ifndef engine_h
#define engine_h

struct engine_t;

struct engine_t* engine_new();
void engine_ctor(struct engine_t*);
void engine_dtor(struct engine_t*);

void engine_turn_on(struct engine_t*);
void engine_turn_off(struct engine_t*);
double engine_get_temperature(struct engine_t*);

#endif
```

Oakey, interfaces prontas note que informamos somente o que queremos que conheçam. Alguns detalhes privados ficarão nos arquivos de implementação 'car.c' e 'engine.c'.

**Arquivo car.c**

```c
#include <stdlib.h> //Necessário para malloc e free
#include "engine.h" // Olha a interface pública ai :)

typedef struct {
    struct engine_t* engine; //Composição
} car_t;

car_t* car_new() {
    return (car_t*) malloc(sizeof(car_t));
}


void car_ctor(car_t* car) {
    car->engine = engine_new();
    engine_ctor(car->engine);
}

void car_dtor(car_t* car) {
    engine_dtor(car->engine);
    free(car->engine);
}

void car_start(car_t* car) {
    engine_turn_on(car->engine);
}

void car_stop(car_t* car) {
    engine_turn_off(car->engine);
}

void car_get_engine_temperature(car_t* car) {
    engine_get_temperature(car->engine);
}

```

**Arquivo engine.c**

```c
#inclide <stdlib.h>

typedef enum {
    ON,
    OFF
} state_t;

typedef struct {
    state_t state;
    double temperature;
} engine_t;

engine_t* engine_new() {
    return (engine_t*) malloc(sizeof(engine_t));
}

void engine_ctor(engine_t* engine) {
    engine->state = OFF;
    engine->temperature = 15;
}

void engine_dtor(engine_t* engine) {
    /* 
    * Por enquanto não há nada para liberar que
    * o método engine_ctor criou. Em outras palavras
    * não houve nenhum malloc no contrutor
    */
}

void engine_turn_on(engine_t* engine) {
    if (engine->state == ON) {
        return;
    }
    engine->state = ON;
    engine->temperature = 75;
}

void engine_turn_off(engine_t* engine) {
    if (engine->state == OFF) {
        return;
    }
    
    engine->state = OFF;
    engine->temperature = 15;
}

double engine_get_temperature(engine_t* engine) {
    return engine->temperature;
}

```

Agora temos nossas implementações feitas. Note que criamos um tipo de dado chamado car_t e engine_t e nessa criação de tipo determinamos que será uma struct e nela vai ter alguns atributos. Tudo o que foi feito nessa declaração do novo tipo não será visto fora deste arquivo de implementação o que nos garante um nível de privacidade implícito, onde somente que tem o código fonte saberá o que tem nessa estrutura.

Vamos usar nossas implementações agora em um programa.

**Arquivo main.c**

```c
#include <stdio.h>
#include <stdlib.h>
#include "car.h"

int main(int argc, char** argv) {
    struct car_t *car = car_new();
    car_ctor(car);
    
    printf(
        "Temperatura do motor antes de ligar o carro: %f\n",
        car_get_engine_temperature(car)
    );
    
    car_start(car);
    printf(
        "Temperatura do motor depois de ligar o carro: %f\n",
        car_get_engine_temperature(car);
    );
    
    car_stop(car);
    printf(
        "Temperatura do motor depois de desligar o carro: %f\n",
        car_get_engine_temperature(car);
    );
    
    car_dtor(car);
    free(car);
    
    return 0;
}
```

Veja que o programa que utiliza nossos objetos só conhece o objeto carro e o objeto carro que manipula o motor através da composição. Portanto o encapsulamento com a composição de objetos funcionou!

Para executar o programa devemos compilar cada arquivo .c e depois lincar os outputs para montar nosso executável.

```bash
gcc -c engine.c -o engine.o
gcc -c car.c -o car.o
gcc -c main.c -o main.o
gcc engine.o car.o main.o -o car

.\car #Executa o programa
```



## Veja também

- [Conceitos de OOP](/posts/20220212_Conceitos_de_OOP)


## Bibliografia

[1]: K. Amini, Extreme C: taking you to the limit in concurrency, OOP, and the most advanced capabilities of C. Birmingham Mumbai: Packt, 2019.