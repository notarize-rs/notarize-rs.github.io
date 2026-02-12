# Transparent Computing Policy Language

A policy language for evaluating the interrelations of observation graphs.

## Overview

This document specifies a policy language for expressing and validating trust
relationships across federated observation graphs. The language provides:

- Schema definitions for observations in each domain
- Constraints within a single chain (branch to branch)
- Seam specifications between chains (cross-domain property matching)
- Coverage tracking for validation completeness
- Static visualization of trust topology
- Audit trails for institutional validation

## Model

Each node in an observation graph has:

- **Attributes:** Observer metadata as CBOR (analogous to X.509 certificate
  fields: subject, issuer, validity, extensions)
- **Observations:** Process observations as CBOR (domain-specific data this
  observer records)

Nodes are identified by a globally unique `(subject, issuer)` tuple, mirroring
X.509 semantics. The issuer is the parent observer; the subject identifies this
observer within the parent's scope.

## File Types

| File      | Provider                     | Purpose                                       |
| --------- | ---------------------------- | --------------------------------------------- |
| `.node`   | Observation domain authority | Declares node identity and observation schema |
| `.policy` | Domain authority or consumer | Specifies constraints on observation branches |
| `.group`  | Consumer                     | Composes policies into validation suites      |

## .node Files

A `.node` file declares that a node exists in an observation domain, its
identity, and the schema of its observations.

**inventory.node:**

```
schema for "software-inventory" issued by "inventory-ca" {
    min_version: int
    releases: [* {
        name: text
        expected_hash: bytes(32)
    }]
}
```

**attestation.node:**

```
schema for "attestation-report" issued by "vcek" {
    measurement: bytes(48)
    policy: bytes(8)
    report_data: bytes(64)
}
```

The filename becomes the canonical short name for referencing this node
elsewhere.

**Schema syntax:**

- Structure mirrors CBOR data shape
- Basic types: `int`, `text`, `bytes`, `bool`, `null`, `float`
- Size constraints: `bytes(32)`
- CBOR tags: `#6.0(text)`
- Occurrences: `*` (zero or more), `+` (one or more), `?` (optional)
- Choice: `int / text`
- Tuples: `[int, text, bytes]`
- Maps: `{ field: type, ... }`

## .policy Files

A `.policy` file specifies value constraints on branches of a node's
observations. One file may contain multiple policy blocks targeting different
nodes and branches.

**amd-milan.policy (vendor):**

```
policy for ark {
    # AMD Root Key constraints
}

policy for ask using ark {
    issuer_key: @ark.subject_key
}

policy for vcek using ark, ask {
    chip_id: bytes(64)
    tcb_version: >=@ask.min_tcb
}

policy for attestation.measurement using vcek {
    # Vendor-guaranteed measurement constraints
}
```

**my-workload.policy (consumer):**

```
policy for attestation.measurement using golden {
    @golden.expected
}

policy for attestation.policy {
    >=0x30000
}
```

**Policy block structure:**

```
policy for <node>.<branch> [using <import>, ...] {
    <constraints>
}
```

- `<node>` — References a `.node` file by short name
- `<branch>` — Path into the schema; this policy owns complete coverage of this
  branch
- `using` — Imports other nodes for cross-reference (seams)
- `<constraints>` — Structural constraints on fields within the branch

**Constraint syntax:**

| Form                         | Meaning                           |
| ---------------------------- | --------------------------------- |
| `field: value`               | Equality (naked value implies ==) |
| `field: >=value`             | Greater or equal                  |
| `field: <=value`             | Less or equal                     |
| `field: >value`              | Greater                           |
| `field: <value`              | Less                              |
| `field: in @import.list`     | Membership                        |
| `field: subset @import.list` | Subset                            |

**References:**

| Form                            | Meaning                                       |
| ------------------------------- | --------------------------------------------- |
| `@import.path`                  | Path into imported node's observations (seam) |
| `@import.path[.field == value]` | Filtered path                                 |
| `name`                          | Sibling field in current scope                |
| `.field`                        | Field of element being filtered               |

## .group Files

Groups compose policies and other groups with `all` / `any` semantics.

**production.group:**

```
all {
    amd-milan.policy
    my-workload.policy
    any {
        approved-v1.policy
        approved-v2.policy
    }
}
```

**Semantics:**

- `all { ... }` — Every child must pass
- `any { ... }` — At least one child must pass

## Path Resolution

Files are resolved via a search path, similar to systemd units:

```
/usr/share/vendor/nodes/
/usr/share/vendor/policies/
/etc/myorg/nodes/
/etc/myorg/policies/
/etc/myorg/groups/
```

Vendors ship node definitions and baseline policies; consumers add their own
constraints.

## Coverage

Each policy block declares a branch it owns. The block must fully constrain all
fields within that branch.

**Validation:**

- Union of all policy branches must cover entire schema
- Gaps (uncovered branches) are reported
- Overlaps (multiple policies claiming same branch) are errors

**Coverage tracking:**

- `all` groups: union of child coverage
- `any` groups: intersection of child coverage

## Architecture

The constraint evaluator receives only CBOR values and policy ASTs. It knows
nothing about observation graphs, chains, or seams. Resolution happens in a
layer above:

```rust
fn validate(
    target: &CborValue,                      // observations being validated
    bindings: &HashMap<String, CborValue>,   // imported observations (resolved seams)
    policies: &[Policy],                     // constraint ASTs
) -> ValidationResult
```

This separation means:

- Schema and policy files can be tested in isolation
- Visualization works without concrete data
- Institutional boundaries stay clear

## Visualization

The AST structure supports static analysis:

1. **Node files** → Observation domain topology
2. **Policy files** → Branch ownership, seam definitions
3. **Group files** → Composition structure

From this we can generate observation schema diagrams, seam relationship graphs,
coverage maps by institution, trust flow visualizations. All without runtime
data.

---

## Open Questions

### Constraint Language

- Filter semantics when results are 0 or >1 matches
- Nested structural depth in policy blocks

### Path Expressions

- Array index vs integer map key disambiguation
- Bytes as map keys

### Coverage

- Branch granularity (can you split `entries` from `entries[*].hash`?)
- Optional field coverage requirements
