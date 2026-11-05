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

### Why a 2D grid + Manhattan distance

A 2D grid (numpy array) where `0` = open path and `1` = wall is a natural representation for a maze:

- Mazes are inherently grid-like spaces with individual cells and obstacles that the navigator cannot go through.
- Positions as `(row, col)` tuples match standard array indexing and make movement logic (up/down/left/right) straightforward.
- Manhattan distance (`|x1−x2| + |y1−y2|`) is a fast, admissible heuristic for grid pathfinding and works well as a fitness signal or guidance term inside a genetic algorithm. This is something we learned in DCS229 when studying Dijkstra’s algorithm.

This keeps the environment simple and focused so the evolutionary algorithm can concentrate on evolving path-finding behavior rather than spending resources on a more complex continuous or graph-based representation. Evolutionary algorithms already use significant resources depending on scale, so keeping the environment lightweight is intentional.

### How encapsulating the maze benefits the architecture

Putting the grid, start, goal, and spatial rules inside the `Maze` class is a clear application of encapsulation and abstraction:

- **Bundles state + behavior**: The data (grid, dimensions, start/goal) and the rules that operate on it (`is_valid_position`, `get_distance`) live together. Users interact with the object rather than manipulating the raw grid directly.
- **Hides implementation details**: The rest of the system (Individual, Population, Selection, Crossover, EvolutionEngine, etc.) only needs to ask the Maze “is this position valid?” or “what is the distance to the goal?” It does not care whether the grid is a Python list, a NumPy array, or something else.
- **Clean separation of concerns**: The evolutionary algorithm owns the search mechanisms (genotypes, variation, selection). The Maze owns the problem domain (what a valid path looks like, how fitness relates to the environment). This mirrors the genotype/phenotype concept in DeJong’s framework.
- **Easier testing and reuse**: You can unit-test the Maze independently, swap in different mazes (or even different environment classes) without changing the GA code, and keep the evolutionary operators polymorphic and reusable.
- **Supports modularity and future extension**: If you later want a different heuristic, a continuous maze, or a multi-agent environment, you can change or subclass `Maze` while the core EvolutionEngine stays the same.

## DeJong Java Code
### DeJong-Style Individual (Java)

```java
public class Individual<G> {
    private final G representation;
    private double fitness;

    public Individual(G representation) {
        this.representation = representation;
        this.fitness = Double.NaN;
    }

    public G getRepresentation() {
        return representation;
    }

    public double getFitness() {
        return fitness;
    }

    public void setFitness(double fitness) {
        this.fitness = fitness;
    }
}

// Strategy Interface for Selection
public interface SelectionOperator<G> {
    Individual<G> select(Population<G> population);
}

// Strategy Interface for Variation (Crossover)
public interface CrossoverOperator<G> {
    List<Individual<G>> cross(Individual<G> p1, Individual<G> p2);
}

public class EvolutionEngine<G> {
    private SelectionOperator<G> selector;
    private CrossoverOperator<G> crossover;
    private FitnessFunction<G> fitnessFunc;

    public Population<G> step(Population<G> current) {
        Population<G> nextGen = new Population<>();

        while (!nextGen.isFull()) {
            Individual<G> p1 = selector.select(current);
            Individual<G> p2 = selector.select(current);

            var offspring = crossover.cross(p1, p2);
            offspring.forEach(child -> fitnessFunc.evaluate(child));

            nextGen.addAll(offspring);
        }

        return nextGen;
    }
}
```

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
