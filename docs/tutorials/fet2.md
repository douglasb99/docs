# Create an FET-2 contract

In this tutorial we are going to implement a subset of the functionality of 
an FET-2 contract in `etch`. 

We will be using `UInt256` for identifiers and the `SHA256()` function to generate the identifiers of the initial tokens. 

We will need two
records: an `Address` stating which tokens an address holds and a token record keeping track of the owner of the token. 

For the first of these tasks, we will use the `State` object in `etch`, and for the second we will use `ShardedState`.


## Initialisation function

Assuming that we have defined an `owner` and a `total_supply`, the initialise function 
will do three things: 

1. Generate a list of token ids. 
2. Create a record of each token owner.
3. Create a record of the tokens that an owner has. 

The dual relationship makes lookups efficient, but comes at the price of twice the book keeping.

First, create the list of token ids:

``` c++
    // Genereating tokens
    var token_id = UInt256("hello world");
    
    for(i in 0:tokens.count())

        var hasher = SHA256();
        hasher.update(token_id);
        token_id = hasher.final();
        tokens[i] = token_id;

    endfor
```

Next, assign an owner:

``` c++
    // Assigning owner
    var owner_state = ShardedState< Address >("tokens.owner");

    for(i in 0:tokens.count())

        var tid = tokens[i];
        owner_state.set(toString(tid), owner);

    endfor
```

Finally, store the list of tokens on the creator's address:

``` c++
    var objects_state = State< Array< UInt256 > >(owner);

    // Storing the tokens on the owners address
    objects_state.set(tokens);
```

## Queries

In this section we will focus on the functions which implement a wallet overview and token details view, namely `balanceOf(owner: Address) : UInt256` and `ownerOf(token_id: UInt256) : Address`. 

Both of these functions are short and easy to implement. 

First, we make it possible to query the balance:

``` c++
@query
function balanceOf(owner: Address) : UInt256

    var objects_state = State< Array< UInt256 > >(owner);
    var tokens = objects_state.get( Array< UInt256 >(0) );
    var ret = UInt256( toUInt64(tokens.count()) );

    return ret;

endfunction
```

Next, we make it possible to query the token owner:

``` c++
@query
function ownerOf(token_id: UInt256) : Address

    var owner_state = ShardedState< Address >("tokens.owner");
    return owner_state.get(toString(token_id)); 

endfunction
```

With these two query functions, it is possible to implement an FET-2 wallet on top of the smart ledger. 

We can also make several optimisations for these functions. For instance, by storing the number of tokens separately, we would not have to deserialise the full array.


## Actions

The standard FET-2 contract has a number of different functions for transferring funds from one party to another. We will implement just one of these; they are all essentially variations of the same mechanism with more or less error checking built into them. 

Implement a single transfer function:

``` c++
function transferFrom(from: Address, to: Address, token_id: UInt256)

    if(!from.signedTx()) 
      panic("Invalid signature from owner.");
    endif

    var owner_state = ShardedState< Address >("tokens.owner");
    var owner = owner_state.get(toString(token_id));

    if(owner != from)
      panic("Owner does not actually own the token");
    endif

    var from_state   = State< Array< UInt256 > >(from);
    var from_objects = from_state.get( Array< UInt256 >(0) );
    var found = false;
    var position : Int32;


    for(i in 0:from_objects.count())
      var tid = from_objects[i];
      if(tid == token_id)
        if(found)
          panic("Contract broken - token is only supposed be represented once.");
        endif

        found = true;
        position = i;
        break;
      endif
    endfor

    if(!found)
      panic("Contract is fundamentally broken - owner has not been updated correctly");
    endif

    from_objects[position] = from_objects[from_objects.count() - 1];
    from_objects.popBack();

    var to_state   = State< Array< UInt256 > >(to);
    var to_objects = to_state.get( Array< UInt256 >(0) );
    to_objects.append(token_id);

    // updating sender
    from_state.set(from_objects);

    // Updating receiver
    to_state.set(to_objects);

    // Updating owner
    owner = to;
    owner_state.set(toString(token_id), owner);

endfunction
```

The above implementation requires the sender to sign the transaction, but can be extended to also requiring the receiver to sign as well. 

The full contract can be found <a href="https://github.com/fetchai/etch-examples/blob/master/03_erc721/contract.etch" target=_blank>here</a>.


<br/>