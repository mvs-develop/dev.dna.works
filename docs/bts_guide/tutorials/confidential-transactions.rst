.. original author: Thom

.. _confidential-transactions-guide:

Confidential Transfers
-----------------------

**How to use the CLI wallet to perform confidential transfers in DNA**


This tutorial shows how to use the CLI wallet to perform confidential transfers
in DNA. A confidential transfer is one that hides both the amount being
sent and the parties involved in the trade. Confidential transfers are also
referred to as blinded transfers. When privacy is important no account should
ever be used twice, and coupled with diligent measures to backup and protect the
wallet and document the cryptograhic keys used it is impossible for any third
party to identify how money is moving by using blockchain analysis alone.

It is important to realize that the current implementation of Steath
functionality requires careful attention to detail and entails a higher level of
risk for loosing your account balance if the steps followed herein are followed
casually and without regard to such potential loss. However, by following this
guide you can rest assured your balance will be held in the blockchain but
remain totally and completely hidden from everyone but you. With that staunch
admonition out of the way let's get started.

We will illustrate the CLI commands required to complete every step in this
tutorial. You must be familiar with the witness_node and cli_wallet software to
follow this guide. For further information on how to build and use that software
consult the `readme file <https://github.com/mvs-org/bitshares-2>`_ on github or the DNA wiki. I will be using
simple passwords and account names to simplify the presentation. You should
change these to more secure values to provide a higher level of security. Also
note that the CLI wallet echos the command entered which is not reflected in the
quoted session logs in the examples that follow.

.. _readme file: https://github.com/mvs-org/bitshares-2

.. contents:: Table of Contents
   :local:

-------


Step 1: Create a Blind Account
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Blind Accounts are not registered on the blockchain like the named accounts of
DNA. Instead a blind account is nothing more than a labeled public key.
The label assigned to the key is only known to your *wallet*. Thus it is
crucial that you create a new wallet for the blind account and back it up after
completing the balance transfer.

The first step in creating a blind account is to create a new wallet and set a
good quality password for it that would be difficult to crack. Then, using this
wallet we'll create a labeled account and protect it with a "brainkey". The
"brainkey" is effectively the private key used by your account. All DNA
cryptography is based on public / private key pairs, one public which can be
shared the other private known only to you.

For confidential accounts the "brainkey" is only stored in the wallet, so if the
wallet file is lost or destroyed and you have not recorded the "brainkey" on
paper or some other place there will be no way to recover your confidential
account balance. Even if you do record your "brainkey" elsewhere outside of the
wallet, I do not believe any recovery methods yet exist to import your "public
key / brainkey" pair into a wallet so you could access your confidential
balance. At least it would be possible to do so at some future date, unlike the
impossibility it would be if you lost the wallet and failed to record the
"brainkey".

::

    >>> create_blind_account alice "alice-brain-key which is a series of words that are usually very long"
    1483572ms th_a       wallet.cpp:743                save_wallet_file     ] saving wallet to file /home/admin/BitShares2/blindAliceWallet
    BTS5Qmr9H9SM39EHmVgXtsVjUGn2xBUtqbF6MdQ6RpnxUWNak7mse
    true

The result of the create_blind_account command is to print out the Public Key
associated with the blind account named alice and the command success or failure
(true). Note that the CLI interface will update the wallet file you specified on
the command line following the -w when you started the cli_wallet. The CLI
wallet has no command to quit or exit the interface so we terminate the session
with a control-C (shown as ^C), which returns us to the operating system shell.

Step 2: Transferring a Balance From a Standard Public Account to a Blind Account
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that we've created a new, unregistered blind account named alice we can
transfer assets into it from any source, public or not. We'll describe the steps
to transfer a balance from one blind account to another momentarily, which is
essential to fully obscure a balance from public view, but for now we'll
illustrate a transfer from a publicly registered account to our newly created
blind account.

