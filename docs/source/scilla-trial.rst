Trying out Scilla
=================

Scilla is under active development. At this stage, there are two ways to try
out Scilla. 


Blockchain IDE
**********************

The simplest way to try out Scilla is through the `Scilla Blokchain IDE`. The
IDE is connected to the Zilliqa blockchain via a testnet wallet and a block
explorer and hence comes with (almost) all the features needed to test a Scilla
contract in a blockchain environment. 

In order to use Scilla Blockchain IDE, a user will have to hold Testnet ZIL
(tokens to use Zilliqa's blockchain infrastructure). These tokens are
periodically distributed for free. Testnet ZIL tokens are required to pay for
gas fees to deploy and run smart contracts. The IDE comes with a set of
preloaded sample smart contracts that can either be deployed or run. 

Each regular payment from a non-contract account to a non-contract account
corresponds to 1 unit of gas. Each contract creation corresponds to 50 units of
gas, while each transition invocation from a non-contract account to a contract
account will cost 10 units of gas. Any message call from a contract account to
a another contract account or otherwise will also cost 10 units of gas. 

For example, a chained invocation, where, a user say Alice calling a contract
``C_1`` that  in turn calls another contract ``C_2`` will require 20 units of
gas in total.

To try out the Blockchain IDE, users need to go through the Zilliqa testnet
wallet.


Interpreter IDE
************************

`Scilla Interpreter IDE` is a simple development environment meant for users
who would like to get their hands dirty with Scilla coding and testing. The
Scilla Interpreter IDE is a standalone environment to test Scilla contracts. It
is  not connected to any blockchain network and hence does not maintain any
persistent state and is not aware of any blockchain-wide parameters such as the
current block number.

As a result, the contract writer or the invoker will have to mimic certain
inputs, for instance, the current contract and blockchain state among others.
Refer to :ref:`interface-label`  to read about the format of the inputs. 

In order to user this IDE, users do not need to hold testnet ZIL.