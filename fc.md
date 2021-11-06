# FC

## Structure 

The FC (Flag-Content) standard, used for node communication, is incredibly simple. It consists only of a purpose-identifying flag and a content section for arbitrary data.
{
    FLAG: <flag>,
    CONT: <content>
}

## Direct Channels - HCLC

To establish a connection between exactly two nodes - the only purpose for which is HCLC merging - a simple DHM exchange is used. The exchange is as follows:
| Flag    | Direction  | Contents                                                                 | Prompted action         |
|---------|------------|--------------------------------------------------------------------------|-------------------------|
| CKEY    | C->H       | DHM context and newly-generated public key                               | CKEY                    | 
| HKEY    | H->C       | Newly-generated public key                                               | First encrypted message |

By the standard DHM algorithm, a shared AES key is generated independently by both parties, which is then used to encrypt the remainder of the exchange.

## Circular Channels - Pooling

To establish a connection between 3+ nodes, usually for pooling or the formation of a verification circle, multi-party DHM is applied, with the procedure below being used:
| Flag    | Direction  | Contents                                                                 | Prompted action         |
|---------|------------|--------------------------------------------------------------------------|-------------------------|
| NCHECK  | N<sub>n</sub>->N<sub>n+1</sub>   | List of involved nodes in the exchange in an arbitrary order (original propagator first) and DHM context | If the recipient node completes the list, NKEY; otherwise continued NCHECK |
| NKEY    | N<sub>n</sub>->N_<sub>n+1</sub> | Pub key integrating node n only                     | NKEY, MVKEY until cycle complete |
| MVKEY   | N<sub>n</sub>->N_<sub>n+1</sub> | Pub key integrating nodes a...n, values of a and n  | If N<sub>n+1</sub> completes the cycle, first encrypted message; otherwise continued MVKEY |
