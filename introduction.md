# Introduction

[Back to the table of contents](README.md)

I spent the last week (from September 25th to 30th) researching the usage of branching and looping in other apps. I wanted to make sure I didn't miss anything important when implementing such features in Nodezator. In this text I present the the approach I intend to follow to implement branching and looping in Nodezator after such research.

First of all, the research wasn't anything deep nor time-consuming. That would require more time and experimentation. In an ideal world, I'd like to go through all of the software listed [here](https://github.com/ivanreese/visual-programming-codex/blob/main/implementations.md) or at least some of it, trying and analyzing them. Instead, I focused in looking for the usage of those features in some select apps, trying to grasp the underlying principles behind such usage. I read some online documentation and watched some videos demonstrating the features in action.

I mostly looked into the [Blueprint Visual Scripting system](https://docs.unrealengine.com/5.0/en-US/blueprints-visual-scripting-in-unreal-engine/). This was not due to personal preference nor familiarity. I actually had never used it. However, in concept it is the closest app to Nodezator in existence that I know of. That is, it is a system whose visual components aim to help organize components from an existing language visually. The difference is that Blueprints uses C++ while Nodezator uses Python.

I also looked into other apps like Ryven, StremeCoder, mostly those already mentioned by some Nodezator users. I even looked into some apps not directly related to node editing like some CAD software.

Before this research I already had a complete design in mind for the implementation of branching (if/elif/else/match/case statements) in Nodezator. The research didn't change my mind at all regarding such design, which means we were already in a good direction. The research was important regardless, since it helped ensure I wasn't missing anything important.

Regarding looping (for/while statements), I was convinced at first that the idea of a looping feature for Nodezator as implemented in other apps wasn't needed at all. This was so because iterating in Python is versatile enough that it can be used with a functional approach, just by chaining existing callables. Of course, adding some other supporting features would still be needed.

Despite this, though, after receiving some initial feedback, I ended up thinking of a way to complement this functional approach. It is something similar to what is seen in other apps and consists of nodes that emulate for-loops. This solution is both more simple and familiar for most Python users.
