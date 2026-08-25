# Relatório - Trabalho 1: Busca

## 1. Identificação do Grupo
* Heitor Sias Leite
* Marcelo Gonda Stangler (587562)

---

## 2. Visão Geral da Implementação (Tarefas 1 e 3)

### Tarefa 1: Busca em Profundidade (DFS)
* **Algoritmo:** A busca em profundidade foi implementada utilizando a estrutura de dados LIFO (Pilha, via `util.Stack`).
* **Busca em Grafo:** Para evitar loops e ciclos infinitos nos labirintos, a busca foi realizada em grafo, rastreando estados já expandidos em um conjunto (`visited = set()`). Um estado é adicionado ao conjunto apenas quando é desempilhado (expandido), garantindo que caminhos mais curtos não sejam descartados precocemente na fronteira.
* **Retorno:** Retorna a lista de ações que leva do estado de início até a meta.

### Tarefa 3: Busca de Custo Uniforme (UCS)
* **Algoritmo:** A UCS foi estruturada de forma semelhante à DFS, porém utilizando uma Fila de Prioridades (`util.PriorityQueue`).
* **Lógica:** O algoritmo retira sempre o estado com o menor custo acumulado da fronteira. A prioridade de cada elemento na fila é definida pelo seu custo acumulado desde a partida (`new_cost = cost + step_cost`). O critério de parada ocorre imediatamente no desempacotamento/desenfileiramento do nó objetivo, garantindo a otimalidade.

---

## 3. Representação de Estado do `CornersProblem` (Tarefa 5)
* **Estrutura do Estado:** `(posicao_atual, cantos_visitados_tuple)`
  * `posicao_atual`: Tupla `(x, y)` contendo as coordenadas do Pacman.
  * `cantos_visitados_tuple`: Uma tupla de 4 booleanos `(bool, bool, bool, bool)`, onde cada índice indica se o canto correspondente da lista de cantos já foi visitado.
* **Justificativa de Minimalidade:** 
  Apenas a coordenada `(x, y)` do Pacman não é suficiente para representar o estado da busca, pois o Pacman pode visitar a mesma posição física mais de uma vez em momentos diferentes da busca (com progressos diferentes no número de cantos visitados). 
  A tupla de 4 booleanos é a estrutura **mínima** necessária para que o algoritmo diferencie estados com as mesmas coordenadas geográficas, mas com cantos visitados distintos (ex: estar em `(1,1)` com 0 cantos visitados é um estado diferente de estar em `(1,1)` com 3 cantos visitados).
* **Consistência:** Representar o estado como tuplas (que são imutáveis em Python) garante que os estados sejam "hasháveis", permitindo inserção rápida e correta no conjunto de visitados.

---

## 4. Heurísticas Implementadas

### Heurística dos Cantos (`cornersHeuristic` - Tarefa 6)
* [Esta seção deve ser completada pelo integrante responsável pela implementação da Tarefa 6]

### Heurística de Comida (`foodHeuristic` - Tarefa 7)

Para resolver o `FoodSearchProblem` usando o algoritmo A*, foram desenvolvidas três tentativas de heurísticas na busca da melhor eficiência e nota máxima.

#### Tentativa 1: Distância de Manhattan até a Comida mais Distante
* **Lógica:** Como o Pacman precisa comer todas as comidas, o custo até a meta será no mínimo igual à distância entre o Pacman e a comida que está mais distante dele.
* **Código Utilizado:**
  ```python
  position, foodGrid = state
  foodList = foodGrid.asList()
  if not foodList:
      return 0
  return max(abs(position[0] - food[0]) + abs(position[1] - food[1]) for food in foodList)
  ```
* **Justificativa de Admissibilidade:** A distância real percorrida no labirinto com paredes é sempre maior ou igual à distância em linha reta (Manhattan). Como o Pacman é obrigado a visitar a comida mais distante para concluir o objetivo, a distância até ela é um limite inferior seguro para o custo total.
* **Desempenho:** **9.551 nós expandidos** no cenário `trickySearch` (Nota: 3/4).

