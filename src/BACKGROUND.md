# Data Model and Concepts

This document defines the observation graph—the data model that notarized
evaluates against policies to make trust decisions.

## Processes and Observers

A **process** has inputs and outputs.

```mermaid
flowchart LR
    inputs --> Process --> outputs
```

This abstraction covers computational processes (functions, services,
pipelines), business processes (approvals, audits, manufacturing), and physical
processes (assembly, shipping, verification).

When we need more trust in a process, we wrap it in an **observer**. The
observer produces **observations** about the process:

```mermaid
flowchart LR
    subgraph Observer
        Process
    end

    inputs --> Process --> outputs
    Observer --> observations
```

The observer doesn't change the process. It produces a parallel output—
observations—that downstream consumers use for trust decisions.

### Examples

This pattern already pervades our systems:

- **Financial Audit.** An auditor observes financial processes. The audit report
  is the observation—assertions about controls, accuracy, compliance.

- **Manufacturing Certificate of Conformance.** A manufacturer observes
  production. The certificate records materials used, tests performed,
  specifications met.

- **Code Signing.** A publisher observes their build pipeline. The signature
  observes that specific source, environment, and toolchain produced a specific
  artifact.

- **X.509 Certificates.** This example might be somewhat surprising. We often
  think of certificates as outputs of a process or credentials use for a
  process, not observations. But a certificate is the _observation_ of a
  business process—private key management. The CA observes that controls protect
  the key's scope and records those observations (subject, validity, key usage,
  policy OIDs). The process is key management. The certificate is the
  observation of the process.

### Observation Principles

For an observation to be trustworthy, the observer must meet certain criteria.
These are ideals. Real systems approximate them to varying degrees.

- **Separation.** The observer is distinct from the process—computationally,
  physically, or organizationally. _An auditor is a different organization than
  the company being audited._

- **Isolation.** The process cannot modify, disable, or influence the observer.
  Information flows one direction: process → observer. _A flight data recorder
  cannot be disabled by the pilot during flight._

- **Disinterest.** The observer has no stake in the process outcome. It cares
  only about accuracy of observation. _A notary public doesn't care what's in
  the document._

- **Scope.** Observations cannot extend beyond the observer's actual access. A
  witness testifies to what they saw—not what they heard from others.

- **Integrity.** Observations cannot be altered after production without
  detection. _Tamper-evident seals on shipping containers._

The gap between ideal and actual affects how much weight to place on an
observation.

### Observations and Properties

An observation is a structured record—a collection of **properties**. Each
property is a named value.

Consider an X.509 certificate. It has properties like `subject`, `issuer`,
`validity`, `public_key`, `extensions`:

```
certificate {
    subject: "example.com"
    issuer: "DigiCert"
    validity: { not_before: ..., not_after: ... }
    public_key: 0x...
    extensions: { ... }
}
```

An attestation report has different properties: `measurement`, `policy`,
`report_data`. A financial audit has findings, material weaknesses, opinion
type. The structure varies by domain, but the pattern holds: observations are
property collections.

Properties are the atomic unit. Trust decisions operate on properties not nodes.
In fact, nodes are just collections of properties. The observation graph is a
grouped (node) property graph.

### Constraints

A **constraint** is an assertion about properties. We call a constraint within a
single observation a **pin**. A self-signed certificate pins
`subject == issuer`. A version check pins `version >= 2.0`.

Constraints are how policies express requirements. For now, think of pins as the
basic case. Later we'll see constraints that relate observations to each other:
bonds (within a chain) and bridges (across domains).

## Observation Chains

### Delegation

An observer can delegate authority to another observer.

```obgraph
node Parent "Parent Observer" @anchored {
}
node Child "Child Observer" {
}
node Process {
}
Child <- Parent : observes
Process <- Child : observes
```

Consider a root CA. It observes the key management practices of a subordinate
CA—physical security, personnel controls, policy compliance. If satisfied, it
issues a certificate: an observation that says "this entity can act as a CA."
The subordinate is now an observer too, authorized to make its own observations.

