The core application connects to a configurable local port for a controller connection.
Usually, this controller will take the form of a UI application running a socket server.

Broadly speaking, there are two classes of requests that a UI can make:
- Additions (a)
- Queries (q)

These are "a" and "q" in the "t" (type) field of the json request.
The "u" type, for user account, is distinguished, though it would be considered a query. 
"q" is reserved for message retrieval.

The format for either is as follows:
```
{
    "t": type ("a" or "q" or "u"),
    "dm": whether or not target tripcode is a DM (0 or 1, only for "a" or "q"),
    "s": target tripcode,
    "c": (json) message content, in format described by intraserver.txt (only for "a"),
    "r": range of messages, backward from blockchain end - [start, end]
}
```
Meanwhile, a response will be of the form below:
```
{
    "err": code for error,
    "c": array of decrypted (json) messages (only for "q" or "u", full declaration(s) for "u"),
    "rr": actual range retrieved - [start, end],
    "p": whether or not this data was requested (0 or 1)
}
```
*#note: in the array of decrypted messages, each one has the "sig": 0 or 1 feature, describing whether verification was successful.*
*#note: Whenever new messages are received by the core while such a connection is open, json in
the response format will be sent containing said messages with p: 0.*