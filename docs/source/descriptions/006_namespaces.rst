Namespaces
==========

Namespaces are used to partition and manage all registered systems and
table resources in the autonomous world, forming resource subsets of
systems and tables. Namespaces are closely related to permission control
in the autonomous world.

Creating a Namespace
--------------------

We typically use the configuration file ``mud.config.ts`` to create a
namespace.

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";

  export default defineWorld({
    namespace: "muddoc",
    tables: ...
    systems: ...
  });

The content of the above configuration file defines a namespace called
``muddoc``, and all tables and systems defined in ``tables`` and
``systems`` will be registered under this namespace.

Manual Creation
^^^^^^^^^^^^^^^

We often need to manually create a namespace when using modules.

.. code-block:: solidity

  IWorld(worldAddress).registerNamespace({namespaceId: WorldResourceIdLib.encodeNamespace("muddoc")});

Creating Multiple Namespaces
----------------------------

Starting from Mud ``2.1.0``, we can create multiple namespaces using a
single configuration file.

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";

  export default defineWorld({
    namespaces: {
      muddoc: {
        tables: ...
        systems: ...
      },
      fun: {
        tables: ...
        systems: ...
      }
    },
    // namespace: ... ❌
    // tables: ... ❌
    // systems: ... ❌
  });

When we use ``namespaces``, it means we've enabled multi-namespace mode,
and the ``namespace`` field will be disabled. Similarly, the ``tables``
and ``systems`` fields are also disabled; these fields need to be used
under each individual namespace in ``namespaces``. Each key-value pair
in ``namespaces`` defines a namespace. The key serves as the namespace
name, and the value is an object containing ``tables`` and ``systems``.

.. note::

  In multi-namespace mode, the usage of ``enums`` and ``userTypes``
  remains unchanged.

Using multi-namespace mode also requires us to organize our project
directory according to namespaces, for example:

.. code-block:: bash

  src
  ├── codegen
  │   └── world
  └── namespaces
      ├── fun
      │   ├── codegen
      │   │   └── tables
      │   └── systems
      └── app
          ├── codegen
          │   └── tables
          └── systems

The recommended location for systems is
``src/namespaces/<namespace-name>/systems/``.

Access Control
------------------

Without any access configuration, all addresses have some basic
access, including:

- Reading all tables
- Using all systems with public access enabled

For registered systems, in addition to basic access, they can also:

- Update all tables within the same namespace
- Use all systems within the same namespace, even if they don't have
  public access enabled

Any additional resource access beyond these designed rights requires
authorization from the namespace owner.

The basic process of access checking is:

1. (If accessing a system) Check if the system has public access enabled.
2. Check if the accessor has been granted access to the namespace of the
   access object.

  .. note::

    If an accessor has namespace access, they can access all systems and
    tables within that namespace. No separate authorization is needed
    for each system and table.

3. Check if the accessor has been granted access to the access object.

Access Management
^^^^^^^^^^^^^^^^^^

Access management include granting and revoking access. The
objects of access management are resources, including tables, systems,
and namespaces. The operator managing access must be the owner
of the namespace to which the operation object belongs.

.. code-block:: solidity

  // table type resource
  ResourceId tableId = WorldResourceIdLib.encode("tb", "muddoc", "Table1");
  // system type resource
  ResourceId systemId = WorldResourceIdLib.encode("sy", "muddoc", "System1");
  // namespace type resource
  ResourceId namespaceId = WorldResourceIdLib.encodeNamespace("muddoc");

  // grant access to a resource
  IWorld(worldAddress).grantAccess({
    resourceId: specificResourceId,
    grantee: granteeAddress
  });

  // revoke access to a resource
  IWorld(worldAddress).revokeAccess({
    resourceId: specificResourceId,
    grantee: granteeAddress
  });

.. note::

  Granting access to a namespace is equivalent to granting access to all
  resources within that namespace.

.. important::

  If an address has been granted access to individual systems or tables,
  revoking its access to the namespace won't affect its access to these
  individual resources.

  In this case, if you want to prohibit access to all internal
  resources, you need to revoke access to each previously
  individually authorized resource.

Namespace Ownership Management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The initial owner of a namespace is its creator. The namespace owner can
transfer or renounce ownership.

.. code-block:: solidity

  // Transfer namespace ownership
  IWorld(worldAddress).transferNamespaceOwnership({
    namespaceId: WorldResourceIdLib.encodeNamespace("muddoc"),
    newOwner: newOwnerAddress
  });

  // Renounce namespace ownership
  IWorld(worldAddress).renounceOwnership({
    namespaceId: WorldResourceIdLib.encodeNamespace("muddoc")
  });

.. note::

  When a namespace is created, it is granted access to its creator.

  When ownership changes, the original owner will revoke their own
  access to the namespace, and the new owner will be granted access.

.. important::

  If the original owner had granted themselves access to individual
  system or table resources, changing ownership won't cause them to lose
  access to these resources. If the new owner wishes to prohibit these
  accesses, they should manually revoke these access.
