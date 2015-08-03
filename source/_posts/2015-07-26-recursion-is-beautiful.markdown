---
layout: post
title: "Recursion is Beautiful, The Language Might Be Ugly"
date: 2015-07-26 14:20:48 -0500
comments: true
categories:
---

## Introduction
For better or for worse, recursion is seen as the weird cousin of loops. It
solves some problems really well, it's often elegant and looks extremely simple,
yet for some reason, it doesn't get the attention that it should. I know of
many people that are out-right scared of recursion, and seeing the lengths some
people go to code Trees iteratively or use a loop and a stack only shows that
somewhere along the line, people get a bad idea about recursion. So let's do the
teen movie treatment and take off recursion's glasses and get rid of the
ponytail.

Like most other CS students, my first introduction to recursive programming was
the usual `factorial` function

```java
public int factorial(int n) {
    if (n == 1) {
        return 1;
    }
    return n * factorial(n - 1);
}
```

At first it was mind-blowing, but after looking at it I realized it's a pretty
simple recursive function. Calling `factorial(5)` reduces to

```
factorial(5)
    5 * factorial(4)
        5 * 4 * factorial(3)
            5 * 4 * 3 * factorial(2)
                5 * 4 * 3 * 2 * factorial(1)
                    5 * 4 * 3 * 2 * 1 // we hit the base case, start unwinding
                5 * 4 * 3 * 2
            5 * 4 * 6
        5 * 24
    120
```

All recursive functions have a base case to stop the recursion and the general
case that reduces the problem to sub-problems. For `factorial`, the base case is
1 and the sub-problem is the factorial `n - 1`.

## Up Your Recursive Game

### Lists

After understanding the `factorial` function, the next recursive problem that
came up was recursive data structures, or data structures that are defined in
terms of themselves. The two most famous being Lists and Trees.

We can define a List as either `Nil` which represents an empty List, or a `Cons`
cell followed by a List. So we can have

```
Cons(4, Cons(3, Cons(2, Cons(1, Nil))))
```

Which we could picture with ASCII art as

```
+---+    +---+    +---+    +---+
| 4 | -> | 3 | -> | 2 | -> | 1 | -> Nil
+---+    +---+    +---+    +---+
```

Lists are recursive data structures and their usual definition in Java is
similar to

```java
public class Node<T> {
    T elem;
    Node<T> next;

    // ...
}
```

While the actual data structure is recursive, I was taught to treat them in an
iterative manner. Looping through the list to find the last element and adding a
new element, or looping through the whole structure to print it. Often times,
the code ending up looking something like this

```
Node temp = this.head;
while (temp != null) {
    // do something
    temp = temp.next;
}
```

Because of this, the recursive nature of Lists wasn't really highlighted and it
took another data structure to really drive the nature of recursive data
structures home

### Binary Trees

