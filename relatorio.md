# **Identificação do Grupo**

## Heitor Sias Leite Nº00577397

## Marcelo Gonda Stangler Nº00587562

# **Visão Geral de Implementação**

Todos os algoritmos de busca implementados seguiram um padrão de busca em grafos, variando principalmente a estrutura de dados utilizada para tratar a fronteira.

- **DFS**: Fronteira representada por pilha (util.Stack), garantindo que o nó expandido seja o recém descoberto.

- **BFS**: Fronteira representada por fila (util.Queue), expandindo nós que distam de camadas diferentes, permitindo otimização de espaço.

- **UCS**: Fronteira representada por fila de prioridades (util.PriorityQueue), orientada pelo custo acumulado
  g(n) do caminho para cada estado, visando o menor custo.

- **A\***: Fronteira representada por fila de prioridades (util.PriorityQueue), assim como o UCS considera o custo acumulado e complementa a busca com uma heurística h(n) que é somada ao custo. Vale ressaltar que quando a heurística é desconsidera o A\* se torna o UCS.

Em geral, todos os casos armazenam os elemento de fronteira ficam armazenados em tuplas (estado, informações de ações [custo acumulado etc])

Os algoritmos de A\* e UCS possuem a lógica de marcar um estado como visitado somente quando são retirados da fronteira, o que é necessário para garantir que estados com custo piores sejam marcados no lugar só por prezarem pelo mesmo estado final. Essa lógica é utilizada para o dfs, mesmo que não mude muito nesse caso, porém o bfs o estado é dado como visitado assim que o estado entra na fronteira.

# **Representação de estado do CornersProblem**

O estado do CornersProblem foi representado por uma tupla:
estado = (posicao_atual, booleano_cantos)

posicao_atual é uma tupla com valores de x e y indicando a posição do pacman no momento. É o mesmo que vinha sendo utilizado.

booleano_cantos é uma tupla contendo os (True or False) referente a cada um dos 4 cantos e o valor diz se foi ou não visitado.

- **Porque é suficiente?**

O teste do IsGoalState pode ser verificado ao checar se todos os cantos estão associados ao True, assim garantindo o fim.

- **Porque é mínima?**

Contém apenas informações diretamente relevantes para a resolução do problema e não gera valores ambíguos, além de não considerar informações inúteis do GameState ou do jogo em si, por exemplo, posição de fantasmas, localização de alimentos extras, entre outros, de acordo com o que foi exigido na descrição do trabalho,

# **Heurística Implementadas**

## **CornersProblem**

Foi utilizada a distância de Manhattan da posição atual até o canto mais próximo não visitado mesmo que essa distância desconsidere a parede. O intuito é a cada passo dado, atualizar esse cálculo sendo feito o caminho mais barato até que todos os cantos sejam visitados. A heurística é utilizada para estimar o custo desse passo a passo até o canto mais próximo e após isso calcular o mesmo para esse canto, visto que ele será a nova posição atual.

- **Pseudocódigo**

```
Total <- 0
posicao_atual <- posicao_do_estado
restantes <- lista de cantos não visitados

While restantes não estiver vazia:
    canto_mais_proximo <- canto_menor_dManh_posicao_atual
    total <- total + disManh (posicao_atual, canto_mais_proximo)
    posicao_atual <- canto_mais_proximo
    pop canto_mais_proximo

return total
```

- **Admissibilidade**

A distância de Manhattan entre dois pontos no labirinto é sempre menor ou igual a distância real entre eles de fato, uma vez que as paredes podem aumentar essa valor, que é o mínimo pois considera as distâncias em linha reta. Dessa forma, como a heurística apenas considera a soma de todas essa distâncias entre os cantos a serem alcançados e para cada caminho o custo real é sempre maior ou igual ao custo especulado, a heurística nunca superestima o custo sendo assim admissível. A consistência nesse caso é justificada pelo fato de que a distância de Manhattan é recalculada a cada passo, o que significa que qualquer movimento gera um cálculo novo que desconsidera no máximo uma unidade para a heurística, visto que a distância de Manhattan opera com distância em linha reta, as mesmas dos passos do pacman.

