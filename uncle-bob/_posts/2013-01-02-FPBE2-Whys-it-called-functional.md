---
layout: post
title: FP Basics E2
tags: ["Tools"]
old_tags: ["Functional Programming"]
---

h4. Why's it called Functional?

In the previous episode I told you what all the functional programming hubbub is all about. Remember: Referential Transparency and the multi-core freight train? Since you are here, reading episode 2, I presume you were convinced by my arguments and want to know more.

So the next question to answer is: Why's it called "Functional" programming? The simple answer to that question is that Functional Programming is programming with functions (duh). As answers go, that one's pretty bad. Oh it's accurate, so far as it goes, and it sounds good enough; but it really doesn't answer the question. After all, Java programs are programs with functions.

So then why the word "functional"?

I'm going to get all the math weenies upset with me now; because I'm going to use an analogy with calculus. I’m not going to let that bother me though, since I simply stole the analogy from Wikipedia.

Do you know what dy/dx signifies? In particular, in the expression dy/dx, what is y? Of course, y is a function. dy/dx is the derivative of that function. So dy/dx takes a function as it's argument and returns the derivative of that function as it's result. Right?

Consider d(x^2)/dx: it equals 2x. Notice that 2x is a function. So the argument and the return values are both functions.

Wait. What do you call something that takes arguments and returns values. You call that a function. So dy/dx is a function that takes a function and returns a function.

What if we had a computer language that was like that? What if we could write functions that took functions as arguments, operated upon them without evaluating them, and then returned new functions as the result? What would you call a language like that? What other word could be better than: functional?

And of course now I have all the functional language purists mad at me because they know that that's a completely inadequate way to define a functional language. But it's a step along the path -- and it's an important one.

So, what does it mean to pass a function as an argument to another function. Let's look at our squares of integers program again. Remember it was just:

    (take 25 (squares-of (integers)))

        

Let's make a function out of it by giving it a name and an argument:

    (defn squint [n]
      (take n (squares-of (integers))))

I'm sure you can work out the syntax on your own. It's not rocket science.

Given this definition of squint, we can print the first 25 squares of integers with this command:

    (println (squint 25))

So far this is nothing more than simple function calls and definitions. But what's that `squares-of` function? How is it defined?

    (defn square [n] (* n n))

    (defn squares-of [list]
      (map square list))

Now that's a little more interesting! The `square` function is no surprise, it just multiplies it's argument by itself. It's the `squares-of` definition that's interesting; because it passes the `square` function as an argument to a function named `map`.

The `map` function is one of the staples of functional programming. It takes two arguments: a function `f` and a list `l`; and it returns a new list by applying `f` to every element of `l`.

Passing a function as an argument, like this, is not something that Java programmers do too often. On the other hand, it’s not a completely alien concept either. Any Java programmer who has studied the Command Pattern, or who has used the Listeners in Swing, will understand what’s going on here.

What’s more, most languages nowadays have begun to incorporate the notion of functions passed as arguments. This feature is sometimes called “blocks” or “lambdas”. It is a very common part of any Ruby program; and has become much more important lately in C\#. It is even rumored that the feature will be added to Java sometimes soon.   

So then maybe we don’t have to learn a new language. Maybe our old languages are becoming more and more functional and we can just use those functional features as they become more and more available.

That way lay madness, wailing, and much gnashing of teeth! A language with functional features is not a functional language; and a program written with a few lambdas here and there is not a functional program.

To get a hint of why this is true, we need to look at the integers function.

       (defn integers []
          (integers-starting-at 1))

OK, that’s not too hard. The `integers-starting-at` function simply takes an integer argument and returns a list of all integers starting with the argument.

       (defn integers-starting-at [n]
          (cons n (lazy-seq (integers-starting-at (inc n)))))

This one’s going to take a little explaining. But don’t fret, it’s actually quite simple.

First there’s the `cons` function; which is short for “construct list”. The `cons` function takes two arguments, an element, and a list. It returns a new list which is the element prepended onto the old list. So `(cons 2 [3 4])` returns `[2 3 4]`.

Now all the Clojure people are upset with me because that’s not exactly right. On the other hand, the difference is something we don’t need to deal with right now. For the moment, just remember this: `cons` constructs a list by prepending it’s first argument to the second; which must be a list.

