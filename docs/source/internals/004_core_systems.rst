.. _internals_core_systems:

Core Systems
============

When developing or using autonomous worlds, most of the functionality we
interact with is implemented by the core systems within the Mud framework.
Their main functions are as follows:

- ``AccessManagementSystem``

  - :ref:`grantAccess <access-management-system-grant-access>`
  - :ref:`revokeAccess <access-management-system-revoke-access>`
  - :ref:`transferOwnership <access-management-system-transfer-ownership>`
  - :ref:`renounceOwnership <access-management-system-renounce-ownership>`

- ``BalanceTransferSystem``

  - :ref:`transferBalanceToNamespace
    <balance-transfer-system-transfer-balance-to-namespace>`
  - :ref:`transferBalanceToAddress
    <balance-transfer-system-transfer-balance-to-address>`

- ``BatchCallSystem``

  - :ref:`batchCall <batch-call-system-batch-call>`
  - :ref:`batchCallFrom <batch-call-system-batch-call-from>`

- ``ModuleInstallationSystem``

  - :ref:`installModule <module-installation-system-install-module>`

- ``StoreRegistrationSystem``

  - :ref:`registerTable <store-registration-system-register-table>`
  - :ref:`registerStoreHook <store-registration-system-register-store-hook>`
  - :ref:`unregisterStoreHook
    <store-registration-system-unregister-store-hook>`

- ``WorldRegistrationSystem``

  - :ref:`registerNamespace <world-registration-system-register-namespace>`
  - :ref:`registerSystemHook
    <world-registration-system-register-system-hook>`
  - :ref:`unregisterSystemHook
    <world-registration-system-unregister-system-hook>`
  - :ref:`registerSystem <world-registration-system-register-system>`
  - :ref:`registerFunctionSelector
    <world-registration-system-register-function-selector>`
  - :ref:`registerRootFunctionSelector
    <world-registration-system-register-root-function-selector>`
  - :ref:`registerDelegation
    <world-registration-system-register-delegation>`
  - :ref:`unregisterDelegation
    <world-registration-system-unregister-delegation>`
  - :ref:`registerNamespaceDelegation
    <world-registration-system-register-namespace-delegation>`
  - :ref:`unregisterNamespaceDelegation
    <world-registration-system-unregister-namespace-delegation>`

``AccessManagementSystem``
---------------------------------------------

The Access Management System is used to manage access control for resources
in the World. It handles granting and revoking access to resources, as well
as transferring and renouncing namespace ownership.

.. _access-management-system-grant-access:

``grantAccess``
^^^^^^^^^^^^^^^^^^^^^^

Grants access to a resource for a specific address.

.. code-block:: solidity

  function grantAccess(ResourceId resourceId, address grantee)

- Requires caller to own the namespace containing the resource
- ``resourceId``: ID of the resource to grant access to
- ``grantee``: Address to grant access to
- Resource must exist

.. _access-management-system-revoke-access:

``revokeAccess``
^^^^^^^^^^^^^^^^^^^^^^^

Revokes access to a resource from a specific address.

.. code-block:: solidity

  function revokeAccess(ResourceId resourceId, address grantee)

- Requires caller to own the namespace containing the resource
- ``resourceId``: ID of the resource to revoke access from
- ``grantee``: Address to revoke access from

.. _access-management-system-transfer-ownership:

``transferOwnership``
^^^^^^^^^^^^^^^^^^^^^^^^^

Transfers ownership of a namespace to a new owner.

.. code-block:: solidity

  function transferOwnership(ResourceId namespaceId, address newOwner)

- Requires caller to own the namespace
- ``namespaceId``: ID of the namespace to transfer
- ``newOwner``: Address to transfer ownership to
- Resource must be a namespace and exist
- Revokes access from old owner and grants access to new owner

.. _access-management-system-renounce-ownership:

``renounceOwnership``
^^^^^^^^^^^^^^^^^^^^^^^^^

Renounces ownership of a namespace, effectively abandoning it.

.. code-block:: solidity

  function renounceOwnership(ResourceId namespaceId)

- Requires caller to own the namespace
- ``namespaceId``: ID of the namespace to renounce
- Resource must be a namespace and exist
- Removes namespace owner and revokes access from current owner

.. important::

  If the original owner previously granted themselves access to individual system or table resources, changing ownership will not cause them to lose access to these resources. If the new owner wants to prevent this access, they should manually revoke these permissions.

``BalanceTransferSystem``
-----------------------------------------

The Balance Transfer System facilitates balance transfers both within the World
(between namespaces) and from the World to external addresses.

.. _balance-transfer-system-transfer-balance-to-namespace:

``transferBalanceToNamespace``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Transfers balance from one namespace to another within the World.

.. code-block:: solidity

  function transferBalanceToNamespace(
    ResourceId fromNamespaceId,
    ResourceId toNamespaceId,
    uint256 amount
  )

