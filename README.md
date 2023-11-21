# Micropayments
Usage of Algorand Blockchain for Simple Micropayment Channels

## What's an SMC?
Simple Micropayment Channels can allow
two parties to transact in the Layer-2 with each other in a trustless manner with minimal setup and settlement
cost in the Layer-1.

In an SMC, one party is the sender and the other the recipient.
The sender locks some amount of tokens in a multi signature account on the condition that, in case of an
uncooperative recipient, they will be able to be refunded in the future.
In contrast, the recipient can observe the locked funds and the logic of the smart contract to be convinced
that he can keep accepting payments (in the form of signed transactions) until the refund condition is met.

## Why on Algorand?
While Algorand boasts notably affordable transaction fees and impressive speed with finality, the cumulative impact of even minor costs across a substantial volume of transactions should not be overlooked. In this project, the Smart Contracts (SMCs) encounter similar challenges as observed in the Bitcoin ecosystem. These challenges include being unidirectional, posing difficulties or high costs for resetting, and involving only two parties. However, despite these drawbacks, SMCs facilitate transactions between two parties without requiring prior knowledge. This means parties involved don't need to predetermine the number of payments or the timing of these transactions.

The implementation on Algorand takes a unique form due to differences in the primitives offered by its Layer-1. Instead of relying on 'UTXO,' 'Timelocks,' and a 'Script language,' Algorand provides a robust set of features at Layer-1 that can be harnessed. This particular implementation leverages elements such as 'native msig accounts,' 'Logic Signatures (lsig),' 'firstBlock,' 'lastBlock,' 'closeTo,' and the 'TEAL language.'

## Design
### Multisignature
We would also like to have as many active channels between the two parties that can be
customized in the amount of initial funding during setup, minimum and maximum lifetime of the channel.

For this reason, we use 2-out-of-3 Algorand Layer-1 msig accounts shared between A (sender), B (recipient) and C (parameter address).
C should be an address that is inert and can never possibly interfere with the coordination
between A and B. We choose a Smart Signature Account (aka contract account) programmed to
always reject upon being asked to sign any kind of transaction.
There are virtually an infinite number of msig accounts shared between A and B with this technique.
Also, the address of this contract account is essentially the hash of the program.
We inject setup parameters into the program to make it easy to detect already open channels
instead of using random bytes.

Since there are a lot of signatures involved, a msig that has not yet reached the end of
its lifetime shouldn't be accepted by the recipient. Deriving the msig address and checking
against a local database is easy to do.

### LogicSignature
Arpan payments to Tushar happens through the use of a lsig. Alice signs a delegation to submit a transaction with
 Tushar as the recipient, Arpan as the closeout and the amount defined in the latest lsig.

Arpan's refund condition also happens through the use of a lsig.
The msig account must delegate the authority executing the refund transaction to Arpan alone. Following Algorand terminology, msig becomes a delegating account and Arpan the
delegated account.

These lsigs are TEAL code signed by the msig account
 (in turn, both Arpan and Tushar since they are the participants of the msig).

### Communication protocol
Both parties should exchange as little information as possible in the Layer-2.
That's why both parties have access to a template library within this package that they can
use to derive the Layer-1 primitives starting from the agreed upon parameters of the SMC.

- Arpan initiates the setup by sending Tushar `(nonce: int, min_block_refund: int, max_block_refund: int)`.
- Tushar must validate, according to his own knowledge of the Layer-1, that
the parameters of the setup are reasonable.
- Tushar has enough information to derive `(C address, msig, lsig)`.
- Tushar signs his part of the refund lsig and sends `(Tushar's signature of lsig, Bob's public address)`.
- Arpan now has all information to derive `(C, msig, lsig)` on his side.
He validates that Tushar's signature of lsig is valid and that the setup proposal has been
accepted.

Once this setup is completed, Arpan can just pay Tushar by signing her part of a lsig that allows Tushar, some time in the future, to submit a payment w/ closeout from msig to Tushar.
Close out field is used in the payment transaction to make sure that Arpan gets back
whatever is left in the channel when tushar closes it.
tushar can accept these lsigs and, at some point, sign the highest value transaction, send it to the network and close the channel.
tushar should always verify in the Layer-1 that the cumulative amount that he has received,
is at most the current balance of the msig address.

Arpan can also sign a transaction with the shared lsig to unilaterally close the channel and be refunded if Tushar is not cooperating.
Although it should be noted that the lsig allows _only_ Arpan to be refunded, Tushar does not own a fully signed lsig.
By the same token, Arpan signs payments _only_ to Tushar but does not own a fully signed payment transaction.

## Future development
SMC are one of the simplest mechanisms in the Layer-2 scene, but they can serve as starting point to implement
bidirectional, trustless, multi-party, fully connected payment networks.
For further info on how that is possible, there is a very interesting SoK paper linked in the introduction.

We plan to enhance this part of project beyond the scope of SMCs, make this production-ready and build an Algorand for our product diSed in Version 1.1 for payments to the AI Product Companies.
