# The "Botech Protocol"
2022-10-30
Written by Rory Watts, Director of Forecast Health Australia.

# Description
The Botech Protocol is a protocol which describes rules for creating graph-based mathematical models.
This protocol can be created, run, or analysed using the Botech-Standard-Utilities, also created by Forecast Health Australia.
The BFS was originally-intended to be a general standard model for health economic purposes, capable of modelling a large number of scenarios in a way which is transparent, replicable, and acceptable for the purposes of modelling in low-resource and low-data settings.
However, in essence, the Botech protocol allows any model which requires transactions between states, and is can therefore entertain a wide variety of use-cases.

# Minimum viable model
The Botech protocol is a JSON file with four main keys: `nodes`, `edges`, `globals` and `metadata`.
`nodes` contain accounts which store values of commodities.
`nodes` also contain properties, which provide data which can be used for additional calculations.
`edges` connect nodes, with a source node, and a target node. 
`edges` contain transaction data, about what quantity of commodities are transferred between nodes. 
`globals` contain information relevant to the initialisation and runtime of the model, such as the duration of a run, and the specifics of the accounts.
`metadata` contains an author, a title of the model, and the version of the Botech protocol it was built in accordance with.

Therefore, the minimum viable model is as follows:
```
{
  "metadata": {
    "author": "",
	"title": "",
	"botech protocol version": "1.0"
  },
  "globals": {},
  "nodes": {},
  "edges": {}
}
```

As this is the minimum viable model, anything less than this is expected to return an error.

# Nodes
# Edges
# Globals
