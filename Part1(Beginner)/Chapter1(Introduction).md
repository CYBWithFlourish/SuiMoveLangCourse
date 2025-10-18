# Chapter 1 — Introduction to [Move](https://sui.io/move) and [Sui](https://sui.io)

## Learning Objectives

By the end of this chapter, you should be able to:

*   Explain what the Move language is and why it was created.
*   Describe the core principles behind Move: resources, modules, abilities, and safety.
*   Understand what Sui is and how it extends Move.
*   Compare Sui Move with other blockchain languages (like Solidity).
*   Set up your mindset for thinking in “resources and ownership” instead of “balances and accounts.”

***

### 1.1 What Is Move?

Move is a smart contract programming language originally developed by Meta (Facebook) for the Diem blockchain project. Its primary goal: **safety, security, and verifiable control over digital assets.**

#### Core Design Philosophy

Move introduces **first-class resources**: a concept inspired by Rust’s ownership model.

| Concept | Description |
| :--- | :--- |
| **Resource** | A type that cannot be copied or accidentally destroyed. It represents something unique and valuable (like tokens, NFTs, or capabilities). |
| **Linear type system** | Move enforces rules: each resource can only exist once, must be moved explicitly, and can’t just “disappear” from memory. |
| **Modules** | Reusable containers (like smart contracts) that define how resources behave. |
| **Verifiability** | Strong static typing and bytecode verification before execution prevent many runtime attacks. |

**Example (simple Move concept)**

```move
module SuiMoveLangCourse::MyCoin {
    struct Coin has key, store {
        value: u64
    }

    public fun mint(amount: u64): Coin {
        Coin { value: amount }
    }

    public fun transfer(coin: Coin, recipient: address) {
        // move ownership of resource to recipient
    }
}
```

> ** Notice:**
> Unlike Solidity’s ERC20 model (which tracks balances in a mapping), Move uses actual resources that are owned, moved, or destroyed explicitly.

***

### 1.2 Why Move Was Built

The creators of Move wanted to solve these blockchain problems:

| Problem in traditional smart contracts | Move’s solution |
| :--- | :--- |
| Anyone can accidentally mint or burn tokens | Only authorized modules can modify resources |
| Assets can be duplicated or lost | Resources cannot be copied or dropped |
| Reentrancy attacks (like DAO hack) | Ownership and strict borrowing rules prevent them |
| Implicit access to global state | Explicit, modular, verified access |

Thus, Move is a **resource-oriented language** not object-oriented or purely functional. Everything revolves around who owns what, what can be done, and who is allowed.

***

### 1.3 What Is Sui?

