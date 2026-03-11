# Annotated Bibliography: Trust Graph Evaluation via Composable Policy

## Thesis

Authorization and attestation are the same operation: a trusted entity makes a
signed observation, and trust propagates through a graph of such observations,
scoped by the semantics of each edge. The IETF RATS architecture's separation of
"evidence appraisal" from "authorization" is an artificial distinction that has
left the hard problem — composable policy for evaluating trust graphs with
cross-chain correlation — unsolved. This bibliography collects the foundational
works that support a unified graph-based model with Datalog-based composable
policy.

---

## I. Theoretical Foundations: "Speaks For" and Trust Propagation

### Lampson, Abadi, Burrows, Wobber — "Authentication in Distributed Systems: Theory and Practice" (1992)

- **Citation:** B. Lampson, M. Abadi, M. Burrows, E. Wobber. ACM Trans. Computer
  Systems 10(4), pp. 265–310, November 1992.
- **URL:** <https://dl.acm.org/doi/10.1145/138873.138874>
- **Full text:**
  <http://bwl-website.s3-website.us-east-2.amazonaws.com/45-AuthenticationTheoryAndPractice/Acrobat.pdf>
- **Relevance:** THE foundational paper. Introduces the "speaks for" relation
  (⇒) between principals: if A ⇒ B, then anything A says can be taken as said by
  B. Trust propagates through chains of speaks-for relationships. This is
  exactly the model needed: a VCEK "speaks for" AMD's root CA on matters of VM
  state; a TPM AK "speaks for" the TPM manufacturer on matters of platform
  measurement. The speaks-for relation is the edge type in the trust graph.
  Lampson's key insight: principals are identified by public keys, not names —
  names are secondary bindings. This eliminates the identity-centric assumptions
  that plague X.509 and RATS.

### Burrows, Abadi, Needham — "A Logic of Authentication" (1990)

- **Citation:** M. Burrows, M. Abadi, R. Needham. ACM Trans. Computer Systems
  8(1), pp. 18–36, February 1990. Expanded version in Proc. Royal Society A 426,
  1871 (Dec. 1989), pp. 233–271.
- **URL:** <https://dl.acm.org/doi/10.1145/77648.77649>
- **Relevance:** The BAN logic. Formalizes reasoning about beliefs in
  authentication protocols: "A believes X," "A said X," "A sees X." Provides the
  logical machinery for analyzing what trust claims actually mean when
  propagated through signed messages. The "A said X" construct maps directly to
  signed attestation claims. Important precursor: shows that authentication and
  authorization require _logical reasoning about statements_, not just signature
  verification.

### Abadi, Burrows, Lampson, Plotkin — "A Calculus for Access Control in Distributed Systems" (1993)

- **Citation:** M. Abadi, M. Burrows, B. Lampson, G. Plotkin. ACM Trans.
  Programming Languages and Systems 15(4), pp. 706–734, September 1993.
- **URL:** <https://dl.acm.org/doi/10.1145/155183.155225>
- **Full text:**
  <http://bwlampson.site/44-CalculusAccessControl/44-CalculusAccessControlOCR.htm>
- **Relevance:** Extends speaks-for into a formal algebraic calculus. Introduces
  compound principals: A ∧ B (conjunction — both must agree), A | B (quoting — A
  speaking on behalf of B), A for R (role restriction). The role restriction
  operator is critical: it formalizes _scoped trust_. A VCEK doesn't speak for
  AMD unconditionally — it speaks for AMD _in the role of attesting a specific
  chip's VM state_. This calculus provides the formal semantics for scoped trust
  propagation in the graph.

---

## II. Trust Management as a Distinct Problem

### Blaze, Feigenbaum, Lacy — "Decentralized Trust Management" (1996)

- **Citation:** M. Blaze, J. Feigenbaum, J. Lacy. Proc. 17th IEEE Symposium on
  Security and Privacy, pp. 164–173, May 1996.
- **URL:** <https://ieeexplore.ieee.org/document/502679/>
- **Full text:**
  <https://www.ieee-security.org/TC/SP2020/tot-papers/blaze-1996.pdf>
