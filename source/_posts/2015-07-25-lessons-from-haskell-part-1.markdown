---
layout: post
title: "Lessons From Haskell Part 1"
date: 2015-07-25 15:16:28-0500
comments: true
categories:
---

After hearing about Scala (and failing the first time I tried to learn it) I had
an interest in functional programming and eventually ending up learning Haskell
in my free time. One of the coolest things that I learned from Haskell was lazy
evaluation.

I know lazy evaluation is sort of  a wedge in the programming community. Some
people like it, other people feel like it makes understanding the program more
complicated especially when it comes to memory usage, but it does have some
perks. I don't want to get into lazy evaluation by default, but rather how
thunks could be used to decrease memory usage.

## Thunk Basics

On the surface, thunks are pretty straight-forward. They are nothing more than
lambdas or anonymous functions that have no parameters. In OCaml an explicit
thunk can be written like this

```ocaml
fun () -> print_endline("hello world")
```

This particular function has type `unit -> unit`, but it's an anonymous
function that has no formal arguments, so just like that we have a thunk!

So once we have a thunk we can evaluate it by evaluating the underlying
function and access the actual value.

```ocaml output from ocaml
let x = fun () -> print_endline("Hello world")
in x ();;
(* prints hello world *)
```

`x ()` supplies the `unit` needed to apply the thunk and we get access to the
underlying value, in this case, a function that returns `()` and prints "Hello
world".

This particular example is pretty boring, but we can use thunks to defer
expensive computations. For example, let's make a function that sleeps for five
seconds and then returns a value. Let's pretend that this function is actually
doing something important in those five seconds like making requests to a server
or computing the Mandlebrot set.

```ocaml
let y = fun () -> UnixThread.sleep 5; 23;;
```

This particular thunk now has type `unit -> int` since it still requires the
mandatory `unit` but now it results in an `int`.  Even though this function
sleeps for five seconds, the function is created just as fast a regular function
and we can choose when to evaluate it. Only when it's actually evaluated will
the call to `sleep` happen.

## Using Thunks with Large Datasets

So far we know that thunks are anonymous functions that have no formal
parameters and they can be used to defer expensive computations, but what kind
of expensive computations can they defer?

In writing a data-serialization library from MATLAB to
[HDF5](https://www.hdfgroup.org/HDF5/) to Python, Java, C,
and R, I quickly realized that work should be done to minimize memory usage
since most of the input files were around 750 MB and some where over a gig. I
took different approaches in each of the target languages, but for the R
version, I settled on using explicit thunks to decrease the memory usage. While
R *is* a lazy language, I was using S4 objects which are strict in their
constructor. So if S4 objects are strict, how could thunks be used?

Instead of storing the actual HDF5 objects, which are read in as lists of values
in R, we can store the names of the HDF5 objects to the datasets and create
thunks that would *eventually* evaluate to the actual HDF5 object. For example,
if we have a HDF5 file with three objects such as

```
+--- /
     +--- foo
     +--- bar
     +--- baz
```

Instead of storing the actual values for all three HDF5 objects, we can just
store three thunks and let the user decide what objects they want to read.  Only
when they are requested is anything actually read. These could then be stored
in a list.

```r
data['foo'] <- function () { return hdf5.read("foo") }
data['bar']  <- function () { return hdf5.read("bar") }
data['baz']  <- function () { return hdf5.read("baz") }
```

And accessing the actual object--evaluating the thunk--is as simple as

```r
data['foo']()
```

Since we're not storing the actual objects themselves, but thunks that *will*
evaluate to them, we can keep the memory usage down. Instead of storing lists of
thousands of elements, we just store a simple, humble function.

Writing some pleasant functions to wrap these operations, we can have code that
looks like this that lazily reads the objects in a HDF5 file.

```R
# only the thunks are created here
hd <- Hdf5Structure("file.h5")
# now we evaluate the "foo" object
foo <- get.entry(hd, "foo")
# do something with test...
```

In this example, only the object `foo` is read and the other objects in the HDF5
file remain wrapped in thunks, waiting to be evaluated. To make it sweeter, we
can also avoid storing `NULL`s in the list.

## Thunks and Streams

While this example is pretty simple, the humble thunk can be used to do all
sorts of cool things. They can be used to create lazy lists or
[Streams](http://www.cs.cornell.edu/courses/cs3110/2011sp/lectures/lec24-streams/streams.htm)
that let you create potentially infinite lists. Using the idea of streams,
we can define an infinite list of Fibonacci numbers in Haskell. (Haskell is
lazy by default so a list in Haskell is already a `Stream`)

```haskell
fibs = 0 : 1 : zipWith (+) fibs (tail fibs)
```

And then get the first 10

```
take 10 fibs -- [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

After this, only the first 10 ten elements in `fibs` are evaluated, the rest of
the list remains wrapped in a thunk.

Just don't try to print the whole list!
