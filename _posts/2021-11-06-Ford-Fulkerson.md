---
layout: post
title:  Graph Thoery; Ford Fulkerson
categories: [Algorithms, Graph Theory]
---
## Maximum Flow Problem

The maximum flow problem consists basically in finding a flow which has the greatest capacity in a graph flow. 

Clarifying, given a directed graph (a graph which its edges has directions) and each edge has a certain capacity (C), you must find the flow given a vertex source (S) and the sink (T), that respects two constraints:

- Flow on an edge doens't exceed the edges capacity;
- Incoming flow must be equal to outgoing flow for every vertex except source (T) and sink (T).

## Motivation

Solving this problem Is useful to any situation where there's the needing to find the ''bottleneck'' of a network. Edges could be streets and the flow could be cars, or pipes and water, eletric cables and eletricy, and so on. 

It's crucial to know how to use the network full potential and avoid something goes wrong.

## Applying Ford Fulkerson

```python
from typing import List

class DirectedGraph:

  def __init__(self, residual_graph: List[List]):
    self.residual_graph = residual_graph
    self.row = len(residual_graph)

  def __breadth_first_search(self,
                             source: int,
                             sink: int,
                             parent: List) -> bool:
 
        queue = []
        visited = [False] * self.row
 
        queue.append(source)
        visited[source] = True
        
        while queue: 
            u = queue.pop(0)
 
            # get u neighbords
            for index, value in enumerate(self.residual_graph[u]):
                
                # set not visited neighbords as visited and enqueue it
                # aka visited[index] is False and value > 0
                if not visited[index] and value: 
                    queue.append(index)
                    visited[index] = True
                    parent[index] = u

                    # If find a connection to sink, finish BFS.
                    if index == sink:
                        return True
 
        return False

  def ford_fulkerson(self, source: int, sink: int) -> int:
    '''
    returns the max flow given a directed graph

    >> g.ford_fulkerson(source=0, sink=5)
    23
    '''
    max_flow = 0
    parent = [-1] * self.row

    while self.__breadth_first_search(
        source=source,
        sink=sink,
        parent=parent):

        path_flow = float("Inf")
        s = sink
        while(s !=  source):
            path_flow = min (path_flow, self.residual_graph[parent[s]][s])
            s = parent[s]

        max_flow +=  path_flow
        v = sink
        while(v !=  source):
            u = parent[v]
            self.residual_graph[u][v] -= path_flow
            self.residual_graph[v][u] += path_flow
            v = parent[v]

    return max_flow

if __name__ == '__main__':
  graph = [[0, 16, 13, 0, 0, 0],
        [0, 0, 10, 12, 0, 0],
        [0, 4, 0, 0, 14, 0],
        [0, 0, 9, 0, 0, 20],
        [0, 0, 0, 7, 0, 4],
        [0, 0, 0, 0, 0, 0]]

  g = DirectedGraph(residual_graph=graph)

  source = 0
  sink = 5

  print(f"Maximum possible flow is {g.ford_fulkerson(source, sink)}")
```

References

[https://www.geeksforgeeks.org/ford-fulkerson-algorithm-for-maximum-flow-problem/](https://www.geeksforgeeks.org/ford-fulkerson-algorithm-for-maximum-flow-problem/)

[https://www.topcoder.com/thrive/articles/Maximum Flow: Part One](https://www.topcoder.com/thrive/articles/Maximum%20Flow:%20Part%20One)