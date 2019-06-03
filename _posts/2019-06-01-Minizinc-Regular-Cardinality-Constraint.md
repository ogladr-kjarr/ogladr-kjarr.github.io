---
layout: post
title: Regular and Cardinality Global Minizinc Constraints
date: 01/06/2019
categories: Minizinc
tags: Minizinc
---

Having enjoyed a module in constraint programming while at university, when I saw that there are Coursera courses on the same subject I couldn't resist signing up.  Now, with the first course finished, that of [Basic Modelling for Discrete Optimization](https://www.coursera.org/learn/basic-modeling), I decided to post on two of the Minizinc global constraints that I either was initially consfused by (regular), or found really useful (global_cardinality).

Code available [here](https://github.com/ogladr-kjarr/blog-minizinc-global).

# Initial Model setups

The model is kept extremely simple.  We have a set of possible speeds that can be reached, and a decision variable with ten positions that can be assigned values from the set of speeds.
~~~
% The set of states that are allowed
enum SPEED = {slow, medium, fast, really_fast};

% the array to populate with speeds
array[1..10] of var SPEED: driving_speed;
~~~

Initially what we want to achieve is for the decision variable to hold the set of speeds that sum to the highest value.  Here we take advantage of the textual speed values in the set corresponding to an incremental numbering system underneath.

~~~
var int: total;
constraint total = sum(i in 1..10)(driving_speed[i]);
solve maximize total;

output ["Speed Array = \(driving_speed)\nTotal Score: \(total)"]      
~~~

As expected, the selection of speeds that maximizes the total is for all the entries to be `really_fast`.

~~~
Speed Array = [really_fast, really_fast, really_fast, really_fast, really_fast,
              really_fast, really_fast, really_fast, really_fast, really_fast]
Total Score: 40
~~~

# Regular Global Constraint

The regular constraint is used to enforce a sequence of states defined using a Finite Automata, which we can model using a state diagram.  For an easy example to use, the state diagram below shows the initial behaviour we will capture in the regular constraint.  This superimposes a set of allowed transitions with the speed states we introduced above.

<img src="/assets/img/minizinc-global-card-post/state1.jpg" alt="First State Drawing" width="300" height="500"/>

From the start state of 1, we can only go to state 2 (slow).  From state 2 we can only go to state 3 (medium) and from there back to 2 (slow) or to 4 (fast).  From state 4 (fast) we can only go to state 5 (really fast).  From there we can continue at state 5 (really fast), or to go state 2 (slow) or state 3 (medium).  The final state must be state 4 (fast).

First we load the global constraint using the `include` statement, and then we use the constraint.  

~~~
include "regular.mzn";
constraint regular(driving_speed, 5, 4, d, 1, {4});
~~~

* The first parameter is the decision variable that will hold the values.  
* The second parameter is the number of states in our state diagram, which we can see is five.  
* The third parameter represents the number of selectable values from the set we assign from, which is four.
* The fourth parameter is the 2d matrix that details the allowed transitions, described in more detail below.
* The fifth parameter is the starting state shown in our state diagram as state 1; there can only be one starting state.
* The sixth parameter is the set of end states, in our case there is only one option in the set.

The matrix that details allowed transitions is shown below.  Each row represents a state as defined in the state diagram, so the first row represents state 1, the second row state 2 etc.  The column values give the states that can be moved to, so, for the first row it can only move to state `2`, for the second row, it can only move to state `3`.  Finally the column position of the values give the actual value the change in state gives, so on the first row, the position of the value `2` matches up with the position of `slow` in the set of speeds, while on the second row `3` matches up with the position of `medium` in the set of speeds.  So moving from state 1 to state 2, moves from the initial state to `slow`.  Similarly the positioning on the second row, means that moving to the third state moves to the value `medium`.

~~~
array[1..5, SPEED] of 0..5: d =
      [| 2, 0, 0, 0
       | 0, 3, 0, 0
       | 2, 0, 4, 0
       | 0, 0, 0, 5
       | 2, 3, 0, 5 |];
~~~

To better understand the last part, we can look at the first two positions of the solver's output using this constraint, as expected it goes `slow -> medium`.  However if we moved the number `2` in the first row, into the third column, the model output for the first two positions would be `fast -> medium`.  This was the part that I felt wasn't explained well in the documentation, and I find it hard to describe even now used to it.  It's also important to remember that the starting state is never the first selected state in the output, the only way the start state could be selected as the first state is if it has a link to itself (like `really_fast` does).

Looking at the output of this model with the relationship constraints implemented using regular, we can see it behaving as expected, moving up the speeds, using `really_fast` as much as possible to increase the `total` value, while ending on `fast`.

~~~
Speed Array = [slow, medium, fast, really_fast, really_fast, really_fast,
              really_fast, really_fast, medium, fast]
Total Score: 31
~~~

# Cardinality constraints

The cardinality constraints are handy for setting min/max bounds on any particular variable values.  In the last output shown we saw that there were two instances of `medium` within the highest scoring combination of values.  Next we will specify that we want between three and four instances of `medium`

~~~
include "global_cardinality_low_up.mzn";
constraint global_cardinality_low_up(driving_speed, [medium], [3], [4]);
~~~

And in the output we see we get a lower value for overall target, but we end up with three instead of two instances of `medium`.  The upper bound of four would never be in the best output, as it would lead to a smaller overall total.

~~~
Speed Array = [slow, medium, fast, really_fast, really_fast, medium, fast,
              really_fast, medium, fast]
Total Score: 28
~~~

If we were determined to have exactly four instances of the `medium` speed, then we would replace the `global_cardinality_low_up` constraint with the following entry instead that states there must be four instances.

~~~
include "global_cardinality.mzn";
constraint global_cardinality(driving_speed, [medium], [4]);
~~~

And with this constraint we can see that we get four instances of `medium`.

~~~
Speed Array = [slow, medium, fast, really_fast, medium, slow, medium, slow,
              medium, fast]
Total Score: 21
~~~
