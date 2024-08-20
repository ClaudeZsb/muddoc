Modules
=======

**Modules are on-chain scripts that add specific functionality to
autonomous worlds in the form of plugins through the registration and
configuration of various resources. Modules can be shared between
autonomous worlds.**

Implementation Process
----------------------

We'll use the official ``uniqueentity`` module as an example to
illustrate how modules introduce specific functionality to autonomous
worlds. This module adds the ability to obtain unique entity IDs for
an autonomous world.

The functionality essentially consists of a ``uint256`` singleton table
and a system that operates on this table to implement auto-increment.
The IDs obtained using this functionality are integers that
auto-increment starting from 1.

First, let's look at the directory structure of the module file:

.. code-block::

  node_modules/@latticexyz/world-modules/src/modules/uniqueentity
  ├── UniqueEntityModule.sol
  ├── UniqueEntitySystem.sol
  ├── constants.sol
  ├── getUniqueEntity.sol
  └── tables
      └── UniqueEntity.sol

``UniqueEntityModule.sol`` is the installation script for the module.
The module installation is completed by calling this script contract.
``UniqueEntitySystem.sol`` is the system that implements auto-increment
for the singleton table.
``constants.sol`` defines the module's constants, including the namespace
for the functionality to be introduced and the ``ResourceId`` for various
required resources.
``getUniqueEntity.sol`` is the usage script for the functionality
introduced by the module, which is obtaining unique entity IDs.
``tables/UniqueEntity.sol`` is the ``uint256`` singleton table.

There are two ways to install modules:

.. code-block:: solidity

  interface IModule is IERC165, IModuleErrors {
    /**
    * @notice Installs the module as a root module.
    * @dev This function is invoked by the World contract during `installRootModule` process.
    * @param encodedArgs The ABI encoded arguments that may be needed during the installation process.
    */
    function installRoot(bytes memory encodedArgs) external;

    /**
    * @notice Installs the module.
    * @dev This function is invoked by the World contract during `installRootModule` process.
    * @param encodedArgs The ABI encoded arguments that may be needed during the installation process.
    */
    function install(bytes memory encodedArgs) external;
  }

The main differences between the two installation methods are in the
installation entry point and the permissions of the installation scripts.

.. code-block:: ts

  --> call, ==> delegatecall

  // Install root module
  User --> IWorld.installRootModule ==> IModule.installRoot
  // Install module
  User --> IWorld.installModule --> IModule.install

To install a root module, we need to call the ``installRootModule``
method of the ``World`` contract, passing in the module's address and
necessary parameters. For installing regular modules, we need to call the
``installModule`` method. Additionally, during the root module installation
process, the ``World`` contract uses ``delegatecall`` to call the module's
``installRoot`` method. As a result, the installation script of the root
module executes with the highest permissions, allowing it to do more than
regular module installation scripts, such as registering resources and
configuring access within the ``root`` namespace.

The installation script for the ``uniqueentity`` module mainly does the
following:

1. Registers a namespace to store the ``UniqueEntity`` table and
   ``UniqueEntitySystem`` system.
2. Registers the ``UniqueEntity`` table.
3. Registers the ``UniqueEntitySystem`` system, registering the function
   selector ``getUniqueEntity()`` for the system method.

Official Modules
----------------

Some commonly used modules are provided officially. The source code can
be found `here <https://github.com/latticexyz/mud/tree/main/packages/world-modules/src/modules/uniqueentity>`_.

- ``uniqueentity``: Provides functionality to obtain unique entity IDs.
- ``keysintable``: Counts existing keys for specified tables, providing
  functionality to get all existing keys.
- ``keyswithvalue``: Creates an index for values in specified tables,
  providing functionality to query keys by value.
- ``callwithsignature``: Provides functionality to call contracts using
  EIP712 signatures.
- ``std-delegations``: Provides several common delegation call patterns,
  including limiting by count, system, and time.
- ``puppet``: Provides functionality to create proxy contracts for
  system contracts, allowing system contracts to be called via proxy
  contracts.
- ``erc20-puppet``: An ``ERC20`` token contract implemented based on the
  ``puppet`` module.
