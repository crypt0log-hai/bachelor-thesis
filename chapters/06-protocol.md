
# Protocol

We describe an analysis of the problematic and the implementation of the cross-chain atomic swap protocol between Bitcoin and Ethereum. The protocol can be generalized for Bitcoin and any other cryptocurrencies that fulfill the same requirements as Bitcoin (e.g. LiteCoin), see the chapter \ref{prerequisites}. This protocol is heavily based on the `BIP-199` (\gls{bip}) from \citep{bip199} for the Bitcoin part. For Ethereum the concept is roughly the same but with fewer prerequisites than Bitcoin.  For sending funds, each participant must generate a specific address called HTLC on each blockchain, lock funds onto these addresses where the other party can take control from the other blockchain.

## Limitations

The most important process of the protocol is `liveness`. Liveness means that participants must be online for respecting the protocol (at least one participant is still online). In the worst scenario where someone doesn't follow the protocol, it can happen the coalition end up and lose the funds. This happens only if a party does not remain online during the swap or it has not claimed the funds in time.

Secondly, there is another factor to take on board which is the `Fees`. Each blockchain has different fees because there are built with different internal parameters and transaction complexity. It is also due to a factor, the block space that depends on the demand. In this project, we use the Bitcoin Blockchain as a tool, more precisely, we use some advanced features that increase the cost of the transaction for bitcoin side as `P2SH`. In general, the transaction is more expensive on Bitcoin than Ethereum, because the transaction cost in Ethereum doesn't depend on the user but by the operation in the contract.

The difficult problem with cross-chain swaps is off-chain coordination. In a few words, it's to find an agreement between the two parties on specific conditions. This agreement can depend on the speed of the protocol (i.e. to considerate that a transaction is confirmed) but the speed is influenced by the slowness and several confirmations required for validating a transaction in each blockchain side. The protocol is slow but it can be extended by way of setups. The only thing we can change from the setups is the ranges of fees that can be consumed in a transaction.

## Scenario

Alice and Bob want to exchange 1 Alice's tokens for 10 Bob's tokens. The problem is that they are not in the same blockchain, Alice token is defined in Bitcoin blockchain, whereas Bob's token is only present in Ethereum blockchain.

\input{fig/atomic-swap} 

Let's see the process in figure \ref{fig:atomicswap} :

1. Alice generates a random set of bytes called value or preimage. The proof should have a size of 32 bytes.
2. Alice hashes the obtained proof to generate the secret.
3. Alice is the instigator of the swap, she starts by initiating the locking script, TX1 in the Bitcoin chain.
4. Upon doing this, Alice broadcasts TX1 to the Bitcoin chain and transfer the secret to Bob.
5. Bob defines locking script transaction TX2 to Ethereum using the hash.
6. Only Alice can unlock the ETH in this address because she has the value which generates that particular hash. It's the claiming transaction.
7. Alice can get her ETH by signing a transaction for Bob’s contract address and Bob can retrieve the BTC by signing a transaction TX3 for Alice's contract address.
8. When Alice signs Bob’s contract address with the value, she unlocks the address and reveals the value to Bob as well.
9. Bob, now knowing the value, signs off the transaction TX4 for Alice’s address and retrieves his BTC.

To summarize the process, the scenario describes the participants and their incentives. `Alice` the sender owns Bitcoin (BTC) and Bob the receiver owns ether (ETH), they want to swap funds. Alice and Bob have already negotiated the price in advance and found an agreement (i.e. an amount of bitcoin for an amount of ether to swap). They are only two possible ways of execution path for both parties :

* `The protocol succeeds` - Alice gets her ETH and Bob his BTC.
* `The protocol failed` - both parties keep their fund (they will lose some amounts because they need to pay some fees for each transaction).

### Successful swap

For having a successful swap, both parties must follow the protocol. They will be four transactions in total, 2 transactions for Bitcoin blockchain and 2 transactions for Ethereum blockchain :

1. Lock the funds in Bitcoin and make it ready for the swap.
2. Lock the funds in Ethereum and make it ready for the swap.
3. Unlock the funds in Ethereum.
4. Unlock the funds in Bitcoin.

When participants unlock the funds, they take control of the output of the contract in the other chain. here is the most simple and optimal way to perform the protocol. Here, no timelock is required but both participants must care about the minimum number of transactions and for the minimal transaction, the funding transaction (locking fund). Confirmations vary between each chain, so it needs to be considered if both parties are expecting the funding transaction to be considered final and are sure to keep going the protocol.

### Swap aborted

The swap is aborted only if one party wants not to continue the process. To get the refund of the locked funds, Alice or Bob must wait for the timelock is reached. When Alice starts, there is no ETH but only BTC locked into a contract. If bob doesn't follow, so Alice waits for her timelock and after that, she can spend the refund. The length of time on each lock is important to ensure that the game can only be played fairly.  Alice’s time lock should be longer and Bob’s lock should be much shorter. This is because Alice knows the hash lock secret and therefore has a major advantage. It is very important because if Alice's timelock had the shorter refund time, Alice could wait until that time expires, refund herself the Bitcoin and after that then enter the secret preimage into Ethereum to claim the ETH that Bob sent. Alice would have both coins and Bob would lose his ETH.

