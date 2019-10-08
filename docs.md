# Accounts documentation

## What are accounts?

The Accounts CorDapp allows a CorDapp developer to split a Corda node's vault up into logical sub-vaults. For further 
information please watch [this video](https://youtu.be/u_Hr79dvUHk?t=182).

Not all states stored in a node's vault need to be allocated to an account. Only those states which are held by 
`PublicKey`s assigned to an account ID are held by accounts.

![High level pictorial description of accounts](docs/high-level.png)

## Accounts and the `AccountInfo` state

The base building block for the Accounts CorDapp is the `AccountInfo` state. It is a `ContractState`, so can be shared
with other nodes and contains basic information about an account:

* Account host (`Party`) which is used to map `PublicKey`s to a host node
* Account name (`String`) which is usually a human readable string to identity the account. It must be unique at the 
  account host level but may not be unique at the network level. The tuple of account host and account name is 
  guaranteed to be unique at the network level.
* Account ID (`UUID`) which is a 128-bit random ID that *should* be unique at the network level. The account ID is used 
  by Corda to map `PublicKey`s to accounts.   

For the time being, unlike regular Corda nodes which have a default key pair, the Accounts CorDapp intentionally doesn't 
create a new key pair and associate the `PublicKey` as the default key for a new account. Instead, when you wish to 
transact with an account the `RequestKey` flow should be used to request that an account on a local or remote node 
generate a new key pair and send back the `PublicKey` for use in a transaction.

`AccountInfo` states can be shared with other nodes using the `ShareAccountInfoFlow` and requested from other nodes 
using the `RequestAccountInfoFlow`. These flows will be covered in more detail later on in the documentation.

> There are also initiating versions of these flows which can be startable by a `CordaService` or via RPC. The naming 
> convention is such that in-line sub flows all have the word `Flow` as a suffix and the initiating versions of the 
> flows startable via RPC and service have the `Flow` suffix removed.

In this light, `AccountInfo`s act as a directory entry for accounts hosted on the local node as well as any other remote 
node. It is possible to enumerate all local accounts, or all accounts for a specific remote node, or indeed all accounts 
the node in question is aware of. By this point you are probably aware that there is a one-to-many relationship between 
node to account: a Corda node can host many accounts and this is depicted particularly well by the image below created 
by Peter Li from R3's developer relations team. 

![How accounts relate to nodes](docs/hierarchy.png)

The image shows that a node can host many accounts and that:

1. an account hosted by a node can transact with an account hosted by another node
2. two accounts hosted by the same node can transact with each other
3. an account hosted by a node can transact with a regular Corda node

### "But what IS an account?"

At this point it is important to remember than an account on a node is _just_ a collection of `PublicKey`s which have
been all assigned to the same `UUID`, which is the `linearId` of the `AccountInfo` state (described above). These 
`PublicKey`s can then be used to participate in (or own) `ContractState`s and that's how Corda determines which account 
a `ContractState` belongs to. When the transaction containing said `ContractState` is committed to the ledger, we can 
then say _that_ `ContractState` is owned or participated in by the account that the `PublicKey` is allocated to. When 
you wish to query the vault it is possible to specify from which account you wish to query states from.

That's all there is to accounts. To re-iterate and summarise:

1. `AccountInfo`s are used as an immutable record of the account ID, host and name at the network level
2. A host node can create new key pairs and allocate those key pairs to an account they host
3. A key pair allocated to an account can be used to participate in a `ContractState` - when that state is committed to
   the ledger with a transaction then we can say that that `ContractState` is owned/participated in by the account to
   which the `PublicKey` is allocated
4. The vault can be queried on a per account basis to return only the `ContractState`s for a specific account or set of
   accounts.   
   
For further information on how accounts work "under the hood" check out the appendix of this document where there is a 
video explaining how it all fits together.   
   
## Typical workflows

### Creating new accounts

A new account can be created by either calling the `CreateAccount` flow via RPC or from an existing flow, 
or by calling `AccountService.createAccount`. The `CreateAccount` method simply calls the `CreateAccount` flow and 
returns a `CordaFuture` which completes when the `CreateAccount` flow successfully terminates.

To create an account, you must specify an account name. the account ID is chosen randomly by the Accounts CorDapp. When
the `CreateAccount` flow terminates, it returns a `StateAndRef<AccountInfo>` for the account which has just been 
created. 

The `CreateAccount` flow does not communicate with any nodes as it performs an internal process only (a notary signature
is not required for state creation when the creating transaction does not include a `TimeWindow`).

### Creating new keys for accounts and assigning state ownership to accounts

### Looking up an account by `PublicKey`
### Looking up a host by `PublicKey`
### Obtaining a list of accounts
### Obtaining all the `PublicKey`s for an account
### Requesting the `AccountInfo` from another node by using the account ID

## Appendix

### Examples

The best way to learn is by looking at existing code. In that light check-out these example CorDapps which use the
Accounts CorDapp:

* [SupplyChain](https://github.com/peterli-r3/Accounts_SupplyChain) (By Peter Li, R3) 
* [Sweepstake]() (By Will Hester, R3)
* [GoldTrading]() (By Stefano Franz, R3)

### Why use accounts?

Please see [this video](https://www.youtube.com/watch?v=u_Hr79dvUHk) where Roger explains why we created accounts and 
the benefits of using accounts.

### How does accounts work under the hood?

Please see [this video](https://www.youtube.com/watch?v=nHljbpe3NDY) where Roger explains how accounts work "under the 
hood".