This is a parent/child relationship. The parent observes a process (key
management, operational controls) and produces an observation that endorses the
child as an observer. The child may have existed before. It may have even
produced observations before. But trust in those observations derives from the
parent's oversight of the child.

**More examples of delegation:**

- A firmware signing key endorses a bootloader signing key.
- A TPM manufacturer CA endorses an Endorsement Key.
- An audit firm's root endorses individual auditors.
- A hardware security module endorses derived keys.

### Chains

With delegation, observers form chains.

```obgraph
node Root @anchored {
}
node IntA "Intermediate" {
}
node IntB "Intermediate" {
}
node Leaf {
}
IntA <- Root
IntB <- IntA
Leaf <- IntB
```

A **non-terminal observation** endorses the next observer. A **terminal
observation** ends the chain—it observes something other than another observer.

The first observer in any chain is special: it has no parent. It can only be
trusted via an explicit grant of trust. This is the leap of faith that grounds
the entire chain.

Integrity can be established in many ways: digital signatures (X.509, code
signing), credential binding (TPM MakeCredential), extend-only registers with
replay validation (PCRs + TCG event log), or physical tamper evidence. The
mechanism varies; the guarantee is the same: a parent's property (its key, its
register state) guarantees the child's entire integrity.

We call this a **link**. A link binds a parent's property to a child's complete
integrity. Links form chains.

**Example: AMD SEV-SNP attestation chain.**

```obgraph
node ARK @anchored {
}
node ASK {
}
node VCEK {
}
node Report "Attestation Report" {
}
ASK <- ARK : sign
VCEK <- ASK : sign
Report <- VCEK : sign
```

| Observer | Type         | Observes             | Produces           |
| -------- | ------------ | -------------------- | ------------------ |
| ARK      | Non-terminal | ASK's key management | ASK certificate    |
| ASK      | Non-terminal | CPU manufacturing    | VCEK certificate   |
| VCEK     | Terminal     | VM launch state      | Attestation report |

ARK is explicitly trusted (the leap of faith). Trust in ASK derives from ARK's
endorsement. Trust in VCEK derives from ASK's endorsement. Trust in the
attestation report derives from VCEK.

### Bonds

Links bind parent properties to child integrity. But within a chain, we also
need to compare properties across observations—does the child's `issuer` match
the parent's `subject`?

We call these **bonds**. A bond compares properties across observations within
the same chain.

**Example: X.509 chain validation.**

```obgraph
node RootCA "Root CA" @anchored {
    subject @constrained
    public_key @constrained
}
node IntCA "Intermediate CA" {
    subject @constrained
    issuer @critical
    public_key @constrained
}
node Leaf {
    subject
    issuer @critical
}

# Links (integrity guarantees)
IntCA <- RootCA : sign
Leaf <- IntCA : sign

# Bonds (property comparison within chain)
IntCA::issuer <= RootCA::subject
Leaf::issuer <= IntCA::subject
```

The arrows labeled "link" show integrity guarantees (the parent's key signs the
child). The arrows labeled "bond" show property comparisons (issuer must match
subject). Break any link or bond, validation fails. Unconstrained critical
properties show a red indicator—a gap in the trust chain. compare properties (issuer must match subject). The
arrows labeled "link" show integrity guarantees (the parent's key signs the
child). Break any link or bond, validation fails.

Some properties are **critical**—they constrain what the child can legitimately
claim. ASK can only endorse VCEKs for chips it observed during manufacturing.
VCEK can only attest to VMs that match the launch policy. Other properties are
**non-critical**—informational, useful for logging or debugging, but not
enforced. Critical properties define the trust boundary; non-critical properties
describe it.

## Federation

### Observation Domains

A chain rooted at an explicitly trusted observer forms an **observation
domain**—a coherent region of trust with unified authority.

AMD's SEV-SNP chain is one domain. A WebPKI CA hierarchy is another. A TPM
manufacturer's certificate chain is a third. Each has its own root, its own
policies, its own scope.

Within a domain, chains establish hierarchy. Constraints validate linkage. Every
observation traces back to the root.

