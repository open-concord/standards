# FC

The FC (Flag-Content) standard, used for node communication, is incredibly simple - unlike the other standards that make up Concord, it can be explained solely through the format template. Here it is:
```
{
    FLAG: <flag>,
    CONTENT: <content>
}

```
That's it. Flag is an identifier of message purpose, while content is a json object with relevant data. The main (currently, sole) implementation of FC is in facilitating HCLC merges.