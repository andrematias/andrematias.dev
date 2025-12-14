---
date: 2023-09-27
Author: André Matias
tags:
  - estrutura-de-dados
title: Árvores binárias e árvores binárias de busca
---

Uma árvore binaria é um tipo de estrutura de dados da família de [[Árvores|árvores]] onde o nó pai, pode ter no máximo duas subárvores. Uma à esquerda e uma a direita. Caso não exista nenhuma subárvore, este será um nó folha.

Podemos utilizar árvores binárias em casos onde é necessário uma ordem. Em banco de dados ou até mesmo na representação de uma resolução de equação matemática, onde existem as ordem de precedências que devem ser respeitadas.

Também são muito úteis para modelar situações em que, cada ponto do processo necessita de uma tomada de decisão entre duas direções.

Entre árvores binárias existem:

1. **Árvore estritamente binária**: Onde cada nó pai tem uma subárvore completa;
2. **Árvore binária quase completa**: Aquelas que a altura entre uma subárvore e outra se difere por apenas 1;
3. **Árvore completa**: É uma árvore estritamente binária em que todos os nós folhas estão no mesmo nível;
	1. Para saber a quantidade de nós em um nível *n* faremos *2^n*;
	2. Se um nível *n* tem *m* nós, o nível abaixo *(n+1)*, terá *2m*;
	3. Uma árvore de altura *h* possui *(2^h)-1* nós
 
 | Nível (n)     | 0   | 1   | 2   |
 | ------------- | --- | --- | --- |
 | Número de nós | 1   | 2   | 4   | 
 | Potência      | 2^0 | 2^1 | 2ˆ2 |

## Estrutura de uma árvore
Como vimos nas definições do que é uma árvore, soubemos que ela é compostas por nós, esses nós são iguais aos de uma lista encadeada, porém, uma duplamente ligada. Onde teremos uma referência para um nó a esquerda e outra referência ao nó da direita.

`binary_tree.h`
```c
typedef struct node* tree_t;

tree_t* tree_create();
void tree_free(tree_t *root);
int tree_insert(tree_t *root, int num);
int tree_remove(tree_t *root, int num);
int tree_is_empty(tree_t *root);
int tree_height(tree_t *root);
int tree_node_len(tree_t *root);
int tree_get(tree_t *root, int num);
int tree_pre_order(tree_t *root);
int tree_in_order(tree_t *root);
int tree_pos_order(tree_t *root);
```

`binary_tree.c`
```c
#include <stdio.h>
#include <stdlib.h>
#include "binary_tree.h" 

struct node {
	int info;
	struct node *left;
	struct node *right;
}
```

> Definimos o tipo `*tree_t` para facilitar a leitura, mas poderíamos utilizar `struct node**`, `tree_t` é um ponteiro para ponteiro do tipo `struct node*`

> Decidimos separar a implementação da estrutura no arquivo com extensão .c para encapsular e modularizar.

## Criando uma árvore
A criação de uma árvore binária de busca é basicamente feita alocando um espaço na memória para armazenar um nó que será sua raiz.

```c
tree_t* tree_create() {
	tree_t *root = (tree_t*)malloc(sizeof(tree_t));
	if (root != NULL) {
		*root = NULL;
	}
	return root;
}
```

> Lembre-se: O tipo `tree_t` é um ponteiro do tipo `struct node*`, portanto ao criarmos `tree_t *root` estamos criando um ponteiro para um ponteiro `struct node**`. Então quando fazemos a referência a `*root` estamos na verdade acessando o ponteiro que esta dentro do ponteiro.

## Liberando uma árvore da memória
A destruição de uma árvore deve ser feita nó por nó para que toda a árvore seja liberada da memória. Este processo pode ser feito utilizando recursão ou iterando nó a nó.

### Utilizando função recursiva

```c
void node_free(struct node *node) {
	if (node == NULL) {
		return NULL;
	}
	node_free(node->left);
	node_free(node->right);
	free(node);
	node = NULL;
}

void tree_free(tree_t *root) {
	if (root == NULL) {
		return;
	}
	node_free(*root);
	free(root);
}
```

## Inserir nós

Uma árvore binária é otimizada para que a busca fique mais rápida, onde a quantidade de nós percorridas serão menores do que em uma lista encadeada. De fato, se uma árvore estiver com muitos nós pendendo somente à esquerda ou à direita, teremos uma árvore desbalanceada, e seu pior caso será *O(n)* onde em uma árvore bem balanceada teremos o caso médio como *O(log n)*.

