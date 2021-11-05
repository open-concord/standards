# Concord Structure

Concord is a system of [nodes](nodes), which form connections using the [FC](fc) standard. FC enables them to gather in [mining pools](pooling) to verify blocks and [verification circles](circling) to ensure accountable data storage, both activities which enable the chain to continue and earn them [fungible hash second tokens](hseconds) which they can spend to send messages. Alongside supporting the chain this way, they form it by constantly receiving updates from neighbors through the [HCLC protocol](hclc). These updates affect the collections of self-organizing [COBS blocks](cobs) that the nodes hold. Block content can belong to two categories:
- Private server messages, the content of which is usually [encrypted and signed](encryption) data in the [CLAF](claf) standard. CLAF messages form internally-regulated and fully secure private servers filled to the brim with messages, which can be accessed through a [UI](ui) connected to the node. 
- Public server messages, constituting sub-ledgers that can be just as complete as COBS, for which the [CSCS](cscs) standard is used. Standard nodes use one such sub-ledger, the [Datrade public server](datrade), to exchange hash seconds and form contracts, enabling the sending privileges resulting from mining to be conditionally transferred to others. Another, the [Decforum public server](decforum), allows users to create profiles in a uniform format that can remove redundancies where information would otherwise be stored separately on multiple servers.

For user and server identity in both categories, the [tripcode system](tripcodes) is used.