- ``fromNamespaceId``: Source namespace to transfer from
- ``toNamespaceId``: Target namespace to transfer to
- ``amount``: Amount to transfer
- Requires caller to have access to source namespace
- Both namespaces must exist and be valid namespace types
- Source namespace must have sufficient balance
- Updates balances of both namespaces

.. _balance-transfer-system-transfer-balance-to-address:

``transferBalanceToAddress``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Transfers balance from a namespace to an external address.

.. code-block:: solidity

  function transferBalanceToAddress(
    ResourceId fromNamespaceId,
    address toAddress,
    uint256 amount
  )

- ``fromNamespaceId``: Source namespace to transfer from
- ``toAddress``: Target address to transfer to
- ``amount``: Amount to transfer
- Requires caller to have access to source namespace
- Source namespace must have sufficient balance
- Updates source namespace balance and transfers funds to target address
- Reverts if external transfer fails


``BatchCallSystem``
--------------------------------

The Batch Call System enables batching multiple system calls into a single
transaction for efficiency. It provides two main functions:

.. _batch-call-system-batch-call:

``batchCall``
^^^^^^^^^^^^^

Makes multiple system calls in a single transaction.

.. code-block:: solidity

  function batchCall(SystemCallData[] calldata systemCalls) returns (bytes[] memory returnDatas)

- ``systemCalls``: Array of system calls containing systemId and callData for
  each call
- ``returnDatas``: Array of return data bytes from each system call
- Executes each system call in sequence
- Reverts entire batch if any call fails

.. _batch-call-system-batch-call-from:

``batchCallFrom``
^^^^^^^^^^^^^^^^^

Makes multiple system calls from specified addresses in a single transaction.

.. code-block:: solidity

  function batchCallFrom(SystemCallFromData[] calldata systemCalls) returns (bytes[] memory returnDatas)

- ``systemCalls``: Array of system calls containing from address, systemId and
  callData for each call
- ``returnDatas``: Array of return data bytes from each system call
- Executes each system call in sequence using specified from address
- Reverts entire batch if any call fails


``ModuleInstallationSystem``
-----------------------------------------

The Module Installation System handles the installation of non-root modules
in the World. It provides functionality to install modules under specific
namespaces.

.. _module-installation-system-install-module:

``installModule``
^^^^^^^^^^^^^^^^^

Installs a non-rootmodule into the World under a specified namespace.

.. code-block:: solidity

  function installModule(IModule module, bytes memory encodedArgs)

- ``module``: The module contract to be installed
- ``encodedArgs``: ABI encoded arguments for module installation
- Validates that module implements IModule interface
- Registers module in InstalledModules table


.. important::

  The Module Installation System cannot install root modules. Root module installation is implemented by the ``World`` contract itself, not the Module Installation System.

``StoreRegistrationSystem``
-----------------------------------------

The Store Registration System handles registration of tables and store hooks
within the World. It provides functionality for registering tables with
specific schemas and layouts, as well as managing storage hooks for tables.

.. _store-registration-system-register-table:

``registerTable``
^^^^^^^^^^^^^^^^^

Registers a new table with specified configuration.

.. code-block:: solidity

  function registerTable(
    ResourceId tableId,
    FieldLayout fieldLayout,
    Schema keySchema,
    Schema valueSchema,
    string[] calldata keyNames,
    string[] calldata fieldNames
  )

- ``tableId``: Resource ID of the table to register
- ``fieldLayout``: Field layout structure for the table
- ``keySchema``: Schema for table keys
- ``valueSchema``: Schema for table values
- ``keyNames``: Names for the table keys
- ``fieldNames``: Names for the table fields
- Requires caller to own the namespace containing the table
- Table name cannot be the root name(empty string)

.. _store-registration-system-register-store-hook:

``registerStoreHook``
^^^^^^^^^^^^^^^^^^^^^

Registers a store hook for a specified table.

.. code-block:: solidity

  function registerStoreHook(
    ResourceId tableId,
    IStoreHook hookAddress,
    uint8 enabledHooksBitmap
  )

- ``tableId``: Resource ID of the table to register the hook for
- ``hookAddress``: Address of the hook contract
- ``enabledHooksBitmap``: Bitmap indicating which hook functionalities are
  enabled
- Requires caller to own the namespace containing the table
- Hook contract must implement the ``IStoreHook`` interface

.. _store-registration-system-unregister-store-hook:

``unregisterStoreHook``
^^^^^^^^^^^^^^^^^^^^^^^

Unregisters a store hook for a specified table.

.. code-block:: solidity

  function unregisterStoreHook(ResourceId tableId, IStoreHook hookAddress)

- ``tableId``: Resource ID of the table to unregister the hook from
- ``hookAddress``: Address of the hook contract to unregister
- Requires caller to own the namespace containing the table


``WorldRegistrationSystem``
-----------------------------------------

The World Registration System provides functions for registering resources
other than tables in the World.

