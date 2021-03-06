.. _interface-label:



Interpreter Interface
=========================

The Scilla interpreter executable provides a calling interface that enables
users to invoke transitions with specified inputs and obtain outputs. Execution
of a contract with supplied inputs will result in a set of outputs and a change
in the smart contract mutable state. 

Calling Interface
###################

A transition defined in a smart contract can be called either by the issuance
of a transaction or by message calls from another smart contract. The same
calling interface will be used to call the contract via external transactions
and inter-contract message calls.

The inputs to the interpreter (``scilla-runner``) consists of four input JSON
files as described below. Every invocation of the interpreter to execute a 
transition must be provided with these four JSON inputs: ::

    ./scilla-runner -init init.json -istate input_state.json -iblockchain input_blockchain.json -imessage input_message.json -o output.json -i input.scilla

The interpreter executable can be run either to create a contract (denoted
``CreateContract``) or to invoke a transition (function) in a contract (``InvokeContract``).
Depending on which of these two, some of the arguments will be absent.
The table below outlays the arguments that should be present in each of
these two cases.  A ``CreateContract`` is distinguished from an
``InvokeContract``, based on the presence of ``input_message.json`` and
``input_state.json``. If these arguments are absent, then the interpreter will 
evaluate it as a ``CreateContract``. Else, it will treat it as an ``InvokeContract``. 
Note that for ``CreateContract``, the interpreter only performs basic checks such as
matching the contract’s immutable parameters with ``init.json`` and whether the
contract definition is free of syntax errors.


+---------------------------+---------------------------+------------------------------------------+
|                           |                           |                 Present                  |
+===========================+===========================+=====================+====================+
| Input                     |    Description            |``CreateContract``   | ``InvokeContract`` |
+---------------------------+---------------------------+---------------------+--------------------+
| ``init.json``             | Immutable contract params | Yes                 |  Yes               |
+---------------------------+---------------------------+---------------------+--------------------+
| ``input_state.json``      | Mutable contract state    | No                  |  Yes               |  
+---------------------------+---------------------------+---------------------+--------------------+
| ``input_blockchain.json`` | Blockchain state          | Yes                 |  Yes               |    
+---------------------------+---------------------------+---------------------+--------------------+
| ``input_message.json``    | Transition and params     | No                  |  Yes               |
+---------------------------+---------------------------+---------------------+--------------------+
| ``output.json``           | Output                    | Yes                 |  Yes               |
+---------------------------+---------------------------+---------------------+--------------------+
| ``input.scilla``          | Input contract            | Yes                 |  Yes               |
+---------------------------+---------------------------+---------------------+--------------------+


Initializing the Immutable State
################################

``init.json`` defines the values of the immutable paramters of a contract.
It does not change between invocations.  The JSON is an array of
objects, each of which contains the following fields:

=====  ==========================================
Field      Description
=====  ==========================================  
vname  Name of the immutable contract parameter
type   Type of the immutable contract parameter
value  Value of the immutable contract parameter
=====  ==========================================  


Example 1
**********

For the ``HelloWorld.scilla`` contract fragment given below, we have only one
immutable variable ``owner``.

.. code-block:: ocaml

    contract HelloWorld
     (* Immutable parameters *)
     (owner: ByStr20)


A sample ``init.json`` for this contract will look like the following:

.. code-block:: json

  [
      { 
          "vname" : "_scilla_version",
          "type" : "Uint32",
          "value" : "0"
      },
      {
          "vname" : "owner",
          "type" : "ByStr20", 
          "value" : "0x1234567890123456789012345678901234567890"
      },
      {
          "vname" : "_creation_block",
          "type" : "BNum",
          "value" : "1"
      }
  ]


Example 2
**********
    
For the ``Crowdfunding.scilla`` contract fragment given below, we have three
immutable variables ``owner``, ``max_block`` and ``goal``.


.. code-block:: ocaml

    contract Crowdfunding
        (* Immutable parameters *)
        (owner     : ByStr20,
         max_block : BNum,
         goal      : UInt128)


A sample ``init.json`` for this contract will look like the following:


.. code-block:: json

  [
    { 
        "vname" : "_scilla_version",
        "type" : "Uint32",
        "value" : "0"
    },
    {
        "vname" : "owner",
        "type" : "ByStr20", 
        "value" : "0x1234567890123456789012345678901234567890"
    },
    {
        "vname" : "max_block",
        "type" : "BNum" ,
        "value" : "199"
    },
    { 
        "vname" : "goal",
        "type" : "Uint128",
        "value" : "500"
    },
    {
        "vname" : "_creation_block",
        "type" : "BNum",
        "value" : "1"
    }
  ]

Input Blockchain State
########################

``input_blockchain.json`` feeds the current blockchain state to the
interpreter. It is similar to ``init.json``, except that it is a fixed size
array of objects, where each object has ``vname`` fields only from a
pre-determined set (which correspond to actual blockchain state variables). 

**Permitted JSON fields:** Only JSONs that differ in the ``value`` field as per
the example below are permitted currently.

.. code-block:: json

    [
        {
            "vname" : "BLOCKNUMBER",
            "type"  : "BNum", 
            "value" : "3265"
        }
    ]

Input Message
###############

``input_message.json`` contains the required information to invoke a
transition. The json is an array containing the following four objects:

=======  ===========================================
Field      Description
=======  ===========================================  
_tag      Transition to be invoked
_amount   Number of ZILs to be transferred
_sender   Address of the invoker
params    An array of parameter objects
=======  ===========================================  


All the four fields are mandatory. ``params`` can be empty if the transition
takes no parameters.

