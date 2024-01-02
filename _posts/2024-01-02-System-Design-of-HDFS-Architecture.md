---
title: System Design of HDFS Architecture
date: 2024-01-02 22:33:00 +0800
categories: [SYSTEM_DESIGN]
tags: [hadoop, hdfs, availability, reliability]

---



The Hadoop Distributed File System (HDFS) is a distributed file system. This article summaries how HDFS Architecture achieves high availability and reliability. 



High availability refers to the ability of a system to remain operational and accessible even in the face of failure or disruption. Reliability means the ability of system to remain operational and delieve services correctly. 



Here are some techniques that HDFS uses to achieve high availability and reliability: 

1. Data Replication

    HDFS has a master/slave architecture. It consists of a single NameNode which manages the metadata of the file system and several DataNodes which manages the data storage and dealing with write and read requests. A file stored in DataNodes will be split into one or more blocks. The blocks of a file are replicated and saved in several other servers for fault tolerance. The number of replication and size of a block are configurable. The number of copies of a block is called the replication factor.

    ![HDFS Architecture](https://hadoop.apache.org/docs/r1.2.1/images/hdfsarchitecture.gif)

2. Heartbeats and Re-replication

    The NameNode makes all decisions regarding replication of blocks. Each DataNode sends a Heartbeat message to the NameNode periodically. If the NameNode detects the absence of a Heartbeat message from a DataNode after a certain amout of time, it marks the DataNode as dead. To keep the replication of data to the defined replication factor, NameNode will make the copies of blocks on the dead DataNode and save on other alive DataNodes. When the dead DataNode comes back with all its data, the NameNode will remove the extra copies of blocks.

3. CheckSum

    It is possible that a block of data fetched from a DataNode arrived currupted. Checksum is used to check the integrity of each block. When a client creates a HDFS file, it computes a checksum of each block of the file and stores these checksum into a seprate hidden file in the same HDFS namespace. When a client retrieves file contents it verifies that the data it received from each DataNode matches the checksum stored in the associated checksum file. If not, then the client can opt to retrieve that block from another DataNode that has a replica of that block.

4. the persistence of file system metadata

    The NameNode uses a transaction log called the EditLog to persistently record every change that occurs to file system metadata. The entire file system namespace, including the mapping of blocks to files and file system properties, is stored in a file called the FsImage. 

    A single NameNode machine is a single point of failaure for an HDFS cluster. One of the solutions for this problem is adding a standby NameNode. If the Namenode is dead, the standby NameNode could take over and the operations are not affected. In case of an absence of Standby NameNode, the fsimage file and EditLog updated on the secondary NameNode can be used to setup a new NameNode. 



**Reference**

- https://www.designgurus.io/blog/high-availability-system-design-basics
- https://www.geeksforgeeks.org/reliability-in-system-design/
- https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html
- https://www.quora.com/What-happens-if-NameNode-fails-in-Hadoop#:~:text=If%20the%20NameNode%20in%20Hadoop,HDFS%20cluster%20cannot%20function%20properly.