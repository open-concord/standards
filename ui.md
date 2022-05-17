### [Direct Reference](#dref)
<hr>

### <a name="dex">Detailed Explanation</a>
The core application connects to a configurable local port for a controller connection.
Usually, this controller will take the form of a UI application running a socket server.

Broadly speaking, there are four classes of requests that a UI can make:
- [Additions](#add) (a)
- [Queries](#query) (q)
- [Identity Additions](#id_add) (u)
- [Identity Modification](#id_mod) (m)
- [Misc. Utilities](#misc) (t)

These are "a", "q", "u", "m", and "t" in the "t" (type) field of the json request.

```
{
    [char] "t": request type
}
```

For an addition or query, the following features are needed:

```
{
    [string] "ch": <target chain tripcode>,
    [string] "u": <identity tripcode>,
    [string] "s": <target server/DM tripcode>
}
```

As you can see, one controller can take on multiple "user" identities (i.e. tripcodes), with separate sig keys and servers.

In **addition** to these features, each type has separate arguments.

Both additions and queries need a message type to add or retrieve, and hence:
```
{
    [char] "mt": <message type> // ("s" for server, "p" for PM (DM), "d" for declaration)
}
```
For additions, we need content as well. 
```
{
    [string || object] "c": [json] <content> // in format described by the relevant of intraserver.txt or declaration.txt
}
```
For queries, on the other hand, we need a range of messages to retrieve;
```
{
    [string] "l": <optional> // hash of last block in record (to allow for UI message caching)
    [int] "i": <conditional: l> // amount of messages from 'l' (-1 signifies to end of chain)
    [int[2]] "r": // range of messages, backward from blockchain end - [start, end] (-1 signifies all)
}
```

We also need to discriminate by intraserver message type in queries:

```
{
    [char] "imt": "p" /** for plain messages*/ "m" /** for member changes */ "r" /** for role changes */ "s" /** for settings changes */
}
```
For identity addition, no data is needed if the user is to be generated. However, if the user is to be set, the following is required:

```
{
    [object] "pub_keys": {
        [string] "sig_pubk": <DSA public key>,
        [string] "enc_pubk": <RSA public key>
    },
    [object] "pri_keys": {
        [string] "sig_prik": <DSA private key>,
        [string] "enc_prik": <RSA private key>
    }
}
```

Identity modification just covers the addition of server AES keys, so it has the following data:

```
{
    [object] "serv_keys": {
        Map: {<server tripcode>: <server AES key>}
    }
} 
```

Misc. Utilities have only a single necessary field, Utility Type (ut).

AES encryption/decryption can be requested:

```
{
    "ut": "e",
    "aes_key": <AES key> // as generated with keygen
    "direction": 0 /** for encryption */ 1 /** for decryption*/
    "plain": plaintext // if encrypting
    "nonce": nonce // if decrypting
    "cipher": ciphertext // if decrypting
}
```

Keygen of any type can also be requested:

```
{
    "ut": "k",
    "kt": <key type> // (AES || DSA || RSA)
}
```


----

Meanwhile, a response to any request will be of the form below:

```
{
    [string] "err": <code for error>,
    [char] "t": <response type> // same as request type
    [object] "c": <type-dependent content>
}
```

For additions, user addition with defined keys, and user modification:

```
{
    "c": {
        [int] "success": 0 || 1
    }
}
```

For queries:

```
{
    [object] "c": {
        [object[]] "m": [<array of decrypted message jsons>],
        [int[2]] "rr": [<start>, <end>] // <real range retrieved>
    }
}
```

For user addition with undefined keys, the "c" field just contains the fields that were not set (i.e. "pub_keys" and "pri_keys"), with the addition of a user tripcode generated from the pub keys:
```
{
    [object] "c": {
        [object] "pub_keys": ...,
        [object] "pri_keys": ...,
        [string] "u": <generated user tripcode>
    }
}
```



For AES encryption, responses are in the same standard as requests, with the "c" field in the same "plain"-"nonce"-"cipher" format. E.g. encryption yields nonce and cipher in the "c" field.

For keygen:

```
{
    "c": {
        "pub": <generated public key>,
        "pri": <generated private key>,
        "kt": <key type> // from the request)
    }
}
```

Some events will be sent as responses without a request, notifying the controller:

"nc", representing the addition of a new chain in cores allowing them.
```
{
    "c": {
        "ch": <new chain tripcode>
    }
}
```

"nb", representing the addition of new blocks to a chain
```
{
    "c": {
        "ch": <chain tripcode>,
        "bc": <block count>
    }
}
```

*#note: the initial contact should involve core sharing a list of available chains*

*#note: for "kt": "AES", both pub and pri are the same symmetric key.*

*#note: in the array of decrypted messages, each one has the "sig": 0 or 1 feature, describing whether verification was successful.*

*#note: Whenever new messages are received by the core while such a connection is open, json in the response format will be sent containing said messages with p: 0.*



---
### <a name="dref">Direct Reference</a>

<details>
    <summary><b><a name="add">Addition</a></b></summary>
<br>

<table>
<tr>
<td> Input </td>
</tr>
<tr>
<td>
    
```
{
    [char] "t": 'a',
    [string] "ch": <target chain tripcode>,
    [string] "u": <identity tripcode>,
    [string] "s": <target->server/DM tripcode>,
    [char] "mt": <message type> // "s" for server, "p" for PM (DM), "d" for declaration,
    [string || JSON object] "c": <content> // in format described by the relevant of intraserver.txt or declaration.txt
}
```
    
</td>
</tr>
</table>

<table>
<tr>
<td> Return </td>
</tr>
<tr>
<td>
    
```
{
    [string] "err": code for error,
    [char] "t": <response type>, // same as request type
    [object] "c": {
        [int] "success": <0 or 1> // treat as a bool
    }
}
``` 

</td>
</tr>
</table>
</details>

<details>
    <summary><b><a name="query">Queries</a></b></summary>
<br>

<table>
<tr>
<td> Input </td>
</tr>
<tr>
<td>
    
```
{
    [char] "t": 'q',
    [string] "ch": <target chain tripcode>,
    [string] "u": <identity tripcode>,
    [string] "s": <target->server/DM tripcode>,
    [char] "mt": <message type>, // "s" for server, "p" for PM (DM), "d" for declaration
    [string] "l": <optional>, // hash of last block in record (to allow for UI message caching)
    [int] "i": <conditional: l>, // amount of messages from 'l' (-1 signifies to end of chain)
    [int[2]] "r": <index0, index1>, // range of messages, backward from blockchain end - [start, end] (-1 signifies all)
    [char] "imt": "p" /** for plain messages*/ "m" /** for member changes */ "r" /** for role changes */ "s" /** for settings changes */
}
```
    
</td>
</tr>
</table>
</details>

<details>
    <summary><b><a name="id_add">Identity Addition</a></b></summary>
<br>

<table>
<tr>
<td> Input (User Generation) </td>
</tr>
<tr>
<td>
    
```
{
	[char] "t": 'u' // literally nothing besides the type
}
```
    
</td>
</tr>
</table>
	<table>
<tr>
<td> Input (User Set) </td>
</tr>
<tr>
<td>
    
```
{
	[char] "t": "a",
	[object] "pub_keys": {
		[string] "sig_pubk": <DSA public key>,
		[string] "enc_pubk": <RSA public key>
	},
	[object] "pri_keys": {
		[string] "sig_prik": <DSA private key>,
		[string] "enc_prik": <RSA private key>
	}
}
```
    
</td>
</tr>
</table>
</details>

<details>
    <summary><b><a name="id_mod">Identity Modification</a></b></summary>
<br>

<table>
<tr>
<td> Input </td>
</tr>
<tr>
<td>
    
```
{
	[char] "t": 'm',
	[map<string, string>] "serv_keys": {
		<server tripcode>: <server AES key>
	}
}
```
    
</td>
</tr>
</table>
</details>
	
<details>
    <summary><b><a name="misc">Misc. Utility</a></b></summary>
<br>

<details>
	<summary><b>AES Encryption</b></summary>
<table>
<tr>
<td> Input (AES Enc) </td>
</tr>
<tr>
<td>
    
```
{
	[char] "t": 't',
	[char] "ut": 'e',
	[string] "aes_key": <AES Key>,
	[int] "direction": 0,
	[string] "plain": <plaintext>
}
```
    
</td>
</tr>
</table>
</details>
<details>
<summary><b>AES Decryption</b></summary>
<table>
<tr>
<td> Input (AES Dec) </td>
</tr>
<tr>
<td>
    
```
{
	[char] "t": 't',
	[char] "ut": 'e',
	[string] "aes_key": <AES Key>,
	[int] "direction": 1,
	[string] "nonce": <nonce>,
	[string] "cipher": <ciphertext>
}
```
    
</td>
</tr>
</table>
</details>
<details>
<summary><b>Key Generation</b></summary>
<table>
<tr>
<td> Input (AES) </td>
</tr>
<tr>
<td>
    
```
{
	[char] "t": 't',
	[char] "ut": 'k',
	[string] "kt": "AES"
}
```
    
</td>
</tr>
</table>
<table>
<tr>
<td> Input (DSA) </td>
</tr>
<tr>
<td>
    
```
{
	[char] "t": 't',
	[char] "ut": 'k',
	[string] "kt": "DSA"
}
```
    
</td>
</tr>
</table>
<table>
<tr>
<td> Input (RSA) </td>
</tr>
<tr>
<td>
    
```
{
	[char] "t": 't',
	[char] "ut": 'k',
	[string] "kt": "RSA"
}
```
    
</td>
</tr>
</table>

</details>
	
[Back To Top](#dex)