Para inserir um elemento na árvore binária de busca temos que sempre comparar os nós para decidir se irá para a esquerda ou direita.

Se temos uma árvore com nós que armazenam números, comparamos se o nó a ser incluso é menor ou maior do que o nó raiz comparado.

Se for maior inserimos á direita, se for menos inserimos a esquerda.

Podemos implementar esse algoritmo descrito acima utilizando funções recursivas ou de modo iterativo. Ambos terão seu objetivo alcançados, porém, precisamos analisar bem qual será o propósito e a proporção dos dados que serão estruturados, pois, utilizando funções recursivas podemos sobrecarregar a região da memória responsável por empilhar e desempilhar as chamadas de funções.
### Inserindo com função recursiva

```c
int tree_insert(tree_t *root, int num) {
   if (*root == NULL) {
        *root = (tree_t*)malloc(sizeof(tree_t));
        (*root)->value = num;
        (*root)->left = NULL;
        (*root)->right = NULL;
		return 1;
    }

	if (num < (*root)->value){
		tree_insert(&((*root)->left), num);
		return 1;
	}
	
	if (num > (*root)->value){
		tree_insert(&((*root)->right), num);
		return 1;
	}
	return 0;
}
```

### Inserindo com iterações

```c
int tree_insert(tree_t *root, int num) {
    node_t *aux = *root;
    // Faremos a variavel LOCAL root ser o nó esquerdo ou direito
    while(aux) {
		if (num == aux->value){
			return 0;
		}
        if (num < aux->value){
            root = &aux->left;
        }else{
            root = &aux->right;
        }
        // Modificamos o aux para ser a nova raiz
        aux = *root;
    }

    // Criamos o novo nó
    aux = (struct node*)malloc(sizeof(struct node));
    aux->value = num;
    aux->left = NULL;
    aux->right = NULL;

    // Adicionamos o no ao conteúdo de root
    *root = aux;
	return 1;
}

```

## Percorrer elementos da árvore
Existem casos onde precisamos percorrer os elementos da árvore seja para alterar seus dados ou simplesmente imprimir todos os nós.

Existem três maneiras de se percorrer os nós:
1. **Pré-order**: Percorre os elementos na ordem em que foram inclusos, iniciando pela sua raiz.
2. **In-order**: Como o nome já diz, percorre os nós de uma forma ordenada, sendo assim, nós a esquerda da raiz, depois a raiz e por último os nós da direita da raiz.
3. **Pós-order**: Essa ordem é onde percorremos primeiro os elementos da esquerda da raiz, depois da direita e por fim a própria raiz.

### Imprimindo os elementos com função recursiva

```c
void tree_pre_order(tree_t *root) {
	if (root == NULL) {
		return;
	}

    if(*root) {
        printf("%d ", (*root)->value);
        tree_pre_order(&((*root)->left));
        tree_pre_order(&((*root)->right));
    }
}

void tree_in_order(tree_t *root) {
	if (root == NULL) {
		return;
	}

    if(*root) {
        tree_in_order(&((*root)->left));
        printf("%d ", (*root)->value);
        tree_in_order(&((*root)->right));
    }
}

void tree_pos_order(tree_t *root) {
	if (root == NULL) {
		return;
	}
    if(*root != NULL) {
        tree_pos_order(&((*root)->left));
        tree_pos_order(&((*root)->right));
        printf("%d ", (*root)->value);
    }
}
```

## Coletando um elemento da árvore

Para resgatar um elemento da árvore precisamos seguir a lógica que a árvore binária de busca implementou na inclusão dos itens. Onde, valores maiores que a sua raiz vão ser armazenadas na subárvore da direita e menores serão armazenados na subárvore da esquerda.

A busca fará essas comparações até chegar ao node cujo o valor é igual ao que foi buscado.

### Utilizando recursão

```c
tree_t* tree_get(tree_t *root, int num){
    if (!tree_is_empty(root)) {
        if ((*root)->value == num) {
            return *root;
        } else if(num < (*root)->value) {
            return tree_get(&((*root)->left), num);
        } else {
            return tree_get(&((*root)->right), num);
        }
    }
    return NULL;
}

```


## Coletando metadados sobre a árvore

Em alguns casos é necessário para nosso programa saber alguns metadados de nossa árvore para tomar uma decisão lógica. Entre esses metadados mais comuns estão:
1. Se a árvore esta vazia ou não
2. A quantidade de nós
3. A altura da árvore

### Verificando se a árvore esta vazia