### Worst scenario

There is a possibility that the protocol can be broken again if a party doesn't follow the rules from the Bitcoin part. If the swap process succeeds with Alice claiming ETH funds and Bob doesn't claim his BTC fund before Alice's timelock, then Alice can spend her refund as soon as her timelock is reached. It will conclude that Bob would lose his funds and Alice would get both coins. In Ethereum this can't happen because when the timelock is reached, claim funds are automatically blocked and Alice cannot claim the fund, only Bob spent the refund to avoid that situation.
To resolve this problem, we must implement a protocol that forces Bob to be not offline or compensate Bob if Alice doesn't follow correctly the protocol.

## Prerequisites

In chapter \ref{scenario}, we describe the conditional process that must be followed to guarantee a swap with atomicity. Bitcoin has a small stack-based script language that allows for conditional execution and timelocks. Whereas Ethereum uses the programming language that allows hashing and timelocks too. The challenge is then to implement the BIP-199 in Ethereum.

### Bitcoin

The bitcoin transactions in this protocol use \gls{gls-segwit} structure that allows reducing the fees. For any other cryptocurrencies with a bitcoin style UTXO model as such e.g. Litecoin, these requirements must be fulfilled for having the same compatibility with this protocol. E.g. Bitcoin Cash isn't compatible.

#### Pre-image
Generation of a valid pre-image  $\alpha \in \mathbb{Z}_{2^{256}}$ of 32 bytes size to a given $h = \mathcal{H}_\textit{256}(\alpha)$ where $\mathcal{H}_\textit{256}$ is the `SHA256` algorithm.

#### Public key hash   
For a public key $Q$ to a given $h_Q = \mathcal{H}_\textit{160}(Q)$ where $\mathcal{H}_\textit{160}$ is the `SHA256` followed by the `RIPEMD-160` algorithm.
$h_Q$ is the hash version of $Q$ that is given to another participant so that they can send it bitcoins. It's shorter than the original public key, and it may provide an extra layer of security for the bitcoins compared to giving the public key directly.

#### Hashlock
Hashlock is for revealing the secret to the other participant. It is a primitive that includes a value to reveal some data (pre-image) that is associated with a given hash where $h_s = \mathcal{H}_\textit{256}(s)$ and handle the spent of the HTLC.

#### Timelock
The timelock is to enable an execution path that is predefined by an amount of time. This amount of time is expressed in a number of block `nSequence` where $t = \textit{nSequence}$. We use the number of the block instead of the amount of time in second for avoiding a problem called `leap second`.

#### Multi signatures

The signatures of both participants are required for creating HTLC only accessible by them if they agree.


### Ethereum

Ethereum doesn't use the same Model as Bitcoin (UTXO) but is based on `Account Model`. Every cryptocurrency with this style model that must fulfill these requirements for having the same compatibility with this protocol. In comparison to Bitcoin, Ethereum uses a smart contract that handles also timelock and hash lock. However, we don't need the `Public key hash` for the verification. We use instead, the `msg.sender` from Smart contract that allows verification.

### Elliptic Curve

Bitcoin and Ethereum do use the same elliptic curves. They use the `secp256k1` curve from Standards for Efficient Cryptography (SEC) with the
\gls{ecdsa} algorithm. The curve is described as follow :

\begin{equation}
\begin{split}
    p&: \text{a prime number;}\ p = 2^{256} - 2^{32} - 2^9 - 2^8 - 2^7 - 2^6 - 2^4 - 1 \\
    a&: \text{an element of } \mathbb{F}_p;\ a = 0 \\
    b&: \text{an element of } \mathbb{F}_p;\ b = 7 \\
    E&: \text{an elliptic curve equation};\ y^2 = x^3 + bx + a \\
    G&: \text{a base point};\ G = \\ (&\texttt{\scriptsize0x79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798},\\ &\texttt{\scriptsize0x483ADA7726A3C4655DA4FBFC0E1108A8FD17B448A68554199C47D08FFB10D4B8}) \\
\end{split}
\end{equation}

\newpage


## Hashed Timelock Contract

The description of the protocol is as follows: Alice moves her bitcoin into a `P2WSH` address where each participant controls a type of transaction using Bitcoin scripting language. Bob does the same into an Ethereum `Smart contract` address that is then used to reveal the secret depending on Alice who claims the ether. Bitcoin and Ethereum transactions are designed in such a way that if a participant follows the protocol, there is no way for losing his coin. If the deal goes through, Alice spends the ether by revealing the secret, thus allowing Bob to spend the locked bitcoin. If the deal is aborted, Bob spends the ether after the second timelock, thus allowing Alice to spend the bitcoin after the first timelock. In both cases, the participants must add transaction fees.
The full protocol is described in Table \ref{tbl:protocol}.

