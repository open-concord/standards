The core application connects to a configurable local port for a controller connection.
Usually, this controller will take the form of a UI application running a socket server.

Broadly speaking, there are four classes of requests that a UI can make:
- Additions (a)
- Queries (q)
- Data changes (c)
- Keygen (g)

These are "a", "q", "c", and "g" in the "t" (type) field of the json request.

For any request, the following features are needed:
```
{
    "t": type ("a", "q", "c", or "g"),
    "ch": target chain tripcode,
    "u": identity tripcode,
    "s": target server/DM tripcode
}
```
As you can see, one controller can take on multiple "user" identities (i.e. tripcodes), with separate sig keys and servers.
In **addition** to these features, each type has separate arguments.
Both additions and queries need a message type to add or retrieve, and hence:
```
{
    "mt": message type ("s" for server, "p" for PM (DM), "d" for declaration)
}
```
For additions, we need content as well. 
```
{
    "c": (json) content, in format described by the relevant of intraserver.txt or declaration.txt
}
```
For queries, on the other hand, we need a range of messages to retrieve;
```
{
    [string] "l": <optional> hash of last block in record (to allow for UI message caching)
    [int] "i": <conditional: l> amount of messages from 'l' (-1 signifies to end of chain)
    "r": range of messages, backward from blockchain end - [start, end] (-1 signifies all)
}
```
We also need to discriminate by intraserver message type:
```
{
    [string] "imt": "p" for plain messages, "m" for member changes, "r" for role changes, "s" for settings changes 
}
```
For changes to user data, a number of fields are available (all fields are optional)
```
{
    "servkeys": [array of {
        "s": server tripcode,
        "k": new key
    }],
    "sigkey": new DSA private key,
    "enckey": new RSA private key
}
```
For keygen, simply key type suffices (in fact, only this is required):
```
{
    "kt": key type ("RSA", "DSA", or "AES")
}
```

----

Meanwhile, a response to any request will be of the form below:
```
{
    "err": code for error,
    "t": response type (same as request type),
    "c": type-dependent content,
}
```
For additions and data changes:
```
{
    "c": {
        "success": 0 or 1
    }
}
```
For queries:
```
{
    "c": {
        "m": [array of decrypted message jsons],
        "rr": real range retrieved - [start, end]
    }
}
```
For keygen:
```
{
    "c": {
        "pub": generated public key,
        "pri": generated private key,
        "kt": returned key type
    }
}
```
Some events will be sent as responses without a request, notifying the controller:<sup>*</sup>:

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

<sup>*</sup>Only one currently.

*#note: the initial contact should involve core sharing a list of available chains*

*#note: for "kt": "AES", both pub and pri are the same symmetric key.*

*#note: in the array of decrypted messages, each one has the "sig": 0 or 1 feature, describing whether verification was successful.*

*#note: Whenever new messages are received by the core while such a connection is open, json in the response format will be sent containing said messages with p: 0.*
