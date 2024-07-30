Tables
======

**Any data that needs to be stored in the contract state must be implemented
through tables.**

**Define and create tables through configuration files, and manipulate
tables using automatically generated table code libraries.**

Table Definition
----------------

This is a simple configuration file that defines a table named ``Users`` with
three fields: ``addr``, ``data``, and ``description``.
Their types are ``address``, ``uint256``, and ``string`` respectively, with the
``addr`` field serving as the primary key.

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";

  export default defineWorld({
    namespace: "muddoc",
    tables: {
      Users: {
        schema: {
          addr: "address",
          data: "uint256",
          description: "string",
        },
        key: ["addr"],
      },
    }
  });

In the configuration file ``mud.config.ts``, each key-value pair in ``tables``
defines a table. The key of the pair determines the table's name, while the
value is an object containing other configuration items for the table, mainly
including the definition of table fields and primary keys.

In the autonomous world defined by Mud, a table is a type of resource, and each
resource is identified by a unique ``ResourceId``.
The ``ResourceId`` of a table is called ``tableId``.
For example, the ``tableId`` of ``Users`` is
``0x74626d7564646f63000000000000000055736572730000000000000000000000``.
Here, ``7462`` is the hexadecimal form of ``tb``,
`online conversion tool <https://www.rapidtables.com/convert/number/ascii-to-hex.html>`_.
``6d7564646f63`` represents ``muddoc``, and ``5573657273`` represents ``Users``.

.. note::

  ``ResourceId`` is a ``bytes32`` data type, concatenated from three
  fixed-length string byte arrays.
  The first string, of length **2**, indicates the resource type, including
  on-chain table ``tb``, off-chain table ``ot``, system ``sy``, namespace
  ``ns``.
  The second string, of length **14**, represents the name of the namespace where
  the resource is located.
  The third string, of length **16**, represents the name of the resource.

The table configuration items are as follows:

- ``schema``: ``object``, field definition of the table, key-value pairs of
  field names and types.

  .. important::

    Each table can have up to **28** fields excluding the primary key, with a
    maximum of **5** :ref:`reference type <field-supported-types>` fields.

    **When defining, reference type fields must be placed at the end.**

- ``key``: ``string[]``, the table's primary key, can be an array of one or
  more fields. It can also be an empty array, indicating a singleton table.

  .. important::

    Primary key fields must be of :ref:`value types <field-supported-types>`.
    The number of primary key fields is mainly limited by EVM stack depth.
    Too many primary key fields will render the table's read/write methods
    unusable.

- ``type``: ``string`` (optional), type of the table. ``table`` (default,
  stored on-chain) or ``offchainTable`` (table data can only be retrieved
  off-chain through ``event``).
- ``codegen``: ``object`` (optional), 3.

  - ``outputDirectory``: ``string``, default: ``"tables"``. Output directory
    for code generation, by default in ``src/codegen/tables`` under the
    configuration file directory.
  - ``tableIdArgument``: ``boolean``, default: ``false``. Whether to generate
    ``tableId`` parameter for read and write methods.
  - ``storeArgument``: ``boolean``, default: ``false``. Whether to generate
    ``store`` parameter for read and write methods.

  .. note::

    When the same type of table (same field and primary key definition) exists
    in multiple namespaces, or exists with different names in the same
    namespace, you can adjust ``tableIdArgument`` to let the automatically
    generated code library operate different tables based on the passed
    ``tableId``.

    When this scenario extends to different autonomous worlds, you can further
    adjust ``storeArgument`` to introduce the ``store`` parameter.

  - ``dataStruct``: ``boolean``, default is ``true`` when there is more than
    one field that is not part of the primary key. Whether to generate a data
    structure for non-primary key fields of the table, default
    ``struct <TableName>Data``.

.. _field-supported-types:

Supported Table Field Types
^^^^^^^^^^^^^^^^^^^^^^^^^^^

+--------------------+--------------------------------------------------+
| Type               |                                                  |
+====================+==================================================+
|| Value Types       || ``uint8`` ~ ``uint256``, ``int8`` ~ ``int256``, |
||                   || ``address``, ``bool``, ``bytes1`` ~ ``bytes32`` |
+--------------------+--------------------------------------------------+
|| Reference Types   || Fixed-length or dynamic arrays of value types,  |
||                   || ``string``, ``bytes``                           |
+--------------------+--------------------------------------------------+
| Enums              | ✅                                               |
+--------------------+--------------------------------------------------+
| User-defined Types | ✅                                               |
+--------------------+--------------------------------------------------+
| ``mapping``        | ❌                                               |
+--------------------+--------------------------------------------------+
| ``string[]``       | ❌                                               |
+--------------------+--------------------------------------------------+
| ``bytes[]``        | ❌                                               |
+--------------------+--------------------------------------------------+
| ``struct``         | ❌                                               |
+--------------------+--------------------------------------------------+

.. important::

  It's not that the Mud framework can't read or write ``mapping``,
  ``string[]``, ``bytes[]``, ``struct`` type data, but rather these data
  types don't need to exist as table fields.

  If we want to implement a ``mapping(uint256 => address)`` type, we can
  create a table with two fields, with types ``uint256`` and ``address``
  respectively, and set the ``uint256`` field as the primary key.

  To implement ``string[]`` or ``bytes[]`` types, we can create a table with
  two fields, types ``uint256`` and ``string`` or ``bytes``, and set the
  ``uint256`` field as the primary key, representing the array index.

  The single row in each singleton table can be viewed as a piece of data of
  ``struct`` type.

