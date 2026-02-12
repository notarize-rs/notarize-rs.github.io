# Terminology

Quick reference for terms used in this documentation.

## Core Concepts

| Term            | Definition                                                               |
| --------------- | ------------------------------------------------------------------------ |
| **Process**     | Something with inputs and outputs—computational, business, or physical.  |
| **Observer**    | Wraps a process and produces observations about it.                      |
| **Observation** | A structured record produced by an observer—a collection of properties.  |
| **Property**    | A named value within an observation. The atomic unit of trust decisions. |

## Relationships

| Term       | Scope                              | Definition                                                                      |
| ---------- | ---------------------------------- | ------------------------------------------------------------------------------- |
| **Link**   | Property → Node                    | A parent's property guarantees a child's complete integrity. Links form chains. |
| **Pin**    | Property → Property (same node)    | Compares properties within a single observation.                                |
| **Bond**   | Property → Property (same chain)   | Compares properties across observations within the same chain.                  |
| **Bridge** | Property → Property (cross-domain) | Compares properties across observation domains.                                 |

## Structure

| Term                   | Definition                                                                                           |
| ---------------------- | ---------------------------------------------------------------------------------------------------- |
| **Chain**              | A sequence of observations connected by links. Each parent guarantees the next child's integrity.    |
| **Observation Domain** | A chain rooted at an explicitly trusted observer. A coherent region of trust with unified authority. |
| **Federation**         | The collection of independent observation domains.                                                   |
| **Tree**               | When chains branch—one parent, multiple children—the structure becomes a tree.                       |

## Observations

| Term                      | Definition                                                                             |
| ------------------------- | -------------------------------------------------------------------------------------- |
| **Non-terminal**          | An observation that endorses another observer. It links to a child.                    |
| **Terminal**              | An observation that ends the chain. It observes something other than another observer. |
| **Critical property**     | A property that constrains what the child can legitimately claim.                      |
| **Non-critical property** | Informational property, not enforced during trust evaluation.                          |
