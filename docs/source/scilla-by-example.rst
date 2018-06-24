Scilla by Example
==================


HelloWorld
###################

We start off by writing a classical ``HelloWorld.scilla`` contract with the
following  specification:


+ It should have an immutable field ``owner`` to be initialized by
  the creator of the contract. ``owner`` will be of type ``Address``. 

+ It should have a mutable field ``welcome_msg`` of type ``String`` initialized to
  ``""``.

+ The ``owner`` and only her should be able to modify ``welcome_msg`` through an
  interface ``SetHello (msg: String)``, where ``msg`` is a value to which
  ``welcome_msg`` should be changed. 

+ It should have an interface ``GetHello()`` that welcomes any caller with
  ``welcome_msg``. 


Defining Contract and its (Im)Mutable Parameters
**************************************************

A contract is declared using the ``contract`` keyword that starts the scope of
the contract. The keyword is followed by the name of the contract which is
``HelloWorld`` in this case. 

.. code-block:: ocaml

    contract HelloWorld


.. note::
	In the current implementation, a scilla contract can only contain a single
	contract declaration and hence any code that follows the ``contract`` is part
	of contract declaration. In other words, there is no explicit keyword to
	declare the end of the contract definition.



Then follows the declaration of immutable parameters, the scope of which is
defined by ``()``.  Each immutable variable is declared in the following way:
``vname: vtype``, where ``vname`` is the variable name and ``vtype`` is the
variable type. Immutable parameters are separted by ``,``. As per the
specification, the contract will have only one immutable variable ``owner`` of
type ``Address``.  


.. code-block:: ocaml

    (owner: Address)

Mutable parameters in a contract are declared through keyword ``field``. Each
mutable variable is declared in the following way: ``field vname : vtype
= init_val``, where ``vname`` is the variable name, ``vtype`` is its 
type and ``init_val`` is the value to which the variable has to be initialized.
The contract has one mutable parameter ``welcome_msg`` of type ``String``
initialized to ``""``.

.. code-block:: ocaml

    field welcome_msg : String = ""


At this stage, the contract fragment will look like the following:

.. code-block:: ocaml

    contract HelloWorld
    (owner: Address)

    field welcome_msg : String = ""


Defining Interfaces `aka` Transitions
***************************************

Interfaces like ``SetHello (msg :  String)`` are referred to as `transitions`
in Scilla. Transitions are similar to `functions` or `methods` in other
languages.  

A transition is identified by the keyword ``transition``. The end of the scope
of a  transition is declared by the keyword ``end``. The ``transition``
keyword is followed by the transition name, which is ``SetHello`` in the
example. Then follows the input prarmeters within ``()``. Each input prameter
is separated by a ``,`` and is declared in the following format: ``vname :
vtype``. According to the specification, ``SetHello`` takes only one parameter
of name ``msg`` of type ``String``.

.. code-block:: ocaml

    transition SetHello (msg : String)

What follows the transition signature is the body of the transition. Code
for the first transition ``SetHello (msg :  String)`` to set ``welcome_msg`` is given below: 



.. code-block:: ocaml

    transition SetHello (msg : String)
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : not_owner_code};
        msgs = one_msg msg;
        send msgs
      | True =>
        welcome_msg := msg;
        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : set_hello_code};
        msgs = one_msg msg;
        send msgs
      end
    end

At first, the sender of the transaction is checked against the ``owner`` using
the instruction ``builtin eq owner _sender`` which returns a boolean value.


.. note::

    Scilla internally defines some variables that have special semantics. These
    special variables are often prefixed by ``_``. For instance, ``_sender`` in
    Scilla means the account address that called the current contract.

Depending on the boolean value, the transition takes different paths using pattern matching, the syntax of which is standard.

.. code-block:: ocaml

	match expr with
	| x => expr_1
	| y => expr_2
        end 

    
In case the caller is different from ``owner``, the contract sends a message
(denoted ``msg``) as an output, with an error code ``code : not_owner_code``.
More concretely, the output message in this case is:

.. code-block:: ocaml

        msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : not_owner_code};

.. note::

	An outgoing message from a contract is a list of ``vname : value`` pairs, the
	scope of which is defined using ``{}``. Each outgoing message must have three
	compulsory fields: `_tag`, ``_recipient`` and  


.. code-block:: ocaml

    transition GetHello ()
        r <- welcome_msg;
        msg = {_tag : Main; _recipient : _sender; _amount : 0; msg : r};
        msgs = one_msg msg;
        send msgs
    end

.. code-block:: ocaml

    (* HelloWorld contract *)


    (***************************************************)
    (*               Associated library                *)
    (***************************************************)
    library HelloWorld

    let one_msg = 
      fun (msg : Message) => 
      let nil_msg = Nil {Message} in
      Cons {Message} msg nil_msg

    let not_owner_code  = 1

    (***************************************************)
    (*             The contract definition             *)
    (***************************************************)

    contract HelloWorld
    (owner: Address)

    field msgstr : String = "Hello World"

    transition SayHello()
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False =>
        msg = {_tag : Main; _recipient : _sender; _amount : 0; code : not_owner_code};
        msgs = one_msg msg;
        send msgs
      | True =>
        greeting <- msgstr;
        msg = {_tag : Main; _recipient : _sender; _amount : 0; welcome_msg : greeting};
        msgs = one_msg msg;
        send msgs
      end
    end

