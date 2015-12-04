# halladoop
Hadoop knock-off for our Distributed Systems Class Project

Consists of three components
- namenode 
  - maintains image of the virtual file system across the cluster
  - ensures file blocks are replicated across 3 datanodes to provide fault-tolerance
- datanodes
  - stores/replicates/streams file blocks
- clients
  - read/write files to the cluster

### Process Diagrams
#### Writes
1. Client requests list of available datanodes from namenode, these nodes form a pipeline
2. Client chunks file into blocks
3. Each block is streamed to the datanodes in individual packets
4. Packets are propagated through datanodes
5. Last datanode sends confirmation of each packet to client
6. When all packets are confirmed client informs namenode that the block in finalized
7. Namenode writes the block to the virtual filesytem image
8. Repeat until all file blocks are finalized
![write diagram](http://files.stevenulibarri.com/halladoop_docs/write.png)

If a datanode failes the client removes the node from the pipeline and replays lost packets to the remaining nodes. The name node will eventually ensure that the blocks are replicated elsewhere.

#### Reads
1. Client requests block manifest for file from namenode which contains a map of each block to the datanodes that contain it
2. For each block the client requests it from one of the nodes
3. The datanode streams the block back to the client in packets
4. Repeat until all blocks are obtained.
![read diagram](http://files.stevenulibarri.com/halladoop_docs/read.png)

If a datanode fails the client attempts to obtain the block from another datanode in the manifest mapping.

### Endpoints
#### NameNode
##### POST /register
called by a newly started datanode to inform the namenode of its existence
###### Request:
````javascript
{
  "nodeId": "id",
  "nodeIP": "1.12.13.1",
  "totalDiskSpaceMB": 1337,
  "availableDiskSpace": 1337
}
````

##### POST /getWritePipeline
called by client to obtain a list of datanodes to recieve blacks for a write
###### Request:
````javascript
{
  "clientIP": "1.1.1.1",
  "filePath": "some/file/path/lol.txt",
  "fileSizeMB": 1234,
}
````

###### Response:
````javascript
{
  "dataNodes": [
    { "nodeIP": "1.1.1.1" },
    { "nodeIP": "1.1.1.2" },
    { "nodeIP": "1.1.1.3" }
  ]
}
````

##### POST /getReadManifest
called by clients to obtain a manifest of blocks and their locations on datanodes for a file
##### Request:
````javascript
{
  "filePath": "some/path/file.txt",
}
````
##### Response:
````javascript
{
  "fileBlockManifest": {
    //describes all blocks for a file and the datanodes that own them
    //includes unique identifier of each block on each node
  }
}
````

##### POST /heartbeat
called by datanodes to communicate datanode status to the namenode
###### Request:
````javascript
{
  "nodeId": "1.1.1.1",
  "availableDiskSpace": 1234,
  "blockManifest": {
    //describes all blocks currently stored on this datanode
  }
}
````
###### Response:
````javascript
{
  "actions": [
    //list of objects that represent actions that need to be taken by this datanode
    //may include requesting block replication from another data node or deleting blocks
  ]
}
````
