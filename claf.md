# CLAF - Concord Limited Access Format

## Basic structure

Once decrypted, Concord messages can be interpreted in any agreed-upon way.
Included in the core framework is a standard for controlling specifically the
access management elements of a chat server. Below, CLAF (the Concord Limited Access Format):

Any message will have the following characteristics:
```
{
    a: author (member) tripcode,
    h: hash of timestamp + server tripcode + for each parent block, that block's hash
    st: message supertype,
    d: message data
}
```
> The point of including a hash is to make sure the block's full context is part of the signature. 
Without it, you could spoof a new copy of a signed message which might have a different meaning when sent at a different position. 
However, if two blocks have exactly the same particulars (parent blocks, timestamps, all content, and server trip), they're identical (and will even have the same block hash). 
Since the hash of these features is part of the content and therefore signed with it, the signature is only good for that specific combination, so copying it achieves nothing.

Four supertypes exist:
- "c", for independent content (messages, links, images, etc; see cmark.txt)
- "a", for access-affecting content; this section includes successor server creation, member additions, and such.
- "r", for permissions-affecting content (expressed in terms of named roles)
- "s", for settings-affecting content (not connected to chain interpretation, but strung together into one settings object)

Of course, independent content can have any data.
However, for the other four supertypes, a fixed set of actions can be invoked.
In these cases, a "t" feature describing type will exist, and data will be prescribed by that type. Actions specified by a "t" value will be referred to as commands.
## Commands by supertype
### Access
```
t: "nserv" - set up a new server (will be ignored if not first nserv message on the server)
d: {
    cms: {
        enc_pubk: RSA public key,
        sig_pubk: DSA public key
        (both for server creator)
    }
    prev_key: predecessor server AES key (if applicable)
}
```
```
t: "invite" - invite one or more users to the server
d: {
    nms: [for each member, {
        enc_pubk: RSA public key,
        sig_pubk: DSA public key
    }]
}
```
> note: hash sig_pubk + enc_pubk to get member trip, by definition
```
t: "rem" - forward to a new server lacking one or more users
d: {
    rms: [removed member tripcodes],
    nst: new server trip,
    archive: [head block hashes, not including this one or its parents],
    nsk: {
        (preserved) member trip: RSA-encrypted AES key for new server
    }
}
```
### Permissions
> Role permissions are expressed as a single integer, encoding 6 bits, alongside a second integer expressing role power.
> - 0 - Is muted
> - 1 - Can invite
> - 2 - Can remove less powerful users (user power = max role power)
> - 3 - Can grant/remove less powerful roles
> - 4 - Can create less powerful roles
> - 5 - Can edit settings

> From the perspective of an end-user, roles could do more than this. However, the access client will need to implement other features by checking server roles and matching names with settings-based configuration. That is to say, these are the CLAF-level features of roles. Other features would be implemented in a Concord access client.

```
t: "crole" - create a role
d: {
    rn: role name (should be unique),
    rp: role primacy,
    pc: permissions char
}
```
```
t: "grole" - grant a role
d: {
    tu: target user trip,
    tr: target role name
}
```
```
t: "rrole - remove a role
d: <symmetric with grole (same args)>
```

### Settings
```
t: "sset" - set settings
d: {
    sn: [[setting keys]],
    po: [bool - parse objects, defaults to true],
    sv: [setting values (JSON)]
}
```
```
t: "cset" - clear settings
d: {
    sn: [[settings keys]]
}
```

## Non-linearity

However, Concord is not a linear format, and consequently neither is CLAF.
This is not terribly disruptive when it comes to independent content messages - aside from the occasional branching when messages are sent simultaneously - but,
in server administration (the other three types), it makes a massive difference. If a user uses a role permission at the same time as that permission is revoked, what happens?
CLAF's non-linear procedures are defined by the following axioms: 
1. Messages are only affected by their predecessors
2. When splits containing different versions of persistent features (e.g. settings, role permissions, etc.) are merged, the state of each feature that has been subjected to the most changes is used. In the case of settings, which is an object, this is done recursively (a particular object can be merged, but if it hasn't been reset then changes to individual sub-objects will be merged likewise).
3. Rem messages only record server messages that are their predecessors (this is done to prevent server activity post-rem; the rem relies on all of its predecessors, so they're known to predate it).

By applying these, it's possible to determine the result of CLAF parsing on any server chain.
As long as branches are quickly resolved in the end, this architecture allows users to act freely without infringing on the operational integrity of the server.