Crowdfunding
###################


.. code-block:: ocaml
    :linenos:

    (***************************************************)
    (*               Associated library                *)
    (***************************************************)
    library Crowdfunding

    let andb = 
      fun (b : Bool) =>
      fun (c : Bool) =>
        match b with 
        | False => False
        | True  =>
          match c with 
          | False => False
          | True  => True
          end
        end

    let orb = 
      fun (b : Bool) => fun (c : Bool) =>
        match b with 
        | True  => True
        | False =>
          match c with 
          | False => False
          | True  => True
          end
        end

    let negb = fun (b : Bool) => 
      match b with
      | True => False
      | False => True
      end

    let one_msg = 
      fun (msg : Message) => 
        let nil_msg = Nil {Message} in
        Cons {Message} msg nil_msg
        
    let check_update = 
      fun (bs : Map Address Int) =>
      fun (_sender : Address) =>
      fun (_amount : Int) =>
        let c = builtin contains bs _sender in
        match c with 
        | False => 
          let bs1 = builtin put bs _sender _amount in
          Some {Map Address Int} bs1 
        | True  => None {Map Address Int}
        end

    let blk_leq =
      fun (blk1 : BNum) =>
      fun (blk2 : BNum) =>
        let bc1 = builtin blt blk1 blk2 in 
        let bc2 = builtin eq blk1 blk2 in 
        orb bc1 bc2

    let accepted_code = 1
    let missed_deadline_code = 2
    let already_backed_code  = 3
    let not_owner_code  = 4
    let too_early_code  = 5
    let got_funds_code  = 6
    let cannot_get_funds  = 7
    let cannot_reclaim_code = 8
    let reclaimed_code = 9
      
    (***************************************************)
    (*             The contract definition             *)
    (***************************************************)
    contract Crowdfunding

    (*  Parameters *)
    (owner     : Address,
     max_block : BNum,
     goal      : Int)

    (* Mutable fields *)
    field backers : Map Address Int = Emp Address Int
    field funded : Bool = False

    transition Donate ()
      blk <- & BLOCKNUMBER;
      in_time = blk_leq blk max_block;
      match in_time with 
      | True  => 
        bs  <- backers;
        res = check_update bs _sender _amount;
        match res with
        | None => 
          msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
                  code : already_backed_code};
          msgs = one_msg msg;
          send msgs
        | Some bs1 =>
          backers := bs1; 
          accept; 
          msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
                  code : accepted_code};
          msgs = one_msg msg;
          send msgs     
        end  
      | False => 
        msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
                code : missed_dealine_code};
        msgs = one_msg msg;
        send msgs
      end 
    end

    transition GetFunds ()
      is_owner = builtin eq owner _sender;
      match is_owner with
      | False => 
        msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
                code : not_owner_code};
        msgs = one_msg msg;
        send msgs
      | True => 
        blk <- & BLOCKNUMBER;
        in_time = blk_leq blk max_block;
        c1 = negb in_time;
        bal <- balance;
        c2 = builtin lt bal goal;
        c3 = negb c2;
        c4 = andb c1 c3;
        match c4 with 
        | False =>  
          msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
                  code : cannot_get_funds};
          msgs = one_msg msg;
          send msgs
        | True => 
          tt = True;
          funded := tt;
          msg  = {_tag : Main; _recipient : owner; _amount : bal; 
                  code : got_funds_code};
          msgs = one_msg msg;
          send msgs
        end
      end   
    end

    (* transition ClaimBack *)
    transition ClaimBack ()
      blk <- & BLOCKNUMBER;
      after_deadline = builtin blt max_block blk;
      match after_deadline with
      | False =>
        msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
                code : too_early_code};
        msgs = one_msg msg;
        send msgs
      | True =>
        bs <- backers;
        bal <- balance;
        (* Goal has not been reached *)
        f <- funded;
        c1 = builtin lt bal goal;
        c2 = builtin contains bs _sender;
        c3 = negb f;
        c4 = andb c1 c2;
        c5 = andb c3 c4;
        match c5 with
        | False =>
          msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
                  code : cannot_reclaim_code};
          msgs = one_msg msg;
          send msgs
        | True =>
          res = builtin get bs _sender;
          match res with
          | None =>
            msg  = {_tag : Main; _recipient : _sender; _amount : 0; 
                    code : cannot_reclaim_code};
            msgs = one_msg msg;
            send msgs
          | Some v =>
            bs1 = builtin remove bs _sender;
            backers := bs1;
            msg  = {_tag : Main; _recipient : _sender; _amount : v; 
                    code : reclaimed_code};
            msgs = one_msg msg;
            send msgs
          end
        end
      end  
    end


