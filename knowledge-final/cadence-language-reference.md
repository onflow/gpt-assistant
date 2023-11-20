---
title: Access control
sidebar_position: 13
---

Access control allows making certain parts of the program accessible/visible
and making other parts inaccessible/invisible.

In Flow and Cadence, there are two types of access control:

1. Access control on objects in account storage using capability security.

    Within Flow, a caller is not able to access an object
    unless it owns the object or has a specific reference to that object.
    This means that nothing is truly public by default.
    Other accounts can not read or write the objects in an account
    unless the owner of the account has granted them access
    by providing references to the objects.

2. Access control within contracts and objects
   using `pub` and `access` keywords.

   For the explanations of the following keywords, we assume that
   the defining type is either a contract, where capability security
   doesn't apply, or that the caller would have valid access to the object
   governed by capability security.

The high-level reference-based security (point 1 above)
will be covered in a later section.

Top-level declarations
(variables, constants, functions, structures, resources, interfaces)
and fields (in structures, and resources) are always only able to be written
to and mutated (modified, such as by indexed assignment or methods like `append`)
in the scope where it is defined (self).

There are four levels of access control defined in the code that specify where
a declaration can be accessed or called.

- **Public** or **access(all)** means the declaration
  is accessible/visible in all scopes.

  This includes the current scope, inner scopes, and the outer scopes.

  For example, a public field in a type can be accessed using the access syntax
  on an instance of the type in an outer scope.
  This does not allow the declaration to be publicly writable though.

  An element is made publicly accessible / by any code
  by using the `pub` or `access(all)` keywords.

- **access(account)** means the declaration is only accessible/visible in the
  scope of the entire account where it is defined. This means that
  other contracts in the account are able to access it,

  An element is made accessible by code in the same account (e.g. other contracts)
  by using the `access(account)` keyword.

- **access(contract)** means the declaration is only accessible/visible in the
  scope of the contract that defined it. This means that other types
  and functions that are defined in the same contract can access it,
  but not other contracts in the same account.

  An element is made accessible by code in the same contract
  by using the `access(contract)` keyword.

- Private or **access(self)** means the declaration is only accessible/visible
  in the current and inner scopes.

  For example, an `access(self)` field can only be
  accessed by functions of the type is part of,
  not by code in an outer scope.

  An element is made accessible by code in the same containing type
  by using the `access(self)` keyword.

**Access level must be specified for each declaration**

The `(set)` suffix can be used to make variables also publicly writable and mutable.

To summarize the behavior for variable declarations, constant declarations, and fields:

| Declaration kind | Access modifier          | Read scope                                           | Write scope       | Mutate scope      |
|:-----------------|:-------------------------|:-----------------------------------------------------|:------------------|:------------------|
| `let`            | `priv` / `access(self)`  | Current and inner                                    | *None*            | Current and inner |
| `let`            | `access(contract)`       | Current, inner, and containing contract              | *None*            | Current and inner |
| `let`            | `access(account)`        | Current, inner, and other contracts in same account  | *None*            | Current and inner |
| `let`            | `pub`,`access(all)`      | **All**                                              | *None*            | Current and inner |
| `var`            | `access(self)`           | Current and inner                                    | Current and inner | Current and inner |
| `var`            | `access(contract)`       | Current, inner, and containing contract              | Current and inner | Current and inner |
| `var`            | `access(account)`        | Current, inner, and other contracts in same account  | Current and inner | Current and inner |
| `var`            | `pub` / `access(all)`    | **All**                                              | Current and inner | Current and inner |
| `var`            | `pub(set)`               | **All**                                              | **All**           | **All**           |

To summarize the behavior for functions:

| Access modifier          | Access scope                                        |
|:-------------------------|:----------------------------------------------------|
| `priv` / `access(self)`  | Current and inner                                   |
| `access(contract)`       | Current, inner, and containing contract             |
| `access(account)`        | Current, inner, and other contracts in same account |
| `pub` / `access(all)`    | **All**                                             |

Declarations of structures, resources, events, and [contracts](./contracts.mdx) can only be public.
However, even though the declarations/types are publicly visible,
resources can only be created from inside the contract they are declared in.

```cadence
// Declare a private constant, inaccessible/invisible in outer scope.
//
access(self) let a = 1

// Declare a public constant, accessible/visible in all scopes.
//
pub let b = 2
```

```cadence
// Declare a public struct, accessible/visible in all scopes.
//
pub struct SomeStruct {

    // Declare a private constant field which is only readable
    // in the current and inner scopes.
    //
    access(self) let a: Int

    // Declare a public constant field which is readable in all scopes.
    //
    pub let b: Int

    // Declare a private variable field which is only readable
    // and writable in the current and inner scopes.
    //
    access(self) var c: Int

    // Declare a public variable field which is not settable,
    // so it is only writable in the current and inner scopes,
    // and readable in all scopes.
    //
    pub var d: Int

    // Declare a public variable field which is settable,
    // so it is readable and writable in all scopes.
    //
    pub(set) var e: Int

    // Arrays and dictionaries declared without (set) cannot be
    // mutated in external scopes
    pub let arr: [Int]

    // The initializer is omitted for brevity.

    // Declare a private function which is only callable
    // in the current and inner scopes.
    //
    access(self) fun privateTest() {
        // ...
    }

    // Declare a public function which is callable in all scopes.
    //
    pub fun privateTest() {
        // ...
    }

    // The initializer is omitted for brevity.

}

let some = SomeStruct()

// Invalid: cannot read private constant field in outer scope.
//
some.a

// Invalid: cannot set private constant field in outer scope.
//
some.a = 1

// Valid: can read public constant field in outer scope.
//
some.b

// Invalid: cannot set public constant field in outer scope.
//
some.b = 2

// Invalid: cannot read private variable field in outer scope.
//
some.c

// Invalid: cannot set private variable field in outer scope.
//
some.c = 3

// Valid: can read public variable field in outer scope.
//
some.d

// Invalid: cannot set public variable field in outer scope.
//
some.d = 4

// Valid: can read publicly settable variable field in outer scope.
//
some.e

// Valid: can set publicly settable variable field in outer scope.
//
some.e = 5

// Invalid: cannot mutate a public field in outer scope.
//
some.f.append(0)

// Invalid: cannot mutate a public field in outer scope.
//
some.f[3] = 1

// Valid: can call non-mutating methods on a public field in outer scope
some.f.contains(0)
```
---
title: Accounts
sidebar_position: 19
---

Every account can be accessed through two types, `PublicAccount` and `AuthAccount`.

## `PublicAccount`

**Public Account** objects have the type `PublicAccount`,
which represents the publicly available portion of an account.

```cadence

pub struct PublicAccount {

    /// The address of the account.
    pub let address: Address

    /// The FLOW balance of the default vault of this account.
    pub let balance: UFix64

    /// The FLOW balance of the default vault of this account that is available to be moved.
    pub let availableBalance: UFix64

    /// The current amount of storage used by the account in bytes.
    pub let storageUsed: UInt64

    /// The storage capacity of the account in bytes.
    pub let storageCapacity: UInt64

    /// The contracts deployed to the account.
    pub let contracts: PublicAccount.Contracts

    /// The keys assigned to the account.
    pub let keys: PublicAccount.Keys

    /// All public paths of this account.
    pub let publicPaths: [PublicPath]

    /// Returns the capability at the given public path.
    pub fun getCapability<T: &Any>(_ path: PublicPath): Capability<T>

    /// Returns the target path of the capability at the given public or private path,
    /// or nil if there exists no capability at the given path.
    pub fun getLinkTarget(_ path: CapabilityPath): Path?

    /// Iterate over all the public paths of an account.
    /// passing each path and type in turn to the provided callback function.
    ///
    /// The callback function takes two arguments:
    ///   1. The path of the stored object
    ///   2. The runtime type of that object
    ///
    /// Iteration is stopped early if the callback function returns `false`.
    ///
    /// The order of iteration, as well as the behavior of adding or removing objects from storage during iteration,
    /// is undefined.
    pub fun forEachPublic(_ function: ((PublicPath, Type): Bool))

    pub struct Contracts {

        /// The names of all contracts deployed in the account.
        pub let names: [String]

        /// Returns the deployed contract for the contract/contract interface with the given name in the account, if any.
        ///
        /// Returns nil if no contract/contract interface with the given name exists in the account.
        pub fun get(name: String): DeployedContract?

        /// Returns a reference of the given type to the contract with the given name in the account, if any.
        ///
        /// Returns nil if no contract with the given name exists in the account,
        /// or if the contract does not conform to the given type.
        pub fun borrow<T: &Any>(name: String): T?
    }

    pub struct Keys {

        /// Returns the key at the given index, if it exists, or nil otherwise.
        ///
        /// Revoked keys are always returned, but they have `isRevoked` field set to true.
        pub fun get(keyIndex: Int): AccountKey?

        /// Iterate over all unrevoked keys in this account,
        /// passing each key in turn to the provided function.
        ///
        /// Iteration is stopped early if the function returns `false`.
        /// The order of iteration is undefined.
        pub fun forEach(_ function: ((AccountKey): Bool))

        /// The total number of unrevoked keys in this account.
        pub let count: UInt64
    }
}
```

  Any code can get the `PublicAccount` for an account address
  using the built-in `getAccount` function:

  ```cadence
  fun getAccount(_ address: Address): PublicAccount
  ```

## `AuthAccount`

**Authorized Account** object have the type `AuthAccount`,
which represents the authorized portion of an account.

