---
title: 1. First Steps
---

In this tutorial, we will learn how to use smart contracts, switch accounts, and view account state.

## What is Cadence?

---

Cadence is a new smart contract programming language for use on the Flow Blockchain.
Cadence introduces new features to smart contract programming that help developers ensure that their code is safe, secure, clear, and approachable. Some of these features are:

- Type safety and a strong static type system
- Resource-oriented programming, a new paradigm that pairs linear types with object capabilities to create a secure and declarative model for digital ownership
  by ensuring that resources (and their associated assets) can only exist in one location at a time, cannot be copied, and cannot be accidentally lost or deleted
- Built-in pre-conditions and post-conditions for functions and [transactions](../language/transactions)
- The utilization of capability-based security, which enforces access control by requiring that access to objects
  is restricted to only the owner and those who have a valid reference to the object

Please see the [Cadence introduction](../index.md) for more information about the high level design of the language.

## What is the Flow Developer Playground?

---

The [Flow Playground](https://play.onflow.org) includes
an in-browser editor and emulator to experiment with Flow.
Using the Flow Playground, you can write Cadence smart contracts,
deploy them to a local Flow emulated blockchain, and submit transactions.

The Flow Playground should be compatible with any standard web browser,
but we recommend that you use Google Chrome with it,
because it has been tested and optimized for only the Chrome browser so far.

## Getting to know the Playground

The Playground contains everything you need to get familiar
with deploying Cadence smart contracts and interacting with transaction and scripts.

The Playground comes pre-loaded with contract and transaction templates
that correspond to each of the tutorials in the docs site.
To load the contracts from a specific tutorial, click the "Examples" link at the top of the Playground.
This opens up a menu with each tutorial.

When you click on one of these links, the tutorial will open in a new tab
and the contracts, transactions, and scripts will be loaded into the templates in the Playground for you to use.

## Accounts and Contracts

The Accounts section on the bottom left part of the screen is where the active accounts are listed and selected.
An account can have multiple smart contracts deployed to it, which will be covered later.
You can click on an account tab to view the contracts that are associated with that account in the main editor.

![Playground Intro](playground-intro.png)

When you have Cadence code open in the account editor that contains a contract,
you can click the deploy button in the bottom-right of the screen
to deploy that contract to the currently selected account.

![Deploy Contract](deploybox.png)

After a few seconds, the contract should deploy. In the accounts section, you should 
now see the name of the contract next to the selected account that you deployed too
and if you click on "Log" in the bottom section of the scrren, you should 
see a message in the console confirming that the contract was deployed and which account it was deployed to.

You can also select transactions and scripts from the left selection menu
and submit them to interact with your deployed smart contracts,
which will be covered in the Hello World tutorial.

This is just a small set of the things you can do with the Playground.
If you would like a more detailed explanation of the different Playground features, look at the Playground Manual.

## Resources

Each tutorial in this package uses several files containing transactions, contracts, and scripts.
All the code you need will be provided in the text of the tutorials for you to copy and paste,
or you can use the pre-generated tutorial setups in the Playground.

## Say Hello, World!

Now that you have the Flow Developer Playground running,
you can [create a smart contract](./02-hello-world.md) for Flow!---
archived: false
draft: false
title: 2. Hello World
description: A smart contract tutorial for Cadence.
date: 2022-05-10
meta:
  keywords:
    - tutorial
    - Flow
    - Cadence
    - Hello World
tags:
  - reference
  - cadence
  - tutorial
socialImageTitle: Hello World
socialImageDescription: Hello world smart contract image.
---

In this tutorial, we'll write and deploy our first smart contract!

<Callout type="success">
  Open the starter code for this tutorial in the Flow Playground: <br />
  <a
    href="https://play.onflow.org/af7aba31-dee9-4477-9e1d-7b46e958468e"
    target="_blank"
  >
    https://play.onflow.org/af7aba31-dee9-4477-9e1d-7b46e958468e
  </a>
  <br />
  The tutorial will ask you to take various actions to interact with this code.
</Callout>

<Callout type="info">
  Instructions that require you to take action are always included in a callout
  box like this one. These highlighted actions are all that you need to do to
  get your code running, but reading the rest is necessary to understand the
  language's design.
</Callout>

## What is a smart contract?

In regular terms, a contract is an agreement between two parties for some exchange of information or assets.
Normally, the terms of a contract are supervised and enforced by a trusted third party, such as a bank or a lawyer.

A smart contract is a computer program stored in a network like a blockchain
that verifies and executes the performance of a contract (like a lawyer does)
without the need for any trusted third party anywhere in the process, because the code itself is trusted.

Programs that run on blockchains are commonly referred to as smart contracts
because they mediate important functionality (such as currency)
without having to rely on a central authority (like a bank).

[Cadence is the resource-oriented programming language](../index.md)
for developing smart contracts on the Flow Blockchain.

This tutorial will walk you through an example of a smart contract that implements basic Cadence features,
including accounts, transactions, and signers.

Our "Hello World" smart contract will:

1. Create and initialize a smart contract with a single field of type `String`
2. Initialize the field with the phrase "Hello, World!"
3. Create a function in the contract that returns our greeting

We will deploy this contract in an account, then use a transaction to interact with the contract,
and finally discuss the role of signers in the transaction.

## Follow Along!

Before we get started if you'd prefer to learn from a video, feel free to join Kim
as she walks you through the basics of accounts, smart contracts, Cadence, transactions & more!

<iframe
  width="560"
  height="315"
  src="https://www.youtube.com/embed/pRz7EzrWchs"
  title="YouTube video player"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen
></iframe>

## How to Use Playground

For this tutorial, you'll be using the [Flow Playground](https://play.onflow.org/),
an interactive web interface that lets you write and run smart contracts in a test environment.
It also allows you to save and share your work with others so that you can test smart contracts collaboratively.

When you work with accounts in the Flow Playground, you start with five default accounts that you can change and reconfigure.
Each account in your environment has a unique address, and you can select an account in the left toolbar,
which will open up the contracts that are saved for that account. 
The `HelloWorld` contracts are loaded by default for each account
unless you load an existing playground project with other saved contracts.

For this tutorial, you'll be working with only the first account `0x01`

## Implementing Hello World

---

You will start by using a smart contract that contains a public function that returns `"Hello World!"`.

Like most other blockchains, the programming model in Flow is centered around accounts and transactions.
All state that persists permanently is stored in [accounts](../language/accounts)
and all accounts have the same core functionality. (users, smart contracts, data storage)

The interfaces to this state (the ways to interact with it, otherwise known as methods or functions) are also stored in accounts.
All code execution takes place within [transactions](../language/transactions),
which are blocks of code that are authorized and submitted by external users
to interact with the persistent state, which includes directly modifying account storage.

A smart contract is a collection of code (its functions) and data (its state) that lives in the contract area of an account in Flow.
Each account can have zero or more contracts and/or contract interfaces.
A contract can be freely added, removed, or updated (with some restrictions) by the owner of the account.
Now let's look at the `HelloWorld` contract that you'll be working through in this tutorial.

<Callout type="info">

If you haven't already, you'll need to follow this link to open a playground session with the Hello World contracts, transactions, and scripts pre-loaded:

{' '}
<a
  href="https://play.onflow.org/dbc06b40-d0b1-42da-9e0d-686bc9972e65"
  target="_blank"
>
  https://play.onflow.org/dbc06b40-d0b1-42da-9e0d-686bc9972e65
</a>

</Callout>

![Playground Intro](playground-intro.png)

<Callout type="info">

Open the Account `0x01` tab with the file called
`HelloWorld.cdc` in the Contract 1 space. <br />
`HelloWorld.cdc` should contain this code:

</Callout>

```cadence HelloWorld.cdc
// HelloWorld.cdc
//
pub contract HelloWorld {

    // Declare a public field of type String.
    //
    // All fields must be initialized in the init() function.
    pub let greeting: String

    // The init() function is required if the contract contains any fields.
    init() {
        self.greeting = "Hello, World!"
    }

    // Public function that returns our friendly greeting!
    pub fun hello(): String {
        return self.greeting
    }
}
```

The line `pub contract HelloWorld ` declares a contract that is accessible in all scopes (public).
It's followed by `pub let greeting: String` which declares a state constant (`let`) of type `String` that is accessible in all scopes(`pub`).

You would have used `var` to declare a variable, which means that the value
can be changed later on instead of remaining constant like with `let`.

You can use `access(all)` and the `pub` keyword interchangeably.
They are both examples of an access control specification that means an interface can be accessed in all scopes, but not written to in all scopes.
For more information about the different levels of access control permitted in Cadence, refer to the [Access Control section of the language reference](../language/access-control).

The `init()` section is called the initializer. It is a special function that only runs when the contract is first created.
Objects similar to contracts, such as other [composite types like structs or resources](../language/composite-types),
require that the `init()` function initialize any fields that are declared in a composite type.
In the above example, the initializer sets the `greeting` field to `"Hello, World!"` when the contract is initialized.

The last part of our `HelloWorld` contract is a public function called `hello()`.
This declaration returns a value of type `String`.
Anyone who imports this contract in their transaction or script can read the public fields,
use the public types, and call the public contract functions; i.e. the ones that have `pub` or `access(all)` specified.

Soon you'll deploy this contract to your account and run a transaction that calls its function, but first, let's look at what accounts and transactions are.

### Accounts and Transactions

---

#### What is an Account?

Each user has an account controlled by one or more private keys with configurable weight.
This means that support for accounts/wallets with [multiple controllers](https://www.coindesk.com/what-is-a-multisignature-crypto-wallet)
is built into the protocol by default.

An account is divided into two main areas:

1. The first area is the [contract area](../language/accounts).
   This is the area that stores smart contracts containing type definitions, fields, and functions that relate to common functionality.
   There is no limit to the number of smart contracts an account can store.
   This area cannot be directly accessed in a transaction unless the transaction is just returning (reading) a copy of the code deployed to an account.
   The owner of an account can directly add, remove, or update/overwrite contracts that are stored in it.

2. The second area is the account storage.
   This area is where an account stores the objects that they own.
   This is an important differentiator between Cadence and other languages,
   because in other languages, assets that accounts own is always stored in the centralized
   smart contract that defines the assets. In Cadence, each account stores its assets
   as objects directly in its own account storage.
   The account storage section also stores code that declares the capabilities
   for controlling how these stored objects can be accessed.
   We'll cover account storage in more detail in a later tutorial.

In this tutorial, we use the account with the address `0x01` to store our `HelloWorld` contract.
Outside the Playground context, account addresses on Flow are completely unique.

### Deploying Code

---

Now that you know what an account is in a Cadence context, you can deploy the `HelloWorld` contract to your account.

<Callout type="info">

Make sure that the account `0x01` tab is selected and that the
`HelloWorld.cdc` file is in the editor. <br />
Click the deploy button to deploy the contents of the editor to account `0x01`.

</Callout>

![Deploy Contract](deploybox.png)

You should see a log in the output area indicating that the deployment succeeded.

    `Deployed Contract To: 0x01`

You'll also see the name of the contract show up in the selected account tab underneath the number for the account.
This indicates that the `HelloWorld` contract has been deployed to the account.
You can always look at this tab to verify which contracts are in which accounts.
In the Flow Playground environment there can be any number of contracts for each account.
To create an additional contract, either open up one of the other ones or click the plus (+)
button next to the contracts section in the playground.

### Creating a Transaction

---

A [Transaction](../language/transactions) in Flow is defined as an arbitrary-sized block of Cadence code that is authorized by one or more accounts.
When an account authorizes a transaction, the code in that transaction has access to the authorizers' private storage.
An account authorizes a transaction by performing a cryptographic signature on the transaction with the account's private key,
which should only be accessible to the account owner. Therefore, authorizers are also known as signers.
In addition to being able to access the authorizer's private assets,
transactions can also read and call functions in public contracts, and access public domains in other users' accounts.
For this tutorial, we use a transaction to call our `hello()` function.

<Callout type="info">

Open the transaction named `Simple Transaction` <br />
`Simple Transaction` should contain this code:

</Callout>

```cadence SayHello.cdc
import HelloWorld from 0x01

transaction {

  prepare(acct: AuthAccount) {}

  execute {
    log(HelloWorld.hello())
  }
}

```

This transaction first imports our `HelloWorld` smart contract from the account `0x01`.
If you haven't deployed the smart contract from the account, the transaction won't have access to it and the import will fail.
This imports the entire contract code from `HelloWorld`, including type definitions and public functions,
so that the transaction can use them to interact with the `HelloWorld` contract in account `0x01`.

To import a smart contract from any other account, type this line at the top of your transaction:

```cadence
// Replace {ContractName} with the name of the contract you want to import
// and {Address} with the account you want to import it from
import {ContractName} from {Address}
```

Transactions are divided into two main phases, `prepare` and `execute`.

1. The `prepare` phase is required but we don't use it in this tutorial.
   We'll cover this phase in a later tutorial.
2. The `execute` phase is the main body of a transaction.
   It can call functions on external contracts and objects and perform operations on data that was initialized in the transaction.
   In this example, the `execute` phase calls `HelloWorld.hello()` which calls the `hello()` function in the `HelloWorld` contract and logs the result(`log(HelloWorld.hello())`) to the console.

<Callout type="info">

In the box at the bottom right of the editor, select Account `0x01` as the transaction signer. <br />
Click the `Send` button to submit the transaction

</Callout>

You should see something like this in the transaction results at the bottom of the screen:

```
Simple Transaction "Hello, World!"
```

Congratulations, you just executed your first Cadence transaction with the account `0x01` as the signer.

In this tutorial, you'll get the same result if you use different signers for the transaction
but later tutorials will use more complex examples that have different results depending on the signer.

## Reviewing HelloWorld

This tutorial covered an introduction to Cadence, including terms like accounts, transactions, and signers.
We implemented a smart contract that is accessible in all scopes.
The smart contract had a `String` field initialized with the value `Hello, World!` and a function to return (read) this value.
Next, we deployed this contract in an account and implemented a transaction to call the function in the smart contract and log the result to the console.
Finally, we used the account `0x01` as the signer for this transaction.

Now that you have completed the tutorial, you have the basic knowledge to write a simple Cadence program that can:

- Deploy a basic smart contract in an account
- Interact with the smart contract using a transaction
- Sign the transaction with one or multiple signers

Feel free to modify the smart contract to implement different functions,
experiment with the available [Cadence types](../language/values-and-types),
and write new transactions that execute multiple functions from your `HelloWorld` smart contract.
---
archived: false
draft: false
title: 3. Resource Contract Tutorial
description: An introduction to resources, capabilities, and account storage in Cadence
date: 2022-05-10
meta:
  keywords:
    - tutorial
    - Flow
    - Cadence
    - Resources
    - Capabilities
tags:
  - reference
  - cadence
  - tutorial
socialImageTitle: Cadence Resources
socialImageDescription: Resource smart contract image.
---

## Overview

<Callout type="success">
  Open the starter code for this tutorial in the Flow Playground: <br />
  <a
    href="https://play.onflow.org/b70199ae-6488-4e58-ae58-9f4ffecbd66a"
    target="_blank"
  >
    https://play.onflow.org/b70199ae-6488-4e58-ae58-9f4ffecbd66a
  </a>
  <br />
  The tutorial will ask you to take various actions to interact with this code.
</Callout>
<Callout type="info">
  Instructions that require you to take action are always included in a callout
  box like this one. These highlighted actions are all that you need to do to
  get your code running, but reading the rest is necessary to understand the
  language's design.
</Callout>

This tutorial builds on the previous `Hello World` tutorial.
Before beginning this tutorial, you should understand :

- [Accounts](../language/accounts)
- [Transactions](../language/transactions)
- Signers
- [Field types](../language/composite-types)

This tutorial will build on your understanding of accounts and how to interact with them by introducing resources.
Resources are one of Cadence's defining features.

In Cadence, resources are a composite type like a struct or a class, but with some special rules.
Here is an example definition of a resource:
```cadence
pub resource Money {
  pub let balance: Int

  init() {
    self.balance = 0
  }
}
```

See, it looks just like a regular `struct` definition! The difference is in the behavior.

Resources are useful when you want to model direct ownership.
Traditional structs or classes from other conventional programming languages are not an ideal way to represent ownership because they can be _copied_.
This means a coding error can easily result in creating multiple copies of the same asset, which breaks the scarcity requirements needed for these assets to have real value.
We have to consider loss and theft at the scale of a house, a car, or a bank account with millions of dollars, or a horse.
Resources, in turn, solve this problem by making creation, destruction, and movement of assets explicit.

In this tutorial, you will:

1. Deploy a contract that declares a resource
2. Save the resource into the account storage
3. Interact with the resource we created using a transaction

## Implementing a Contract with Resources

---

To interact with resources, you'll learn a few important concepts:

- Using the create keyword
- The move operator `<-`
- The [Account Storage API](../language/accounts#account-storage-api)

Let's start by looking at how to create a resource with the `create` keyword and the move operator `<-`.

You use the `create` keyword used to initialize a resource.
Resources **must** be created before you can use them.

The move operator `<-` is used to move a resource into a variable.
You cannot use the assignment operator `=` with resources,
so when you initialize a resource you will need to use the move operator `<-`.

<Callout type="info">

Open the Account `0x01` tab with file named `HelloWorldResource.cdc`. <br />
`HelloWorldResource.cdc` should contain the following code:

</Callout>

```cadence HelloWorldResource.cdc
pub contract HelloWorld {

    // Declare a resource that only includes one function.
    pub resource HelloAsset {

        // A transaction can call this function to get the "Hello, World!"
        // message from the resource.
        pub fun hello(): String {
            return "Hello, World!"
        }
    }

    // We're going to use the built-in create function to create a new instance
    // of the HelloAsset resource
    pub fun createHelloAsset(): @HelloAsset {
        return <-create HelloAsset()
    }

    init() {
        log("Hello Asset")
    }
}
```

<Callout type="info">

Deploy this code to account `0x01` using the `Deploy` button.

</Callout>

We start by declaring a new `HelloWorld` contract in account `0x01`, inside this new `HelloWorld` contract we:

1. Declare the resource `HelloAsset` with public scope `pub`
2. Declare the resource function `hello()` inside `HelloAsset` with public scope `pub`
3. Declare the contract function `createHelloAsset()` which `create`s a `HelloAsset` resource
4. The `createHelloAsset()` function uses the move operator (`<-`) to return the resource

This is another example of what we can do with a contract.
Cadence can declare type definitions within deployed contracts.
A type definition is simply a description of how a particular set of data is organized.
It **isn't** a copy or instance of that data on its own.
Any account can import these definitions and use them to create an object
that follows the imported definition or to interact with other objects of those types.

This contract that we just deployed declares a definition
for the `HelloAsset` resource and a function to create the resource.

Let's walk through this contract in more detail, starting with the resource.
Resources are one of the most important things that Cadence introduces to the smart contract design experience:

```cadence
pub resource HelloAsset {
    pub fun hello(): String {
        return "Hello, World!"
    }
}
```

### Resources

The key difference between a resource and a struct or class is the access scope for resources:

- Each instance of a resource can only exist in exactly one location and cannot be copied.
  Here, location refers to account storage, a temporary variable in a function, a storage field in a contract, etc.
- Resources must be explicitly moved from one location to another when accessed.
- Resources also cannot go out of scope at the end of function execution.
  They must be explicitly stored somewhere or destroyed.

These characteristics make it impossible to accidentally lose a resource from a coding mistake.

```cadence
init() {
// ...
```

All composite types like contracts, resources,
and structs can have an optional `init()` function that only runs when the object is initially created.
Cadence requires that all fields must be explicitly initialized,
so if the object has any fields,
this function has to be used to initialize them.

Contracts also have read and write access to the storage of the account that they are deployed to by using the built-in
[`self.account`](../language/contracts) object.
This is an [`AuthAccount` object](../language/accounts#authaccount)
that gives them access to many different functions to interact with the private storage of the account.

This contract's `init` function is simple, it logs the phrase `"Hello Asset"` to the console.


**A resource can only be created in the scope that it is defined in.**

This prevents anyone from being able to create arbitrary amounts of resource objects that others have defined.

### The Move Operator (`<-`)

In this example, we declared a function that can create `HelloAsset` resources:
```cadence
pub fun createHelloAsset(): @HelloAsset {
    return <-create HelloAsset()
}
```
The `@` symbol specifies that it is a resource of the type `HelloAsset`, which we defined in the contract.
This function uses the move operator to create a resource of type `HelloAsset` and return it.
To create a new resource object, we use the `create` keyword

Here we use the `<-` symbol. [This is the move operator](../language/resources#the-move-operator--).
The move operator `<-` replaces the assignment operator `=` in assignments that involve resources.
To make the assignment of resources explicit, the move operator `<-` must be used when:

- the resource is the initial value of a constant or variable,
- the resource is moved to a different variable in an assignment,
- the resource is moved to a function as an argument
- the resource is returned from a function.

When a resource is moved, the old location is invalidated, and the object moves into the context of the new location.

So if I have a resource in the variable `first_resource`, like so:

```cadence
// Note the `@` symbol to specify that it is a resource
var first_resource: @AnyResource <- create AnyResource()
```

and I want to assign it to a new variable, `second_resource`,
after I do the assignment, `first_resource` is invalid because the underlying resource has been moved to the new variable.

```cadence
var second_resource <- first_resource
// first_resource is now invalid. Nothing can be done with it
```

Regular assignments of resources are not allowed because assignments only copy the value.
Resources can only exist in one location at a time, so movement must be explicitly shown in the code by using the move operator `<-`.

### Create Hello Transaction

Now we're going to use a transaction to that calls the `createHelloAsset()` function
and saves a `HelloAsset` resource to the account's storage.

<Callout type="info">

Open the transaction named `Create Hello`.

<br />

`Create Hello` should contain the following code:

</Callout>

```cadence CreateHello.cdc
// Transaction1.cdc
// This transaction calls the createHelloAsset() function from the contract
// to create a resource, then saves the resource in account storage using the "save" method.
import HelloWorld from 0x01

transaction {

	prepare(acct: AuthAccount) {
        // Here we create a resource and move it to the variable newHello,
        // then we save it in the account storage
        let newHello <- HelloWorld.createHelloAsset()

        acct.save(<-newHello, to: /storage/HelloAssetTutorial)
    }

    // In execute, we log a string to confirm that the transaction executed successfully.
	execute {
        log("Saved Hello Resource to account.")
	}
}
```

Here's what this transaction does:

1. Import the `HelloWorld` definitions from account `0x01`
2. Uses the `createHelloAsset()` function to create a resource and move it to `newHello`
3. `save` the created resource in the account storage of the account that deployed this contract, at the path `/storage/HelloAssetTutorial`
4. `log` the text `HelloAsset created and stored` to the console.

This is our first transaction using the `prepare` phase!
The `prepare` phase is the only place that has access to the signing accounts'
[private `AuthAccount` object](../language/accounts#authaccount).
`AuthAccount` objects have many different methods that are used to interact with account storage.
You can see the documentation for all of these in the [account section of the language reference](../language/accounts#authaccount).
In this tutorial, we'll be using `AuthAccount` methods to save and load from `/storage/`.
The `prepare` phase can also create `/private/` and `/public/` links to the objects in `/storage/`,
called [capabilities](../language/capabilities) (more on these later).

By not allowing the execute phase to access account storage,
we can statically verify which assets and areas of the signers' storage a given transaction can modify.
Browser wallets and applications that submit transactions for users can use this to show what a transaction could alter,
giving users information about transactions that wallets will be executing for them,
and confidence that they aren't getting fed a malicious or dangerous transaction from an app or wallet.

Let's go over the transaction in more detail.
To create a `HelloAsset` resource, we accessed the function `createHelloAsset()` from our contract, and moved the
resource it created to the variable `newHello`.

```cadence
let newHello <- HelloWorld.createHelloAsset()
```

Next, we save the resource to the account storage.
We use the [account storage API](../language/accounts#account-storage-api) to interact with the account storage in Flow.
To save the resource, we'll be using the
[`save()`](../language/accounts#account-storage-api)
method from the account storage API to store the resource in the account at the path `/storage/HelloAssetTutorial`.

```cadence
acct.save(<-newHello, to: /storage/HelloAssetTutorial)
```
The first parameter to `save` is the object that is being stored,
and the `to` parameter is the path that the object is being stored at.
The path must be a storage path, so only the domain `/storage/` is allowed as the `to` parameter.

If there is already an object stored under the given path, the program aborts.
Remember, the Cadence type system ensures that a resource can never be accidentally lost.
When moving a resource to a field, into an array, into a dictionary, or into storage,
there is the possibility that the location already contains a resource.
Cadence forces the developer to handle the case of an existing resource so that it is not accidentally lost through an overwrite.

It is also very important when choosing the name of your paths to pick an identifier
that is very specific and unique to your project.
Currently, account storage paths are global, so there is a chance that projects could use the same storage paths,
**which could cause path conflicts**!
This could be a headache for you, so choose unique path names to avoid this problem.

Finally, in the execute phase we log the phrase `"Saved Hello Resource to account."` to the console.

```cadence
log("Saved Hello Resource to account.")
```

<Callout type="info">

Select account `0x01` as the only signer. Click the `Send` button to submit
the transaction.

</Callout>

You should see something like this:

```
"Saved Hello Resource to account."
```

<Callout type="info">

You can also try removing the line of code that saves `newHello` to storage.

<br />
You should see an error for `newHello` that says `loss of resource`.
This means that you are not handling the resource properly.
If you ever see this error in any of your programs,
it means there is a resource somewhere that is not being explicitly stored or destroyed, meaning the program is invalid.
<br />
Add the line back to make the transaction checks properly.
</Callout>

In this case, this is the first time we have saved anything with the selected account,
so we know that the storage spot at `/storage/HelloAssetTutorial` is empty.
In real applications, we would likely perform necessary checks and actions with the location path we are storing in
to make sure we don't abort a transaction because of an accidental overwrite.

Now that you have executed the transaction, account `0x01` should have the newly created `HelloWorld.HelloAsset`
resource stored in its storage. You can verify this by clicking on account `0x01` on the bottom left.
This should open a view of the different contracts and objects in the account.
You should see this entry for the `HelloWorld` contract and the `HelloAsset` resource:

```
Deployed Contracts: 
[
  {
    "contract": "HelloWorld",
    "height": 6
  }
] 
Account Storage: 
{
    "Private": null,
    "Public": {},
    "Storage": {
        "HelloAssetTutorial": {
            "Fields": [
                39
            ],
            "ResourceType": {
                "Fields": [
                    {
                        "Identifier": "uuid",
                        "Type": {}
                    }
                ],
                "Initializers": null,
                "Location": {
                    "Address": "0x0000000000000005",
                    "Name": "HelloWorld",
                    "Type": "AddressLocation"
                },
                "QualifiedIdentifier": "HelloWorld.HelloAsset"
            }
        }
    }
}
```


### Load Hello Transaction

Now we're going to use a transaction to call the `hello()` method from the `HelloAsset` resource.

<Callout type="info">

Open the transaction named `Load Hello`.

<br />

`Load Hello` should contain the following code:

</Callout>

```cadence LoadHello.cdc
import HelloWorld from 0x01

// This transaction calls the "hello" method on the HelloAsset object
// that is stored in the account's storage by removing that object
// from storage, calling the method, and then putting it back in storage

transaction {

    prepare(acct: AuthAccount) {

        // Load the resource from storage, specifying the type to load it as
        // and the path where it is stored
        let helloResource <- acct.load<@HelloWorld.HelloAsset>(from: /storage/HelloAssetTutorial)

        // We use optional chaining (?) because the value in storage
        // may or may not exist, and thus is considered optional.
        log(helloResource?.hello())

        // Put the resource back in storage at the same spot
        // We use the force-unwrap operator `!` to get the value
        // out of the optional. It aborts if the optional is nil
        acct.save(<-helloResource!, to: /storage/HelloAssetTutorial)
    }
}
```

Here's what this transaction does:

1. Import the `HelloWorld` definitions from account `0x01`
2. Moves the `HelloAsset` object from storage to `helloResource` with the move operator
  and the `load` function from the [account storage API](../language/accounts#account-storage-api)
3. Calls the `hello()` function of the `HelloAsset` resource stored in `helloResource` and logs the result
4. Saves the resource in the account that we originally moved it from at the path `/storage/HelloAssetTutorial`

We're going to be using the `prepare` phase again to load the resource because it
has access to the signing accounts' [private `AuthAccount` object](../language/accounts#authaccount).

Let's go over the transaction in more detail.
To remove an object from storage, we use the `load` method from the [account storage API](../language/accounts#account-storage-api)

```cadence
let helloResource <- acct.load<@HelloWorld.HelloAsset>(from: /storage/HelloAssetTutorial)
```

If no object of the specified type is stored under the given path, the function returns nothing, or `nil`.
(This is an [Optional](../language/values-and-types#optionals),
a special type of data that we will cover later)

If the object at the given path is not of the specified type, Cadence will throw an error and the transaction will fail.

If there is an object of the specified type at the path,
the function returns that object and the account storage will no longer contain an object under the given path.

The type parameter for the object type to load is contained in `<>`.
In this case, we're basically saying that we expect to load a `HelloWorld.HelloAsset` resource object from this path.
A type argument for the parameter must be provided explicitly.
(Note the `@` symbol to specify that it is a resource)

The path `from` must be a storage path, so only the domain `/storage/` is allowed.

Next, we call the `hello()` function and log the output.

```cadence
log(helloResource?.hello())
```

We use `?` because the values in the storage are returned as [optionals](../language/values-and-types#optionals).
Optionals are values that are able to represent either the presence or the absence of a value.
Optionals have two cases: either there is a value of the specified type, or there is nothing (`nil`).
An optional type is declared using the `?` suffix.

```cadence
let newResource: HelloAsset?  // could either have a value of type `HelloAsset`
                              // or it could have a value of `nil`, which represents nothing
```

Optionals allow developers to account for `nil` cases more gracefully.
Here, we explicitly have to account for the possibility that the `helloResource` object we got with `load` is `nil`
(because `load` will return `nil` if there is nothing there to load).

Using `?` "unwraps" the optional, meaning that it gets the value if it is there, before calling `hello`,
but only if the value isn't `nil`. If the value is nil, the `?` returns `nil`.

Because `?` is used when calling the `hello` function, the function call only happens if the stored value is not `nil`.
In this case, the result of the `hello` function will be returned as an optional.
However, if the stored value was `nil`, the function call would not occur and the result is `nil`.

Next, we use `save` again to put the object back in storage in the same spot:

```cadence
acct.save(<-helloResource!, to: /storage/HelloAssetTutorial)
```

Remember, `helloResource` is still an optional, so we have to handle the possibility that it is `nil`.
Here, we use the force-unwrap operator (`!`). This operator gets the value in the optional if it contains a value,
and aborts the entire transaction if the object is `nil`.
It is a more risky way of dealing with optionals, but if your program is ever in a state where a value being `nil`
would defeat the purpose of the whole transaction, then the force-unwrap operator might be a good choice to deal with that.

Refer to [Optionals In Cadence](../language/values-and-types#optionals) to learn more about optionals and how they are used.

<Callout type="info">

Select account `0x01` as the only signer. Click the `Send` button to submit
the transaction.

</Callout>

You should see something like this:

```
"Hello, World!"
```

## Reviewing the Resource Contract

This tutorial covered an introduction to resources in Cadence,
using the account storage API and interacting with resources using transactions.

You implemented a smart contract that is accessible in all scopes.
The smart contract had a resource declared that implemented a function called `hello()`
that returns the string `"Hello, World!"`
and declared a function that can create a resource.

Next, you deployed this contract in an account and implemented a transaction to create the resource in the smart contract
and save it in the account `0x01` by using it as the signer for this transaction.

Finally, you used a transaction to move the `HelloAsset` resource from account storage, call the `hello` method,
and return it to the account storage.

Now that you have completed the tutorial, you have the basic knowledge to write a simple Cadence program that can:

- Implement a resource in a smart contract
- Save, move, and load resources using the account storage API and the move operator `<-`
- Use the `prepare` phase of a transaction to load resources from account storage

Feel free to modify the smart contract to create different resources,
experiment with the available [account storage API](../language/accounts#account-storage-api),
and write new transactions and scripts that execute different functions from your smart contract.
Have a look at the [resource reference page](../language/resources)
to find out more about what you can do with resources.

You're on the right track to building more complex applications with Cadence,
now is a great time to check out the [Cadence Best Practices document](../design-patterns)
and [Anti-patterns document](../anti-patterns)
as your applications become more complex.
---
archived: false
draft: false
title: 4. Capability Tutorial
description: An introduction to capabilities and how they interact with resources in Cadence
date: 2022-05-10
meta:
  keywords:
    - tutorial
    - Flow
    - Cadence
    - Resources
    - Capabilities
tags:
  - reference
  - cadence
  - tutorial
socialImageTitle: Cadence Resources
socialImageDescription: Capability smart contract image.
---
## Overview
<Callout type="success">
  Open the starter code for this tutorial in the Flow Playground. It is the same code that was in the previous tutorial: <br />
  <a
    href="https://play.onflow.org/a7f45bcd-8fda-45f6-b443-4b77302a1687"
    target="_blank"
  >
    https://play.onflow.org/a7f45bcd-8fda-45f6-b443-4b77302a1687
  </a>
  <br />
  The tutorial will ask you to take various actions to interact with this code.
</Callout>
<Callout type="info">
  Instructions that require you to take action are always included in a callout
  box like this one. These highlighted actions are all that you need to do to
  get your code running, but reading the rest is necessary to understand the
  language's design.
</Callout>

This tutorial builds on the [previous `Resource` tutorial](./03-resources.md).
Before beginning this tutorial, you should have an idea of how accounts,transactions,resources, and signers work with basic field types.
This tutorial will build on your understanding of accounts and resources.
You'll learn how to interact with resources using [capabilities](../language/capabilities)
In Cadence, resources are a composite type like a struct or a class, but with some special rules:
- Each instance of a resource can only exist in exactly one location and cannot be copied.
- Resources must be explicitly moved from one location to another when accessed.
- Resources also cannot go out of scope at the end of function execution, they must be explicitly stored somewhere or destroyed.

### Use-Cases for Capabilities and Scripts

Let's look at why you would want to use capabilities to expand access to resources in a real-world context.

A real user's account will contain functions and fields that need varying levels of access scope and privacy.
For example, if you're working on an app that allows users to exchange tokens.
While you definitely want to make a feature like withdrawing tokens
from an account only accessible by the owner of the tokens,
your app should allow anybody to deposit tokens.

Capabilities are what allows for this detailed control of owned assets.
They allow a user to indicate which of the functionality of their account
should be accessible to themselves, their trusted friends, and the public.

For example, a user might want to allow a friend of theirs to use some of their money to spend,
in this case, they could create a capability that gives the friend access to only this part of their account,
instead of having to give full control over.

Or if a user authenticates a trading app for the first time,
the can could ask the user for a capability object that allows
the app to access the trading functionality of a user's account so that
the app doesn't need to ask the user for a signature every time.

In this tutorial, you will:
1. Interact with the resource we created using transactions
2. Create capabilities to extend the resource access scope
3. Execute a script that interacts with the resource

## Accessing Resources with Capabilities

---
Before following this tutorial, you should have the `HelloWorld` contract deployed in account `0x01`,
just like in the [previous `Resource` contract tutorial](./03-resources.md).

<Callout type="info">

Open the Account `0x01` tab with file named `HelloWorldResource.cdc`. <br />
`HelloWorldResource.cdc` should contain the following code:

</Callout>

```cadence HelloWorldResource-2.cdc
pub contract HelloWorld {

    // Declare a resource that only includes one function.
    pub resource HelloAsset {

        // A transaction can call this function to get the "Hello, World!"
        // message from the resource.
        pub fun hello(): String {
            return "Hello, World!"
        }
    }

    // We're going to use the built-in create function to create a new instance
    // of the HelloAsset resource
    pub fun createHelloAsset(): @HelloAsset {
        return <-create HelloAsset()
    }

    init() {
        log("Hello Asset")
    }
}
```

<Callout type="info">

Deploy this code to account `0x01` using the `Deploy` button.

</Callout>

<Callout type="info">

Click on the `Create Hello` transaction and send it with `0x01` as the signer.

</Callout>

The contract and transaction above creates and stores the resource we'll be using in this tutorial.
For a more detailed breakdown of the contract, have a look at the [previous tutorial](./03-resources.md).

### Creating Capabilities and References to Stored Resources

---
You need explicit permission from the owner of an account to access its storage.
Capabilities allow an account owner to grant access to specific fields and functions
stored in their accounts. (Explained more below)

In this transaction, you create a new capability,
then use the `link` function to create a public link to your `HelloAsset` resource object.
Next you use that link to borrow a [reference](../language/references)
to the underlying object and call the `hello()` function.
A detailed explanation of what is happening in this transaction
is below the transaction code so, if you feel lost, keep reading!

<Callout type="info">

Open the transaction named `Create Link`.

<br />

`Create Link` should contain the following code:

</Callout>

```cadence CreateLink
import HelloWorld from 0x01

// This transaction creates a new capability
// for the HelloAsset resource in storage
// and adds it to the account's public area.
//
// Other accounts and scripts can use this capability
// to create a reference to the private object to be able to
// access its fields and call its methods.

transaction {
  prepare(account: AuthAccount) {

    // Create a public capability by linking the capability to
    // a `target` object in account storage.
    // The capability allows access to the object through an
    // interface defined by the owner.
    // This does not check if the link is valid or if the target exists.
    // It just creates the capability.
    // The capability is created and stored at /public/Hello, and is
    // also returned from the function.
    let capability = account.link<&HelloWorld.HelloAsset>(/public/HelloAssetTutorial, target: /storage/HelloAssetTutorial)

    // Use the capability's borrow method to create a new reference
    // to the object that the capability links to
    // We use optional chaining "??" to get the value because
    // result of the borrow could fail, so it is an optional.
    // If the optional is nil,
    // the panic will happen with a descriptive error message
    let helloReference = capability.borrow()
      ?? panic("Could not borrow a reference to the hello capability")

    // Call the hello function using the reference
    // to the HelloAsset resource.
    //
    log(helloReference.hello())
  }
}
```

<Callout type="info">

Ensure account `0x01` is still selected as a transaction signer. <br />
Click the `Send` button to send the transaction.

</Callout>

In this transaction, we use the prepare phase to:
1. Create a capability with the `link` method to the stored object `HelloWorld.HelloAsset` from the account path `/storage/HelloAssetTutorial`
2. Store the capability in the account path `/public/HelloAssetTutorial`
3. Use the `borrow` method to create a reference to the object we linked to called `helloReference`
4. Call the `hello()` function using the reference we created, `helloReference`

You should see `"Hello, World"` show up in the console again.
You might be confused that we were able to call a method on the `HelloAsset` object
without actually being directly in control of it!
It is also stored in the `/storage/` domain of the account, which should be private.

This is because we created a [**capability**](../language/capabilities) for the `HelloAsset` object.
Capabilities are kind of like pointers in other languages, but which much more fine-grained control.

### Capability Based Access Control

[Capabilities](../language/capabilities) allow the owners of objects
to specify what functionality of their private objects is available to others.
Think of it kind of like an account's API, if you're familiar with the concept.
The account owner has private objects stored in their storage, like their collectibles or their money,
but they might still want others to be able to see what collectibles they have in their account,
or they want to allow anyone to deposit more money of a certain currency in their account.
Since these objects are stored in private storage by default, the owner has to do something
to open up access to these while still retaining full control.
We create capabilities to accomplish this.

In our example, the owner of `HelloAsset` might still want to let other people call the `hello` method.
This is what capabilities are for. They represent a link to an object in an account's storage that has the type specified when the link is created.

It is important to remember that someone else who has this capability cannot move or destroy the object that the capability is linked to!
They can only access fields that the owner has explicitly declared in the type specification of the `link` method (described below).

Capabilities do not have any meaningful functionality on their own, but every capability has a `borrow` method,
which creates a reference to the object that the capability is linked to.
This reference is used to read fields or call methods on the object they reference
as if the owner of the reference had the actual object.

Note that this only allows access to fields and methods.
It does not allow copying, moving, or modifying the original object directly.

Let's break down what is happening in this transaction.

First, we create a public link to the private `HelloAsset` object in `/storage/`:

```cadence
let capability = account.link<&HelloWorld.HelloAsset>(/public/Hello, target: /storage/Hello)
```

The `link` method returns a capability that can be used to access this link.

The `HelloAsset` object is stored in `/storage/HelloAssetTutorial`, which only the account owner can access.
They want any user in the network to be able to call the `hello()` method. So they make a public capability in `/public/HelloAssetTutorial`.

To create a capability, we use the `AuthAccount.link` method to link a new capability to an object in storage.
The type contained in `<>` is the restricted reference type that the capability represents.
The capability says that whoever borrows a reference from this capability can only have access to the fields and methods
that are specified by the type in `<>`.
The specified type has to be a subtype of the type of the object being linked to,
meaning that it cannot contain any fields or functions that the linked object doesn't have.

A reference is referred to by the `&` symbol. Here, the capability references the `HelloAsset` object,
so we specify `<&HelloWorld.HelloAsset>` as the type, which gives access to everything in the `HelloAsset` object.

The first argument to the `link` function is the path where you want to store the link for the capability
and the `target` argument is the path to the object in storage that is to be linked to.
We always store links for capabilities in the `/private/` or `/public/` domains:
- We choose `/private/` if we only want to allow one or a small number of users to access it
- We choose `/public/` if we want any user in the network to be able to access it.

Capabilities always link to objects in the `/storage/` domain.

To borrow a reference to an object from the capability, we use the capability's `borrow` method.

```cadence
let helloReference = capability.borrow()
    ?? panic("Could not borrow a reference to the hello capability")
```

This method creates the reference as the type we specified in `<>` in the `link` function.
While borrowing the reference, we use
[optional chaining](../language/composite-types#accessing-fields-and-functions-of-composite-types-using-optional-chaining)
because the borrowing of the reference could fail.
The reference could be `nil` if the targeted storage slot is empty, is already borrowed,
or if the requested type exceeds what is allowed by the capability.
We panic with a descriptive error message so the caller can know better what went wrong.

We separate this process into capabilities and references to protect against reentrancy attacks.
A reentrancy attack is where a malicious actor could call into an object multiple times.
These attacks have plagued other smart contract languages.
Only one reference to an object can exist at a time, so this type of vulnerability isn't possible for objects in storage when you use Cadence.

Additionally, the owner of an object can effectively revoke capabilities they have created by moving the underlying object or destroying the link with the `unlink` method.
If the referenced object is moved or the link is destroyed, capabilities that have been created from that link are invalidated.

You can find more [detailed documentation about capabilities in the language reference.](../language/capabilities)

Now, anyone can call the `hello()` method on your `HelloAsset` object by borrowing a reference with your public capability in `/public/Hello`!
(Covered in the next section)

Lastly, we call the `hello()` method with our borrowed reference:

```cadence
// Call the hello function using the reference to the HelloAsset resource
log(helloReference.hello())
```

At the end of the transaction execution, the `helloReference` value is lost,
but that is ok because while it references a resource, it isn't the actual resource itself, so it is ok to lose it.

In the next section, we look at how capabilities can expand the access a script has to an account.

### Executing Scripts

---

A script is a very simple transaction type in Cadence that cannot perform
any writes to the blockchain and can only read the state of an account or contract.

To execute a script, write a function called `pub fun main()`.
You can click the execute script button to run the script.
The result of the script will be printed to the console output.

<Callout type="info">

Open the file `Script1.cdc`.

<br />

`Script1.cdc` should look like the following:

</Callout>

```cadence Script1.cdc
import HelloWorld from 0x01

pub fun main() {

    // Cadence code can get an account's public account object
    // by using the getAccount() built-in function.
    let helloAccount = getAccount(0x01)

    // Get the public capability from the public path of the owner's account
    let helloCapability = helloAccount.getCapability<&HelloWorld.HelloAsset>(/public/HelloAssetTutorial)

    // borrow a reference for the capability
    let helloReference = helloCapability.borrow()
        ?? panic("Could not borrow a reference to the hello capability")

    // The log built-in function logs its argument to stdout.
    //
    // Here we are using optional chaining to call the "hello"
    // method on the HelloAsset resource that is referenced
    // in the published area of the account.
    log(helloReference.hello())
}
```

Here's what this script does:
1. It fetches the `PublicAccount` object with `getAccount` and assigns it to the variable `helloAccount`
2. Uses the `getCapability` method to get the capability from the `Create Link` transaction
3. Borrows a reference for the capability using the `borrow` method and assigns it to `helloReference`
4. Logs the result of the `hello()` function from `helloReference` to the console.

```cadence
let helloAccount = getAccount(0x01)
```

The `PublicAccount` object is available to anyone in the network for every account,
but only has access to a small subset of functions that can be read from the `/public/` domain in an account.

Then, the script gets the capability that was created in `Create Link`.

```cadence
// Get the public capability from the public path of the owner's account
let helloCapability = helloAccount.getCapability(/public/HelloAssetTutorial)
```

To get a capability that is stored in an account, use the `account.getCapability()` function.
This function is available on `AuthAccount`s and on `PublicAccount`s.
`getCapability()` returns a capability for the link at the path that is specified,
also with the type that is specified.
It does not check if the target exists, so the borrow will fail if the capability is invalid.

After that, the script borrows a reference from the capability.

```cadence
let helloReference = helloCapability.borrow()
```

Then, the script uses the reference to call the `hello()` function and prints the result.

Let's execute the script to see it run correctly.

<Callout type="info">

Click the `Execute` button in the playground.

</Callout>

<img src="https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/playground-execute.png" />

You should see something like this print:

```
> "Hello, World"
> Result > "void"
```

Good work!
Your script ran successfully.

One other really cool feature of scripts is that since they can't actually change anything on chain,
they can access any accounts' private storage and objects.
This allows scripts even more power to understand the full state of the chain
and it is safe because they can't actually make any changes.
Also, everything on-chain is publicly readable anyway,
so it is a logical feature for a blockchain programming language to have.

A script can get the AuthAccount for an account address using the built-in getAuthAccount function:

```
fun getAuthAccount(_ address: Address): AuthAccount
```

See the [language reference](../language/accounts) for more information about accounts.

## Reviewing Capabilities

This tutorial expanded on the idea of resources in Cadence by expanding access scope to a resource using capabilities
and covering more account storage API use-cases.

You deployed a smart contract with a resource, then created a capability to grant access to that resource.
With the capability, you used the borrow method to create a reference and used the reference to call the
resource's `hello()` function.
Finally, you used a script to borrow the same capability and create a reference so that the script can
call the resource's `hello()` function. This is important because script's cannot access account storage
without using capabilities.

Now that you have completed the tutorial, you have the basic knowledge to write a simple Cadence program that can:
- Implement a resource in a smart contract
- Create capabilities to grant access to resources in an account
- Interact with resources using both signed transactions and scripts

Feel free to modify the smart contract to create different resources,
experiment with the available [account storage API](../language/accounts#account-storage-api),
and write new transactions and scripts that execute different functions from your smart contract.
Have a look at the [capability-based access control page](../language/capabilities)
to find out more about what you can do with capabilities.

You're on the right track to building more complex applications with Cadence,
now is a great time to check out the [Cadence Best Practices document](../design-patterns)
and [Anti-patterns document](../anti-patterns) as your applications become more complex.
---
archived: false
draft: false
title: 5.1 Non-Fungible Token Tutorial Part 1
description: An introduction to NFTs on Cadence
date: 2022-05-10
meta:
  keywords:
    - tutorial
    - Flow
    - NFT
    - Non-Fungible Tokens
    - Cadence
    - Resources
    - Capabilities
tags:
  - reference
  - NFT
  - Non-Fungible Token
  - cadence
  - tutorial
socialImageTitle: Non-Fungible Tokens in Cadence
socialImageDescription: NFT social image.
---

In this tutorial, we're going to deploy, store, and transfer **Non-Fungible Tokens (NFTs)**.

---

<Callout type="success">

Open the starter code for this tutorial in the Flow Playground:

<a href="https://play.onflow.org/a21087ad-b22c-4981-b49e-17297e916fa6" 
  target="_blank">
  https://play.onflow.org/a21087ad-b22c-4981-b49e-17297e916fa6
</a> 
<br/>
The tutorial will ask you to take various actions to interact with this code.
</Callout>
<Callout type="info">
Instructions that require you to take action are always included in a callout box like this one.
These highlighted actions are all that you need to do to get your code running,
but reading the rest is necessary to understand the language's design.
</Callout>

The NFT is an integral part of blockchain technology.
An NFT is a digital asset that represents ownership of a unique asset.
NFTs are also indivisible, you can't trade part of an NFT.
Possible examples of NFTs include:
CryptoKitties, Top Shot Moments, and tickets to a really fun concert.

Instead of being represented in a central ledger, like in most smart contract languages,
Cadence represents each NFT as a [resource object](../language/composite-types)
that users store in their accounts.
This allows NFTs to benefit from the resource ownership rules
that are enforced by the [type system](../language/values-and-types) -
resources can only have a single owner, they cannot be duplicated,
and they cannot be lost due to accidental or malicious programming errors.
These protections ensure that owners know that their NFT is safe and can represent an asset that has real value.

NFTs in a real-world context make it possible to trade assets and
prove who the owner of an asset is.
On Flow, NFTs are interoperable -
so the NFTs in an account can be used in different smart contracts
and app contexts.
All NFTs on Flow implement the [NFT Token Standard](https://github.com/onflow/flow-nft)
which defines a basic set of properties for NFTs on Flow.
This tutorial, will teach you a basic method of creating an NFT
to illustrate important language concepts.
After completing the NFT tutorials, readers should visit
[the NFT standard github repository](https://github.com/onflow/flow-nft)
to learn how full, production-ready NFTs are created.

To get you comfortable using NFTs, this tutorial will teach you to:

1. Deploy a basic NFT contract and type definitions.
2. Create an NFT object and store it in your account storage.
3. Create an NFT collection object to store multiple NFTs in your account.
4. Create an `NFTMinter` and use it to mint an NFT.
5. Create references to your collection that others can use to send you tokens.
6. Set up another account the same way.
7. Transfer an NFT from one account to another.
8. Use a script to see what NFTs are stored in each account's collection.

<Callout type="warning">
  It is important to remember that while this tutorial implements a working
  non-fungible token, it has been simplified for educational purposes and is not
  what any project should use in production. See the
  <a href="https://github.com/onflow/flow-nft" target="_blank">Flow Fungible Token standard</a>
  for the standard interface and example implementation.
</Callout>

**Before proceeding with this tutorial**, we highly recommend
following the instructions in [Getting Started](./01-first-steps.md),
[Hello, World!](./02-hello-world.md),
[Resources](./03-resources.md),
and [Capabilities](./04-capabilities.md)
to learn how to use the Playground tools and to learn the fundamentals of Cadence.
This tutorial will build on the concepts introduced in those tutorials.

## Non-Fungible Tokens on the Flow Emulator

---

In Cadence, each NFT is represented by a resource with an integer ID.
Resources are a perfect type to represent NFTs
because resources have important ownership rules that are enforced by the type system.
They can only have one owner, cannot be copied, and cannot be accidentally or maliciously lost or duplicated.
These protections ensure that owners know that their NFT is safe and can represent an asset that has real value.
For more information about resources, see the [resources tutorial](./03-resources.md)

An NFT is also usually represented by some sort of metadata like a name or a picture.
Historically, most of this metadata has been stored off-chain,
and the on-chain token only contains a URL or something similar that points to the off-chain metadata.
In Flow, this is possible, but the goal is to make it possible for all the metadata associated with a token to be stored on-chain.
This is out of the scope of this tutorial though.
This paradigm has been defined by the Flow community and the details are contained in
[the NFT metadata proposal.](https://github.com/onflow/flow/pull/636/files)

When users on Flow want to transact with each other,
they can do so peer-to-peer and without having to interact with a central NFT contract
by calling resource-defined methods in both users' accounts.

## Adding an NFT Your Account

We'll start by looking at a basic NFT contract, that adds an NFT to an account.
The contract will:

1. Create a smart contract with the NFT resource type.
2. Declare an ID field, a metadata field and an `init()` function in the NFT resource
3. Create an `init()` function for the contract that saves an NFT to an account

This contract relies on the [account storage API](../language/accounts#authaccount) to save NFTs in the
`AuthAccount` object.

---

<Callout type="info">

First, you'll need to follow this link to open a playground session
with the Non-Fungible Token contracts, transactions, and scripts pre-loaded:

<a href="https://play.onflow.org/ae2f2a83-6698-4e03-93cf-70d35627e28e" target="_blank">
  https://play.onflow.org/ae2f2a83-6698-4e03-93cf-70d35627e28e
</a>

</Callout>

<Callout type="info">

Open Account `0x01` to see `BasicNFT.cdc`.
`BasicNFT.cdc` should contain the following code:

</Callout>

```cadence BasicNFT.cdc
pub contract BasicNFT {

    // Declare the NFT resource type
    pub resource NFT {
        // The unique ID that differentiates each NFT
        pub let id: UInt64

        // String mapping to hold metadata
        pub var metadata: {String: String}

        // Initialize both fields in the init function
        init(initID: UInt64) {
            self.id = initID
            self.metadata = {}
        }
    }

    // Function to create a new NFT
    pub fun createNFT(id: UInt64): @NFT {
        return <-create NFT(initID: id)
    }

    // Create a single new NFT and save it to account storage
    init() {
        self.account.save<@NFT>(<-create NFT(initID: 1), to: /storage/BasicNFTPath)
    }
}
```

In the above contract, the NFT is a resource with an integer ID and a field for metadata.

Each NFT resource has a unique ID, so they cannot be combined or duplicated, unless the smart contract allows it.

Another unique feature of this design is that each NFT can contain its own metadata.
In this example, we use a simple `String`-to-`String` mapping, but you could imagine a [much more rich
version](https://github.com/onflow/flow-nft#nft-metadata)
that can allow the storage of complex file formats and other such data.

An NFT could even own other NFTs! This functionality is shown in the next tutorial.

In the contract's `init` function, we create a new NFT object and move it into the account storage.

```cadence
// put it in storage
self.account.save<@NFT>(<-create NFT(initID: 1), to: /storage/BasicNFTPath)
```

Here we access the `AuthAccount` object on the account the contract is deployed to and call its `save` method,
specifying `@NFT` as the type it is being saved as.
We also create the NFT in the same line and pass it as the first argument to `save`.
We save it to the `/storage` domain, where objects are meant to be stored.

<Callout type="info">

Deploy `NFTv1` by clicking the Deploy button in the top right of the editor.

</Callout>

You should now have an NFT in your account. Let's run a transaction to check.

<Callout type="info">

Open the `NFT Exists` transaction, select account `0x01` as the only signer, and send the transaction.<br/>
`NFT Exists` should look like this:

</Callout>

```cadence NFTExists.cdc
import BasicNFT from 0x01

// This transaction checks if an NFT exists in the storage of the given account
// by trying to borrow from it. If the borrow succeeds (returns a non-nil value), the token exists!
transaction {
    prepare(acct: AuthAccount) {
        if acct.borrow<&BasicNFT.NFT>(from: /storage/BasicNFTPath) != nil {
            log("The token exists!")
        } else {
            log("No token found!")
        }
    }
}
```

Here, we are trying to directly borrow a reference from the NFT in storage.
If the object exists, the borrow will succeed and the reference optional will not be `nil`,
but if the borrow fails, the optional will be `nil`.

You should see something that says `"The token exists!"`.

Great work! You have your first NFT in your account. Let's move it to another account!

## Performing a Basic Transfer

With these powerful assets in your account, you'll probably want to
move them around to other accounts. There are many ways to transfer objects in Cadence,
but we'll show the simplest one first.

This will also be an opportunity for you to try to write some of your own code!

<Callout type="info">

Open the `Basic Transfer` transaction.<br/>
`Basic Transfer` should look like this:

</Callout>

```cadence
import BasicNFT from 0x01

/// Basic transaction for two accounts to authorize
/// to transfer an NFT

transaction {
    prepare(signer1: AuthAccount, signer2: AuthAccount) {

        // Fill in code here to load the NFT from signer1
        // and save it into signer2's storage

    }
}
```

We've provided you with a blank transaction with two signers.

While a transaction is open, you can select one or more accounts to sign a transaction.
This is because, in Flow, multiple accounts can sign the same transaction,
giving access to their private storage. If multiple accounts are selected as signers,
this needs to be reflected in the signature of the transaction to show multiple signers,
as is shown in the "Basic Transfer" transaction.

All you need to do is `load()` the NFT from `signer1`'s storage and `save()` it
into `signer2`'s storage. You have used both of these operations before,
so this hopefully shouldn't be too hard to figure out.
Feel free to go back to earlier tutorials to see examples of these account methods.

You can also scroll down a bit to see the correct code:

---
---
---
---
---
---
---
---
---

Here is the correct code to load the NFT from one account and save it to another account.

```cadence
import BasicNFT from 0x01

/// Basic transaction for two accounts to authorize
/// to transfer an NFT

transaction {
    prepare(signer1: AuthAccount, signer2: AuthAccount) {

        // Load the NFT from signer1's account
        let nft <- signer1.load<@BasicNFT.NFT>(from: /storage/BasicNFTPath)
            ?? panic("Could not load NFT")

        // Save the NFT to signer2's account
        signer2.save<@BasicNFT.NFT>(<-nft, to: /storage/BasicNFTPath)

    }
}
```

<Callout type="info">

Select both Account `0x01` and Account `0x02` as the signers.<br/>
Click the "Send" button to send the transaction.

</Callout>

Now, the NFT should be stored in the storage of Account `0x02`!
You should be able to run the "NFT Exists" transaction again with `0x02` as the signer
to confirm that it is in their account.

## Enhancing the NFT Experience

Hopefully by now, you have an idea of how NFTs can be represented by resources in Cadence.
You might have noticed by now that if we required users
to remember different paths for each NFT and to use a multisig transaction for transfers,
we would not have a very friendly developer and user experience.

This is where the true utility of Cadence is shown.
Continue on to the [next tutorial](./05-non-fungible-tokens-2.md)
to find out how we can use capabilities and resources owning other resources
to enhance the ease of use and safety of our NFTs.

------
archived: false
draft: false
title: 5.2 Non-Fungible Token Tutorial Part 2
description: An introduction to NFTs on Cadence
date: 2022-05-10
meta:
  keywords:
    - tutorial
    - Flow
    - NFT
    - Non-Fungible Tokens
    - Cadence
    - Resources
    - Capabilities
tags:
  - reference
  - NFT
  - Non-Fungible Token
  - cadence
  - tutorial
socialImageTitle: Non-Fungible Tokens in Cadence
socialImageDescription: NFT social image.
---

In this tutorial, we're going to learn about
a full implementation for **Non-Fungible Tokens (NFTs)**.

---

<Callout type="success">
  Open the starter code for this tutorial in the Flow Playground:
  <a 
    href="https://play.onflow.org/f08e8e0d-d28e-4cbe-8d72-3afe2349c629" 
    target="_blank"
  >
    https://play.onflow.org/f08e8e0d-d28e-4cbe-8d72-3afe2349c629
  </a>
  <br/>
  The tutorial will ask you to take various actions to interact with this code.
</Callout>

<Callout type="info">
Instructions that require you to take action are always included in a callout box like this one.
These highlighted actions are all that you need to do to get your code running,
but reading the rest is necessary to understand the language's design.
</Callout>


## Storing Multiple NFTs in a Collection

In the [last tutorial](./05-non-fungible-tokens-1.md),
we created a simple `NFT` resource, stored in at a storage path,
then used a multi-sig transaction to transfer it from one account to another.

It should hopefully be clear that the setup and operations that we used
in the previous tutorial are not very scalable. Users need a way
to manage all of their NFTs from a single place.

There are some different ways we could accomplish this.

* We could store all of our NFTs in an array or dictionary, like so.
```cadence
// Define a dictionary to store the NFTs in
let myNFTs: @{Int: BasicNFT.NFT} = {}

// Create a new NFT
let newNFT <- BasicNFT.createNFT(id: 1)

// Save the new NFT to the dictionary
myNFTs[newNFT.id] <- newNFT

// Save the NFT to a new storage path
account.save(<-myNFTs, to: /storage/basicNFTDictionary)

```

## Dictionaries

This example uses a [**Dictionary**: a mutable, unordered collection of key-value associations](../language/values-and-types#dictionaries).

```cadence
// Keys are `Int`
// Values are `NFT`
pub let myNFTs: @{Int: NFT}
```

In a dictionary, all keys must have the same type, and all values must have the same type.
In this case, we are mapping integer (`Int`) IDs to `NFT` resource objects
so that there is one `NFT` for each `Int` that exists in the dictionary.

Dictionary definitions don't usually have the `@` symbol in the type specification,
but because the `myNFTs` mapping stores resources, the whole field also has to become a resource type,
which is why the field has the `@` symbol indicating that it is a resource type.

This means that all the rules that apply to resources apply to this type.

Using a dictionary to store our NFTs would solve the problem
of having to use different storage paths for each NFT, but it doesn't solve all the problems.
This types are relatively opaque and doesn't have much useful functionality on its own.

Instead, we can use a powerful feature of Cadence, resources owning other resources!
We'll define a new `Collection` resource as our NFT storage place
to enable more-sophisticated ways to interact with our NFTs.

The next contract we look at is called `ExampleNFT`, it's stored in Contract 1 in account `0x01`.

This contract expands on the `BasicNFT` we looked at by adding:
1. An `idCount` contract field that tracks unique NFT ids.
2. An `NFTReceiver` interface that exposes three public functions for the collection.
3. Declares a resource called `Collection` that implements the `NFTReceiver` interface
4. The `Collection` will declare fields and functions to interact with it,
including `ownedNFTs`, `init()`, `withdraw()`, `destroy()`, and other important functions
5. Next, the contract declares functions that create a new NFT (`mintNFT()`) and an empty collection (`createEmptyCollection()`)
7. Finally, the contract declares an `init()` function that initializes the path fields,
creates an empty collection as well as a reference to it,
and saves a minter resource to account storage.

This contract introduces a few new concepts, we'll look at the new contract, then break down all the new
concepts this contract introduces.

<Callout type="info">

Open Account `0x01` to see `ExampleNFT.cdc`.<br/>
Deploy the contract by clicking the Deploy button in the bottom right of the editor.<br/>
`ExampleNFT.cdc` should contain the code below.
It contains what was already in `BasicNFT.cdc` plus additional resource declarations in the contract body.

</Callout>

```cadence ExampleNFT.cdc
// ExampleNFT.cdc
//
// This is a complete version of the ExampleNFT contract
// that includes withdraw and deposit functionalities, as well as a
// collection resource that can be used to bundle NFTs together.
//
// Learn more about non-fungible tokens in this tutorial: https://developers.flow.com/cadence/tutorial/non-fungible-tokens-1

pub contract ExampleNFT {

    // Declare Path constants so paths do not have to be hardcoded
    // in transactions and scripts

    pub let CollectionStoragePath: StoragePath
    pub let CollectionPublicPath: PublicPath
    pub let MinterStoragePath: StoragePath

    // Tracks the unique IDs of the NFT
    pub var idCount: UInt64

    // Declare the NFT resource type
    pub resource NFT {
        // The unique ID that differentiates each NFT
        pub let id: UInt64

        // Initialize both fields in the init function
        init(initID: UInt64) {
            self.id = initID
        }
    }

    // We define this interface purely as a way to allow users
    // to create public, restricted references to their NFT Collection.
    // They would use this to publicly expose only the deposit, getIDs,
    // and idExists fields in their Collection
    pub resource interface NFTReceiver {

        pub fun deposit(token: @NFT)

        pub fun getIDs(): [UInt64]

        pub fun idExists(id: UInt64): Bool
    }

    // The definition of the Collection resource that
    // holds the NFTs that a user owns
    pub resource Collection: NFTReceiver {
        // dictionary of NFT conforming tokens
        // NFT is a resource type with an `UInt64` ID field
        pub var ownedNFTs: @{UInt64: NFT}

        // Initialize the NFTs field to an empty collection
        init () {
            self.ownedNFTs <- {}
        }

        // withdraw
        //
        // Function that removes an NFT from the collection
        // and moves it to the calling context
        pub fun withdraw(withdrawID: UInt64): @NFT {
            // If the NFT isn't found, the transaction panics and reverts
            let token <- self.ownedNFTs.remove(key: withdrawID)!

            return <-token
        }

        // deposit
        //
        // Function that takes a NFT as an argument and
        // adds it to the collections dictionary
        pub fun deposit(token: @NFT) {
            // add the new token to the dictionary with a force assignment
            // if there is already a value at that key, it will fail and revert
            self.ownedNFTs[token.id] <-! token
        }

        // idExists checks to see if a NFT
        // with the given ID exists in the collection
        pub fun idExists(id: UInt64): Bool {
            return self.ownedNFTs[id] != nil
        }

        // getIDs returns an array of the IDs that are in the collection
        pub fun getIDs(): [UInt64] {
            return self.ownedNFTs.keys
        }

        destroy() {
            destroy self.ownedNFTs
        }
    }

    // creates a new empty Collection resource and returns it
    pub fun createEmptyCollection(): @Collection {
        return <- create Collection()
    }

    // mintNFT
    //
    // Function that mints a new NFT with a new ID
    // and returns it to the caller
    pub fun mintNFT(): @NFT {

        // create a new NFT
        var newNFT <- create NFT(initID: self.idCount)

        // change the id so that each ID is unique
        self.idCount = self.idCount + 1

        return <-newNFT
    }

	init() {
        self.CollectionStoragePath = /storage/nftTutorialCollection
        self.CollectionPublicPath = /public/nftTutorialCollection
        self.MinterStoragePath = /storage/nftTutorialMinter

        // initialize the ID count to one
        self.idCount = 1

        // store an empty NFT Collection in account storage
        self.account.save(<-self.createEmptyCollection(), to: self.CollectionStoragePath)

        // publish a reference to the Collection in storage
        self.account.link<&{NFTReceiver}>(self.CollectionPublicPath, target: self.CollectionStoragePath)
	}
}
```

This smart contract more closely resembles a contract
that a project would actually use in production.

Any user who owns one or more `ExampleNFT` will have an instance
of this `@ExampleNFT.Collection` resource stored in their account.
This collection stores all of their NFTs in a dictionary that maps integer IDs to `@NFT`s.

Each collection has a `deposit` and `withdraw` function.
These functions allow users to follow the pattern of moving tokens in and out of
their collections through a standard set of functions.

When a user wants to store NFTs in their account,
they will create an empty `Collection` by calling the `createEmptyCollection()` function in the `ExampleNFT` smart contract.
This returns an empty `Collection` object that they can store in their account storage.

There are a few new features that we use in this example, so let's walk through them.

## The Resource Dictionary

We discussed above that when a dictionary stores a resource, it also becomes a resource!

This means that the collection has to
have special rules for how to handle its own resource.
You wouldn't want it getting lost by accident!

As we learned in the resource tutorial, you can destroy any resource
by explicity invoking the `destroy` command.

If the NFT `Collection` resource is destroyed with the `destroy` command,
it needs to know what to do with the resources it stores in the dictionary.
This is why resources that store other resources have to include
a `destroy` function that runs when `destroy` is called on it.
This destroy function has to either explicitly destroy the contained resources
or move them somewhere else. In this example, we destroy them.

```cadence
destroy() {
    destroy self.ownedNFTs
}
```

When the `Collection` resource is created, the `init` function is run
and must explicitly initialize all member variables.
This helps prevent issues in some smart contracts where uninitialized fields can cause bugs.
The init function can never run again after this.
Here, we initialize the dictionary as a resource type with an empty dictionary.

```cadence
init () {
  self.ownedNFTs <- {}
}
```

Another feature for dictionaries is the ability to get an array
of the keys of the dictionary using the built-in `keys` function.

```cadence
// getIDs returns an array of the IDs that are in the collection
pub fun getIDs(): [UInt64] {
    return self.ownedNFTs.keys
}
```

This can be used to iterate through the dictionary or just to see a list of what is stored.
As you can see, [a variable length array type](../language/values-and-types#arrays)
is declared by enclosing the member type within square brackets (`[UInt64]`).

## Resources Owning Resources

This NFT Collection example in `ExampleNFT.cdc` illustrates an important feature: resources can own other resources.

In the example, a user can transfer one NFT to another user.
Additionally, since the `Collection` explicitly owns the NFTs in it,
the owner could transfer all of the NFTs at once by just transferring the single collection.

This is an important feature because it enables numerous additional use cases.
In addition to allowing easy batch transfers,
this means that if a unique NFT wants to own another unique NFT,
like a CryptoKitty owning a hat accessory,
the Kitty literally stores the hat in its own storage and effectively owns it.
The hat belongs to the CryptoKitty that it is stored in,
and the hat can be transferred separately or along with the CryptoKitty that owns it.

This also brings up an interesting wrinkle in Cadence in regards to ownership.
In other ledger-based languages, ownership is indicated by account addresses.
Cadence is a fully object-oriented language, so ownership is indicated by where
an object is stored, not just an entry on a ledger.

Resources can own other resources, which means that with some interesting logic,
a resource can have more control over the resources it owns than the actual
person whose account it is stored in!

You'll encounter more fascinating implications of ownership and interoperability
like this as you get deeper into Cadence. 

Now, back to the tutorial!

## Restricting Access to the NFT Collection

In the NFT Collection, all the functions and fields are public,
but we do not want everyone in the network to be able to call our `withdraw` function.
This is where Cadence's second layer of access control comes in.
Cadence utilizes [capability security](../language/capabilities),
which means that for any given object, a user is allowed to access a field or method of that object if they either:

- Are the owner of the object
- Have a valid reference to that field or method (note that references can only be created from capabilities, and capabilities can only be created by the owner of the object)

When a user stores their NFT `Collection` in their account storage, it is by default not available for other users to access.
A user's authorized account object (`AuthAccount`, which gives access to private storage)
is only accessible by its owner. To give external accounts access to the `deposit` function,
the `getIDs` function, and the `idExists` function, the owner creates an interface that only includes those fields:

```cadence
pub resource interface NFTReceiver {

    pub fun deposit(token: @NFT)

    pub fun getIDs(): [UInt64]

    pub fun idExists(id: UInt64): Bool
}
```

Then, using that interface, they would create a link to the object in storage,
specifying that the link only contains the functions in the `NFTReceiver` interface.
This link creates a capability. From there, the owner can then do whatever they want with that capability:
they could pass it as a parameter to a function for one-time-use,
or they could put in the `/public/` domain of their account so that anyone can access it.
If a user tried to use this capability to call the `withdraw` function,
it wouldn't work because it doesn't exist in the interface that was used to create the capability.

The creation of the link and capability is seen in the `ExampleNFT.cdc` contract `init()` function

```cadence
// publish a reference to the Collection in storage
self.account.link<&{NFTReceiver}>(self.CollectionPublicPath, target: self.CollectionStoragePath)
```

The `link` function specifies that the capability is typed as `&AnyResource{NFTReceiver}` to only expose those fields and functions.
Then the link is stored in `/public/` which is accessible by anyone.
The link targets the `/storage/NFTCollection` (through the `self.CollectionStoragePath` contract field) that we created earlier.

Now the user has an NFT collection in their account `/storage/`,
along with a capability for it that others can use to see what NFTs they own and to send an NFT to them.

Let's confirm this is true by running a script!

## Run a Script

---

Scripts in Cadence are simple transactions that run without any account permissions and only read information from the blockchain.

<Callout type="info">

Open the script file named `Print 0x01 NFTs`.
`Print 0x01 NFTs` should contain the following code:

</Callout>

```cadence
import ExampleNFT from 0x01

// Print the NFTs owned by account 0x01.
pub fun main() {
    // Get the public account object for account 0x01
    let nftOwner = getAccount(0x01)

    // Find the public Receiver capability for their Collection
    let capability = nftOwner.getCapability<&{ExampleNFT.NFTReceiver}>(ExampleNFT.CollectionPublicPath)

    // borrow a reference from the capability
    let receiverRef = capability.borrow()
            ?? panic("Could not borrow receiver reference")

    // Log the NFTs that they own as an array of IDs
    log("Account 1 NFTs")
    log(receiverRef.getIDs())
}
```

<Callout type="info">

Execute `Print 0x01 NFTs` by clicking the Execute button in the top right of the editor box.<br/>
This script prints a list of the NFTs that account `0x01` owns.

</Callout>

Because account `0x01` currently doesn't own any in its collection, it will just print an empty array:

```
"Account 1 NFTs"
[]
Result > "void"
```

If the script cannot be executed, it probably means that the NFT collection hasn't been stored correctly in account `0x01`.
If you run into issues, make sure that you deployed the contract in account `0x01` and that you followed the previous steps correctly.

## Mint and Distribute Tokens

---

One way to create NFTs is by having an admin mint new tokens and send them to a user.
For the purpose of learning, we are simply implementing minting as a public function here.
Normally, most would implement restricted minting by having an NFT Minter resource.
This would restrict minting, because the owner of this resource is the only one that can mint tokens.

You can see an example of this in the [Marketplace tutorial](./08-marketplace-compose.md).

<Callout type="info">

Open the file named `Mint NFT`.
Select account `0x01` as the only signer and send the transaction.<br/>
This transaction deposits the minted NFT into the account owner's NFT collection:

</Callout>

```cadence MintNFT.cdc
import ExampleNFT from 0x01

// This transaction allows the Minter account to mint an NFT
// and deposit it into its collection.

transaction {

    // The reference to the collection that will be receiving the NFT
    let receiverRef: &{ExampleNFT.NFTReceiver}

    prepare(acct: AuthAccount) {
        // Get the owner's collection capability and borrow a reference
        self.receiverRef = acct.getCapability<&{ExampleNFT.NFTReceiver}>(ExampleNFT.CollectionPublicPath)
            .borrow()
            ?? panic("Could not borrow receiver reference")
    }

    execute {
        // Use the minter reference to mint an NFT, which deposits
        // the NFT into the collection that is sent as a parameter.
        let newNFT <- ExampleNFT.mintNFT()

        self.receiverRef.deposit(token: <-newNFT)

        log("NFT Minted and deposited to Account 1's Collection")
    }
}
```

<Callout type="info">

Reopen `Print 0x01 NFTs` and execute the script.
This prints a list of the NFTs that account `0x01` owns.

</Callout>

```cadence Print0x01NFTs.cdc
import ExampleNFT from 0x01

// Print the NFTs owned by account 0x01.
pub fun main() {
    // Get the public account object for account 0x01
    let nftOwner = getAccount(0x01)

    // Find the public Receiver capability for their Collection
    let capability = nftOwner.getCapability<&{ExampleNFT.NFTReceiver}>(ExampleNFT.CollectionPublicPath)

    // borrow a reference from the capability
    let receiverRef = capability.borrow()
            ?? panic("Could not borrow receiver reference")

    // Log the NFTs that they own as an array of IDs
    log("Account 1 NFTs")
    log(receiverRef.getIDs())
}

```

You should see that account `0x01` owns the NFT with `id = 1`

```
"Account 1 NFTs"
[1]
```

## Transferring an NFT

Before we are able to transfer an NFT to another account, we need to set up that account
with an NFTCollection of their own so they are able to receive NFTs.

<Callout type="info">

Open the file named `Setup Account` and submit the transaction, using account `0x02` as the only signer.

</Callout>

```cadence SetupAccount.cdc
import ExampleNFT from 0x01

// This transaction configures a user's account
// to use the NFT contract by creating a new empty collection,
// storing it in their account storage, and publishing a capability
transaction {
    prepare(acct: AuthAccount) {

        // Create a new empty collection
        let collection <- ExampleNFT.createEmptyCollection()

        // store the empty NFT Collection in account storage
        acct.save<@ExampleNFT.Collection>(<-collection, to: ExampleNFT.CollectionStoragePath)

        log("Collection created for account 2")

        // create a public capability for the Collection
        acct.link<&{ExampleNFT.NFTReceiver}>(ExampleNFT.CollectionPublicPath, target: ExampleNFT.CollectionStoragePath)

        log("Capability created")
    }
}
```

Account `0x02` should now have an empty `Collection` resource stored in its account storage.
It has also created and stored a capability to the collection in its `/public/` domain.

<Callout type="info">

Open the file named `Transfer`, select account `0x01` as the only signer, and send the transaction.<br/>
This transaction transfers a token from account `0x01` to account `0x02`.

</Callout>

```cadence Transfer.cdc
import ExampleNFT from 0x01

// This transaction transfers an NFT from one user's collection
// to another user's collection.
transaction {

    // The field that will hold the NFT as it is being
    // transferred to the other account
    let transferToken: @ExampleNFT.NFT

    prepare(acct: AuthAccount) {

        // Borrow a reference from the stored collection
        let collectionRef = acct.borrow<&ExampleNFT.Collection>(from: ExampleNFT.CollectionStoragePath)
            ?? panic("Could not borrow a reference to the owner's collection")

        // Call the withdraw function on the sender's Collection
        // to move the NFT out of the collection
        self.transferToken <- collectionRef.withdraw(withdrawID: 1)
    }

    execute {
        // Get the recipient's public account object
        let recipient = getAccount(0x02)

        // Get the Collection reference for the receiver
        // getting the public capability and borrowing a reference from it
        let receiverRef = recipient.getCapability<&{ExampleNFT.NFTReceiver}>(ExampleNFT.CollectionPublicPath)
            .borrow()
            ?? panic("Could not borrow receiver reference")

        // Deposit the NFT in the receivers collection
        receiverRef.deposit(token: <-self.transferToken)

        log("NFT ID 1 transferred from account 1 to account 2")
    }
}
```

Now we can check both accounts' collections to make sure that account `0x02` owns the token and account `0x01` has nothing.

<Callout type="info">

Execute the script `Print all NFTs` to see the tokens in each account:

</Callout>

```cadence Script2.cdc
import ExampleNFT from 0x01

// Print the NFTs owned by accounts 0x01 and 0x02.
pub fun main() {

    // Get both public account objects
    let account1 = getAccount(0x01)
	let account2 = getAccount(0x02)

    // Find the public Receiver capability for their Collections
    let acct1Capability = account1.getCapability(ExampleNFT.CollectionPublicPath)
    let acct2Capability = account2.getCapability(ExampleNFT.CollectionPublicPath)

    // borrow references from the capabilities
    let receiver1Ref = acct1Capability.borrow<&{ExampleNFT.NFTReceiver}>()
        ?? panic("Could not borrow account 1 receiver reference")
    let receiver2Ref = acct2Capability.borrow<&{ExampleNFT.NFTReceiver}>()
        ?? panic("Could not borrow account 2 receiver reference")

    // Print both collections as arrays of IDs
    log("Account 1 NFTs")
    log(receiver1Ref.getIDs())

    log("Account 2 NFTs")
    log(receiver2Ref.getIDs())
}
```

You should see something like this in the output:

```
"Account 1 NFTs"
[]
"Account 2 NFTs"
[1]
```

Account `0x02` has one NFT with ID=1 and account `0x01` has none.
This shows that the NFT was transferred from account `0x01` to account `0x02`.

<img src="https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/accounts-nft-storage.png" />

Congratulations, you now have a working NFT!

## Putting It All Together

---

This was only a basic example how a NFT might work on Flow.
Please refer to the [Flow NFT Standard repo](https://github.com/onflow/flow-nft)
for information about the official Flow NFT standard and an example implementation of it.

## Fungible Tokens

---

Now that you have a working NFT, you will probably want to be able to trade it. For that you are going to need to
understand how fungible tokens work on Flow, so go ahead and move to the next tutorial!
---
title: 6. Fungible Tokens
---

In this tutorial, we're going to deploy, store, and transfer fungible tokens.

---

<Callout type="success">
  Open the starter code for this tutorial in the Flow Playground:
  <br />
  <a
    href="https://play.onflow.org/e63bfce9-3324-4385-9542-626845ae0363"
    target="_blank"
  >
    https://play.onflow.org/e63bfce9-3324-4385-9542-626845ae0363
  </a>
  <br />
  The tutorial will ask you to take various actions to interact with this code.
</Callout>

<Callout type="info">
  Instructions that require you to take action are always included in a callout
  box like this one. These highlighted actions are all that you need to do to
  get your code running, but reading the rest is necessary to understand the
  language's design.
</Callout>

## Follow Along!
Developer advocate Kim dives deep on an array of topics, building on top of the information she shared in the Hello World tutorial. Learn core concepts such as creating a fungible token smart contract by using resources, resource interfaces, and using transactions to mint and transfer tokens!

<iframe width="560" height="315" src="https://www.youtube.com/embed/DInibYmxUsc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

Some of the most popular contract classes on blockchains today are fungible tokens.
These contracts create homogeneous tokens that can be transferred to other users and spent as currency (e.g., ERC-20 on Ethereum).

In traditional software and smart contracts, balances for each user are tracked by a central ledger, such as a dictionary:

```cadence
// DO NOT USE THIS CODE FOR YOUR PROJECT
contract LedgerToken {
    // Tracks every user's balance
    access(contract) let balances: {Address: UFix64}

    // Transfer tokens from one user to the other
    // by updating their balances in the central ledger
    access(all) fun transfer(from: Address, to: Address, amount: UFix64) {
        balances[from] = balances[from] - amount
        balances[to] = balances[to] + amount
    }
}
```

With Cadence, we use the new resource-oriented paradigm to implement fungible tokens and avoid using a central ledger
because there are inherent problems with using a central ledger
that are detailed in [the Fungible Tokens section below.](#fungible-tokens-an-in-depth-exploration)

### Flow Network Token

In Flow, the native network token
[(FLOW) is implemented as a normal fungible token smart contract](https://github.com/onflow/flow-core-contracts/blob/master/contracts/FlowToken.cdc)
using a smart contract similar to the one in this tutorial.

There are special transactions and hooks that allow it to be used for transaction execution fees, storage fees, and staking,
but besides that, developers and users are able to treat it and use it just like any other token in the network!

<Callout type="warning">
  It is important to remember that while this tutorial implements a working
  fungible token, it has been simplified for educational purposes and is not
  what any project should use in production. See the
  <a href="https://github.com/onflow/flow-ft" target="_blank">Flow Fungible Token standard</a>
  for the standard interface and example implementation.
</Callout>

We're going to take you through these steps to get comfortable with the fungible token:

1. Deploy the fungible token contract to account `0x01`
2. Create a fungible token object and store it in your account storage.
3. Create a reference to your tokens that others can use to send you tokens.
4. Set up another account the same way.
5. Transfer tokens from one account to another.
6. Use a script to read the accounts' balances.

**Before proceeding with this tutorial**, we recommend following the instructions in [Getting Started](./01-first-steps.md)
and [Hello, World!](./02-hello-world.md) to learn the basics of the language and the playground.

# Fungible Tokens on the Flow Emulator

---

<Callout type="info">
  First, you'll need to follow this link to open a playground session with the
  Fungible Token contracts, transactions, and scripts pre-loaded:{" "}
  <a
    href="https://play.onflow.org/e63bfce9-3324-4385-9542-626845ae0363"
    target="_blank"
  >
    https://play.onflow.org/e63bfce9-3324-4385-9542-626845ae0363
  </a>
</Callout>

<Callout type="info">

Open the account `0x01` tab to see the file named
`BasicToken.cdc`. `BasicToken.cdc` should contain the full code for the
fungible token, which provides the core functionality to store fungible tokens
in your account and transfer to and accept tokens from other users.

</Callout>

The concepts involved in implementing a fungible token in Cadence can be unfamiliar at first.
For an in-depth explanation of this functionality and code, continue reading the next section.

Or, if you'd like to go immediately into deploying it and using it in the playground,
you can skip to the [Interacting with the Fungible Token](#interacting-with-the-fungible-token-in-the-flow-playground) section of this tutorial.

# Fungible Tokens: An In-Depth Exploration

---

How Flow implements fungible tokens is different from other programming languages. As a result:

- Ownership is decentralized and does not rely on a central ledger
- Bugs and exploits present less risk for users and less opportunity for attackers
- There is no risk of integer underflow or overflow
- Assets cannot be duplicated, and it is very hard for them to be lost, stolen, or destroyed
- Code can be composable
- Rules can be immutable
- Code is not unintentionally made public

## Decentralizing Ownership

---

Instead of using a central ledger system, Flow ties ownership to each account via a new paradigm for asset ownership.
The example below showcases how Solidity (the smart contract language for the Ethereum Blockchain, among others)
implements fungible tokens, with only the code for storage and transferring tokens shown for brevity.

```solidity ERC20.sol
contract ERC20 {
    // Maps user addresses to balances, similar to a dictionary in Cadence
    mapping (address => uint256) private _balances;

    function _transfer(address sender, address recipient, uint256 amount) {
        // ensure the sender has a valid balance
        require(_balances[sender] >= amount);

        // subtract the amount from the senders ledger balance
        _balances[sender] = _balances[sender] - amount;

        // add the amount to the recipient’s ledger balance
        _balances[recipient] = _balances[recipient] + amount
    }
}
```

As you can see, Solidity uses a central ledger system for its fungible tokens. There is one contract that manages the state of the tokens
and every time that a user wants to do anything with their tokens, they have to interact with the central ERC20 contract,
calling its functions to update their balance. This contract handles access control for all functionality, implements all of its own correctness checks,
and enforces rules for all of its users.

Instead of using a central ledger system, Flow utilizes a few different concepts
to provide better safety, security, and clarity for smart contract developers and users.
In this section, we'll show how Flow's resources, interfaces, and other features are employed via a fungible token example.

## Intuiting Ownership with Resources

---

An important concept in Cadence is **Resources**, which are linear types.
A resource is a composite type (like a struct) that has its own defined fields and functions.
The difference is that resource objects have special rules that keep them from being copied or lost.
Resources are a new paradigm for asset ownership. Instead of representing token ownership in a central ledger smart contract,
each account owns its own resource object in its account storage that records the number of tokens they own.
This way, when users want to transact with each other, they can do so peer-to-peer without having to interact with a central token contract.
To transfer tokens to each other, they call a `transfer` function (or something equivalent)
on their own resource object and other users' resources, instead of a central `transfer` function.

This approach simplifies access control because instead of a central contract having to check the sender of a function call,
most function calls happen on resource objects stored in users' account,
and each user controls who is able to call the functions on resources in their account.
This concept, called Capability-based security, will be explained more in a later section.

This approach also helps protect against potential bugs. In a Solidity contract with all the logic and state contained in a central contract,
an exploit is likely to affect all users who are involved in the contract.

In Cadence, if there is a bug in the resource logic, an attacker would have to exploit the bug in each token holder's account individually,
which is much more complicated and time-consuming than it is in a central ledger system.

Below is an example of a resource for a fungible token vault. Every user who owns these tokens would have this resource stored in their account.
It is important to remember that each account stores only a copy of the `Vault` resource, and not a copy of the entire `ExampleToken` contract.
The `ExampleToken` contract only needs to be stored in the initial account that manages the token definitions.

```cadence Token.cdc
access(all) resource Vault: Provider, Receiver {

    // Balance of a user's Vault
    // we use unsigned fixed point numbers for balances
    // because they can represent decimals and do not allow negative values
    access(all) var balance: UFix64

    init(balance: UFix64) {
        self.balance = balance
    }

    access(all) fun withdraw(amount: UFix64): @Vault {
        self.balance = self.balance - amount
        return <-create Vault(balance: amount)
    }

    access(all) fun deposit(from: @Vault) {
        self.balance = self.balance + from.balance
        destroy from
    }
}
```

This piece of code is for educational purposes and is not comprehensive. However, it still showcases how a resource for a token works.

### Token Balances and Initialization

Each token resource object has a balance and associated functions (e.g., `deposit`, `withdraw`, etc).
When a user wants to use these tokens, they instantiate a zero-balance copy of this resource in their account storage.
The language requires that the initialization function `init`, which is only run once, must initialize all member variables.

```cadence
// Balance of a user's Vault
// we use unsigned fixed-point integers for balances because they do not require the
// concept of a negative number and allow for more clear precision
access(all) var balance: UFix64

init(balance: UFix64) {
    self.balance = balance
}
```

If you remove the initializer from your `ExampleToken` contract, it will cause an error because
the balance field is no longer initialized.

### Deposit

Then, the deposit function is available for any account to transfer tokens to.

```cadence
access(all) fun deposit(from: @Vault) {
    self.balance = self.balance + from.balance
    destroy from
}
```

### Transferring Tokens

When an account wants to send tokens to a different account, the sending account calls their own withdraw function first,
which subtracts tokens from their resource’s balance and temporarily creates a new resource object that holds this balance.
The sending account then calls the recipient account’s deposit function, which literally moves the resource instance to the other account,
adds it to their balance, and then destroys the used resource.
The resource needs to be destroyed because Cadence enforces strict rules around resource interactions.
A resource can never be left hanging in a piece of code. It either needs to be explicitly destroyed or stored in an account's storage.

When interacting with resources, you use the `@` symbol to specify the type, and a special “move operator” `<-`
when moving the resource, such as assigning the resource, when passing it as an argument to a function, or when returning it from a function.

```cadence
access(all) fun withdraw(amount: UInt64): @Vault {
```

This `@` symbol is required when specifying a resource **type** for a field, an argument, or a return value.
The move operator `<-` makes it clear that when a resource is used in an **assignment**, parameter, or return value,
it is moved to a new location and the old location is invalidated. This ensures that the resource only ever exists in one location at a time.

If a resource is moved out of an account's storage, it either needs to be moved to an account’s storage or explicitly destroyed.

```cadence
destroy from
```

This rule ensures that resources, which often represent real value, do not get lost because of a coding error.

You’ll notice that the arithmetic operations aren't explicitly protected against overflow or underflow.

```cadence
self.balance = self.balance - amount
```

In Solidity, this could be a risk for integer overflow or underflow, but Cadence has built-in overflow and underflow protection, so it is not a risk.
We are also using unsigned numbers in this example, so as mentioned earlier, the vault`s balance cannot go below 0.

Additionally, the requirement that an account contains a copy of the token’s resource type in its storage
ensures that funds cannot be lost by being sent to the wrong address.

If an address doesn’t have the correct resource type imported, the transaction will revert, ensuring that transactions sent to the wrong address are not lost.

**Important note: This protection is not in place for the Flow network currency,**
**because every Flow account is initialized with a default Flow Token Vault**
**in order to pay for [storage fees and transaction fees](https://developers.flow.com/build/basics/fees.md#fees).**

### Function Parameters

The line in `withdraw` that creates a new `Vault` has the parameter name `balance` specified in the function call.

```cadence
return <-create Vault(balance: amount)
```

This is another feature that Cadence uses to improve the clarity of code.
All function calls are required to specify the names of the arguments they are sending
unless the developer has specifically overridden the requirement in the funtion declaration.

## Interacting with the Fungible Token in the Flow Playground

Now that you have read about how the Fungible Token works,
we can deploy a basic version of it to your account and send some transactions to interact with it.

<Callout type="info">

Make sure that you have opened the Fungible Token templates in the playground
by following the link at the top of this page. You should have Account `0x01`
open and should see the code below.

</Callout>

```cadence
// BasicToken.cdc
//
// The BasicToken contract is a sample implementation of a fungible token on Flow.
//
// Fungible tokens behave like everyday currencies -- they can be minted, transferred or
// traded for digital goods.
//
// This is a basic implementation of a Fungible Token and is NOT meant to be used in production
// See the Flow Fungible Token standard for real examples: https://github.com/onflow/flow-ft

access(all) contract BasicToken {

    // Vault
    //
    // Each user stores an instance of only the Vault in their storage
    // The functions in the Vault are governed by the pre and post conditions
    // in the interfaces when they are called.
    // The checks happen at runtime whenever a function is called.
    //
    // Resources can only be created in the context of the contract that they
    // are defined in, so there is no way for a malicious user to create Vaults
    // out of thin air. A special Minter resource or constructor function needs to be defined to mint
    // new tokens.
    //
    access(all) resource Vault {

		// keeps track of the total balance of the account's tokens
        access(all) var balance: UFix64

        // initialize the balance at resource creation time
        init(balance: UFix64) {
            self.balance = balance
        }

        // withdraw
        //
        // Function that takes an integer amount as an argument
        // and withdraws that amount from the Vault.
        //
        // It creates a new temporary Vault that is used to hold
        // the money that is being transferred. It returns the newly
        // created Vault to the context that called so it can be deposited
        // elsewhere.
        //
        access(all) fun withdraw(amount: UFix64): @Vault {
            self.balance = self.balance - amount
            return <-create Vault(balance: amount)
        }

        // deposit
        //
        // Function that takes a Vault object as an argument and adds
        // its balance to the balance of the owners Vault.
        //
        // It is allowed to destroy the sent Vault because the Vault
        // was a temporary holder of the tokens. The Vault's balance has
        // been consumed and therefore can be destroyed.
        access(all) fun deposit(from: @Vault) {
            self.balance = self.balance + from.balance
            destroy from
        }
    }

    // createVault
    //
    // Function that creates a new Vault with an initial balance
    // and returns it to the calling context. A user must call this function
    // and store the returned Vault in their storage in order to allow their
    // account to be able to receive deposits of this token type.
    //
    access(all) fun createVault(): @Vault {
        return <-create Vault(balance: 30.0)
    }

    // The initializer for the contract.
    // All fields in the contract must be initialized at deployment.
    // This is just an example of what an implementation could do in the init initializer.
    init() {
        // create the Vault with the initial balance and put it in storage
        // account.save saves an object to the specified `to` path
        // The path is a literal path that consists of a domain and identifier
        // The domain must be `storage`, `private`, or `public`
        // the identifier can be any name
        let vault <- self.createVault()
        self.account.storage.save(<-vault, to: /storage/CadenceFungibleTokenTutorialVault)
    }
}
```

<Callout type="info">

Click the `Deploy` button at the top right of the editor to deploy the code.

</Callout>

![Deploy BasicToken on 0x01](./deploy_basic_token.png)

This deployment stores the contract for the basic fungible token
in the selected account (account `0x01`) so that it can be imported into transactions.

A contract's initializer runs at contract creation, and never again afterwards.
In our example, this function stores an instance of the `Vault` object with an initial balance of 30.

```cadence
// create the Vault with the initial balance and put it in storage
// account.save saves an object to the specified `to` path
// The path is a literal path that consists of a domain and identifier
// The domain must be `storage`, `private`, or `public`
// the identifier can be any name
let vault <- self.createVault()
self.account.storage.save(<-vault, to: /storage/CadenceFungibleTokenTutorialVault)
```

This line saves the new `@Vault` object to storage.
Account storage is indexed with paths, which consist of a domain and identifier. `/domain/identifier`. Only three domains are allowed for paths:

- `storage`: The place where all objects are stored. Only accessible by the owner of the account.
- `private`: Stores links, otherwise known as capabilities, to objects in storage. Only accessible by the owner of the account
- `public`: Stores links to objects in storage: Accessible by anyone in the network.

Contracts have access to the private `AuthAccount` object of the account it is deployed to, using `self.account`.
This object has methods that can modify storage in many ways.
See the [account](../language/accounts) documentation for a list of all the methods it can call.

In this line, we call the `save` method to store an object in storage.
The first argument is the value to store, and the second argument is the path where the value is being stored.
For `save` the path has to be in the `/storage/` domain.

You are now ready to run transactions that use the fungible tokens!

### Perform a Basic Transfer

As we talked about above, a token transfer with resources is not a simple update to a ledger.
In Cadence, you have to first withdraw tokens from your vault, then deposit them to the vault
that you want to transfer to. We'll start a simple transaction that withdraws tokens from a vault
and deposits them back into the same vault.

<Callout type="info">

Open the transaction named `Basic Transfer`. <br/>
`Basic Transfer` should contain the following code for withdrawing and depositing with a stored Vault:

</Callout>

```cadence BasicTransfer.cdc
// Basic Transfer

import BasicToken from 0x01

// This transaction is used to withdraw and deposit tokens with a Vault

transaction {

  prepare(acct: auth(BorrowValue) &Account) {
    // withdraw tokens from your vault by borrowing a reference to it
    // and calling the withdraw function with that reference
    let vaultRef = acct.storage.borrow<auth(FungibleToken.Withdrawable) &BasicToken.Vault>(from: /storage/CadenceFungibleTokenTutorialVault)
        ?? panic("Could not borrow a reference to the owner's vault")

    let temporaryVault <- vaultRef.withdraw(amount: 10.0)

    // deposit your tokens to the Vault
    vaultRef.deposit(from: <-temporaryVault)

    log("Withdraw/Deposit succeeded!")
  }
}
```

<Callout type="info">
  Select account `0x01` as the only signer. <br />
  Click the `Send` button to submit the transaction. <br />
  This transaction withdraws tokens from the main vault and deposits them back
  to it.
</Callout>

This transaction is a basic example of a transfer within an account.
It withdraws tokens from the main vault and deposits back to the main vault.
It is simply to illustrate the basic functionality of how transfers work.

You'll see in this transaction that
you can borrow a reference directly from an object in storage.

```cadence
// Borrow a reference to the stored, private Vault resource
let vaultRef = acct.storage.borrow<&BasicToken.Vault>(from: /storage/CadenceFungibleTokenTutorialVault)
    ?? panic("Could not borrow a reference to the owner's vault")
```

This allows you to efficiently access objects in storage without having to load them,
which is a much more costly interaction.

In production code, you'll likely be transferring tokens to other accounts.
Capabilities allow us to accomplish this safely.

## Ensuring Security in Public: Capability Security

---

Another important feature in Cadence is its utilization of [**Capability-Based Security.**](../language/capabilities)
This feature ensures that while the withdraw function is declared public on the resource,
no one except the intended user and those they approve of can withdraw tokens from their vault.

Cadence's security model ensures that objects stored in an account's storage can only be accessed by the account that owns them.
If a user wants to give another user access to their stored objects, they can link a public capability,
which is like an "API" that allows others to call specified functions on their objects.

An account only has access to the fields and methods of an object in a different account if they hold a capability to that object
that explicitly allows them to access those fields and methods.
Only the owner of an object can create a capability for it.
Therefore, when a user creates a Vault in their account, they only publish a capability
that exposes the deposit function and the balance field.

The withdraw function can remain hidden as a function that only the owner can call.

This removes the need to check the address of the account that made the function call
(`msg.sender` in Ethereum) for access control purposes, because this functionality is handled by the protocol and type checker.
If you aren't the owner of an object or don't have a valid reference to it that was created by the owner, you cannot access the object at all!

### Using Interfaces to Secure Implementations

---

The next important concept in Cadence is design-by-contract,
which uses preconditions and postconditions to document and programmatically assert the change in state caused by a piece of a program.
These conditions are specified in interfaces that enforce rules about how types are defined and behave.
They can be stored on-chain in an immutable fashion so that certain pieces of code can import and implement them to ensure that they meet certain standards.

Here is an example of how interfaces for the `Vault` resource we defined above would look.

```cadence Interfaces.cdc
// Interface that enforces the requirements for withdrawing
// tokens from the implementing type
//
access(all) resource interface Provider {
    access(all) fun withdraw(amount: UFix64): @Vault {
        post {
            result.balance == amount:
                "Withdrawal amount must be the same as the balance of the withdrawn Vault"
        }
    }
}
// Interface that enforces the requirements for depositing
// tokens into the implementing type
//
access(all) resource interface Receiver {

    // There aren't any meaningful requirements for only a deposit function
    // but this still shows that the deposit function is required in an implementation.
    access(all) fun deposit(from: @Vault)
}

// Balance
//
// Interface that specifies a public `balance` field for the vault
//
access(all) resource interface Balance {
    access(all) var balance: UFix64
}
```

In our example, the `Vault` resource will implement all three of these interfaces.
The interfaces ensure that specific fields and functions are present in the resource implementation
and that the function arguments, fields of the resource, and any return value are in a valid state before and/or after execution.
These interfaces can be stored on-chain and imported into other contracts or resources
so that these requirements are enforced by an immutable source of truth that is not susceptible to human error.

You can also see that functions and fields have the `access(all)` keyword next to them.
We have explicitly defined these fields as public because all fields and functions in Cadence are private by default,
meaning that the local scope can only access them. Users have to make parts of their owned types explicitly public.
This helps prevent types from having unintentionally public code.

This does bring up an important security consideration though.
While we have made all our fields and functions public here, it is actually recommended to default to making
fields private unless it is explicitly needed to be public.

<Callout type="warning">
  This is especially important for array and dictionary types,
  which can have their contents maliciously mutated if they are made public.
  This is one of THE MOST COMMON security mistakes that Cadence developers make,
  so it is vitally important to be aware of this.

  See the [Cadence Best Practices document](../anti-patterns#array-or-dictionary-fields-should-be-private) for more details.
</Callout>

## Adding Interfaces to Our Fungible Token

Now, we are going to add these interfaces to our Fungible token along with a minter resource.

Open account `0x02` in the playground. You should see the `ExampleToken` contract.
In addition to everything that is in the `BasicToken` contract,
we have also added the `Provider`, `Receiver`, and `Balance` interfaces described above.

Now that our `ExampleToken.Vault` type has declared that it implements these interfaces on line 93,
it is required to have their fields and functions, and their pre and post-conditions will also
be evaluated every time their respective functions are called.
We can also use these interfaces to create restricted capabilities, described in the next section.

Additionally, `ExampleToken` changes `createVault()` to `createEmptyVault()`
so that token minting is restricted to the newly added `VaultMinter` resource.
This illustrates another powerful feature of Cadence resources.
Instead of the contract maintaining a list of minter addresses,
accounts that the owner wants to be minters can be giving a special resource
that directly gives them the authority to mint tokens.
This method for authorization can be used in many different ways
and further decentralizes the control of the contract.

We also store the `VaultMinter` object to `/storage/`
in the initializer in the same way as the vault, but in a different storage path:

```cadence
self.account.storage<-create VaultMinter(), to: /storage/CadenceFungibleTokenTutorialMinter)
```

Now is an important time to remind you that account storage not namespaced by contract,
meaning that path names could potentially conflict. This is why it is important to
choose unique names for your paths like we have done here so there is a very low chance
of them conflicting with other projects paths.

## Create, Store, and Publish Capabilities and References to a Vault

---

Capabilities are like pointers in other languages. They are a link to an object in an account's storage
and can be used to read fields or call functions on the object they reference. They cannot move or modify the object directly.

There are many different situations in which you would create a capability to your fungible token vault.
You might want a simple way to call methods on your `Vault` from anywhere in a transaction.
You could also send a capability that only exposes withdraw function in your `Vault` so that others can transfer tokens for you.
You could also have one that only exposes the `Balance` interface, so that others can check how many tokens you own.
There could also be a function that takes a capability to a `Vault` as an argument, borrows a reference to the capability,
makes a single function call on the reference, then finishes and destroys the reference.

We already use this pattern in the `VaultMinter` resource in the `mintTokens` function, shown here:

```cadence
// Function that mints new tokens and deposits into an account's vault
// using their `Receiver` capability.
// We say `&AnyResource{Receiver}` to say that the recipient can be any resource
// as long as it implements the ExampleToken.Receiver interface
access(all) fun mintTokens(amount: UFix64, recipient: Capability<&AnyResource{Receiver}>) {
    let recipientRef = recipient.borrow()
        ?? panic("Could not borrow a receiver reference to the vault")

    ExampleToken.totalSupply = ExampleToken.totalSupply + UFix64(amount)
    recipientRef.deposit(from: <-create Vault(balance: amount))
}
```

The function takes a capability as an argument.
This syntax might be unclear to you:

```cadence
recipient: Capability<&AnyResource{Receiver}>
```

This means that `recipient` has to be a Capability that is restricted to the type contained in `<>`.
The type outside of the curly braces `{}` has to be a concrete type and the type in the curly braces
has to be an interface type. Here we are saying that the type
can be any resource that implements the `ExampleToken.Receiver` interface.
If that is true, this function borrows a reference from this capability
and uses the reference to call the `deposit` function of that resource because we know that
the `deposit` function will be there since it is in the `ExampleToken.Receiver` interface.

Let's create capabilities to your `Vault` so that a separate account can send tokens to you.

<Callout type="info">

Before we submit a transaction interacting with ExampleToken resources, we'll need to deploy the contract to account `0x02`:<br/>
1. Select Contract 2 in the playground sidebar (the ExampleToken contract)<br/>
2. Make sure that signer `0x02` is selected as the deploying address<br/>
3. Click "Deploy"

</Callout>

![Deploy ExampleToken to 0x02](./deploy_example_token.png)

Now we can continue on to configure Capabilities on the ExampleToken Vault.

<Callout type="info">

Open the transaction named `Create Link`. <br/>
`Create Link` should contain the following code for creating a reference to the stored Vault:

</Callout>

```cadence CreateLink.cdc
// Create Link

import ExampleToken from 0x02

// This transaction creates a capability
// that is linked to the account's token vault.
// The capability is restricted to the fields in the `Receiver` interface,
// so it can only be used to deposit funds into the account.
transaction {
  prepare(acct: AuthAccount) {

    // Create a link to the Vault in storage that is restricted to the
    // fields and functions in `Receiver` and `Balance` interfaces,
    // this only exposes the balance field
    // and deposit function of the underlying vault.
    //
    acct.link<&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}>(/public/CadenceFungibleTokenTutorialReceiver, target: /storage/CadenceFungibleTokenTutorialVault)

    log("Public Receiver reference created!")
  }

  post {
    // Check that the capabilities were created correctly
    // by getting the public capability and checking
    // that it points to a valid `Vault` object
    // that implements the `Receiver` interface
    getAccount(0x02).getCapability<&ExampleToken.Vault{ExampleToken.Receiver}>(/public/CadenceFungibleTokenTutorialReceiver)
                    .check():
                    "Vault Receiver Reference was not created correctly"
    }
}
```

In order to use a capability, we have to first create a link to that object in storage.
A reference can then be created from a capability, and references cannot be stored.
They need to be lost at the end of a transaction execution.
This restriction is to prevent reentrancy attacks which are attacks where a malicious user calls into the same function over and over again
before the original execution has finished. Only allowing one reference at a time for an object prevents these attacks for objects in storage.

To create a capability, we use the `link` function.

```cadence
// Create a link to the Vault in storage that is restricted to the
// fields and functions in `Receiver` and `Balance` interfaces,
// this only exposes the balance field
// and deposit function of the underlying vault.
//
acct.link<&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}>.
(/public/CadenceFungibleTokenTutorialReceiver, target: /storage/CadenceFungibleTokenTutorialVault)
```

`link` creates a new capability that is kept at the path in the first argument, targeting the `target` in the second argument.
The type restriction for the link is specified in the `<>`. We use `&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}`
to say that the link can be any resource as long as it implements and is cast as the Receiver interface.
This is the common format for describing references.
You first have a `&` followed by the concrete type, then the interface in curly braces to ensure that
it is a reference that implements that interface and only includes the fields specified in that interface.

We put the capability in `/public/CadenceFungibleTokenTutorialReceiver` because we want it to be publicly accessible.
The `public` domain of an account is accessible to anyone in the network via an account's `PublicAccount` object, which is fetched by using the `getAccount(address)` function.

Next is the `post` phase of the transaction.

```cadence
post {
// Check that the capabilities were created correctly
// by getting the public capability and checking
// that it points to a valid `Vault` object
// that implements the `Receiver` interface
getAccount(0x01).getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                .check<&ExampleToken.Vault{ExampleToken.Receiver}>():
                "Vault Receiver Reference was not created correctly"
}
```

The `post` phase is for ensuring that certain conditions are met after the transaction has been executed.
Here, we are getting the capability from its public path and calling its `check` function to ensure
that the capability contains a valid link to a valid object in storage that is the specified type.

<Callout type="info">

Now that we understand the transaction, time to submit it:<br/>

1. Select account `0x02` as the only signer.<br/>
2. Click the `Send` button to submit the transaction.<br/>
3. This transaction creates a new public reference to your `Vault` and checks that it was created correctly.

</Callout>

## Transfer Tokens to Another User

---

Now, we are going to run a transaction that sends 10 tokens to account `0x03`.
We will do this by calling the `withdraw` function on account `0x02`'s Vault,
which creates a temporary Vault object for moving the tokens,
then deposits those tokens into account `0x03`'s vault by calling the `deposit` function on their vault.

Here we encounter another safety feature that Cadence introduces. Owning tokens requires you to have a `Vault` object stored in your account,
so if anyone tries to send tokens to an account who isn't prepared to receive them, the transaction will fail.
This way, Cadence protects the user if they accidentally enter the account address incorrectly when sending tokens.

<Callout type="info">

Account `0x03` has not been set up to receive tokens, so we will do that now:

1. Open the transaction `Setup Account`.<br/>
2. Select account `0x03` as the only signer.<br/>
3. Click the `Send` button to set up account `0x03` so that it can receive tokens.

</Callout>

```cadence SetupAccount.cdc
// Setup Account

import ExampleToken from 0x02

// This transaction configures an account to store and receive tokens defined by
// the ExampleToken contract.
transaction {
	prepare(acct: AuthAccount) {
		// Create a new empty Vault object
		let vaultA <- ExampleToken.createEmptyVault()

		// Store the vault in the account storage
		acct.storage.save(<-vaultA, to: /storage/CadenceFungibleTokenTutorialVault)

        log("Empty Vault stored")

        // Create a public Receiver capability to the Vault
		let ReceiverRef = acct.link<&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}>(/public/CadenceFungibleTokenTutorialReceiver, target: /storage/CadenceFungibleTokenTutorialVault)

        log("References created")
	}

    post {
        // Check that the capabilities were created correctly
        getAccount(0x03).getCapability<&ExampleToken.Vault{ExampleToken.Receiver}>(/public/CadenceFungibleTokenTutorialReceiver)
                        .check():
                        "Vault Receiver Reference was not created correctly"
    }
}
```

Here we perform the same actions that account `0x02` did to set up its `Vault`, but all in one transaction.
Account `0x03` is ready to start building its fortune! As you can see, when we created the Vault for account `0x03`,
we had to create one with a balance of zero by calling the `createEmptyVault()` function.
Resource creation is restricted to the contract where it is defined, so in this way, the Fungible Token smart contract can ensure that
nobody is able to create new tokens out of thin air.

As part of the initial deployment process for the ExampleToken contract, account `0x02` created a `VaultMinter` object.
By using this object, the account that owns it can mint new tokens.
Right now, account `0x02` owns it, so it has sole power to mint new tokens.
We could have had a `mintTokens` function defined in the contract,
but then we would have to check the sender of the function call to make sure that they are authorized,
which is not the recommended way to perform access control.

As we explained before, the resource model plus capability security handles this access control for us as a built in language construct
instead of having to be defined in the code. If account `0x02` wanted to authorize another account to mint tokens,
they could either move the `VaultMinter` object to the other account, or give the other account a private capability to the single `VaultMinter`.
Or, if they didn't want minting to be possible after deployment, they would simply mint all the tokens at contract initialization
and not even include the `VaultMinter` in the contract.

In the next transaction, account `0x02` will mint 30 new tokens and deposit them into account `0x03`'s newly created Vault.

<Callout type="info">

1. Open the `Mint Tokens` transaction.<br/>
2. Select only account `0x02` as a signer and send `Mint Tokens` to mint 30 tokens for account `0x03`.

</Callout>

`Mint Tokens` should contain the code below.

```cadence MintTokens.cdc
// Mint Tokens

import ExampleToken from 0x02

// This transaction mints tokens and deposits them into account 3's vault
transaction {

    // Local variable for storing the reference to the minter resource
    let mintingRef: &ExampleToken.VaultMinter

    // Local variable for storing the reference to the Vault of
    // the account that will receive the newly minted tokens
    var receiver: Capability<&ExampleToken.Vault{ExampleToken.Receiver}>

	prepare(acct: AuthAccount) {
        // Borrow a reference to the stored, private minter resource
        self.mintingRef = acct.borrow<&ExampleToken.VaultMinter>(from: /storage/CadenceFungibleTokenTutorialMinter)
            ?? panic("Could not borrow a reference to the minter")

        // Get the public account object for account 0x03
        let recipient = getAccount(0x03)

        // Get their public receiver capability
        self.receiver = recipient.getCapability<&ExampleToken.Vault{ExampleToken.Receiver}>
(/public/CadenceFungibleTokenTutorialReceiver)

	}

    execute {
        // Mint 30 tokens and deposit them into the recipient's Vault
        self.mintingRef.mintTokens(amount: 30.0, recipient: self.receiver)

        log("30 tokens minted and deposited to account 0x03")
    }
}
```

This is the first example of a transaction where we utilize local transaction variables that span different stages in the transaction.
We declare the `mintingRef` and `receiverRef` variables outside of the prepare stage but must initialize them in prepare.
We can then use them in later stages in the transaction.

Then we borrow a refernce to the `VaultMinter`. We specify the borrow as a `VaultMinter` reference
and have the reference point to `/storage/CadenceFungibleTokenTutorialMinter`.
The reference is borrowed as an optional so we use the nil-coalescing operator (`??`) to make sure the value isn't nil.
If the value is nil, the transaction will execute the code after the `??`. The code is a panic, so it will revert and print the error message.

You can use the `getAccount()` built-in function to get any account's public account object.
The public account object lets you get capabilities from the `public` domain of an account, where public capabilities are stored.

We use the `getCapability` function to get the public capability from a public path,
then use the `borrow` function on the capability to get the reference from it, typed as a `ExampleToken.Vault{ExampleToken.Receiver}`.

```cadence
// Get the public receiver capability
let cap = recipient.getCapability(/public/CadenceFungibleTokenTutorialReceiver)

// Borrow a reference from the capability
self.receiverRef = cap.borrow<&ExampleToken.Vault{ExampleToken.Receiver}>()
        ?? panic("Could not borrow a reference to the receiver")
```

In the execute phase, we simply use the reference to mint 30 tokens and deposit them into the `Vault` of account `0x03`.

## Check Account Balances

Now, both account `0x02` and account `0x03` should have a `Vault` object in their storage that has a balance of 30 tokens.
They both should also have a `Receiver` capability stored in their `/public/` domains that links to their stored `Vault`.

<img src="https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/account-balances.png" />

An account cannot receive any token type unless it is specifically configured to accept those tokens.
As a result, it is difficult to send tokens to the wrong address accidentally.
But, if you make a mistake setting up the `Vault` in the new account, you won't be able to send tokens to it.

Let's run a script to make sure we have our vaults set up correctly.

You can use scripts to access an account's public state. Scripts aren't signed by any account and cannot modify state.

In this example, we will query the balance of each account's vault. The following will print out the balance of each account in the emulator.

<Callout type="info">

Open the script named `Get Balances` in the scripts pane.

</Callout>

`Get Balances` should contain the following code:

```cadence Script1.cdc
// Get Balances

import FungibleToken from 0x02

// This script reads the Vault balances of two accounts.
access(all) fun main() {
    // Get the accounts' public account objects
    let acct2 = getAccount(0x02)
    let acct3 = getAccount(0x03)

    // Get references to the account's receivers
    // by getting their public capability
    // and borrowing a reference from the capability
    let acct2ReceiverRef = acct2.getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                            .borrow<&FungibleToken.Vault{FungibleToken.Balance}>()
                            ?? panic("Could not borrow a reference to the acct2 receiver")
    let acct3ReceiverRef = acct3.getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                            .borrow<&FungibleToken.Vault{FungibleToken.Balance}>()
                            ?? panic("Could not borrow a reference to the acct3 receiver")

    // Use optional chaining to read and log balance fields
    log("Account 2 Balance")
	log(acct2ReceiverRef.balance)
    log("Account 3 Balance")
    log(acct3ReceiverRef.balance)
}

```

<Callout type="info">

Execute `Get Balances` by clicking the Execute button.

</Callout>

This should ensure the following:

- Account `0x02`'s balance is 30
- Account `0x03`'s balance is 30

If correct, you should see the following lines:

```
"Account 1 Balance"
30
"Account 2 Balance"
30
Result > "void"
```

If there is an error, this probably means that you missed a step earlier
and might need to restart from the beginning.

To restart the playground, close your current session and open the link at the top of the tutorial.

Now that we have two accounts, each with a `Vault`, we can see how they transfer tokens to each other!

<Callout type="info">

1. Open the transaction named `Transfer Tokens`. <br/>
2. Select account `0x03` as a signer and send the transaction. <br/>
3. `Transfer Tokens` should contain the following code for sending tokens to another user:

</Callout>

```cadence TransferTokens.cdc
// Transfer Tokens

import ExampleToken from 0x02

// This transaction is a template for a transaction that
// could be used by anyone to send tokens to another account
// that owns a Vault
transaction {

  // Temporary Vault object that holds the balance that is being transferred
  var temporaryVault: @ExampleToken.Vault

  prepare(acct: AuthAccount) {
    // withdraw tokens from your vault by borrowing a reference to it
    // and calling the withdraw function with that reference
    let vaultRef = acct.borrow<&ExampleToken.Vault>(from: /storage/CadenceFungibleTokenTutorialVault)
        ?? panic("Could not borrow a reference to the owner's vault")

    self.temporaryVault <- vaultRef.withdraw(amount: 10.0)
  }

  execute {
    // get the recipient's public account object
    let recipient = getAccount(0x02)

    // get the recipient's Receiver reference to their Vault
    // by borrowing the reference from the public capability
    let receiverRef = recipient.getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                      .borrow<&ExampleToken.Vault{ExampleToken.Receiver}>()
                      ?? panic("Could not borrow a reference to the receiver")

    // deposit your tokens to their Vault
    receiverRef.deposit(from: <-self.temporaryVault)

    log("Transfer succeeded!")
  }
}

```

In this example, the signer withdraws tokens from their `Vault`,
which creates and returns a temporary `Vault` resource object with `balance=10`
that is used for transferring the tokens. In the execute phase,
the transaction moves that resource to another user's `Vault` using their `deposit` method.
The temporary `Vault` is destroyed after its balance is added to the recipient's `Vault`.

You might be wondering why we have to use two function calls to complete a token transfer when it is possible to do it in one.
This is because of the way resources work in Cadence.
In a ledger-based model, you would just call transfer, which just updates the ledger,
but in Cadence, the location of the tokens matters,
and therefore most token transfer situations will not just be a direct account-to-account transfer.

Most of the time, tokens will be used for a different purpose first,
like purchasing something, and that requires the `Vault` to be separately sent
and verified before being deposited to the storage of an account.

Separating the two also allows us to take advantage of being able
to statically verify which parts of accounts can be modified in the `prepare` section of a transaction,
which will help users have peace of mind when getting fed transactions to sign from an app.

<Callout type="info">

Execute `Get Balances` again.

</Callout>

If correct, you should see the following lines indicating that account `0x02`'s balance is 40 and account `0x03`'s balance is 20:

```
"Account 2 Balance"
40
"Account 3 Balance"
20
Result > "void"
```

You now know how a basic fungible token is used in Cadence and Flow!

From here, you could try to extend the functionality of fungible tokens by making:

- A faucet for these tokens
- An escrow that can be deposited to (but only withdrawn when the balance reaches a certain point)
- A function to the resource that mints new tokens!

## Create a Flow Marketplace

---
Now that you have an understanding of how fungible tokens work on Flow and have a working NFT, you can learn how to create 
a marketplace that uses both fungible tokens and NFTs. Move on to the next tutorial to learn about Marketplaces in Cadence!
---
title: 7. Marketplace Setup
---

In this tutorial, we're going to create a marketplace that uses both the fungible
and non-fungible token (NFTs) contracts that we have learned about in previous tutorials.
This page requires you to execute a series of transactions to setup your accounts to complete the Marketplace tutorial.
The next page contains the main content of the tutorial.

When you are done with the tutorial, check out the [NFTStorefront repo](https://github.com/onflow/nft-storefront)
for an example of a production ready marketplace that you can use right now on testnet or mainnet!

---

<Callout type="success">
  Open the starter code for this tutorial in the Flow Playground: 
  <a 
    href="https://play.onflow.org/49ec2856-1258-4675-bac3-850b4bae1929"
    target="_blank"
  >
    https://play.onflow.org/49ec2856-1258-4675-bac3-850b4bae1929
  </a>
  <br/>
  The tutorial will be asking you to take various actions to interact with this code.
</Callout>

If you have already completed the Marketplace tutorial, please move on to [Composable Resources: Kitty Hats](./10-resources-compose.md).

This guide will help you quickly get the playground to the state you need to complete the Marketplace tutorial.
The marketplace tutorial uses the Fungible Token and Non-Fungible token contracts
to allow users to buy and sell NFTs with fungible tokens.

The state of the accounts is the same as if you had completed the Fungible Token
and Non-Fungible Token tutorials in the same playground session.
Having your playground in this state is necessary to follow the [Composable Smart Contracts: Marketplace](./08-marketplace-compose.md) tutorial.

---

1. Open account `0x01`. Make sure the Fungible Token definitions in `ExampleToken.cdc` from the fungible token tutorial are in this account.
2. Deploy the ExampleToken code to account `0x01`.
3. Switch to the ExampleNFT contract (Contract 2)
4. Make sure you have the NFT definitions in `ExampleNFT.cdc` from the Non-fungible token tutorial in account `0x02`.
5. Deploy the NFT code to account `0x02` by selecting it as the deploying signer.
6. Run the transaction in Transaction 1. This is the `SetupAccount1Transaction.cdc` file.
   Use account `0x01` as the only signer to set up account `0x01`'s storage.

```cadence SetupAccount1Transaction.cdc
// SetupAccount1Transaction.cdc

import ExampleToken from 0x01
import ExampleNFT from 0x02

// This transaction sets up account 0x01 for the marketplace tutorial
// by publishing a Vault reference and creating an empty NFT Collection.
transaction {
  prepare(acct: AuthAccount) {
    // Create a public Receiver capability to the Vault
    acct.link<&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}>
             (/public/CadenceFungibleTokenTutorialReceiver, target: /storage/CadenceFungibleTokenTutorialVault)

    log("Created Vault references")

    // store an empty NFT Collection in account storage
    acct.save<@ExampleNFT.Collection>(<-ExampleNFT.createEmptyCollection(), to: /storage/nftTutorialCollection)

    // publish a capability to the Collection in storage
    acct.link<&{ExampleNFT.NFTReceiver}>(ExampleNFT.CollectionPublicPath, target: ExampleNFT.CollectionStoragePath)

    log("Created a new empty collection and published a reference")
  }
}
```

7. Run the transaction in Transaction 2. This is the `SetupAccount2Transaction.cdc` file.
Use account `0x02` as the only signer to set up account `0x02`'s storage.

```cadence SetupAccount2Transaction.cdc
// SetupAccount2Transaction.cdc

import ExampleToken from 0x01
import ExampleNFT from 0x02

// This transaction adds an empty Vault to account 0x02
// and mints an NFT with id=1 that is deposited into
// the NFT collection on account 0x01.
transaction {

  // Private reference to this account's minter resource
  let minterRef: &ExampleNFT.NFTMinter

  prepare(acct: AuthAccount) {
    // create a new vault instance with an initial balance of 30
    let vaultA <- ExampleToken.createEmptyVault()

    // Store the vault in the account storage
    acct.save<@ExampleToken.Vault>(<-vaultA, to: /storage/CadenceFungibleTokenTutorialVault)

    // Create a public Receiver capability to the Vault
    let ReceiverRef = acct.link<&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}>(/public/CadenceFungibleTokenTutorialReceiver, target: /storage/CadenceFungibleTokenTutorialVault)

    log("Created a Vault and published a reference")

    // Borrow a reference for the NFTMinter in storage
    self.minterRef = acct.borrow<&ExampleNFT.NFTMinter>(from: ExampleNFT.MinterStoragePath)
        ?? panic("Could not borrow owner's NFT minter reference")
  }
  execute {
    // Get the recipient's public account object
    let recipient = getAccount(0x01)

    // Get the Collection reference for the receiver
    // getting the public capability and borrowing a reference from it
    let receiverRef = recipient.getCapability(ExampleNFT.CollectionPublicPath)
                               .borrow<&{ExampleNFT.NFTReceiver}>()
                               ?? panic("Could not borrow nft receiver reference")

    // Mint an NFT and deposit it into account 0x01's collection
    receiverRef.deposit(token: <-self.minterRef.mintNFT())

    log("New NFT minted for account 1")
  }
}
```

8. Run the transaction in Transaction 3. This is the `SetupAccount1TransactionMinting.cdc` file.
   Use account `0x01` as the only signer to mint fungible tokens for account 1 and 2.

```cadence SetupAccount1TransactionMinting.cdc
// SetupAccount1TransactionMinting.cdc

import ExampleToken from 0x01
import ExampleNFT from 0x02

// This transaction mints tokens for both accounts using
// the minter stored on account 0x01.
transaction {

  // Public Vault Receiver References for both accounts
  let acct1Capability: Capability<&AnyResource{ExampleToken.Receiver}>
  let acct2Capability: Capability<&AnyResource{ExampleToken.Receiver}>

  // Private minter references for this account to mint tokens
  let minterRef: &ExampleToken.VaultMinter

  prepare(acct: AuthAccount) {
    // Get the public object for account 0x02
    let account2 = getAccount(0x02)

    // Retrieve public Vault Receiver references for both accounts
    self.acct1Capability = acct.getCapability<&AnyResource{ExampleToken.Receiver}>(/public/CadenceFungibleTokenTutorialReceiver)
    self.acct2Capability = account2.getCapability<&AnyResource{ExampleToken.Receiver}>(/public/CadenceFungibleTokenTutorialReceiver)

    // Get the stored Minter reference for account 0x01
    self.minterRef = acct.borrow<&ExampleToken.VaultMinter>(from: /storage/CadenceFungibleTokenTutorialMinter)
        ?? panic("Could not borrow owner's vault minter reference")
  }

  execute {
    // Mint tokens for both accounts
    self.minterRef.mintTokens(amount: 20.0, recipient: self.acct2Capability)
    self.minterRef.mintTokens(amount: 10.0, recipient: self.acct1Capability)

    log("Minted new fungible tokens for account 1 and 2")
  }
}
```

9. Run the script `CheckSetupScript.cdc` file in Script 1 to ensure everything is set up.

```cadence CheckSetupScript.cdc
// CheckSetupScript.cdc

import ExampleToken from 0x01
import ExampleNFT from 0x02

// This script checks that the accounts are set up correctly for the marketplace tutorial.
//
// Account 0x01: Vault Balance = 40, NFT.id = 1
// Account 0x02: Vault Balance = 20, No NFTs
pub fun main() {
    // Get the accounts' public account objects
    let acct1 = getAccount(0x01)
    let acct2 = getAccount(0x02)

    // Get references to the account's receivers
    // by getting their public capability
    // and borrowing a reference from the capability
    let acct1ReceiverRef = acct1.getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                          .borrow<&ExampleToken.Vault{ExampleToken.Balance}>()
                          ?? panic("Could not borrow acct1 vault reference")

    let acct2ReceiverRef = acct2.getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                          .borrow<&ExampleToken.Vault{ExampleToken.Balance}>()
                          ?? panic("Could not borrow acct2 vault reference")

    // Log the Vault balance of both accounts and ensure they are
    // the correct numbers.
    // Account 0x01 should have 40.
    // Account 0x02 should have 20.
    log("Account 1 Balance")
    log(acct1ReceiverRef.balance)
    log("Account 2 Balance")
    log(acct2ReceiverRef.balance)

    // verify that the balances are correct
    if acct1ReceiverRef.balance != 40.0 || acct2ReceiverRef.balance != 20.0 {
        panic("Wrong balances!")
    }

    // Find the public Receiver capability for their Collections
    let acct1Capability = acct1.getCapability(ExampleNFT.CollectionPublicPath)
    let acct2Capability = acct2.getCapability(ExampleNFT.CollectionPublicPath)

    // borrow references from the capabilities
    let nft1Ref = acct1Capability.borrow<&{ExampleNFT.NFTReceiver}>()
        ?? panic("Could not borrow acct1 nft collection reference")

    let nft2Ref = acct2Capability.borrow<&{ExampleNFT.NFTReceiver}>()
        ?? panic("Could not borrow acct2 nft collection reference")

    // Print both collections as arrays of IDs
    log("Account 1 NFTs")
    log(nft1Ref.getIDs())

    log("Account 2 NFTs")
    log(nft2Ref.getIDs())

    // verify that the collections are correct
    if nft1Ref.getIDs()[0] != 1 || nft2Ref.getIDs().length != 0 {
        panic("Wrong Collections!")
    }
}
```

10. The script should not panic and you should see something like this output

```
"Account 1 Balance"
40.00000000
"Account 2 Balance"
20.00000000
"Account 1 NFTs"
[1]
"Account 2 NFTs"
[]
```

---

With your playground now in the correct state, you're ready to continue with the next tutorial.

You do not need to open a new playground session for the marketplace tutorial. You can just continue using this one.
---
title: 8. Marketplace
---

In this tutorial, we're going to create a marketplace that uses both the fungible
and non-fungible token (NFTs) contracts that we have learned about in previous tutorials.
This is only for educational purposes and is not meant to be used in production
See a production-ready marketplace in the [NFT storefront repo.](https://github.com/onflow/nft-storefront)
This contract is already deployed to testnet and mainnet and can be used by anyone for any generic NFT sale!

---

<Callout type="success">
  Open the starter code for this tutorial in the Flow Playground: 
  <a 
    href="https://play.onflow.org/49ec2856-1258-4675-bac3-850b4bae1929"
    target="_blank"
  >
    https://play.onflow.org/49ec2856-1258-4675-bac3-850b4bae1929
  </a>
  <br/>
  The tutorial will be asking you to take various actions to interact with this code.
  [The marketplace setup guide](./07-marketplace-setup.md) shows you how to get the playground set up to do this tutorial.
</Callout>

<Callout type="info">
Instructions that require you to take action are always included in a callout box like this one.
These highlighted actions are all that you need to do to get your code running,
but reading the rest is necessary to understand the language's design.
</Callout>

Marketplaces are a popular application of blockchain technology and smart contracts.
When there are NFTs in existence, users usually want to be able to buy and sell them with their fungible tokens.

Now that there is an example for both fungible and non-fungible tokens,
we can build a marketplace that uses both. This is referred to as **composability**:
the ability for developers to leverage shared resources, such as code or userbases,
and use them as building blocks for new applications.

Flow is designed to enable composability because of the way that interfaces, resources and capabilities are designed.

- [Interfaces](../language/interfaces) allow projects to support any generic type as long as it supports a standard set of functionality specified by an interface.
- [Resources](../language/resources) can be passed around and owned by accounts, contracts or even other resources, unlocking different use cases depending on where the resource is stored.
- [Capabilities](../language/capabilities) allow exposing user-defined sets of functionality through special objects that enforce strict security with Cadence's type system.

The combination of these allows developers to do more with less, re-using known safe code and design patterns
to create new, powerful, and unique interactions!

<Callout type="info">

At some point before or after this tutorial, you should definitely check out the formal documentation
linked above about interfaces, resources, and capabilities. It will help complete your understanding
of these complex, but powerful features.

</Callout>

To create a marketplace, we need to integrate the functionality of both fungible
and non-fungible tokens into a single contract that gives users control over their money and assets.
To accomplish this, we're going to take you through these steps to create a composable smart contract and get comfortable with the marketplace:

1. Ensure that your fungible token and non-fungible token contracts are deployed and set up correctly.
2. Deploy the marketplace type declarations to account `0x03`.
3. Create a marketplace object and store it in your account storage, putting an NFT up for sale and publishing a public capability for your sale.
4. Use a different account to purchase the NFT from the sale.
5. Run a script to verify that the NFT was purchased.

**Before proceeding with this tutorial**, you need to complete the [Fungible Tokens](./06-fungible-tokens.md)
and [Non-Fungible Token](./05-non-fungible-tokens-1.md) tutorials
to understand the building blocks of this smart contract.

## Marketplace Design

---

One way to implement a marketplace is to have a central smart contract that users deposit their NFTs and their price into,
and have anyone come by and be able to buy the token for that price.
This approach is reasonable, but it centralizes the process and takes away options from the owners.
We want users to be able to maintain ownership of the NFTs that they are trying to sell while they are trying to sell them.

Instead of taking this centralized approach, each user can list a sale from within their own account.

Then, users could either provide a link to their sale to an application that can list it centrally on a website,
or to a central sale aggregator smart contract if they want the entire transaction to stay on-chain.
This way, the owner of the token keeps custody of their token while it is on sale.

<Callout type="info">

Before we start, we need to confirm the state of your accounts. <br/>
If you haven't already, please perform the steps in the [marketplace setup guide](./07-marketplace-setup.md)
to ensure that the Fungible Token and Non-Fungible Token contracts are deployed to account 1 and 2 and own some tokens.<br/>
Your accounts should look like this:

</Callout>

<img src="https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/accounts-nft-storage.png" />

<Callout type="info">

You can run the `1. CheckSetupScript.cdc` script to ensure that your accounts are correctly set up:

</Callout>

```cadence CheckSetupScript.cdc
// CheckSetupScript.cdc

import ExampleToken from 0x01
import ExampleNFT from 0x02

// This script checks that the accounts are set up correctly for the marketplace tutorial.
//
// Account 0x01: Vault Balance = 40, NFT.id = 1
// Account 0x02: Vault Balance = 20, No NFTs
pub fun main() {
    // Get the accounts' public account objects
    let acct1 = getAccount(0x01)
    let acct2 = getAccount(0x02)

    // Get references to the account's receivers
    // by getting their public capability
    // and borrowing a reference from the capability
    let acct1ReceiverRef = acct1.getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                          .borrow<&ExampleToken.Vault{ExampleToken.Balance}>()
                          ?? panic("Could not borrow acct1 vault reference")

    let acct2ReceiverRef = acct2.getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                          .borrow<&ExampleToken.Vault{ExampleToken.Balance}>()
                          ?? panic("Could not borrow acct2 vault reference")

    // Log the Vault balance of both accounts and ensure they are
    // the correct numbers.
    // Account 0x01 should have 40.
    // Account 0x02 should have 20.
    log("Account 1 Balance")
    log(acct1ReceiverRef.balance)
    log("Account 2 Balance")
    log(acct2ReceiverRef.balance)

    // verify that the balances are correct
    if acct1ReceiverRef.balance != 40.0 || acct2ReceiverRef.balance != 20.0 {
        panic("Wrong balances!")
    }

    // Find the public Receiver capability for their Collections
    let acct1Capability = acct1.getCapability(ExampleNFT.CollectionPublicPath)
    let acct2Capability = acct2.getCapability(ExampleNFT.CollectionPublicPath)

    // borrow references from the capabilities
    let nft1Ref = acct1Capability.borrow<&{ExampleNFT.NFTReceiver}>()
        ?? panic("Could not borrow acct1 nft collection reference")

    let nft2Ref = acct2Capability.borrow<&{ExampleNFT.NFTReceiver}>()
        ?? panic("Could not borrow acct2 nft collection reference")

    // Print both collections as arrays of IDs
    log("Account 1 NFTs")
    log(nft1Ref.getIDs())

    log("Account 2 NFTs")
    log(nft2Ref.getIDs())

    // verify that the collections are correct
    if nft1Ref.getIDs()[0] != 1 || nft2Ref.getIDs().length != 0 {
        panic("Wrong Collections!")
    }
}
```

You should see something similar to this output if your accounts are set up correctly.
They are in the same state that they would have been in if you followed
the [Fungible Tokens](./06-fungible-tokens.md)
and [Non-Fungible Tokens](./05-non-fungible-tokens-1.md) tutorials in succession:

```
"Account 1 Balance"
40.00000000
"Account 2 Balance"
20.00000000
"Account 1 NFTs"
[1]
"Account 2 NFTs"
[]
```

Now that your accounts are in the correct state, we can build a marketplace that enables the sale of NFT's between accounts.

## Setting up an NFT **Marketplace**

---

Every user who wants to sell an NFT will store an instance of a `SaleCollection` resource in their account storage.

Time to deploy the marketplace contract:

<Callout type="info">

1. Switch to the ExampleMarketplace contract (Contract 3).<br/>
2. With `ExampleMarketplace.cdc` open, select account `0x03` from the deployment modal in the bottom right and deploy.

</Callout>

`ExampleMarketplace.cdc` should contain the following contract definition:

```cadence ExampleMarketplace.cdc
import ExampleToken from 0x01
import ExampleNFT from 0x02

// ExampleMarketplace.cdc
//
// The ExampleMarketplace contract is a very basic sample implementation of an NFT ExampleMarketplace on Flow.
//
// This contract allows users to put their NFTs up for sale. Other users
// can purchase these NFTs with fungible tokens.
//
// Learn more about marketplaces in this tutorial: https://developers.flow.com/cadence/tutorial/marketplace-compose
//
// This contract is a learning tool and is not meant to be used in production.
// See the NFTStorefront contract for a generic marketplace smart contract that
// is used by many different projects on the Flow blockchain:
//
// https://github.com/onflow/nft-storefront

pub contract ExampleMarketplace {

    // Event that is emitted when a new NFT is put up for sale
    pub event ForSale(id: UInt64, price: UFix64, owner: Address?)

    // Event that is emitted when the price of an NFT changes
    pub event PriceChanged(id: UInt64, newPrice: UFix64, owner: Address?)

    // Event that is emitted when a token is purchased
    pub event TokenPurchased(id: UInt64, price: UFix64, seller: Address?, buyer: Address?)

    // Event that is emitted when a seller withdraws their NFT from the sale
    pub event SaleCanceled(id: UInt64, seller: Address?)

    // Interface that users will publish for their Sale collection
    // that only exposes the methods that are supposed to be public
    //
    pub resource interface SalePublic {
        pub fun purchase(tokenID: UInt64, recipient: Capability<&AnyResource{ExampleNFT.NFTReceiver}>, buyTokens: @ExampleToken.Vault)
        pub fun idPrice(tokenID: UInt64): UFix64?
        pub fun getIDs(): [UInt64]
    }

    // SaleCollection
    //
    // NFT Collection object that allows a user to put their NFT up for sale
    // where others can send fungible tokens to purchase it
    //
    pub resource SaleCollection: SalePublic {

        /// A capability for the owner's collection
        access(self) var ownerCollection: Capability<&ExampleNFT.Collection>

        // Dictionary of the prices for each NFT by ID
        access(self) var prices: {UInt64: UFix64}

        // The fungible token vault of the owner of this sale.
        // When someone buys a token, this resource can deposit
        // tokens into their account.
        access(account) let ownerVault: Capability<&AnyResource{ExampleToken.Receiver}>

        init (ownerCollection: Capability<&ExampleNFT.Collection>,
              ownerVault: Capability<&AnyResource{ExampleToken.Receiver}>) {

            pre {
                // Check that the owner's collection capability is correct
                ownerCollection.check():
                    "Owner's NFT Collection Capability is invalid!"

                // Check that the fungible token vault capability is correct
                ownerVault.check():
                    "Owner's Receiver Capability is invalid!"
            }
            self.ownerCollection = ownerCollection
            self.ownerVault = ownerVault
            self.prices = {}
        }

        // cancelSale gives the owner the opportunity to cancel a sale in the collection
        pub fun cancelSale(tokenID: UInt64) {
            // remove the price
            self.prices.remove(key: tokenID)
            self.prices[tokenID] = nil

            // Nothing needs to be done with the actual token because it is already in the owner's collection
        }

        // listForSale lists an NFT for sale in this collection
        pub fun listForSale(tokenID: UInt64, price: UFix64) {
            pre {
                self.ownerCollection.borrow()!.idExists(id: tokenID):
                    "NFT to be listed does not exist in the owner's collection"
            }
            // store the price in the price array
            self.prices[tokenID] = price

            emit ForSale(id: tokenID, price: price, owner: self.owner?.address)
        }

        // changePrice changes the price of a token that is currently for sale
        pub fun changePrice(tokenID: UInt64, newPrice: UFix64) {
            self.prices[tokenID] = newPrice

            emit PriceChanged(id: tokenID, newPrice: newPrice, owner: self.owner?.address)
        }

        // purchase lets a user send tokens to purchase an NFT that is for sale
        pub fun purchase(tokenID: UInt64, recipient: Capability<&AnyResource{ExampleNFT.NFTReceiver}>, buyTokens: @ExampleToken.Vault) {
            pre {
                self.prices[tokenID] != nil:
                    "No token matching this ID for sale!"
                buyTokens.balance >= (self.prices[tokenID] ?? 0.0):
                    "Not enough tokens to by the NFT!"
                recipient.borrow != nil:
                    "Invalid NFT receiver capability!"
            }

            // get the value out of the optional
            let price = self.prices[tokenID]!

            self.prices[tokenID] = nil

            let vaultRef = self.ownerVault.borrow()
                ?? panic("Could not borrow reference to owner token vault")

            // deposit the purchasing tokens into the owners vault
            vaultRef.deposit(from: <-buyTokens)

            // borrow a reference to the object that the receiver capability links to
            // We can force-cast the result here because it has already been checked in the pre-conditions
            let receiverReference = receiver.borrow()!

            // deposit the NFT into the buyers collection
            receiverReference.deposit(token: <-self.ownerCollection.borrow()!.withdraw(withdrawID: tokenID))

            emit TokenPurchased(id: tokenID, price: price, owner: self.owner?.address, buyer: receiverReference.owner?.address)
        }

        // idPrice returns the price of a specific token in the sale
        pub fun idPrice(tokenID: UInt64): UFix64? {
            return self.prices[tokenID]
        }

        // getIDs returns an array of token IDs that are for sale
        pub fun getIDs(): [UInt64] {
            return self.prices.keys
        }
    }

    // createCollection returns a new collection resource to the caller
    pub fun createSaleCollection(ownerCollection: Capability<&ExampleNFT.Collection>,
                                 ownerVault: Capability<&AnyResource{ExampleToken.Receiver}>): @SaleCollection {
        return <- create SaleCollection(ownerCollection: ownerCollection, ownerVault: ownerVault)
    }
}
```

This marketplace contract has resources that function similarly to the NFT `Collection`
that was explained in [Non-Fungible Tokens](./05-non-fungible-tokens-1.md), with a few differences and additions:

- This marketplace contract has methods to add and remove NFTs, but instead of storing the NFT resource object in the sale collection,
  the user provides a capability to their main collection that allows the listed NFT to be withdrawn and transferred when it is purchased.
  When a user wants to put their NFT up for sale, they do so by providing the ID and the price to the `listForSale` function.
  Then, another user can call the `purchase` method, sending their `ExampleToken.Vault` that contains the currency they are using to make the purchase.
  The buyer also includes a capability to their NFT `ExampleNFT.Collection` so that the purchased token
  can be immediately deposited into their collection when the purchase is made.
- This marketplace contract stores a capability: `pub let ownerVault: Capability<&AnyResource{FungibleToken.Receiver}>`.
  The owner of the sale saves a capability to their Fungible Token `Receiver` within the sale.
  This allows the sale resource to be able to immediately deposit the currency that was used to buy the NFT
  into the owners `Vault` when a purchase is made.
- This marketplace contract includes events. Cadence supports defining events within contracts
  that can be emitted when important actions happen. External apps can monitor these events to know the state of the smart contract.

```cadence
    // Event that is emitted when a new NFT is put up for sale
    pub event ForSale(id: UInt64, price: UFix64, owner: Address?)

    // Event that is emitted when the price of an NFT changes
    pub event PriceChanged(id: UInt64, newPrice: UFix64, owner: Address?)

    // Event that is emitted when a token is purchased
    pub event TokenPurchased(id: UInt64, price: UFix64, seller: Address?, buyer: Address?)

    // Event that is emitted when a seller withdraws their NFT from the sale
    pub event SaleCanceled(id: UInt64, seller: Address?)
```

This contract has a few new features and concepts that are important to cover:

### Events

[Events](../language/events) are special values that can be emitted during the execution of a program.
They usually contain information to indicate that some important action has happened in a smart contract,
such as an NFT transfer, a permission change, or many other different things.
Off-chain applications can monitor events using a Flow SDK to know what is happening on-chain without having to query a smart contract directly.

Many applications want to maintain an off-chain record of what is happening on-chain so they can have faster performance
when getting information about their users' accounts or generating analytics.

Events are declared by indicating [the access level](../language/access-control), `event`,
and the name and parameters of the event, like a function declaration:
```cadence
pub event ForSale(id: UInt64, price: UFix64, owner: Address?)
```

Events cannot modify state at all; they indicate when important actions happen in the smart contract.

Events are emitted with the `emit` keyword followed by the invocation of the event as if it were a function call.
```cadence
emit ForSale(id: tokenID, price: price, owner: self.owner?.address)
```

External applications can monitor the blockchain to take action when certain events are emitted.

### Resource-Owned Capabilities

We have covered capabilities in previous [tutorials](./04-capabilities.md),
but only the basics. Capabilities can be used for so much more!

As you hopefully understand, [capabilites](../language/capabilities.md)
are links to private objects in account storage that specify and expose a subset in the public or private namespace of public or private paths
where the Capability is linked.

To create a capability, a user typically uses [the `AuthAccount.link`](../language/accounts.mdx#authaccount)
method to create a link to a resource in their private storage, specifying a type to link the capability as:

```cadence
// Create a public Receiver + Balance capability to the Vault
// acct is an `AuthAccount`
// The object being linked to has to be an `ExampleToken.Vault`,
// and the link only exposes the fields in the `ExampleToken.Receiver` and `ExampleToken.Balance` interfaces.
acct.link<&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}>
    (/public/CadenceFungibleTokenTutorialReceiver, target: /storage/CadenceFungibleTokenTutorialVault)
```

Then, users can get that capability if it was created [in a public path](../language/accounts.mdx#paths),
borrow it, and access the functionality that the owner specified.

```cadence
// Get account 0x01's PublicAccount object
let publicAccount = getAccount(0x01)

// Retrieve a Vault Receiver Capability from the account's public storage
let acct1Capability = acct.getCapability<&AnyResource{ExampleToken.Receiver}>(
        /public/CadenceFungibleTokenTutorialReceiver
    )

// Borrow a reference
let acct1ReceiverRef = acct1Capability.borrow()
    ?? panic("Could not borrow a receiver reference to the vault")

// Deposit tokens
acct1ReceiverRef.deposit(from: <-tokens)
```

With the marketplace contract, we are utilizing a new feature of capabilities.
Capabilities can be stored anywhere! Lots of functionality is contained within resources,
and developers will sometimes want to be able to access some of the functionality of resources from within different resources or contracts.

We store two different capabilities in the marketplace sale collection:

```cadence
/// A capability for the owner's collection
access(self) var ownerCollection: Capability<&ExampleNFT.Collection>

// The fungible token vault of the owner of this sale.
// When someone buys a token, this resource can deposit
// tokens into their account.
access(account) let ownerVault: Capability<&AnyResource{ExampleToken.Receiver}>
```

If an object like a contract or resource owns a capability, they can borrow a reference to that capability at any time
to access that functionality without having to get it from the owner's account every time.

This is especially important if the owner wants to expose some functionality that is only intended for one person,
meaning that the link for the capability is not stored in a public path.
We do that in this example, because the sale collection stores a capability that can access all of the functionality
of the `ExampleNFT.Collection`. It needs this because it withdraws the specified NFT in the `purchase()` method to send to the buyer.

It is important to remember that control of a capability does not equal ownership of the underlying resource.
You can use the capability to access that resource's functionality, but you can't use it to fake ownership.
You need the actual resource (identified by the prefixed `@` symbol) to prove ownership.

Additionally, these capabilities can be stored anywhere, but if a user decides that they no longer want the capability
to be used, they can revoke it with the `AuthAccount.unlink()` method so any capabilities that use that link are rendered invalid.

One last piece to consider about capabilities is the decision about when to use them instead of storing the resource directly.
This tutorial used to have the `SaleCollection` directly store the NFTs that were for sale, like so:

```cadence
pub resource SaleCollection: SalePublic {

    /// Dictionary of NFT objects for sale
    /// Maps ID to NFT resource object
    /// Not recommended
    access(self) var forSale: @{UInt64: ExampleNFT.NFT}
}
```

This is a logical way to do it, and illustrates another important concept in Cadence, that resources can own other resources!
Check out the [Kitty Hats tutorial](./10-resources-compose.md) for a little more exploration of this concept.

In this case however, nesting resources doesn't make sense. If a user decides to store their for-sale NFTs in a separate place from their main collection,
then those NFTs are not available to be shown to any app or smart contract that queries the main collection,
so it is as if the owner doesn't actually own the NFT!

In cases like this, we usually recommend using a capability to the main collection so that the main collection can remain unchanged and fully usable by
other smart contracts and apps. This also means that if a for-sale NFT gets transferred by some means other than a purchase, then you need a way to get 
rid of the stale listing. That is out of the scope of this tutorial though.

Enough explaining! Lets execute some code!

## Using the Marketplace

At this point, we should have an `ExampleToken.Vault` and an `Example.NFT.Collection` in both accounts' storage.
Account `0x01` should have an NFT in their collection and the `ExampleMarketplace` contract should be deployed to `0x03`.

You can create a `SaleCollection` and list account `0x01`'s token for sale by following these steps:

<Callout type="info">

1. Open Transaction 4, `CreateSale.cdc` <br/>
2. Select account `0x01` as the only signer and click the `Send` button to submit the transaction.

</Callout>

```cadence Transaction4.cdc
// CreateSale.cdc

import ExampleToken from 0x01
import ExampleNFT from 0x02
import ExampleMarketplace from 0x03

// This transaction creates a new Sale Collection object,
// lists an NFT for sale, puts it in account storage,
// and creates a public capability to the sale so that others can buy the token.
transaction {

    prepare(acct: AuthAccount) {

        // Borrow a reference to the stored Vault
        let receiver = acct.getCapability<&{ExampleToken.Receiver}>(/public/CadenceFungibleTokenTutorialReceiver)

        // borrow a reference to the nftTutorialCollection in storage
        let collectionCapability = acct.link<&ExampleNFT.Collection>(/private/nftTutorialCollection, target: ExampleNFT.CollectionStoragePath)
          ?? panic("Unable to create private link to NFT Collection")

        // Create a new Sale object,
        // initializing it with the reference to the owner's vault
        let sale <- ExampleMarketplace.createSaleCollection(ownerCollection: collectionCapability, ownerVault: receiver)

        // List the token for sale by moving it into the sale object
        sale.listForSale(tokenID: 1, price: 10.0)

        // Store the sale object in the account storage
        acct.save(<-sale, to: /storage/NFTSale)

        // Create a public capability to the sale so that others can call its methods
        acct.link<&ExampleMarketplace.SaleCollection{ExampleMarketplace.SalePublic}>(/public/NFTSale, target: /storage/NFTSale)

        log("Sale Created for account 1. Selling NFT 1 for 10 tokens")
    }
}
```

This transaction:

1. Gets a `Receiver` capability on the owners `Vault`.
1. Gets a private `ExampleNFT.Collection` Capability from the owner.
1. Creates the `SaleCollection`, which stores their `Vault` and `ExampleNFT.Collection` capabilities.
1. Lists the token with `ID = 1` for sale and sets its price as 10.0.
1. Stores the `SaleCollection` in their account storage and links a public capability that allows others to purchase any NFTs for sale.

Let's run a script to ensure that the sale was created correctly.

1. Open Script 2: `GetSaleIDs.cdc`
1. Click the `Execute` button to print the ID and price of the NFT that account `0x01` has for sale.

```cadence GetSaleIDs.cdc
// GetSaleIDs.cdc

import ExampleToken from 0x01
import ExampleNFT from 0x02
import ExampleMarketplace from 0x03

// This script prints the NFTs that account 0x01 has for sale.
pub fun main() {
    // Get the public account object for account 0x01
    let account1 = getAccount(0x01)

    // Find the public Sale reference to their Collection
    let acct1saleRef = account1.getCapability(/public/NFTSale)
                       .borrow<&AnyResource{ExampleMarketplace.SalePublic}>()
                       ?? panic("Could not borrow acct2 nft sale reference")

    // Los the NFTs that are for sale
    log("Account 1 NFTs for sale")
    log(acct1saleRef.getIDs())
    log("Price")
    log(acct1saleRef.idPrice(tokenID: 1))
}
```

This script should complete and print something like this:

```
"Account 1 NFTs for sale"
[1]
"Price"
10
```

## Purchasing an NFT

---

The buyer can now purchase the seller's NFT by using the transaction in `Transaction2.cdc`:

<Callout type="info">

1. Open Transaction 5: `PurchaseSale.cdc` file<br/>
2. Select account `0x02` as the only signer and click the `Send` button

</Callout>

```cadence PurchaseSale.cdc
// PurchaseSale.cdc

import ExampleToken from 0x01
import ExampleNFT from 0x02
import ExampleMarketplace from 0x03

// This transaction uses the signers Vault tokens to purchase an NFT
// from the Sale collection of account 0x01.
transaction {

    // Capability to the buyer's NFT collection where they
    // will store the bought NFT
    let collectionCapability: Capability<&AnyResource{ExampleNFT.NFTReceiver}>

    // Vault that will hold the tokens that will be used to
    // but the NFT
    let temporaryVault: @ExampleToken.Vault

    prepare(acct: AuthAccount) {

        // get the references to the buyer's fungible token Vault and NFT Collection Receiver
        self.collectionCapability = acct.getCapability<&AnyResource{ExampleNFT.NFTReceiver}>(from: ExampleNFT.CollectionPublicPath)

        let vaultRef = acct.borrow<&ExampleToken.Vault>(from: /storage/CadenceFungibleTokenTutorialVault)
            ?? panic("Could not borrow owner's vault reference")

        // withdraw tokens from the buyers Vault
        self.temporaryVault <- vaultRef.withdraw(amount: 10.0)
    }

    execute {
        // get the read-only account storage of the seller
        let seller = getAccount(0x01)

        // get the reference to the seller's sale
        let saleRef = seller.getCapability(/public/NFTSale)!
                            .borrow<&AnyResource{ExampleMarketplace.SalePublic}>()
                            ?? panic("Could not borrow seller's sale reference")

        // purchase the NFT the seller is selling, giving them the capability
        // to your NFT collection and giving them the tokens to buy it
        saleRef.purchase(tokenID: 1, recipient: self.collectionCapability, buyTokens: <-self.temporaryVault)

        log("Token 1 has been bought by account 2!")
    }
}
```

This transaction:

1. Gets the capability to the buyer's NFT receiver
1. Get a reference to their token vault and withdraws the sale purchase amount
1. Gets the public account object for account `0x01`
1. Gets the reference to the seller's public sale
1. Calls the `purchase` function, passing in the tokens and the `Collection` reference. Then `purchase` deposits the bought NFT directly into the buyer's collection.

## Verifying the NFT Was Purchased Correctly

---

You can run now run a script to verify that the NFT was purchased correctly because:

- account `0x01` has 50 tokens and does not have any NFTs for sale or in their collection and account
- account `0x02` has 10 tokens and an NFT with id=1

To run a script that verifies the NFT was purchased correctly, follow these steps:

<Callout type="info">

1. Open Script 3: `VerifyAfterPurchase.cdc`<br/>
2. Click the `Execute` button

</Callout>

`VerifyAfterPurchase.cdc` should contain the following code:

```cadence Script3.cdc
// VerifyAfterPurchase.cdc

import ExampleToken from 0x01
import ExampleNFT from 0x02
import ExampleMarketplace from 0x03

// This script checks that the Vault balances and NFT collections are correct
// for both accounts.
//
// Account 1: Vault balance = 50, No NFTs
// Account 2: Vault balance = 10, NFT ID=1
pub fun main() {
    // Get the accounts' public account objects
    let acct1 = getAccount(0x01)
    let acct2 = getAccount(0x02)

    // Get references to the account's receivers
    // by getting their public capability
    // and borrowing a reference from the capability
    let acct1ReceiverRef = acct1.getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                           .borrow<&ExampleToken.Vault{ExampleToken.Balance}>()
                           ?? panic("Could not borrow acct1 vault reference")

    let acct2ReceiverRef = acct2.getCapability(/public/CadenceFungibleTokenTutorialReceiver)
                            .borrow<&ExampleToken.Vault{ExampleToken.Balance}>()
                            ?? panic("Could not borrow acct2 vault reference")

    // Log the Vault balance of both accounts and ensure they are
    // the correct numbers.
    // Account 0x01 should have 50.
    // Account 0x02 should have 10.
    log("Account 1 Balance")
	log(acct1ReceiverRef.balance)
    log("Account 2 Balance")
    log(acct2ReceiverRef.balance)

    // verify that the balances are correct
    if acct1ReceiverRef.balance != 50.0 || acct2ReceiverRef.balance != 10.0 {
        panic("Wrong balances!")
    }

    // Find the public Receiver capability for their Collections
    let acct1Capability = acct1.getCapability(ExampleNFT.CollectionPublicPath)
    let acct2Capability = acct2.getCapability(ExampleNFT.CollectionPublicPath)

    // borrow references from the capabilities
    let nft1Ref = acct1Capability.borrow<&{ExampleNFT.NFTReceiver}>()
        ?? panic("Could not borrow acct1 nft collection reference")

    let nft2Ref = acct2Capability.borrow<&{ExampleNFT.NFTReceiver}>()
        ?? panic("Could not borrow acct2 nft collection reference")

    // Print both collections as arrays of IDs
    log("Account 1 NFTs")
    log(nft1Ref.getIDs())

    log("Account 2 NFTs")
    log(nft2Ref.getIDs())

    // verify that the collections are correct
    if nft2Ref.getIDs()[0] != 1 || nft1Ref.getIDs().length != 0 {
        panic("Wrong Collections!")
    }

    // Get the public sale reference for Account 0x01
    let acct1SaleRef = acct1.getCapability(/public/NFTSale)
                       .borrow<&AnyResource{ExampleMarketplace.SalePublic}>()
                       ?? panic("Could not borrow acct1 nft sale reference")

    // Print the NFTs that account 0x01 has for sale
    log("Account 1 NFTs for sale")
    log(acct1SaleRef.getIDs())
    if acct1SaleRef.getIDs().length != 0 { panic("Sale should be empty!") }
}
```

If you did everything correctly, the transaction should succeed and it should print something similar to this:

```
"Account 1 Vault Balance"
50
"Account 2 Vault Balance"
10
"Account 1 NFTs"
[]
"Account 2 NFTs"
[1]
"Account 1 NFTs for Sale"
[]
```

Congratulations, you have successfully implemented a simple marketplace in Cadence and used it to allow one account to buy an NFT from another!

## Scaling the Marketplace

---

A user can hold a sale in their account with these resources and transactions.
Support for a central marketplace where users can discover sales is relatively easy to implement and can build on what we already have.
If we wanted to build a central marketplace on-chain, we could use a contract that looks something like this:

```cadence CentralMarketplace.cdc
// Marketplace would be the central contract where people can post their sale
// references so that anyone can access them
pub contract Marketplace {
    // Data structure to store active sales
    pub var tokensForSale: {Address: Capability<&SaleCollection>)}

    // listSaleCollection lists a users sale reference in the array
    // and returns the index of the sale so that users can know
    // how to remove it from the marketplace
    pub fun listSaleCollection(collection: Capability<&SaleCollection>) {
        let saleRef = collection.borrow()
            ?? panic("Invalid sale collection capability")

        self.tokensForSale[saleRef.owner!.address] = collection
    }

    // removeSaleCollection removes a user's sale from the array
    // of sale references
    pub fun removeSaleCollection(owner: Address) {
        self.tokensForSale[owner] = nil
    }

}
```

This contract isn't meant to be a working or production-ready contract, but it could be extended to make a complete central marketplace by having:

- Sellers list a capability to their `SaleCollection` in this contract
- Other functions that buyers could call to get info about all the different sales and to make purchases.

A central marketplace in an off-chain application is easier to implement because:

- The app could host the marketplace and a user would simply log in to the app and give the app its account address.
- The app could read the user's public storage and find their sale reference.
- With the sale reference, the app could get all the information they need about how to display the sales on their website.
- Any buyer could discover the sale in the app and login with their account, which gives the app access to their public references.
- When the buyer wants to buy a specific NFT, the app would automatically generate the proper transaction to purchase the NFT from the seller.

## Creating a **Marketplace for Any Generic NFT**

---

The previous examples show how a simple marketplace could be created for a specific class of NFTs.
However, users will want to have a marketplace where they can buy and sell any NFT they want, regardless of its type.
There are a few good examples of generic marketplaces on Flow right now.

- The Flow team has created a completely decentralized example of a generic marketplace in the [NFT storefront repo.](https://github.com/onflow/nft-storefront)
  This contract is already deployed to testnet and mainnet and can be used by anyone for any generic NFT sale!
- [VIV3](https://viv3.com/) is a company that has a generic NFT marketplace.


## Composable Resources on Flow

---

Now that you have an understanding of how composable smart contracts and the marketplace work on Flow, you're ready to play with composable resources!
Check out the [Kitty Hats tutorial!](./10-resources-compose.md)
---
title: 9. Voting Contract
---

In this tutorial, we're going to deploy a contract that allows users to vote on multiple proposals that a voting administrator controls.

---

<Callout type="success">
  Open the starter code for this tutorial in the Flow Playground: 
  <a 
    href="https://play.onflow.org/d120f0a7-d411-4243-bc59-5125a84f99b3" 
    target="_blank"
  >
    https://play.onflow.org/d120f0a7-d411-4243-bc59-5125a84f99b3
  </a>
  <br/>
  The tutorial will be asking you to take various actions to interact with this code.
</Callout>

<Callout type="info">
Instructions that require you to take action are always included in a callout box like this one.
These highlighted actions are all that you need to do to get your code running,
but reading the rest is necessary to understand the language's design.
</Callout>

With the advent of blockchain technology and smart contracts,
it has become popular to try to create decentralized voting mechanisms that allow large groups of users to vote completely on chain.
This tutorial will provide a trivial example for how this might be achieved by using a resource-oriented programming model.

We'll take you through these steps to get comfortable with the Voting contract.

1. Deploy the contract to account `0x01`
2. Create proposals for users to vote on
3. Use a transaction with multiple signers to directly transfer the `Ballot` resource to another account.
4. Record and cast your vote in the central Voting contract
5. Read the results of the vote

Before proceeding with this tutorial, we highly recommend following the instructions in [Getting Started](./01-first-steps.md)
and [Hello, World!](./02-hello-world.md) to learn how to use the Playground tools and to learn the fundamentals of Cadence.

## A Voting Contract in Cadence

In this contract, a Ballot is represented as a resource.

An administrator can give Ballots to other accounts, then those accounts mark which proposals they vote for
and submit the Ballot to the central smart contract to have their votes recorded.

Using a [resource](../language/resources.mdx) type is logical for this application, because if a user wants to delegate their vote,
they can send that Ballot to another account, and the use case of voting ballots benefits from the uniqueness and existence guarantees
inherent to resources.

## Deploy the Contract

Time to deploy the contract we'll be working with:

<Callout type="info">

1. Open Contract 1 - the `ApprovalVoting` contract.<br/>
2. In the bottom right deployment modal, press the arrow to expand and make sure account `0x01` is selected as the signer.<br/>
3. Click the Deploy button to deploy it to account `0x01`

</Callout>

![Deploy ApprovalVoting to account 0x01](deploy_approval_voting.png)

The deployed contract should have the following contents:

```cadence ApprovalVoting.cdc
/*
*
*   In this example, we want to create a simple approval voting contract
*   where a polling place issues ballots to addresses.
*
*   The run a vote, the Admin deploys the smart contract,
*   then initializes the proposals
*   using the initialize_proposals.cdc transaction.
*   The array of proposals cannot be modified after it has been initialized.
*
*   Then they will give ballots to users by
*   using the issue_ballot.cdc transaction.
*
*   Every user with a ballot is allowed to approve any number of proposals.
*   A user can choose their votes and cast them
*   with the cast_vote.cdc transaction.
*
*/

pub contract ApprovalVoting {

    //list of proposals to be approved
    pub var proposals: [String]

    // number of votes per proposal
    pub let votes: {Int: Int}

    // This is the resource that is issued to users.
    // When a user gets a Ballot object, they call the `vote` function
    // to include their votes, and then cast it in the smart contract
    // using the `cast` function to have their vote included in the polling
    pub resource Ballot {

        // array of all the proposals
        pub let proposals: [String]

        // corresponds to an array index in proposals after a vote
        pub var choices: {Int: Bool}

        init() {
            self.proposals = ApprovalVoting.proposals
            self.choices = {}

            // Set each choice to false
            var i = 0
            while i < self.proposals.length {
                self.choices[i] = false
                i = i + 1
            }
        }

        // modifies the ballot
        // to indicate which proposals it is voting for
        pub fun vote(proposal: Int) {
            pre {
                self.proposals[proposal] != nil: "Cannot vote for a proposal that doesn't exist"
            }
            self.choices[proposal] = true
        }
    }

    // Resource that the Administrator of the vote controls to
    // initialize the proposals and to pass out ballot resources to voters
    pub resource Administrator {

        // function to initialize all the proposals for the voting
        pub fun initializeProposals(_ proposals: [String]) {
            pre {
                ApprovalVoting.proposals.length == 0: "Proposals can only be initialized once"
                proposals.length > 0: "Cannot initialize with no proposals"
            }
            ApprovalVoting.proposals = proposals

            // Set each tally of votes to zero
            var i = 0
            while i < proposals.length {
                ApprovalVoting.votes[i] = 0
                i = i + 1
            }
        }

        // The admin calls this function to create a new Ballot
        // that can be transferred to another user
        pub fun issueBallot(): @Ballot {
            return <-create Ballot()
        }
    }

    // A user moves their ballot to this function in the contract where
    // its votes are tallied and the ballot is destroyed
    pub fun cast(ballot: @Ballot) {
        var index = 0
        // look through the ballot
        while index < self.proposals.length {
            if ballot.choices[index]! {
                // tally the vote if it is approved
                self.votes[index] = self.votes[index]! + 1
            }
            index = index + 1;
        }
        // Destroy the ballot because it has been tallied
        destroy ballot
    }

    // initializes the contract by setting the proposals and votes to empty
    // and creating a new Admin resource to put in storage
    init() {
        self.proposals = []
        self.votes = {}

        self.account.save<@Administrator>(<-create Administrator(), to: /storage/VotingAdmin)
    }
}

```

This contract implements a simple voting mechanism where an `Administrator` can initialize a vote with an array of proposals to vote on by using the `initializeProposals` function.

```cadence
// function to initialize all the proposals for the voting
pub fun initializeProposals(_ proposals: [String]) {
    pre {
        ApprovalVoting.proposals.length == 0: "Proposals can only be initialized once"
        proposals.length > 0: "Cannot initialize with no proposals"
    }
    ApprovalVoting.proposals = proposals

    // Set each tally of votes to zero
    var i = 0
    while i < proposals.length {
        ApprovalVoting.votes[i] = 0
        i = i + 1
    }
}
```

Then they can give `Ballot` resources to other accounts. The other accounts can record their votes on their `Ballot` resource by calling the `vote` function.

```cadence
pub fun vote(proposal: Int) {
    pre {
        self.proposals[proposal] != nil: "Cannot vote for a proposal that doesn't exist"
    }
    self.choices[proposal] = true
}
```

After a user has voted, they submit their vote to the central smart contract by calling the `cast` function, which records the votes in the `Ballot` and destroys the used `Ballot`.

```cadence
// A user moves their ballot to this function in the contract where
// its votes are tallied and the ballot is destroyed
pub fun cast(ballot: @Ballot) {
    var index = 0
    // look through the ballot
    while index < self.proposals.length {
        if ballot.choices[index]! {
            // tally the vote if it is approved
            self.votes[index] = self.votes[index]! + 1
        }
        index = index + 1;
    }
    // Destroy the ballot because it has been tallied
    destroy ballot
}
```

When the voting time ends, the administrator can read the tallies for each proposal to see if a proposal has received the right number of votes.

## Perform Voting

Performing the common actions in this voting contract only takes three types of transactions.

1. Initialize Proposals
2. Send `Ballot` to a voter
3. Cast Vote

We have a transaction for each step that we provide for you. With the `ApprovalVoting` contract to account `0x01`: 

<Callout type="info">

1. Open Transaction 1 which should have `Transaction1.cdc`<br/>
2. Submit the transaction with account `0x01` selected as the only signer.

</Callout>

```cadence Transaction1.cdc
import ApprovalVoting from 0x01

// This transaction allows the administrator of the Voting contract
// to create new proposals for voting and save them to the smart contract

transaction {
    prepare(admin: AuthAccount) {

        // borrow a reference to the admin Resource
        let adminRef = admin.borrow<&ApprovalVoting.Administrator>(from: /storage/VotingAdmin)!

        // Call the initializeProposals function
        // to create the proposals array as an array of strings
        adminRef.initializeProposals(
            ["Longer Shot Clock", "Trampolines instead of hardwood floors"]
        )

        log("Proposals Initialized!")
    }

    post {
        ApprovalVoting.proposals.length == 2
    }

}
```

This transaction allows the `Administrator` of the contract to create new proposals for voting and save them to the smart contract. They do this by calling the `initializeProposals` function on their stored `Administrator` resource, giving it two new proposals to vote on.
We use the `post` block to ensure that there were two proposals created, like we wished for.

Next, the `Administrator` needs to hand out `Ballot`s to the voters. There isn't an easy `deposit` function this time for them to send a `Ballot` to another account, so how would they do it?

This is where multi-signed transactions can come in handy!

## Selecting multiple Accounts as Signers

A transaction has access to the private account objects of every account that signed it, so if both the admin and the voter sign a transaction, the admin can directly move a `Ballot` resource object to the other account's storage.

In the Flow playground, you can select multiple accounts to sign a transaction to be able to access the private account objects of both accounts.

To select multiple signers, you first need to include two arguments in the `prepare` block of your transaction:

`prepare(acct1: AuthAccount, acct2: AuthAccount)`

The playground will give you an error if the number of selected signers is different than the number of arguments to the prepare block. The playground also maps the accounts you select as signers to the arguments in the order that you select them. The first account you select will be the first argument, and the second account you select is the second argument.

<Callout type="info">

1. Open Transaction 2 which should have `Transaction2.cdc`.<br/>
2. Select account `0x01` as a signer first, then also select account `0x02`.<br/>
3. Submit the transaction by clicking the `Send` button

</Callout>

```cadence Transaction2.cdc

import ApprovalVoting from 0x01

// This transaction allows the administrator of the Voting contract
// to create a new ballot and store it in a voter's account
// The voter and the administrator have to both sign the transaction
// so it can access their storage

transaction {
    prepare(admin: AuthAccount, voter: AuthAccount) {

        // borrow a reference to the admin Resource
        let adminRef = admin.borrow<&ApprovalVoting.Administrator>(from: /storage/VotingAdmin)!

        // create a new Ballot by calling the issueBallot
        // function of the admin Reference
        let ballot <- adminRef.issueBallot()

        // store that ballot in the voter's account storage
        voter.save<@ApprovalVoting.Ballot>(<-ballot, to: /storage/Ballot)

        log("Ballot transferred to voter")
    }
}

```

This transaction has two signers as `prepare` parameters, so it is able to access both of their private `AuthAccount` objects, and therefore their private account storage.

Because of this, we can perform a direct transfer of the `Ballot` by creating it with the admin's `issueBallot` function and then directly store it in the voter's storage by using the `save` function.

Account `0x02` should now have a `Ballot` resource object in its account storage. You can confirm this by selecting `0x02` from the lower-left sidebar and seeing `Ballot` resource listed under the `Storage` field.

## Casting a Vote

Now that account `0x02` has a `Ballot` in their storage, they can cast their vote. To do this, they will call the `vote` method on their stored resource, then cast that `Ballot` by passing it to the `cast` function in the main smart contract.

<Callout type="info">

1. Open Transaction 3 which should contain `Transaction3.cdc`.<br/>
2. Select account `0x02` as the only transaction signer.<br/>
3. Click the `send` button to submit the transaction.

</Callout>

```cadence Transaction3.cdc
import ApprovalVoting from 0x01

// This transaction allows a voter to select the votes they would like to make
// and cast that vote by using the castVote function
// of the ApprovalVoting smart contract

transaction {
    prepare(voter: AuthAccount) {

        // take the voter's ballot our of storage
        let ballot <- voter.load<@ApprovalVoting.Ballot>(from: /storage/Ballot)!

        // Vote on the proposal
        ballot.vote(proposal: 1)

        // Cast the vote by submitting it to the smart contract
        ApprovalVoting.cast(ballot: <-ballot)

        log("Vote cast and tallied")
    }
}
```

In this transaction, the user votes for one of the proposals, and then moves their Ballot back to the smart contract via the `cast()` method where the vote is tallied.

## Reading the result of the vote

At any time, anyone could read the current tally of votes by directly reading the fields of the contract. You can use a script to do that, since it does not need to modify storage.

<Callout type="info">

1. Open a Script 1 which should contain the code below.<br/>
2. Click the `execute` button to run the script.

</Callout>

```cadence Script1.cdc
import ApprovalVoting from 0x01

// This script allows anyone to read the tallied votes for each proposal
//

pub fun main() {

    // Access the public fields of the contract to log
    // the proposal names and vote counts

    log("Number of Votes for Proposal 1:")
    log(ApprovalVoting.proposals[0])
    log(ApprovalVoting.votes[0])

    log("Number of Votes for Proposal 2:")
    log(ApprovalVoting.proposals[1])
    log(ApprovalVoting.votes[1])

}
```

You should see something like this print:

```
"Number of Votes for Proposal 1:"
"Longer Shot Clock"
0
"Number of Votes for Proposal 2:"
"Trampolines instead of hardwood floors"
1
```

This shows that one vote was cast for proposal 1 and no votes were cast for proposal 2.

## Other Voting possibilities

This contract was a very simple example of voting in Cadence. It clearly couldn't be used for a real-world voting situation, but hopefully you can see what kind of features could be added to it to ensure practicality and security.
---
title: 10. Composable Resources
---

In this tutorial, we're going to walk through how resources can own other resources by creating, deploying, and moving composable NFTs.

---

<Callout type="success">
  Open the starter code for this tutorial in the Flow Playground:
  <a 
    href="https://play.onflow.org/01f812d7-799a-42fd-b9cb-9ffe556e02ad"
    target="_blank"
  >
    https://play.onflow.org/01f812d7-799a-42fd-b9cb-9ffe556e02ad
  </a>
  <br/>
  The tutorial will be asking you do take various actions to interact with this code.
</Callout>

<Callout type="info">
Instructions that require you to take action are always included in a callout box like this one.
These highlighted actions are all that you need to do to get your code running,
but reading the rest is necessary to understand the language's design.
</Callout>

Resources owning other resources is a powerful feature in the world of blockchain and smart contracts.
To showcase how this feature works on Flow, this tutorial will take you through these steps with a composable NFT:

1. Deploy the `Kitty` and `KittyHat` definitions to account `0x01`
2. Create a `Kitty` and two `KittyHat`s and store them in your account
3. Move the Kitties and Hats around to see how composable NFTs function on Flow

**Before proceeding with this tutorial**, we recommend following the instructions in [Getting Started](./01-first-steps.md)
and [Hello, World!](./02-hello-world.md) to learn about the Playground and Cadence.


## Resources Owning Resources

---

The NFT collections talked about in [Non-Fungible Tokens](./05-non-fungible-tokens-1.md) are examples of resources that own other resources.
We have a resource, the NFT collection, that has ownership of the NFT resources that are stored within it.
The owner and anyone with a reference can move these resources around,
but they still belong to the collection while they are in it and the code defined in the collection has ultimate control over the resources.

When the collection is moved or destroyed, all of the NFTs inside of it are moved or destroyed with it.

If the owner of the collection transferred the whole collection resource to another user's account,
all of the tokens will move to the other user's account with it. The tokens don't stay in the original owner's account.
This is like handing someone your wallet instead of just a dollar bill. It isn't a common action, but certainly is possible.

References cannot be created for resources that are stored in other resources.
The owning resource has control over it and therefore controls the type of access that external calls have on the stored resource.

## Resources Owning Resources: An Example

---

The NFT collection is a simple example of how resources can own other resources, but innovative and more powerful versions can be made.

An important feature of CryptoKitties (and other applications on the Ethereum blockchain) is that any developer can make new experiences around the existing application.
Even though the original contract didn't include specific support for CryptoKitty accessories (like hats), an independent developer was still able to make hats that Kitties from the original contract could use.

Here is a basic example of how we can replicate this feature in Cadence:


<Callout type="info">

1. Open Contract 1, the `KittyVerse.cdc` contract<br/>
2. In the bottom right deployment modal, press the arrow to expand and make sure account `0x01` is selected as the signer.<br/>
3. Click the Deploy button to deploy the contract to account `0x01`

</Callout>

![Deploy KittyVerse to account 0x01](deploy_kittyverse.png)

The deployed contract should have the following contents:

```cadence KittyVerse.cdc
// KittyVerse.cdc
//
// The KittyVerse contract defines two types of NFTs.
// One is a KittyHat, which represents a special hat, and
// the second is the Kitty resource, which can own Kitty Hats.
//
// You can put the hats on the cats and then call a hat function
// that tips the hat and prints a fun message.
//
// This is a simple example of how Cadence supports
// extensibility for smart contracts, but the language will soon
// support even more powerful versions of this.
//

pub contract KittyVerse {

    // KittyHat is a special resource type that represents a hat
    pub resource KittyHat {
        pub let id: Int
        pub let name: String

        init(id: Int, name: String) {
            self.id = id
            self.name = name
        }

        // An example of a function someone might put in their hat resource
        pub fun tipHat(): String {
            if self.name == "Cowboy Hat" {
                return "Howdy Y'all"
            } else if self.name == "Top Hat" {
                return "Greetings, fellow aristocats!"
            }

            return "Hello"
        }
    }

    // Create a new hat
    pub fun createHat(id: Int, name: String): @KittyHat {
        return <-create KittyHat(id: id, name: name)
    }

    pub resource Kitty {

        pub let id: Int

        // place where the Kitty hats are stored
        pub var items: @{String: KittyHat}

        init(newID: Int) {
            self.id = newID
            self.items <- {}
        }

        pub fun getKittyItems(): @{String: KittyHat} {
            var other: @{String:KittyHat} <- {}
            self.items <-> other
            return <- other
        }

        pub fun setKittyItems(items: @{String: KittyHat}) {
            var other <- items
            self.items <-> other
            destroy other
        }

        pub fun removeKittyItem(key: String): @KittyHat? {
            var removed <- self.items.remove(key: key)
            return <- removed
        }

        destroy() {
            destroy self.items
        }
    }

    pub fun createKitty(): @Kitty {
        return <-create Kitty(newID: 1)
    }

}

```

These definitions show how a Kitty resource could own hats.

The hats are stored in a variable in the Kitty resource.

```cadence
    // place where the Kitty hats are stored
    pub var items: <-{String: KittyHat}
```

A Kitty owner can take the hats off the Kitty and transfer them individually. Or the owner can transfer a Kitty that owns a hat, and the hat will go along with the Kitty.

Here is a transaction to create a `Kitty` and a `KittyHat`, store the hat in the Kitty, then store it in your account storage.

1. Open `Transaction1.cdc`.
1. Select account `0x01` as the only signer.
1. Send the transaction by clicking the Send button.

The transaction you sent just executed the following code:

```cadence Transaction1.cdc
import KittyVerse from 0x01

// This transaction creates a new kitty, creates two new hats and
// puts the hats on the cat. Then it stores the kitty in account storage.
transaction {
    prepare(acct: AuthAccount) {

        // Create the Kitty object
        let kitty <- KittyVerse.createKitty()

        // Create the KittyHat objects
        let hat1 <- KittyVerse.createHat(id: 1, name: "Cowboy Hat")
        let hat2 <- KittyVerse.createHat(id: 2, name: "Top Hat")

        let kittyItems <- kitty.getKittyItems()

        // Put the hat on the cat!
        let oldCowboyHat <- kittyItems["Cowboy Hat"] <- hat1
        destroy oldCowboyHat
        let oldTopHat <- kittyItems["Top Hat"] <- hat2
        destroy oldTopHat

        kitty.setKittyItems(items: <-kittyItems)

        log("The cat has the hats")

        // Store the Kitty in storage
        acct.save(<-kitty, to: /storage/kitty)
    }
}
```

You should see an output that looks something like this:

```
> "The Cat has the Hats"
```

Now we can run a transaction to move the Kitty along with its hat, remove the cowboy hat from the Kitty, then make the Kitty tip its hat.


<Callout type="info">

1. Open `Transaction2.cdc`.<br/>
2. Select account `0x01` as the only signer.<br/>
3. Send the transaction.

</Callout>

In this transaction, we executed the following code:

```cadence Transaction2.cdc
import KittyVerse from 0x01

// This transaction moves a kitty out of storage, takes the cowboy hat off of the kitty,
// calls its tip hat function, and then moves it back into storage.
transaction {
    prepare(acct: AuthAccount) {

        // Move the Kitty out of storage, which also moves its hat along with it
        let kitty <- acct.load<@KittyVerse.Kitty>(from: /storage/kitty)
            ?? panic("Kitty doesn't exist!")

        // Take the cowboy hat off the Kitty
        let cowboyHat <- kitty.removeKittyItem(key: "Cowboy Hat")
            ?? panic("cowboy hat doesn't exist!")

        // Tip the cowboy hat
        log(cowboyHat.tipHat())
        destroy cowboyHat

        // Tip the top hat that is on the Kitty
        log(kitty.items["Top Hat"]?.tipHat())

        // Move the Kitty to storage, which
        // also moves its hat along with it.
        acct.save(<-kitty, to: /storage/kitty)
    }
}
```

You should see something like this output:

```
> "Howdy Y'all"
> "Greetings, fellow aristocats!"
```

Whenever the Kitty is moved, its hats are implicitly moved along with it. This is because the hats are owned by the Kitty.

## The Future is Meow! Extensibility is coming!

---

The above is a simple example of composable resources. We had to explicitly say that a Kitty could own a Hat in this example, but in the near future, Cadence will support more powerful ways of achieving resource extensibility where developers can declare types that separate resources can own even if the owning resource never specified the ownership possibility in the first place. This is a very complex problem to solve in a safe way, and the Flow community is working very hard to design a solution for this, but it is coming.

Practice what you're learned in the Flow Playground!
