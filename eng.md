What are transducers? Using transducers is easy enough—but how do they work
underneath the hood?

This article explores transducers by ignoring transducers. Instead we will
examine two ordinary functions,`map` and `filter`. We’ll play with them and
scrutinize them. And we’ll marvel at the power of higher-order functions as we 
apply abstractions. And perhaps, if we’re lucky, we’ll bump into transducers 
along the way.

And since we ignore transducers, you won’t need to know what transducers are
to follow along. If you don’t know Clojure or a Lisp,[this quick primer][1] may
help.

Lastly, I encourage you to type these examples into your REPL, or use 
[clojurescript.net][2]. The source code from this post can be found [here][3]

## Power of reduce {#power-of-reduce}

You are probably familiar with `map` and `filter`, and know that we can combine
them together, like this:

    (<span class="kw">map</span> <span class="kw">inc</span> (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ (1 2 3 4 5 6 7 8 9 10)</span>
    
    (<span class="kw">filter</span> <span class="kw">even?</span> '(<span class="dv">1</span> <span class="dv">2</span> <span class="dv">3</span> <span class="dv">4</span> <span class="dv">5</span> <span class="dv">6</span> <span class="dv">7</span> <span class="dv">8</span> <span class="dv">9</span> <span class="dv">10</span>))
    <span class="co">; ⇒ (2 4 6 8 10)</span>
    
    (<span class="kw">filter</span> <span class="kw">even?</span> (<span class="kw">map</span> <span class="kw">inc</span> (<span class="kw">range</span> <span class="dv">10</span>)))
    <span class="co">; ⇒ (2 4 6 8 10)</span>

A key insight, however, is that `map` and `filter` can be defined using 
`reduce`. Let’s implement the expression `(map inc (range 10))` in terms of 
`reduce`:

    (<span class="kw">defn</span><span class="fu"> map-inc-reducer</span>
      [result input]
      (<span class="kw">conj</span> result (<span class="kw">inc</span> input)))
    
    (<span class="kw">reduce</span> map-inc-reducer [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [1 2 3 4 5 6 7 8 9 10]</span>

But note `map-inc-reducer`’s explicit use of `inc` as its transformer. What if
we extract that out and let the user pass in whatever function they want? We can
define a new function that takes a transforming function like`inc` and returns
a new function.

    (<span class="kw">defn</span><span class="fu"> map-reducer</span>
      [f]
      (<span class="kw">fn</span> [result input]
        (<span class="kw">conj</span> result (f input))))
    
    (<span class="kw">reduce</span> (map-reducer <span class="kw">inc</span>) [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [1 2 3 4 5 6 7 8 9 10]</span>

Functions like `map-reducer` are called a higher-order functions because they
accept functions and return functions. Let’s play around:

    (<span class="kw">reduce</span> (map-reducer <span class="kw">dec</span>) [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [-1 0 1 2 3 4 5 6 7 8]</span>
    
    (<span class="kw">reduce</span> (map-reducer #(<span class="kw">*</span> % %)) [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [0 1 4 9 16 25 36 49 64 81]</span>

Let’s also implement the expression `(filter even? '(1 2 3 4 5 6 7 8 9 10))` in
terms of`reduce`:

    (<span class="kw">defn</span><span class="fu"> filter-even-reducer</span>
      [result input]
      (<span class="kw">if</span> (<span class="kw">even?</span> input)
        (<span class="kw">conj</span> result input)
        result))
    
    (<span class="kw">reduce</span> filter-even-reducer [] '(<span class="dv">1</span> <span class="dv">2</span> <span class="dv">3</span> <span class="dv">4</span> <span class="dv">5</span> <span class="dv">6</span> <span class="dv">7</span> <span class="dv">8</span> <span class="dv">9</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [2 4 6 8 10]</span>

Again, notice that `filter-even-reducer` explicitly uses `even?` as its
predicate. As before, let’s extract that out and let the user pass in whatever 
they want.

    (<span class="kw">defn</span><span class="fu"> filter-reducer</span>
      [predicate]
      (<span class="kw">fn</span> [result input]
        (<span class="kw">if</span> (predicate input)
          (<span class="kw">conj</span> result input)
          result)))
    
    (<span class="kw">reduce</span> (filter-reducer <span class="kw">even?</span>) [] '(<span class="dv">1</span> <span class="dv">2</span> <span class="dv">3</span> <span class="dv">4</span> <span class="dv">5</span> <span class="dv">6</span> <span class="dv">7</span> <span class="dv">8</span> <span class="dv">9</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [2 4 6 8 10]</span>

We can even compose `map-reducer` and `filter-reducer` together:

    (<span class="kw">reduce</span>
      (filter-reducer <span class="kw">even?</span>)
      []
      (<span class="kw">reduce</span>
        (map-reducer <span class="kw">inc</span>)
        []
        (<span class="kw">range</span> <span class="dv">10</span>)))
    <span class="co">; ⇒ [2 4 6 8 10]</span>

The expression above is equivalent (ignoring vectors versus lists) to the
expression below.

    (<span class="kw">filter</span> <span class="kw">even?</span> (<span class="kw">map</span> <span class="kw">inc</span> (<span class="kw">range</span> <span class="dv">10</span>)))
    <span class="co">; ⇒ (2 4 6 8 10)</span>

We see that with higher-order functions, we are able to define `map` and 
`filter` in terms of `reduce`.

Both versions, however, required intermediate vectors—one for map and one for
filter. One important property of transducers is that they should employ only 
one collection regardless of the number of transformations. How can we 
accomplish that?

## Another step in abstraction {#another-step-in-abstraction}

Let’s scrutinize `map-reducer` and `filter-reducer`. Here they are again:

    (<span class="kw">defn</span><span class="fu"> map-reducer</span>
      [f]
      (<span class="kw">fn</span> [result input]
        (<span class="kw">conj</span> result (f input))))
    
    (<span class="kw">defn</span><span class="fu"> filter-reducer</span>
      [predicate]
      (<span class="kw">fn</span> [result input]
        (<span class="kw">if</span> (predicate input)
          (<span class="kw">conj</span> result input)
          result)))

What do you observe? `conj` is used in both of them. Why? What’s so special
about it? Can we use other functions in place of`conj`?

Well, notice that `result` and `input` can be of any type. If `result` is `10`
and`input` is `1`, `conj` would not work here; `(conj 10 1)` throws an error.
Instead of`conj`, we would want something like `+`, because `(+ 10 1)` makes
sense.

We can say that `conj` and `+` are both **reducing functions**. Reducing
functions have the type`result, input -> result`; they take a result and an
input, and returns a*new* result. For example:

    (<span class="kw">conj</span> [<span class="dv">1</span> <span class="dv">2</span> <span class="dv">3</span>] <span class="dv">4</span>)
    <span class="co">; ⇒ [1 2 3 4]</span>
    
    (<span class="kw">+</span> <span class="dv">10</span> <span class="dv">1</span>)
    <span class="co">; ⇒ 11</span>

Now, instead of always using `conj` in `map-reducer` and `filter-reducer`, what
if we let the user pass in whatever reducing function they want?

This will result in another higher-order function that takes our map’s
transform function and filter’s predicate function, as usual. But we now will 
return a function that accepts a reducing function. Let’s use the names
`mapping` and `filtering` for our new functions.

    (<span class="kw">defn</span><span class="fu"> mapping</span>
      [f]
      (<span class="kw">fn</span> [reducing]
        (<span class="kw">fn</span> [result input]
          (reducing result (f input)))))
    
    (<span class="kw">defn</span><span class="fu"> filtering</span>
      [predicate]
      (<span class="kw">fn</span> [reducing]
        (<span class="kw">fn</span> [result input]
          (<span class="kw">if</span> (predicate input)
            (reducing result input)
            result))))

And now let’s use them as before:

    (<span class="kw">reduce</span>
      ((filtering <span class="kw">even?</span>) <span class="kw">conj</span>)
      []
      (<span class="kw">reduce</span>
        ((mapping <span class="kw">inc</span>) <span class="kw">conj</span>)
        []
        (<span class="kw">range</span> <span class="dv">10</span>)))
    <span class="co">; ⇒ [2 4 6 8 10]</span>

We see here that we can *choose* the reducing function. In this case, we choose
`conj`.

## Arriving at transducers {#arriving-at-transducers}

Take note of the functions `((mapping inc) conj)` and 
`((filtering even?) conj)`. Their types are `result, input -> result`. We
can test this:

    (((mapping <span class="kw">inc</span>) <span class="kw">conj</span>) [] <span class="dv">1</span>)
    <span class="co">; ⇒ [2]</span>
    
    (((mapping <span class="kw">inc</span>) <span class="kw">conj</span>) [<span class="dv">2</span>] <span class="dv">2</span>)
    <span class="co">; ⇒ [2 3]</span>
    
    (((mapping <span class="kw">inc</span>) <span class="kw">conj</span>) [<span class="dv">2</span> <span class="dv">3</span>] <span class="dv">3</span>)
    <span class="co">; ⇒ [2 3 4]</span>
    
    (((filtering <span class="kw">even?</span>) <span class="kw">conj</span>) [<span class="dv">2</span> <span class="dv">4</span>] <span class="dv">5</span>)
    <span class="co">; ⇒ [2 4]</span>
    
    (((filtering <span class="kw">even?</span>) <span class="kw">conj</span>) [<span class="dv">2</span> <span class="dv">4</span>] <span class="dv">6</span>)
    <span class="co">; ⇒ [2 4 6]</span>

This means that `((mapping inc) conj)` and `((filtering even?) conj)` **are
also reducing functions**, just like `conj` and `+`.

So what happens if we compose these two functions like this:

    ((mapping <span class="kw">inc</span>) ((filtering <span class="kw">even?</span>) <span class="kw">conj</span>))

This is also a function. But what is its type? Go on, evaluate it.

It turns out, this function *also* has the type `result, input -> result`.
It is also a reducing function. This means that we can use it via`reduce`:

    (<span class="kw">reduce</span> ((mapping <span class="kw">inc</span>) ((filtering <span class="kw">even?</span>) <span class="kw">conj</span>)) [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [2 4 6 8 10]</span>

This is a bit messy, so let’s clean it up by using `comp` instead. Recall that
`(comp a b c d)` returns the function

    (<span class="kw">fn</span> [r] (a (b (c (d r)))))

Here’s the cleaned up version, using `comp`:

    (<span class="kw">def</span><span class="fu"> xform</span>
      (<span class="kw">comp</span>
        (mapping <span class="kw">inc</span>)
        (filtering <span class="kw">even?</span>)))
    
    (<span class="kw">reduce</span> (xform <span class="kw">conj</span>) [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [2 4 6 8 10]</span>

And what about something more complex:

    (<span class="kw">defn</span><span class="fu"> square </span>[x] (<span class="kw">*</span> x x))
    
    (<span class="kw">def</span><span class="fu"> xform</span>
      (<span class="kw">comp</span>
        (filtering <span class="kw">even?</span>) 
        (filtering #(<span class="kw"><</span> % <span class="dv">10</span>))
        (mapping square)
        (mapping <span class="kw">inc</span>)))
    
    (<span class="kw">reduce</span> (xform <span class="kw">conj</span>) [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [1 5 17 37 65]</span>

Beautiful.

But we were talking about transducers. Where are our transducers?

It turns out, `mapping` and `filtering` are transducer-returning functions. The
functions`(mapping inc)`, `(filtering even?)` and `xform` **are the very
transducers we were looking for.**

Transducers are functions that accept a reducing function and return a reducing
function. For example, the function`(mapping inc)` is a transducer because it
accepts a reducing function like`conj`, and returns another reducing function,
as we observed above. As Rich Hickey pointed out in[Transducers are coming][4]

    (result, input -> result) -> (result, input -> result)

Also observe that no intermediate collections are created when we evaluate 
`(reduce (xform conj) [] (range 10))`, other than the initial vector. This
satisfies our goal of not allocating intermediate collections.

## A more intuitive understanding {#a-more-intuitive-understanding}

It may be difficult to understand why the `xform` transducer works because it
’s quite complicated. Let’s try to get a better understanding. Here’s`xform` as
before:

    (<span class="kw">defn</span><span class="fu"> square </span>[x] (<span class="kw">*</span> x x))
    
    (<span class="kw">def</span><span class="fu"> xform</span>
      (<span class="kw">comp</span>
        (filtering <span class="kw">even?</span>)
        (filtering #(<span class="kw"><</span> % <span class="dv">10</span>))
        (mapping square)
        (mapping <span class="kw">inc</span>)))

Say we invoke this composed function by passing in some reducing function,
perhaps our favorite,`conj`. This would be passed to the transducer 
`(mapping inc)`, and we would have `((mapping inc) conj)`. We know that this
returns*another* reducing function. This new reducing function is then passed
into the function`(mapping square)`, which is another transducer. Naturally,
this returns*another* reducing function. And we do this all the way to the
first transducer in our composition,`(filtering even?)`.

This means that when we give a reducing function to `xform`, like 
`(xform conj)`, we get back a function that will apply the left-most reducing
function first, then down the stack until the last reducing function,`conj`, is
applied to the current result and input.

Imagine this transducer is being used in some reduce function, and we have so
far collected in our results the vector`[1 5 17]`. Say the current input in
question is`12`. Since `12` is even, it will pass the first filter. This first
filter will then call its reducing function, passing in`[1 5 17]` and `12`. In
this case, the reducing function is the “rest” of the transformation, which is 
the second filter`#(< % 10)`. Since `12` fails the second filter, the third
reducing function is*not* called, and the result-so-far `[1 5 17]` is returned
.

But if the input in question is `6`, it would pass both filters and arrive at
the mapping transforms, which will transform`6` to `37`. We then pass this
input to the final reducing function,`conj`, which will join `[1 5 17]` with
the new value`37`.

    ((xform <span class="kw">conj</span>) [<span class="dv">1</span> <span class="dv">5</span> <span class="dv">17</span>] <span class="dv">12</span>)
    <span class="co">; ⇒ [1 5 17]</span>
    
    ((xform <span class="kw">conj</span>) [<span class="dv">1</span> <span class="dv">5</span> <span class="dv">17</span>] <span class="dv">6</span>)
    <span class="co">; ⇒ [1 5 17 37]</span>
    
    (<span class="kw">reduce</span> (xform <span class="kw">conj</span>) [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [1 5 17 37 65]</span>

Being able to compose transducers is important. We see that it’s quite simple
to do, and that ordinary functions power all of it.

## Transducers in core.async {#transducers-in-core.async}

Another major selling point of transducers is that a transducer can work across
core.async channels. For example, we should be able to take our`xform`
transducer and use it to filter and transform items in a channel.

Using Clojure’s transducer library:

    (<span class="kw">defn</span><span class="fu"> square </span>[x] (<span class="kw">*</span> x x))
    
    (<span class="kw">def</span><span class="fu"> xform</span>
      (<span class="kw">comp</span>
        (<span class="kw">filter</span> <span class="kw">even?</span>)
        (<span class="kw">filter</span> #(<span class="kw"><</span> % <span class="dv">10</span>))
        (<span class="kw">map</span> square)
        (<span class="kw">map</span> <span class="kw">inc</span>)))
    
    (<span class="kw">def</span><span class="fu"> my-chan </span>(async/chan <span class="dv">1</span> xform))
    
    <span class="co">; Waiting for an item to print...</span>
    (async/take! my-chan <span class="kw">println</span>)
    
    (async/put! my-chan <span class="dv">3</span>)
    <span class="co">; nothing printed to screen, since 3 is not even</span>
    
    (async/put! my-chan <span class="dv">4</span>)
    <span class="co">; "17" printed to screen, since 4 is even and less than 10</span>

How do transducers work across core.async channels?

First, note that channel buffers are linked lists underneath (in fact, 
`java.util.LinkedList`s). When you put an item into a channel, the internal
helper method`add!` is called to add your item into the buffer.

But if a transducer `xform` is supplied, core.async will use `add!` as the
reducing function passed into`xform`:

This means that any item put into a channel will first be transformed by our
transducer. And if the transducer filters out an item (e.g. due to
`(filter even?)`), then the final reducing function `add!` is never called.
Thus the item is never added to the channel’s buffer and no takers ever see it.

The pertinent code can be found in the core.async sources, [here][5].

## Conclusion {#conclusion}

We’ve come a long ways. We started with regular `map` and `filter` and observed
how they can be implemented using`reduce`. We then abstracted our reducing
functions until we found ourselves with transducer-building functions,`mapping`
`filtering`.

By building, analyzing and using transducers, I hope that you gained a better
understanding of how they work. They are, after all, just functions.

If you’re interested in learning more, I encourage you to tackle the problems
below.[Solutions can be found here][6].

**Write a `transduce` helper function**

Right now, our use of `reduce` is a bit clunky. Write a function `transduce`
that will allow us to use transducers like this:

    (transduce xform <span class="kw">conj</span> [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [5 17 37 65]</span>

**The Caesar Cipher**

In our examples above, we used `conj` and `+` as reducing functions. Let’s
write a more complex one.

Given a string, use transducers to:

*   Filter out vowels and non-ASCII characters
*   Filter out upper-case characters
*   Rotate all remaining characters via a [Caesar cipher][7],
*   And reduce the rotated characters into a map counting the number of
    occurrences of each character.
   

Example:

    (<span class="kw">defn</span><span class="fu"> caesar-count</span>
      [string cipher]
      ???)
    
    (caesar-count <span class="st">"abc"</span> <span class="dv">0</span>)
    <span class="co">; ⇒ {\c 1, \b 1}</span>
    
    (caesar-count <span class="st">"abc"</span> <span class="dv">1</span>)
    <span class="co">; ⇒ {\d 1, \c 1}</span>
    
    (caesar-count <span class="st">"hello world"</span> <span class="dv">0</span>)
    <span class="co">; ⇒ {\d 1, \r 1, \w 1, \l 3, \h 1}</span>
    
    (caesar-count <span class="st">"hello world"</span> <span class="dv">13</span>)
    <span class="co">; ⇒ {\q 1, \e 1, \j 1, \y 3, \u 1}</span>

**Write a `mapcat` transducer**

Write `mapcatting`, a function that returns a `mapcat` transducer.

Examples of `mapcat` (no transducers):

    (<span class="kw">defn</span><span class="fu"> twins </span>[x] [x x])
    
    (<span class="kw">mapcat</span> twins (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ (0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9)</span>

The transducer should work like this:

    (<span class="kw">defn</span><span class="fu"> mapcatting </span>[f] ???)
    
    (<span class="kw">reduce</span> ((mapcatting twins) <span class="kw">conj</span>) [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9]</span>

**Write a `take` transducer**

Write `taking`, a function that returns a `take` transducer.

    (<span class="kw">defn</span><span class="fu"> taking </span>[n] ???)
    
    (<span class="kw">reduce</span> ((taking <span class="dv">3</span>) <span class="kw">conj</span>) [] (<span class="kw">range</span> <span class="dv">10</span>))
    <span class="co">; ⇒ [0 1 2]</span>

Note that you may need to keep some state for this one.

[Source code for this post][3]

[Tom Ashworth: CSP and transducers in JavaScript][8]. This is the original blog
post that helped me understand transducers.

[Rich Hickey: Transducers are coming][4]

 [1]: http://elbenshira.com/p/clojure-primer-js/
 [2]: http://clojurescript.net/
 [3]: https://gist.github.com/elben/da8864e120c373e5fcf0
 [4]: http://blog.cognitect.com/blog/2014/8/6/transducers-are-coming

 [5]: https://github.com/clojure/core.async/blob/ac0f1bfb40237a18dc0f03c0db5df41657cd23a6/src/main/clojure/clojure/core/async/impl/channels.clj#L287

 [6]: https://gist.github.com/elben/da8864e120c373e5fcf0#file-understanding-transducers-clj-L204
 [7]: http://en.wikipedia.org/wiki/Caesar_cipher
 [8]: http://phuu.net/2014/08/31/csp-and-transducers.html