To begin, we need a wallet that has an account with assets. It can be any
account with assets, so we'll use Peter's Public account and transfer funds from
that into the alice account. We will also need to have the public key handy that
was printed for the alice account when it was created. Since the alice account
is not registered we need a way to refer to it before we can do the transfer. So
in this CLI session we'll also show you how to create a label to do that and
then we'll use that label to transfer assets into the alice blind account.

::

    >>> list_account_balances "peters-public-registered-account"
    5000 DNA

    >>> set_key_label "DNA5Qmr9H9SM39EHmVgXtsVjUGn2xBUtqbF6MdQ6RpnxUWNak7mse" "Alice-is-Blind"
    true

    >>> transfer_to_blind "peters-public-registered-account" DNA [[Alice-is-Blind,100]] true
    3369305ms th_a       wallet.cpp:3794               transfer_to_blind    ] to_amounts: [["Alice-is-Blind","100"]]
    peters-public-registered-account sent 100 DNA to 1 blinded balance fee: 40 DNA
    100 DNA to  Alice-is-Blind
              receipt: 2B2vTjJ19hgqzGp8qdc8MEWmsgEUGECNJcoQTYNQqMU8bRofmbQYemXs56FoUc4Z5PdVM65nsySZgwJMq9Z SkpWQFhEqLGuZi1N3jQm8yBwaLD2DQzwY5AEW1rSK9HWJbfqNLtx8U4kc3o9xKtJoED2SgHW6jDQ7igBTcVhuUiKSwFu3DFa6LTeS5 Wm5khjgy1LrR5uhmp

    >>> list_account_balances "peters-public-registered-account"
    4860 DNA

The above 2 steps transmit DNA assets from a public, registered account named
"peters-public-registered-account" into a single unregistered blind account
named alice using a label to refer to it named "Alice-is-Blind".  It is
important to note that these labels are NOT persistent from one CLI session to
the next, so every time you transfer assets from a source account such as
"peters-public-registered-account" used here to a blind account you will need to
set a label to refer to the blind account.

**Adding a Contact**


There is currently no facility to transfer assets to a blind account from the
light wallet or the OpenLedger web wallet. They only support the WIF (Wallet
Import Format) and thus will not accept your blind account's "brainkey" as a
valid private key. In the future you may be able to avoid setting labels each
time you transfer from a public to a blind account by defining a contact.
However, keep in mind that every association you establish in the path between a
public account and a confidential account may make it that much easier to trace
your steps, so think twice about the tradeoffs you make for the sake of
convenience. They just might circumvent the measures you are taking to hide
your balance. This is also true if you transfer assets directly between a
public account to a confidential account and leave them in the confidential
account. To totally obscure where your balance is held you need to transfer to
at least 2 different confidential accounts. We will cover this in a bit more
detail later. In the next step we'll look at how to receive the transmitted
assets into alice's blind account.

Step 3: Receiving an Asset Balance Transmitted From Another Account
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Transferring assets from one account to a confidential account involves at least
2 steps, the first to transmit the assets and the second to receive them into
the confidential account. We covered the process required to transmit assets in
Step 2, now lets see what it takes to complete the transfer and verify we have
the correct balance:

::

    >>> receive_blind_transfer "2B2vTjJ19hgqzGp8qdc8MEWmsgEUGECNJcoQTYNQqMU8bRofmbQYemXs56FoUc4Z5PdVM65nsySZgwJMq9ZSkpWQFhEqLGuZi1N3jQm8yBwaLD2DQzwY5AEW1rSK9HWJbfqNLtx8U4kc3o9xKtJoED2SgHW6jDQ7igBTcVhuUiKSwFu3DFa6LTeS5Wm5khjgy1LrR5uhmp "peter" "from Peter"
    100 DNA  peter  =>  alice   "from Peter"

