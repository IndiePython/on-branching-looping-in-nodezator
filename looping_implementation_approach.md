# Approach to implement looping in Nodezator

[Back to table of contents](README.md)

A looping feature per-se, that is, the representation of for-loops and while-loops in Nodezator is not needed and actually doesn't make sense, as I'll discuss in detail. However, this is not to say that in its current state Nodezator is capable of tapping into the full potential of iterating/looping over data. Let's revisit some concepts and tools regarding iteration in Python and, as we reviews some concepts and tools I'll present the current problems and my proposal to solve them.

Looping as I found in other node editors consist of just being able to pass items from a collection through a set of nodes individually, instead of just passing the whole collection at once. However, the implementation of iteration/looping in Python is already super versatile and flexible, more than people usually care to learn.

In summary, rather than needing for-loops to iterate over each item in a collection, Python also provides built-in functions that do that for you, as well as several standard library functions as well. If this is not enough, you can even define generator-functions using the `yield` or `yield from` statements. Now, for-loops are fine, it is just that Python allows as much power and flexibility without using them and even an advantage, as we'll present shortly.

The map() built-in function takes a callable (for instance, a function) and one or more iterables (for instance, a collection) and applies the function to each item or group of items, returning an iterator containing the result of each call. For instance, if you have a list of strings and you want to make sure there's no whitespace at the beginning and ending of the strings, you can just...

```python
### ...do this
new_list = list(map(str.strip, list_of_strings))

### ...instead of this
new_list =  [
    item.strip()
    for item in list_of_strings
]
```

Note that, since map() returns an iterator, you need to wrap the call into a call to list(), so the items are stored in a list.

In Nodezator, this can be done like this...

![nodezator graph as SVG from exporting ndz file](images/img08_map_example.svg)

However, there are 02 things in the nodezator graph about which I'm not happy: first, accessing string methods is too convoluted (it took 03 nodes to access str.strip(), as shown in the image). Nonetheless, this problem is temporary, since I intend to make string methods available by default in Nodezator, each with its own node, since they are very useful.

The second and more deep problem, is that even if we had a `str.strip()` node, we would still not be able to reference the str.strip callable directly. As we know, nodes represent calls, not the callables themselves. Which means we cannot reference the callables directly in the graph. This is probably the biggest problem regarding the usage of iteration in Nodezator because what is the point of being able to use map() to apply functions over items in a collection if we can't access our node callables directly and thus would have to write custom functions anyway?

The answer is simple: there should be a way to access the callables of the nodes directly in Nodezator. After all, this is actually super simple, no? In plain Python code, the only thing differentiating a function call from the function itself is the usage of parentheses besides the callable. That's what people mean when they say functions are first-class citizens in Python. Functions are just objects that we can call. In Python, callables are just another kind of object, a kind of data describing a behavior.

So here's my proposed solution: to be able to reference the callable from a node directly in the graph. In practice, with the single click of a button we should be able to change a node in the graph so that rather than represent the call, it would represent the callable itself, and could thus itself be fed into other nodes. In other words, the same way we plan to be able to mute nodes by pressing "m" in the future, by pressing a single button we'd change the node to this special form.

Let's see this in action. Let's pretend we already introduced new default (app-defined) nodes in Nodezator representing string methods as we intend to do. One of such nodes would be the str.strip() node. Here's a representation:

![str.strip() node represented as an SVG image](images/str_strip_node.svg)

So this node is already useful as it is. You can just pass a string to it and have a stripped version of it come out as its output. However, as Nodezator is right now, we cannot directly use the `str.strip` callable in conjunction with the map() node. The only thing we can do with it right now is just perform the single call which this node represents and obtain the output.

As we proposed, we want to be able to press a single button and have the node switch into a mode that allows us to reference its callable. For now, let's use the 'c' key. Effectively, we'd be making it so the node represents data (the callable itself), rather than a call. So, by pressing the "c" key (for "callable reference", or "citizen", from "first-class citizen" nomenclature), the node switches to the new mode and changes shape to indicate it. It now has a single output socket that allows us to pass on a reference to the callable:

![str.strip() node in citizen mode represented as an SVG image](images/str_strip_citizen_mode.svg)

And so we can use it in conjuntion with the map() node seamlessly:

![str.strip() node in citizen mode used with map node, all represented as an SVG image](images/str_strip_citizen_mode_in_action.svg)

You'll be able to do this with any callable node you want, be it app-defined or the ones you define yourself. This means your nodes will be even more versatile. You'll be able to write nodes that focus in processing a single piece of data only. Then, when/if you need to process several items, you just have to convert it to "callable reference" mode and pass the reference to the map() node, along with your collection of items. Using this mechanism you'll be able to perform all kinds of complex stuff, like chaining functions using map() to treat multiple items at once:

![multiple citizen mode nodes chained with map nodes, all represented as an SVG image](images/citizen_mode_nodes_in_action.svg)

So, with this problem out of the way with the help of this new support feature proposed, we can tap into the full potential of iterating/looping over data in Nodezator. Let's keep exploring more use-cases.

Within the body of a for-loop, we may not just apply a function to each item in our collection. We sometimes want to filter out the items, keeping only those that meet certain criteria. For this, we have functions like the built-in filter(), which takes a function or None as the first argument and an iterable as the second argument, and returns an iterator containing only the items for which applying function(item) evaluates to a truthy value. If the first argument is `None`, instead, all truthy items are kept.

<svg width="260" height="30" style="background-color:red;">
    <text x="5" y="20" fill="white">Image with usage of filter() node</text>
</svg>

If we need, there's also a standard library function opposite to filter(), which is itertools.filterfalse(), which keeps only items which evaluate to falsy values.

<svg width="350" height="30" style="background-color:red;">
    <text x="5" y="20" fill="white">Image with usage of itertools.filterfalse() node</text>
</svg>

itertools.dropwhile() and itertools.takewhile() can also be used to filter items. The former ignores items while they evaluate to truthy values with the given function, until one of them evaluates to a falsy value, at which point such item and all subsequent ones are kept. The latter, as the name implies, does the opposite, keeping successive items that evaluate to truthy values until an item evaluates to a falsy value, at which point the item is dropped along with all remaining ones.

<svg width="350" height="30" style="background-color:red;">
    <text x="5" y="20" fill="white">Image with usage of itertools.dropwhile() node</text>
</svg>

<svg width="350" height="30" style="background-color:red;">
    <text x="5" y="20" fill="white">Image with usage of itertools.takewhile() node</text>
</svg>


---

Having the flow cycle through the same node is also incompatible with the [DAG structure](https://en.wikipedia.org/wiki/Directed_acyclic_graph) used by Nodezator.

I came to the conclusion that due to Nodezator's directed acyclic graph execution approach,
