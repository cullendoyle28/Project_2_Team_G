# Planning and Analysis

## Problem Formulation

The objective of the Microservice Build Pipeline problem is to determine the minimum amount of time to build an entire system consisting of n microservices. We are given a collection of microservices, where each service has an associated build time, as well as a set of directed dependency edges indicating which services must be completed before others can begin building. We represent this system as a directed graph G = (V,E), where each vertex represents a microservice and each directed edge (u,v) indicates that service v depends on service u. Each vertex v also has an associated nonnegative build duration t(v).

Immediately, we want to determine whether or not the dependency graph contains a directed cycle. If a cycle exists, then the build pipeline is invalid because at least one service would depend, directly or indirectly, on itself. If the graph has no cycles, then we must determine the critical path, which is the longest-duration path through the dependency graph. Since independent microservices can be built simultaneously, the critical path determines the minimum possible time required to complete the entire build.

As an additional optimization objective, we want to identify the single dependency edge whose removal would produce the greatest reduction in the total build time. We assume that an edge can only be removed if the corresponding dependency is genuinely unnecessary, since removing a required dependency could result in an invalid build.

## Baseline Solution

We implemented a brute-force approach that explores possible build sequences and calculates the total completion time for each valid sequence. It first checks whether the dependency graph contains a directed cycle. If a cycle is detected, then the algorithm reports that the build pipeline is invalid.

If the graph contains no cycles, then the algorithm generates all possible topological orderings of the microservices. A topological ordering is a sequence in which every service appears after all of its dependencies. For each valid ordering, we calculate the earliest completion time of every service by considering the completion times of its dependencies. The maximum completion time represents the total time required to complete that build schedule.

After examining every valid ordering, the algorithm returns the minimum total build time and the corresponding critical path.

For the bonus problem, we test each dependency edge individually. We temporarily remove an edge, recalculate the minimum build time, and compare the result with the original build time. We restore the edge before testing the next one. The algorithm records the edge that produces the greatest reduction in build time.

```text
Algorithm buildPipelineBaseline(G, durations):
    input:
        G = Directed dependency graph (V, E)
        durations = Build duration for each microservice

    output:
        Minimum build time, critical path,
        and best dependency edge to remove

    if G contains a directed cycle:
        return "Invalid build pipeline"

    best_time = infinity
    best_path = []

    Generate every valid topological ordering of G

    for each ordering in all orderings:

        Calculate the earliest completion time
        of every service while respecting dependencies

        current_time = Maximum completion time

        if current_time < best_time:
            best_time = current_time
            best_path = corresponding critical path

    original_time = best_time

    best_edge = NONE
    greatest_improvement = 0

    for each edge (u, v) in E:

        Temporarily remove edge (u, v)

        if the modified graph is valid:
            Recalculate the minimum build time

            improvement = original_time - new_time

            if improvement > greatest_improvement:
                greatest_improvement = improvement
                best_edge = (u, v)

        Restore edge (u, v)

    return original_time, best_path, best_edge
```

## Algorithmic Strategy

## Complexity Analysis
