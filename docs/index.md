# `libtoml` reference

## The public interface

* `enum toml_type`

  Defines the type of the TOML node.
Possible values are `TOML_ROOT`, `TOML_TABLE`, `TOML_LIST`, `TOML_INT`, `TOML_FLOAT`,
`TOML_STRING`, `TOML_DATE`, `TOML_BOOLEAN`, `TOML_TABLE_ARRAY`, `TOML_INLINE_TABLE`.

  In additional that there is the special value `TOML_MAX`. `toml_type` values should always
be smaller than this.

- `const char* toml_type_string(enum toml_type type)`

  Returns a string corresponding to the `toml_type`.

## Inspecting a TOML node

- `enum toml_type toml_get_type(struct toml_node*)`

  Returns the `toml_type` of the node.

- `const char * toml_get_name(struct toml_node*)`

  Returns the name of the node. May return a null-pointer.
