# IOI23
## Closing Time
### Minimum (subtask 1)
Since the distance from X to Y is greater than 2K, we can't have a city reachable from both X and Y. So we can find a solution by picking which cities to make reachable from X/Y greedily in the following way:
 - For each city i calculate length of paths from i to X and Y and let cost[i] be the smallest of the two lengths. Note that cost[X] = cost[Y] = 0.
 - Let i be the city with smallest value of cost that hasn't been made reachable and make it reachable (which corresponds to setting c[i] appropriately, but we don't have to explicitly compute the c[]). Adjust the budget by setting it to K - cost[i].
 - Repeat until we ran out of budget.
 
To perform the second step efficiently we can use a priority queue to keep track of the costs.

### Bronze (subtasks 1,2,3 + maybe 4)
In these subtasks we are assuming the tree forms a line, which will look something like:

```
    -----X------Y-----
```
    
where the '-----' represent a sequence of nodes (possible empty).

Given some assignment of closing times, call a node a:
 - type 0 node if it isn't reachable from either X or Y.
 - type 1 node if it is reachable from exactly one of X or Y (so it counts once towards the convenience score).
 - type 2 node if it is reachable from both X and Y.

Our goal is now to assign to each node a type 0/1/2 (note that not all assignments are valid). First observe that a solution will follow one of these two formats:

```
    -----X------Y-----
    001111110001111000 -> two contiguous segments of 1s
    001111112221111100 -> two contiguous segments of 1s with a contiguous segment of 2s in between
```

where in the second format the segment of 2s can extend beyond the X/Y nodes, and it can also be empty (i.e. the solution might contain only 1s). Note that the first format corresponds exactly to the format of subtask 1, so we can use the same priority queue solution as above to find the best solution that follows this format. So we now have to focus on the second format.

There is a "brute force" solution that gets subtasks 2 and 3 that consists on doing the following. Consider all possible contiguous segments of nodes (there are N^2 of them). Given one such segment, set all nodes to type 1. If this goes over the budget ignore this configuration. Otherwise we want to find the best segment of 2s to assign, which we can do by using a 2-pointers/sliding window type of algorithm to find the largest segment that doesn't go over the budget. All together this works in O(N^3) time since the 2-pointers algorithm is a O(N) algorithm and we have to repeat it for each of the N^2 segments.

There's also a simpler algorithm that gets subtasks 1, 2, 3 and 4 (so it actually works in O(N log N) time). Again, suppose you are in the second format. In a solution of the second format all the nodes between X and Y are of type at least 1 (so they can be 1 or 2), so set them all to type 1 originally. Now run the following algorithm, which is very similar to the subtask 1 one:
 - For each node i calculate the cost of upgrading it, meaning if i is a type 1 node (so its in the path between X and Y) the cost of turning it into a type 2 node (which is exactly max(distX[i], distY[i]) - min(distX[i], distY[i])), and if it is a type 0 node the cost of turning it into a type 1 node.
 - Let i be the city with smallest value of cost. If it's a type 1, upgrade it to a type 2. If it's a type 0, upgrade it to a type 1 and calculate the cost of upgrading it again (but now from a type 1 to a type 2). Adjust the budget by setting it to K - cost[i].
 - Repeat until we ran out of budget.

Again we can use a priority queue to keep track of the costs.

