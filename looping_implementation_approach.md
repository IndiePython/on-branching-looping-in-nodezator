# Approach to implement looping in Nodezator

[Back to table of contents](README.md)

If one is aware of how versatile iterating/looping over data is in Python, one could say that a looping feature, that is, the representation of for-loops and while-loops in Nodezator, is not strictly needed in Nodezator. This is so because Python programming allows multiple paradigms and one of such paradigms is functional programming, which is specially compatible with Nodezator, requiring very few changes in Nodezator in order to tap in the full potential of iteration.

Because of that I was actually convinced of not implementing looping in Nodezator as found in other node editors. Such looping from other node editors consists of being able to pass items from a collection through a set of nodes individually, instead of just passing the whole collection at once.

However, this functional approach I wanted to pursue may still be confusing/intimidating to some users due to the higher popularity of for-loops. And to be honest, for-loops are simple and fine enough on their own, and known by most, which is a big plus.

Because of that, rather than pursuing only the functional approach, I decided to actually come up with new kinds of nodes meant to emulate for-loops as the main tool for looping in Nodezator. And also implement the functional approach as an alternative and complement to it.

Also, for the sake of simplicity, let's pretend we introduced new default (app-defined) nodes in Nodezator representing string methods. This is actually something we intend to do in the near future. One of such nodes, for instance, would be the str.strip() node. Here's a first representation (the final form will probably have an string entry widget attached to it):

![str.strip() node represented as an SVG image](images/str_strip_node.svg)

Without further ado, I'll present the nodes emulating a for-loop and after that I'll present the functional approach for those that might be interested.


## Nodes emulating a for-loop

To emulate a for-loop, I plan to introduce 02 new nodes. The first one can be used by itself. It is called a `for_loop` node. The second one can only be used in conjunction with the first one and is called the `item_catching` node. Here's a representation of the nodes:

![the `for_loop` and `item_catching` nodes, in SVG](images/for_loop_and_item_catching_nodes.svg)


### Emulating a "regular" for-loop

When used by itself, the first node represents a regular for-loop in Python. In other words, you pick one item from the given iterable at a time and pass it through each connected node. Here's an example:

![`for_loop` node used by itself](images/for_loop_node_in_action.svg)

In the example above, we convert and save a list of JSON files (`file_a.json`, `file_b.json`, ...) as YAML files. The list of file names (in the `list_from_args` node) is passed to a `for_loop` node. Since this node emulates a for-loop, the nodes following it form the body of a for-loop. The `for_loop` node thus passes each item through the following nodes one at a time, causing each item to have their JSON content loaded, converted into YAML and saved using the same file name, but with the suffix changed from `.json` to `.yaml`.


### Emulating a for-loop used in a comprehension/generator expression

When used in conjunction, the `for_loop` and `item_catching` node work as a for-loop inside a comprehension on generator-expression. As before, the first node takes an iterable and sends each item one by one to the nodes connected to it. Once the iteration finishes, the `item_catching` node returns an iterator containing all the items that reached it. Here's an example:

![the `for_loop` node and `item_catching` nodes in action, in SVG](images/for_loop_and_item_catching_nodes_in_action.svg)

In the image, the body of the comprehension/generator expression is clearly defined by the nodes between the `for_loop` and `item_catching` nodes. That is, in this loop, each item of the list given at the beginning is passed to a str.strip and then to str.title. The resulting iterator contain the all the individual items that reached the `item_catching` node, so we catch all those items within a new list object. And thus, we have a complete example demonstrating how we took the items of a list and used them to generate a new list by processing the items of the first one.

Note that the `item` output of the `for_loop` node and the `item` input of the `item_catching` node are represented by half-circles. This is just to represent that the nodes complement each other, that items comming from the `for_loop` node are expected to reach the `item_catching` node and that the data that travels between them is fragmented into individual items that travel one at a time.


### Using the nodes to filter

In this other example, we extend the node layout from the previous example to also filter out items according to a given criteria. To do this, we just need to ensure only the items that meet our criteria reach the `item_catching` node. For this, we can use a `if_elif_else` node presented in an early section.

![`for-loop nodes` with `branching nodes` to filter items, in SVG](images/using_looping_and_branching_nodes_to_filter.svg)

As can be seen in the image above, only the items that evaluate to True by the str.isidentifier() function reach the `item` parameter of the `item_catching` node and thus are integrated in the new list created.


## Functional approach to looping/iterating over data

However, this is not to say that in its current state Nodezator isn't capable of tapping into the full potential of iterating/looping over data. Let's revisit some concepts and tools regarding iteration in Python and, as we review some concepts and tools I'll present the functional approach to achieve such results.

