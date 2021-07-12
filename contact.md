## Contact

*notes*: 
- first and last describe order in the transmission process, i.e. the first block is the server's terminal block
and the last block is the server's block with index 0.
- When the contact is "restarted with reversed roles," everything from the READY communications is repeated - the same connection is used.
- This document uses '/' as a detonation between content; the first element is the flag, all subsequent elements will be under CONTENT

**end notes**

Concord communication follows the JSON standard, communications between clients follows as such;
```
{
    FLAG: <flag>,
    CONTENT: <content>
}

```
The client will initiate the request by sending a `{FLAG: READY, CONTENT: [<chain>, <k>]}` communication, where <chain> is the desired chain's tripcode, and <k> is the number of subsequent blocks between client checks.
At this point, the server will send `<k>` blocks in an array. The message will adhere to the following format:
`{FLAG: BLOCKS, CONTENT: [<blocks>]}`
where each block in `<blocks>` is composed as such;
```
{
    "#": block index
    "b": [date, prevhash, hash, nonce, nonce contributor trip, server trip, content]
}
```

**Note the spacially compressed structure (single character indices, array for distinct characteristics).**

----

*the following section will be sent in this format;*
```
{
    FLAG: <flag>
    CONTENT: [<sub_flag>, <conditional_content>]
}
```
After every <k> blocks, one of four responses following this will be sent by the client:
| FLAG    | SUBFLAG | CONDITIONAL CONTENT                |
|---------|---------|------------------------------------|
| PRESENT | T       | `<first block sent which qualified>` |
| PRESENT | NT      | `<first block sent which qualified>` |
| ABSENT  | V       | NULL                               |
| ABSENT  | NV      | `<first block sent which qualified>` |

**T** and **NT** denote whether a block is terminal or nonterminal in the client's chain.

**V** and **NV** denote whether a block is valid, considering its contents, its hashes, and the block ahead of it.

ABSENT/V is sent if no other signals qualify. The first non-ABSENT/V block is described in the signal.

During this process, the client will store each received server block.

----

At any signal other than ABSENT/V or on the last block, the server stops transmitting blocks and sends a STOP signal. Five relevant cases exist:

| FLAG                                             | MEANING                                                                                                                                                                                                                                                  | ACTION                                                                                                                                                                                                                                                                                                  |
|--------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PRESENT/T                                        | The server has the same chain preceding that block, and has just transmitted all succeeding blocks. The client is now up-to-date.                                                                                                                        | Transmit END and finish the contact.                                                                                                                                                                                                                                                                    |
| PRESENT/NT on first block (i.e. server terminal) | The client has the server's full chain, but also has more blocks after it.                                                                                                                                                                               | Repeat the contact with reversed roles.                                                                                                                                                                                                                                                                 |
| PRESENT/NT on non-first block                    | The chains are shared up to this point, but branch beyond it.                                                                                                                                                                                            | These nodes can't exactly reconcile their chains - response is settings dependent, but generally transmit END and finish the contact. A client set to "give in" may implement the server's version, and a server set likewise might repeat the contact with reversed roles to get the client's version. |
| ABSENT/V                                         | Assuming this is the server's last block, the client either has a totally different chain or none at all. In the former case, the client can't reconcile - see PRESENT/NT on non-first block. In the latter, the client has now received the full chain. | Transmit END and finish the contact.                                                                                                                                                                                                                                                                    |
| ABSENT/NV                                        | According to client, there are errors in the server's chain.                                                                                                                                                                                             | The server will verify its chain. If it agrees, it will discard the faulty segment and restart this process.                                                                                                                                                                                            |
