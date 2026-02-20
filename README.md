# Texas Holdem Monte Carlo #

## Overview ##
This repository contains code that simulates hundreds of thousands of poker hands and collects data for estimating hand equity, a useful resource for providing insight into player behavior, game dynamics, and strategies that can improve performance in poker games. Equity estimates are based on features engineered to describe similar hands as they appear over Monte-Carlo simulation data.

## [Section 01: Hand Simulation](./notebooks/01_hand_simulation.ipynb) ##

First, 100,000 hands of Texas Holdem poker are simulated, each with 9 players, and stored as rows of a dataframe. For each hand, two hole cards are dealt to each player, as well as 5 community cards. The strength of each player's hand at each stage in a round, or street, is measured using a `deuces.evaluator` object. Then, each player is given a "showdown order", which represents the ranking of that player's hand among the 9 total hands per row. This forms the wide-form hands dataset.

Secondly, the wide-form data is converted to a long-form, where each row corresponds to an individual player. This 
method of conversion preserves the relationship between different players and their showdown orders from the wide 
dataframe. Since the wide dataset had 100,000 hands and 9 players, the resulting long data should have 900,000 rows,
which represent each player individually over every hand. The use of the `deuces` library ensures efficient hand 
evaluation, making this dataset a valuable resource for estimating equity, for example. 
The `showdown_order_` feature of the long-form dataset will be essential in estimating hand equity.

## [Section 02: Hand Frequency Distribution](./notebooks/02_hand_freq_distribution.ipynb) ##

Now that the long-form dataset has been generated, its validity can be verifying partially by comparing the frequencies
of different types of hands with their known likelihoods. Subsequently, the RMSE between frequencies and likelihoods can
be used as an approximation for accuracy. The following visualization demonstrates the distribution of frequencies:

![Hand Frequency Distribution](./figures/hand_freq_dist.png)

 In conclusion, the RMSE between frequencies and likelihoods was quite low, which is a good sanity check before equity 
 estimation.
 
## [Section 03: Pre-flop Equity](./notebooks/03_preflop_equity.ipynb) ##

Using the long-form dataset, features meant to aggregate for equity estimation can be engineered. Pre-flop, the most 
valuable information for determining equity will be the rank of each hole card, whether the cards are suited or not, and 
the total number of players. 


To calculate the number of times a hand would win at showdown given its showdown order $O \in [1,9]$ and number of 
players $P \in [2,9]$, as well as the total number of hands to consider for $P$ players, we need to use two different discrete combinations.

To determine how many hands to consider for $P$ players, choose $P-1$ opponents from the 8 total opponents (since one player is the target).

$\text{total hands} = \binom{8}{P-1}$

To determine how many times a hand would win at showdown given its showdown order $O$ and number of players 
$P$, there are $9-O$ players with less than or equal showdown order. Therefore, choose $P-1$ opponents from these $9-O$ players.

$\text{wins} = \binom{9 - O}{P-1}$

When the resulting dataframe is grouped by Pre-flop features, and the total wins and hands are summed up, 
the equity estimation can be calculated given a fixed set of features:

$\text{equity} = \frac{\text{sum(wins)}}{\text{sum(total hands)}} $

In result, a heads-up Pre-flop equity matrix can be visualized, providing insight into which hole cards have the best winning 
odds:

![Preflop Equity Matrix](./figures/preflop_equity_matrix.png)

## [Section 04: Flop Equity](./notebooks/04_flop_equity.ipynb) ##

The Flop equity for each player's hand is then estimated using a various engineered features derived from the hole cards
and flop cards. These features include:
- Hand Classification: Pair, High Card, etc.
- Pocket Pair: Boolean indicating if the hole cards are a pocket pair.
- Suited: Boolean indicating if the hole cards are suited.
- Flush Potential: Complete, Open, Backdoor, or None.
- Straight Potential: Complete, Open, Gut-shot, Backdoor, or None.
- Overcards: Number of hole cards that are higher than the highest card on the flop.
- Board Texture: Number of unique suits/ranks on the board.
- Board Connectivity: Number of cards required to complete a straight on the board.

After engineering features, multiplying rows by number of players, 2-9, and calculating total wins and hands, the 
dataframe is grouped in a similar way to get the equity for a known set of features. Now, $\text{equity} = \frac{\text{sum(wins)}}{\text{sum(total hands)}}$ for any fixed set of features.

The following visual demonstrates Flop equity with fixed "Hand Class" and "Players":

![Flop Equity Matrix](./figures/flop_equity_matrix.png)

The following visual demonstrates Flop equity with fixed "Flush Potential" and "Suit Texture":

![Flop Flush Draw](./figures/flop_flush_draw.png)

## [Section 05: Turn & River Equity](./notebooks/05_turn_river_equity.ipynb) ##

In a similar way to Section 04, features are engineered for the Turn and the River in order to group similar hands and 
aggregate for equity calculations. River features are not concerned with some features such as "Flush Potential" because
the hand will no longer see new cards. 

The following visualization demonstrates Turn equity with fixed "Hand Class" and "Players":

![Turn Equity Matrix](./figures/turn_equity_matrix.png)

The following visualization demonstrates River equity with fixed "Hand Class" and "Players":

![River Equity Matrix](./figures/river_equity_matrix.png)

## Results ##

Each heatmap generated using equity estimates show reasonable patterns. The datasets which generated the heatmaps are 
likely valuable resources to gain insights into player behavior, game dynamics, and strategies that can improve 
performance in poker games. The `.csv` files are hosted online [here](https://www.kaggle.com/datasets/benjaminniesmertelny/texas-holdem-monte-carlo-data).

## Next Steps ##

Finally, this project concludes by producing a few useful datasets. Each of these datasets can be used to give a reasonable estimate of equity for any hand at any stage of the game. Next steps for this project include developing a naive strategy that uses these equity estimations to determine an ideal bet amount. Furthermore, a statistical model developed in order to detect patterns in opponent strategies paired with the equity estimation strategy could prove to be a highly successful model for AI poker playing or for developing a virtual practice tool. 
