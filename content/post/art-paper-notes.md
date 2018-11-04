---
title: "ART-ful radix tries"
date: 2018-11-03T22:04:31-07:00
author: Henry
permalink: /paper-notes-art/
draft: true
layout: post
categories:
    - Paper notes
    - Key-value stores
---
{{< key_value_series >}}

### The Adaptive Radix Tree: ARTful Indexing for Main-Memory Databases

_Leis et. al., ICDE 2013_

Tries are, in some sense, the underloved third way of building key-value storage, after search trees
(like B-trees) and hash tables. Yet they have a number of very appealing properties that make them
worth of consideration - for example, the height of a trie is independent of the number of keys it
contains, and a trie requires no rebalancing when updated. Weighing against those advantages is the
heavy memory cost that vanilla radix tries can incur, because each node contains a pointer for every
possible value of the 'next' character in the key. With ASCII as an example, that's 256 pointers for
every node in the tree.

But the astute reader will feel in their bones that this is naive - there must be more efficient
ways to store a set of pointers, indexed by a fixed size set of keys (the trie's alphabet). Indeed,
there are - several of them, in fact, distinguished by the number of children the node _actually_
has, not just how many it might _potentially_ have.

This is where the *Adaptive Radix Tree* (ART) comes in. In this breezy, easy-to-read paper, the
authors show how to reduce the memory cost of a regular radix trie by _adapting_ the data structure
used for each node to the number of children that it needs to store. In doing so they show, perhaps
surprisingly, that the amount of space consumed by a single key can be bounded no matter how long
the key is.

<!--more-->

##### What's wrong with ordinary tries?

Let's imagine how we might implement a trie, as simply as we can. A single node in a trie
corresponds to a _key prefix_; a sequence of characters that is a prefix of one or more keys in the
tree.

{{< highlight "C++" "style=dracula">}}
    struct TrieNode {
        Value* value;
        TrieNode* children[256];
    }

{{< / highlight >}}

Not much to it. Every node contains an optional `Value*` which is `NULL` if there's no key that
actually ends here - just keys with the path through this node as a prefix.

Other than that, we need to know how to get to all the nodes that represent key prefixes one
character longer than the current one, and that's what the `children` array is for. Here we're
assuming an 8-bit alphabet, with 256 possible characters per letter, or a _fan-out_ of 256.

The paper calls the width of a single character the _span_ of the trie, and it's a critical
parameter as it determines a trade-off between the fan-out of each node (and therefore the size of
the array in `TrieNode` above), and the height of the trie, since if you can pack more of a value
into a single node by having a larger span, you need fewer nodes to describe the whole string. We'll
talk more about the span below, but for now you can assume that tries perform best when the span is
more than just a couple of bits.

The problem, as you can no-doubt see, is that there's a possibility of a lot of wasted space in each
node. Imagine a radix trie containing `foo`, `fox` and `fat`. The root node would have one valid
pointer, to `f`, which would have two valid pointers, to `a` and `o`. `a` would have a pointer to
`t`, and `o` would have pointers to `o` and `x`.

So our trie would have 6 nodes, but a total of 6 * 26 = 156 pointers, of which 150 / 156 = 96% are
empty and therefore wasted space! At 8-bytes per pointer, that's already over 1K wasted space to
store just 9 bytes of information.

This example may seem to be contrived, partly because it is. But the memory overhead of tries is
real, and the paper establishes this in Fig 3, which plots the height of the trie against the space
required to store it for a keyset of 1M 32-bit values. The span is varied, which changes both the
height and the storage cost. At a span of 8-bits, the trie has height 4, but takes more than 128MB
to store. As the span gets larger, the memory overhead gets much, much worse for little gain in the
height of the trie; if the span is reduced the tree height gets larger very quickly for not much
memory benefit.

##### Adaptive node structures

The most significant change that ART makes to the standard trie structure is that it introduces the
ability to change the datastructure used for each internal node depending on how many children the
node actually has, rather than how many it might have. This is the 'small datastructures'.
