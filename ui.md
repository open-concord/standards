The core application connects to a configurable local port for a controller connection.
Usually, this controller will take the form of a UI application running a socket server.

Broadly speaking, there are four classes of requests that a UI can make:
- Additions (a)
- Queries (q)
- Data changes (c)
- Keygen (g)

These are "a", "q", and "k" in the "t" (type) field of the json request.
The "u" type, for user account, is distinguished, though it would be considered a query
by most. "q" is reserved for message retrieval.

For any request, the following features are needed:
```
{
    "t": type ("a", "q", or "c"),
    "u": identity tripcode,
    "s": target server/DM tripcode
}
```
As you can see, one controller can take on multiple "user" identities (i.e. tripcodes), with separate sig keys and servers.
In **addition** to these features, each type has separate arguments.
Both additions and queries need a message type to add or retrieve, and hence:
```
{
    "mt": message type ("s" for server, "dm" for DM, "dec" for declaration)
}
```
For additions, we need content as well. 
```
{
    "c": (json) content, in format described by the relevant of intraserver.txt or declaration.txt
}
```
For queries, on the other hand, we need a range of messages to retrieve:
```
{
    "r": range of messages, backward from blockchain end - [start, end]
}
```
For changes to user data, a number of fields are available (all fields are optional)
```
{
    "targets": [array of fields from those below to be changed],
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
Meanwhile, a response to any request will be of the form below:
```
{
    "err": code for error,
    "c": type-dependent content,
    "p": whether or not this data was requested (0 or 1)
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
*#note: for "kt": "AES", both pub and pri are the same symmetric key.*

*#note: in the array of decrypted messages, each one has the "sig": 0 or 1 feature, describing whether verification was successful.*

*#note: Whenever new messages are received by the core while such a connection is open, json in the response format will be sent containing said messages with p: 0.*