### Independent Authorities

Different authorities run different observation domains. No single root governs
them all.

- **Hardware manufacturers** run domains for device identity. AMD has ARK/ASK.
  Intel has their attestation CA. TPM manufacturers have their own CAs.

- **Software publishers** run domains for release integrity. Microsoft signs
  Windows. Mozilla signs Firefox. Each has their own signing roots.

- **Vulnerability databases** run domains for security findings. MITRE publishes
  CVEs. NIST maintains the NVD. These are observations too.

- **Enterprises** run domains for policy. Approved software versions, acceptable
  configurations, compliance requirements.

- **Auditors** run domains for assurance. SOC 2 reports, financial audits,
  security assessments.

```obgraph
domain "AMD SEV-SNP" {
    node ARK @anchored {
    }
    node ASK {
    }
    node VCEK {
    }
}
domain "TPM Manufacturer" {
    node MfgCA "Mfg CA" @anchored {
    }
    node EK {
    }
}
domain "Software Publisher" {
    node PubRoot "Pub Root" @anchored {
    }
    node Release {
    }
}
domain "Enterprise Policy" {
    node Policy "Policy Root" @anchored {
    }
    node Approved "Approved Versions" {
    }
}
ASK <- ARK
VCEK <- ASK
EK <- MfgCA
Release <- PubRoot
Approved <- Policy
```

Same pattern in each: a root, a chain (or tree), observations with properties.
But no hierarchy connects them. AMD doesn't sign Microsoft's releases. Microsoft
doesn't sign AMD's chips. They're independent authorities over independent
processes.

This collection of independent domains is a **federation**.

## Cross-Domain Constraints

Domains are independent, but trust decisions often span them.

"Run this workload" requires:

- Hardware attestation (AMD SEV-SNP domain)
- Software integrity (publisher domain)
- No known vulnerabilities (CVE domain)
- Approved by policy (enterprise domain)

These come from different roots. No chain connects them. But their observations
share properties that relate:

| Domain 1             | Property      | Domain 2              | Property            |
| -------------------- | ------------- | --------------------- | ------------------- |
| Runtime attestation  | `measurement` | Software publisher    | `release.hash`      |
| Hardware attestation | `chip.id`     | Manufacturer database | `device.serial`     |
| Software release     | `version`     | CVE database          | `affected_versions` |
| Software release     | `version`     | Enterprise policy     | `approved_versions` |

When `attestation.measurement == release.hash`, the runtime is running exactly
what the publisher released. When `release.version not in cve.affected`, the
software has no known vulnerabilities.

These are cross-domain comparisons. They're like bonds, but across observation
domains instead of within a single chain.

### Bridges

We call these **bridges**. A bridge compares properties across observation
domains.

Links guarantee integrity (property-to-node). Bonds and bridges compare
properties (property-to-property). Bonds work within a chain. Bridges work
across domains.

```obgraph
domain "Runtime Attestation" {
    node HW "HW Root" @anchored {
    }
    node FW {
    }
    node App {
        measurement @critical
    }
}
domain "Software Publisher" {
    node Pub "Pub Root" @anchored {
    }
    node Release {
        hash @constrained
    }
}
FW <- HW
App <- FW
Release <- Pub

# Bridge (cross-domain constraint)
App::measurement <= Release::hash
```

The bridge connects runtime attestation to the software publisher chain. When
the properties match, trust flows: the runtime is running exactly what the
publisher released.

Bridging doesn't require exact equality. It requires **deterministic
derivation**—some known transformation from A to B such that the relationship
can be verified. Hash equality is the simplest case. But a bridge could also
verify that a version falls within a range, that a certificate was issued before
a revocation date, or that a measurement matches any of several approved values.

**Bridge examples:**

| Domain 1             | Domain 2              | Bridge Property                       |
| -------------------- | --------------------- | ------------------------------------- |
| Runtime attestation  | Software publisher    | measurement == release.hash           |
| Runtime attestation  | CVE database          | component.version not in cve.affected |
| Hardware attestation | Manufacturer database | chip.id == device.serial              |
| Software release     | Enterprise policy     | release.version in policy.approved    |

