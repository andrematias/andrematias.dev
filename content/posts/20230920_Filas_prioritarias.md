---
date: 2023-09-20
author: André Matias
tags:
  - literature
  - programming
  - graduacao
  - c-cpp
  - ciencia-da-computacao
  - literature
media: Book
title: Filas prioritarias
---

Uma fila prioritária é uma implementação de filas onde existe uma ordenação baseada em um atributo do elemento a ser incluído que define sua prioridade.
Com este atributo definido, teremos um critério para inclusão, remoção e recuperação dos dados que estarão na lista.
Existem alguns tipos de implementações que podem ser consideradas no momento de criar o algoritmo para armazenar os dados. São elas:
1. Lista dinâmicamente encadeada
2. `array` ordenado
3. `array` desordenado
3. heap binária

Todas essas alternativas tem seus prós e contras, devemos analisar qual se adequa melhor na resolução do nosso problema.
Abaixo é exibido uma tabela com as complexidades em cada operação:

| Implementação            | Inserção | Remoção  |
| ------------------------ | -------- | -------- |
| Lista dinâmica encadeada | O(N)     | O(1)     |
| Array ordenado           | O(1)     | O(N)     |
| Array desordenado        | O(N)     | O(1)     |
| Heap binária             | O(log N) | O(log N) | 

## Implementação com um array ordenado
Para fins didáticos, vamos seguir com uma implementação simplificada utilizando um `array` ordenado.

### Estruturas utilizadas
Em nosso arquivo header vamos criar nossos novos tipos, decidir o tamanho do array que armazenará os dados e as operações públicas e privadas da interface de nossa lista prioritária.
Para separar o que o cliente que irá utilizar e o que ele não terá visão vamos separar nossos arquivos cabeçalho em dois, `priority_queue.h` e `priority_queue_p.h`. 
Vamos definir em `priority_queue.h` nosso tipo "opaco"[^1]

> Imagine que você esta fazendo uma biblioteca que contém essa implementação, mas não queremos que o cliente que usa essa biblioteca saiba a composição da estrutura que definimos para a lista, pois se ele souber, pode simplesmente reescrever essa estrutura e hackear nossa implementação para armazenar mais informações ou outros tipos de falhas podem ocorrer.
> Para isso, na distribuição dos arquivos de cabeçalho junto com a nossa biblioteca fornecemos somente o arquivo `priority_queue.h` e a biblioteca compilada `libpriority_queue.a` ou `libpriority_queue.so` em ambiente Unix-Like ou `libpriority_queue.dll` no windows.

`priority_queue.h`
```c
#include <bool.h>
#define MAX 100

// Definição opaca da estrutura de fila prioritaria
typedef struct priority_queue prio_queue_t;

prio_queue_t* prio_queue_create();
void prio_queue_free(prio_queue_t *prio_queue);
bool prio_queue_get(prio_queue_t *prio_queue, char *name);
bool prio_queue_insert(prio_queue_t *prio_queue, char *name, int prio);
bool prio_queue_remove(prio_queue_t *prio_queue);
int prio_queue_size(prio_queue_t *prio_queue);
bool prio_queue_is_empty(prio_queue_t *prio_queue);
bool prio_queue_is_full(prio_queue_t *prio_queue);
```

`priority_queue_p.h`
```c
#include "priority_queue.h"

struct pacient {
	char name[30];
	int prio;
};

// Definição do tipo fila prioritária
struct priority_queue {
	int qtd;
	struct pacient data[max];
};
```

### Criando a estrutura da fila
A criação da fila é feita alocando memória suficiente com a função `malloc`.

`priority_queue.c` 
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "priority_queue.h"
#include "priority_queue_p.h"

prio_queue_t* prio_queue_create(){
	pro_queue_t *pqueue = (prio_queue_t*)malloc(sizeof(prio_queue_t));
	if (pqueue != NULL){
		pqueue->qtd = 0;
	}
	return pqueue;
}
```

### Inserindo elementos (com array ordenado)
Neste tipo de implementação o `array` utilizado estará ordenado na ordem crescente, ou seja, elementos com uma prioridade maior estarão no fim do vetor (inicio da fila), e os com prioridades menores estarão no começo (final da fila).
Esta implementação com um `array` ordenado nos proporciona uma operação de inclusão linear O(n), portanto seu caso dependerá da quantidade de elementos já inclusos nela, pois teremos que deslocar todos os elementos com a prioridade maior ou igual uma posição a frente, para assim termos um lugar no vetor para inserir o novo. Dessa forma a inclusão respeitará a ordenação por prioridades.

`priority_queue.c`
```c
// todo o conteúdo anterior