[Binary Trees](https://en.wikipedia.org/wiki/Binary_tree) are also recursive
data structures that are actually treated as such. Similar to other recursive
data structures, their definition is also simple: a Binary Tree has a root which
is a node, a node has at most two children, a left child and a right child,
these children nodes are also binary trees. Here's a simple Binary Tree in
ASCII art

                4
            /       \
           3         5
        /     \        \
       1       2        7


This particular tree is a Binary Search Tree since each element to the left of a
node is less than the root and each element to the right is greater than the
root. In this example, not only is the tree rooted at 4 a Tree, but

            3
        /       \
       1         2

Is a Tree and so is

        5
            \
             7

A simple Binary Search Tree could be implemented in Java like this

```java
public class BST<T extends Comparable<T>> {
    Node<T> root;

    public BST() {
        this.root = null;
    }

    public void insert(T elem) {
        if (this.root == null) {
            this.root = new Node<T>(elem);
        } else {
            insert(elem, this.root);
        }
    }

    public void insert(T elem, Node<T> node) {
        // the new element is less than this node's value
        if (elem.compareTo(node.elem) < 0) {
            if (node.left == null) {
                node.left = new Node<T>(elem);
            } else {
                insert(elem, node.left);
            }
        }

        // the new element is greater than this node's value
        if (elem.compareTo(node.elem) > 0) {
            if (node.right == null) {
                node.right = new Node<T>(elem);
            } else {
                insert(elem, node.right);
            }
        }
    }

    private class Node<T> {
        T elem;
        Node<T> left;
        Node<T> right;

        public Node(T elem) {
            this.elem = elem;
            this.left = null;
            this.right = null;
        }
    }
}
```

The code is pretty simple, but it's a bit verbose. A `Node` has an element and
two children, `left` and `right`, and a `BST` has a `Node` as a root. Inserting
an element into a `BST` first checks if the root is `null` or if the Tree is
empty, if it is, we create a new node and this node becomes the root. If the
root isn't `null`, we have to check if the element to be added is less than or
greater than the root's element and insert it to the correct side accordingly.

Binary Search Trees made recursion really click for me, but I had one assignment
that studied two separate implementations of trees, one recursive and another
with arrays. The recursive implementation always blew the stack and the one with
the array ran out of space since most of the array is not used, leaving the
impression that recursion simply isn't a viable alternative. So aside from some
special algorithms that would be a mess if they were written iteratively, I
almost never used recursion in my day-to-day programming. Instead I stuck to
loops and thought of recursion as a cool--if a but murky--concept, but
ultimately one that was ineffective.

## Embracing Recursion

While he was teaching us recursion, my professor made a particular remark that
stayed with me. He said something along the lines of, "It's hard to think in
terms of recursion unless you program in Lisp...those people could do anything
with recursion." I looked up Lisp because I really wanted to understand
recursion and saw all the parens and quickly gave up on learning anything that
resembled Lisp. But it left me wondering why Lisp programmers would be better at
recursion than other programmers.

Fast-forward a year or so and there I was learning Haskell, frustrated that it
didn't have loops and thinking how anyone could get anything done with it. I
knew of `map`, `filter`, and `fold`, but thought they were novelties and that
it would be difficult to think in terms of only those three functions. When
recursion came up, I remembered the few times I used it in class and felt uneasy
using *only* recursion.

After seeing code for Trees in Haskell that was similar to this

```haskell
data Tree a = Node a (Tree a) (Tree a) | Empty

insert :: (Ord a) => a -> Tree a -> Tree a
insert Empty elem = Node elem Empty Empty
insert (Node x left right) elem
    | x < elem = Node x (insert left elem) right
    | x > elem = Node x left (insert right elem)
```

I was blown away by how clear it was. First, the defintion of a `Tree` is
exactly what it is. We don't have to create two separate classes, instead we
just create an ADT and say that a Tree is either a `Node` with an element and
two other `Trees` or it's `Empty`. Nothing more and nothing less. The function
to insert an element is equally concise, if the `Tree` is `Empty` we create a
new `Node` and if the `Tree` is a `Node` we either add it to the left or to the
right.

Seeing this function got me rethinking recursion and it made me realize that
recursion *is* a valid tool in the programmer's toolbox, it's just that in most
languages recursion is an after-thought that often leads to the stack being
blown.

I still shudder when I see iterative code to manipulate Trees.

## The Language Might Be Ugly

### Tail Call Optimization

So if recursion is a valid tool, why isn't it used more? I know one of the main
reasons that I avoid recursion because of the stack.

Let's take the familiar factorial function. In Java, we could write

```java
int factorial(int n) {
    if (n == 1) {
        return 1;
    }
    return n * factorial(n - 1);
}
```

But we're limited to a certain `n` since the stack will eventually run out of
space. For the factorial function, the stack space that is required is linear
to `n`. Since we have to maintain the previous calls in order to complete the
function, the stack continues to grow and grow until BAM! the stack is blown. In
Java you get a `StackOverflowError`, in C you get a `SIGSEGV` error (at least on
OS X) and in Python you get a nice `RuntimeError: maximum recursion depth error`
message. However in a language that embraces recursion, what could we do?

In Scala we could translate the code directly and write

```scala
def factorial(n: Int): Int =
  if (n == 1) 1 else n * factorial(n - 1)
```

But we're limited by the same thing. Since Scala is a JVM language it's bounded
by stack space as well, but unlike Java, Scala has support for
[tail-call optimization](http://stackoverflow.com/questions/310974/what-is-tail-call-optimization)
so instead of simply translating the Java code, we could re-write
it to take advantage of tail-call optimization.

```scala
def factorial(n: Int): Int = {
  def loop(n: Int, acc: Int): Int =
    if (n == 1) acc else loop(n - 1, n * acc)

  loop(n, 1)
}
```

This version of factorial *is* tail-recursive. So it uses constant stack space.
Instead of the factorial function evaluating like it did in the example at the
beginning of this article, it now evaluates like this

```
factorial(5)
    loop(5, 1) => loop(4, 5) => loop(3, 20) => loop(2, 60) => loop (1, 120) => 120
```

Instead of `loop` building on the stack, the previous call can be replaced since
the variables from the previous call don't have to be maintained. Tail call
optimization can transform recursive functions to tight `while` loops that don't
require extra space and since it's just a loop, there's no need for the
'unwinding' that usually accompanies recursive functions. Scala also supports
the
[@tailrec](http://www.scala-lang.org/api/current/index.html#scala.annotation.tailrec)
annotation enabling the compiler to verify that the function is
indeed tail recursive. If it's not, the compiler will issue an error before you
even have the chance to blow the stack with what you thought was a
tail-recursive function.

#### An Example
Let's compare the difference that tail-call optimization makes by comparing two
programs. They both compute the factorial of a number.

The first one is in Java, and here's the code

```java
import java.math.BigInteger;

public class Test {
    public static void main(String[] args) {
        System.out.println(factorialIter(17000).toString().length());
        // the stack is blown here
        System.out.println(factorial(17000).toString().length());
    }

    public static BigInteger factorial(int n) {
        if (n == 0) {
            return BigInteger.valueOf(1);
        } else {
            return BigInteger.valueOf(n).multiply(factorial(n - 1));
        }
    }

    public static BigInteger factorialIter(int n) {
        BigInteger res = BigInteger.valueOf(1);

        for (int i = n; i > 1; i--) {
            res = res.multiply(BigInteger.valueOf(i));
        }

        return res;
    }

}
```

`factorial` blows the stack when trying to compute `17000!` while
`factorialIter` does just fine. The result is huge, being a number with 64,538
digits.

The second is in Scala

```scala
import scala.annotation.tailrec
import scala.math.BigInt

def factorial(n: BigInt): BigInt = {
  @tailrec
  def loop(n: BigInt, acc: BigInt): BigInt =
    if (n == 1) acc else loop(n - 1, n * acc)
  loop(n, 1)
}

println(factorial(17000).toString.length)
```

This version happily computes `17000!` so let's try to push it a little bit.

Calling `factorial(100000)` says that the resulting number has 45,654 digits.

`factorial(200000)` results in a number with 973,351 digits.

And finally, `factorial(1000000)` results in a number with 5,565,709 digits.
After **some** time of course.

Tail-call optimization is the most important thing that a language could offer
to make using recursive functions more pleasant. The second most important
thing?


### Syntax
I know syntax isn't the most important thing to consider when deciding what
language to use, but it definitely plays a role. As an example, consider the
difference between the Java and the Haskell implementation for Trees. They both
accomplish the *same* thing: construction and insertion, but the Java version is
easily five times longer. But this isn't necessary a problem with Java, rather
it's a positive of Haskell.

Haskell and other functional languages are simply built with recursion in mind.
Combined with ADTs where you could much more easily express recursive structures
and the ability to define inner functions, writing recursive functions in these
languages is simply easier and cleaner. In Java you have to pollute the
namespace with "helper" functions, while in a functional language you can simply
define the "helper" function as an inner function or avoid it all together.
Theoretically, *if* Java supported tail-call optimization, the factorial
function would be something similar to this

```
public BigInteger factorial(int n) {
    return factorial(n, 1);
}

private BigInteger factorial(int n, BigInteger acc) {
    if (n == 1) {
        return acc;
    } else {
        return factorial(n - 1, n * acc);
    }
}
```

The problem with this is that the private `factorial` function will **never** be
used outside of the public `factorial`, yet there it is, at the top-level,
available for other functions in the same file to call erroneously.

Finally Algebraic Data Types similarly make recursive code cleaner because it
lets you define recursive data structures cleaner. Classes in Java or Python are
sometimes too clunky to use when all you need is a simple representation.
Consider the definition of a Tree

```
data Tree a = Node a (Tree a) (Tree a) | Empty
```

It captures the nature of a Tree without overwhelming you with extraneous
methods or clunky getters and setters. ADTs aren't the answer all the time, but
being able to reach for them when a class will simply be heavy-handed is
helpful.

## Conclusion

Recursion has a bad name for a lot of programmers, often being seen as not
practical or too expensive to actually use, but the truth is that recursion is a
great tool to have in your toolbox. It's not a panacea and it's not always
better than a loop, but some functions are naturally recursive so writing these
functions iteratively would not only be messy but also a much greater task.
Loops are fine for indexing into an array, but if you have a recursive data
structure such as a List, iterating through the List using recursion makes a lot
more sense.

To transform an array, this makes sense

```java
for (int i = 0; i < array.length; i++) {
    array[i] = array[i] * 2;
}
```

But with a List, this makes more sense

```haskell
map :: (a -> b) -> [a] -> [b]
map f [] = []
map f (x:xs) = f x : map xs
```

The way you approach a data structure should be defined by it's nature. You
shouldn't try to approach a tree iteratively just like you shouldn't try to
index into a list and treat it like an array. The problem with this is that the
more common languages don't have good support for recursion so a lot of
programmers could be left thinking that recursion is more-or-less a novelty.

I didn't appreciate recursion until I had no choice but to actually use it.
After actually using it, I started to appreciate recursion for it's simplicity
and it's strength, it was only after I took the
[leap of faith](https://www.cs.berkeley.edu/~bh/pdf/v1ch08.pdf)
that I realized how clear and elegant recursion can be. So before thinking that
recursion is the problem, think about the language first. It might be the
language that is ugly and not recursion itself.
