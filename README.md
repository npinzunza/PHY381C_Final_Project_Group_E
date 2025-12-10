# Group Members:
Sofia McDonough, Afsana Mimi Raka, Nicholas Inzunza

# Topic:
Predicting and Solving Solitaire

# Goal Statement:
Solitaire is a one-player card game, traditionally played with poker cards, where the player lays out the cards into piles and plays by organizing them in a certain way, winning by completely organizing the deck into all four suits.  In solitaire, it is possible that the player can become irreversibly stuck, unable to win.  This project compares the winrates of a computer solver outlined in Yan et al 2005 and one devised by project member Nicholas Inzunza.

# Directory Structure:
```solitaire_italian.ipynb``` and ```solitaire_full.ipynb``` are the main two notebooks.
```Yan_et_al_05.pdf``` and ```Bjarnason_et_al_07.pdf``` are papers about computer solitaire.
```statistics.ipynb``` is a notebook containing some graphs and statistics comparing different solitaire strategies.

# Documentation:
## Background
If you are unfamiliar with the rules of solitaire, you can read them in Section 2 of ```Bjarnason_et_al_07.pdf```.

The solitaire simulator is based on that outlined in ```Yan_et_al_05.pdf``` and works as follows:
1. Start with a shuffled deck and deal it as one would to start a game of solitaire
2. The computer is given a set of rules to the game telling it where the cards can be played
3. The human programmer gives the computer a strategy so it knows what moves to make in order to progress through the game, otherwise it will make moves randomly

A strategy is given by a heurisitic and priority function.  These functions take a proposed move and assign it a number determined by the human.  The computer then makes the move which has the highest number associated with it.  For moves with equal heuristics, the move is then given a priority, another number based on the move.  If moves have equal heuristics and equal priorities, then the computer chooses randomly between them.  This form of play is called "greed" according to ```Bjarnason_et_al_07.pdf``` as it prioritizes moves which are benefitial in the immediate state and does not look ahead.

## Code
### Preface
First, we would like to note that this code is FAR from peak efficiency.  There are several places in the code which one can IMMEDIATELY detect as being open to simplification and condensation.  As much as we would love to spend time trimming this down (dead serious by the way) the code is able to run quickly enough that it did not pose much of a problem for the purposes of this class.  If we, however, were investigating questions related to solitaire in the name of SCIENCE, then we would DEFINITELY need to rewrite and shorten the base code.

This being said, we start by noting that we created 2 base files: ```solitaire_italian.ipynb``` and ```solitaire_full.ipynb``` which play with an Italian deck (10 cards per suit) and a full card deck (13 cards per suit, also called a French deck), respectively.  That is the only difference between them.  We distinguished between these two as games with the Italian deck are much shorter (on the order of 1 milisecond).

There are some other slight differences from traditional solitaire in these notebooks.  Traditional solitaire is played by drawing 3 cards at a time, but here, we have decided to let the computer play any card from the draw pile at any time (effectively 1-draw solitaire, also called Baby Solitaire).  We did this likewise in the interest of time for the computer to solve and to reduce chances of loss or getting stuck in infinite loops.  In solitaire experiments, they have computers play a version called Thoughtful Solitaire, in which all of the face down cards on the board are known, though they cannot be played until they are unblocked, so that the player is not moving blindly ahead.  We created our program with this knowledge as well.  The last difference is that we allowed the movement of "partial tableau stacks" as ```Bjarnason_et_al_07.pdf``` calls them in Section 2, which were not allowed in ```Yan_et_al_05.pdf```.  These types of movements in addition with the other differences from traditional solitaire which we implemented should make the game more easily and more quickly solvable.

### Implementation and Basic Usage
First note that this software uses the ```random``` package to make random choices and to shuffle cards.  Everything else is written in base python.

If this software is like a burger, then the meat is the ```Board``` class and the tomato, lettuce, and onions are the ```Hand```, ```Tableau```, and ```Suitstacks``` classes which are each made of up of ```Card``` objects and all have different properties associated with them.  All of which are defined in the main notebooks containing EXTENSIVE comments.

We will clarify some terminology.  The "board" is likened to the colloquial term for the playspace of the game: board.  The "tableau" is the same as the word used in the literature for the cards placed on the board at the beginning of a solitaire game.  The tableau is made up of "piles".  The "hand" is what we call the rest of the deck not on the tableau at the beginning of the game which the player draws from.  The "suitstacks" are where the player aims to organize the cards into the 4 suits to win the game.  These are slightly different names than what is common in the literature.

