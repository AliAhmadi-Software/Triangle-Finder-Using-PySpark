<<<<<<< HEAD
# Triangle Finder Using PySpark

This project finds triangular relationships in a randomly generated graph using PySpark.

## Overview

The code generates a random graph of edges represented by pairs of nodes (u, v) and checks for triangular relationships. A triangle is formed when three nodes are mutually connected. The implementation utilizes the distributed computing capabilities of PySpark to efficiently process potentially large datasets.

## Requirements

- Python
- PySpark
- Pandas

You can install PySpark using pip:

```bash
pip install pyspark
```

## Code Explanation

### 1. Install PySpark
```python
!pip install pyspark
```
- This command installs the PySpark library, which is essential for running Spark applications.

### 2. Import Necessary Libraries
```python
from pyspark.sql import SparkSession
import pandas as pd
import random
from itertools import combinations
```
- **`SparkSession`**: The entry point to using DataFrame and SQL functionalities.
- **`pandas`**: Used for data manipulation and to create the CSV file.
- **`random`**: Generates random integers for creating the edges of the graph.
- **`combinations`**: A function from the `itertools` library to generate pairs of nodes for checking triangles.

### 3. Create the CSV File with Random Edges
```python
edges = []
for _ in range(5000):
    u = random.randint(1, 100)
    v = random.randint(1, 100)
    if u < v:
        edges.append((u, v))
```
- An empty list `edges` is initialized.
- A loop runs 5000 times to generate random pairs of nodes (u, v):
  - Random integers `u` and `v` are generated in the range of 1 to 100.
  - The condition `if u < v:` ensures that the first number is always smaller than the second, preventing duplicate edges like (u, v) and (v, u).
- Each valid edge (u, v) is appended to the `edges` list.

### 4. Create a DataFrame and Save to CSV
```python
edges_df = pd.DataFrame(edges, columns=['u', 'v'])
edges_df.to_csv('random_edges.csv', index=False, header=False)
```
- A Pandas DataFrame is created from the list of edges.
- The DataFrame is saved as a CSV file named `random_edges.csv`, without the index and header.

### 5. Initialize Spark Session
```python
spark = SparkSession.builder.appName("Triangle Finder").getOrCreate()
```
- A Spark session is initialized with the application name "Triangle Finder".

### 6. Load the CSV File into an RDD
```python
edges = spark.sparkContext.textFile("random_edges.csv").map(lambda line: tuple(map(int, line.split(','))))
```
- The CSV file is loaded into an RDD using `sparkContext.textFile()`.
- Each line is split by a comma and converted into a tuple of integers (u, v).

### 7. Convert Edges to a Set for Quick Lookup
```python
edge_set = set(edges.collect())  # Collect edges in a set for O(1) lookup
```
- The `collect()` method retrieves all edges from the RDD and stores them in a set named `edge_set`.
- This allows for O(1) average time complexity for checking the existence of an edge.

### 8. Group by Node to Get All Connected Nodes
```python
grouped_edges = edges.groupByKey().mapValues(list)
```
- The `groupByKey()` method groups edges by their starting node, resulting in a mapping of nodes to lists of their connected neighbors.
- `mapValues(list)` converts the grouped values into lists for easy iteration.

### 9. Reduce Phase: Find Triangular Relationships Using edge_set
```python
triangles = grouped_edges.flatMap(lambda node_neighbors: [
    (node_neighbors[0], (a, b)) for a, b in combinations(node_neighbors[1], 2)
    if (a, b) in edge_set or (b, a) in edge_set
])
```
- For each node and its list of connected nodes:
  - The `combinations` function generates all possible pairs of connected nodes.
  - The `flatMap` function creates tuples of the form (node_neighbors[0], (a, b)) for each pair (a, b).
  - The condition checks if the edge (a, b) or (b, a) exists in `edge_set`, confirming a triangle formation (u, a, b).

### 10. Collect Results
```python
result = triangles.collect()
```
- The `collect()` method gathers the results from the RDD into a list named `result`.

### 11. Print Results
```python
for triangle in result:
    print(f"Triangle found: {triangle}")
```
- This loop iterates over the `result` list and prints any triangles found.

### 12. Stop the Spark Session
```python
spark.stop()
```
- Finally, the Spark session is stopped to release resources.

