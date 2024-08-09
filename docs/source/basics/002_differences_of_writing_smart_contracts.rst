.. _dev-differences:

Differences of writing smart contracts
======================================

Read&Write State
----------------

This is a simple example that stores a single number and retrieves it.

.. code-block:: solidity

  // SPDX-License-Identifier: GPL-3.0
  pragma solidity >=0.4.16 <0.9.0;

  contract SimpleStorage {
    uint storedUint;

    function setUint(uint x) public {
      storedUint = x;
    }

    function getUint() public view returns (uint) {
      return storedUint;
    }
  }

With mud framework, you should write the same contract like this:

.. code-block:: solidity

  // SPDX-License-Identifier: MIT
  pragma solidity >=0.8.24;

  import { System } from "@latticexyz/world/src/System.sol";
  import { StoredUint } from "../codegen/index.sol";

  contract SimpleStorageSystem is System {
    function setUint(uint x) public {
      StoredUint.set(x);
    }

    function getUint() public view returns (uint) {
      return StoredUint.get();
    }
  }

First, you'll notice that ``SimpleStorageSystem``, while still a
``contract``, additionally inherits from
``@latticexyz/world/src/System.sol``. This is because in projects built
with Mud, all functionalities are implemented through individual systems.
``SimpleStorageSystem`` is also a system, providing the functionality to
store and read a number.

Secondly, you might be curious about where ``StoredUint`` is declared,
and how it's stored and read?

This is one of the important features of the Mud framework. It encapsulates
contract states into a familiar database-like structure, allowing you to
declare, read, and update contract states as if you were operating on database
tables. To achieve the above effect, you only need to pair it with a "table"
configuration file like this:

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";

  export default defineWorld({
    namespace: "muddoc",
    tables: {
      StoredUint: {
        schema:{
          value: "uint256",
        },
        key: [],
      },
    },
  });

It may seem that using native ``Solidity`` would be simpler and more direct if
you're only storing and reading a ``uint``. Indeed, even for ``Array`` or
``Mapping`` type variables, in single contract use cases, Mud might be overkill.
**This is one of the reasons we don't recommend using Mud for simple contracts.
However, we still encourage everyone to try using Mud.**

This is because we often need complex data structures and frequent contract
interactions, which require defining user-friendly methods for data structures
and data interfaces for other contracts. Having to manually write all of these
and manage them well can be a time-consuming and mentally draining task.

So, what benefits can Mud bring to data reading and writing?

* Automatically generated data interfaces, existing as ``libraries``, ready to
  use when needed.
* More compact data packing compared to native ``Solidity``, and easier
  management and maintenance of internal states compared to ``Diamond``.
* Standardized and automated off-chain data indexing through a set of generic
  ``events``, eliminating the need to create data update ``event`` and
  corresponding listeners.

.. note::

  Currently, contracts cannot view other contracts' ``slot`` like they can view
  their own, and can only rely on open data interfaces. However, any on-chain
  data is transparent to any off-chain program.

* Open and standardized data reading interfaces, making it convenient for any
  external contract to read any data from the contract.

.. _dev-differences_contract_interaction:

Message Call
------------

This is a simple example that retrieves and updates a single number from above
``SimpleStorage`` contract.

.. code-block:: solidity

  // SPDX-License-Identifier: GPL-3.0
  pragma solidity >=0.4.16 <0.9.0;

  contract SimpleStorageCaller {
    SimpleStorage simpleStorage;

    constructor(address _simpleStorage) {
      simpleStorage = SimpleStorage(_simpleStorage);
    }

    function setUintToSimpleStorage(uint x) public {
      simpleStorage.setUint(x);
    }

    function getUintFromSimpleStorage() public view returns (uint) {
      return simpleStorage.getUint();
    }
  }

With mud framework, you should write it like this:

.. code-block:: solidity

  // SPDX-License-Identifier: MIT
  pragma solidity >=0.8.24;

  import { System } from "@latticexyz/world/src/System.sol";
  import { IWorld } from "../codegen/world/IWorld.sol";

  contract SimpleStorageCallerSystem is System {
    function setUintToSimpleStorageSystem(uint x) public {
      IWorld(_world()).muddoc__setUint(x);
    }

    function getUintFromSimpleStorageSystem() public view returns (uint) {
      return IWorld(_world()).muddoc__getUint();
    }
  }

Here, ``SimpleStorageCallerSystem`` is also a system, a separate contract
with a different address from ``SimpleStorageSystem``. However, unlike
native contract interactions, ``SimpleStorageCallerSystem`` doesn't
directly call ``SimpleStorageSystem``'s functions. Instead, it calls
another contract address returned by ``_world()``, and the function name
changes to ``muddoc__setUint()`` with a prefix.

This is another important feature of the Mud framework: all system function
entries are centralized on a contract called ``World``. It acts like a
router, responsible for dispatching all system function calls to the
correct systems, along with function name resolution. This is why when one
system calls another system's function, the call is directed to the
``World`` contract returned by ``_world()``, and the function name
undergoes subtle changes.

If you're familiar with the Diamond protocol, it's not hard to guess how
this is achieved. The ``World`` contract is very similar to the
``Diamond`` contract. In fact, they both have a centralized data storage
contract, and all business logic ``System`` or ``Facet`` contracts exist as
on-chain libraries. They don't actually store data, but connect with the
data management contract through ``delegateCall`` or ``call`` to perform
operations on the data.

Some might ask, if the ``World`` contract is similar to ``Diamond`` and all
data is centrally stored, why not let ``SimpleStorageCallerSystem`` directly
read and write ``StoredUint`` instead of using contract interaction?

Indeed, if the cross-contract interaction requirement is just to read and write
a specific piece of data, it can be written like this:

.. code-block:: solidity

  // SPDX-License-Identifier: MIT
  pragma solidity >=0.8.24;

  import { System } from "@latticexyz/world/src/System.sol";
  import { StoredUint } from "../codegen/index.sol";

  contract SimpleStorageCallerSystem is System {
    function setUint2(uint x) public {
      StoredUint.set(x);
    }

    function getUint2() public view returns (uint) {
      return StoredUint.get();
    }
  }

.. important::

  Even if the contract interaction logic is as simple as modifying a single
  state, it may not always be possible to directly operate on the table where
  the state resides. Mud has a strict access control mechanism. If your project
  has only one custom namespace, such as ``muddoc``, and all tables and systems
  belong to it, then the above conversion is feasible. Otherwise, it still
  needs to be analyzed on a case-by-case basis.

.. note::

  Here, we simply want to use this minimalist example to demonstrate that table
  resources are shared within a certain scope and don't require specially
  written interaction methods. In real application scenarios, each system
  method should be carefully designed and reused as much as possible.