Now, perhaps you are thinking that `cons` just sticks the element on the front of the list; but that’s not quite right. Indeed, it’s not even close to being right. You see `cons` doesn’t change the argument list at all. Instead, it returns an entirely new list with the element prepended to the contents of the argument list.

Oh boy, now the Clojure people are really mad at me; because that’s not right either. And you’re probably thinking I’m crazy too because what fool would copy a whole list into another list just to prepend an element?

OK, so, yes, `cons` actually *does* prepend the element onto the list, and does not copy the list into a new list. On the other hand, it does this in such a way that you can *pretend* that the input and output lists are completely different. For all intents and purposes, the output of `cons` is an entirely new list -- even though it isn’t.

Confused? That’s ok, it’s not actually that difficult to understand. Consider the list `[1 2 3]`. If we implement that as a linked list, it would look like this: `1->2->3`. Now let’s say we cons a `0` on the front. That gives us `0->1->2->3`. Notice, however, that the old list still exists inside the new list. That’s the secret! The old list remains unchanged inside the new list. So we can pretend that `cons` returns a whole new list, leaving the old list unchanged.

This is just a hint at something that all true functional languages do. They allow you to create what appear to be wholly new data structures, while preserving the old data structures unchanged; and they do so without copying. In functional circles, this is known as persistence; and the data structures that behave this way are known as persistent data structures. Such data structures maintain their history at all times. Nothing ever gets deleted or modified within them. However, they have many different “entry-points” each of which provides a different view of the data.  Think of it like a source code control system. Though you delete and modify lines of code; nothing ever gets deleted or changed in the source code repository. The entire history is preserved. You just have many different entry points into the source code repository that allow you to see snapshots of that history.

So, now, let’s go back to our program. We were looking at `integers-starting-at`.

       (defn integers-starting-at [n]
          (cons n (lazy-seq (integers-starting-at (inc n)))))

What is that `cons` prepending, and what is it prepending it to? The first time through, `n` is going to be `1`, and so the `cons` will return a list that starts with `1`. That makes perfect sense because our goal is to print the square of `1`.

OK, but what is the `cons` prepending the `1` onto? Clearly the `1` is being prepended to some list that’s returned by `lazy-seq`.

Here’s where the magic begins. You see, `lazy-seq` is a function that returns a lazy sequence. A lazy sequence is just a list - but with a twist. Instead of a linked list of values like `1->2->3`, a lazy sequence is a value linked to a function: `1->f`. That function, when it is called, returns the next value of the list, linked to another function: `2->f`. And what function are those values linked to? Look close! It’s the argument of `lazy-seq`.

The argument of `lazy-seq` is a recursive call to `integers-starting-at` with an argument of `(inc n)`. The `inc` function simply returns it’s argument incremented by `1`.

So, what is a lazy sequence? A lazy sequence is a list who’s elements are calculated when they are needed, and not before. Every time you ask a lazy sequence for the next item on the list, it simply calls the function that the current item is linked to.

Thus, a lazy sequence has no size. You can keep on asking for more and more elements endlessly; and yet those elements won’t be calculated until they are needed.

The `map` function also returns a lazy sequence. Here’s a simple implementation:

       (defn map [f l]
          (if (empty? l)
            []
            (cons (f (first l)) (lazy-seq (map f (rest l))))))

The `first` function simply returns the first element of it’s argument. The `rest` function simply returns the argument list minus it’s first element; i.e. the rest of the list. So `map` is just a simple recursive function that invokes the function `f` on each element of `l` creating a new lazy sequence of the results.

One thing remains before we can put this all together: the `take` function. After all we’ve been through, this one is really very simple.

       (defn take [n l]
          (if (zero? n)
            []
            (cons (first l) (take (dec n) (rest l)))))

As you can see, the `take` function returns a list composed of the first `n` elements of `l`.

Now, lets practice a little Referential Transparency, and evaluate (by hand) the value of:

       (squint 2)

First we replace `squint` with it’s definition:

       (take 2 (squares-of (integers)))

