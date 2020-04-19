
# Overview of Graph Problems

In graphs vertices model entities and edges model relationships between entities. The point of graph algorithms is:

- Take things that are domain specific
- Model it into a graph
- Solve it using a generic graph algorithms

Classes of problems we can solve using graphs:

## Type 1: Shortest path problems:
  - Can be undirected or directed: This doesnt matter much and same algo can be used for either one
  - Can be unweighted or weighted: Here the distinction matters. Unweighted uses BFS; Weighted uses Dijkstra

## Type 2: Connected Components:
  - Things that has items/elements that are connected to each other. Examples include number of islands problem, Evaluate division, Photoshop's magic wand etc
  - Here we have a couple of different algorithms that we can use such as DFS

How Connected Components problems are different from Shortest Paths problems:

- Unlike shortest paths here we only need to know that a path exists. 
- We care about all the vertices unlike shortest paths problem where we have a fixed source and destination
- Weights are ignored because it is about connectivity
  - Directed/undirected graphs are a bit trickier because A may be connected to B but B may not be connected to A. In this case we say B is downstream from A or the level of B is lower than that of A.

## Type 3: Dependency Ordering
  - Problems where if there is an edge from A to B, there is NEVER an edge from B to A. This is strictly directional.
  - These are directed acyclic (no cycles between nodes) graph problems and is solved using Topological Sort
  - If we have to set an ordering of entities or find out if something has a dependency on something else then topological sort can be considered.
 