It's not obvious that this algorithm always works since we can only upgrade a node if the nodes next to it have been upgraded accordingly. It's not too hard to show that when the graph is a line, this algorithm always works (it's not obvious either), but it doesn't quite work on general trees (the full solution is actually based on this idea, but it uses some extra observations to modify the algorithm).

### Best (subtasks 1,2,3,4,5,6,7,8)

When thinking about general graphs our observations about segments don't quite apply, but we can actually generalize them in a simple way. Given a valid assignment of closing times, the set of nodes that are of type 2 form a connected component, and the set of vertices that are either of type 1 or 2 also form a connected component. When the graph is a line, a connected component is a segment, hence this generalizes our observations from the previous solution. Now, here are two observations:
 - Suppose we order all nodes by the cost to upgrade them from a type 1 to a type 2, increasingly. Then if we pick nodes one by one using this order, at every step of the process the set of picked nodes form a connected component.
 - Suppose we order all nodes by the cost to upgrade them from a type 0 to a type 1, increasingly. Pick all nodes on the path from X to Y. Then if we pick nodes one by one using this order (excluding the nodes on the path from X to Y, which we pre picked), at every step of the process the set of picked nodes form a connected component.
 
These two observations are useful to show that the algorithm from the previous section works. Note that they aren't quite enough because these observations don't help us decide when to pick to upgrade a node from type 1 to type 2 or a node from type 0 to type 1, they just say that as long as we upgrade nodes in this order, then we are surely going to end up with valid assignments. In the previous algorithm we used a greedy strategy to decide when to upgrade a node from either type 0 or type 1, and this works for lines, but doesn't necessarily work for general trees.

So, instead of using a greedy strategy to decide when to pick the next node to upgrade, let's use DP instead. Suppose that all nodes on the path from X to Y are of type 1, and everyone else is type 0. Let DP[i][j] be the smallest cost of a solution that upgraded i nodes from type 0 to type 1, and j nodes from type 1 to type 2. Then DP[i][j] is either:
 - let v be the ith node in the sequence of nodes sorted by cost of upgrading from type 0 to type 1 (excluding the nodes in the path from X to Y). Then consider DP[i - 1][j] + cost1[v], which corresponds to updating a node from a type 0 to a type 1 and cost1[v] is the cost to upgrade v to type 1.
 - consider the set S of nodes that have been upgraded to type 1, which is exactly all the nodes in the path from X to Y plus the first i nodes in order of cost to upgrade from type 0 to type 1. Let v be the jth node in the sequence of nodes in S sorted by cost of upgrading from type 1 to type 2. Then consider DP[i][j - 1] + cost2[v], which corresponds to updating a node from a type 1 to a type 2 and cost2[v] is the cost to upgrade v to type 2.
 
If we implement this DP carefully, each transition can be computed in constant time, and since there are N^2 states, it runs in O(N^2) time.

## Longest Trip
### Minimum (subtasks 1,2)
When D=3 the graph is a complete graph, so any permutation of N numbers is a valid answer.

When D=2 the graph isn't complete, but it is pretty dense. A natural idea is to try to build a path one vertex at a time. Start by finding two connected vertices. We can do this by picking 3 arbitrary vertices and we know two of them are connected, so with less than 3 queries we will find what we need. So we now have a path of length two. Call the two vertices in this path a and b. Pick an arbitrary new vertex v and check if it is connected to either a or b. Since D=2, we know that v has to be connected to either a or b. If v is connected to a, then extend the path by connecting v to a. If v is connected to b, then extend the path by connecting v to b. We can repeat this procedure by looking at the two endpoints of the current path, picking a new vertex (one not currently on the path) and adding it to one of the ends of the current path. This takes 2N queries to run, so it's well under the limit.

### Bronze (subtasks 1,2,3)
To do subtask 3 we can try to extend the ideas from subtask 2. Again, it is natural to try to build a path one vertex at a time, but when D=1 we can't be certain that a new vertex will be connected to either endpoint of the current path (e.g. the endpoints might be connected, forming a cycle, but not connected to anything else). Since on subtask 3 we only need to build a path of length l/2, one idea could be to try to build two paths in parallel, hope that we can use up all of the vertices, and then at the end pick the largest of the two paths we built.

So suppose we start with two vertices, v1 and v2 (i.e. two paths of length 1). Now pick an arbitrary new vertex v. If v is connected to v1, then extend the first path by connecting v to v1. If v is connected to v2, then extend the second path by connecting v to v2. If v isn't connected to either v1 or v2, then we know (from the D=1 condition) that v1 and v2 are connected, so we can connect the two paths to form a long path by connecting v1 to v2, and start a new second path of length 1 containing only v. So we can repeat this until we use up all vertices, and in the end we will have two disjoint paths whose length sums to N, which means that at least one of them will have length ceil(N/2). Choose the longest of the two and output it. (note that l is at most N, so this is guaranteed to work).

### Best (subtasks 1,2,3 and part of 4)
To solve any part of subtask 4 we can't get away with a path of half the best length. One idea would be to try to adapt the ideas from subtask 3 to try to build a path of length N, but is this even possible? Before answering that question, it is worth it to think about whether the graph is even guaranteed to be connected. Suppose the graph isn't connected. It can't have 3 connected components, since the D=1 condition forces an edge between any 3 vertices, so if the graph had three connected components then by picking one vertex per component we'd have to have an edge between at least two of them, a contradiction. What about 2? Suppose there are 2 connected components, then take any two vertices a and b in the first connected component and a third vertex c in the second connected component. Since there is no edge between a and c and b and c, the D=1 condition forces an edge between a and b, which means that each connected component must form a complete graph. We conclude that either the graph is connected, or it contains two connected components forming complete graphs. Let's focus on the case of a connected graph, and we will see how the two disconnected complete graphs will easily follow (and in fact, if you think about it, the solution from subtask 3 already covers this case).

Suppose we run the algorithm from subtask 3. As we saw, once it terminates we are left with two disjoint paths whose lenghts sum to N. If the graph is a disconnected graph, then each of the paths will be in a different connected component, so the longest of the two will be a path that uses all of the vertices in the largest connected component. We can determine if we are in this situation using a single query, taking A to be all the vertices in the first path, and B all the vertices in the second path. If the graph is not disconnected, then we might want to try to combine the two paths to form a new path that uses all the vertices. We can use two queries to check if there is an edge from one of the endpoints of the first path to another endpoint of the second path, if so we can combine them and form a path of length N. If the two paths are not connected, we can be sure that the two paths actually form cycles (i.e. the endpoints of each path are connected between themselves), which follows from the D=1 condition. This means that if we find an edge between the two cycles, we can cut up the cycles in such a way to form a single path containing all of the vertices. The graph is connected, so we know that such an edge exists. To efficiently find such an edge, we can use binary search twice in the following way. Let C = (c1, c2, ..., cl) be the first cycle and C' the second cycle. Check if A = {c1, c2, ..., c_{l/2}} is connected to B = C', if so, then check A = {c1, c2, ..., c_{l/4}}, and so on. At every step we are reducing the interval of potential vertices by a factor of 2. Do the same thing with the roles of C and C' swapped, and we'll find two vertices with an edge between them.

How efficient is this solution? Well running the initial subtask 3 algorithm takes 2N queries, plus 3+2logN to run the subsequent checks and binary searches. If you do the math, this is enough for the case  400 < q <= 550. To get to 100 points we would need to be more clever about how to run the initial algorithm in order to optimize the number of queries.

## Soccer Stadium
### Minimum (subtask 1)
If there is at most one tree it's not hard to see that a solution has to be of one of the following formats:

```
######   ######   ###...   ....##
######   ######   ###...   ....##
...T##   ###T..   ###T..   ...T##
....##   ###...   ######   ######
....##   ###...   ######   ######
```

(# means pick that cell, . means empty cell not picked, and T is the tree cell)

## Bronze (subtask 1,2)
The number of grids with N<=3 is small, but not small enough that we can solve it manually (meaning, solve each case with pen and paper, and then hardcode the solutions to a program). There are at most 2^(3*3)=512 possible grids, but we actually only care about a much smaller amount since two grids are equivalent if they are rotations or reflections of each other. Also, the empty grid and the grids with a single tree are covered by the solution from subtask 1. This still leaves us with about ~100 possible grids, which is too much to do by hand. However, note that we can apply the same logic to the possible shapes of regular stadiums. If you try to write down all the possible shapes of regular stadiums (up to rotations or reflections) you get that the only possibilities are the following 9:

```
###  ###  ###  ###  ###  ##T  ##T  ##T  #TT
###  ###  ##T  #TT  TTT  ##T  #TT  TTT  TTT
#TT  TTT  TTT  TTT  TTT  TTT  TTT  TTT  TTT
```

(# means a cell part of the regular stadium and T means a cell that isn't part of the regular stadium)

So we can use the following solution: generate all possible rotations and reflections of the above 9 shapes. Now, given a grid, try to place one of the above shapes somewhere in the grid, so that the # are all in empty cells. Return the size of the largest pattern you were able to fit.

This solution is simple, but the implementation can be quite tricky. For these types of problems plan your implementation carefully before starting to code.

Also, note that your solution also needs to work for N=2 and N=1. Thankfully, for such small N you can actually solve it manually, so you can hard code the solution.

One last note, if you were completely stuck on all other problems while solving this problem and you couldn't think of the rotating/reflecting shapes solution, then manually solving the problem for each of the ~100 grids would not take that much time. If you write a small script to generate all possible unique grids (you can do this with a few lines of Python, or in C++ if you prefer), you could go one by one and label all of them in about 10 minutes. Then, you'd want to hardcode all of the ~100 grids and respective solutions in your code. It's really important that if you were to do this, don't manually copy the grids into C++ code! Use the Python or C++ script you wrote to generate the matrices to generate C++ code. This approach is a bit risky though, because debugging this wouldn't be fun. If you accidentally get one of the grids wrong you'll get a WA and you might have to go over each of the ~100 grids to find the mistake.

## Best (subtask 1,2, 25% of 3,4,5,6 + maybe 3)
When N is greater than 3 it is hopeless to try to compute all possible regular stadiums manually, so we better first understand what a regular stadium looks like. If we are able to do this efficiently, we can get 25% of the points in the remaining subtasks.

Suppose we are given a grid and we want to determine if the empty cells form a regular stadium. First of all, it is easy to note that any column or row has to contain only a single contiguous segment of 0s, so it can't look like 010. Is this a sufficient condition? Not quite, consider the following 3 by 3 example:

```
101
001
100
```

The issue is that we can't have these "s" shapes. One way to quantify this property is by consider each segment as an interval (assuming the grid is made up of segments, if not it isn't regular anyway), so we could describe the above example as (1, 1), (0, 1), (1, 2), meaning the first row contains empty cells going from column 1 to column 1, the second row has empty cells going from column 0 to column 1 and the third row from column 1 to column 2. We can do a similar type of decomposition column wise, which would be (1, 1), (0, 2), (2, 2) for this example. Now the condition we need is that any two intervals have to have a trivial intersection (meaning one has to be a subinterval of the other). If all intervals satisfy this property, then the grid is a regular stadium. Also, we only need to check for row-wise intervals, since if they satisfy this condition the column ones also do (but only if the columns form intervals).

This gives us a O(N^2) solution to detecting if a grid is regular: first determine if the rows and columns have a single segment, and compute the intervals of each row. Then for every pair of intervals corresponding to rows, check if one is a subinterval of the other.

To solve any more subtasks, one needs to use these observations about what a regular stadium looks like to come up with an algorithm. To solve subtask 3 it suffices to write a brute force recursive search algorithm that tries to add intervals to partially constructed regular stadiums. It helps to use the following observation that follows from the description of a regular stadium. Consider some regular stadium and find the longest interval (row-wise), breaking ties arbitrarily. Now if we move up or downwards from this longest interval each interval has to be a non-strict subinterval of the previous one, which means they must stay the same or be shorter than the previous interval. So a possible algorithm is the following, consider all maximal intervals (so you can't grow them by adding a cell on its left or right) and assume each of them is the longest interval of the biggest stadium. Now try to recursively add intervals above and below this longest interval. Note that by following this order, any new interval you add has to be a subinterval of all of the intervals currently in the partial regular stadium, so it is easy to decide which new intervals we can consider. This leads to a pretty simple exponential time solution that solves subtask 3. To solve subtask 4 we have to turn this recursive solution into a dynamic programming one, by recording the rows we've used and the range of the current intersection of all intervals. Subsequent subtasks are solved by optimizing this dynamic programming using more observations.