Enums
"""""""""""""""""

We can define enums in the configuration file and use them in table fields.

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";

  export default defineWorld({
    namespace: "muddoc",
    enums: {
      UserStatus: ["active", "inactive"],
    },
    tables: {
      UserStates: {
        schema: {
          addr: "address",
          status: "UserStatus",
        },
        key: ["addr"],
      },
    }
  });

Each key-value pair in ``enums`` defines an enum. The key determines the
name of the enum, and the value is an array of strings containing all
enum member names.

All enums are generated and stored in ``src/common.sol`` by
``CLI: mud tablegen``.

User-defined Types
""""""""""""""""""

In the configuration file, we can import user-defined types via file paths and
use these imported user-defined types in table fields.

User-defined types need to be prepared in advance. ``CLI: mud tablegen``
automatically generates corresponding imports for the table code library based
on the import paths in the configuration file.

These user-defined types can come from either the current project or third-
party libraries.

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";

  export default defineWorld({
    namespace: "muddoc",
    userTypes: {
      MyUint256: {
        type: "uint256",
        filePath: "./src/utils/MyUint256s.sol",
      },
      ShortString: {
        type: "bytes32",
        filePath: "@openzeppelin/contracts/utils/ShortStrings.sol",
      }
    },
    tables: {
      UserStates: {
        schema: {
          addr: "address",
          data: "MyUint256",
          label: "ShortString",
        },
        key: ["addr"],
      },
    }
  });

``./src/utils/MyUint256s.sol`` is a relative path with respect to the
configuration file. Its content is roughly as follows:

.. code-block:: solidity

  // SPDX-License-Identifier: MIT
  pragma solidity >=0.8.24;

  type MyUint256 is uint256;

  library MyUint256s {
    // MyUint256 utils
  }

Table Definition Shorthand
^^^^^^^^^^^^^^^^^^^^^^^^^^

For convenience in defining tables with only one field or those not requiring
additional configuration, several shorthand methods can be used. Here, ``T*``
represents the shorthand table definition, while the corresponding ``Table*``
represents the equivalent complete table definition.

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";

  export default defineWorld({
    namespace: "muddoc",
    tables: {
      T1: "address",
      T2: "uint256[]",
      T3: "uint8[10]",
      T4: {
        id: "address",
        value: "uint256",
        data: "string",
      },
      Table1: {
        schema: {
          id: "bytes32",
          value: "address",
        },
        key: ["id"],
      },
      Table2: {
        schema: {
          id: "bytes32",
          value: "uint256[]",
        },
        key: ["id"],
      },
      Table3: {
        schema: {
          id: "bytes32",
          value: "uint8[10]",
        },
        key: ["id"],
      },
      Table4: {
        schema: {
          id: "address",
          value: "uint256",
          data: "string",
        },
        key: ["id"],
      },
    }
  });


Table Usage
-----------

The main operations on tables include creating, reading, updating, and
deleting. All operations rely on the code library generated by
``CLI: mud tablegen`` based on the table definitions. The code library for
each table is a separate ``solidity library`` named after the table,
containing the ``tableId``, table structure, and CRUD methods.

By simply importing the table's code library into the contract, you can
directly call the CRUD methods.

.. code-block:: solidity

  // SPDX-License-Identifier: MIT
  pragma solidity >=0.8.24;

  import { System } from "@latticexyz/world/src/System.sol";
  import { Users } from "../codegen/index.sol";

  contract TableOperationSystem is System {
    function CRUD() public {
      Users.register(); // Don't do this. It's just for demonstration purposes.
      (uint256 data, string memory description) = Users.get(address(0));
      Users.set(address(0), 1 /* data */, "address zero" /* description */);
      Users.deleteRecord(address(0));
    }
  }

- ``register()``, registers the table in the autonomous world. One-time
  operation.

  .. note::

    Tables defined in the configuration file are automatically registered
    during deployment, requiring no manual operation.

  .. note::

    ``register()`` is typically used in modules to register the table in the
    autonomous world where the module resides.

- ``get()``, ``set()``, read/write data by row. The ``codegen.dataStruct``
  config item in table definition affects ``get()``'s return type.
- ``get<Fieldname>()``, ``set<Fieldname>()``, read/write a single field.
- ``getItem<Fieldname>`` reads a reference type field element by index.
- ``update<Fieldname>``, updates a reference type field element by index.
- ``length<Fieldname>``, gets reference type field length, not for fixed
  arrays like ``uint8[4]``.
- ``push<Fieldname>``, ``pop<Fieldname>``, add/remove element at end of
  reference type field, not for fixed-length arrays.

Internal CRUD Methods
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When examining a table's library, you'll notice each CRUD method has a
similar counterpart with a different name. These methods start with ``_``,
like ``_register()``, conventionally indicating internal methods.
**Here, internal methods refer to those that, compared to the methods
mentioned above, can only be used within the context of the autonomous
world's main contract.**

.. note::

  These internal methods can be used in systems under the ``root`` namespace.
  If your project uses custom namespaces, avoid these internal methods.
  Don't worry about data security; using these methods will only result in
  errors or unexpected effects, without damaging project data.

CRUD Methods with ``tableId`` Parameter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In some cases, we need to distinguish tables using a ``tableId`` parameter.
In the config file, adding the ``codegen.tableIdArgument`` config item to
the required table definition introduces the ``tableId`` parameter to all
CRUD methods.

CRUD Methods with ``store`` Parameter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sometimes, we need to specify the autonomous world of the operated table
using a ``store`` parameter. In the config file, adding the
``codegen.storeArgument`` config item to the required table definition
generates an additional set of CRUD methods in the library with the ``store``
parameter. These methods have the same names without the ``_`` prefix.
