# Branching and looping in Nodezator

I spent the last week (from September 25th to 30th) researching the usage of branching and looping in other apps. I wanted to make sure I didn't miss anything important when implementing such features in Nodezator. In this text I present the conclusions I achieved and the approach I intend to follow to implement branching in Nodezator. I also explain why looping can already be performed in Nodezator and thus doesn't need to be implemented.

First of all, the research wasn't anything deep nor time-consuming. That would require more time and experimentation. In an ideal world, I'd like to go through all or at least some of the software listed [here](https://github.com/ivanreese/visual-programming-codex/blob/main/implementations.md), trying and analyzing them. Instead, I focused in looking for the usage of those features in some select apps, trying to grasp the underlying principles behind such usage. I read some online documentation and watched some videos demonstrating the features in action.

I mostly looked into the [Blueprint Visual Scripting system](https://docs.unrealengine.com/5.0/en-US/blueprints-visual-scripting-in-unreal-engine/). This was not due to personal preference nor familiarity. I actually had never used it. However, in concept it is the closest app to Nodezator in existence that I know of. That is, it is a system whose visual components aim to help organize components from an existing language visually. The difference is that Blueprints uses C++ while Nodezator uses Python.

I also looked into other apps, mostly those already mentioned by some Nodezator users like Ryven, StremeCoder and even apps not directly related to node editing like some CAD software.

Before this research I already had a complete design in mind for the implementation of branching (if/elif/else/match/case statements) in Nodezator. The research didn't change my mind at all regarding such design, which means we were already in a good direction. The research was important regardless, since it helped ensure I wasn't missing anything important. Regarding looping (for/while statements), I came to the conclusion that the idea of a looping feature for Nodezator isn't needed at all.


## Execution flow in Nodezator

Before we jump into how to implement branching into Nodezator, it is useful to understand how execution happens in Nodezator.

Nodezator structures its node layouts as a [graph](https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FGraph_%28discrete_mathematics%29), more specifically as a [DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph)(a directed acyclic graph). In other words, the node layout is strucutured using vertices and directed edges. Each vertice is a call, and the sockets represent the gates through which the data flows into and out of the vertices. Incomming data that go in the call as arguments and outgoing data leave the call as the return value.

Each edge is a connection between the output sockets of a node and the input sockets of the next one, represented by a line. The edges have a clear direction (from output to input sockets). The graph is acyclic, that is, the flow doesn't form cycles. This means a vertice is never visited more than once. In other words, the edges never lead the data into a node already visited. Even though the lines in the figure below aren't used in the software, they are implied due to the relationship between the different sockets in the nodes. We know that data comes from output sockets into input sockets, so the arrows are not needed.

And like this, whenever the node layout is executed, each node is visitied, called, and its output sent to the next nodes. This goes on until each one is executed. But in which order are the nodes executed? The answer is as soon as it has all data needed to execute. So a node will be executed regardless of its position in the 2D space. No matter if it is the first one from left to right or the last one, or whether it is in a higher spot. Think of it like fire spreading through the nodes and their connections, starting from the nodes with enough data to execute (for instance, nodes that don't need to wait for incomming data from other nodes).

As soon as a node finishes executing, Nodezator sends its output(s) to the other nodes connected through each connection. This way, each and all of the connections in the entire node layout are used to forward data, sooner or later, when the node executes.


## Approach to implement branching in Nodezator

Now let's think of the connections (edges) between the nodes (vertices) as paths. As we just explained, all paths in a node layout are travelled sooner or later when the node layout executes. It means no path in particular is chosen over another, all paths are treated equal. Data doesn't pick a specific path, it travels all of them.

Thus, in order to allow branching in Nodezator, we simply need a way to tell Nodezator to use specific paths and ignore others.


## Why a looping feature isn't needed

Regarding for-loops, Nodezator is already flexible and versatile enough to handle all the use-cases I could find or think of. Most use-cases I could find handled looping the same way: by passing each item from an iterable, one at a time, through a series of nodes until all the items were visited. Then, the flow resumed by following another predefined path triggered by the end of the iteration. However, this can be easily reproduced in Nodezator with help from the built-in node that uses the map() function, where we can specify a callable to go through all of the items inside an iterable. Python already provides an innumerable ever-increasing collection of callabes that process. Moreover iteration in Python can be customised in any level desirable using iterator functions with `yield` and `yield from` statements.

For while-loops the situation is even


Having the flow cycle through the same node is also incompatible with the [DAG structure](https://en.wikipedia.org/wiki/Directed_acyclic_graph) used by Nodezator.

I came to the conclusion that due to Nodezator's directed acyclic graph execution approach,
