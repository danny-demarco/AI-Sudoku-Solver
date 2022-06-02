# Solving Sudoku with Backtracking

A University of Bath project in Artificial Intelligence in which I was awarded a Distinction.

Forenote: _There were, in reality, a lot more iterations of improvement of the stated algorithms and their related functions, however for the sake of brevity I will focus on what I feel are the most important changes and learnings from this assignment_

I had never played a game of Sudoku before being given this assignment, so my first step was to pull up a Google search and find out _'How to play Sudoku'_. This led me to find there are three simple rules to the game. Given access to a set of numbers S = {1,2,3,4,5,6,7,8,9}, you must fill the cells on a given board according to the following:

1. No numbers in the set S may appear more than once in any _row_
2. No numbers in the set S may appear more than once in any _column_
3. No numbers in the set S may appear more than once in any _square_ (3x3 block of the grid)

## Initial Thoughts

My initial thoughts were as to how I was going to navigate through the sudoku environment, and at each cell, how was I going to choose a value from the set to place in said cell. I could envisage two different methods of doing so. In both methods I would iterate through the cells searching for a 0 (Empty cell):

1. I could step into each empty cell, try every number in the set within that cell, and for each value proposed that conformed to the 3 rules above, try all possible values in the next cell, and so on throughout the board until the board was full, I would keep track of all routes along the way and then choose any valid path from start to end. Thinking through this method, I realised that this is a breadth-first search.

2. I could step into each empty cell, try one number from the set, see if that number conformed to the 3 rules above and if so move on to the next cell. If at any point I find a cell that violates any of the 3 rules, I return to the previous cell (backtracking), increment to the next value in the set and continue. As soon as the last cell is filled and checked for conformity a valid path has been obtained. Thinking through this method, I realised that this is a depth-first search that uses backtracking.

Assignment directions had suggested a depth-first search could be sufficient for this assignment, and intuitively I thought this method must be faster than a breadth-first search, as in a BFS you would need to consider every branch in the search tree, whereas the DFS had the possibility of finding a valid route with fewer explored states, so I decided I would try to implement option 2 as an algorithm at this point thinking I had devised a method good enough to solve the problem. 

## First Attempt Algorithm

In my first attempt at writing the algorithm, I decided to set up a separate function for the backtracking part of the algorithm, and another function to deal with the process of checking whether a suggested cell value was valid. The backtracking function was implemented using a recursive call. Each step through the Sudokus empty cells would choose a value from set S = {1,2,3,4,5,6,7,8,9} and then move onto the next cell via a recursive call until either a valid route was found or an invalid value was chosen, at which point backtracking occurred. I went through the following sequence:

1. Check board is valid
2. Iterate through each cell until an empty space is located, if no empty cells game is complete
3. Choose the first value from set S and try in this cell
4. Check board is valid
5. If valid return to step 2, else continue
6. Increment to next value in set S and update the cell, if no values are left in set S, go back to the previous cell and increment set S
7. Check board is valid
8. If valid return to 2, else return to 6

This was a working algorithm and was able to solve the vast majority of the Sudokus given in the test batch, however, it was not operating fast enough and so struggled to solve the hard Sudokus within a reasonable amount of time. It was capable of solving the vast majority of the very easy, easy, and medium Sudokus in 0.0 seconds, but 5 of the hard sudokus were taking over 30 seconds with the slowest taking 103.765625 seconds to solve. But what was slowing it down so much? The first issue was that at every cell, the algorithm was checking every possible value from the set S, even if it was in a row, column, or square where that value already existed, it would then run through the whole algorithm for every value until it found a valid one. This was leading to a massive number of loops of checking numbers that had no chance of being correct. The following function would be called for every number tried at each cell leading to a huge amount of additional and unnecessary computation. 


```Python
def check_valid(sudoku, number, position):
    """
    Checks to see whether the suggested number to be allocated to a cell fits the 3 constraints
    1: Check Row
    2: Check Column
    3: Check Square
    """
    for row in range(sudoku.shape[0]):
        if sudoku[position[0]][row] == number and position[1] != row:
            return False

    for column in range(sudoku.shape[1]):
        if sudoku[column][position[1]] == number and position[0] != column:
            return False

    square_x = position[1] // 3
    square_y = position[0] // 3
    for r in range(square_y * 3, square_y * 3 + 3):
        for c in range(square_x * 3, square_x * 3 + 3):
            if sudoku[r][c] == number and (r, c) != position:
                return False

    return True
```

But this was not the only issue that was slowing down my algorithm, on further inspection, I realised that every time I had placed a value in a cell and checked its validity, I would then search for the next empty space in the sudoku, and to do this I was iterating through the entire sudoku array in the quest of finding the next 0 value(empty cell). This was creating a longer and longer search for an empty cell as I progressed through my algorithm. This was my original function:

```Python
def locate_empty_space(state, cell):
    """Iterates through the Sudoku until a zero (empty space) is found, and then returns its index"""
     for idx, x in np.ndenumerate(state):
         if x == 0:
             return idx
     return False
```

So my next step was to think through a way to optimise my current algorithm by eliminating all the extra unnecessary computation that was taking place.

## Reflecting on Initial Attempt and Planning Improvements