This is shown in the following image:
![dep ordering](https://i.imgur.com/oX4nniz.png)

- Here D depends on B and C both of which depend on A. So there are 2 ways to do D: Either do [A,B,C,D] or do [A,C,B,D]
- Similarly the graph can be represented in the form of a layering of the things that can be done in parallel. Here A is done first. B, C are in the same layer and can be done parallel. D must be done last
- It is mostly used for unweighted problems but we may have weighted variations. So we can ask something like what is the critical/slowest path (26) etc.

## Type 4: Spanning Trees

- Given a graph what is the maximum number of edges that can be deleted so the graph remains connected
- Done for undirected graphs

## Type 5: Bipartite Matching

- Aim is to have a 1:1 correspondence between set 1 and set 2
- 'n' jobs and 'n' people. A person can do only 1 job. Can we perform a matching between the 2? Problem: Not every person can do every job. Can we perform a matching given those conditions?
  - If edges have weights can you maximize weights?

  
# Algorithms

## BFS

- Create queue and add source node to it
- Have a map for distances and add {source, 0} to it
- While queue is not empty pop node
  - For all neighbors of node not in map
  - Add neighbour to map with distance node+1

**The BFS Algorithm is:**

    private Map<String, Integer> shortestPath(Map<String, Set<String>> graph, String src, String destination) {
        Queue<String> q = new LinkedList<>();
        q.add(src);
        Map<String, Integer> dist = new HashMap<>();
        dist.put(src, 0);
        while(!q.isEmpty()) {
            String node = q.remove();
            for(String s: graph.get(node)) {
                if(!dist.containsKey(s)) {
                    dist.put(s, dist.get(node)+1);
                    q.add(s);
                    // OPTIONAL: When given a destination, this returns as soon as destination is found
                    if(s.equals(destination))
                        return dist;
                }
            }
        }
        return dist;
    }
    
    // Getting shortest path from map of distances. -1 if no path exists.
    int shortestPath = dist.containsKey(destination)? dist.get(destination): -1;
    
**Time complexity:**

- Time to traverse all edges of each vertex: 2 * E because each edge processed twice (A->B and B->A is same) + Processing all vertices V through the queue.
- Time complexity: 2E + V = O(V+E)
- Size of the adjacency list is V+2E so BFS is linearly processing the adjacency list.

**To save the path itself do the following:**

In the distance map add an extra attribute for parent. It should be: 
  
    Map dist <String, Pair(distance, parent)>

And when adding to dist do:

    dist.put(s, new Pair(dist.get(node)+1, node));

## Dijkstra

Same as BFS however instead of storing candidates in a queue store it in a priority queue. Priority queue must contain information for: Node, Distance, Parent.

- Create distance map and add <source, {dist:0, parent: null}>
- Create priority queue candidates and add {node: source, dist: 0, parent: null}
- While heap is not empty do:
  - node current = min element
  - If current not in map, add current to the map. For all neighbours n of current add current.dist+edge cost to heap
  - Else remove current from the heap
- Return distance map.

## DFS

We cannot find shortest path using DFS. We just find a path/all connected components.
It is the same as BFS except we use a stack instead of a queue. Because we use a stack we can use recursion (which implicitly uses a stack)
  
**Recursive DFS (exhaustive) is:**
  
    private void dfs(Map <Integer, Set<Integer>> graph, Set<Integer> visited, int current) {
        if(visited.contains(current))
            return;
        visited.add(current);
        Set<Integer> neighbours = graph.get(current);
        for(int i: neighbours) {
            dfs(graph, visited, i);
        }
    }
    
**Recursive DFS: Stop on finding destination**

    private boolean dfs(Map <Integer, Set<Integer>> graph, Set<Integer> visited, int current, int dest) {
        if(visited.contains(current))
            return false;
        visited.add(current);
        Set<Integer> neighbours = graph.get(current);
        for(int i: neighbours) {
            if(i==d)
                return true;
            if(dfs(graph, visited, i, d))
                return true;
        }
        return false;
    }

# Cycle Detection in Graphs

### Undirected Graphs: Use BFS

The definition of a cycle in an undirected graph is:
- The cycle should have 3 or more nodes
- Every node must be unique until we reach the start of the cycle.

The reason we care about this is because:
- If an undirected graph does not have a cycle then it is basically 1 or more trees. The trees can be found using connected components.
- This means we can do tree operations on the graph

Cycle detection for undirected graphs can be solved using BFS or DFS

**The process is as follows:**

Just use BFS and if you reach the same node from 2 different places then the graph has a cycle. Consider the following graph:

![BFS-cycle](https://i.imgur.com/n13qXcu.png)

- We start at A. Label A.distance = 0; A.parent = null
- Explore neighbours of A 
  - B.distance = 1; B.parent = A
  - C.distance = 1; C.parent = A
  - Add B, C to queue
- Go to B and remove it from queue
  - We get A. However A is parent of B so ignore it
  - Next we get C. C is NOT parent of B however C has already been visited and is in distance map
  - Hence it has a cycle.

So there exists a cycle in a graph if we visit a node N that has already been visited AND if the node visiting N is not a direct child of N.


### Directed Graphs: Use DFS

The idea is to use DFS and keep track of all the nodes in the stack. If we visit a node that exist in the stack, there is a cycle.

**NOTE: Just because a node is visited it does not mean there is a cycle. It HAS to be on the stack.**

Consider this graph:

![Dir-cycle](https://i.imgur.com/6I2RwCP.png)

- Here we go to A -> B -> D and marking them as visited. When B, D are popped we go to C
- C goes to D. D is already visited however because D is not on the stack, we know there is no cycle.

Hence there exists a cycle in a directed graph if:
- A node is visited
- AND the node is currently on the stack.

In order to know what is currently on the stack, keep all the contents of the stack on a set. If the set contains the current element the there is a cycle.

Hence in this scenario a node has 3 states: Visited; Visited and in Stack; Unvisited. We can also do cycle detection by storing these states within the node.

# Topological sort

Please see interview prep v2: https://github.com/anirudhrathinam94/interview-prep-v2/tree/master/Graphs
        

# Questions:

### 1. Shortest path problem 1

**Given multiple sources and multiple destinations find shortest path between any source and destination**

1. What if there are multiple destinations?

- Let dest = set of destinations
- BFS from source If we reach any destination in dest, return

2. What if there are multiple sources?

- Add all sources to queue, add all sources to distance map with a distance of 0
- Do regular BFS

### 2. Shortest path problem 2

**Given 2 sources and 2 destinations delete maximum number of nodes such that there is a path between S1, D1 and S2, D2**

Work in progress

### 3. Connected components problem 1

**Suppose we have greyscale colorings on a canvas and 2 pixels are similar if they lie within a threshold of 5. Try to simulate the magic wand tool in photoshop where if we click 1 pixel all surrounding pixels that are similar are selected.**

This is a connnected components problem where we have to label each pixel with its correcponding connected component. If we have the following grid:

![grid_cc_q](https://i.imgur.com/3dEGPnj.png)

We would iterate on every element and if element was not visited before we would do bfs and label it so it looked like this:

![grid_cc_r](https://i.imgur.com/osarV0R.png)


The adjacency list is defined as follows:
- 2 nodes must be adjacent
- The nodes must be below the threshold

So the list would look like this:

    [0,0]: {(0,1)}
    [0,1]: {(0,0), (0,2)}
    [0,2]: {(0,1), (1,2)}
    ...

### 4. Connected components problem 2

**Given a matrix of booleans where T represents forward slash and F represents backward slash find total number of regions**

Each cell in the matrix can be split into 2 nodes L/R.

### 5. Dijkstra Problem 1

**Cross a river that has a bunch of stepping stones. Each of the stepping stones has a probability of sinking. You can jump from one stepping stone to another only if the distance between them is R. Minimize probablity of sinking**

The problem can be modelled in the form of a graph as follows:

![skipping_stones_dijkstra](https://i.imgur.com/h3XJO8o.png)

After this just do a variation of dijkstra.

- Instead of minimizing the distance, maximize it since it is about probabilities
- Since it is probability instead of addition, do multiplication when updating candidate heap where distance of new neighbour is:
  - distance of current node * edge cost


### 6. Topological Sort problem 1: Alien dictionary

Please see interview prep v2: https://github.com/anirudhrathinam94/interview-prep-v2/tree/master/Graphs
