# tiebreakers-playground
Metrics for simulating and measuring the impact of tiebreakers in competitions

Outline
-------
Tiebreakers are used in cases where the primary method of ranking/teams (henceforth just called entities) does not fully resolve them into a ordinal order.

In some tournaments, there are implicit primary tiebreakers - for example in a bo3 round robin the implicit primary tiebreaker could be the # of series won - these are obviously measurable, but not within the scope of this analysis.

There are two main categories of tiebreakers: gathering additional data (i.e. more games), or best-effort tiebreaking based on already-played games by using statistics which are secondary to the core 'goal' of the event either as: 
* indirect properties (highest goal difference in football)
* derived properties (head-to-head matches between the tied entities)

This approach looks at the 'best-effort' tierbeakers (both indirect and derived).

Simulation
----------

The approach used here is to simulate multiple tournaments (of a specific format type) with entities of known skill, looking for cases when there are ties - and then seeing how multiple tiebreaker approaches would perform at using the existing data (but not the hidden true skill data) to separate the tied teams. 

Although it's a bit counterintuitive in running a tournament when you know the exact skill of each competitor and then try measure the "accuracy" of tiebreakers to order the entities correctly there are two redeeming facts. Firstly the tournament itself has some known variance and this approach only works on sane formats, and secondly all we're doing here is measuring the resillience to upsets (i.e. we're attempting to extract some signal from the noisy results). 

Tiebreakers can have multiple evaluatable outcomes, in a similar way to how we can score individual rounds of [Mastermind](https://en.wikipedia.org/wiki/Mastermind_(board_game)). For example let's say there are 3 entities tied that we wish to tiebreak [A, B, C] such that the true skills are A > B > C. Some possible outcomes are:

* A > B > C (3 correct comparisons)
* A > [B = C] (2 correct comparisons, 1 abstained comparison)
* A > C > B (1 correct comparison, 2 incorrect comparisons)
* C > B > A (3 incorrect comparisons)

Abstaining from making a decision isn't a bad quality of a tiebreaker: if we had a tiebreaker which was 99% correct but only gave a conclusive result 1% of the time this would be an incredible tiebreaker (we could call it the 'Dr House' of tiebreakers). Multiple excellent tiebreakers cascaded together mean we would very infrequently get the wrong answer.

Our goal is primarily to see which tiebreakers are absolutely correct when they do break any tie between any k number of tied entities. In the above examples: if, for any tiebreak there are X correct comparisons, Y abstained comparisons and Z incorrect comparisons then we consider the tiebreak "bad" if Z > 0, "good" if X > 0 and Z == 0, and "irrelevant" in all other cases. The term "absolute accuracy" is simply #good / (#good + #bad).

Separately we can evaluate the unweighted accuracy for each k choose 2 pair of entities in a tiebreak: which is simply the # correct comparisons / (# incorrect comparisons + # correct comparisons). This can give a good indication about how well the tiebreaker might perform as an upper bound if used only in 2-way ties (if it's too low then the tiebreaker is just poor in all cases and/or isn't correlated to the skill of the team).


Sample Output
-------------

*These are each from 100k simulations in a bo3 round robin*

Format is: `metric name`, `absolute accuracy (x100)`, (`sample size`, `unweighted accuracy (x100)`).

For k=8
```
2-0s: 78.469 (n=100403, unweighted=95.885)
game_diff: 64.364 (n=117169, unweighted=92.567)
game_wins: 62.919 (n=99762, unweighted=92.371)
series_neustadtl: 48.463 (n=108610, unweighted=87.458)
series_h2h: 47.126 (n=105514, unweighted=86.749)
games_h2h: 43.581 (n=130613, unweighted=87.135)
```

For k=6
```
2-0s: 79.731 (n=66146, unweighted=96.284)
game_diff: 61.136 (n=79760, unweighted=91.982)
game_wins: 60.452 (n=66488, unweighted=92.052)
series_neustadtl: 48.681 (n=70196, unweighted=87.494)
series_h2h: 47.619 (n=68441, unweighted=86.888)
games_h2h: 43.260 (n=88757, unweighted=87.285)
```

Interpretting Results
---------------------

Using 2-0's (the number of series won 2-0) is the best metric for evaluating ties in the 8-team bo3 round robin format. In 100k runs it was used 100403 times (remember that there can be multiple ties in a single iteration of a simulation), and in 78.47% of those cases it did some work which contained no errors. Of all the individual rankings it considered, it was correct 95.89% of the time.

Further Work
------------

This just evaluates primary tiebreakers, and ordering secondary, tertiary, etc. tiebreakers is a separate problem although my conjecture is that the ordering for X+1-th tiebreakers is never significantly different from X-th tiebreakers with the best tiebreaker removed.

There are also possible "ensemble tiebreakers" where multiple tiebreakers are applied together and require some sort of quorum (% actually breaking a tie) and agreement (% agreeing on a specific ordering or partial ordering) which could yield much more accurate rankings.

