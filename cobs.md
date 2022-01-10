# COBS - Concord Opaque Block Standard

## Basic structure

The atomic unit of the Concord blockhain is, of course, the block. In fact, Concord core stores the chain as a map of hashes to blocks. The information encoded in these blocks is described by the data structure below.

| Feature  | Type                     | Data                                             |
|----------|--------------------------|--------------------------------------------------|
| time     | unsigned long long int   | Time of block creation                           |
| nonce    | string (24 chars)        | Random content added to meet POW requisite       |
| s_trip   | string (24 chars)        | Tripcode for relevant server                     |
| c_trip   | string (24 chars)        | Tripcode for user responsible for POW (optional) |
| cont     | string                   | Content of the block                             |
| hash     | string                   | Hash generated from all features except c_trip   |
| p_hashes | unordered set of strings | Block hashes used to generate this block         |

> Core also records the block's child hashes - hashes of derivative blocks - but this can easily be derived from the parent hashes (p_hashes).

## Generation procedures

> Tripcodes have their own section, so they won't be described here.

### Parent hashes (p_hashes)

An arbitrary number of blocks are selected, entirely at the discretion of the party generating the block, and their hashes are included. The one requirement is that, for server chain comprehensibility, all blocks need to have at least one parent belonging to the same server. This way, a server can be traversed purely by looking at blocks within it, making analysis faster by a massive margin.

### Content

The actual data here is outlined in the section on encryption. The key point, though, is that it's not relevant to the Concord blockchain - that's what makes it the Concord **Opaque** Block Standard. Nonetheless, block contents should always be in a base64 format for easy transmission and processing.

### Time

The blockchain is fully timestamped, but there's no way to verify the accuracy of these stamps. The main function of this is just to make obvious time-misrepresentation difficult (adjacent blocks will have completely disparate times). Time is naturally represented as an unsigned long long int, but for transmission it's expressed as a base64 string for convenience.

### Hash

Hashes allow the blockchain to have structure - the hash of each block depends on its features and the hashes of previous blocks, making the hash of every block sensitive to the details of every predecessor block. The order of features in the seed, arbitrary but necessarily deterministic, is as follows:
```
(base64-encoded time) + (server tripcode) + (creator tripcode) + (content) + (parent hashes, ordered by standard string comparison) + nonce
```
This seed is then hashed with SHA256, the raw output encoded with base64.

### Nonce

So that generating blocks requires computation ("work"), the miner for a particular block must generate a random string that produces a hash beginning with some *n* '0's when added to the end of the seed. Since base64 has 64 possible chars and '0' is but one, the likelihood of this for any particular string is 64^(-n) = (2^6)^(-n) = 2^(-6n). *n* is a feature of the specific Concord chain, described as the POW requisite. Concord uses char comparison for ease of verification, though comparing individual bits to get more granular POW requisities (likely at the cost of verification speed) is a possible future optional feature. 

### Creator Tripcode

To allow credit (and potentially rewards) to be given to miners who solve POW for hashes, the publisher of a particular block can attach their universal user tripcode to it, enabling recognition of their contribution to the chain. This is entirely optional and Concord currently has no miner rewards beyond respect and gratitude, though the usage of this feature allows rewards to be distributed retroactively.

## Transmission Protocol

For merging (see HCLC), blocks are converted into a json format. The encoding is as follows:
```
{
    d: [base64 time, nonce, server trip, creator trip, cont, hash],
    p: [array of p_hashes in standard string order]
}
```
Conversion between formats is essentially just a matter of mapping values.
