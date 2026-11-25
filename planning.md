# Planning and Analysis

## Problem Formulation

The objective of the Microservice Build Pipeline problem is to determine the minimum amount of time to build an entire system consisting of n microservices. We are given a collection of microservices, where each service has an associated build time, as well as a set of directed dependency edges indicating which services must be completed before others can begin building. We represent this system as a directed graph D = (V,E), where each vertex represents a microservice and each directed edge (u,v) indicates that service v depends on service u. Each vertex v also has an associated nonnegative build duration t(v).

Immediately, we want to determine whether or not the dependency graph contains a directed cycle. If a cycle exists, then the build pipeline is invalid because at least one service would depend, directly or indirectly, on itself. If the graph has no cycles, then we must determine the critical path, which is the longest-duration path through the dependency graph. Since independent microservices can be built simultaneously, the critical path determines the minimum possible time required to complete the entire build.

As an additional optimization objective, we want to identify the single dependency edge whose removal would produce the greatest reduction in the total build time. We assume that an edge can only be removed if the corresponding dependency is genuinely unnecessary, since removing a required dependency could result in an invalid build.

## Baseline Solution

We implemented a brute-force approach that explores every possible directed path in the dependency graph using DFS (Depth-First Search). It first checks whether the dependency graph contains a directed cycle. If a cycle is detected, then the algorithm reports that the build pipeline is invalid.

If the graph contains no cycles, the algorithm identifies all starting vertices, which are vertices with an indegree of zero. It begins a DFS traversal from each of these vertices. During each traversal, we keep track of the current path and the sum of the build durations of all services along that path. Whenever a path has a greater total duration than the longest path found so far, we update the maximum build time and save that path.

By starting at every vertex with no incoming edges, we ensure that every complete dependency chain is considered. After examining all possible paths from these starting vertices, the algorithm returns the longest-duration path and its total build time, which represent the critical path and minimum possible build time.

For the bonus problem, we test each dependency edge individually. We temporarily ignore one edge, recalculate the longest path, and compare the resulting build time with the original build time. The algorithm records the edge that produces the greatest reduction in build time.


```text
Algorithm analyzePipelineBaseline(D, durations, ignored_edge = NONE):

    if V is empty:
        return 0, []

    if D contains a directed cycle:
        return "Invalid build pipeline"

    best_time = negative infinity
    best_path = []
    current_path = []

    for each vertex v in V:
        indegree[v] = 0
    
    for each edge (u, v) in E:
        if (u, v) != ignored_edge:
            indegree[v] = indegree[v] + 1

    Procedure findLongestPath(v, path_time):

        Add v to current_path

        if path_time > best_time:
            best_time = path_time
            best_path = copy(current_path)

        for each edge (v, next) in E:
            if (v, next) != ignored_edge:
                findLongestPath(next, path_time + durations[next])

        Remove the last vertex from current_path

    for each vertex start in V:
        if indegree[start] == 0:
            findLongestPath(start, durations[start])
    
    return best_time, best_path


Algorithm buildPipelineBaseline(D, durations):

    result = analyzePipelineBaseline(D, durations)

    if result is "Invalid build pipeline":
        return "Invalid build pipeline"

    original_time, original_path = result

    best_edge = NONE
    greatest_improvement = 0

    for each edge (u, v) in E:
        new_time, new_path = analyzePipelineBaseline(D, durations, (u, v))

        improvement = original_time - new_time

        if improvement > greatest_improvement:
            greatest_improvement = improvement
            best_edge = (u, v)

    return original_time, original_path, best_edge
```

## Algorithmic Strategy

We use topological sorting to process services in dependency order and dynamic programming to reuse the finish times already calculated. This lets us process each service once instead of checking every possible ordering. We assume that independent services can build at the same time, with no limit on available workers.

First, we count each service's dependencies. This count is called its indegree. Services with no dependencies go into a queue. For each service v, we store its earliest finish time finish(v) and a previous service, called its predecessor, to help trace the critical path.

We start with finish(v) = t(v). For each edge (u,v), we compare finish(v) with finish(u) + t(v). If the second value is larger, we save it and record u as the predecessor. We then lower v's indegree by one and add v to the queue when the count reaches zero.

The rule is finish(v) = t(v) + max finish(u), where u is a direct dependency of v. If v has no dependencies, we use zero for the maximum. This works because a service can start as soon as all of its dependencies finish.

If we process fewer than n services, the graph has a directed cycle and the pipeline is invalid. Otherwise, the largest finish time is the minimum total build time. To find a critical path, we start at a service with this finish time, follow its predecessors, and reverse the path. If several paths take the same longest time, we can return any one. Separate groups of services also work, and an empty graph returns zero time and an empty path.

For the bonus problem, we ignore one edge at a time, calculate the build time again, and save the edge that reduces it the most. Removing an edge from a graph with no cycles cannot create a cycle. If no edge saves time, we return NONE. Before removing the chosen dependency from the actual pipeline, we must check that it is not needed.

```text
Algorithm analyzePipeline(D, durations, ignored_edge = NONE):
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

Algorithm buildPipelineEfficient(D, durations):
    result = analyzePipeline(D, durations)

    if result is "Invalid build pipeline":
        return "Invalid build pipeline"

    original_time, best_path = result
    best_edge = NONE
    greatest_improvement = 0

    for each edge (u, v) in E:
        new_time, new_path = analyzePipeline(D, durations, (u, v))
        improvement = original_time - new_time

        if improvement > greatest_improvement:
            greatest_improvement = improvement
            best_edge = (u, v)

    return original_time, best_path, best_edge
```

## Complexity Analysis

Let n be the number of services and m be the number of dependency edges. We store each service's outgoing edges in a list and assume each calculation with build times takes constant time.

For the baseline solution, we use DFS (Depth-First Search) to explore every possible directed path in the graph. In the worst case, a DAG (directed acyclic graph) can contain exponentially many possible paths, growing up to O(2^n). Since maintaining and copying each path can require up to O(n) time, the worst-case time complexity is O(n2^n).

For the baseline bonus, the algorithm repeats this process for every edge in the graph. Since there are m edges, the worst-case time complexity becomes O((m+1)n2^n).

Also, the space complexity is O(n+m), which accounts for the graph, recursion stack, current path, and best path. 

For the graph-based solution, counting dependencies takes O(n + m) time. Each service enters and leaves the queue once, and each edge is checked once during processing. Finding the largest finish time and tracing the path take O(n) time. Together, checking for cycles, finding the minimum build time, and finding a critical path take O(n + m) time.

Storing the graph takes O(n + m) space. The dependency counts, finish times, predecessors, queue, and path use O(n) extra space, giving O(n + m) total space.

For the graph-based bonus, we run the calculation once for the original graph and once for each edge, giving O((m + 1) * (n + m)) time. We reuse the working space for each test, so total space stays O(n + m).
