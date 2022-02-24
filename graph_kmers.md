
# Finding all kmers in a sequence graph


A sequence graph contains of **nodes** (sequences) that are connected by **edges**. The edges connect sequences into paths that can represent individual genomes.

Example:
* One person A has (on a very short chromosome) the sequence `AAACTTT`
* Another person B has on the same chromosome the sequence `AAAGTTT`.
* The only difference is thus that person A has C on position 3 while the other has G (a *SNP* from C to G)

We can represent this variation with a graph with 4 nodes:
* node 1 with sequence `AAA`
* node 2 with sequence `C`
* node 3 with sequence `G`
* node 4 with sequence `TTT`

... and four edges:
* Edge from node 1 to node 2
* Edge from node 1 to node 3
* Edge from node 2 to node 4
* Edge from node 3 to node 4

Drawing with node ID's:
```
    3 
  /   \
1 - 2 - 4
```


Drawing with sequences:
```python
      G 
    /   \
AAA - C -  TTT
```

This information can also be represented using dict structures in Pytyhon, e.g. like this:

```python
nodes = {1: "TTT", 2: "C", 3: "G", 4: "TTT"}
edges = {1: [2, 3], 2: [4], 3: [4]}
```

### Finding kmers from this graph
In order to find out whether reads support variants in a graph, we want to create an index (or a collection of some sort) of all the possible kmers that can be found in a graph. For each kmer, we want to know the start node and start position on that node, and the nodes it covers. For instance, for the example graph above, we have these 3-mers:

* AAA -- start node 1, position 0 -- nodes 1
* AAG -- start node 1, position 1 -- nodes 1,3 
* AAC -- start node 1, position 1 -- nodes 1,2 
* ACT -- start node 1, position 2 -- nodes 1,2,3 
* ... and so on

Note that some kmers may exist multiple places in the graph, but that shouldn't be a problem. We just want to store a list of all the non-unique kmers togheter with start positions and nodes (grouping similar kmers into an index is a separate problem).


#### Task
Implement code that given a graph finds all the kmers of a given length.

* It may be a good idea to implement a Graph-class with some utility methods. The class can e.g. take the node and edge dicts as arguments in the init.

Make some unit-tests to check whether things are working.

Example:

```python
def test_graph_with_one_node_and_no_edges():
    g = Graph({1: "AACCCT"}, {})
    
    kmers = find_kmers(g)
    
    # check that AAC,start node 1 position 0, nodes 1 and so on have been found
    assert ...
    
    
def test_graph_with_two_nodes_and_one_edge():
    g = Graph({1: "AACC", 2: "ACGA"}, {1: [2]})
    # ...

```




