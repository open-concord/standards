# HCLC - Host Client Layer Comparison
The main purpose of node communication is to facilitate merging, for which the HCLC (Host-Client Layer Comparison) protocol is used. HCLC uses the [FC format](fc.md).

To begin an HCLC interaction, a node (referred to as the client) will initiate by sending a `{FLAG: COPEN, CONTENT: {chain: <chain>, val: [<valence hashes>]}}` communication, where `<chain>` is the desired chain's tripcode, and `<valence hashes` is an array of hashes for the chain's valence layer on the client's side. The recipient will be referred to as the host.

----

The ready block prompts the server to begin sending blocks. Packages of blocks are sent in such a manner that, until no blocks remain, they'll continually prompt more packages. To lay out the protocol, it's first worth laying out the flags it supports:
| Flag    | Direction  | Contents                                                                 | Next Action |
|---------|------------|--------------------------------------------------------------------------|-----------------|
| COPEN | C->H | Client valence hashes and target chain | HOPEN |
| HOPEN | H->C | Host valence hashes and needed client blocks | BLOCKS |
| BLOCKS | Any | Content of previously-requested blocks and parent requests for blocks received | BLOCKS or DONE |
| END | Any | None (occurs when no blocks are needed and none were requested) | Close Connection | 

In order to make this feedback loop as efficient as possible, blocks are transmitted only when reported as needed. To illustrate:

```Client (COPEN): Sends its valence messages and target chain.```

```Host (HOPEN): Sends its valence messages, alongside the client valence messages that are missing from the host chain.```
 
```Client (BLOCKS): Sends blocks requested and requests for host valence messages that are missing from the host chain.```

```Host (BLOCKS): Sends blocks requested and requests for parents of received blocks missing from the host chain.```

```Client (BLOCKS): [Same as above]```

```Host (Blocks): ...```

And so on, until one side has no blocks to request and none requested from it.