# MASSim Scenario Documentation

## Agents in the City (2017)

### Background Story

_In the year 2044, following the great water conflicts of 2033, humankind has finally colonized and terraformed Mars; so much in fact that it greatly (structurally) resembles parts of old Earth. Unfortunately, scientists are still working on making the atmosphere breathable, so humans still need (costly) special equipment to leave their homes. Luckily, some entrepreneurial individuals found remnants of the old 'All Terrain Planetary Vehicles' which they retrofitted to do some serious business. Wanting to improve life for everyone on Mars (and to make a living off of that), they are sending their ATPVs out to gather the planet's natural resources and supply everyone in need with useful items._

### Introduction

The scenario consists of two __teams__ of a number of agents each moving through the streets of a realistic city. The goal for each team is to earn as much money as possible, which is rewarded for completing certain __jobs__. An agent is characterized by its __battery__ (i.e. how long it can move without recharging), its __capacity__ (i.e. how much volume it can carry) and its __speed__. The scenario features 4 distinct __roles__: drones, motorcycles, cars and trucks, sorted by increasing capacity and energy, and decreasing speed.
The city map is taken from __OpenStreetMap__ data and routing is provided by the Contest server. As the simulation is divided into discrete steps, each agent can move a fixed distance each turn (unless its destination is closer than this distance).

Each simulation features a set of random __items__. Items differ by their volume and how they can be acquired.

The agents are positioned randomly on the map, as are a number of __facilities__, among them __shops__, __charging stations__, __workshops__, __resource nodes__, __storage__ facilities, and __dumps__.

__Jobs__ comprise the acquisition, assembling, and transportation of goods. These jobs can be created by either the system (environment) or one of the agent teams. There are two types of jobs: __regular jobs__ and __auctions__.

A team can accept an __auction__ job by bidding on it. The bid will be the reward, i.e. the team is willing to procure the items for that amount of money. Thus, if both teams bid, the lowest bid wins. If a job is not completed in time, the corresponding team is fined to discourage auction “hoarding”.

__Regular jobs__ have their rewards defined upfront, which is given to the first team to complete that job, while the other team goes away empty-handed. The teams have to decide which jobs to complete and how to do that.
While auction jobs have to be more thoroughly assessed in order to determine a minimum threshold at which the team would still earn money, they provide some safety since the team that did not win the auction is effectively barred from completing it.

__Tournament points__ are distributed according to the amount of money a team owns at the end of the simulation. To get the most points, a team has to beat the other, as well as surpass the seed capital given to the team at the beginning of the simulation, i.e. make an overall profit.

## Locations

Locations in the scenario are given as pairs of latitude and longitude double values. To simplify reaching a location, these values are rounded and approximated with the help of two parameters:

__Proximity:__ The proximity value specifies which precise locations are still considered the same, which is important when an agent needs to be at the same location as e.g. another agent or a facility.

Latitude and longitude are divided by the proximity value and rounded to the nearest number. Thus, each location _snaps_ to an intersection in a more coarse grid. All locations snapping to the same intersection are considered to be 'equal'.

In other words, __proximity__ determines how many decimal places of the latitude and longitude values are compared.

_Example:_ 51.1111 and 51.1112 would still be considered equal with a proximity of 0.001 (both values are multiplied with 1000 and rounded to 51,111) and different with a proximity of 0.0001 (both values are multiplied with 10,000, which already yields two different integers).

__CellSize:__ The cell size specifies the length an agent with speed 1 could travel in a single step. To simplify distance calculation, we assume latitude and longitude to form a uniform grid. The cell size is given in degrees. (For example, with a cell size of 4.5, an agent could travel around the world in 80 steps.)

## Items

Each simulation features a (random) set of item types. Items have a unique __name__ and a __volume__, which comes into play when items must be carried or stored.

Also, many items need to be _assembled_ from other items. Each of those
items has a number of __requirements__, i.e. quantities of other items which need to be present to assemble the item. These items are consumed during the process. Also, some items require certain tools.

### Tools

Tools are special types of items.
* They never need to be assembled but are requried to assemble other items.
* Each tool is only usable by a subset of all agent roles.
* Assembly requires at most 1 of any tool type (but possibly multiple types).
* Tools are not destroyed/consumed during assembly.

## Facilities

Facilities are placed randomly on the map. Each facility has a unique name and location.

#### Shops

In __shops__, items can be bought at prices specific to the particular shop. Each shop only offers limited quantities of a subset of all items. However, items are restocked after a number of steps.

E.g. if a shop has __restock__ 5, one piece of each missing item is added to the shop's stock each 5 steps.

#### Charging stations

