<pre>
<code class="language-python">
"""
Build a spiral in a matrix of size NxN, the spiral direction is defined 
by the DIRECTIONS constant when choosing the next direction, complexity
is O(n . m), the DIRECTIONS adds a negligible complexity on the lookup,
we could break it into lists, however this is simpler to maintain, being
a single structure.
"""

DIRECTIONS = {
    "R": {
        "moves": (0, 1),
        "next": "D",
    },
    "D": {
        "moves": (1, 0),
        "next": "L",
    },
    "L": {
        "moves": (0, -1),
        "next": "U",
    },
    "U": {
        "moves": (-1, 0),
        "next": "R",
    },
}

def build_spiral(matrix: list) -> list:
    # if the matrix is not NxN or if it's a vector, stop
    matrix_len = len(matrix)
    if matrix_len < 2:
        return []
    for n in range(len(matrix)):
        if len(matrix[n]) != matrix_len:
            return []

    start = [matrix_len // 2, matrix_len // 2]
    direction = "R"
    steps = 0
    lim = 2
    within_limit = False
    while True:
        moves = DIRECTIONS[direction]["moves"]
        for i in range(steps, lim):
            position_x = start[0] + (i * moves[0])
            position_y = start[1] + (i * moves[1])
            # if on edge corner, we should stop the loop
            if position_x < matrix_len and position_y < matrix_len:
                matrix[position_x][position_y] = 'X'
            else:
                if within_limit:
                    break
                else:
                    within_limit = True
        start = [position_x, position_y]
        steps = 0
        lim += 1
        # edge lines can still turn if they have room, but evaluating out of range is wrong
        if lim > matrix_len:
            if within_limit:
                break
            else:
                lim -= 1
                within_limit = True
        direction = DIRECTIONS[direction]["next"]
    return matrix

if __name__ == "__main__":
    matrix = [
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
        [0,0,0,0,0,0,0,0,0,0,0,],
    ]
    print(build_spiral(matrix))


    matrix = [
        [0,0,0,0,0],
        [0,0,0,0,0],
        [0,0,0,0,0],
        [0,0,0,0,0],
    ]
    print(build_spiral(matrix)) # not NxN

</code>
</pre>

---
<div align="center">
  <a href="../../main/indexes/2025.md" title="Back to year's index">📖</a>
</div>