bool prio_queue_insert(prio_queue_t *prio_queue, char *name, int prio){
	if (prio_queue == NULL) {
		return false;
	}

	if (prio_queue->qtd == MAX) {
		return false;
	}

	int i = prio_queue->qtd - 1;
	while(i >= 0 && prio_queue->data[i].prio >= prio) {
		prio_queue[i + 1] = prio_queue->data[i];
		i--;
	}

	strcpy(prio_queue->data[i + 1].name, name);
	prio_queue->data[i + 1].prio = prio;
	prio_queue->qtd++;
	return true;
}

```

Neste método, verificamos se o ponteiro para a lista foi criado com sucesso e se o vetor de dados não esta totalmente cheio. Logo depois fazemos uma iteração em todos os elementos que estão na lista para saber se a prioridade do novo item a ser incluído é menor do que já estão lá no vetor. Quando o loop encontrar uma prioridade menor do que o novo elemento, saímos do loop `while` e copiamos o novo item para a posição em que a variável `i` mais 1 `(i + 1)` e incrementamos o contador de quantidade de itens da fila.

### Liberar memória
O processo para liberar memória neste tipo de implementação é simples, pois o único momento em que alocamos espaço na memória foi quando criamos a fila. Os dados são armazenados no vetor incluso na estrutura de fila.

`priority_queue.c`
```c
void prio_queue_free(prio_queue_t *prio_queue) {
	free(prio_queue);
}
```

### Pegar elementos da fila
Lembre-se, nossa fila esta ordenada na ordem crescente, ou seja, o elemento com a prioridade maior esta no final do vetor de dados, Portanto, para recuperarmos uma informação basta acessar com o índice que será sempre o total de itens menos 1: `prio_queue->data[prio_queue->qtd - 1]`.
Em nosso exemplo estamos implementando uma fila de pacientes, a recuperação do paciente com prioridade mais alta será uma string com o nome.

`priority_queue.c`
```c
bool prio_queue_get(prio_queue_t *prio_queue, char *name) {
	if (prio_queue == NULL || prio_queue->qtd == 0) {
		return false;
	}
	strcpy(name, prio_queue->data[prio_queue->qtd - 1].name);
	return true;
}
```


### Removendo um elemento da fila prioritária
Basicamente, quando estamos tratando de uma fila prioritária implementada com um array ordenado, gerenciamos somente os índices, já que o vetor esta na região de memória `stack` e não na `heap`, isso nos proporciona somente substituir o conteúdo nas inclusões. Para gerenciar esse índice usamos o atributo de nossa estrutura de fila que conta quantos itens existem: `prio_queue->qtd`.

`priority_queue.c`
```c
bool prio_queue_remove(prio_queue_t *prio_queue){
	if (prio_queue == NULL || prio_queue->qtd == 0) {
		return false;
	}
	/*
	* Faz com que o último item (primeiro da fila com a prioridade maior)
	* seja descartado
	*/
	prio_queue->qtd--; 
	return true;
}
```

### Coletando informações sobre a fila

#### Tamanho da fila
`priority_queue.c`
```c
int prio_queue_size(prio_queue_t *prio_queue) {
	if (prio_queue == NULL || prio_queue->qtd == 0) {
		return false;
	}
	return prio_queue->qtd;
}

```

#### Fila cheia ou vazia
`priority_queue.c`
```c
bool prio_queue_is_empty(prio_queue_t *prio_queue) {
	if (prio_queue == NULL || prio_queue->qtd == 0) {
		return false;
	}
	return prio_queue->qtd == 0;
}

bool prio_queue_is_full(prio_queue_t *prio_queue){
	if (prio_queue == NULL || prio_queue->qtd == 0) {
		return false;
	}
	return prio_queue->qtd == MAX;
}
```


## Relacionados
- [[Dados homogêneos]]
- [[Filas sequenciais estáticas]]
- [[Gerenciamento de Memória]]

### Referências
- Backes, André Ricardo. **Algoritmos e estrutura de dados em linguagem C**. 1 Edição. Rio de Janeiro: LTC, 2023.
- Ito, Olavo Tomohisa. **Estruturas de Dados / Olavo Tomohisa Ito**. – São Paulo: Editora Sol, 2022.
- Tipo abstrato de dado. **Wikipédia**, 19 de setembro de 2023. Disponível em: <https://pt.wikipedia.org/wiki/Tipo_abstrato_de_dado>. Acesso em: 20, Setembro de 2023.
## Notas de rodapé
[^1]: Tipos opacos em C são estruturas que escondem seus detalhes para manter sua composição privada.