**Example: SEV-SNP and TPM bridged via TCG event log.**

Two independent attestation chains—AMD SEV-SNP and TPM—each with their own root.
The TCG event log bridges them: the TPM quote covers a PCR digest, the event log
replays to validate that digest, and an event entry contains the SEV-SNP chip
ID. Solid arrows show integrity guarantees (signatures, credential binding,
extend-only registers). Dashed arrows show bridges (property matching).

```obgraph
domain "SEV-SNP" {
    node ARK @anchored {
        subject @constrained
        issuer @critical
        public_key @constrained
    }
    node ASK {
        subject @constrained
        issuer @critical
        public_key @constrained
    }
    node VCEK {
        subject @constrained
        issuer @critical
        public_key @constrained
        chip_id @constrained
    }
    node Report "Attestation Report" @selected {
        chip_id @critical
    }
}
domain "TPM" {
    node MfgCA "TPM Mfg CA" @anchored {
        subject @constrained
        issuer @critical
        public_key @constrained
    }
    node EK {
        subject @constrained
        issuer @critical
        public_key @constrained
    }
    node AK {
        public_key @critical
    }
    node Quote "TPM Quote" {
        pcr_digest @constrained
        signature @critical
    }
}
node TCGLog "TCG Event Log" {
    event_entries @critical
}

# Anchors (integrity guarantees)
ASK <- ARK : sign
VCEK <- ASK : sign
Report <- VCEK : sign
EK <- MfgCA : sign
AK <- EK : make_credential
Quote <- AK : sign
TCGLog <- Quote : replay_validate

# Self-signed pins
ARK::issuer <= ARK::subject : self_signed
MfgCA::issuer <= MfgCA::subject : self_signed

# Chain bonds
ASK::issuer <= ARK::subject
VCEK::issuer <= ASK::subject
EK::issuer <= MfgCA::subject
Report::chip_id <= VCEK::chip_id

# Integrity constraints
AK::public_key <= EK::public_key : make_credential
Quote::signature <= AK::public_key : verified_by

# Cross-domain bridges
TCGLog::event_entries <= Quote::pcr_digest : replay_validates
Report::chip_id <= TCGLog::event_entries : contains
```

Just as delegation composes into chains, bridging composes into a federation.
Each bridge creates a path for trust to flow between domains. A trust
decision—"should I run this workload?"—can draw on observations from across the
entire federation.

## Observation Trees

Most attestations today are linear:

```obgraph
node Firmware @anchored {
}
node Bootloader {
}
node Kernel {
}
node Application {
}
Bootloader <- Firmware
Kernel <- Bootloader
Application <- Kernel
```

Real systems aren't linear. Components stay resident while later stages boot.
Live updates arrive without reboots. Operating systems run many processes in
parallel, each with its own isolation boundary—containers, VMs, Wasm modules, V8
isolates.

The observation graph of a real system is a tree:

```obgraph
node HW "Hardware Root" @anchored {
}
node Firmware {
}
node BMC {
}
node PSP "PSP/Caliptra" {
}
node OS "OS Kernel" {
}
node AppA "App A" {
}
node AppB "App B" {
}
node AppC "App C" {
}
node Wasm1 "Wasm 1" {
}
node Wasm2 "Wasm 2" {
}
node Wasm3 "Wasm 3" {
}
Firmware <- HW
BMC <- HW
PSP <- HW
OS <- Firmware
OS <- BMC
OS <- PSP
AppA <- OS
AppB <- OS
AppC <- OS
Wasm1 <- AppC
Wasm2 <- AppC
Wasm3 <- AppC
```

Two operations on an observation tree:

- **Inventory:** "What is the complete state of this system?" Returns the entire
  tree—every observer, every observation.

- **Identity:** "What trust path defines this specific workload?" Returns
  `root → intermediate → ... → terminal`, excluding sibling branches.

Identity allows selective disclosure. A workload can prove its trust path
without revealing what else runs on the same machine.
