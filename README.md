# The "Botech Protocol"
| Parameter | Value      |
|-----------|------------|
| Started   | 2022-10-30 |
| Authors   | Rory Watts |

# Description
The Botech Protocol is a protocol which describes rules for creating graph-based mathematical models.
This protocol can be created, run, or analysed using the Botech-Standard-Utilities, also created by Forecast Health Australia.
The BFS was originally-intended to be a general standard model for health economic purposes, capable of modelling a large number of scenarios in a way which is transparent, replicable, and acceptable for the purposes of modelling in low-resource and low-data settings.
However, in essence, the Botech protocol allows any model which requires transactions between states, and is can therefore entertain a wide variety of use-cases.

# Minimum viable model
The Botech protocol is a JSON file with four main keys: `nodes`, `edges`, `globals` and `metadata`.
`nodes` contain ledgers which store values of commodities in accounts.
`nodes` also contain properties, which provide data which can be used for additional calculations.
`edges` connect nodes, with a source node, and a target node. 
`edges` contain transaction data, about what quantity of commodities are transferred between nodes. 
`globals` contain information relevant to the initialisation and runtime of the model, such as the duration of a run, and the specifics of the accounts.
`metadata` contains, at least, a list of authors, a title of the model, and the version of the Botech protocol it was built in accordance with.

Therefore, the minimum viable model is as follows:
```
{
  "metadata": {
    "authors": [],
	"title": "",
	"botech protocol version": "1.0"
  },
  "globals": {},
  "nodes": [],
  "edges": []
}
```

As this is the minimum viable model, anything less than this is expected to return an error.

# Globals
Globals specify parameters which are expected to be broadly important for the initialisation and runtime.
That is, they are not specific to a particular node, or a particular edge.

## start and stop
`start` specifies an integer. This integer represents the unit of time which is the first unit the botech model will run.

`stop` specifies an integer. This integer represents the unit of time which is the last unit the botech model will run.

It is required that stop > start. 
Importantly, both start and stop are masks over an index, which runs from 0 to `stop` - `start`.
An example is as follows:
```
"start": 2020
"stop": 2039
```
In this example, the model will represent it's start value as 2020, and its stop value as 2039.
The botech model will not run in the year 2040, as it finished after 2039.
An index will run from 0 to (2039 - 2020) 19, which is twenty units.

## ledgers
`ledgers` are dictionaries contained in the `globals` and `nodes` dictionaries.
However, here we are speaking about their role in the `globals` dictionary.
Within the `globals` dictionary, `ledgers` provide information about how they are implemented during initialisation of the model.

Labels, if included, require a `label`. The label is a common-sense name for the thing that is being exchanged. For example, persons may be referred to as PAX, or Australian Dollars may be referred to as AUD etc. 

A `label` is a string
```
"label": "AUD"
```

Labels can take an additional property called `shape`. Typically, the unit of something being held is 1-dimensional. For example, if we are holding Australian Dollars, we are holding, for example, $200. Here, $200 is a scalar, or a 1-dimensional value. However, there are times when we want to specify an account with more than one dimension, or arbitrary numbers of dimensions. One particular example is with the propagation of people across nodes. Perhaps one node only requires 14-year old females (e.g. a vaccination programme). In this instance, we require two dimensions to the commodity*: one dimension to convey information about sex (male, female), and one dimension to convey information about age (0 -> 100, or similar).

Note, commodity is clearly not the right word to refer to people. We use this nomenclature as it relates to accounting practices, wherein ledgers and accounts are used. 
A `shape` is a list of integers, with each integer specifying the lenght of the dimension
```
"shape": [2, 101]
```
Note: If a `shape` is not given, it is implied the shape is [1].

# Nodes
# Edges
