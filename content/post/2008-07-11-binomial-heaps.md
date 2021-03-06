---
id: 47
title: Binomial Heaps
date: 2008-07-11T17:35:44+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=46
permalink: /binomial-heaps/
categories:
  - Data structures
---
(The python code for this article is available [here](http://hnr.dnsalias.net/binomial_heap.py))

The standard binary heaps that everyone learns as part of a first algorithms course are very cool. They give guaranteed  \\(n\\) sorting cost, can be stored compactly in memory since they're full binary trees and allow for very fast implementations of priority queues. However, there are a couple of operations that we might be interested in that binary trees don't give us, at least not cheaply.

In particular, we might be concerned with merging two heaps together. Say, for example, that we're shutting down a processor with its own priority queue for schedulable processes, and we want to merge the workload in with another processor. One way to do this would be to insert every item in the first processor's queue into the receiving processor's queue. However, this takes  \\(O(n)\\) time - at least, depending on how the queues are implemented. We'd like to be able to do that more efficiently.

Step forward binomial heaps. Binomial heaps are rather different to binary heaps - although they share a few details in common. Binomial heaps allow us to merge two heaps together in  \\(O(\log n)\\) time, in return for some extra cost when finding the minimum. However, extracting the minimum still takes \\(O(\log n)\\) , which is the same as a binary heap.

<!--more-->

Binomial heaps are basically an ordered list of binomial trees, which are so-called because the number of nodes at depth  \\(i\\) follows the distribution of the binomial coefficient. Because of the way that the binomial trees are organised, there will never be more than  \\(O(\log n)\\) of them, all in heap order with the minimum element at the root. Therefore finding the minimum is a matter of looking at  \\(O(\log n)\\) tree roots. Merging two binomial heaps is a matter of merging two lists of trees - occasionally linking two of them together in constant time - which can also be done in  \\(O(\log n)\\) time.

## Binomial Heaps

On to the details. We want our data structure to support a few simple operations:

  * extractMin( H ) - removes and returns the smallest element in heap H
  * minimum( H ) - returns without removing the smalles element in heap H
  * merge( A, B ) - creates a new binomial heap from heaps A and B
  * makeHeap( ) - construct an empty heap
  * insert( H, v ) - insert value v into heap H
  * delete( H, v ) - delete value v from heap H

Standard binary heaps gives almost all of these very cheaply - makeHeap and minimum are both  \\(\Theta(n)\\) and almost all the other operations are  \\(\Theta(\log n)\\) in the worst case. The exception is merge, which requires a new heap to be built from scratch from the two heaps. This costs  \\(\Theta(n)\\) time which is a bit of a pain.

The aim of binomial heaps is to provide a data structure that supports cheap merging of heaps, while keeping the cost for all the other operations low. The basic idea is to implement each heap as a linked list of trees in order of size. We can merge two ordered linked lists together in  \\(\Theta(l)\\) time where  \\(l\\) is the length of the two lists. We can also use this operation to give us efficient insertion of a new value - we simply create a new binomial heap with one tree containing only the value, and merge that heap with our existing heap. Assuming constant time heap creation, this also costs \\(\Theta(l)\\) . This representation also gives us a simple way to find the minimum, if every tree is in heap order. The minimum will be the smallest root of all the trees, so finding it is a matter of scanning through  \\(l\\) trees. The only thing we haven't tackled is deletion, which is a bit complicated and so we'll leave until later.

There is, of course, a snag in our reasoning. All of our operations depend on the number of trees in the list, and we haven't given any guarantees on how big that number might get. This is where the properties of binomial trees become useful.

## Binomial Trees

Each tree in a binomial heap has the same basic structure. We can define this inductively. The first binomial tree,  \\(B_0\\) has one node. The  \\(n+1^{th}\\) binomial tree  \\(B_{n+1}\\) is formed from two  \\(B_n\\) trees by making the root of one the leftmost child of the root of the other. This gives the sequence seen in figure 1.

<!-- <img style='color:white;' src="http://hnr.dnsalias.net/images/binomials.png" alt="The first three binomial trees" /> -->

(Figure 1: the first three binomial trees \\(B_0, B_1, B_2\\)

It's easy to see that the number of nodes in  \\(B_{n+1}\\) is twice the number of nodes in \\(B_n\\) , and in general  \\(B_n\\) has  \\(2^n\\) nodes. We refer to  \\(n\\) as the _degree_ of a binomial tree.

It's also easy to see that the height of  \\(B_n\\) is simply \\(n+1\\) . Adding a new tree increases the height of the previous tree by one.

Note that each node can have arbitrary arity - the root of a degree  \\(n\\) binomial tree has  \\(n\\) children. So we can't use our pre-existing binary tree code for binomial trees

What about binomial trees makes them useful for our purpose? Simply that we can link two of them of equal degree together in constant time to make a binomial tree of degree one greater, by making one tree the leftmost child of the root of the other. Figure 1 shows how this works.  As a bonus, if the two binomial trees observe the heap property (that the root of every subtree is the smallest element in that tree) then the resulting  combined tree also observes the heap property, as long as we make sure that we make the tree with the larger root the child of the root of the tree with the smaller root, not the other way around.

As we will see, being able to link binomial trees together efficiently is fundemental to the rest of the data structure.

## Putting it together

Remeber we want to keep the number of trees down. To do this, we add an extra invariant to our list of binomial trees - that there is only ever at most one tree of a given degree. If there are more than one - say because we have just merged together two separate heaps - then we fix the problem by linking together both trees to make a tree with degree one larger. If this then creates a duplicate, then we keep linking again and again, until the invariant is satisfied.

So we are left with a list of trees. How many are there, as a function of the number of values in the heap \\(n\\) ? Recall that a binomial tree of degree  \\(k\\) has exactly  \\(2^k\\) nodes. There is only exactly one way of forming an integer  \\(n\\) from the sum of distinct powers of 2, and this sum contains  \\(\log n\\) powers of 2 - this is exactly the same as a binary representation of \\(n\\) .

Therefore, if we have  \\(n\\) nodes, we will have  \\(\Theta(\log n)\\) binomial trees in our heap. This means that the efficiency of merge, minimum and insert are all \\(\Theta(\log n)\\) . Note that minimum will always take  \\(\Theta(\log n)\\) time, as we always have to inspect all the tree roots. This means that it takes longer than a standard binary heap, where the minimum is always the root of the tree. This is one area where binomial heaps come out a bit worse than binary heaps.

We now need to attack the last set of operations that our heap needs to support. Both extractMin and delete require destructive update of the heap, so it seems likely that both will make use of the same technique. Both involve removing an element from a single tree, and then reconstructing one or more binomial trees from what remains.

Deleting the minimum is quite simple. Once we've found the tree with minimal root, we remove it from the linked list of trees. Each child of the root is the root of a binomial tree, with degree decreasing from left to right. So if we simply construct a new binomial heap with these trees in reverse order (i.e. increasing degree), we can merge that heap with our original heap in logarithmic time.

We can use a very similar approach for delete. We assume that we already have a pointer to the tree node containing the value to be deleted. We can then swap this value up to the root of the tree that contains it by pretending that it's smaller than any other element in the tree. So we swap it with its parent, one by one, until it is now the root, and then delete it like we did the minimum. A binomial tree of degree  \\(d\\) has depth \\(d+1\\) , and that therefore we need to do at most  \\(d\\) swaps to bring a value to the root. The largest degree binomial tree in a heap with  \\(n\\) nodes is \\(\log n\\) , so along with the merge step deletion takes \\(\Theta(\log n)\\) .

## Conclusions and Code

I performed a fairly unscientific shoot-out of a binary heap versus a simple binomial heap implementation. You can see the results below. Insertion performance was fairly similar, with the binary heap being a bit quicker (I was testing value-by-value insertion, not constructing a heap from a known array, where the binary heap is blazingly fast). Extraction performance was curiously poor for the binomial heap - ten times as bad as the binary heap. I expected the binary heap to win because all the manipulations are done in place in an array, but I'm going to investigate why the binomial heap was quite so bad.

Heap merging was as predicted: the binomial heap absolutely smoked the binary heap (I implemented merging by copying both heaps into one array, and then heapifying that) by some 4 orders of magnitude.

All tests were run on an Intel Core 2 Duo @ 2.2GHz with 2Gb of memory , and with 10000 insertions / deletions etc.

<pre>Binomial heap took 0.156891107559 seconds, 63738.4754023 ins/s
Binary heap took 0.176416873932 seconds, 56683.920178 ins/s
Binomial heap took 16.3578870296 seconds for extraction, 611.325899358 ext/s
Binary heap took 1.23731517792 seconds for extraction, 8082.01513929 ext/s
Binomial heap merged in 2.00271606445e-05 seconds
Binary heap merged in 0.144304037094 seconds</pre>

The code for the binomial heap is available [here](http://hnr.dnsalias.net/binomial_heap.py), written in Python. Let me know if you have any questions or comments!
