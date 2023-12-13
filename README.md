# Botech
Authors: Rory Watts, Director, Forecast Health Australia

## Overview
Botech (back of the envelope technology), is a protocol for the creation and simulation of state transition models.
State transition models are composed of nodes and edges. 
Nodes store values as a `balance`, edges transmit values using a `transition rate`.
Edges have a direction. 
Therefore, the representation of these models is a `DiGraph` or `Directional Graph`.
Fundamentally, **all components of the model are nodes or edges**: people, resources, costs, constants, these should all be represented as nodes, or edges.
Furthermore, a node's balance is an **array** of 202 values, from males aged 0, to females aged 100.

Botech is a superset of the functionality of traditional state-transition models.
That is, it contains all the expected behaviour of state-transition models, and adds features to them.
Botech itself is not a piece of software, but has implementations.

Botech is loosely designed around the unix philosophy of doing one thing well.
The implementation takes a `model configuration file` and produces an `observation file` based on simulating that model.
Therefore, an implementation requires:
- Code to interpret the model file, and create the runtime environment
- Code to simulate the model and its behaviours
- Code to log and export the observations

Botech *should be shareable*, and so both the model configuration file and observation file are intended to be plain text (e.g. a JSON configuration and a CSV of observations).

## Background
The Botech protocol arose from the need to do the following things:
- Produce complex models...
- that people understand...
- which are reproducible

Complex models mean complex behaviours can be captured (e.g. feedback loops, dynamic changes to the model).
Understandable models mean that different groups can assess and interpret models without programming expertise.
Reproducibility means that assumptions are transparent, and that groups can adapt and rebuild models.
We also wanted the protocol and implementation to be open-source. Intelligence is distributed, and good modelling requires a mix of good software, good topic-experts, and good designers.
Making botech open-source means that we can leverage this distributed intelligence.


## Clarifications
- The protocol is under development. It is primarily under development through trial and error via the `botech-python` implementation.
- We use `edges` and `links` interchangeably in this document
- When referring to programming conventions, we are typically using Python naming conventions e.g. dictionaries for `{}` and lists for `[]`


## Specification: The Model Configuration File
### Data Format
The intention is for the model configuration to be plain text.
We believe `json`, `yaml`, `xml` and similar formats to be suitable.

### Architecture
A typical configuration file will consist of the following:
```
{
  "nodes": [],
  "links": [],
  "runtime": {},
  "metadata": {},
  "subloops": []
}
```

`nodes` is a list of dictionaries representing nodes.  
`links` is a list of dictionaries representing links.  
`runtime` is optional, and is a dictionary of runtime variables.  
`metadata` is optional, and is a dictionary of key value pairs to be considered metadata.  
`subloops` is optional, and is a list of dictionaries representing subloop configurations.  

### Nodes and Links
Nodes and links are a collection of properties which forms the model structure. 
For our purposes, we will deal with nodes and links collectively when we can, and call them `components`.

#### ID [required]
Components have an ID. An ID should be unique. We choose to use `ULID`. 

#### Label [nodes only, required]
Nodes have a plain-text, human-readable label.

#### Order [optional]
An integer.

Order for nodes specifies the runtime order.
E.g. if `foo`, `bar` and `baz` have orders 0, 2, and 1, then for any `subloop` where they are all invoked, they will run in order `foo`, `baz`, `bar`.
More than one node can have the same order, and this is useful if there is no meaningful difference when two nodes run.

Order for links specifies their `batch` order.
Batch order is best illustrated through an example.
Consider a node with balance 10 called `foo`, it has two links, to `bar` and to `baz` (`foo -> bar`, `foo -> baz`).
Both `foo -> bar` and `foo -> baz` have transition rates of 0.5, meaning they will take 50% of `foo`'s balance.

In the first scenario, `foo -> bar` has order 0, and `foo -> baz` has order 1. This means:
- `foo` will transmit 50% of its balance to `bar` (5)
- `foo` subtracts 5 from its balance
- `foo` will then transmit 50% of its remaining balance to `baz` (2.5)
- `foo` subtracts 2.5 from its balance
- the balance in `foo` is 2.5, the balance in `bar` is 5, and the balance in `baz` is 2.5

