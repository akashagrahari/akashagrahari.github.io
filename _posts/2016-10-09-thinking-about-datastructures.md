---
layout: post
title: Thinking about Datastructures
sub-title: Probabilistic Datastructures and More
permalink: thinking-about-datastructures
---

With the growing demand to store more and more fault-proof data of systems for big companies, it is very important to come up with generic (and sometimes specific) ways and algorithms to store data in a meaningful way which can be later extracted for business and system analytics.

While working on such a problem, I came across many probabislistic data structures which excel in storing massive amounts of information. I will cover two of those here which are important for me to go deeper into the subject.

<!--break-->

## The Problem

As an example, imagine we want to count the number of distinct IP addresses visiting across a large website like Wikipedia. The traditional approach would be to resolve to a hash table which maps each such IP address with a count with a count. As of May, 2015, there were 430.54 million distinct Wikipedia users. Due to the scale, the hash table approach won't just work.

## The Probabilistic Way

When you really grasp the size of the above problem, you realize that you just can't ignore probabilistic models. These models give away some accuracy in numbers to offer a more capacity efficient and performant way to store such data. On scale like these a max error of say 0.5% to 2% doesn't really matter, because most of big-data applications are more about trends than hard facts. Maybe it's time to relook at the [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem). Although not strictly applicable to this problem; much like the CAP theorem, we must choose between consistency (accuracy) and availability (online). With CAP, we can typically adjust the level in which that trade-off occurs. This space/performance-accuracy exchange behaves very much the same way.

There are some popular models which I would list down but wont go into what they are. The names would be a good reference for you look them up and learn about them - Bloom Filters (Default/Scalable/Stable/Counting/Inverse), Min-Hash.


## HyperLogLog

This model is the bread and better for a lot of OLAP engines. Quick fact, not so long ago, redis released HyperLogLog as a new data-structure in the redis bundle.

The problem it solves is cardinality estimation. Let's say we have a stream which collects the name of the person who logged in to a website at any given time. And we want to find out the distinct first names we have as the user base (Surely, you can think of a better example). The conventional approach would be think of it as a [Commutative Monoid](https://en.wikipedia.org/wiki/Monoid#Commutative_monoid). If every event is a set containing the first name of a user, then the union of these events will give a set which has all the first names, appearing only once). The order in which this happens doesn't matter because of the properties of a commutative monoid.

> {Akash} U {Amrit} U {Indra} U {Akash} U ... = {Akash, Amrit, Indra}

Once you have the final set, all one needs to do is count the number of elements in the set to get the cardinality of the set. However, this approach has a tendency to explode, as the size of the resulting set in unbounded. Hence, this traditional solution is not practical. That begs the question - Is there a bounded way to calculate the unique elements in a stream. 

Well more or less. Not accurately, but with a certain degree of error.

Let's say you use an appropriate hash method to hash each value, say between 0 and 1. So every unique value will effectively lie on some random point on the number line joining 0 and 1. If your hash function is good enough, they will be well spread. 

> 0 <-------------^-------------------^------------------^--------------> 1

Let's say that we hash three values and they occupy the points given as *^*. The average distance, say *d* between any adjacent values is related to *N*, the number of unique values. In an ideal case, with values spreading out perfectly, *d = 1/N+1*. This value *d* is also the distance between 0 and the first value on the number line. Hence, just keeping the Min() of all the hash values can give us the cardinality N as

> N = 1/e - 1

If we got the following values for the above example.
 
> Akash = 0.763 ; Amrit = 0.452; Indra = 0.345

N would turn out to be roughly 2. Now that is obviously not the desired solution, since our hash function didn't behave in the *perfect, desired way*. To make up for this we employ multiple hash functions and produce values as - 

> {H1(Akash), H2(Akash), H3(Akash) . . . }<br>
{H1(Amrit), H2(Amrit), H3(Amrit) . . . }<br>
{H1(Indra), H2(Indra), H3(Indra) . . . }

