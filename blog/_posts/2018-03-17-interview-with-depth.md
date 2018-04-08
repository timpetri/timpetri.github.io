---
author: Tim Petri
title: An Interview question with a lot of depth
published: false
---

(Side note, this is my first blog post)

[When my friend Jeremy asked me about x, I immeditaly remembered bridgewater and thought i was gonna ace it]

The interview question is simple. Given a classic cell phone number pad and the Knight chess piece, return the number of possible phone numbers of length 10 and with a last digit of 0 that you can generate by making valid moves with the piece. 

Here is a valid sequence: `4040404040`

### Notice
* The key `5` is actually never used.
* From `4` and `6` the knight can reach three new keys.
* From the rest of the keys the knight can reach two new keys.

## Brute-force solution
The most brute-force solution here would be to basically step across the number path using a depth-first-search to discover the number of different paths. Start from 0 digit pad and keep track of how numbers are left in your sequence, stop when none left, and backtrack as necessary. Easy enough.

Our original function number can thus be expressed recursively as follows:
```python
def number_of_sequences(sequence_length, starting_digit):
  # Stop condition
  if sequence_length == 10: 
    return 1

  neighbor_digits = get_neighbors(starting_digit)

  result = 0
  for digit in neighbor_digits:
    result += number_of_sequences(sequence_length + 1, digit)

  return result

# Helper method specific to knight
def get_neighbors(digit):
  moves = {1: [6,8], 2: [6,9], 3:[4,8],
           4: [0,3,9], 5: [], 6:[0,7,1],
           7: [2, 6], 8: [1,3], 9: [2,4],
                      0: [4,6]}
  return moves[digit]
```

And we would get our solution by calling `number_of_sequences(10, 0)`

Cool. It's easy to realize that the run-time of this probably isn't great. And at this point we're also realizing that in the mess of all these function calls that only accept two parameters; `sequence_length`, and `starting_digit`, we're probably calling the exact same function signature more than once. Let's look at a picture that makes this clear.

![Function call tree](/blog/assets/function-calls.png)

## Top-down Memoization

In classic top-down memoization style, we don't let ourselves compute the same answer twice, and instead rely on a memory, like follows:

```python
def number_of_sequences(sequence_length, starting_digit, memory):

  # Check if this specific call has been made before
  call = (sequence_length, starting_digit)
  if call in memory:
    return memory[call]

  # Stop condition
  if sequence_length == 0: 
    return 1

  neighbor_digits = get_neighbors(starting_digit)

  result = 0
  for digit in neighbor_digits:
    result += number_of_sequences(sequence_length - 1, digit, memory)

  # Update memory and return result the first time it's computed
  memory[call] = result
  return result
```

Now we get our solution by providing our fuction with a memory: 
```python 
number_of_sequences(10, 0, {})
```

## Going deeper
Let's think about the fact that we were really only calling the function with two parameters, both of which are really limited. Our `sequence_length` is between 0 and 10, and our `starting_digit` is between 0 and 9. We can structure all our computed results in an m\*n matrix where m is the digits and n is the sequence length. 0 means that it was never computed.

$$
\begin{array}{c c c c c c c c c c}
    1 & 2 & 6 & 12 & 32 & 66 & 0 & 358 & 0 & 1934 \\ 
    1 & 2 & 5 & 10 & 26 & 53 & 0 & 285 & 0 & 0 \\ 
    1 & 2 & 5 & 11 & 27 & 0 & 145 & 0 & 0 & 0 \\ 
    1 & 2 & 5 & 10 & 0 & 53 & 0 & 285 & 0 & 0 \\ 
    1 & 3 & 6 & 16 & 33 & 0 & 179 & 0 & 967 & 0 \\ 
    1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\ 
    1 & 3 & 6 & 16 & 33 & 85 & 179 & 0 & 967 & 0 \\ 
    1 & 2 & 5 & 11 & 27 & 60 & 0 & 324 & 0 & 0 \\ 
    1 & 2 & 4 & 10 & 20 & 0 & 106 & 0 & 0 & 0 \\ 
    1 & 2 & 5 & 11 & 0 & 60 & 0 & 324 & 0 & 0
\end{array}
$$

