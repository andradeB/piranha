# PolyglotPiranha Python API

This document describes the main classes and functions exposed by the
`polyglot_piranha` Python package. It mirrors the Rust API used in the
command-line interface.

## execute_piranha

```python
from polyglot_piranha import execute_piranha, PiranhaArguments

result = execute_piranha(PiranhaArguments(language="java", code_snippet="..."))
```

Executes Piranha with the provided arguments and returns a list of
`PiranhaOutputSummary` objects representing the transformation applied to each
file.

## PiranhaArguments

`PiranhaArguments` captures the configuration for running Piranha. Fields marked
as optional use sensible defaults when omitted.

| Name | Type | Description |
| ---- | ---- | ----------- |
| `language` | `str` | Target language (e.g. `java`, `python`, `swift`) |
| `paths_to_codebase` | `list[str]` | Paths to source files or directories |
| `include` | `list[str]` | Glob patterns of files to include |
| `exclude` | `list[str]` | Glob patterns of files to exclude |
| `substitutions` | `dict[str, str]` | Values to instantiate rule variables |
| `path_to_configurations` | `str` | Directory containing `rules.toml` and `edges.toml` |
| `rule_graph` | `RuleGraph` | Custom rule graph built via the API |
| `code_snippet` | `str` | In‑memory code to transform instead of files |
| `dry_run` | `bool` | Disable in‑place rewrites |
| `cleanup_comments` | `bool` | Remove comments related to deleted code |
| `cleanup_comments_buffer` | `int` | Number of lines to search for comments |
| `number_of_ancestors_in_parent_scope` | `int` | Ancestors to consider for `Parent` scope |
| `delete_consecutive_new_lines` | `bool` | Collapse blank lines after edits |
| `global_tag_prefix` | `str` | Prefix for generated global tags |
| `delete_file_if_empty` | `bool` | Remove files that become empty |
| `path_to_output` | `str` | Where to write the JSON summary |
| `allow_dirty_ast` | `bool` | Allow parsing errors in input files |
| `should_validate` | `bool` | Validate the rule graph before running |
| `experiment_dyn` | `bool` | Internal feature flag |
| `path_to_custom_builtin_rules` | `str` | Override built‑in cleanup rules |

## Rule

Represents a single match/replace rule.

```python
Rule(
    name="replace_method",
    query="cs :[x].isEnabled()",
    replace_node="*",
    replace="true",
    is_seed_rule=True,
)
```

`query` can be a tree-sitter query, a regex (prefix with `rgx`), or a concrete
syntax template (prefix with `cs`).

## Filter

Filters restrict where a rule is applied.

```python
Filter(enclosing_node="cs class :[c] { :[body] }")
```

## OutgoingEdges and RuleGraph

`OutgoingEdges` connect rules in a `RuleGraph`. The `scope` determines where the
subsequent rules run (`Parent`, `Method`, `Class`, or `Global`).

```python
edge = OutgoingEdges("replace_method", to=["boolean_literal_cleanup"], scope="Parent")
rg = RuleGraph(rules=[rule], edges=[edge])
```

## PiranhaOutputSummary

The result produced by `execute_piranha` for each file.

```python
summary.path          # Path to the file
summary.content       # Final content
summary.matches       # Matches for match-only rules
summary.rewrites      # List of edits performed
```

See `demo/` for runnable examples of these APIs.