In summary, rather than needing for-loops to iterate over each item in a collection, Python also provides built-in functions that do that for you, as well as several standard library functions as well. If this is not enough, you can even define generator-functions using the `yield` or `yield from` statements. Now, for-loops are fine, it is just that Python allows as much power and flexibility without using them. There's also an advantage of using the functional approach, which I'll also present.


### The map() built-in function

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


### Problems in last example and solution

However, there are 02 things in the nodezator graph above about which I'm not happy: first, accessing string methods is too convoluted (it took 03 nodes to access str.strip(), as shown in the image). Nonetheless, this problem is temporary, since I intend to make string methods available by default in Nodezator, each with its own node, since they are very useful.

The second and more deep problem, is that even if we had a `str.strip()` node, we would still not be able to reference the str.strip() callable directly. As we know, nodes represent calls, not the callables themselves. Which means we cannot reference the callables directly in the graph. This is probably the biggest problem regarding the usage of iteration in Nodezator because what is the point of being able to use map() to apply functions over items in a collection if we can't access our node callables directly and thus would have to write custom functions anyway?

There should be a way to access the callables of the nodes directly in Nodezator. After all, this is actually super simple, no? In plain Python code, the only thing differentiating a function call from the function itself is the usage of parentheses besides the callable. That's what people mean when they say functions are first-class citizens in Python. Functions are just objects like any other. Objects that we can call. A kind of data describing behavior.

So here's my proposed solution: to be able to reference the callable from a node directly in the graph. In practice, with the single click of a button we should be able to change a node in the graph so that rather than represent the call, it would represent the callable itself, and could thus itself be fed into other nodes. In other words, the same way we plan to be able to mute nodes by pressing "m" in the future, by pressing a single button we'd change the node to this special form.

Let's see this in action.

As we know, the `str.strip()` node is already useful as it is. You can just pass a string to it and have a stripped version of it come out as its output. However, as Nodezator is right now, we cannot directly use the `str.strip` callable in conjunction with the map() node. The only thing we can do with it right now is just perform the single call which this node represents and obtain the output.

As we proposed, we want to be able to press a single button and have the node switch into a mode that allows us to reference its callable. For now, let's use the 'c' key. Effectively, we'd be making it so the node represents data (the callable itself), rather than a call. So, by pressing the "c" key (for "callable reference", or "citizen", from "first-class citizen" nomenclature), the node switches to the new mode and changes shape to indicate it. It now has a single output socket that allows us to pass on a reference to the callable:

![str.strip() node in citizen mode represented as an SVG image](images/str_strip_citizen_mode.svg)

You'll be able to do this with any callable node you want, be it app-defined or the ones you define yourself. This means your nodes will be even more versatile. You'll be able to write nodes that focus in processing a single piece of data only. Then, when/if you need to process several items, you just have to convert it to "callable reference"/citizen mode and pass the reference to the map() node, along with your collection of items.


### Combining new solution with map()

And so we can use our new solution in conjuntion with the map() node seamlessly:

![str.strip() node in citizen mode used with map node, all represented as an SVG image](images/str_strip_citizen_mode_in_action.svg)

Using this mechanism you'll be able to perform all kinds of complex stuff, like chaining functions using map() to treat multiple items at once:

![multiple citizen mode nodes chained with map nodes, all represented as an SVG image](images/citizen_mode_nodes_in_action.svg)

So, with this problem out of the way with the help of this new support feature proposed, we can tap into the full potential of iterating/looping over data in Nodezator using the functional approach. Let's keep exploring more use-cases.


### Filter with built-in filter() and other functions

Within the body of a for-loop, we may not just apply a function to each item in our collection. We sometimes want to filter out the items, keeping only those that meet certain criteria, like in a past example where we use str.isidentifer() to filter out strings which don't represent valid Python identifiers.

For this, we have functions like the built-in filter(), which takes a function or None as the first argument and an iterable as the second argument, and returns an iterator containing only the items for which applying function(item) evaluates to a truthy value. If the first argument is `None`, instead, all truthy items are kept. In the example below, we pass the items of our list through the str.title() function using map() and also filter the result so only string which evaluate to True by str.isidentifier() are kept.

![Image with usage of filter() node](images/filter_node_in_action.svg)

If we need, there's also a standard library function opposite to filter(), which is itertools.filterfalse(), which keeps only items which evaluate to falsy values. If we were to replace the `filter` node in the example above by a `filterfalse` node, instead of items that evaluate to True by the str.isidentifier node we'd have items that evaluate to False, that is, strings which do not represent valid Python identifiers. If the first argument is passed to filterfalse() is `None` instead, all falsy items are kept.