.. _world-registration-system-register-namespace:

``registerNamespace``
^^^^^^^^^^^^^^^^^^^^^

Registers a new namespace.

.. code-block:: solidity

  function registerNamespace(ResourceId namespaceId)

- ``namespaceId``: Resource ID for the new namespace
- Namespace must not already exist
- Namespace name should be valid that not contain reserved characters like
  ``__`` and not end with ``_``

.. _world-registration-system-register-system-hook:

``registerSystemHook``
^^^^^^^^^^^^^^^^^^^^^^

Registers a system hook for a specified system.

.. code-block:: solidity

  function registerSystemHook(
    ResourceId systemId,
    ISystemHook hookAddress,
    uint8 enabledHooksBitmap
  )

- ``systemId``: Resource ID of the system to register the hook for
- ``hookAddress``: Address of the hook contract
- ``enabledHooksBitmap``: Bitmap indicating which hook functionalities are
  enabled
- Requires caller to own the namespace containing the system
- Hook contract must implement the ``ISystemHook`` interface

.. _world-registration-system-unregister-system-hook:

``unregisterSystemHook``
^^^^^^^^^^^^^^^^^^^^^^^^

Unregisters a system hook for a specified system.

.. code-block:: solidity

  function unregisterSystemHook(ResourceId systemId, ISystemHook hookAddress)

- ``systemId``: Resource ID of the system to unregister the hook from
- ``hookAddress``: Address of the hook contract to unregister
- Requires caller to own the namespace containing the system

.. _world-registration-system-register-system:

``registerSystem``
^^^^^^^^^^^^^^^^^^

Registers or upgrades a system at the given ID.

.. code-block:: solidity

  function registerSystem(ResourceId systemId, System system, bool publicAccess)

- ``systemId``: Resource ID for the system
- ``system``: The system contract being registered
- ``publicAccess``: Flag indicating if access control check is bypassed
- Requires caller to own the namespace containing the system
- Requires the system not registered before with different resource ID
- Can upgrade existing systems
- System must implement ``WorldContextConsumer`` interface

.. _world-registration-system-register-function-selector:

``registerFunctionSelector``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Registers a new World function selector.

.. code-block:: solidity

  function registerFunctionSelector(
    ResourceId systemId,
    string memory systemFunctionSignature
  ) returns (bytes4 worldFunctionSelector)

- ``systemId``: Resource ID of the system
- ``systemFunctionSignature``: Signature of the system function
- Requires the selector to be unique
- Creates mapping between World function and system function with namespace
  prefix
- Requires caller to own the namespace containing the system

.. _world-registration-system-register-root-function-selector:

``registerRootFunctionSelector``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Registers a root World function selector.

.. code-block:: solidity

  function registerRootFunctionSelector(
    ResourceId systemId,
    string memory worldFunctionSignature,
    string memory systemFunctionSignature
  ) returns (bytes4 worldFunctionSelector)

- ``systemId``: Resource ID of the system
- ``worldFunctionSignature``: Signature of the World function
- ``systemFunctionSignature``: Signature of the system function
- Requires the selector to be unique
- Creates mapping between World function and system function without namespace
  prefix
- Requires caller to own the root namespace

.. _world-registration-system-register-delegation:

``registerDelegation``
^^^^^^^^^^^^^^^^^^^^^^

Registers a delegation for the caller.

.. code-block:: solidity

  function registerDelegation(
    address delegatee,
    ResourceId delegationControlId,
    bytes memory initCallData
  )

- ``delegatee``: Address of the delegatee
- ``delegationControlId``: ID controlling the delegation
- ``initCallData``: Initialization data for the delegation
- Creates new delegation from caller to specified delegatee

.. _world-registration-system-unregister-delegation:

``unregisterDelegation``
^^^^^^^^^^^^^^^^^^^^^^^^

Unregisters a delegation.

.. code-block:: solidity

  function unregisterDelegation(address delegatee)

- ``delegatee``: Address of the delegatee to unregister
- Deletes the delegation from caller to specified delegatee

.. _world-registration-system-register-namespace-delegation:

``registerNamespaceDelegation``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Registers a delegation for a namespace.

.. code-block:: solidity

  function registerNamespaceDelegation(
    ResourceId namespaceId,
    ResourceId delegationControlId,
    bytes memory initCallData
  )

- ``namespaceId``: ID of the namespace
- ``delegationControlId``: ID controlling the delegation
- ``initCallData``: Initialization data for the delegation
- Sets up delegation control for a specific namespace
- Requires caller to own the namespace
- Delegation cannot be unlimited

.. _world-registration-system-unregister-namespace-delegation:

``unregisterNamespaceDelegation``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Unregisters a delegation for a namespace.

.. code-block:: solidity

  function unregisterNamespaceDelegation(ResourceId namespaceId)

- ``namespaceId``: ID of the namespace
- Deletes the delegation control for a specific namespace
- Requires caller to own the namespace

