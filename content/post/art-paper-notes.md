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
node actually has, rather than how many it might have. As the number of children in a node changes,
ART swaps out one implementation of the node structure for another.

The paper distinguishes four different cases, for nodes with up to 4, 16, 48 and 256 children
respectively. Assume we have a union type like the following:

{{< highlight "C++" "style=dracula" >}}
    union Node {
        Node4;
        Node16;
        Node48;
        Node256;
    }
{{< / highlight >}}

This allows us to talk about all the different types of `Node`, and pointers to them, in the
following without trying to make them all inherit from a base class.

#### `Node4`

For nodes with up to four children, ART stores all the keys in a list, and the child pointers in a
parallel list. Looking up the next character in a string means searching the list of child keys, and
then using the index to look up the corresponding pointer. Although this is superficially a less
efficient lookup algorithm, the small size of the child keys arrays means that the constant factor
will dominate.

{{< highlight "C++" "style=dracula" >}}
    struct Node4 {
        Value* value;
        char child_keys[4];
        Node* child_pointers[4];
        byte num_children;
    }

    Node* find_child(char c, Node4* node) {
        for (int i = 0; i < node->num_children; ++i) {
            if (child_keys[i] == c) return child_pointers[i];
        }

        return NULL;
    }
{{< / highlight >}}

Pointers are assumed to be 8 bytes, so a single `Node4` is 45 bytes, so sits in a single cache
line. The search loop can also be unrolled, but the branch in each loop iteration can't be obviously
elided and is not easily predicted. Still, the locality benefits make this a very efficient
structure for very sparsely populated trie nodes.

#### `Node16`

Nodes with from 5 to 16 children have an identical layout to `Node4`, just with 16 children per node:

{{< highlight "C++" "style=dracula" >}}
    struct Node16 {
        Value* value;
        char child_keys[16];
        Node* child_pointers[16];
        byte num_children;
    }
{{< / highlight >}}

Keys in a `Node16` are stored sorted, so binary search could be used to find a particular key. Since
there are only 16 of them, it's also possible to search all the keys in parallel using SIMD. What
follows is an annotated version of the algorithm presented in the paper's Fig 8.

{{< highlight "C++" "style=dracula" >}}
    // Find the child in `node` that matches `c` by examining all child nodes, in parallel.
    Node* find_child(char c, Node16* node) {
        // key_vec is 16 repeated copies of the searched-for byte, one for every possible position
        // in child_keys that needs to be searched.
        __mm128i key_vec = _mm_set1_epi8(c);

        // Compare all child_keys to 'c' in parallel. Don't worry if some of the keys aren't valid,
        // we'll mask the results to only consider the valid ones below.
        __mm128i results = _mm_cmpeq_epi8(key_vec, node->child_keys);

        // Build a mask to select only the first node->num_children values from the comparison
        // (because the other values are meaningless)
        int mask = (1 << node->num_children) - 1;

        // Change the results of the comparison into a bitfield, masking off any invalid comparisons.
        int bitfield = _mm_movemask_epi8(results) & mask;

        // No match if there are no '1's in the bitfield.
        if (bitfield == 0) return NULL;

        // Find the index of the first '1' in the bitfield by counting the leading zeros.
        int idx = ctz(bitfield);

        return node->child_pointers[idx];
    }
{{< / highlight >}}

This is superior to binary-search: no branches (except for the test when bitfield is 0), and all the
comparisons are done in parallel. The `Node16` takes up 150 bytes, but the `child_keys` field is
only 16 bytes.

#### `Node48`

The next node can hold up to four times as many keys as a `Node16`. As the paper says, when there
are more than 16 children, searching for the key can become expensive, so instead the keys are
stored implicitly in an array of 256 indexes. The entries in that array index a separate array of up
to 48 pointers.

{{< highlight "C++" "style=dracula" >}}
    struct Node48 {
        Value* value;
        // Indexed by the key value, i.e. the child pointer for 'f'
        // is at child_ptrs[child_ptr_indexes['f']]
        char child_ptr_indexes[256];

        Node* child_ptrs[48];
        char num_children;
    }
{{< / highlight >}}

The idea here is that this is superior to just storing an array of 256 `Node` pointers because you
can store 48 children in 640 bytes (where 256 pointers would take 2k). Looking up the pointer does
take an extra indirection:

{{< highlight "C++" "style=dracula" >}}
    Node* find_child(char c, Node48* node) {
        int idx = node->child_ptr_indexes[c];
        if (idx == -1) return NULL;

        return node->child_ptrs[idx];
    }
{{< / highlight >}}

The paper notes that in fact only 6 bytes (i.e. \\(log_2(48)\\)) are needed for each index; both in
the paper and here it's simpler to use a byte per index to avoid any shifting and masking.

#### `Node256`

The final node type is the traditional trie node, used when a node has between 49 and 256 children.

{{< highlight "C++" "style=dracula" >}}
    struct Node256 {
        Value* value;

        Node* child_ptrs[256];
    }

    Node* find_child(char c, Node256* node) {
        return child_ptrs[c];
    }
{{< / highlight >}}

Looking up child pointers is obviously very efficient - the most efficient of all the node types -
and when occupancy is at least 49 children the wasted space is less significant (although not 0 by
any stretch of the imagination).

##### Storing values

My version of the above

##### Other tricks: lazy compression and path expansion

Two techniques are described that allows the trie to shrink its height if there are nodes that only
have one child. The paper claims these are "well-known", and therefore not novel, but they reduce
the impact of having either strings with a unique suffix (lazy expansion) or a common prefix (path
compression).

#### Lazy expansion

If some node in a trie has only one child, and that node only has one child, and that node only has
one child, and so on all the way to a leaf, there isn't much sense in storing all the overhead of a
node per character. That path in the trie describes exactly one string, so why not just store that
string in the first node and get rid of all the rest? This is called _lazy expansion_, so-termed
because any futher insertions into this path must cause it to be expanded so the trie can be
searched as normal.

Implementing lazy expansion really just means having a pointer in every node to a key.
