---
layout: post
title:  "Measuring when ListBuffer and ArrayBuffer Inserts Become Linear"
---

The
[Performance Characteristics](https://docs.scala-lang.org/overviews/collections/performance-characteristics.html)
documentation lists the runtime costs of Scala's standard collection objects.

When recently looking over some Scala code, I was curious to find out when
linear-time insert actually mattered, e.g. how big the collection has to be
before it starts to effect an actual linear-time performance cost.
Often, for very small collections (say, a dozen elements), O(N) and O(1)
are practically indistinguishable.

To test this, `ListBuffer` and `ArrayBuffer` insert performance was measured
using the following approach:
1. The buffer was initialized with `N` elements.
2. A fixed number of elements were inserted (100 in this case).
3. Steps 1 and 2 were repeated 20 times and the average run time was recorded,
ignoring the first several iterations to allow the cache and JIT to warm up.

It should be noted that this is not a high fidelity benchmark, as the test
harness is only minimally decoupled from the code under measurement. It
also uses `System.nanoTime` which is notoriously inaccurate. That
said, rolling my own was the best option I could find that met my needs.
The data suggests that these inaccuracies may not matter.

All of this data was collected on a 2015 Macbook Pro with 3.1 GHz i7.
[The code is available on Github](https://github.com/kunalarya/scala-perf-experiment).

<h2>ListBuffer</h2>

`ListBuffer`'s performance is tied very closely to its implementation, which is
a reference to the first element (its head) and recursively defined "tail",
i.e. the remainder of the `List` [see this SO post for details](https://stackoverflow.com/a/5753935).

Note that both axes are plotted on log scales to allow me to run finer
grains, making it clearer *where* the cutoff point is.

As the figure shows, for list buffers, the linear insert time penalty
happens after roughly 100 items are in the list. This is for a list of integers;
it's possible that larger datatypes will trigger a lower cutoff point between
effectively constant and linear.

{% include image.html name="list_buffer.png" %}

<h2>ArrayBuffer</h2>

Appending to an array buffer is effectively constant time, as is outlined by
the documentation.

Inserting at the head of an `ArrayBuffer` is the worst case insert cost. The
initial noise in the data is something I'd love to hear suggestions for;
for now, I'm assuming it's noise related to the specific interaction of its
implementation and the JIT.

{% include image.html name="array_buffer.png" %}

<h2>Queue</h2>

Since we only insert at the head and pop off the tail of queues, we would
expect at least one of these operations to be constant. In fact, despite the
documentation, neither `enqueue` and `dequeue` operations were linear for the
tests run.

What's interesting here is the graph on the left: you can see a somewhat
stepwise function that either corresponds to the cache size or an internal
cutoff between buffer allocations.

{% include image.html name="queue.png" %}
