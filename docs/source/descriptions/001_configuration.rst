Configuration
=============

The MUD configuration file is ``packages/contracts/mud.config.ts``.

The configuration file is mainly used for:

1. Defining the project's namespace ``namespace``.
2. Defining tables ``tables`` in the project.
3. Defining systems ``systems`` in the project.
4. Defining modules ``modules`` to be installed.
5. Configuring code generation
6. Configuring deployment

For detailed explanations about namespace, tables, systems, and modules in
the configuration file, please refer to their respective sections.

Here's a brief explanation of code generation and deployment configurations.

- ``codegen``: ``object``. Code generation configuration.

  - ``worldInterfaceName``: ``string``, default: ``"IWorld"``. Autonomous
    world interface name.
  - ``worldgenDirectory``: ``string``, default: ``"world"``. Directory for
    storing autonomous world interface.
  - ``worldImportPath``: ``string``, default: ``"@latticexyz/world/src"``.
    Import path for World protocol related code.
  - ``outputDirectory``: ``string``, default: ``"codegen"``. Output directory
    for generated code.
  - ``indexFilename``: ``string``, default: ``"index.sol"``. Table index
    filename.
  - ``storeImportPath``: ``string``, default: ``"@latticexyz/store/src"``.
    Import path for Store protocol related code.
  - ``userTypesFilename``: ``string``, default: ``"common.sol"``. User types
    filename.
- ``deploy``: ``object``. Deployment configuration.

  - ``postDeployScript``: ``string``, default: ``"PostDeploy"``. Name of the
    script to run after deployment. Script filename must end with ``.s.sol``.
  - ``deploysDirectory``: ``string``, default: ``"deploys"``. Directory for
    storing deployment information.
  - ``worldsFile``: ``string``, default: ``"worlds.json"``. A JSON file
    summarizing autonomous world addresses and their respective chains.
  - ``upgradeableWorldImplementation``: ``boolean``, default: ``false``.
    Whether to allow upgrading the core implementation of the autonomous
    world. Officially recommended to set to ``true``.

Complete configuration file example:

.. code-block:: ts

  import { defineWorld } from "@latticexyz/world";
  import { encodeAbiParameters, parseAbiParameters, toHex } from 'viem';

  export default defineWorld({
    // should be consistent with foundry's configuration
    sourceDirectory: "src",
    namespace: "muddoc",
    // for multiple namespaces, which is conflict with `namespace`
    namespaces: {
      muddoc: {
        tables: {},
        systems: {},
      },
    },
    enums: {
      UserStatus: ["active", "inactive"],
    },
    userTypes: {
      ShortString: {
        type: "bytes32",
        filePath: "@openzeppelin/contracts/utils/ShortStrings.sol",
      }
    },
    tables: {
      Users: {
        name: "Users",
        type: "table",
        schema: {
          addr: "address",
          data: "uint256",
          description: "string",
        },
        key: ["addr"],
        codegen: {
          outputDirectory: "tables",
          dataStruct: false,
          tableIdArgument: false,
          storeArgument: false,
        },
        deploy: {
          disabled: true,
        },
      },
    }
    systems: {
      SimpleStorageSystem: {
        name: "SimpleStorage",
        openAccess: false,
        accessList: [],
        deploy: {
          disabled: false,
          registerWorldFunctions: true,
        },
      },
    },
    excludeSystems: [],
    modules: [
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
    codegen: {
      worldInterfaceName: "IWorld",
      worldgenDirectory: "world",
      worldImportPath: "@latticexyz/world/src",
      outputDirectory: "codegen",
      indexFilename: "index.sol",
      storeImportPath: "@latticexyz/store/src",
      userTypesFilename: "common.sol",
    },
    deploy: {
      postDeployScript: "PostDeploy",
      deploysDirectory: "./deploys",
      worldsFile: "./worlds.json",
      upgradeableWorldImplementation: false,
    },
  });