Using the balance receipt value returned from the transfer_to_blind command in
Step 2 we can receive (i.e. import) the balance into alice's blind account.
Note that the source of the balance must be labeled which is the parameter that
follows the long balance receipt key. It is meant to represent to source account
from which the assets are being transferred, however it need not be. The last of
the 3 parameters is a memo text field which is an arbitrary text value. Note
that all 3 parameters are required. In the next section we will describe how to
list the confidential accounts and their balances so that we can verify our
transfer is correct and complete.

Listing Blind Accounts and Their Balances
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For any wallet in which you have created confidential accounts you can list the
accounts present using the "get_my_blind_accounts" CLI command, and use the
accounts returned from that to obtain their balances:

::

    >>> get_my_blind_accounts
    [[
    "alice",
    "DNA5Qmr9H9SM39EHmVgXtsVjUGn2xBUtqbF6MdQ6RpnxUWNak7mse"
    ]]

    >>> get_blind_balances "alice"
    100 DNA

To review, you have learned how to:

 1. create a new CLI wallet and add a blind account to it
 2. create a label to refer to a blind account
 3. send assets from a public account to a blind account
 4. receive or import assets sent from another account into a blind account
 5. list the blind accounts contained in a cli wallet
 6. list the asset balances of blind accounts

These are the basic steps for a simple unidirectional transfer of a single asset
from a *public account* to a single blind account. On the blockchain, this balance
is stored in a structure known as a "Pedersen Commitment".  Direct inspection of
the commitment object will reveal neither the balance contained within, nor the
identity (public key) of the party which controls it.  However, do note that,
while the commitment object itself does not identify the *source* of the funds,
there nevertheless exists in the blockchain a transaction in which
"peters-public-registered-account" spent funds and produced the resulting
commitment object. Thus by inference, one knows both the balance and the source
of the funds.  The ability to make inferences about the contents and origin of
blind balances by analyzing the transactions that created them is known as
"traceability".  A privacy scheme which is resistant to tracing is known as
"untraceable," and it is important to bear in mind that the "blind transfers"
feature as currently implemented in DNA *DOES NOT POSSESS* the property of
untraceability, and so to protect privacy, it is very important to be aware of
one's usage patterns, and what records are left behind.

**I will repeat this for emphasis:** The blind transfers feature *IS NOT UN-TRACEABLE*.
It hides the balance amount, and it hides the identity of the party that is
currently in control of the balance.  But the transactional history of that balance
*may reveal* details of the balance that can be deduced by inference.

The art of maintaining privacy with blind balances is to create balances which have
a rich transactional history, involving multiple parties and large aggregate sums.
Thus mixing blind sums received from multiple parties helps privacy. But please note
that there is no "quick and easy solution" by which you can send a balance from your
public account to a friend's public account using confidential transactions in
the middle. Tracing will reveal the connection between you and your friend.  Your
friend would be best advised to keep the balance blinded, and attempt to spend it
on to yet another party *without* unblinding it to a public account. The
more it is spent between parties, and mixed with funds from other parties, the
less useful information will be revealed by tracing.

In the next section, we cover how to send out blind balance to a second blind
account and
finally we will see how to transfer from a blind account back into a public
account to wrap up our look into protecting your assets with confidential
accounts using the CLI wallet.


Step 4: Transferring Assets Between Confidential Accounts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's start by creating a second wallet and confidential account we will use as
our hypothetical final destination. We'll call this account bobby. We've already
shown how to do this in part 1, but you may wish to review those basic steps
before you continue.

::

    >>> create_blind_account bobby "bobby-brain-key which is a series of words that are usually very long"
    1434971ms th_a       wallet.cpp:743                save_wallet_file     ] saving wallet to file /home/admin/BitShares2/blindBobWallet
    BTS6V829H9SM39EHmVgXtsVjUGn2xBUtqbF6MdQ6RpnxUWNakaV26
    true

