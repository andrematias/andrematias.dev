---
date: 2023-09-13
author: André Matias
tags:
  - graduacao
  - estrutura-de-dados
  - programming
  - c-cpp
media: Book
title: Filas com lista dinamicamente encadeadas
---

Uma fila com alocação dinâmica é uma estrutura composta por descritores que contém dois ponteiros referenciando o inicio e o final dela. Estes ponteiros são nodos de uma lista encadeada. Além destas informações podemos incluir outras como por exemplo um contador para facilitar as operações para saber o tamanho da fila.

Esta estrutura tem critérios para inclusão e remoção de elementos que irão compo-la. Para incluir novos elementos é necessário sempre inclui-los no final da fila. Já para remover, precisamos fazer a operação utilizando o primeiro item da fila.

Diferente do que vemos acontecer com a estrutura de [[Pilhas]], onde, o último que entra é o primeiro a sair (LIFO - Last In, First Out), em filas temos o conceito de o primeiro a entrar é o primeiro a sair (FIFO - First In, First Out).

Essa característica torna a complexidade deste algoritmo como O(1), onde podemos acessar as informações com uma única operação.

Independente do tipo de implementação que decidirmos fazer, seja [[Filas sequenciais estáticas]] ou filas com listas dinamicamente encadeadas, teremos as mesmas operações de:
1. Criar uma fila
2. Inserir elementos no final da fila
3. Remover elementos do início da fila
4. Acesso ao primeiro elemento da fila
5. Verificar o tamanho da fila e se esta vazia ou cheia

Em uma fila com lista encadeada tem a vantagem de só encher se não houver mais espaço em memória e não precisamos saber de antemão o tamanho da lista já que ela cresce conforme a utilização, durante o tempo de execução.

## Implementação da fila com lista dinamicamente encadeada
Para compreender melhor como esta estrutura linear funciona na prática, faremos uma implementação de fila utilizando a linguagem C.
### A estrutura de uma fila
Para definirmos uma estrutura de fila, utilizaremos uma `struct` composta por um descritor que contém dois ponteiros do tipo nó utilizado em lista encadeada. Um para o *início* e outro para o *final* da fila. Também colocaremos um inteiro para realizar o incremento e o decremento ao incluir e remover elementos da fila.

*queue.h*

```c
#include <stdbool.h>

struct person {
	int age;
	char *name;
};

typedef struct person person_t;

struct node {
	person_t *person;
	struct node *next;
};

typedef struct node node_t;

/**
* Ocultamos a construção da estrutura de fila para que o cliente
* que utilizar esse cabeçalho não conheça a implementação dessa
* estrutura de fila.
* O nome dessa técnica é dados opacos, que será definido no arquivo
* de implementação.
*/
typedef struct queue queue_t; 

queue_t* queue_create();
bool queue_free(queue_t *queue);
int queue_qtd(queue_t *queue);
bool queue_insert(queue_t *queue, person_t *person);
bool queue_remove(queue_t *queue);
bool queue_get(queue_t *queue, person_t *person);
```

*queue.c*
```c
#include <stdio.h>
#include <stdlib.h>
#include "queue.h"

struct queue {
	node_t *start;
	node_t *end;
	int qtd;
}
```

### Criando uma fila
A criação de uma fila é feita com a alocação dinâmica de memória para reservar espaço suficiente para uma estrutura do tipo `queue_t`. Depois, iniciamos os valores dessa estrutura:
1. **start**: Iniciamos com NULL, pois ainda não incluimos nenhum nó da lista para compor os elementos de uma fila.
2. **end**: O mesmo que start
3. **qtd**: iniciamos uma fila vazia, portanto a quantidade de elementos inicialmente será 0

*queue.c*
```c
queue_t* queue_create(){
	queue_t *queue = (queue_t*)malloc(sizeof(queue_t));
	if (queue != NULL) {
		queue->start = NULL;
		queue->end = NULL;
		queue->qtd = 0;
	}
	return queue;
}

int main(){
	queue_t *queue = queue_create();
	if (queue == NULL) {
		printf("\nFalha ao criar a fila!");
		exit(1);
	}
	return 0;
}

```

### Liberação de memória da fila
Para realizar a liberação de memória alocada para uma fila, temos que percorrer todos os elementos dessa fila e liberar um a um, e por fim, liberar a própria fila.

