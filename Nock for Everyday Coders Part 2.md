# Nock for Everyday Coders, Part 2: the Rest of Nock

## Detour: Odds and Ends from the [Prior Section](Nock for Everyday Coders Part 1.md)


### Use of the Quoter/Constant Function, `1`
- heavy use
- because

### How to CRASH EVERYTHING
To create a crash in Nock, just access a non-existent memory slot--the obvious one being slot 0.
```
~zod:dojo> .*(50 [0 0])
ford: %ride failed to execute:
```
Why would you do this? One example would be an if statement that crashes if the input data is in the wrong format. 
For example, the `?` mold in Hoon returns 0 if its input is `%.y`, 1 if `%.n`, and crashed on any other value. Here is its code,
taken directly from the battery in the dojo:
```
[6 [5 [1 0] 0 6] [1 0] 6 [5 [1 1] 0 6] [1 1] 0 0]
```
We will see the `6` function right below, but for now, just look at the very end: it's a `[0 0]` that gets called if all the "if"
tests fail.

## Hoon Connections
* Why we always use the `[battery payload]` structure? It arises organically from adding to the front of the list
  - if we added to the first element, battery would keep getting buried deeper
  - if we appended to the end of payload, it would be too expensive
* Why does Hoon resolve "left"-first?
  - because new variables attach to the front ("left") of the payload
* How are we making new subjects? Just use `2`
