# Sudoku Solver

Author: Seena Shafai
Date: 12/03/2021
### Introduction
The code presented in the Jupyter notebook uses two AI techniques to solve any 9x9 sudoku given in the form of a 2D array. If the sudoku has no solution, the program returns a 9x9 matrix occupied with -1's; otherwise it returns the solved sudoku. 

#### Algorithms:

This implementation makes use of two AI algorithms:
 - Constraint satisfaction
 - Depth-first search
 
Combined, this technique is known as a **backtracking search**
    
#### Constraint Satisfaction
To achieve an efficient implementation of a sudoku solver, we can use constraint propagation which requires us to consider every possible value that an empty box of the sudoku could have. As the program checks each value's peers (its column, row and 3x3 box) it can eliminate certain possibilities from each individual square, until there is only one candidate (number) remaining, which is then placed into that box. In my implementation, I begin by converting the input 2D NumPy array into a dictionary, where each key represents a square using standard sudoku notation, which is shown below:

```
A1 A2 A3 | A4 A5 A6 | A7 A8 A9
B1 B2 B3 | B4 B5 B6 | B7 B8 B9
C1 C2 C3 | C4 C5 C6 | C7 C8 C9
---------|----------|----------
D1 D2 D3 | D4 D5 D6 | D7 D8 D9
E1 E2 E3 | E4 E5 E6 | E7 E8 E9
F1 F2 F3 | F4 F5 F6 | F7 F8 F9
---------|----------|----------
G1 G2 G3 | G4 G5 G6 | G7 G8 G9
H1 H2 H3 | H4 H5 H6 | H7 H8 H9
I1 I2 I3 | I4 I5 I6 | I7 I8 I9
```


The main body of `sudoku_solver(sDict)` function iterates through each value of the input array. If it detects a 0, the numbers 1-9 are set as a string value to that square's key. E.g. `{A1: '123456789'}`. If the square already has a value, it is simply transferred into the dictionary. In this way, we can see every candidate for each position in the sudoku, and the algorithm can use these to systematically eliminate options, creating new constraints and propagating these constraints across the board to either greatly reduce, or eliminate all extraneous candidates in a given position. Once this parsing is complete, the resulting dictionary is passed to the solving function: `solve(sDict)`. The process of eliminating these candidates from each square progressively is known as **constraint propagation**. 

##### Solving with Constraint Propagation:
I have employed two sudoku solving strategies: 
- ***Elimination***:
    Checks the row, column and 3x3 unit of the square being processed. Given our constraints: the square cannot contain any value which already exists in its row/column/unit. This allows us to eliminate many of the candidates inside our dictionary for each box.

- ***Naked singles*** (a.k.a. Only Choice)
    Checks the row, column and 3x3 unit of the square being processed. After the elimination function has been run on the dictionary, there will be certain boxes of multiple candidates, where one of those candidates will not appear in anywhere else in its 3x3 unit, row or column. This means that this candidate is the *only choice* for the given square, and should be placed there.
    
Both of these strategies benefit from constraint propagation: as more values become known, candidates can be removed from other squares, progressively solving the sudoku.
    

This method can be used on its own to solve some sudokus, but not all. In harder sudokus, it was found that constraint propagation resulted in some squares being left with zero possible candidates, meaning the sudoku is not solveable in its current form. Backtracking can be used here, to revert the sudoku back to a previously acceptable state, and trying a new combination of candidates.
    

#### Depth-first search:
In cases where constraint propagation has resulted in an empty square with no candidates, we consider our algorithm as 'stuck'. This means we need to 'rewind' and try a different combination of values. our depth-first search uses backtracking to revert the incorrect placements which led to the algorithm becoming stuck, and tries new combinations. This algorithm operates recursively until either a solution is found (where each square has a single valid candidate) or the algorithm remains stuck and at least one square has no possible candidates, meaning the sudoku is not solveable in any form.

To detect if the algorithm is stuck, my `apply(gridDict)` function records the total number of candidates across the board, before applying the elimination & naked singles strategies. If this number of candidates doesn't change, we know these strategies cannot solve the board in its current configuration any further, and we can move on to solving the board using a depth-first search.

##### Solving with Depth-first search
To fix a 'stuck' sudoku, we locate an empty square with the least number of candidates, and we create a range of new sudokus, each with a different possible candidate for the empty square. The `search(gridDict)` function is then called recursively, and attempts to solve the resulting sudokus until a solution is found. If the board remains stuck with no solution, we know that it is unsolveable.

#### Solution detection
At each level of the recursive search function, we check the keys of our grid dictionary containing the possible candidates for each square. If all keys consist of a single value: `all(len(v) == 1 for k,v in sDict.items())` we know that all squares have been filled, meaning that this is a potential solution. The final step is to re-run the validation check to ensure that no duplicates appear in any rows or columns. On successful validation, we can output the solved sudoku. If the validation fails, we know that the algorithm failed to solve the sudoku successfully, so we can output a matrix of -1's to show this. 


#### Notes/Issues:
When running the program in Jupyter Notebook, a different number of the *hard* sudokus are solved each time the kernel is reset, ranging between 9 and 15, but averaging at around 14 out of 15 total hard puzzles. This problem is not encountered on my local IDE but may become an issue during marking? I have a feeling it has something to do with time complexity/lag but I haven't figured it out yet...
