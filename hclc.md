# FC and HCLC standards

Concord node-level communication adheres to FC (Flag-Content) standard - messages are sent in the following format:
```
{
    FLAG: <flag>,
    CONTENT: <content>
}

```
The main purpose of node communication is to facilitate merging, for which the HCLC (Host-Client Layer Comparison) protocol is used. Its specification is below:
The client will initiate the request by sending a `{FLAG: READY, CONTENT: {chain: <chain>, k: <k>}}` communication, where `<chain>` is the desired chain's tripcode, and `<k>` is the number of subsequent layers between client checks (more on what that means in a moment).

----

The ready block prompts the server to begin sending blocks. Packages of blocks are sent in such a manner that, until no blocks remain, they'll continually prompt more packages. To lay out the protocol, it's first worth laying out the flags it supports:
| Flag    | Direction  | Contents                                                                 | Prompted action |
|---------|------------|--------------------------------------------------------------------------|-----------------|
| READY   | C->H       | Interaction details to establish connection                              | HBLOCKS         | 
| HBLOCKS | H->C       | Block hashes of next server layers, requested from prev. layers          | CBLOCKS or CEND |
| CBLOCKS | C->H       | Blocks not included in prompt layers, hashes not on client end from same | HBLOCKS         |
| CEND    | C->H       | Same as CBLOCKS                                                          | HEND            |
| HEND    | H->C       | Sames as HBLOCKS, but lacking the next layer hashes                      | CEND            |

To make this feedback loop as efficient as possible, the concept of a **layer** is used. This relies on partitioning the chain as follows:
- The first layer has all of the childless blocks in the chain.
- The nth layer has all of the blocks that are both parents of blocks in the (n-1)th layer and not present in previous layer (i.e. layers with ordinality < n)

The utility of this concept is that, if two chains are identical on a particular layer, they are also identical over all higher layers. 

This is because all blocks in higher layers will be predecessors of the blocks in that layer, meaning all higher blocks were included in determining at least one block
in it. Similarly, though this is incidental, this trivially demonstrates that the layer structure above the compared layers will be the same.
This property is convenient because it allows the merge process to be halted as soon as possible - when the client receives a layer that's identical to its own, it can begin end procedures. Here's an example of the process in more detail, using this principle:

 ```The host retrieves its chain's first through kth layers and sends their hashes under HBLOCKS.```

 ```The client retrieves its chain's first through kth and compares. Any blocks it has are sent to the server, and so are requests for any hashes it lacks, both under CBLOCKS.```
 
 ```The host retrieves its chain's second through (k+1)th layers and sends their hashes. It adds any blocks that the client sent to its chain, and sends the blocks it requested to the client. All under HBLOCK```

...

 ```The client retrieves its chain's nth through (n+k-1)th layer and compares. Any blocks it has are sent to the server, and so are requests for any hashes it lacks, both under CBLOCKS. It adds any blocks it received based on previous requests to its chain.```

 ```The host retrieves its chain's (n+1)th through (n+k)th layers and sends their hashes. It adds any blocks that the client sent to its chain, and sends the blocks it requested to the client. All under HBLOCK.```

 ```The client retrieves its chain's (n+1)th through (n+k)th layers and compares. Any blocks it has are sent to the server, and so are requests for any hashes it lacks. It adds any blocks it received based on previous requests to its chain. While analyzing the layers, the client finds that one pair is identical, so it stops there and uses the CEND flag in the transmission.```

 ```The host adds any blocks received to its chain and sends the blocks requested under HEND.```

 ```The client adds any blocks received to its chain. The chains are now merged.```

As you can see, the use of layers allows it to be easily recognized when the chains are fully merged. 
Also for the purpose of efficiency, the sending only of block hashes over the full layer - with block contents being sent when they're observed missing (host's chain) or reported as such (client's chain) - ensures that only full contents which are absent on one side will be transmitted.