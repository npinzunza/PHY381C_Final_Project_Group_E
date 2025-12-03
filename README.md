#Group Members: Sofia McDonough, Afsana Mimi Raka, Nicholas Inzunza
#Topic: Human vs Computer: Predicting and Solving Solitaire
#Goal statement: This project aims to compare the win rates between human players and computers in Solitaire.  
#
Planned directory structure of the project: Solitaire/ src/ computer_solver.py # Strategic solver that analyzes the current state and chooses the move leading to the largest progress human_solver.py # Rule-based player that moves all possible cards to the foundation immediately ml_model.py  

 
class Card:
    def __init__(self, rank, suit, is_face_up=False):
        self.rank = rank      # e.g., 'A', '2', ..., 'K'
        self.suit = suit      # e.g., 'H', 'D', 'C', 'S'
        self.is_face_up = is_face_up

    # ... Methods for color, comparison, and flipping ...

 
class GameState:
    def __init__(self, deal):
        # The main data structure for the board:
        self.tableau = [[] for _ in range(7)] 
        self.foundation = [[] for _ in range(4)]
        self.stock = []
        self.waste = []
        # ... Function to deal the initial 52 cards ...

    def get_legal_moves(self):
        # Returns a list of all possible move actions (e.g., move_stack, flip_card, etc.)
        # This is the branching factor for the search tree.
        pass

    def apply_move(self, move):
        # Returns a NEW GameState object resulting from the move (critical for search)
        pass