Access to an `AuthAccount` means having full access to its [storage](#account-storage),
public keys, and code.

Only [signed transactions](./transactions.md) can get the `AuthAccount` for an account.
For each signer of the transaction that signs as an authorizer, the corresponding `AuthAccount` object is passed
to the `prepare` phase of the transaction.

```cadence

pub struct AuthAccount {

    /// The address of the account.
    pub let address: Address

    /// The FLOW balance of the default vault of this account.
    pub let balance: UFix64

    /// The FLOW balance of the default vault of this account that is available to be moved.
    pub let availableBalance: UFix64

    /// The current amount of storage used by the account in bytes.
    pub let storageUsed: UInt64

    /// The storage capacity of the account in bytes.
    pub let storageCapacity: UInt64

    /// The contracts deployed to the account.
    pub let contracts: AuthAccount.Contracts

    /// The keys assigned to the account.
    pub let keys: AuthAccount.Keys

    /// The inbox allows bootstrapping (sending and receiving) capabilities.
    pub let inbox: AuthAccount.Inbox

    /// All public paths of this account.
    pub let publicPaths: [PublicPath]

    /// All private paths of this account.
    pub let privatePaths: [PrivatePath]

    /// All storage paths of this account.
    pub let storagePaths: [StoragePath]

    /// **DEPRECATED**: Use `keys.add` instead.
    ///
    /// Adds a public key to the account.
    ///
    /// The public key must be encoded together with their signature algorithm, hashing algorithm and weight.
    pub fun addPublicKey(_ publicKey: [UInt8])

    /// **DEPRECATED**: Use `keys.revoke` instead.
    ///
    /// Revokes the key at the given index.
    pub fun removePublicKey(_ index: Int)

    /// Saves the given object into the account's storage at the given path.
    ///
    /// Resources are moved into storage, and structures are copied.
    ///
    /// If there is already an object stored under the given path, the program aborts.
    ///
    /// The path must be a storage path, i.e., only the domain `storage` is allowed.
    pub fun save<T: Storable>(_ value: T, to: StoragePath)

    /// Reads the type of an object from the account's storage which is stored under the given path,
    /// or nil if no object is stored under the given path.
    ///
    /// If there is an object stored, the type of the object is returned without modifying the stored object.
    ///
    /// The path must be a storage path, i.e., only the domain `storage` is allowed.
    pub fun type(at: StoragePath): Type?

    /// Loads an object from the account's storage which is stored under the given path,
    /// or nil if no object is stored under the given path.
    ///
    /// If there is an object stored,
    /// the stored resource or structure is moved out of storage and returned as an optional.
    ///
    /// When the function returns, the storage no longer contains an object under the given path.
    ///
    /// The given type must be a supertype of the type of the loaded object.
    /// If it is not, the function panics.
    ///
    /// The given type must not necessarily be exactly the same as the type of the loaded object.
    ///
    /// The path must be a storage path, i.e., only the domain `storage` is allowed.
    pub fun load<T: Storable>(from: StoragePath): T?

    /// Returns a copy of a structure stored in account storage under the given path,
    /// without removing it from storage,
    /// or nil if no object is stored under the given path.
    ///
    /// If there is a structure stored, it is copied.
    /// The structure stays stored in storage after the function returns.
    ///
    /// The given type must be a supertype of the type of the copied structure.
    /// If it is not, the function panics.
    ///
    /// The given type must not necessarily be exactly the same as the type of the copied structure.
    ///
    /// The path must be a storage path, i.e., only the domain `storage` is allowed.
    pub fun copy<T: AnyStruct>(from: StoragePath): T?

    /// Returns a reference to an object in storage without removing it from storage.
    ///
    /// If no object is stored under the given path, the function returns nil.
    /// If there is an object stored, a reference is returned as an optional,
    /// provided it can be borrowed using the given type.
    /// If the stored object cannot be borrowed using the given type, the function panics.
    ///
    /// The given type must not necessarily be exactly the same as the type of the borrowed object.
    ///
    /// The path must be a storage path, i.e., only the domain `storage` is allowed
    pub fun borrow<T: &Any>(from: StoragePath): T?

    /// Returns true if the object in account storage under the given path satisfies the given type,
    /// i.e. could be borrowed using the given type.
    ///
    /// The given type must not necessarily be exactly the same as the type of the borrowed object.
    ///
    /// The path must be a storage path, i.e., only the domain `storage` is allowed.
    pub fun check<T: Any>(from: StoragePath): Bool

    /// Creates a capability at the given public or private path,
    /// which targets the given public, private, or storage path.
    ///
    /// The target path leads to the object that will provide the functionality defined by this capability.
    ///
    /// The given type defines how the capability can be borrowed, i.e., how the stored value can be accessed.
    ///
    /// Returns nil if a link for the given capability path already exists, or the newly created capability if not.
    ///
    /// It is not necessary for the target path to lead to a valid object; the target path could be empty,
    /// or could lead to an object which does not provide the necessary type interface:
    /// The link function does **not** check if the target path is valid/exists at the time the capability is created
    /// and does **not** check if the target value conforms to the given type.
    ///
    /// The link is latent.
    ///
    /// The target value might be stored after the link is created,
    /// and the target value might be moved out after the link has been created.
    pub fun link<T: &Any>(_ newCapabilityPath: CapabilityPath, target: Path): Capability<T>?

    /// Creates a capability at the given public or private path which targets this account.
    ///
    /// Returns nil if a link for the given capability path already exists, or the newly created capability if not.
    pub fun linkAccount(_ newCapabilityPath: PrivatePath): Capability<&AuthAccount>?

    /// Returns the capability at the given private or public path.
    pub fun getCapability<T: &Any>(_ path: CapabilityPath): Capability<T>

    /// Returns the target path of the capability at the given public or private path,
    /// or nil if there exists no capability at the given path.
    pub fun getLinkTarget(_ path: CapabilityPath): Path?

    /// Removes the capability at the given public or private path.
    pub fun unlink(_ path: CapabilityPath)

    /// Iterate over all the public paths of an account,
    /// passing each path and type in turn to the provided callback function.
    ///
    /// The callback function takes two arguments:
    ///   1. The path of the stored object
    ///   2. The runtime type of that object
    ///
    /// Iteration is stopped early if the callback function returns `false`.
    ///
    /// The order of iteration is undefined.
    ///
    /// If an object is stored under a new public path,
    /// or an existing object is removed from a public path,
    /// then the callback must stop iteration by returning false.
    /// Otherwise, iteration aborts.
    ///
    pub fun forEachPublic(_ function: ((PublicPath, Type): Bool))

    /// Iterate over all the private paths of an account,
    /// passing each path and type in turn to the provided callback function.
    ///
    /// The callback function takes two arguments:
    ///   1. The path of the stored object
    ///   2. The runtime type of that object
    ///
    /// Iteration is stopped early if the callback function returns `false`.
    ///
    /// The order of iteration is undefined.
    ///
    /// If an object is stored under a new private path,
    /// or an existing object is removed from a private path,
    /// then the callback must stop iteration by returning false.
    /// Otherwise, iteration aborts.
    pub fun forEachPrivate(_ function: ((PrivatePath, Type): Bool))

    /// Iterate over all the stored paths of an account,
    /// passing each path and type in turn to the provided callback function.
    ///
    /// The callback function takes two arguments:
    ///   1. The path of the stored object
    ///   2. The runtime type of that object
    ///
    /// Iteration is stopped early if the callback function returns `false`.
    ///
    /// If an object is stored under a new storage path,
    /// or an existing object is removed from a storage path,
    /// then the callback must stop iteration by returning false.
    /// Otherwise, iteration aborts.
    pub fun forEachStored(_ function: ((StoragePath, Type): Bool))

    pub struct Contracts {

        /// The names of all contracts deployed in the account.
        pub let names: [String]

        /// Adds the given contract to the account.
        ///
        /// The `code` parameter is the UTF-8 encoded representation of the source code.
        /// The code must contain exactly one contract or contract interface,
        /// which must have the same name as the `name` parameter.
        ///
        /// All additional arguments that are given are passed further to the initializer
        /// of the contract that is being deployed.
        ///
        /// The function fails if a contract/contract interface with the given name already exists in the account,
        /// if the given code does not declare exactly one contract or contract interface,
        /// or if the given name does not match the name of the contract/contract interface declaration in the code.
        ///
        /// Returns the deployed contract.
        pub fun add(
            name: String,
            code: [UInt8]
        ): DeployedContract

        /// **Experimental**
        ///
        /// Updates the code for the contract/contract interface in the account.
        ///
        /// The `code` parameter is the UTF-8 encoded representation of the source code.
        /// The code must contain exactly one contract or contract interface,
        /// which must have the same name as the `name` parameter.
        ///
        /// Does **not** run the initializer of the contract/contract interface again.
        /// The contract instance in the world state stays as is.
        ///
        /// Fails if no contract/contract interface with the given name exists in the account,
        /// if the given code does not declare exactly one contract or contract interface,
        /// or if the given name does not match the name of the contract/contract interface declaration in the code.
        ///
        /// Returns the deployed contract for the updated contract.
        pub fun update__experimental(name: String, code: [UInt8]): DeployedContract

        /// Returns the deployed contract for the contract/contract interface with the given name in the account, if any.
        ///
        /// Returns nil if no contract/contract interface with the given name exists in the account.
        pub fun get(name: String): DeployedContract?

        /// Removes the contract/contract interface from the account which has the given name, if any.
        ///
        /// Returns the removed deployed contract, if any.
        ///
        /// Returns nil if no contract/contract interface with the given name exists in the account.
        pub fun remove(name: String): DeployedContract?

        /// Returns a reference of the given type to the contract with the given name in the account, if any.
        ///
        /// Returns nil if no contract with the given name exists in the account,
        /// or if the contract does not conform to the given type.
        pub fun borrow<T: &Any>(name: String): T?
    }

    pub struct Keys {

        /// Adds a new key with the given hashing algorithm and a weight.
        ///
        /// Returns the added key.
        pub fun add(
            publicKey: PublicKey,
            hashAlgorithm: HashAlgorithm,
            weight: UFix64
        ): AccountKey

        /// Returns the key at the given index, if it exists, or nil otherwise.
        ///
        /// Revoked keys are always returned, but they have `isRevoked` field set to true.
        pub fun get(keyIndex: Int): AccountKey?

        /// Marks the key at the given index revoked, but does not delete it.
        ///
        /// Returns the revoked key if it exists, or nil otherwise.
        pub fun revoke(keyIndex: Int): AccountKey?

        /// Iterate over all unrevoked keys in this account,
        /// passing each key in turn to the provided function.
        ///
        /// Iteration is stopped early if the function returns `false`.
        ///
        /// The order of iteration is undefined.
        pub fun forEach(_ function: ((AccountKey): Bool))

        /// The total number of unrevoked keys in this account.
        pub let count: UInt64
    }

    pub struct Inbox {

        /// Publishes a new Capability under the given name,
        /// to be claimed by the specified recipient.
        pub fun publish(_ value: Capability, name: String, recipient: Address)

        /// Unpublishes a Capability previously published by this account.
        ///
        /// Returns `nil` if no Capability is published under the given name.
        ///
        /// Errors if the Capability under that name does not match the provided type.
        pub fun unpublish<T: &Any>(_ name: String): Capability<T>?

        /// Claims a Capability previously published by the specified provider.
        ///
        /// Returns `nil` if no Capability is published under the given name,
        /// or if this account is not its intended recipient.
        ///
        /// Errors if the Capability under that name does not match the provided type.
        pub fun claim<T: &Any>(_ name: String, provider: Address): Capability<T>?
    }
}
```

A script can get the `AuthAccount` for an account address using the built-in `getAuthAccount` function:

```cadence
fun getAuthAccount(_ address: Address): AuthAccount
```

This `AuthAccount` object can perform all operations associated with authorized accounts,
and as such this function is only available in scripts,
which discard their changes upon completion.
Attempting to use this function outside of a script will cause a type error.

## Account Creation

Accounts can be created by calling the `AuthAccount` constructor
and passing the account that should pay for the account creation for the `payer` parameter.

The `payer` must have enough funds to be able to create an account.
If the account does not have the required funds, the program aborts.

```cadence
transaction() {
    prepare(signer: AuthAccount) {
        let account = AuthAccount(payer: signer)
    }
}
```

## Account Keys

An account (both `PublicAccount` and `AuthAccount`) has keys associated with it.
An account key has the following structure.

```cadence
struct AccountKey {
    let keyIndex: Int
    let publicKey: PublicKey
    let hashAlgorithm: HashAlgorithm
    let weight: UFix64
    let isRevoked: Bool
}
```

Refer to the [`PublicKey` section](./crypto.mdx#publickey) for more details on the creation and validity of public keys.

### Account Key API
Account key API provides a set of functions to manage account keys.

#### Add Account Keys
To authorize access to the account, keys can be added using the `add()` function.
Keys can only be added to an `AuthAccount`.

For example, to create an account and have the signer of the transaction pay for the account
creation, and authorize one key to access the account:

```cadence
transaction(publicKey: [UInt8]) {
    prepare(signer: AuthAccount) {
        let key = PublicKey(
            publicKey: publicKey,
            signatureAlgorithm: SignatureAlgorithm.ECDSA_P256
        )

        let account = AuthAccount(payer: signer)

        account.keys.add(
            publicKey: key,
            hashAlgorithm: HashAlgorithm.SHA3_256,
            weight: 10.0
        )
    }
}
```

To add a public key to an existing account, which signed the transaction:
```cadence
transaction(publicKey: [UInt8]) {
    prepare(signer: AuthAccount) {
        let key = PublicKey(
            publicKey: publicKey,
            signatureAlgorithm: SignatureAlgorithm.ECDSA_P256
        )

        signer.keys.add(
            publicKey: key,
            hashAlgorithm: HashAlgorithm.SHA3_256,
            weight: 10.0
        )
    }
}
```

<Callout type="info">
⚠️  Note: Keys can also be added using the `addPublicKey` function.
However, this method is currently deprecated and is available only for the backward compatibility.
The `addPublicKey` method accepts the public key encoded together with their signature algorithm,
hashing algorithm and weight.

```cadence
transaction(key: [UInt8]) {
    prepare(signer: AuthAccount) {
        let account = AuthAccount(payer: signer)
        account.addPublicKey(key)
    }
}
```
</Callout>


#### Get Account Keys

Keys that are added to an account can be retrieved using `get()` function, using the index of the key.
Revoked keys are always returned, but they have `isRevoked` field set to true.
Returns `nil` if there is no key available at the given index.
Keys can be retrieved from both `PublicAccout` and `AuthAccount`.

```cadence
transaction() {
    prepare(signer: AuthAccount) {
        // Get a key from an auth account.
        let keyA = signer.keys.get(keyIndex: 2)

        // Get a key from the public aacount.
        let publicAccount = getAccount(0x42)
        let keyB = publicAccount.keys.get(keyIndex: 2)
    }
}
```

#### Revoke Account Keys

Keys that have been added to an account can be revoked using `revoke()` function.
Revoke function only marks the key at the given index as revoked, but never deletes it.
Keys can only be revoked from an `AuthAccount`.

```cadence
transaction() {
    prepare(signer: AuthAccount) {
        // Get a key from an auth account.
        let keyA = signer.keys.revoke(keyIndex: 2)
    }
}
```

<Callout type="info">
⚠️  Note: Keys can also be removed using the `removePublicKey` function.
However, this method is deprecated and is available only for the backward compatibility.
</Callout>

## Account Inbox

Accounts also possess an `Inbox` that can be used to make [Capabilities](./capabilities.md) available to specific accounts.
The functions in this `Inbox` provide a convenient means to "bootstrap" Capabilities,
setting up an initial connection between two accounts that will later allow them to transfer data or permissions through a Capability.

### Publishing a Capability

An account (the provider) that would like to provide a Capability to another account (the recipient) can do so using the `publish` function:

```cadence
fun publish(_ value: Capability, name: String, recipient: Address)
```

This publishes the specified Capability using the provided string as an identifier, to be later claimed by the recipient.
Note, however, that until the recipient does claim this Capability, it is stored on the provider's account,
and contributes towards their Account Storage total.

Calling this function emits an event, `InboxValuePublished`,
that includes the address of both the provider and the recipient, as well as the name and the type of the published Capability.
Refer to the [`Core Events` section](./core-events.md#inbox-value-published) for more details on this event.

### Claiming a Capability

The intended recipient of a Capability can claim that Capability from the provider using the `claim` function:

```cadence
fun claim<T: &Any>(_ name: String, provider: Address): Capability<T>?
```

This looks up the specified name in the provider's inbox, returning it to the recipient if it is present,
conforms to the provided type argument, and is intended for the calling recipient.
If the provider has no Capability stored under the provided name,
or if the calling recipient is not the intended recipient of the Capability, the function returns `nil`.
If the borrow type of the Capability is not a subtype of the provided type argument, the function will error at runtime.

Upon successful completion of the `claim` function, the claimed Capability is removed from the provider's inbox.
Note that this means a given Capability can only be claimed once.

Calling this function emits an event, `InboxValueClaimed`,
that includes the address of both the provider and the recipient, as well as the name of the claimed Capability.
Refer to the [`Core Events` section](./core-events.md#inbox-value-claimed) for more details on this event.

### Unpublishing a Capability

If the provider of a Capability no longer wishes for it to be published for some reason (e.g. they no longer wish to pay for its storage costs),
they can unpublish it using the `unpublish` function:

```cadence
fun unpublish<T: &Any>(_ name: String): Capability<T>?
```

This looks up the specified name in the provider's inbox, returning it to the provider if it is present and conforms to the provided type argument.
If the provider has no Capability stored under the provided name, the function returns `nil`.
If the borrow type of the Capability is not a subtype of the provided type argument, the function will error at runtime.

Upon successful completion of the `unpublish` function, the unpublished Capability is removed from the provider's inbox.

Calling this function emits an event, `InboxValueUnpublished`,
that includes the address of the provider, and the name of the claimed Capability.
Refer to the [`Core Events` section](./core-events.md#inbox-value-unpublished) for more details on this event.

## Account Storage

All accounts have storage.
Both resources and structures can be stored in account storage.

### Paths

Objects are stored under paths.
Paths consist of a domain and an identifier.

Paths start with the character `/`, followed by the domain, the path separator `/`,
and finally the identifier.
For example, the path `/storage/test` has the domain `storage` and the identifier `test`.

There are only three valid domains: `storage`, `private`, and `public`.

Objects in storage are always stored in the `storage` domain.

Paths in the storage domain have type `StoragePath`,
in the private domain `PrivatePath`,
and in the public domain `PublicPath`.

`PrivatePath` and `PublicPath` are subtypes of `CapabilityPath`.

Both `StoragePath` and `CapabilityPath` are subtypes of `Path`.

<table>
  <tr>
    <td colspan="3">Path</td>
  </tr>
  <tr>
    <td colspan="2">CapabilityPath</td>
    <td colspan="2" rowspan="2">StoragePath</td>
  </tr>
  <tr>
    <td>PrivatePath</td>
    <td>PublicPath</td>
  </tr>
</table>

#### Path Functions

```cadence
fun toString(): String
```

Returns the string representation of the path.

```cadence
let storagePath = /storage/path

storagePath.toString()  // is "/storage/path"
```

There are also utilities to produce paths from strings:

```cadence
fun PublicPath(identifier: string): PublicPath?
fun PrivatePath(identifier: string): PrivatePath?
fun StoragePath(identifier: string): StoragePath?
```

Each of these functions take an identifier and produce a path of the appropriate domain:

```cadence
let pathID = "foo"
let path = PublicPath(identifier: pathID) // is /public/foo
```

### Account Storage API

Account storage is accessed through the following functions of `AuthAccount`.
This means that any code that has access to the authorized account has access
to all its stored objects.

```cadence
fun save<T>(_ value: T, to: StoragePath)
```

Saves an object to account storage.
Resources are moved into storage, and structures are copied.

`T` is the type parameter for the object type.
It can be inferred from the argument's type.

If there is already an object stored under the given path, the program aborts.

The path must be a storage path, i.e., only the domain `storage` is allowed.


```cadence
fun type(at: StoragePath): Type?
```

Reads the type of an object from the account's storage which is stored under the given path, or nil if no object is stored under the given path.

If there is an object stored, the type of the object is returned without modifying the stored object.

The path must be a storage path, i.e., only the domain `storage` is allowed


```cadence
fun load<T>(from: StoragePath): T?
```

Loads an object from account storage.
If no object is stored under the given path, the function returns `nil`.
If there is an object stored, the stored resource or structure is moved
out of storage and returned as an optional.
When the function returns, the storage no longer contains an object
under the given path.

`T` is the type parameter for the object type.
A type argument for the parameter must be provided explicitly.

The type `T` must be a supertype of the type of the loaded object.
If it is not, execution will abort with an error.
The given type does not necessarily need to be exactly the same as the type of the loaded object.

The path must be a storage path, i.e., only the domain `storage` is allowed.


```cadence
fun copy<T: AnyStruct>(from: StoragePath): T?
```

Returns a copy of a structure stored in account storage, without removing it from storage.

If no structure is stored under the given path, the function returns `nil`.
If there is a structure stored, it is copied.
The structure stays stored in storage after the function returns.

`T` is the type parameter for the structure type.
A type argument for the parameter must be provided explicitly.

The type `T` must be a supertype of the type of the copied structure.
If it is not, execution will abort with an error.
The given type does not necessarily need to be exactly the same as
the type of the copied structure.

The path must be a storage path, i.e., only the domain `storage` is allowed.

```cadence
// Declare a resource named `Counter`.
//
resource Counter {
    pub var count: Int

    pub init(count: Int) {
        self.count = count
    }
}

// In this example an authorized account is available through the constant `authAccount`.

// Create a new instance of the resource type `Counter`
// and save it in the storage of the account.
//
// The path `/storage/counter` is used to refer to the stored value.
// Its identifier `counter` was chosen freely and could be something else.
//
authAccount.save(<-create Counter(count: 42), to: /storage/counter)

// Run-time error: Storage already contains an object under path `/storage/counter`
//
authAccount.save(<-create Counter(count: 123), to: /storage/counter)

// Load the `Counter` resource from storage path `/storage/counter`.
//
// The new constant `counter` has the type `Counter?`, i.e., it is an optional,
// and its value is the counter resource, that was saved at the beginning
// of the example.
//
let counter <- authAccount.load<@Counter>(from: /storage/counter)

// The storage is now empty, there is no longer an object stored
// under the path `/storage/counter`.

// Load the `Counter` resource again from storage path `/storage/counter`.
//
// The new constant `counter2` has the type `Counter?` and is `nil`,
// as nothing is stored under the path `/storage/counter` anymore,
// because the previous load moved the counter out of storage.
//
let counter2 <- authAccount.load<@Counter>(from: /storage/counter)

// Create another new instance of the resource type `Counter`
// and save it in the storage of the account.
//
// The path `/storage/otherCounter` is used to refer to the stored value.
//
authAccount.save(<-create Counter(count: 123), to: /storage/otherCounter)

// Load the `Vault` resource from storage path `/storage/otherCounter`.
//
// The new constant `vault` has the type `Vault?` and its value is `nil`,
// as there is a resource with type `Counter` stored under the path,
// which is not a subtype of the requested type `Vault`.
//
let vault <- authAccount.load<@Vault>(from: /storage/otherCounter)

// The storage still stores a `Counter` resource under the path `/storage/otherCounter`.

// Save the string "Hello, World" in storage
// under the path `/storage/helloWorldMessage`.

authAccount.save("Hello, world!", to: /storage/helloWorldMessage)

// Copy the stored message from storage.
//
// After the copy, the storage still stores the string under the path.
// Unlike `load`, `copy` does not remove the object from storage.
//
let message = authAccount.copy<String>(from: /storage/helloWorldMessage)

// Create a new instance of the resource type `Vault`
// and save it in the storage of the account.
//
authAccount.save(<-createEmptyVault(), to: /storage/vault)

// Invalid: Cannot copy a resource, as this would allow arbitrary duplication.
//
let vault <- authAccount.copy<@Vault>(from: /storage/vault)
```

As it is convenient to work with objects in storage
without having to move them out of storage,
as it is necessary for resources,
it is also possible to create references to objects in storage:
This is possible using the `borrow` function of an `AuthAccount`:

```cadence
fun borrow<T: &Any>(from: StoragePath): T?
```

Returns a reference to an object in storage without removing it from storage.
If no object is stored under the given path, the function returns `nil`.
If there is an object stored, a reference is returned as an optional.

`T` is the type parameter for the object type.
A type argument for the parameter must be provided explicitly.
The type argument must be a reference to any type (`&Any`; `Any` is the supertype of all types).
It must be possible to create the given reference type `T` for the stored /  borrowed object.
If it is not, execution will abort with an error.
The given type does not necessarily need to be exactly the same as the type of the borrowed object.

The path must be a storage path, i.e., only the domain `storage` is allowed.

```cadence
// Declare a resource interface named `HasCount`, that has a field `count`
//
resource interface HasCount {
    count: Int
}

// Declare a resource named `Counter` that conforms to `HasCount`
//
resource Counter: HasCount {
    pub var count: Int

    pub init(count: Int) {
        self.count = count
    }
}

// In this example an authorized account is available through the constant `authAccount`.

// Create a new instance of the resource type `Counter`
// and save it in the storage of the account.
//
// The path `/storage/counter` is used to refer to the stored value.
// Its identifier `counter` was chosen freely and could be something else.
//
authAccount.save(<-create Counter(count: 42), to: /storage/counter)

// Create a reference to the object stored under path `/storage/counter`,
// typed as `&Counter`.
//
// `counterRef` has type `&Counter?` and is a valid reference, i.e. non-`nil`,
// because the borrow succeeded:
//
// There is an object stored under path `/storage/counter`
// and it has type `Counter`, so it can be borrowed as `&Counter`
//
let counterRef = authAccount.borrow<&Counter>(from: /storage/counter)

counterRef?.count // is `42`

// Create a reference to the object stored under path `/storage/counter`,
// typed as `&{HasCount}`.
//
// `hasCountRef` is non-`nil`, as there is an object stored under path `/storage/counter`,
// and the stored value of type `Counter` conforms to the requested type `{HasCount}`:
// the type `Counter` implements the restricted type's restriction `HasCount`

let hasCountRef = authAccount.borrow<&{HasCount}>(from: /storage/counter)

// Create a reference to the object stored under path `/storage/counter`,
// typed as `&{SomethingElse}`.
//
// `otherRef` is `nil`, as there is an object stored under path `/storage/counter`,
// but the stored value of type `Counter` does not conform to the requested type `{Other}`:
// the type `Counter` does not implement the restricted type's restriction `Other`

let otherRef = authAccount.borrow<&{Other}>(from: /storage/counter)

// Create a reference to the object stored under path `/storage/nonExistent`,
// typed as `&{HasCount}`.
//
// `nonExistentRef` is `nil`, as there is nothing stored under path `/storage/nonExistent`
//
let nonExistentRef = authAccount.borrow<&{HasCount}>(from: /storage/nonExistent)
```

## Storage Iteration

It is possible to iterate over an account's storage using the following iteration functions:

```cadence
fun forEachPublic(_ function: ((PublicPath, Type): Bool))
fun forEachPrivate(_ function: ((PrivatePath, Type): Bool))
fun forEachStored(_ function: ((StoragePath, Type): Bool))
```

Each of these iterates over every element in the specified domain (public, private, and storage),
applying the function argument to each.
The first argument of the function is the path of the element, and the second is its runtime type.
In the case of the `private` and `public` path iteration functions,
this is the runtime type of the capability linked at that path.
The `Bool` return value determines whether iteration continues;
`true` will proceed to the next stored element,
while `false` will terminate iteration.
The specific order in which the objects are iterated over is undefined,
as is the behavior when a path is added or removed from storage.

<Callout type="warning">
The order of iteration is undefined. Do not rely on any particular behaviour.

Saving to or removing from storage during iteration can cause the order in which values are stored to change arbitrarily.

Continuing to iterate after such an operation will cause Cadence to panic and abort execution.
In order to avoid such errors, we recommend not modifying storage during iteration.
If you do, return `false` from the iteration callback to cause iteration to end after the mutation like so:

```cadence
account.save(1, to: /storage/foo1)
account.save(2, to: /storage/foo2)
account.save(3, to: /storage/foo3)
account.save("qux", to: /storage/foo4)

account.forEachStored(fun (path: StoragePath, type: Type): Bool {
    if type == Type<String>() {
        account.save("bar", to: /storage/foo5)
        // returning false here ends iteration after storage is modified, preventing a panic
        return false
    }
    return true
})
```
</Callout>

<Callout type="info">
    The iteration will skip any broken elements in the storage.
    An element could be broken due to invalid types associated with the stored value.
    e.g: A value belongs to type `T` of a contract with syntax/semantic errors.
</Callout>

## Storage limit

An account's storage is limited by its storage capacity.

An account's storage used is the sum of the size of all the data that is stored in an account (in MB).
An account's storage capacity is a value that is calculated from the amount of FLOW
that is stored in the account's main FLOW token vault.

At the end of every transaction, the storage used is compared to the storage capacity.
For all accounts involved in the transaction, if the account's storage used is greater than its storage capacity, the transaction will fail.

An account's storage used and storage capacity can be checked using the `storageUsed` and `storageCapacity` fields.
The fields represent current values of storage which means this would be true:

```cadence
let storageUsedBefore = authAccount.storageUsed
authAccount.save(<-create Counter(count: 123), to: /storage/counter)
let storageUsedAfter = authAccount.storageUsed

let storageUsedChanged = storageUsedBefore != storageUsedAfter // is true
```
---
title: Attachments
sidebar_position: 21
---

<Callout type="warning">
⚠️ This section describes a feature that is not yet released on Mainnet.
</Callout>

Attachments are a feature of Cadence designed to allow developers to extend a struct or resource type
(even one that they did not declare) with new functionality,
without requiring the original author of the type to plan or account for the intended behavior.

## Declaring Attachments

Attachments are declared with the `attachment` keyword, which would be declared using a new form of composite declaration:
`pub attachment <Name> for <Type>: <Conformances> { ... }`, where the attachment functions and fields are declared in the body.
As such, the following would be examples of legal declarations of attachments:

```cadence
pub attachment Foo for MyStruct {
    // ...
}

attachment Bar for MyResource: MyResourceInterface {
    // ...
}

attachment Baz for MyInterface: MyOtherInterface {
    // ...
}
```

Specifying the kind (struct or resource) of an attachment is not necessary, as its kind will necessarily be the same as the type it is extending.
Note that the base type may be either a concrete composite type or an interface.
In the former case, the attachment is only usable on values specifically of that base type,
while in the case of an interface the attachment is usable on any type that conforms to that interface.

As with other type declarations, attachments may only have a `pub` access modifier (if one is present).

The body of the attachment follows the same declaration rules as composites.
In particular, they may have both field and function members,
and any field members must be initialized in an `init` function.
Only resource-kinded attachments may have resource members,
and such members must be explicitly handled in the `destroy` function.
The `self` keyword is available in attachment bodies, but unlike in a composite,
`self` is a **reference** type, rather than a composite type:
In an attachment declaration for `A`, the type of `self` would be `&A`, rather than `A` like in other composite declarations.

If a resource with attachments on it is `destroy`ed, the `destroy` functions of all its attachments are all run in an unspecified order;
`destroy` should not rely on the presence of other attachments on the base type in its implementation.
The only guarantee about the order in which attachments are destroyed in this case is that the base resource will be the last thing destroyed.

Within the body of an attachment, there is also a `base` keyword available,
which contains a reference to the attachment's base value;
that is, the composite to which the attachment is attached.
Its type, therefore, is a reference to the attachment's declared base type.
So, for an attachment declared `pub attachment Foo for Bar`, the `base` field of `Foo` would have type `&Bar`.

So, for example, this would be a valid declaration of an attachment:

```
pub resource R {
    pub let x: Int

    init (_ x: Int) {
        self.x = x
    }

    pub fun foo() { ... }
}

pub attachment A for R {
    pub let derivedX: Int

    init (_ scalar: Int) {
        self.derivedX = base.x * scalar
    }

    pub fun foo() {
        base.foo()
    }
}

```

For the purposes of external mutation checks or [access control](./access-control.md),
the attachment is considered a separate declaration from its base type.
A developer cannot, therefore, access any `priv` fields
(or `access(contract)` fields if the base was defined in a different contract to the attachment)
on the `base` value, nor can they mutate any array or dictionary typed fields.

```cadence
pub resource interface SomeInterface {
    pub let b: Bool
    priv let i: Int
    pub let a: [String]
}
pub attachment SomeAttachment for SomeContract.SomeStruct {
    pub let i: Int
    init(i: Int) {
        if base.b {
            self.i = base.i // cannot access `i` on the `base` value
        } else {
            self.i = i
        }
    }
    pub fun foo() {
        base.a.append("hello") // cannot mutate `a` outside of the composite where it was defined
    }
}
```

### Attachment Types

An attachment declared with `pub attachment A for C { ... }` will have a nominal type `A`.

It is important to note that attachments are not first class values in Cadence, and as such their usage is limited in certain ways.
In particular, their types cannot appear outside of a reference type.
So, for example, given an  attachment declaration `attachment A for X {}`, the types `A`, `A?`, `[A]` and `((): A)` are not valid type annotations,
while `&A`, `&A?`, `[&A]` and `((): &A)` are valid.

## Creating Attachments

An attachment is created using an `attach` expression,
where the attachment is both initialized and attached to the base value in a single operation.
Attachments are not first-class values in Cadence; they cannot exist independently of a base value,
nor can they be moved around on their own.
This means that an `attach` expression is the only place in which an attachment constructor can be called.
Tightly coupling the creation and attaching of attachment values helps to make reasoning about attachments simpler for the user.
Also for this reason, resource attachments do not need an expliict `<-` move operator when they appear in an `attach` expression.

An attach expression consists of the `attach` keyword, a constructor call for the attachment value,
the `to` keyword, and an expression that evaluates to the base value for that attachment.
Any arguments required by the attachment's `init` function are provided in its constructor call.

```cadence
pub resource R {}
pub attachment A for R {
    init(x: Int) {
        //...
    }
}

// ...
let r <- create R()
let r2 <- attach A(x: 3) to <-r
```

The expression on the right-hand side of the `to` keyword must evaluate to a composite value whose type is a subtype of the attachment's base,
and is evaluated before the call to the constructor on the left side of `to`.
This means that the `base` value is available inside of the attachment's `init` function,
but it is important to note that the attachment being created will not be accessible on the `base`
(see the accessing attachments section below) until after the constructor finishes executing.


```cadence
pub resource interface I {}
pub resource R: I {}
pub attachment A for I {}

// ...
let r <- create R() // has type @R
let r2 <- attach A() to <-r // ok, because `R` is a subtype of `I`, still has type @R
```

Because attachments are stored on their bases by type, there can only be one attachment of each type present on a value at a time.
Cadence will raise a runtime error if a user attempts to add an attachment to a value when one it already exists on that value.
The type returned by the `attach` expression is the same type as the expression on the right-hand side of the `to`;
attaching an attachment to a value does not change its type.

## Accessing Attachments

Attachments are accessed on composites via type-indexing:
composite values function like a dictionary where the keys are types and the values are attachments.
So given a composite value `v`, one can look up the attachment named `A` on `v` using indexing syntax:

```cadence
let a = v[A] // has type `&A?`
```

This syntax requires that `A` is a nominal attachment type,
and that `v` has a composite type that is a subtype of `A`'s declared base type.
As mentioned above, attachments are not first-class values in Cadence,
so this indexing returns a reference to the attachment on `v`, rather than the attachment itself.
If the attachment with the given type does not exist on `v`, this expression returns `nil`.

Because the indexed value must be a subtype of the indexing attachment's base type,
the owner of a resource can restrict which attachments can be accessed on references to their resource using restricted types,
much like they would do with any other field or function. E.g.

```cadence
struct R: I {}
struct interface I {}
attachment A for R {}
fun foo(r: &{I}) {
    r[A] // fails to type check, because `{I}` is not a subtype of `R`
}
```

Hence, if the owner of a resource wishes to allow others to use a subset of its attachments,
they can create a capability to that resource with a borrow type that only allows certain attachments to be accessed.

## Removing Attachments

Attachments can be removed from a value with a `remove` statement.
The statement consists of the `remove` keyword, the nominal type for the attachment to be removed,
the `from` keyword, and the value from which the attachment is meant to be removed.

The value on the right-hand side of `from` must be a composite value whose type is a subtype of the attachment type's declared base.

Before the statement executes, the attachment's `destroy` function (if present) will be executed.
After the statement executes, the composite value on the right-hand side of `from` will no longer contain the attachment.
If the value does not contain `t`, this statement has no effect.

Attachments may be removed from a type in any order,
so users should take care not to design any attachments that rely on specific behaviors of other attachments,
as there is no to require that an attachment depend on another or to require that a type has a given attachment when another attachment is present.

If a resource containing attachments is `destroy`ed, all its attachments will be `destroy`ed in an arbitrary order.
---
title: Built-in Functions
sidebar_position: 28
---

## panic
#

```cadence
fun panic(_ message: String): Never
```

  Terminates the program unconditionally
  and reports a message which explains why the unrecoverable error occurred.

  ```cadence
  let optionalAccount: AuthAccount? = // ...
  let account = optionalAccount ?? panic("missing account")
  ```

## assert

```cadence
fun assert(_ condition: Bool, message: String)
```

  Terminates the program if the given condition is false,
  and reports a message which explains how the condition is false.
  Use this function for internal sanity checks.

  The message argument is optional.

## `revertibleRandom`

```cadence
fun revertibleRandom(): UInt64
```

Returns a pseudo-random number.

The sequence of returned random numbers is independent for
every transaction in each block.
Under the hood, Cadence instantiates a cryptographically-secure pseudo-random number
generator (CSPRG) for each transaction independently, where the seeds of any two transactions
are different with near certainty.

The random numbers returned are unpredictable
(unpredictable for miners at block construction time,
and unpredictable for cadence logic at time of call),
verifiable, as well as unbiasable by miners and previously-running Cadence code.
See [Secure random number generator for Flow’s smart contracts](https://forum.flow.com/t/secure-random-number-generator-for-flow-s-smart-contracts/5110)
and [FLIP120](https://github.com/onflow/flips/pull/120) for more details.

Nevertheless, developers need to be mindful to use `revertibleRandom()` correctly:

<Callout type="warning">

A transaction can atomically revert all its action.
It is possible for a transaction submitted by an untrusted party
to post-select favorable results and revert the transaction for unfavorable results.

</Callout>

The function usage remains safe when called by a trusted party that does not
perform post-selection on the returned random numbers.

This limitation is inherent to any smart contract platform that allows transactions to roll back atomically
and cannot be solved through safe randomness alone.

This limitation is inherent to any smart contract platform that allows transactions to roll back atomically
and cannot be solved through safe randomness alone.
In cases where a non-trusted party can interact through their own transactions
with smart contracts generating random numbers,
it is recommended to use [commit-reveal schemes](https://github.com/onflow/flips/pull/123)
as outlined in this [tentative example](https://github.com/onflow/flips/blob/main/protocol/20230728-commit-reveal.md#tutorials-and-examples) (full tutorial coming soon).

## `unsafeRandom`

This function is superseded by `revertibleRandom()`.
`unsafeRandom` has the same interface and implementation as `revertibleRandom()` although
it is called unsafe. The name is retained for downwards compatibility
despite it technically being no longer unsafe (see `revertibleRandom()` for details).

<Callout type="info">
`unsafeRandom` is deprecated and will be removed in an upcoming release of Cadence.
Use `revertibleRandom()` instead.
</Callout>


## RLP

RLP (Recursive Length Prefix) serialization allows the encoding of arbitrarily nested arrays of binary data.

Cadence provides RLP decoding functions in the built-in `RLP` contract, which does not need to be imported.

-
    ```cadence
    fun decodeString(_ input: [UInt8]): [UInt8]
    ```

    Decodes an RLP-encoded byte array (called string in the context of RLP).
    The byte array should only contain of a single encoded value for a string; if the encoded value type does not match, or it has trailing unnecessary bytes, the program aborts.
    If any error is encountered while decoding, the program aborts.



    ```cadence
    fun decodeList(_ input: [UInt8]): [[UInt8]]`
    ```

    Decodes an RLP-encoded list into an array of RLP-encoded items.
    Note that this function does not recursively decode, so each element of the resulting array is RLP-encoded data.
    The byte array should only contain of a single encoded value for a list; if the encoded value type does not match, or it has trailing unnecessary bytes, the program aborts.
    If any error is encountered while decoding, the program aborts.
---
title: Capability-based Access Control
sidebar_position: 20
---

Users will often want to make it so that specific other users or even anyone else
can access certain fields and functions of a stored object.
This can be done by creating a capability.

As was mentioned before, access to stored objects is governed by the
tenets of [Capability Security](https://en.wikipedia.org/wiki/Capability-based_security).
This means that if an account wants to be able to access another account's
stored objects, it must have a valid capability to that object.

Capabilities are identified by a path and link to a target path, not directly to an object.
Capabilities are either public (any user can get access),
or private (access to/from the authorized user is necessary).

Public capabilities are created using public paths, i.e. they have the domain `public`.
After creation they can be obtained from both authorized accounts (`AuthAccount`)
and public accounts (`PublicAccount`).

Private capabilities are created using private paths, i.e. they have the domain `private`.
After creation they can be obtained from authorized accounts (`AuthAccount`),
but not from public accounts (`PublicAccount`).

Once a capability is created and obtained, it can be borrowed to get a reference
to the stored object.
When a capability is created, a type is specified that determines as what type
the capability can be borrowed.
This allows exposing and hiding certain functionality of a stored object.

Capabilities are created using the `link` function of an authorized account (`AuthAccount`):

-
    ```cadence
    fun link<T: &Any>(_ newCapabilityPath: CapabilityPath, target: Path): Capability<T>?
    ```

    `newCapabilityPath` is the public or private path identifying the new capability.

    `target` is any public, private, or storage path that leads to the object
    that will provide the functionality defined by this capability.

    `T` is the type parameter for the capability type.
    A type argument for the parameter must be provided explicitly.

    The type parameter defines how the capability can be borrowed,
    i.e., how the stored value can be accessed.

    The link function returns `nil` if a link for the given capability path already exists,
    or the newly created capability if not.

    It is not necessary for the target path to lead to a valid object;
    the target path could be empty, or could lead to an object
    which does not provide the necessary type interface:

    The link function does **not** check if the target path is valid/exists at the time
    the capability is created and does **not** check if the target value conforms to the given type.

    The link is latent.
    The target value might be stored after the link is created,
    and the target value might be moved out after the link has been created.

Capabilities can be removed using the `unlink` function of an authorized account (`AuthAccount`):

-
    ```cadence
    fun unlink(_ path: CapabilityPath)
    ```

    `path` is the public or private path identifying the capability that should be removed.

To get the target path for a capability, the `getLinkTarget` function
of an authorized account (`AuthAccount`) or public account (`PublicAccount`) can be used:


-
    ```cadence
    fun getLinkTarget(_ path: CapabilityPath): Path?
    ```

    `path` is the public or private path identifying the capability.
    The function returns the link target path,
    if a capability exists at the given path,
    or `nil` if it does not.

Existing capabilities can be obtained by using the `getCapability` function
of authorized accounts (`AuthAccount`) and public accounts (`PublicAccount`):

-
    ```cadence
    fun getCapability<T>(_ at: CapabilityPath): Capability<T>
    ```

    For public accounts, the function returns a capability
    if the given path is public.
    It is not possible to obtain private capabilities from public accounts.
    If the path is private or a storage path, the function returns `nil`.

    For authorized accounts, the function returns a capability
    if the given path is public or private.
    If the path is a storage path, the function returns `nil`.

    `T` is the type parameter that specifies how the capability can be borrowed.
    The type argument is optional, i.e. it need not be provided.

The `getCapability` function does **not** check if the target exists.
The link is latent.
The `check` function of the capability can be used to check if the target currently exists and could be borrowed,

-
    ```cadence
    fun check<T: &Any>(): Bool
    ```

    `T` is the type parameter for the reference type.
    A type argument for the parameter must be provided explicitly.

    The function returns true if the capability currently targets an object
    that satisfies the given type, i.e. could be borrowed using the given type.

Finally, the capability can be borrowed to get a reference to the stored object.
This can be done using the `borrow` function of the capability:

-
    ```cadence
    fun borrow<T: &Any>(): T?
    ```

    The function returns a reference to the object targeted by the capability,
    provided it can be borrowed using the given type.

    `T` is the type parameter for the reference type.
    If the function is called on a typed capability, the capability's type is used when borrowing.
    If the capability is untyped, a type argument must be provided explicitly in the call to `borrow`.

    The function returns `nil` when the targeted path is empty, i.e. nothing is stored under it.
    When the requested type exceeds what is allowed by the capability (or any interim capabilities),
    execution will abort with an error.

```cadence
// Declare a resource interface named `HasCount`, that has a field `count`
//
resource interface HasCount {
    count: Int
}

// Declare a resource named `Counter` that conforms to `HasCount`
//
resource Counter: HasCount {
    pub var count: Int

    pub init(count: Int) {
        self.count = count
    }

    pub fun increment(by amount: Int) {
        self.count = self.count + amount
    }
}

// In this example an authorized account is available through the constant `authAccount`.

// Create a new instance of the resource type `Counter`
// and save it in the storage of the account.
//
// The path `/storage/counter` is used to refer to the stored value.
// Its identifier `counter` was chosen freely and could be something else.
//
authAccount.save(<-create Counter(count: 42), to: /storage/counter)

// Create a public capability that allows access to the stored counter object
// as the type `{HasCount}`, i.e. only the functionality of reading the field
//
authAccount.link<&{HasCount}>(/public/hasCount, target: /storage/counter)
```

To get the published portion of an account, the `getAccount` function can be used.

Imagine that the next example is from a different account as before.

```cadence

// Get the public account for the address that stores the counter
//
let publicAccount = getAccount(0x1)

// Get a capability for the counter that is made publicly accessible
// through the path `/public/hasCount`.
//
// Use the type `&{HasCount}`, a reference to some object that provides the functionality
// of interface `HasCount`. This is the type that the capability can be borrowed as
// (it was specified in the call to `link` above).
// See the example below for borrowing using the type `&Counter`.
//
// After the call, the declared constant `countCap` has type `Capability<&{HasCount}>`,
// a capability that results in a reference that has type `&{HasCount}` when borrowed.
//
let countCap = publicAccount.getCapability<&{HasCount}>(/public/hasCount)

// Borrow the capability to get a reference to the stored counter.
//
// This borrow succeeds, i.e. the result is not `nil`,
// it is a valid reference, because:
//
// 1. Dereferencing the path chain results in a stored object
//    (`/public/hasCount` links to `/storage/counter`,
//    and there is an object stored under `/storage/counter`)
//
// 2. The stored value is a subtype of the requested type `{HasCount}`
//    (the stored object has type `Counter` which conforms to interface `HasCount`)
//
let countRef = countCap.borrow()!

countRef.count  // is `42`

// Invalid: The `increment` function is not accessible for the reference,
// because it has the type `&{HasCount}`, which does not expose an `increment` function,
// only a `count` field
//
countRef.increment(by: 5)

// Again, attempt to get a get a capability for the counter, but use the type `&Counter`.
//
// Getting the capability succeeds, because it is latent, but borrowing fails
// (the result s `nil`), because the capability was created/linked using the type `&{HasCount}`:
//
// The resource type `Counter` implements the resource interface `HasCount`,
// so `Counter` is a subtype of `{HasCount}`, but the capability only allows
// borrowing using unauthorized references of `{HasCount}` (`&{HasCount}`)
// instead of authorized references (`auth &{HasCount}`),
// so users of the capability are not allowed to borrow using subtypes,
// and they can't escalate the type by casting the reference either.
//
// This shows how parts of the functionality of stored objects
// can be safely exposed to other code
//
let countCapNew = publicAccount.getCapability<&Counter>(/public/hasCount)
let counterRefNew = countCapNew.borrow()

// `counterRefNew` is `nil`, the borrow failed

// Invalid: Cannot access the counter object in storage directly,
// the `borrow` function is not available for public accounts
//
let counterRef2 = publicAccount.borrow<&Counter>(from: /storage/counter)
```

The address of a capability can be obtained from the `address` field of the capability:

-
    ```cadence
    let address: Address
    ```

    The address of the capability.
---
title: Composite Types
sidebar_position: 11
---

Composite types allow composing simpler types into more complex types,
i.e., they allow the composition of multiple values into one.
Composite types have a name and consist of zero or more named fields,
and zero or more functions that operate on the data.
Each field may have a different type.

Composite types can only be declared within a [contract](./contracts.mdx) and nowhere else.

There are two kinds of composite types.
The kinds differ in their usage and the behaviour
when a value is used as the initial value for a constant or variable,
when the value is assigned to a variable,
when the value is passed as an argument to a function,
and when the value is returned from a function:

- [**Structures**](#structures) are **copied**, they are value types.

    Structures are useful when copies with independent state are desired.

- [**Resources**](#resources) are **moved**, they are linear types and **must** be used **exactly once**.

    Resources are useful when it is desired to model ownership
    (a value exists exactly in one location and it should not be lost).

    Certain constructs in a blockchain represent assets of real, tangible value,
    as much as a house or car or bank account.
    We have to worry about literal loss and theft,
    perhaps even on the scale of millions of dollars.

    Structures are not an ideal way to represent this ownership because they are copied.
    This would mean that there could be a risk of having multiple copies
    of certain assets floating around,
    which breaks the scarcity requirements needed for these assets to have real value.

    A structure is much more useful for representing information
    that can be grouped together in a logical way,
    but doesn't have value or a need to be able to be owned or transferred.

    A structure could for example be used to contain the information associated
    with a division of a company,
    but a resource would be used to represent the assets that have been allocated
    to that organization for spending.

Nesting of resources is only allowed within other resource types,
or in data structures like arrays and dictionaries,
but not in structures, as that would allow resources to be copied.

## Composite Type Declaration and Creation

Structures are declared using the `struct` keyword
and resources are declared using the `resource` keyword.
The keyword is followed by the name.

```cadence
pub struct SomeStruct {
    // ...
}

pub resource SomeResource {
    // ...
}
```

Structures and resources are types.

Structures are created (instantiated) by calling the type like a function.

```cadence
// instantiate a new struct object and assign it to a constant
let a = SomeStruct()
```

The constructor function may require parameters if the [initializer](#composite-type-fields)
of the composite type requires them.

Composite types can only be declared within [contracts](./contracts.mdx)
and not locally in functions.


Resource must be created (instantiated) by using the `create` keyword
and calling the type like a function.

Resources can only be created in functions and types
that are declared in the same contract in which the resource is declared.

```cadence
// instantiate a new resource object and assign it to a constant
let b <- create SomeResource()
```

## Composite Type Fields

Fields are declared like variables and constants.
However, the initial values for fields are set in the initializer,
**not** in the field declaration.
All fields **must** be initialized in the initializer, exactly once.

Having to provide initial values in the initializer might seem restrictive,
but this ensures that all fields are always initialized in one location, the initializer,
and the initialization order is clear.

The initialization of all fields is checked statically
and it is invalid to not initialize all fields in the initializer.
Also, it is statically checked that a field is definitely initialized before it is used.

The initializer's main purpose is to initialize fields, though it may also contain other code.
Just like a function, it may declare parameters and may contain arbitrary code.
However, it has no return type, i.e., it is always `Void`.

The initializer is declared using the `init` keyword.

The initializer always follows any fields.

There are three kinds of fields:

- **Constant fields** are also stored in the composite value,
    but after they have been initialized with a value
    they **cannot** have new values assigned to them afterwards.
    A constant field must be initialized exactly once.

    Constant fields are declared using the `let` keyword.

- **Variable fields** are stored in the composite value
    and can have new values assigned to them.

    Variable fields are declared using the `var` keyword.

| Field Kind           | Assignable         | Keyword     |
|----------------------|--------------------|-------------|
| **Variable field**   | Yes                | `var`       |
| **Constant field**   | **No**             | `let`       |

In initializers, the special constant `self` refers to the composite value
that is to be initialized.

If a composite type is to be stored, all its field types must be storable. Non-storable types are:

- Functions
- [Accounts (`AuthAccount` / `PublicAccount`)](./accounts.mdx)
- [Transactions](./transactions.md)
- [References](./references.md): References are ephemeral.
  Consider [storing a capability and borrowing it](./capabilities.md) when needed instead.

Fields can be read (if they are constant or variable) and set (if they are variable),
using the access syntax: the composite value is followed by a dot (`.`)
and the name of the field.

```cadence
// Declare a structure named `Token`, which has a constant field
// named `id` and a variable field named `balance`.
//
// Both fields are initialized through the initializer.
//
// The public access modifier `pub` is used in this example to allow
// the fields to be read in outer scopes. Fields can also be declared
// private so they cannot be accessed in outer scopes.
// Access control will be explained in a later section.
//
pub struct Token {
    pub let id: Int
    pub var balance: Int

    init(id: Int, balance: Int) {
        self.id = id
        self.balance = balance
    }
}
```

Note that it is invalid to provide the initial value for a field in the field declaration.

```cadence
pub struct StructureWithConstantField {
    // Invalid: It is invalid to provide an initial value in the field declaration.
    // The field must be initialized by setting the initial value in the initializer.
    //
    pub let id: Int = 1
}
```

The field access syntax must be used to access fields –  fields are not available as variables.

```cadence
pub struct Token {
    pub let id: Int

    init(initialID: Int) {
        // Invalid: There is no variable with the name `id` available.
        // The field `id` must be initialized by setting `self.id`.
        //
        id = initialID
    }
}
```

The initializer is **not** automatically derived from the fields, it must be explicitly declared.

```cadence
pub struct Token {
    pub let id: Int

    // Invalid: Missing initializer initializing field `id`.
}
```

A composite value can be created by calling the constructor and providing
the field values as arguments.

The value's fields can be accessed on the object after it is created.

```cadence
let token = Token(id: 42, balance: 1_000_00)

token.id  // is `42`
token.balance  // is `1_000_000`

token.balance = 1
// `token.balance` is `1`

// Invalid: assignment to constant field
//
token.id = 23
```

Initializers do not support overloading.

## Composite Type Functions

Composite types may contain functions.
Just like in the initializer, the special constant `self` refers to the composite value
that the function is called on.

```cadence
// Declare a structure named "Rectangle", which represents a rectangle
// and has variable fields for the width and height.
//
pub struct Rectangle {
    pub var width: Int
    pub var height: Int

    init(width: Int, height: Int) {
        self.width = width
        self.height = height
    }

    // Declare a function named "scale", which scales
    // the rectangle by the given factor.
    //
    pub fun scale(factor: Int) {
        self.width = self.width * factor
        self.height = self.height * factor
    }
}

let rectangle = Rectangle(width: 2, height: 3)
rectangle.scale(factor: 4)
// `rectangle.width` is `8`
// `rectangle.height` is `12`
```

 Functions do not support overloading.

## Composite Type Subtyping

Two composite types are compatible if and only if they refer to the same declaration by name,
i.e., nominal typing applies instead of structural typing.

Even if two composite types declare the same fields and functions,
the types are only compatible if their names match.

```cadence
// Declare a structure named `A` which has a function `test`
// which has type `((): Void)`.
//
struct A {
    fun test() {}
}

// Declare a structure named `B` which has a function `test`
// which has type `((): Void)`.
//
struct B {
    fun test() {}
}

// Declare a variable named which accepts values of type `A`.
//
var something: A = A()

// Invalid: Assign a value of type `B` to the variable.
// Even though types `A` and `B` have the same declarations,
// a function with the same name and type, the types' names differ,
// so they are not compatible.
//
something = B()

// Valid: Reassign a new value of type `A`.
//
something = A()
```

## Composite Type Behaviour

### Structures

Structures are **copied** when
used as an initial value for constant or variable,
when assigned to a different variable,
when passed as an argument to a function,
and when returned from a function.

Accessing a field or calling a function of a structure does not copy it.

```cadence
// Declare a structure named `SomeStruct`, with a variable integer field.
//
pub struct SomeStruct {
    pub var value: Int

    init(value: Int) {
        self.value = value
    }

    fun increment() {
        self.value = self.value + 1
    }
}

// Declare a constant with value of structure type `SomeStruct`.
//
let a = SomeStruct(value: 0)

// *Copy* the structure value into a new constant.
//
let b = a

b.value = 1
// NOTE: `b.value` is 1, `a.value` is *`0`*

b.increment()
// `b.value` is 2, `a.value` is `0`
```

### Accessing Fields and Functions of Composite Types Using Optional Chaining

If a composite type with fields and functions is wrapped in an optional,
optional chaining can be used to get those values or call the function without
having to get the value of the optional first.

Optional chaining is used by adding a `?`
before the `.` access operator for fields or
functions of an optional composite type.

When getting a field value or
calling a function with a return value, the access returns
the value as an optional.
If the object doesn't exist, the value will always be `nil`

When calling a function on an optional like this, if the object doesn't exist,
nothing will happen and the execution will continue.

It is still invalid to access an undeclared field
of an optional composite type.

```cadence
// Declare a struct with a field and method.
pub struct Value {
    pub var number: Int

    init() {
        self.number = 2
    }

    pub fun set(new: Int) {
        self.number = new
    }

    pub fun setAndReturn(new: Int): Int {
        self.number = new
        return new
    }
}

// Create a new instance of the struct as an optional
let value: Value? = Value()
// Create another optional with the same type, but nil
let noValue: Value? = nil

// Access the `number` field using optional chaining
let twoOpt = value?.number
// Because `value` is an optional, `twoOpt` has type `Int?`
let two = twoOpt ?? 0
// `two` is `2`

// Try to access the `number` field of `noValue`, which has type `Value?`.
// This still returns an `Int?`
let nilValue = noValue?.number
// This time, since `noValue` is `nil`, `nilValue` will also be `nil`

// Try to call the `set` function of `value`.
// The function call is performed, as `value` is not nil
value?.set(new: 4)

// Try to call the `set` function of `noValue`.
// The function call is *not* performed, as `noValue` is nil
noValue?.set(new: 4)

// Call the `setAndReturn` function, which returns an `Int`.
// Because `value` is an optional, the return value is type `Int?`
let sixOpt = value?.setAndReturn(new: 6)
let six = sixOpt ?? 0
// `six` is `6`
```

This is also possible by using the force-unwrap operator (`!`).

Forced-Optional chaining is used by adding a `!`
before the `.` access operator for fields or
functions of an optional composite type.

When getting a field value or calling a function with a return value,
the access returns the value.
If the object doesn't exist, the execution will panic and revert.

It is still invalid to access an undeclared field
of an optional composite type.

```cadence
// Declare a struct with a field and method.
pub struct Value {
    pub var number: Int

    init() {
        self.number = 2
    }

    pub fun set(new: Int) {
        self.number = new
    }

    pub fun setAndReturn(new: Int): Int {
        self.number = new
        return new
    }
}

// Create a new instance of the struct as an optional
let value: Value? = Value()
// Create another optional with the same type, but nil
let noValue: Value? = nil

// Access the `number` field using force-optional chaining
let two = value!.number
// `two` is `2`

// Try to access the `number` field of `noValue`, which has type `Value?`
// Run-time error: This time, since `noValue` is `nil`,
// the program execution will revert
let number = noValue!.number

// Call the `set` function of the struct

// This succeeds and sets the value to 4
value!.set(new: 4)

// Run-time error: Since `noValue` is nil, the value is not set
// and the program execution reverts.
noValue!.set(new: 4)

// Call the `setAndReturn` function, which returns an `Int`
// Because we use force-unwrap before calling the function,
// the return value is type `Int`
let six = value!.setAndReturn(new: 6)
// `six` is `6`
```

### Resources

Resources are explained in detail [in the following page](./resources.mdx).

## Unbound References / Nulls

There is **no** support for `null`.

## Inheritance and Abstract Types

There is **no** support for inheritance.
Inheritance is a feature common in other programming languages,
that allows including the fields and functions of one type in another type.

Instead, follow the "composition over inheritance" principle,
the idea of composing functionality from multiple individual parts,
rather than building an inheritance tree.

Furthermore, there is also **no** support for abstract types.
An abstract type is a feature common in other programming languages,
that prevents creating values of the type and only
allows the creation of values of a subtype.
In addition, abstract types may declare functions,
but omit the implementation of them
and instead require subtypes to implement them.

Instead, consider using [interfaces](./interfaces.mdx).
---
title: Constants and Variable Declarations
sidebar_position: 2
---

Constants and variables are declarations that bind
a value and [type](./type-safety.md) to an identifier.
Constants are initialized with a value and cannot be reassigned afterwards.
Variables are initialized with a value and can be reassigned later.
Declarations can be created in any scope, including the global scope.

Constant means that the *identifier's* association is constant,
not the *value* itself –
the value may still be changed if it is mutable.

Constants are declared using the `let` keyword. Variables are declared
using the `var` keyword.
The keywords are followed by the identifier,
an optional [type annotation](./type-annotations.md), an equals sign `=`,
and the initial value.

```cadence
// Declare a constant named `a`.
//
let a = 1

// Invalid: re-assigning to a constant.
//
a = 2

// Declare a variable named `b`.
//
var b = 3

// Assign a new value to the variable named `b`.
//
b = 4
```

Variables and constants **must** be initialized.

```cadence
// Invalid: the constant has no initial value.
//
let a
```

The names of the variable or constant
declarations in each scope must be unique.
Declaring another variable or constant with a name that is already
declared in the current scope is invalid, regardless of kind or type.

```cadence
// Declare a constant named `a`.
//
let a = 1

// Invalid: cannot re-declare a constant with name `a`,
// as it is already used in this scope.
//
let a = 2

// Declare a variable named `b`.
//
var b = 3

// Invalid: cannot re-declare a variable with name `b`,
// as it is already used in this scope.
//
var b = 4

// Invalid: cannot declare a variable with the name `a`,
// as it is already used in this scope,
// and it is declared as a constant.
//
var a = 5
```

However, variables can be redeclared in sub-scopes.

```cadence
// Declare a constant named `a`.
//
let a = 1

if true {
    // Declare a constant with the same name `a`.
    // This is valid because it is in a sub-scope.
    // This variable is not visible to the outer scope.

    let a = 2
}

// `a` is `1`
```

A variable cannot be used as its own initial value.

```cadence
// Invalid: Use of variable in its own initial value.
let a = a
```
---
title: Contract Updatability
sidebar_position: 23
---

## Introduction
A [contract](./contracts.mdx) in Cadence is a collection of data (its state) and
code (its functions) that lives in the contract storage area of an account.
When a contract is updated, it is important to make sure that the changes introduced do not lead to runtime
inconsistencies for already stored data.
Cadence maintains this state consistency by validating the contracts and all their components before an update.

## Validation Goals
The contract update validation ensures that:

- Stored data doesn't change its meaning when a contract is updated.
- Decoding and using stored data does not lead to runtime crashes.
  - For example, it is invalid to add a field because existing stored data won't have the new field.
  - Loading the existing data will result in garbage/missing values for such fields.
  - A static check of the access of the field would be valid, but the interpreter would crash when accessing the field,
    because the field has a missing/garbage value.

However, it **does not** ensure:
- Any program that imports the updated contract stays valid. e.g:
  - Updated contract may remove an existing field or may change a function signature.
  - Then any program that uses that field/function will get semantic errors.

## Updating a Contract
Changes to contracts can be introduced by adding new contracts, removing existing contracts, or updating existing
contracts. However, some of these changes may lead to data inconsistencies as stated above.

#### Valid Changes
- Adding a new contract is valid.
- Removing a contract/contract-interface that doesn't have enum declarations is valid.
- Updating a contract is valid, under the restrictions described in the below sections.

#### Invalid Changes
- Removing a contract/contract-interface that contains enum declarations is not valid.
  - Removing a contract allows adding a new contract with the same name.
  - The new contract could potentially have enum declarations with the same names as in the old contract, but with
    different structures.
  - This could change the meaning of the already stored values of those enum types.

A contract may consist of fields and other declarations such as composite types, functions, constructors, etc.
When an existing contract is updated, all its inner declarations are also validated.

### Contract Fields
When a contract is deployed, the fields of the contract are stored in an account's contract storage.
Changing the fields of a contract only changes the way the program treats the data, but does not change the already
stored data itself, which could potentially result in runtime inconsistencies as mentioned in the previous section.

See the [section about fields below](#fields) for the possible updates that can be done to the fields, and the restrictions
imposed on changing fields of a contract.

### Nested Declarations
Contracts can have nested composite type declarations such as structs, resources, interfaces, and enums.
When a contract is updated, its nested declarations are checked, because:
 - They can be used as type annotation for the fields of the same contract, directly or indirectly.
 - Any third-party contract can import the types defined in this contract and use them as type annotations.
 - Hence, changing the type definition is the same as changing the type annotation of such a field (which is also invalid,
   as described in the [section about fields fields](#fields) below).

Changes that can be done to the nested declarations, and the update restrictions are described in following sections:
 - [Structs, resources and interface](#structs-resources-and-interfaces)
 - [Enums](#enums)
 - [Functions](#functions)
 - [Constructors](#constructors)

## Fields
A field may belong to a contract, struct, resource, or interface.

#### Valid Changes:
- Removing a field is valid
  ```cadence
  // Existing contract

  pub contract Foo {
      pub var a: String
      pub var b: Int
  }


  // Updated contract

  pub contract Foo {
      pub var a: String
  }
  ```
  - It leaves data for the removed field unused at the storage, as it is no longer accessible.
  - However, it does not cause any runtime crashes.

- Changing the order of fields is valid.
  ```cadence
  // Existing contract

  pub contract Foo {
      pub var a: String
      pub var b: Int
  }


  // Updated contract

  pub contract Foo {
      pub var b: Int
      pub var a: String
  }
  ```

- Changing the access modifier of a field is valid.
  ```cadence
  // Existing contract

  pub contract Foo {
      pub var a: String
  }


  // Updated contract

  pub contract Foo {
      priv var a: String   // access modifier changed to 'priv'
  }
  ```

#### Invalid Changes
- Adding a new field is not valid.
  ```cadence
  // Existing contract

  pub contract Foo {
      pub var a: String
  }


  // Updated contract

  pub contract Foo {
      pub var a: String
      pub var b: Int      // Invalid new field
  }
  ```
    - Initializer of a contract only run once, when the contract is deployed for the first time. It does not rerun
      when the contract is updated. However it is still required to be present in the updated contract to satisfy type checks.
    - Thus, the stored data won't have the new field, as the initializations for the newly added fields do not get
      executed.
    - Decoding stored data will result in garbage or missing values for such fields.

- Changing the type of existing field is not valid.
  ```cadence
  // Existing contract

  pub contract Foo {
      pub var a: String
  }


  // Updated contract

  pub contract Foo {
      pub var a: Int      // Invalid type change
  }
  ```
    - In an already stored contract, the field `a` would have a value of type `String`.
    - Changing the type of the field `a` to `Int`, would make the runtime read the already stored `String`
      value as an `Int`, which will result in deserialization errors.
    - Changing the field type to a subtype/supertype of the existing type is also not valid, as it would also
      potentially cause issues while decoding/encoding.
      - e.g: Changing an `Int64` field to `Int8` - Stored field could have a numeric value`624`, which exceeds the value space
        for `Int8`.
      - However, this is a limitation in the current implementation, and the future versions of Cadence may support
        changing the type of field to a subtype, by providing means to migrate existing fields.

## Structs, Resources and Interfaces

#### Valid Changes:
- Adding a new struct, resource, or interface is valid.
- Adding an interface conformance to a struct/resource is valid, since the stored data only
  stores concrete type/value, but doesn't store the conformance info.
  ```cadence
  // Existing struct

  pub struct Foo {
  }


  // Upated struct

  pub struct Foo: T {
  }
  ```
  - However, if adding a conformance also requires changing the existing structure (e.g: adding a new field that is
    enforced by the new conformance), then the other restrictions (such as [restrictions on fields](#fields)) may
    prevent performing such an update.

#### Invalid Changes:
- Removing an existing declaration is not valid.
  - Removing a declaration allows adding a new declaration with the same name, but with a different structure.
  - Any program that uses that declaration would face inconsistencies in the stored data.
- Renaming a declaration is not valid. It can have the same effect as removing an existing declaration and adding
  a new one.
- Changing the type of declaration is not valid. i.e: Changing from a struct to interface, and vise versa.
  ```cadence
  // Existing struct

  pub struct Foo {
  }


  // Changed to a struct interface

  pub struct interface Foo {    // Invalid type declaration change
  }
  ```
- Removing an interface conformance of a struct/resource is not valid.
  ```cadence
  // Existing struct

  pub struct Foo: T {
  }


  // Upated struct

  pub struct Foo {
  }
  ```

### Updating Members
Similar to contracts, these composite declarations: structs, resources, and interfaces also can have fields and
other nested declarations as its member.
Updating such a composite declaration would also include updating all of its members.

Below sections describes the restrictions imposed on updating the members of a struct, resource or an interface.
- [Fields](#fields)
- [Nested structs, resources and interfaces](#structs-resources-and-interfaces)
- [Enums](#enums)
- [Functions](#functions)
- [Constructors](#constructors)

## Enums

#### Valid Changes:
- Adding a new enum declaration is valid.

#### Invalid Changes:
- Removing an existing enum declaration is invalid.
  - Otherwise, it is possible to remove an existing enum and add a new enum declaration with the same name,
    but with a different structure.
  - The new structure could potentially have incompatible changes (such as changed types, changed enum-cases, etc).
- Changing the name is invalid, as it is equivalent to removing an existing enum and adding a new one.
- Changing the raw type is invalid.
  ```cadence
  // Existing enum with `Int` raw type

  pub enum Color: Int {
    pub case RED
    pub case BLUE
  }


  // Updated enum with `UInt8` raw type

  pub enum Color: UInt8 {    // Invalid change of raw type
    pub case RED
    pub case BLUE
  }
  ```
  - When the enum value is stored, the raw value associated with the enum-case gets stored.
  - If the type is changed, then deserializing could fail if the already stored values are not in the same value space
    as the updated type.

### Updating Enum Cases
Enums consist of enum-case declarations, and updating an enum may also include changing the enums cases as well.
Enum cases are represented using their raw-value at the Cadence interpreter and runtime.
Hence, any change that causes an enum-case to change its raw value is not permitted.
Otherwise, a changed raw-value could cause an already stored enum value to have a different meaning than what
it originally was (type confusion).

#### Valid Changes:
- Adding an enum-case at the end of the existing enum-cases is valid.
  ```cadence
  // Existing enum

  pub enum Color: Int {
    pub case RED
    pub case BLUE
  }


  // Updated enum

  pub enum Color: Int {
    pub case RED
    pub case BLUE
    pub case GREEN    // valid new enum-case at the bottom
  }
  ```
#### Invalid Changes
- Adding an enum-case at the top or in the middle of the existing enum-cases is invalid.
  ```cadence
  // Existing enum

  pub enum Color: Int {
    pub case RED
    pub case BLUE
  }


  // Updated enum

  pub enum Color: Int {
    pub case RED
    pub case GREEN    // invalid new enum-case in the middle
    pub case BLUE
  }
  ```
- Changing the name of an enum-case is invalid.
  ```cadence
  // Existing enum

  pub enum Color: Int {
    pub case RED
    pub case BLUE
  }


  // Updated enum

  pub enum Color: Int {
    pub case RED
    pub case GREEN    // invalid change of names
  }
  ```
  - Previously stored raw values for `Color.BLUE` now represents `Color.GREEN`. i.e: The stored values have changed
    their meaning, and hence not a valid change.
  - Similarly, it is possible to add a new enum with the old name `BLUE`, which gets a new raw value. Then the same
    enum-case `Color.BLUE` may have used two raw-values at runtime, before and after the change, which is also invalid.

- Removing the enum case is invalid. Removing allows one to add and remove an enum-case which has the same effect
  as renaming.
  ```cadence
  // Existing enum

  pub enum Color: Int {
    pub case RED
    pub case BLUE
  }


  // Updated enum

  pub enum Color: Int {
    pub case RED

    // invalid removal of `case BLUE`
  }
  ```
- Changing the order of enum-cases is not permitted
  ```cadence
  // Existing enum

  pub enum Color: Int {
    pub case RED
    pub case BLUE
  }


  // Updated enum

  pub enum Color: UInt8 {
    pub case BLUE   // invalid change of order
    pub case RED
  }
  ```
  - Raw value of an enum is implicit, and corresponds to the defined order.
  - Changing the order of enum-cases has the same effect as changing the raw-value, which could cause storage
    inconsistencies and type-confusions as described earlier.

## Functions

Adding, changing, and deleting a function definition is always valid, as function definitions are never stored as data
(function definitions are part of the code, but not data).

- Adding a function is valid.
- Deleting a function is valid.
- Changing a function signature (parameters, return types) is valid.
- Changing a function body is valid.
- Changing the access modifiers is valid.

However, changing a *function type* may or may not be valid, depending on where it is used:
If a function type is used in the type annotation of a composite type field (direct or indirect),
then changing the function type signature is the same as changing the type annotation of that field (which is invalid).

## Constructors
Similar to functions, constructors are also not stored. Hence, any changes to constructors are valid.

## Imports
A contract may import declarations (types, functions, variables, etc.) from other programs. These imported programs are
already validated at the time of their deployment. Hence, there is no need for validating any declaration every time
they are imported.
---
title: Contracts
sidebar_position: 22
---

A contract in Cadence is a collection of type definitions
of interfaces, structs, resources, data (its state), and code (its functions)
that lives in the contract storage area of an account in Flow.

Contracts are where all composite types like structs, resources,
events, and interfaces for these types in Cadence have to be defined.
Therefore, an object of one of these types cannot exist
without having been defined in a deployed Cadence contract.

Contracts can be created, updated, and removed using the `contracts`
object of [authorized accounts](./accounts.mdx).
This functionality is covered in the [next section](#deploying-updating-and-removing-contracts)

Contracts are types.
They are similar to composite types, but are stored differently than
structs or resources and cannot be used as values, copied, or moved
like resources or structs.

Contracts stay in an account's contract storage area
and can only be added, updated, or removed by the account owner with special commands.

Contracts are declared using the `contract` keyword. The keyword is followed
by the name of the contract.

```cadence
pub contract SomeContract {
    // ...
}
```

Contracts cannot be nested in each other.

```cadence
pub contract Invalid {

    // Invalid: Contracts cannot be nested in any other type.
    //
    pub contract Nested {
        // ...
    }
}
```

One of the simplest forms of a contract would just be one with a state field,
a function, and an `init` function that initializes the field:

```cadence
pub contract HelloWorld {

    // Declare a stored state field in HelloWorld
    //
    pub let greeting: String

    // Declare a function that can be called by anyone
    // who imports the contract
    //
    pub fun hello(): String {
        return self.greeting
    }

    init() {
        self.greeting = "Hello World!"
    }
}
```

This contract could be deployed to an account and live permanently
in the contract storage.  Transactions and other contracts
can interact with contracts by importing them at the beginning
of a transaction or contract definition.

Anyone could call the above contract's `hello` function by importing
the contract from the account it was deployed to and using the imported
object to call the hello function.

```cadence
import HelloWorld from 0x42

// Invalid: The contract does not know where hello comes from
//
log(hello())        // Error

// Valid: Using the imported contract object to call the hello
// function
//
log(HelloWorld.hello())    // prints "Hello World!"

// Valid: Using the imported contract object to read the greeting
// field.
log(HelloWorld.greeting)   // prints "Hello World!"

// Invalid: Cannot call the init function after the contract has been created.
//
HelloWorld.init()    // Error
```

There can be any number of contracts per account
and they can include an arbitrary amount of data.
This means that a contract can have any number of fields, functions, and type definitions,
but they have to be in the contract and not another top-level definition.

```cadence
// Invalid: Top-level declarations are restricted to only be contracts
//          or contract interfaces. Therefore, all of these would be invalid
//          if they were deployed to the account contract storage and
//          the deployment would be rejected.
//
pub resource Vault {}
pub struct Hat {}
pub fun helloWorld(): String {}
let num: Int
```

Another important feature of contracts is that instances of resources and events
that are declared in contracts can only be created/emitted within functions or types
that are declared in the same contract.

It is not possible create instances of resources and events outside the contract.

The contract below defines a resource interface `Receiver` and a resource `Vault`
that implements that interface.  The way this example is written,
there is no way to create this resource, so it would not be usable.

```cadence
// Valid
pub contract FungibleToken {

    pub resource interface Receiver {

        pub balance: Int

        pub fun deposit(from: @{Receiver}) {
            pre {
                from.balance > 0:
                    "Deposit balance needs to be positive!"
            }
            post {
                self.balance == before(self.balance) + before(from.balance):
                    "Incorrect amount removed"
            }
        }
    }

    pub resource Vault: Receiver {

        // keeps track of the total balance of the accounts tokens
        pub var balance: Int

        init(balance: Int) {
            self.balance = balance
        }

        // withdraw subtracts amount from the vaults balance and
        // returns a vault object with the subtracted balance
        pub fun withdraw(amount: Int): @Vault {
            self.balance = self.balance - amount
            return <-create Vault(balance: amount)
        }

        // deposit takes a vault object as a parameter and adds
        // its balance to the balance of the Account's vault, then
        // destroys the sent vault because its balance has been consumed
        pub fun deposit(from: @{Receiver}) {
            self.balance = self.balance + from.balance
            destroy from
        }
    }
}
```

If a user tried to run a transaction that created an instance of the `Vault` type,
the type checker would not allow it because only code in the `FungibleToken`
contract can create new `Vault`s.

```cadence
import FungibleToken from 0x42

// Invalid: Cannot create an instance of the `Vault` type outside
// of the contract that defines `Vault`
//
let newVault <- create FungibleToken.Vault(balance: 10)
```

The contract would have to either define a function that creates new
`Vault` instances or use its `init` function to create an instance and
store it in the owner's account storage.

This brings up another key feature of contracts in Cadence.  Contracts
can interact with its account's `storage` and `published` objects to store
resources, structs, and references.
They do so by using the special `self.account` object that is only accessible within the contract.

Imagine that these were declared in the above `FungibleToken` contract.

```cadence

    pub fun createVault(initialBalance: Int): @Vault {
        return <-create Vault(balance: initialBalance)
    }

    init(balance: Int) {
        let vault <- create Vault(balance: 1000)
        self.account.save(<-vault, to: /storage/initialVault)
    }
```

Now, any account could call the `createVault` function declared in the contract
to create a `Vault` object.
Or the owner could call the `withdraw` function on their own `Vault` to send new vaults to others.

```cadence
import FungibleToken from 0x42

// Valid: Create an instance of the `Vault` type by calling the contract's
// `createVault` function.
//
let newVault <- create FungibleToken.createVault(initialBalance: 10)
```

## Account access

Contracts have the implicit field `let account: AuthAccount`,
which is the account in which the contract is deployed too.
This gives the contract the ability to e.g. read and write to the account's storage.

## Deploying, Updating, and Removing Contracts

In order for a contract to be used in Cadence, it needs to be deployed to an account.
The deployed contracts of an account can be accessed through the `contracts` object.

### Deployed Contracts

Accounts store "deployed contracts", that is, the code of the contract:

```cadence
pub struct DeployedContract {
    /// The address of the account where the contract is deployed at.
    pub let address: Address

    /// The name of the contract.
    pub let name: String

    /// The code of the contract.
    pub let code: [UInt8]

    /// Returns an array of `Type` objects representing all the public type declarations in this contract
    /// (e.g. structs, resources, enums).
    ///
    /// For example, given a contract
    /// ```
    /// contract Foo {
    ///       pub struct Bar {...}
    ///       pub resource Qux {...}
    /// }
    /// ```
    /// then `.publicTypes()` will return an array equivalent to the expression `[Type<Bar>(), Type<Qux>()]`
    pub fun publicTypes(): [Type]
}
```

Note that this is not the contract instance that can be acquired by importing it.

### Deploying a New Contract

A new contract can be deployed to an account using the `add` function:

  ```cadence
  fun add(
      name: String,
      code: [UInt8],
      ... contractInitializerArguments
  ): DeployedContract
  ```

  Adds the given contract to the account.

  The `code` parameter is the UTF-8 encoded representation of the source code.
  The code must contain exactly one contract or contract interface,
  which must have the same name as the `name` parameter.

  All additional arguments that are given are passed further to the initializer
  of the contract that is being deployed.

  Fails if a contract/contract interface with the given name already exists in the account,
  if the given code does not declare exactly one contract or contract interface,
  or if the given name does not match the name of the contract/contract interface declaration in the code.

  Returns the [deployed contract](#deployed-contracts).

For example, assuming the following contract code should be deployed:

```cadence
pub contract Test {
    pub let message: String

    init(message: String) {
        self.message = message
    }
}
```

The contract can be deployed as follows:

```cadence
// Decode the hex-encoded source code into a byte array
// using the built-in function `decodeHex`.
//
// (The ellipsis ... indicates the remainder of the string)
//
let code = "70756220636f6e...".decodeHex()

// `code` has type `[UInt8]`

let signer: AuthAccount = ...
signer.contracts.add(
    name: "Test",
    code: code,
    message: "I'm a new contract in an existing account"
)
```

### Updating a Deployed Contract

<Callout type="info">

🚧 Status: Updating contracts is **experimental**.

Updating contracts is currently limited to maintain data consistency.
[Certain restrictions are imposed](./contract-updatability.md).

</Callout>

A deployed contract can be updated using the `update__experimental` function:

  ```cadence
  fun update__experimental(name: String, code: [UInt8]): DeployedContract
  ```

  Updates the code for the contract/contract interface in the account.

  The `code` parameter is the UTF-8 encoded representation of the source code.
  The code must contain exactly one contract or contract interface,
  which must have the same name as the `name` parameter.

  Does **not** run the initializer of the contract/contract interface again.
  The contract instance in the world state stays as is.

  Fails if no contract/contract interface with the given name exists in the account,
  if the given code does not declare exactly one contract or contract interface,
  or if the given name does not match the name of the contract/contract interface declaration in the code.

  Returns the [deployed contract](#deployed-contracts) for the updated contract.

For example, assuming that a contract named `Test` is already deployed to the account
and it should be updated with the following contract code:

```cadence
pub contract Test {
    pub let message: String

    init(message: String) {
        self.message = message
    }
}
```

The contract can be updated as follows:

```cadence
// Decode the hex-encoded source code into a byte array
// using the built-in function `decodeHex`.
//
// (The ellipsis ... indicates the remainder of the string)
//
let code = "70756220636f6e...".decodeHex()

// `code` has type `[UInt8]`

let signer: AuthAccount = ...
signer.contracts.update__experimental(name: "Test", code: code)
```

Updating a contract does **not** currently change any existing stored data.
Only the code of the contract is updated.

### Getting a Deployed Contract

A deployed contract can be gotten from an account using the `get` function:

  ```cadence
  fun get(name: String): DeployedContract?
  ```

  Returns the [deployed contract](#deployed-contracts) for the contract/contract interface with the given name in the account, if any.

  Returns `nil` if no contract/contract interface with the given name exists in the account.

For example, assuming that a contract named `Test` is deployed to an account, the contract can be retrieved as follows:

```cadence
let signer: AuthAccount = ...
let contract = signer.contracts.get(name: "Test")
```

### Borrowing a Deployed Contract

In contrast to a static contract import `import T from 0x1`,
which will always perform an import of a type,
contracts can be "borrowed" to effectively perform a dynamic import dependent on a specific execution path.

A reference to a deployed contract contract can obtained using the `borrow` function:

  ```cadence
  fun borrow<T: &Any>(name: String): T?
  ```

  This returns a reference to the contract value stored with that name on the account,
  if it exists, and if it has the provided type `T`.

  Returns `nil` if no contract/contract interface with the given name exists in the account.

For example, assuming that a contract named `Test` which conforms to the `TestInterface` interface is deployed to an account, the contract can be retrieved as follows:

```cadence
let signer: AuthAccount = ...
let contract: &TestInterface = signer.contracts.borrow<&TestInterface>(name: "Test")

### Removing a Deployed Contract

A deployed contract can be removed from an account using the `remove` function:

```cadence
fun remove(name: String): DeployedContract?
```

 Removes the contract/contract interface from the account which has the given name, if any.

 Returns the removed [deployed contract](#deployed-contracts), if any.

 Returns `nil` if no contract/contract interface with the given name exist in the account.

For example, assuming that a contract named `Test` is deployed to an account, the contract can be removed as follows:

```cadence
let signer: AuthAccount = ...
let contract = signer.contracts.remove(name: "Test")
```

## Contract Interfaces

Like composite types, contracts can have interfaces that specify rules
about their behavior, their types, and the behavior of their types.

Contract interfaces have to be declared globally.  Declarations
cannot be nested in other types.

If a contract interface declares a concrete type, implementations of it
must also declare the same concrete type conforming to the type requirement.

If a contract interface declares an interface type, the implementing contract
does not have to also define that interface.  They can refer to that nested
interface by saying `{ContractInterfaceName}.{NestedInterfaceName}`

```cadence
// Declare a contract interface that declares an interface and a resource
// that needs to implement that interface in the contract implementation.
//
pub contract interface InterfaceExample {

    // Implementations do not need to declare this
    // They refer to it as InterfaceExample.NestedInterface
    //
    pub resource interface NestedInterface {}

    // Implementations must declare this type
    //
    pub resource Composite: NestedInterface {}
}

pub contract ExampleContract: InterfaceExample {

    // The contract doesn't need to redeclare the `NestedInterface` interface
    // because it is already declared in the contract interface

    // The resource has to refer to the resource interface using the name
    // of the contract interface to access it
    //
    pub resource Composite: InterfaceExample.NestedInterface {
    }
}
```
---
title: Control Flow
sidebar_position: 7
---

Control flow statements control the flow of execution in a function.

## Conditional branching: if-statement

If-statements allow a certain piece of code to be executed only when a given condition is true.

The if-statement starts with the `if` keyword, followed by the condition,
and the code that should be executed if the condition is true
inside opening and closing braces.
The condition expression must be boolean.
The braces are required and not optional.
Parentheses around the condition are optional.

```cadence
let a = 0
var b = 0

if a == 0 {
   b = 1
}

// Parentheses can be used around the condition, but are not required.
if (a != 0) {
   b = 2
}

// `b` is `1`
```

An additional, optional else-clause can be added to execute another piece of code
when the condition is false.
The else-clause is introduced by the `else` keyword followed by braces
that contain the code that should be executed.

```cadence
let a = 0
var b = 0

if a == 1 {
   b = 1
} else {
   b = 2
}

// `b` is `2`
```

The else-clause can contain another if-statement, i.e., if-statements can be chained together.
In this case the braces can be omitted.

```cadence
let a = 0
var b = 0

if a == 1 {
   b = 1
} else if a == 2 {
   b = 2
} else {
   b = 3
}

// `b` is `3`

if a == 1 {
   b = 1
} else {
    if a == 0 {
        b = 2
    }
}

// `b` is `2`
```

## Optional Binding

Optional binding allows getting the value inside an optional.
It is a variant of the if-statement.

If the optional contains a value, the first branch is executed
and a temporary constant or variable is declared and set to the value contained in the optional;
otherwise, the else branch (if any) is executed.

Optional bindings are declared using the `if` keyword like an if-statement,
but instead of the boolean test value, it is followed by the `let` or `var` keywords,
to either introduce a constant or variable, followed by a name,
the equal sign (`=`), and the optional value.

```cadence
let maybeNumber: Int? = 1

if let number = maybeNumber {
    // This branch is executed as `maybeNumber` is not `nil`.
    // The constant `number` is `1` and has type `Int`.
} else {
    // This branch is *not* executed as `maybeNumber` is not `nil`
}
```

```cadence
let noNumber: Int? = nil

if let number = noNumber {
    // This branch is *not* executed as `noNumber` is `nil`.
} else {
    // This branch is executed as `noNumber` is `nil`.
    // The constant `number` is *not* available.
}
```

## Switch

Switch-statements compare a value against several possible values of the same type, in order.
When an equal value is found, the associated block of code is executed.

The switch-statement starts with the `switch` keyword, followed by the tested value,
followed by the cases inside opening and closing braces.
The test expression must be equatable.
The braces are required and not optional.

Each case is a separate branch of code execution
and starts with the `case` keyword,
followed by a possible value, a colon (`:`),
and the block of code that should be executed
if the case's value is equal to the tested value.

The block of code associated with a switch case
[does not implicitly fall through](#no-implicit-fallthrough),
and must contain at least one statement.
Empty blocks are invalid.

An optional default case may be given by using the `default` keyword.
The block of code of the default case is executed
when none of the previous case tests succeeded.
It must always appear last.

```cadence
fun word(_ n: Int): String {
    // Test the value of the parameter `n`
    switch n {
    case 1:
        // If the value of variable `n` is equal to `1`,
        // then return the string "one"
        return "one"
    case 2:
        // If the value of variable `n` is equal to `2`,
        // then return the string "two"
        return "two"
    default:
        // If the value of variable `n` is neither equal to `1` nor to `2`,
        // then return the string "other"
        return "other"
    }
}

word(1)  // returns "one"
word(2)  // returns "two"
word(3)  // returns "other"
word(4)  // returns "other"
```

### Duplicate cases

Cases are tested in order, so if a case is duplicated,
the block of code associated with the first case that succeeds is executed.

```cadence
fun test(_ n: Int): String {
    // Test the value of the parameter `n`
    switch n {
    case 1:
        // If the value of variable `n` is equal to `1`,
        // then return the string "one"
        return "one"
    case 1:
        // If the value of variable `n` is equal to `1`,
        // then return the string "also one".
        // This is a duplicate case for the one above.
        return "also one"
    default:
        // If the value of variable `n` is neither equal to `1` nor to `2`,
        // then return the string "other"
        return "other"
    }
}

word(1) // returns "one", not "also one"
```

### `break`

The block of code associated with a switch case may contain a `break` statement.
It ends the execution of the switch statement immediately
and transfers control to the code after the switch statement

### No Implicit Fallthrough

Unlike switch statements in some other languages,
switch statements in Cadence do not "fall through":
execution of the switch statement finishes as soon as the block of code
associated with the first matching case is completed.
No explicit `break` statement is required.

This makes the switch statement safer and easier to use,
avoiding the accidental execution of more than one switch case.

Some other languages implicitly fall through
to the block of code associated with the next case,
so it is common to write cases with an empty block
to handle multiple values in the same way.

To prevent developers from writing switch statements
that assume this behaviour, blocks must have at least one statement.
Empty blocks are invalid.

```cadence
fun words(_ n: Int): [String] {
    // Declare a variable named `result`, an array of strings,
    // which stores the result
    let result: [String] = []

    // Test the value of the parameter `n`
    switch n {
    case 1:
        // If the value of variable `n` is equal to `1`,
        // then append the string "one" to the result array
        result.append("one")
    case 2:
        // If the value of variable `n` is equal to `2`,
        // then append the string "two" to the result array
        result.append("two")
    default:
        // If the value of variable `n` is neither equal to `1` nor to `2`,
        // then append the string "other" to the result array
        result.append("other")
    }
    return result
}

words(1)  // returns `["one"]`
words(2)  // returns `["two"]`
words(3)  // returns `["other"]`
words(4)  // returns `["other"]`
```

## Looping

### while-statement

While-statements allow a certain piece of code to be executed repeatedly,
as long as a condition remains true.

The while-statement starts with the `while` keyword, followed by the condition,
and the code that should be repeatedly
executed if the condition is true inside opening and closing braces.
The condition must be boolean and the braces are required.

The while-statement will first evaluate the condition.
If it is true, the piece of code is executed and the evaluation of the condition is repeated.
If the condition is false, the piece of code is not executed
and the execution of the whole while-statement is finished.
Thus, the piece of code is executed zero or more times.

```cadence
var a = 0
while a < 5 {
    a = a + 1
}

// `a` is `5`
```

### For-in statement

For-in statements allow a certain piece of code to be executed repeatedly for
each element in an array.

The for-in statement starts with the `for` keyword, followed by the name of
the element that is used in each iteration of the loop,
followed by the `in` keyword, and then followed by the array
that is being iterated through in the loop.

Then, the code that should be repeatedly executed in each iteration of the loop
is enclosed in curly braces.

If there are no elements in the data structure, the code in the loop will not
be executed at all. Otherwise, the code will execute as many times
as there are elements in the array.

```cadence
let array = ["Hello", "World", "Foo", "Bar"]

for element in array {
    log(element)
}

// The loop would log:
// "Hello"
// "World"
// "Foo"
// "Bar"
```

Optionally, developers may include an additional variable preceding the element name,
separated by a comma.
When present, this variable contains the current
index of the array being iterated through
during each repeated execution (starting from 0).

```cadence
let array = ["Hello", "World", "Foo", "Bar"]

for index, element in array {
    log(index)
}

// The loop would log:
// 0
// 1
// 2
// 3
```

To iterate over a dictionary's entries (keys and values),
use a for-in loop over the dictionary's keys and get the value for each key:

```cadence
let dictionary = {"one": 1, "two": 2}
for key in dictionary.keys {
    let value = dictionary[key]!
    log(key)
    log(value)
}

// The loop would log:
// "one"
// 1
// "two"
// 2
```

Alternatively, dictionaries carry a method `forEachKey` that avoids allocating an intermediate array for keys:

```cadence
let dictionary = {"one": 1, "two": 2, "three": 3}
dictionary.forEachKey(fun (key: String): Bool {
    let value = dictionary[key]
    log(key)
    log(value)

    return key != "two" // stop iteration if this returns false
})
```

### `continue` and `break`

In for-loops and while-loops, the `continue` statement can be used to stop
the current iteration of a loop and start the next iteration.

```cadence
var i = 0
var x = 0
while i < 10 {
    i = i + 1
    if i < 3 {
        continue
    }
    x = x + 1
}
// `x` is `8`


let array = [2, 2, 3]
var sum = 0
for element in array {
    if element == 2 {
        continue
    }
    sum = sum + element
}

// `sum` is `3`

```

The `break` statement can be used to stop the execution
of a for-loop or a while-loop.

```cadence
var x = 0
while x < 10 {
    x = x + 1
    if x == 5 {
        break
    }
}
// `x` is `5`


let array = [1, 2, 3]
var sum = 0
for element in array {
    if element == 2 {
        break
    }
    sum = sum + element
}

// `sum` is `1`
```

## Immediate function return: return-statement

The return-statement causes a function to return immediately,
i.e., any code after the return-statement is not executed.
The return-statement starts with the `return` keyword
and is followed by an optional expression that should be the return value of the function call.
---
title: Core Events
sidebar_position: 25
---

Core events are events emitted directly from the FVM (Flow Virtual Machine).
The events have the same name on all networks and do not follow the standard naming (they have no address).

Refer to the [`PublicKey` section](./crypto.mdx#publickey) for more details on the information provided for account key events.

### Account Created

Event that is emitted when a new account gets created.

Event name: `flow.AccountCreated`


```cadence
pub event AccountCreated(address: Address)
```

| Field             | Type      | Description                              |
| ----------------- | --------- | ---------------------------------------- |
| `address`         | `Address` | The address of the newly created account |


### Account Key Added

Event that is emitted when a key gets added to an account.

Event name: `flow.AccountKeyAdded`

```cadence
pub event AccountKeyAdded(
    address: Address,
    publicKey: PublicKey
)
```

| Field         | Type        | Description                                     |
| ------------- | ----------- | ----------------------------------------------- |
| `address`     | `Address`   | The address of the account the key is added to  |
| `publicKey`   | `PublicKey` | The public key added to the account             |


### Account Key Removed

Event that is emitted when a key gets removed from an account.

Event name: `flow.AccountKeyRemoved`

```cadence
pub event AccountKeyRemoved(
    address: Address,
    publicKey: PublicKey
)
```

| Field       | Type        | Description                                         |
| ----------- | ----------- | --------------------------------------------------- |
| `address`   | `Address`   | The address of the account the key is removed from  |
| `publicKey` | `PublicKey` | Public key removed from the account                 |


### Account Contract Added

Event that is emitted when a contract gets deployed to an account.

Event name: `flow.AccountContractAdded`

```cadence
pub event AccountContractAdded(
    address: Address,
    codeHash: [UInt8],
    contract: String
)
```

| Field       | Type   | Description                                                  |
| ----------- | ------ | ------------------------------------------------------------ |
| `address`   | `Address` | The address of the account the contract gets deployed to  |
| `codeHash`  | `[UInt8]` | Hash of the contract source code                          |
| `contract`  | `String`  | The name of the the contract                              |

### Account Contract Updated

Event that is emitted when a contract gets updated on an account.

Event name: `flow.AccountContractUpdated`

```cadence
pub event AccountContractUpdated(
    address: Address,
    codeHash: [UInt8],
    contract: String
)
```

| Field       | Type      | Description                                              |
| ----------- | --------- | -------------------------------------------------------- |
| `address`   | `Address` | The address of the account where the updated contract is deployed  |
| `codeHash`  | `[UInt8]` | Hash of the contract source code                         |
| `contract`  | `String`  | The name of the the contract                             |


### Account Contract Removed

Event that is emitted when a contract gets removed from an account.

Event name: `flow.AccountContractRemoved`

```cadence
pub event AccountContractRemoved(
    address: Address,
    codeHash: [UInt8],
    contract: String
)
```

| Field       | Type      | Description                                               |
| ----------- | --------- | --------------------------------------------------------- |
| `address`   | `Address` | The address of the account the contract gets removed from |
| `codeHash`  | `[UInt8]` | Hash of the contract source code                          |
| `contract`  | `String`  | The name of the the contract                              |

### Inbox Value Published

Event that is emitted when a Capability is published from an account.

Event name: `flow.InboxValuePublished`

```cadence
pub event InboxValuePublished(provider: Address, recipient: Address, name: String, type: Type)
```

| Field             | Type      | Description                                  |
| ----------------- | --------- | -------------------------------------------- |
| `provider`        | `Address` | The address of the publishing account        |
| `recipient`       | `Address` | The address of the intended recipient        |
| `name`            | `String`  | The name associated with the published value |
| `type`            | `Type`    | The type of the published value              |

To reduce the potential for spam,
we recommend that user agents that display events do not display this event as-is to their users,
and allow users to restrict whom they see events from.

### Inbox Value Unpublished

Event that is emitted when a Capability is unpublished from an account.

Event name: `flow.InboxValueUnpublished`

```cadence
pub event InboxValueUnpublished(provider: Address, name: String)
```

| Field           | Type      | Description                                  |
| --------------- | --------- | -------------------------------------------- |
| `provider`      | `Address` | The address of the publishing account        |
| `name`          | `String`  | The name associated with the published value |

To reduce the potential for spam,
we recommend that user agents that display events do not display this event as-is to their users,
and allow users to restrict whom they see events from.

### Inbox Value Claimed

Event that is emitted when a Capability is claimed by an account.

Event name: `flow.InboxValueClaimed`

```cadence
pub event InboxValueClaimed(provider: Address, recipient: Address, name: String)
```

| Field           | Type      | Description                                  |
| --------------- | --------- | -------------------------------------------- |
| `provider`      | `Address` | The address of the publishing account        |
| `recipient`     | `Address` | The address of the claiming recipient        |
| `name`          | `String`  | The name associated with the published value |

To reduce the potential for spam,
we recommend that user agents that display events do not display this event as-is to their users,
and allow users to restrict whom they see events from.---
title: Crypto
sidebar_position: 30
---

## Hashing

The built-in enum `HashAlgorithm` provides the set of hashing algorithms that
are supported by the language natively.

```cadence
pub enum HashAlgorithm: UInt8 {
    /// SHA2_256 is SHA-2 with a 256-bit digest (also referred to as SHA256).
    pub case SHA2_256 = 1

    /// SHA2_384 is SHA-2 with a 384-bit digest (also referred to as  SHA384).
    pub case SHA2_384 = 2

    /// SHA3_256 is SHA-3 with a 256-bit digest.
    pub case SHA3_256 = 3

    /// SHA3_384 is SHA-3 with a 384-bit digest.
    pub case SHA3_384 = 4

    /// KMAC128_BLS_BLS12_381 is an instance of KECCAK Message Authentication Code (KMAC128) mac algorithm.
    /// Although this is a MAC algorithm, KMAC is included in this list as it can be used hash
    /// when the key is used a non-public customizer.
    /// KMAC128_BLS_BLS12_381 is used in particular as the hashing algorithm for the BLS signature scheme on the curve BLS12-381.
    /// It is a customized version of KMAC128 that is compatible with the hashing to curve
    /// used in BLS signatures.
    /// It is the same hasher used by signatures in the internal Flow protocol.
    pub case KMAC128_BLS_BLS12_381 = 5

    /// KECCAK_256 is the legacy Keccak algorithm with a 256-bits digest, as per the original submission to the NIST SHA3 competition.
    /// KECCAK_256 is different than SHA3 and is used by Ethereum.
    pub case KECCAK_256 = 6

    /// Returns the hash of the given data
    pub fun hash(_ data: [UInt8]): [UInt8]

    /// Returns the hash of the given data and tag
    pub fun hashWithTag(_ data: [UInt8], tag: string): [UInt8]
}
```

The hash algorithms provide two ways to hash input data into digests, `hash` and `hashWithTag`.

## Hashing

`hash` hashes the input data using the chosen hashing algorithm.
`KMAC` is the only MAC algorithm on the list
and configured with specific parameters (detailed in [KMAC128 for BLS](#KMAC128-for-BLS))

For example, to compute a SHA3-256 digest:

```cadence
let data: [UInt8] = [1, 2, 3]
let digest = HashAlgorithm.SHA3_256.hash(data)
```

## Hashing with a domain tag

`hashWithTag` hashes the input data along with an input tag.
It allows instanciating independent hashing functions customized with a domain separation tag (DST).
For most of the hashing algorithms, mixing the data with the tag is done by pre-fixing the data with the tag and
hashing the result.

- `SHA2_256`, `SHA2_384`, `SHA3_256`, `SHA3_384`, `KECCAK_256`:
  If the tag is non-empty, the hashed message is `bytes(tag) || data` where `bytes()` is the UTF-8 encoding of the input string,
  padded with zeros till 32 bytes.
  Therefore tags must not exceed 32 bytes.
  If the tag used is empty, no data prefix is applied, and the hashed message is simply `data` (same as `hash` output).
- `KMAC128_BLS_BLS12_381`: refer to [KMAC128 for BLS](#KMAC128-for-BLS) for details.


### KMAC128 for BLS

`KMAC128_BLS_BLS12_381` is an instance of the cSHAKE-based KMAC128.
Although this is a MAC algorithm, KMAC can be used as a hash when the key is used as a non-private customizer.
`KMAC128_BLS_BLS12_381` is used in particular as the hashing algorithm for the BLS signature scheme on the curve BLS12-381.
It is a customized instance of KMAC128 and is compatible with the hashing to curve used by BLS signatures.
It is the same hasher used by the internal Flow protocol, and can be used to verify Flow protocol signatures on-chain.

To define the MAC instance, `KMAC128(customizer, key, data, length)` is instanciated with the following parameters
(as referred to by the NIST [SHA-3 Derived Functions](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-185.pdf)):
  - `customizer` is the UTF-8 encoding of `"H2C"`.
  - `key` is the UTF-8 encoding of `"FLOW--V00-CS00-with-BLS_SIG_BLS12381G1_XOF:KMAC128_SSWU_RO_POP_"` when `hash` is used. It includes the input `tag`
  when `hashWithTag` is used and key becomes the UTF-8 encoding of `"FLOW-" || tag || "-V00-CS00-with-BLS_SIG_BLS12381G1_XOF:KMAC128_SSWU_RO_POP_"`.
  - `data` is the input data to hash.
  - `length` is 1024 bytes.

## Signing Algorithms

The built-in enum `SignatureAlgorithm` provides the set of signing algorithms that
are supported by the language natively.

```cadence
pub enum SignatureAlgorithm: UInt8 {
    /// ECDSA_P256 is ECDSA on the NIST P-256 curve.
    pub case ECDSA_P256 = 1

    /// ECDSA_secp256k1 is ECDSA on the secp256k1 curve.
    pub case ECDSA_secp256k1 = 2

    /// BLS_BLS12_381 is BLS signature scheme on the BLS12-381 curve.
    /// The scheme is set-up so that signatures are in G_1 (subgroup of the curve over the prime field)
    /// while public keys are in G_2 (subgroup of the curve over the prime field extension).
    pub case BLS_BLS12_381 = 3
}
```

## PublicKey

`PublicKey` is a built-in structure that represents a cryptographic public key of a signature scheme.

```cadence
struct PublicKey {
    let publicKey: [UInt8]
    let signatureAlgorithm: SignatureAlgorithm

    /// Verifies a signature under the given tag, data and public key.
    /// It uses the given hash algorithm to hash the tag and data.
    pub fun verify(
        signature: [UInt8],
        signedData: [UInt8],
        domainSeparationTag: String,
        hashAlgorithm: HashAlgorithm
    ): Bool

    /// Verifies the proof of possession of the private key.
    /// This function is only implemented if the signature algorithm
    /// of the public key is BLS (BLS_BLS12_381).
    /// If called with any other signature algorithm, the program aborts
    pub fun verifyPoP(_ proof: [UInt8]): Bool
}
```

`PublicKey` supports two methods `verify` and `verifyPoP`.
`verifyPoP` will be covered under [BLS multi-signature](#proof-of-possession-pop).

### Public Key construction

A `PublicKey` can be constructed using the raw key and the signing algorithm.

```cadence
let publicKey = PublicKey(
    publicKey: "010203".decodeHex(),
    signatureAlgorithm: SignatureAlgorithm.ECDSA_P256
)
```

The raw key value depends on the supported signature scheme:

- `ECDSA_P256` and `ECDSA_secp256k1`:
  The public key is an uncompressed curve point `(X,Y)` where `X` and `Y` are two prime field elements.
  The raw key is represented as `bytes(X) || bytes(Y)`, where `||` is the concatenation operation,
  and `bytes()` is the bytes big-endian encoding left padded by zeros to the byte-length of the field prime.
  The raw public key is 64-bytes long.

- `BLS_BLS_12_381`:
  The public key is a G_2 element (on the curve over the prime field extension).
  The encoding follows the compressed serialization defined in the
  [IETF draft-irtf-cfrg-pairing-friendly-curves-08](https://www.ietf.org/archive/id/draft-irtf-cfrg-pairing-friendly-curves-08.html#name-point-serialization-procedu).
  A public key is 96-bytes long.

### Public Key validation

A public key is validated at the time of creation. Only valid public keys can be created.
The validation of the public key depends on the supported signature scheme:

- `ECDSA_P256` and `ECDSA_secp256k1`:
  The given `X` and `Y` coordinates are correctly serialized, represent valid prime field elements, and the resulting
  point is on the correct curve (no subgroup check needed since the cofactor of both supported curves is 1).

- `BLS_BLS_12_381`:
  The given key is correctly serialized following the compressed serialization in [IETF draft-irtf-cfrg-pairing-friendly-curves-08](https://www.ietf.org/archive/id/draft-irtf-cfrg-pairing-friendly-curves-08.html#name-point-serialization-procedu).
  The coordinates represent valid prime field extension elements. The resulting point is on the curve, and is on the correct subgroup G_2.
  Note that the point at infinity is accepted and yields the identity public key. Such identity key can be useful when aggregating multiple keys.

<Callout type="info">

🚧 Status: Accepting the BLS identity key is going to be available in the upcoming release of Cadence on Mainnet.

</Callout>

Since the validation happens only at the time of creation, public keys are immutable.

```cadence
publicKey.signatureAlgorithm = SignatureAlgorithm.ECDSA_secp256k1   // Not allowed
publicKey.publicKey = []                                            // Not allowed

publicKey.publicKey[2] = 4      // No effect
```

Invalid public keys cannot be constructed so public keys are always valid.

### Signature verification

A signature can be verified using the `verify` function of the `PublicKey`:

```cadence
let pk = PublicKey(
    publicKey: "96142CE0C5ECD869DC88C8960E286AF1CE1B29F329BA4964213934731E65A1DE480FD43EF123B9633F0A90434C6ACE0A98BB9A999231DB3F477F9D3623A6A4ED".decodeHex(),
    signatureAlgorithm: SignatureAlgorithm.ECDSA_P256
)

let signature = "108EF718F153CFDC516D8040ABF2C8CC7AECF37C6F6EF357C31DFE1F7AC79C9D0145D1A2F08A48F1A2489A84C725D6A7AB3E842D9DC5F8FE8E659FFF5982310D".decodeHex()
let message : [UInt8] = [1, 2, 3]

let isValid = pk.verify(
    signature: signature,
    signedData: message,
    domainSeparationTag: "",
    hashAlgorithm: HashAlgorithm.SHA2_256
)
// `isValid` is false
```

The inputs to `verify` depend on the signature scheme used:

- ECDSA (`ECDSA_P256` and `ECDSA_secp256k1`):
  - `signature` expects the couple `(r,s)`. It is serialized as `bytes(r) || bytes(s)`, where `||` is the concatenation operation,
  and `bytes()` is the bytes big-endian encoding left padded by zeros to the byte-length of the curve order.
  The signature is 64 bytes-long for both curves.
  - `signedData` is the arbitrary message to verify the signature against.
  - `domainSeparationTag` is the expected domain tag. Multiple valid tag values can be used (check [`hashWithTag`](#hashing) for more details).
  - `hashAlgorithm` is either `SHA2_256`, `SHA3_256` or `KECCAK_256`. It is the algorithm used to hash the message along with the given tag (check the `hashWithTag`[function](#hashing) for more details).

As noted in [`hashWithTag`](#hashing) for `SHA2_256`, `SHA3_256` and `KECCAK_256`, using an empty `tag` results in hashing the input data only. If a signature verification
needs to be done against data without any domain tag, this can be done by using an empty domain tag `""`.

ECDSA verification is implemented as defined in ANS X9.62 (also referred by [FIPS 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf) and [SEC 1, Version 2.0](https://www.secg.org/sec1-v2.pdf)).
A valid signature would be generated using the expected `signedData`, `domainSeparationTag` and `hashAlgorithm` used to verify.

- BLS (`BLS_BLS_12_381`):
  - `signature` expects a G_1 point (on the curve over the prime field).
  The encoding follows the compressed serialization defined in the [IETF draft-irtf-cfrg-pairing-friendly-curves-08](https://www.ietf.org/archive/id/draft-irtf-cfrg-pairing-friendly-curves-08.html#name-point-serialization-procedu).
  A signature is 48-bytes long.
  - `signedData` is the arbitrary message to verify the signature against.
  - `domainSeparationTag` is the expected domain tag. All tags are accepted (check [KMAC128 for BLS](#KMAC128-for-BLS)).
  - `hashAlgorithm` only accepts `KMAC128_BLS_BLS12_381`. It is the algorithm used to hash the message along with the given tag (check [KMAC128 for BLS](#KMAC128-for-BLS)).

BLS verification performs the necessary membership check on the signature while the membership check of the public key is performed at the creation of the `PublicKey` object
and is not repeated during the signature verification. In order to prevent equivocation issues, a verification under the identity public key always returns `false`.

<Callout type="info">

🚧 Status: Returning `false` when verifying against a BLS identity key is going to be available in the upcoming release of Cadence on Mainnet.
Currently, BLS identity keys can only be constructed as an output of `aggregatePublicKeys`.

</Callout>

The verificaction uses a hash-to-curve algorithm to hash the `signedData` into a `G_1` point, following the `hash_to_curve` method described in the [draft-irtf-cfrg-hash-to-curve-14](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-hash-to-curve-14#section-3).
While KMAC128 is used as a hash-to-field method resulting in two field elements, the mapping to curve is implemented using the [simplified SWU](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-hash-to-curve-14#section-6.6.3).

A valid signature would be generated using the expected `signedData` and `domainSeparationTag`, as well the same hashing to curve process.

## BLS multi-signature

BLS signature scheme allows efficient multi-signature features. Multiple signatures can be aggregated
into a single signature which can be verified against an aggregated public key. This allows authenticating
multiple signers with a single signature verification.
While BLS provides multiple aggregation techniques,
Cadence supports basic aggregation tools that cover a wide list of use-cases.
These tools are defined in the built-in `BLS` contract, which does not need to be imported.

### Proof of Possession (PoP)

Multi-signature verification in BLS requires a defense against rogue public-key attacks. Multiple ways are
available to protect BLS verification. Cadence provides the proof of possession of private key as a defense tool.
The proof of possession of private key is a BLS signature over the public key itself.
The PoP signature follows the same requirements of a BLS signature (detailed in [Signature verification](#Signature-verification)),
except it uses a special domain separation tag. The key expected to be used in KMAC128 is the UTF-8 encoding of `"BLS_POP_BLS12381G1_XOF:KMAC128_SSWU_RO_POP_"`.
The expected message to be signed by the PoP is the serialization of the BLS public key corresponding to the signing private key ([serialization details](#PublicKey)).
The PoP can only be verified using the `PublicKey` method `verifyPoP`.

### BLS signature aggregation

```cadence
fun aggregateSignatures(_ signatures: [[UInt8]]): [UInt8]?
```

Aggregates multiple BLS signatures into one.
Signatures could be generated from the same or distinct messages, they
could also be the aggregation of other signatures.
The order of the signatures in the slice does not matter since the aggregation is commutative.
There is no subgroup membership check performed on the input signatures.
If the array is empty or if decoding one of the signatures fails, the program aborts.

The output signature can be verified against an aggregated public key to authenticate multiple
signers at once. Since the `verify` method accepts a single data to verify against, it is only possible to
verfiy multiple signatures of the same message.

### BLS public key aggregation

```cadence
fun aggregatePublicKeys(_ publicKeys: [PublicKey]): PublicKey?
```

Aggregates multiple BLS public keys into one.

The order of the public keys in the slice does not matter since the aggregation is commutative.
The input keys are guaranteed to be in the correct subgroup since subgroup membership is checked
at the key creation time.
If the array is empty or any of the input keys is not a BLS key, the program aborts.
Note that the identity public key is a valid input to this function and it represents the
identity element of aggregation.

The output public key can be used to verify aggregated signatures to authenticate multiple
signers at once. Since the `verify` method accepts a single data to verify against, it is only possible to
verfiy multiple signatures of the same message.
The identity public key is a possible output of the function, though signature verifications
against identity result in `false`.

In order to prevent rogue key attacks when verifying aggregated signatures, it is important to verfiy the
PoP of each individual key involved in the aggregation process.

## Crypto Contract

The built-in contract `Crypto` can be used to perform cryptographic operations.
The contract can be imported using `import Crypto`.

### Key Lists

The crypto contract also allows creating key lists to be used for multi-signature verification.
For example, to verify two signatures with equal weights for some signed data:

```cadence
import Crypto

pub fun test main() {
    let keyList = Crypto.KeyList()

    let publicKeyA = PublicKey(
        publicKey:
            "db04940e18ec414664ccfd31d5d2d4ece3985acb8cb17a2025b2f1673427267968e52e2bbf3599059649d4b2cce98fdb8a3048e68abf5abe3e710129e90696ca".decodeHex(),
        signatureAlgorithm: SignatureAlgorithm.ECDSA_P256
    )
    keyList.add(
        publicKeyA,
        hashAlgorithm: HashAlgorithm.SHA3_256,
        weight: 0.5
    )

    let publicKeyB = PublicKey(
        publicKey:
            "df9609ee588dd4a6f7789df8d56f03f545d4516f0c99b200d73b9a3afafc14de5d21a4fc7a2a2015719dc95c9e756cfa44f2a445151aaf42479e7120d83df956".decodeHex(),
        signatureAlgorithm: SignatureAlgorithm.ECDSA_P256
    )
    keyList.add(
        publicKeyB,
        hashAlgorithm: HashAlgorithm.SHA3_256,
        weight: 0.5
    )

    let signatureSet = [
        Crypto.KeyListSignature(
            keyIndex: 0,
            signature:
                "8870a8cbe6f44932ba59e0d15a706214cc4ad2538deb12c0cf718d86f32c47765462a92ce2da15d4a29eb4e2b6fa05d08c7db5d5b2a2cd8c2cb98ded73da31f6".decodeHex()
        ),
        Crypto.KeyListSignature(
            keyIndex: 1,
            signature:
                "bbdc5591c3f937a730d4f6c0a6fde61a0a6ceaa531ccb367c3559335ab9734f4f2b9da8adbe371f1f7da913b5a3fdd96a871e04f078928ca89a83d841c72fadf".decodeHex()
        )
    ]

    // "foo", encoded as UTF-8, in hex representation
    let signedData = "666f6f".decodeHex()

    let isValid = keyList.verify(
        signatureSet: signatureSet,
        signedData: signedData
    )
}
```


The API of the Crypto contract related to key lists is:

```cadence
pub struct KeyListEntry {
    pub let keyIndex: Int
    pub let publicKey: PublicKey
    pub let hashAlgorithm: HashAlgorithm
    pub let weight: UFix64
    pub let isRevoked: Bool

    init(
        keyIndex: Int,
        publicKey: PublicKey,
        hashAlgorithm: HashAlgorithm,
        weight: UFix64,
        isRevoked: Bool
    )
}

pub struct KeyList {

    init()

    /// Adds a new key with the given weight
    pub fun add(
        _ publicKey: PublicKey,
        hashAlgorithm: HashAlgorithm,
        weight: UFix64
    )

    /// Returns the key at the given index, if it exists.
    /// Revoked keys are always returned, but they have `isRevoked` field set to true
    pub fun get(keyIndex: Int): KeyListEntry?

    /// Marks the key at the given index revoked, but does not delete it
    pub fun revoke(keyIndex: Int)

    /// Returns true if the given signatures are valid for the given signed data
    pub fun verify(
        signatureSet: [KeyListSignature],
        signedData: [UInt8]
    ): Bool
}

pub struct KeyListSignature {
    pub let keyIndex: Int
    pub let signature: [UInt8]

    pub init(keyIndex: Int, signature: [UInt8])
}
```
---
title: Enumerations
sidebar_position: 15
---

Enumerations are sets of symbolic names bound to unique, constant values,
which can be compared by identity.

## Enum Declaration

Enums are declared using the `enum` keyword,
followed by the name of the enum, the raw type after a colon,
and the requirements, which must be enclosed in opening and closing braces.

The raw type must be an integer subtype, e.g. `UInt8` or `Int128`.

Enum cases are declared using the `case` keyword,
followed by the name of the enum case.

Enum cases must be unique.
Each enum case has a raw value, the index of the case in all cases.

The raw value of an enum case can be accessed through the `rawValue` field.

The enum cases can be accessed by using the name as a field on the enum,
or by using the enum constructor,
which requires providing the raw value as an argument.
The enum constructor returns the enum case with the given raw value,
if any, or `nil` if no such case exists.

Enum cases can be compared using the equality operators `==` and `!=`.

```cadence
// Declare an enum named `Color` which has the raw value type `UInt8`,
// and declare three enum cases: `red`, `green`, and `blue`
//
pub enum Color: UInt8 {
    pub case red
    pub case green
    pub case blue
}
// Declare a variable which has the enum type `Color` and initialize
// it to the enum case `blue` of the enum
let blue: Color = Color.blue
// Get the raw value of the enum case `blue`.
// As it is the third case, so it has index 2
//
blue.rawValue // is `2`
// Get the `green` enum case of the enum `Color` by using the enum
// constructor and providing the raw value of the enum case `green`, 1,
// as the enum case `green` is the second case, so it has index 1
//
let green: Color? = Color(rawValue: 1)  // is `Color.green`
// Get the enum case of the enum `Color` that has the raw value 5.
// As there are only three cases, the maximum raw value / index is 2.
//
let nothing = Color(rawValue: 5)  // is `nil`
// Enum cases can be compared
Color.red == Color.red  // is `true`
Color(rawValue: 1) == Color.green  // is `true`
// Different enum cases are not the same
Color.red != Color.blue  // is `true`
```
---
title: Environment Information
sidebar_position: 29
---

## Transaction Information

To get the addresses of the signers of a transaction,
use the `address` field of each signing `AuthAccount`
that is passed to the transaction's `prepare` phase.

There is currently no API that allows getting other transaction information.
Please let us know if your use-case demands it by request this feature in an issue.

## Block Information

To get information about a block, the functions `getCurrentBlock` and `getBlock` can be used:

-
    ```cadence
    fun getCurrentBlock(): Block
    ```

  Returns the current block, i.e. the block which contains the currently executed transaction.

-
    ```cadence
    fun getBlock(at: UInt64): Block?
    ```

  Returns the block at the given height.
  If the given block does not exist the function returns `nil`.

The `Block` type contains the identifier, height, and timestamp:

```cadence
pub struct Block {
    /// The ID of the block.
    ///
    /// It is essentially the hash of the block.
    ///
    pub let id: [UInt8; 32]

    /// The height of the block.
    ///
    /// If the blockchain is viewed as a tree with the genesis block at the root,
    // the height of a node is the number of edges between the node and the genesis block
    ///
    pub let height: UInt64

    /// The view of the block.
    ///
    /// It is a detail of the consensus algorithm. It is a monotonically increasing integer
    /// and counts rounds in the consensus algorithm. It is reset to zero at each spork.
    ///
    pub let view: UInt64

    /// The timestamp of the block.
    ///
    /// Unix timestamp of when the proposer claims it constructed the block.
    ///
    /// NOTE: It is included by the proposer, there are no guarantees on how much the time stamp can deviate from the true time the block was published.
    /// Consider observing blocks’ status changes off-chain yourself to get a more reliable value.
    ///
    pub let timestamp: UFix64
}
```

---
title: Events
sidebar_position: 24
---

Events are special values that can be emitted during the execution of a program.

An event type can be declared with the `event` keyword.

```cadence
event FooEvent(x: Int, y: Int)
```

The syntax of an event declaration is similar to that of
a [function declaration](./functions.mdx#function-declarations);
events contain named parameters, each of which has an optional argument label.

Event parameters may only have a valid event parameter type.
Valid types are boolean, string, integer, arrays and dictionaries of these types,
and structures where all fields have a valid event parameter type.
Resource types are not allowed, because when a resource is used as an argument, it is moved.

Events can only be declared within a [contract](./contracts.mdx) body.
Events cannot be declared globally or within resource or struct types.

```cadence
// Invalid: An event cannot be declared globally
//
event GlobalEvent(field: Int)

pub contract Events {
    // Event with explicit argument labels
    //
    event BarEvent(labelA fieldA: Int, labelB fieldB: Int)

    // Invalid: A resource type is not allowed to be used
    // because it would be moved and lost
    //
    event ResourceEvent(resourceField: @Vault)
}

```

### Emitting events

To emit an event from a program, use the `emit` statement:

```cadence
pub contract Events {
    event FooEvent(x: Int, y: Int)

    // Event with argument labels
    event BarEvent(labelA fieldA: Int, labelB fieldB: Int)

    fun events() {
        emit FooEvent(x: 1, y: 2)

        // Emit event with explicit argument labels
        // Note that the emitted event will only contain the field names,
        // not the argument labels used at the invocation site.
        emit BarEvent(labelA: 1, labelB: 2)
    }
}
```

Emitting events has the following restrictions:

- Events can only be invoked in an `emit` statement.

  This means events cannot be assigned to variables or used as function parameters.

- Events can only be emitted from the location in which they are declared.
---
title: Functions
sidebar_position: 6
---

Functions are sequences of statements that perform a specific task.
Functions have parameters (inputs) and an optional return value (output).
Functions are typed: the function type consists of the parameter types and the return type.

Functions are values, i.e., they can be assigned to constants and variables,
and can be passed as arguments to other functions.
This behavior is often called "first-class functions".

## Function Declarations

Functions can be declared by using the `fun` keyword, followed by the name of the declaration,
 the parameters, the optional return type,
 and the code that should be executed when the function is called.

The parameters need to be enclosed in parentheses.
The return type, if any, is separated from the parameters by a colon (`:`).
The function code needs to be enclosed in opening and closing braces.

Each parameter must have a name, which is the name that the argument value
will be available as within the function.

An additional argument label can be provided to require function calls to use the label
to provide an argument value for the parameter.

Argument labels make code more explicit and readable.
For example, they avoid confusion about the order of arguments
when there are multiple arguments that have the same type.

Argument labels should be named so they make sense from the perspective of the function call.

Argument labels precede the parameter name.
The special argument label `_` indicates
that a function call can omit the argument label.
If no argument label is declared in the function declaration,
the parameter name is the argument label of the function declaration,
and function calls must use the parameter name as the argument label.

Each parameter needs to have a type annotation,
which follows the parameter name after a colon.

Function calls may provide arguments for parameters
which are subtypes of the parameter types.

There is **no** support for optional parameters,
i.e. default values for parameters,
and variadic functions,
i.e. functions that take an arbitrary amount of arguments.

```cadence
// Declare a function named `double`, which multiples a number by two.
//
// The special argument label _ is specified for the parameter,
// so no argument label has to be provided in a function call.
//
fun double(_ x: Int): Int {
    return x * 2
}

// Call the function named `double` with the value 4 for the first parameter.
//
// The argument label can be omitted in the function call as the declaration
// specifies the special argument label _ for the parameter.
//
double(2)  // is `4`
```

It is possible to require argument labels for some parameters,
and not require argument labels for other parameters.

```cadence
// Declare a function named `clamp`. The function takes an integer value,
// the lower limit, and the upper limit. It returns an integer between
// the lower and upper limit.
//
// For the first parameter the special argument label _ is used,
// so no argument label has to be given for it in a function call.
//
// For the second and third parameter no argument label is given,
// so the parameter names are the argument labels, i.e., the parameter names
// have to be given as argument labels in a function call.
//
fun clamp(_ value: Int, min: Int, max: Int): Int {
    if value > max {
        return max
    }

    if value < min {
        return min
    }

    return value
}

// Declare a constant which has the result of a call to the function
// named `clamp` as its initial value.
//
// For the first argument no label is given, as it is not required by
// the function declaration (the special argument label `_` is specified).
//
// For the second and this argument the labels must be provided,
// as the function declaration does not specify the special argument label `_`
// for these two parameters.
//
// As the function declaration also does not specify argument labels
// for these parameters, the parameter names must be used as argument labels.
//
let clamped = clamp(123, min: 0, max: 100)
// `clamped` is `100`
```

```cadence
// Declare a function named `send`, which transfers an amount
// from one account to another.
//
// The implementation is omitted for brevity.
//
// The first two parameters of the function have the same type, so there is
// a potential that a function call accidentally provides arguments in
// the wrong order.
//
// While the parameter names `senderAddress` and `receiverAddress`
// are descriptive inside the function, they might be too verbose
// to require them as argument labels in function calls.
//
// For this reason the shorter argument labels `from` and `to` are specified,
// which still convey the meaning of the two parameters without being overly
// verbose.
//
// The name of the third parameter, `amount`, is both meaningful inside
// the function and also in a function call, so no argument label is given,
// and the parameter name is required as the argument label in a function call.
//
fun send(from senderAddress: Address, to receivingAddress: Address, amount: Int) {
    // The function code is omitted for brevity.
    // ...
}

// Declare a constant which refers to the sending account's address.
//
// The initial value is omitted for brevity.
//
let sender: Address = // ...

// Declare a constant which refers to the receiving account's address.
//
// The initial value is omitted for brevity.
//
let receiver: Address = // ...

// Call the function named `send`.
//
// The function declaration requires argument labels for all parameters,
// so they need to be provided in the function call.
//
// This avoids ambiguity. For example, in some languages (like C) it is
// a convention to order the parameters so that the receiver occurs first,
// followed by the sender. In other languages, it is common to have
// the sender be the first parameter, followed by the receiver.
//
// Here, the order is clear – send an amount from an account to another account.
//
send(from: sender, to: receiver, amount: 100)
```

The order of the arguments in a function call must
match the order of the parameters in the function declaration.

```cadence
// Declare a function named `test`, which accepts two parameters, named `first` and `second`
//
fun test(first: Int, second: Int) {
    // ...
}

// Invalid: the arguments are provided in the wrong order,
// even though the argument labels are provided correctly.
//
test(second: 1, first: 2)
```

Functions can be nested,
i.e., the code of a function may declare further functions.

```cadence
// Declare a function which multiplies a number by two, and adds one.
//
fun doubleAndAddOne(_ x: Int): Int {

    // Declare a nested function which multiplies a number by two.
    //
    fun double(_ x: Int) {
        return x * 2
    }

    return double(x) + 1
}

doubleAndAddOne(2)  // is `5`
```

Functions do not support overloading.

## Function Expressions

Functions can be also used as expressions.
The syntax is the same as for function declarations,
except that function expressions have no name, i.e., they are anonymous.

```cadence
// Declare a constant named `double`, which has a function as its value.
//
// The function multiplies a number by two when it is called.
//
// This function's type is `((Int): Int)`.
//
let double =
    fun (_ x: Int): Int {
        return x * 2
    }
```

## Function Calls

Functions can be called (invoked). Function calls
need to provide exactly as many argument values as the function has parameters.

```cadence
fun double(_ x: Int): Int {
    return x * 2
}

// Valid: the correct amount of arguments is provided.
//
double(2)  // is `4`

// Invalid: too many arguments are provided.
//
double(2, 3)

// Invalid: too few arguments are provided.
//
double()
```

## Function Types

Function types consist of the function's parameter types
and the function's return type.

The parameter types need to be enclosed in parentheses,
followed by a colon (`:`), and end with the return type.
The whole function type needs to be enclosed in parentheses.

```cadence
// Declare a function named `add`, with the function type `((Int, Int): Int)`.
//
fun add(a: Int, b: Int): Int {
    return a + b
}
```

```cadence
// Declare a constant named `add`, with the function type `((Int, Int): Int)`
//
let add: ((Int, Int): Int) =
    fun (a: Int, b: Int): Int {
        return a + b
    }
```

If the function has no return type, it implicitly has the return type `Void`.

```cadence
// Declare a constant named `doNothing`, which is a function
// that takes no parameters and returns nothing.
//
let doNothing: ((): Void) =
    fun () {}
```

Parentheses also control precedence.
For example, a function type `((Int): ((): Int))` is the type
for a function which accepts one argument with type `Int`,
and which returns another function,
that takes no arguments and returns an `Int`.

The type `[((Int): Int); 2]` specifies an array type of two functions,
which accept one integer and return one integer.

Argument labels are not part of the function type.
This has the advantage that functions with different argument labels,
potentially written by different authors are compatible
as long as the parameter types and the return type match.
It has the disadvantage that function calls to plain function values,
cannot accept argument labels.

```cadence
// Declare a function which takes one argument that has type `Int`.
// The function has type `((Int): Void)`.
//
fun foo1(x: Int) {}

// Call function `foo1`. This requires an argument label.
foo1(x: 1)

// Declare another function which takes one argument that has type `Int`.
// The function also has type `((Int): Void)`.
//
fun foo2(y: Int) {}

// Call function `foo2`. This requires an argument label.
foo2(y: 2)

// Declare a variable which has type `((Int): Void)` and use `foo1`
// as its initial value.
//
var someFoo: ((Int): Void) = foo1

// Call the function assigned to variable `someFoo`.
// This is valid as the function types match.
// This does neither require nor allow argument labels.
//
someFoo(3)

// Assign function `foo2` to variable `someFoo`.
// This is valid as the function types match.
//
someFoo = foo2

// Call the function assigned to variable `someFoo`.
// This does neither require nor allow argument labels.
//
someFoo(4)
```

## Closures

A function may refer to variables and constants of its outer scopes
in which it is defined.
It is called a closure, because
it is closing over those variables and constants.
A closure can read from the variables and constants
and assign to the variables it refers to.

```cadence
// Declare a function named `makeCounter` which returns a function that
// each time when called, returns the next integer, starting at 1.
//
fun makeCounter(): ((): Int) {
    var count = 0
    return fun (): Int {
        // NOTE: read from and assign to the non-local variable
        // `count`, which is declared in the outer function.
        //
        count = count + 1
        return count
    }
}

let test = makeCounter()
test()  // is `1`
test()  // is `2`
```

## Argument Passing Behavior

When arguments are passed to a function, they are copied.
Therefore, values that are passed into a function
are unchanged in the caller's scope when the function returns.
This behavior is known as
[call-by-value](https://en.wikipedia.org/w/index.php?title=Evaluation_strategy&oldid=896280571#Call_by_value).

```cadence
// Declare a function that changes the first two elements
// of an array of integers.
//
fun change(_ numbers: [Int]) {
    // Change the elements of the passed in array.
    // The changes are only local, as the array was copied.
    //
    numbers[0] = 1
    numbers[1] = 2
    // `numbers` is `[1, 2]`
}

let numbers = [0, 1]

change(numbers)
// `numbers` is still `[0, 1]`
```

Parameters are constant, i.e., it is not allowed to assign to them.

```cadence
fun test(x: Int) {
    // Invalid: cannot assign to a parameter (constant)
    //
    x = 2
}
```

## Function Preconditions and Postconditions

Functions may have preconditions and may have postconditions.
Preconditions and postconditions can be used to restrict the inputs (values for parameters)
and output (return value) of a function.

Preconditions must be true right before the execution of the function.
Preconditions are part of the function and introduced by the `pre` keyword,
followed by the condition block.

Postconditions must be true right after the execution of the function.
Postconditions are part of the function and introduced by the `post` keyword,
followed by the condition block.
Postconditions may only occur after preconditions, if any.

A conditions block consists of one or more conditions.
Conditions are expressions evaluating to a boolean.

Conditions may be written on separate lines,
or multiple conditions can be written on the same line,
separated by a semicolon.
This syntax follows the syntax for [statements](./syntax.md#semicolons).

Following each condition, an optional description can be provided after a colon.
The condition description is used as an error message when the condition fails.

In postconditions, the special constant `result` refers to the result of the function.

```cadence
fun factorial(_ n: Int): Int {
    pre {
        // Require the parameter `n` to be greater than or equal to zero.
        //
        n >= 0:
            "factorial is only defined for integers greater than or equal to zero"
    }
    post {
        // Ensure the result will be greater than or equal to 1.
        //
        result >= 1:
            "the result must be greater than or equal to 1"
    }

    if n < 1 {
       return 1
    }

    return n * factorial(n - 1)
}

factorial(5)  // is `120`

// Run-time error: The given argument does not satisfy
// the precondition `n >= 0` of the function, the program aborts.
//
factorial(-2)
```

In postconditions, the special function `before` can be used
to get the value of an expression just before the function is called.

```cadence
var n = 0

fun incrementN() {
    post {
        // Require the new value of `n` to be the old value of `n`, plus one.
        //
        n == before(n) + 1:
            "n must be incremented by 1"
    }

    n = n + 1
}
```

## Functions are Values

Functions are values ("first-class"), so they may be assigned to variables and fields
or passed to functions as arguments.


```cadence
// Declare a function named `transform` which applies a function to each element
// of an array of integers and returns a new array of the results.
//
pub fun transform(function: ((Int): Int), integers: [Int]): [Int] {
    var newIntegers: [Int] = []
    for integer in integers {
        newIntegers.append(function(integer))
    }
    return newIntegers
}

pub fun double(_ integer: Int): Int {
    return integer * 2
}

let newIntegers = transform(function: double, integers: [1, 2, 3])
// `newIntegers` is `[2, 4, 6]`
```

---
title: Glossary
sidebar_position: 32
---


<Callout type="info">
Tip: <kbd>CTRL</kbd>/<kbd>⌘</kbd> + <kbd>F</kbd> and type in the symbol or operator you want to look up.
</Callout>

## `&` (ampersand)

The `&` (ampersand) symbol has several uses.

### Reference

If an expression starts with the `&` (ampersand) symbol, it creates a [reference](./references.md).

```cadence
let a: String = "hello"
let refOfA: &String = &a as &String
```

References may also be authorized if the `&` symbol is preceded by `auth` (otherwise the reference is unauthorized).

Authorized references have the `auth` modifier, i.e. the full syntax is `auth &T`,
whereas unauthorized references do not have a modifier.

```cadence
let a: String = "hello"
let refOfA: &String = &a as auth &String
```

### Logical Operator

It can be also used as a [logical operator (AND)](./operators.md#logical-operators),
by appearing twice in succession (i.e. `&&`):

```cadence
let a = true
let b = false

let c = a && b // false
```

## `@` (at)

The `@` (at) symbol before a type is used to annotate whether the type is a [resource](./resources.mdx).

The `@` symbol must appear at the beginning of the type, not inside.
For example, an array of `NFT`s is `@[NFT]`, not `[@NFT]`.
This emphasizes the whole type acts like a resource.

```cadence
// Declare a resource named `SomeResource`
pub resource SomeResource {
    pub var value: Int

    init(value: Int) {
        self.value = value
    }
}

// we use the '@' symbol to reference a resource type
let a: @SomeResource <- create SomeResource(value: 0)

// also in functions declarations
pub fun use(resource: @SomeResource) {
    destroy resource
}
```

## `:` (colon)

The `:` (colon) symbol has several uses.

### Type Declaration

If a `:` (colon) follows a variable/constant/function declaration, it is used to declare its type.

```cadence
let a: Bool = true // declares variable `a` with type `Bool`

// or

fun addOne(x: Int): Int { // return type of Int
    return x + 1
}
```

### Ternary Conditional Operator

The `:` (colon) is also be used in [ternary operations](./operators.md#ternary-conditional-operator) to represent the "otherwise" section,
such as the following:

```cadence
let a = 1 > 2 ? 3 : 4
// should be read as:
//   "is 1 greater than 2?"
//   "if YES, then set a = 3,
//   "otherwise, set a = 4.
```

## `=` (equals)

The `=` (equals) symbol has several uses.

### Variable Declaration

```cadence
let a = 1 // declares a variable `a` with value `1`
```

### Assignment

```cadence
a = 1  // assigns the value `1` to variable `a `
```

## `!` (exclamation mark)

The `!` (exclamation mark) symbol has a different effect whether it precedes or succeeds a variable.

When it immediately **precedes** a boolean-type variable, it negates it.

```cadence
let a: Bool = true
let b: Bool = !a

// b is false
```

When it immediately **succeeds** an *optional* variable, it [force-unwraps](./operators.md#force-unwrap-operator-) it.
Force-unwrapping returns the value inside an optional if it contains a value,
or panics and aborts the execution if the optional has no value, i.e. the optional value is nil.

```cadence
let a: Int? = nil
let b: Int? = 3

let c: Int = a! // panics, because = nil
let d: Int = b! // initialized correctly as 3
```

## `/` (forward slash)

The `/` (forward slash) symbol has several uses.

### Division Operator

Inbetween two expressions, the forward slash acts as the [division operator](./operators.md#arithmetic-operators).

```cadence
let result = 4 / 2
```

### Path separator

In a [Path](./accounts.mdx#paths), the forward slash separates the domain (e.g. `storage`, `private`, `public`) and the identifier.

```cadence
let storagePath = /storage/path
storagePath.toString()  // is "/storage/path"
```

## `<-` (lower than, hyphen) (Move operator)

The [move operator `<-`](./resources.mdx#the-move-operator--) is like the assignment operator `=`,
but must be used when the value is a [resource](./resources.mdx).
To make assignment of resources explicit, the move operator `<-` must be used when:

- The resource is the initial value of a constant or variable,
- The resource is moved to a different variable in an assignment,
- The resource is moved to a function as an argument
- The resource is returned from a function.

```cadence
resource R {}

let a <- create R() // we instantiate a new resource and move it into a
```

## `<-!` (lower than, hyphen, exclamation mark) (Force-assignment move operator)

The [force-assignment move operator `<-!`](./operators.md#force-assignment-operator--) moves a resource value to an optional variable.
If the variable is `nil`, the move succeeds.
If it is not nil, the program aborts.

```cadence
pub resource R {}

var a: @R? <- nil
a <-! create R()
```

## `<->` (lower than, hyphen, greater than) (Swap operator)

The [swapping operator `<->`](./operators.md#swapping-operator--) swaps two resource between the variables to the left and right of it.


## `+` (plus), `-` (minus), `*` (asterisk), `%` (percentage sign)

These are all typical [arithmetic operators](./operators.md#arithmetic-operators):

- Addition: `+`
- Subtraction: `-`
- Multiplication: `*`
- Remainder: `%`

## `?` (question mark)

The `?` (question mark) symbol has several uses.

### Optional

If a `?` (question mark) follows a variable/constant, it represents an optional.
An optional can either have a value or *nothing at all*.

```cadence
// Declare a constant which has an optional integer type
//
let a: Int? = nil
```

### Ternary Conditional Operator

The `?` (question mark) is also be used in [ternary operations](./operators.md#ternary-conditional-operator) to represent the "then" section,
such as the following:

```cadence
let a = 1 > 2 ? 3 : 4
// should be read as:
//   "is 1 greater than 2?"
//   "if YES, then set a = 3,
//   "otherwise, set a = 4.
```

### Nil-Coalescing Operator

The `?` (question mark) is also used in the [nil-coalescing operator `??`](./operators.md#nil-coalescing-operator-).

It returns the value inside the optional, if the optional contains a value,
or returns an alternative value if the optional has no value, i.e., the optional value is nil.

```cadence
// Declare a constant which has an optional integer type
//
let a: Int? = nil

// Declare a constant with a non-optional integer type,
// which is initialized to `a` if it is non-nil, or 42 otherwise.
//
let b: Int = a ?? 42
// `b` is 42, as `a` is nil


// Invalid: nil-coalescing operator is applied to a value which has a non-optional type
// (the integer literal is of type `Int`).
//
let c = 1 ?? 2
```

## `_` (underscore)

The `_` (underscore) symbol has several uses.

### Names

The `_` (underscore) can be used in names, e.g. in variables and types.

```cadence
let _a = true // used as a variable name
let another_one = false
```

### Number Literals

The `_` (underscore) can also be used to split up numerical components.

```cadence
let b = 100_000_000 // used to split up a number (supports all number types, e.g. 0b10_11_01)
```

### Argument Labels

The `_` (underscore) can also be to indicate that a parameter in a [function](./functions.mdx) has no argument label.

```cadence
// The special argument label _ is specified for the parameter,
// so no argument label has to be provided in a function call.

fun double(_ x: Int): Int {
    return x * 2
}

let result = double(4)
```
---
title: Imports
sidebar_position: 18
---

Programs can import declarations (types, functions, variables, etc.) from other programs.

Imports are declared using the `import` keyword.

It can either be followed by a location, which imports all declarations;
or it can be followed by the names of the declarations that should be imported,
followed by the `from` keyword, and then followed by the location.

If importing a local file, the location is a string literal, and the path to the file.
Deployment of code with file imports requires the usage for the [Flow CLI](https://developers.flow.com/tools/flow-cli/).

If importing an external type in a different account,
the location is an address literal, and the address
of the account where the declarations are deployed to and published.

```cadence
// Import the type `Counter` from a local file.
//
import Counter from "./examples/counter.cdc"

// Import the type `Counter` from an external account.
//
import Counter from 0x299F20A29311B9248F12
```
---
title: The Cadence Programming Language
sidebar_label: Language Reference
---

## Introduction

The Cadence Programming Language is a new high-level programming language
intended for smart contract development.

The language's goals are, in order of importance:

- **Safety and security**:
  Provide a strong static type system, design by contract (preconditions and postconditions),
  and resources (inspired by linear types).

- **Auditability**:
  Focus on readability: Make it easy to verify what the code is doing,
  and make intentions explicit, at a small cost of verbosity.

- **Simplicity**: Focus on developer productivity and usability:
  Make it easy to write code, provide good tooling.

## Terminology

In this document, the following terminology is used to describe syntax
or behavior that is not allowed in the language:

- `Invalid` means that the invalid program will not even be allowed to run.
  The program error is detected and reported statically by the type checker.

- `Run-time error` means that the erroneous program will run,
  but bad behavior will result in the execution of the program being aborted.

## Syntax and Behavior

Much of the language's syntax is inspired by Swift, Kotlin, and TypeScript.

Much of the syntax, types, and standard library is inspired by Swift,
which popularized e.g. optionals, argument labels,
and provides safe handling of integers and strings.

Resources are based on linear types which were popularized by Rust.

Events are inspired by Solidity.

**Disclaimer:** In real Cadence code, all type definitions and code
must be declared and contained in [contracts](./contracts.mdx) or [transactions](./transactions.md),
but we omit these containers in examples for simplicity.
---
title: Interfaces
sidebar_position: 14
---

An interface is an abstract type that specifies the behavior of types
that *implement* the interface.
Interfaces declare the required functions and fields,
the access control for those declarations,
and preconditions and postconditions that implementing types need to provide.

There are three kinds of interfaces:

- **Structure interfaces**: implemented by [structures](./composite-types.mdx#structures)
- **Resource interfaces**: implemented by [resources](./composite-types.mdx#resources)
- **Contract interfaces**: implemented by [contracts](./contracts.mdx)

Structure, resource, and contract types may implement multiple interfaces.

There is no support for event and enum interfaces.

Nominal typing applies to composite types that implement interfaces.
This means that a type only implements an interface
if it has explicitly declared the conformance,
the composite type does not implicitly conform to an interface,
even if it satisfies all requirements of the interface.

Interfaces consist of the function and field requirements
that a type implementing the interface must provide implementations for.
Interface requirements, and therefore also their implementations,
must always be at least public.

Variable field requirements may be annotated
to require them to be publicly settable.

Function requirements consist of the name of the function,
parameter types, an optional return type,
and optional preconditions and postconditions.

Field requirements consist of the name and the type of the field.
Field requirements may optionally declare a getter requirement and a setter requirement,
each with preconditions and postconditions.

Calling functions with preconditions and postconditions on interfaces
instead of concrete implementations can improve the security of a program,
as it ensures that even if implementations change,
some aspects of them will always hold.

## Interface Declaration

Interfaces are declared using the `struct`, `resource`, or `contract` keyword,
followed by the `interface` keyword,
the name of the interface,
and the requirements, which must be enclosed in opening and closing braces.

Field requirements can be annotated to
require the implementation to be a variable field, by using the `var` keyword;
require the implementation to be a constant field, by using the `let` keyword;
or the field requirement may specify nothing,
in which case the implementation may either be a variable or a constant field.

Field requirements and function requirements must specify the required level of access.
The access must be at least be public, so the `pub` keyword must be provided.
Variable field requirements can be specified to also be publicly settable
by using the `pub(set)` keyword.

Interfaces can be used in types.
This is explained in detail in the section [Interfaces in Types](#interfaces-in-types).
For now, the syntax `{I}` can be read as the type of any value that implements the interface `I`.

```cadence
// Declare a resource interface for a fungible token.
// Only resources can implement this resource interface.
//
pub resource interface FungibleToken {

    // Require the implementing type to provide a field for the balance
    // that is readable in all scopes (`pub`).
    //
    // Neither the `var` keyword, nor the `let` keyword is used,
    // so the field may be implemented as either a variable
    // or as a constant field.
    //
    pub balance: Int

    // Require the implementing type to provide an initializer that
    // given the initial balance, must initialize the balance field.
    //
    init(balance: Int) {
        pre {
            balance >= 0:
                "Balances are always non-negative"
        }
        post {
            self.balance == balance:
                "the balance must be initialized to the initial balance"
        }

        // NOTE: The declaration contains no implementation code.
    }

    // Require the implementing type to provide a function that is
    // callable in all scopes, which withdraws an amount from
    // this fungible token and returns the withdrawn amount as
    // a new fungible token.
    //
    // The given amount must be positive and the function implementation
    // must add the amount to the balance.
    //
    // The function must return a new fungible token.
    // The type `{FungibleToken}` is the type of any resource
    // that implements the resource interface `FungibleToken`.
    //
    pub fun withdraw(amount: Int): @{FungibleToken} {
        pre {
            amount > 0:
                "the amount must be positive"
            amount <= self.balance:
                "insufficient funds: the amount must be smaller or equal to the balance"
        }
        post {
            self.balance == before(self.balance) - amount:
                "the amount must be deducted from the balance"
        }

        // NOTE: The declaration contains no implementation code.
    }

    // Require the implementing type to provide a function that is
    // callable in all scopes, which deposits a fungible token
    // into this fungible token.
    //
    // No precondition is required to check the given token's balance
    // is positive, as this condition is already ensured by
    // the field requirement.
    //
    // The parameter type `{FungibleToken}` is the type of any resource
    // that implements the resource interface `FungibleToken`.
    //
    pub fun deposit(_ token: @{FungibleToken}) {
        post {
            self.balance == before(self.balance) + token.balance:
                "the amount must be added to the balance"
        }

        // NOTE: The declaration contains no implementation code.
    }
}
```

Note that the required initializer and functions do not have any executable code.

Struct and resource Interfaces can only be declared directly inside contracts,
i.e. not inside of functions.
Contract interfaces can only be declared globally and not inside contracts.

## Interface Implementation

Declaring that a type implements (conforms) to an interface
is done in the type declaration of the composite type (e.g., structure, resource):
The kind and the name of the composite type is followed by a colon (`:`)
and the name of one or more interfaces that the composite type implements.

This will tell the checker to enforce any requirements from the specified interfaces
onto the declared type.

A type implements (conforms to) an interface if it declares
the implementation in its signature, provides field declarations
for all fields required by the interface,
and provides implementations for all functions required by the interface.

The field declarations in the implementing type must match the field requirements
in the interface in terms of name, type, and declaration kind (e.g. constant, variable)
if given. For example, an interface may require a field with a certain name and type,
but leaves it to the implementation what kind the field is.

The function implementations must match the function requirements in the interface
in terms of name, parameter argument labels, parameter types, and the return type.

```cadence
// Declare a resource named `ExampleToken` that has to implement
// the `FungibleToken` interface.
//
// It has a variable field named `balance`, that can be written
// by functions of the type, but outer scopes can only read it.
//
pub resource ExampleToken: FungibleToken {

    // Implement the required field `balance` for the `FungibleToken` interface.
    // The interface does not specify if the field must be variable, constant,
    // so in order for this type (`ExampleToken`) to be able to write to the field,
    // but limit outer scopes to only read from the field, it is declared variable,
    // and only has public access (non-settable).
    //
    pub var balance: Int

    // Implement the required initializer for the `FungibleToken` interface:
    // accept an initial balance and initialize the `balance` field.
    //
    // This implementation satisfies the required postcondition.
    //
    // NOTE: the postcondition declared in the interface
    // does not have to be repeated here in the implementation.
    //
    init(balance: Int) {
        self.balance = balance
    }

    // Implement the required function named `withdraw` of the interface
    // `FungibleToken`, that withdraws an amount from the token's balance.
    //
    // The function must be public.
    //
    // This implementation satisfies the required postcondition.
    //
    // NOTE: neither the precondition nor the postcondition declared
    // in the interface have to be repeated here in the implementation.
    //
    pub fun withdraw(amount: Int): @ExampleToken {
        self.balance = self.balance - amount
        return create ExampleToken(balance: amount)
    }

    // Implement the required function named `deposit` of the interface
    // `FungibleToken`, that deposits the amount from the given token
    // to this token.
    //
    // The function must be public.
    //
    // NOTE: the type of the parameter is `{FungibleToken}`,
    // i.e., any resource that implements the resource interface `FungibleToken`,
    // so any other token – however, we want to ensure that only tokens
    // of the same type can be deposited.
    //
    // This implementation satisfies the required postconditions.
    //
    // NOTE: neither the precondition nor the postcondition declared
    // in the interface have to be repeated here in the implementation.
    //
    pub fun deposit(_ token: @{FungibleToken}) {
        if let exampleToken <- token as? ExampleToken {
            self.balance = self.balance + exampleToken.balance
            destroy exampleToken
        } else {
            panic("cannot deposit token which is not an example token")
        }
    }
}

// Declare a constant which has type `ExampleToken`,
// and is initialized with such an example token.
//
let token <- create ExampleToken(balance: 100)

// Withdraw 10 units from the token.
//
// The amount satisfies the precondition of the `withdraw` function
// in the `FungibleToken` interface.
//
// Invoking a function of a resource does not destroy the resource,
// so the resource `token` is still valid after the call of `withdraw`.
//
let withdrawn <- token.withdraw(amount: 10)

// The postcondition of the `withdraw` function in the `FungibleToken`
// interface ensured the balance field of the token was updated properly.
//
// `token.balance` is `90`
// `withdrawn.balance` is `10`

// Deposit the withdrawn token into another one.
let receiver: @ExampleToken <- // ...
receiver.deposit(<-withdrawn)

// Run-time error: The precondition of function `withdraw` in interface
// `FungibleToken` fails, the program aborts: the parameter `amount`
// is larger than the field `balance` (100 > 90).
//
token.withdraw(amount: 100)

// Withdrawing tokens so that the balance is zero does not destroy the resource.
// The resource has to be destroyed explicitly.
//
token.withdraw(amount: 90)
```

The access level for variable fields in an implementation
may be less restrictive than the interface requires.
For example, an interface may require a field to be
at least public (i.e. the `pub` keyword is specified),
and an implementation may provide a variable field which is public,
but also publicly settable (the `pub(set)` keyword is specified).

```cadence
pub struct interface AnInterface {
    // Require the implementing type to provide a publicly readable
    // field named `a` that has type `Int`. It may be a variable
    // or a constant field.
    //
    pub a: Int
}

pub struct AnImplementation: AnInterface {
    // Declare a publicly settable variable field named `a` that has type `Int`.
    // This implementation satisfies the requirement for interface `AnInterface`:
    // The field is at least publicly readable, but this implementation also
    // allows the field to be written to in all scopes.
    //
    pub(set) var a: Int

    init(a: Int) {
        self.a = a
    }
}
```

## Interfaces in Types

Interfaces can be used in types: The type `{I}` is the type of all objects
that implement the interface `I`.

This is called a [restricted type](./restricted-types.md):
Only the functionality (members and functions) of the interface can be used
when accessing a value of such a type.

```cadence
// Declare an interface named `Shape`.
//
// Require implementing types to provide a field which returns the area,
// and a function which scales the shape by a given factor.
//
pub struct interface Shape {
    pub fun getArea(): Int
    pub fun scale(factor: Int)
}

// Declare a structure named `Square` the implements the `Shape` interface.
//
pub struct Square: Shape {
    // In addition to the required fields from the interface,
    // the type can also declare additional fields.
    //
    pub var length: Int

    // Provided the field `area`  which is required to conform
    // to the interface `Shape`.
    //
    // Since `area` was not declared as a constant, variable,
    // field in the interface, it can be declared.
    //
    pub fun getArea(): Int {
        return self.length * self.length
    }

    pub init(length: Int) {
        self.length = length
    }

    // Provided the implementation of the function `scale`
    // which is required to conform to the interface `Shape`.
    //
    pub fun scale(factor: Int) {
        self.length = self.length * factor
    }
}

// Declare a structure named `Rectangle` that also implements the `Shape` interface.
//
pub struct Rectangle: Shape {
    pub var width: Int
    pub var height: Int

    // Provided the field `area  which is required to conform
    // to the interface `Shape`.
    //
    pub fun getArea(): Int {
        return self.width * self.height
    }

    pub init(width: Int, height: Int) {
        self.width = width
        self.height = height
    }

    // Provided the implementation of the function `scale`
    // which is required to conform to the interface `Shape`.
    //
    pub fun scale(factor: Int) {
        self.width = self.width * factor
        self.height = self.height * factor
    }
}

// Declare a constant that has type `Shape`, which has a value that has type `Rectangle`.
//
var shape: {Shape} = Rectangle(width: 10, height: 20)
```

Values implementing an interface are assignable to variables that have the interface as their type.

```cadence
// Assign a value of type `Square` to the variable `shape` that has type `Shape`.
//
shape = Square(length: 30)

// Invalid: cannot initialize a constant that has type `Rectangle`.
// with a value that has type `Square`.
//
let rectangle: Rectangle = Square(length: 10)
```

Fields declared in an interface can be accessed
and functions declared in an interface
can be called on values of a type that implements the interface.

```cadence
// Declare a constant which has the type `Shape`.
// and is initialized with a value that has type `Rectangle`.
//
let shape: {Shape} = Rectangle(width: 2, height: 3)

// Access the field `area` declared in the interface `Shape`.
//
shape.area  // is `6`

// Call the function `scale` declared in the interface `Shape`.
//
shape.scale(factor: 3)

shape.area  // is `54`
```

## Interface Nesting

<Callout type="info">

🚧 Status: Currently only contracts and contract interfaces support nested interfaces.

</Callout>

Interfaces can be arbitrarily nested.
Declaring an interface inside another does not require implementing types
of the outer interface to provide an implementation of the inner interfaces.

```cadence
// Declare a resource interface `OuterInterface`, which declares
// a nested structure interface named `InnerInterface`.
//
// Resources implementing `OuterInterface` do not need to provide
// an implementation of `InnerInterface`.
//
// Structures may just implement `InnerInterface`.
//
resource interface OuterInterface {

    struct interface InnerInterface {}
}

// Declare a resource named `SomeOuter` that implements the interface `OuterInterface`.
//
// The resource is not required to implement `OuterInterface.InnerInterface`.
//
resource SomeOuter: OuterInterface {}

// Declare a structure named `SomeInner` that implements `InnerInterface`,
// which is nested in interface `OuterInterface`.
//
struct SomeInner: OuterInterface.InnerInterface {}

```

## Interface Default Functions

Interfaces can provide default functions:
If the concrete type implementing the interface does not provide an implementation
for the function required by the interface,
then the interface's default function is used in the implementation.

```cadence
// Declare a struct interface `Container`,
// which declares a default function `getCount`.
//
struct interface Container {

    let items: [AnyStruct]

    fun getCount(): Int {
        return self.items.length
    }
}

// Declare a concrete struct named `Numbers` that implements the interface `Container`.
//
// The struct does not implement the function `getCount` of the interface `Container`,
// so the default function for `getCount` is used.
//
struct Numbers: Container {
    let items: [AnyStruct]

    init() {
        self.items = []
    }
}

let numbers = Numbers()
numbers.getCount()  // is 0
```

Interfaces cannot provide default initializers or default destructors.

Only one conformance may provide a default function.

## Nested Type Requirements

<Callout type="info">

🚧 Status: Currently only contracts and contract interfaces support nested type requirements.

</Callout>

Interfaces can require implementing types to provide concrete nested types.
For example, a resource interface may require an implementing type to provide a resource type.

```cadence
// Declare a resource interface named `FungibleToken`.
//
// Require implementing types to provide a resource type named `Vault`
// which must have a field named `balance`.
//
resource interface FungibleToken {
    pub resource Vault {
        pub balance: Int
    }
}
// Declare a resource named `ExampleToken` that implements the `FungibleToken` interface.
//
// The nested type `Vault` must be provided to conform to the interface.
//
resource ExampleToken: FungibleToken {
    pub resource Vault {
        pub var balance: Int
        init(balance: Int) {
            self.balance = balance
        }
    }
}
```
---
title: Operators
sidebar_position: 5
---

Operators are special symbols that perform a computation
for one or more values.
They are either unary, binary, or ternary.

- Unary operators perform an operation for a single value.
  The unary operator symbol appears before the value.

- Binary operators operate on two values.
    The binary operator symbol appears between the two values (infix).

- Ternary operators operate on three values.
  The first operator symbol appears between the first and second value,
  the second operator symbol appears between the second and third value (infix).

## Assignment Operator (`=`)

The binary assignment operator `=` can be used
to assign a new value to a variable.
It is only allowed in a statement and is not allowed in expressions.

```cadence
var a = 1
a = 2
// `a` is `2`


var b = 3
var c = 4

// Invalid: The assignment operation cannot be used in an expression.
a = b = c

// Instead, the intended assignment must be written in multiple statements.
b = c
a = b
```

Assignments to constants are invalid.

```cadence
let a = 1
// Invalid: Assignments are only for variables, not constants.
a = 2
```

The left-hand side of the assignment operand must be an identifier.
For arrays and dictionaries, this identifier can be followed
by one or more index or access expressions.

```cadence
// Declare an array of integers.
let numbers = [1, 2]

// Change the first element of the array.
//
numbers[0] = 3

// `numbers` is `[3, 2]`
```

```cadence
// Declare an array of arrays of integers.
let arrays = [[1, 2], [3, 4]]

// Change the first element in the second array
//
arrays[1][0] = 5

// `arrays` is `[[1, 2], [5, 4]]`
```

```cadence
let dictionaries = {
  true: {1: 2},
  false: {3: 4}
}

dictionaries[false][3] = 0

// `dictionaries` is `{
//   true: {1: 2},
//   false: {3: 0}
//}`
```

## Force-assignment operator (`<-!`)

The force-assignment operator (`<-!`) assigns a resource-typed value
to an optional-typed variable if the variable is nil.
If the variable being assigned to is non-nil,
the execution of the program aborts.

The force-assignment operator is only used for [resource types](./resources.mdx).

## Swapping Operator (`<->`)

The binary swap operator `<->` can be used
to exchange the values of two variables.
It is only allowed in a statement and is not allowed in expressions.

```cadence
var a = 1
var b = 2
a <-> b
// `a` is `2`
// `b` is `1`

var c = 3

// Invalid: The swap operation cannot be used in an expression.
a <-> b <-> c

// Instead, the intended swap must be written in multiple statements.
b <-> c
a <-> b
```

Both sides of the swap operation must be variable,
assignment to constants is invalid.

```cadence
var a = 1
let b = 2

// Invalid: Swapping is only possible for variables, not constants.
a <-> b
```

Both sides of the swap operation must be an identifier,
followed by one or more index or access expressions.

## Arithmetic Operators

The unary pefix operator  `-` negates an integer:

```cadence
let a = 1
-a  // is `-1`
```

There are four binary arithmetic operators:

- Addition: `+`
- Subtraction: `-`
- Multiplication: `*`
- Division: `/`
- Remainder: `%`

```cadence
let a = 1 + 2
// `a` is `3`
```

The arguments for the operators need to be of the same type.
The result is always the same type as the arguments.

The division and remainder operators abort the program when the divisor is zero.

Arithmetic operations on the signed integer types
`Int8`, `Int16`, `Int32`, `Int64`, `Int128`, `Int256`,
and on the unsigned integer types
`UInt8`, `UInt16`, `UInt32`, `UInt64`, `UInt128`, `UInt256`,
do not cause values to overflow or underflow.

```cadence
let a: UInt8 = 255

// Run-time error: The result `256` does not fit in the range of `UInt8`,
// thus a fatal overflow error is raised and the program aborts
//
let b = a + 1
```

```cadence
let a: Int8 = 100
let b: Int8 = 100

// Run-time error: The result `10000` does not fit in the range of `Int8`,
// thus a fatal overflow error is raised and the program aborts
//
let c = a * b
```

```cadence
let a: Int8 = -128

// Run-time error: The result `128` does not fit in the range of `Int8`,
// thus a fatal overflow error is raised and the program aborts
//
let b = -a
```

Arithmetic operations on the unsigned integer types
`Word8`, `Word16`, `Word32`, `Word64`
may cause values to overflow or underflow.

For example, the maximum value of an unsigned 8-bit integer is 255 (binary 11111111).
Adding 1 results in an overflow, truncation to 8 bits, and the value 0.

```cadence
//    11111111 = 255
// +         1
// = 100000000 = 0
```

```cadence
let a: Word8 = 255
a + 1 // is `0`
```

Similarly, for the minimum value 0,
subtracting 1 wraps around and results in the maximum value 255.

```cadence
//    00000000
// -         1
// =  11111111 = 255
```

```cadence
let b: Word8 = 0
b - 1  // is `255`
```

### Arithmetics on number super-types

Arithmetic operators are not supported for number supertypes
(`Number`, `SignedNumber`, `FixedPoint`, `SignedFixedPoint`, `Integer`, `SignedInteger`),
as they may or may not succeed at run-time.

```cadence
let x: Integer = 3 as Int8
let y: Integer = 4 as Int8

let z: Integer = x + y    // Static error
```

Values of these types need to be cast to the desired type before performing the arithmetic operation.

```cadence
let z: Integer = (x as! Int8) + (y as! Int8)
```

## Logical Operators

Logical operators work with the boolean values `true` and `false`.

- Logical NOT: `!a`

  This unary prefix operator logically negates a boolean:

  ```cadence
  let a = true
  !a  // is `false`
  ```

- Logical AND: `a && b`

  ```cadence
  true && true  // is `true`

  true && false  // is `false`

  false && true  // is `false`

  false && false  // is `false`
  ```

  If the left-hand side is false, the right-hand side is not evaluated.

- Logical OR: `a || b`

  ```cadence
  true || true  // is `true`

  true || false  // is `true`

  false || true  // is `true`

  false || false // is `false`
  ```

  If the left-hand side is true, the right-hand side is not evaluated.

## Comparison Operators

Comparison operators work with boolean and integer values.

- Equality: `==`, is supported for booleans, numbers, addresses, strings, characters, enums, paths, `Type` values, references, and `Void` values (`()`). Variable-sized arrays, fixed-size arrays, dictionaries, and optionals also support equality tests if their inner types do.

  Both sides of the equality operator may be optional, even of different levels,
  so it is for example possible to compare a non-optional with a double-optional (`??`).

  ```cadence
  1 == 1  // is `true`

  1 == 2  // is `false`
  ```

  ```cadence
  true == true  // is `true`

  true == false  // is `false`
  ```

  ```cadence
  let x: Int? = 1
  x == nil  // is `false`
  ```

  ```cadence
  let x: Int = 1
  x == nil  // is `false`
  ```

  ```cadence
  // Comparisons of different levels of optionals are possible.
  let x: Int? = 2
  let y: Int?? = nil
  x == y  // is `false`
  ```

  ```cadence
  // Comparisons of different levels of optionals are possible.
  let x: Int? = 2
  let y: Int?? = 2
  x == y  // is `true`
  ```

  ```cadence
  // Equality tests of arrays are possible if their inner types are equatable.
  let xs: [Int] = [1, 2, 3]
  let ys: [Int] = [1, 2, 3]
  xs == ys // is `true`

  let xss: [[Int]] = [xs, xs, xs]
  let yss: [[Int]] = [ys, ys, ys]
  xss == yss // is `true`
  ```

  ```cadence
  // Equality also applies to fixed-size arrays. If their lengths differ, the result is a type error.
  let xs: [Int; 2] = [1, 2]
  let ys: [Int; 2] = [0 + 1, 1 + 1]
  xs == ys // is `true`
  ```

  ```cadence
  // Equality tests of dictionaries are possible if the key and value types are equatable.
  let d1 = {"abc": 1, "def": 2}
  let d2 = {"abc": 1, "def": 2}
  d1 == d2 // is `true`

  let d3 = {"abc": {1: {"a": 1000}, 2: {"b": 2000}}, "def": {4: {"c": 1000}, 5: {"d": 2000}}}
  let d4 = {"abc": {1: {"a": 1000}, 2: {"b": 2000}}, "def": {4: {"c": 1000}, 5: {"d": 2000}}}
  d3 == d4 // is `true`
  ```

- Inequality: `!=`, is supported for booleans, numbers, addresses, strings, characters, enums, paths, `Type` values, references, and `Void` values (`()`).
  Variable-sized arrays, fixed-size arrays, dictionaries, and optionals also support inequality tests if their inner types do.

  Both sides of the inequality operator may be optional, even of different levels,
  so it is for example possible to compare a non-optional with a double-optional (`??`).

  ```cadence
  1 != 1  // is `false`

  1 != 2  // is `true`
  ```

  ```cadence
  true != true  // is `false`

  true != false  // is `true`
  ```

  ```cadence
  let x: Int? = 1
  x != nil  // is `true`
  ```

  ```cadence
  let x: Int = 1
  x != nil  // is `true`
  ```

  ```cadence
  // Comparisons of different levels of optionals are possible.
  let x: Int? = 2
  let y: Int?? = nil
  x != y  // is `true`
  ```

  ```cadence
  // Comparisons of different levels of optionals are possible.
  let x: Int? = 2
  let y: Int?? = 2
  x != y  // is `false`
  ```

  ```cadence
  // Inequality tests of arrays are possible if their inner types are equatable.
  let xs: [Int] = [1, 2, 3]
  let ys: [Int] = [4, 5, 6]
  xs != ys // is `true`
  ```

  ```cadence
  // Inequality also applies to fixed-size arrays. If their lengths differ, the result is a type error.
  let xs: [Int; 2] = [1, 2]
  let ys: [Int; 2] = [1, 2]
  xs != ys // is `false`
  ```

  ```cadence
  // Inequality tests of dictionaries are possible if the key and value types are equatable.
  let d1 = {"abc": 1, "def": 2}
  let d2 = {"abc": 1, "def": 500}
  d1 != d2 // is `true`

  let d3 = {"abc": {1: {"a": 1000}, 2: {"b": 2000}}, "def": {4: {"c": 1000}, 5: {"d": 2000}}}
  let d4 = {"abc": {1: {"a": 1000}, 2: {"b": 2000}}, "def": {4: {"c": 1000}, 5: {"d": 2000}}}
  d3 != d4 // is `false`
  ```

- Less than: `<`, for integers, booleans, characters and strings

  ```cadence
  1 < 1  // is `false`

  1 < 2  // is `true`

  2 < 1  // is `false`

  false < true // is `true`

  true < true  // is `false`

  "a" < "b"    // is `true`

  "z" < "a"    // is `false`

  "a" < "A"    // is `false`

  "" < ""      // is `false`

  "" < "a"     // is `true`

  "az" < "b"   // is `true`

  "xAB" < "Xab"  // is `false`
  ```

- Less or equal than: `<=`, for integers, booleans, characters and strings

  ```cadence
  1 <= 1  // is `true`

  1 <= 2  // is `true`

  2 <= 1  // is `false`

  false <= true // is `true`

  true <= true  // is `true`

  true <= false // is `false`

  "c"  <= "a"   // is `false`

  "z"  <= "z"   // is `true`

  "a" <= "A"    // is `false`

  "" <= ""      // is `true`

  "" <= "a"     // is `true`

  "az" <= "b"   // is `true`

  "xAB" <= "Xab"  // is `false`
  ```

- Greater than: `>`, for integers, booleans, characters and strings

  ```cadence
  1 > 1  // is `false`

  1 > 2  // is `false`

  2 > 1  // is `true`

  false > true // is `false`

  true > true  // is `false`

  true > false // is `true`

  "c"  > "a"   // is `true`

  "g"  > "g"   // is `false`

  "a" > "A"    // is `true`

  "" > ""      // is `false`

  "" > "a"     // is `false`

  "az" > "b"   // is `false`

  "xAB" > "Xab"  // is `true`
  ```

- Greater or equal than: `>=`, for integers, booleans, characters and strings

  ```cadence
  1 >= 1  // is `true`

  1 >= 2  // is `false`

  2 >= 1  // is `true`

  false >= true // is `false`

  true >= true  // is `true`

  true >= false // is `true`

  "c"  >= "a"   // is `true`

  "q"  >= "q"   // is `true`

  "a" >= "A"    // is `true`

  "" >= ""      // is `true`

  "" >= "a"     // is `true`

  "az" >= "b"   // is `true`

  "xAB" >= "Xab"  // is `false`
  ```

### Comparing number super-types

Similar to arithmetic operators, comparison operators are also not supported for number supertypes
(`Number`, `SignedNumber` `FixedPoint`, `SignedFixedPoint`, `Integer`, `SignedInteger`),
as they may or may not succeed at run-time.

```cadence
let x: Integer = 3 as Int8
let y: Integer = 4 as Int8

let z: Bool = x > y    // Static error
```

Values of these types need to be cast to the desired type before performing the arithmetic operation.

```cadence
let z: Bool = (x as! Int8) > (y as! Int8)
```

## Bitwise Operators

Bitwise operators enable the manipulation of individual bits of unsigned and signed integers.
They're often used in low-level programming.

- Bitwise AND: `a & b`

  Returns a new integer whose bits are 1 only if the bits were 1 in *both* input integers:

  ```cadence
  let firstFiveBits = 0b11111000
  let lastFiveBits  = 0b00011111
  let middleTwoBits = firstFiveBits & lastFiveBits  // is 0b00011000
  ```

- Bitwise OR: `a | b`

  Returns a new integer whose bits are 1 only if the bits were 1 in *either* input integers:

  ```cadence
  let someBits = 0b10110010
  let moreBits = 0b01011110
  let combinedbits = someBits | moreBits  // is 0b11111110
  ```

- Bitwise XOR: `a ^ b`

  Returns a new integer whose bits are 1 where the input bits are different,
  and are 0 where the input bits are the same:

  ```cadence
  let firstBits = 0b00010100
  let otherBits = 0b00000101
  let outputBits = firstBits ^ otherBits  // is 0b00010001
  ```

### Bitwise Shifting Operators

- Bitwise LEFT SHIFT: `a << b`

  Returns a new integer with all bits moved to the left by a certain number of places.

  ```cadence
  let someBits = 4  // is 0b00000100
  let shiftedBits = someBits << 2   // is 0b00010000
  ```

- Bitwise RIGHT SHIFT: `a >> b`

  Returns a new integer with all bits moved to the right by a certain number of places.

  ```cadence
  let someBits = 8  // is 0b00001000
  let shiftedBits = someBits >> 2   // is 0b00000010
  ```

For unsigned integers, the bitwise shifting operators perform [logical shifting](https://en.wikipedia.org/wiki/Logical_shift),
for signed integers, they perform [arithmetic shifting](https://en.wikipedia.org/wiki/Arithmetic_shift).

## Ternary Conditional Operator

There is only one ternary conditional operator, the ternary conditional operator (`a ? b : c`).

It behaves like an if-statement, but is an expression:
If the first operator value is true, the second operator value is returned.
If the first operator value is false, the third value is returned.

The first value must be a boolean (must have the type `Bool`).
The second value and third value can be of any type.
The result type is the least common supertype of the second and third value.

```cadence
let x = 1 > 2 ? 3 : 4
// `x` is `4` and has type `Int`

let y = 1 > 2 ? nil : 3
// `y` is `3` and has type `Int?`
```

## Casting Operators

### Static Casting Operator (`as`)

The static casting operator `as` can be used to statically type cast a value to a type.

If the static type of the value is a subtype of the given type (the "target" type),
the operator returns the value as the given type.

The cast is performed statically, i.e. when the program is type-checked.
Only the static type of the value is considered, not the run-time type of the value.

This means it is not possible to downcast using this operator.
Consider using the [conditional downcasting operator `as?`](#conditional-downcasting-operator-as) instead.

```cadence
// Declare a constant named `integer` which has type `Int`.
//
let integer: Int = 1

// Statically cast the value of `integer` to the supertype `Number`.
// The cast succeeds, because the type of the variable `integer`,
// the type `Int`, is a subtype of type `Number`.
// This is an upcast.
//
let number = integer as Number
// `number` is `1` and has type `Number`

// Declare a constant named `something` which has type `AnyStruct`,
// with an initial value which has type `Int`.
//
let something: AnyStruct = 1

// Statically cast the value of `something` to `Int`.
// This is invalid, the cast fails, because the static type of the value is type `AnyStruct`,
// which is not a subtype of type `Int`.
//
let result = something as Int
```

### Conditional Downcasting Operator (`as?`)

The conditional downcasting operator `as?` can be used to dynamically type cast a value to a type.
The operator returns an optional.
If the value has a run-time type that is a subtype of the target type
the operator returns the value as the target type,
otherwise the result is `nil`.

The cast is performed at run-time, i.e. when the program is executed,
not statically, i.e. when the program is checked.

```cadence
// Declare a constant named `something` which has type `AnyStruct`,
// with an initial value which has type `Int`.
//
let something: AnyStruct = 1

// Conditionally downcast the value of `something` to `Int`.
// The cast succeeds, because the value has type `Int`.
//
let number = something as? Int
// `number` is `1` and has type `Int?`

// Conditionally downcast the value of `something` to `Bool`.
// The cast fails, because the value has type `Int`,
// and `Bool` is not a subtype of `Int`.
//
let boolean = something as? Bool
// `boolean` is `nil` and has type `Bool?`
```

Downcasting works for concrete types, but also works e.g. for nested types (e.g. arrays), interfaces, optionals, etc.

```cadence
// Declare a constant named `values` which has type `[AnyStruct]`,
// i.e. an array of arbitrarily typed values.
//
let values: [AnyStruct] = [1, true]

let first = values[0] as? Int
// `first` is `1` and has type `Int?`

let second = values[1] as? Bool
// `second` is `true` and has type `Bool?`
```

### Force-downcasting Operator (`as!`)

The force-downcasting operator `as!` behaves like the
[conditional downcasting operator `as?`](#conditional-downcasting-operator-as).
However, if the cast succeeds, it returns a value of the given type instead of an optional,
and if the cast fails, it aborts the program instead of returning `nil`,

```cadence
// Declare a constant named `something` which has type `AnyStruct`,
// with an initial value which has type `Int`.
//
let something: AnyStruct = 1

// Force-downcast the value of `something` to `Int`.
// The cast succeeds, because the value has type `Int`.
//
let number = something as! Int
// `number` is `1` and has type `Int`

// Force-downcast the value of `something` to `Bool`.
// The cast fails, because the value has type `Int`,
// and `Bool` is not a subtype of `Int`.
//
let boolean = something as! Bool
// Run-time error
```

## Optional Operators

### Nil-Coalescing Operator (`??`)

The nil-coalescing operator `??` returns
the value inside an optional if it contains a value,
or returns an alternative value if the optional has no value,
i.e., the optional value is `nil`.

If the left-hand side is non-nil, the right-hand side is not evaluated.

```cadence
// Declare a constant which has an optional integer type
//
let a: Int? = nil

// Declare a constant with a non-optional integer type,
// which is initialized to `a` if it is non-nil, or 42 otherwise.
//
let b: Int = a ?? 42
// `b` is 42, as `a` is nil
```

The nil-coalescing operator can only be applied
to values which have an optional type.

```cadence
// Declare a constant with a non-optional integer type.
//
let a = 1

// Invalid: nil-coalescing operator is applied to a value which has a non-optional type
// (a has the non-optional type `Int`).
//
let b = a ?? 2
```

```cadence
// Invalid: nil-coalescing operator is applied to a value which has a non-optional type
// (the integer literal is of type `Int`).
//
let c = 1 ?? 2
```

The type of the right-hand side of the operator (the alternative value) must be a subtype
of the type of left-hand side, i.e. the right-hand side of the operator must
be the non-optional or optional type matching the type of the left-hand side.

```cadence
// Declare a constant with an optional integer type.
//
let a: Int? = nil
let b: Int? = 1
let c = a ?? b
// `c` is `1` and has type `Int?`

// Invalid: nil-coalescing operator is applied to a value of type `Int?`,
// but the alternative has type `Bool`.
//
let d = a ?? false
```

### Force Unwrap Operator (`!`)

The force-unwrap operator (`!`) returns
the value inside an optional if it contains a value,
or panics and aborts the execution if the optional has no value,
i.e., the optional value is `nil`.

```cadence
// Declare a constant which has an optional integer type
//
let a: Int? = nil

// Declare a constant with a non-optional integer type,
// which is initialized to `a` if `a` is non-nil.
// If `a` is nil, the program aborts.
//
let b: Int = a!
// The program aborts because `a` is nil.

// Declare another optional integer constant
let c: Int? = 3

// Declare a non-optional integer
// which is initialized to `c` if `c` is non-nil.
// If `c` is nil, the program aborts.
let d: Int = c!
// `d` is initialized to 3 because c isn't nil.

```

The force-unwrap operator can only be applied
to values which have an optional type.

```cadence
// Declare a constant with a non-optional integer type.
//
let a = 1

// Invalid: force-unwrap operator is applied to a value which has a
// non-optional type (`a` has the non-optional type `Int`).
//
let b = a!
```

```cadence
// Invalid: The force-unwrap operator is applied
// to a value which has a non-optional type
// (the integer literal is of type `Int`).
//
let c = 1!
```


## Precedence and Associativity

Operators have the following precedences, highest to lowest:

- Unary precedence: `-`, `!`, `<-`
- Cast precedence: `as`, `as?`, `as!`
- Multiplication precedence: `*`, `/`, `%`
- Addition precedence: `+`, `-`
- Bitwise shift precedence: `<<`, `>>`
- Bitwise conjunction precedence: `&`
- Bitwise exclusive disjunction precedence: `^`
- Bitwise disjunction precedence: `|`
- Nil-Coalescing precedence: `??`
- Relational precedence: `<`, `<=`, `>`, `>=`
- Equality precedence: `==`, `!=`
- Logical conjunction precedence: `&&`
- Logical disjunction precedence: `||`
- Ternary precedence: `? :`

All operators are left-associative, except for the following operators which are right-associative:
- Ternary operator
- Nil-coalescing operator

Expressions can be wrapped in parentheses to override precedence conventions,
i.e. an alternate order should be indicated, or when the default order should be emphasized
e.g. to avoid confusion.
For example, `(2 + 3) * 4` forces addition to precede multiplication,
and `5 + (6 * 7)` reinforces the default order.
---
title: References
sidebar_position: 17
---

It is possible to create references to objects, i.e. resources or structures.
A reference can be used to access fields and call functions on the referenced object.

References are **copied**, i.e. they are value types.

References are created by using the `&` operator, followed by the object,
the `as` keyword, and the type through which they should be accessed.
The given type must be a supertype of the referenced object's type.

References have the type `&T`, where `T` is the type of the referenced object.

```cadence
let hello = "Hello"

// Create a reference to the "Hello" string, typed as a `String`
//
let helloRef: &String = &hello as &String

helloRef.length // is `5`

// Invalid: Cannot create a reference to `hello`
// typed as `&Int`, as it has type `String`
//
let intRef: &Int = &hello as &Int
```

If you attempt to reference an optional value, you will receive an optional reference.
If the referenced value is nil, the reference itself will be nil. If the referenced value
exists, then forcing the optional reference will yield a reference to that value:

```cadence
let nilValue: String? = nil
let nilRef = &nilValue as &String? // r has type &String?
let n = nilRef! // error, forced nil value

let strValue: String? = ""
let strRef = &strValue as &String? // r has type &String?
let n = strRef! // n has type &String
```

References are covariant in their base types.
For example, `&T` is a subtype of `&U`, if `T` is a subtype of `U`.

```cadence

// Declare a resource interface named `HasCount`,
// that has a field `count`
//
resource interface HasCount {
    count: Int
}

// Declare a resource named `Counter` that conforms to `HasCount`
//
resource Counter: HasCount {
    pub var count: Int

    pub init(count: Int) {
        self.count = count
    }

    pub fun increment() {
        self.count = self.count + 1
    }
}

// Create a new instance of the resource type `Counter`
// and create a reference to it, typed as `&Counter`,
// so the reference allows access to all fields and functions
// of the counter
//
let counter <- create Counter(count: 42)
let counterRef: &Counter = &counter as &Counter

counterRef.count  // is `42`

counterRef.increment()

counterRef.count  // is `43`
```

References may be **authorized** or **unauthorized**.

Authorized references have the `auth` modifier, i.e. the full syntax is `auth &T`,
whereas unauthorized references do not have a modifier.

Authorized references can be freely upcasted and downcasted,
whereas unauthorized references can only be upcasted.
Also, authorized references are subtypes of unauthorized references.

```cadence

// Create an unauthorized reference to the counter,
// typed with the restricted type `&{HasCount}`,
// i.e. some resource that conforms to the `HasCount` interface
//
let countRef: &{HasCount} = &counter as &{HasCount}

countRef.count  // is `43`

// Invalid: The function `increment` is not available
// for the type `&{HasCount}`
//
countRef.increment()

// Invalid: Cannot conditionally downcast to reference type `&Counter`,
// as the reference `countRef` is unauthorized.
//
// The counter value has type `Counter`, which is a subtype of `{HasCount}`,
// but as the reference is unauthorized, the cast is not allowed.
// It is not possible to "look under the covers"
//
let counterRef2: &Counter = countRef as? &Counter

// Create an authorized reference to the counter,
// again with the restricted type `{HasCount}`, i.e. some resource
// that conforms to the `HasCount` interface
//
let authCountRef: auth &{HasCount} = &counter as auth &{HasCount}

// Conditionally downcast to reference type `&Counter`.
// This is valid, because the reference `authCountRef` is authorized
//
let counterRef3: &Counter = authCountRef as? &Counter

counterRef3.count  // is `43`

counterRef3.increment()

counterRef3.count  // is `44`
```

References are ephemeral, i.e they cannot be [stored](./accounts.mdx#account-storage).
Instead, consider [storing a capability and borrowing it](./capabilities.md) when needed.
---
title: Resources
description: Enables resource-oriented programming on Flow
sidebar_position: 12
---

Resources are types that can only exist in **one** location at a time
and **must** be used **exactly once**.

Resources **must** be created (instantiated) by using the `create` keyword.

At the end of a function which has resources (variables, constants, parameters) in scope,
the resources **must** be either **moved** or **destroyed**.

They are **moved** when used as an initial value for a constant or variable,
when assigned to a different variable,
when passed as an argument to a function,
and when returned from a function.

Resources can be explicitly **destroyed** using the `destroy` keyword.

Accessing a field or calling a function of a resource does not move or destroy it.

When the resource is moved, the constant or variable
that referred to the resource before the move becomes **invalid**.
An **invalid** resource cannot be used again.

To make the usage and behaviour of resource types explicit,
the prefix `@` must be used in type annotations
of variable or constant declarations, parameters, and return types.

### The Move Operator (`<-`)

To make moves of resources explicit, the move operator `<-` must be used
when the resource is the initial value of a constant or variable,
when it is moved to a different variable,
when it is moved to a function as an argument,
and when it is returned from a function.

```cadence
// Declare a resource named `SomeResource`, with a variable integer field.
//
pub resource SomeResource {
    pub var value: Int

    init(value: Int) {
        self.value = value
    }
}

// Declare a constant with value of resource type `SomeResource`.
//
let a: @SomeResource <- create SomeResource(value: 0)

// *Move* the resource value to a new constant.
//
let b <- a

// Invalid: Cannot use constant `a` anymore as the resource that it referred to
// was moved to constant `b`.
//
a.value

// Constant `b` owns the resource.
//
b.value // equals 0

// Declare a function which accepts a resource.
//
// The parameter has a resource type, so the type annotation must be prefixed with `@`.
//
pub fun use(resource: @SomeResource) {
    // ...
}

// Call function `use` and move the resource into it.
//
use(resource: <-b)

// Invalid: Cannot use constant `b` anymore as the resource
// it referred to was moved into function `use`.
//
b.value
```

A resource object cannot go out of scope and be dynamically lost.
The program must either explicitly destroy it or move it to another context.

```cadence
{
    // Declare another, unrelated value of resource type `SomeResource`.
    //
    let c <- create SomeResource(value: 10)

    // Invalid: `c` is not used before the end of the scope, but must be.
    // It cannot be lost.
}
```

```cadence
// Declare another, unrelated value of resource type `SomeResource`.
//
let d <- create SomeResource(value: 20)

// Destroy the resource referred to by constant `d`.
//
destroy d

// Invalid: Cannot use constant `d` anymore as the resource
// it referred to was destroyed.
//
d.value
```

To make it explicit that the type is a resource type
and must follow the rules associated with resources,
it must be prefixed with `@` in all type annotations,
e.g. for variable declarations, parameters, or return types.

```cadence
// Declare a constant with an explicit type annotation.
//
// The constant has a resource type, so the type annotation must be prefixed with `@`.
//
let someResource: @SomeResource <- create SomeResource(value: 5)

// Declare a function which consumes a resource and destroys it.
//
// The parameter has a resource type, so the type annotation must be prefixed with `@`.
//
pub fun use(resource: @SomeResource) {
    destroy resource
}

// Declare a function which returns a resource.
//
// The return type is a resource type, so the type annotation must be prefixed with `@`.
// The return statement must also use the `<-` operator to make it explicit the resource is moved.
//
pub fun get(): @SomeResource {
    let newResource <- create SomeResource()
    return <-newResource
}
```

Resources **must** be used exactly once.

```cadence
// Declare a function which consumes a resource but does not use it.
// This function is invalid, because it would cause a loss of the resource.
//
pub fun forgetToUse(resource: @SomeResource) {
    // Invalid: The resource parameter `resource` is not used, but must be.
}
```

```cadence
// Declare a constant named `res` which has the resource type `SomeResource`.
let res <- create SomeResource()

// Call the function `use` and move the resource `res` into it.
use(resource: <-res)

// Invalid: The resource constant `res` cannot be used again,
// as it was moved in the previous function call.
//
use(resource: <-res)

// Invalid: The resource constant `res` cannot be used again,
// as it was moved in the previous function call.
//
res.value
```

```cadence
// Declare a function which has a resource parameter.
// This function is invalid, because it does not always use the resource parameter,
// which would cause a loss of the resource.
//
pub fun sometimesDestroy(resource: @SomeResource, destroyResource: Bool) {
    if destroyResource {
        destroy resource
    }
    // Invalid: The resource parameter `resource` is not always used, but must be.
    // The destroy statement is not always executed, so at the end of this function
    // it might have been destroyed or not.
}
```

```cadence
// Declare a function which has a resource parameter.
// This function is valid, as it always uses the resource parameter,
// and does not cause a loss of the resource.
//
pub fun alwaysUse(resource: @SomeResource, destroyResource: Bool) {
    if destroyResource {
        destroy resource
    } else {
        use(resource: <-resource)
    }
    // At the end of the function the resource parameter was definitely used:
    // It was either destroyed or moved in the call of function `use`.
}
```

```cadence
// Declare a function which has a resource parameter.
// This function is invalid, because it does not always use the resource parameter,
// which would cause a loss of the resource.
//
pub fun returnBeforeDestroy(move: Bool) {
    let res <- create SomeResource(value: 1)
    if move {
        use(resource: <-res)
        return
    } else {
        // Invalid: When this function returns here, the resource variable
        // `res` was not used, but must be.
        return
    }
    // Invalid: the resource variable `res` was potentially moved in the
    // previous if-statement, and both branches definitely return,
    // so this statement is unreachable.
    destroy res
}
```

### Resource Variables

Resource variables cannot be assigned to,
as that would lead to the loss of the variable's current resource value.

Instead, use a swap statement (`<->`) or shift statement (`<- target <-`)
to replace the resource variable with another resource.

```cadence
pub resource R {}

var x <- create R()
var y <- create R()

// Invalid: Cannot assign to resource variable `x`,
// as its current resource would be lost
//
x <- y

// Instead, use a swap statement.
//
var replacement <- create R()
x <-> replacement
// `x` is the new resource.
// `replacement` is the old resource.

// Or use the shift statement (`<- target <-`)
// This statement moves the resource out of `x` and into `oldX`,
// and at the same time assigns `x` with the new value on the right-hand side.
let oldX <- x <- create R()
// oldX still needs to be explicitly handled after this statement
destroy oldX
```

### Resource Destructors

Resource may have a destructor, which is executed when the resource is destroyed.
Destructors have no parameters and no return value and are declared using the `destroy` name.
A resource may have only one destructor.

```cadence
var destructorCalled = false

pub resource Resource {

    // Declare a destructor for the resource, which is executed
    // when the resource is destroyed.
    //
    destroy() {
        destructorCalled = true
    }
}

let res <- create Resource()
destroy res
// `destructorCalled` is `true`
```

### Nested Resources

Fields in composite types behave differently when they have a resource type.

If a resource type has fields that have a resource type,
it **must** declare a destructor,
which **must** invalidate all resource fields, i.e. move or destroy them.

```cadence
pub resource Child {
    let name: String

    init(name: String)
        self.name = name
    }
}

// Declare a resource with a resource field named `child`.
// The resource *must* declare a destructor
// and the destructor *must* invalidate the resource field.
//
pub resource Parent {
    let name: String
    var child: @Child

    init(name: String, child: @Child) {
        self.name = name
        self.child <- child
    }

    // Declare a destructor which invalidates the resource field
    // `child` by destroying it.
    //
    destroy() {
        destroy self.child
    }
}
```

Accessing a field or calling function on a resource field is valid,
however moving a resource out of a variable resource field is **not** allowed.
Instead, use a swap statement to replace the resource with another resource.

```cadence
let child <- create Child(name: "Child 1")
let parent <- create Parent(name: "Parent", child: <-child)

child.name  // is "Child 1"
parent.child.name  // is "Child 1"

// Invalid: Cannot move resource out of variable resource field.
let childAgain <- parent.child

// Instead, use a swap statement.
//
var otherChild <- create Child(name: "Child 2")
parent.child <-> otherChild
// `parent.child` is the second child, Child 2.
// `otherChild` is the first child, Child 1.
```

### Resources in Closures

Resources can not be captured in closures, as that could potentially result in duplications.

```cadence
resource R {}

// Invalid: Declare a function which returns a closure which refers to
// the resource parameter `resource`. Each call to the returned function
// would return the resource, which should not be possible.
//
fun makeCloner(resource: @R): ((): @R) {
    return fun (): @R {
        return <-resource
    }
}

let test = makeCloner(resource: <-create R())
```

### Resources in Arrays and Dictionaries

Arrays and dictionaries behave differently when they contain resources:
It is **not** allowed to index into an array to read an element at a certain index or assign to it,
or index into a dictionary to read a value for a certain key or set a value for the key.

Instead, use a swap statement (`<->`) or shift statement (`<- target <-`)
to replace the accessed resource with another resource.

```cadence
resource R {}

// Declare a constant for an array of resources.
// Create two resources and move them into the array.
// `resources` has type `@[R]`
//
let resources <- [
    <-create R(),
    <-create R()
]

// Invalid: Reading an element from a resource array is not allowed.
//
let firstResource <- resources[0]

// Invalid: Setting an element in a resource array is not allowed,
// as it would result in the loss of the current value.
//
resources[0] <- create R()

// Instead, when attempting to either read an element or update an element
// in a resource array, use a swap statement with a variable to replace
// the accessed element.
//
var res <- create R()
resources[0] <-> res
// `resources[0]` now contains the new resource.
// `res` now contains the old resource.

// Use the shift statement to move the new resource into
// the array at the same time that the old resource is being moved out
let oldRes <- resources[0] <- create R()
// The old object still needs to be handled
destroy oldRes
```

The same applies to dictionaries.

```cadence
// Declare a constant for a dictionary of resources.
// Create two resources and move them into the dictionary.
// `resources` has type `@{String: R}`
//
let resources <- {
    "r1": <-create R(),
    "r2": <-create R()
}

// Invalid: Reading an element from a resource dictionary is not allowed.
// It's not obvious that an access like this would have to remove
// the key from the dictionary.
//
let firstResource <- resources["r1"]

// Instead, make the removal explicit by using the `remove` function.
let firstResource <- resources.remove(key: "r1")

// Invalid: Setting an element in a resource dictionary is not allowed,
// as it would result in the loss of the current value.
//
resources["r1"] <- create R()

// Instead, when attempting to either read an element or update an element
// in a resource dictionary, use a swap statement with a variable to replace
// the accessed element.
//
// The result of a dictionary read is optional, as the given key might not
// exist in the dictionary.
// The types on both sides of the swap operator must be the same,
// so also declare the variable as an optional.
//
var res: @R? <- create R()
resources["r1"] <-> res
// `resources["r1"]` now contains the new resource.
// `res` now contains the old resource.

// Use the shift statement to move the new resource into
// the dictionary at the same time that the old resource is being moved out
let oldRes <- resources["r2"] <- create R()
// The old object still needs to be handled
destroy oldRes
```

Resources cannot be moved into arrays and dictionaries multiple times,
as that would cause a duplication.

```cadence
let resource <- create R()

// Invalid: The resource variable `resource` can only be moved into the array once.
//
let resources <- [
    <-resource,
    <-resource
]
```

```cadence
let resource <- create R()

// Invalid: The resource variable `resource` can only be moved into the dictionary once.
let resources <- {
    "res1": <-resource,
    "res2": <-resource
}
```

Resource arrays and dictionaries can be destroyed.

```cadence
let resources <- [
    <-create R(),
    <-create R()
]
destroy resources
```

```cadence
let resources <- {
    "r1": <-create R(),
    "r2": <-create R()
}
destroy resources
```

The variable array functions like `append`, `insert`, and `remove`
behave like for non-resource arrays.
Note however, that the result of the `remove` functions must be used.

```cadence
let resources <- [<-create R()]
// `resources.length` is `1`

resources.append(<-create R())
// `resources.length` is `2`

let first <- resource.remove(at: 0)
// `resources.length` is `1`
destroy first

resources.insert(at: 0, <-create R())
// `resources.length` is `2`

// Invalid: The statement ignores the result of the call to `remove`,
// which would result in a loss.
resource.remove(at: 0)

destroy resources
```

The variable array function `contains` is not available, as it is impossible:
If the resource can be passed to the `contains` function,
it is by definition not in the array.

The variable array function `concat` is not available,
as it would result in the duplication of resources.

The dictionary functions like `insert` and `remove`
behave like for non-resource dictionaries.
Note however, that the result of these functions must be used.

```cadence
let resources <- {"r1": <-create R()}
// `resources.length` is `1`

let first <- resource.remove(key: "r1")
// `resources.length` is `0`
destroy first

let old <- resources.insert(key: "r1", <-create R())
// `old` is nil, as there was no value for the key "r1"
// `resources.length` is `1`

let old2 <- resources.insert(key: "r1", <-create R())
// `old2` is the old value for the key "r1"
// `resources.length` is `1`

destroy old
destroy old2
destroy resources
```

### Resource Identifier

Resources have an implicit unique identifier associated with them,
implemented by a predeclared public field `let uuid: UInt64` on each resource.

This identifier will be automatically set when the resource is created, before the resource's initializer is called
(i.e. the identifier can be used in the initializer),
and will be unique even after the resource is destroyed,
i.e. no two resources will ever have the same identifier.

```cadence
// Declare a resource without any fields.
resource R {}

// Create two resources
let r1 <- create R()
let r2 <- create R()

// Get each resource's unique identifier
let id1 = r1.uuid
let id2 = r2.uuid

// Destroy the first resource
destroy r1

// Create a third resource
let r3 <- create R()

let id3 = r3.uuid

id1 != id2  // true
id2 != id3  // true
id3 != id1  // true
```

<Callout type="warning">
The details of how the identifiers are generated is an implementation detail.

Do not rely on or assume any particular behaviour in Cadence programs.
</Callout>

## Resource Owner

Resources have the implicit field `let owner: PublicAccount?`.
If the resource is currently [stored in an account](./accounts.mdx#account-storage),
then the field contains the publicly accessible portion of the account.
Otherwise the field is `nil`.

The field's value changes when the resource is moved from outside account storage
into account storage, when it is moved from the storage of one account
to the storage of another account, and when it is moved out of account storage.
---
title: Restricted Types
sidebar_position: 16
---

Structure and resource types can be **restricted**. Restrictions are interfaces.
Restricted types only allow access to a subset of the members and functions
of the type that is restricted, indicated by the restrictions.

The syntax of a restricted type is `T{U1, U2, ... Un}`,
where `T` is the restricted type, a concrete resource or structure type,
and the types `U1` to `Un` are the restrictions, interfaces that `T` conforms to.

Only the members and functions of the union of the set of restrictions are available.

Restricted types are useful for increasing the safety in functions
that are supposed to only work on a subset of the type.
For example, by using a restricted type for a parameter's type,
the function may only access the functionality of the restriction:
If the function accidentally attempts to access other functionality,
this is prevented by the static checker.

```cadence
// Declare a resource interface named `HasCount`,
// which has a read-only `count` field
//
resource interface HasCount {
    pub let count: Int
}

// Declare a resource named `Counter`, which has a writeable `count` field,
// and conforms to the resource interface `HasCount`
//
pub resource Counter: HasCount {
    pub var count: Int

    init(count: Int) {
        self.count = count
    }

    pub fun increment() {
        self.count = self.count + 1
    }
}

// Create an instance of the resource `Counter`
let counter: @Counter <- create Counter(count: 42)

counter.count  // is `42`

counter.increment()

counter.count  // is `43`

// Move the resource in variable `counter` to a new variable `restrictedCounter`,
// but typed with the restricted type `Counter{HasCount}`:
// The variable may hold any `Counter`, but only the functionality
// defined in the given restriction, the interface `HasCount`, may be accessed
//
let restrictedCounter: @Counter{HasCount} <- counter

// Invalid: Only functionality of restriction `Count` is available,
// i.e. the read-only field `count`, but not the function `increment` of `Counter`
//
restrictedCounter.increment()

// Move the resource in variable `restrictedCounter` to a new variable `unrestrictedCounter`,
// again typed as `Counter`, i.e. all functionality of the counter is available
//
let unrestrictedCounter: @Counter <- restrictedCounter

// Valid: The variable `unrestrictedCounter` has type `Counter`,
// so all its functionality is available, including the function `increment`
//
unrestrictedCounter.increment()

// Declare another resource type named `Strings`
// which implements the resource interface `HasCount`
//
pub resource Strings: HasCount {
    pub var count: Int
    access(self) var strings: [String]

    init() {
        self.count = 0
        self.strings = []
    }

    pub fun append(_ string: String) {
        self.strings.append(string)
        self.count = self.count + 1
    }
}

// Invalid: The resource type `Strings` is not compatible
// with the restricted type `Counter{HasCount}`.
// Even though the resource `Strings` implements the resource interface `HasCount`,
// it is not compatible with `Counter`
//
let counter2: @Counter{HasCount} <- create Strings()
```

In addition to restricting concrete types is also possible
to restrict the built-in types `AnyStruct`, the supertype of all structures,
and `AnyResource`, the supertype of all resources.
For example, restricted type `AnyResource{HasCount}` is any resource type
for which only the functionality of the `HasCount` resource interface can be used.

The restricted types `AnyStruct` and `AnyResource` can be omitted.
For example, the type `{HasCount}` is any resource that implements
the resource interface `HasCount`.

```cadence
pub struct interface HasID {
    pub let id: String
}

pub struct A: HasID {
    pub let id: String

    init(id: String) {
        self.id = id
    }
}

pub struct B: HasID {
    pub let id: String

    init(id: String) {
        self.id = id
    }
}

// Create two instances, one of type `A`, and one of type `B`.
// Both types conform to interface `HasID`, so the structs can be assigned
// to variables with type `AnyResource{HasID}`: Some resource type which only allows
// access to the functionality of resource interface `HasID`

let hasID1: {HasID} = A(id: "1")
let hasID2: {HasID} = B(id: "2")

// Declare a function named `getID` which has one parameter with type `{HasID}`.
// The type `{HasID}` is a short-hand for `AnyStruct{HasID}`:
// Some structure which only allows access to the functionality of interface `HasID`.
//
pub fun getID(_ value: {HasID}): String {
    return value.id
}

let id1 = getID(hasID1)
// `id1` is "1"

let id2 = getID(hasID2)
// `id2` is "2"
```

Only concrete types may be restricted, e.g., the restricted type may not be an array,
the type `[T]{U}` is invalid.

Restricted types are also useful when giving access to resources and structures
to potentially untrusted third-party programs through [references](./resources.mdx),
which are discussed in the next section.
---
title: Run-time Types
sidebar_position: 27
---

Types can be represented at run-time.
To create a type value, use the constructor function `Type<T>()`, which accepts the static type as a type argument.

This is similar to e.g. `T.self` in Swift, `T::class`/`KClass<T>` in Kotlin, and `T.class`/`Class<T>` in Java.

For example, to represent the type `Int` at run-time:

```cadence
let intType: Type = Type<Int>()
```

This works for both built-in and user-defined types. For example, to get the type value for a resource:

```cadence
resource Collectible {}

let collectibleType = Type<@Collectible>()

// `collectibleType` has type `Type`
```

Type values are comparable.

```cadence

Type<Int>() == Type<Int>()

Type<Int>() != Type<String>()
```

The method `fun isSubtype(of: Type): Bool` can be used to compare the run-time types of values.

```cadence
Type<Int>().isSubtype(of: Type<Int>()) // true

Type<Int>().isSubtype(of: Type<String>()) // false

Type<Int>().isSubtype(of: Type<Int?>()) // true
```

To get the run-time type's fully qualified type identifier, use the `let identifier: String` field:

```cadence
let type = Type<Int>()
type.identifier  // is "Int"
```

```cadence
// in account 0x1

struct Test {}

let type = Type<Test>()
type.identifier  // is "A.0000000000000001.Test"
```

### Getting the Type from a Value

The method `fun getType(): Type` can be used to get the runtime type of a value.

```cadence
let something = "hello"

let type: Type = something.getType()
// `type` is `Type<String>()`
```

This method returns the **concrete run-time type** of the object, **not** the static type.

```cadence
// Declare a variable named `something` that has the *static* type `AnyResource`
// and has a resource of type `Collectible`
//
let something: @AnyResource <- create Collectible()

// The resource's concrete run-time type is `Collectible`
//
let type: Type = something.getType()
// `type` is `Type<@Collectible>()`
```

### Constructing a Run-time Type

Run-time types can also be constructed from type identifier strings using built-in constructor functions.

```cadence
fun CompositeType(_ identifier: String): Type?
fun InterfaceType(_ identifier: String): Type?
fun RestrictedType(identifier: String?, restrictions: [String]): Type?
```

Given a type identifier (as well as a list of identifiers for restricting interfaces
in the case of `RestrictedType`), these functions will look up nominal types and
produce their run-time equivalents. If the provided identifiers do not correspond
to any types, or (in the case of `RestrictedType`) the provided combination of
identifiers would not type-check statically, these functions will produce `nil`.

```cadence
struct Test {}
struct interface I {}
let type: Type = CompositeType("A.0000000000000001.Test")
// `type` is `Type<Test>`

let type2: Type = RestrictedType(
    identifier: type.identifier,
    restrictions: ["A.0000000000000001.I"]
)
// `type2` is `Type<Test{I}>`
```

Other built-in functions will construct compound types from other run-types.

```cadence
fun OptionalType(_ type: Type): Type
fun VariableSizedArrayType(_ type: Type): Type
fun ConstantSizedArrayType(type: Type, size: Int): Type
fun FunctionType(parameters: [Type], return: Type): Type
// returns `nil` if `key` is not valid dictionary key type
fun DictionaryType(key: Type, value: Type): Type?
// returns `nil` if `type` is not a reference type
fun CapabilityType(_ type: Type): Type?
fun ReferenceType(authorized: bool, type: Type): Type
```

### Asserting the Type of a Value

The method `fun isInstance(_ type: Type): Bool` can be used to check if a value has a certain type,
using the concrete run-time type, and considering subtyping rules,

```cadence
// Declare a variable named `collectible` that has the *static* type `Collectible`
// and has a resource of type `Collectible`
//
let collectible: @Collectible <- create Collectible()

// The resource is an instance of type `Collectible`,
// because the concrete run-time type is `Collectible`
//
collectible.isInstance(Type<@Collectible>())  // is `true`

// The resource is an instance of type `AnyResource`,
// because the concrete run-time type `Collectible` is a subtype of `AnyResource`
//
collectible.isInstance(Type<@AnyResource>())  // is `true`

// The resource is *not* an instance of type `String`,
// because the concrete run-time type `Collectible` is *not* a subtype of `String`
//
collectible.isInstance(Type<String>())  // is `false`
```

Note that the **concrete run-time type** of the object is used, **not** the static type.

```cadence
// Declare a variable named `something` that has the *static* type `AnyResource`
// and has a resource of type `Collectible`
//
let something: @AnyResource <- create Collectible()

// The resource is an instance of type `Collectible`,
// because the concrete run-time type is `Collectible`
//
something.isInstance(Type<@Collectible>())  // is `true`

// The resource is an instance of type `AnyResource`,
// because the concrete run-time type `Collectible` is a subtype of `AnyResource`
//
something.isInstance(Type<@AnyResource>())  // is `true`

// The resource is *not* an instance of type `String`,
// because the concrete run-time type `Collectible` is *not* a subtype of `String`
//
something.isInstance(Type<String>())  // is `false`
```

For example, this allows implementing a marketplace sale resource:

```cadence
pub resource SimpleSale {

    /// The resource for sale.
    /// Once the resource is sold, the field becomes `nil`.
    ///
    pub var resourceForSale: @AnyResource?

    /// The price that is wanted for the purchase of the resource.
    ///
    pub let priceForResource: UFix64

    /// The type of currency that is required for the purchase.
    ///
    pub let requiredCurrency: Type
    pub let paymentReceiver: Capability<&{FungibleToken.Receiver}>

    /// `paymentReceiver` is the capability that will be borrowed
    /// once a valid purchase is made.
    /// It is expected to target a resource that allows depositing the paid amount
    /// (a vault which has the type in `requiredCurrency`).
    ///
    init(
        resourceForSale: @AnyResource,
        priceForResource: UFix64,
        requiredCurrency: Type,
        paymentReceiver: Capability<&{FungibleToken.Receiver}>
    ) {
        self.resourceForSale <- resourceForSale
        self.priceForResource = priceForResource
        self.requiredCurrency = requiredCurrency
        self.paymentReceiver = paymentReceiver
    }

    destroy() {
        // When this sale resource is destroyed,
        // also destroy the resource for sale.
        // Another option could be to transfer it back to the seller.
        destroy self.resourceForSale
    }

    /// buyObject allows purchasing the resource for sale by providing
    /// the required funds.
    /// If the purchase succeeds, the resource for sale is returned.
    /// If the purchase fails, the program aborts.
    ///
    pub fun buyObject(with funds: @FungibleToken.Vault): @AnyResource {
        pre {
            // Ensure the resource is still up for sale
            self.resourceForSale != nil: "The resource has already been sold"
            // Ensure the paid funds have the right amount
            funds.balance >= self.priceForResource: "Payment has insufficient amount"
            // Ensure the paid currency is correct
            funds.isInstance(self.requiredCurrency): "Incorrect payment currency"
        }

        // Transfer the paid funds to the payment receiver
        // by borrowing the payment receiver capability of this sale resource
        // and depositing the payment into it

        let receiver = self.paymentReceiver.borrow()
            ?? panic("failed to borrow payment receiver capability")

        receiver.deposit(from: <-funds)
        let resourceForSale <- self.resourceForSale <- nil
        return <-resourceForSale
    }
}
```

---
title: Scope
sidebar_position: 8
---

Every function and block (`{` ... `}`) introduces a new scope for declarations.
Each function and block can refer to declarations in its scope or any of the outer scopes.

```cadence
let x = 10

fun f(): Int {
    let y = 10
    return x + y
}

f()  // is `20`

// Invalid: the identifier `y` is not in scope.
//
y
```

```cadence
fun doubleAndAddOne(_ n: Int): Int {
    fun double(_ x: Int) {
        return x * 2
    }
    return double(n) + 1
}

// Invalid: the identifier `double` is not in scope.
//
double(1)
```

Each scope can introduce new declarations, i.e., the outer declaration is shadowed.

```cadence
let x = 2

fun test(): Int {
    let x = 3
    return x
}

test()  // is `3`
```

Scope is lexical, not dynamic.

```cadence
let x = 10

fun f(): Int {
   return x
}

fun g(): Int {
   let x = 20
   return f()
}

g()  // is `10`, not `20`
```

Declarations are **not** moved to the top of the enclosing function (hoisted).

```cadence
let x = 2

fun f(): Int {
    if x == 0 {
        let x = 3
        return x
    }
    return x
}
f()  // is `2`
```

---
title: Syntax
sidebar_position: 1
---

## Comments

Comments can be used to document code.
A comment is text that is not executed.

*Single-line comments* start with two slashes (`//`).
These comments can go on a line by themselves or they can go directly after a line of code.

```cadence
// This is a comment on a single line.
// Another comment line that is not executed.

let x = 1  // Here is another comment after a line of code.
```

*Multi-line comments* start with a slash and an asterisk (`/*`)
and end with an asterisk and a slash (`*/`):

```cadence
/* This is a comment which
spans multiple lines. */
```

Comments may be nested.

```cadence
/* /* this */ is a valid comment */
```

Multi-line comments are balanced.

```cadence
/* this is a // comment up to here */ this is not part of the comment */
```

### Documentation Comments
Documentation comments (also known as "doc-strings" or "doc-comment") are a special set of comments that can be
processed by tools, for example to generate human-readable documentation, or provide documentation in an IDE.

Doc-comments either start with three slashes (`///`) on each line,
or are surrounded by `/**` and `**/`.

```cadence
/// This is a documentation comment for `x`.
/// It spans multiple lines.

let x = 1
```

```cadence
/**
  This is a documentation comment
  which also spans multiple lines.
**/
```

## Names

Names may start with any upper or lowercase letter (A-Z, a-z)
or an underscore (`_`).
This may be followed by zero or more upper and lower case letters,
underscores, and numbers (0-9).
Names may not begin with a number.

```cadence
// Valid: title-case
//
PersonID

// Valid: with underscore
//
token_name

// Valid: leading underscore and characters
//
_balance

// Valid: leading underscore and numbers
_8264

// Valid: characters and number
//
account2

// Invalid: leading number
//
1something

// Invalid: invalid character #
_#1

// Invalid: various invalid characters
//
!@#$%^&*
```

### Conventions

By convention, variables, constants, and functions have lowercase names;
and types have title-case names.

## Semicolons

Semicolons (;) are used as separators between declarations and statements.
A semicolon can be placed after any declaration and statement,
but can be omitted between declarations and if only one statement appears on the line.

Semicolons must be used to separate multiple statements if they appear on the same line.

```cadence
// Declare a constant, without a semicolon.
//
let a = 1

// Declare a variable, with a semicolon.
//
var b = 2;

// Declare a constant and a variable on a single line, separated by semicolons.
//
let d = 1; var e = 2
```
---
title: Transactions
sidebar_position: 26
---

Transactions are objects that are signed by one or more [accounts](./accounts.mdx)
and are sent to the chain to interact with it.

Transactions are structured as such:

First, the transaction can import any number of types from external accounts
using the import syntax.

```cadence
import FungibleToken from 0x01
```

The body is declared using the `transaction` keyword and its contents
are contained in curly braces.

Next is the body of the transaction,
which first contains local variable declarations that are valid
throughout the whole of the transaction.

```cadence
transaction {
    // transaction contents
    let localVar: Int

    ...
}
```

Then, four optional main phases:
Preparation, preconditions, execution, and postconditions, in that order.
The preparation and execution phases are blocks of code that execute sequentially.

The following empty Cadence transaction contains no logic,
but demonstrates the syntax for each phase, in the order these phases will be executed:

```cadence
transaction {
    prepare(signer1: AuthAccount, signer2: AuthAccount) {
        // ...
    }

    pre {
        // ...
    }

    execute {
        // ...
    }

    post {
        // ...
    }
}
```

Although optional, each phase serves a specific purpose when executing a transaction
and it is recommended that developers use these phases when creating their transactions.
The following will detail the purpose of and how to use each phase.

## Transaction Parameters

Transactions may declare parameters.
Transaction parameters are declared like function parameters.
The arguments for the transaction are passed in the sent transaction.

Transaction parameters are accessible in all phases.

```cadence
// Declare a transaction which has one parameter named `amount`
// that has the type `UFix64`
//
transaction(amount: UFix64) {

}
```

## Prepare phase

The `prepare` phase is used when access to the private `AuthAccount` object
of **signing accounts** is required for your transaction.

Direct access to signing accounts is **only possible inside the `prepare` phase**.

For each signer of the transaction the signing account is passed as an argument to the `prepare` phase.
For example, if the transaction has three signers,
the prepare **must** have three parameters of type `AuthAccount`.

```cadence
 prepare(signer1: AuthAccount) {
      // ...
 }
```

As a best practice, only use the `prepare` phase to define and execute logic that requires access
to the `AuthAccount` objects of signing accounts,
and *move all other logic elsewhere*.
Modifications to accounts can have significant implications,
so keep this phase clear of unrelated logic to ensure users of your contract are able to easily read
and understand logic related to their private account objects.

The prepare phase serves a similar purpose as the initializer of a contract/resource/structure.

For example, if a transaction performs a token transfer, put the withdrawal in the `prepare` phase,
as it requires access to the account storage, but perform the deposit in the `execute` phase.

`AuthAccount` objects have the permissions
to read from and write to the `/storage/` and `/private/` areas
of the account, which cannot be directly accessed anywhere else.
They also have the permission to create and delete capabilities that
use these areas.

## Pre Phase

The `pre` phase is executed after the `prepare` phase, and is used for checking
if explicit conditions hold before executing the remainder of the transaction.
A common example would be checking requisite balances before transferring tokens between accounts.

```cadence
pre {
    sendingAccount.balance > 0
}
```

If the `pre` phase throws an error, or does not return `true` the remainder of the transaction
is not executed and it will be completely reverted.

## Execute Phase

The `execute` phase does exactly what it says, it executes the main logic of the transaction.
This phase is optional, but it is a best practice to add your main transaction logic in the section,
so it is explicit.

```cadence
execute {
    // Invalid: Cannot access the authorized account object,
    // as `account1` is not in scope
    let resource <- account1.load<@Resource>(from: /storage/resource)
    destroy resource

    // Valid: Can access any account's public Account object
    let publicAccount = getAccount(0x03)
}
```

You **may not** access private `AuthAccount` objects in the `execute` phase,
but you may get an account's `PublicAccount` object,
which allows reading and calling methods on objects
that an account has published in the public domain of its account (resources, contract methods, etc.).

## Post Phase

Statements inside of the `post` phase are used
to verify that your transaction logic has been executed properly.
It contains zero or more condition checks.

For example, a transfer transaction might ensure that the final balance has a certain value,
or e.g. it was incremented by a specific amount.

```cadence
post {
    result.balance == 30: "Balance after transaction is incorrect!"
}
```

If any of the condition checks result in `false`, the transaction will fail and be completely reverted.

Only condition checks are allowed in this section.
No actual computation or modification of values is allowed.

**A Note about `pre` and `post` Phases**

Another function of the `pre` and `post` phases is to help provide information
about how the effects of a transaction on the accounts and resources involved.
This is essential because users may want to verify what a transaction does before submitting it.
`pre` and `post` phases provide a way to introspect transactions before they are executed.

For example, in the future the phases could be analyzed and interpreted to the user
in the software they are using,
e.g. "this transaction will transfer 30 tokens from A to B.
The balance of A will decrease by 30 tokens and the balance of B will increase by 30 tokens."

## Summary

Cadence transactions use phases to make the transaction's code / intent more readable
and to provide a way for developer to separate potentially 'unsafe' account
modifying code from regular transaction logic,
as well as provide a way to check for error prior / after transaction execution,
and abort the transaction if any are found.

The following is a brief summary of how to use the `prepare`, `pre`, `execute`,
and `post` phases in a Cadence transaction.

```cadence
transaction {
    prepare(signer1: AuthAccount) {
        // Access signing accounts for this transaction.
        //
        // Avoid logic that does not need access to signing accounts.
        //
        // Signing accounts can't be accessed anywhere else in the transaction.
    }

    pre {
        // Define conditions that must be true
        // for this transaction to execute.
    }

    execute {
        // The main transaction logic goes here, but you can access
        // any public information or resources published by any account.
    }

    post {
        // Define the expected state of things
        // as they should be after the transaction executed.
        //
        // Also used to provide information about what changes
        // this transaction will make to accounts in this transaction.
    }
}
```
---
title: Type Annotations
sidebar_position: 3
---

When declaring a constant or variable,
an optional *type annotation* can be provided,
to make it explicit what type the declaration has.

If no type annotation is provided, the type of the declaration is
[inferred from the initial value](./type-inference.md).

For function parameters a type annotation must be provided.

```cadence
// Declare a variable named `boolVarWithAnnotation`, which has an explicit type annotation.
//
// `Bool` is the type of booleans.
//
var boolVarWithAnnotation: Bool = false

// Declare a constant named `integerWithoutAnnotation`, which has no type annotation
// and for which the type is inferred to be `Int`, the type of arbitrary-precision integers.
//
// This is based on the initial value which is an integer literal.
// Integer literals are always inferred to be of type `Int`.
//
let integerWithoutAnnotation = 1

// Declare a constant named `smallIntegerWithAnnotation`, which has an explicit type annotation.
// Because of the explicit type annotation, the type is not inferred.
// This declaration is valid because the integer literal `1` fits into the range of the type `Int8`,
// the type of 8-bit signed integers.
//
let smallIntegerWithAnnotation: Int8 = 1
```

If a type annotation is provided, the initial value must be of this type.
All new values assigned to variables must match its type.
This type safety is explained in more detail in a [separate section](./type-safety.md).

```cadence
// Invalid: declare a variable with an explicit type `Bool`,
// but the initial value has type `Int`.
//
let booleanConstant: Bool = 1

// Declare a variable that has the inferred type `Bool`.
//
var booleanVariable = false

// Invalid: assign a value with type `Int` to a variable which has the inferred type `Bool`.
//
booleanVariable = 1
```
---
title: Type Inference
sidebar_position: 10
---

If a variable or constant declaration is not annotated explicitly with a type,
the declaration's type is inferred from the initial value.

### Basic Literals
Decimal integer literals and hex literals are inferred to type `Int`.

```cadence
let a = 1
// `a` has type `Int`

let b = -45
// `b` has type `Int`

let c = 0x02
// `c` has type `Int`
```

Unsigned fixed-point literals are inferred to type `UFix64`.
Signed fixed-point literals are inferred to type `Fix64`.

```cadence
let a = 1.2
// `a` has type `UFix64`

let b = -1.2
// `b` has type `Fix64`
```

Similarly, for other basic literals, the types are inferred in the following manner:

| Literal Kind      | Example           | Inferred Type (x) |
|:-----------------:|:-----------------:|:-----------------:|
| String literal    | `let x = "hello"` |  String           |
| Boolean literal   | `let x = true`    |  Bool             |
| Nil literal       | `let x = nil`     |  Never?           |


### Array Literals
Array literals are inferred based on the elements of the literal, and to be variable-size.
The inferred element type is the _least common super-type_ of all elements.

```cadence
let integers = [1, 2]
// `integers` has type `[Int]`

let int8Array = [Int8(1), Int8(2)]
// `int8Array` has type `[Int8]`

let mixedIntegers = [UInt(65), 6, 275, Int128(13423)]
// `mixedIntegers` has type `[Integer]`

let nilableIntegers = [1, nil, 2, 3, nil]
// `nilableIntegers` has type `[Int?]`

let mixed = [1, true, 2, false]
// `mixed` has type `[AnyStruct]`
```

### Dictionary Literals
Dictionary literals are inferred based on the keys and values of the literal.
The inferred type of keys and values is the _least common super-type_ of all keys and values, respectively.

```cadence
let booleans = {
    1: true,
    2: false
}
// `booleans` has type `{Int: Bool}`

let mixed = {
    Int8(1): true,
    Int64(2): "hello"
}
// `mixed` has type `{Integer: AnyStruct}`

// Invalid: mixed keys
//
let invalidMixed = {
    1: true,
    false: 2
}
// The least common super-type of the keys is `AnyStruct`.
// But it is not a valid type for dictionary keys.
```

### Ternary Expression
Ternary expression type is inferred  to be the least common super-type of the second and third operands.
```cadence
let a = true ? 1 : 2
// `a` has type `Int`

let b = true ? 1 : nil
// `b` has type `Int?`

let c = true ? 5 : (false ? "hello" : nil)
// `c` has type `AnyStruct`
```

### Functions
Functions are inferred based on the parameter types and the return type.

```cadence
let add = (a: Int8, b: Int8): Int {
    return a + b
}

// `add` has type `((Int8, Int8): Int)`
```

Type inference is performed for each expression / statement, and not across statements.

## Ambiguities
There are cases where types cannot be inferred.
In these cases explicit type annotations are required.

```cadence
// Invalid: not possible to infer type based on array literal's elements.
//
let array = []

// Instead, specify the array type and the concrete element type, e.g. `Int`.
//
let array: [Int] = []

// Or, use a simple-cast to annotate the expression with a type.
let array = [] as [Int]
```

```cadence
// Invalid: not possible to infer type based on dictionary literal's keys and values.
//
let dictionary = {}

// Instead, specify the dictionary type and the concrete key
// and value types, e.g. `String` and `Int`.
//
let dictionary: {String: Int} = {}

// Or, use a simple-cast to annotate the expression with a type.
let dictionary = {} as {String: Int}
```
---
title: Type Safety
sidebar_position: 9
---

The Cadence programming language is a *type-safe* language.

When assigning a new value to a variable, the value must be the same type as the variable.
For example, if a variable has type `Bool`,
it can *only* be assigned a value that has type `Bool`,
and not for example a value that has type `Int`.

```cadence
// Declare a variable that has type `Bool`.
var a = true

// Invalid: cannot assign a value that has type `Int` to a variable which has type `Bool`.
//
a = 0
```

When passing arguments to a function,
the types of the values must match the function parameters' types.
For example, if a function expects an argument that has type `Bool`,
*only* a value that has type `Bool` can be provided,
and not for example a value which has type `Int`.

```cadence
fun nand(_ a: Bool, _ b: Bool): Bool {
    return !(a && b)
}

nand(false, false)  // is `true`

// Invalid: The arguments of the function calls are integers and have type `Int`,
// but the function expects parameters booleans (type `Bool`).
//
nand(0, 0)
```

Types are **not** automatically converted.
For example, an integer is not automatically converted to a boolean,
nor is an `Int32` automatically converted to an `Int8`,
nor is an optional integer `Int?`
automatically converted to a non-optional integer `Int`,
or vice-versa.

```cadence
fun add(_ a: Int8, _ b: Int8): Int8 {
    return a + b
}

// The arguments are not declared with a specific type, but they are inferred
// to be `Int8` since the parameter types of the function `add` are `Int8`.
add(1, 2)  // is `3`

// Declare two constants which have type `Int32`.
//
let a: Int32 = 3_000_000_000
let b: Int32 = 3_000_000_000

// Invalid: cannot pass arguments which have type `Int32` to parameters which have type `Int8`.
//
add(a, b)
```
---
title: Values and Types
sidebar_position: 4
---

Values are objects, like for example booleans, integers, or arrays.
Values are typed.

## Booleans

The two boolean values `true` and `false` have the type `Bool`.

## Numeric Literals

Numbers can be written in various bases. Numbers are assumed to be decimal by default.
Non-decimal literals have a specific prefix.

| Numeral system  | Prefix | Characters                                                            |
| :-------------- | :----- | :-------------------------------------------------------------------- |
| **Decimal**     | _None_ | one or more numbers (`0` to `9`)                                      |
| **Binary**      | `0b`   | one or more zeros or ones (`0` or `1`)                                |
| **Octal**       | `0o`   | one or more numbers in the range `0` to `7`                           |
| **Hexadecimal** | `0x`   | one or more numbers, or characters `a` to `f`, lowercase or uppercase |

```cadence
// A decimal number
//
1234567890  // is `1234567890`

// A binary number
//
0b101010  // is `42`

// An octal number
//
0o12345670  // is `2739128`

// A hexadecimal number
//
0x1234567890ABCabc  // is `1311768467294898876`

// Invalid: unsupported prefix 0z
//
0z0

// A decimal number with leading zeros. Not an octal number!
00123 // is `123`

// A binary number with several trailing zeros.
0b001000  // is `8`
```

Decimal numbers may contain underscores (`_`) to logically separate components.

```cadence
let largeNumber = 1_000_000

// Invalid: Value is not a number literal, but a variable.
let notNumber = _123
```

Underscores are allowed for all numeral systems.

```cadence
let binaryNumber = 0b10_11_01
```

## Integers

Integers are numbers without a fractional part.
They are either _signed_ (positive, zero, or negative)
or _unsigned_ (positive or zero).

Signed integer types which check for overflow and underflow have an `Int` prefix
and can represent values in the following ranges:

- **`Int8`**: -2^7 through 2^7 − 1 (-128 through 127)
- **`Int16`**: -2^15 through 2^15 − 1 (-32768 through 32767)
- **`Int32`**: -2^31 through 2^31 − 1 (-2147483648 through 2147483647)
- **`Int64`**: -2^63 through 2^63 − 1 (-9223372036854775808 through 9223372036854775807)
- **`Int128`**: -2^127 through 2^127 − 1
- **`Int256`**: -2^255 through 2^255 − 1

Unsigned integer types which check for overflow and underflow have a `UInt` prefix
and can represent values in the following ranges:

- **`UInt8`**: 0 through 2^8 − 1 (255)
- **`UInt16`**: 0 through 2^16 − 1 (65535)
- **`UInt32`**: 0 through 2^32 − 1 (4294967295)
- **`UInt64`**: 0 through 2^64 − 1 (18446744073709551615)
- **`UInt128`**: 0 through 2^128 − 1
- **`UInt256`**: 0 through 2^256 − 1

Unsigned integer types which do **not** check for overflow and underflow,
i.e. wrap around, have the `Word` prefix
and can represent values in the following ranges:

- **`Word8`**: 0 through 2^8 − 1 (255)
- **`Word16`**: 0 through 2^16 − 1 (65535)
- **`Word32`**: 0 through 2^32 − 1 (4294967295)
- **`Word64`**: 0 through 2^64 − 1 (18446744073709551615)

The types are independent types, i.e. not subtypes of each other.

See the section about [arithmetic operators](./operators.md#arithmetic-operators) for further
information about the behavior of the different integer types.

```cadence
// Declare a constant that has type `UInt8` and the value 10.
let smallNumber: UInt8 = 10
```

```cadence
// Invalid: negative literal cannot be used as an unsigned integer
//
let invalidNumber: UInt8 = -10
```

In addition, the arbitrary precision integer type `Int` is provided.

```cadence
let veryLargeNumber: Int = 10000000000000000000000000000000
```

Integer literals are [inferred](./type-inference.md) to have type `Int`,
or if the literal occurs in a position that expects an explicit type,
e.g. in a variable declaration with an explicit type annotation.

```cadence
let someNumber = 123

// `someNumber` has type `Int`
```

Negative integers are encoded in two's complement representation.

Integer types are not converted automatically. Types must be explicitly converted,
which can be done by calling the constructor of the type with the integer type.

```cadence
let x: Int8 = 1
let y: Int16 = 2

// Invalid: the types of the operands, `Int8` and `Int16` are incompatible.
let z = x + y

// Explicitly convert `x` from `Int8` to `Int16`.
let a = Int16(x) + y

// `a` has type `Int16`

// Invalid: The integer literal is expected to be of type `Int8`,
// but the large integer literal does not fit in the range of `Int8`.
//
let b = x + 1000000000000000000000000
```

### Integer Functions

Integers have multiple built-in functions you can use.

-
    ```cadence
    fun toString(): String
    ```

    Returns the string representation of the integer.

    ```cadence
    let answer = 42

    answer.toString()  // is "42"
    ```

-
    ```cadence
    fun toBigEndianBytes(): [UInt8]
    ```

    Returns the byte array representation (`[UInt8]`) in big-endian order of the integer.

    ```cadence
    let largeNumber = 1234567890

    largeNumber.toBigEndianBytes()  // is `[73, 150, 2, 210]`
    ```

All integer types support the following functions:

-
    ```cadence
    fun T.fromString(_ input: String): T?
    ```

    Attempts to parse an integer value from a base-10 encoded string, returning `nil` if the string is invalid.

    For a given integer `n` of type `T`, `T.fromString(n.toString())` is equivalent to wrapping `n` up in an [optional](#optionals).

    Strings are invalid if:
      - they contain non-digit characters
      - they don't fit in the target type

    For signed integer types like `Int64`, and `Int`, the string may optionally begin with `+` or `-` sign prefix.

    For unsigned integer types like `Word64`, `UInt64`, and `UInt`, sign prefices are not allowed.

    Examples:

    ```cadence
    let fortyTwo: Int64? = Int64.fromString("42") // ok

    let twenty: UInt? = UInt.fromString("20") // ok

    let nilWord: Word8? = Word8.fromString("1024") // nil, out of bounds

    let negTwenty: Int? = Int.fromString("-20") // ok
    ```

-
    ```cadence
    fun T.fromBigEndianBytes(_ bytes: [UInt8]): T?
    ```
    Attempts to parse an integer value from a byte array representation (`[UInt8]`) in big-endian order, returning `nil` if the input bytes are invalid.

    For a given integer `n` of type `T`, `T.fromBigEndianBytes(n.toBigEndianBytes())` is equivalent to wrapping `n` up in an [optional](#optionals).

    The bytes are invalid if:
      - length of the bytes array exceeds the number of bytes needed for the target type
      - they don't fit in the target type

    Examples:

    ```cadence
    let fortyTwo: UInt32? = UInt32.fromBigEndianBytes([42]) // ok

    let twenty: UInt? = UInt.fromBigEndianBytes([0, 0, 20]) // ok

    let nilWord: Word8? = Word8.fromBigEndianBytes("[0, 22, 0, 0, 0, 0, 0, 0, 0]") // nil, out of bounds

    let nilWord2: Word8? = Word8.fromBigEndianBytes("[0, 0]") // nil, size (2) exceeds number of bytes needed for Word8 (1)

    let negativeNumber: Int64? = Int64.fromBigEndianBytes([128, 0, 0, 0, 0, 0, 0, 1]) // ok -9223372036854775807
    ```

## Fixed-Point Numbers

<Callout type="info">

🚧 Status: Currently only the 64-bit wide `Fix64` and `UFix64` types are available.
More fixed-point number types will be added in a future release.

</Callout>

Fixed-point numbers are useful for representing fractional values.
They have a fixed number of digits after decimal point.

They are essentially integers which are scaled by a factor.
For example, the value 1.23 can be represented as 1230 with a scaling factor of 1/1000.
The scaling factor is the same for all values of the same type
and stays the same during calculations.

Fixed-point numbers in Cadence have a scaling factor with a power of 10, instead of a power of 2,
i.e. they are decimal, not binary.

Signed fixed-point number types have the prefix `Fix`,
have the following factors, and can represent values in the following ranges:

- **`Fix64`**: Factor 1/100,000,000; -92233720368.54775808 through 92233720368.54775807

Unsigned fixed-point number types have the prefix `UFix`,
have the following factors, and can represent values in the following ranges:

- **`UFix64`**: Factor 1/100,000,000; 0.0 through 184467440737.09551615

### Fixed-Point Number Functions

Fixed-Point numbers have multiple built-in functions you can use.

-
    ```cadence
    fun toString(): String
    ```

    Returns the string representation of the fixed-point number.

    ```cadence
    let fix = 1.23

    fix.toString()  // is "1.23000000"
    ```
-
    ```cadence
    fun toBigEndianBytes(): [UInt8]
    ```

    Returns the byte array representation (`[UInt8]`) in big-endian order of the fixed-point number.

    ```cadence
    let fix = 1.23

    fix.toBigEndianBytes()  // is `[0, 0, 0, 0, 7, 84, 212, 192]`
    ```

All fixed-point types support the following functions:

-
    ```cadence
    fun T.fromString(_ input: String): T?
    ```

    Attempts to parse a fixed-point value from a base-10 encoded string, returning `nil` if the string is invalid.

    For a given fixed-point numeral `n` of type `T`, `T.fromString(n.toString())` is equivalent to wrapping `n` up in an `optional`.

    Strings are invalid if:
      - they contain non-digit characters.
      - they don't fit in the target type.
      - they're missing a decimal or fractional component. For example, both "0." and ".1" are invalid strings, but "0.1" is accepted.

    For signed types like `Fix64`, the string may optionally begin with `+` or `-` sign prefix.

    For unsigned types like `UFix64`, sign prefices are not allowed.

    Examples:

    ```cadence
    let nil1: UFix64? = UFix64.fromString("0.") // nil, fractional part is required

    let nil2: UFix64? = UFix64.fromString(".1") // nil, decimal part is required

    let smol: UFix64? = UFix64.fromString("0.1") // ok

    let smolString: String = "-0.1"

    let nil3: UFix64? = UFix64.fromString(smolString) // nil, unsigned types don't allow a sign prefix

    let smolFix64: Fix64? = Fix64.fromString(smolString) // ok
    ```

-
    ```cadence
    fun T.fromBigEndianBytes(_ bytes: [UInt8]): T?
    ```
    Attempts to parse an integer value from a byte array representation (`[UInt8]`) in big-endian order, returning `nil` if the input bytes are invalid.

    For a given integer `n` of type `T`, `T.fromBigEndianBytes(n.toBigEndianBytes())` is equivalent to wrapping `n` up in an [optional](#optionals).

    The bytes are invalid if:
      - length of the bytes array exceeds the number of bytes needed for the target type
      - they don't fit in the target type

    Examples:

    ```cadence
    let fortyTwo: UFix64? = UFix64.fromBigEndianBytes([0, 0, 0, 0, 250, 86, 234, 0]) // ok, 42.0

    let nilWord: UFix64? = UFix64.fromBigEndianBytes("[100, 22, 0, 0, 0, 0, 0, 0, 0]") // nil, out of bounds

    let nilWord2: Fix64? = Fix64.fromBigEndianBytes("[0, 22, 0, 0, 0, 0, 0, 0, 0]") // // nil, size (9) exceeds number of bytes needed for Fix64 (8)

    let negativeNumber: Fix64? = Fix64.fromBigEndianBytes([255, 255, 255, 255, 250, 10, 31, 0]) // ok, -1
    ```

## Minimum and maximum values

The minimum and maximum values for all integer and fixed-point number types are available through the fields `min` and `max`.

For example:

```cadence
let max = UInt8.max
// `max` is 255, the maximum value of the type `UInt8`
```

```cadence
let max = UFix64.max
// `max` is 184467440737.09551615, the maximum value of the type `UFix64`
```

## Saturation Arithmetic

Integers and fixed-point numbers support saturation arithmetic:
Arithmetic operations, such as addition or multiplications, are saturating at the numeric bounds instead of overflowing.

If the result of an operation is greater than the maximum value of the operands' type, the maximum is returned.
If the result is lower than the minimum of the operands' type, the minimum is returned.

Saturating addition, subtraction, multiplication, and division are provided as functions with the prefix `saturating`:

- `Int8`, `Int16`, `Int32`, `Int64`, `Int128`, `Int256`, `Fix64`:

  - `saturatingAdd`
  - `saturatingSubtract`
  - `saturatingMultiply`
  - `saturatingDivide`

- `Int`:

  - none

- `UInt8`, `UInt16`, `UInt32`, `UInt64`, `UInt128`, `UInt256`, `UFix64`:

  - `saturatingAdd`
  - `saturatingSubtract`
  - `saturatingMultiply`

- `UInt`:
  - `saturatingSubtract`

```cadence
let a: UInt8 = 200
let b: UInt8 = 100
let result = a.saturatingAdd(b)
// `result` is 255, the maximum value of the type `UInt8`
```

## Floating-Point Numbers

There is **no** support for floating point numbers.

Smart Contracts are not intended to work with values with error margins
and therefore floating point arithmetic is not appropriate here.

Instead, consider using [fixed point numbers](#fixed-point-numbers).

## Addresses

The type `Address` represents an address.
Addresses are unsigned integers with a size of 64 bits (8 bytes).
Hexadecimal integer literals can be used to create address values.

```cadence
// Declare a constant that has type `Address`.
//
let someAddress: Address = 0x436164656E636521

// Invalid: Initial value is not compatible with type `Address`,
// it is not a number.
//
let notAnAddress: Address = ""

// Invalid: Initial value is not compatible with type `Address`.
// The integer literal is valid, however, it is larger than 64 bits.
//
let alsoNotAnAddress: Address = 0x436164656E63652146757265766572
```

Integer literals are not inferred to be an address.

```cadence
// Declare a number. Even though it happens to be a valid address,
// it is not inferred as it.
//
let aNumber = 0x436164656E636521

// `aNumber` has type `Int`
```

Address can also be created using a byte array or string.

```cadence
// Declare an address with hex representation as 0x436164656E636521.
let someAddress: Address = Address.fromBytes([67, 97, 100, 101, 110, 99, 101, 33])

// Invalid: Provided value is not compatible with type `Address`. The function panics.
let invalidAddress: Address = Address.fromBytes([12, 34, 56, 11, 22, 33, 44, 55, 66, 77, 88, 99, 111])

// Declare an address with the string representation as "0x436164656E636521".
let addressFromString: Address? = Address.fromString("0x436164656E636521")

// Invalid: Provided value does not have the "0x" prefix. Returns Nil
let addressFromStringWithoutPrefix: Address? = Address.fromString("436164656E636521")

// Invalid: Provided value is an invalid hex string. Return Nil.
let invalidAddressForInvalidHex: Address? = Address.fromString("0xZZZ")

// Invalid: Provided value is larger than 64 bits. Return Nil.
let invalidAddressForOverflow: Address? = Address.fromString("0x436164656E63652146757265766572")
```

### Address Functions

Addresses have multiple built-in functions you can use.

-
    ```cadence
    fun toString(): String
    ```

    Returns the string representation of the address.
    The result has a `0x` prefix and is zero-padded.

    ```cadence
    let someAddress: Address = 0x436164656E636521
    someAddress.toString()   // is "0x436164656E636521"

    let shortAddress: Address = 0x1
    shortAddress.toString()  // is "0x0000000000000001"
    ```

-
    ```cadence
    fun toBytes(): [UInt8]
    ```

    Returns the byte array representation (`[UInt8]`) of the address.

    ```cadence
    let someAddress: Address = 0x436164656E636521

    someAddress.toBytes()  // is `[67, 97, 100, 101, 110, 99, 101, 33]`
    ```

## AnyStruct and AnyResource

`AnyStruct` is the top type of all non-resource types,
i.e., all non-resource types are a subtype of it.

`AnyResource` is the top type of all resource types.

```cadence
// Declare a variable that has the type `AnyStruct`.
// Any non-resource typed value can be assigned to it, for example an integer,
// but not resource-typed values.
//
var someStruct: AnyStruct = 1

// Assign a value with a different non-resource type, `Bool`.
someStruct = true

// Declare a structure named `TestStruct`, create an instance of it,
// and assign it to the `AnyStruct`-typed variable
//
struct TestStruct {}

let testStruct = TestStruct()

someStruct = testStruct

// Declare a resource named `TestResource`

resource TestResource {}

// Declare a variable that has the type `AnyResource`.
// Any resource-typed value can be assigned to it,
// but not non-resource typed values.
//
var someResource: @AnyResource <- create TestResource()

// Invalid: Resource-typed values can not be assigned
// to `AnyStruct`-typed variables
//
someStruct <- create TestResource()

// Invalid: Non-resource typed values can not be assigned
// to `AnyResource`-typed variables
//
someResource = 1
```

However, using `AnyStruct` and `AnyResource` does not opt-out of type checking.
It is invalid to access fields and call functions on these types,
as they have no fields and functions.

```cadence
// Declare a variable that has the type `AnyStruct`.
// The initial value is an integer,
// but the variable still has the explicit type `AnyStruct`.
//
let a: AnyStruct = 1

// Invalid: Operator cannot be used for an `AnyStruct` value (`a`, left-hand side)
// and an `Int` value (`2`, right-hand side).
//
a + 2
```

`AnyStruct` and `AnyResource` may be used like other types,
for example, they may be the element type of [arrays](#arrays)
or be the element type of an [optional type](#optionals).

```cadence
// Declare a variable that has the type `[AnyStruct]`,
// i.e. an array of elements of any non-resource type.
//
let anyValues: [AnyStruct] = [1, "2", true]

// Declare a variable that has the type `AnyStruct?`,
// i.e. an optional type of any non-resource type.
//
var maybeSomething: AnyStruct? = 42

maybeSomething = "twenty-four"

maybeSomething = nil
```

`AnyStruct` is also the super-type of all non-resource optional types,
and `AnyResource` is the super-type of all resource optional types.

```cadence
let maybeInt: Int? = 1
let anything: AnyStruct = maybeInt
```

[Conditional downcasting](./operators.md#conditional-downcasting-operator-as) allows coercing
a value which has the type `AnyStruct` or `AnyResource` back to its original type.

## Optionals

Optionals are values which can represent the absence of a value. Optionals have two cases:
either there is a value, or there is nothing.

An optional type is declared using the `?` suffix for another type.
For example, `Int` is a non-optional integer, and `Int?` is an optional integer,
i.e. either nothing, or an integer.

The value representing nothing is `nil`.

```cadence
// Declare a constant which has an optional integer type,
// with nil as its initial value.
//
let a: Int? = nil

// Declare a constant which has an optional integer type,
// with 42 as its initial value.
//
let b: Int? = 42

// Invalid: `b` has type `Int?`, which does not support arithmetic.
b + 23

// Invalid: Declare a constant with a non-optional integer type `Int`,
// but the initial value is `nil`, which in this context has type `Int?`.
//
let x: Int = nil
```

Optionals can be created for any value, not just for literals.

```cadence
// Declare a constant which has a non-optional integer type,
// with 1 as its initial value.
//
let x = 1

// Declare a constant which has an optional integer type.
// An optional with the value of `x` is created.
//
let y: Int? = x

// Declare a variable which has an optional any type, i.e. the variable
// may be `nil`, or any other value.
// An optional with the value of `x` is created.
//
var z: AnyStruct? = x
```

A non-optional type is a subtype of its optional type.

```cadence
var a: Int? = nil
let b = 2
a = b

// `a` is `2`
```

Optional types may be contained in other types, for example [arrays](#arrays) or even optionals.

```cadence
// Declare a constant which has an array type of optional integers.
let xs: [Int?] = [1, nil, 2, nil]

// Declare a constant which has a double optional type.
//
let doubleOptional: Int?? = nil
```

See the [optional operators](./operators.md#optional-operators) section for information
on how to work with optionals.

## Never

`Never` is the bottom type, i.e., it is a subtype of all types.
There is no value that has type `Never`.
`Never` can be used as the return type for functions that never return normally.
For example, it is the return type of the function [`panic`](./built-in-functions.mdx#panic).

```cadence
// Declare a function named `crashAndBurn` which will never return,
// because it calls the function named `panic`, which never returns.
//
fun crashAndBurn(): Never {
    panic("An unrecoverable error occurred")
}

// Invalid: Declare a constant with a `Never` type, but the initial value is an integer.
//
let x: Never = 1

// Invalid: Declare a function which returns an invalid return value `nil`,
// which is not a value of type `Never`.
//
fun returnNever(): Never {
    return nil
}
```

## Strings and Characters

Strings are collections of characters.
Strings have the type `String`, and characters have the type `Character`.
Strings can be used to work with text in a Unicode-compliant way.
Strings are immutable.

String and character literals are enclosed in double quotation marks (`"`).

```cadence
let someString = "Hello, world!"
```

String literals may contain escape sequences. An escape sequence starts with a backslash (`\`):

- `\0`: Null character
- `\\`: Backslash
- `\t`: Horizontal tab
- `\n`: Line feed
- `\r`: Carriage return
- `\"`: Double quotation mark
- `\'`: Single quotation mark
- `\u`: A Unicode scalar value, written as `\u{x}`,
  where `x` is a 1–8 digit hexadecimal number
  which needs to be a valid Unicode scalar value,
  i.e., in the range 0 to 0xD7FF and 0xE000 to 0x10FFFF inclusive

```cadence
// Declare a constant which contains two lines of text
// (separated by the line feed character `\n`), and ends
// with a thumbs up emoji, which has code point U+1F44D (0x1F44D).
//
let thumbsUpText =
    "This is the first line.\nThis is the second line with an emoji: \u{1F44D}"
```

The type `Character` represents a single, human-readable character.
Characters are extended grapheme clusters,
which consist of one or more Unicode scalars.

For example, the single character `ü` can be represented
in several ways in Unicode.
First, it can be represented by a single Unicode scalar value `ü`
("LATIN SMALL LETTER U WITH DIAERESIS", code point U+00FC).
Second, the same single character can be represented
by two Unicode scalar values:
`u` ("LATIN SMALL LETTER U", code point U+0075),
and "COMBINING DIAERESIS" (code point U+0308).
The combining Unicode scalar value is applied to the scalar before it,
which turns a `u` into a `ü`.

Still, both variants represent the same human-readable character `ü`.

```cadence
let singleScalar: Character = "\u{FC}"
// `singleScalar` is `ü`
let twoScalars: Character = "\u{75}\u{308}"
// `twoScalars` is `ü`
```

Another example where multiple Unicode scalar values are rendered as a single,
human-readable character is a flag emoji.
These emojis consist of two "REGIONAL INDICATOR SYMBOL LETTER" Unicode scalar values.

```cadence
// Declare a constant for a string with a single character, the emoji
// for the Canadian flag, which consists of two Unicode scalar values:
// - REGIONAL INDICATOR SYMBOL LETTER C (U+1F1E8)
// - REGIONAL INDICATOR SYMBOL LETTER A (U+1F1E6)
//
let canadianFlag: Character = "\u{1F1E8}\u{1F1E6}"
// `canadianFlag` is `🇨🇦`
```

### String Fields and Functions

Strings have multiple built-in functions you can use:

-
    ```cadence
    let length: Int
    ```

    Returns the number of characters in the string as an integer.

    ```cadence
    let example = "hello"

    // Find the number of elements of the string.
    let length = example.length
    // `length` is `5`
    ```
-
    ```cadence
    let utf8: [UInt8]
    ```

    The byte array of the UTF-8 encoding

    ```cadence
    let flowers = "Flowers \u{1F490}"
    let bytes = flowers.utf8
    // `bytes` is `[70, 108, 111, 119, 101, 114, 115, 32, 240, 159, 146, 144]`
    ```

-
    ```cadence
    fun concat(_ other: String): String
    ```

    Concatenates the string `other` to the end of the original string,
    but does not modify the original string.
    This function creates a new string whose length is the sum of the lengths
    of the string the function is called on and the string given as a parameter.

    ```cadence
    let example = "hello"
    let new = "world"

    // Concatenate the new string onto the example string and return the new string.
    let helloWorld = example.concat(new)
    // `helloWorld` is now `"helloworld"`
    ```

-
    ```cadence
    fun slice(from: Int, upTo: Int): String
    ```

    Returns a string slice of the characters
    in the given string from start index `from` up to,
    but not including, the end index `upTo`.
    This function creates a new string whose length is `upTo - from`.
    It does not modify the original string.
    If either of the parameters are out of the bounds of the string,
    or the indices are invalid (`from > upTo`), then the function will fail.

    ```cadence
    let example = "helloworld"

    // Create a new slice of part of the original string.
    let slice = example.slice(from: 3, upTo: 6)
    // `slice` is now `"low"`

    // Run-time error: Out of bounds index, the program aborts.
    let outOfBounds = example.slice(from: 2, upTo: 10)

    // Run-time error: Invalid indices, the program aborts.
    let invalidIndices = example.slice(from: 2, upTo: 1)
    ```
-
    ```cadence
    fun decodeHex(): [UInt8]
    ```

    Returns an array containing the bytes represented by the given hexadecimal string.

    The given string must only contain hexadecimal characters and must have an even length.
    If the string is malformed, the program aborts

    ```cadence
    let example = "436164656e636521"

    example.decodeHex()  // is `[67, 97, 100, 101, 110, 99, 101, 33]`
    ```

-
    ```cadence
    fun toLower(): String
    ```

    Returns a string where all upper case letters are replaced with lowercase characters

    ```cadence
    let example = "Flowers"

    example.toLower()  // is `flowers`
    ```

The `String` type also provides the following functions:

-
    ```cadence
    fun String.encodeHex(_ data: [UInt8]): String
    ```

    Returns a hexadecimal string for the given byte array

    ```cadence
    let data = [1 as UInt8, 2, 3, 0xCA, 0xDE]

    String.encodeHex(data)  // is `"010203cade"`
    ```

`String`s are also indexable, returning a `Character` value.

```cadence
let str = "abc"
let c = str[0] // is the Character "a"
```

-
    ```cadence
    fun String.fromUTF8(_ input: [UInt8]): String?
    ```

    Attempts to convert a UTF-8 encoded byte array into a `String`. This function returns `nil` if the byte array contains invalid UTF-8,
    such as incomplete codepoint sequences or undefined graphemes.

    For a given string `s`, `String.fromUTF8(s.utf8)` is equivalent to wrapping `s` up in an [optional](#optionals).

### Character Fields and Functions

`Character` values can be converted into `String` values using the `toString` function:

-
    ```cadence
    fun toString(): String`
    ```

    Returns the string representation of the character.

    ```cadence
    let c: Character = "x"

    c.toString()  // is "x"
    ```

-
    ```cadence
    fun String.fromCharacters(_ characters: [Character]): String
    ```

    Builds a new `String` value from an array of `Character`s. Because `String`s are immutable, this operation makes a copy of the input array.

    ```cadence
    let rawUwU: [Character] = ["U", "w", "U"]
    let uwu: String = String.fromCharacters(rawUwU) // "UwU"
    ```

## Arrays

Arrays are mutable, ordered collections of values.
Arrays may contain a value multiple times.
Array literals start with an opening square bracket `[` and end with a closing square bracket `]`.

```cadence
// An empty array
//
[]

// An array with integers
//
[1, 2, 3]
```

### Array Types

Arrays either have a fixed size or are variably sized, i.e., elements can be added and removed.

Fixed-size array types have the form `[T; N]`, where `T` is the element type,
and `N` is the size of the array. `N` has to be statically known, meaning
that it needs to be an integer literal.
For example, a fixed-size array of 3 `Int8` elements has the type `[Int8; 3]`.

Variable-size array types have the form `[T]`, where `T` is the element type.
For example, the type `[Int16]` specifies a variable-size array of elements that have type `Int16`.

All values in an array must have a type which is a subtype of the array's element type (`T`).

It is important to understand that arrays are value types and are only ever copied
when used as an initial value for a constant or variable,
when assigning to a variable,
when used as function argument,
or when returned from a function call.

```cadence
let size = 2
// Invalid: Array-size must be an integer literal
let numbers: [Int; size] = []

// Declare a fixed-sized array of integers
// which always contains exactly two elements.
//
let array: [Int8; 2] = [1, 2]

// Declare a fixed-sized array of fixed-sized arrays of integers.
// The inner arrays always contain exactly three elements,
// the outer array always contains two elements.
//
let arrays: [[Int16; 3]; 2] = [
    [1, 2, 3],
    [4, 5, 6]
]

// Declare a variable length array of integers
var variableLengthArray: [Int] = []

// Mixing values with different types is possible
// by declaring the expected array type
// with the common supertype of all values.
//
let mixedValues: [AnyStruct] = ["some string", 42]
```

Array types are covariant in their element types.
For example, `[Int]` is a subtype of `[AnyStruct]`.
This is safe because arrays are value types and not reference types.

### Array Indexing

To get the element of an array at a specific index, the indexing syntax can be used:
The array is followed by an opening square bracket `[`, the indexing value,
and ends with a closing square bracket `]`.

Indexes start at 0 for the first element in the array.

Accessing an element which is out of bounds results in a fatal error at run-time
and aborts the program.

```cadence
// Declare an array of integers.
let numbers = [42, 23]

// Get the first number of the array.
//
numbers[0] // is `42`

// Get the second number of the array.
//
numbers[1] // is `23`

// Run-time error: Index 2 is out of bounds, the program aborts.
//
numbers[2]
```

```cadence
// Declare an array of arrays of integers, i.e. the type is `[[Int]]`.
let arrays = [[1, 2], [3, 4]]

// Get the first number of the second array.
//
arrays[1][0] // is `3`
```

To set an element of an array at a specific index, the indexing syntax can be used as well.

```cadence
// Declare an array of integers.
let numbers = [42, 23]

// Change the second number in the array.
//
// NOTE: The declaration `numbers` is constant, which means that
// the *name* is constant, not the *value* – the value, i.e. the array,
// is mutable and can be changed.
//
numbers[1] = 2

// `numbers` is `[42, 2]`
```

### Array Fields and Functions

Arrays have multiple built-in fields and functions
that can be used to get information about and manipulate the contents of the array.

The field `length`, and the functions `concat`, and `contains`
are available for both variable-sized and fixed-sized or variable-sized arrays.

-
    ```cadence
    let length: Int
    ```

    The number of elements in the array.

    ```cadence
    // Declare an array of integers.
    let numbers = [42, 23, 31, 12]

    // Find the number of elements of the array.
    let length = numbers.length

    // `length` is `4`
    ```


-
    ```cadence
    fun concat(_ array: T): T
    ```

    Concatenates the parameter `array` to the end
    of the array the function is called on,
    but does not modify that array.

    Both arrays must be the same type `T`.

    This function creates a new array whose length is the sum of the length of the array
    the function is called on and the length of the array given as the parameter.

    ```cadence
    // Declare two arrays of integers.
    let numbers = [42, 23, 31, 12]
    let moreNumbers = [11, 27]

    // Concatenate the array `moreNumbers` to the array `numbers`
    // and declare a new variable for the result.
    //
    let allNumbers = numbers.concat(moreNumbers)

    // `allNumbers` is `[42, 23, 31, 12, 11, 27]`
    // `numbers` is still `[42, 23, 31, 12]`
    // `moreNumbers` is still `[11, 27]`
    ```

-
    ```cadence
    fun contains(_ element: T): Bool
    ```

    Returns true if the given element of type `T` is in the array.

    ```cadence
    // Declare an array of integers.
    let numbers = [42, 23, 31, 12]

    // Check if the array contains 11.
    let containsEleven = numbers.contains(11)
    // `containsEleven` is `false`

    // Check if the array contains 12.
    let containsTwelve = numbers.contains(12)
    // `containsTwelve` is `true`

    // Invalid: Check if the array contains the string "Kitty".
    // This results in a type error, as the array only contains integers.
    //
    let containsKitty = numbers.contains("Kitty")
    ```

-
    ```cadence
    fun firstIndex(of: T): Int?
    ```

    Returns the index of the first element matching the given object in the array, nil if no match.
    Available if `T` is not resource-kinded and equatable.

    ```cadence
     // Declare an array of integers.
     let numbers = [42, 23, 31, 12]

     // Check if the array contains 31
     let index = numbers.firstIndex(of: 31)
     // `index` is 2

     // Check if the array contains 22
     let index = numbers.firstIndex(of: 22)
     // `index` is nil
    ```

-
    ```cadence
    fun slice(from: Int, upTo: Int): [T]
    ```

    Returns an array slice of the elements
    in the given array from start index `from` up to,
    but not including, the end index `upTo`.
    This function creates a new array whose length is `upTo - from`.
    It does not modify the original array.
    If either of the parameters are out of the bounds of the array,
    or the indices are invalid (`from > upTo`), then the function will fail.

    ```cadence
    let example = [1, 2, 3, 4]

    // Create a new slice of part of the original array.
    let slice = example.slice(from: 1, upTo: 3)
    // `slice` is now `[2, 3]`

    // Run-time error: Out of bounds index, the program aborts.
    let outOfBounds = example.slice(from: 2, upTo: 10)

    // Run-time error: Invalid indices, the program aborts.
    let invalidIndices = example.slice(from: 2, upTo: 1)
    ```

#### Variable-size Array Functions

The following functions can only be used on variable-sized arrays.
It is invalid to use one of these functions on a fixed-sized array.

-
    ```cadence
    fun append(_ element: T): Void
    ```

    Adds the new element `element` of type `T` to the end of the array.

    The new element must be the same type as all the other elements in the array.

    This function [mutates](./access-control.md) the array.

    ```cadence
    // Declare an array of integers.
    let numbers = [42, 23, 31, 12]

    // Add a new element to the array.
    numbers.append(20)
    // `numbers` is now `[42, 23, 31, 12, 20]`

    // Invalid: The parameter has the wrong type `String`.
    numbers.append("SneakyString")
    ```

-
    ```cadence
    fun appendAll(_ array: T): Void
    ```

    Adds all the elements from `array` to the end of the array
    the function is called on.

    Both arrays must be the same type `T`.

    This function [mutates](./access-control.md) the array.

    ```cadence
    // Declare an array of integers.
    let numbers = [42, 23]

    // Add new elements to the array.
    numbers.appendAll([31, 12, 20])
    // `numbers` is now `[42, 23, 31, 12, 20]`

    // Invalid: The parameter has the wrong type `[String]`.
    numbers.appendAll(["Sneaky", "String"])
    ```

-
    ```cadence
    fun insert(at: Int, _ element: T): Void
    ```

    Inserts the new element `element` of type `T`
    at the given `index` of the array.

    The new element must be of the same type as the other elements in the array.

    The `index` must be within the bounds of the array.
    If the index is outside the bounds, the program aborts.

    The existing element at the supplied index is not overwritten.

    All the elements after the new inserted element
    are shifted to the right by one.

    This function [mutates](./access-control.md) the array.

    ```cadence
    // Declare an array of integers.
    let numbers = [42, 23, 31, 12]

    // Insert a new element at position 1 of the array.
    numbers.insert(at: 1, 20)
    // `numbers` is now `[42, 20, 23, 31, 12]`

    // Run-time error: Out of bounds index, the program aborts.
    numbers.insert(at: 12, 39)
    ```
-
    ```cadence
    fun remove(at: Int): T
    ```

    Removes the element at the given `index` from the array and returns it.

    The `index` must be within the bounds of the array.
    If the index is outside the bounds, the program aborts.

    This function [mutates](./access-control.md) the array.

    ```cadence
    // Declare an array of integers.
    let numbers = [42, 23, 31]

    // Remove element at position 1 of the array.
    let twentyThree = numbers.remove(at: 1)
    // `numbers` is now `[42, 31]`
    // `twentyThree` is `23`

    // Run-time error: Out of bounds index, the program aborts.
    numbers.remove(at: 19)
    ```

-
    ```cadence
    fun removeFirst(): T
    ```

    Removes the first element from the array and returns it.

    The array must not be empty.
    If the array is empty, the program aborts.

    This function [mutates](./access-control.md) the array.

    ```cadence
    // Declare an array of integers.
    let numbers = [42, 23]

    // Remove the first element of the array.
    let fortytwo = numbers.removeFirst()
    // `numbers` is now `[23]`
    // `fortywo` is `42`

    // Remove the first element of the array.
    let twentyThree = numbers.removeFirst()
    // `numbers` is now `[]`
    // `twentyThree` is `23`

    // Run-time error: The array is empty, the program aborts.
    numbers.removeFirst()
    ```

-
    ```cadence
    fun removeLast(): T
    ```

    Removes the last element from the array and returns it.

    The array must not be empty.
    If the array is empty, the program aborts.

    This function [mutates](./access-control.md) the array.

    ```cadence
    // Declare an array of integers.
    let numbers = [42, 23]

    // Remove the last element of the array.
    let twentyThree = numbers.removeLast()
    // `numbers` is now `[42]`
    // `twentyThree` is `23`

    // Remove the last element of the array.
    let fortyTwo = numbers.removeLast()
    // `numbers` is now `[]`
    // `fortyTwo` is `42`

    // Run-time error: The array is empty, the program aborts.
    numbers.removeLast()
    ```

## Dictionaries

Dictionaries are mutable, unordered collections of key-value associations.
Dictionaries may contain a key only once
and may contain a value multiple times.

Dictionary literals start with an opening brace `{`
and end with a closing brace `}`.
Keys are separated from values by a colon,
and key-value associations are separated by commas.

```cadence
// An empty dictionary
//
{}

// A dictionary which associates integers with booleans
//
{
    1: true,
    2: false
}
```

### Dictionary Types

Dictionary types have the form `{K: V}`,
where `K` is the type of the key,
and `V` is the type of the value.
For example, a dictionary with `Int` keys and `Bool`
values has type `{Int: Bool}`.

In a dictionary, all keys must have a type that is a subtype of the dictionary's key type (`K`)
and all values must have a type that is a subtype of the dictionary's value type (`V`).

```cadence
// Declare a constant that has type `{Int: Bool}`,
// a dictionary mapping integers to booleans.
//
let booleans = {
    1: true,
    0: false
}

// Declare a constant that has type `{Bool: Int}`,
// a dictionary mapping booleans to integers.
//
let integers = {
    true: 1,
    false: 0
}

// Mixing keys with different types, and mixing values with different types,
// is possible by declaring the expected dictionary type with the common supertype
// of all keys, and the common supertype of all values.
//
let mixedValues: {String: AnyStruct} = {
    "a": 1,
    "b": true
}
```

Dictionary types are covariant in their key and value types.
For example, `{Int: String}` is a subtype of `{AnyStruct: String}`
and also a subtype of `{Int: AnyStruct}`.
This is safe because dictionaries are value types and not reference types.

### Dictionary Access

To get the value for a specific key from a dictionary,
the access syntax can be used:
The dictionary is followed by an opening square bracket `[`, the key,
and ends with a closing square bracket `]`.

Accessing a key returns an [optional](#optionals):
If the key is found in the dictionary, the value for the given key is returned,
and if the key is not found, `nil` is returned.

```cadence
// Declare a constant that has type `{Int: Bool}`,
// a dictionary mapping integers to booleans.
//
let booleans = {
    1: true,
    0: false
}

// The result of accessing a key has type `Bool?`.
//
booleans[1]  // is `true`
booleans[0]  // is `false`
booleans[2]  // is `nil`

// Invalid: Accessing a key which does not have type `Int`.
//
booleans["1"]
```

```cadence
// Declare a constant that has type `{Bool: Int}`,
// a dictionary mapping booleans to integers.
//
let integers = {
    true: 1,
    false: 0
}

// The result of accessing a key has type `Int?`
//
integers[true] // is `1`
integers[false] // is `0`
```

To set the value for a key of a dictionary,
the access syntax can be used as well.

```cadence
// Declare a constant that has type `{Int: Bool}`,
// a dictionary mapping booleans to integers.
//
let booleans = {
    1: true,
    0: false
}

// Assign new values for the keys `1` and `0`.
//
booleans[1] = false
booleans[0] = true
// `booleans` is `{1: false, 0: true}`
```

### Dictionary Fields and Functions


-
    ```cadence
    let length: Int
    ```

    The number of entries in the dictionary.

    ```cadence
    // Declare a dictionary mapping strings to integers.
    let numbers = {"fortyTwo": 42, "twentyThree": 23}

    // Find the number of entries of the dictionary.
    let length = numbers.length

    // `length` is `2`
    ```

-
    ```cadence
    fun insert(key: K, _ value: V): V?
    ```

    Inserts the given value of type `V` into the dictionary under the given `key` of type `K`.

    The inserted key must have the same type as the dictionary's key type, and the inserted value must have the same type as the dictionary's value type.

    Returns the previous value as an optional
    if the dictionary contained the key,
    otherwise `nil`.

    Updates the value if the dictionary already contained the key.

    This function [mutates](./access-control.md) the dictionary.

    ```cadence
    // Declare a dictionary mapping strings to integers.
    let numbers = {"twentyThree": 23}

    // Insert the key `"fortyTwo"` with the value `42` into the dictionary.
    // The key did not previously exist in the dictionary,
    // so the result is `nil`
    //
    let old = numbers.insert(key: "fortyTwo", 42)

    // `old` is `nil`
    // `numbers` is `{"twentyThree": 23, "fortyTwo": 42}`
    ```

-
    ```cadence
    fun remove(key: K): V?
    ```

    Removes the value for the given `key` of type `K` from the dictionary.

    Returns the value of type `V` as an optional
    if the dictionary contained the key,
    otherwise `nil`.

    This function [mutates](./access-control.md) the dictionary.

    ```cadence
    // Declare a dictionary mapping strings to integers.
    let numbers = {"fortyTwo": 42, "twentyThree": 23}

    // Remove the key `"fortyTwo"` from the dictionary.
    // The key exists in the dictionary,
    // so the value associated with the key is returned.
    //
    let fortyTwo = numbers.remove(key: "fortyTwo")

    // `fortyTwo` is `42`
    // `numbers` is `{"twentyThree": 23}`

    // Remove the key `"oneHundred"` from the dictionary.
    // The key does not exist in the dictionary, so `nil` is returned.
    //
    let oneHundred = numbers.remove(key: "oneHundred")

    // `oneHundred` is `nil`
    // `numbers` is `{"twentyThree": 23}`
    ```

-
    ```cadence
    let keys: [K]
    ```

    Returns an array of the keys of type `K` in the dictionary. This does not
    modify the dictionary, just returns a copy of the keys as an array.
    If the dictionary is empty, this returns an empty array. The ordering of the keys is undefined.

    ```cadence
    // Declare a dictionary mapping strings to integers.
    let numbers = {"fortyTwo": 42, "twentyThree": 23}

    // Find the keys of the dictionary.
    let keys = numbers.keys

    // `keys` has type `[String]` and is `["fortyTwo","twentyThree"]`
    ```

-
    ```cadence
    let values: [V]
    ```

    Returns an array of the values of type `V` in the dictionary. This does not
    modify the dictionary, just returns a copy of the values as an array.
    If the dictionary is empty, this returns an empty array.

    This field is not available if `V` is a resource type.

    ```cadence
    // Declare a dictionary mapping strings to integers.
    let numbers = {"fortyTwo": 42, "twentyThree": 23}

    // Find the values of the dictionary.
    let values = numbers.values

    // `values` has type [Int] and is `[42, 23]`
    ```

-
    ```cadence
    fun containsKey(key: K): Bool
    ```

    Returns true if the given key of type `K` is in the dictionary.

    ```cadence
    // Declare a dictionary mapping strings to integers.
    let numbers = {"fortyTwo": 42, "twentyThree": 23}

    // Check if the dictionary contains the key "twentyFive".
    let containsKeyTwentyFive = numbers.containsKey("twentyFive")
    // `containsKeyTwentyFive` is `false`

    // Check if the dictionary contains the key "fortyTwo".
    let containsKeyFortyTwo = numbers.containsKey("fortyTwo")
    // `containsKeyFortyTwo` is `true`

    // Invalid: Check if the dictionary contains the key 42.
    // This results in a type error, as the key type of the dictionary is `String`.
    //
    let containsKey42 = numbers.containsKey(42)
    ```

-
    ```cadence
    fun forEachKey(_ function: ((K): Bool)): Void
    ```

    Iterate through all the keys in the dictionary, exiting early if the passed function returns false.
    This is more efficient than calling `.keys` and iterating over the resulting array, since an intermediate allocation is avoided.
    The order of key iteration is undefined, similar to `.keys`.

    ```cadence
    // Take in a targetKey to look for, and a dictionary to iterate through.
    fun myContainsKey(targetKey: String, dictionary: {String: Int}) {
      // Declare an accumulator that we'll capture inside a closure.
      var found = false

      // At each step, `key` will be bound to another key from `dictionary`.
      dictionary.forEachKey(fun (key: String): Bool {
        found = key == targetKey

        // The returned boolean value, signals whether to continue iterating.
        // This allows for control flow during the iteration process:
        //  true = `continue`
        //  false = `break`
        return !found
      })

      return found
    }
    ```


### Dictionary Keys

Dictionary keys must be hashable and equatable.

Most of the built-in types, like booleans and integers,
are hashable and equatable, so can be used as keys in dictionaries.
