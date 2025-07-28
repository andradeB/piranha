# PolyglotPiranha

PolyglotPiranha is a lightweight toolset for automating code transformations at scale. Uber primarily uses it to remove code associated with stale feature flags.

Only languages used at Uber are officially supported in this repository. Additional forks might offer support for other languages.

## Installation

PolyglotPiranha can be used either as a Python library or via its command-line interface.

### Python

Install from PyPI:

```bash
pip install polyglot-piranha
```

### Command line

Download a platform specific binary from the [releases](https://github.com/uber/piranha/releases) page or build it from source:

1. Install [Rust](https://www.rust-lang.org/tools/install).
2. Clone the repository: `git clone https://github.com/uber/piranha.git`.
3. Run `cargo build --release` (use `--no-default-features` on macOS).
4. The binary will be located under `target/release`.

## Example

The snippet below shows how to apply a custom rewrite using the Python API.

```python
from polyglot_piranha import execute_piranha, PiranhaArguments, Rule, RuleGraph, OutgoingEdges

code = """
if (obj.isLocEnabled() || x > 0) {
    // do something
} else {
    // do something else!
}
"""

rule = Rule(
    name="replace_method",
    query="cs :[x].isLocEnabled()",  # 'cs' denotes concrete syntax
    replace_node="*",
    replace="true",
    is_seed_rule=True,
)

edge = OutgoingEdges("replace_method", to=["boolean_literal_cleanup"], scope="Parent")

args = PiranhaArguments(
    code_snippet=code,
    language="java",
    rule_graph=RuleGraph(rules=[rule], edges=[edge]),
)

result = execute_piranha(args)
print(result[0].content)
```

For more examples and details about the DSL, see [POLYGLOT_README.md](POLYGLOT_README.md).
Complete API documentation is available in [API.md](API.md).

## Feature flags

Feature flags help with gradual rollouts or experiments. Once a flag is obsolete the related code should be removed to avoid clutter and potential bugs. PolyglotPiranha can automate this process. The tool takes the flag name, its expected behaviour and a list of APIs interacting with that flag to perform the cleanup.

PolyglotPiranha (since May 2022) unifies support for multiple languages and feature flag APIs. Legacy language-specific implementations are available under [this tag](https://github.com/uber/piranha/releases/tag/last-version-having-legacy-piranha).

Additional resources:

- Research paper at [PLDI 2024](https://dl.acm.org/doi/10.1145/3656429)
- [Technical report](report.pdf) on Piranha at Uber
- [Blog post](https://eng.uber.com/piranha/)
- [6 minute video](https://www.youtube.com/watch?v=V5XirDs6LX8&feature=emb_logo)

## Support

Questions or bugs? [Open a GitHub issue](https://github.com/uber/piranha/issues).

## Piranha development

We maintain custom patches for several tree-sitter grammars. While these patches are upstreamed, discrepancies may exist. A playground for testing grammars and queries is available at:

https://uber.github.io/piranha/tree-sitter-playground/

## License

Piranha is licensed under the Apache 2.0 license. See the LICENSE file for details.

## Note

This is not an official Uber product and is provided as is.