- **Relevance:** Names the trust management problem and argues it is distinct
  from both authentication and access control. The central question: "Is the key
  used to sign this request authorized to take this action?" PolicyMaker answers
  this by treating credentials and policies as programs in a general language.
  Key contribution for our purposes: the argument that trust management should
  be handled by a general-purpose engine, not ad-hoc per-application logic —
  which is exactly what RATS failed to provide by declaring appraisal policy out
  of scope.

### Blaze, Feigenbaum, Keromytis — "KeyNote: Trust Management for Public-Key Infrastructures" (1998/1999)

- **Citation:** M. Blaze, J. Feigenbaum, A. Keromytis. Security Protocols, LNCS
  1550, Springer, 1998. Also IETF RFC 2704, September 1999.
- **URL:** <https://link.springer.com/chapter/10.1007/3-540-49135-X_9>
- **Relevance:** KeyNote refines PolicyMaker into a practical credential
  language with defined semantics. Credentials are signed assertions about what
  keys are authorized to do, with conditions. Li & Mitchell later showed KeyNote
  is less expressive than RT₁_C and less tractable in the worst case — a
  cautionary result for language design. Important negative example: overly
  general policy languages sacrifice tractability.

### Blaze, Feigenbaum, Strauss — "Compliance Checking in the PolicyMaker Trust Management System" (1998)

- **Citation:** M. Blaze, J. Feigenbaum, M. Strauss. Proc. Financial
  Cryptography '98, LNCS 1465, pp. 254–274, Springer, 1998.
- **URL:** <https://link.springer.com/chapter/10.1007/BFb0055488>
- **Relevance:** Proves that PolicyMaker's compliance-checking problem is
  undecidable in its most general form and NP-hard even under natural
  restrictions. Identifies a polynomial-time special case. This is the key
  tractability result: if you let policy be arbitrary programs (as PolicyMaker
  does), you lose decidability. Motivates the restriction to Datalog, which
  guarantees polynomial-time evaluation via fixpoint semantics.

---

## III. SPKI/SDSI: Authorization Certificates and Key-Centric Identity

### Ellison, Frantz, Lampson, Rivest, Thomas, Ylonen — "SPKI Certificate Theory" (1999)

- **Citation:** C. Ellison, B. Frantz, B. Lampson, R. Rivest, B. Thomas, T.
  Ylonen. IETF RFC 2693, September 1999.
- **URL:** <https://www.rfc-editor.org/rfc/rfc2693>
- **Relevance:** SPKI/SDSI is the closest historical precursor to the trust
  graph model. Principals are public keys (not names). Two certificate types:
  name certificates (bind local names to keys — analogous to property-to-node
  connections) and authorization certificates (delegate specific permissions
  from one key to another — analogous to property-to-property with scoping).
  SPKI explicitly treats authorization as the primary function of certificates,
  with naming as secondary. This inverts X.509's priorities and aligns with the
  thesis that attestation IS authorization.

### Rivest, Lampson — "SDSI: A Simple Distributed Security Infrastructure" (1996)

- **Citation:** R. Rivest, B. Lampson. Available at
  <http://theory.lcs.mit.edu/~rivest/sdsi11.html>
- **URL:** <http://groups.csail.mit.edu/cis/sdsi.html>
- **Relevance:** Introduces local name spaces anchored to public keys. Each key
  defines its own name space; names are resolved by following certificate
  chains. This is the graph structure: keys are nodes, name certificates are
  edges, and name resolution is graph traversal. The merger with SPKI adds
  authorization semantics to this graph.

### Li, Mitchell — "Understanding SPKI/SDSI Using First-Order Logic" (2003)

- **Citation:** N. Li, J. Mitchell. Proc. 16th IEEE Computer Security
  Foundations Workshop, pp. 89–103, June 2003. Extended version in International
  Journal of Information Security 5(2), pp. 95–109, 2006.
- **URL:** <https://link.springer.com/article/10.1007/s10207-005-0073-0>
- **Relevance:** CRITICAL CAUTIONARY RESULT. Provides FOL semantics for
  SPKI/SDSI and proves that RFC 2693's standard proof procedure is _semantically
  incomplete_ — there exist valid authorization chains that the specified
  algorithm cannot discover. Also shows that authorization tag intersection (how
  SPKI composes permissions along chains) is algorithmically problematic, making
  a complete proof procedure unlikely. This directly motivates using Datalog
  (where evaluation is complete by construction) rather than ad-hoc proof
  procedures.

