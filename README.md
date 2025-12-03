#Group Members: Sofia McDonough, Afsana Mimi Raka, Nicholas Inzunza

#Topic: Human vs Computer: Predicting and Solving Solitaire

#Goal statement: This project aims to compare the win rates between human players and computers in Solitaire.  
#Planned directory structure of the project: Solitaire/ src/ computer_solver.py # Strategic solver that analyzes the current state and chooses the move leading to the largest progress human_solver.py # Rule-based player that moves all possible cards to the foundation immediately ml_model.py # Machine learning model predicting the solvability of a given initial deck. win_rate_comparison.py # Scripts to record and visualize win rate differences between human and computer solvers data/ generate_initial_state.py # Generate multiple shuffled initial decks for simulation initial_state/ # Store generated initial deck configurations initial_state_vs_solvable.csv # Record each deck ID with its solvability label (solvable or not) results/ win_rate_comparison.png # Show win rate comparisons ml_confusion_matrix.png # Record prediction results

# the only package that this script requires is the random package
# otherwise, base python has everything we need
import random

 
# here are objects which are globally defined

class Constants:
    SUITS = ["Spades", "Clubs", "Diamonds", "Hearts"] # the suits
    VALUES = range(1, 14) # number of cards (can be changed)
    SYMBOLS = ['A','2','3','4','5','6','7','8','9','10','J','Q','K'] # symbols associated with each card
    # POSITIONS = ["Hand","Tableau","Suitstacks"] # these are what I named each position of the board
    # the hand means the draw deck, the tableau means the table of cards in piles,
    # the suitstacks are where the suits are organized to win

    # here are where legal moves for the game are defined
    # it takes ``cartota`` ``playota`` and ``board`` as input
    # sorry for the weird and annoying names of the variables
    # the first two objects, ``cartota and ``playota`` are tuples
    # cartota = (Card, (position, extra position info)) and same formatting for ``playota``
    # the cartota is the card which is being played
    # and the playota is the card where playing on
    # the Card class is defined below
    # the position is a string, ^see above line POSITIONS
    # the extra info means where in that position our card is
    # for the Hand it is always None, for the Tableau, it says which pile we are in
    # for the Suitstacks it says which suit we are in
    # this function is called in the ``check_playable()`` method of the Board class (defined below)
    def legal_move(cartota, playota, board): # returns whether cartota can be played on playota, True or False

        card = cartota[0]
        location = cartota[1][0]
        loc_extra = cartota[1][1]

        playon_card = playota[0]
        playon_loc = playota[1][0]
        playon_extra = playota[1][1]


        if playon_loc == "Tableau": # if we are playing on the Tableau
            if playon_card.empty: # if that spot in the Tableau is empty
                return card.value == 13 # only the King can go there
            
            # otherwise tell us if the cards are "braided"
            # meaning opposite color and descending value
            else: return playon_card.value - card.value == 1 and card.color != playon_card.color
            
        else: # playon_loc == "Suitstacks": # we are playing on the suitstacks
            if playon_extra == card.suit: # the card must go in the appropriate suitstack, i.e. matching suits
                if playon_card.empty: return card.value == 1 # if the stack is empty, only the Ace goes there

                # if we are playing from the Tableau, double check that we are using an unblocked card
                # otherwise, the location is the hand
                elif location == "Tableau" and board.tableau.piles[loc_extra].face_up[-1] == card or location == "Hand":
                    return card.value - playon_card.value == 1 # the cards in the suitstacks must be in ascending order
                
                else: return False

            else: return False
           

# this is the Card class
# a non-empty card has a value and a suit
# otherwise, it is empty.  it is useful to define empty spaces for the solitaire rules^
class Card:
    def __init__(self, value=None, suit=None, empty=False):
        if empty==True:
            self.empty=True
            self.name = "Empty"
        else:
            self.suit = suit
            if self.suit == "Spades" or self.suit == "Clubs": self.color = "Black"
            else: self.color = "Red"
            self.value = value
            self.name = str(Constants().SYMBOLS[self.value-1])+" of "+self.suit # this makes it easy to identify cards in a print
            self.empty=False

# the deck class essentially tells us how to shuffle and deal
# if we want to specify some deck order, then we can do so.
class Deck:
    def __init__(self, desired_order=None):
        self.cards = []

        if desired_order is not None: self.cards = desired_order

        else: # just put the cards in order
            for suit in Constants().SUITS:
                for value in Constants().VALUES:
                    self.cards += [Card(value,suit)]
                      
    def shuffle(self): # then shuffle them
        random.shuffle(self.cards) # the ``shuffle`` method of ``random`` does exactly that

    # here is the particular way in which cards are dealt
    def deal(self):
        num_piles = 7 # 7 piles
        num_tableau = int(num_piles/2*(num_piles+1)) # 28 total in the tableau
        piles=[]
        for i in range(num_piles): # this loop creates the piles
            cards=self.cards[int(i/2*(i+1)):int((i+1)/2*(i+2))]
            if i==0: face_up,face_down = (cards,[]) # the first pile has no face down cards
            else: face_up,face_down=(cards[-1:],cards[:-1]) # otherwise only the last card is face up

            # the Pile class is defined below, it is initialized with face up cards, face down cards, and an index
            # which indicates which pile it is on the tableau
            piles += [Pile(face_up_cards=face_up, face_down_cards=face_down, pile_index=i)]

        # the Tableau class is defined below, it is initialized with a list of piles
        tableau = Tableau(piles=piles)

        # the Hand class is defined below, it is initialized with a list of cards
        hand = Hand(self.cards[num_tableau:]) # the rest of the cards not in the Tableau are put into the Hand

        # the Suitstacks class is defined below as well as the Stack class
        # Suitstacks are initialized with stacks and a Stack is initialized with a Suit and a potential list of cards
        # here, when the cards are dealt, all stacks are empty, but we can initialize a board with nonempty stacks
        suitstacks = Suitstacks(stacks=[Stack(suit=suite) for suite in Constants().SUITS])

        # the Board class is defined below, it is initialized with a Tableau, a Hand, and a Suitstacks
        # this is meant to represent a game of Solitaire
        return Board(tableau=tableau, hand=hand, suitstacks=suitstacks)
    
