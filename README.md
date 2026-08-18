# Taxonomy of Documentation Violations

**TDV** is a language-scoped taxonomy of canonical documentation violations.

It provides stable, shared identifiers for documentation problems independently of the tools that detect them.

```text
TDV-PY-2001
│   │  │
│   │  └─ violation identifier
│   └──── language registry
└──────── Taxonomy of Documentation Violations
```

TDV is currently a **proposal**. Python is the first available language registry.

## Motivation

Documentation tools use independent rule namespaces. The same underlying problem may be reported under different identifiers by different tools, while superficially similar rules may represent different semantic predicates.

TDV provides a common semantic layer across those ecosystems.

A tool can keep its own :

* rule identifiers ;
* detection algorithms ;
* configuration ;
* severity model ;
* emission policy ;
* suppression and de-duplication behavior ;
* automatic fixes.

TDV does not replace those mechanisms. It provides canonical identifiers describing **what a documentation violation means**.

## Language registries

Each language has its own registry at the repository root :

```text
tdv-py.yml
tdv-ts.yml
tdv-rs.yml
...
```

Identifiers follow this form :

```text
TDV-<LANG>-<CODE>
```

For example :

```text
TDV-PY-2001
TDV-TS-2001
TDV-RS-2001
```

Numeric codes are local to their language registry. Two identifiers with the same numeric portion do not necessarily represent the same violation :

```text
TDV-PY-2001 ≠ TDV-TS-2001
```

Registries should nevertheless reuse the common TDV structure and equivalent semantic positions where doing so is natural.

Currently available :

| Language | Namespace | Registry                   |
| -------- | --------- | -------------------------- |
| Python   | `TDV-PY`  | [`tdv-py.yml`](tdv-py.yml) |

## Common taxonomy structure

TDV registries start from a common top-level organization where applicable :

| Range  | Area                                    |
| ------ | --------------------------------------- |
| `0xxx` | Parsing and validation                  |
| `1xxx` | Missing documentation                   |
| `2xxx` | Parameters                              |
| `3xxx` | Produced values                         |
| `4xxx` | Exceptions                              |
| `5xxx` | Structured objects                      |
| `6xxx` | Representation and documentation styles |
| `7xxx` | Quality and semantics                   |
| `8xxx` | Editorial policy and references         |
| `9xxx` | Language-specific concerns              |

This structure is a **common baseline**, not a requirement that every language use every range identically.

A language registry may :

* reuse a range when the underlying concept applies ;
* leave a range partially or entirely unused ;
* specialize a range to reflect language-specific semantics ;
* use the `9xxx` range for concerns that do not generalize cleanly across languages.

More detailed subdivisions are defined independently by each language registry.

## What a TDV rule represents

A TDV rule defines a **semantic documentation violation**, not a detection algorithm.

For example :

```yaml
TDV-XX-2001:
  name: some-documentation-violation
```

Different tools may detect the same semantic violation using different techniques and expose it under different native identifiers :

```yaml
TDV-XX-2001:
  name: some-documentation-violation
  implementations:
    tool-a:
      code: A123
    tool-b:
      code: B456
```

TDV therefore standardizes the identity and meaning of the violation rather than implementation behavior.

The taxonomy may also include analysis conditions required to represent cases where documentation cannot be evaluated reliably, such as parsing or validation failures. These occupy the `0xxx` family rather than being treated as ordinary documentation violations.

## Registry format

Each language registry is a YAML mapping whose keys are TDV identifiers.

A minimal entry contains a canonical name :

```yaml
TDV-XX-2001:
  name: undocumented-parameter
```

Additional metadata can describe the meaning of the violation, known implementations, semantic relationships, mappings, and automatic correction paths.

### `definition`

`definition` states, in plain language, what the violation means:

```yaml
TDV-XX-2001:
  name: undocumented-parameter
  definition: >
    A parameter present in the callable signature is absent from
    the effective documentation.
```

Definitions describe the semantic predicate, not a detection algorithm. When a rule is inspired by an existing tool, the definition is phrased independently of the tool's own wording.

### `implementations`

`implementations` records known native codes used by tools that detect the corresponding violation.

```yaml
TDV-XX-2001:
  name: some-documentation-violation
  implementations:
    tool-a:
      code: A123
    tool-b:
      code: B456
```

