---
tags:
  - c-cpp
  - programming
author: Fábio Akita
date: 2022-02-02
media: Video
title: Entendendo C - Video Fabio Akita
description: Anotações feitas assistindo ao vídeo Fabio Akita Hello World Como Você Nuca Viu! | Entendendo C
---

## **Notas do vídeo**

- Normalmente linguagens de alto nível aloca um array maior do que o solicitado quando há espaço em memória suficiente, mas quando realmente chegamos ao final do array é necessário criar um novo array com o tamanho desejado contando o novo elemento, copiado item a item para o novo array e excluído o array antigo.
- Em C strings sem bibliotecas extras é uma sequência de caracteres (um array de char `char texto[] = "teste teste") com o carácter nulo no final.
- Tudo tem endereço na memória
- As chamadas de função são alocadas na parte da memória virtual chamada Stack, toda vez que a função executa a instrução return ela sai do Program Counter que é responsável por armazenar os endereços e valores na Stack.
- Quando usamos ponteiros utilizamos a parte da memória virtual Stack somente para armazenar os endereços dos objetos que ficarão na parte da memória virtual chamada Heap, que é uma região mais baixa da memória.
- Uma string idealmente deveria ser imutável onde somente se necessário modificá-la podemos realizar a operação de criar um novo espaço na memória com o tamanho suficiente para conter o novo valor e liberar o espaço anterior.
- Para trabalhar com ponteiros precisamos da biblioteca stdlib.h
- Structs são uma sequência de elementos que contém dimensões diferentes.
- Quando um programa é finalizado o sistema operacional realiza a limpeza da stack, portanto mesmo utilizando ponteiros quando o programa finalizar as referencias serão limpas mas os objetos que ficam na heap não. Por este motivo é necessário ter o gerenciamento dessa memória. Em linguagens de auto nível existem Garbage Collectors que realizam automaticamente essa limpeza.

## **Códigos de exemplo**

**Um simples hello World em C**

```c
#include <stdio.h>

void main() {
    char hello[] = "Hello World";
    printf("%s\n", hello);
    return;
}
```

**Exibindo endereços de memória**

```c
#include <stdio.h>

void f2(char hello[]) {
    printf("from f2: %s\n", hello);
}

void f1(char hello[]) {
    printf("from f1: %d\n", &hello);
    f2(hello);
}

void main() {
    char hello[] = "Hello World";
    printf("from main: %d\n", &hello);
    f1(hello);
    return;
}
```

Este código simplesmente declara uma string 'hello' exibe seu endereço na memória, passa o seu valor para a função f1 que faz o mesmo para a nova alocação e depois é passado para a função f2 que exibe o valor que esta alocado no endereço de memória.

A passagem desses parâmetros é chamado de passagem por valor, onde é realizado uma cópia do valor em memória. Também existe a passagem por referencia onde o valor é somente apontado e que não é feito uma nova cópia.

**Um estouro da pilha Stack**

```c
#include <stdio.h>

void f3(char hello[]) {
    printf("from f2: %x\n", &hello);
    f3(hello)
}

void main() {
    char hello[] = "Hello World";
    printf("from main: %d\n", &hello);
    f3(hello);
    return;
}
```

Esta função fica em loop até acabar o espaço na Stack por que não há uma condicional de parada, portando o Program Counter fica registrando os valores na Stack até acabar o espaço.

**Utilizando o Ponteiro em C**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void main() {
    char hello[] = "Hello World";
    printf("from main: %d\n", &hello);

    char *hello2 = malloc(sizeof(hello));
    strcpy(hello2, hello);
    printf("hello2: %x\n", hello2);

    char *hello3 = hello2 + 6;
    printf("from hello2: %s\n", hello2);
    printf("from hello3: %s\n", hello3);

    return;
}
```

A função `malloc` de stdlib reserva espaço na região Head da memória para armazenar a quantidade solicitada de bytes que a função `sizeof` retorna ao calcular a quantidade de bytes que o array de char hello tem, no caso 12 bytes contando o caracter nulo `\0`.

A função `strcpy` da biblioteca string.h realiza a cópia do valor que esta no endereço da variavel hello para o endereço da variavel hello2, uma vêz que ambas são ponteiros, sendo um array também um ponteiro, a função realiza a cópia de item a item que esta neste local.

Na variável hello3 podemos realizar operações aritméticas com ponteiros porque será levado em conta a quantidade de bytes e não o valor armazenado. Portanto nesta operação `hello2 + 6` estamos deslocando 6 bytes a frente, que no caso coloca nosso ponteiro hello3 para iniciar no endereço que contém a letra `W` de Hello World.

Quando utilizamos a notação `hello2[6]` também estamos fazendo esta operação aritimética `hello2 + 6`.

Para exibir o endereço de uma variável usamos o `&` i.e.: '&hello2' e para o valor que esta neste ponteiro utilizamos o caracter `*` i.e.: '\*hello2'

**Um struct simples**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <inttypes.h>

struct Person {
    char name[10];
    uint8_t age;
    uint8_t height;
}

void main() {
    struct Person person;
    strcpy(person.name, "Matias");
    person.age = 32;
    person.height = 185;
    printf("\n%s %d %d", person.name, person.age, person.height);
    printf("\n%x", &person);

    return;
}
```

Quando criamos um struct ele aloca espaço suficiente na região da memória stack conforme os tipos declarado internamente.

**Ponteiro de funções e callback**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <inttypes.h>

struct Person {
    char name[10];
    uint8_t age;
    uint8_t height;
};

struct Person person_new(char name[], uint8_t age, uint8_t height) {
    struct Person person;
    strcpy(person.name, name);
    person.age = age;
    person.height = height;
    return person;
}

void person_introduce(struct Person person, void (*speak)(struct Person)) {
    (*speak)(person);
}

void happy_speaker(struct Person person) {
    printf("\nHappy person");
    printf(
       "\n Hello guys, I'm %s, %d years old and %d of height",
        person.name,
        person.age,
        person.height
    );
}

void angry_speaker(struct Person person) {
    printf("\nAngry person");
    printf(
       "\n Fuck guys, I'm %s, %d years old and %d of height. I don't care.",
        person.name,
        person.age,
        person.height
    );
}

void main() {
    struct Person person = person_new("André Matias", 32, 185);
    person_introduce(person, &happy_speaker);
    person_introduce(person, &angry_speaker);
}
```

No bloco de código acima temos a função `person_introduce` onde recebe como paramêtros uma struct Person e um ponteiro de função que retorna void e tem como parametro uma struct Person.

Isso nos permite passar por referencia qualquer função que siga essa assinatura, o que nos remete a funções de callback.

Para utilizar a função passada por referencia utilizamos a sintax `(*ponteiro)(parametros)`. Isso é como se estivessemos chamando o nome da função que foi passada e fornecendo os seus parâmetros.

Neste exemplo estamos duplicando a variável person criada pela função `person_new` cada vêz que a passamos como parametro à uma função, ou seja, estamos realizando a passagem por valor.

**Pseudo OOP com C**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <inttypes.h>

struct Person {
    char name[10];
    uint8_t age;
    uint8_t height;
    void (*speak)(struct Person *);
};

void angry_speaker(struct Person *person) {
    printf("\nAngry person");
    printf(
       "\n Fuck guys, I'm %s, %d years old and %d of height. I don't care.",
        person->name,
        person->age,
        person->height
    );
}

struct Person * person_new(char name[], uint8_t age, uint8_t height) {
    struct Person *person = malloc(sizeof(struct Person));
    strcpy(person->name, name);
    person->age = age;
    person->height = height;
    person->speak = &angry_speaker;
    return person;
}

void main() {
    struct Person *person = person_new("André Matias", 32, 185);
    person->speak(person);
    free(person);
}
```

Neste exemplo a struct Person recebe um ponteiro de função, o que podemos encarar como um tipo de método, mas este não tem nenhum comportamento associado ainda.

A função `person_new` é responsável por 'instanciar' o 'objeto' person, associando valores e comportamentos a ele. Por padrão o comportamento do método `speak` é definido na função `angry_speaker`.

Para que não haja duplicidade da variável person como no exemplo anterior, passamos como referencia utilizando um ponteiro criado pela função `person_new`.

Assim como na linguagem python que recebe o primeiro argumento de um método a variável `self` podemos encarar o argumento de `speak` como o próprio objeto que sofrerá uma modificação do seu estado ou realizará uma determinada atividade.

### Referências

- [Fabio Akita Hello World Como Você Nuca Viu! | Entendendo C](https://www.youtube.com/watch?v=Gp2m8ZuXoPg)
- [Hack The Virtual Memory - Holberton School article](https://blog.holbertonschool.com/hack-the-virtual-memory-drawing-the-vm-diagram)
