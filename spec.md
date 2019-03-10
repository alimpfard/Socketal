

## File Format
UTF-8 Files with the extension `.sock`.

## Comments
comments start with two dashes `--` and end at EOL, 
or start with a `{-` and end with a `-}`.

These do _not_ nest.

## File structure

[meta specifications]
[socket declarations]
[node specifications]
[architecture/topology]
[socket descriptions]
[node interactions]

all specs span a line and start with a single octothorp (`#`) followed by the property/directive name and its arguments

### Meta specifications

+ subnet [subnet range]:
this entry, if specified, will translate node identifiers to IP addresses in the valid range

example: 
`#subnet 192.168.0.0/24` will cause the node ident `3` to have an IP address of `192.168.0.3`, and ident `259` to have an IP address of `192.168.1.4`

+ label [label file]
This entry, if specified, will read pairs of [ident, IP] from the given file.

### socket declaration

TODO

### node specification

TODO

### Architecture / Topology
This entry is mandatory, and a single file may only contain one.
+ model [model spec]
Specifies the topology of the described network, see below for details

#### Model Spec
An expression of the given forms that describes a topology
```
modelspec := '{' pair+ '}'
modelspec := modelspec '-' modelspec
modelspec := modelspec '+' modelspec
modelspec := '(' modelspec ')'
modelspec := metaFnName '{' ident* '}'
```

where `metaFnName` is defined by any extensions to the system.
the default metaFn names are:
+ `mesh` : generates a fully-connected set of pairs
`mesh{1,2,3}` -> `{<1,2>, <2,1>, <1,3>, <3,1>, <2,3>, <3,2>}`

+ `ring` : generates a set of circular link pairs, in the order specified
`ring{1,2,3}` -> `{<1,2>, <2,1>, <2,3>, <3,2>, <3,1>, <1,3>}`

+ `star` : generates a hub-client set of pairs
`star{1,2,3,4}` -> `{<1,2>, <2,1>, <1,3>, <3,1>, <1,4>, <4,1>}`

and where `-` is the set difference opeator, so `mesh{1,2,3} - mesh{1,2}` will generate the set difference between the expressions

and '+' is the set union operator.

### Socket Descriptions
Sockets are identified by the endpoints they are connected to, so `<1,2>` will reference the socket connecting node 1 to node 2 (one way).

Sockets must contain two parameters:
+ Function : what this socket does on the data that passes through it
+ Endpoint : which endpoint does the calculations [receiver or sender]

(This section is under heavy review)
Description of a socket:
`socket [socket name] [argspec] {on [endpoint]} = [expression]`

for example:
`socket <1,3> (x: int, y: int) on receiver = x + y`

if the endpoint is not specified, it defaults to `sender`.

### Node Interactions
Nodes can decide to pass away the data/operations requested from them to other nodes
(This section is also under heavy review)

`node 3 dispatches receive from node 1 to node 2`
This will simply connect the socket <1,3> to <3,2> (both operations will be performed)

`node 3 dispatches operation from node 1 to node 2`
This will ignore the operation defined on socket <1,3> and forward the data to socket <3,2>


## Example program (WIP)

```
#subnet 192.168.1.0/12
#model ring{1,2,3} + {<3,4>}

socket <1,2> x: String = x + "!"
socket <3,2> x: String = "[" + x + "]"
socket <2,3> x: String = "Hello, " + x
socket <3,4> x: Unit   = Print# x

node 1 dispatches receive to node 1
node 2 dispatches receive to node 3
node 3 dispatches reveive from node 2 to node 4

send "World" to node 1
```
