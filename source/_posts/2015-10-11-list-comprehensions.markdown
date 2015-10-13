---
layout: post
title: "List Comprehensions"
date: 2015-10-11 16:07:08 -0500
comments: true
categories:
---

## The Initial Question
A couple of years ago, a question was asked about list comprehensions on the
Python Google+ Community. Unfortunately, the question was deleted by the person
who asked it, but it was essentially something along the lines of this

> Can someone explain list comprehensions to me? They look ugly and the syntax
> doesn't seem to be Pythonic

Immediately, answers were posted that showed that this list comprehension

```python
[x * 2 for x in range(0, 100)]
```

is equivalent to this loop

```python
xs = []
for x in range(0, 100):
    xs.append(x * 2)
```

Explanations were given that mentioned that the syntax for list
comprehensions--while not immediately clear--is similar to
[set-builder notation](https://en.wikipedia.org/wiki/Set-builder_notation)
in math.

> Ok, but why?

This is when the execution time came up, using the `timeit` module, I found out
that list comprehensions were actually faster.

Comparing the two code snippets above, we get

```
$ python -m timeit -n 100000 "[x * 2 for x in range(0, 100)]"
100000 loops, best of 3: 11.9 usec per loop
$ python -m timeit -n 100000 "xs = []" "for x in range(0, 100): xs.append(x * 2)"
100000 loops, best of 3: 23.5 usec per loop
```

After this, the conversation quickly devolved into people saying that if you
really care about speed you shouldn't be using Python to begin with, Python is
*so* slow, Python sucks because `_`, *ad infinitum*

I was really amazed that list comprehensions were faster and I always wanted to
look deeper into them, but I was school in then, and I didn't have the time for
it.

But now I have the time, so let's see if we can find out *why* list comprehensions
are faster.

## Dissecting the Bytecode for List Comprehensions
*The following bytecode is produced by Python version 2.7.10. I know Python
doesn't have a standard so the bytecode produced by different versions my very
well be different*

We can use the awesome [dis](https://docs.python.org/2/library/dis.html) module
to get the bytecode so let's write a simple function that builds up the list and returns it

```python
def list_comp():
    return [x * 2 for x in range(0, 100)]
```

Now we can call `dis.dis(list_comp)` and here's the annotated bytecode in all
its glory.

```
          0 BUILD_LIST               0            # pushes [] onto the stack
          3 LOAD_GLOBAL              0 (range)    # pushes `range` onto the stack
          6 LOAD_CONST               1 (0)        # pushes 0 onto the stack
          9 LOAD_CONST               2 (100)      # pushes 100 onto the stack
         12 CALL_FUNCTION            2            # calls `range(0, 100)`
         15 GET_ITER                              # gets the iterator from the list
>>   16 FOR_ITER                 16 (to 35)   # calls `next` on the iterator and pushes the value on the stack
         19 STORE_FAST               0 (x)        # stores the element from the iterator at co_varnames[0]
         22 LOAD_FAST                0 (x)        # pushes co_varnames[0] onto the stack
         25 LOAD_CONST               3 (2)        # loads 2 onto the stack
         28 BINARY_MULTIPLY                       # multiplies the two top variables and stores it on the stack
         29 LIST_APPEND              2            # appends the multiplied value to the list
         32 JUMP_ABSOLUTE           16            # continues the iteration...
>>   35 RETURN_VALUE
```

Now let's try to dissect it step-by-step and see if we can see how everything
works. (You can see the documentation for all the bytecode operations
[here](https://docs.python.org/2/library/dis.html#python-bytecode-instructions))

- `BUILD_LIST` creates a new list with 0 elements and pushes it onto the stack. So
the stack at the end of the call looks like this

```
[]
```

- `LOAD_GLOBAL` pushes the global function `range` onto the stack.

```
range
[]
```

- `LOAD_CONST` pushes 0 onto the stack

```
0
range
[]
```

- `LOAD_CONST` pushes 100 onto the stack

```
100
0
range
[]
```

- `CALL_FUNCTION 2` finally makes this ineresting. `CALL_FUNCTION` is used to
actually call the `range` function. It uses the 0 and 100 from the stack as
arguments for the `range` function. Finally, after the function has been called,
it pushes the return value on the stack, so after calling the function, the
stack looks like this.

```
[0, ..., 99]
[]
```

- `GET_ITER` implements `TOS = iter(TOS)`, where TOS stands for "top of stack".
So this operation takes the list returned by `range` and replaces it with an
iterator object on the stack.

```
iter([0, ..., 99])
[]
```

Now we begin the actual iteration part and we can start building the list

- `FOR_ITER` calls the `next` method of the iterator that is currently on top of
the stack and pushes that value on the stack.

```
0
iter([0, ..., 99])
[]
```

- `STORE_FAST` stores the top of the stack value in `co_varnames[0]`--in a
variable named `x`

```
iter([0, ..., 99])
[]
```

- `LOAD_FAST` pushes a *reference* from `co_varnames` onto the stack

```
&x
iter([0, ..., 99])
[]
```

- `LOAD_CONST` pushes 2 onto the stack

```
2
&x
iter([0, ..., 99])
[]
```

- `BINARY_MULTIPLY` multiples the two uppermost elements on the stack and
pushes the resulting value onto the stack. So in this case, it multiplies 2 with
\&x where &x is dereferenced from `co_varnames[0]`. From the `STORE_FAST`
operation earlier, `co_varnames[0]` contains 0.

```
0
iter([0, ..., 99])
[]
```

- `LIST_APPEND` is used for list comprehensions (surprise surprise!) and stores
the topmost element on the stack in the list we created earlier with `BUILD_LIST`.

```
iter([0, ..., 99])
[0]
```

- `JUMP_ABSOLUTE` jumps to the `FOR_ITER` call and we repeat until the iterator
is exhausted. Once it's exhausted, we pop the iterator and return the list.

Whew! that was quite the whirlwind, but aside from pushing values onto the stack
to use them as arguments to certain function calls, all the operations are
pretty straight-forward and we can see how it all comes together to build a new
list. The first thing that was done was creating a list from the `range`
function, once we had the list, we created an iterator from it and after we had
the iterator, we were able to begin getting elements from it and multiplying
them by two and storing them in the list that would eventually be returned. Once
the iterator is empty, all we do is return the list and we're done.

## Dissecting the Bytecode for For Loops
Similar to what we did with the list comprehension, let's write an equivalent
function but use the `for ... in` construct instead

```python
def list_iter():
    xs = []
    for x in range(0, 100):
        xs.append(x * 2)

    return x
```

And here's the annotated bytecode produced by `dis`

```
          0 BUILD_LIST               0          # pushes [] onto the stack
          3 STORE_FAST               0 (xs)     # stores the list in co_varnames[0]

          6 SETUP_LOOP              40 (to 49)  # pushes the for loop onto the block stack
          9 LOAD_GLOBAL              0 (range)  # pushes `range` onto the stack
         12 LOAD_CONST               1 (0)      # pushes 0 onto the stack
         15 LOAD_CONST               2 (100)    # pushes 100 onto the stack
         18 CALL_FUNCTION            2          # calls `range(0, 100)`
         21 GET_ITER                            # gets the iterator from the list
>>   22 FOR_ITER                23 (to 48)  # calls `next` on the iterator and pushes the value on the stack
         25 STORE_FAST               1 (x)      # stores the element from the iterator at co_varnames[1]

         28 LOAD_FAST                0 (xs)     # pushes co_varnames[0] (xs) onto the stack
         31 LOAD_ATTR                1 (append) # pushes the `append` function onto the stack
         34 LOAD_FAST                1 (x)      # pushes a reference from co_varnames[1] onto the stack
         37 LOAD_CONST               3 (2)      # pushes 2 onto the stack
         40 BINARY_MULTIPLY                     # multiplies the two top variables and stores the result on the stack
         41 CALL_FUNCTION            1          # calls `append` with xs and the result from the previous call
         44 POP_TOP                             # pops xs from the stack
         45 JUMP_ABSOLUTE           22          # continues the iteration...
>>   48 POP_BLOCK                           # pops the for block from the block stack

>>   49 LOAD_FAST                0 (xs)     # pushes xs back onto the stack
         52 RETURN_VALUE                        # returns the value at the top of the stack
```

The bytecode is obviously longer and there's some new operations in there, but
we can also see parts from the list comprehension bytecode. Nevertheless, let's
go through it step-by-step.

- `BUILD_LIST` creates a new list with 0 elements and pushes it onto the stack.

```
[]
```

- `STORE_FAST` stores the value at the top of the stack at `co_varnames[0]`, in
this case it's named `xs`

```
empty
```
- `SETUP_LOOP` pushes a block of code onto the *block* stack

```
empty
```

- `LOAD_GLOBAL` pushes the `range` function onto the stack

```
range
```

- `LOAD_CONST` pushes 0 onto the stack

```
0
range
```

- `LOAD_CONST` pushes 100 onto the stack

```
100
0
range
```

- `CALL_FUNCTION` calls `range(0, 100)` and pushes the resulting list onto the
stack

```
[0, ..., 99]
```

- `GET_ITER` gets an iterator object from the list at the top of the stack

```
iter([0, ..., 99])
```

- `FOR_ITER` calls `next` on the iterator object from the previous operation and
stores it on the stack

```
0
iter([0, ..., 99])
```

- `STORE_FAST` stores the value at the top of the stack at `co_varnames[1]`

```
iter([0, ..., 99])
```

- `LOAD_FAST` pushes a *reference* of `xs` onto the stack

```
&xs
iter([0, ..., 99])
```

- `LOAD_ATTR` replaces the top value of the stack with an attribute function. In
this case `xs.append`

```
xs.append
iter([0, ..., 99])
```

- `LOAD_FAST` pushes a reference to `x` onto the stack

```
&x
xs.append
iter([0, ..., 99])
```

- `LOAD_CONST` pushes 2 onto the stack

```
2
&x
xs.append
iter([0, ..., 99])
```

- `BINARY_MULTIPLY` multiplies 2 and 0 and stores it on the stack

```
0
xs.append
iter([0, ..., 99])
```

- `CALL_FUNCTION` calls `xs.append(0)`, pops 0 from the stack, and pushes the
return value onto the stack--in this case `None`

```
None
iter([0, ..., 99])
```

- `POP_TOP` simply pops the top element from the stack

```
iter([0, ..., 99])
```

- `JUMP_ABSOLUTE` goes back to the `FOR_ITER` operation and begins the whole
thing again

This makes the list comprehension's bytecode pale in comparision, and after
going through the bytecode, we can see that the this version has to do many
more things such as pushing the for block onto the block stack, pushing `xs`
onto the stack just to load the list attribute `append`, and popping `None` from
the stack after the call to `append`.

## Using Map
Maybe having to push `xs` onto the stack and popping it off the stack in the for
loop version is the reason why it takes double the time as the list
comprehension, so for completeness, let's look at one more way to build up a
list: using `map`

```python
map(lambda x: x * 2, range(0, 100))
```

Now let's time it

```
$ python -m timeit -n 100000 "map(lambda x: x * 2, range(0, 100))"
100000 loops, best of 3: 23.9 usec per loop
```

Ok, so using `map` takes longer than the list comprehension and is roughly
equivalent to the for loop version, let's get the bytecode and see what
happens behind the scenes

```
 0 LOAD_GLOBAL              0 (map)
 3 LOAD_CONST               1 (<code object <lambda> at 0x104ff16b0, file "<stdin>", line 2>)
 6 MAKE_FUNCTION            0
 9 LOAD_GLOBAL              1 (range)
12 LOAD_CONST               2 (0)
15 LOAD_CONST               3 (100)
18 CALL_FUNCTION            2
21 CALL_FUNCTION            2
24 RETURN_VALUE
```

The bytecode is actually *shorter* than any other version, so that makes the
hypothesis that the number of operations affects the run time and more
operations means more time false and untenable.

## Conclusion
I'm not claiming to be a master of Python bytecode at all, but after dissecting
three different ways to build up a list, I suspect the `LIST_APPEND` operation
to be the reason behind the list comprehension being faster than the other
versions. By using `LIST_APPEND` the list comprehension avoids calling
`LOAD_ATTR` and `CALL_FUNCTION` like the for loop. The full documentation for
`LIST_APPEND` is

> Calls `list.append(TOS[-i], TOS)`. **Used to implement list comprehensions.**
> While the appended value is popped off, the list object remains on the stack
> so that it is available for further iterations of the loop.
(emphasis added)

Which leads me to think that `LIST_APPEND` is an optimization used to make list
comprehensions faster.

All of these thoughts come with an asterisk of course. I have no idea if this is
the reason why list comprehensions are faster, but I definitely suspect
`LIST_APPEND` to be at least part of the reason.
