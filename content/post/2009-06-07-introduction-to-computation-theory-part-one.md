---
id: 221
title: 'Introduction to Computation Theory &#8211; Part One'
date: 2009-06-07T16:16:17+00:00
author: Henry
layout: post
guid: http://hnr.dnsalias.net/wordpress/?p=221
permalink: /introduction-to-computation-theory-part-one/
categories:
  - Computation Theory
  - computer science
---
(This article represents a brief divergence from our usual focus on distributed systems and algorithms. Normal service will be resumed shortly.)

One of the courses I do some teaching for is the dryly named &#8216;Theory of Computation&#8217;. Although such a course name is unlikely to pique a lot of interest, in actual fact it is probably the most fundamental and important course that you could take about computer science. Computation theory allows us to ask, and answer, questions about the limits of computers and the programs they execute. Is every problem solvable by a computer program? What do we even mean by a &#8216;problem&#8217;?

It turns out that there are profound and beautiful truths to be discovered when considering these questions. In a short series of articles, I&#8217;m going to give a brief introduction to the theory of computability, aimed at those who have never studied Turing machines, Cantor diagonalisation or Godel incompleteness. An appreciation for some of these ideas and results should give a perspective on what computers are, and are not, capable of.

This first article presents a bit of background material on the nature of infinity, which is right at the heart of computation theory. In the next article I&#8217;ll tie this in with the idea of computability and show how these ideas elegantly allow us to establish limits on what we can do with computers &#8211; even in theory, let alone practice.
  
<!--more-->

## Different Types of Infinity

A very important and fundamental question to computation theory is this: &#8220;how many unique computer programs are there?&#8221;

Although we&#8217;re not in a position to be precise about the details of such a question, we can at least give some thought to the kind of answers it might have. Are there just a finite number of computations &#8211; different executions with different results that a theoretical computer could perform?

It&#8217;s clear that this is unlikely. For example, we can conceive of a program that prints out a unique integer, and we can think of infinitely many programs that each print out a different integer. So it seems that there should be infinitely many programs, or computations. But let&#8217;s not be too hasty throwing that word &#8216;infinite&#8217; around. What do we understand by infinity?

Let&#8217;s get one thing straight right away. We can&#8217;t say that &#8216;infinity&#8217; is a quantity, or a number. We can&#8217;t add one to infinity. Infinity minus infinity is _meaningless_, not zero. Rather, infinity is a _predicate_ on sets of things. If I describe to you a set, if we say that it does not contain a finite number of things then we say that the set has the _property_ of being infinite. An infinite set is one that doesn&#8217;t have a number that we can call its size.

Bear with me a moment while I get pedantic. What does &#8216;size&#8217; mean? What&#8217;s the size of a set? Obviously, the size of a set is the number of items in it. But how do we arrive at that number? We count the items, one by one. So what we do is give every item in the set a label in order &#8216;1&#8217;, &#8216;2&#8217;, &#8216;3&#8217;&#8230; and so on until all the elements in the set have a label. The value of the last label that we assigned is what we call the size, or _cardinality_ of a set. It&#8217;s the number of labels we need to uniquely label every element in the set. If a set is infinite, there is no finite number of labels that we can use to uniquely label every element. No matter what number of labels we use, there will always be elements (in fact, infinitely many of them) left over at the end that didn&#8217;t get a label. _This_ is what infinite means. If we take a finite number of things out of an infinite set the set will never be empty, no matter how many we take out.

An example of an infinite set is the set of all words that are some number of As followed by the same number of Bs. For example, AB, AABB, AAABBB are all in the set. There are infinitely many words like this. How do we prove that a set is infinite? One way to do it is to take a set that we already know is infinite and show that for every element in our known set there is exactly one element in our candidate infinite set. This is similar to our labelling technique for finite sets &#8211; we take an infinite set and show that we can use it to label our candidate set. Since there are infinitely many labels from our infinite set, and we have shown that we need all of them to label our candidate set, we can conclude that our candidate set must itself be infinite, otherwise we would only need a finite number of labels to label it. By coming up with a labelling that uses every element of the infinite set, no matter what it is, we show that no finite labelling can exist.

