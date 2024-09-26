# Tron Game
this game consists of two real-time agents which are trying the create as more walls than their opponent while preventing collision into each other and into boundery walls.The unity Framework is from chillin monitors games.

## Game Start
At the beginning of the game, each agent starts at a specific point with a predetermined direction. The map is surrounded by an AreaWall. Additionally, some points inside the map may also contain AreaWall. The maps are always symmetrical, ensuring that neither team has an advantage at the start. Therefore, when implementing your code, you don't need to worry about whether the team is Yellow or Blue.

## Game Actions
Change Direction:

The agent can change its direction to one of four directions: up, right, down, or left.
Changing direction to the opposite of the current direction is not allowed (e.g., if you're moving right, you cannot change to left).
Activate Wall Breaker:

By activating this feature, you can break through YellowWall or BlueWall without losing any Health, converting them to Empty cells.
Once activated, this ability remains active for a limited time. After it ends, you must wait some time before you can activate it again.
This feature does not affect collisions with the opponent's Agent or AreaWall.
## Movement:

In each cycle, the agent moves one cell forward in its current direction.
With each move, a wall is built in the previous cell.
If the new cell you move into is an AreaWall, you die, and your score decreases by a certain amount.
If the new cell is either a YellowWall or BlueWall:
If the wall breaker is activated or the agent has more than one Health, the wall breaks, and the team that owned the wall loses points equivalent to the score for wall construction (e.g., regardless of which team you belong to, encountering a YellowWall deducts points from the Yellow team).
If the wall breaker is not activated, and the agent has only one Health left, the agent dies, and the score of the team that owned the wall decreases.
Scoring System
The score for each team is calculated by multiplying the number of walls belonging to that team by a certain coefficient. Additionally, if an agent dies, the team's score is reduced based on the cause of death.

## Game Termination Conditions
The game ends under any of the following conditions:

If the number of cycles reaches the maximum limit.
If the agents collide with each other.
If an agent collides with an AreaWall.
If a team's Health reaches zero.

## Game Result
The team with the higher score at the end of the game is declared the winner.

## Game Entities
The AI class contains two functions: initialize and decide:

The initialize function is executed only once at the start of the game (cycle 0, before the decide function), where initial values can be set (this function is optional).
The decide function is executed once per game cycle, where the AI logic should be implemented. The game is real-time, meaning both teams can act in each cycle.
The AI class has the following properties:

World: Represents the game world.
my_side: The name of your team (Yellow or Blue).
other_side: The name of the opponent team.
current_cycle: The current cycle number of the game.
cycle_duration: The decision-making time per cycle in seconds.
World Class
The World class contains all game-related entities and their properties:

board: A 2D array representing the game board.
agents: A map containing agent information for each team (the team name as the key and the agent as the value).
scores: A map containing the scores of each team (the team name as the key and the score as the value).
constants: A Constants object holding constant values.
The board is a 2D array of type ECell, with the origin located at the top left corner.

## ECell Enum
This enum represents the type of cell on the board:

Empty: An empty cell.
AreaWall: A wall around the map.
BlueWall: A wall belonging to the Blue team.
YellowWall: A wall belonging to the Yellow team.

![image](https://github.com/user-attachments/assets/c31782ce-7d77-4037-8bea-be87164ba720)


## Agent Class
The Agent class contains the following properties:

health: The remaining health of the agent.
position: The position of the agent.
direction: The direction in which the agent is moving.
wall_breaker_cooldown: The number of cycles remaining before the wall breaker can be activated again.
wall_breaker_rem_time: The number of cycles left for the wall breaker to remain active.

## Position Class
The Position class contains the following properties:

x: The horizontal position of the agent.
y: The vertical position of the agent.

## EDirection Enum
This enum represents the direction of movement of the agent:

Up: Upward direction.
Right: Rightward direction.
Down: Downward direction.
Left: Leftward direction.

## Constants Class
The Constants class holds values that do not change during the game:

max_cycles: The maximum number of cycles in the game.
init_health: The initial health of the agents.
wall_breaker_cooldown: The number of cycles needed to reactivate the wall breaker.
wall_breaker_duration: The number of cycles the wall breaker remains active.
wall_score_coefficient: The score coefficient for each wall belonging to a team.
area_wall_crash_score: The score deducted if an agent dies from hitting the AreaWall.
my_wall_crash_score: The score deducted if an agent dies from hitting its own team's wall.
enemy_wall_crash_score: The score deducted if an agent dies from hitting the opponent's wall.

## Code Execution Duration

The duration of the first game cycle (Cycle 0) is 3 seconds, during which the `initialize` and `decide` functions will be executed. This cycle, being longer, is suitable for heavy preprocessing and storing results for use in subsequent cycles.

The execution duration for other cycles (from Cycle 1 onwards) is 0.5 seconds, and only the `decide` function is executed in these cycles.

## Sequence of Events in the Game
1. Execution of commands, if they are valid.
2. Checking the remaining cooldown time and the status of the wall-breaking ability.
3. Building a wall at the current location of the agents.
4. Moving the agents forward based on their direction.
5. Checking for collisions with the opponent's agents.
6. Checking for collisions with the environment walls.
7. Checking for collisions with friendly or enemy walls, breaking them, or dying based on the specified conditions.


# Algorithm
The algorithm we devised for this game is a combination of Genetic Algorithm and Minimax, where the Minimax algorithm is used as the fitness function for the Genetic Algorithm. Given that each cycle takes 8 seconds, we build and evaluate a tree with a depth of 6 in each cycle, as deeper trees require more time. In each tree, even depths belong to the actions of our agent, while odd depths correspond to the actions of the opponent. Despite the game being real-time, we had to consider it as turn-based for using Minimax. Therefore, we assume that we make a move first (at depth 0 of the Minimax tree), and the opponent decides their next move afterward, giving them an advantage at each stage. With this tree, we can predict the next three moves of both our agent and the opponent (keep in mind that we are blind to the opponent's agent during the game, and we don't know where its leading part is, only being aware of the cells it has built).
Regarding the implementation of the Genetic Algorithm, one strategy is to pass the fittest offspring to the Minimax and act based on the move predicted by this function.
To implement the algorithms, it is necessary to access the features of the world and the agent, and review their functions without sending these modification commands to the server. Therefore, to better simulate the game environment for the Minimax function, we define a package named classes in the pythonClient folder, which includes two files: world and agent. Each of these two classes, which were previously defined in models and the pythonServer folder, has been redefined and reconstructed according to the functionality intended for them.
### PythonClient/Classes/Worlds.py:
In this section, using the required parts of the game defined in ks, we proceed to reconstruct the game world.
First, we define the get_d_cords function, which adjusts the agent's coordinates based on the direction of movement.
In the change_board function, we implement the game moves and score calculation. If our agent's wall_breaker is on, it prioritizes moving into cells owned by the opponent, then empty cells, and lastly cells owned by itself. If no option is possible, our agent hits the area wall, and the game ends. If the wall_breaker is off, the priority is to move into empty cells if possible; otherwise, the agent will try to turn on the wall_breaker and move into the opponent's cell, or move into its own cell as a last resort, or, in the worst case, hit the area wall. If it's not possible to turn on the wall_breaker and no empty cell is available, the order of visiting cells remains the same. The score for the simulation is calculated by adding one point for each new wall built by us and subtracting one point if the opponent's wall is destroyed. If the agent enters a walled cell without turning on the wall_breaker, it loses 150 points due to the loss of one health. The reason for considering this amount for health loss is that it has a significant score multiplier during the game, and we only have 3 health points in total. Once reduced to zero, our agent loses, and the game ends. Thus, by considering this multiplier, we intend to inform the agent to avoid such actions as much as possible.
By changing the depth of the tree, the agents' information in the swap function needs to be updated to predict the opponent's next move.
### PythonClient/ai.py:
In this file, we have implemented the game's Minimax algorithm.
We have defined the following functions for this purpose:
### get_next_nodes:
This function is defined as an auxiliary function for Minimax. It initially defines a list as the best answer with the initial value of the Minimax function. This function then deep copies the game world for each depth in the Minimax tree, depending on whether the depth is for the opponent or our agent, rebuilds the Minimax tree, calls the Minimax function, assigns a new value to the best answer, and returns it.
Update_world_and_agent:
As mentioned earlier, we need to simulate the game's environment for the Minimax prediction. This function simulates the features of our agent, the opponent's agent, and the game world with the desired values for these features.
### Minimax:
In this function, we aim to implement the Minimax function in such a way that the difference in points between the two agents is maximized. For this purpose, we build a tree with a depth of 6.
If we reach depth 6, we return a list of moves that maximize the point difference. Then, we check whether the game termination conditions are met. These conditions are:
 - An agent collides with the area wall.
 - The cycles are finished.
 - The two agents collide with each other.
 - One agent's health is reduced to zero.
As explained in the world class regarding the priority of the agent's movement in each situation, we guide it accordingly. Note that if only one cycle is left for the wall_breaker, our priority in the next cycle is to enter empty cells to preserve the agent's health.

![image](https://github.com/user-attachments/assets/5e1e6b53-c87d-4fcf-aaff-9f0ff7e3359a)

![image](https://github.com/user-attachments/assets/b49f835e-056f-4925-a2ba-f28548baeb80)



