# Object-Oriented Approaches to Evolutionary Algorithms
### A Genetic Algorithm Maze Solver using DeJong’s Unified Framework

A modular, object-oriented implementation of a Genetic Algorithm that finds paths through grid-based mazes. This project explores how classic OOP principles (encapsulation, abstraction, and the Strategy pattern) can be applied to evolutionary algorithms, drawing directly from Kenneth DeJong’s unified framework for evolutionary computation.

## Motivation
I chose OOP for this project because of my time in DCS229: Data Structures and Algorithms. Within the first few weeks of DCS340, I found that most of the implementations of evolutionary algorithms could be improved if we just modularized the algorithms and different structures into the models we were building. Throughout my research, it became more and more apparent that object-oriented programming was necessary in larger evolutionary algorithms from a performance and resource management perspective.

## Approach
This project follows DeJong’s unified evolutionary architecture by cleanly separating:

- **Genotype** — the representation (here, a fixed-length string of movement genes)
- **Phenotype / Fitness** — evaluation against the maze environment
- **Population dynamics** — selection, crossover, and mutation

These concepts are mapped to distinct classes so that algorithmic components can be swapped without rewriting the core engine.

### Key Classes
- **`Maze`** — Encapsulates the grid, start/goal positions, validity checks, and distance heuristic.
- **`Individual`** — Holds the genome and its fitness evaluation logic.
- **`Population`** — Manages the collection of individuals, selection (tournament), elitism, crossover, and mutation.

## Design Decisions
[Briefly restate or expand on the explanation you wrote in the Colab notebook. Focus on:]
- Why a 2D grid + Manhattan distance was chosen
- How encapsulating the maze benefits modularity and testing
- How this mirrors the genotype/phenotype split

## Running the Project
### Colab Notebook
The main interactive notebook is available here:  
(https://colab.research.google.com/drive/1dIDazAWaVwX_Q8tqFYaHjDDqrmPf98kW)

## Running the Project
The main notebook is included in this repository as `maze_solver.ipynb`.

You can also view/run the interactive version in Colab:  
(https://colab.research.google.com/drive/1dIDazAWaVwX_Q8tqFYaHjDDqrmPf98kW)

To run locally:
```bash
pip install -r requirements.txt
jupyter notebook maze_solver.ipynb