Then we can calculate an element wide min of the matrices - 

> {Min(H1), Min(H2), Min(H3)} 

The average of the above values will give a better approximation of the value *d* and hence the cardinality *N*.

This is more or less how HyperLogLog works. And for most applications, a hundred to thousand of such hash functions would yield a number which has a non growing error of less than 2%, which is pretty awesome.

Point to be noted here is that the key is lost in this data-structure. Every query has to come via a key and we cannot as questions like *"Who appeared the most number of times?"*


## Count Min-Sketch

So let's think about how can we solve the above question; and more. Let's solve the frequenct question - *how many times did an event occur?* The traditional way to go about would be treat every event as a map, e.g. - 

> {Akash: 1}, {Amrit: 1}, {Indra: 1}, ...

And we keep aggregating each event in the final giant map which increments the number which will correspond to the count. Again, this will blow up as the size of the data increases.

Let's say we use another hash function to hash each event value to give a K (here 8) bit matrix. So,

> Akash *becomes* {0,0,0,0,1,0,0,0}<br>
Amrit *becomes* {0,0,0,1,0,0,0,0}<br>
Indra *becomes* {0,0,0,0,1,0,0,0}

And get the resulting matrix by doing an element sum across these.

So let's say we got the following events - 

> Akash, Amrit, Akash, Indra

The resulting matrix would be - 
> {0,0,0,1,3,0,0,0}

Now when we are trying to ascertain how many times a name occured, we just hash it and find the counts at the position where the result of the hash gives a one. In *Akash's* case, it was the 5th bit and the value we get is 3.

This is obviously not right, because there were only 2 events of Akash. The error turns out to be higher while finding the frequency for *Indra*. But we can minimize the effect of this collision among the events by using a sufficiently large number of hash functions, and the estimated frequency would be the one which is the lowest across all the hashes. 

![Count Min Sketch ](/public/count_min_sketch.png){:class="img-responsive"}

There is still a possibility that there was collision and hence this number acts as an upper bound to the frequency.

## Going further

Let's say we are going to solve an audience segmentation problem. For a business, we can divide the audience on the basis of gender, spend-capacity, location, etc.

For logical groups like gender which can have a maximum of three possible values, it's pretty easy to maintain 3 separate HyperLogLog (HLL) registers and pass each event through these. At any point, the count resulting from the register gives the cardinality.

However, for groups like location which can have potentially thousands and lakhs of distinct values, it seems rather unwise to manually create, name and handle individual HLL registers. This would a lot clearer if talked about in the context of redis. The following commands would create a new HLL register in the redis cluster and add four user ids corresponding to events.

> PFADD male "USER1"<br>
PFADD male "USER2"<br>
PFADD male "USER1"<br>
PFADD male "USER3"

If now we try to find out the unique values, we do

> PFCOUNT male

which is 3

It's pretty obvious that it doesn't make a lot of sense to individually and manually maintain registers for each and every possible value in a group, especially, when all of them are not even active.

While trying to solve this problem some time back, I came up with the following solution. A count-min sketch (named the logical group, "location" in this instance), which has HLL registers instead of counters in the matrices named by the hash values generated by hash function of the count-min sketch.

Replacing pure counters with HLLs means that you can get unique count as opposed to the total count. And creating HLL registers only when an event occurs corresponding to a value saves disk space.

## To End It All

Probablistic data-structures are an exciting field and they change the way we traditionally approach data-structures and alogrithms. It pushes you not just to think in terms of final results, but also long term feasibility.

### Resources

* An excellent talk by Avi Bryant about Abstract Algebra in Analytics - [Add All the Things](https://www.infoq.com/presentations/abstract-algebra-analytics)
* [The original HyperLogLog paper](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)
* A brilliant JS visualization of how HLL works - [link](https://research.neustar.biz/2012/10/25/sketch-of-the-day-hyperloglog-cornerstone-of-a-big-data-infrastructure/)

{% include twitter_plug.html %}
