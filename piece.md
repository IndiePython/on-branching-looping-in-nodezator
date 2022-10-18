# Branching and looping in Nodezator

I spent the last week (from September 25th to 30th) researching the usage of branching and looping in other apps. I wanted to make sure I didn't miss anything important when implementing such features in Nodezator. In this text I present the conclusions I achieved and the approach I intend to follow to implement branching in Nodezator. I also explain why looping can already be performed in Nodezator and thus doesn't need to be implemented.

First of all, the research wasn't anything deep nor time-consuming. That would require more time and experimentation. In an ideal world, I'd like to go through all or at least some of the software listed [here](https://github.com/ivanreese/visual-programming-codex/blob/main/implementations.md), trying and analyzing them. Instead, I focused in looking for the usage of those features in some select apps, trying to grasp the underlying principles behind such usage. I read some online documentation and watched some videos demonstrating the features in action.

I mostly looked into the [Blueprint Visual Scripting system](https://docs.unrealengine.com/5.0/en-US/blueprints-visual-scripting-in-unreal-engine/). This was not due to personal preference nor familiarity. I actually had never used it. However, in concept it is the closest app to Nodezator in existence that I know of. That is, it is a system whose visual components aim to help organize components from an existing language visually. The difference is that Blueprints uses C++ while Nodezator uses Python.

I also looked into other apps, mostly those already mentioned by some Nodezator users like Ryven, StremeCoder and even apps not directly related to node editing like some CAD software. The sad part about the research is that I couldn't actually find a lot of written detailed material on the official websites/online sources for most of the apps researched. Don't get me wrong, I'm not blaming the people behind those projects, specially the ones maintaned by few unpaid selfless people. I know being a maintainer of an open-source project is very time-consuming.

Even so, the fact that most of those apps don't have a centralized hub of detailed information regarding the most basic info about the usage and elements of their apps is undoubtedly damaging to their adoption and survival. The key here is the word **detailed**, as the info made available is not enough.

Nodezator is not in a much better situation, but I'm relieved the decision to release a [manual](https://manual.nodezator.com) along with it has paid off and helped a lot of people. There's still a lot I need to write about in the manual, but the most critical information needed to understand and use Nodezator is there already, with enough detail and examples. Additionally, Nodezator was released less than 04 months ago, while some of the apps researched have been around for much longer without comprehensive manuals/guides.

Before this research I already had a complete design in mind for the implementation of branching (if/elif/else/match/case statements) in Nodezator. It turns out the research didn't change my mind at all. Regarding looping (for/while statements), however, I came to the conclusion that the idea of a looping feature for Nodezator isn't needed.


## Approach to implement branching in Nodezator


## Why a looping feature isn't needed

Regarding for-loops, Nodezator is already flexible and versatile enough to handle all the use-cases I could find or think of. Most use-cases I could find handled looping the same way: by passing each item from an iterable, one at a time, through a series of nodes until all the items were visited. Then, the flow resumed by following another predefined path triggered by the end of the iteration. However, this can be easily reproduced in Nodezator with help from the built-in node that uses the map() function, where we can specify a callable to go through all of the items inside an iterable. Python already provides an innumerable ever-increasing collection of callabes that process. Moreover iteration in Python can be customised in any level desirable using iterator functions with `yield` and `yield from` statements.

For while-loops the situation is even


Having the flow cycle through the same node is also incompatible with the [DAG structure](https://en.wikipedia.org/wiki/Directed_acyclic_graph) used by Nodezator.

I came to the conclusion that due to Nodezator's directed acyclic graph execution approach,