The ``params`` array is encoded similar to how ``init.json`` is encoded, with
each parameter specifying the (``vname``, ``type``, ``value``) that has to be
passed to the transition that is being invoked. 

Example 1
**********
For the following transition:

.. code-block:: ocaml

    transition SayHello()

an example ``input_message.json`` is given below:

.. code-block:: json

    {
        "_tag"    : "SayHello",
        "_amount" : "0",
        "_sender" : "0x1234567890123456789012345678901234567890",
        "params"  : []
    }

Example 2
**********
For the following transition:

.. code-block:: ocaml

    transition TransferFrom (from : ByStr20, to : ByStr20, tokens : Uint128)

an example ``input_message.json`` is given below:

.. code-block:: json

    {
      "_tag"    : "TransferFrom",
      "_amount" : "0",
      "_sender" : "0x64345678901234567890123456789012345678cd",
      "params"  : [
        {
          "vname" : "from",
          "type"  : "ByStr20",
          "value" : "0x1234567890123456789012345678901234567890"
        },
        {
          "vname" : "to",
          "type"  : "ByStr20",
          "value" : "0x78345678901234567890123456789012345678cd"
        },
        {
          "vname" : "tokens",
          "type"  : "Uint128",
          "value" : "500"
        }
      ]
    }




Interpreter Output
#####################

The interpreter will return a JSON object (``output.json``)  with the following
fields:

=====================   ====================================================================
Field                   Description
=====================   ====================================================================
scilla_major_version    The major version of the scilla language of this contract.
gas_remaining           The remaining gas after invoking or deploying a contract.
_accepted               Whether the contract has accepted ZIL (Either ``true``/``false``)
message                 The emitted message to another contract/non-contract account.
states                  An array of objects that form the new contract state
=====================   ====================================================================

+ ``message`` is a JSON object that will have a similar format to
  ``input_message.json``, except that instead of ``_sender`` field, it will have
  a ``_recipient`` field. The fields in ``message`` are given below:

  ===========       =======================================================
  Field              Description
  ===========       =======================================================  
  _tag               Transition to be invoked
  _amount            Number of ZILs to be transferred
  _recipient         Address of the recipient
  params             An array of parameter objects to be passed
  ===========       =======================================================


  The ``params`` array is encoded similar to how ``init.json`` is encoded, with
  each parameter specifying the (``vname``, ``type``, ``value``) that has to be
  passed to the transition that is being invoked. 

+ ``states`` is an array of objects that represents the mutable state of the
  contract. Each entry of the ``states`` array also specifies (``vname``,
  ``type``, ``value``). 


Example 1
*********

The example below is an output generated by ``HelloWorld.scilla``. 

.. code-block:: json

  {
    "scilla_major_version": "0",
    "gas_remaining": "7402",
    "_accepted": "false",
    "message": {
      "_tag": "Main",
      "_amount": "0",
      "_recipient": "0x1234567890123456789012345678901234567890",
      "params": [ 
        { 
          "vname": "code", 
          "type": "Int32", 
          "value": "2" 
        } 
      ]
    },
    "states": [
      { 
        "vname": "_balance", 
        "type": "Uint128", 
        "value": "0" 
      },
      { 
        "vname": "welcome_msg", 
        "type": "String",
        "value": "Hello World" 
      }
    ],
    "events": []
  }


Example 2
*********

Another slightly more involved example with ``Map`` in ``states``.

.. code-block:: json

  {
    "scilla_major_version": "0",
    "gas_remaining": "7359",
    "_accepted": "true",
    "message": null,
    "states": [
      { 
        "vname": "_balance", 
        "type": "Uint128", 
        "value": "100" 
      },
      {
        "vname": "backers",
        "type": "Map (ByStr20) (Uint128)",
        "value": [
          { 
            "key": "0x12345678901234567890123456789012345678ab", 
            "val": "100" 
          }
        ]
      },
      {
        "vname": "funded",
        "type": "Bool",
        "value": { "constructor": "False", "argtypes": [], "arguments": [] }
      }
    ],
    "events": [
      {
        "_eventname": "DonationSuccess",
        "params": [
          {
            "vname": "donor",
            "type": "ByStr20",
            "value": "0x12345678901234567890123456789012345678ab"
          },
          { 
            "vname": "amount", 
            "type": "Uint128", 
            "value": "100" 
          },
          { "vname": "code", 
            "type": "Int32", 
            "value": "1" 
          }
        ]
      }
    ]
  }



.. note::

    For mutable variables of type ``Map``, the first entry in the ``value``
    field are the types of the ``key`` and ``value``. Also, note that the
    ``value`` field of a variable of type ``ADT`` has several fields namely,
    ``constructor``, ``argtypes`` and ``arguments``.

Input Mutable Contract State
############################

``input_state.json`` contains the current value of mutable state variables. It
has the same forms  as the ``states`` field in ``output.json``.  An example of
``input_state.json`` for ``Crowdfunding.scilla`` is given below. 

.. code-block:: json

  [
    {
      "vname": "backers",
      "type": "Map (ByStr20) (Uint128)",
      "value": [
        { 
          "key": "0x12345678901234567890123456789012345678cd", 
          "val": "200" 
        },
        { 
          "key": "0x12345678901234567890123456789012345678ab", 
          "val": "100" 
        }
      ]
    },
    {
      "vname": "funded",
      "type": "Bool",
      "value": { 
        "constructor": "False", 
        "argtypes": [], 
        "arguments": [] 
      }
    },
    {
      "vname": "_balance",
      "type": "Uint128",
      "value": "300"
    }
  ]