Next we replace `take` with it’s definition. This is a bit tricky because we have to recurse.

       (if (zero? 2)
          []
          (cons (first (squares-of (integers)))
                (if (zero? (dec 2))
                  []
                  (cons (first (rest (squares-of (integers))))
                        (if (zero? (dec (dec 2)))
                          []
                          ...)))))

I stopped at the third `if` statement because `(dec (dec 2))` is zero. Indeed, we know the status of each of those `if` statements, so we can eliminate them all.

       (cons (first (squares-of (integers)))
              (cons (first (rest (squares-of (integers))))
                    []))

Clearly, given that there are two calls to `cons`, this is going to return a list with two elements in it. We can do a little housekeeping by replacing `integers` with its definition.

    (cons (first (squares-of (integers-starting-at 1)))
         (cons (first (rest (squares-of (integers-starting-at 1))))
               []))

Since the call to `(squares-of (integers-starting-at 1))` occurs twice, we’ll evaluate it once and then replace the calls with it’s value. We begin by replacing `squares-of`:

    (map square (integers-starting-at 1)

 And then `integers-starting-at`:

    (map square (cons 1 (lazy-seq (integers-starting-at 2)))

Now let’s replace `map`. Since `map` begins with `(if (empty? l) ...)`, and since the `(cons 1...)` guarantees that the list won’t be empty, we can skip the `if` statement and just use the body.

    (cons (square (first (cons 1 (lazy-seq...)))) 
          (map square (rest (lazy-seq (integers-starting-at 2)))))

The call to `first` can easily be reduced:

    (cons (square 1) 
          (map square (rest (lazy-seq (integers-starting-at 2)))))

Now a bit more magic. The call to `rest` returns the "rest" of the list by invoking the function passed into `lazy-seq`.

    (cons (square 1) 
          (map square (integers-starting-at 2)))

We can repeat this analysis for `(map square (integers-starting-at 2))`:

    (cons (square 1)
          (cons (square 2) 
                (map square (integers-starting-at 3)))

And then we can reduce the `squares`.

    (cons 1
          (cons 4 
                (map square (integers-starting-at 3)))

We left our previous analysis of the entire function in the following state:

    (cons (first (squares-of (integers-starting-at 1)))
         (cons (first (rest (squares-of (integers-starting-at 1))))
               []))

Now we can plug in our value for `(squares-of (integers-starting-at 1))`.

    (cons 
      (first (cons 1
               (cons 4 
                 (map square (integers-starting-at 3))))
      (cons 
        (first (rest (cons 1
                           (cons 4 
                           (map square (integers-starting-at 3)))))
               []))

That first `first` is easy to reduce. `(first (cons x l))` is simply `x`; so we'll just ignore the second argument of the `cons`.

    (cons 
      1           
      (cons 
        (first (rest (cons 1
                           (cons 4 
                           (map square (integers-starting-at 3)))))
               []))

The `(first (rest...))` is easy to evaluate too.

    (cons 
      1           
      (cons 4 []))

And the result of that, of course, is `[1 4]`.

Did you notice what happened to that `(integers-starting-at 3)`? It never got evaluated. Why? Because the original `(take 2...)` only needed 2 values, so the third was never requested.

And that leads to a powerful realization. Most of us have written loops that push data through the rest of the program. But `(take 25 (squares-of (integers)))` is a loop that *pulls* data out of the rest of the program. This is a profound difference, and something we’re going to spend a lot more time on later.

By this time all the Clojure people, all the functional programming people, and all the math people are furious with me; because I’ve oversimplified so much. And it’s true; there’s a bunch of stuff I’ve glossed over and waved my hands about. Still, what I’ve told you is essentially correct. It is also a pretty good demonstration of the power of treating functions as data that can be passed into and returned from functions.

And that brings us to the climax of this episode; for we can now answer the question posed by the title. We call this programming style functional because it’s all about treating functions as data elements that are manipulated no differently than integers or a characters. The functional programming people refer to that as functions being *first-class citizens*.   A functional language is a language that has functions as first-class citizens, enforces referential transparency by eliminating assignment, and maintains data history with persistent data structures.

As we’ll learn in subsequent episodes, this definition is neither complete, nor fully accurate; but for the moment... it will do.
