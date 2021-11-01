# FC

The FC (Flag-Content) standard, used for node communication, is incredibly simple. It consists of a purpose-identifying flag, a content section for arbitrary data, and a signature section where the combination of the above two (in that order) can be signed.
```
{
    FLAG: <flag>,
    CONT: <content>,
    SIG: <sig(flag + content)> 
}

```
