# Components in the client software.

### 2020/04/26

While, going through ClientMain.java, I found that it can be rewritten in more
structured fashion. I found after the date\_time check, and certificate
verification, following components should be started as singleton services
running their indpendent threads. These can pass messages to each other for
implementing all possible use case scenarios.Some singleton objects will simply
represent object as abstraction of configuration and do not require independent
thread runninge. 

One important point is the use of *synchronized blocks to avoid race condition*.
Whatever group of instruction has to be executed as an atomic step, should be
put in synchronized block.

Unfortunately, this has not been taken care of in the current code. It may show
sometime inconsistency due to race condition and will have tremendous bearing on
the performance. This will be typically problem with input task Queues of
singleton object.

Each singleton thread should be considered as thread which takes up tasks from
the input queue and updates its on data structures and generates task which are
dispatched to input queues of the other singleton objects running their own
service thread.

1. RTManager (threaded loop) - It will do periodic exchange with neighbours,
trigger merge operations. Input queue will contain routing table updates
received from neighbours.

2. DHTRouter (threaded loop) - It will use DHTable of appropriate layer and
decide the next node to whom the message is to be forwarded. Input queue will
contain messages to be routed.

3. DHTable (threaded loop) - Maintain the data structure of DHTable for all
layers. The data structure will contain endpoint address of all the nodes in the
table alongwith the transport, and direct/proxy mode to be used. Input queue
will be timer based events to purge the table entry, transmission of pings to
the neigbours for heart beat message.

   The above three should handle multiple DHT Layers. So, number of DHTables should
be equal to number of layers.

4. SpilloverTable (threaded loop) - Spillover table manages the range of
hashIDs to next node for forwarding the query. Thread takes care of timeouts
based removal. 

5. UI (threaded loop) - to display outputs and take inputs from user. It will
also play media outputs, captures media inputs and dispatches it to mediabridge.
It will also take user inputs and send messages to various componenets for
making system work as per user's wish.

6. CommMgr (threaded loop) - All communications with all possible transport will
be received, and queued up for various components to retrieve the messages from
buffer.

7. MulticastManager (threaded loop) - Publishing/removing the forwarder entry in
DHT. Managing the MediaBridge forwarding and mixing table. Communicating with
the parent, grandparent and children nodes in overlaid multicast topology. It
will also set the routing information and media translation configuration in
media bridge.

7. Media Bridge (threaded loop) - receives the feed from multiple sources, fuses
them to correct differntial delay, forwards them to children nodes, forwards a
stream to UI for playback.

8.  ProxyRouter (threaded loop) - Will maintain the table of nodes (behind NAT/
firewall/ http\_proxies) on whose behalf it will receive the messages from
others. These will be sent to CommMgr, from where the messages will be pushed to
nodes on thier setup connections or pulled by the nodes via http.

9. IndexingMgr (threaded loop) - Stores (key, value) pairs for which the current
node is root node. It receives, query, publish messages and responds after
searching database/filesystem. It republishes when republish timer expires and
removes. Removes when removal timer expires. In case of shortage of space,
searches for proxyroot nodes, creates the spillover table. Searches for backup
node. Monitor root nodes for which it is backup. When a monitored root node is
dead, republishes all the backedup entries. It can work with multiple layers
simultaneously. In storage layer, it will store the file fragments.

TBD.

