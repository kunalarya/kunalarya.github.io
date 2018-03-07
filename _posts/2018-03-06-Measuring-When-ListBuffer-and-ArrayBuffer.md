---
layout: post
title:  "Measuring when ListBuffer and ArrayBuffer Inserts Become Linear"
---

The
[Performance Characteristics](https://docs.scala-lang.org/overviews/collections/performance-characteristics.html)
documentation lists the various runtime costs of Scala's standard collections.

When recently looking over some Scala code, I was curious to find out when
linear-time insert actually matters, e.g. how big the collection has to be
before it starts to effect an actual linear-time performance cost.
Often, for very small collections (say, a dozen elements), O(N) and O(1)
are practically indistinguishable.

To test this, `ListBuffer` and `ArrayBuffer` insert performance was measured
using the following approach:
1. The buffer was initialized with `N` elements.
2. A fixed number of elements were inserted (100 in this case).
3. Steps 1 and 2 were repeated 20 times and the average run time was recorded,
ignoring the first several iterations to allow the cache and JIT to warm up.

<h2>ListBuffer</h2>

`ListBuffer`s performance is tied very closely to its implementation, which is
a reference to the first element (its head) and recursively defined "tail",
i.e. the remainder of the `List`. (see [1]).

{% include image.html name="list_buffer.png" %}

<h2>ArrayBuffer</h2>

Appending to an array buffer is effectively constant time, as is outlined by
the documentation.

{% include image.html name="array_buffer.png" %}

<h2>Queue</h2>

Since we only insert at the head and pop off the tail of queues, we would
expect at least one of these operations to be constant. In fact, despite the
documentation, neither `enqueue` and `dequeue` operations were linear for the
tests run.

{% include image.html name="queue.png" %}



[1] https://stackoverflow.com/a/5753935
