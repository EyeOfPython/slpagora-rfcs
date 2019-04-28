# Version 2 of the protocol

## Motivation

I made the first version of the SLP Agora protocol in a rather quick-and-dirty fashion in order to get it work.

Now that it's starting to gain traction, I think it's time re-engineer a protocol that can be used intuitively for all sorts of applications, focused on, but not limited to, trading. This RFC is meant as a living document collecting ideas, constrains and possible solutions.

## Current protocol

Currently, a trade consists of two transactions, created in this order:

1. SLP transaction containing the smart contract as output
2. EXCH transaction containing all the relevent metadata for spending the smart contract

The SLP transaction is created first, and looks like this:

Outputs:
1. vout 0: OP_RETURN: SLP SEND metadata, see https://github.com/jcramer/slp-specification/blob/master/slp-token-type-1.md. Sends some amount of the token to the smart contract.
    1. `SLP\x00` string, 4 bytes
    2. Token type: `1`, 1 byte
    3. `SEND`, 4 bytes
    4. Token trade sell amount for the smart contract, 8 bytes, big endian
    5. etc. arbitrary token output amounts, 8 bytes each, big endian
2. vout 1: P2SH smart contract, the opcodes are as follows, in Rust, where `outputs_pre` are the enforced outputs (here, one OP_RETURN and one BCH P2PKH output):
    ```rust
    let output_slp = SLPSendOutput {
        token_type: 1,
        token_id,
        output_quantities: vec![0, trade.sell_amount],
    };
    let output_buy_amount = P2PKHOutput {
        value: trade.buy_amount,
        address: trade.receiving_address,
    };
    let outputs_pre = serialize(vec![
        Box::new(output_slp),
        Box::new(output_buy_amount),
    ]);
    Script::new(vec![
        Op::Code(OpIf),

        Op::Push(outputs_pre),
        Op::Code(OpSwap),
        Op::Code(OpCat),
        Op::Code(OpHash256),
        Op::Code(OpCat),
        Op::Code(OpSwap),
        Op::Code(OpCat),
        Op::Code(OpSha256),
        Op::Code(Op3Dup),
        Op::Code(OpDrop),
        Op::Push(vec![0x41]),
        Op::Code(OpCat),
        Op::Code(OpSwap),
        Op::Code(OpCheckSigVerify),
        Op::Code(OpRot),
        Op::Code(OpCheckDataSig),

        Op::Code(OpElse),

        Op::Code(OpDup),
        Op::Code(OpHash160),
        Op::Push(trade.cancel_address),
        Op::Code(OpEqualVerify),
        Op::Code(OpCheckSig),

        Op::Code(OpEndIf),
    ])
    ```
    
    A more detailed explanation can be found in this video: https://youtu.be/Ng2fspoWWUY
3. vout ≥2: Arbitrary outputs for sending the remaining tokens/BCH back to the wallet 

The EXCH transaction is created second, and looks like this:

Outputs:
1. vout 0: OP_RETURN: The trade listing metadata:
    1. `EXCH`, 4 bytes
    2. Version `1`, 1 byte
    3. `SELL` 1 byte
    4. Transaction ID of the transaction above, 32 bytes, **little endian**
    5. Output index for the smart contract output of the transaction above, 4 bytes, **big endian**
    6. Sell amount of the token (in base units), 8 bytes, **big endian**
    7. Buy amount of BCH (in satoshis), 8 bytes, **big endian**
    8. Receiving address of the seller (public key hash), 20 bytes, **big endian**
    9. Cancel address of the seller (public key hash), 20 bytes, **big endian**
   
