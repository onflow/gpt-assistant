---
title: Cadence Anti-Patterns
sidebar_position: 6
sidebar_label: Anti-Patterns
---

This is an opinionated list of issues that can be improved if they are found in Cadence code intended for production.

# Security and Robustness

## Avoid using `AuthAccount` as a function parameter

### Problem

Some may choose to authenticate or perform operations for their users by using the users' account addresses.
In order to do this, a commonly seen case would be to pass the user's `AuthAccount` object
as a parameter to a contract function to use for querying the account or storing objects directly.
This is problematic, as the `AuthAccount` object allows access to ALL private areas of the account,
for example, all of the user's storage, authorized keys, etc.,
which provides the opportunity for bad actors to take advantage of.

### Example:

```cadence
...
// BAD CODE
// DO NOT COPY

// Imagine this code is in a contract that uses AuthAccount to authenticate users
// To transfer NFTs

// They could deploy the contract with an Ethereum-style access control list functionality

pub fun transferNFT(id: UInt64, owner: AuthAccount) {
    assert(owner(id) == owner.address)

    transfer(id)
}

// But they could upgrade the function to have the same signature
// so it looks like it is doing the same thing, but they could also drain a little bit
// of FLOW from the user's vault, a totally separate piece of the account that
// should not be accessible in this function
// BAD

pub fun transferNFT(id: UInt64, owner: AuthAccount) {
    assert(owner(id) == owner.address)

    transfer(id)

    // Sneakily borrow a reference to the user's Flow Token Vault
    // and withdraw a bit of FLOW
    // BAD
    let vaultRef = owner.borrow<&FlowToken.Vault>(/storage/flowTokenVault)!
    let stolenTokens <- vaultRef.withdraw(amount: 0.1)

    // deposit the stolen funds in the contract owners vault
    // BAD
    contractVault.deposit(from: <-stolenTokens)
}
...
```

### Solution

Projects should find other ways to authenticate users, such as using resources and capabilities as authentication objects.
They should also expect to perform most storage and linking operations within transaction bodies
rather than inside contract utility functions.

There are some scenarios where using an `AuthAccount` object is necessary, such as a cold storage multi-sig,
but those cases are extremely rare and `AuthAccount` usage should still be avoided unless absolutely necessary.

## Auth references and capabilities should be avoided

### Problem

[Authorized references](./language/references.md) allow downcasting restricted
types to their unrestricted type and should be avoided unless necessary.
The type that is being restricted could expose functionality that was not intended to be exposed.
If the `auth` keyword is used on local variables they will be references.
References are ephemeral and cannot be stored.
This prevents any reference casting to be stored under account storage.
Additionally, if the `auth` keyword is used to store a public capability, serious harm
could happen since the value could be downcasted to a type
that has functionality and values altered.

### Example

