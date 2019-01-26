---
title: "Functional Turing Machine"
date: 2018-01-23T21:30:52+01:00
---

A week ago or so I managed to complete the [Advent of Code
2017](http://adventofcode.com/2017) and it was a really fun experience. Most of
the problems was really interesting(the spiral memory is probably my favorite),
but the problem for day 25 especially attracted me. Basically it was about
implementing a [Turing Machine](https://en.wikipedia.org/wiki/Turing_machine).

This is not rocket science since a Turing Machine is a simple idea(like all the
brilliant ideas) and a lot of ink has been spilled on it. However, since I was
using Haskell to solve the problem, I started to think at ways to model a Turing
Machine using functional programming.

## Quick refresher about Turing Machines

The Turing Machine was invented by Alan Turing and it consists of:

- an infinite tape of cells all initialized to 0
- a cursor that reads and writes data from the cell under it
- a state register which keeps track of which state the machine is in
- a table of instructions to execute according to the current state and the
  value of the current cell. An instruction can overwrite the value stored at
  the current cell, move the cursor left or right and eventually change the
  state of the machine.

It should come as no surprise that this model is at the foundation of imperative
programming since it's all about changing state and telling the processor how to
do things. Put it in another way: it is all about mutability.

## Naive mutable implementation

The easiest implementation of a Turing machine is probably to have a vector as
the tape, an index into that vector as the cursor and another variable as the
current state. For the sake of simplicity let's assume that the instructions are
known ahead of time and that we can model them as functions in the source code.

Here's some example python code that should make the idea crystal clear:

```python
class TuringMachine:
    def __init__(self):
      self.tape = [0] * 32
      self.cursor = 0
      self.state = 0 # could be anything really

    def example_instruction(self):
      if self.state == 0:
        self.tape[self.cursor] += 1
        self.cursor += 1
        self.state = 1
      else:
        self.tape[self.cursor] -= 1
        self.cursor -= 1
        self.state = 0

tm = TuringMachine()
tm.example_instruction()
print(tm.tape, tm.cursor, tm.state)
# ([1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0], 1, 1)

tm.example_instruction()
print(tm.tape, tm.cursor, tm.state)
# ([1, -1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0], 0, 0)
```

Dead simple yet working.

Mostly.

The main problem is that it ignores the fact that the tape is potentially
infinite. "No problem, I'll just wrap all the accesses to the current cell and
reallocate the vector when necessary" I hear you say. It surely works, but it
brings a new problem interesting on its own that is how to reallocate the tape.
You probably don't want to just insert a single item into the vector since the
overhead of reallocating and copying the vector each time would be bigger than
the benefits. Therefore we could allocate more space than what we actually need
to overcome this issue. Let's ignore the fact that in this way we loose the
possibility of knowing which cells were actually visited(without adding other
variables). A possible implementation could look like the following:

```python
def get_current_cell(self):
  if self.cursor < 0:
    self.tape = [0] * 32 + self.tape
    self.cursor = 32
  elif self.cursor >= len(self.tape):
    self.tape.extend([0] * 32)

  return self.tape[self.cursor]
```

This implementation has some problems though. First of all, the method is meant
to be a getter method(it's named `get_current_cell` after all), but it's
actually mutating the internal state of the Turing machine! This is certain
cause of bugs. Please don't do this. Ever.

Another concern is that this implementation is potentially wasteful in terms of
memory. Consider, for example, a program that steps to the left causing a
reallocation and then always moves to the right. It should be obvious that we'd
be storing data we won't need since we've allocated much more cells than what
were actually visited.

Last but not least, working with indexes is never fun and always error prone.

It looks like that a vector is not the best data structure to model a Turing
machine after all. In fact a
[`defaultdict(int)`](https://docs.python.org/3/library/collections.html) would
probably be a lot better in terms of clarity and even in terms of space
efficiency since we'd only store elements that are required(if you ignore the
fact that hashmaps do not occupy all the available slots internally). I'd be
quite satisfied with an implementation like this, but I feel we can do better
especially in a functional programming mindset.

## Functional Turing Machine

Let's take a moment and rethink about what the tape is and which operations we
can perform on it.

A good implementation should allow us to change the current cell's value and
move either to the left or right. It doesn't look like we need random access
operations at all since the position can move in sequential order only. Also, I
like living in a world without indices as much as possible since we all know
that off by 1 errors are extremely common :wink:.

After a bit of thinking, I realized that a tape can be seen as three parts: the
current cell, the cells to its left and the cells to its right. Moving basically
means pushing the current cell to either the left or right cells and replacing
the current cell with the last value of the other cells. Therefore, the left and
right cells can be seen as stacks since the last element pushed in is the first
element that's popped out(LIFO order). Given that stacks can support pushing and
popping in O(1) time then all the operations on the tape are O(1)! Nice!

Here is an Haskell implementation of this idea(note that the left stack is
actually in reverse order because lists in Haskell support O(1) deletes from the
head only):

```haskell
data Tape = Tape
 { _leftCells  :: [Int] -- reverse order
 , _curCell    :: Int
 , _rightCells :: [Int]
 } deriving (Show)

incrCur :: Tape -> Tape
incrCur t = t { _curCell = _curCell t + 1 }

decrCur :: Tape -> Tape
decrCur t = t { _curCell = _curCell t - 1 }

moveRight :: Tape -> Tape
moveRight (Tape l c (c':r)) = Tape (c : l) c' r

moveLeft :: Tape -> Tape
moveLeft (Tape (c':l) c r) = Tape l c' (c : r)
```

Note that all the functions return a new `Tape` object since Haskell is
~~mostly~~ an immutable language.

Accurate readers(and GHC itself) will complain that `moveRight` and `moveLeft`
are [partial functions](https://en.wikipedia.org/wiki/Partial_function). That's
only a fancy term that means that we're not considering the possibility that
either the right or left stack could be empty. Time to ask ourselves what we
should do in these cases. Well, the tape is actually infinite so an empty list
can be seen as full of 0.

Here are the final versions that support the infiniteness of the tape.

```haskell
moveRight :: Tape -> Tape
moveRight (Tape l c []    ) = Tape (c : l) 0 []
moveRight (Tape l c (c':r)) = Tape (c : l) c' r

moveLeft :: Tape -> Tape
moveLeft (Tape []     c r) = Tape [] 0 (c : r)
moveLeft (Tape (c':l) c r) = Tape l c' (c : r)
```

The compiler not only prevented us to include an obvious bug in the program, but
it actually forced us to implement a feature! This is super cool!

Now that we have an efficient model to represent a `Tape` it's not hard to write
the definition of a `TuringMachine`.

```haskell
data TuringMachine = TuringMachine
 { _state :: Int
 , _tape  :: Tape
 } deriving (Show)
```

The last thing we need to model are the instructions. In our simplified model
they are just pure functions that given a `TuringMachine` they return another
`TuringMachine`. However, it's not difficult to represent them as an
[ADT](https://en.wikipedia.org/wiki/Algebraic_data_type) and then to interpret
them given a `Tape`, but we'll keep it simple for now.

Here's the `example_instruction` from before implemented in Haskell along with
an example `main` function:

```haskell
exampleInstruction :: TuringMachine -> TuringMachine
exampleInstruction ma
  | _state ma == 0 = TuringMachine
    { _state = 1
    , _tape  = moveRight . incrCur . _tape $ ma
    }
  | otherwise = TuringMachine
    { _state = 0
    , _tape  = moveLeft . decrCur . _tape $ ma
    }

main :: IO ()
main = do
  let tm  = TuringMachine 0 (Tape [] 0 [])
  let tm' = exampleInstruction tm
  print tm'
  -- TuringMachine {_state = 1, _tape = Tape {_leftCells = [1], _curCell = 0, _rightCells = []}}

  print . exampleInstruction $ tm'
  -- TuringMachine {_state = 0, _tape = Tape {_leftCells = [], _curCell = 1, _rightCells = [-1]}}
```

## Conclusion

Am I saying that even problems that look inherently mutable are better modeled
using functional programming and immutability? Maybe. In this case I think so.

However, if you're looking for the most performant implementation of a Turing
machine then I think the mutable one is still more efficient mainly because
vectors can leverage the cache much more than lists can. To be completely fair,
GHC is actually able to optimize *a lot* and in fact the Haskell version is
faster than the mutable Python version, but I don't think it can beat a mutable
Rust implementation.

On the other hand, if you're interested in pure algorithm complexity, then the
functional Turing machine is, theoretically speaking, always O(1) while the
naive Turing machine one is O(1) amortized.

### Links

- my Advent of Code 2017 solutions: https://github.com/d-dorazio/hs-misc/tree/master/aoc17