Para saber se temos uma árvore vazia, basicamente precisamos verificar se o nó raiz contém filhos, caso este nó aponte para a constante NULL saberemos que a árvore esta vazia.

```c
int tree_is_empty(tree_t *root) {
	if (root == NULL) {
		return 1;
	}

	if (*root == NULL) {
		return 1;
	}
	return 0;
}
```


### Verificando a quantidade de nós da árvore

Para saber quantos nós nossa árvore contém, não temos outro meio a não ser contar nó a nó da esquerda e direita depois somar com a própria raiz.

```c
int tree_node_len(tree_t *root) {
	if (!tree_is_empty(root)){
		int total_left = tree_node_len(&((*root)->left));
		int total_right = tree_node_len(&((*root)->right));
		return total_left + total_dir + 1;
	}
	return 0;
}
```

### Verificando a altura da árvore
A altura da árvore é dada pela subárvore mais alta, ou seja, aquela subárvore que contém a folha mais distante.

```c
int tree_height(tree_t *root){
	if(!tree_is_empty(root)){
		int left_height = tree_height(&((*root)->left));
		int right_height = tree_height(&((*root)->right));
		if (left_height > right_height) {
			return left_height + 1;
		}
		return right_height + 1;
	}
	return 0;
}
```

## Removendo nós de uma árvore binária de busca

A remoção de nós em uma árvore binária de busca exige algumas regras para que a busca não seja impactada e que suas características permaneçam a mesma.
Quando removemos um nó, e este é uma folha, podemos simplesmente remove-lo sem impacto. Mas quando este nó a ser removido é um *nó interno*, ou seja, tem pelo menos um filho, precisaremos atualizar a referência do *nó pai*. Seguiremos alguns conceitos nestes casos.
1. Se o nó tem apenas um filho, temos que fazer com que o pai do nó a ser removido aponte para este nó filho do que será removido.
2. Se o nó tem dois filhos, temos que atualizar o nó pai deste que será removido para apontar para um nó filho, porém, como temos duas direções temos que escolher entre o nó filho da esquerda ou direita. Se escolhermos o nó:
	1. Esquerdo: Temos que utilizar a última folha da subárvore mais a direita.
	2. Direito: Temos que utilizar a última folha da subárvore mais a esquerda.
 Fazendo essa troca de referencia, o nó a ser removido agora será um nó folha, no qual podemos simplesmente remove-lo sem impacto.

### Utilizando função recursiva

```c
tree_t* tree_remove(tree_t *root, int num){
	if (tree_is_empty(root)) {
		return NULL;
	}

    if ((*root)->value == num) {
        // Verifica se é uma folha
        if ((*root)->left == NULL  && (*root)->right == NULL) {
            free(*root);
            return NULL;
        } 

        // Verifica se tem dois filhos
        if ((*root)->left != NULL && (*root)->right != NULL) {
			// Escolhemos a subárvore da esquerda
            struct node *aux = (*root)->left;
            while(aux->right != NULL) {
				/*
				* Temos que "andar" até a última folha da direita 
				* para utiliza-la como referencia para o nó pai 
				* do nó que será removido
				*/
                aux = aux->right;
            }
            root->value = aux->value;
            aux->value = num;
            (*root)->left = tree_remove(&((*root)->left), num);
            return root;
        }
        
        // Tem somente um filho na esquerda ou direita
        node_t *aux;
        if ((*root)->left != NULL) {
            aux = (*root)->left;
        } else {
            aux = (*root)->right;
        }
        free(*root);
        return aux;
    } 

    if (num < root->value) {
        root->left = tree_remove(root->left, num);
    } else {
        root->right = tree_remove(root->right, num);
    }
    return root;
}
```

## Relacionados
- [Filas com lista dinamicamente encadeadas](/posts/20230913_Filas_com_lista_dinamicamente_encadeadas)
- [Árvores](/posts/20230926_Arvores)
- [Árvores balanceadas](/posts/20231001_Arvores_balanceadas)

## Referências
- Ito, Olavo Tomohisa. **Estruturas de Dados / Olavo Tomohisa Ito.** – São Paulo: Editora Sol, 2022.
- Backes, André Ricardo. **Algoritmos e estrutura de dados em linguagem C**. 1 Edição. Rio de Janeiro: LTC, 2023.
- Gaspar, Wagner. Árvore Binária de Busca. **Programe seu futuro**, 16, Junho de 2021. Curso de Programação C Completo. Disponível em: <https://www.youtube.com/playlist?list=PLqJK4Oyr5WSii6sFzwC6xTuhfALuJVEKT>. Acesso em: 28, setembro de 2023.