Tables
======

**Any data that needs to be stored in the contract state must be implemented
through tables.**

**Tables are defined and created through configuration files.**

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
  The first string, of length 2, indicates the resource type, including
  on-chain table ``tb``, off-chain table ``ot``, system ``sy``, namespace
  ``ns``.
  The second string, of length 14, represents the name of the namespace where
  the resource is located.
  The third string, of length 16, represents the name of the resource.

The table configuration items are as follows:

- ``schema``: ``object``, field definition of the table, key-value pairs of
  field names and types.
- ``key``: ``string[]``, primary key of the table, can be an array of one or
  more fields. It can also be an empty array, meaning it's a singleton table.
- ``type``: ``string`` (optional), type of the table. ``table`` (default,
  stored on-chain) or ``offchainTable`` (table data can only be retrieved
  off-chain through ``event``).
- ``codegen``: ``object`` (optional), 3.

  - ``outputDirectory``: ``string``, default: ``"tables"``. Output directory
    for code generation, by default in ``src/tables`` under the configuration
    file directory.
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

Supported Table Field Types
^^^^^^^^^^^^^^^^^^^^^^^^^^^

+------------------+--------------------------------------------------+
| Type             |                                                  |
+==================+==================================================+
|| Value Types     || ``uint8`` ~ ``uint256``, ``int8`` ~ ``int256``, |
||                 || ``address``, ``bool``, ``bytes1`` ~ ``bytes32`` |
||                 || ``enum``                                        |
+------------------+--------------------------------------------------+
|| Reference Types || Fixed-length or dynamic arrays of value types,  |
||                 || ``string``, ``bytes``                           |
+------------------+--------------------------------------------------+
| Custom Types     | Aliases for value types / reference types        |
+------------------+--------------------------------------------------+
| ``mapping``      | ❌                                               |
+------------------+--------------------------------------------------+
| ``string[]``     | ❌                                               |
+------------------+--------------------------------------------------+
| ``bytes[]``      | ❌                                               |
+------------------+--------------------------------------------------+
| ``struct``       | ❌                                               |
+------------------+--------------------------------------------------+

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