### Clarke, Elien, Ellison, Fredette, Morcos, Rivest — "Certificate Chain Discovery in SPKI/SDSI" (2001)

- **Citation:** D. Clarke, J.-E. Elien, C. Ellison, M. Fredette, A. Morcos, R.
  Rivest. Journal of Computer Security 9(4), pp. 285–322, 2001.
- **Relevance:** Algorithms for discovering proof chains in SPKI/SDSI's
  certificate graph. Formalizes the graph search problem: given a set of
  certificates and an authorization query, find a subgraph that constitutes a
  proof. The credential graph representation and reachability-based proof
  discovery map to trust graph evaluation.

---

## IV. Datalog and the RT Framework: Formal Foundations for Composable Policy

### Li, Mitchell — "Datalog with Constraints: A Foundation for Trust Management Languages" (2003)

- **Citation:** N. Li, J. Mitchell. Proc. PADL 2003, LNCS 2562, pp. 58–73,
  Springer, 2003.
- **URL:** <https://link.springer.com/chapter/10.1007/3-540-36388-2_6>
- **Full text:**
  <https://crypto.stanford.edu/~ninghui/papers/cdatalog_padl03.pdf>
- **Relevance:** THE formal foundation paper. Proves that Constraint Datalog
  (Datalog^C) provides the right balance of expressiveness and tractability for
  trust management. Defines "linearly decomposable unary constraint domains" and
  proves Datalog extended with any combination of such constraints remains
  tractable (polynomial-time evaluation). Shows that structured resource
  permissions (file hierarchies, value ranges) fall into this class. Directly
  compares against KeyNote: RT₁_C is more expressive AND more tractable. This is
  the theoretical justification for using Datalog as the policy language for
  trust graph evaluation.

### Li, Mitchell, Winsborough — "Design of a Role-Based Trust Management Framework" (2002)

- **Citation:** N. Li, J. Mitchell, W. Winsborough. Proc. IEEE Symposium on
  Security and Privacy, pp. 114–130, May 2002.
- **URL:** <https://ieeexplore.ieee.org/document/1004366>
- **Relevance:** Introduces the RT framework — a family of role-based trust
  management languages with increasing expressiveness, all with Datalog
  semantics. RT₀ through RT₁*C. The key construct for the trust graph model is
  \_linked roles*: `A.r ← A.r₁.r₂` — "A's role r includes anyone who is in role
  r₂ as defined by anyone in A's role r₁." This captures transitive trust
  through intermediaries. Credential semantics defined by translation to Datalog
  rules: `SP(A.r ← B.r₁) = m(A, r, ?X) :- m(B, r₁, ?X)`. Credential graph as
  searchable representation, with reachability sound and complete w.r.t.
  set-theoretic semantics. The composability of Datalog fixpoint evaluation is
  what makes independent policy fragments composable.

### Li, Winsborough, Mitchell — "Distributed Credential Chain Discovery in Trust Management" (2003)

- **Citation:** N. Li, W. Winsborough, J. Mitchell. Journal of Computer Security
  11(1), pp. 35–86, 2003.
- **Relevance:** Addresses the practical problem of _finding_ credential chains
  in a distributed system where credentials are stored at different locations.
  Introduces a type system for credential storage that guarantees well-typed
  credential chains are discoverable. Evaluation based on reachability in
  credential graphs is proven sound and complete. Directly applicable to
  distributed trust graph evaluation where attestation evidence comes from
  multiple independent sources.

### Li, Mitchell, Winsborough — "Beyond Proof-of-Compliance: Safety and Availability Analysis in Trust Management" (2003)

- **Citation:** N. Li, W. Winsborough, J. Mitchell. Proc. IEEE Symposium on
  Security and Privacy, pp. 123–139, May 2003. Extended version in J. ACM 52(3),
  pp. 474–514, 2005.
- **Relevance:** Goes beyond "does this credential chain authorize this
  request?" to "can this policy ever lead to unauthorized access?" (safety) and
  "can this policy ever prevent authorized access?" (availability). These
  meta-analyses are possible _because_ the policy language has Datalog semantics
  with known complexity bounds. Relevant for analyzing attestation policies: can
  a malicious attester construct evidence that passes policy checks it
  shouldn't?

