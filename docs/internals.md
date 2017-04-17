# Internals of `libtoml`

## The TOML data structure

After being parsed, the TOML document is represented in memory as a tree of `toml_node`s.

The internal structure of a `toml_node` is the following (from `toml_private.h`):
```c
struct toml_node {
	enum toml_type type;
	char *name;
	union {
		struct list_head map;
		struct list_head list;
		int64_t integer;
		struct {
			double value;
			int precision;
		} floating;
		char *string;
		struct {
			time_t epoch;
			int sec_frac;
			bool offset_sign_negative;
			uint8_t offset;
			bool offset_is_zulu;
		} rfc3339_time;
	} value;
};
```

### Linked lists

The `map` and `list` fields are linked lists implemnted using the `ccan/list` data structure.
The relevant structures (from `ccan/list/list.h`) are:
```c
struct list_node
{
	struct list_node *next, *prev;
};

struct list_head
{
	struct list_node n;
};
```

The actual data of each list element is wrapped around each `list_node` using
the following types:

```c
struct toml_table_item {
	struct list_node map;
	struct toml_node node;
};

struct toml_list_item {
	struct list_node list;
	struct toml_node node;
};
```

The macros `container_of` and `container_of_var` from `ccan/list` can be used to "dereference"
to the TOML structure corresponding to a `list_head`, e.g.:
```c
struct list_node * node = ...; // node to be dereferenced
struct toml_table_item * item = container_of(node, struct toml_table_item, map);
```
There are also routines for looping over the linked list etc.

If the `list_node` of the `list_head` points to itself then the list is empty
and this particular `list_node` does not represent a value in the list (i.e. there is nothing
wrapped around it and trying to "dereference" that will produce nonsense).
The first element of the list (if there is one) will the referenced by `head.n->next`.

### `map` vs `list`

Tables and the root node are dictionaries and the children are stored in `.map`.
Currently this is a semantic difference since the implementation is technically
identical to `.list`.

The linked list contains an ordered list of `toml_node`s, each corresponding to an element
in the dictionary. `.name` contains the key and `.value` contains the value.


### TOML nodes

Each TOML node has a type, which can be accessed using the `toml_get_type` function.
Internally it is stored as a `struct toml_type` in the `toml_node->type` field.
The intepretation of the rest of the `toml_node` fields depends on the type of the node.

  * `TOML_ROOT`: the root node of a document.

  In this case, `name` is not available (internally stored as a null-pointer).
  `toml_get_name` will fail with an error.