So how do we apply this technique to our A&#8230;B&#8230; set? Well, we take a known infinite set, in our case the natural numbers  <img src='http://s0.wp.com/latex.php?latex=N%3D%7B1%2C2%2C3%2C%26%238230%3B%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='N={1,2,3,&#8230;}' title='N={1,2,3,&#8230;}' class='latex' />and find a labelling. Here the labelling is easy to find. Let&#8217;s write every element of  <img src='http://s0.wp.com/latex.php?latex=A&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A' title='A' class='latex' />as  <img src='http://s0.wp.com/latex.php?latex=A%5EnB%5En&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A^nB^n' title='A^nB^n' class='latex' />where  <img src='http://s0.wp.com/latex.php?latex=A%5En&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A^n' title='A^n' class='latex' />means &#8216;repeat  <img src='http://s0.wp.com/latex.php?latex=A&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A' title='A' class='latex' /> <img src='http://s0.wp.com/latex.php?latex=n&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='n' title='n' class='latex' />times&#8217;. Then there is exactly one word in  <img src='http://s0.wp.com/latex.php?latex=A&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A' title='A' class='latex' />for every positive integer  <img src='http://s0.wp.com/latex.php?latex=n&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='n' title='n' class='latex' />(negative values of  <img src='http://s0.wp.com/latex.php?latex=n&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='n' title='n' class='latex' />don&#8217;t make a lot of sense). This makes it very easy to come up with a labelling from  <img src='http://s0.wp.com/latex.php?latex=N&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='N' title='N' class='latex' />to <img src='http://s0.wp.com/latex.php?latex=A&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A' title='A' class='latex' />. A word  <img src='http://s0.wp.com/latex.php?latex=A%5EnB%5En&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A^nB^n' title='A^nB^n' class='latex' />in  <img src='http://s0.wp.com/latex.php?latex=A&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A' title='A' class='latex' />has a label  <img src='http://s0.wp.com/latex.php?latex=n&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='n' title='n' class='latex' />in <img src='http://s0.wp.com/latex.php?latex=N&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='N' title='N' class='latex' />. For every natural number, there is exactly one word. Therefore  <img src='http://s0.wp.com/latex.php?latex=A&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A' title='A' class='latex' />must be infinite!

This technique is the most useful when it comes to reasoning about the cardinality of various sets, and is hugely important when talking about computability. Computer scientists will call the act of finding a labelling _establishing a bijection between  <img src='http://s0.wp.com/latex.php?latex=A&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='A' title='A' class='latex' />and <img src='http://s0.wp.com/latex.php?latex=N&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='N' title='N' class='latex' />_. A bijection is simply a precise word for a relationship that pairs elements of two sets together such that every element of each set has a unique partner in the other.

Now, we have taken as given that the set of integers is infinite, and no-one would argue with us. Is that the end of the story? Do all infinite sets have a bijection with the integers?

