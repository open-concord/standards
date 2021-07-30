The core application connects to a configurable local port for a controller connection.
Usually, this controller will take the form of a UI application running a socket server.

Broadly speaking, there are four classes of requests that a UI can make:
- [Additions](#add) (a)
- Queries (q)
- User Additions (u)
- User Modification (m)
- Misc. Utilities (t)

These are "a", "q", "u", "m", and "t" in the "t" (type) field of the json request.

```
{
    [char] "t": request type
}
```

For an addition or query, the following features are needed:

```
{
    [string] "ch": target chain tripcode,
    [string] "u": identity tripcode,
    [string] "s": target server/DM tripcode
}
```

As you can see, one controller can take on multiple "user" identities (i.e. tripcodes), with separate sig keys and servers.

In **addition** to these features, each type has separate arguments.

Both additions and queries need a message type to add or retrieve, and hence:
```
{
    [char] "mt": message type ("s" for server, "p" for PM (DM), "d" for declaration)
}
```
For additions, we need content as well. 
```
{
    [string || object] "c": (json) content, in format described by the relevant of intraserver.txt or declaration.txt
}
```
For queries, on the other hand, we need a range of messages to retrieve;
```
{
    [string] "l": <optional> hash of last block in record (to allow for UI message caching)
    [int] "i": <conditional: l> amount of messages from 'l' (-1 signifies to end of chain)
    [int[2]] "r": range of messages, backward from blockchain end - [start, end] (-1 signifies all)
}
```

We also need to discriminate by intraserver message type in queries:

```
{
    [char] "imt": "p" for plain messages, "m" for member changes, "r" for role changes, "s" for settings changes 
}
```
For user addition, no data is needed if the user is to be generated. However, if the user is to be set, the following is required:

```
{
    [object] "pub_keys": {
        [string] "sig_pubk": DSA public key,
        [string] "enc_pubk": RSA public key
    },
    [object] "pri_keys": {
        [string] "sig_prik": DSA private key,
        [string] "enc_prik": RSA private key
    }
}
```

User modification just covers the addition of server AES keys, so it has the following data:

```
{
    [object] "serv_keys": {
        Map: server tripcode -> server AES key
    }
} 
```

Misc. Utilities have only a single necessary field, Utility Type (ut).

AES encryption/decryption can be requested:

```
{
    "ut": "e",
    "aes_key": AES key, as generated with keygen,
    "direction": 0 for encryption, 1 for decryption,
    "plain": plaintext (if encrypting),
    "nonce": nonce (if decrypting),
    "cipher": ciphertext (if decrypting)
}
```

Keygen of any type can also be requested:

```
{
    "ut": "k",
    "kt": key type (AES, DSA, or RSA)
}
```


----

Meanwhile, a response to any request will be of the form below:

```
{
    [string] "err": code for error,
    [char] "t": response type (same as request type),
    [object] "c": type-dependent content
}
```

For additions, user addition with defined keys, and user modification:

```
{
    "c": {
        [int] "success": 0 or 1
    }
}
```

For queries:

```
{
    [object] "c": {
        [object[]] "m": [array of decrypted message jsons],
        [int[2]] "rr": real range retrieved - [start, end]
    }
}
```

For user addition with undefined keys, the "c" field just contains the fields that were not set (i.e. "pub_keys" and "pri_keys"), with the addition of a user tripcode generated from the pub keys:
```
{
    [object] "c": {
        [object] "pub_keys": ...,
        [object] "pri_keys": ...,
        [string] "u": generated user tripcode
    }
}
```



For AES encryption, responses are in the same standard as requests, with the "c" field in the same "plain"-"nonce"-"cipher" format. E.g. encryption yields nonce and cipher in the "c" field.

For keygen:

```
{
    "c": {
        "pub": generated public key,
        "pri": generated private key,
        "kt": key type (from the request)
    }
}
```

Some events will be sent as responses without a request, notifying the controller:

"nc", representing the addition of a new chain in cores allowing them.
```
{
    "c": {
        "ch": new chain tripcode
    }
}
```

"nb", representing the addition of new blocks to a chain
```
{
    "c": {
        "ch": chain tripcode,
        "bc": block count
    }
}
```

*#note: the initial contact should involve core sharing a list of available chains*

*#note: for "kt": "AES", both pub and pri are the same symmetric key.*

*#note: in the array of decrypted messages, each one has the "sig": 0 or 1 feature, describing whether verification was successful.*

*#note: Whenever new messages are received by the core while such a connection is open, json in the response format will be sent containing said messages with p: 0.*



---
###Direct Reference

<a name="add">**Addition**</a>

```
{
    [char] "t": "a", // request type
    [string] "ch": <target chain tripcode>,
    [string] "u": <identity tripcode>,
    [string] "s": <target->server/DM tripcode>,
    [char] "mt": <message type> // "s" for server, "p" for PM (DM), "d" for declaration,
    [string || JSON object] "c": <content> // in format described by the relevant of intraserver.txt or declaration.txt
}
``` 