In the second scenario, `foo -> bar` and `foo -> baz` have order 0. This means:
- `foo` keeps track of its balance before it transmits anything
- `foo` will transmit 50% of that balance to `bar` (5)
- `foo` will transmit to% of that balance to `baz` (5)
- `foo` subtracts that balance from itself
- the balance in `foo` is 0, the balance in `bar` is 5, and the balance in `baz` is 5

Therefore, batches for links are more complicated, but important to understand. One is more suitable if a user is thinking about rates. One is more suitable if a user is trying to divide a node in the traditional sense.


#### Source and Target [links only, required]
Links require two properties: `source` and `target`.
These are the ID of the source node, and the ID of the target node respectively.

#### Description [optional]
Components may have a plain text description, which informs users about the purpose of the element.

#### Subtype [optional]
If specified, may change the behaviour of the node or edge. 
For example, the `surrogate` subtype of nodes provides different behaviours (discussed below).

#### Generate Array [optional]
Nodes can generate arrays to store in their `balance`. 
Links can generate arrays to create their `transition rate`.
This property is a dictionary with the following features:
- `method`: a string which matches the name of the method to generate the array
- `parameters`: a dictionary containing key value pairs of the parameters required to generate the array
- `data_fetcher_label`: [optional] a string representing the label of the attached `datafetcher` [see below]
- `cache`: [optional] a boolean. True if the original value should be cached (useful for intensive requests e.g. from `datafetcher`s)
- `fetch`: [optional] instructions on when in the runtime to fetch the array (coming soon)

#### Flush [nodes only, optional]
A boolean representing whether, at the the end of each timepoint (e.g. one year), the balance of the nodes should be reset to 0s ([[0, 0, 0...], [...0]]).
This can be useful if the node is not a state, but a collection or constant of some sort.

#### Age [nodes only, optional]
A boolean representing whether, when triggered, the balance of the nodes should be "aged".
Here, this means the values of each element in an array are shifted by 1 index.
Therefore, the first array will contain 0, and the final array will be removed.
This simulated ageing: males aged 1 will now be aged 2, females aged 50 will now be aged 51 etc.
This also implies anyone aged 100 will be removed from the array.

### Runtime
#### start and end [optional]
start: An integer indicating the index at which a model should start e.g. 2023
end: An integer indicating the index at which a model should end (inclusive of that year) e.g. 2030
This is optional, and defaults to 0 and 1 respectively.

Using the above example:
A model with this specification should have eight timepoints: 2023, 2024, 2025, 2026, 2027, 2028, 2029, 2030

Note - the specification of a timepoint is arbitrary.

### Metadata
A series of key value pairs containing strings that can be used to hold metadata about the model.
For example:
```
{
  "metadata": {
    "author": "Botech",
    "title": "Metadata Example",
    "description": "An example metadata dictionary",
    "year": "2023"
  }
}
```


### Subloops
The order of operations in a model is important.
Subloops allow users to specify when things happen within runtime.
For example, each year you may wish for ageing to occur first, then births, then deaths etc.
These are behaviours which can be ordered using subloops.

Subloops are a list of dictionaries, with each dictionary providing information about the behaviour to occur. 
Importantly, the list will run from top to bottom, so the order matters.

The properties of subloops are as follows
```
subloops = [
  {
    "method": "age",
    "narration": "Age the population",
    "all_nodes": False,
    "parameters": {
      "nodes_to_include": [],
      "nodes_to_exclude": []
    }
  }
]
```
`method` is a string, corresponding to a property or method that a node can execute e.g. "age", "flush", "generate_balance".
`narration` is a string which will be written to the `observation file` for each `Event` that occurs during the subloop.
This is useful to distinguish between subloops when debugging or reading logs.
`all_nodes` is a boolean which, if True, will attempt to run the `method` on every node, regardless of whether or not it has an `order`.
`nodes_to_include` is a list of IDs of nodes to apply the `method` to. 
`nodes_to_exclude` is a list of IDs of nodes to avoid applying the `method` to.

