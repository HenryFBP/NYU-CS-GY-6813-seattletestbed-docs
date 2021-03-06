# Seattle Chord Assignment

For this assignment the students first implement a DHT-like message routing system based on Chord, and test their implementations on local Seattle resources. The students then run their code on globally distributed Seattle resources. Chord works well over LAN, but has performance and correctness problems in a global scale deployment with non-transitive connectivity. After explaining the reasons behind Chord's poor performance, the instructor can have the class discuss solutions to these problems. Students can then implement these solutions to achieve better performance and reliability.

This assignment reinforces several important ideas. First, the students will reuse their implementation, which emphasizes good software engineering practices. Second, the assignment demonstrates that test and deployment environments may differ significantly. Third, students will hone their debugging skills as they attempt to understand why their "correct code" does not work in a global deployment. Fourth, by creating their implementation from scratch, and motivated by a real problem, students can arrive at unique solutions. Lastly, the instructor can easily evaluate student implementations using a small set of metrics.

This assignment is based in large part on a Chord assignment used by Tom Doeppner in his Distributed Computing Systems [course](http://www.cs.brown.edu/courses/csci1380/asgn.html).

**NOTE WINDOWS USERS:** You must have Python2.5 or Python2.6, the programming language, installed on your computer in order to complete this assignment. Follow the instructions [InstallPythonOnWindows here] to check if Python2.5 or Python2.6 is currently installed on your system and to get directions on how to install Python2.6 if it is not installed.

----

----


# Introduction



Chord is a distributed hash table (DHT) protocol currently under
development at MIT. It was proposed in 2001 in a paper titled
``Chord: A Scalable Peer-to-peer Lookup Service for Internet
Applications" (see References section below). From an application's perspective, Chord
simply provides a service that can store key-value pairs and find
the value associated with a key reasonably quickly. Behind this
simple interface, Chord distributes objects over a dynamic network
of nodes, and implements a protocol for finding these objects once
they have been placed in the network. Every node in this network is                                                                                                                                                                                                                                                        
a server capable of looking up keys for client applications, but
also participates as key store. Hence, Chord is a decentralized
system in which no particular node is necessarily a performance
bottleneck or a single point of failure (if the system is
implemented to be fault-tolerant).



## Keys

Every key inserted into the DHT must be hashed to fit into the
keyspace supported by the particular implementation of Chord. The
hashed value of the key will take the form of an m bit unsigned
integer. Thus, the keyspace (the range of possible hashes) for the
DHT resides between 0 and ```2^m-1``` inclusive. Standard practice
for most DHT implementations is to use a 128 or 160 bit hash, where
the hash is produced by a message digest algorithm such as MD5 or
SHA-1. Using these hashing algorithms ensures with high probability
that the hashes generated from keys are distributed evenly
throughout the keyspace. Note that this does not restrict the number
of distinct keys that may be stored by the DHT, as the hash only
provides a guide for locating the key in the network, rather than
providing the identifier for the key. It is possible, though
unlikely, for the hash values of distinct keys to collide.



## The Ring

Just as each key that is inserted into the DHT has hash value, each
node in the system also has a hash value in the keyspace of the DHT.
To get this hash value, we could simply give each node a distinct
name (or use the combination of IP and port) and then take the hash
of the name, using the same hashing algorithm we use to hash keys
inserted into the DHT. Once each node has a hash value, we are able
to give the nodes an ordering based on those hashes. Chord orders
the node in a circular fashion, in which each node's successor is
the node with the next highest hash. The node with the largest hash,
however, has the node with the smallest hash as its successor. It is
easy to imagine the nodes placed in a ring, where each node's
successor is the node after it when following a clockwise rotation.

To locate the node at which a particular key-value pair is stored,
one need only find the successor to the hash value of the key.



## The Overlay

As the paper in which Chord was introduced states, it would be
possible to look up any particular key by sending an iterative 
request around the ring. Each node would determine whether its
successor is the owner of the key, and if so perform the request at
its successor. Otherwise, the node asks its successor to find the
successor of the key and the same process is repeated.
Unfortunately, this method of finding successors would be incredibly
slow on a large network of nodes. For this reason, Chord employs a
clever overlay network that, when the topology of the network is
stable, routes requests to the successor of a particular key in
log(**n**) time, where **n** is the number of nodes in the network.

This optimized search for successors is made possible by maintaining
a ``finger" table at each node. The number of entries in the finger
table is equal to **m**, where **m** is the number of bits representing
a hash in the keyspace of the DHT. Entry **i** in the table, with
0 <= **i** < **m**, is the node which the owner of the table believes is
the successor for the hash ```h+2^i``` (**h** is the current node's hash).
When node A services a request to find the successor of the key
k, it first determines whether its own successor is the owner of
k (the successor is simply entry 0 in the finger table). If it is,
then A returns its successor in response to the request.
Otherwise, node A finds node B in its finger table such that B
has the largest hash smaller than the hash k, and forwards the
request to B.



## Dynamics

Chord would be far less useful if it were not designed to support
the dynamic addition and removal of nodes from the network,
requiring a static allocation of nodes instead. A production ready
implementation of Chord would support the ability to add and remove
nodes from the network at arbitrary times, as well as cope with the
failure of some nodes, all without interrupting the ongoing client
requests being served by the DHT. This functionality complicates the
implementation considerably, though.

To allow membership in the ring to change, protocols for creating a
ring, adding a node to the ring, and leaving the ring must be
defined. Creating the ring is easy. The first node fills its finger
table with references to itself, and has no predecessor. Then, when
node A joins the network, it asks an existing node in the ring to
find the successor of the hash of A. The node returned from that
request becomes the successor of A. The predecessor of A is
undefined until some other node notifies A that it believes that
A is its successor. In order to determine the successor and
predecessor relationships between nodes as they are added to the
network (and voluntarily removed) and refresh finger table, each
Chord node performs ''stabilize'' and ```fix_fingers```
periodically (```find_successor``` is summarized in the section on Overlay):

```
n - this node
h - hash of n
m - the number of bits in a hash

stabilize()
    x = successor.predecessor
    if x in (n, successor]
        successor = x
    successor.notify(n)

fix_fingers()
    # next stores the index of the finger table entries.
    next = next + 1
    if next > m
        next = 1
    next_hash = h + 2<sup>i mod 2</sup>m
    finger[i] = find_successor(next_hash)

notify(p)
    if predecessor is None or p in [predecessor, n)
        predecessor = p
        transfer appropriate keys to predecessor
```

The following method is called when a node leaves the network:

```
leave()
    transfer all keys to successor
    successor.predecessor = None
    predecessor.successor = predecessor
```

Fault tolerance is achieved by maintaining successor lists, rather
than a single successor so that the failure of a few nodes is not
enough to send the system into disrepair. Keys must also be
replicated across a number of nodes so that they are available in
the event that some of the nodes storing them fail. The
implementation details of replication are beyond the scope of this
assignment.

In order to understand how the system works as a whole, it is
important to see that although optimizations based on the finger
table may not always be available, it is always possible to find the
correct successor node for a given hash. This is an invariant of the
system, even when nodes are joining and leaving the network with
great frequency.



# Assignment
----

You will be implementing a simple version of Chord. Your
implementation needs not be fault tolerant, and we are stressing
correctness over performance in this assignment. To remove
unnecessary complication, you should treat key-value pairs in the
DHT as **immutable**. This means that once a key-value pair is
inserted into the DHT, it cannot be deleted and the value associated
with the key may not change (you should enforce this requirement).
Your implementation of Chord should support dynamic insertion
and removal of nodes, and continue to serve ```get``` and ```put```
requests simultaneously and correctly (if a value
exists for a key, it must always be accessible). Keys should never
reside at more than two nodes at any given time, and only one node
when the ring is in a stable state.

As specified above, you should implement the following methods 
(the SHA hash algorithm is already provided in the [wiki:SeattleLib Standard Library].)

```
create()
join(node)
leave()

get(key)
put(key, value)

find_successor(key)
stabilize()
fix_fingers()
notify(p)
```


You will implement the Chord communication over Repy's UDP functions
or reliable ones from your previous assignments. 



# Testing
----

In order to test the correctness of your implementation, you need to
dump the overlay state at some time point and run a verification
algorithm over the dumped state. ```dump.repy``` will collect the state from each node,  
put them into a list and print the list on the console 
(or write the list out to an output file). 
Then, you can run some script to parse the output from
```dump.repy``` and verify the correctness of your 
implementation. You may want to use the pickle module in the 
standard library for repy so that you can easily write and read
objects from disk.

Once your code works in LAN environments, test it in WAN environments.   
You may notice errors due to non-transitive connectivity.   In order to
get around this problem, your Chord implementation should route messages
through other nodes in your leafset when you have continued communications
failures.   You should design and test a heuristic for determining when to
route messages through an intermediary and describe this in your design
document.



# What to turn in?
----


## Design

Turn in a documentation of your design specification. This doc should 
describe the architecture overview (how you are going to modularize
your system), and the communication protocol including message 
formats and protocol logic (using sequence chart for example).



## Implementation

Turn in a tar file called ```chord_NAME1_NAME2.tar``` that contains a
directory called ```chord_ NAME1_NAME2```. This directory contains:

 * **chordlib.repy**: the library implementation of all 
the methods in the Assignment section.

 * **chord.repy**: ```python repy.py restrictions.test chord [seedip seedport] myip myport timeout``` 
will start a chord ring at myip:myport if seedip:seedport is empty, join to the ring 
through seedip:seedport if it's not empty. The chord node will leave the ring
when timeout.

 * **get.repy**: ```python repy.py restrictions.test get ip port key``` will
print the value by executing ```get``` method on node with
ip:port.

 * **put.repy**: ```python repy.py restrictions.test put ip port key value``` will
store a key/value by executing ```put``` method on node with
ip:port.

 * **dump.repy**: ```python repy.py restrictions.test dump``` will print
the state from all nodes specified in ```nodes.conf```. Each line
in ```nodes.conf``` represents a node address in the format of
ip:port.

 * **Revised doc (5-10 pages)**: completed and revised version of your design
documentation including details of implementation.

 * **README**: any bugs you have in your code,
any extra features you added, and anything else you think the TAs
should know about your project.



# References
----
 * Ion Stoica, Robert Morris, David R. Karger, M. Frans Kaashoek, and Hari Balakrishnan. "Chord: A scalable peer-to-peer lookup service for internet applications." In Proceedings of SIGCOMM, pages 149???160, 2001. 
