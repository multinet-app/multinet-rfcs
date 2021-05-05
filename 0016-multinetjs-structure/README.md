---
RFC: 0016
Author: Jacob Nesbitt
Created: 2021-05-05
Last-Modified: 2021-05-05
Status: accepted
---

# RFC 0016: MultinetJS Structure

Currently, the API surface for MultinetJS is flat, containing all functions at the top level. This clutters both the top level, as well as every function signature, as if you want to (for example) make several calls to the same workspace, you're always passing in that workspace name for every call, as well as your actual function parameters. I propose a structure that contains classes for the relevant models in Multinet, with each class containing its relevant functions.

## Proposals

### Classes and Interfaces

The following classes would be added:

* `Workspace`
* `Table`
* `Network`

The following interfaces would be added:
* `Node`
* `Edge`


The following demonstrates how such a structure would function. I've added manual type information for illustration.
```typescript
// Makes no calls, just updates the baseURL
const workspace: Workspace = api.workspace('test');
const table: Table = workspace.table('table');

// Actual calls are made
const rows: Node[] = table.rows();
const metadata = table.metadata();

// Makes no calls, just updates the baseURL
const network: Network = workspace.network('network');

// Actual calls are made
const edges: Edge[] = network.edges();
const nodes: Node[] = network.nodes();
```

Notably, construction of these classes doesn't involve any api calls. Instead, it just updates the base URL, for any methods that exist on that class. Given the following example

```typescript
const api = multinetApi('localhost:8000/api/');
const workspace = api.workspace('test');
```
The base URL for any methods under `workspace` would be `localhost:8000/api/workspaces/test/`.

### Interface Generics

When it comes to the previously mentioned interfaces, these should be implemented as generic interfaces. Since individual users may have their own domain specific interfaces, allowing extension in this way will be useful. E.g. `Node<SomeObject>`.

## Backwards Compatibility

This RFC is not backwards compatible with the current MultinetJS *API surface*, but the return type for each function should remain unchanged.