*queue.c*
```c
bool queue_free(queue_t *queue) {
	if (queue == NULL) {
		return false;
	}
	node_t *node;
	while(queue->start != NULL) {
		node = queue->start;
		//Aponta para o próximo node do atual inicio
		queue->start = queue->start->next;
		free(node);
	}
	free(queue);
	return true;
}
```

### Verificando o status da fila
Em uma estrutura de fila não podemos tentar remover um elemento da fila se ela estiver vazia, portanto, precisamos ter ferramentas para saber se uma fila contém elementos. Podemos fazer isso aproveitando o atributo `qtd` da estrutura da fila que colocamos para contar os integrantes da fila.

*queue.c*
```c
int queue_qtd(queue_t *queue) {
	if (queue == NULL) {
		return 0;
	}
	return queue->qtd;
}
```

### Incluir elementos na fila
A inclusão de novos elementos em uma fila é relativamente simples. Podemos comparar com a inclusão em uma lista, porém, teremos que mudar as referencias em dois ponteiros, início e fim. Também precisamos incrementar o nosso contador.
Além dessas tarefas precisamos identificar se a fila é valida e se esta vazia ou não para tomarmos algumas decisões.
Em uma fila o primeiro elemento também é o último, pois é o único presente.

*queue.c*
```c
bool queue_insert(queue_t *queue, person_t *person){
	if (queue == NULL) {
		return false;
	}
	node_t *node = (node_t*)malloc(sizeof(node_t));
	if (node == NULL){
		return false;
	}
	node->person = person;
	node->next = NULL;
	if (queue->end == NULL){
		queue->start = node;
	} else {
		queue->end->next = node;
	}
	queue->end = node;
	queue->qtd++;
	return true;
}
```

Na implementação acima temos os seguintes passos:
1. Verificamos se a fila recebida como referencia é valida
2. Criamos um novo nó, logo após verificamos se foi realmente alocado memória para ele.
3. Preenchemos os dados desse novo nó com base na referencia recebida na função
4. Verificamos se a fila esta vazia checando se a referencia para o final (queue->end) é igual a NULL.
5. Se for NULL significa que este será nosso primeiro elemento. Então atribuímos o novo nó a referencia de inicio da fila (queue->start).
6. Caso já existir algum elemento associado ao final da fila, temos que incluir o novo como o próximo nó que esta no descritor `next` do nó final (queue->end->next).
7. Adicionamos o novo nó na referencia final da nossa fila (queue->end), para dizer que este é o último nó adicionado.
8. Incrementamos nosso contador de elementos na fila (queue->qtd).

### Removendo um elemento
Basicamente a operação para remover um elemento da fila é: Armazenar em uma variavel auxiliar o elemento inicial, depois modificar o elemento inicial para que agora ele seja o seguinte após ele próprio (queue->start = queue->start->next), liberamos o nó armazenado na variável auxiliar, verificamos se agora o inicio da fila é vazia para que possamos modificar o final e dizer que a fila é vazia, e por fim, decrementamos o contador. Vamos a implementação.

*queue.c*
```c
bool queue_remove(queue_t *queue) {
	if (queue == NULL) {
		return false;
	}

	if (queue->start == NULL) {
		return false;
	}

	node_t *aux = queue->start;
	queue->start = queue->start->next;
	free(aux);
	if (queue->start == NULL){
		queue->end = NULL;
	}
	queue->qtd--;
	return true;
}
```

### Consultando a fila
Como já vimos uma fila é sempre recuperada pelo seu início, portanto, não há a possibilidade de um item que esta no meio ser recuperado com uma simples operação O(1).

*queue.c*
```c
bool queue_get(queue_t *queue, person_t *person){
	if (queue == NULL) {
		return false;
	}

	if (queue->start == NULL){
		return false;
	}

	*person = queue->start->person;
	return true;
}
```

## Relacionados
- [[Alocação Dinâmica]]
- [[Listas encadeadas]]
- [[Pilhas]]

### References
- Backes, André Ricardo. **Algoritmos e estrutura de dados em linguagem C**. 1 Edição. Rio de Janeiro: LTC, 2023.
- Ito, Olavo Tomohisa. **Estruturas de Dados / Olavo Tomohisa Ito**. – São Paulo: Editora Sol, 2022.