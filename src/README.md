# notarized

A CA that evaluates observation graphs against policies and issues certificates
for trust decisions. Call it an attestation verifier if you want. But it's
broader—a system for reasoning about trust while keeping that reasoning
transparent and auditable.

Observation graphs compose freely—chains within domains, bridges across them.
Policies must compose to match. But composition can obscure trust. What am I
actually trusting? Where does risk accumulate? The antidote is transparency:
composition that can be queried, audited, visualized.

To solve this problem, notarized provides three core components:

1. **A data model.** Observations are statements by observers about processes.
   They organize into chains, bridge across domains, compose into a federation.

2. **A policy language.** Constraints on observations: what properties must
   hold, how chains relate, what coverage is required.

3. **A verification engine.** Evaluates observations against policies, returns
   trust decisions. Not yet implemented.

## Transparent Computing

notarized implements the Transparent Computing paradigm.

Transparent Computing makes trust visible. Every trust decision traces to a
graph of observations: who observed what, when, and under what authority. The
graph can be queried, audited, visualized, and formally verified.

Trust is topology. Observations link into graphs. Policies are graph queries.
Audits are graph traversals. Risk is graph structure.

## Documentation

| Document                      | Contents                                                                   |
| ----------------------------- | -------------------------------------------------------------------------- |
| [Data Model](BACKGROUND.md)   | Processes, observers, observations. Chains, bridges, federation.           |
| [Policy Language](POLICY.md)  | Schema definitions, constraints, groups. Policy file format and semantics. |
| [Terminology](TERMINOLOGY.md) | Quick reference for terms used in this documentation.                      |

## Status

Design phase. The data model and policy language are under active development.
No implementation exists yet.
