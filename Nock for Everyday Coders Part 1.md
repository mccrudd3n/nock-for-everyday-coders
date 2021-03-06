## Table of Contents
1. [Getting Started and Confusing Points](#getting-started--confusing-points)
2. [Nock's Simplest Functions, 0 and 1](#nocks-simplest-functions-0-and-1)
3. [4, the Incrementing Function](#4-the-incrementing-function)
4. [Cell-Maker (aka the Distribution Rule)](#the-cell-maker-aka-the-distribution-rule)
5. [3 and 5, the "Is This a Cell?" and "Equality Test" Functions](#3-and-5-the-is-this-a-cell-and-equality-test-functions)
6. [2, the "Subject-Altering" Function](#2-the-subject-altering-function)
7. [Summary and First Hoon Connections](#summary-and-first-hoon-connections)
8. [End of Part 1](#end-of-part-1)

## Getting Started--Confusing Points
When people first look at Nock, they see the [definition page](https://urbit.org/docs/tutorials/nock/definition/), which, tbh, is fairly intimidating.

I'm talking about lines like this in the spec:
```
/[(a + a) b]        /[2 /[a b]]
```

The problem is, the programmer already probably knows that Nock code he's seen looks more like the below--just lists of numbers, with no symbols:
```
[6
  [5 [0 6] [1 0]]
  [0 7]
  [9 4 [[0 2] [2 [0 6] [0 5]] [4 0 7]]]
]
```

So what gives? Which one is the "real" Nock?

### Point of Confusion
The key thing to understand is that there are **two different audiences** here:
1. People who want to write interpreters for Nock
2. People who want to write Nock code

The symbols and spec are for the former. They are pseudocode, **not real Nock code**. They could just as easily be written in English, and they **will never be written down as actual Nock code and given to an interpreter**. We'll use the pseudocode as a substitute for English below where it's helpful, but it will always be clearly separated from actual Nock code.

The lists of numbers are the **actual Nock code**. That is what we will be focusing on, and we'll use a combination of English and the pseudocode symbols (when helpful) to understand them.

### How Nock Works
It's simple.
1. You feed a list of numbers to a working Nock interpreter
2. The interpreter runs it
3. The interpreter produces a noun as output. A noun can be
```
:: a number
782
:: a cell (pair with two elements)
[782 9872]
:: each element can itself be a pair
[782 [9872 89728]]
:: the above can be written as
[782 9872 89728]
```

### How to Run Nock Code
The Urbit dojo has a Hoon function that runs Nock code (i.e. a Nock interpreter). To use it:
1. Start up a dojo session (see [here](https://urbit.org/using/develop/#creating-a-development-ship) for how to create a fake ship)
2. At the prompt, we can execute Nock with `.*(NOCK_SUBJECT, NOCK_FORMULA)`

### Subject, Formula?
Let's keep this simple:
* subject = an argument to a function
* formula = the function

That's it. We'll see below how this works, going really slowly with examples.

### Evaluating Our First Nock Code
OK, so the interpreter takes two arguments, a "subject" and a "formula". Both are nouns (a number or a cell). Let's run some insanely simple Nock code and see how that works.
```
~zod:dojo> .*(42 [0 1])
42
```
In the above, `42` is our subject. `[0 1]` is our formula.

Formulas are always cells, and the first element of the cell is a number that you can think of as ***the name of the function***.

In this case, our function name is `0`, which is the memory slot function. It is always followed by 1 number, in this case `1`, which is the number of the memory slot to fetch in the subject.

Whenever we look at Nock code, we want to ask:
* What is the subject (function argument)? In this case, it's `42`.
* What is the formula (function)? In this case, it's `[0 1]`.
* What value does that formula (function) produce when called on this subject (argument)? In this case, the return value is `42`.

Why is the return value `42`? How does this formula work?

## Nock's Simplest Functions, `0` and `1`
The two most basic Nock functions are `0` and `1`. The goal here is to get strong intuitions of what they do, how they handle edge cases, and how this relates to the Nock spec/pseudocode.

### `0`, the "Memory Slot" Function
*note: the below examples are adapted/yoinked from the [Nock Explanation](https://urbit.org/docs/tutorials/nock/explanation/) in the "Tree Addressing" section.*

Every noun in Nock can be thought of as a tree, which means we can give an exact number to access any position in the tree. This means that, no matter how big our subject (argument) is, we can yank a value out of any part of it.

How do you say which slot number you want from the tree?
* 1 is the tree root
* The head of every node n is 2n
* the tail is 2n+1

Let's take an example tree to illustrate. In Nock cell form, the tree is:
```
[[4 5] [6 14 15]]
```
Drawn as ASCII art (this is **not** real Nock code), the tree looks like:
```
     1
  2      3
4   5  6   7
         14 15
```
So
* `1` is the address of the whole tree, `[[4 5] [6 14 15]]`
* `2` is the address of the left branch, `[4 5]`
* `3` is the address of the right branch, `[[4 5] [6 14 15]]`
* `15` is the value `15`

Let's play around now in the dojo Nock interpreter so that we can confirm this. In each example, our subject (argument) will be the tree `[[4 5] [6 14 15]]`.
```
:: formula (function) [0 1]: get the whole tree
~zod:dojo> .*([[4 5] [6 14 15]] [0 1])
[[4 5] 6 14 15]

:: formula (function) [0 2]: get the left branch
~zod:dojo> .*([[4 5] [6 14 15]] [0 2])
[4 5]

:: formula (function) [0 7]: get the subtree in slot 7
~zod:dojo> .*([[4 5] [6 14 15]] [0 7])
[14 15]
```
Feel free to experiment with different slots in the formula `[0 slot]`.

#### Important Note
The memory slot number **cannot** be a nested function. It **must be just an atom.** The following will crash:
```
~zod:dojo> .*([51 2] [0 [0 3]])
ford: %ride failed to execute:
```
Why? Because instead of a number as the slot, we give the cell `[0 3]`. You might think that that would first grab the slot `3` in the subject, and then pass that value (`2`) to the top-level, making the expression `.*([51 2] [0 2])`. However, this is **not** how it works. Why?

This is a good time to look at the Nock symbols/pseudocode to see how opcode `0` is defined.
```
*[a 0 b]  /[b a]
```
Here `a` is the subject, `b` is the formula, and `/[b a]` is pseudocode that means "grab the memory slot `b` in subject `a`." The key thing here is that `b` is just `b`--it's not `*(b)`, which is pseudocode for "run the Nock interpreter on this expression again.

So because the interpreter isn't run again on `b`, it **must** be just a number, otherwise the instruction to "get the subject value at this memory slot" makes no sense.

### Summary of `0`
`0` is **everywhere** in Nock, because "get something from a memory slot" really means "store stuff in a place and get it whenever I want," which is another name for creating variables. These memory slot numbers take the place of variable names.

If you're familiar with assembly or C, they're conceptually similar to memory pointers or registers in how Nock uses them. Keep in mind though, Nock is functional/immutable, so it doesn't update memory locations: it creates copies of data structures with altered values.

### `1`, the "Quoter" Function
This is another really simple function. You can think of it as a "quoter": it just returns any value passed to it **exactly as it is**. It ignores the subject, and just quotes the value after it. Let's look at a couple examples.
```
~zod:dojo> .*([20 30] [1 67])
67
:: [1 2 587] is same as [1 [2 [587]]]
~zod:dojo> .*([20 30] [1 [2 [587])
[2 587]
~zod:dojo>
```
It doesn't matter how much information is after the `1`: `1` is a dumb function that just returns it all.

The pseudocode for `1` is:
```
*[a 1 b]  b
```

In English, this means: "ignore the subject `a`, and just return everything after the `1` **exactly as it is**.

Let's look at our first example, `.*([20 30] [1 67])`. The subject `a` is `[20 30]`, so we ignore that. What's after the `1`? `67`, so we return that.

In the second example, "everything after the `1`" is longer, but the same rule applies: the Nock interpreter just returns it exactly as it is, after stripping out unnecessary brackets.

## `4`, the Incrementing Function
`0` and `1` are simple functions that don't have any nested behavior. Now we're going to move to a function that **does** have nested children.

Here are code examples of opcode `4`, and then we'll look at its pseudocode and break down the examples.
```
:: example 1
~zod:dojo> .*(50 [4 0 1])
51

::example 2
~zod:dojo> .*(50 [4 4 0 1])
52

:: example 3
~zod:dojo> .*([100 150] [4 4 0 3])
152

:: example 4
~zod:dojo> .*(50 [4 1 98])
99

::example 5
~zod:dojo> .*(50 [4 1 [0 2]])
ford: %ride failed to execute:
```

Here's `4`'s pseudocode, juxtaposed with that of `0` and `1`:
```
:: 4
*[a 4 b]  +*[a b]

:: 0
*[a 0 b]  /[b a]

:: 1
*[a 1 b]  b
```
In English, this says "when we have subject `a`, function `4`, and Nock code `b`, first evaluate `[a b]` as `[subject formula]`, and then add 1 to the result."

If we contrast with `0` and `1`, we see that the right side has a `*` symbol. This symbol means "evaluate the expression again in the Nock interpreter." `0` and `1` **did not** have this symbol, and that's why they couldn't evaluate nested Nock expressions.

### Example 1 Breakdown
```
.*(50 [4 0 1])
```
Let's start by translating the dojo's `.*(subject formula)` to pseudocode of the form `*[a b]`, and then expand it line by line:
```
:: change to pseudocode
*[50 [4 0 1]]
:: move the 4 to the outside as +
+*[50 [0 1]]
:: run the 0 function to get memory slot 1
+(50)
:: evaluate
51
```

The `[a b]` part of `+*[a b]` expands to:
```
[50 [0 1]]
```
OK, this we know how to handle! It's just our `0` function, and it wants the value in memory slot `1` of the subject. That's `50`.

So now we that `*[a b] = 50`, and we just have to add 1 to it (the `+` in `+*[a b]`). That is `51`, which is exactly what the interpreter gave us.

### Example 2 Breakdown
This one is similar, we just have an extra `4`. We again start by translating the dojo's `.*(subject formula)` to pseudocode, and then expand
```
*[50 [4 4 0 1]]
:: the first 4 moves outside as a '+'
+*[50 [4 0 1]]
:: 2nd 4 becomes a '+'...OK, we have the 0 function again!
++*[50 [0 1]]
:: run the 0 function to get memory slot 1
++(50)
+(51)
52
```

### Example 3 Breakdown
Here we again see lots of `4`s applied consecutively, and we also see how we can yank values out of a more complicated subject and manipulate them. Notice how the subject is a cell, not an atom. The rest is the same as in Example 2.
```
a (the subject) = [100 150]
*[[100 150] [4 4 0 3]]
+*[[100 150] [4 0 3]]
:: Now we've extracted all the increments ("+" signs)
:: So we just grab the value at memory slot 3 ([0 3] command)
++*[[100 150] [0 3]]
++(150)
+(151)
152
```

### Example 4 Breakdown
In the below example, we see how we can use the quote/constant function `1` to generate the value `98` and increment it. We ignore the subject `50` completely.
```
*[50 [4 1 98]]
:: formula is the "quoter" function: [1 98]
+*[50 [1 98]]
+(98)
99
```

### Example 5 Breakdown
Just as opcode `0` had values it couldn't handle (non-atoms), so opcode `4` needs the nested value inside it to evaluate to an atom.
```
*[50 [4 1 [0 2]]]
:: OK cool, the nested value is a 1 opcode, let's see what happens
+*[50 [1 [0 2]]
:: 1 ignores the subject (50) and just returns [0 2]
+([0 2])
:: oh crap, [0 2] isn't an atom, how can we increment it???
ford: %ride failed to execute:
```

#### Summary
In these examples, we've seen that function `4` can be called as many times in a row as we want. At the end of those calls, it always ends up incrementing a number that either is yanked from the subject (*memory slot function* `0`) or quoted as it is (*quote function* `1`).

## the Cell-Maker (aka the Distribution Rule)
The Nock interpreter is allowed to return nouns, which are atoms (positive numbers) or cells (pairs of nouns).
What have our functions/opcodes been returning so far?
* `0`: atoms or cells, depending what's in the memory slot that we yoink
* `1`: atoms or cells, depending on what we quote
* `4`: just atoms

But what if my subject was `[51 67 89]`, and I wanted to increment every value and return that as `[52 68 90]`? How can I do that when it's a cell, and `4` only seems able to return atoms?

The answer is something that the Nock docs call the "Distribution Rule" or "implicit cons" (hello, fellow LISPers!), but that I find easiest to think of as the "Cell-Maker Rule."

### A Quick Detour into Nock Formulas
We haven't talked much yet about what values are legal to feed into the Nock interpreter (the `.*(subject formula)` function in the dojo). So far, we've only been using formulas that start with numbers (our functions/opcodes `0`/`1`/`4`).

Let's now fully solidify our understanding of what's a legal formula by taking a quick look at the 3 possible cases (*the format is`.*(subject formula)`*):
```
:: formula is a cell starting with an atom--works; we knew this
~zod:dojo> .*(50 [0 1])
50

:: formula is just an atom (0)--error, not valid
~zod:dojo> .*(50 0)
ford: %ride failed to execute:

:: now we do: .*(subject [cell1 cell2])
:: so...formula is a cell that starts with a cell...wtf, this works?
~zod:dojo> .*(50 [[0 1] [1 203]])
[50 203]
```

### The Cell-Maker in All His Glory
So apparently a formula cell can start with a cell. The Cell-Maker rule is (from the Nock docs):
```
*[subject [formula-x formula-y]]
=>  [*[subject formula-x] *[subject formula-y]]
```

In our example above, `formula-x` is `[0 1]`, and `formula-y` is `[1 203]`. They each evaluate individually against the subject, and the end result is a cell.

### Using Our New Powers

We can make as many cells in a row as we want!
```
~zod:dojo> .*(50 [[0 1] [1 203] [0 1] [1 19] [1 76]])
[50 203 50 19 76]
```

We can put any operation inside each cell!
```
:: [19 20] is our subject, just to be clear
:: the rest is formulas you've seen before
~zod:dojo> .*([19 20] [[0 1] [1 76] [4 4 0 3]])
[[19 20] 76 22]
```
If we take the returned collection `[[19 20] 76 22]` in order, we can write in English how they connect to our collection of formulas that we passed:
* `[19 20]`: `[0 1]`, get memory slot 1
* `76`: `[1 76]`, return the quoted value `76`
* `22`: `[4 4 0 3]`, increment twice the value in memory slot 3 (`20`)

So we can pass one small subject (`[19 20]`) and make an arbitrarily long collection of values from it, using any functions we want. Cell-Maker ftw!

## `3` and `5`, the "Is This a Cell?" and "Equality Test" Functions
Now we come to functions/opcodes `3` and `5`, which are pretty straightforward after we've seen how `4` and the Cell-Maker work.
Functions `3` and `5`, like `4`, allow nested evaluation. Let's put all their pseudocode definitions together to compare:
```
::  function/opcode 3
*[a 3 b]  ?*[a b]
:: function/opcode 4
*[a 4 b]  +*[a b]
:: function/opcode 5
*[a 5 b c]  =[*[a b] *[a c]]
```
First of all, notice how the right side of all these "equations" has the evaluation operator, `*`. This means that these functions can have nested formulas, since they keep evaluating all the way down.

There are some new pseudocode symbols here that we need to translate into English. We already know `+`: "increment the value after this". Now we also see:
* `?`: "check whether the value after this is a cell. Return 0 if yes, 1 if no"
* `=`: "first run the function in `b` with subject `a` as the argument, and same for the function in `c`. If the results are equal, return 0, if not, return 1.

### Dojo + Pseudocode Examples of `3`, the "Is-This-A-Cell?" Function
Example 1 -- not a cell
```
~zod:dojo> .*(50 [3 0 1])
1
:: PSEUDOCODE OF THE ABOVE
*[50 [3 0 1]]
?*[50 [0 1]]
::get memory slot 1 of the subject
?(50)
:: is 50 a cell? No, so return 1
1
```
Example 2 -- yes, this is a cell
```
~zod:dojo> .*([50 51] [3 0 1])
0
:: PSEUDOCODE OF THE ABOVE
*[[50 51] [3 0 1]]
?*[[50 51] [0 1]]
::get memory slot 1 of the subject: [50 51]
?([50 51])
:: is [50 51] a cell? Yes, so return 0
0
```
Example 3 -- nested evaluation with one of the other `3`/`4`/`5` functions/opcodes
```
~zod:dojo> .*([50 51] [4 4 3 0 1])
2
:: PSEUDOCODE OF THE ABOVE
*[[50 51] [4 4 3 0 1]]
+*[[50 51] [4 3 0 1]]
:: whatever comes out of the 3 function, we're gonna increment twice
++*[[50 51] [3 0 1]]
:: down to just fetching memory slot 1
++?*[[50 51] [0 1]]
++?([50 51])
:: is [50 51] a cell? Yes, so return 0
++(0)
+(1)
2
```
Example 4 -- check whether multiple things are cells using Cell-Maker
```
~zod:dojo> .*([[50 51] 52] [[3 0 2] [3 0 3]])
[0 1]
*[[[50 51] 52] [[3 0 2] [3 0 3]]]
?[*[[50 51] 52] [0 2]]
   *[[50 51] [0 3]]]
:: yank memory slots 2 and 3
?*[[50 51] 52]
:: first is a cell, second is not
[0 1]
```

### Dojo +  Pseudocode Examples of `5`, the "Equals" Function
Because `5` compares the results of 2 formulas, it **always** makes 2 inner evaluations of the subject. It's similar to Cell-Maker in this way.

Example 1 -- compare two equal values
```
~zod:dojo> .*([50 51] [5 [0 2] [0 2]])
0
:: PSEUDOCODE
*[[50 51] [5 [0 2] [0 2]]]
:: factor out the =
=[*[[50 51] [0 2]] *[[50 51] [0 2]]]
:: get memory slot 2 twice
=(50 50)
0
```
Example 2 -- compare two unequal values
```
~zod:dojo> .*([50 51] [5 [0 2] [0 3]])
1
:: PSEUDOCODE
*[[50 51] [5 [0 2] [0 3]]]
:: factor out the =
=[*[[50 51] [0 2]] *[[50 51] [0 3]]]
:: get memory slot 2 and memory slot 3
=(50 51)
1
```

Example 3 -- compare two values, one of which comes from a nested function
```
~zod:dojo> .*([50 51] [5 [4 0 2] [0 3]])
0
:: PSEUDOCODE
*[[50 51] [5 [4 0 2] [0 3]]]
:: factor out the =
=[*[[50 51] [4 0 2]]
   *[[50 51] [0 3]]]
:: factor out the +
=[+*[[50 51] [0 2]]
    *[[50 51] [0 3]]]
:: get memory slots 2 and 3
=[+50 51]
:: evaluate the +
=([51 51])
0
```

Example 4 -- `5` knows how to compare cells, not just atoms. Erotic.
```
~zod:dojo> .*([99 99] [5 [1 [99 99]] [0 1]])
0
:: PSEUDOCODE
*[[99 99] [5 [1 [99 99]] [0 1]]]
=[*[[99 99] [1 [99 99]]]
   *[[99 99] [0 1]]]
:: first cell is the result of quoter function (ignores subject)
:: second cell is the result of fetching memory slot 1 in the subject
=[[99 99] [99 99]]
:: true
0
```

## `2`, the "Subject-Altering" Function
In all our examples so far, the subject has been defined at the start when we call the interpreter, and never changes. But what if we want a different subject?


### Motivation, or "Why Would I Want a Different Subject?"

A different subject? Why would we want that? Here's an easy example. Say you found the following piece of Nock code on the [interwebz](https://urbit.org/docs/tutorials/nock/example/):
```
[8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1]
```
This is the code for a Nock function that expects a subject that is an atom, and decrements that subject by 1.  You could actually enter it in the Dojo right now:
```
~zod:dojo> .*(100 [8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1])
99
```
It works! You **do not have to understand the code right now**; just try entering different numbers instead of `100` to see the program working.

But now imagine that I have a Nock program with a different subject
```
~zod:dojo> .*([50 51] some-formula)
```
And somewhere inside `some-formula`, I want to decrement the number `51`.  I can't pass `[50 51]` as the subject to my decrementer code above though, because that's a cell, and it expects just an atom (a number).

### Subject Altering to the Rescue
The function/opcode `2` is designd to handle this problem for us. Here's the pseudocode, and then I'll explain what the pseudocode means.
```
:: PSEUDOCODE
*[a 2 b c]  *[*[a b] *[a c]]
```

`2` expects 2 formulas after the subject `a`: `b` and `c`. With those, it:
1. runs formula `b` against the subject to set up a new environment/subject derived from the subject
2. runs formula `c` against the subject to prepare a 2nd function.
3. run that 2nd function against the new environment/subject from step ***(1)***

Note that the pseudocode for `2` has nested `*`s.
```
*[*[a b] *[a c]]
```
The 2 inner `*`s run steps ***(1)*** and ***(2)***, and the outer one, around the whole expression, runs step ***(3)***.

### Examples

#### Example 1 — change the subject, have step ***(2)*** just call a constant value
```
~zod:dojo> .*([50 51] [2 [0 3] [1 [4 0 1]]])
52
:: PSEUDOCODE
*[[50 51] [2 [0 3] [1 [4 0 1]]]]
:: separate b and c to each run against the subject (steps 1 and 2)
*[*[[50 51] [0 3]] *[[50 51] [1 [4 0 1]]]]
:: after steps 1 and 2, we have a new subject, 51!
:: note how we're back in normal *[subject formula] form
*[51 [4 0 1]]
:: apply the 4 function as we're used to
+*[51 [0 1]]
:: grab 51 from memory slot 1
+(51)
52
```

#### Example 2 — grab a block of code from the subject in step ***(2)***, then run it in step ***(3)***.

```
~zod:dojo> .*([[4 0 1] 51] [2 [0 3] [0 2]]])
52
:: PSEUDOCODE, subject is [[4 0 1] 51]
*[[[4 0 1] 51] [2 [0 3] [0 2]]]]
*[*[[[4 0 1] 51] [0 3]
  *[[[4 0 1] 51][0 2]]
:: step 1 gets memory slot 3, step 2 grabs memory slot 2
*[51 [4 0 1]]
:: looks like a normal 4 opcode to me!
+*[51 [0 1]]
:: grab memory slot 1
+(51)
52
```

#### Example 3: Back to Our Motivating Case
Remember our decrementing block of code that we couldn't use when the subject was `[50 51]`, instead of just an atom? Opcode `2` makes handling that issue a piece of cake.  

We simply use `2` to transform our subject into an atom, and use `1` to quote the decrement block of code before it evaluates in step ***(3)***.
```
~zod:dojo> .*([50 51] [2 [0 2] [1 [8 [1 0] 8 [1 6 [5 [0 7] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1]]])
49
:: PSEUDOCODE (subject is [50 51])
:: I substitute "decrement-formula" for that block of Nock code for clarity
*[[50 51] [2 [0 2] [1 decrement-formula]]]
*[*[[50 51] [0 2]] *[[50 51] [1 decrement-formula]]]
:: 1, the quoting function, just returns the decrement-formula, like in Example 1
*[50 decrement-formula]
::decrement-formula has the atom subject that it wants now!
49
```

## Summary and First Hoon Connections
For those who know a bit of Hoon, Example 2 above is similar-ish to calling an arm that produces a gate, and then running the gate. Most of Hoon runs this type of code-getting + subject-altering Nock, and it all uses opcode `2` at its base.

And for those who know Hoon, `[[4 0 1] 51]` should already be looking a lot like `[battery payload]`...that's not a coincidence.

We're starting to see the first glimpses of how Hoon's cores, arms and subjects/subject mutations flow out of the fundamental structure of Nock.

We also see how Hoon/Nock lend themselves well to throwing around chunks of code, and adjusting the subject as necessary to create the correct subject/environment against which to run that code.

## End of Part 1

You now have seen all the fundamental functions/opcodes in Nock. In Part 2, I'll introduce the remaining functions, which no longer need pseudocode: we have enough scaffolding now to build the rest of Nock from Nock itself. Instead, these new opcodes will just be shortcuts/macros/code expansions of opcodes `1`-`5`.

If you already feel like you're comfortable with Nock now, you can skip my Part 2, and jump directly to the section of the [Nock Explanation](https://urbit.org/docs/tutorials/nock/explanation/) titled **Sugar** to see the rest of the opcodes explained.

We'll also start to connect Nock to Hoon, and see how most of the fundamental (and slightly weird) features of Hoon flow directly from Nock's structure, and make a lot more sense in combination with it.
