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

With mud framework, you can write the same contract like this:

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

You might be wondering where ``StoredUint`` is declared, and how it's stored
and read?

This is one of the important features of the Mud framework. It encapsulates
contract states into a familiar database-like structure, allowing you to
declare, read, and update contract states as if you were operating on database
tables. To achieve the above effect, you only need to pair it with a "table"
configuration file like this:

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";

  export default defineWorld({
    namespace: "app",
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

Message Call
------------

This is a simple example that retrieves a single number from above
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

.. note::

  ``SimpleStorageSystem`` and ``SimpleStorageCallerSystem`` are two different
  contracts and represent two different systems in the same world. And most
  importantly, they're not in ``root`` namespace.
  We use different way to call a system in ``root`` namespace.

.. code-block:: solidity

  // SPDX-License-Identifier: MIT
  pragma solidity >=0.8.24;

  import { System } from "@latticexyz/world/src/System.sol";
  import { IWorld } from "../codegen/world/IWorld.sol";

  contract SimpleStorageCallerSystem is System {
    function setUintToSimpleStorageSystem(uint x) public {
      IWorld(_world()).app__setUint(x);
    }

    function getUintFromSimpleStorageSystem() public view returns (uint) {
      return IWorld(_world()).app__getUint();
    }
  }

The ``IWorld`` here is an automatically generated interface that includes all
the external interfaces of the ``System`` contracts in the entire project. When
we place ``SimpleStorageSystem`` and ``SimpleStorageCallerSystem`` within the
same project, we automatically obtain an interface collection containing the
external methods of both contracts. Naturally, we can use this interface
collection to call the ``setUint`` and ``getUint`` methods of
``SimpleStorageSystem``.

.. note::

  ``_world()`` is an internal function introduced by ``System``, used to get
  the project's ``World`` address. You can temporarily understand it as the
  project's main contract, where all method entries are established. More
  information will be covered in later chapters.

From the example, we can see that after using the Mud framework, we didn't
manually establish a relationship between the two contracts. Instead, we called
a main contract called ``World`` to complete the reading of ``StoredUint``.
This main contract is actually not the ``SimpleStorageSystem`` contract itself.

.. note::

  There are actually many differences between the ``World`` contract and the
  ``Diamond`` contract, but we'll discuss this in detail later.

If you're familiar with the Diamond protocol, it's not hard to see that the
``World`` contract is very similar to the ``Diamond`` contract. In fact, they
both have a centralized data storage contract, and all business logic
``System`` or ``Facet`` contracts exist as on-chain code libraries. They don't
actually store data, but connect with the data management contract through
``delegateCall`` or ``call`` to perform operations on the data.

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
  has only one custom namespace, such as ``app``, and all tables and systems
  belong to it, then the above conversion is feasible. Otherwise, it still
  needs to be analyzed on a case-by-case basis.

.. note::

  Here, we simply want to use this minimalist example to demonstrate that table
  resources are shared within a certain scope and don't require specially
  written interaction methods. In real application scenarios, each system
  method should be carefully designed and reused as much as possible.


