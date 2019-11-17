# Avoid several threads cracking identical hashes simultaneously 
*Selma Haslund Meyer (s163740)*

## Definition and motivation
The idea of the experiment is to avoid several treads cracking the same hash at the same time. As of now the latest version has four treads working on cracking hashes simultaneously and a queue based on FIFO principle. Therefore identical hashes shouldn't be too close to each other in the queue, in order to avoid threads working on the same hash. If a hash is cracked it will be put in a hash table, and it will be quick to look up identical hashes which come later on. Furthermore we know that the repetition percentage is set to 20%.  

> In the examples the hashes are replaced with intergers easier readability. 

**Example 1:** Identical hashes far from each other in the queue is ok.


![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/queueOK.png)

**Example 2:** Identical hashes close to each other in the queue, there is a risk that treads will work on the same hash simultaneously. 


![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/QueueSKOD.png)

Implementation wise the struct 'Request' will contain a socket array holding up to 5 sockets. If the hash of a new request is found within the top 5 newest hashes in the queue, we will not line up the request in the queue. Instead we will just add the socket (from the new request) to the identical hash's socket array, which already is in line in the queue. Lastly, when the hash from the queue is cracked a respond will be send to all sockets in the socket array. See figure **3** and **4**.

**Example 3:** Before the implementation hashes will just get queued evwn if they are equal. Each containing there "own" socket.

![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/QueueSock.png)
 
 **Example 4:** After the implementation, hashes will not get queued if there already exists an equal hash close by in the queue. The request int the queue, will contain both it's "own" socket, and the socket from the request which didn't get queued.
 
![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/QueueSockArray.png)

## Experiment Set-up

The client configuration is as follows:

- SERVER="192.168.101.10"
- PORT=5003
- SEED=345245
- TOTAL=100
- START=0
- DIFFICULTY=3000000
- REP_PROB_PERCENT=20
- DELAY_US=750000
- PRIO_LAMBDA=1.5

## Results

## Discussion