### Li, Grosof, Feigenbaum — "Delegation Logic: A Logic-Based Approach to Distributed Authorization" (2003)

- **Citation:** N. Li, B. Grosof, J. Feigenbaum. ACM Trans. Information and
  System Security (TISSEC) 6(1), pp. 128–171, February 2003.
- **Relevance:** D1LP extends Datalog with delegation constructs including
  delegation depth and complex principals (k-out-of-n thresholds). Shows that
  rich delegation semantics can be expressed while maintaining tractability
  under commonly-met restrictions. The threshold construct is relevant for
  composite attestation: requiring agreement from multiple independent
  attestation chains.

### Jim — "SD3: A Trust Management System with Certified Evaluation" (2001)

- **Citation:** T. Jim. Proc. IEEE Symposium on Security and Privacy, pp.
  106–115, May 2001.
- **Relevance:** SD3 (Secure Dynamically Distributed Datalog) uses Datalog
  directly as its policy language, with credentials stored as signed Datalog
  clauses distributed across the network. "Certified evaluation" means the
  system can produce a proof trace that a third party can verify. This
  proof-carrying property is valuable for attestation: the verifier can produce
  a proof that the evidence satisfies the policy, which a relying party can
  check without re-evaluating.

### deTreville — "Binder: A Logic-Based Security Language" (2002)

- **Citation:** J. deTreville. Proc. IEEE Symposium on Security and Privacy, pp.
  105–113, May 2002.
- **Relevance:** Binder extends Datalog with a "says" operator for attributed
  statements: `A says fact(X)` means principal A asserted fact(X). The says
  construct maps directly to signed attestation claims. Binder's context
  mechanism allows different principals' statements to be kept separate while
  still being combined by rules — exactly the scoped composition needed for
  trust graph evaluation.

---

## V. Modern Systems: Graph-Based Authorization and Cryptographic Enforcement

### Andersen, Kumar, AbdelBaky, Fierro, Kolb, Kim, Culler, Popa — "WAVE: A Decentralized Authorization Framework with Transitive Delegation" (2019)

- **Citation:** M. Andersen, S. Kumar, M. AbdelBaky, G. Fierro, J. Kolb, H.-S.
  Kim, D. Culler, R. Popa. Proc. 28th USENIX Security Symposium, pp. 1375–1392,
  August 2019.
- **URL:**
  <https://www.usenix.org/conference/usenixsecurity19/presentation/andersen>
- **Relevance:** Most architecturally similar modern system. Graph-based
  authorization where principals are vertices and attestations (delegations) are
  edges. Proofs are compact subgraphs, not just linear chains. Key innovations:
  (1) RTree policy on hierarchical URI-structured resources. (2)
  Cryptographically enforced via Unequivocable Log Derived Map (three Merkle
  trees). (3) Reverse Discoverable Encryption — attestations discoverable by
  authorized parties without revealing to others. (4) No ordering constraints on
  delegation creation. Does NOT use Datalog — uses specialized graph traversal
  with permission intersection. Tradeoff: loses Datalog's composability and
  tractability guarantees; gains cryptographic privacy properties.

### Couprie, Delafargue — Eclipse Biscuit (2021–present)

- **Specification:** <https://www.biscuitsec.org/>
- **Rust implementation:** `biscuit-auth` crate —
  <https://crates.io/crates/biscuit-auth>
- **Tutorial:**
  <https://www.clever.cloud/blog/engineering/2021/04/15/biscuit-tutorial/>
- **Relevance:** Most practically relevant existing implementation. Bearer
  authorization token using modified Datalog for policy. Key properties for the
  trust graph model: (1) _Offline attenuation_ — tokens restricted (never
  expanded) without contacting issuer, via chained cryptographic blocks. (2)
  _Scoped trust via `trusting` annotations_ — third-party blocks signed by
  specific keys inject facts visible only to rules explicitly trusting that key.
  (3) Facts from token (identity), request (context), or application (ACLs);
  rules compose via fixpoint computation. (4) Authority block grants base
  rights, additional blocks can only restrict. Biscuit's scoped Datalog is the
  closest existing implementation to composable policy fragments over a trust
  graph. Available in Rust, Haskell, Go, Java, JS, WASM, C, Python, C#.

---

## VI. Negative Results: What RATS Got Wrong (and Why)