Sui is a next-generation Layer-1 blockchain built by [Mysten Labs](https://www.mystenlabs.com/) (founded by former Meta engineers). It uses the Move language, but extends it with new ideas:

| Feature | Description |
| :--- | :--- |
| **Object-centric storage** | Every on-chain item is an `object` with a unique ID. |
| **Parallel execution** | Sui can process independent transactions simultaneously — huge performance advantage. |
| **Low-latency finality** | Most transactions finalize instantly (under 1 second). |
| **Native Move support** | Sui Move adds new abilities and APIs for object management. |

#### Example: Everything is an Object

In Sui, coins, NFTs, and even users’ assets are represented as **objects** — not just resource structs.

Each object:
*   Has a unique ID
*   Has an owner (address or shared)
*   Lives in storage until transferred or destroyed

***

### 1.4 How Sui Move Differs from Core Move

| Concept | Core Move | Sui Move |
| :--- | :--- | :--- |
| **Storage model** | Key-value storage | Object storage (each item = object) |
| **Transactions** | Call modules on accounts | Interact with owned/shared objects |
| **Execution** | Sequential | Parallel (when objects are disjoint) |
| **Ownership** | Implicit (address owns everything) | Explicit ownership per object |
| **Abilities** | `copy`, `drop`, `store`, `key` | Same, but with Sui object semantics |

**Example — Sui Move object definition:**

```move
module SuiMoveLangCourse::MyObject {
    struct Counter has key, store {
        id: UID, // Unique object ID
        value: u64,
    }

    public entry fun increment(counter: &mut Counter) {
        counter.value = counter.value + 1;
    }
}
```

> **UID** is a Sui-specific type that gives your object an identity on-chain.

***

### 1.5 Comparing Move and Solidity

| Feature | Move | Solidity |
| :--- | :--- | :--- |
| **Safety** | Compiler-enforced resource safety | Depends on developer discipline |
| **Parallel execution** | Supported in Sui | Sequential (EVM single-threaded) |
| **Ownership** | Explicit | Implicit via `mapping` |
| **Language type** | Strongly typed, verifiable bytecode | Dynamically interpreted at runtime |
| **Gas model** | Per-object / per-operation | Per-EVM instruction |
| **Upgrades** | Strict & explicit | Flexible but risky |

***

### 1.6 Thinking in Move (Mindset Shift)

**Instead of:**
> “Balances stored in a map under user addresses,”

**think:**
> “Each user owns a `Coin` resource that lives as an object in the ledger.”

**Instead of:**
> “Contracts can mint tokens arbitrarily,”

**think:**
> “Only a module with a specific capability can mint new `Coins`.”

***
# The Four Move Abilities

## What Are Abilities?

In Move, **abilities** are type-level permissions that describe what you can do with a value of a certain type.

They tell the compiler:
> “Is it safe to copy, drop, store, or use this type as a key?”

Every `struct` you define in Move must explicitly list the abilities it possesses. This system is one of Move’s core safety features — preventing dangerous behavior like duplicating or accidentally destroying assets.

***

## The Four Abilities

| Ability | Meaning | Analogy | Example Usage |
| :--- | :--- | :--- | :--- |
| **`copy`** | The value can be duplicated (copied into another variable). | Copying a number or string. | `u64`, `bool`, and other primitive types. |
| **`drop`** | The value can be discarded when no longer needed. | Dropping a variable at the end of a function. | Temporary data or integers. |
| **`store`** | The value can be stored inside another `struct` or in global storage. | Putting an item inside a box (`struct`). | Embedding types as fields of other types. |
| **`key`** | The value can exist in global storage and have a unique ID. | Putting a file in a global database accessible by ID. | `structs` representing on-chain objects (like NFTs, coins). |

### Key Notes

*   **Resources** (like tokens) usually don’t have `copy` or `drop` — this prevents duplication or accidental loss.
    ```move
    struct MyCoin has key, store {
        id: UID,
        value: u64,
    }
    ```
    ✅ Has `key` (can exist globally)  
    ✅ Has `store` (can be contained in other structs)  
    ❌ No `copy` or `drop` (so it can’t be cloned or thrown away)

*   **Value types** (like `u64`, `bool`, etc.) usually have all abilities.
    ```move
    struct Stats has copy, drop, store {
        score: u64,
        active: bool,
    }
    ```
    ✅ Safe to copy, drop, store.

*   The Move compiler enforces these at compile time. If you try to copy a non-`copy` type, you will get a compiler error.

***

## Example: Resource vs Value

```move
module SuiMoveLangCourse::Abilities {

    // This is a VALUE type — can be copied, dropped, and stored freely.
    struct AbilityConfig has copy, drop, store {
        version: u8,
    }

    // This is a RESOURCE type — represents something valuable and unique.
    struct MyCoin has key, store {
        id: UID,
        balance: u64,
    }

    public entry fun create_config(): AbilityConfig {
        AbilityConfig { version: 1 }
    }

    public entry fun mint_coin(): MyCoin {
        // In Sui, object::new() creates the UID for the object
        MyCoin { id: object::new(), balance: 100 }
    }
}
```

***
## Quick Recap Table

| Type | `copy` | `drop` | `store` | `key` | Description |
| :--- | :---: | :---: | :---: | :---: | :--- |
| `u64`, `bool` | ✅ | ✅ | ✅ | ❌ | Simple values |
| `Config` (value type) | ✅ | ✅ | ✅ | ❌ | Safe-to-copy struct |
| `Coin` (resource type)| ❌ | ❌ | ✅ | ✅ | Unique on-chain asset |

### 1.7 Chapter Exercise: Write Your First Concept Note

**Create a small note (README or journal entry):**

Explain in your own words:
1.  What is Move?
2.  Why is it different from Solidity or Rust?
3.  How does Sui improve Move?
4.  Describe how you imagine a “Coin” behaves in Sui (as an object).

**Optional:** Draw a simple diagram showing:

> Alice ─ owns ─► Coin(ObjectID: 0x1234)
> 
> Bob   ─ owns ─► Coin(ObjectID: 0x5678)

***

### 1.8 Checkpoint — Quick Quiz

Try answering these before moving on:

1.  What’s the main difference between a Move resource and a normal variable?
2.  What’s the purpose of the `UID` in Sui?
3.  Why can Sui process many transactions in parallel?
4.  What are the four Move abilities and what do they control?
