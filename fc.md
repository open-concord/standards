# FC - Flag Content (Standard)
## Header
Each message will be proceeded by a 4 digit byte size. This byte size is for the incoming message, and does not include the header itself. <br>
Any given message's first 4 digits is the size of the message - `0025QWERTYUIOPASDFGHJKLZXCVBN`; 25 bytes stated in the header, 25 bytes of message.

## Structure 

FC is used for node communication, and is incredibly simple. It consists only of a purpose-identifying flag and a content section for arbitrary data.
```
{
    FLAG: <flag>,
    CONT: <content>
}
```

## ecDH(M) Context
Concord assumes EC P-256, as per NIST [recommendations](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-78-4.pdf) for federal employee verification. However, if for some reason, you want to use another ecDH(M) context, it can be set before the `Key Exchange` FC. If you're the connecting node, it would be wise to send this information intially.
| Flag    | Direction  | Contents                                                                 | Next Action         |
|---------|------------|--------------------------------------------------------------------------|-------------------------|
| CTX-E     |    C -> H  | Alias for requested ecDH(M) context                                      | CTX-R       |
| CTX-R     |    H -> C | Confirmation that host has received context (no content) | Channel establishment |  


## Suspend & Heartbeat
Sometimes during an extended or semi-permanent connection, nothing happens. The suspend and hearbeat actions enable this without wasting system resources on silently dropped connections. The direction is intially bilateral, but after the `SUSPEND` flag is send, the direction is between `Sender` and `Receiver`.
| Flag    | Direction | Contents                                                                | Next Action                 |
|---------|-----------|-------------------------------------------------------------------------|-----------------------------|
| SUSPEND |    S -> R | Timeout interval after last heartbeat, as n milliseconds                | (R) Wait n milliseconds     |
| HBEAT   |    S -> R | Nothing                                                                 | (R) Restart timeout         |
| \<any\> | Bilateral | \<appropriate content\>                                                 | \<appropriate next action\> |  

## Direct Channels - [HCLC](hclc.md)

To establish a connection between exactly two nodes - the only purpose for which is HCLC merging - a simple DHM exchange is used. The exchange is as follows:
| Flag    | Direction  | Contents                                                                 | Next Action         |
|---------|------------|--------------------------------------------------------------------------|-------------------------|
| KE      | Bilateral  | Newly Generated p2p Public Key                                           | Socket Connection       | 

By the standard DHM algorithm, a shared AES key is generated independently by both parties, which is then used to encrypt the remainder of the exchange.

## Circular Channels - used for mining pools and verification circles

To establish a connection between 3+ nodes, usually for pooling or the formation of a verification circle, multi-party DHM is applied, with the procedure below being used:
| Flag    | Direction  | Contents                                                                 | Next Action         |
|---------|------------|--------------------------------------------------------------------------|-------------------------|
| NCHECK  | N<sub>n</sub>->N<sub>n+1</sub>   | List of involved nodes in the exchange in an arbitrary order (original propagator first) | If the recipient node completes the list, NKEY; otherwise continued NCHECK |
| NKEY    | N<sub>n</sub>->N<sub>n+1</sub> | Pub key integrating node n only                     | NKEY, MVKEY until cycle complete |
| MVKEY   | N<sub>n</sub>->N<sub>n+1</sub> | Pub key integrating nodes a...n, values of a and n  | If N<sub>n+1</sub> completes the cycle, first encrypted message; otherwise continued MVKEY |
Reminder: CTX messages establish a channel. In the case that an alternative DHM context is used for a circular channel, it should be sent before each NCHECK message so that it's active in every connection making up the cycle.
