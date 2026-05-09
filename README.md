# GAP Research Paper 002: Governed Agentic Programming

## GAP -- A Structural Instruction Language for Construction-Level Governance of Agentic AI Systems

**Author:** the.PM (thePM001)  
**Affiliation:** New Lisbon Agency / Iberian Peninsula Human Civilization Continuation Project (IPHCCP)  
**Date:** May 2026  
**Correspondence:** support@newlisbon.agency  

---

### About This Paper

This paper introduces **GAP (Governed Agentic Programming)**, a structural instruction language whose documents are generated under token-level constrained decoding from a JSON Schema 2020-12 Meta-Schema compiled into a context-free grammar.

### Key Contributions

1. **GAP Meta-Schema and structural instruction language** -- canonical JSON data model for governed agent instructions
2. **Constrained decoding and No-Bypass Theorem** -- three sufficient engineering invariants (prefix soundness, termination soundness, viable non-empty continuation) guaranteeing zero structural bypass probability
3. **Structural-semantic separation** -- identifies the intrinsic boundary of constraint masking
4. **Governance Runtime Interface** -- minimal abstract contract for deterministic downstream adjudication
5. **Governance-preserving composition** -- operational semantics for six composition operators
6. **Self-generation containment** -- child instructions constrained by the same Meta-Schema
7. **Deployment-tier analysis** -- three deployment tiers covering all inference configurations

### Paper File

The full paper is available in this repository:

📄 **[GAP-research-paper.md](./GAP-research-paper.md)** (142KB, 1274 lines)

### Abstract

Generative language models produce structured outputs with no formal guarantee of structural conformance. Existing approaches to output governance rely on prompt-level instructions, post-hoc classifiers or reinforcement learning, all of which are probabilistic and furnish no convergence proof. Token-level constraint masking (as implemented in llama.cpp, Outlines and similar systems) provides structural guarantees but lacks a unified governance framework. We introduce GAP (Governed Agentic), a structural instruction language whose documents are generated under token-level constrained decoding from a JSON Schema 2020-12 Meta-Schema compiled into a context-free grammar. We prove the No-Bypass Theorem, which isolates three sufficient engineering invariants (prefix soundness, termination soundness and viable non-empty continuation) whose joint satisfaction yields zero structural bypass probability. We prove a separate necessity result (Proposition 3.2): prefix soundness and termination soundness are necessary for zero invalid terminated output over all models assigning positive probability to permitted tokens, and viable non-empty continuation is necessary for deadlock freedom.

GAP separates structural guarantees from semantic governance. The language guarantees that every instruction contains the required address, pattern, action, composition and governance fields. Semantic and policy correctness are delegated to a deterministic governance runtime satisfying a minimal abstract interface: total evaluation, deterministic verdicts, hard-predicate soundness, monotonic child-governance constraints and evidence production. This abstraction permits GAP artifacts to be evaluated by different adjudication implementations without coupling the language to a specific runtime architecture.

### Keywords

constrained decoding, formal grammars, token masking, agent governance, instruction semantics, adjudication lattices, self-governing systems, JSON Schema

### License

See [LICENSE](./LICENSE) file.