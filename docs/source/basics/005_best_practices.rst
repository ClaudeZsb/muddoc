.. _best-practices:

################
 Best Practices
################

****************
 Gas Efficiency
****************

Choose Smaller Data Types
=========================

Although Mud can help us save overall storage space through compact
encoding when storing multiple data points, when a table field is
defined as ``uint256``, it still requires at least one ``slot`` of space
to store. So, while preserving future expandability, we can try to use
smaller data types to save storage space.

.. code:: ts

   Users: {
     schema: {
       id: "uint256",
       age: "uint256",
       weight: "uint256",
       height: "uint256",
     },
     key: ["id"],
   },

In the ``Users`` table above, we defined three fields of type
``uint256``, which means each user occupies 3 ``slots`` of space. If we
believe that the ``age``, ``weight``, and ``height`` fields, in most
cases, will not exceed the range of ``uint32``:

.. code:: ts

   Users: {
     schema: {
       id: "uint256",
       age: "uint32",
       weight: "uint32",
       height: "uint32",
     },
     key: ["id"],
   },

We can define their types as ``uint32``, so each user only needs to
occupy 1 ``slot`` of space. And this one ``slot`` of space is not fully
filled, so we can still add more fields to the ``Users`` table in the
future without significantly affecting the read and write costs of the
original fields.

Read and write all fields of a table at once whenever possible
==============================================================

If a table contains at least two fields, the automatically generated
code library will not only generate a ``get()`` method, but also
``get<Fieldname>()`` methods. Similarly, in addition to the ``set()``
method, ``set<Fieldname>()`` methods will also be generated.

In a single system call, if multiple fields of the same table need to be
read or written, try to use the ``get()`` and ``set()`` methods as much
as possible. Compared to using ``get<Fieldname>()`` and
``set<Fieldname>()`` methods consecutively, this approach saves more
gas.

.. code:: ts

   T: {
     schema: {
       id: "uint256",
       F1: "uint32",
       F2: "uint32",
     },
     key: ["id"],
   },

   // gasCostOf(T.getF1() + T.getF2()) > gasCostOf(T.get())
   // gasCostOf(T.setF1() + T.setF2()) > gasCostOf(T.set())

.. note::

   The more fields that need to be read or written in the same table,
   the more gas is saved by using ``get()`` and ``set()`` to read and
   write all fields at once.

.. important::

   If only one field of the table needs to be read or written, using the
   ``get<Fieldname>()`` and ``set<Fieldname>()`` methods is more
   gas-efficient.

Design table structure based on field usage frequency
=====================================================

If you have an object with many attributes, for example 10, deciding
whether to define each attribute as a separate table or merge them into
a single table becomes a trade-off issue.

Obviously, defining each attribute as a separate table makes the table
structure clearer and easier to understand. However, each table occupies
at least one ``slot`` of space, which significantly increases the cost
of reading and writing.

On the other hand, combining all fields into one table indiscriminately,
while saving storage space, may not necessarily save on read and write
costs. It can make the table structure bloated, difficult to understand
and maintain.

From a gas-saving perspective, we can distribute the fields into
different tables based on their usage frequency, and reduce the cost of
table read and write operations in each system call by using the
``get()`` and ``set()`` methods as much as possible. Moreover, this
classification method often aligns with categorization based on the
inherent meaning of the fields, without adding extra complexity to
understanding the tables. For example, the length, width, and height
attributes of a house are all inherent properties of the house and are
usually used simultaneously in most cases.

.. note::

   How to summarize and organize fields and design table structures
   needs to be considered in the context of specific business scenarios.
   There is no uniform standard. Categorizing fields based on their
   usage frequency is a design approach that aligns more with saving
   gas.

.. important::

   If a field's type is a :ref:`reference type <field-supported-types>`,
   it is more suitable to define it as a separate table.

   If there are other fields, whether numeric or reference types, that
   are always used together with it, they are suitable to be defined in
   the same table.

Using ``IWorld.call()`` to invoke system contracts
==================================================

We are typically accustomed to using the generated world interface from
the CLI to call system contracts. For example:

.. code:: solidity

   // SpawnSystem is a system contract under the root namespace, providing a spawn() method
   //   Call from outside the world, or from a non-root system contract
   IWorld(worldAddress).spawn();
   //   Call from within the world, e.g., from a root system contract
   worldAddress.delegatecall(abi.encodeCall(IWorld(worldAddress).spawn, ()));

   // ListSystem is a system contract under the muddoc namespace, providing a list() method
   IWorld(worldAddress).mudddoc_list();

This calling method is simple and clear.

If you wish to save more gas, you can also use the ``IWorld.call()``
method to invoke system contracts. This approach saves gas by explicitly
specifying the system resource ID and method call parameters, thus
avoiding the lookup of corresponding system resources and system
function selectors through registered autonomous world function
selectors. For example:

.. code:: ts

   // SpawnSystem is a system contract under the root namespace, providing a spawn() method
   //   Call from outside the world, or from a non-root system contract
   IWorld(worldAddress).call(
     WorldResourceIdLib.encode("sy", "", "SpawnSystem"),
     abi.encodeCall(SpawnSystem.spawn, ())
   );
   //   Call from within the world, e.g., from a root system contract
   worldAddress.delegatecall(
     abi.encodeCall(
       IWorld(worldAddress).call,
       (
         WorldResourceIdLib.encode("sy", "", "SpawnSystem"),
         abi.encodeCall(SpawnSystem.spawn, ())
       )
     )
   );

   // ListSystem is a system contract under the muddoc namespace, providing a list() method
   IWorld(worldAddress).call(
     WorldResourceIdLib.encode("sy", "muddoc", "ListSystem"),
     abi.encodeCall(ListSystem.list, ())
   );