## Specification: The Botech Runtime Environment
The goal of the Runtime Environment (hereafter `environment`) is to instantiate the set of methods and data sources required for the model to be appropriately simulated. Importantly, once the environment is correctly set up, it can run the `simulate` method and produce the `observation file`.

The environment takes the following data structures.

### Model configuration
The model configuration is discussed above (the plain text configuration file for models).

The environment accepts one model configuration and creates a Botech Graph, which is discussed below.

### DataFetchers and DataSources

Discussed elsewhere. Briefly, datafetchers and datasources are ways of generating data, either synthetically, or through datasources. For example, we may want to generate a random value as a transition rate, and we could create a datafetcher to provide this method. Another example, we may wish to obtain the age-specific fertility rates for Uganda for 2020, and we would use a datafetcher as an API on this specific datasource.

### Hooks
Discussed elsewhere. 

### Observations
A user can attach the `observation file` to the environment (discussed below).

## Specification: The Botech Graph and Graph Components

The `environment`, when passed a model configuration, will create a `graph` and the components of the graph: `nodes` and `edges`. This is the process of translating the nodes and links of the `configuration` into objects (or methods, depending on the implementation) that can:
- maintain state
- adjust their balance (nodes)
- transmit values (edges)
- generate arrays (nodes and edges)


## Specification: The Simulation Method and Runtime Events

The `simulate` method is triggered from the `environment` and propagated to the `graph`, where the the model is simulated. Here is an example from the `botech-python` implementation.
```
def _simulation_loop(self):
    while not self._environment.counter_is_max():

        for subloop in self._subloops:

            nodes_to_include = self.determine_nodes_to_include(subloop)
            method_name = subloop.get("method")
            narration = subloop.get("narration", "")
            parameters = subloop.get("parameters", {})

            self._environment.set_simulation_narration(narration)
            for node in nodes_to_include:
                method = getattr(node, method_name)
                method(**parameters)

        self._environment.increment_counter()
        self._environment.reset_local_event_counter()
```
Here are the important points about the `simulation` method:
- It runs from the start time to the end time (including the end time)
- It iterates through the subloops which were configured in the `model configuration`
- It determines which nodes should be included
- It fetches the appropriate method, based on the method specified in the subloop label
- It performs that method on each node

## Specification: Events and the Observation File
Botech produces observations about the simulation. 
This is different to producing `results` as results are typically a curation of observations.
Observations are taken anytime anything *happens*, which means we can understand exactly what a model is doing.

Something *happens* in the model when:
- The balance of a `node` changes
- An array is generated, calculated, or retrieved from a cache
- values are moved in a link

When these things happen, we call it an `event`, and we record it.
The record has the following features (although we plan to make this configurable in the future):
- id: a unique identifier
- timestamp: a mock-timestamp to provide time-series functionality
- time index: an integer, 0-indexed (the first time is time 0)
- time reference: an integer
- event number: an integer starting at 0, incrementing for each event
- local number: an integer starting at 0 for each time period
- narration: the `subloop` narration
- event type: a string representing the type of event (e.g. "balance change")
- component type: a string indicating a node or edge
- component id: the unique identifier of the component
- component label: the string label of the component
- component subtype: the subtype of the component
- value: the value calculated with the event type

## Additional Features
In the overview, we mentioned that botech is a superset of traditional state-transition models. By this, we mean *state-transition models pass values between states, and our models can do more than this*. The following features are considered additional to that traditional functionality.

### On-Graph Calculations 
Rather than pass values between states, edges can use the value of a source node to inform them about how to interact with a target node.

Consider the following example: two nodes, `foo` and `bar`, connected by edge `foo -> bar`. `foo` has value 0.5, `bar` has value 10, `foo -> bar` has transition rate 1.

In a traditional setting, `foo` should transfer 100% of its value to `bar`, resulting in the final balances: `foo` (0), `bar` (10.5).

However, during configuration, we can tell `foo -> bar` to adopt the following behaviour:
- Don't take any values from `foo`
- Use `foo`'s value as information
- Use that information to multiply `bar`'s value

This means that the value of 0.5 from `foo` is multiplied against `bar`'s value of 10, resulting in the final balances: `foo` (0.5), `bar` (5).