These mappings allow tool-specific output to be related to TDV without requiring tools to adopt TDV as their native namespace.

### `mapping`

A `mapping` field may qualify a relationship when an implementation-specific rule does not correspond straightforwardly to the canonical TDV predicate.

`mapping` takes one of a restricted set of values :

* `broader` — the implementation rule covers more than the canonical predicate ;
* `narrower` — the implementation rule covers less than the canonical predicate ;
* `overlap` — the implementation rule and the canonical predicate overlap partially ;
* `aggregate` — the implementation rule combines several canonical predicates ;
* `historical` — the implementation rule is historical or superseded ;
* `policy-dependent` — the correspondence holds only under a specific configuration or convention ;
* `review-required` — the correspondence needs human review before it can be trusted.

A free-form `note` may complement `mapping` to explain the qualification :

```yaml
TDV-XX-2001:
  name: undocumented-parameter
  implementations:
    tool-a:
      code: A123
      mapping: broader
      note: also covers keyword-only parameters
```

The absence of `mapping` means that no qualification is currently recorded. It should not be interpreted as a formal equivalence guarantee between implementations.

### `related`

Rules may declare semantic relationships with other rules :

```yaml
related:
  - code: TDV-XX-2002
    relation: overlaps
```

Relationships may express concepts such as specialization, aggregation, or overlap.

They describe the taxonomy itself. They do not prescribe emission precedence, suppression, or de-duplication behavior to implementations.

### `fixers`

Known automatic correction paths may be recorded :

```yaml
fixers:
  - some-fixer
```

The absence of `fixers` means only that no fixer is currently recorded.

## Scope

TDV concerns documentation associated with software and the relationship between that documentation and the program, API, interface, or behavior it describes.

Depending on the language, registries may cover areas such as :

* missing or incomplete documentation ;
* documented parameters and values ;
* types and annotations ;
* return, yield, or equivalent output contracts ;
* exceptions and error contracts ;
* structured objects and members ;
* documentation syntax and representation ;
* documentation conventions and styles ;
* factual accuracy ;
* semantic consistency and drift ;
* examples ;
* internal and external references ;
* editorial policy ;
* public API exposure ;
* language-specific documentation concerns.

Product, infrastructure, configuration, and source-code diagnostics that do not describe documentation or its analyzability are outside TDV's scope.

## Relationship to existing tools

TDV is not a linter, parser, formatter, documentation generator, or fixing engine.

It is intended to provide a common semantic vocabulary across tools and ecosystems.

A tool may :

* map its native rules to TDV ;
* emit TDV identifiers directly ;
* use TDV only for interchange or reporting ;
* implement only a subset of a registry ;
* detect additional violations not yet represented by TDV.

Mapping a tool rule to TDV does not imply that the tool must adopt any particular detection, severity, fixing, or reporting behavior.

## Stability

TDV identifiers are intended to be stable references.

Once published, an identifier should not be repurposed to represent a different semantic violation.

When distinctions between existing tools correspond to genuinely different semantic predicates, those distinctions should be preserved rather than collapsed merely to simplify the taxonomy.

Reserved ranges may be used to leave room for future additions without reorganizing existing identifiers.

## Adding a language registry

A new language registry should :

1. use the `TDV-<LANG>-<CODE>` namespace ;
2. be stored at the repository root as `tdv-<lang>.yml` ;
3. reuse the common top-level taxonomy where the concepts apply naturally ;
4. define language-specific subdivisions where necessary ;
5. avoid forcing concepts from another language where no meaningful equivalent exists ;
6. preserve semantic distinctions between existing ecosystem rules ;
7. record known mappings to established tools where available.

The goal is a **shared structure without forcing a universal ontology onto language-specific documentation models**.

## Contributing

Contributions may include :

* new canonical violations ;
* corrections to existing definitions ;
* mappings to additional tools ;
* mapping qualifications ;
* semantic relationships between rules ;
* known automatic fixers ;
* reserved-range proposals ;
* new language registries.

A new rule should represent a distinct semantic predicate rather than a particular detection technique or implementation detail.

## Repository layout

```text
.
├── README.md
├── LICENSE
├── tdv-py.yml
├── tdv-ts.yml
├── tdv-rs.yml
└── ...
```

Only existing registries are present in practice ; additional filenames above illustrate the intended layout.

## License

See [`LICENSE`](LICENSE).

