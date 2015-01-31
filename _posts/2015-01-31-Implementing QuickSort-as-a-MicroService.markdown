---
layout: post
title:  "Implementing Quicksort-as-a-(Micro)Service"
date:   2015-01-31 15:00:00
categories: jolie microservices
---
*Scenario*: for the release of the new [Jolie website](http://jolie-lang.org) we decided to add external blog posts in a dedicated "[blogs](http://www.jolie-lang.org/planet)" section. After ~230 lines of Jolie code we had our own news retrieval  service. The service reads the RSS feed from our collection of blogs and serves them as a single flow of news. 

*Problem*: we had different sources that returned a stream of articles ordered by date. We collected them into a single stream but still they were not ordered properly.

*Solution*: one word (well actually a compound, but still) **[QuickSort](http://en.wikipedia.org/wiki/Quicksort)-as-a-(micro)service**

## QuickSort ...

Best thing I love about QuickSort is its elegant recursive implementation.

As Wikipedia puts it (slightly modified):

<pre>
<code class="language-javascript">function quicksort(A)
	less, equal, greater := three empty arrays
	if length(A) > 1  
  	pivot := select any element of A
  	for each x in A
      if x <= pivot then add x to less
      if x > pivot then add x to greater
  	quicksort(less)
  	quicksort(greater)
  A := concatenate(less, pivot, greater)</code>
</pre>

In English, you want to order `A`, which is an array of sortable (comparable) elements. You take a pivot element (e.g., from the middle of the array) and cycle through the whole array to divide it into two halves, `less` and `greater`. `less` will contain all elements lesser than or equal to the pivot and `greater` the remaining greater-than-pivot elements. Done that you can apply quicksort to `less` and `greater` which will order them. Finally you assemble `less`, the `pivot`, and `greater` (in this order) and return the result.

## ... as a (Micro)Service

Let us take that pseudo-code implementation of quicksort and translate it into a microservice. (Btw, if you are into THAT kind of microservice-porn, [here](http://docs.jolie-lang.org/#!documentation/locations/local.html) you can find a recursive algorithm of the [Tower of Hanoi](http://en.wikipedia.org/wiki/Tower_of_Hanoi)).

In the pseudo-code we define a function called `quicksort`. Being a little more detailed we can add some typing to it. We want quicksort to take an array and return it sorted, hence we have something like `function quicksort( Array[ T ] : A ) : Array[ T ]`. `T` being the type of the elements we want to sort.

In Jolie we can model function calls as [request-response operations](http://docs.jolie-lang.org/#!documentation/basics/communication_ports.html#using-communications-ports). Hence we can define the interface of our `quicksort` service as

<pre>
<code class="language-jolie">interface QuickSortInterface {
	RequestResponse: quicksort( QSType )( QSType )
}</code></pre>

Jolie `interface`s describe which operations a service can implement and the type of their input and output. The [type](http://docs.jolie-lang.org/#!documentation/basics/communication_ports.html#data-types) of `QSType` can be something like

<pre>
<code class="language-jolie">type QSType: void {
	.e* : T
}</code></pre>

Remember that every variable in Jolie is a tree [^1], therefore our type will have a root, say `QSType`, which is empty (`void`) and then we allow it to have 0 to n leaves e[0], ..., e[n-1] of type `T`.

[^1]: <small>I know, at the beginning it might feel awkward but native tree-structured variables come really handy when dealing with JSON/XML-like data. Good topic for another post though.<small>

As said, let us begin with the request-response operation

<pre>
<code class="language-jolie">quicksort( req )( res ){ ... }</code>
</pre>

As you might guess, `req` and `res` have both type `QSType`. The body of the operation will contain the behaviour of the quicksort.

<div style="overflow:auto;"><div style="width:49%;float:left">
<pre>
<code class="language-jolie">function quicksort( A )
 if length(A) > 1
  pivot := select any element of A
  for each x in A
   if x <= pivot then add x to less
   if x > pivot then add x to greater
  quicksort(less)
  quicksort(greater)
 A := concatenate(less, pivot, greater)
</code></pre></div><div style="width:49%;float:right">
<pre>
<code class="language-jolie">quicksort( req )( res ){
 if( #req.e > 1 ){
  // 1. partition the request
  pivot = #req.e / 2;
  for ( i = 0, i < #req.e, i++ ) {
   if( i != pivot ) {
    if( req.e[ i ] <= req.e[ pivot ] ) {
     less.e[ #less.e ] << req.e[ i ]
    } else {
     greater.e[ #greater.e ] << req.e[ i ]
    }
   }
  };
  // 2. order the two partitions
  // 3. merge the result
 } else {
  res << req
 }
}</code>
</pre></div></div>

First we check if the array in the request has less than 2 elements. Jolie provides the `#` operator that returns the number of leaves in the branch at its right. Its behaviour is not different from the usual `req.e.length()` of C/Java.
In case there are less than 2 elements (else branch), we return as response exactly the request. `res << req` (`<<` being the [deep-copy operator](http://docs.jolie-lang.org/#!documentation/basics/data_structures.html#copying-an-entire-tree-structure)) means "copy the whole structure and values of variable `req` in `res`".

If there are more elements we can sort them. Although there are plenty of optimisations [[1]](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.39.1103)[[2]](http://comjnl.oxfordjournals.org/content/27/3/276.full.pdf+html)[[3]](http://www.cs.dartmouth.edu/~doug/mdmspe.pdf)[[4]](http://cs.fit.edu/~pkc/classes/writing/samples/bentley93engineering.pdf), we will stick to the good ol' "take the middle element as pivot". We use `#` to address the *n-th+1* leaf and store the whole structure of the `i-th` element of `req.e`.

---

*Beware: pedantic observation!*. We are using `<<` to copy the values of the leaves. Why didn't we use the expression `less.e[ #less.e ] = req.e[ i ]`? Since every variable in Jolie is a tree, `req.e[ i ]` is actually a shorthand for `req.e[ i ][ 0 ]`, i.e., the first leaf of tree `req.e[ i ]`. If we assume to have a one-element data structure, using `=` to copy it would yield the same result as `<<`. On the contrary, if we want to bring along a sub-structure rooted in `req.e[ i ]` (e.g., blog posts) we need to deep-copy it.

---

Let us get to the recursive calls. We need to create two [ports](http://docs.jolie-lang.org/#!documentation/basics/communication_ports.html#ports), one inputPort and one outputPort.

<pre>
<code class="language-jolie">outputPort SelfOut {
	Interfaces: QuicksortInterface
}

inputPort SelfIn {
	Location: "local"
	Interfaces: QuicksortInterface
}</code></pre>

We use the [`"local"`](http://docs.jolie-lang.org/#!documentation/locations/local.html) location, which is a lightweight medium that exploits in-memory messages between services running in the same interpreter. In this case we use it to let the QuickSort Service call itself.

Almost finished. Let us invoke `quicksort` on the two partitions created earlier.

<pre>
<code class="language-jolie">// 2. order the two partitions
quicksort@SelfOut( less )( res );
quicksort@SelfOut( greater )( greater )</code>
</pre>

Note that we store the sorting of `less` in `res`. This comes handy when we will merge the result together

<pre><code class="language-jolie">// merge the result
res.e[ #res.e ] << req.e[ pivot ];
for ( i=0, i<#greater.e, i++) {
	res.e[ #res.e ] << greater.e[ i ]
}</code></pre>

We append the pivot and the elements of `greater` to `res,` which concludes the behaviour of quicksort. 

Just a couple of things and we are ready to run our service. First, we set to `concurrent` the [execution mode](http://docs.jolie-lang.org/#!documentation/basics/processes.html) of our service. In `concurrent` mode Jolie creates a new instance of the service at any new request.

<pre><code class="language-jolie">execution{ concurrent }</code></pre>

and second we add the `init` procedure which, like the `main`, is a special procedure. If present `init` is executed before the `main`. To enable quicksort call itself we need to set the `local` location of outputPort SelfOut to the value returned by the runtime service

<pre><code class="language-jolie">init { getLocalLocation@Runtime()( SelfOut.location ) }</code></pre>

[Runtime](http://docs.jolie-lang.org/#!documentation/jsl/Runtime.html) is a service of the Jolie Standard Library, imported with the line `include "runtime.iol"`


## To wrap up

Let us pull together the parts we wrote above into a single Jolie service (for simplicity I set the type of the content of the array to `int`). Note that we enclosed operation `quicksort` into the `[main](http://docs.jolie-lang.org/#!documentation/basics/define.html)` procedure.

<pre>
<code class="language-jolie">include "runtime.iol"

execution{ concurrent }

type QSType: void{
	.e*: int
}

interface QuicksortInterface {
  RequestResponse: quicksort( QSType )( QSType )
}

inputPort SelfIn {
  Location: "local"
  Interfaces: QuicksortInterface
}

inputPort SelfHttp {
  Location: "socket://localhost:8000"
  Protocol: http
  Interfaces: QuicksortInterface
}

outputPort SelfOut{
  Location: "local"
  Interfaces: QuicksortInterface
}

init { getLocalLocation@Runtime()( SelfOut.location ) }

main
{
  quicksort( req )( res ){
  if( #req.e > 1 ){
   // 1. partition the request
   pivot = #req.e / 2;
   for ( i = 0, i < #req.e, i++ ) {
    if( i != pivot ) {
     if( req.e[ i ] <= req.e[ pivot ] ) {
      less.e[ #less.e ] << req.e[ i ]
     } else {
      greater.e[ #greater.e ] << req.e[ i ]
     }
    }
   };
  // 2. order the two partitions
  quicksort@SelfOut( less )( res );
  quicksort@SelfOut( greater )( greater );
  // merge the result
  res.e[ #res.e ] << req.e[ pivot ];
  for ( i=0, i<#greater.e, i++) {
    res.e[ #res.e ] << greater.e[ i ]
    }
  } else {
   res << req
   }
  }
}</code></pre>

Did you noticed I also added a new port to the service?

<pre>
<code class="language-jolie">inputPort SelfHttp {
  Location: "socket://localhost:8000"
  Protocol: http
  Interfaces: QuicksortInterface
}</code></pre>

What it does is to enable the service to accept a request from any browser.
How cool is that?! We developed an application for in-memory requests and with just 4 lines of declarative code we have it accepting calls from HTTP! Fire up your terminal and try this:

<pre>curl -H "content-type: text/xml" -d "<quicksort><e>5</e><e>21</e><e>13</e><e>34</e><e>1</e><e>1</e><e>2</e><e>3</e></quicksort>" http://localhost:8000/quicksort</pre>

Still here? Get yourself a [Jolie](http://www.jolie-lang.org/downloads.html) and start tinkering with microservices :)

## I want more

You want more? Really?
Ok, one last candy. As you may be aware of (or maybe not) the `;` in Jolie does not mark the end of a statement. It is the *sequential* composition operator by which `A ; B` means after the execution of A executes B. Jolie also provides the *parallel composition* operator which executes two statements concurrently.

Want to optimise for multi-core computation?

<pre><code class="language-jolie">{ quicksort@SelfOut( less )( res ) | quicksort@SelfOut( greater )( greater ) }</code></pre>

you can send two parallel requests to quicksort. Note that we enclosed the two requests into a [scope](http://docs.jolie-lang.org/#!documentation/fault_handling/basics.html). In this way the instructions after the scope will be performed only when both requests return their results.

Want more? Take a cloud, deploy several copies of quicksort and call them 

<pre>
<code class="language-jolie">{ quicksort@Instance1( less )( res ) | quicksort@Instance2( greater )( greater ) }</code></pre>

And this is only the tip of the iceberg as you can [aggregate](http://docs.jolie-lang.org/#!documentation/architectural_composition/aggregation.html) a cloud of services into a single point of workflow distribution, as well as [proxies](http://docs.jolie-lang.org/#!documentation/architectural_composition/redirection.html), load-balancers, and many other architectural patterns. 

Always without leaving the Jolie language.

Pardon me if I ask it again. What are you waiting for getting yourself a [Jolie](http://www.jolie-lang.org/downloads.html)? 

Happy microservices :)