#### Tentativa 2: Distância da mais Próxima + Distância da mais Próxima até a mais Longe
* **Lógica:** Um limite inferior mais robusto é a soma do custo para ir até a comida mais próxima ($f_{\text{close}}$) e, a partir de lá, a distância de Manhattan até a comida mais distante ($f_{\text{far}}$).
* **Código Utilizado:**
  ```python
  position, foodGrid = state
  foodList = foodGrid.asList()
  if not foodList:
      return 0
  distances_to_food = [abs(position[0] - food[0]) + abs(position[1] - food[1]) for food in foodList]
  min_dist = min(distances_to_food)
  closest_food = foodList[distances_to_food.index(min_dist)]
  max_dist = max(distances_to_food)
  furthest_food = foodList[distances_to_food.index(max_dist)]
  dist_between = abs(closest_food[0] - furthest_food[0]) + abs(closest_food[1] - furthest_food[1])
  return min_dist + dist_between
  ```
* **Justificativa de Admissibilidade:** O Pacman precisa ir até pelo menos uma comida (no mínimo a mais próxima) e depois cobrir a distância entre essa comida e a última restante. A soma dessas duas distâncias sob a métrica Manhattan é garantidamente menor ou igual ao custo real do caminho.
* **Desempenho:** **8.617 nós expandidos** no cenário `trickySearch` (Nota: 4/4 - Nota Máxima).

#### Tentativa 3: Árvore Geradora Mínima (MST) de Distância Real pelo Labirinto (Cacheada)
* **Lógica:** A menor árvore que conecta todas as comidas restantes (MST) usando a distância real do labirinto (considerando as paredes) é um limite inferior excelente e muito preciso para o caminho de cobertura das comidas. Somando isso à distância real do Pacman até a comida mais próxima, mantemos a consistência e a admissibilidade perfeitas.
* **Código Utilizado:**
  ```python
  position, foodGrid = state
  foodList = foodGrid.asList()
  if not foodList:
      return 0
  def getDist(p1, p2):
      key = (min(p1, p2), max(p1, p2))
      if key not in problem.heuristicInfo:
          problem.heuristicInfo[key] = mazeDistance(p1, p2, problem.startingGameState)
      return problem.heuristicInfo[key]
  n = len(foodList)
  visited = [False] * n
  min_dist = [float('inf')] * n
  min_dist[0] = 0
  mst_weight = 0
  for _ in range(n):
      u = -1
      for i in range(n):
          if not visited[i] and (u == -1 or min_dist[i] < min_dist[u]):
              u = i
      visited[u] = True
      mst_weight += min_dist[u]
      for v in range(n):
          if not visited[v]:
              d = getDist(foodList[u], foodList[v])
              if d < min_dist[v]:
                  min_dist[v] = d
  min_pacman_dist = min(getDist(position, food) for food in foodList)
  return mst_weight + min_pacman_dist
  ```
* **Otimização:** Como o cálculo de distâncias reais com busca interna (BFS) é custoso, todas as distâncias reais calculadas pela função `mazeDistance` são armazenadas no dicionário global `problem.heuristicInfo` (Memoization). Isso garante que a distância entre qualquer par de comidas seja computada no máximo uma única vez, servindo como cache $O(1)$ para as milhares de avaliações seguintes da heurística.
* **Justificativa de Admissibilidade e Consistência:** Qualquer caminho conectando todas as comidas é também uma árvore geradora do grafo de comidas. Portanto, o peso da Árvore Geradora Mínima (MST) é um limite inferior para o caminho ótimo. Ao separar a posição do Pacman no cálculo (medindo a distância do Pacman apenas ao nó da MST mais próximo), impedimos que a heurística varie por mais de 1 unidade por passo, garantindo consistência.
* **Desempenho:** **255 nós expandidos** no cenário `trickySearch` (Nota: 5/4 - Crédito Extra).

---

## 5. Uso de Ferramentas de IA
As seguintes ferramentas de IA foram utilizadas no desenvolvimento deste trabalho:
* **Antigravity (Google DeepMind):** Utilizado para pair-programming, auxiliando na verificação de consistência e admissibilidade matemática das heurísticas, na estruturação do estado para o CornersProblem e na programação das estruturas de busca genéricas (DFS, UCS).
