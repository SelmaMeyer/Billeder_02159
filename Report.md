# Avoid several threads cracking identical hashes simultaneously 

## Definition and motivation
The idea of the experiment is to avoid several treads cracking the same hash at the same time. As of now the latest version has four treads working on cracking hashes simultaneously and a queue based on FIFO principle. Therefore identical hashes shouldn't be too close to each other in the queue, in order to avoid threads working on the same hash. If a hash is cracked it will be put in a hash table, and it will be quick to look up identical hashes which come later on. 


*Example 1*
![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/queueOK.png)

*Example 2*
![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/QueueSKOD.png)

Furthermore we know that the repetition percentage is set to 20%.  


## Experiment Set-up

## Results

## Discussion