__Charging stations__ have to be frequently visited by agents in order to recharge their battery. They have a __rate__ which expresses the amount of charge that
can be restored in one step (i.e. a charging station with a rate of 40 would
restore 40 charge units of an agent's battery).

#### Workshops

In __workshops__, agents can assemble items out of other required items. Most items also need specific tools, which can only be used by a particular role. Agents may (and might have to) cooperatively assemble. In that case, the resources of those agents are combined.

* The __initiating agent__ receives the finished item (if all prerequisites are satisfied).
* Items are preferably taken from the initiating agent.
* If multiple __assistants__ carry the same required item, it is not guranteed which item is consumed.

#### Dumps

Items can be destroyed at __dumps__ (to free capacity). Those items cannot be retrieved.

#### Storage

__Storage__ facilities allow to store items up to a specific volume and also are the target for completing jobs.

The storage has a limited __capacity__ which counts for all teams, i.e. one team can fill a storage while not leaving any space for other teams.

Each team has a separate (unlimited) compartment in each storage. This compartment cannot be filled directly but only as a consequence of other actions:

* If a team delivers items towards job completion, but another team is faster, the partial delivery is moved to this compartment.
* If a team posted a job which another team completed, the items which were required for this job are moved to the posting team's compartment.

#### Resouce nodes

TODO

### Teams

The scenario can be played with any number of teams simultaneously. Each team contains the same number of agents per __role__.

### Roles

The roles in the scenario can be configured under the top-level `roles` key in the simulation JSON object (which is one element of the `match` array).

```JSON
"roles" : {
  "Car" : {
    "speed" : 3,
    "load" : 550,
    "battery" : 500,
    "roads" : ["road"]
  },
  "Drone" : {
    "speed" : 5,
    "load" : 100,
    "battery" : 250,
    "roads" : ["air"]
  },
  "Motorcycle" : {
    "speed" : 4,
    "load" : 300,
    "battery" : 350,
    "roads" : ["road"]
  },
  "Truck" : {
    "speed" : 2,
    "load" : 3000,
    "battery" : 1000,
    "roads" : ["road"]
  }
}
```

Each role has its name as key and the following parameters:

* __speed__: how many 'units' the agent can move in one step
* __load__: how much volume the agent may carry
* __battery__: the agent's battery size
* __roads__: which roads the agent can navigate (currently 'road'  for all roads and 'air' for travelling linear distances between two points)

These 4 roles will also be used in the contest, however, the parameters are still subject to change.

### Job
A job is the general way to earn money in this scenario.

- __begin__: the job begins in this step, i.e. this is the first step where the job
is perceived by all agents
- __end__: the last step in which the job can be completed. At the end of
this step, the job cannot be perceived anymore.
- __reward__: the amount of money that is earned by completing the job
- __storage__: to complete the job, items have to be delivered to this storage

### AuctionJob
An auction job has an initial phase in which agent teams can bid for it.
The teams are bidding the amount of money which they want to be paid for completing
the job. Of course, the team with the lowest bid wins.

This type of job has the advantage, that a team can be sure to be the only one
working on it if it won the auction. However, the teams have to estimate
how low they can bid to still have an advantage.

Each step during the auction phase, the current lowest bid and corresponding team
can be perceived by all agents.

- __begin__: the step in which the auctioning begins.
- __auctionTime__: the duration of the auction phase. The winner of the auction
will be determined at the end of step (begin + auctionTime - 1).
- __end__: the latest step in which items can be delivered to complete the job
- __reward__: for this job, the reward is the maximum reward possible, i.e. bids
that go above this value will be ignored.
- __fine__: at the end of the auction job, if it has not been completed, the
team that won the auction has to pay this fine.

## Actions

In each step, an agent may execute _exactly one_ action. The actions are gathered and executed in random order.

All actions have the same probability to just fail randomly.

Some actions may lead to __conflicts__. For example, two agents might want to buy the same item from a shop (and only one of these items is left). In that case, the agent whose action is executed first gets the item, while the action of the other agent is treated as though no item is available (which is actually true).

Each action has a number of `parameters`. The exact number depends on the type of action. Also, the position of each parameter determines its meaning. Parameters are always string values.

#### goto
Moves the agent towards a destination. Consumes __10__ charge units if successful. Can be used with 0, 1 or 2 parameters. If 0 parameters are used, the agent needs to have an existing route which can be followed.

No | Parameter | Meaning
--- | --- | ---
0 | Facility | The name of a facility the agent wants to move to.

No | Parameter | Meaning
--- | --- | ---
0 | latitude | The latitude of the agent's desired destination.
1 | longitude | The longitude of the agent's desired destination.

Failure Code | Reason
--- | ---
failed_wrong_param | The agent has no route to follow (0 parameters), more than 2 parameters were given or the given coordinates were not valid double values (2 parameters).
failed_unknown_facility | No facility by the given name exists (1 parameter).
failed_no_route | No route to the destination exists or the charge is insufficient to reach the next waypoint.

#### give

Gives a number of items to another agent in the same location.

No | Parameter | Meaning
--- | --- | ---
0 | Agent | Name of the agent to receive the items.
1 | Item | Name of the item to give.
2 | Amount | How many items to give.

Failure Code | Reason
--- | ---
failed_wrong_param | More or less than 3 parameters have been given, no agent by the name is known or an amount < 0 was specified.
failed_unknown_item | No item by the given name is known.
failed_counterpart | The receiving agent did not use the receive action.
failed_location | The agents are not in the same location.
failed_item_amount | The giving agent does not carry enough items to give.
failed_capacity | The receiving agent could not carry all given items.

#### receive

Receives items from other agents. Can receive items from multiple agents in the same step.

No parameters.

Failure Code | Reason
--- | ---
failed_counterpart | No agent gave items to this agent.

#### store

Stores a number of items in a storage facility.

No | Parameter | Meaning
--- | --- | ---
0 | Item | Name of the item to store.
1 | Amount | How many items to store.

Failure Code | Reason
--- | ---
failed_wrong_param | More or less than 2 parameters were given.
failed_location | The agent is not located in a facility.
failed_wrong_facility | The agent is not in a storage facility.
failed_unknown_item | No item by the given name is known.
failed_item_amount | The given amount is not an integer, less than 1 or greater than what the agent is carrying.
failed_capacity | The storage does not have enough free space.
failed | An unforeseen error has occurred.

#### retrieve & retrieve_delivered

Retrieves a number of items from a storage. The first can be used to retrieve items that have been stored before, while the second is used to retrieve items from the team's 'special' compartment (see [Storage](#storage)).

No | Parameter | Meaning
--- | --- | ---
0 | Item | Name of the item to retrieve.
1 | Amount | How many items to retrieve.

Failure Code | Reason
--- | ---
failed_wrong_param | More or less than 2 parameters have been given.
failed_location | The agent is not located in a facility.
failed_wrong_facility | The agent is not in a storage facility.
failed_unknown_item | No item by the given name is known.
failed_item_amount | The given amount is not an integer, less than 1 or more than available.
failed_capacity | The agent has not enough free space to carry the items.

#### assemble

Assembles an item.

No | Parameter | Meaning
--- | --- | ---
0 | Item | Name of the item to assemble.

Failure Code | Reason
--- | ---
failed_wrong_param | Not exactly 1 parameter has been given.
failed_location | The agent is not in a facility.
failed_wrong_facility | The agent is not in a workshop.
failed_unknown_item | No item by the given name is known.
failed_item_type | The item cannot be assembled (since it has no requirements).
failed_tools | Some tool is missing.
failed_item_amount | At least one required item is missing.
failed_capacity | The agent does not have enough free space to carry the assembled item (after required items have been removed).

#### assist_assemble

Marks the agent as an assistant for assembly.

No | Parameter | Meaning
--- | --- | ---
0 | Agent | Name of an agent who uses the _assemble_ action and whom this agent should help.

Failure Code | Reason
--- | ---
failed_wrong_param | Not exactly 1 parameter has been given.
failed_unknown_agent | No agent by the given name is known.
failed_counterpart | The initiator's action has failed or is not _assemble_.
failed_tools | Some tool is missing.
failed_location | The given agent is too far away.

#### buy

Buys a number of items in a shop.

No | Parameter | Meaning
--- | --- | ---
0 | Item | Name of the item to buy.
1 | Amount | How many items to buy.

Failure Code | Reason
--- | ---
failed_wrong_param | More or less than 2 parameters have been given.
failed_location | The agent is not in a facility.
failed_wrong_facility | The agent is not in a shop.
failed_unknown_item | No item by the given name is known.
failed_item_amount | The given amount is not an integer, less than 1, or greater than the shop's available quantity.
failed_capacity | The agent does not have enough free space to carry the items.

#### deliver_job

Delivers items towards the completion of a job. The agent is automatically drained of all items matching the job's remaining requirements.

No | Parameter | Meaning
--- | --- | ---
0 | Job | The name of the job to deliver items for.

Failure Code | Reason
--- | ---
failed_wrong_param | Not exactly 1 parameter has been given.
failed_unknown_job | No job by the given name is known.
failed_job_status | The given job is not active, or the job is an auction and has not been assigned yet or has not been assigned to the agent's team.
failed_location | The agent is not in the storage associated with the job.
partial_success | Not really a failue. Items have been delivered but the job has not been completed by this action.
useless | The agent does not have any items to contribute to the job.

#### bid_for_job

Places a bid for an auction job. The bid has to be lower than the current lowest bid.

No | Parameter | Meaning
--- | --- | ---
0 | Job | Name of the job to bid on.
1 | Bid | The bid to place.

Failure Code | Reason
--- | ---
failed_wrong_param | Not exactly 2 parameters have been given, or the bid is not a positive integer.
failed_unknown_job | No job by the given name is known.
failed_job_type | The job is not an auction.
failed_job_status | The job's auctioning phase is over.

#### post_job

Posts a job that the other teams may complete. Only regular jobs may be posted.

The number of jobs a team may have posted at any one time is limited. Also, a job cannot be retracted once posted.

A successfully posted job starts being active in the next step.

No | Parameter | Meaning
--- | --- | ---
0 | Reward | How much to pay if the job is completed successfully.
1 | Duration | After how many steps the job should end.
2 | Storage | The target storage for the job.
3, 5, 7, ... | Item | The name of an item required for the job.
4, 6, 8, ... | Amount | How many items to require.

Failure Code | Reason
--- | ---
failed_wrong_param | Less than 5 or an even number of parameters have been given.
 | Reward or duration is less than 1.
failed_wrong_facility | No storage by the given name is known.
failed_job_status | The agent's team has reached its post limit.
failed_unkown_item | Some name given is not the name of an item.
failed_item_amount | Some amount given is not a positive integer.

#### dump

Destroys a number of items at a dump facility.

No | Parameter | Meaning
--- | --- | ---
0 | Item | Name of the item to destroy.
1 | Amount | How many items to destroy.

Failure Code | Reason
--- | ---
failed_wrong_param | Not exactly 2 parameters have been given.
failed_location | The agent is not in a facility.
failed_wrong_facility | The agent is not at a dump location.
failed_unknown_item | No item by the given name is known.
failed_item_amount | The given amount is not a positive integer.

#### charge

Charges the agent's battery at a charging station.

No parameters.

Failure Code | Reason
--- | ---
failed_wrong_param | Parameters have been given.
failed_location | The agent is not in a facility.
failed_wrong_facility | The agent is not in a charging station.

#### continue & skip

Follows an agent's route or does nothing if the agent has no route.

No parameters.

Failure Code | Reason
--- | ---
failed_wrong_param | Parameters have been given.
failed_no_route | The agent's route could not be followed any longer (charge may be too low).

#### abort

Does nothing and clears the agent's route (if it exists).

#### unknownAction

This action is substituted if an agent submitted an action of unknown type.

#### randomFail

This action is substituted if the agent's action randomly failed.

#### noAction

This action is substituted if the agent did not send an action in time.

## Percepts

TODO

## Configuration

Each simulation configuration is one object in the `match` array.

Example:

```JSON
{
  "id" : "2017-TestSim-1of1",
  "scenarioClass" : "city.CitySimulation",
  "steps" : 1000,
  "map" : "london",
  "seedCapital" : 50000,
  "minLon" : -0.1978,
  "maxLon" : -0.0354,
  "minLat" : 51.4647,
  "maxLat" : 51.5223,
  "centerLat" : 51.4885438,
  "centerLon" : -0.1112036,
  "proximity" : 0.0002,
  "cellSize" : 0.001,
  "randomSeed" : 17,
  "randomFail" : 1,
  "postJobLimit": 10,

  "roles" : {},

  "entities" : [
    {"Car" : 2},
    {"Drone" : 2},
    {"Motorcycle" : 1},
    {"Truck" : 1}
  ],

  "generate" : {}
}
```

The `roles` object has already been discussed in the [Roles](#roles) section, while the `generate` block will be subject of the [Random generation](#random-generation) section.

For each simulation, the following parameters may be specified:

* __id__: a name for the simulation; e.g. used in replays together with the starting time
* __scenarioClass__: the class containing the scenario; needs to be written as above for this scenario
* __steps__: the number of steps the simulation will take
* __map__: the map to use; needs to be in the `server/osm` folder as `XYZ.osm.pbf` file
* __seedCapital__: the amount of money owned by each team at the start of the simulation
* __min/maxLon/Lat__: the map bounds; the declared area must be present in the map file
* __centerLon/Lat__: an internal value used for positioning things and routing; basically, any location of the map is considered reachable if there exists a route between this "center" and that location in both directions
* __proximity__: the proximity value (see [Locations](#locations) section)
* __cellSize__: the cellSize value (see [Locations](#locations) section)
* __randomSeed__: the random seed that is used for map generation and action execution
* __randomFail__: the probability for any action to fail (in %)
* __postJobLimit__: the number of simultaneously active jobs for each team (i.e. a team may not post more jobs)

The number of agents per role is defined in the `entities` array. Each object may have only one key (the name of the role). The value for the key is the number of agents for that role.

Agents are assigned their role according to their position in the team config. In the above example, the first two agents would be Car agents, the next two would be Drone agents, etc.

## Random generation

TODO