# Declaration

A user may declare their persistent self to the block.

***This is not required for participation in Concord messaging.***

Doing so requires submitting an unencrypted block of this form (Declarant trip should be used as server trip):

```{
    "cont": "{
        "d": <declarant trip code (for validation)>,
        "keys": {
            "sig_pubk": <DSA pubkey>,
            "enc_pubk": <RSA pubkey>
        },
        (optional) "n": <declarant user name>,
        (optional) "imgli": <declarant profile image link>
    }",
    "sig": <signature with the broadcasted pub key for the "cont" section>
}
```
Alternatively, for server-specific identification, a user may make a declaration specifically to a server.

Once a user declares themself thusly, their trip code can be considered as a server without membership.
That is, all normal procedures for a server apply, except:
 - RSA is used instead of asymmetric encryption so that only they can receive messages.
 - Any pubkey is allowed, though client-side filtration may be applied.
