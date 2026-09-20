# Random Module in Python

`random` is a Python standard library module for generating **pseudorandom** numbers, selecting items, and rearranging sequences. Pseudorandom values come from an algorithm with internal state: they appear random, but a controlled starting state lets you reproduce a sequence of results.

The module is useful for simulations, games, test data, and randomized algorithms. Its default generator is not suitable for security tokens or passwords; Python provides `secrets` for those uses.

## Generating Numbers

### Integers: `randint()` and `randrange()`

Both select uniformly among the allowed integers, meaning each allowed integer has the same chance of being returned.

| Function | Possible results |
| --- | --- |
| `random.randint(a, b)` | Integers from `a` through `b`, **including both endpoints** |
| `random.randrange(stop)` | Integers from `0` up to, but excluding, `stop` |
| `random.randrange(start, stop, step)` | Values from `range(start, stop, step)` |

For `randrange()`, `start` defaults to `0` and `step` defaults to `1`. Like `range()`, the stop value is excluded. `randint(a, b)` is equivalent to `randrange(a, b + 1)`.

```python
import random

roll = random.randint(1, 6)         # One of: 1, 2, 3, 4, 5, 6
same_range = random.randrange(1, 7)  # Same possible values; a separate draw
even_roll = random.randrange(2, 7, 2)  # One of: 2, 4, 6
die_index = random.randrange(6)    # One of: 0, 1, 2, 3, 4, 5

print(roll, same_range, even_roll, die_index)  # Results vary
```

Arguments must be integers. An empty range, such as `randrange(0)` or `randint(6, 1)`, raises `ValueError`; a zero step also raises `ValueError`.

### Floating-Point Values: `random()` and `uniform()`

`random.random()` produces a float in **`[0.0, 1.0)`**. It can return `0.0`, but cannot return `1.0`.

`random.uniform(a, b)` produces a float between the two bounds. Floating-point rounding means the second endpoint may or may not be returned, so do not treat it as strictly excluded.

```python
chance = random.random()
bonus = random.uniform(0.5, 2.0)

print(chance)  # 0.0 <= chance < 1.0
print(bonus)   # Between 0.5 and 2.0

# Trigger an event with probability approximately 25% per trial.
bonus_awarded = chance < 0.25
print(bonus_awarded)  # True or False
```

## Selecting Items

The selection functions below work with **sequences**, such as lists, tuples, strings, and ranges. Convert a set or dictionary view to a sequence first; for example, `list(mapping)` gives a list of dictionary keys.

### One Item: `choice()`

`random.choice(sequence)` returns one element without changing the sequence. Each position is equally likely, so repeated values occupy multiple chances to be selected.

```python
players = ["Asha", "Ben", "Chen", "Dia"]

starter = random.choice(players)
print(starter)  # One player name, such as 'Chen'
print(players)  # ['Asha', 'Ben', 'Chen', 'Dia']: unchanged

# 'Asha' occupies two of the three positions, giving her a 2/3 chance.
weighted_starter = random.choice(["Asha", "Asha", "Ben"])
```

An empty sequence raises `IndexError`. `choice()` does not remove the selected item, so repeated calls can return the same item.

### Multiple Items: `choices()` and `sample()`

Both return a new list and leave the original sequence unchanged. Their difference is whether an input position remains eligible after being selected:

| Function | Selection rule | Size constraint |
| --- | --- | --- |
| `random.choices(population, k=k)` | **With replacement:** the same position can be selected repeatedly | `k` can exceed the population size |
| `random.sample(population, k)` | **Without replacement:** each position can be selected at most once | `0 <= k <= len(population)` |

With replacement means every draw considers the full population again. Without replacement means each selected occurrence is excluded from the remaining draws within that call.

```python
# Continue with the four players.
turns = random.choices(players, k=6)
team = random.sample(players, k=2)

print(turns)  # Six names; repeats are allowed
print(team)   # Two different names because players contains unique names
print(players)  # ['Asha', 'Ben', 'Chen', 'Dia']: unchanged

# sample() does not deduplicate equal values at different positions.
print(random.sample(["Asha", "Asha"], k=2))  # ['Asha', 'Asha']
```

`choices()` defaults to `k=1` and still returns a **list**, whereas `choice()` returns a single element. For positive `k`, `choices()` needs a nonempty population. `sample()` raises `ValueError` if `k` is negative or exceeds the available population size.

#### Weighted Selection with `choices()`

The optional `weights` argument changes the relative chance of selecting each position. Weights align with the population by index, so **`weights` must have the same length as `players`**, with one weight per player. Different lengths raise `ValueError`. The weights do not need to add up to `1`.

```python
turns = random.choices(players, weights=[4, 2, 1, 1], k=6)
print(turns)  # Asha is more likely, but any particular result can vary
```

The weights total `8`, so each draw gives Asha probability `4/8`, Ben `2/8`, and Chen and Dia `1/8` each. Six draws are not guaranteed to match these proportions.

Weights must be finite and nonnegative, with at least one positive weight. A zero weight excludes that position from selection.

## Shuffling a Sequence

`random.shuffle(sequence)` rearranges a mutable sequence **in place** and returns `None`.

```python
# Preserve players by shuffling a copy.
turn_order = players.copy()
result = random.shuffle(turn_order)

print(turn_order)  # The same four names, in a random order
print(result)     # None
print(players)    # ['Asha', 'Ben', 'Chen', 'Dia']: unchanged

# Another way to obtain a shuffled copy, also usable with a tuple or string:
another_order = random.sample(players, k=len(players))
```

Shuffling changes positions, not the items or their counts. The resulting order can happen to match the original order. Do not assign `turn_order = random.shuffle(turn_order)`, because that would replace the list variable with `None`.

## Reproducing Results

### Setting the Starting State with `seed()`

`random.seed(value)` initializes the module's generator. Reusing the same seed and the same sequence of calls reproduces results in the same Python environment, provided no other code changes that generator's state between calls.

```python
random.seed(42)
first_rolls = [random.randint(1, 6) for _ in range(5)]

random.seed(42)
repeated_rolls = [random.randint(1, 6) for _ in range(5)]

print(first_rolls == repeated_rolls)  # True
```

Seed once before a run. Seeding with the same value before every draw restarts the generator each time instead of advancing through its sequence. Without an explicit seed, Python initializes the generator automatically.

Functions such as `randint()`, `choice()`, and `shuffle()` use the module's shared generator. Adding an extra call can therefore change later results. Do not assume every function produces identical seeded results across Python versions.

### Independent Generators with `Random`

`random.Random(seed)` creates a generator with its own state and the same familiar methods. Use one when a simulation or test needs reproducibility without changing the module's shared generator.

```python
game = random.Random(42)
replay = random.Random(42)

game_rolls = [game.randint(1, 6) for _ in range(5)]
replay_rolls = [replay.randint(1, 6) for _ in range(5)]
print(game_rolls == replay_rolls)  # True

game.choice(players)  # Advances only game's state

# replay still advances exactly as it would without the call above.
next_replay_roll = replay.randint(1, 6)
```