We need to restart the CLI wallet with the alice account, where we have a 100
DNA balance. We will create a label to refer to Bob's confidential account
(bobby) and transfer some DNA assets from alice to bobby. Note that the process
is the same as before, and we need to set a label for the bobby (destination)
account to do the transfer.

::

    >>> set_key_label "DNA6V829H9SM39EHmVgXtsVjUGn2xBUtqbF6MdQ6RpnxUWNakaV26" "bobby"
    true

    >>> blind_transfer alice bobby 80 DNA true
    318318ms th_a       wallet.cpp:743                save_wallet_file     ] saving wallet to file /home/admin/BitShares2/blindAliceWallet
    blind_transfer_operation temp-account fee: 15 DNA
    5 DNA to  alice
              receipt: iiMe3q3X4DqW1AqCXfkYEcuRsRATxMwSvJpaUuCbMTcxRUUGeBPPwYU1SRRs4tEQGPNmP$Js4jTJkDGEHzUm33o6h14wa1XNsmedLJCKnwmyGeqFB4vPRk9ZxnaizbMNu8bHr62xQaTc73ALxAZEPRdkNLyqMk$oDEFja3vCPgcyDYCQmkVnNiAQaKeMG83KrW11QZMHQZfzZ8ofTSTEy8qruLAa27vrjAM6q2ckbD8ZTNMWnkSWniq$4fay3Tbcd2zsy9EgxuxN

    80 DNA to  bobby
              receipt: iiMe3q3X4DqW1AqCXfkYEcuRsRATxMwSvJpaUuCbMTcxRUUsn1qUtjfqLYUaNycrpKHfmUG1PR9mxd2nVKB15RYSryyjSn54ADzNBaFzxTY1s699iJWWHw2itiagfcKtvwizhN9Ru8nfnzgx8c5vi7RCLNB2PgrcTxSjYUJW1sfMicFyLRgYrCHFyNd1VhBeWpsLMwagcTGkUTf4rNDyXTrRqqLf2Nhy6P3ohk3J5WbshYyHxuLJGY2E7B5nPpFuf4Bnf9paD6jW

There is a bit more output printed than what is shown above, but the important
results are provided. From this you can see we first set a label to refer to the
newly created "bobby" account, and the blind_transfer command fee was 15 DNA,
which sent 80 DNA of the balance (100 DNA was transferred to the alice account
in Part 1) to the bobby confidential account and provided 2 balance receipts:
the first for 5 DNA coming back to the alice account as returned change
(leftover funds), and the second which is the receipt for the 80 DNA being sent
to the bobby account, which we will need in order to receive the transfer in the
bobby account contained in the blindBobWallet file.

As you can see using confidential in the CLI wallet is a rather tedious "manual"
process. Do note however that you do not need to do a "receive_blind_transfer"
to import the 5 DNA change back into the alice account, at least that is taken
care of. Also important to note is as far as the outside world can see alice
sent some amount less than 100 DNA to two new outputs, one of which is the
change returned, which makes it yet that much more difficult to track what is
going on, especially since the amounts of each output are invisible.


Step 5: Transferring Assets From a Confidential Account Back to a Public Account
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this final step of our round-trip process we will transfer some of the DNA
from the bobby confidential account back to original public account named peter
we started out with. This is a simple procedure, but it is worth mentioning that the source
address for transfers coming into a public account may be inferable by tracing,
so it is advisable to take into account the transactional history of the balance being
ublinded.  (If your friend blinded it and sent it directly to you, then unblinding it
straightaway will leave a direct traceable link between you and your friend.)

First, be sure that the 'bobby' account has imported the blind receipt from Alice:

::

    >>> receive_blind_transfer "iiMe3q3X4DqW1AqCXfkYEcuRsRATxMwSvJpaUuCbMTcxRUUsn1qUtjfqLYUaNycrpKHfmUG1PR9mxd2nVKB15RYSryyjSn54ADzNBaFzxTY1s699iJWWHw2itiagfcKtvwizhN9Ru8nfnzgx8c5vi7RCLNB2PgrcTxSjYUJW1sfMicFyLRgYrCHFyNd1VhBeWpsLMwagcTGkUTf4rNDyXTrRqqLf2Nhy6P3ohk3J5WbshYyHxuLJGY2E7B5nPpFuf4Bnf9paD6jW "alice" "from Alice"
    100 DNA  alice  =>  bobby   "from Alice"