\input{fig/protocol}

\newpage

### Time parameters 

\begin{equation}
\begin{split}
    t_0& : \text{ Alice's timelock} \\   
    t_1& : \text{ Bob's timelock}  
\end{split}
\end{equation}

We use two timelocks t0 and t1 that are defined during lock swap. t0 sets the time for Alice where it is safe to execute the exchange. When t0 is passed, the refund may start. t1 sets the response time during which Alice is required for claiming the coin from Bob and reveals her preimage. When t1 is passed, Bob can get his ether back and allows Alice to redeem her bitcoin. Note that all timelocks are defined in Relative Timelock, see \ref{relative-timelock}.

### Bitcoin Script

#### Swaplock
\gls{p2sh} is used to lock funds and defines the two base execution paths :

 1. swap execution [success]. 
 2. refund execution [fail]. 
   
The script is defined with Bob's $h_B$ public key hash, Alice's $h_A$ public key hash and the preimage $h_s$  in the Listing \ref{lst:scriptprotocol}:

\begin{minipage}{\linewidth}\centering
\begin{lstlisting}[mathescape=true, caption={Swaplock script.},label=lst:scriptprotocol]
OP_IF
    OP_SHA256 <$h_s$> OP_EQUALVERIFY OP_DUP 
    OP_HASH160 <$h_B$>  
OP_ELSE
    <$t_0$> OP_CHECKSEQUENCEVERIFY OP_DROP OP_DUP
    OP_HASH160 <$h_A$>
OP_ENDIF
OP_EQUALVERIFY
OP_CHECKSIG
\end{lstlisting}
\end{minipage}

The Swaplock is executed when `OP_IF` reads a `TRUE` value from the stack. It expects a secret value, an \gls{ecdsa} signature and the \gls{pkh}. It hashes the secret and checks that it matches a given hash, then it checks PKH followed by the signature against the given public key. When the value `FALSE` from the stack is read, it executes the `OP_ELSE` branch.

#### Claim Fund

Bob takes control of bitcoin in using the pre-image $s$ and his public key hash $h_B$ from Alice to redeem the `Swaplock P2SH`. To redeem the HTLC from this way, Bob uses the following script in the input of a transaction:

\begin{minipage}{\linewidth}\centering
\begin{lstlisting}[mathescape=true, caption={Bob's script signature},label=lst:claim]
   <$sig_B$> <$h_B$> <$s$> OP_TRUE
\end{lstlisting}
\end{minipage}

#### Spend Refund

With this contract Alice can spend this output with her public key hash $h_A$ after the timelock $t_0$ with the script signature :

\begin{minipage}{\linewidth}\centering
\begin{lstlisting}[mathescape=true, caption={Alice's script signature},label=lst:refund]
   <$sig_A$> <$h_A$> OP_FALSE
\end{lstlisting}
\end{minipage}


### Ethereum Smart Contract

Ethereum doesn't use script language Bitcoin but the programming language for the smart contract. The smart contract allows to create functions from the UML diagram in figure \ref{fig:htlcuml} :

\input{fig/htlc-diagram}

\newpage

#### Function lock()
A function that will create a contract with all the prerequisites and lock it with the address of the sender `Bob` and the address of the receiver `Alice`.

#### Function unlock()
Function when it is called, check if the pre-image $s$ is correct and then transfer the fund to `Alice`.

#### Function refund()
Function when it is called, check if the timelock $t_1$ is reached and then transfer the fund to `Bob`.

### Transactions

All the transactions are described in figure \ref{fig:list_transactions}.

#### Funding transaction

The funding transaction is the transaction sending fund to the contract address. 
$\text{BTX}_\textit{lock}$, bitcoin transaction with 1 or more inputs from Alice and the output (vout) to the `Swaplock P2SH`.
$\text{ETX}_\textit{lock}$, ethereum transaction from Bob that sends funds to the Smart Contract address. 

#### Claim Transaction

The claim transaction is a transaction that allows the sender to spend the funds.
$\text{BTX}_\textit{unlock}$, bitcoin transaction with 1 inputs consuming `Swaplock P2SH` ( $\text{BTX}_\textit{lock}$ ) and 1 output vout to Bob.
$\text{ETX}_\textit{unlock}$, ethereum transaction from the `Smart Contract` that call the function `unlock` to send the funds to Alice. 

#### Refund transaction

The refund transaction is a transaction that allows the sender to abort the swap and get his funds back.
$\text{BTX}_\textit{refund}$, bitcoin transaction with 1 inputs consuming `Swaplock P2SH` ( $\text{BTX}_\textit{lock}$ ) and 1 output vout to Alice.
$\text{ETX}_\textit{refund}$, ethereum transaction from `Smart Contract` that call the function `refund` to send back the funds to Bob. 

\input{fig/list_transactions}