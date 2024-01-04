---
title: Strategies for Fault Tolerance used in MapReduce
date: 2024-01-04 15:56:00 +0800
categories: [SYSTEM_DESIGN]
tags: [mapreduce, fault_tolerance]

---


# MapReduce

MapReduce is a programming Model. It allows users to define a map function that processes a key/value pair to generate a set of intermediate key/value pair, and a reduce function that merges all intermediate values associated with the same intermediate key.

- map function

    ```
    map (k1, v1) -> list (k2, v2)
    ```

- reduce function

    ```
     reduce (k2, list(v2)) -> list(v3)
    ```

![image-20240104121854669](/assets/img/posts/image-20240104121854669.png)

The MapReduce Architecture contains following parts:

- Worker: workers will be assigned be assigned with map or reduce tasks. 

- Master: Master picks idle workers and assigns each one a map task or a reduced task. Also, the master stores the state of each map and reduce task and the identity of the worker machine. After map functions completed, the master stores the locations and sizes of the intermediate file regions. Then the master will notifiy reduce workers to do tasks. 

# Fault Tolerance

## Worker Failure

The master pings every master periodically. If no response is received from a worker in a certain amount time, the master marks the worker as failed. 

For map and reduced tasks assigned to a fail worker:

- Completed map tasks reset to idle state because their output is store on the local disks of the fail machine and is therefore inaccessible
- Map tasks in progress reset to idle state and becomes eligible for rescheduling.
- Completed reduced tasks do not need to be re-executed since their output is stored in a global file system
- Reduced tasks in progress reset to idle state and becomes eligible for rescheduling.

## Master Failure

- Single master: clients can check this condition and retry the MapReduce operation
- Secondary master: the master can write periodic checkpoints. If the master task dies, a new copy can be started from the last checkpointed state.