Next, Bobby will use the `transfer_from_blind` operation to transfer a blind balance
to a public account.  Note that in the following command form the first name argument
(`bobby`) is a key label, but the second (`peter`) is a registered public DNA
account.

::

    >>> transfer_from_blind bobby peter 50 DNA true
    2263915ms th_a       wallet.cpp:743                save_wallet_file     ] saving wallet to file /home/admin/BitShares2/blindBobWallet
    blind_transfer_operation temp-account fee: 15 DNA
    15 DNA to  bobby
              receipt: boqRZqyKaZW6bExrystPwFdXvzUBJSjGeaqy482NxBJ6S9Un4zima1mzysTrUipBiBpm4CrLTvCJZfqDaAaqEpmxWAWAKhi2GmnuT7nLU6n18GWjLxUnpskyywA8qCBw9VTAvaxtrNpFRtxx16NzJiZEYk6zfndvLJ2txvjq9cTT16QRXdqPQ75GJxuTAWKNdvzYm3NyK3w3K3462AbutEF9TyNGEfHidvAff49Q3yBATFs1g5NkGAMsmx4ffgwnFeMPBqi58cSZ

    50 DNA to  peter
              receipt: boqRZqyKaZW6bExrystPwFdXvzUBJSjGeaqy482NxBJ6S9VPCqArXCypszWZnpCeG7jfS3oUnbtmn5bmmVH5HCXJg9QxCmn4pocbJ8ipRHfzgeq1mLMewQNn6HGrkb5WbosSntj3o4LcSEMpw2etsR2GjnBxcdxN879rBwxm6inhbpsoYn1nGwS4H o3SqoCF43MRDK3ouYrFBcAK2TTPXfnnvAU3r1UvhNHpxuNaS1cexbd88Nn6BTxSifKdJ8ysFft98e88Cbek

    >>> get_blind_balances bobby
    get_blind_balances bobby
    15 DNA

The explanation for this CLI session is essentially the same as it was for step
4. Although the account information is different the commands used and their
role in the transfer process are the same.  Also, Peter's public account will now
show (publicly) that it has received 50 DNA from an "unknown" source.

One last example demonstrates how to split a balance between multiple
confidential accounts when initially blinding a public balance.
This is very useful because it not only saves on transfer
fees it also obscures what amounts end up where. The point of showing this is
primarily to illustrate the syntax of the command.

