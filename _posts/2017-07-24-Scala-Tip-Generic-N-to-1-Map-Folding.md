---
layout: post
title: "Scala Tip: Generic N-to-1 Map Folding"
---

If you have `N` maps and would like to reduce them to a single map, the following will help.
If more than one map contains the same key, the map that appears later in the list will
take precedence.

{% highlight scala %}
def mergeMaps[TKey, TValue](maps: Seq[Map[TKey, TValue]]): Map[TKey, TValue] = {
  maps.foldLeft(Map.empty[TKey, TValue]) {
    case (a, b) => a ++ b
  }
}
{% endhighlight %}