This is really just the memory used above and as the recursive memoization approach above is working, it is slowly filling in this 2-d array with values. Since we know that these are the only answers we'll compute, we could just "fill in" the matrix, starting from the end we're familiar with. This would look something like this:

```python
def number_of_sequences(sequence_length, starting_digit):
  NUM_DIGITS = 10

  # NUM_DIGITS * sequence length array
  matrix = [[1 if x is 0 else 0 for x in range(sequence_length)] for y in range(NUM_DIGITS)]

  # range includes first but not last digit
  for length in range(1, sequence_length):
    for digit in range(NUM_DIGITS):

      # Get neighbor digits
      neighbor_digits = get_neighbors(digit)

      for neighbor_digit in neighbor_digits:
        matrix[digit][length] += matrix[neighbor_digit][length - 1]

  # The result is available at location for digit 0 in last column
  return matrix[0][sequence_length - 1]

```

Great. As an added bonus we ended up computing the results for all similar problems but where the knight ends on a different digit:

$$
\begin{array}{c c c c c c c c c c}
    1 & 2 & 6 & 12 & 32 & 66 & 170 & 358 & 904 & 1934 \\ 
    1 & 2 & 5 & 10 & 26 & 53 & 137 & 285 & 726 & 1537 \\ 
    1 & 2 & 5 & 11 & 27 & 60 & 145 & 324 & 776 & 1743 \\ 
    1 & 2 & 5 & 10 & 26 & 53 & 137 & 285 & 726 & 1537 \\ 
    1 & 3 & 6 & 16 & 33 & 85 & 179 & 452 & 967 & 2406 \\ 
    1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\ 
    1 & 3 & 6 & 16 & 33 & 85 & 179 & 452 & 967 & 2406 \\ 
    1 & 2 & 5 & 11 & 27 & 60 & 145 & 324 & 776 & 1743 \\ 
    1 & 2 & 4 & 10 & 20 & 52 & 106 & 274 & 570 & 1452 \\ 
    1 & 2 & 5 & 11 & 27 & 60 & 145 & 324 & 776 & 1743 
\end{array}
$$

However, at any given point we really only look at 2 columns of the array defined above. The one we're currently populating and the previously populated column. We could thus cut down the memory usage to be constant. It would look more like this.

```python
def number_of_sequences(sequence_length, starting_digit):
  NUM_DIGITS = 10

  # NUM_DIGITS * sequence length array
  matrix = [[1 if x is 0 else 0 for x in range(2)] for y in range(NUM_DIGITS)] # Changed

  # range includes first but not last digit
  for length in range(1, sequence_length):
    for digit in range(NUM_DIGITS):

      # Get neighbor digits
      neighbor_digits = get_neighbors(digit)

      for neighbor_digit in neighbor_digits:
        matrix[digit][length % 2] += matrix[neighbor_digit][length % 2 - 1] # Changed

  # The result is available at location for digit 0 in last column
  return matrix[0][1] # Changed

```

In each iteration for each digit, we're summing up the values from neighboring digits in the previous column. This sort of piece-wise summing can actually be done a matrix multiplication. We can define an adjecancy matrix $$A$$, and simply multiply the current column to get the next. Let's try that for the first column of all 1's. 


$$
x = 
\left(\begin{matrix} 
  1 \\
  1 \\
  1 \\
  1 \\
  1 \\
  1 \\
  1 \\
  1 \\
  1 \\
  1 \\
\end{matrix}\right)
\quad 
A =  \left(\begin{matrix} 
  0 & 0 & 0 & 0 & 1 & 0 & 1 & 0 & 0 & 0 \\
  0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 1 & 0 \\
  0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 \\
  0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 \\
  1 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 1 \\
  0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
  1 & 1 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 \\ 
  0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 \\
  0 & 1 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \\
  0 & 0 & 1 & 0 & 1 & 0 & 0 & 0 & 0 & 0 \\
\end{matrix}\right)

x^T A = 
\left(\begin{matrix} 
   2 \\
   2 \\
   2 \\
   2 \\
   3 \\
   0 \\
   3 \\
   2 \\
   2 \\
   2 \\
\end{matrix}\right)

$$