---
layout: post
title: "The HOF of HOF"
date: 2015-10-04 13:15:04 -0500
comments: true
categories:
---

One of the first things that could strike fear into anyone who wants to learn
functional programming that comes from an imperative background is the lack of
loops. Languages like
[OCaml](https://ocaml.org/learn/tutorials/if_statements_loops_and_recursion.html#Forloopsandwhileloops)
supports the familiar for-loop
and
[Clojure](https://clojuredocs.org/clojure.core/loop)
kind of supports for-loops, but
these are seen more as escape hatches and aren't considered 'pure'. They are
nice for the convenience and the familiarity of them, but a higher order
function would be better to use in a functional language and there are a couple
of basic ones at your disposal: `map`, `fold`, and `filter` (or variations
thereof for certain languages). Using these in place of for-loops not only
results in cleaner code, but also forces you to think about what you plan on
doing with your for-loop in any language. Each of these functions has a
particular use, so let's enter **The Hall of Fame of Higher-Order Functions**
and see them one at a time.

## First Attraction: Map
![](http://reactivex.io/documentation/operators/images/map.png)
`map` transforms a collection by applying a function to each element. `map`
essentially replaces something like this in Java

```java
for (int i = 0; i < arr.length; i++) {
    arr[i] = arr[i] * 2;
}
```

with this

```haskell
map (+ 2) arr
```

It's Haskell specific signature is `map :: (a -> b) -> [a] -> [b]` which can be
translated to: `map` takes a function that takes `a`s as input and produces
`b`s, a list of `a`s and finally produces a list of `b`s. While that's kind of
awkward to try to distill in English. Some examples definitely help.

Assuming you have a list of `Int`s and you want a list of `String`s, you can
convert the while list with this simple one-liner

```haskell
map show [1, 2, 3, 4, 5] -- ["1", "2", "3", "4", "5"]
```

Want to convert a `String` to all upper case? Map can do that too

```haskell
map toUpper "hola mundo"
```

which results in the `String` "HOLA MUNDO"

## Second Attraction: Filter
![](http://reactivex.io/documentation/operators/images/filter.png)
`filter` lives up to it's name and filters elements from a list. `filter`
applies a predicate to each element in the lists and only keeps those elements
if the predicate is true for that element. The Haskell-specific signature for
`filter` is `filter :: (a -> Bool) -> [a] -> [a]`. Skipping the verbosity of a
translated description, let's go straight to some examples.

Removing odd elements from a list becomes ridiculously easy

```haskell
filter even [1, 2, 3, 4, 5]
```

While that example is straight-forward, `filter` could be used for more complex
things. Let's say for some peculiar reason you use `ghci` as your grocery list
and you want to remove elements that you think are too expensive.  We could
code this up in Haskell and use `filter` to remove items that cost more than
$3.00

```haskell
data Product = Product {
    name :: String,
    price :: Float
}

ps = [Product "Milk" 2.34, Product "Eggs" 3.45, Product "Bread" 2.20]

filter (\p -> price p < 3.00) ps
```

which is far more succinct--and explicit--than something along the lines of

```java
public class Product {
    float price;
    String name;

    public Product(String name, float price) {
        this.name = name;
        this.price = price;
    }

    // getters and setters...

}

Product ps[] = new Product[]{new Product("Milk", 2.34), new Product("Eggs",
        3.45), new Product("Bread", 2.20)};

LinkedList<Product> cheapProducts = new LinkedList<Product>();
for (int i = 0; i < ps.length; i++) {
    if (ps[i].getPrice() < 2.0) {
        cheapProducts.add(ps[i]);
    }
}
```

## Our Final Attraction: Fold
<p style="text-align:center;">
<img src="https://wiki.haskell.org/wikiupload/5/5a/Left-fold-transformation.png">
</p>

<p style="text-align:center;">
<img src="https://wiki.haskell.org/wikiupload/3/3e/Right-fold-transformation.png">
</p>

`fold` is *the* higher-order recursive function. `map` and `filter` can be
expressed in terms of fold. In The HOF of HOF `fold` is the GOAT. `fold` is
complex at first, but once you get a hold of it, you'll see that it can be
applied to so many different areas. So if `map` is for transforming elements
and `filter` is for removing elements, what's the purpose of `fold`? Well,
`fold` is for reducing a collection of values to a single value, which explains
why `fold` is also known as `reduce` or `aggregate` in so many other languages.

Haskell has two primary `fold` functions. `foldr` starts from the right end of
the list, and `foldl` starts at the left-end of the list. When you're using an
associative function, the difference isn't all that important, but once you
start using functions that aren't associative like subtraction, the difference
is appearant. For some reason, I had a hard time remembering the difference
between `foldl` and `foldr`, in order to remember them, I had to think about
where the parentheses end up being grouped together.  I'm getting a little
ahead of myself, so time to show the signature and some examples.

Since Haskell doesn't have a generic `fold` function, I'm going to use `foldl`
since it has the most analogues in other languages. The signature for `foldl` is
`foldl :: (b -> a -> b) -> b -> [a] -> b`. This function would be really verbose
to translate, so to the examples we go.

The sum of a list can be expressed in terms of `fold`

```
foldl (+) 0 [1, 2, 3, 4, 5]
```

and so can the product

```haskell
foldl (*) 1 [1, 2, 3, 4, 5]
```

But a brief walk through is in order since `foldl` is really the most
complicated of the three functions.

### Walk Through
Let's take the sum example and dissect it step-by-step.

The initial call is `foldl (+) 0 [1, 2, 3, 4, 5]`

The first argument is the function to use to reduce the list and the second
argument is the initial value to use. Eventually, summing a list with `foldl`
results in something along the lines of

```
(((((0 + 1) + 2) + 3) + 4) + 5)
```

But it uses the structure of recursive lists to accomplish this.

We can write an un-optimized `fold`

```haskell
fold f acc [] = acc
fold f acc (x:xs) = fold f (f acc x) xs
```

and use it throughout this example.

Ok, so with the initial call to `fold` we follow through the second function
definition since the list isn't empty. If we expand the function we get

```
fold (+) 1 [2, 3, 4, 5]
```

In the first reduction, the initial value is simply added to the initial
accumulator and the list is reduced and now has one less element.

The second call would expand to `fold (+) 1 [2, 3, 4, 5]` so once again we use
the second definition and the recursive call would be

```
fold (+) 3 [3, 4, 5]
```

we do much the same thing with 3, 4, 5, and it's not until the end of the list
that we reach the first definition of `fold`. Since the list is now empty and
we've accumulated all the values, the only thing we need to do is return the
accumulated value, which in this example is 15.

This is a high-level overview of `fold` and the truth is that the magic happens
in the function that is passed to `fold`, so instead of using the shortcut of
`(+)`, let's expand it to a proper lambda. `(+)` could be expanded to
`\x y -> x + y`, and for this particular example, let's give the variables some
meaningful names. If we give the variables better names, we get
`\acc x -> acc + x`.

Now that we've expanded the function used to reduce the list, let's walk through
the example again, giving explicit values to `acc` and `x`.

The initial call is `fold (\acc x -> acc + x) 0 [1, 2, 3, 4, 5]`.

And here's a table to show what happens at each step

<table style="width:40%" border="1">
    <tr>
        <th> List </th>
        <th> acc </th>
        <th> x </th>
    </tr>
    <tr>
        <td> [2, 3, 4, 5] </td>
        <td> 0 </td>
        <td> 1 </td>
    </tr>
    <tr>
        <td> [3, 4, 5] </td>
        <td> 1 </td>
        <td> 2 </td>
    </tr>
    <tr>
        <td> [4, 5] </td>
        <td> 3 </td>
        <td> 3 </td>
    </tr>
    <tr>
        <td> [5] </td>
        <td> 6 </td>
        <td> 4 </td>
    </tr>
    <tr>
        <td> [] </td>
        <td> 10 </td>
        <td> 5 </td>
    </tr>
</table>
<br>

At this point, all there is to do is add 10 and 5 and return the result which is
15.

### The Expressiveness of fold
As mentioned earlier, fold is *the* recursive higher-order function, one of
the reasons for this is that the return type is entirely open-ended. Unlike
`map` and `filter` which have to return a list, `fold` is free to return
anything, as long as it's the *accumulated* value for that list. This means that
`map` could be written in terms of fold and the accumulated value is simply the
original list but with the function applied to each element.

With that in mind, `map` could be expressed in terms of `fold` and the folding
function has to accumulate the new list with the function applied.

```haskell
myMap :: (a -> b) -> [a] -> [b]
myMap f = foldr (\x acc -> f x : acc) []
```

And similarly, `filter` could be expressed in terms of `fold` and the
accumulated value is simply a list where the elements satisfy the given
predicate.

```haskell
myFilter :: (a -> Bool) -> [a] -> [a]
myFilter p = foldr (\x acc -> if p x then x : acc else acc) []
```

## Tips and Tricks
As alluded to earlier, I had a hard time when I first encountered `fold`. It
felt like I was learning loops again, and I think this has to do with the fact
that it's so general compared to `filter` and `map`. You could reduce a list in
so many ways and reduction is open-ended in every case, it could be a single
value or a new collection. While `map`ing and `filter`ing have straight-forward
behavior, `fold`ing and reducing could vary from context to context. As the
previous section shows, `fold` can be used express a lot of other functions.
To save someone else the ride on `fold`s learning-curve, I want to share a
couple of things that really helped `fold` click for me.

### It's All in the Reducing Function
`fold` is driven by the reducing function and as such, understanding how the
reducing function applies in different contexts is extremely helpful. So to
illustrate that concept, let's implement `head` and `last` in terms of `fold`.
`head` returns the head of a list so `head [1, 2, 3, 4, 5]` should return `1`
and `last` returns the last element in a list so `last [1, 2, 3, 4, 5]` should
return `5`. Let implement these functions in terms of `fold`.

For these particular functions, I'm going to be using a variation of `foldl`
called `foldl1`. `foldl1` is almost the same as `foldl` except it uses the first
element in the list instead of an explicit initializer. Because of this it has a
simpler signature, which is `foldl1 :: (a -> a -> a) -> [a] -> a`.

So how would we write `head` in terms of `foldl1`? Well we need to write the
proper function. In this case, the function should simply return the first
element that it sees and ignore everything else. In Haskell, this function
could be written as an anonymous function as `(\acc x -> acc)` which simply
ignores any other element in the list since it keeps passing the first element
(the accumulated value). With the function defined, we could write `head` as

```
myHead = foldl1 (\acc x -> acc)
```

With each recursive call, `myHead` will reduce the list by one element and
discard that element, instead it passes the initializer all the way down the list.
Obviously, this isn't the most asymptotically efficient function since it has to
iterate through the entirety of the list just to return the first element, but
this shows that it can be done in terms of `fold`.

Now onto `last`. If `head` ignored any subsequent element after the initializer,
`last` does the exact opposite. It ignores any previously seen elements and just
keeps passing the new value. This function is very similar to the one used for
`head`: `(\acc x -> x)`. This lambda reduces the list by one element and keeps
passing the new element until we reach the end. Now we can write our version of
`last`.

```haskell
myLast = foldl1 (\acc x -> x)
```

Finally, the most important thing to remember about the reducing function is
that the order of arguments makes a difference. Conveniently, there's a simple
mnemonic device to remember it. When using `foldl` the accumulator should be on
the left side so the lambda should follow something like `\acc x -> ...` and
with `foldr` the accumulator should be on the right side like `\x acc -> ...`.

### One Last Example
So finally, for the last example we'll write a simple function that reads a
`String` and converts it to an `Int`. We'll use `fold` to traverse the `String`
and the reducing function will handle building up the final value. The first
thing we need to decide is which `fold` to use. Since we need to traverse the
`String` from the beginning of the list, we should use `foldl`. With that out of
the way, we can move on to the folding function. Since we're using `foldl` we
can start off with the following function template `\acc x -> ...` and we just
need to fill in the rest.

The function needs to handle reading the `Char` and converting it to an `Int`.
Luckily, Haskell has the function `digitToInt` that resides in the `Data.Char`
module. `digitToInt` lives up to its name quite well. The signature is
`digitToInt :: Char -> Int`, and `digitToInt '7'` returns `7` and `digitToInt
'2'` returns `2`. `digitToInt` doesn't handle `String`s, but that's exactly the
function that we are writing! So let's start calling this function `stringToInt`.

Ok, so we have reading the `Char` as an `Int` and we just need to handle adding
the new digit and multiplying the accumulated value to account for the different
places in the number. Fortunately, we're using standard base-10 numbers so all
this is ridiculously easy. The folding function is `\acc x -> digitToInt x + acc
* 10`. Now, we can tie it all together and `stringToInt` is an awesome
one-liner:

```haskell
stringToInt :: String -> Int
stringToInt s = foldl (\acc x -> digitToInt x + acc * 10) 0 s
```

So let's try out a new function, to see if it works

- `stringToInt "23541"` returns `23541`

- `stringToInt "1"` returns `1`

- `stringToInt "873132"` returns `873132`

The really cool thing about `stringToInt` is that we could use it as a template
and use it to read binary string or hexadecimal strings. What do we have to
change to read binary strings? Well, instead of multiplying by 10 we just
multiply by 2. <sup>(Ignoring error-checking of course)</sup>

```haskell
binaryStringToInt :: String -> Int
binaryStringToInt s = foldl (\acc x -> digitToInt x + acc * 2) 0 s
```

And since `digitToInt` also handles hexadecimal digits, all we need to change is
the base to convert hexadecimal strings.

```haskell
hexStringToInt :: String -> Int
hexStringToInt s = foldl (\acc x -> digitToInt x + acc * 16) 0 s
```

Obviously, there's a simple pattern here, so let's take it one step further and
make a function that could read a number in *any* base (well actually any base
up to 16 since we're relying on `digitToInt`) and converts it to an `Int`.

```haskell
baseStringToInt :: Int -> String -> Int
baseStringToInt base s = foldl (\acc x -> digitToInt x + acc * base) 0
```

and we could refactor our previous functions to use it

```haskell
stringToInt = baseStringToInt 10

binaryStringToInt = baseStringToInt 2

hexStringToInt = baseStringToInt 16
```

## Conclusion
So that's the brief tour of The HOF of HOF. We saw `map`, `filter`, and finally
the GOAT `fold`. These functions rely on other functions and transform
collections in some particular way. The really great thing about learning these
functions is that they improve your code in any language! I have nothing
against "raw loops"--or loops that use explicit indexes--and understand that
they have their place, but instead of writing loops that do three different
things in the body, it makes code a whole lot cleaner to write a loop that does
one single thing and one thing well. Personally, I used to write for loops with
bodies that were >90 lines and did everything they possibly could. After
learning about `map`, `filter`, and `fold` I now write loop bodies that
accomplish their task and quickly exit.

The body of a loop is a dangerous place, you first have to worry about the
index and not trying to access imaginary elements, and after that you have to
be careful in the actual body and make sure you're not launching missiles,
creating way more server requests than you need, or `free`ing a struct twice.
`map`, `filter`, and `fold` help in reducing the risk involved in for-loops.
Unlike raw loops where the body is anything you want it to be, these functions
require functions that you can easily test outside of these recursive functions.
While not every language supports these functions, learning what you want to
accomplish in a loop can help a lot. Maybe for other people this is
common-sense, but for me, it wasn't until learning `map`, `filter`, and `fold`,
that I started approaching for-loops with a whole lot more discipline. Now, when
I write a for-loop, I'm always aware if I want to remove elements, change
elements, or something more general.

Just like anything else, the functions in The HOF of HOF aren't a cure-all.
They won't result in magic code that never fails or performs exactly as you
want. You can still mess up and have your code blow up in your face, but these
functions enforce the thing that every would-be programmer hears in class or
sees in an article somewhere: don't repeat yourself--use functions. Write
functions to break up behavior and approach your code in a sensible way. Maybe
functional programming is a pain, but ultimately, it forces you to use
higher-level constructs that are just a little bit safer and little bit
clearer.

For-loops have their place and everyone would be lost without them, but
instead of approaching them and just saying "I need to do *x* this many times,"
think about exactly what you need to do. Do you need to remove elements, change
elements or reduce them in some way? Well, it just so happens that that's a
pretty common operation and there's a nice--and safe--way to do that.

*This marks the end of The Hall of Fame of Higher-Order Functions. Maybe there
will be new inductees in the future, but this marks the end for now. The exits
are to the left and to the right. Thank you for your time.*

*... and don't forget to tip your tour guide*