This initial attempt at implementing the algorithm left me thinking about the state space being searched and the time complexity of my algorithm. There are 81 different cells in a Sudoku, with 9 possible values in each cell, that is 9<sup>81</sup> = 1.966271 × 10<sup>77</sup> possible states in a 9x9 Sudoku, or if we account for the numbers already present in a hard Sudoku, say we have 16 values supplied, 9<sup>(81 - 16)</sup> = 1.061117 × 10<sup>62</sup> states, which is considerable. The space complexity in this algorithm is not so much of a worry. We can model the Sudoku as a tree search problem: at each cell, we would have 9 possible branches (b=9), each of which is an element of set S = {1,2,3,4,5,6,7,8,9}. The maximal length of any path through the tree (m), and the shallowest node of the goal state as (d). So our state complexity _O(bm) = 9 * (9*9) = 729_. If we compare this to the possible state complexity of one of the original ideas of traversing the tree with a breadth-first search, we would need to visit each possible node to the bottom of the tree until we could determine a valid path, which would result in _O(b<sup>d</sup>) = 9<sup>81</sup> = 1.966271 × 10<sup>77</sup>. A huge difference that highlights the benefit of the depth-first search for this problem. This is because the DFS does not need to keep track of a node once all of its children have been explored and ruled out as a valid path, whereas the BFS algorithm does.

With these reflections in mind, I turned my thoughts to how I could improve my current algorithm. The existing algorithm is completing a search through paths that are irrelevant and cannot lead to a goal, and it's only upon completing these searches it can rule them out. If I could ensure that only paths that have the possibility of being relevant are searched, I could substantially reduce the amount of computation and thus the time to find a valid path to the goal node. For example, if I am traversing through the first row of a sudoku, and that row already has values {3,6} used from state S, the respective column has {5}, and the related square has {1,9}, how can I ensure that only states {2,4,7,8} are suggested, rather than {1,2,3,4,5,6,7,8,9}. This is a constraint satisfaction problem and effectively trims the branches off of the search tree of non-valid routes before they are searched according to the values of other known nodes in the tree.

To apply constraint satisfaction to my current algorithm, it was clear that I need to change the way I was checking if a particular value was valid, and so my existing check_valid function listed above was not going to be sufficient. To overcome my problem I would need a way to keep track of which values had been used in the 3 different domains: row, column, and square. If I stepped into a new cell (node), I would need to know which values from state S had already been used in any of the 3 domains so these values could be eliminated from my possible choices for that cell. I also needed to change the way I was searching for empty spaces on the sudoku board.


## The Final Solution

I knew I needed a way to allocate 9 possible values to each cell {1,2,3,4,5,6,7,8,9}, and a way to remove each value as it was used in that respective cell's row, column, or square. Then when allocating a value to an empty cell, I could just choose one that had not already been allocated. But, overall that amounts to 9 values per cell, in 81 cells. That's 729 different values, how do I keep a track of all of those? Drilling down and looking at each separate recursive call individually, if a value exists in a row, column, or square, we cannot use it. So maybe I could have an array to represent each row, col, and square, each with 9 values {1,2,3,4,5,6,7,8,9}, rather than an array for each cell, and I could hold these arrays within a matrix for fast easy access. That reduces the number of values to 243, an array containing 9 values, so 27 arrays in one matrix. At each cell, I could check which values had not been used from its related row, column, and square arrays, choose a valid option and mark the value unavailable in those 3 arrays. So this was the method I decided to follow through on. My new amended function for checking validity was a lot more elegant:

```Python
def check_allowed_values(number, row, col, allowed):
    """Checks the allowed values matrix (generated below) for a valid value"""
    if allowed[row, number-1] != 0 and allowed[col+9, number-1] != 0 and allowed[square[(row//3, col//3)], number-1] != 0:
        return True
    else:
        return False
```

The next challenge was how I could index the square arrays using just the row and column values. I managed to do this by creating a dictionary and effectively treating the 9 x 9 sudoku matrix as a 3 x 3 matrix and just taking the modulo values of the rows and columns to produce each index. I had to also make some minor changes to my original recursive algorithm to incorporate my new method of allocating a value to a cell, which included marking each value as a 0 in each of the respective row, column, and square as they were allocated and thus removing them from a later allocation. 

I then had to fix the method of locating an empty space in the sudoku. I realised that I could cut out all the additional searching if I could just start my search from the cell that I last placed a value in. So, I created a new variable 'cell' that would be initialised automatically once my backtracking algorithm was called, and would increase every time my algorithm recursed or if my search for an empty space was not in the next cell, effectively moving the starting position of my search on each iteration. The new function looked like this:

```Python
def locate_empty_space(state, cell):
    """Iterates through the Sudoku until a zero (empty space) is found, and then returns its index"""
    while cell < 81:
        row = int(cell//9)
        col = cell%9
        if state[row, col] == 0:
            return row, col, cell
        else:
            cell += 1
    return False
```

The improvement made from these alterations is considerable. The largest performance improvement came from the alteration of my locate_empty_space function. All sudokus are now being solved well within the target time of 30 seconds, with the longest problem taking just 4.859375 seconds to solve. The vast majority of problems are solved within 0 seconds and the algorithm manages to solve all 60 given sudokus in just 21.3 seconds 

This has been a very fun and challenging exercise! 

Further optimisation of the algorithm could be achieved by using a minimum remaining values heuristic.


***

