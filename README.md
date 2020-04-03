# SARL AirSim interface

![SARL Airsim multiagent simulation](https://i.imgur.com/JGREzTo.png "SARL Airsim multiagent simulation")

Example of usage of the AirSim simulation environment with SARL agents.

## Principles

The simulation is performed within AirSim (computation of the collision, of the perceptions, etc.) and
the decision are taken by SARL agents (the intelligent behavior).

So, AirSim provides perceptions, SARL agent computes and emits influences, AirSim computes the reactions and the new perceptions,
and so on. 

Between the SARL individual agents and AirSim, there is the **SimulationControllerAgent**.

The purpose of this agent is to control the simulation. It uses strategies to define:
	- How the the influences are propagated to AirSim (immediately, or under some conditions)
	- How the perceptions are propagated from AirSim to the agents
	- How the bodies are linked to the agents
	- How the simulation will run (real-time, fixed-step, or more complex strategies)
	
The perceptions and influences are supposed to be propagated using SARL events.

The **BootAgent** is in charge of spawning the SimulationControllerAgent and all the required agents. 

## Working on the drone agents

If you run this example, agents will be spawned, able to manage a drone.
The drones are able to perceive the environment, to communicate with the other drones, and to emit influences.

The behaviors of the drones are located in the package **io.sarl.airsim.behaviors**.
The perceptions events are located in **io.sarl.airsim.perceptions** (it's up to the SimulationControllerAgent
to define which perceptions will be provided to the drones).
The influences events are located in **io.sarl.airsim.influences** (influences will be transmitted by
the SimulationControllerAgent and should be transmitted as-is, but it does not mean they will be applied,
the application of the influences is up to AirSim).
The communication events are located in **io.sarl.airsim.communication**.

## Creating a simulation from scratch

The example is made to work with AirSim and with drones, but the framework can be easily adapted so
you can make your own simulations.

You should have a good understanding of this simulation framework before trying to do it on your own,
but here is a list of things to do to make your own simulation. 

### Defining the perceptions

You should start by defining which perceptions your agents should be able to receive.
Of course, you will be limited by the possibilities of AirSim (it provides a lot of sensors and possibilities, but not __everything__).

Once the list of available perceptions is set, you need to:
	- Create the associated events (look at io.sarl.airsim.perceptions for inspiration)
	- Allow them to be retrieved by AirSim in SimulationControllerAgent and propagated to the agents (look at
	  SimulationControllerAgent **retrieveAgentPerceptions**)

### Defining the influences

Then, the approach is similar with influences:
	- Define which kind of influence the agents should be able to express (with AirSim possibilities as a limitation)
	- Create the associated events
	- Allow them to be emitted by the SimulationControllerAgent to AirSim: the SimulationControllerAgent must be listening
	  to influence events and emit them to AirSim, you can see how it's done currently in the **on MoveByVelocity** and others
	  of the SimulationControllerAgent 

### Defining the behaviors and spawning your agents

Now that the list of available perceptions and influences is set, you can link them by defining the agents
behaviors. You can look at **io.sarl.airsim.behaviors.ForwardWithBasicAvoidance** for a simple example.

Then, this behavior can be attached to an agent, like in **DroneAgent**.
The agent with the attached behavior could be created by the BootAgent.

### Defining the simulation control strategies

In the SimulationControllerAgent, there are several strategies which can be defined to specify how the simulation should run.

The choice of a strategies can heavily impact the realism of the simulation. These settings should be chosen wisely.

One which is definitely required to override or adapt is the allocation strategy.

#### Allocation strategy

The allocation strategy provides a way to define how the agents are attached to bodies.
Note: it's not possible with AirSim, for the moment, to dynamically spawn a body, to it must exists in the simulation environment to
be spawned.

A strategy is provided here: the first-come first-served, fixed pool strategy: the pool is filled with a list of available bodies
at the beginning, then every-time an agent emits an influence, it will be attached to a body from the pool (if it's not already
attached to one).

For your own simulation, you should at least adapt the pool with the name of the bodies of your simulation.

If you need to go further, you could also override the strategy and provide one where the agents will be attached to bodies
as soon as they are spawned (and not only when they want to express an influence).

#### Influence/Reaction strategy

The influence/reaction strategy describes how and when the influences are emitted toward AirSim.

With the **DirectInfluenceReactionStrategy**, the influences are transmitted by the SimulationControllerAgent as soon as they are received.

With the **BatchedInfluenceReactionStrategy**, the influences are stored in a collection before being transmitted. The size of the 
collection holding the influences can be changed, as well as the condition triggering the emission of the influences. This one can
be useful, if you want to prevent the influences of some agents from being emitted if other agents are still computing their influence. 

#### Simulation strategy

The simulation strategy describes the global simulation loop.

With the **RealTimeSimulationStrategy**, the loop body consists of retrieving the perceptions. The simulation is supposed to be run in real-time.

With the **FixedStepSimulationStrategy**, the loop body will pause the simulation, retrieve the perceptions, continue the simulation
for a given amount of time (the step time), and wait until this time is over (before starting a new loop).

With the **FixedStepBlockingSimulationStrategy**, it's similar, except that we can provide a condition for the loop continuation (and 
block the simulation until this condition is met).
It's used in the current example with the condition being "if some agents have not expressed their influences, don't continue", which could be
translated to "continue as soon as every one has expressed an influence".

Warning: if you are using DirectInfluenceReactionStrategy with RealTimeSimulationStrategy, you may expect your agents to react like they would
in a real scenario (i.e. running with a physical body and not a simulated body), with the idea that the simulation will continue even if
they don't express influences (like in the real world: if the agent don't move, the world will not stop for him). But, actually there is a bias.
The perceptions will take some time to be retrieved from AirSim, a time which is probably different from the time required to get the perceptions
from real sensors. During this time, the agents won't be able to make a decision, even though the simulated world will continue to evolve.
This can lead to inconsistency in the simulation results. That's why, it's better to use a fixed step simulation strategy where the simulation
won't continue while the perceptions are retrieved. It's up to the agents or the SimulationControllerAgent to emulate the sensor lag if required.

## Citation

If you're using this repository in your research or in your work, please use the following for citations:

    @misc{alombardSarlAirsim19,
        author = {Alexandre Lombard},
        title = {S{A}{R}{L} {A}ir{S}im interface},
        year = {2019},
        publisher = {GitHub},
        journal = {GitHub repository},
        howpublished = {\url{https://github.com/alexandrelombard/sarl-arisim-interface}}
    }

Thank you!
