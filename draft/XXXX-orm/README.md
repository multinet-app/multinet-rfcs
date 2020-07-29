---
RFC: XXXX
Author: Jacob Nesbitt
Status: draft
Created: 2020-07-28
Last-Modified: 2020-07-28
---

# RFC XXXX: Multinet ORM

This RFC addresses the issue of organizing database operations into classes which interact with each other. This aims to reduce the complexity, while improving the clarity and extendability of our database operations.

## Motivation

Currently, our database operations are structured in a purely functional way. That is, all database operations are done through simple functions, with no use of classes. This becomes problematic for readability, as our current `db.py` file has grown to almost 600 lines long. This presents a risk, as lack of code readability/clarity likely leads to more mistakes/bugs being introduced.

Furthermore, due to the flat use of stateless functions, performance becomes an issue, as the same few functions are called repeatedly by multiple functions, as they share no state. The temporary solution to this was caching, but by introducing an ORM, much of the need for this caching will go away, as this state can be preserved in a class instance.

## Proposals

### Classes
The following subsections describe the classes that make up the Multinet ORM. Type annotations are included to more accurately describe the connections between the classes. If a type annotation is left out, it is either due to uncertainty, or because that method returns nothing useful.


##### User
```
# Previously used the term "cookie"
def get_session() -> str: ...
def set_session(): ...
def delete_session(): ...

# Serializes the object to JSON
def serialize() -> Dict: ...

# Static
def exists(user: UserInfo) -> bool: ...
def search(query: str) -> Iterable[User]: ...
def register(user: UserInfo) -> User: ...
def from_session(session_id: str) -> User: ...
```

##### Table
```
def rows(offset: int = 0, limit: int = 0) -> Iterable[Dict]: ...
def row_count() -> int: ...
def keys() -> Iterable[str]: ...
def is_edge() -> bool: ...
```

##### Graph
```
def nodes() -> Iterable[Dict]: ...
def node_tables() -> Iterable[Table]: ...
def edge_table() -> Table: ...
```


##### Workspace
```

def table(name: str) -> Table: ...
def tables() -> Iterable[Table]: ...

def graph(name: str) -> Graph: ...
def graphs() -> Iterable[Graph]: ...

def metadata() -> Dict: ...

def rename(new_name: str): ...
def delete(): ...

def create_table(name: str, rows: Iterable[Dict], edge: bool = False) -> Table: ...
def create_aql_table(name: str, aql: str) -> Table: ...
def delete_table(name: str): ...

def create_graph(
  name: str,
  edge_table: str,
  from_collections: Iterable[str],
  to_collections: Iterable[str],
) -> Graph: ...
def delete_graph(name: str): ...

def get_permissions() -> Dict: ...
def set_permissions(permissions: Dict): ...

# Static
def create(name: str) -> Workspace: ...
def exists(name: str) -> bool: ...
```


### Open Questions

* Should nodes be included in the ORM, or should they remain as dictionaries?
* Should `create`, `delete`, etc. be attached to the classes they are operating on? This was the original proposal, but after discussion was decided against. The reason it's mentioned here is to highlight the benefit of such a model, mainly:
  * Methods are more tightly coupled to the classes they are affecting
  * Methods are spread more evenly between classes, instead of `Workspace` containing the bulk of the operations

  However, the main reason against this approach is that it involves instantiation of objects possibly before they exist, which may be confusing.

## Backwards Compatibility

Ideally, this change has no effect on the API, as it is focused on changing the way our database operations are structured, not their behavior.