This means users can perform *on-graph calculations*, which can be useful to make calculations explicit, rather than occurring before the model has been created. It also means users can model dynamic processes that modify eachother.

### Surrogates (Nodes which are edges)
Surrogates defined very simply are *nodes which change the transition rates around them*. This can be unintuitive, and let us provide some context first. 

Imagine the following graph with nodes `foo`, `bar` and edge `foo -> bar`. `foo` has value 10, `bar` has value `5`. `foo -> bar` has the crypic value of 0.1234598798, some complicated number, and it is not immediately clear where this value came from. It turns out, that 0.1234598798 is the resulting value from a series of calculations made in an excel spreadsheet which, the whereabouts of which have gone missing. Now we are left with a value, and not much explanation of what it means! 

This example highlights what `surrogate` nodes intend to resolve: **Transition rates can represent a simplification of a complex process.**

One solution to this is to utilise the on-graph calculations we explained previously. However, we are presented with the issue that `values` are a property of nodes, and these values are transmitted through edges by rates. Surrogates allow us to pretend a node is an edge, by affecting the edge coming into it, and the edge going out of it.

Here's an example.

Consider the following graph: `foo -> bar -> baz`, and assume that `foo` has a value of 10. `bar` is a surrogate, so it means we want `bar` to pretend to be a transition rate. Here's how it works: when the value of `bar` changes:
- the value of `foo -> bar` is set to the value of `bar`
- the value of `bar -> baz` is set to 1. 
So, if we wanted a graph where 50% of values move between `foo` to `baz`, we would set the value of `bar` to 0.5 and the following would happen:
- the value of `foo -> bar` would be set to 0.5
- the value of `bar -> baz` would be set to 1
- 50% of people would move from `foo` and immediately pushed to `bar -> baz`, where 100% of them would move to `baz`.

This is clearly a complicated process, but it allows users to build progressively complex models, by slowly expanding processes by unpacking transition rates into their component calculations. This also (perhaps surprisingly) provides transparency, because users can see what a transition rate is composed of. For example, the transition between `foo` and `bar` might be to do with the coverage rate, and the population in need, which would have been obscured if the transition rate was one value.


### Triggers
Triggers work as follows: *when something happens, if a condition is met, another thing happens*. 

Consider the following graph: `foo -> bar -> baz` where `foo` has value 10, and transmits a flat value of 1 to `bar` each year. `bar -> baz` has a `trigger` which says "every 3 values of `bar`, increment `baz` by 1, but dont remove values from `bar`". Therefore, the following will happen:
- foo (10), bar (0), baz (0)
- foo (9), bar (1), baz (0)
- foo (8), bar (2), baz (0)
- foo (7), bar (3), baz (1)
- foo (6), bar (4), baz (1)
- foo (5), bar (5), baz (1)
- foo (4), bar (6), baz (2)
- foo (3), bar (7), baz (2)
- ...

This can be used to create complex behaviours, or simplify behaviours. For example, if each new value going in to `breast cancer treatment` should incur a set of resources and costs, we could attach these resources and costs as nodes that are `triggered` when there this occurs.


## Getting Data (Datafetchers and DataSources)
By default, data is not "input" into nodes as balances, or into edges as transition rates. 
Rather, data is returned from a `datafetcher`.
A datafetcher is an API, which accepts a `method` and some `parameters` relating to that method and returns a value in the form of an 2x101 array.
Datafetchers are also able to retrieve information from the `environment` such as the current time period, and metadata.
Importantly, datafetchers are meant to be expanded and customised.
Users are expected to register methods with a datafetcher, and then register the `datafetcher` to the `environment`.

Consider the following node in a hypothetical `model configuration file`:
```
{
   "id": "foo",
   "label": "foo",
   "generate_array": {
    "data_fetcher_label": "single_value_generator",
    "method": "generate_a_single_value",
    "parameters": {
      "value": 10
    }
   }
}
```
Here, the configuration requires that the `environment` has a `datafetcher` and that the `datafetcher` has been registered as "single_value_generator".
Within the `datafetcher` is should have a method, as follows (or similar):
```
def generate_a_single_value(value: int) -> np.ndarray:
    return np.ones((2, 101)) * value
```
Here, the method is defined as returning an array with shape 2x101, with the value that was provided.

