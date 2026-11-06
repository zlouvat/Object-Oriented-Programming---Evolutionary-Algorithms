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

Putting the grid, start, goal, and spatial rules inside the `Maze` class is clear usage of encapsulation and abstraction:

- **Bundles state + behavior**: The data (grid, dimensions, start/goal) and the rules that operate on it (`is_valid_position`, `get_distance`) live together. Users interact with the object rather than manipulating the raw grid directly.
- **Hides implementation details**: The rest of the system (Individual, Population, Selection, Crossover, EvolutionEngine, etc.) only needs to ask the Maze “is this position valid?” or “what is the distance to the goal?” It does not care whether the grid is a Python list, a NumPy array, or something else.
- **Clean separation of concerns**: The evolutionary algorithm owns the search mechanisms (genotypes, variation, selection). The Maze owns the problem domain (what a valid path looks like, how fitness relates to the environment). This mirrors the genotype/phenotype concept in DeJong’s framework.
- **Easier testing and reuse**: You can unit-test the Maze independently, swap in different mazes (or even different environment classes) without changing the GA code, and keep the evolutionary operators polymorphic and reusable.
- **Supports modularity and future extension**: If you later want a different heuristic, a continuous maze, or a multi-agent environment, you can change or subclass `Maze` while the core EvolutionEngine stays the same.

## DeJong Java Code
### DeJong’s Unified Framework in Object-Oriented Form

The following design maps Kenneth DeJong’s evolutionary architecture into three core pieces:

- **Individual**  
  Encapsulates a genotype (`representation`) and its fitness score. The representation is generic so the same class can hold bit strings, real vectors, path genes, or any other encoding without changing the rest of the engine.

- **Selection & Crossover Operators (Strategy Pattern)**  
  Selection and crossover are defined as interfaces. Concrete implementations (tournament selection, single-point crossover, etc.) can be swapped at runtime. This keeps the evolutionary loop independent of any particular operator.

- **EvolutionEngine**  
  Owns the generational loop. On each step it:
  1. Selects two parents from the current population
  2. Produces offspring via the crossover operator
  3. Evaluates the offspring with a fitness function
  4. Adds them to the next generation until the population is full

This structure directly reflects DeJong’s genotype/phenotype split and the idea that variation operators and selection methods are interchangeable components of a unified evolutionary algorithm. Because the engine only depends on the operator interfaces, new selection or crossover strategies can be added without modifying the core loop.

While building the slides I used Mark Meysenburg’s Doane Evolutionary Algorithm (DEA) Java library as a reference. It implements many of the same ideas (different chromosomal representations, crossovers, mutations, and elitist selection) and closely follows DeJong’s framework, which helped me connect the chapter concepts to concrete class design even though the code was in Java.

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

## Reflection & Connections
Working on the maze solver notebook made me connect DCS229 and DeJong’s framework concretely. The Manhattan-distance measurement we studied with Dijkstra’s became our fitness signal, while the modular class design let me treat selection and crossover as interchangeable parts, in the same way we discussed when looking at reusable components in class. The most useful part of the project was seeing how little of the evolutionary loop has to change when operators or the environment change. Swapping selection methods or mutation strategies lets the same algorithm take on a different mechanism without rewriting the core engine. The mazes used here are relatively simple; a natural next step would be more complex mazes with different wall types that require finer solution strategies. This directly mirrors the genotype/phenotype separation DeJong describes and the modularity we were looking for in the previous assignments.

## Running the Project
### Colab Notebook
The main interactive notebook is available here:  
[Open in Colab](https://colab.research.google.com/drive/1dIDazAWaVwX_Q8tqFYaHjDDqrmPf98kW)
1. Maze environment — Defines the grid, validity checks, and Manhattan-distance heuristic.
2. Individual — Represents a candidate path as a fixed-length string of moves and evaluates fitness against the maze.
3. Population — Handles tournament selection (size 5), elitism, crossover, and mutation.
4. Running the algorithm — Demonstrates the solver on a simple 6×6 maze and a larger 10×10 maze.
   
The notebooks were initially generated with Gemini, with documentation added manually after the fact. The primary packages used are NumPy (for the grid representation) and Python’s built-in random module.

To run locally:
```bash
pip install -r requirements.txt
jupyter notebook Maze_Solver_Object_Oriented_Programming_3.ipynb
```

## References
DeJong, K. A. Evolutionary Computation: A Unified Approach Chapter 1. MIT Press, 2006.
Manhattan Distance explanation: [GeeksforGeeks](https://www.geeksforgeeks.org/dsa/manhattan-distance/)
Mark Meysenburg’s Doane Evolutionary Algorithm (DEA) Java library [(used as a structural reference for the slides)](https://github.com/mmeysenburg/DEA)
NumPy documentation: https://numpy.org/doc/
AI assistance: Gemini was used to generate initial versions of the color-picker, maze-solver, 8 Queens, and Game of Life notebooks; the maze solver was then extensively revised by me alone, along with adding manual documentation to all 4 codebooks, with a bit more done with the Maze solver, including my reflection.
