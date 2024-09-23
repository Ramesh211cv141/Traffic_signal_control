# Deep Q-Learning Agent for Traffic Signal Control
A framework where a deep Q-Learning Reinforcement Learning agent tries to choose the correct traffic light phase at an intersection to maximize traffic efficiency.

## The code structure
The main file is training_main.py. It handles the main loop that starts an episode on every iteration. It also saves the network weights and three plots: negative reward, cumulative wait time, and average queues.

Overall the algorithm is divided into classes that handle different parts of the training.

The Model class defines everything about the deep neural network, and it also contains some functions used to train the network and predict the outputs. In the model.py file, two different model classes are defined: one used only during the training and only during the testing.
The Memory class handle the memorization for the experience replay mechanism. A function adds a sample into the memory, while another function retrieves a batch of samples from the memory.
The Simulation class handles the simulation. In particular, the function run allows the simulation of one episode. Also, other functions are used during run to interact with SUMO, for example: retrieving the state of the environment (get_state), set the next green light phase (_set_green_phase) or preprocess the data to train the neural network (_replay). Two files contain a slightly different Simulation class: training_simulation.py and testing_simulation.py. Which one is loaded depends if we are doing the training phase or the testing phase.
The TrafficGenerator class contains the function dedicated to defining every vehicle's route in one episode. The file created is episode_routes.rou.xml, which is placed in the "intersection" folder.

# The Deep Q-Learning Agent
## Framework: Q-Learning with deep neural network.

Context: traffic signal control of 1 intersection.

Environment: a 4-way intersection with 4 incoming lanes and 4 outgoing lanes per arm. Each arm is 750 meters long. Each incoming lane defines the possible directions that a car can follow: left-most lane dedicated to left-turn only; right-most lane dedicated to right-turn and straight; two middle lanes dedicated to only going straight. The layout of the traffic light system is as follows: the left-most lane has a dedicated traffic-light, while the other three lanes share the same traffic light.

Traffic generation: For every episode, 1000 cars are created. The car arrival timing is defined according to a Weibull distribution with shape 2 (a rapid increase of arrival until the mid-episode, then slow decreasing). 75% of vehicles spawned will go straight, 25% will turn left or right. Every vehicle has the same probability of being spawned at the beginning of every arm. In every episode, the cars are randomly generated, so it is impossible to have two equivalent episodes regarding the vehicle's arrival layout.

Agent ( Traffic Signal Control System - TLCS):

State: discretization of oncoming lanes into presence cells, which identify the presence or absence of at least 1 vehicle inside them. There are 20 cells per arm. 10 of them are placed along the left-most lane while the other 10 are placed in the other three lanes. 80 cells in the whole intersection.

Action: choice of the traffic light phase from 4 possible predetermined phases, described below. Every phase has a duration of 10 seconds. When the phase changes, a yellow phase of 4 seconds is activated.
North-South Advance: green for lanes in the north and south arm dedicated to turning right or going straight.
North-South Left Advance: green for lanes in the north and south arm dedicated to turning left.
East-West Advance: green for lanes in the east and west arm dedicated to turning right or going straight.
East-West Left Advance: green for lanes in the east and west arm dedicated to turning left.

Reward: change in cumulative waiting time between actions, where the waiting time of a car is the number of seconds spent with speed=0 since the spawn; cumulative means that every waiting time of every car located in an incoming lane is summed. When a car leaves an oncoming lane (i.e. crossed the intersection), its waiting time is no longer counted. Therefore this translates to a positive reward for the agent.

Learning mechanism: the agent make use of the Q-learning equation Q(s,a) = reward + gamma • max Q'(s',a') to update the action values and a deep neural network to learn the state-action function. The neural network is fully connected with 80 neurons as input (the state), 5 hidden layers of 400 neurons each, and the output layers with 4 neurons representing the 4 possible actions. Also, an experience replay mechanism is implemented: the experience of the agent is stored in a memory and, at the end of each episode, multiple batches of randomized samples are extracted from the memory and used to train the neural network, once the action values have been updated with the Q-learning equation.