::

    >>> list_account_balances "peters-public-registered-account"
    4860 DNA

    >>> set_key_label "DNA5Qmr9H9SM39EHmVgXtsVjUGn2xBUtqbF6MdQ6RpnxUWNak7mse" "alice"
    true

    >>> set_key_label "DNA6V829H9SM39EHmVgXtsVjUGn2xBUtqbF6MdQ6RpnxUWNakaV26" "bobby"
    true

    >>> transfer_to_blind peters-public-registered-account DNA [[alice,800],[alice,2000],[bobby,2000]] true
    peters-registered-account sent 4800 DNA to 3 blinded balances fee: 40 DNA
    800 DNA to  alice
      receipt: 2Dif6AK9AqYGDLDLYcpcwBmzA36dZRmuXXJR8tTQsXg32nBGs6AetDT2E4u4GSVbMKEiTi54sqYu1Bc23cPvzSAyPGEJTLkVpihaot4e1FUDnNPz41uFfu2G6rug1hcRf2Qp5kkRm4ucsAi4Fzb2M3MSfw4r56ucztRisk9JJjLdqFjUPuiAiTdM99JdfKZy8WTkKF2npd

    2000 DNA to  alice
      receipt: 28HrgG1nzkGEDNnL1eZmNvN9JmTVQp7X88nf7rfayjM7sACY8yA7FjV1cW5QXHi1sqv1ywCqfnGiNBqDQWMwpcGB1KdRwDcJPaTMZ5gZpw7Vw4BhdnVeZHY88GV5n8j3uGmZuGBEq18zgHDCFiLJ6WAYvs5PiFvjaNjwQmvBXaC6CqAJWJKXeKCCgmoVJ3CQCw2ErocfVH

    2000 DNA to  bobby
      receipt: 82NxBJ6S9Un4zima1mzyboqRZqyKaZW6bExrystPwFdXvzUBJSjGeaqy4sTrUipBiBpm4CrLTvCJZfqDaAaqEpmxWAWAKhi2GmnuT7nLU6n18GWjLxUnpskyywA8qCBw9VTAvaxtrk6zfndvLJ2txvjq9cTT16QRXdqPQ75GJNpFRtxx16NzJiZEY49Q3yBATFs1g5NkGAMsmx4ffgwnFeMPBqi58cSZxuTAWKNdvzYm3NyK3w3K3462AbutEF9TyNGEfHidvAff

In this case the only thing the public sees is that account 'peters-public-registered-account' sent 4800
DNA to three different blinded destinations. Note that although a sum of 2800 DNA were sent to
the alice confidential account, and 2000 to the bobby confidential account, there is nothing on the
blockchain that identifies either Alice or Bob as the recipients.  (Although Peter is visible as the sender,
since the balance was initially public.)

**Conclusion**: The outside world has no idea *how much* is in each individual blinded output,
or who is in control of each one, only that together they all add up to 4800 DNA.


Summary
^^^^^^^

We covered several steps and several operations with Confidential Transactions, a.k.a. "blind" transfers, in DNA, using the CLI (command line) wallet.  A brief summary of the wallet commands:

* **transfer_to_blind** - (Public --> Blind) - This was used to create blind balances from a public source.  Note that the resulting balances, while they do not reveal their contents directly, are trivially traceable and simple blockchain analysis can reveal their contents.  (The transaction which created them is permanently stored in the blockchain.)
* **blind_transfer** - (Blind --> Blind) - This spends from a blind balance to one or more new blind balances, by "consuming" the outputs of previous blind transactions and producing new blind outputs.  Note that the blockchain will record which old outputs were consumed and which new outputs are created, however details of *who* is doing the transaction, and how much is in each output, are obscured. Be sure that you send the *blind receipt* to the party that will receive the transfer.  (They will need it to detect the incoming transaction.)
* **receive_blind_transfer** - This was used to make your wallet aware of an incoming transaction.  To recieve the blind transfer, the party that sent it to you must also send you the "blind receipt", which you must give as an argument to this command.  (This step is very important.  Without it your wallet will NOT detect the incoming transaction, and you will lose the funds.  Note however that if you are sending to yourself - i.e., to a key stored in your own wallet - then the CLI wallet is smart enough to perform this step for you.)
* **transfer_from_blind** - (Blind --> Public) - This was used to send a blind balance to a public account destination.  Before doing this, it is important to be aware that blockchain analysis could be used to "trace" the history of such a balance.  By looking at the ancestor transactions to the one that created the balance you are unblinding, it will be possible to construct a list of all possible origins of the balance.  (Which is to say, all public parties that blinded a balance that ended up contributing to the history of your balance.)  This list could be very large, if the balance was transfered often and among many different parties, and mixed with balances from multiple sources, or the list could be very short.  If your friend just blinded the balance last week and proceeded to send it to you, then your friend is the only one on the list.  But if the list is very large, then the person doing the analysis can do nothing better than guess at who is the "real" source of the funds.  And of course, all the "blind to blind" transfers in between are effectively anonymous.

|

--------------------
