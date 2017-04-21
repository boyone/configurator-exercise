# Configurator™

This repo is a programming exercise designed to practice [Discovery Testing](https://github.com/testdouble/contributing-tests/wiki/Discovery-Testing).

## System Overview

Configurator™ is a command-line tool that manages transforming, encrypting and decrypting configurations for a web application. It has two main functions: storage and retrieval of versioned archives from a key/value store webservice.

```
+-------------------+ +---------------------------------+ +--------------------------------------------+
| webapp configs    | | configuration.archive           | | key/value store webservice                 |
+-------------------+ +---------------------------------+ +--------------------------------------------+
| myapp             | | files[config1.json]=7Yxi3bzz9ls | | myapp[configs:0.0.1]=myapp.configs.archive |
|   config +----------> files[config2.yaml]=20xt639jfg1 +-> myapp[configs:0.0.2]=myapp.configs.archive |
|     config1.json  | | files[config3.js]=1337fhQwGads  | | myapp[configs:0.0.3]=myapp.configs.archive |
|     config2.yaml  | | ...                             | | ...                                        |
|     config3.js    | |                                 | |                                            |
+-------------------+ +---------------------------------+ +--------------------------------------------+
```

### Storage

`configurator store <app-identifier> <path-to-config-folder>`

The storage portion of the CLI might be most easily understood via the following pseudocode:

```
store(appId, path)
  readConfigsFrom(path)
  for each config
    encrypt(config)
  write(archive)
  keyValueStoreWebService.saveArchive(archive)
```

### Retrieval

`configurator fetch <app-identifier> [version:semver-string]`

The retrieval portion of the CLI might be most easily understood via the following pseudocode:

```
fetch(appId, version=latest)
  keyValueStoreWebService.getArchive(appId, version)
  for each config in archive
    decrypt(config)
    write(decryptedConfig, configFilename)
```

## Implementation

Your task is to implement a solution to Configurator™ using the [Discovery Testing Recursive Worfklow](https://github.com/testdouble/contributing-tests/wiki/Discovery-Testing#intro):

> Discovery Testing prescribes a recursive workflow for TDD, seeking to defining a **tree of objects** to implement a feature

> 1. Start by identifying an entry point and writing a collaboration test of it
> 2. For each dependency the first collaboration test identifies:
> 3. if it needs to be broken down further, write another collaboration test for it (e.g. GOTO 1)
> 4. if its task is a straightforward data transformation, implement it as a pure-function leaf node
> 5. if its task requires interaction with a third-party, implement a wrapper object

For Configurator™, that **tree of objects** _might_ look something like this:

```
                                +---------------+
                                |               |
                                | Configurator™ |
                                |               |
                                +-------+-------+
                                        |
                                +-------v-------+
               +----------------|      CLI      |----------------+
               |                +---------------+                |
               |                        |                        |
               |                        |                        |
+--------------v--------+  +------------v--------+  +------------v---------+
| parsesCommandLineArgs |  | storesConfiguration |  | fetchesConfiguration |
+-----------------------+  +-------------------+-+  +-+--------------------+
                                               |      |
                    +---------------------+    |      |    +----------------+
                    | readsConfigurations |<---+      +--->| fetchesArchive |
                    +---------------------+    |      |    +----------------+
                 +------------------------+    |      |    +------------------------+
                 | encryptsConfigurations |<---+      +--->| decryptsConfigurations |
                 +------------------------+    |      |    +------------------------+
                          +---------------+    |      |    +----------------------+
                          | writesArchive |<---+      +----| writesConfigurations |
                          +---------------+    |           +----------------------+
                          +---------------+    |
                          | storesArchive |<---+
                          +---------------+
```

### Notes / Hints

- encryption and decryption can be done via base64 encoding/decoding
- interaction with the key/value store webservice can happen via a flatfile if that makes it easier

## Discussion Points

- If interaction with the key/value store webservice happens asynchronously how does that impact our tests?
- Can we write our tests such that they don't differ significantly when working with sync or async code?
- Are there any collaborators missing in the tree above?
- Are there any leaf nodes missing in the tree above?
- Which 3rd party libraries, if any, do you anticipate needing to integrate with?
- How would writing tests for nodes in the tree in reverse-order (from the bottom-up) affect the design of our code?

## TODO

- spike out an actual simple key/val web service on something like firebase that can be regularly purged and used for this exercise