- ``erc721-puppet``: An ``ERC721`` token contract implemented based on
  the ``puppet`` module.

Module Installation
-------------------

Installation via Configuration File
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We recommend installing modules by editing the ``mud.config.ts``
configuration file.

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";
  import { resolveTableId } from "@latticexyz/world/internal";
  import { encodeAbiParameters, parseAbiParameters, toHex } from 'viem';

  export default defineWorld({
    tables: {
      StoredUint: {
        schema:{
          value: "uint256",
        },
        key: [],
      },
    },
    modules: [
      {
        artifactPath: "@latticexyz/world-modules/out/KeysInTableModule.sol/KeysInTableModule.json",
        root: true,
        args: [resolveTableId("StoredUint")],
      },
      {
        artifactPath: "@latticexyz/world-modules/out/PuppetModule.sol/PuppetModule.json",
        root: true,
        args: [],
      },
      {
        artifactPath: "@latticexyz/world-modules/out/ERC20Module.sol/ERC20Module.json",
        root: false,
        args: [
          {type: "bytes", value: encodeAbiParameters(
            parseAbiParameters('bytes14 namespace, (uint8 decimals, string name, string symbol)'),
            [toHex("token", { size: 14 }), {decimals: 18, name: "muddoc", symbol: "MUDOC"}],
          )}
        ],
      },
    ],
  });

The configuration file above completes the installation of three modules:
``keysintable``, ``puppet``, and ``erc20-puppet``. It's important to note
the installation method for each module. Some modules support installation
as both root and regular modules, while others only support one method.
Before installing a module, check its supported installation methods.
Secondly, we need to provide necessary installation parameters. For example,
when installing the ``keysintable`` module, we need to specify which table
to track existing keys for. Similarly, when installing the ``erc20-puppet``
module, we need to provide the namespace of the ``ERC20`` token system, the
token's name, symbol, and decimals.

Module configuration options:

- ``artifactPath``: Path to the JSON file of the compiled module contract.
  Supports both local relative paths and package import paths.
- ``root``: Whether to install as a root module.
- ``args``: Parameters required for module installation.

Manual Installation
^^^^^^^^^^^^^^^^^^^

To manually install a module, call either the ``installModule`` or
``installRootModule`` method of the ``World`` contract, depending on the
installation method.

The following manual installation script does the same thing as the
configuration file above:

.. code-block:: solidity

  // SPDX-License-Identifier: MIT
  pragma solidity ^0.8.0;

  import { Script } from "forge-std/Script.sol";
  import { StoredUint } from "../src/codegen/index.sol";
  import { IWorld } from "../src/codegen/world/IWorld.sol";
  import { KeysInTableModule } from "@latticexyz/world-modules/src/modules/keysintable/KeysInTableModule.sol";
  import { PuppetModule } from "@latticexyz/world-modules/src/modules/puppet/PuppetModule.sol";
  import { registerERC20 } from "@latticexyz/world-modules/src/modules/erc20-puppet/registerERC20.sol";
  import { ERC20MetadataData } from "@latticexyz/world-modules/src/modules/erc20-puppet/tables/ERC20Metadata.sol";
  import { IERC20Mintable } from "@latticexyz/world-modules/src/modules/erc20-puppet/IERC20Mintable.sol";

  contract InstallModule is Script {
    function run(address worldAddress) public {
      // install `keysintable` module to count keys for `StoredUint` table
      IWorld(worldAddress).installRootModule(new KeysInTableModule(), abi.encode(StoredUint._tableId));
      // install `puppet` module
      IWorld(worldAddress).installRootModule(new PuppetModule(), new bytes(0));
      // install `erc20-puppet` module, create an `ERC20` token
      IERC20Mintable token = registerERC20(
        IWorld(worldAddress),
        bytes14("token"),
        ERC20MetadataData({ decimals: 6, name: "muddoc", symbol: "MUDOC" })
      );
    }
  }

Writing Modules
---------------

Here's an official `module writing tutorial <https://mud.dev/guides/modules>`_
available for reference.