It turns out that the answer is no, not all infinite sets can be placed in correspondence with the integers. This result is extremely famous, and due to [Georg Cantor](http://en.wikipedia.org/wiki/Georg_Cantor), a man who spent much of his mathematical career looking into infinity and drove himself mad as a consequence.

## &#8216;Smaller&#8217;, But Still The Same Size

Let&#8217;s consider two different sets and challenge our intuitions about their size as compared to the integers. Firstly, consider the even integers. Is the set of even integers infinite? We would think so, because no matter what finite collection of them we can put together, we can always think of one that we&#8217;ve left out. This is a giveaway property of an infinite set. But at the same time, it seems like there should be fewer even integers than all the integers combined &#8211; intuitively we would expect there to be twice as many odd and even integers as just the even ones. But &#8216;twice as many&#8217; is a murky concept when dealing with infinity, which as we discussed earlier doesn&#8217;t yield to normal arithmetic.

Indeed, an attempt to find a bijection between even integers and the whole set of integers turns up a correspondence pretty quickly. Recall that we can write every even integer in the form  <img src='http://s0.wp.com/latex.php?latex=2x&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='2x' title='2x' class='latex' />for all <img src='http://s0.wp.com/latex.php?latex=x%5Cin+%5Cmathbb%7BN%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='x\in \mathbb{N}' title='x\in \mathbb{N}' class='latex' />. This trivially gives us our bijection &#8211; label every even integer with the integer you get by dividing it by two. Conversely, every integer has a corresponding even integer that is the result of multiplying the integer by two. So 1 maps to 2, 7 maps to 14 and 316 maps to 632.

So it turns out that, completely counter-intuitively, there are as many integers as there are _both even and odd integers_! Infinity does not always behave as we expect. In particular it&#8217;s not sufficient to show that one set is a proper subset of another in order to demonstrate their different sizes when both sets are infinite.

## Bigger Than Infinite?

I said earlier that there _are_ infinite sets that are not of the same size as the integers. We&#8217;ve tried to find a set that was smaller, but had no luck. Let&#8217;s consider an apparently larger set and see where we get.

The _real numbers_ are a good candidate. The real numbers are fairly complex to formally define, but we know that they include all the integers, as well as all the numbers with decimal expansions between the integers. Indeed, some real numbers have expansions that are infinitely long &#8211; a fact that will prove very important when we talk about the limits of Turing machines.

Is there a bijection between real numbers and integers? This is a tricky question, and required a genuine bit of ingenuity to resolve. The problem is that when a bijection is not obviously forthcoming (it&#8217;s hard to write each real number in terms of a single integer) we have to look in the other direction and prove that there is no such bijection. This is a good deal harder &#8211; a positive proof just requires us to come up with a single exemplar, whereas a negative proof has to be valid for every single possible bijection!

Cantor&#8217;s attack on the negative proof went as follows. Imagine there is such a bijection. Then we could conceive of a list with all the real numbers in order, as labelled by their integer counterparts. So at the top would be real number 1, followed by real number 2 and so on. Obviously, the list would be infinite so we couldn&#8217;t write it down. That&#8217;s no problem, we just have to imagine it.

Note at this point how the list doesn&#8217;t depend at all on the nature of the bijection itself &#8211; no matter what the bijection is, we can construct this list. This generality is crucial for easily dismissing the entire set of bijections at once.

The next step is the neat one. If this list was a bijection, it would contain every single real number. So if we could come up with some real number that wasn&#8217;t in the list, we would show that it was not a bijection. Cantor&#8217;s great insight was to see that there is a way to construct such a real number by showing that it could not be equal to any real number in the list.

Cantor imagined the list of real numbers as a table, with the rows being real numbers, and the columns being the individual digits of each real number. So the 3rd column would contain the 3rd digits of every real number. Therefore we can identify the  <img src='http://s0.wp.com/latex.php?latex=j%5E%7Bth%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='j^{th}' title='j^{th}' class='latex' />digit from the  <img src='http://s0.wp.com/latex.php?latex=i%5E%7Bth%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='i^{th}' title='i^{th}' class='latex' />real number by talking about the digit <img src='http://s0.wp.com/latex.php?latex=d_%7Bij%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='d_{ij}' title='d_{ij}' class='latex' />.

To keep the presentation straightforward, we&#8217;re going to consider only the real numbers between 0 and 1 &#8211; if there are more of even these kinds of number than real numbers, we&#8217;ll still have established the result.<table cellpadding=5> 

</table> 

This table is the first few rows of an example bijection.

Now the key observation here is that, for a real number  <img src='http://s0.wp.com/latex.php?latex=r_p&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='r_p' title='r_p' class='latex' />to be different from  <img src='http://s0.wp.com/latex.php?latex=r_q&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='r_q' title='r_q' class='latex' />it is enough for  <img src='http://s0.wp.com/latex.php?latex=r_p&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='r_p' title='r_p' class='latex' />to differ from  <img src='http://s0.wp.com/latex.php?latex=r_q&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='r_q' title='r_q' class='latex' />at just one digit. That is, as long as there exists a  <img src='http://s0.wp.com/latex.php?latex=k&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='k' title='k' class='latex' />for which  <img src='http://s0.wp.com/latex.php?latex=d_%7Bpk%7D+%5Cneq+d_%7Bqk%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='d_{pk} \neq d_{qk}' title='d_{pk} \neq d_{qk}' class='latex' />then <img src='http://s0.wp.com/latex.php?latex=r_p+%5Cneq+r_q&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='r_p \neq r_q' title='r_p \neq r_q' class='latex' />.

So to find a real number that is not in the list, we just have to find a way to construct one that is different from every real number in the list at at least one digit.

Obviously it&#8217;s no good just changing a single digit &#8211; say we set the 3rd digit of our real number to 7 &#8211; because it&#8217;s guaranteed that there are some real numbers whose 3rd digit is 7 and we don&#8217;t know whether or not the rest of our real number matches them.

However, just as there are an infinite number of real numbers in our list, there are infinitely many columns in our table &#8211; so infinitely many digits to play with. So if we just make sure that each digit is different from its counterpart in one real number each, that will be sufficient because there are enough digits to go around. We will construct a real number  <img src='http://s0.wp.com/latex.php?latex=C%3Dc_1c_2c_3%5Chdots&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='C=c_1c_2c_3\hdots' title='C=c_1c_2c_3\hdots' class='latex' />so that each digit  <img src='http://s0.wp.com/latex.php?latex=c_k&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='c_k' title='c_k' class='latex' />of  <img src='http://s0.wp.com/latex.php?latex=C&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='C' title='C' class='latex' />is different from digit  <img src='http://s0.wp.com/latex.php?latex=d_%7Bik%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='d_{ik}' title='d_{ik}' class='latex' />of the  <img src='http://s0.wp.com/latex.php?latex=i%5E%7Bth%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='i^{th}' title='i^{th}' class='latex' />real number in the list, and that  <img src='http://s0.wp.com/latex.php?latex=i&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='i' title='i' class='latex' />is different for all <img src='http://s0.wp.com/latex.php?latex=c_k&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='c_k' title='c_k' class='latex' />.

There are two challenges here. The first is to come up with a method that lets us always pick a digit that is different. The second is to come up with a mapping &#8211; in fact, another bijection &#8211; between the columns and the rows so that every digit in our constructed real number is responsible for being different from exactly one real number in the table.

Both challenges are easily resolved. If the digit we wish to be different from is <img src='http://s0.wp.com/latex.php?latex=x&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='x' title='x' class='latex' />, then it&#8217;s simple to construct  <img src='http://s0.wp.com/latex.php?latex=y+%5Cneq+x&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='y \neq x' title='y \neq x' class='latex' />by adding one to  <img src='http://s0.wp.com/latex.php?latex=x&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='x' title='x' class='latex' />(and if  <img src='http://s0.wp.com/latex.php?latex=x&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='x' title='x' class='latex' />is 9, wrapping  <img src='http://s0.wp.com/latex.php?latex=y&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='y' title='y' class='latex' />around to 0).

We can also neatly deal with the second challenge. In our constructed real number <img src='http://s0.wp.com/latex.php?latex=C&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='C' title='C' class='latex' />, we&#8217;ll give digit  <img src='http://s0.wp.com/latex.php?latex=c_1&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='c_1' title='c_1' class='latex' />the responsibility of being different from the 1st real number in the list, at digit <img src='http://s0.wp.com/latex.php?latex=d_%7B11%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='d_{11}' title='d_{11}' class='latex' />. Similarly for <img src='http://s0.wp.com/latex.php?latex=c_2+%5Cneq+d_%7B22%7D&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='c_2 \neq d_{22}' title='c_2 \neq d_{22}' class='latex' />. And so on. Every digit of  <img src='http://s0.wp.com/latex.php?latex=C&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='C' title='C' class='latex' />causes it to be different from a unique real number in the list. By combining all these digits together, we get a new, infinitely long real number that is different from _every single listed real number_.<table cellpadding=5> 

<td align='center' bgcolor='lightgreen'>
  7
</td>

<td align='center' bgcolor='lightgreen'>
  3
</td>

<td align='center' bgcolor='lightgreen'>
  9
</td>

</table> 

In the case of our example bijection, we would have the beginnings of  <img src='http://s0.wp.com/latex.php?latex=C&#038;bg=ffffff&#038;fg=000000&#038;s=0' alt='C' title='C' class='latex' />as 0.840&#8230; &#8211; different from 0.7394&#8230; at digit 1, different from 0.9328&#8230; at digit 2 and different from 0.4298&#8230; at digit 3. Different from every real number in the table!

This concludes the proof. What we have shown is that there is no bijection between the real numbers and the integers. No matter what the bijection, we can come up with some real number that&#8217;s not been included. Therefore we can conclude that there _are_ more real numbers than integers.

The shape that the &#8216;differing digits&#8217; make in the table of real numbers moves along the diagonal. This is why this is called &#8216;Cantor&#8217;s diagonal proof&#8217;. In actual fact, the diagonal pattern is just one of infinitely many bijections we could have come up with. It is the simplest, and the easiest to understand.

## Two Kinds of Infinity

So this leads us to the strange idea that there are two different kinds of infinity. The first kind of infinite sets, like the integers, are called _countably infinite_ to emphasise the idea that they can be placed in one-to-one correspondence with the integers, an action that we cal &#8216;counting&#8217;. The second kind of infinite sets that we have discovered, like the real numbers, are called _uncountably infinite_ by way of contrast. Uncountably infinite sets are unimaginably more vast than countable ones. Between any two real numbers, there are uncountably infinite intermediate real numbers &#8211; more than there are integers. There are more real numbers between 0 and 1 than there are integers.

This contrast will play a significant role in the development of a theory of computation. We will be able to identify the number of possible computations with a countably infinite set, and therefore describe, using our established language of infinity, computations that our best efforts cannot perform, not even with all the resources in the world.