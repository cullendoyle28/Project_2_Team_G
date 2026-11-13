# Planning and Analysis

## Problem Formulation
The objective of the Microservice Build Pipeline problem is to determine the minimum amount of time to build an entire system consisting of n microservices. We are given a collection of microservices, where each service has an associated build time, as well as a set of directed dependency edges indicating which services must be completed before others can begin building. We represent this system as a directed graph G = (V,E), where each vertex represents a microservice and each directed edge (u,v) indicates that service v depends on service u. Each vertex v also has an associated nonnegative build duration t(v).

Immediately, we want to determine whether or not the dependency graph contains a directed cycle. If a cycle exists, the build pipeline is invalid because at least one service would depend, directly or indirectly, on itself. If the graph is cycleless, we must determine the critical path, which is the longest-duration path through the dependency graph. Since independent microservices can be built simultaneously, the critical path determines the minimum possible time required to complete the entire build.

As an additional optimization objective, we want to identify the single dependency edge whose removal would produce the greatest reduction in the total build time. We assume that an edge can only be removed if the corresponding dependency is genuinely unnecessary, since removing a required dependency could result in an invalid build.


## Baseline Solution

## Algorithmic Strategy

## Complexity Analysis
