---
date: 2023-10-01
author: André Matias
tags:
  - estrutura-de-dados
title: Árvores balanceadas
---

Antes de entender o que é uma árvore balanceada, precisamos entender melhor o que é uma árvore desbalanceada e o que isso pode significar no algoritmo do nosso programa.

Árvores binárias foram criadas justamente para resolver casos onde a busca deveriam ser realizadas rápidas utilizando o conceito de [[Dividir para conquistar|dividir para conquistar]], onde elementos que são maiores do que sua raiz estarão na metade a direita e menores na direção oposta, eliminando assim metade do conteúdo e otimizando a busca para que no pior caso fique como *O(log n)*.

Uma árvore desbalanceada é quando na inclusão ou remoção de nós a árvore fica com subárvores pendendo somente para uma direção. Portanto, a busca será comprometida, uma vez que, se parecerá com uma lista ligada, onde, a busca é linear *O(n)*.

Assim, uma árvore balanceada é aquela que a altura das subárvore se difere no máximo em 1 nível. Essa diferença é conhecida como *fator de balanceamento (fb)* de uma subárvore.

Para resolver estes casos de balanceamento, foram criados alguns algoritmos, entre eles os mais conhecidos são: 
1. Árvores AVL
2. Árvores Red Black ou também conhecida como Rubro Negra
3. Árvores 2-3
4. Árvores 2-3-4
5. Árvores B, B+ e B*

## Veja também
- [Árvores](/posts/20230906_Arvores)
- [Árvores binárias e árvores binárias de busca](/posts/20230927_Arvores_binarias_e_arvores_binarias_de_busca)

## Referências
- Backes, André Ricardo. **Algoritmos e estrutura de dados em linguagem C**. 1 Edição. Rio de Janeiro: LTC, 2023.
- Akita, Fábio. **Árvores: O começo de TUDO | Estruturas de Dados e Algoritmos**, AkitaOnRails, 06 de Abril de 2021, Disponível em: <https://www.akitaonrails.com/2021/04/06/akitando-95-arvores-o-comeco-de-tudo>, Acessado em: 01 de Outubro de 2023. 