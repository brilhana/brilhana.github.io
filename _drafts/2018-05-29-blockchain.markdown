---
layout: post
title:  "Blockchain"
date:   2018-05-29 10:01:12
description: "Blockchain."
categories:
- blog
---

```julia
using SHA

# An individual block structure.
struct Block
    index::Int
    timestamp::DateTime
    data::String
    previous_hash::String
    hash::String
    """
    The constructor which will also create hash for the block.
    It will use 256 bit encryption for the blockchain's security needs.
    """
    function Block(index, timestamp, data, previous_hash)
        hash = sha2_256(string(index, timestamp, data, previous_hash))
        new(index, timestamp, data, previous_hash, bytes2hex(hash))
    end
end

# Add another block to the chain.
function next_block(tail_block::Block)
    new_index = tail_block.index + 1
    return Block(new_index, Dates.now(), string("This is block #", new_index), tail_block.hash)
end

# Create the special first block of the blockchain.
Blockchain = [Block(0, Dates.now(), "Genesis Block", "0")]

println("Genesis Block: 1")
println("Hash: ", Blockchain[1].hash)

# Size of the blockchain.
blockchain_limit = 10

# To make addition of blocks to the blockchain, a proof of work task is required. We'll keep it simple.
for tail = 1:blockchain_limit
    # Link the new block to the chain.
    append!(Blockchain, [next_block(blockchain[tail])])
    # The details of the block.
    println("Block: $(tail+1)")
    println("Hash: $(blockchain[tail+1].hash)")
end
```