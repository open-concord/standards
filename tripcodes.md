# Tripcodes

## Users

Concord users are uniquely identified by two keypairs - DSA and RSA. The DSA keypair allows the messages of users to be verified and properly attributed, while the RSA keypair enables the formation of user-specific secret channels so that access information can be sent to them.

So that users can be referred to easily, the public portions of both of these keypairs are hashed to provide a unique identifier that is necessitated by the keys and can easily be verified based on them by others.

That is, user tripcodes are (in pseudocode):
```
SHA256-hash(b64(DSA public key) + b64(RSA public key))
```

## Private Servers

Like users, tripcodes are used to identify servers. Private servers are specified by a single symmetric key, making the implementation of tripcodes much simpler. In the vein of the above, private server tripcodes are:
```
SHA256-hash(b64(AES key))
```
This is used to identify the server on the greater Concord chain in such a way that it can be found by users with only the access-conferring AES key.

## Public Servers

Public servers are distinct blockchains hosted on a Concord chain, so they're uniquely defined by their key acceptance protocol - a transparent Haskell function encoded within them. Their tripcodes are formed from this function, alongside a unique server number to allow clones of a particular function to be distinct without superfluous code changes to be necessary. Overall, the tripcode is expressible this way:
```
SHA256-hash(b64(server number)+acceptance protocol code)
```