### IETF RFC 9334 — "Remote ATtestation procedureS (RATS) Architecture" (2023)

- **Citation:** H. Birkholz, D. Thaler, M. Richardson, N. Smith, W. Pan. IETF
  RFC 9334, January 2023.
- **URL:** <https://www.rfc-editor.org/rfc/rfc9334>
- **Problem 1:** Pipeline model (Attester → Verifier → Relying Party) assumes
  evidence flows one direction. Composite attestation (TPM + SEV-SNP) requires
  correlation of multiple independent chains — a graph, not a pipeline.
- **Problem 2:** Appraisal policy formats/protocols explicitly declared out of
  scope. This is equivalent to designing a database and declaring the query
  language out of scope.
- **Problem 3:** Evidence treated as opaque blobs that only the Verifier
  interprets. But evidence has internal graph structure (cert chains, nested
  signatures, cross-references) that the policy language must reason about.

### IETF CoRIM (Concise Reference Integrity Manifest) — drafts

- **Citation:** Various IETF drafts on CoRIM/CoMID.
- **Problem:** Confuses reference values with policy. A reference measurement is
  a policy fragment ("this measurement is acceptable"), but CoRIM encodes it as
  static data separate from evaluation logic. Cannot express relationships
  between claims from different attestation chains. Explicitly states it is "not
  meant to encode complex policy decisions" — which is the entire problem.

### Intel Trust Authority Policy V2

- **Problem:** Supports composite attestation (TDX + TPM, SEV-SNP + vTPM, TDX +
  GPU) but as flat JSON with sections per TEE. No graph structure, no
  composability, no formal semantics. Each deployment must hand-craft policy for
  each TEE combination.

---

## VII. Supplementary: Attestation Protocol and Hardware Trust

### Schneider — "Enforceable Security Policies" (2000)

- **Citation:** F. Schneider. ACM Trans. Information and System Security 3(1),
  pp. 30–50, February 2000.
- **Relevance:** Characterizes which security policies can be enforced by
  reference monitors. Shows that enforceable policies are exactly the safety
  properties (bad things never happen). Attestation evaluation is a safety
  property: "untrusted evidence never produces an accept decision." This means
  attestation policy is enforceable by a reference monitor — the trust graph
  evaluator.

### CCxTrust — "Scalable and Flexible Attestation" (2024)

- **Citation:** arxiv 2412.03842
- **Relevance:** Combines CPU-TEE (SEV-SNP/TDX) black-box Root of Trust with TPM
  flexible white-box Root of Trust. Independent Roots of Trust for Measurement,
  collaborative Root of Trust for Report. Addresses composite attestation
  _protocol_ but not composable _policy_ for evaluation. Defines the problem
  space for cross-chain correlation without solving the policy language problem.

---

## Summary: Genealogy of Ideas

```
Lampson/Abadi/Burrows (1990-93)
  "speaks for", principals as keys, scoped trust
        │
        ├──► SPKI/SDSI (Ellison, Rivest, 1996-99)
        │      Authorization certificates, key-centric identity
        │      ⚠ Li & Mitchell: proof procedure incomplete
        │
        ├──► Blaze/Feigenbaum (1996-98)
        │      Trust management as distinct problem
        │      PolicyMaker/KeyNote
        │      ⚠ Compliance checking undecidable in general
        │
        └──► Abadi calculus (1993)
               Formal algebra of compound principals, role restriction
                      │
                      ▼
              Li/Mitchell/Winsborough (2002-03)
              RT framework: Datalog semantics for trust management
              Constraint Datalog: tractable + expressive
                      │
                      ├──► SD3, Binder, Delegation Logic
                      │      Datalog-based trust management systems
                      │
                      ├──► Biscuit (2021-present)
                      │      Scoped Datalog in cryptographic tokens
                      │      Offline attenuation, composable policy
                      │
                      └──► WAVE (2019)
                             Graph-based auth, cryptographic enforcement
                             (but no Datalog — missed composability)

                  RATS (2023) ◄── Gets the problem domain right,
                                   punts on the hard part (policy)
```

The gap: no existing system combines Datalog-based composable policy
(Li/Mitchell), graph-structured evidence with cross-chain correlation (WAVE's
insight), and hardware attestation appraisal (RATS's domain) into a unified
trust graph evaluation framework with formal tractability guarantees.
