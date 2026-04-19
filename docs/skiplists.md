---
title: Skip Lists
layout: default
nav_order: 2
---

# Skip Lists In YAKV

## Memtables

When building an LSM storage engine a very important component is the
"memtable". The memtable is a in-memory data structure to efficiently buffer
recent mutating operations on the logs.

Ideally the memtable should be implemented such that it can efficiently
maintain a sort on a list of items and enable relatively quick insertions and
deletions.

There's many different ways to implement a memtable:
- a vector
- a singly linked list
- a skiplist
- red-black tree
- ...

For YAKV, a skiplist was chosen for the following reasons:
- concurrent writers are possible
- good time complexity characteristics for this application
- ease of implementation

## The perils of using a linked-list

When working with a singly-linked list, one of the main drawbacks is that
navigating to a given node (to see if it exists or find an insertion point)
can be quite costly. In particular, if the node you are looking for is at the
end of the list, you would need to traverse all of the nodes in the worst case.

Using a doubly linked list instead helps slightly by giving you a pointer to
the last element in the list but now navigating to the middle of the linked list
could require you to read up to half the nodes in the worst case. This isn't
super great as our lists get bigger.

## Skiplists

The skiplist looks to solve this issue, by creating pointers to "jump"
across multiple nodes at a single time. Consider you have a singly linked list
but for every other node you create a pointer to the next next node (i.e. you
jump two nodes ahead instead of one).

![linked list with skips every other nodes]({{site.baseurl}}/assets/images/skiplists/linked-list-with-skips.png)

This is a slight improvement, as you could imagine that this would allow us to
make larger jumps to traverse faster using an algorithm that roughly looks like
the following:

```
node = head
while node.key != target_key:
    if node.farjump.key <= target_key:
        node = node.farjump
    else:
        node = node.next

if node == target:
    return node
```

Of course in this case having the skips every other node isn't optimal
(in this case say a skip every 3 nodes would be better for this particular
list), however, this general framework of adding "skips" is useful.

With the optimal choice for distance between skips (which is the square root
of the number of elements in the list, I'll leave this as an exercise to the
reader) we get a worst case traversal of O(√n), which isn't terrible.

This is a bit idealistic though. The number of elements in the list changes over
time, so in practice it's tricky to guarantee you could achieve and maintain
that performance after you choose that constant.

Regardless, we can do even better if we use some additional memory.
The idea is intuitive: if we're adding skips to our underlying list,
why not add skips to our skips?

Taking our example from last time imagine we added a skip to every other skip:

![linked list with skips on skips]({{site.baseurl}}/assets/images/skiplists/linked-list-with-leveled-skips.png)

You can then use the same process as the algorithm suggested above in order
to find the appropriate skip you want to visit.

You could imagine that as your top "level" of skips grows to be too big
you can simple just add another layer of skips onto that layer. This gives
a logarithmic profile for traversing the list while giving good performance
for inserts, deletes, and writes (O(1) for pointer updates + logarithmic
runtime for traversal) which is pretty good.

Often times to reinforce the idea of "levels" in a skiplist, skiplists are
illustrated as follows having "levels" of pointers with the bottom layer just
being the pointer between adjacent elements in the list while the "higher"
level skips allow for very large jumps to cut down the search space very quickly.

![linked list with levels illustration]({{site.baseurl}}/assets/images/skiplists/skiplist.png)

## Probabilistic magic

One problem that a keen reader may have noticed: what happens if you delete
a node that contains a very high level skip? Well, you'd need to delete it
and its associated skips. This could be a big problem since you now have fewer
high-level skips acting as guideposts, requiring you to search a much bigger
range of nodes in the layer below.

Skiplists solve this in a pretty simple way: they will determine *at random*
the height of nodes, this makes the implementation far simpler as once a height
is determined it remains constant and thus no rebalancing is ever required.

The height of a node is determined on insert using an algorithm similar to:

```
height = 0
while random() > promotion %
    height++
```

In this case the programmer would pick a % chance (50% is quite common) that
the node would be promoted to the next level and repeatedly sample this until
they didn't get the desired outcome from their randomly generated values.

For example: consider the chance was 50% that a node be promoted up then
there would be a 50% chance it remains level 0, 25% chance it reaches level 1,
12.5% that it reaches level 2 and so on.

By using random sampling to determine the height of nodes we avoid
problematic scenarios where we deleted all of our top level nodes. This gives
an average time complexity for insert, delete, and get to be O(log(n)) but
a worse case of O(n).
