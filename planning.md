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

We use topological sorting to process services in dependency order and dynamic programming to reuse the finish times already calculated. This lets us process each service once instead of checking every possible ordering. We assume that independent services can build at the same time, with no limit on available workers.

First, we count each service's dependencies. This count is called its indegree. Services with no dependencies go into a queue. For each service v, we store its earliest finish time finish(v) and a previous service, called its predecessor, to help trace the critical path.

We start with finish(v) = t(v). For each edge (u,v), we compare finish(v) with finish(u) + t(v). If the second value is larger, we save it and record u as the predecessor. We then lower v's indegree by one and add v to the queue when the count reaches zero.

The rule is finish(v) = t(v) + max finish(u), where u is a direct dependency of v. If v has no dependencies, we use zero for the maximum. This works because a service can start as soon as all of its dependencies finish.

If we process fewer than n services, the graph has a directed cycle and the pipeline is invalid. Otherwise, the largest finish time is the minimum total build time. To find a critical path, we start at a service with this finish time, follow its predecessors, and reverse the path. If several paths take the same longest time, we can return any one. Separate groups of services also work, and an empty graph returns zero time and an empty path.

For the bonus problem, we ignore one edge at a time, calculate the build time again, and save the edge that reduces it the most. Removing an edge from a graph with no cycles cannot create a cycle. If no edge saves time, we return NONE. Before removing the chosen dependency from the actual pipeline, we must check that it is not needed.

```text
Algorithm analyzePipeline(G, durations, ignored_edge = NONE):
    Count dependencies using every edge except ignored_edge
    queue = All vertices with indegree zero

    for each vertex v in V:
        finish[v] = durations[v]
        predecessor[v] = NONE

    processed = 0

    while queue is not empty:
        u = Remove a vertex from queue
        processed = processed + 1

        for each outgoing edge (u, v):
            if (u, v) is ignored_edge:
                continue

            if finish[u] + durations[v] > finish[v]:
                finish[v] = finish[u] + durations[v]
                predecessor[v] = u

            indegree[v] = indegree[v] - 1

            if indegree[v] == 0:
                Add v to queue

    if processed < number of vertices:
        return "Invalid build pipeline"

    if V is empty:
        return 0, []

    last_service = A vertex with maximum finish value
    total_time = finish[last_service]
    critical_path = []

    while last_service is not NONE:
        Add last_service to the end of critical_path
        last_service = predecessor[last_service]

    Reverse critical_path

    return total_time, critical_path

Algorithm buildPipelineEfficient(G, durations):
    result = analyzePipeline(G, durations)

    if result is "Invalid build pipeline":
        return "Invalid build pipeline"

    original_time, best_path = result
    best_edge = NONE
    greatest_improvement = 0

    for each edge (u, v) in E:
        new_time, new_path = analyzePipeline(G, durations, (u, v))
        improvement = original_time - new_time

        if improvement > greatest_improvement:
            greatest_improvement = improvement
            best_edge = (u, v)

    return original_time, best_path, best_edge
```

## Complexity Analysis

Let n be the number of services and m be the number of dependency edges. We store each service's outgoing edges in a list and assume each calculation with build times takes constant time.

For the baseline solution, checking for cycles takes O(n + m) time. There can be n! valid orderings, and calculating finish times for each takes O(n + m) time. This gives O(n! * (n + m)) time in the worst case, assuming each ordering also takes at most O(n + m) time to generate. All valid orderings give the same earliest finish times, so checking every ordering repeats work.

The baseline bonus repeats this calculation for each edge, giving O((m + 1) * n! * (n + m)) time. If we store all K valid orderings, total space is O(K * n + n + m), where K can be as large as n!. If we generate and check one ordering at a time, undoing choices to try others, total space can be O(n + m).

For the graph-based solution, counting dependencies takes O(n + m) time. Each service enters and leaves the queue once, and each edge is checked once during processing. Finding the largest finish time and tracing the path take O(n) time. Together, checking for cycles, finding the minimum build time, and finding a critical path take O(n + m) time.

Storing the graph takes O(n + m) space. The dependency counts, finish times, predecessors, queue, and path use O(n) extra space, giving O(n + m) total space.

For the graph-based bonus, we run the calculation once for the original graph and once for each edge, giving O((m + 1) * (n + m)) time. We reuse the working space for each test, so total space stays O(n + m).