This is a simple method, but a necessary one. However, it demonstrates that users can create arbitrary and complicated methods to generate synthetic data, such as:
- picking random values
- generating distributions
- changing values over time
- etc...

So far, we've talked about generating synthetic data, but `datafetchers` are also used for retrieving stored data too.
Here is another hypothetical node.
```
{
   "id": "foo",
   "label": "foo",
   "generate_array": {
    "data_fetcher_label": "undp_wpp",
    "method": "get_the_population_for_a_country_in_a_particular_year",
    "parameters": {
      "country": "UGA"
    }
   }
}
```
This method would obviously be a little more complicated, but would look something like this:
```
def get_the_population_for_a_country_in_a_particular_year(self, country: str) -> np.ndarray:
    current_year = self.get_the_current_year()
    values = self.retrieve_undp_population(self.conn, current_year, country)
    return values
```
This code is intentionally sloppy to make two things obvious:
- you can retrieve information from the `environment`
- `datafetchers` often have some kind of connection to a `datasource`, as indicated by `self.conn()`

Although not strictly necessary, `datasources` are useful because they allow users to separate the API to obtain data, and the connection to that data.
This can be useful if two groups access the data differently e.g. on a local network, or via a web-API.

Finally, an `environment` can handle an arbitrary number of `datafetchers`.
This means that users can maintain datasets whichever way they like. 

## Using the observations as a datasource
One of the requirements of botech, was that it was replicable, so that users could share models.
Using `datafetchers` and `datasources` this is indeed possible, but potentially very cumbersome.
If one group has 10 datasources, they would need to allow users to access these, either online (e.g. via an API) or by sending local copies to each user.
To address this, botech allows users to attach an `observation file` to the `environment`.
If this is done, instead of accessing `datafetchers` to obtain data, the environment will access data from the `observation file`.
This means that users don't have to share their databases, just their results, and other users will be able to completely replicate the model.
This also means that users can change the assumptions in that file, re-run the model, and observe the changes.
This has one important restriction - the runtime order has to be identical.


## Providing added behaviours through Hooks
Hooks allow the user to add functionality to a node which isn't currently available through a separate API, or would be confusing to add directly to the graph.

An example of this is "disability", which means (the sum of the values in a node, multiplied against a disability weight). This can be achieved through a node and link, or it may be added as a hook, in which users can specify the weight, and how it should be used.

## Implementations
### Python
`botech-python` is currently being developed by Forecast Health Australia.

## Contributing
If you would like to contribute to the development of botech, please email info@forecasthealth.org

##  License
Botech and its implementations are licensed under the Apache 2.0 license [A copy can be found here](https://github.com/ForecastHealth/botech-protocol/blob/master/LICENSE)

## Similar Projects
There are other excellent efforts for creating and simulating health models:
- [The OneHealth Tool (OHT) and Integrated Health Tool (IHT)](https://www.avenirhealth.org/)
- [The C4P Implementation by Levin Morgan LLC](https://www.levinmorgan.com/publications)
- [The TLO Model from the Thanzi La Onse collaboration](https://www.tlomodel.org/)
- [Vivarium by IHME](https://github.com/ihmeuw/vivarium)
- [Squiggle](https://github.com/quantified-uncertainty/squiggle)
- [TreeAge Software](https://www.treeage.com/)
- [The Policy1 Modelling Platform](https://www.policy1.org/)
- [The Optima Model](https://optimamodel.com/)

## Acknowledgments
We acknowledge the great contributions and support made by the World Health Organization. In particular we acknowledge the contributions made by Dr Andre Ilbawi, Dr Filip Meheus, Dr Melanie Bertram, Dr Slim Slama and Dr Tessa Edejer.
We acknowledge contributions made by Annalisa Belloni of Cancer Research UK.
We acknowledge contributions from Mr Joseph Dunne, and Mr Timothy Knowles, who have both contributed to the `botech-python` implementation, as well as other related projects.
We acknowledge Mr Angus Watkins, for the design of the Botech logo.