A commonly seen pattern in NFT smart contracts is including a public borrow function
that borrows an auth reference to an NFT (eg. [NBA Top Shot](https://github.com/dapperlabs/nba-smart-contracts/blob/95fe72b7e94f43c9eff28412ce3642b69dcd8cd5/contracts/TopShot.cdc#L889-L906)).
This allows anyone to access the stored metadata or extra fields that weren't part
of the NFT standard. While generally safe in most scenarios, not all NFTs are built the same.
Some NFTs may have privileged functions that shouldn't be exposed by this method,
so please be cautious and mindful when imitating NFT projects that use this pattern.

### Another Example

When we create a public capability for our `FungibleToken.Vault` we do not use an auth capability:

```cadence
// GOOD: Create a public capability to the Vault that only exposes
// the balance field through the Balance interface
signer.link<&FlowToken.Vault{FungibleToken.Balance}>(
    /public/flowTokenBalance,
    target: /storage/flowTokenVault
)
```

If we were to use an authorized type for the capability, like so:

```cadence
// BAD: Create an Authorized public capability to the Vault that only exposes
// the balance field through the Balance interface
// Authorized referenced can be downcasted to their unrestricted types, which is dangerous
signer.link<auth &FlowToken.Vault{FungibleToken.Balance}>(
    /public/flowTokenBalance,
    target: /storage/flowTokenVault
)
```

Then anyone in the network could take that restricted reference
that is only supposed to expose the balance field and downcast it to expose the withdraw field
and steal all our money!

```cadence
// Exploit of the auth capability to expose withdraw
let balanceRef = getAccount(account)
    .getCapability<auth &FlowToken.Vault{FungibleToken.Balance}>(/public/flowTokenBalance)
    .borrow()!

let fullVaultRef = balanceRef as! &FlowToken.Vault

// Withdraw the newly exposed funds
let stolenFunds <- fullVaultRef.withdraw(amount: 1000000)
```

## Events from resources may not be unique

### Problem

Public functions in a contract can be called by anyone, e.g. any other contract or any transaction.
If that function creates a resource, and that resource has functions that emit events,
that means any account can create an instance of that resource and emit those events.
If those events are meant to indicate actions taken using a single instance of that resource
(eg. admin object, registry), or instances created through a particular workflow,
it's possible that events from other instances may be mixed in with the ones you're querying for -
making the event log search and management more cumbersome.

### Solution

To fix this, if there should be only a single instance of the resource,
it should be created and `link()`ed to a public path in an admin account's storage
during the contracts's initializer.

# Access Control

## Public functions and fields should be avoided

### Problem

Be sure to keep track of access modifiers when structuring your code, and make public only what should be public.
Accidentally exposed fields can be a security hole.

### Solution

When writing your smart contract, look at every field and function and make sure
that they are all `access(self)`, `access(contract)`, or `access(account)`, unless otherwise needed.

## Capability-Typed public fields are a security hole

This is a specific case of "Public Functions And Fields Should Be Avoided", above.

### Problem

The values of public fields can be copied. Capabilities are value types,
so if they are used as a public field, anyone can copy it from the field
and call the functions that it exposes.
This almost certainly is not what you want if a capability
has been stored as a field on a contract or resource in this way.

### Solution

For public access to a capability, place it in an accounts public area so this expectation is explicit.

## Array or dictionary fields should be private

<Callout type="info">

This anti-pattern has been addressed with [FLIP #703](https://github.com/onflow/flips/blob/main/cadence/20211129-cadence-mutability-restrictions.md)

</Callout>

### Problem

This is a specific case of "Public Functions And Fields Should Be Avoided", above.
Public array or dictionary fields are not directly over-writable,
but their members can be accessed and overwritten if the field is public.
This could potentially result in security vulnerabilities for the contract
if these fields are mistakenly made public.

Ex:

```cadence
pub contract Array {
    // array is intended to be initialized to something constant
    pub let shouldBeConstantArray: [Int]
}
```

Anyone could use a transaction like this to modify it:

```cadence
import Array from 0x01

transaction {
    execute {
        Array.shouldbeConstantArray[0] = 1000
    }
}
```

### Solution

Make sure that any array or dictionary fields in contracts, structs, or resources
are `access(contract)` or `access(self)` unless they need to be intentionally made public.

```cadence
pub contract Array {
    // array is inteded to be initialized to something constant
    access(self) let shouldBeConstantArray: [Int]
}
```

## Public admin resource creation functions are unsafe

This is a specific case of "Public Functions And Fields Should Be Avoided", above.

### Problem

A public function on a contract that creates a resource can be called by any account.
If that resource provides access to admin functions then the creation function should not be public.

### Solution

To fix this, a single instance of that resource should be created in the contract's `init()` method,
and then a new creation function can be potentially included within the admin resource, if necessary.
The admin resource can then be `link()`ed to a private path in an admin's account storage during the contract's initializer.

### Example

```cadence
// Pseudo-code

// BAD
pub contract Currency {
    pub resource Admin {
        pub fun mintTokens()
    }

    // Anyone in the network can call this function
    // And use the Admin resource to mint tokens
    pub fun createAdmin(): @Admin {
        return <-create Admin()
    }
}

// This contract makes the admin creation private and in the initializer
// so that only the one who controls the account can mint tokens
// GOOD
pub contract Currency {
    pub resource Admin {
        pub fun mintTokens()

        // Only an admin can create new Admins
        pub fun createAdmin(): @Admin {
            return <-create Admin()
        }
    }

    init() {
        // Create a single admin resource
        let firstAdmin <- create Admin()

        // Store it in private account storage in `init` so only the admin can use it
        self.account.save(<-firstAdmin, to: /storage/currencyAdmin)
    }
}
```

## Do not modify smart contract state or emit events in public struct initializers

This is another example of the risks of having publicly accessible parts to your smart contract.

### Problem

Data structure definitions in Cadence currently must be declared as public so that they can be used by anyone.
Structs do not have the same restrictions that resources have on them,
which means that anyone can create a new instance of a struct without going through any authorization.

### Solution

Any contract state-modifying operations related to the creation of structs
should be contained in restricted resources instead of the initializers of structs.

### Example

This used to be a bug in the NBA Top Shot smart contract, so we'll use that as an example.
Before, when it created a new play,
[it would initialize the play record with a struct,](https://github.com/dapperlabs/nba-smart-contracts/blob/55645478594858a6830e4ab095034068ef9753e9/contracts/TopShot.cdc#L155-L158)
which increments the number that tracks the play IDs and emits an event:

```cadence
// Simplified Code
// BAD
//
pub contract TopShot {

    // The Record that is used to track every unique play ID
    pub var nextPlayID: UInt32

    pub struct Play {

        pub let playID: UInt32

        init() {

            self.playID = TopShot.nextPlayID

            // Increment the ID so that it isn't used again
            TopShot.nextPlayID = TopShot.nextPlayID + 1

            emit PlayCreated(id: self.playID, metadata: metadata)
        }
    }
}
```

This is a risk because anyone can create the `Play` struct as many times as they want,
which could increment the `nextPlayID` field to the max `UInt32` value,
effectively preventing new plays from being created. It also would emit bogus events.

This bug was fixed by
[instead updating the contract state in the admin function](https://github.com/dapperlabs/nba-smart-contracts/blob/master/contracts/TopShot.cdc#L682-L685)
that creates the plays.


```cadence
// Update contract state in admin resource functions
// GOOD
//
pub contract TopShot {

    // The Record that is used to track every unique play ID
    pub var nextPlayID: UInt32

    pub struct Play {

        pub let playID: UInt32

        init() {
            self.playID = TopShot.nextPlayID
        }
    }

    pub resource Admin {

        // Protected within the private admin resource
        pub fun createPlay() {
            // Create the new Play
            var newPlay = Play()

            // Increment the ID so that it isn't used again
            TopShot.nextPlayID = TopShot.nextPlayID + UInt32(1)

            emit PlayCreated(id: newPlay.playID, metadata: metadata)

            // Store it in the contract storage
            TopShot.playDatas[newPlay.playID] = newPlay
        }
    }
}
```
---
title: Contract Upgrades with Incompatible Changes
---

### Problem

I have an incompatible upgrade for a contract. How can I deploy this?

### Solution

Please don't perform incompatible upgrades between contract versions in the same account.
There is too much that can go wrong.

You can make [compatible upgrades](./language/contract-updatability.md) and then run a post-upgrade function on the new contract code if needed.

If you must replace your contract rather than update it,
the simplest solution is to add or increase a suffix on any named paths in the contract code
(e.g. `/public/MyProjectVault` becomes `/public/MyProjectVault002`) in addition to making the incompatible changes,
then create a new account and deploy the updated contract there.

⚠️ Flow identifies types relative to addresses, so you will also need to provide _upgrade transactions_ to exchange the old contract's resources for the new contract's ones. Make sure to inform users as soon as possible when and how they will need to perform this task.

If you absolutely must keep the old address when making an incompatible upgrade, then you do so at your own risk. Make sure you perform the following actions in this exact order:

1. Delete any resources used in the contract account, e.g. an Admin resource.
2. Delete the contract from the account.
3. Deploy the new contract to the account.

⚠️ Note that if any user accounts contain `structs` or `resources` from the _old_ version of the contract that have been replaced with incompatible versions in the new one, **they will not load and will cause transactions that attempt to access them to crash**. For this reason, once any users have received `structs` or `resources` from the contract, this method of making an incompatible upgrade should not be attempted!
---
title: Cadence Design Patterns
sidebar_position: 5
sidebar_label: Design Patterns
---

This is a selection of software design patterns developed by core Flow developers
while writing Cadence code for deployment to Flow Mainnet.

Many of these design patters apply to most other programming languages, but some are specific to Cadence.

[Design patterns](https://en.wikipedia.org/wiki/Software_design_pattern) are building blocks for software development.
They may provide a solution to a problem that you encounter when writing smart contracts in Cadence.
If they do not clearly fit, these patterns may not be the right solution for a given situation or problem.
They are not meant to be rules to be followed strictly, especially where a better solution presents itself.

# General

These are general patterns to follow when writing smart contracts.

## Use named value fields for constants instead of hard-coding

### Problem

Your contracts, resources, and scripts all have to refer to the same value.
A number, a string, a storage path, etc.
Entering these values manually in transactions and scripts is a potential source of error.
See [Wikipedia's page on magic numbers](https://en.wikipedia.org/wiki/Magic_number_(programming))

### Solution

Add a public (`pub`), constant (`let`) field, e.g. a `Path` , to the contract responsible for the value,
and set it in the contract's initializer.
Refer to that value via this public field rather than specifying it manually.

Example Snippet:

```cadence

// BAD Practice: Do not hard code storage paths
pub contract NamedFields {
    pub resource Test {}

    init() {
        // BAD: Hard-coded storage path
        self.account.save(<-create Test(), to: /storage/testStorage)
    }
}

// GOOD practice: Instead, use a field
//
pub contract NamedFields {
    pub resource Test {}

    // GOOD: field storage path
    pub let TestStoragePath: StoragePath

    init() {
        // assign and access the field here and in transactions
        self.TestStoragePath = /storage/testStorage
        self.account.save(<-create Test(), to: self.TestStoragePath)
    }
}

```

[Example Code](https://github.com/onflow/flow-core-contracts/blob/master/contracts/LockedTokens.cdc#L718)

## Script-Accessible public field/function

Data availability is important in a blockchain environment.
It is useful to publicize information about your smart contract and the assets it controls
so other smart contracts and apps can easily query it.

### Problem

Your contract, resource or struct has a field or resource that will need to be read and used on or off-chain, often in bulk.

### Solution

Make sure that the field can be accessed from a script (using a `PublicAccount`)
rather than requiring a transaction (using an `AuthAccount`).
This saves the time and fees required to read a property using a transaction.
Making the field or function `pub` and exposing it via a `/public/` capability will allow this.

Be careful not to expose any data or functionality that should be kept private when doing so.

Example:

```cadence
// BAD: Field is private, so it cannot be read by the public
access(self) let totalSupply: UFix64

// GOOD: Field is public, so it can be read and used by anyone
pub let totalSupply: UFix64
```

## Script-Accessible report

### Problem

Your contract has a resource that you wish to access fields of.
Resources are often stored in private places and are hard to access.
Additionally, scripts cannot return resources to the external context,
so a struct must be used to hold the data.

### Solution

Return a reference to a resource if the data from a single resource is all that is needed.
Otherwise, declare a struct to hold the data that you wish to return from the script.
Write a function that fills out the fields of this struct with the data
from the resource that you wish to access.
Then call this on the resource that you wish to access the fields of in a script,
and return the struct from the script.

See [Script-Accessible public field/function](#script-accessible-public-fieldfunction), above, for how best to expose this capability.

### Example Code

```cadence
pub contract AContract {
    pub let BResourceStoragePath: StoragePath
    pub let BResourcePublicPath: PublicPath

    init() {
        self.BResourceStoragePath = /storage/BResource
        self.BResourcePublicPath = /public/BResource
    }

    // Resource definition
    pub resource BResource {
        pub var c: UInt64
        pub var d: String


        // Generate a struct with the same fields
        // to return when a script wants to see the fields of the resource
        // without having to return the actual resource
        pub fun generateReport(): BReportStruct {
            return BReportStruct(c: self.c, d: self.d)
        }

        init(c: UInt64, d: String) {
            self.c = c
            self.d = d
        }
    }

    // Define a struct with the same fields as the resource
    pub struct BReportStruct {
        pub var c: UInt64
        pub var d: String

        init(c: UInt64, d: String) {
            self.c = c
            self.d = d
        }

    }
}
...
// Transaction
import AContract from 0xAContract

transaction {
        prepare(acct: AuthAccount) {
            //...
            acct.link<&AContract.BResource>(AContract.BResourcePublicPath, target: AContract.BResourceStoragePath)
        }
}
// Script
import AContract from 0xAContract

// Return the struct with a script
pub fun main(account: Address): AContract.BReportStruct {
    // borrow the resource
    let b = getAccount(account)
        .getCapability(AContract.BResourcePublicPath)
        .borrow<&AContract.BResource>()
    // return the struct
    return b.generateReport()
}
```

## Init Singleton

### Problem

An admin resource must be created and delivered to a specified account.
There should not be a function to do this, as that would allow anyone to create an admin resource.

### Solution

Create any one-off resources in the contract's `init()` function
and deliver them to an address or `AuthAccount` specified as an argument.

See how this is done in the LockedTokens contract init function:

[LockedTokens.cdc](https://github.com/onflow/flow-core-contracts/blob/master/contracts/LockedTokens.cdc#L718)

and in the transaction that is used to deploy it:

[admin_deploy_contract.cdc](https://github.com/onflow/flow-core-contracts/blob/master/transactions/lockedTokens/admin/admin_deploy_contract.cdc)


## Use descriptive names for fields, paths, functions and variables

### Problem

Smart contracts often are vitally important pieces of a project and often have many other
smart contracts and applications that rely on them.
Therefore, they need to be clearly written and easy to understand.

### Solution

All fields, functions, types, variables, etc., need to have names that clearly describe what they are used for.

`account` / `accounts` is better than `array` / `element`.

`providerAccount` / `tokenRecipientAccount` is better than `acct1` / `acct2`.

`/storage/bestPracticesDocsCollectionPath` is better than `/storage/collection`

### Example
```cadence
// BAD: Unclear naming
//
pub contract Tax {
    // Do not use abbreviations unless absolutely necessary
    pub var pcnt: UFix64

    // Not clear what the function is calculating or what the parameter should be
    pub fun calculate(num: UFix64): UFix64 {
        // What total is this referring to?
        let total = num + (num * self.pcnt)

        return total
    }
}

// GOOD: Clear naming
//
pub contract TaxUtilities {
    // Clearly states what the field is for
    pub var taxPercentage: UFix64

    // Clearly states that this function calculates the
    // total cost after tax
    pub fun calculateTotalCostPlusTax(preTaxCost: UFix64): UFix64 {
        let postTaxCost = preTaxCost + (preTaxCost * self.taxPercentage)

        return postTaxCost
    }
}
```

## Include concrete types in type constraints, especially "Any" types

### Problem

When specifying type constraints for capabilities or borrows, concrete types often do not get specified,
making it unclear if the developer actually intended it to be unspecified or not.
Paths also use a shared namespace between contracts, so an account may have stored a different object
in a path that you would expect to be used for something else.
Therefore, it is important to be explicit when getting objects or references to resources.


### Solution

A good example of when the code should specify the type being restricted is checking the FLOW balance:
The code must borrow `&FlowToken.Vault{FungibleToken.Balance}`, in order to ensure that it gets a FLOW token balance,
and not just `&{FungibleToken.Balance}`, any balance – the user could store another object
that conforms to the balance interface and return whatever value as the amount.

When the developer does not care what the concrete type is, they should explicitly indicate that
by using `&AnyResource{Receiver}` instead of `&{Receiver}`.
In the latter case, `AnyResource` is implicit, but not as clear as the former case.

## Plural names for arrays and maps are preferable

e.g. `accounts` rather than `account` for an array of accounts.

This signals that the field or variable is not scalar.
It also makes it easier to use the singular form for a variable name during iteration.

## Use transaction post-conditions when applicable

### Problem

Transactions can contain any amount of valid Cadence code and access many contracts and accounts.
The power of resources and capabilities means that there may be some behaviors of programs that are not expected.

### Solution

It is usually safe to include post-conditions in transactions to verify the intended outcome.

### Example

This could be used when purchasing an NFT to verify that the NFT was deposited in your account's collection.

```cadence
// Psuedo-code

transaction {

    pub let buyerCollectionRef: &NonFungibleToken.Collection

    prepare(acct: AuthAccount) {

        // Get tokens to buy and a collection to deposit the bought NFT to
        let temporaryVault <- vaultRef.withdraw(amount: 10.0)
        let self.buyerCollectionRef = acct.borrow(from: /storage/Collection)

        // purchase, supplying the buyers collection reference
        saleRef.purchase(tokenID: 1, recipient: self.buyerCollectionRef, buyTokens: <-temporaryVault)

    }
    post {
        // verify that the buyer now owns the NFT
        self.buyerCollectionRef.idExists(1) == true: "Bought NFT ID was not deposited into the buyers collection"
    }
}
```

## Avoid excessive load and save storage operations (prefer in-place mutations)

### Problem

When modifying data in account storage, `load()` and `save()` are costly operations.
This can quickly cause your transaction to reach the gas limit or slow down the network.

This also applies to contract objects and their fields (which are implicitly stored in storage, i.e. read from/written to),
or nested resources. Loading them from their fields just to modify them and save them back
is just as costly.

For example, a collection contains a dictionary of NFTs. There is no need to move the whole dictionary out of the field,
update the dictionary on the stack (e.g. adding or removing an NFT),
and then move the whole dictionary back to the field, it can be updated in-place.
The same goes for a more complex data structure like a dictionary of nested resources:
Each resource can be updated in-place by taking a reference to the nested object instead of loading and saving.

### Solution

For making modifications to values in storage or accessing stored objects,
`borrow()` should always be used to access them instead of `load` or `save` unless absolutely necessary.
`borrow()` returns a reference to the object at the storage path instead of having to load the entire object.
This reference can be assigned to or can be used to access fields or call methods on stored objects.

### Example

```cadence
// BAD: Loads and stores a resource to use it
//
transaction {

    prepare(acct: AuthAccount) {

        // Removes the vault from storage, a costly operation
        let vault <- acct.load<@ExampleToken.Vault>(from: /storage/exampleToken)

        // Withdraws tokens
        let burnVault <- vault.withdraw(amount: 10)

        destroy burnVault

        // Saves the used vault back to storage, another costly operation
        acct.save(to: /storage/exampleToken)

    }
}

// GOOD: Uses borrow instead to avoid costly operations
//
transaction {

    prepare(acct: AuthAccount) {

        // Borrows a reference to the stored vault, much less costly operation
        let vault <- acct.borrow<&ExampleToken.Vault>(from: /storage/exampleToken)

        let burnVault <- vault.withdraw(amount: 10)

        destroy burnVault

        // No `save` required because we only used a reference
    }
}
```

# Capabilities

## Capability Bootstrapping

### Problem

An account must be given a [capability](./language/capabilities.md)
to a resource or contract in another account. To create, i.e. link the capability,
the transaction must be signed by a key which has access to the target account.

To transfer / deliver the capability to the other account, the transaction also needs write access to that one.
It is not as easy to produce a single transaction which is authorized by two accounts
as it is to produce a typical transaction which is authorized by one account.

This prevents a single transaction from fetching the capability
from one account and delivering it to the other.

### Solution

The solution to the bootstrapping problem in Cadence is provided by the [Inbox API](./language/accounts.mdx#account-inbox)

Account A (which we will call the provider) creates the capability they wish to send to B (which we will call the recipient),
and stores this capability on their account in a place where the recipient can access it using the `Inbox.publish` function on their account.
They choose a name for the capability that the recipient can later use to identify it, and specify the recipient's address when calling `publish`.
This call to `publish` will emit an `InboxValuePublished` event that the recipient can listen for off-chain to know that the Capability is ready for them to claim.

The recipient then later can use the `Inbox.claim` function to securely grab the capability from the provider's account.
They must provide the name and type with which the capability was published, as well as the address of the provider's account
(all of this information is available in the `InboxValuePublished` event emitted on `publish`).
This will remove the capability from the provider's account and emit an `InboxValueClaimed` event that the provider can listen for off-chain.

One important caveat to this is that the published capability is stored on the provider's account until the recipient claims it,
so the provider can also use the `Inbox.unpublish` function to remove the capability from their account if they no longer wish to pay for storage for it.
This also requires the name and type which the capability was published,
and emits an `InboxValueUnpublished` event that the recipient can listen for off-chain.

It is also important to note that the recipient becomes the owner of the capability object once they have claimed it,
and can thus store it or copy it anywhere they have access to.
This means providers should only publish capabilities to recipients they trust to use them properly,
or limit the type with which the capability is restricted in order to only give recipients access to the functionality
that the provider is willing to allow them to copy.

## Capability Revocation

### Problem

A capability provided by one account to a second account must able to be revoked
by the first account without the co-operation of the second.

See the [Capability Controller FLIP](https://github.com/onflow/flow/pull/798) for a proposal to improve this in the future.

### Solution

The first account should create the capability as a link to a capability in `/private/`,
which then links to a resource in `/storage/`. That second-order link is then handed
to the second account as the capability for them to use.

**Account 1:** `/private/capability` → `/storage/resource`

`/private/revokableLink` -> `/private/capability`

**Account 2:** `/storage/capability -> (Capability(→Account 1: /private/revokableLink))`

If the first account wants to revoke access to the resource in storage,
they should delete the `/private/` link that the second account's capability refers to.
Capabilities use paths rather than resource identifiers, so this will break the capability.

The first account should be careful not to create another link at the same location
in its private storage once the capability has been revoked,
otherwise this will restore the second account's capability.


## Check for existing links before creating new ones

When linking a capability, the link might be already present.
In that case, Cadence will not panic with a runtime error but the link function will return nil.
The documentation states that: The link function does not check if the target path is valid/exists
at the time the capability is created and does not check if the target value conforms to the given type.
In that sense, it is a good practice to check if the link does already exist with `AuthAccount.getLinkTarget`
before creating it with `AuthAccount.link()`.
`AuthAccount.getLinkTarget` will return nil if the link does not exist.

### Example

```cadence
transaction {
    prepare(signer: AuthAccount) {
        // Create a public capability to the Vault that only exposes
        // the deposit function through the Receiver interface
        //
        // Check to see if there is a link already and unlink it if there is

        if signer.getLinkTarget(/public/exampleTokenReceiver) != nil {
            signer.unlink(/public/exampleTokenReceiver)
        }

        signer.link<&ExampleToken.Vault{FungibleToken.Receiver}>(
            /public/exampleTokenReceiver,
            target: /storage/exampleTokenVault
        )
    }
}
```
---
title: Introduction to Cadence
sidebar_position: 1
sidebar_label: Introduction
---
 
In a blockchain environment like Flow, programs that are stored on-chain in accounts are commonly referred to as smart contracts.
A smart contract is a program that verifies and executes the performance of a contract without the need for a trusted third party.
Programs that run on blockchains are commonly referred to as smart contracts because they mediate important functionality (such as currency)
without having to rely on a central authority (like a bank).

## A New Programming Language

---

Cadence is a resource-oriented programming language that introduces new features to smart contract programming
that help developers ensure that their code is safe, secure, clear, and approachable. Some of these features are:

- Type safety and a strong static type system.
- Resource-oriented programming, a new paradigm that pairs linear types with object capabilities
to create a secure and declarative model for digital ownership by ensuring that resources (which are used to represent scarce digital assets)
can only exist in one location at a time, cannot be copied, and cannot be accidentally lost or deleted.
- Built-in pre-conditions and post-conditions for functions and transactions.
- The utilization of capability-based security, which enforces that access to objects
is restricted to only the owner of the object and those who have a valid reference to it.
This is Cadence's main form of access control.

Cadence’s syntax is inspired by popular modern general-purpose programming languages
like [Swift](https://developer.apple.com/swift/), [Kotlin](https://kotlinlang.org/), and [Rust](https://www.rust-lang.org/).
Its use of resource types maps well to that of [Move](https://medium.com/coinmonks/overview-of-move-programming-language-a860ffd8f55d),
the programming language being developed by the Diem team.

## Cadence's Programming Language Pillars

---

Cadence, a new high-level programming language, observes the following requirements:

- **Safety and Security:** Safety is the underlying reliability of any smart contract (i.e., it’s bug-free and performs its function).
Security is the prevention of attacks on the network or smart contracts (i.e., unauthorized actions by malicious actors).
Safety and security are critical in smart contracts because of the immutable nature of blockchains,
and because they often deal with high-value assets. While auditing and reviewing code will be a crucial part of smart contract development,
Cadence maximizes efficiency while maintaining the highest levels of safety and security at its foundation.
It accomplishes this via a strong static type system, design by contract, and ownership primitives inspired by linear types (which are useful when dealing with assets).
- **Clarity:** Code needs to be easy to read, and its meaning should be as unambiguous as possible.
It should also be suited for verification so that tooling can help with ensuring safety and security guarantees.
These guarantees can be achieved by making the code declarative and allowing the developer to express their intentions directly.
We make those intentions explicit by design, which, along with readability, make auditing and reviewing more efficient, at a small cost to verbosity.
- **Approachability:** Writing code and creating programs should be as approachable as possible.
Incorporating features from languages like Swift and Rust, developers should find Cadence’s syntax and semantics familiar.
Practical tooling, documentation, and examples enable developers to start creating programs quickly and effectively.
- **Developer Experience:** The developer should be supported throughout the entire development lifecycle, from initial application logic to on-chain bugfixes.
- **Intuiting Ownership with Resources:** Resources are a composite data type, similar to a struct, that expresses direct ownership of assets.
Cadence’s strong static type system ensures that resources can only exist in one location at a time and cannot be copied or lost because of a coding mistake.
Most smart contract languages currently use a ledger-style approach to record ownership,
where an asset like a fungible token is stored in the smart contract as an entry in a central ledger.
Cadence’s resources directly tie an asset’s ownership to the account that owns it by saving the resource in the account’s storage.
As a result, ownership isn’t centralized in a smart contract’s storage. Each account owns its assets,
and the assets can be transferred freely between accounts without the need for arbitration by a central smart contract.

## Addressing Challenges with Existing Languages

---

Other languages pioneered smart contract development, but they lack in areas that affect the long-term viability of next-generation applications.

### Safety

Safety is the reliability of a smart contract to perform its function as intended.
It is heavily influenced by the unchangeable-once-deployed nature of smart contracts:
Developers must take certain precautions in order to avoid introducing any potentially catastrophic vulnerabilities
prior to publishing a smart contract on the blockchain.
It is standard across many blockchains that modifying or updating a smart contract, even to fix a vulnerability, is not allowed.
Thus, any bugs that are present in the smart contract will exist forever.

For example, in 2016, an overlooked vulnerability in an Ethereum DAO smart contract (Decentralized Autonomous Organization)
saw millions of dollars siphoned from a smart contract,
eventually leading to a fork in Ethereum and two separate active blockchains (Ethereum and Ethereum Classic).

Bug fixes are only possible if a smart contract is designed to support changes,
a feature that introduces complexity and security issues.
Lengthy auditing and review processes can ensure a bug-free smart contract.
Still, they add substantial time to the already time-consuming task of getting the smart contract’s core logic working correctly.

Overlooked mistakes cause the most damaging scenarios.
It is easy to lose or duplicate monetary value or assets in existing languages because they don’t check relevant invariants
or make it harder to express them.
For example, a plain number represents a transferred amount that can be accidentally (or maliciously) multiplied or ignored.

Some languages also express behaviors that developers tend to forget about.
For example, a fixed-range type might express monetary value, without considerations for a potential overflow or underflow.
In Solidity, Ethereum's smart contract language, an overflow causes the value to wrap around, as shown [here](https://ethfiddle.com/CAp-kQrDUP).
Solidity also allows contracts to declare variables without initializing them.
If the developer forgets to add an initialization somewhere,
and then tries to read the variable somewhere else in the code expecting it to be a specific value, issues will occur.

Cadence is type safe and has a strong static type system,
which prevents important classes of erroneous or undesirable program behavior at compile-time (i.e., before the program is run on-chain).
Types are checked statically and are not implicitly converted. Cadence also improves the safety of programs by preventing arithmetic underflow and overflow,
introduces optionals to make nil-cases explicit, and always requires variables to be initialized.
This helps ensure the behavior of these smart contracts is apparent and not dependent on context.

### Security

Security, in combination with safety, ensures the successful execution of a smart contract over time
by preventing unsanctioned access and guaranteeing that only authorized actions can be performed in the protocol.
In some languages, functions are public by default, creating vulnerabilities that allow malicious users to find attack vectors.
Cadence utilizes capability-based security, which allows the type system to enforce access control based on rules that users and developers have control over.

Security is a consideration when interacting with other smart contracts. Any external call potentially allows malicious code to be executed.
For example, in Solidity, when the called function signature does not match any of the available ones, it triggers Solidity’s fallback functions.
These functions can be used in malicious ways. Language features such as multiple inheritances and overloading or dispatch can also make it difficult
to determine which code is invoked.

In Cadence, the safety and security of programs are enhanced by **Design By Contract** and **Ownership Primitives.**
Design by contract allows developers to state pre-conditions and post-conditions for functions and interfaces in a declarative manner
so that callers can be certain about the behavior of called code. Ownership primitives are inspired by linear types and increase safety when working with assets.
They ensure that valuable assets are, for example, not accidentally or maliciously lost or duplicated.

### Clarity and Approachability

Implicitness, context-dependability, and expressiveness are language-based challenges that developers often encounter.
They affect the clarity (i.e. the readability of code and the ability to determine its intended function)
and the approachability (i.e. the ability to interpret or write code) of the language and the programs built using it.
For example, in Solidity, storage must be implemented in a low-level key-value manner, which obfuscates the developer’s intentions.
Syntax confusion is another example, with “=+” being legal syntax leading to an assignment instead of a probably-intended increment.
Solidity also has features with uncommon behaviors that can lead to unintended results.
[Multiple inheritance may lead to unexpected behaviours in the program](https://medium.com/consensys-diligence/a-case-against-inheritance-in-smart-contracts-d7f2c738f78e),
and testing and auditing the code is unlikely to identify this issue.

The Ethereum blockchain’s code immutability showcases the need for considerations around extensibility and mechanisms that allow ad-hoc fixes.
Developers using custom-made approaches such as the 'data separation' approach to upgradability
may run into problems with the complexity of data structures,
while developers using ‘delegatecall-based proxies` may run into problems with the consistency of memory layouts.
Either way, these challenges compromise approachability and overall extensibility.
Cadence has [contract upgradability built in by default](./language/contract-updatability.md),
and contracts can be made immutable by removing all keys from an account.

Cadence improves the clarity and extensibility of programs by utilizing interfaces to allow extensibility, code reuse, and interoperability between contracts.
Cadence modules also have configurable and transparent upgradeability built-in to enable projects to test and iterate before making their code immutable.

Cadence allows the use of argument labels to describe the meaning of function arguments.
It also provides a rich standard library with useful data structures (e.g., dictionaries, sets) and data types for common use cases,
like fixed-point arithmetic, which helps when working with currencies.

## Intuiting Ownership with Resources

Most smart contract languages currently use a ledger-style approach to record ownership,
where an asset is stored in the smart contract as an entry in a central ledger, and this ledger is the source of truth around asset ownership.
There are many disadvantages to this design, especially when it comes to tracking the ownership of multiple assets belonging to a single account.
To find out all of the assets that an account owns, you would have to enumerate all the possible smart contracts that could potentially include this account
and search to see if the account owns these assets.

In a resource-oriented language like Cadence, resources directly tie an asset to the account that owns it
by saving the resource in the account’s storage. As a result, ownership isn’t centralized in a single, central smart contract’s storage.
Instead, each account owns and stores its own assets, and the assets can be transferred freely between accounts without the need for arbitration by a central smart contract.

Resources are inspired by linear types and increase safety when working with assets, which often have real, intrinsic value.
Resources, as enforced by Cadence’s type system, ensure that assets are correctly manipulated and not abused.

- Every resource has exactly one owner. If a resource is used as a function parameter, an initial value for a variable, or something similar, the object is not copied.
Instead, it is moved to the new location, and the old location is immediately invalidated.
- The language will report an error if ownership of a resource was not properly transferred, i.e.,
when the program attempts to introduce multiple owners for the resource or the resource ends up in a state where it does not have an owner.
For example, a resource can only be assigned to exactly one variable and cannot be passed to functions multiple times.
- Resources cannot go out of scope. If a function or transaction removes a resource from an account’s storage,
it either needs to end the transaction in an account's storage, or it needs to be explicitly and safely deleted. There is no “garbage collection” for resources.

The special status of Resource objects must be enforced by the runtime; if they were just a compiler abstraction it would be easy for malicious code to break the value guarantees.

Resources change how assets are used in a programming environment to better resemble assets in the real world.
Users store their currencies and assets in their own account, in their own wallet storage, and they can do with them as they wish.
Users can define custom logic and structures for resources that give them flexibility with how they are stored.
Additionally, because everyone stores their own assets, the calculation and charging of state rent is fair and balanced across all users in the network.

## An Interpreted Language

---

Currently, Cadence is an interpreted language, as opposed to a compiled language. This means that there is no Cadence Assembly, bytecode, compiler, or Cadence VM.

The structure of the language lends itself well to compilation (for example, static typing),
but using an interpreter for the first version allows us to refine the language features more quickly as we define them.

## Getting Started with Cadence

---

Now that you've learned about the goals and design of Cadence and Flow, you're ready to get started with the Flow emulator and tools!
Go to the [Getting Started](./tutorial/01-first-steps.md) page to work through language fundamentals and tutorials.
---
title: JSON-Cadence Data Interchange Format
sidebar_label: JSON-Cadence format
---

> Version 0.3.1

JSON-Cadence is a data interchange format used to represent Cadence values as language-independent JSON objects.

This format includes less type information than a complete [ABI](https://en.wikipedia.org/wiki/Application_binary_interface), and instead promotes the following tenets:

- **Human-readability** - JSON-Cadence is easy to read and comprehend, which speeds up development and debugging.
- **Compatibility** - JSON is a common format with built-in support in most high-level programming languages, making it easy to parse on a variety of platforms.
- **Portability** - JSON-Cadence is self-describing and thus can be transported and decoded without accompanying type definitions (i.e. an ABI).

# Values

---

## Void

```json
{
  "type": "Void"
}
```

### Example

```json
{
  "type": "Void"
}
```

---

## Optional

```json
{
  "type": "Optional",
  "value": null | <value>
}
```

### Example

```json
// Non-nil

{
  "type": "Optional",
  "value": {
    "type": "UInt8",
    "value": "123"
  }
}

// Nil

{
  "type": "Optional",
  "value": null
}
```

---

## Bool

```json
{
  "type": "Bool",
  "value": true | false
}
```

### Example

```json
{
  "type": "Bool",
  "value": true
}
```

---

## String

```json
{
  "type": "String",
  "value": "..."
}

```

### Example

```json
{
  "type": "String",
  "value": "Hello, world!"
}
```

---

## Address

```json
{
  "type": "Address",
  "value": "0x0" // as hex-encoded string with 0x prefix
}
```

### Example

```json
{
  "type": "Address",
  "value": "0x1234"
}
```

---

## Integers

`[U]Int`, `[U]Int8`, `[U]Int16`, `[U]Int32`,`[U]Int64`,`[U]Int128`, `[U]Int256`,  `Word8`, `Word16`, `Word32`, or `Word64`

Although JSON supports integer literals up to 64 bits, all integer types are encoded as strings for consistency.

While the static type is not strictly required for decoding, it is provided to inform client of potential range.

```json
{
  "type": "<type>",
  "value": "<decimal string representation of integer>"
}
```

### Example

```json
{
  "type": "UInt8",
  "value": "123"
}
```

---

## Fixed Point Numbers

`[U]Fix64`

Although fixed point numbers are implemented as integers, JSON-Cadence uses a decimal string representation for readability.

```json
{
    "type": "[U]Fix64",
    "value": "<integer>.<fractional>"
}
```

### Example

```json
{
    "type": "Fix64",
    "value": "12.3"
}
```

---

## Array

```json
{
  "type": "Array",
  "value": [
    <value at index 0>,
    <value at index 1>
    // ...
  ]
}
```

### Example

```json
{
  "type": "Array",
  "value": [
    {
      "type": "Int16",
      "value": "123"
    },
    {
      "type": "String",
      "value": "test"
    },
    {
      "type": "Bool",
      "value": true
    }
  ]
}
```

---

## Dictionary

Dictionaries are encoded as a list of key-value pairs to preserve the deterministic ordering implemented by Cadence.

```json
{
  "type": "Dictionary",
  "value": [
    {
      "key": "<key>",
      "value": <value>
    },
    ...
  ]
}
```

### Example

```json
{
  "type": "Dictionary",
  "value": [
    {
      "key": {
        "type": "UInt8",
        "value": "123"
      },
      "value": {
        "type": "String",
        "value": "test"
      }
    }
  ],
  // ...
}
```

---

## Composites (Struct, Resource, Event, Contract, Enum)

Composite fields are encoded as a list of name-value pairs in the order in which they appear in the composite type declaration.

```json
{
  "type": "Struct" | "Resource" | "Event" | "Contract" | "Enum",
  "value": {
    "id": "<fully qualified type identifier>",
    "fields": [
      {
        "name": "<field name>",
        "value": <field value>
      },
      // ...
    ]
  }
}
```

### Example

```json
{
  "type": "Resource",
  "value": {
    "id": "0x3.GreatContract.GreatNFT",
    "fields": [
      {
        "name": "power",
        "value": {"type": "Int", "value": "1"}
      }
    ]
  }
}
```

---

## Path

```json
{
  "type": "Path",
  "value": {
    "domain": "storage" | "private" | "public",
    "identifier": "..."
  }
}
```

### Example

```json
{
  "type": "Path",
  "value": {
    "domain": "storage",
    "identifier": "flowTokenVault"
  }
}
```

---

## Type Value

```json
{
  "type": "Type",
  "value": {
    "staticType": <type>
  }
}
```

### Example

```json
{
  "type": "Type",
  "value": {
    "staticType": {
      "kind": "Int",
    }
  }
}
```

---

## Capability

```json
{
  "type": "Capability",
  "value": {
    "path": <path>,
    "address": "0x0",  // as hex-encoded string with 0x prefix
    "borrowType": <type>,
  }
}
```

### Example

```json
{
  "type": "Capability",
  "value": {
    "path": {
      "type": "Path",
      "value": {
        "domain": "public",
        "identifier": "someInteger"
      }
    },
    "address": "0x1",
    "borrowType": {
      "kind": "Int"
    }
  }
}
```

---

## Functions

```json
{
  "type": "Function",
  "value": {
    "functionType": <type>
  }
}
```

Function values can only be exported, they cannot be imported.

### Example

```json
{
  "type": "Function",
  "value": {
    "functionType": {
      "kind": "Function",
      "typeID": "(():Void)",
      "parameters": [],
      "return": {
        "kind": "Void"
      }
    }
  }
}
```

---

# Types

## Simple Types

These are basic types like `Int`, `String`, or `StoragePath`.

```json
{
  "kind": "Any" | "AnyStruct" | "AnyResource" | "AnyStructAttachment" | "AnyResourceAttachment" | "Type" |
    "Void" | "Never" | "Bool" | "String" | "Character" |
    "Bytes" | "Address" | "Number" | "SignedNumber" |
    "Integer" | "SignedInteger" | "FixedPoint" |
    "SignedFixedPoint" | "Int" | "Int8" | "Int16" |
    "Int32" | "Int64" | "Int128" | "Int256" | "UInt" |
    "UInt8" | "UInt16" | "UInt32" | "UInt64" | "UInt128" |
    "UInt256" | "Word8" | "Word16" | "Word32" | "Word64" |
    "Fix64" | "UFix64" | "Path" | "CapabilityPath" | "StoragePath" |
    "PublicPath" | "PrivatePath" | "AuthAccount" | "PublicAccount" |
    "AuthAccount.Keys" | "PublicAccount.Keys" | "AuthAccount.Contracts" |
    "PublicAccount.Contracts" | "DeployedContract" | "AccountKey" | "Block"
}
```

### Example

```json
{
  "kind": "UInt8"
}
```

---

## Optional Types

```json
{
  "kind": "Optional",
  "type": <type>
}
```

### Example

```json
{
  "kind": "Optional",
  "type": {
    "kind": "String"
  }
}
```

---

## Variable Sized Array Types

```json
{
  "kind": "VariableSizedArray",
  "type": <type>
}
```

### Example

```json
{
  "kind": "VariableSizedArray",
  "type": {
    "kind": "String"
  }
}
```

---

## Constant Sized Array Types

```json
{
  "kind": "ConstantSizedArray",
  "type": <type>,
  "size": <length of array>,
}
```

### Example

```json
{
  "kind": "ConstantSizedArray",
  "type": {
    "kind": "String"
  },
  "size":3
}
```

---

## Dictionary Types

```json
{
  "kind": "Dictionary",
  "key": <type>,
  "value": <type>
}
```

### Example

```json
{
  "kind": "Dictionary",
  "key": {
    "kind": "String"
  },
  "value": {
    "kind": "UInt16"
  },
}
```

---

## Composite Types

```json
{
  "kind": "Struct" | "Resource" | "Event" | "Contract" | "StructInterface" | "ResourceInterface" | "ContractInterface",
  "type": "", // this field exists only to keep parity with the enum structure below; the value must be the empty string
  "typeID": "<fully qualified type ID>",
  "initializers": [
    <initializer at index 0>,
    <initializer at index 1>
    // ...
  ],
  "fields": [
    <field at index 0>,
    <field at index 1>
    // ...
  ],
}
```

### Example

```json
{
  "kind": "Resource",
  "type": "",
  "typeID": "0x3.GreatContract.GreatNFT",
  "initializers":[
    [
      {
        "label": "foo",
        "id": "bar",
        "type": {
          "kind": "String"
        }
      }
    ]
  ],
  "fields": [
    {
      "id": "foo",
      "type": {
        "kind": "String"
      }
    }
  ]
}
```

---

## Field Types

```json
{
  "id": "<name of field>",
  "type": <type>
}
```

### Example

```json
{
  "id": "foo",
  "type": {
    "kind": "String"
  }
}
```

---

## Parameter Types

```json
{
  "label": "<label>",
  "id": "<identifier>",
  "type": <type>
}
```

### Example

```json
{
  "label": "foo",
  "id": "bar",
  "type": {
    "kind": "String"
  }
}
```

---

## Initializer Types

Initializer types are encoded a list of parameters to the initializer.

```json
[
  <parameter at index 0>,
  <parameter at index 1>,
  // ...
]
```

### Example

```json
[
  {
    "label": "foo",
    "id": "bar",
    "type": {
      "kind": "String"
    }
  }
]
```

---

## Function Types

```json
{
  "kind": "Function",
  "typeID": "<function name>",
  "parameters": [
    <parameter at index 0>,
    <parameter at index 1>,
    // ...
  ],
  "return": <type>
}
```

### Example

```json
{
  "kind": "Function",
  "typeID": "foo",
  "parameters": [
    {
      "label": "foo",
      "id": "bar",
      "type": {
        "kind": "String"
      }
    }
  ],
  "return": {
    "kind": "String"
  }
}
```

---

## Reference Types

```json
{
  "kind": "Reference",
  "authorized": true | false,
  "type": <type>
}
```

### Example

```json
{
  "kind": "Reference",
  "authorized": true,
  "type": {
    "kind": "String"
  }
}
```

---

## Restricted Types

```json
{
  "kind": "Restriction",
  "typeID": "<fully qualified type ID>",
  "type": <type>,
  "restrictions": [
    <type at index 0>,
    <type at index 1>,
    //...
  ]
}
```

### Example

```json
{
  "kind": "Restriction",
  "typeID": "0x3.GreatContract.GreatNFT",
  "type": {
    "kind": "AnyResource",
  },
  "restrictions": [
    {
        "kind": "ResourceInterface",
        "typeID": "0x1.FungibleToken.Receiver",
        "fields": [
            {
                "id": "uuid",
                "type": {
                    "kind": "UInt64"
                }
            }
        ],
        "initializers": [],
        "type": ""
    }
  ]
}
```

---

## Capability Types

```json
{
  "kind": "Capability",
  "type": <type>
}
```

### Example

```json
{
  "kind": "Capability",
  "type": {
    "kind": "Reference",
    "authorized": true,
    "type": {
      "kind": "String"
    }
  }
}
```

---

## Enum Types

```json
{
  "kind": "Enum",
  "type": <type>,
  "typeID": "<fully qualified type ID>",
  "initializers":[],
  "fields": [
    {
      "id": "rawValue",
      "type": <type>
    }
  ]
}
```

### Example

```json
{
  "kind": "Enum",
  "type": {
    "kind": "String"
  },
  "typeID": "0x3.GreatContract.GreatEnum",
  "initializers":[],
  "fields": [
    {
      "id": "rawValue",
      "type": {
        "kind": "String"
      }
    }
  ]
}
```

## Repeated Types

When a composite type appears more than once within the same JSON type encoding, either because it is
recursive or because it is repeated (e.g. in a composite field), the composite is instead
represented by its type ID.

### Example

```json
{
  "type":"Type",
  "value": {
    "staticType": {
      "kind":"Resource",
      "typeID":"0x3.GreatContract.NFT",
      "fields":[
        {"id":"foo",
        "type": {
          "kind":"Optional",
          "type":"0x3.GreatContract.NFT" // recursive NFT resource type is instead encoded as an ID
          }
        }
      ],
      "initializers":[],
      "type":""
    }
  }
}
```
---
title: Measuring Time In Cadence
sidebar_label: Measuring Time
---

## Accessing Time From Cadence

Both the [block height and the block timestamp](./language/environment-information.md#block-information) are accessible from within Cadence code.

This means that they can be used to calculate dates and durations by smart contracts on Flow
that need to lock resources until a particular point in the future, calculate values between a range of dates,
or otherwise deal with the passage of time.

There are two popular strategies that are used to measure time on blockchains:

1. Use the timestamp, and optionally check that the average duration of the last n blocks
is close enough to the block target duration to make an attack unlikely.
2. Use the block height directly. Block height can be treated intuitively
(a hundred blocks, a thousand blocks) or can be related to estimated timestamps
and thereby to time off-chain by the methods described in this article.

## Time On The Flow Blockchain

> Flow targets 1 second block times but the protocol is still early in its development
and further optimizations are needed to achieve that.
As of Feb 2021, the rate of block finalization on Mainnet is more than 0.5 blocks/s; with a standard deviation of ±0.1 blocks/s.
Hence, a new block is finalized on average every 2 seconds.
Note that block height only has a loose correlation with time,
as [the block rate naturally fluctuates](https://developers.flow.com/references/run-and-secure/nodes/faq/operators.mdx#does-the-blockheight-go-up-1-every-second).

In addition to the natural variation described above,
there are several theoretical block production attacks that could skew this relationship even further.
These attacks are unlikely on Flow in the absence of byzantine nodes.
The timestamp cannot be earlier than the timestamp of the previous block,
and cannot be too far into the future ([currently ten seconds](https://github.com/onflow/flow-go/blob/master/module/builder/consensus/builder.go#L60))
- proposed blocks that fail to satisfy these conditions will be rejected by Flow's consensus algorithm.
But the mere possibility of these attacks places an additional limit on the confidence
with which we can use block heights or block timestamps to determine off-chain time from protocol-level data on-chain.

The block timestamp is not the only way to identify a block within the flow of off-chain time.
Each block is numbered successively by its "height", block 70000 is followed by block 70001, 70002,
and so on. Blocks with heights out of sequence are rejected by Flow's consensus algorithm.
In theory the timestamp on a block should be roughly equivalent to the timestamp on the Flow genesis block,
plus the block height multiplied by the target block rate.
But as we have seen both the target and the on-chain average rate of block production may vary over time.
This makes such calculations more difficult.

### Using The Timestamp

Given that [Flow consensus will reject new blocks with a timestamp more than ten seconds into the future from the previous block](https://github.com/onflow/flow-go/blob/1e8a2256171d5fd576f442d0c335c9bcc06e1e09/module/builder/consensus/builder.go#L525-L536),
as long as you do not require an accuracy of less than ten seconds
it is probably safe to use the block timestamp for events lasting a few days - in the absence of a change in block production rate targets.
Or, more intuitively, your timestamp is highly likely to be the correct hour,
very likely to be the correct minute, and may well be within ten seconds of the correct second.
Which of these scales is tolerable for your use case depends on how long the events you need to represent will take.
In an auction lasting several days, you are probably safe with any scale above ten seconds.

```cadence
// To get the timestamp of the block that the code is being executed in
getCurrentBlock().timestamp

// To get the timestamp of a known previous block, if available
getBlock(at: 70001)?.timestamp
```

### Using The Block Height

In theory block numbers are more reliable than timestamps,
as the block height is incremented for each block in a fork.
But in practice we must still relate block numbers to off-chain time values,
and to do this requires that we assume that the average block time will hold.
This can vary due to factors other than attacks.
Given that block time targets will vary as Flow development continues,
this will affect any calculations you may make in order to relate block numbers to calendar time.

```cadence
// To get the block number of the block that the code is being executed in
getCurrentBlock().height

// To get the block number of a known previous block, if available
getBlock(at: 70001)?.height
```

## Recommendations

If your contract code can tolerate the limitations described above, use block timestamps.
If not, you may need to consider more exotic solutions (time oracles, etc.).

Whichever method you use, be careful not to hardcode any assumptions
about block rates production rates into your code, on-chain or off,
in a way that cannot be updated later.

On-chain auctions and similar mechanisms should always have an extension mechanism.
If someone bids at the last moment (which is easier to do with a block production attack),
the end time for the auction extends (if necessary) to N minutes past the last bid.
(10 minutes, 30 minutes, an hour). As N increases, this becomes more secure:
N=5 should be more than enough. with the current parameters of the Flow blockchain.
---
title: Flow Smart Contract Project Development Standards
sidebar_label: Development Standards
sidebar_position: 7
description: "Learn how to effectively organize and manage a Cadence project"
---

# Smart Contract Project Development Standards

## Context

Smart Contracts are the bedrock piece of security for many important parts
of the Flow blockchain, as well as for any project that is deployed to a blockchain.

They are also the most visible technical parts of any project,
since users will be querying them for data, building other smart contracts that interact with them,
and using them as learning materials and templates for future projects.
Furthermore, when deployed they are publicly available code on the blockchain
and often also in public Github repos.

Therefore, the process around designing, building, testing, documenting,
and managing these projects needs to reflect the critical importance they hold in the ecosystem.

Every software project strikes a balance between effort spent on product/feature delivery
vs the many other demands of the software development lifecycle, whether testing, technical debt,
automation, refactoring, or documentation etc. Building in Web3 we face the same trade-offs,
but in a higher risk and consequence environment than what is typical for most software.
A mismanaged or untested smart contract may result in **significant** financial losses
as a result of vulnerabilities which were overlooked then exploited.
We highly recommend builders adopt these best practices to help mitigate these risks.

If they do so, they will be able to build better smart contracts, avoid potential bugs,
support user and third-party adoption of their projects, and increase their chances of success
by being a model for good software design. Additionally, the more projects that adopt
good software design and management standards normalizes this behavior,
encouraging other projects in the ecosystem to do the same which creates a healthier
and more vibrant community.

Ensuring appropriate levels of testing results in better smart contracts which have
pro-actively modeled threats and engineered against them. Ensuring appropriate levels 
of standards adoption ([FungibleToken](https://github.com/onflow/flow-ft),
[NFT Catalog](https://www.flow-nft-catalog.com/), [NFT StoreFront](https://github.com/onflow/nft-storefront), etc) by dapp 
builders amplifies the network effects for all in the ecosystem. NFTs in one dapp can be 
readily consumed by other dapps through on-chain events with no new integration 
required. With your help and participation we can further accelerate healthy and vibrant 
network effects across the Flow ecosystem!

Some of these suggestions might seem somewhat unnecessary,
but it is important to model what a project can do to manage its smart contracts the best
so that hopefully all of the other projects follow suit.

This also assumes standard software design best practices also apply.
Indeed, many of these suggestions are more general software design best practices,
but there may be others that are assumed but not included here.

### Implementing These Practices

This document serves as mostly an outline of best practices the projects should follow.
As with all best practices, teams will choose which applies to them and their work process,
however, we recommend that teams explicitly define a minimum acceptable set of standards
for themselves along with the mechanisms to ensure they are being observed.

Some teams may also have their own set of development standards that achieve a similar goal
to these. These recommendations are not meant to be the only paths to success,
so if a team disagrees with some of these and wants to do things their own way,
they are welcome to pursue that. This document just shows some generic suggestions
for teams who might not know how they want to manage their project.

## Design Process

Smart contracts usually manage a lot of value, have many users, and are difficult to upgrade
for a variety of reasons. Therefore, it is important to have a clearly defined design
process for the smart contracts before much code is written so that the team
can set themselves up for success.

Here are some recommendations for how projects can organize the foundations of their projects.

### Projects should ensure that there is strong technical leadership for their smart contracts

Developing a dapp requires a clear vision for the role of the smart contract and how it's integrated.
Security vulnerabilities may arise from bugs directly in smart contract code (and elsewhere in the system).
Asynchronous interaction vectors may lead to forms of malicious abuse,
DOS etc in a contract triggering explosive gas costs for the developer or other problems.

We recommend that engineers leading a project and deploying to mainnet have an understanding
of software and security engineering fundamentals and have been thorough
in their Cadence skills development. More in-depth resources for learning Cadence
are available [here](./index.md).

The technical leader should be someone who understands Cadence well and has written Cadence smart contracts
before. Production-level smart contracts are not the place for beginners to get their start.

It should be this person’s responsibility to lead design discussions
with product managers and the community, write most of the code and tests,
solicit reviews, make requested changes and make sure the project gets completed in a timely manner.

The leader should also understand how to sign transactions with the CLI
to deploy/upgrade smart contracts, run admin transactions, and troubleshoot problems, etc.
If something goes wrong in relation to the smart contract
that needs to be handled with a bespoke transaction, it is important that the owner
knows how to build and run transactions and scripts safely to address the issues
and/or upgrade the smart contracts.

The project should also have a clear plan of succession in case the original owner 
is not available or leaves the project. It is important that there are others who
can fill in who have a clear understanding of the code and requirements so they can give good feedback,
perform effective reviews, and make changes where needed.

### Projects should maintain a well-organized open source Repo for their smart contracts

As projects like NBA Topshot have shown, when a blockchain product becomes successful
others can and do to build on top of what you are doing.
Whether that is analytics, tools, or other value adds that could help grow your project ecosystem,
composability is key and that depends on open source development.
If there isn’t already an open source repo, builders should consider creating one.

Builders can start from the [the Flow open source template](https://github.com/onflow/open-source-template)
and make sure all of their repo is set up with some initial documentation for what the repo is for
before any code is written. External developers and users should have an easily accessible home page
to go to to understand any given project.

The repo should also have some sort of high-level design document that lays out
the intended design and architecture of the smart contract.
The project leads should determine what is best for them to include in the document,
but some useful things to include are basic user stories, architecture of the smart contracts,
and any questions that still need to be answered about it.
    - Where applicable, diagrams should be made describing state machines, user flows, etc.
    - This document should be shared in an issue in the open source repo
    where the contracts or features are being developed,
    then later moved to the README or another important docs page.

A high level design is a key opportunity to model threats
and understand the risks of the system. The process of collaborating
and reviewing designs together helps ensure that more edge-cases are captured and addressed.
It's also a lot less effort to iterate on a design than on hundreds of lines of Cadence.    

## Development Process Recommendations

### The Development process should be iterative, if possible

The project should develop an MVP first, get reviews, and test thoroughly,
then add additional features with tests. This ensures that the core features are designed
thoughtfully and makes the review process easier because they can focus on each feature
one at a time instead of being overwhelmed by a huge block of code.

### Comments and field/function descriptions are essential!

Our experience writing many Cadence smart contracts has taught us how important documentation 
is. It especially matters what is documented and for whom, and in that way we are no different from 
any software language. The Why is super important, if for example something - an event - that 
happens in one contract leads to outcomes in a different contract. The What helps give context, 
the reason for the code turning out the way it is. The How, you don't document - you've written 
the code. Comments should be directed to those who will follow after you in changing the code. 

Comments should be written at the same time (or even before) the code is written.
This helps the developer and reviewers understand the work-in-progress code better,
as well as the intentions of the design (for testing and reviewing).
Functions should be commented with a
    - Description
    - Parameter descriptions
    - Return value descriptions


Top Level comments and comments for types, fields, events,
and functions should use `///` (three slashes) to be recognised by the
[Cadence Documentation Generator](https://github.com/onflow/cadence-tools/tree/master/docgen).
Regular comments within functions should only use two slashes (`//`)

## Testing Recommendations

Summarized below is a list of testing related recommendations 
which are noteworthy to mention for a typical smart contract project.

Popular testing frameworks to use for cadence are listed here:
Javascript: [Flow JS Testing](https://developers.flow.com/tools/flow-js-testing/index.md)
Go: [Overflow](https://github.com/bjartek/overflow)
Cadence: [Cadence Testing Framework](https://github.com/onflow/cadence/blob/ac05b6a0d6005cde468573f0a7a2e3a67f49bd90/docs/testing-framework.mdx)
Tests written in Cadence!

The same person who writes the code should also write the tests.
They have the clearest understanding of the code paths and edge cases.


Tests should be **mandatory**, not optional, even if the contract is copied from somewhere else.
There should be thorough emulator unit tests in the public repo.
[See the flow fungible token repo](https://github.com/onflow/flow-ft/tree/master/lib/js/test)
for an example of unit tests in javascript.


Every time there is a new Cadence version or emulator version,
the dependencies of the repo should be updated to make sure the tests are all still passing.


Tests should avoid being monolithic;
Individual test cases should be set up for each part of the contract to test them in isolation. 
See the [`FlowEpoch` smart contract tests](https://github.com/onflow/flow-core-contracts/blob/master/lib/go/test/flow_epoch_test.go)
for examples written in Go where test cases are split
into separate blocks for different features.
There are some exceptions, like contracts that have to run through a state machine
to test different cases. Positive and negative cases need to be tested.

Integration tests should also be written to ensure that your app and/or backend can interact
properly with the smart contracts.

## Managing Project Keys and Deployments

Smart contract keys and deployments are very important and need to be treated as such.

### Private Keys should be stored securely

Private Keys for the contract and/or admin accounts should not be kept in plain text format anywhere.
Projects should determine a secure solution that works best for them to store their private keys.
We recommend storing them in a secure key store such as google KMS or something similar.

### Deployments to Testnet or Mainnet should be handled transparently

As projects become more successful, communities around them grow.
In a trustless ecosystem, that also means more of others building on your contracts.
Before deploying or upgrading a contract, it is important to maintain
clear community communications with sufficient notice, since changes will always bring added risk.
Giving community members time to review and address issues with upgrades
before they happen builds trust and confidence in projects.
Here are a few suggestions for how to manage a deployment or upgrade.

- Communicate to all stake-holders well in advance
    - Share the proposal with the community at least a week in advance (unless it is a critical bug fix)
        - Examples of places to share are your project's chat, forum, blog, email list, etc.
        - This will allow the community and other stakeholders to have plenty of time
        to view the upcoming changes and provide feedback if necessary.
    - Share the time of the deployment and the deployment transaction with branch/commit hash information to ensure the transaction itself is correct.
    - Coordinate deployment with stakeholders to make sure it is done correctly and on time.

## Responsibilities to the Community

Web3 brings tremendous possibilities for engineering applications with trustlessness
and composability in mind, with Cadence and Flow offering unique features to achieve this.
If every project treats their community and the Flow community with respect and care,
the things we can all build together will be very powerful.

### Projects should have thorough documentation

Encouraging adoption of project contracts to the broader ecosystem
raises the bar around code providing clear high-level descriptions,
with detailed and useful comments within contracts, transactions, and scripts.
The more that a project can be understood, that it adheres to standards,
and can be built upon with ease, the more likely others will build against it in turn. 

Each project should have a detailed README.md with these sections:
    - Explanation of the project itself with links to the app
    - Addresses on various networks
    - High-level technical description of the contracts with emphasis on important types and functionality
    - Architecture diagram (if applicable)
    - Include links to tutorials if they are external
    - Flow smart contract standards that a project implements

Additionally, each contract, transaction, and script should have high-level descriptions
at the top of their files. This way, anyone in the community can easily 
come in and understand what each one is doing without having to parse confusing code.

### Projects should engage with and respond to their own Community

Once a contract is deployed, the work doesn’t stop there.
Project communities require ongoing nurturing and support.
As the developer of a public project on a public blockchain,
the owners have an obligation to be helpful and responsive to the community
so that they can encourage composability and third party interactions.

- Keep issues open in the repo.
- The owner should turn on email notifications for new issue creation in the repo.
- Respond to issues quickly and clean up unimportant ones.
- Consider blog posts to share more details on technical aspects of the project and upcoming changes.

### Projects should contribute to the greater Flow and Cadence community

Flow has a vibrant and growing community of contributors around the world.
Through our mutual collaboration we've had numerous community Flow Improvement Proposals
([FLIP](https://github.com/onflow/flow/tree/master/flips)s) shipped.
If you have an interest in a particular improvement for Flow or Cadence,
we host open meetings which you are welcome to join (announced on discord)
and can participate anytime on any of the FLIPs
[already proposed](https://github.com/onflow/flow/pulls?q=is%3Aopen+is%3Apr+label%3AFLIP).

Responsible project maintainers should contribute to discussions
about important proposals (new cadence features, standard smart contracts, metadata, etc)
and generally be aware about evolving best practices and anti-pattern understandings.
Projects who contribute to these discussions are able to influence them to ensure
that the language/protocol changes are favorable to them
and the rest of the app developers in the ecosystem.
It also helps the owner to promote the project and themselves.

Resources for Best Practices:

- [cadence/design-pattern](./design-patterns.md)
- [cadence/anti-patterns](./anti-patterns.md)
- [cadence/security-best-practices](./security-best-practices.md)

Composability and extensibility should also be priorities while designing, developing,
and documenting their projects. (Documentation for these topics coming soon)



If you have any feedback about these guidelines, please create an issue in the onflow/cadence-style-guide repo or make a PR updating the guidelines so we can start a discussion.
---
title: Cadence Security Best Practices
sidebar_label: Security Best Practices
sidebar_position: 7
---

This is an opinionated list of best practices Cadence developers should follow to write more secure Cadence code.

Some practices listed below might overlap with advice in the [Cadence Anti-Patterns](./design-patterns.md) section, which is a recommended read as well.

## References

[References](./language/references.md) are ephemeral values and cannot be stored. If persistence is required, store a capability and borrow it when needed.

Authorized references (references with the `auth` keyword) allow downcasting, e.g. a restricted type to its unrestricted type and should only be used in some specific cases.

When exposing functionality, provide the least access necessary. Do not use authorized references, as they can be downcasted, potentially allowing a user to gain access to supposedly restricted functionality. For example, the fungible token standard provides an interface to get the balance of a vault, without exposing the withdrawal functionality.

Be aware that the subtype or unrestricted type could expose functionality that was not intended to be exposed. Do not use authorized references when exposing functionality. For example, the fungible token standard provides an interface to get the balance of a vault, without exposing the withdrawal functionality.

## Account Storage

Don't trust a users’ [account storage](./language/accounts.mdx#account-storage). Users have full control over their data and may reorganize it as they see fit. Users may store values in any path, so paths may store values of “unexpected” types. These values may be instances of types in contracts that the user deployed.

Always [borrow](./language/capabilities.md) with the specific type that is expected. Or, check if the value is an instance of the expected type.

## Auth Accounts

Access to an `AuthAccount` gives full access to the account's storage, keys, and contracts. Therefore, [avoid using AuthAccount](./anti-patterns.md#avoid-using-authaccount-as-a-function-parameter) as a function parameter unless absolutely necessary.

It is preferable to use capabilities over direct `AuthAccount` storage when exposing account data. Using capabilities allows the revocation of access by unlinking and limits the access to a single value with a certain set of functionality – access to an `AuthAccount` gives full access to the whole storage, as well as key and contract management.

## Capabilities

Don’t store anything under the [public capability storage](./language/capabilities.md) unless strictly required. Anyone can access your public capability using `AuthAccount.getCapability`. If something needs to be stored under `/public/`, make sure only read functionality is provided by restricting its type using either a resource interface or struct interface.

When linking a capability, the link might already be present. In that case, Cadence will not panic with a runtime error but the link function will return `nil`.

It is a good practice to check if the link already exists with `getLinkTarget` before creating it. This function will return `nil` if the link does not exist.

If it is necessary to handle the case where borrowing a capability might fail, the `account.check` function can be used to verify that the target exists and has a valid type.

Ensure capabilities cannot be accessed by unauthorized parties. For example, capabilities should not be accessible through a public field, including public dictionaries or arrays. Exposing a capability in such a way allows anyone to borrow it and perform all actions that the capability allows.

## Transactions

Audits of Cadence code should also include [transactions](./language/transactions.md), as they may contain arbitrary code, just, like in contracts. In addition, they are given full access to the accounts of the transaction’s signers, i.e. the transaction is allowed to manipulate the signers’ account storage, contracts, and keys.

Signing a transaction gives access to the `AuthAccount`, i.e. full access to the account’s storage, keys, and contracts.

Do not blindly sign a transaction. The transaction could for example change deployed contracts by upgrading them with malicious statements, revoking or adding keys, transferring resources from storage, etc.

## Types

Use [restricted types and interfaces](./language/restricted-types.md). Always use the most specific type possible, following the principle of least privilege. Types should always be as restrictive as possible, especially for resource types.

If given a less-specific type, cast to the more specific type that is expected. For example, when implementing the fungible token standard, a user may deposit any fungible token, so the implementation should cast to the expected concrete fungible token type.

## Access Control

Declaring a field as [`pub/access(all)`](./language/access-control.md) only protects from replacing the field’s value, but the value itself can still be mutated if it is mutable. Remember that containers, like dictionaries, and arrays, are mutable.

Prefer non-public access to a mutable state. That state may also be nested. For example, a child may still be mutated even if its parent exposes it through a field with non-settable access.

Do not use the `pub/access(all)` modifier on fields and functions unless necessary. Prefer `priv/access(self)`, or `access(contract)` and `access(account)` when other types in the contract or account need to have access.
---
title: Guide for Solidity developers
sidebar_position: 8
---
Cadence introduces a different way to approach smart contract development which may feel unfamiliar to
Solidity developers. There are fundamental mindset and platform differences, and also several new language
features that have no real equivalent in Solidity. This guide outlines high level design and conceptual
aspects of Flow and Cadence that are essential to understand, platform and integration differences, as well
as detailed guidance on how to perform certain common Solidity development tasks using Cadence idioms. We also
provide details on how best to leverage Cadence's unique features and how to avoid common pitfalls that may come
up while transitioning.

# Conceptual foundations for Cadence

A fundamental difference to get used to when adjusting to Cadence from Solidity is of mindset. Security and
interoperability on Ethereum are designed around addresses (or more specifically the account associated with an
address), resulting in all contracts having to carefully track and evaluate access and authorizations.

<img src="https://storage.googleapis.com/flow-resources/documentation-assets/solidity-to-cadence/ethereum-ownership.png" width="450" align="right"/>

Transactions are based on who authorized them, which is provided as `msg.sender` in the transaction context.
User to contract, or contract to contract interactions must be explicitly coded to ensure the appropriate approvals
have been made before interacting with a contract. The contract based nature of storage means that user ownership
in Ethereum is represented in a mapping, for example from owner to balance, or token ID to owner. Put another
way, ownership is tracked in ledger records similar to a person's bank balance. Crypto wallets help combine
balances from multiple token types into a convenient view for the user.

Cadence introduces new primitives and distinct functionalities, namely Resources and Capabilities, that are designed
around Flow's account model. Resources are first-class language types which are unique, non-copyable, and which cannot
be discarded. These properties make Resources ideal for representing digital assets like currency or tokens that are always limited in numbers.
Resources are always stored in account storage and contracts control access to them using Capabilities. Capabilities
are another special type that secure protected resources without the need for tracking addresses. Cadence makes
working with these straightforward and intuitive to those familiar with object-oriented programming languages.

Newcomers to Cadence should ensure they understand the following major concepts before development.

## Flow account model

The [Flow account model](https://developers.flow.com/build/basics/accounts.md) in Cadence combines storage for the keys and code
(”smart contracts”) associated with an account with storage for the assets owned by that account. That’s right:
In Cadence, your tokens are stored in your account, and not in a smart contract. Of course, smart contracts still
define these assets and how they behave, but those assets can be securely stored in a user’s account through the
magic of Resources.

<img src="https://storage.googleapis.com/flow-resources/documentation-assets/solidity-to-cadence/account-structure.png"
     width="300" align="right"/>

There is only one account type in Cadence also with an account address, similar to an Externally-Owned-Account
(EOA) address in Ethereum. However, unlike Ethereum contract-accounts, accounts in Cadence also store contract code.
Accounts realize ownership on Flow in being the container where keys, Resources, and contracts are stored on-chain.


### PublicAccount and AuthAccount

`PublicAccount` is the publicly available part of an account through which public functions or objects can be
accessed by any account using the `getAccount` function.

`AuthAccount` is available only to the signer of the transaction. An `AuthAccount` reference provides full access
to that account's storage, keys configuration, and contract code. Capabilities ensure that resources held in an account can be
safely shared/accessed; access to `AuthAccount` should never be given to other accounts.

## Resources

Resources are unique, [linear-types](https://en.wikipedia.org/wiki/Substructural_type_system#Linear_type_systems) which
can never be copied or implicitly discarded, only moved between accounts. If, during development, a function fails to
store a Resource obtained from an account in the function scope, semantic checks will flag an error. The run-time
enforces the same strict rules in terms of allowed operations. Therefore, contract functions which do not properly
handle Resources in scope before exiting will abort, reverting them to the original storage. These features of
Resources make them perfect for representing tokens, both fungible and non-fungible. Ownership is tracked by where
they are stored, and the assets can’t be duplicated or accidentally lost since the language itself enforces correctness.

Flow encourages storing of data and compute on-chain and Resource-types makes this easier than ever. Since Resources
are always stored in accounts, any data and code that exists in Resource instances is seamlessly managed on-chain
without any explicit handling needed.

## Capability-based access

Remote access to stored objects can be managed via [Capabilities](./language#capability-based-access-control). This
means that if an account wants to be able to access another account's stored objects, it must have been provided
with a valid Capability to that object. Capabilities can be either public or private. An account can share a public
Capability if it wants to give all other accounts access. (For example, it’s common for an account to accept fungible
token deposits from all sources via a public Capability.) Alternatively, an account can grant private Capabilities
to specific accounts in order to provide access to restricted functionality. For example, an NFT project often
controls minting through an “administrator Capability” that grants specific accounts with the power to mint new tokens.

## Contract standards

There are numerous widely-used contract standards established to benefit the ecosystem, for example
[Fungible Token](https://developers.flow.com/build/flow.md#flow-token)(FT) and [Non-Fungible Token](https://developers.flow.com/build/flow.md#flow-nft#overview)(NFT)
standards which are conceptually equivalent to Ethereum's ERC-20 and ERC-721 standards. Cadence's object-oriented
design means standards apply through contract sub-types such as Resources, Resource interfaces, or other types
declared in the contract standard. Standards can define and limit behaviour and/or set conditions which
implementations of the standard cannot violate.

Detailed information about available standards and other core contracts can be found in [Introduction to Flow](https://developers.flow.com/build/flow.md).

### NFT standard and metadata

Solidity must manage NFT metadata off-chain and NFTs typically link to IPFS JSON from on-chain.

The Cadence NFT standard provides in-built support for metadata with specific types called [views](https://developers.flow.com/build/flow.md#flow-nft#overview#nft-metadata).
Views can be added to NFTs when minted and will always be available as part of the NFT. While metadata is stored
on-chain, graphics and video content are stored off-chain. Cadence provides [utility views](https://developers.flow.com/build/flow.md#flow-nft#overview#list-of-common-views)
for both HTTP and IPFS based media storage which remain linked to your NFT.

Using NFT metadata views is a requirement to get listed in the [Flow NFT Catalog](https://www.flow-nft-catalog.com/).
Projects are encouraged leverage the NFT catalog since wallets and other ecosystem partners can seamlessly integrate
new collections added there with no input from project creators.

NFT metadata on Flow opens the door to exciting new possibilities that help builders innovate. Check out this
recent [case study](https://flow.com/post/flovatar-nft-flow-blockchain-case-study) where a community partner
leveraged SVG based metadata to make combined 2D + 3D versions of their PFPs, all on-chain inside the NFTs
metadata!

# Security and access control

Decentralized application development places significant focus on security and access and can fairly be described
as security engineering. Understanding how Resources, Capabilities and the account model solve this may not
be obvious when viewed from a Solidity perspective.

## msg.sender considered harmful

The first question that every Solidity developer asks when they start programming in Cadence is:

**"How do I get the account who authorized the transaction?"**

In Ethereum this account is referred to as `msg.sender` and informs the program flow in a function depending on who
authorized it. Doing so is key to access and security, and is the basis of identity and ownership on Ethereum.

Cadence does not have `msg.sender` and there is no transaction-level way for Cadence code to uniquely identify the
calling account. Even if there was a way to access it, Cadence supports [multi-sig](#multi-key-multi-signature-support)
transactions, meaning that a list of all the signers' accounts would be returned, making it impossible to identify a single authorizer.

<img src="https://storage.googleapis.com/flow-resources/documentation-assets/solidity-to-cadence/access-based-security.png"
     width="225" align="right" alt="Solidity access applies to the subject and possessed and validated by the protected
     resource"/>

The reason `msg.sender` is both unsupported and strongly advised against is because Cadence uses Capabilities for
access rather than addresses. The mindset change that developers need to adjust to is that a capability must first be
obtained by the authorizing account (called provider or signer in Cadence) from the contract that will require it,
which then enables the requesting account to access the protected function or Resource. This means the contract never
needs to know who the signer is before proceeding because the capability **IS** the authorization.

<img src="https://storage.googleapis.com/flow-resources/documentation-assets/solidity-to-cadence/capability-based-security.png"
     width="245" align="left" alt="In Cadence, the subject must possess the Capability to access the protected resource"/>

The [capability-based security](https://en.wikipedia.org/wiki/Capability-based_security) model frames access in
the opposite direction than the [access-based security](https://en.wikipedia.org/wiki/Access-control_list) model.


## Access control using Capabilities

Solidity lacks specific types or other primitives to aid with permission management. Developers have to inline
guards to `require` at every function entry-point, thus validating the `msg.sender` of the transaction.

[Capabilities](./language/capabilities.md) are defined by linking storage paths (namespaces for contract
storage) to protected objects and then making that linked capability available to other accounts. Public and private
scopes defined for storage paths and Capabilities themselves align precisely with
[PublicAccount](./language/accounts.mdx#publicaccount) / [AuthAccount](./language/accounts.mdx#authaccount) account scopes.

Any account can get access to an account's public Capabilities. Public capabilities are created using public paths,
i.e. they have the domain `public`. For example, all accounts have a default public capability linked to the
`FlowToken.Vault` Resource. This vault is exposed as a public capability, scoped to the `FungibleToken.Receiver` resource
interface, to allow any account to `borrow()` a reference to the Vault to make a `deposit()`. Since only the functions
defined under the `[FungibleToken.Receiver](https://github.com/onflow/flow-ft/blob/master/contracts/FungibleToken.cdc#L105)`
interface are exposed, the borrower of the vault reference cannot call `withdraw()` since it is scoped in the `provider`
interface. Public capabilities can be obtained from both authorized accounts (`AuthAccount`) and public accounts
(`PublicAccount`).

Private capabilities are specifically granted to accounts. They are created using private paths, i.e. they have the
domain `private`. After creation, they can be obtained from authorised account objects (`AuthAccount`) but not public
accounts (`PublicAccount`). To share a private Capability with another account, the owning account must `publish` it
to another account which places in the [account inbox](./language/accounts.mdx#account-inbox). The recipient can later
claim the Capability from the account inbox using then `claim` function.

Capabilities can be `unpublished` and can also be [revoked](./design-patterns.md#capability-revocation) by the creating
account.

To aid automation, events are emitted for `publish`, `claim` and `unpublish` actions completed for a Capability.

Detailed information can be found in [Capabilities](./language/capabilities.md).

## Hygiene factors for protecting value

While capabilities grant account access to a protected resource, it's still necessary to impose controls on value
accessed through them. For example, if your use-case requires delegating access to a `FlowToken.Vault` to
`withdraw()` funds, it's important to limit the amount. Tokens implementing FT/NFT standards are the primary type of
value being exchanged by accounts on Flow. The standard provides the primitives needed to implement capability
limiting best-practices.

### Token isolation

All FTs reside in a `Vault` Resource and each different FT will exist as a separate `Vault` in an account. Similarly,
all NFTs implement a `Collection` Resource, in which those NFTs held by an account for that collection are stored.

Whenever access to the `withdraw()` function has to be delegated to another account, the simplest way to limit how
many tokens of a given type can be withdrawn is to create a new `Vault` Resource for that token type and move a smaller
amount of the tokens in the main token `Vault`. A capability is then linked to that `Vault` instance before being
made available to another account.

A similar pattern can be used for NFTs, where a new `Collection` Resource can be created into which only those NFTs
which should be exposed are moved. A capability is then linked to that `Collection` instance before being made
available to another account.

### Bespoke control strategies

For more complex use-cases one might create a new Resource that implements the relevant interfaces to match those of
the protected Resource(s) which it wraps. The code for the new Resource can then enforce limits as required and control
how and when delegation to the underlying resource occurs.

## Admin roles

Compared to Solidity, creating an admin role in Cadence requires a little more code, all of which is encapsulated
within a Resource. The admin object design can be highly customized and employ Capabilities for fine-grained control
such as limiting access to individual functions, on a per-account basis if required. The complexity needed for admin
roles may vary, for example, larger organizations may require more complex role-based-access schemes. The use of a
Resource in this context is key - the instance can't be copied and the account with the first edition mint of the admin
serves as the root-admin. The admin can be implemented to mint additional admin Resource instances, which only the
root-admin can grant to selected user accounts via a Capability. Conveniently, because the admin role is only
accessible via a Capability it's easy to manage with [Capability Revocation](./design-patterns.md#capability-revocation).

The admin role originates from the [init singleton pattern](./design-patterns.md#init-singleton) and uses the
[Capability Bootstrapping](./design-patterns.md#capability-bootstrapping) pattern for making the Capability available to
other accounts.

An example admin role implementation is available in [Cadence cookbook](https://cookbook.onflow.org/?preview=13).

### Role-based access

Implementing role-based-access can be achieved by defining roles as Resources managed by the root-admin account.
Roles can provide limited access to functions which guard other protected resources, with access levels and/or
what is exposed varying from role to role. The root-admin can grant accounts access to individual roles through a
private capability. Functions that the roles are permitted to invoke may be scoped as `access(contract)` to
enforce that they can only be called by code paths in the root-admin contract.

# Other best practices and conventions

Certain well established best practices for Solidity may not apply or are handled differently.

## Check effects interactions

Solidity contracts must use the [check effect interaction](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html)
because functions are public by default and address-based access means that guards must exist when program flow
concedes control to an external contract. There are two reasons why this is significantly less of a problem in
Cadence. Functions are private by default and the language provides a range of [access scopes](./language/access-control.md).
More importantly, 'risks associated with ceding control to an external contract' is an Ethereum phenomenon;
the risk no longer applies. This is primarily because Cadence contracts are not static singletons, so control is
never lost to another contract during the scope of a transaction.

## Guard Check

Solidity uses `revert`, `require` & `assert` to validate inputs. `require` is a product of the address-based nature
of Solidity which Capabilities replace. `revert` is similar to Cadence's `panic` in that a transaction is aborted.
Cadence provides an `assert` operator which mirrors `assert` in Solidity.

## Modifiers

Modifiers are extensively used in Solidity when enforcing pre-checks within a function. This is a powerful language
feature. However, modifiers can also mutate state which introduces risks to program control flow.

Cadence uses `pre` and `post` blocks to validate input values or the function execution outputs. Notably, `pre` and
`post` block prohibit changing of state and may only enforce conditions.

Another difference is that modifiers in Solidity can be re-used within the contract multiple times. Cadence
`pre` & `post` blocks are associated to individual functions only, reducing the likelihood of errors but which
results in a small amount of code duplication.

## Error handling

Solidity offers try/catch block to handle errors, however, there is presently no equivalent in Cadence.

# Integration differences

## Scripts and transactions

Another major difference between Cadence and Solidity is that deployed contracts are not the only code being executed
in the VM. Cadence offers scripts, of which a subset are transactions, and both permit arbitrary code. Scripts or
transactions are not deployed on-chain and always exist off-chain, however, they are the top-level code payload
being executed by the execution runtime. Clients send scripts and transactions through the Flow Access API gRPC or
REST endpoints, returning results to clients when applicable. Scripts and transactions enable more efficient and
powerful ways to integrate dapps with the underlying blockchain, where contracts can more purely be thought of as
services or components, with scripts or transactions becoming the dapp-specific API interface for chain interactions.

Scripts are read-only in nature, requiring only a `main` function declaration and which perform
[queries](https://github.com/onflow/flow-ft/blob/master/transactions/scripts/get_balance.cdc) against chain state, eg:

```jsx
// This script reads the balance field of an account's ExampleToken Balance
import FungibleToken from "../../contracts/FungibleToken.cdc"
import ExampleToken from "../../contracts/ExampleToken.cdc"

pub fun main(account: Address): UFix64 {
    let acct = getAccount(account)
    let vaultRef = acct.getCapability(ExampleToken.VaultPublicPath)
        .borrow<&ExampleToken.Vault{FungibleToken.Balance}>()
        ?? panic("Could not borrow Balance reference to the Vault")

    return vaultRef.balance
}
```

[Transactions](https://github.com/onflow/flow-ft/tree/master/transactions) are an ACID (Atomic, Consistent,
Isolated and Durable) version of scripts having only `prepare` and `execute` functions that either succeed in
full and mutate chain state as described, or otherwise fail and mutate nothing. They also support setting of `pre`
and `post` conditions. In the example transaction below `ExampleToken`s are deposited into multiple `receiver`
vaults for each address in the input map.

```jsx
import FungibleToken from "../contracts/FungibleToken.cdc"
import ExampleToken from "../contracts/ExampleToken.cdc"

/// Transfers tokens to a list of addresses specified in the `addressAmountMap` parameter
transaction(addressAmountMap: {Address: UFix64}) {

    // The Vault resource that holds the tokens that are being transferred
    let vaultRef: &ExampleToken.Vault

    prepare(signer: AuthAccount) {

        // Get a reference to the signer's stored vault
        self.vaultRef = signer.borrow<&ExampleToken.Vault>(from: ExampleToken.VaultStoragePath)
			?? panic("Could not borrow reference to the owner's Vault!")
    }

    execute {

        for address in addressAmountMap.keys {

            // Withdraw tokens from the signer's stored vault
            let sentVault <- self.vaultRef.withdraw(amount: addressAmountMap[address]!)

            // Get the recipient's public account object
            let recipient = getAccount(address)

            // Get a reference to the recipient's Receiver
            let receiverRef = recipient.getCapability(ExampleToken.ReceiverPublicPath)
                .borrow<&{FungibleToken.Receiver}>()
                ?? panic("Could not borrow receiver reference to the recipient's Vault")

            // Deposit the withdrawn tokens in the recipient's receiver
            receiverRef.deposit(from: <-sentVault)

        }
    }
}
```

Transactions can encompass an arbitrary number withdrawals/deposits, across multiple FTs, sending to multiple
addresses, or other more complex variations, all of which will succeed or fail in their entirety given their
ACID properties.

## Contract imports and dynamic contract borrowing

Contracts in Ethereum are similar to static singletons in that interactions happen directly between users and the
functions declared on the contract instance itself. The object-oriented nature of Cadence means that contracts are
more accurately viewed as imported dependencies. The imported contract makes its object graph available for the
code at runtime. Rather than interacting with a contract singleton instance, account interactions to access
capabilities are the primary integration entry point, allowing the user to interact with the returned objects.

Dynamic borrowing of a contract inlines the loading of a contract based on its contract address. The loaded
contract can be cast to the contract standards it conforms to, eg: NFT standard, and then interacted with in the
same way were it imported. Consider the implications of this for composability of contracts..

Detailed information about deploying, updating, removing or borrowing contracts can be found in [Contracts](./language/contracts.mdx)

## Multi-key, multi-signature support

Solidity supports only one kind of multi-signature scheme where n out of m (assuming m > n) approvals need to be
obtained to execute the transaction from the multi-signature smart contract. The most used multi-signature smart
contract in the Ethereum ecosystem is the gnosis [safe contract](https://github.com/safe-global/safe-contracts/blob/main/contracts/Safe.sol).
However, Solidity lacks support for signature aggregation or BLS signature schemes.

Cadence offers a wide range of options to implement various multi-signature schemes.

- Inherent support for multi-sign transactions.
- Resource transfer scheme.
- Inherent support of BLS signature scheme.

Flow account keys have assigned weights, where a 1000 unit weight is the cumulative weight needed from signing
keys to execute a transaction successfully. One can divide weights across multiple keys and distribute those partial
weighted keys to authorized signers. When signing the transaction, all signers must sign the transaction together
in a short period of time in order for the cumulative weight to reach 1000 units.

See [BLS Signature scheme](./language/crypto.mdx#bls-multi-signature) for a detailed
overview of the inherent support of BLS signatures.

### Resource transfer scheme

The main limitation of multi-sig transactions is that signatures must all be made for the transaction within a
relatively short time window. If this window is missed, the transaction will abort. The resource transfer scheme is
very similar to the Solidity multi-signature smart contract. A Resource is created that has the functionality to
proxy the execution of a fund transfer. This Resource is handed from one signer to the next to collect signatures.
Once the threshold of required signatures is met the transaction is executed. The elapsed time The main drawback
with this approach is that does not support execution of arbitrary functionality.

# Other platform differences

The following differences unrelated to implementing Cadence contracts are useful to understand in the context
of application design.

## Events

Flow uses [events](./language/events.md) extensively to provide real-time signals to off-chain systems about particular
actions that occurred during a transaction. The main difference on Flow is that events remain part of the history
and are not purged from storage. Events can be populated with arbitrary data that will assist consumers of the
event. Builders are encouraged to leverage events for seamless UX as users perform transactions.

## Contract upgradeability

Flow supports limited upgradability of Cadence contracts which is most helpful during development. The following
function shows how an account owner can update a contract. Upgrades are analyzed for prohibited changes once
uploaded for upgrade. Upgradeability is still an early phase feature, which will continue to improve over time.

```solidity
fun update__experimental(name: String, code: [UInt8]): DeployedContract
```

To enforce immutability once a contract is tested and ready to deploy, account owners can optionally revoke keys
from the account containing the contract.

Detailed information about the cadence upgradeability is available in [Contract updatability](./language/contract-updatability.md).

## Account key formulation

In EVM-based chains, an address is derived from a cryptographically generated public key and can have a single private
key, supporting one type of signature curve, i.e. ECDSA. They are not verifiable off-chain and typos/truncation in an
address may result in funds being lost.

Flow account addresses have a special format and are verifiable off-chain. Verifying address format validity can be
done using an error detection algorithm based on linear code. While this does not also confirm that an address is active
on-chain the extra verifiability is a useful safeguard.

## Contract size constraints

Solidity developers will be well aware of the [EIP-170](https://eips.ethereum.org/EIPS/eip-170) deployable contract
bytecode size limit of 24KB. This can burden builders who need to optimize contract bytecode size, sometimes even
requiring a re-design of contracts to break it into smaller contract parts.

By contrast, Cadence has no inherent or defined smart contract size limit. However, it is restricted by the transaction
size limit which is 1.5MB. With very rare exceptions, it’s unlikely that this limit would pose a problem to those
developing Cadence contracts. Should it be needed, there is a known way to deploy a contract exceeding 1.5 MB which we
will document at a later time.

# Low level language differences

## Arithmetic

Historically, Solidity, smart contracts lost millions of dollars because of improper handling of arithmetic
under/overflows. Contemporary Solidity versions offer inbuilt handling of under/overflow for arithmetic operations.

Cadence implements [saturating math](https://en.wikipedia.org/wiki/Saturation_arithmetic) that avoids
overflow/underflow.

## Optional support

[Optional bindings](./language/control-flow.md) provide in-built conditional handling of nil values. Regular data types in
Cadence must always have a value and cannot be nil. Optionals enable variables / constants that might contain a
certain type or a nil value Optionals have two cases: either there is a value, or there is nothing; they fork
program flow similar to `if nil; else; end;`.

## Iterable Dictionaries

Solidity offers the mapping type, however, it is not iterable. Because of that dApp developers have to maintain
off-chain tracking to be have access to keys. This also pushes builders to create custom datatypes like `EnumerableMap`
which adds to gas costs.

Cadence offers the [Dictionary](./language/control-flow.md) type, an unordered collection of key-value associations
which is iterable.

## Rich support for type utility functions

Cadence offers numerous native-type utility functions to simplify development. For example, the String type
provides:
- utf8
- concat()
- slice()
- decodeHex()
- encodeHex()
- toLower()
- length

## Argument labelling

Argument labels in Cadence help to disambiguate input values. They makes code more readable and explicit. They
also eliminate confusion around the order of arguments when working with the same type. They must be included in the function call. This restriction can be skipped if the label is preceded by `_ ` on its declaration.

Eg:
`fun foo(balance: UFix64)`
called as
`self.foo(balance: 30.0)`

`fun foo( _balance: UFix64)`
can be called as
`self.foo(balance: 30.0)`
or as
`self.foo(30.0)`

## Additional resources
* Cadence or Solidity: [On-Chain Token Transfer Deep Dive](https://flow.com/engineering-blogs/flow-blockchain-programming-language-smart-contract-cadence-solidity-comparison-ethereum)
* Implementing the [Bored Ape Yacht Club](https://flow.com/post/implementing-the-bored-ape-yacht-club-smart-contract-in-cadence) smart contract in Cadence---
title: Cadence Testing Framework
sidebar_label: Testing
---

The Cadence testing framework provides a convenient way to write tests for Cadence programs in Cadence.
This functionality is provided by the built-in `Test` contract.

<Callout type="info">
The testing framework can only be used off-chain, e.g. by using the [Flow CLI](https://developers.flow.com/tools/flow-cli).
</Callout>

Tests must be written in the form of a Cadence script.
A test script may contain testing functions that starts with the `test` prefix,
a `setup` function that always runs before the tests,
a `tearDown` function that always runs at the end of all test cases,
a `beforeEach` function that runs before each test case,
and an `afterEach` function that runs after each test case.
All the above four functions are optional.

```cadence
// A `setup` function that always runs before the rest of the test cases.
// Can be used to initialize things that would be used across the test cases.
// e.g: initialling a blockchain backend, initializing a contract, etc.
access(all) fun setup() {
}

// The `beforeEach` function runs before each test case. Can be used to perform
// some state cleanup before each test case, among other things.
access(all) fun beforeEach() {
}

// The `afterEach` function runs after each test case.  Can be used to perform
// some state cleanup after each test case, among other things.
access(all) fun afterEach() {
}

// Valid test functions start with the 'test' prefix.
access(all) fun testSomething() {
}

access(all) fun testAnotherThing() {
}

access(all) fun testMoreThings() {
}

// Test functions cannot have any arguments or return values.
access(all) fun testInvalidSignature(message: String): Bool {
}

// A `tearDown` function that always runs at the end of all test cases.
// e.g: Can be used to stop the blockchain back-end used for tests, etc. or any cleanup.
access(all) fun tearDown() {
}
```
## Test Standard Library

The testing framework can be used by importing the built-in `Test` contract:

```cadence
import Test
```

## Assertions

### Test.assert

```cadence
fun assert(_ condition: Bool, message: String)
```

Fails a test-case if the given condition is false, and reports a message which explains why the condition is false.

The message argument is optional.

```cadence
import Test

access(all) fun testExample() {
    Test.assert(2 == 2)
    Test.assert([1, 2, 3].length == 0, message: "Array length is not 0")
}
```

### Test.fail

```cadence
fun fail(message: String)
```

Immediately fails a test-case, with a message explaining the reason to fail the test.

The message argument is optional.

```cadence
import Test

access(all) fun testExample() {
    let array = [1, 2, 3]

    if array.length != 0 {
        Test.fail(message: "Array length is not 0")
    }
}
```

### Test.expect

```cadence
fun expect(_ value: AnyStruct, _ matcher: Matcher)
```

The `expect` function tests a value against a matcher (see [matchers](#matchers) section), and fails the test if it's not a match.

```cadence
import Test

access(all) fun testExample() {
    let array = [1, 2, 3]

    Test.expect(array.length, Test.equal(3))
}
```

### Test.assertEqual

```cadence
fun assertEqual(_ expected: AnyStruct, _ actual: AnyStruct)
```

The `assertEqual` function fails the test-case if the given values are not equal, and
reports a message which explains how the two values differ.

```cadence
import Test

access(all) struct Foo {
    access(all) let answer: Int

    init(answer: Int) {
        self.answer = answer
    }
}

access(all) fun testExample() {
    Test.assertEqual("this string", "this string")
    Test.assertEqual(21, 21)
    Test.assertEqual(true, true)
    Test.assertEqual([1, 2, 3], [1, 2, 3])
    Test.assertEqual(
        {1: true, 2: false, 3: true},
        {1: true, 2: false, 3: true}
    )

    let address1 = Address(0xf8d6e0586b0a20c7)
    let address2 = Address(0xf8d6e0586b0a20c7)
    Test.assertEqual(address1, address2)

    let foo1 = Foo(answer: 42)
    let foo2 = Foo(answer: 42)

    Test.assertEqual(foo1, foo2)

    let number1: Int64 = 100
    let number2: UInt64 = 100
    // Note that the two values need to have exactly the same type,
    // and not just value, otherwise the assertion fails:
    // assertion failed: not equal: expected: 100, actual: 100
    Test.assertEqual(number1, number2)
}
```

### Test.expectFailure

```cadence
fun expectFailure(_ functionWrapper: ((): Void), errorMessageSubstring: String)
```

The `expectFailure` function wraps a function call in a closure, and expects it to fail with
an error message that contains the given error message portion.

```cadence
import Test

access(all) struct Foo {
    access(self) let answer: UInt8

    init(answer: UInt8) {
        self.answer = answer
    }

    access(all) fun correctAnswer(_ input: UInt8): Bool {
        if self.answer != input {
            panic("wrong answer!")
        }
        return true
    }
}

access(all) fun testExample() {
    let foo = Foo(answer: 42)

    Test.expectFailure(fun(): Void {
        foo.correctAnswer(43)
    }, errorMessageSubstring: "wrong answer!")
}
```

## Matchers

A matcher is an object that consists of a test function and associated utility functionality.

```cadence
access(all) struct Matcher {

    access(all) let test: fun(AnyStruct): Bool

    access(all) init(test: fun(AnyStruct): Bool) {
        self.test = test
    }

    /// Combine this matcher with the given matcher.
    /// Returns a new matcher that succeeds if this and the given matcher succeed.
    ///
    access(all) fun and(_ other: Matcher): Matcher {
        return Matcher(test: fun (value: AnyStruct): Bool {
            return self.test(value) && other.test(value)
        })
    }

    /// Combine this matcher with the given matcher.
    /// Returns a new matcher that succeeds if this or the given matcher succeeds.
    ///
    access(all) fun or(_ other: Matcher): Matcher {
        return Matcher(test: fun (value: AnyStruct): Bool {
            return self.test(value) || other.test(value)
        })
    }
}
```

The `test` function defines the evaluation criteria for a value, and returns a boolean indicating whether the value
conforms to the test criteria defined in the function.

The `and` and `or` functions can be used to combine this matcher with another matcher to produce a new matcher with
multiple testing criteria.
The `and` method returns a new matcher that succeeds if both this and the given matcher are succeeded.
The `or` method returns a new matcher that succeeds if at-least this or the given matcher is succeeded.

A matcher that accepts a generic-typed test function can be constructed using the `newMatcher` function.

```cadence
fun newMatcher<T: AnyStruct>(_ test: fun(T): Bool): Test.Matcher
```

The type parameter `T` is bound to `AnyStruct` type. It is also optional.

For example, a matcher that checks whether a given integer value is negative can be defined as follows:

```cadence
import Test

access(all) fun testExample() {
    let isNegative = Test.newMatcher(fun (_ value: Int): Bool {
        return value < 0
    })

    Test.expect(-15, isNegative)
    // Alternatively, we can use `Test.assert` and the matcher's `test` function.
    Test.assert(isNegative.test(-15), message: "number is not negative")
}

access(all) fun testCustomMatcherUntyped() {
    let matcher = Test.newMatcher(fun (_ value: AnyStruct): Bool {
        if !value.getType().isSubtype(of: Type<Int>()) {
            return false
        }

        return (value as! Int) > 5
    })

    Test.expect(8, matcher)
}

access(all) fun testCustomMatcherTyped() {
    let matcher = Test.newMatcher<Int>(fun (_ value: Int): Bool {
        return value == 7
    })

    Test.expect(7, matcher)
}
```

The `Test` contract provides some built-in matcher functions for convenience.

### Test.equal

```cadence
fun equal(_ value: AnyStruct): Matcher
```

The `equal` function returns a matcher that succeeds if the tested value is equal to the given value.
Accepts an `AnyStruct` value.

```cadence
import Test

access(all) fun testExample() {
    let array = [1, 2, 3]

    Test.expect([1, 2, 3], Test.equal(array))
}
```

### Test.beGreaterThan

```cadence
fun beGreaterThan(_ value: Number): Matcher
```

The `beGreaterThan` function returns a matcher that succeeds if the tested value is a number and
greater than the given number.

```cadence
import Test

access(all) fun testExample() {
    let str = "Hello, there"

    Test.expect(str.length, Test.beGreaterThan(5))
}
```

### Test.beLessThan

```cadence
fun beLessThan(_ value: Number): Matcher
```

The `beLessThan` function returns a matcher that succeeds if the tested value is a number and
less than the given number.

```cadence
import Test

access(all) fun testExample() {
    let str = "Hello, there"

    Test.expect(str.length, Test.beLessThan(15))
}
```

### Test.beNil

```cadence
fun beNil(): Matcher
```

The `beNil` function returns a new matcher that checks if the given test value is nil.

```cadence
import Test

access(all) fun testExample() {
    let message: String? = nil

    Test.expect(message, Test.beNil())
}
```

### Test.beEmpty

```cadence
fun beEmpty(): Matcher
```

The `beEmpty` function returns a matcher that succeeds if the tested value is an array or dictionary,
and the tested value contains no elements.

```cadence
import Test

access(all) fun testExample() {
    let array: [String] = []

    Test.expect(array, Test.beEmpty())

    let dictionary: {String: String} = {}

    Test.expect(dictionary, Test.beEmpty())
}
```

### Test.haveElementCount

```cadence
fun haveElementCount(_ count: Int): Matcher
```

The `haveElementCount` function returns a matcher that succeeds if the tested value is an array or dictionary,
and has the given number of elements.

```cadence
import Test

access(all) fun testExample() {
    let array: [String] = ["one", "two", "three"]

    Test.expect(array, Test.haveElementCount(3))

    let dictionary: {String: Int} = {"one": 1, "two": 2, "three": 3}

    Test.expect(dictionary, Test.haveElementCount(3))
}
```

### Test.contain

```cadence
fun contain(_ element: AnyStruct): Matcher
```

The `contain` function returns a matcher that succeeds if the tested value is an array that contains
a value that is equal to the given value, or the tested value is a dictionary
that contains an entry where the key is equal to the given value.

```cadence
access(all) fun testExample() {
    let array: [String] = ["one", "two", "three"]

    Test.expect(array, Test.contain("one"))

    let dictionary: {String: Int} = {"one": 1, "two": 2, "three": 3}

    Test.expect(dictionary, Test.contain("two"))
}
```

### Test.beSucceeded

```
fun beSucceeded(): Matcher
```

The `beSucceeded` function returns a new matcher that checks if the given test value is either
a ScriptResult or TransactionResult and the ResultStatus is succeeded.
Returns false in any other case.

```cadence
import Test

access(all) fun testExample() {
    let blockchain = Test.newEmulatorBlockchain()
    let result = blockchain.executeScript(
        "access(all) fun main(): Int {  return 2 + 3 }",
        []
    )

    Test.expect(result, Test.beSucceeded())
    Test.assertEqual(5, result.returnValue! as! Int)
}
```

### Test.beFailed

```cadence
fun beFailed(): Matcher
```

The `beFailed` function returns a new matcher that checks if the given test value is either
a ScriptResult or TransactionResult and the ResultStatus is failed.
Returns false in any other case.

```cadence
import Test

access(all) fun testExample() {
    let blockchain = Test.newEmulatorBlockchain()
    let account = blockchain.createAccount()

    let tx = Test.Transaction(
        code: "transaction { execute{ panic(\"some error\") } }",
        authorizers: [],
        signers: [account],
        arguments: [],
    )

    let result = blockchain.executeTransaction(tx)

    Test.expect(result, Test.beFailed())
}
```

## Matcher combinators

The built-in matchers, as well as custom matchers, can be combined with the three available combinators:

- `not`,
- `or`,
- `and`

in order to create more elaborate matchers and increase re-usability.

### not

```cadence
fun not(_ matcher: Matcher): Matcher
```

The `not` function returns a new matcher that negates the test of the given matcher.

```cadence
import Test

access(all) fun testExample() {
    let isEven = Test.newMatcher<Int>(fun (_ value: Int): Bool {
        return value % 2 == 0
    })

    Test.expect(8, isEven)
    Test.expect(7, Test.not(isEven))

    let isNotEmpty = Test.not(Test.beEmpty())

    Test.expect([1, 2, 3], isNotEmpty)
}
```

### or

```cadence
fun or(_ other: Matcher): Matcher
```

The `Matcher.or` function combines this matcher with the given matcher.
Returns a new matcher that succeeds if this or the given matcher succeed.
If this matcher succeeds, then the other matcher would not be tested.

```cadence
import Test

access(all) fun testExample() {
    let one = Test.equal(1)
    let two = Test.equal(2)

    let oneOrTwo = one.or(two)

    Test.expect(2, oneOrTwo)
}
```

### and

```cadence
fun and(_ other: Matcher): Matcher
```

The `Matcher.and` function combines this matcher with the given matcher.
Returns a new matcher that succeeds if this and the given matcher succeed.

```cadence
import Test

access(all) fun testExample() {
    let sevenOrMore = Test.newMatcher<Int>(fun (_ value: Int): Bool {
        return value >= 7
    })
    let lessThanTen = Test.newMatcher<Int>(fun (_ value: Int): Bool {
        return value <= 10
    })

    let betweenSevenAndTen = sevenOrMore.and(lessThanTen)

    Test.expect(8, betweenSevenAndTen)
}
```

## Blockchain

A blockchain is an environment to which transactions can be submitted to, and against which scripts can be run.
It imitates the behavior of a real network, for testing.

```cadence
/// Blockchain emulates a real network.
///
access(all) struct Blockchain {

    access(all) let backend: AnyStruct{BlockchainBackend}

    init(backend: AnyStruct{BlockchainBackend}) {
        self.backend = backend
    }

    /// Executes a script and returns the script return value and the status.
    /// `returnValue` field of the result will be `nil` if the script failed.
    ///
    access(all) fun executeScript(_ script: String, _ arguments: [AnyStruct]): ScriptResult {
        return self.backend.executeScript(script, arguments)
    }

    /// Creates a signer account by submitting an account creation transaction.
    /// The transaction is paid by the service account.
    /// The returned account can be used to sign and authorize transactions.
    ///
    access(all) fun createAccount(): Account {
        return self.backend.createAccount()
    }

    /// Add a transaction to the current block.
    ///
    access(all) fun addTransaction(_ tx: Transaction) {
        self.backend.addTransaction(tx)
    }

    /// Executes the next transaction in the block, if any.
    /// Returns the result of the transaction, or nil if no transaction was scheduled.
    ///
    access(all) fun executeNextTransaction(): TransactionResult? {
        return self.backend.executeNextTransaction()
    }

    /// Commit the current block.
    /// Committing will fail if there are un-executed transactions in the block.
    ///
    access(all) fun commitBlock() {
        self.backend.commitBlock()
    }

    /// Executes a given transaction and commits the current block.
    ///
    access(all) fun executeTransaction(_ tx: Transaction): TransactionResult {
        self.addTransaction(tx)
        let txResult = self.executeNextTransaction()!
        self.commitBlock()
        return txResult
    }

    /// Executes a given set of transactions and commits the current block.
    ///
    access(all) fun executeTransactions(_ transactions: [Transaction]): [TransactionResult] {
        for tx in transactions {
            self.addTransaction(tx)
        }

        var results: [TransactionResult] = []
        for tx in transactions {
            let txResult = self.executeNextTransaction()!
            results.append(txResult)
        }

        self.commitBlock()
        return results
    }

    /// Deploys a given contract, and initializes it with the arguments.
    ///
    access(all) fun deployContract(
        name: String,
        code: String,
        account: Account,
        arguments: [AnyStruct]
    ): Error? {
        return self.backend.deployContract(
            name: name,
            code: code,
            account: account,
            arguments: arguments
        )
    }

    /// Set the configuration to be used by the blockchain.
    /// Overrides any existing configuration.
    ///
    access(all) fun useConfiguration(_ configuration: Configuration) {
        self.backend.useConfiguration(configuration)
    }

    /// Returns all the logs from the blockchain, up to the calling point.
    ///
    access(all) fun logs(): [String] {
        return self.backend.logs()
    }

    /// Returns the service account of the blockchain. Can be used to sign
    /// transactions with this account.
    ///
    access(all) fun serviceAccount(): Account {
        return self.backend.serviceAccount()
    }

    /// Returns all events emitted from the blockchain.
    ///
    access(all) fun events(): [AnyStruct] {
        return self.backend.events(nil)
    }

    /// Returns all events emitted from the blockchain,
    /// filtered by type.
    ///
    access(all) fun eventsOfType(_ type: Type): [AnyStruct] {
        return self.backend.events(type)
    }

    /// Resets the state of the blockchain to the given height.
    ///
    access(all) fun reset(to height: UInt64) {
        self.backend.reset(to: height)
    }

    /// Moves the time of the blockchain by the given delta,
    /// which should be passed in the form of seconds.
    ///
    access(all) fun moveTime(by delta: Fix64) {
        self.backend.moveTime(by: delta)
    }
}
```

The `BlockchainBackend` provides the actual functionality of the blockchain.

```cadence
/// BlockchainBackend is the interface to be implemented by the backend providers.
///
access(all) struct interface BlockchainBackend {

    access(all) fun executeScript(_ script: String, _ arguments: [AnyStruct]): ScriptResult

    access(all) fun createAccount(): Account

    access(all) fun addTransaction(_ tx: Transaction)

    access(all) fun executeNextTransaction(): TransactionResult?

    access(all) fun commitBlock()

    access(all) fun deployContract(
        name: String,
        code: String,
        account: Account,
        arguments: [AnyStruct]
    ): Error?

    access(all) fun useConfiguration(_ configuration: Configuration)

    access(all) fun logs(): [String]

    access(all) fun serviceAccount(): Account

    access(all) fun events(_ type: Type?): [AnyStruct]

    access(all) fun reset(to height: UInt64)

    access(all) fun moveTime(by delta: Fix64)
}

```

### Creating a blockchain

A new blockchain instance can be created using the `Test.newEmulatorBlockchain` method.
It returns a `Blockchain` which is backed by a new [Flow Emulator](https://developers.flow.com/tools/emulator) instance.

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()
```

### Creating accounts

It may be necessary to create accounts during tests for various reasons, such as for deploying contracts, signing transactions, etc.
An account can be created using the `createAccount` function.

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()
access(all) let account = blockchain.createAccount()

access(all) fun testExample() {
    log(account.address)
}
```

Running the above command, from the command-line, we would get:

```bash
flow test tests/test_sample_usage.cdc
3:31PM DBG LOG: 0x01cf0e2f2f715450

Test results: "tests/test_sample_usage.cdc"
- PASS: testExample
```

The returned account consists of the `address` of the account, and a `publicKey` associated with it.

```cadence
/// Account represents info about the account created on the blockchain.
///
access(all) struct Account {
    access(all) let address: Address
    access(all) let publicKey: PublicKey

    init(address: Address, publicKey: PublicKey) {
        self.address = address
        self.publicKey = publicKey
    }
}
```

### Executing scripts

Scripts can be run with the `executeScript` function, which returns a `ScriptResult`.
The function takes script-code as the first argument, and the script-arguments as an array as the second argument.

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()

access(all) fun testExample() {
    let code = "access(all) fun main(name: String): String { return \"Hello, \".concat(name) }"
    let args = ["Peter"]

    let scriptResult = blockchain.executeScript(code, args)

    // Assert that the script was successfully executed.
    Test.expect(scriptResult, Test.beSucceeded())

    // returnValue has always the type `AnyStruct`,
    // so we need to type-cast accordingly.
    let returnValue = scriptResult.returnValue! as! String

    Test.assertEqual("Hello, Peter", returnValue)
}
```

The script result consists of the `status` of the script execution, and a `returnValue` if the script execution was
successful, or an `error` otherwise (see [errors](#errors) section for more details on errors).

```cadence
/// The result of a script execution.
///
access(all) struct ScriptResult {
    access(all) let status: ResultStatus
    access(all) let returnValue: AnyStruct?
    access(all) let error: Error?

    init(status: ResultStatus, returnValue: AnyStruct?, error: Error?) {
        self.status = status
        self.returnValue = returnValue
        self.error = error
    }
}
```

### Executing transactions

A transaction must be created with the transaction code, a list of authorizes,
a list of signers that would sign the transaction, and the transaction arguments.

```cadence
/// Transaction that can be submitted and executed on the blockchain.
///
access(all) struct Transaction {
    access(all) let code: String
    access(all) let authorizers: [Address]
    access(all) let signers: [Account]
    access(all) let arguments: [AnyStruct]

    init(code: String, authorizers: [Address], signers: [Account], arguments: [AnyStruct]) {
        self.code = code
        self.authorizers = authorizers
        self.signers = signers
        self.arguments = arguments
    }
}
```

The number of authorizers must match the number of `AuthAccount` arguments in the `prepare` block of the transaction.

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()
access(all) let account = blockchain.createAccount()

// There are two ways to execute the created transaction.

access(all) fun testExample() {
    let tx = Test.Transaction(
        code: "transaction { prepare(acct: AuthAccount) {} execute{} }",
        authorizers: [account.address],
        signers: [account],
        arguments: [],
    )

    // Executing the transaction immediately
    // This may fail if the current block contains
    // transactions that have not being executed yet.
    let txResult = blockchain.executeTransaction(tx)

    Test.expect(txResult, Test.beSucceeded())
}

access(all) fun testExampleTwo() {
    let tx = Test.Transaction(
        code: "transaction { prepare(acct: AuthAccount) {} execute{} }",
        authorizers: [account.address],
        signers: [account],
        arguments: [],
    )

    // Add to the current block
    blockchain.addTransaction(tx)

    // Execute the next transaction in the block
    let txResult = blockchain.executeNextTransaction()!

    Test.expect(txResult, Test.beSucceeded())
}
```

The result of a transaction consists of the status of the execution, and an `Error` if the transaction failed.

```cadence
/// The result of a transaction execution.
///
access(all) struct TransactionResult {
    access(all) let status: ResultStatus
    access(all) let error: Error?

    init(status: ResultStatus, error: Error?) {
        self.status = status
        self.error = error
    }
 }
```

### Commit block

`commitBlock` block will commit the current block, and will fail if there are any un-executed transactions in the block.

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()
access(all) let account = blockchain.createAccount()

access(all) fun testExample() {
    let tx = Test.Transaction(
        code: "transaction { prepare(acct: AuthAccount) {} execute{} }",
        authorizers: [account.address],
        signers: [account],
        arguments: [],
    )

    blockchain.commitBlock()

    blockchain.addTransaction(tx)

    // This will fail with `error: internal error: pending block with ID 1f9...c0b7740d2 cannot be committed before execution`
    blockchain.commitBlock()
}

```

### Deploying contracts

A contract can be deployed using the `deployContract` function of the `Blockchain`.

Suppose we have this contract (`Foo.cdc`):
```cadence
access(all) contract Foo {
    access(all) let msg: String

    init(_ msg: String) {
        self.msg = msg
    }

    access(all) fun sayHello(): String {
        return self.msg
    }
}
```

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()
access(all) let account = blockchain.createAccount()

access(all) fun testExample() {
    let contractCode = Test.readFile("Foo.cdc")
    let err = blockchain.deployContract(
        name: "Foo",
        code: contractCode,
        account: account,
        arguments: ["hello from args"],
    )

    Test.expect(err, Test.beNil())
}
```

An `Error` is returned if the contract deployment fails. Otherwise, a `nil` is returned.

### Configuring import addresses

A common pattern in Cadence projects is to define the imports as file locations and specify the addresses
corresponding to each network in the [Flow CLI configuration file](https://developers.flow.com/tools/flow-cli/flow.json/configuration.md#contracts).
When writing tests for such a project, it may also require to specify the addresses to be used during the tests as well.
However, during tests, since accounts are created dynamically and the addresses are also generated dynamically,
specifying the addresses statically in a configuration file is not an option.

Hence, the test framework provides a way to specify the addresses using the
`useConfiguration(_ configuration: Test.Configuration)` function in `Blockchain`.

The `Configuration` struct consists of a mapping of import locations to their addresses.

```cadence
/// Configuration to be used by the blockchain.
/// Can be used to set the address mapping.
///
access(all) struct Configuration {
    access(all) let addresses: {String: Address}

    init(addresses: {String: Address}) {
        self.addresses = addresses
    }
}
```

<Callout type="info">
The `Blockchain.useConfiguration` is a run-time alternative for
[statically defining contract addresses in the flow.json config file](https://developers.flow.com/tools/flow-cli/flow.json/configuration.md#advanced-format).
</Callout>

The configurations can be specified during the test setup as a best-practice.

e.g: Assume running a script that imports the above `Foo.cdc` contract.
The import location for the contract can be specified using the placeholder `"Foo"`.
This placeholder can be any unique string.

Suppose this script is saved in `say_hello.cdc`.
```cadence
import "Foo"

access(all) fun main(): String {
    return Foo.sayHello()
}
```

Then, before executing the script, the address mapping can be specified as follows:

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()
access(all) let account = blockchain.createAccount()

access(all) fun setup() {
    blockchain.useConfiguration(Test.Configuration({
        "Foo": account.address
    }))

    let contractCode = Test.readFile("Foo.cdc")
    let err = blockchain.deployContract(
        name: "Foo",
        code: contractCode,
        account: account,
        arguments: ["hello from args"],
    )

    Test.expect(err, Test.beNil())
}

access(all) fun testExample() {
    let script = Test.readFile("say_hello.cdc")
    let scriptResult = blockchain.executeScript(script, [])

    Test.expect(scriptResult, Test.beSucceeded())

    let returnValue = scriptResult.returnValue! as! String

    Test.assertEqual("hello from args", returnValue)
}
```

The subsequent operations on the blockchain (e.g: contract deployment, script/transaction execution) will resolve the
import locations to the provided addresses.

### Errors

An `Error` maybe returned when an operation (such as executing a script, executing a transaction, etc.) has failed.
It contains a message indicating why the operation failed.

```cadence
// Error is returned if something has gone wrong.
//
access(all) struct Error {
    access(all) let message: String

    init(_ message: String) {
        self.message = message
    }
}
```

An `Error` can be asserted against its presence or absence.

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()
access(all) let account = blockchain.createAccount()

access(all) fun testExample() {
    let script = Test.readFile("say_hello.cdc")
    let scriptResult = blockchain.executeScript(script, [])

    // If we expect a script to fail, we can use Test.beFailed() instead
    Test.expect(scriptResult, Test.beSucceeded())

    let tx = Test.Transaction(
        code: "transaction { prepare(acct: AuthAccount) {} execute{} }",
        authorizers: [account.address],
        signers: [account],
        arguments: [],
    )
    let txResult = blockchain.executeTransaction(tx)

    // If we expect a transaction to fail, we can use Test.beFailed() instead
    Test.expect(txResult, Test.beSucceeded())

    let err: Test.Error? = txResult.error

    if err != nil {
        log(err!.message)
    }
}
```

## Blockchain events

We can also assert that certain events were emitted from the blockchain, up to the latest block.

Suppose we have this contract (`Foo.cdc`):
```cadence
access(all) contract Foo {
    access(all) let msg: String

    access(all) event ContractInitialized(msg: String)

    init(_ msg: String) {
        self.msg = msg
        emit ContractInitialized(msg: self.msg)
    }

    access(all) fun sayHello(): String {
        return self.msg
    }
}
```

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()
access(all) let account = blockchain.createAccount()

access(all) fun setup() {
    blockchain.useConfiguration(Test.Configuration({
        "Foo": account.address
    }))

    let contractCode = Test.readFile("Foo.cdc")
    let err = blockchain.deployContract(
        name: "Foo",
        code: contractCode,
        account: account,
        arguments: ["hello from args"],
    )

    Test.expect(err, Test.beNil())

    // As of now, we have to construct the composite type by hand,
    // until the testing framework allows developers to import
    // contract types, e.g.:
    // let typ = Type<FooContract.ContractInitialized>()
    let typ = CompositeType("A.01cf0e2f2f715450.Foo.ContractInitialized")!
    let events = blockchain.eventsOfType(typ)
    Test.assertEqual(1, events.length)

    // We can also fetch all events emitted from the blockchain
    log(blockchain.events())
}
```

## Commonly used contracts

The commonly used contracts are already deployed on the blockchain, and can be imported without any
additional setup.

Suppose this script is saved in `get_type_ids.cdc`.
```cadence
import "FungibleToken"
import "FlowToken"
import "NonFungibleToken"
import "MetadataViews"
import "ViewResolver"
import "ExampleNFT"
import "NFTStorefrontV2"
import "NFTStorefront"

access(all) fun main(): [String] {
    return [
        Type<FlowToken>().identifier,
        Type<NonFungibleToken>().identifier,
        Type<MetadataViews>().identifier
    ]
}
```

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()

access(all) fun testExample() {
    let script = Test.readFile("get_type_ids.cdc")
    let scriptResult = blockchain.executeScript(script, [])

    Test.expect(scriptResult, Test.beSucceeded())

    let returnValue = scriptResult.returnValue! as! [String]

    let expected = [
        "A.0ae53cb6e3f42a79.FlowToken",
        "A.f8d6e0586b0a20c7.NonFungibleToken",
        "A.f8d6e0586b0a20c7.MetadataViews"
    ]

    Test.assertEqual(expected, returnValue)
}
```

## Reading from files

Writing tests often require constructing source-code of contracts/transactions/scripts in the test script.
Testing framework provides a convenient way to load programs from a local file, without having to manually construct
them within the test script.

```cadence
let contractCode = Test.readFile("./sample/contracts/FooContract.cdc")
```

`readFile` returns the content of the file as a string.

## Logging

The `log` function is available for usage both in test scripts, as well as contracts/scripts/transactions.

The `Blockchain.logs()` method aggregates all logs from contracts/scripts/transactions.

```cadence
import Test

access(all) let blockchain = Test.newEmulatorBlockchain()
access(all) let account = blockchain.createAccount()

access(all) fun testExample() {
    let tx = Test.Transaction(
        code: "transaction { prepare(acct: AuthAccount) {} execute{ log(\"in a transaction\") } }",
        authorizers: [account.address],
        signers: [account],
        arguments: [],
    )

    let txResult = blockchain.executeTransaction(tx)

    Test.expect(txResult, Test.beSucceeded())
    Test.assertEqual(["in a transaction"], blockchain.logs())
}
```

## Examples

This [repository](https://github.com/m-Peter/flow-code-coverage) contains some functional examples
that demonstrate most of the above features, both for contrived and real-world smart contracts.
It also contains a detailed explanation on using code coverage from within the testing framework.
---
title: Why Use Cadence?
sidebar_position: 2
---

## Security and Safety

Cadence provides security and safety guarantees that greatly simplify the development of secure smart contracts. As smart contracts often deal with valuable assets, Cadence provides the resource-oriented programming paradigm, which guarantees that assets can only exist in one location at a time, cannot be copied, and cannot be accidentally lost or deleted.
Cadence includes several language features which prevent entire classes of bugs.
These security and safety features allow smart contract developers to focus on the business logic of their contract instead of preventing accidents and attacks.

## Composability

Cadence enables composability. Resources (which are arbitrary user-defined data types) are stored directly in users’ accounts, and can flow freely between contracts: They can be passed as arguments to functions, returned from functions, or even combined in arbitrary data structures. This makes implementing business logic easier, more natural and promotes reuse of existing logic.

## Simplicity

Cadence’s syntax is inspired by popular modern general-purpose programming languages like [Swift](https://developer.apple.com/swift/), [Kotlin](https://kotlinlang.org/), and [Rust](https://www.rust-lang.org/), so developers will find the syntax and the semantics familiar.
Practical tooling, documentation, and examples enable developers to start creating programs quickly and effectively. Hundreds of developers were able to learn Cadence quickly and develop production-quality smart contracts with it shortly.