There are also other useful functions in the standard library that can be used to filter iterables. The functions itertools.dropwhile() and itertools.takewhile() are some examples. The former ignores items while they evaluate to truthy values with the given function, until one of them evaluates to a falsy value, at which point such item and all subsequent ones are kept. The latter, as the name implies, does the opposite, keeping successive items that evaluate to truthy values until an item evaluates to a falsy value, at which point the item is dropped along with all remaining ones.


### Differences between for-loops and the functional approach

As we just demonstrated, the functional approach can be used to both apply operation in all items from an iterable as well as filter items from it. In that sense, both for-loops and functional approaches are equivalent. However, sometimes you just want to repeat calls without worrying about catching the transformed/filtered items. For instance, in a previous example, we executed a for-loop to convert JSON files into YAML files, using only the `for-loop` node, not caring to catch any of the generated data (we didn't use an `item_catching` node). When this is the case, for-loops are virtually irreplaceable. For all other cases, both approaches are more or less interchangeable.

We say virtually irreplaceable because one could in fact use map/filter functions even when there's no interest in catching the generated items. However, in such cases, simple for-loops are simpler and using map/filter functions becomes a bit cumbersome because you'd have to create a dummy collection in the end to catch return values (`None`) in which you are not interested. For instance, this is how the example we mentioned (converting JSON files into YAML files) would be using a functional approach.

![json to yaml example using functional approach](images/json_to_yaml_functional_approach.svg)

But surely, the functional approach is convoluted here. As we said early, we needed to create a dummy collection, using a `list()` node here, in order to consume the values generated by the iterator, even though we are not interested in the valus generated (just `None` values). Additionally, we had to use `repeat` nodes (itertools.repeat) to feed the other arguments to the `replace` node (str.replace) continuously until all items from the list of paths are depleted.

There's a technical reason why the dummy collection is needed. It is actually a good thing when used for the right kind of the problem. We'll summarize it in the next subsection.


### Advantages of chaining functions that return iterators

In the functional approach as implemented in Python, the evaluation is lazy, and happens only when the final iterator is consumed. Such consumption happens when the iterator encounters a regular for-loop or a function that returns a collection (because internally the functions consumes all items, instead of treating them one by one). While the iterators created and passed along are not consumed, the processing piles over and over. They are lazily evaluated, that is, the processing only happens when the iterators are consumed, each items is consumed one at a time, going through each function on the chain. Internally, this results in a gain of speed.

I won't go into details on the gain in speed of chaining functions that return iterators, but such gain is properly documented by David Beazley in many of its [tutorials available online](https://www.dabeaz.com/tutorials.html). More specifically, you should look into these tutorials:

- Generator Tricks for Systems Programmers;
- Python Generator Hacking;
- Generators: The Final Frontier;

Since it is not the purpose of this article to go deep in the subject, I tried to keep it as simple as possible and free of complex stuff. If you want to know more about iterators, generator functions and iteration in Python in general I also recommend content from Luciano Ramalho. Specifically, I recommend chapter 14 from his [Fluent Python book](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/), 1st edition, or chapter 17 of the 2nd edition, both containing info on iterators, generators and more.

Even so, it doesn't mean the functional approach with iterators is better than plain for-loops. I'd say that, except in very specific scenarios, a simple for-loop is probably enough. I strongly recommend you to look into the materials mentioned in order to get a firm grasp of the useful concepts needed in order to make such decisions.


### Using custom generator-functions

As I mentioned earlier, you can also write custom generator functions that can be chained to take advantage of the gain in speed. Of course, it is up to you whether you use this approach or not. It depends on your use-case, on the purpose of your node pack(s).

For instance, I have a node pack I use to create data representing movement in 2d space for animating game objects/characters. I didn't publish yet, but don't worry, I plan to release it to the public domain in the near future. Here's an example of it in action (clicking the thumbnail will redirect to the spot in an youtube video where I demonstrate the node pack in action):

[![thumb of youtube video](https://img.youtube.com/vi/GlQJvuU7Z_8/hqdefault.jpg)](https://www.youtube.com/watch?v=GlQJvuU7Z_8&t=1766s)

[keep writing here]

---

<!-- Other loose excerpts about stuff to discuss here (you can ignore this for now):

"Having the flow cycle through the same node is also incompatible with the [DAG structure](https://en.wikipedia.org/wiki/Directed_acyclic_graph) used by Nodezator."

"I came to the conclusion that due to Nodezator's directed acyclic graph execution approach..."
-->