## Summary
The code effectively generates a random graph of edges, checks for triangular relationships within that graph using PySpark, and outputs any triangles found. The approach takes advantage of distributed computing capabilities to handle potentially large datasets efficiently. The key improvement in this version is the removal of unnecessary duplicate edge handling due to the condition u < v.

## How to Run
1. Copy the code into a Python environment that supports PySpark.
2. Ensure that the required libraries are installed.
3. Run the code to generate the graph and find triangles.

If you have any questions or need further clarification, feel free to ask!
=======
# Triangle Finding Algorithms Using PySpark

This repository contains implementations of two triangle counting algorithms—One Three Partition (OTP) and Enhanced Two-Tiered Partitioning (ETTP)—using PySpark. The project also includes a random edge generator and utilities for visualizing and understanding the triangle counting process.

## Overview

Triangles are significant structures in graph theory, and counting them is essential in various applications, such as social network analysis and biological networks. This project implements two efficient algorithms to count triangles in large-scale graphs using Apache Spark's distributed computing capabilities.

---

## Article Link

[📄 Graph partitioning MapReduce-based algorithms for counting triangles in large-scale graphs](https://pmc.ncbi.nlm.nih.gov/articles/PMC9813377/)



## Algorithms

### 1. One Three Partition (OTP) Algorithm

The OTP algorithm partitions the graph into sub-graphs and counts triangles within those partitions. It distinguishes between three types of triangles based on their partitioning:
- **Type-1**: All three vertices of the triangle are in the same partition.
- **Type-2**: Two vertices are in the same partition, and the third vertex is in a different partition.
- **Type-3**: All three vertices are in different partitions.

#### Implementation Details

- The graph is represented using a list of edges, which is generated randomly for testing purposes.
- The edges are mapped to their corresponding partitions based on their vertex values.
- The triangles are counted by examining the neighbors of each edge in the partitions.

#### Code Overview
```python
# Code to implement the OTP algorithm
# (Code snippet as provided in previous implementations)
```

### 2. Enhanced Two-Tiered Partitioning (ETTP) Algorithm

The ETTP algorithm enhances the OTP by introducing a two-tiered partitioning mechanism to improve performance when counting triangles. This allows for a more efficient triangle detection process by reducing the number of comparisons needed.

#### Implementation Details

- Similar to OTP, the ETTP algorithm first partitions the graph into two layers.
- It counts triangles based on different types of edges—those within the same partition and those across partitions.

#### Code Overview
```python
# Code to implement the ETTP algorithm
# (Code snippet as provided in previous implementations)
```

## Comparison of Algorithms

The article compares the OTP and ETTP algorithms in terms of performance and efficiency. Here are the key findings:

- **Execution Time**: The ETTP algorithm typically demonstrates a lower execution time than the OTP algorithm for larger datasets due to its enhanced partitioning strategy. The two-tiered approach reduces the number of triangle checks needed.
  
- **Triangle Counting Accuracy**: Both algorithms yield accurate triangle counts, but the ETTP's method of handling partitions allows it to capture a more comprehensive view of triangles that might be missed by OTP in certain configurations.

- **Scalability**: ETTP is shown to scale better with larger datasets, making it a preferred choice when working with extensive graphs.

### Results Summary

- **OTP Performance**: Generally effective for smaller datasets, but may struggle with execution time and efficiency as graph size increases.
- **ETTP Performance**: Offers improved speed and scalability, making it more suitable for large-scale graph analysis.

## Running the Algorithms

1. **Dependencies**: Ensure you have Python and PySpark installed. You can install PySpark via pip:
    ```bash
    pip install pyspark
    ```

2. **Running in Google Colab**: The implementations are suitable for execution in Google Colab. Upload the required CSV files or generate them using the provided code snippets.

3. **Execution**: Each algorithm can be run in its respective Jupyter notebook. The output will display the triangles found in the graph, providing insights into the connectivity of the nodes.

## Example Input
- Random edges are generated and saved into a CSV file, which is then read by the algorithms to find triangles.

## Output
- The output for both algorithms includes the sets of vertices forming triangles, printed to the console for review.

## Conclusion

This project provides an efficient framework for counting triangles in large graphs using Spark. The implementation of both OTP and ETTP algorithms demonstrates the versatility of Spark for graph processing tasks. Users can adapt and extend these algorithms for more complex applications in network analysis.

If you have any questions or need further clarification, feel free to reach out!
>>>>>>> d2525ea (Add OTP & ETTP Algorithm Base On Article)
