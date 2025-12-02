Group Members: Sofia McDonough, Afsana Mimi Raka, Nicholas Inzunza

Topic: Human vs Computer: Predicting and Solving Solitaire

Goal statement: This project aims to compare the win rates between human players and computers in Solitaire.  

Planned directory structure of the project: Solitaire/ src/ computer_solver.py # Strategic solver that analyzes the current state and chooses the move leading to the largest progress human_solver.py # Rule-based player that moves all possible cards to the foundation immediately ml_model.py # Machine learning model predicting the solvability of a given initial deck. win_rate_comparison.py # Scripts to record and visualize win rate differences between human and computer solvers data/ generate_initial_state.py # Generate multiple shuffled initial decks for simulation initial_state/ # Store generated initial deck configurations initial_state_vs_solvable.csv # Record each deck ID with its solvability label (solvable or not) results/ win_rate_comparison.png # Show win rate comparisons ml_confusion_matrix.png # Record prediction results