The ```Board``` class contains the data for which cards are stored where.  Its attributes ```.hand```, ```.tableau``` and ```.suitstacks``` are ```Hand```, ```Tableau```, and ```Suitstacks``` objects, respectively, containing the cards in the hand, tableau, and suitstacks.  The ```Deck``` class is used to shuffle cards and to easily initialize a board.  One does so as follows:
```
deck = Deck() # a list of Card objects organized by rank and suit
deck.shuffle() # this shuffles the deck
board = deck.deal() # the .deal() method returns a Board object made from the cards in the Deck object
```
Play proceeds by calling the ```.random_play()``` or ```.greed_play()``` method.
```
board.random_play() # the simulator checks which moves can be made and chooses one randomly
```
A strategy is passed through ```.greed_play()``` which is a tuple of a heuristic and priority function.
```
# functions yan_heuristic and yan_priority have 
# the same properties as those defined in Yan et al 2005
yan_strategy = (yan_heuristic, yan_priority)

# this plays "greedily" with the strategy passed through .greed_play()
board.greed_play(strategy = yan_strategy)
```
The board's hand, tableau, and suitstacks are then updated based on the move it makes.  It's also possible to make custom moves with the ```.move()``` method which has certain weirdly-defined objects passed as its arguments (see the comments in the notebooks).

The game is effectively won once all the cards are flipped face up.  This is captured in the ```.check_win()``` Boolean method.  The game is definitively lost if there are no available moves, which is contained in the ```.no_moves``` Boolean attribute.

A simulation looks something like this:
```
num_trials = 1000 # number of games to play
num_max_moves = 1000 # maximum number of moves we let the computer play
strat = (heuristic,priority) # strategy functions

num_wins = 0 # keep track of how many wins we have
for i in range(num_trials)
  deck.shuffle() # shuffle before we being
  board = deck.deal()
  for j in range(num_max_moves):
    board.greed_play(strategy=strat)
    if board.check_win():
      num_wins+=1
      break
    if board.no_moves: break
```
### Results
According to ```Yan_et_al_05.pdf```, their strategy resulted in a base winrate of ~13% (Section 5, Table 1).  In ```Bjarnason_et_al_07.pdf``` their strategy resulted in a base winrate of ~16%.  In this project, we compared the strategy in ```Yan_et_al_05.pdf``` with a strategy developed by project member Nicholas Inzunza.  Each of their strategies are as follows:
1. Yan et al 2005:
Heuristic:
 - if a card is played onto the Suitstacks, then that is +5
 - if a card is being played from the Hand to the Tableau, that is +5
 - if a card is being played from the Suitstacks to the Tableau, that -10
 - OTHERWISE it is +0
Priority:
 - For a card being played onto the Tableau,
    - If we are moving the card across the Tableau, then if this card would flip a face down card, its priority is k+1 where k is the number of total face down cards in the pile where the card is being flipped
    - If we are playing from the hand and the card is not a King (or 10 in Italian decks) then the priority is 1
    - If it is a King (or 10):
       - then if the Queen (or 9) with which it may become braided (i.e. opposite color) is anywhere except face down in the Tableau, then the priority is 1
       - otherwise it is -1
  - OTHERWISE it is +0
2. Nicholas Inzunza:
Heuristic:
 - if a card is moving from the Tableau and would flip a card face up, then it is 5+k where k is the number of face down cards in its pile
 - otherwise, if it is moving into the Suitstacks, it is +5
 - all other scenarios are +0
Priority:
 - We chose to parrot priorities of Yan et al 2005 involving Kings in the Hand, but in general priority from the Hand is 0
 - A card played from anywhere which when played has the potential to move a card which would unblock a face down card from the Tableau has priority k+1 where k is the number of face down cards in the pile which would be unblocked
 - A card which is otherwise needlessly moved from the Suitstacks has priority -5
 - Every other scenario has priority 0

The strategy implemented by the priorities may perhaps be cheating, as this would effectively look ahead by one move, but the information which it uses is only that which is currently available.

The winrates and number of moves it takes to win are compared in ```statistics.ipynb``` but are reiterated here.

In the modified version of solitaire which we simulated, the Yan strategy had a winrate of ~20% using an Italian deck.  Our strategy had a winrate of ~73.6%.  We took 1000 trials.  The average number of moves it took the Yan strategy to win was ~55 while ours was ~40.  Using the full deck, the winrates were ~6.5% for the Yan strategy and ~53% for our strategy with an average of 73 and 55 moves to win, respectively.
