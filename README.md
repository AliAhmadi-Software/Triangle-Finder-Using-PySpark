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
