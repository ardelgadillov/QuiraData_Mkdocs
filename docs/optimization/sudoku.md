# Sudoku Solver

Using Mixed Integer Linear Programming (MILP) to solve a Sudoku puzzle is an
interesting application of optimization techniques.

## Understanding the Sudoku Puzzle

Sudoku involves filling a $n^2 \times n^2 $ for $n \in \mathbb{Z}$
(with $n=3$ being the most common Sudoku Puzzle) grid such that:

- Each row contains the numbers 1 to $n^2$ exactly once.
- Each column contains the numbers 1 to $n^2$ exactly once.
- Each $n \times n$ subgrid contains the numbers 1 to $n^2$ exactly once.
- Some cells are pre-filled with known values.

## Formulating Sudoku as a MILP Problem**

For Sudoku, the goal is to find a feasible solution that satisfies all constraints
without needing an explicit optimization objective.

### Sets

- $ i $ represents the row index (1 to $n^2$)
- $ j $ represents the column index (1 to $n^2$)
- $ k $ represents the digit (1 to $n^2$)
- $ p $ represents the subgrid index in the horizontal direction (1 to $n$)
- $ q $ represents the subgrid index in the vertical direction (1 to $n$)

The following table illustrates the indexing for $n=3$.

<table class="sudoku">
    <!-- Header row with individual columns -->
    <tr>
        <th style="width: 13%;">Row $i$ Column $j$</th> <!-- Reduce width of the first column -->
        <th>$1$</th>
        <th>$2$</th>
        <th>$3$</th>
        <th>$4$</th>
        <th>$5$</th>
        <th>$6$</th>
        <th>$7$</th>
        <th>$8$</th>
        <th>$9$</th>
    </tr>

    <!-- First 3 rows -->
    <tr>
        <th>$1$</th>
        <td colspan="3" rowspan="3">$p=1$, $q=1$</td>
        <td colspan="3" rowspan="3">$p=1$, $q=2$</td>
        <td colspan="3" rowspan="3">$p=1$, $q=3$</td>
    </tr>
    <tr>
        <th>$2$</th>
    </tr>
    <tr>
        <th>$3$</th>
    </tr>
    
    <!-- Next 3 rows -->
    <tr>
        <th>$4$</th>
        <td colspan="3" rowspan="3">$p=2$, $q=1$</td>
        <td colspan="3" rowspan="3">$p=2$, $q=2$</td>
        <td colspan="3" rowspan="3">$p=2$, $q=3$</td>
    </tr>
    <tr>
        <th>$5$</th>
    </tr>
    <tr>
        <th>$6$</th>
    </tr>
    
    <!-- Last 3 rows -->
    <tr>
        <th>$7$</th>
        <td colspan="3" rowspan="3">$p=3$, $q=1$</td>
        <td colspan="3" rowspan="3">$p=3$, $q=2$</td>
        <td colspan="3" rowspan="3">$p=3$, $q=3$</td>
    </tr>
    <tr>
        <th>$8$</th>
    </tr>
    <tr>
        <th>$9$</th>
    </tr>
</table>

### Variables

Define a binary decision variable $ x_{ijk} $, where:

The variable $ x_{ijk} = 1 $ if digit $ k $ is placed in cell $ (i, j) $, 
otherwise # x_{ijk} = 0 $.

#### Constraints

The problem is modeled by introducing several constraints:

1. **Each cell must contain exactly one digit**
   $$
   \sum_{k} x_{ijk} = 1 \quad \forall i, j
   $$

2. **Each digit appears exactly once in each row**
   $$
   \sum_{i} x_{ijk} = 1 \quad \forall j, k
   $$

3. **Each digit appears exactly once in each column**
   $$
   \sum_{j} x_{ijk} = 1 \quad \forall i, k
   $$

4. **Each digit appears exactly once in each subgrid**
   $$
   \sum_{i=n \cdot (p-1) + 1}^{n \cdot p} \sum_{j=n \cdot (q-1) + 1}^{n \cdot q} x_{ijk} = 1 \quad \forall p, q, k
   $$

5. **Pre-filled cells must match given values**
   If cell $ (i, j) $ is pre-filled with digit $ k $:
   $$
   x_{ijk} = 1
   $$

### Objective Function

There’s no optimization goal in Sudoku (it’s a feasibility problem), 
so the objective function can be trivial, like minimizing $ 0 $.

## Example MILP Code (Using Python and Pyomo)

Here's a basic Python implementation with Pyomo:

```python
import pyomo.environ as pyo
from pyomo.core.util import quicksum
from pyomo.opt import SolverFactory

# Create a model
def sudoku_model(n):
    """ define the optimization model
    """
    
    size = n ** 2
    
    model = pyo.ConcreteModel()

    """Sets"""
    model.i = pyo.RangeSet(1, size)  # rows - 1 to size
    model.j = pyo.RangeSet(1, size)  # columns - 1 to size
    model.k = pyo.RangeSet(1, size)  # digits - 1 to size

    model.p = pyo.RangeSet(1, n)  # row boxes - 1 to base
    model.q = pyo.RangeSet(1, n)  # column boxes - 1 to base

    """Variables"""
    model.x = pyo.Var(model.i, model.j, model.k, bounds=(0, 1), domain=pyo.NonNegativeReals)  #domain=pyo.Binary)

    """Objective Function"""

    def obj_expression(m):
        return 0  

    """Constraints"""

    # Unique digits
    def c_digits(m, i, j):
        return quicksum(m.x[i, j, k] for k in m.k) == 1

    # Unique in rows
    def c_rows(m, j, k):
        return quicksum(m.x[i, j, k] for i in m.i) == 1

    # Unique in columns
    def c_columns(m, i, k):
        return quicksum(m.x[i, j, k] for j in m.j) == 1

    # Unique in boxes
    def c_boxes(m, k, p, q):
        return quicksum(m.x[i, j, k] for i in range(n * (p - 1) + 1, n * p + 1)
                        for j in range(n * (q - 1) + 1, n * q + 1)) == 1

    """Define optimization problem"""
    model.obj = pyo.Objective(rule=obj_expression, sense=pyo.maximize)
    """Add constraints to optimization model"""
    model.c_digits = pyo.Constraint(model.i, model.j, rule=c_digits)
    model.c_rows = pyo.Constraint(model.j, model.k, rule=c_rows)
    model.c_columns = pyo.Constraint(model.i, model.k, rule=c_columns)
    model.c_boxes = pyo.Constraint(model.k, model.p, model.q, rule=c_boxes)

    # return model
    return model
```

### Advantages

- MILP ensures the solution satisfies all constraints.
- Can handle more complex Sudoku variants by adding new constraints.

## Dash Plotly app
Here you can find a webapp using Dash Plotly to solve any sudoku, I hope you enjoy it!

<div>
    <iframe src="https://sudoku-rdgs.onrender.com/" onload="this.width=screen.width;this.height=screen.height;" frameBorder="0"></iframe>
</div>