- **Dados**

Tempo: 0,0 segundos

Nós expadindo: 692

Custo do caminho: 106

A heurística implementado junta dois fatores ao mesmo tempo, com o intuito de minimizar o custo para consumo de toda comida no labirinto.

### **Evolução do Desenvolvimento (Tentativas)**

Durante a criação da `foodHeuristic`, fizemos 3 tentativas até atingir o melhor nível no labirinto `trickySearch`:

1. **Maior Distância de Manhattan:** Inicialmente, usamos apenas a distância de Manhattan do Pacman até a comida mais distante. Funcionou e foi admissível, mas expandiu **9.551 nós** (pontuação 3/4).
2. **Distância até a mais próxima + distância entre extremidades:** Na segunda tentativa, somamos a distância até a comida mais próxima com a distância dessa comida até a mais distante. A estimativa ficou mais justa, reduzindo para **8.617 nós** (pontuação 4/4).
3. **MST com Distância Real (escolhida):** Depois, combinamos a Árvore Geradora Mínima (MST) das comidas utilizando a distância real pelo labirinto (via BFS e memoização no `heuristicInfo`), somada à distância do Pacman até a comida mais próxima. Isso reduziu a busca para apenas **255 nós expandidos** (pontuação 5/4).


- **Peso de MST**

Considera a árvore que contém as comidas restantes no labirinto tendo a distância de cada par de comida como as arestas dessa árvore, assim a MST representa o rascunho do caminho mínimo para o consumo de tudo para obter o estado final.

- **Distância entre posição atual e Comida restante**

Estima o custo do pacman para a comida mais próxima entre as restantes.

A heurísitica então retorna a soma desses componentes.Como o cálculo das distância é feito via BFS, problem.heuristicInfo guarda as informações em cache, evitando cálculos extras e desnecessários.

- **Pseudocódigo**

```
If comida == 0: return 0

Constrói mst com pontos de comida restante
peso_mst <- soma arestas da mst via algoritmo de prim

menor_dist <- menor distância real entre o pacman e cada comida restante

return menor_dist + peso_mst
```

- **Admissibilidade**

Para alcançar o objetivo são necessárias duas coisas: chegar a primeira comida e percorrer todas as comidas. Como todas as distâncias entre cada comida e pacman são distâncias reais, o custo até cada comida logo é sempre maior ou igual a variável menor_dist, visto que ela busca a comida mais próxima considerando a distância real até ela. Além disso, como a MST é a árvore mínima para determinado caminho ela já busca o menor caminho entre cada comida que deverá ser consumida pelo pacman e, portanto, o peso dessa árvore é o custo mínimo para se percorrer esse caminho. Assim, garantindo que tanto o custo para a comida mais próxima, como o trajeto para as demais é menor ou igual que o custo real a ser traçado, a heurística é admissível. Fora isso, a consistência da heurística se firma no fato de que apenas distâncias reais são utilizadas, o que é calculado via BFS, evitando assim problemas com estimativas, ou seja, mudanças na trajetória não irão levar a valores irregulares da heurística, visto que qualquer passo irá alterar em no máximo 1 unidade para o cálculo da distância real entre o pacman e a comida. Além de que consumir uma comida apenas irá remover um nó do conjunto, sendo recalculada a mst, o que não altera o fato de que o peso da mst continuará sendo o mínimo entre as comidas restantes.

## **Dados**

- TestSearch

Nós expandidos: 7

Tempo: 0,0 segundos

Custo do caminho: 7

- TrickySearch

Nós expandidos: 255

Tempo: 0,1 segundos

Custo do caminho: 60

# **Uso de IA**

- Nosso grupo utilizou da llm Claude, com o intuito de compreender bem o código fornecido, além de compreender como fazer relatórios em arquivos do tipo .md e para uma linha de código do remaining, utilizada durante o problema CornersHeuristic (Problema 6), a linha estará em destaque abaixo.

```Python
 remaining = [corner for corner, visited in zip(corners, visited_corners) if not visited]
```
