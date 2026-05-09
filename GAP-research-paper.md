## GAP Research Paper 001: GAP - Governed Agentic Programming - A Structural Instruction Language for Construction-Level Governance of Agentic AI Systems

Research paper for the GAP: Governed Agentic Programming language by the New Lisbon Agency/Iberian Peninsula Human Civilization Continuation Project (IPHCCP).

## Files

- GAP-research-paper.md - Full paper (this file)
- GAP-research-paper.pdf - PDF rendering
- GAP-research-paper.pdf.ots - OpenTimestamps proof of prior existence (to be generated)

## Verify timestamp

Install OpenTimestamps (https://opentimestamps.org/) and run:

    ots verify GAP-research-paper.pdf.ots


### Author: the.PM (https://x.com/thePM_001)
### Iberian Peninsula Human Civilization Continuation Project
### New Lisbon Agency
### Correspondence: support@newlisbon.agency
### May 2026

Authorship commitment: This document was timestamped via OpenTimestamps. The matching .ots proof is available upon authorship dispute claim.

## Abstract

Generative language models produce structured outputs with no formal guarantee of structural conformance. Existing approaches to output governance rely on prompt-level instructions, post-hoc classifiers or reinforcement learning, all of which are probabilistic and furnish no convergence proof. Token-level constraint masking (as implemented in llama.cpp, Outlines and similar systems) provides structural guarantees but lacks a unified governance framework. We introduce GAP (Governed Agentic), a structural instruction language whose documents are generated under token-level constrained decoding from a JSON Schema 2020-12 Meta-Schema compiled into a context-free grammar. We prove the No-Bypass Theorem, which isolates three sufficient engineering invariants (prefix soundness, termination soundness and viable non-empty continuation) whose joint satisfaction yields zero structural bypass probability. We prove a separate necessity result (Proposition 3.2): prefix soundness and termination soundness are necessary for zero invalid terminated output over all models assigning positive probability to permitted tokens, and viable non-empty continuation is necessary for deadlock freedom.

GAP separates structural guarantees from semantic governance. The language guarantees that every instruction contains the required address, pattern, action, composition and governance fields. Semantic and policy correctness are delegated to a deterministic governance runtime satisfying a minimal abstract interface: total evaluation, deterministic verdicts, hard-predicate soundness, monotonic child-governance constraints and evidence production. This abstraction permits GAP artifacts to be evaluated by different adjudication implementations without coupling the language to a specific runtime architecture.

We formalize GAP's instruction-as-atomic-unit model, define operational semantics for six composition operators and prove that compositions cannot bypass runtime governance. Self-generating instructions are constrained by the same Meta-Schema and must satisfy monotonic governance constraints before activation. We describe three v1.1 extensions (likelihood ratio testing for self-generation, MLE weight estimation and latent governance inference) that add statistical rigour without altering structural invariants. We show that GAP's 15-step evaluation chain satisfies the structural requirements of the Deterministic Adjudication Lattice framework [1], distinguishing structural predicates (sound by construction) from semantic predicates (sound relative to pattern libraries). Three deployment tiers (logits-level, post-processing validation gate and provider JSON Schema mode) provide coverage across all inference configurations. We identify the structural-semantic gap as the intrinsic boundary of constraint masking: constrained decoding eliminates malformed instructions but cannot by itself prove open-ended semantic safety. GAP therefore provides a language-level foundation for governed agency: structural validity is enforced during generation, while semantic acceptance remains an explicit, auditable property of deterministic downstream adjudication.

Keywords: constrained decoding, formal grammars, token masking, agent governance, instruction semantics, adjudication lattices, self-governing systems, JSON Schema

## 1  Introduction

## 1.1  The Unconstrained Output Problem

Large language models generate text by sampling from a learned probability distribution over token sequences. At each decoding step, the model assigns a probability to every token in its vocabulary and selects one. The resulting output is statistically plausible but structurally unconstrained: the model may produce well-formed JSON, malformed YAML, executable code, natural language or any combination thereof. When the output must conform to a specific structural specification, the gap between statistical plausibility and structural conformance constitutes a fundamental verification problem.

For autonomous agent systems that generate structured instructions, this gap has direct operational consequences. An agent that produces a syntactically invalid instruction halts its own execution pipeline. An agent that produces a structurally valid but governance-violating instruction may escalate privileges, bypass safety constraints or exceed resource budgets. The unconstrained output problem is therefore both a correctness problem and a safety problem.

## 1.2  Why Prompting and Guardrails Are Insufficient

The prevailing approach to constraining language model outputs is prompt engineering: system messages that instruct the model to produce output in a specific format. This approach has three fundamental deficiencies.

First, prompt instructions are advisory. The model assigns nonzero probability to every token in its vocabulary at every decoding step, including tokens that violate the prompt's instructions. No prompt can reduce this probability to exactly zero.

Second, guardrail classifiers [2, 3] operate post-hoc. They evaluate completed outputs and reject those that fail classification criteria. Post-hoc filtering has a nonzero false-negative rate ε: with probability ε, a defective output passes the filter. Cascading k independent filters reduces the residual defect probability to ε^k but never reaches zero. The independence assumption is unrealistic for filters checking related properties [1].

Third, reinforcement learning from human feedback (RLHF) [4, 5] shifts the output distribution toward preferred outputs but cannot eliminate the probability mass on non-preferred outputs. The resulting distribution still assigns nonzero probability to structurally invalid outputs.

All three approaches treat output governance as a statistical property of the model's distribution. GAP treats it as a structural property of the decoding process.

## 1.3  The Constraint Masking Intuition

The core mechanism of GAP is constraint masking at the token level. Before the model selects each token, a mask derived from the GAP Meta-Schema sets the probability of every structurally invalid token to zero. The model selects only from tokens that maintain conformance to the schema. The output is structurally valid by construction, not by statistical tendency.

This mechanism requires no model training, no fine-tuning and no in-context examples. An LLM that has never encountered a GAP instruction during training will produce perfectly valid GAP instructions because the invalid tokens are absent from the selection space. The constraint resides in the decoding engine, external to the model.

## 1.4  Contribution Summary

Token-level constraint masking is mature engineering practice. Llama.cpp GBNF grammars [6], Outlines [11], PICARD [23] and grammar-aligned decoding [24] all implement variants of the mechanism described in Section 3. The constraint masking framework is not the novelty; the novelty is what GAP builds on top of it.

This paper makes seven formal contributions:

1. **GAP Meta-Schema and structural instruction language.** We define a canonical JSON data model for governed agent instructions, including address, pattern, action, composition, type and governance fields, and establish that the Meta-Schema language is context-free under a precisely stated restricted JSON Schema fragment (Proposition 2.1).

2. **Constrained decoding and No-Bypass Theorem.** We formalise schema-constrained token generation and prove the No-Bypass Theorem (Theorem 3.2), which isolates three sufficient engineering invariants (prefix soundness, termination soundness and viable non-empty continuation) whose joint satisfaction yields zero structural bypass probability. A separate necessity result (Proposition 3.2) establishes that prefix soundness and termination soundness are necessary for zero invalid terminated output and that viable non-empty continuation is necessary for deadlock freedom.

3. **Structural-semantic separation.** We identify the structural-semantic gap as the intrinsic boundary of constraint masking and show which guarantees belong to the language layer versus the runtime layer. We provide explicit bounds on residual semantic defect probability as a function of pattern library completeness (Section 6.6), using correct conditional and union-bound formulations.

4. **Governance Runtime Interface.** We define a minimal abstract contract for deterministic downstream adjudication: totality, determinism, hard-predicate soundness, monotonic child governance and evidence production (Section 6.7). This abstraction decouples the language from any specific runtime implementation.

5. **Governance-preserving composition.** We give operational semantics for six composition operators and prove that composition cannot bypass runtime governance (Theorem 4.1). The governance preorder (Definition 4.6) formalises the monotonicity property.

6. **Self-generation containment.** We prove that child instructions generated through GAP remain structurally valid and must satisfy monotonic governance constraints before activation (Theorem 4.2). The v1.1 extensions (LRT-gated self-generation, MLE weight estimation and latent governance inference) add statistical rigour without altering structural invariants.

7. **Deployment-tier analysis.** We analyse three deployment tiers (logits-level, post-processing gate and provider JSON Schema mode) and prove coverage properties for each, including graceful degradation and governance depth variation when logits-level enforcement is unavailable.

## 1.5  Paper Organization

Section 2 establishes mathematical preliminaries, canonical serialisation convention and notation. Section 3 develops the constraint masking framework, proves the No-Bypass Theorem, identifies the structural-semantic gap and defines two-phase type enforcement. Section 4 formalizes the instruction model (including event-triggered evaluation, pipeline semantics and the governance preorder), self-governance and v1.1 extensions (LRT, MLE weight estimation, latent governance inference). Section 5 analyses the deployment architecture including the binary compilation target. Section 6 presents the 15-step adjudication lattice (with Step 3.5 for complex structure validation), analyses semantic predicate unsoundness and defines the abstract Governance Runtime Interface with optional extensions (R6-R8) and validation depth levels. Section 7 develops formal semantics. Section 8 compares GAP to existing approaches. Section 9 discusses limitations and open problems. Section 10 concludes. Appendices A through C contain expanded proofs, the Meta-Schema grammar construction (in canonical JSON) and an informal category-theoretic perspective on instruction composition. Appendix D presents illustrative analytical scenarios.

## 2  Preliminaries and Notation

## 2.1  Token-Level Language Model Decoding

## Definition 2.1 (Autoregressive Language Model).

An autoregressive language model is a stochastic map M: V* → Δ(V), where V is a finite token vocabulary of size |V| = n and Δ(V) is the probability simplex over V. Given a prefix w = (t₁, ..., t_{i-1}) ∈ V*, the model produces a probability distribution P_M(· | w) ∈ Δ(V). Token selection samples t_i ~ P_M(· | w).

The vocabulary V is model-specific. Contemporary LLMs use subword tokenizers with |V| ranging from 32 000 to 128 000. The distribution P_M is defined by the model's parameters and is fixed after training. We treat the model as a black box: we access P_M only through the probability vector it produces at each step.

## Definition 2.2 (Unconstrained Decoding).

Unconstrained decoding generates a sequence (t₁, t₂, ..., t_m) by iteratively sampling t_i ~ P_M(· | t₁, ..., t_{i-1}) until a termination condition is met (end-of-sequence token or maximum length).

Under ideal unconstrained decoding with full softmax, finite logits and positive temperature, every token in V has nonzero probability at every step. Practical decoding strategies may already impose additional truncation or masking: top-k sampling restricts selection to the k highest-probability tokens, nucleus (top-p) sampling truncates below a cumulative probability threshold, logit biases can suppress specific tokens and finite-precision arithmetic may cause underflow for very low-probability tokens. Under any decoding strategy that preserves positive probability for all tokens, the output may be any string in V*.

## 2.2  Context-Free Grammars and GBNF

## Definition 2.3 (Context-Free Grammar).

A context-free grammar (CFG) is a tuple G = (N, Σ, P, S), where N is a finite set of non-terminal symbols, Σ is a finite set of terminal symbols (the alphabet), P is a finite set of production rules of the form A → α where A ∈ N and α ∈ (N ∪ Σ)* and S ∈ N is the start symbol. The language L(G) ⊆ Σ* is the set of terminal strings derivable from S.

GBNF (GGML BNF) is a concrete syntax for context-free grammars used by the llama.cpp inference engine [6]. A GBNF grammar defines a set of production rules over Unicode characters and can be compiled to operate at the token level by mapping each token to its character-level expansion and tracking parse states through a pushdown automaton (PDA).

## 2.3  Canonical Serialization

**Serialization convention.** The formal development in this paper treats GAP documents as canonical JSON serializations of a JSON data model. GAP source files may be authored in YAML for human readability, but YAML authoring is a surface syntax: all YAML documents are parsed into the JSON data model (per the YAML 1.2 JSON schema) before validation. The formal results (Proposition 2.1, Theorem 3.2 and all downstream theorems) apply to the canonical JSON rendering. The accepted YAML subset forbids anchors, aliases, non-JSON tags and duplicate object member names. Implementations that accept YAML input must reject documents containing these features before schema validation. Constrained-decoding grammars (Section 3.1) target canonical JSON syntax directly.

## 2.4  JSON Schema as a Constraint Language

JSON Schema 2020-12 [7] defines structural constraints on JSON documents. A JSON Schema document S specifies:

- Required and optional properties of objects
- Allowed types for each property (string, number, boolean, object, array, null)
- Enumeration constraints (allowed values for a property)
- String pattern constraints (regular expressions)
- Numeric range constraints (minimum, maximum)
- Conditional validation (if/then/else based on property values)
- Recursive definitions via `$ref`

## Proposition 2.1 (GAP Meta-Schema Defines a Context-Free Language).

The language of canonical serialized GAP documents (the set of JSON strings validating against the GAP Meta-Schema S) is context-free under the following restrictions: deterministic key ordering (or equivalently, all permutations of the finite property set are enumerated as alternative productions), duplicate object member names are forbidden before schema validation, the property sets at each nesting level are finite, `uniqueItems` is not used, cross-object `dependentRequired` is not used and all conditional validations are local (the condition and constrained clause reference the same object). This proposition does not claim that arbitrary JSON Schema documents define context-free languages; JSON Schema features such as `uniqueItems` and cross-property `dependentRequired` can impose context-sensitive constraints.

## Proof.

JSON syntax is context-free (it is defined by a BNF grammar in RFC 8259 [8]). We show that each JSON Schema feature used by the GAP Meta-Schema preserves context-freeness.

**Duplicate keys.** GAP canonical JSON forbids duplicate object member names. This restriction is enforced syntactically: because the property set at each level is finite and `additionalProperties: false` prohibits unlisted keys, the grammar enumerates all valid key permutations as finite alternatives. Duplicate keys cannot arise from these productions.

**Object member order.** JSON objects are semantically unordered, but grammars generate ordered strings. Because the GAP Meta-Schema specifies a finite set of allowed properties at each nesting level (with `additionalProperties: false`), the grammar includes all permutations of the allowed keys as alternative productions. The number of permutations is bounded by k! where k is the number of properties at that level (at most 8 for the top level). This is finite and does not affect context-freeness.

**Type and enumeration constraints** restrict values to finite alternatives, preserving context-freeness. **Pattern constraints** (regular expressions) restrict string values to regular languages; the intersection of a CFL with a regular language is a CFL [9]. **Required-property constraints** impose that certain keys must appear, expressible as mandatory non-terminals. **Local conditional constraints** (where the condition and the constrained clause reference the same object, as in the GAP Meta-Schema's `action.type`-dependent requirements) are encoded as alternative productions in the CFG. **Recursive `$ref`** references introduce recursive productions, which are native to CFGs. The recursion in the GAP Meta-Schema is structurally regular (bounded by `max_depth` in composition types) and does not impose cross-branch equality or uniqueness constraints.

The GAP fragment does not require arbitrary intersection of context-free languages (CFLs are not closed under arbitrary intersection). Each constraint is compiled structurally into the same object grammar: finite key alternatives for objects, regular subgrammars for strings, finite alternatives for enums and local conditional branches. The resulting grammar is constructed directly by finite composition of productions, preserving context-freeness.

□

## 2.5  Constraint Masking (formal definition)

## Definition 2.4 (Constraint Function).

Given a context-free grammar G = (N, Σ, P, S) with language L(G) and a finite token vocabulary V with a mapping decode: V → Σ* that maps each token to its character-level expansion, the constraint function C_G: V* → 2^V is defined by:

C_G(w) = { t ∈ V : ∃ u ∈ V* such that decode(w) · decode(t) · decode(u) ∈ L(G) }

where decode extends to sequences by concatenation. Informally, C_G(w) is the set of tokens whose selection preserves the possibility of completing the sequence to a valid document.

## Definition 2.5 (Constrained Decoding).

Constrained decoding with constraint function C generates a sequence (t₁, t₂, ..., t_m) by sampling:

t_i ~ P_C(· | t₁, ..., t_{i-1})

where P_C(t | w) = P_M(t | w) · 𝟙[t ∈ C_G(w)] / Z(w) and Z(w) = Σ_{t ∈ C_G(w)} P_M(t | w) is the normalization constant.

The constraint function zeroes out the probability of every token not in C_G(w). The model selects only from structurally valid continuations. The distribution P_C preserves the relative ordering of valid tokens: if the model prefers token t₁ over t₂ and both are in C_G(w), then P_C(t₁ | w) > P_C(t₂ | w). The constrained distribution preserves the model's relative probabilities among permitted tokens.

## Definition 2.6 (Sound Compilation).

A compilation of schema S into constraint function C_S is sound if for all prefixes w ∈ V* and tokens t ∈ V:

t ∈ C_S(w) ⟹ ∃ u ∈ V* such that decode(w · t · u) ∈ L(S)

That is, every token permitted by C_S can be extended to a valid completion.

## Definition 2.7 (Complete Compilation).

A compilation is complete if for all w ∈ V* and t ∈ V:

(∃ u ∈ V* such that decode(w · t · u) ∈ L(S)) ⟹ t ∈ C_S(w)

Completeness ensures no valid continuation is excluded. Sound and complete compilation yields C_S(w) = C_G(w) exactly.

## 2.6  The Self-Generation Property

## Definition 2.8 (Self-Generation).

A language L ⊆ V* has the self-generation property under constraint function C if for every string s ∈ L that encodes an instruction to "generate a new element of L," the constrained decoding process applied to s produces an output s' ∈ L.

GAP's self-generation property means that GAP instructions that generate new GAP instructions are themselves constrained by the GAP Meta-Schema. The language enforces itself recursively.

## 3  The Constraint Masking Framework

## 3.1  Meta-Schema Compilation to Token Masks

The GAP Meta-Schema [10] is a JSON Schema 2020-12 document that defines every legal structure of a `.gap` file. The schema specifies eight top-level fields: six required (`address`, `pattern`, `action`, `weight`, `composition`, `metadata`) and two optional (`types`, `execution`). `additionalProperties: false` prohibits all fields not in this set of eight. The grammar must enumerate all eight fields, with the two optional fields generating the empty production when absent. Each field has a defined type, allowed values and conditional validation rules.

The compilation pipeline transforms the Meta-Schema into a token mask through three stages:

**Stage 1: Schema to CFG.** The JSON Schema is parsed and each structural constraint is translated into CFG production rules. Object structure becomes non-terminal productions for key-value sequences. Enum constraints become terminal alternatives. String patterns become regular sub-grammars (which embed in CFGs). Numeric ranges become digit-sequence constraints. Conditional validation (if/then/else) becomes conditional productions guarded by previously parsed values.

**Stage 2: CFG to PDA.** The context-free grammar is compiled into a pushdown automaton that tracks the parse state during sequential token generation. The PDA maintains a stack representing the current nesting depth and expected structural elements. At each step, the PDA's transition function determines which grammar productions are applicable given the current state and stack.

**Stage 3: PDA to token mask.** At each decoding step, the PDA state determines a set of valid next characters. The token vocabulary V is scanned: a token t is included in the mask if and only if its character-level expansion decode(t) is consistent with a valid transition sequence from the current PDA state. This scan produces C_S(w) for the current prefix w.

For the GAP Meta-Schema, the resulting grammar has the following structure at the top level:

- The document must be a JSON object containing an `"address"` object with `"domain"` (matching regex `^[a-z][a-z0-9]*(\.[a-z][a-z0-9]*)+$`) and `"id"` (matching regex `^[a-z][a-z0-9-]*(\.[a-z][a-z0-9-]*)*(\.(v[0-9]+))?$`)
- `action.type` must be one of exactly five values: `"template"`, `"structured"`, `"code"`, `"reference"`, `"pipeline"`
- `weight` must be a number in [0.0, 1.0]
- `composition.type` must be one of exactly six values: `atomic`, `sequence`, `conditional`, `loop`, `parallel`, `abstraction`
- `metadata.stability` must be one of exactly four values: `experimental`, `beta`, `stable`, `deprecated`

At the token level, when the model has generated the prefix `{"action":{"type":"`, the constraint mask permits only tokens that begin one of the five allowed values. No other token exists in the selection space. The model cannot produce `prompt`, `generate`, `call_api` or any other string; those tokens have probability zero.

## 3.2  The No-Bypass Theorem (formal proof)

We now state and prove the foundational structural result on which the governance framework depends.

## Theorem 3.2 (No-Bypass Theorem).

Let S be the GAP Meta-Schema and let C_S be a compiled constraint function satisfying the following three invariants:

1. **Prefix soundness.** For every non-EOS token t: t ∈ C_S(w) implies ∃ u ∈ V* such that decode(w · t · u) ∈ L(S). That is, every permitted non-EOS token preserves extendability to a valid document.

2. **Termination soundness.** EOS ∈ C_S(w) if and only if decode(w) ∈ L(S). That is, the end-of-sequence token is enabled exactly when the accumulated prefix is a complete valid document.

3. **Viable non-empty continuation.** For every viable but incomplete prefix w (where decode(w) ∉ L(S) but w is extendable), C_S(w) contains at least one non-EOS token. This ensures that constrained decoding is not trapped in a state where it can neither continue nor terminate validly.

We treat EOS as a control token with decode(EOS) = ε (the empty string). Equivalently, the emitted document excludes EOS. Then any terminated output of constrained decoding (Definition 2.5) with constraint function C_S satisfies decode(t₁ · t₂ · ... · t_m) ∈ L(S), where t_m = EOS.

In words: every output produced by constrained decoding with the GAP Meta-Schema is a valid GAP document. The probability of producing a structurally invalid GAP document is exactly zero.

## Proof.

We prove by induction on the sequence length that after each token selection, the accumulated prefix is extendable to a valid document.

**Invariant.** After selecting tokens t₁, ..., t_i, the prefix w_i = t₁ · ... · t_i satisfies: ∃ u ∈ V* such that decode(w_i · u) ∈ L(S). We call such a prefix *viable*.

**Base case (i = 0).** The empty prefix w₀ = ε is viable because L(S) ≠ ∅ (the GAP Meta-Schema admits at least one valid document).

**Inductive step.** Suppose w_{i-1} is viable. Token t_i is selected from C_S(w_{i-1}). If t_i is not EOS, then by prefix soundness (invariant 1), t_i ∈ C_S(w_{i-1}) implies ∃ u ∈ V* such that decode(w_{i-1} · t_i · u) ∈ L(S). Therefore w_i = w_{i-1} · t_i is viable.

**Termination.** Generation terminates when the end-of-sequence token EOS is selected. By termination soundness (invariant 2), EOS ∈ C_S(w_{m-1}) implies decode(w_{m-1}) ∈ L(S). Therefore the final output decode(t₁ · ... · t_{m-1}) is a complete valid GAP document.

**Liveness.** By invariant 3, constrained decoding is never trapped: at every viable but incomplete prefix, at least one non-EOS token is available. This ensures local deadlock freedom at viable incomplete prefixes. It does not by itself guarantee almost-sure termination; a stochastic process could in principle keep selecting non-EOS tokens indefinitely if the grammar allows unbounded recursion. Termination can be enforced separately by grammar boundedness, maximum-length limits or decoding policies. The theorem concerns validity of *terminated* outputs.

Therefore, constrained decoding terminates only with outputs in L(S).

□

## Proposition 3.1 (Zero Bypass Probability).

Under constrained decoding with a sound constraint function C_S, the probability of producing an output o ∉ L(S) is exactly zero:

Pr[decode(t₁ · ... · t_m) ∉ L(S)] = 0

## Proof.

By the No-Bypass Theorem, every sequence (t₁, ..., t_m) that can be produced by constrained decoding satisfies decode(t₁ · ... · t_m) ∈ L(S). The event {decode(t₁ · ... · t_m) ∉ L(S)} is empty. An empty event has probability zero.

□

## Proposition 3.2 (Necessity for Model-Independent Zero-Bypass).

Consider the class M of language models that assign positive probability to every permitted token. (a) If a constraint function violates prefix soundness, there exists a model in M and a finite decoding path with nonzero probability that emits an output not in L(S). (b) If a constraint function violates termination soundness, there exists a model in M and a finite decoding path with nonzero probability that emits an incomplete or invalid document. (c) If viable non-empty continuation fails at some viable prefix w, there exists a model in M such that constrained decoding reaches w and deadlocks (no token, including EOS, is available), failing to produce any terminated output.

## Proof sketch.

(a) If prefix soundness fails, there exists a prefix w and non-EOS token t such that t ∈ C_S(w) but no completion of w · t is in L(S). A model that assigns high probability to t at prefix w selects t with high probability, reaching a prefix from which no valid document can be produced.

(b) If termination soundness fails in the "if" direction, EOS is permitted at a non-accepting prefix; a model that favours EOS terminates with an incomplete document. If it fails in the "only if" direction, EOS is never permitted at accepting prefixes; the model cannot terminate even at a complete document (this interacts with deadlock rather than invalidity).

(c) If C_S(w) = ∅ at a viable prefix w, decoding halts with no output. Note that this is a deadlock/liveness failure, not an invalid-output failure: the decoder produces no terminated output rather than an invalid one.

**Important distinction.** Prefix soundness and termination soundness are necessary for zero invalid *terminated* output. Viable non-empty continuation is necessary for progress/deadlock freedom, not directly for output validity. The three invariants address different failure modes (invalid output, incomplete output and no output respectively).

□

**Remark on the theorem's contribution.** The No-Bypass Theorem is a definitional consequence of constrained decoding: if you prevent the system from selecting invalid tokens, it cannot output invalid tokens. The theorem's value lies in three places. First, it makes the prefix soundness requirement explicit: the guarantee holds only if the constraint function C_S correctly identifies valid continuations; every practical failure of constrained decoding traces to a compiler bug rather than a model behaviour. Second, it formalizes the termination condition (EOS permitted only at accepting PDA states), which is non-obvious: a constraint function that allows premature termination can produce structurally incomplete documents even if every individual token was valid. Third, it establishes the liveness condition: constrained decoding must never reach a dead state where no token (including EOS) is available. The theorem isolates these three sufficient engineering invariants (prefix soundness, termination soundness and viable non-empty continuation), separating them from model properties that are irrelevant. Proposition 3.2 establishes the corresponding necessity results with an important distinction: prefix soundness and termination soundness are necessary for zero invalid *terminated* output, while viable non-empty continuation is necessary for *deadlock freedom* (a liveness property, not a safety property).

The model M may be arbitrarily weak, poorly trained or adversarial. The constraint function C_S is external to the model and is verified independently. A sound compilation is a static property of the compiler, verifiable by standard software testing.

## 3.3  Coverage and Completeness

## Theorem 3.3 (Completeness Preservation).

If the compilation of S into C_S is both sound and complete (Definitions 2.6 and 2.7), then for every valid document d ∈ L(S) with nonzero probability under the unconstrained model, there exists a nonzero-probability path through constrained decoding that produces d.

## Proof.

Let d ∈ L(S) with tokenization (t₁, ..., t_m). At each step i, token t_i must be in C_S(t₁ · ... · t_{i-1}) by completeness (since t_i can be extended to d, which is valid). The constrained probability P_C(t_i | t₁, ..., t_{i-1}) = P_M(t_i | t₁, ..., t_{i-1}) / Z > 0 since P_M assigns positive probability to t_i by hypothesis. The path probability is the product of these positive terms, which is positive.

□

This theorem ensures that constraint masking does not eliminate valid outputs from the space of producible documents. The model can produce any valid GAP instruction that it would produce under unconstrained decoding; it simply cannot produce invalid ones.

## 3.4  Orthogonality of Constraint Dimensions

The GAP Meta-Schema enforces constraints along multiple independent dimensions:

1. **Structural constraints.** Required fields, field ordering, nesting structure.
2. **Type constraints.** Value types for each field (string, number, boolean, enum).
3. **Range constraints.** Numeric bounds (weight ∈ [0.0, 1.0], retry.max_attempts ∈ [1, 10]).
4. **Pattern constraints.** Regex patterns on string values (domain, id formats).
5. **Conditional constraints.** Conditional requirements based on field values (action.type = "code" requires language and content).

## Proposition 3.3 (Orthogonality of Schema Constraints).

The five constraint dimensions listed above are pairwise orthogonal in the sense of [1, Definition 3.1]: there exist documents that satisfy any proper subset of the five dimensions while violating the complement.

## Proof.

By explicit construction. A document with all required fields present but with `weight: 1.5` satisfies structural and type constraints but violates range constraints. A document with `action.type: "code"` but missing `language` satisfies structural, type and range constraints but violates conditional constraints. A document with `domain: "UPPERCASE"` satisfies structural, type, range and conditional constraints but violates pattern constraints.

□

The pairwise orthogonality established above is *logical orthogonality*: each constraint dimension rules out defects not ruled out by others. Logical orthogonality does not by itself imply multiplicative probability reduction. To obtain the exponential defect reduction of [1, Theorem 3.1], an additional assumption is required: that defect events across constraint dimensions are probabilistically independent (or at least conditionally independent under the model distribution). Under this independence assumption, the joint acceptance probability under the constraint mask is bounded below by the product of per-dimension acceptance probabilities. Each additional independent constraint dimension multiplicatively reduces the probability of a defective output surviving the mask. Without the independence assumption, the defect reduction is additive (union bound) rather than multiplicative.

## 3.5  The Structural-Semantic Gap

The No-Bypass Theorem (Theorem 3.2) guarantees structural validity: every output of constrained decoding conforms to the GAP Meta-Schema. It does not guarantee semantic correctness. This distinction is fundamental and must be stated plainly.

A structurally valid GAP instruction may contain semantically harmful content. An LLM could produce a structurally perfect instruction with `action.type: "code"` and `content: "rm -rf /"`. The token mask permits this because `"rm -rf /"` is a valid JSON string; the Meta-Schema constrains the field's type (string) but not its semantic content. The instruction is structurally valid, governance-violating and potentially dangerous.

This gap is intrinsic to any system that enforces constraints via context-free grammars. CFGs can enforce syntactic structure (field presence, value types, enum membership, pattern matching) but cannot enforce semantic properties (is this code safe? does this action align with the user's intent? will this instruction cause harm in an unforeseen context?). Semantic properties require predicates that inspect the meaning of values, not their structure.

GAP addresses the structural-semantic gap through its three-layer architecture. Layer 1 (constraint masking) eliminates structural defects by construction. Layer 2 (gapc validation) checks static semantic properties: tool permission lists, composition cycle detection, trust floor compatibility. Layer 3 (15-step lattice) checks dynamic semantic properties: covenant evaluation (Step 7), Rego policy enforcement (Step 8), content scanning for injection, PII, secrets and jailbreak patterns (Step 14). Each layer narrows the gap between structural validity and semantic safety.

The gap is never fully closed. Open-ended semantic properties (is this instruction "useful"? will this action cause unintended consequences in an unforeseen context?) remain outside the scope of formal verification. Section 6.6 analyses the consequences of this gap for lattice convergence. Section 9.1 discusses mitigations.

## 3.6  Two-Phase Type Enforcement

GAP type enforcement operates in two phases.

**Phase 1 (logits-level).** Scalar types (string, integer, float, boolean, enum), structural constraints (required fields, nesting structure, enum membership, regex patterns, numeric ranges) and format constraints (JSON, YAML) are enforced during token generation by the constraint mask C_S. The No-Bypass Theorem (Theorem 3.2) applies to Phase 1: the probability of producing a Phase-1-violating output is exactly zero under sound constrained decoding.

**Phase 2 (lattice Step 3.5).** Mathematical types (Quaternion unit-norm, Matrix positive semi-definiteness, Tensor shape integrity, AABB min-less-than-max, OBB half-extent positivity, Frustum convex closure, spatial containment) are validated after generation by named validators at lattice Step 3.5. Phase 2 is a post-generation gate: deterministic, total and sound relative to validator implementation. Phase 2 is not token-level; the mathematical check runs on the complete generated output.

The gapc compiler determines the Phase 1/Phase 2 split based on format and field types (Section 14 of [10]). Both phases are mandatory when both apply. Phase 1 eliminates structural invalidity by construction. Phase 2 eliminates mathematical invalidity by post-generation validation. The conjunction provides defence in depth: a Quaternion field is structurally constrained to four floats at Phase 1 and unit-norm validated at Phase 2.

Step 3.5 is activated by the subprotocol field in the instruction's pattern. GAP v1.1 defines 11 built-in subprotocols: ui (scene graph, z-bands, skin bindings), workflows (state machines, phase verdicts), rest-api (HTTP methods, endpoints, request/response schemas), events (topics, payloads, producer permissions), iac (Kubernetes, Terraform, CI/CD), commerce (cart, checkout, payment, fulfillment, spend), tensor (shape, dtype, value range, NaN/Inf, positive semi-definiteness), molecular (atom types, valence, bond types, ring topology), drone (waypoints, altitude bands, sensor modes, formation), robotics (joint angles, torques, end-effector, collision zones) and material-science (crystal symmetries, alloy compositions, phase constraints). Custom subprotocols are registered via gapc subprotocol register. When no subprotocol is declared, Step 3.5 short-circuits to ALLOW at zero cost.

## 4  The Instruction Model

## 4.1  Instruction as Atomic Unit

## Definition 4.1 (GAP Instruction).

A GAP instruction is a tuple I = (A, P, R, w, C, G, T, E) where:

- A ∈ Addr is the address (domain × id, globally unique identifier)
- P is the pattern (guard condition defining when the instruction fires)
- R is the action (resolution producing the output)
- w ∈ [0, 1] is the weight (confidence and priority)
- C is the composition (relationship to other instructions)
- G is the governance metadata (provenance, trust ring, covenants, scanners, budget, proof, fleet governance, knowledge scope, perception modalities, tool permissions, write paths and event emissions)
- T is the type environment (inline type definitions with constraints and validators)
- E is the execution configuration (retry, timeout, exhaustion policy)

The governance metadata G contains 11 fields validated by the lattice: trust_ring (execution privilege, Ring 0-3, Step 2), covenants (behavioural constraints with [hard]/[soft] severity, Step 7), scanners (content scanner activation, Step 14), budget (token and cost limits, Step 10), proof (signing, ledger, timestamping), fleet (multi-instance caps, spawn policy with trust reduction and ring ceiling), knowledge (scoped knowledge base access with anti-context-rot ordering), perception (modality governance for speech, notification, sensor and visual channels), tools (allowed and forbidden tool lists, gapc enforces mutual exclusion), writes (filesystem paths the instruction may write to) and emits (typed event emissions that re-enter the routing system). All 11 fields are defined in the Meta-Schema and validated by the appropriate lattice step.

Every executable or governable entity in GAP is represented as an instruction. Agents, workflows, validators, compositions and governance rules are all expressed in the instruction format. This uniformity eliminates the distinction between "code" and "configuration" that plagues conventional agent frameworks.

## Definition 4.2 (Instruction Evaluation).

The evaluation of instruction I on input x is defined by:

```
eval(I, x) =
  let m = match(I.P, x)
  in  if m = FAIL then SKIP
      else let g = lattice(I.G, x, m)
           in  if g = DENY then REJECT(g.error)
               else let r = resolve(I.R, x, m)
                    in  let v = validate(r, I.P.output)
                        in  if v = FAIL then RETRY(v.error, I.E)
                            else COMMIT(r, I.G.proof)
```

The evaluation proceeds through four deterministic phases: pattern matching, governance (lattice) verification, action resolution and output validation. Each phase either succeeds or produces a structured error. The evaluation function is total: every input produces one of SKIP, REJECT, RETRY or COMMIT.

**Event-triggered evaluation.** Instructions may declare event subscriptions in the pattern field via pattern.on (event triggers or cron schedules). When the subscribed event fires, the instruction's eval function is invoked with the event payload as input x. The governance path is identical to direct invocation: pattern matching, lattice evaluation, action resolution and output validation all apply. Event-triggered evaluation does not bypass any lattice step.

Instructions may also declare event emissions in metadata.emits. Each emission carries a typed schema reference. Emitted events re-enter the routing system to trigger downstream instructions. The lattice validates emitted events at Step 14 (Content Scanners) before propagation. Event subscriptions and emissions together create governed event-driven composition where every link in the event chain passes through the full lattice.

## 4.2  Composition Operators (formal semantics)

GAP provides six composition operators, each with precise operational semantics.

## Definition 4.3 (Composition Operators).

Let I₁, ..., I_n be instructions over input space X.

1. **Atomic.** atomic(I)(x) = eval(I, x). Single instruction evaluation.

2. **Sequence.** seq(I₁, ..., I_n)(x) = eval(I_n, ... eval(I₂, eval(I₁, x)) ...). Sequential composition with output-to-input chaining. Evaluation halts on first REJECT after retry exhaustion.

3. **Conditional.** cond(c, I_then, I_else)(x) = if c(x) then eval(I_then, x) else eval(I_else, x). Branch based on runtime predicate c.

4. **Loop.** loop(I, until, k)(x) = let x₀ = x in for i = 1 to k: x_i = eval(I, x_{i-1}); if until(x_i) then return x_i. Bounded iteration with termination predicate. The bound k (max_iterations) is enforced by the lattice.

5. **Parallel.** par(I₁, ..., I_n, merge)(x) = merge({eval(I_j, x) : j ∈ {1, ..., n}}). Concurrent evaluation with merge strategy (all, any, majority).

6. **Abstraction.** abs(I₁, ..., I_n, select)(x) = eval(select(x, I₁, ..., I_n), x). Parameterized instruction family with variant selection based on pattern matching.

**Remark.** Gate is listed as a first-class composition primitive in the GAP specification's design principles. In the Meta-Schema v1.1, gate is realized as the >> operator within pipeline action types (Section 6.5 of [10]) and as lattice Step 11 (Gate Check) within the evaluation chain. It is not a standalone composition.type enum value. A future Meta-Schema revision may promote gate to a composition type; in v1.1, gate behaviour is accessed through pipeline steps or through Step 11 activation.

## Definition 4.4 (Pipeline Evaluation).

A pipeline action (action.type: pipeline) specifies an ordered list of steps with inline operators. Pipeline evaluation decomposes into composition operators:

- **Pipe** (arrow operator →): pipe(A, B)(x) = eval(B, eval(A, x)). Sequential with output chaining.
- **Parallel pipe** (bar operator |): par_pipe(A, B)(x) = merge_all({eval(A, x), eval(B, x)}). Concurrent execution, all must succeed.
- **Gate** (gate operator >>): gate(G)(x) = if gate_check(G, x) = ALLOW then x else SUSPEND(x). Activates lattice Step 11.

Pipeline steps are instruction references resolved at runtime. Each step passes through eval (Definition 4.2), which includes the lattice check. Pipeline evaluation therefore preserves governance by the same argument as Theorem 4.1: no step bypasses eval.

## Theorem 4.1 (Composition Preserves Governance).

Let I = compose(I₁, ..., I_n) be a composition under any of the six operators. If I passes gapc validation (Layer 2) and each constituent instruction I_j passes the 15-step lattice evaluation on its respective input, then:

(a) Every intermediate and final output has been individually lattice-validated.
(b) The composition's declared ring is at least as privileged as every constituent's required ring: ring(I) ≤ ring(I_j) for all j (since lower ring numbers denote greater privilege). No constituent instruction requires greater privilege than the composition declares.
(c) No composition cycle exists (the instruction dependency graph is a DAG).

## Proof.

**(a)** Each composition operator is defined (Definition 4.3) in terms of eval applied to constituent instructions. The eval function (Definition 4.2) includes the lattice check as its second phase: if lattice(I_j.G, x, m) = DENY, eval returns REJECT and the composition halts. No composition operator provides an alternative code path that bypasses eval. We verify this per operator:

- Sequence: seq(I₁, ..., I_n)(x) = eval(I_n, ... eval(I₁, x) ...). Each eval includes the lattice check. If any step returns REJECT, the chain halts (output-to-input chaining requires a successful output from the previous step).
- Parallel: par(I₁, ..., I_n, m)(x) = m({eval(I_j, x)}). Each eval executes independently with its own lattice check. Under merge strategy "all," any single REJECT causes the composition to fail.
- Conditional: cond(c, I_t, I_e)(x) evaluates exactly one branch via eval, which includes the lattice check.
- Loop: loop(I, p, k)(x) calls eval(I, ·) at each iteration. Each call includes the lattice check. The bound k prevents infinite iteration.
- Abstraction: abs(I₁, ..., I_n, s)(x) selects one variant and calls eval, which includes the lattice check.

**(b)** The trust floor compatibility property is enforced by gapc at validation time (Layer 2). For sequence compositions, gapc checks that ring(I) ≤ ring(I_j) for all j (i.e., the composition's ring number is no higher than any constituent's ring number, meaning the composition is at least as privileged as every constituent requires). If any constituent instruction requires greater privilege than the composition declares, gapc produces error E-COMPOSITION-TRUST-FLOOR and validation fails. The instruction cannot be loaded.

**(c)** gapc performs topological sort on the instruction dependency graph at validation time. A back-edge in the sort indicates a cycle, producing error E-COMPOSITION-CYCLE. Since the topological sort is deterministic and the graph is finite, cycle detection always terminates. Loop compositions are bounded by max_iterations and do not create dependency cycles (the loop body is a single instruction invoked repeatedly, not a circular reference).

□

## 4.3  The Self-Generation Fixed Point

## Definition 4.5 (Self-Generating Instruction).

An instruction I is self-generating if I.C.self_generate = true. When the resolution quality of I on input class K exceeds I.C.generation_constraints.min_quality_threshold, I produces a child instruction I' specialized to K.

## Definition 4.6 (Governance Preorder).

Define a governance preorder ⪰ on governance configurations: G₂ ⪰ G₁ (G₂ is at least as restrictive as G₁) if and only if every action denied by G₁ is also denied by G₂:

∀ a, deny_{G₁}(a) ⟹ deny_{G₂}(a)

This preorder captures the notion that child governance must be at least as restrictive as parent governance. Covenant set inclusion (G₂.covenants ⊇ G₁.covenants) is a sufficient condition for ⪰ when combined with the forbid-wins conflict resolution semantics (Section 6.2): adding covenants can only add deny verdicts, never remove them. However, covenant set inclusion alone does not guarantee ⪰ if covenants could conflict (e.g., one covenant permits an action that another forbids). The forbid-wins semantics resolves this: if a parent covenant forbids an action, the child inherits it and cannot override it, so the deny is preserved.

## Theorem 4.2 (Self-Governance Fixed Point).

If the three-layer enforcement pipeline (Layer 1: constrained decoding, Layer 2: gapc validation, Layer 3: 15-step lattice) is applied to the generation of child instructions, then for every parent instruction I and every child I' generated by I:

(a) I' ∈ L(S), where S is the GAP Meta-Schema (structural validity, enforced by Layer 1)
(b) G(I') ⪰ G(I), where ⪰ is the governance preorder of Definition 4.6 (governance monotonicity, enforced by Layer 3, Step 7 covenant inheritance with forbid-wins semantics)
(c) trust(I') ≤ trust(I) - I.G.fleet.spawn.trust_reduction (trust reduction, enforced by Layer 3, Step 9)
(d) ring(I') ≥ I.G.fleet.spawn.ring_ceiling (ring ceiling, enforced by Layer 3, Step 2; since lower ring numbers denote greater privilege, the child's ring number must be no lower than the ceiling, ensuring the child is equally or less privileged than the ceiling permits)
(e) depth(I') ≤ I.C.generation_constraints.max_generation_depth (bounded recursion, enforced by Layer 2)
(f) retry(I') ≤ retry(I) (retry cap monotonicity: self-generated children inherit the parent's execution configuration and cannot increase max_attempts beyond the parent's value, enforced by Layer 2)

## Proof.

Property (a) follows from the No-Bypass Theorem (Theorem 3.2) applied to the child generation process. The child is generated under the same constraint function C_S, so every token is constrained to the Meta-Schema.

Properties (b) through (d) follow from the 15-step lattice evaluation (Layer 3) applied to the child instruction itself. The lattice validates the instruction as a governed output:

- Step 7 (Covenant Evaluation) checks that I'.G.covenants ⊇ I.G.covenants. If any parent covenant is absent from the child, the lattice produces DENY. Combined with forbid-wins conflict resolution, this covenant set inclusion implies G(I') ⪰ G(I): every deny verdict of the parent is preserved in the child because the parent's forbid covenants are inherited and cannot be overridden.
- Step 9 (Capability + Trust) checks that trust(I') ≤ trust(I) - trust_reduction. If the child's trust exceeds the permitted level, DENY.
- Step 2 (Ring Capability) checks that ring(I') ≥ ring_ceiling (since lower numbers mean greater privilege, the child's ring number must not be lower than the ceiling). If the child is more privileged than the ceiling permits, DENY.

Property (e) is enforced by gapc validation (Layer 2), which is structurally prior to the lattice. The generation_constraints.max_generation_depth field is a static property of the instruction hierarchy. gapc counts the depth of the parent chain at load time. If depth(I') exceeds the bound, validation produces error E-GENERATION-DEPTH-EXCEEDED and the child is rejected before reaching the lattice.

Property (f) is enforced by gapc validation (Layer 2). The execution.retry.max_attempts field of a self-generated child is checked against the parent's value at load time. If the child's max_attempts exceeds the parent's, validation produces error E-EXECUTION-RETRY-EXCEEDED and the child is rejected before reaching the lattice.

Since Layer 2 blocks depth and retry-cap violations before activation and Layer 3 blocks governance violations deterministically, no child instruction can violate any of properties (a) through (f) and become active.

□

The self-governance fixed point means that GAP enforces itself recursively. Instructions generating instructions are constrained by the same Meta-Schema and the same lattice. The governance preorder can only strengthen: G(I') ⪰ G(I) ensures that child governance is at least as restrictive as parent governance, trust can only decrease and ring ceilings can only constrain. This monotonic governance property prevents governance erosion across generations.

## 4.4  Statistical Self-Generation via LRT (v1.1)

GAP v1.1 replaces the threshold-only self-generation trigger with a formal hypothesis test based on the likelihood ratio test (LRT). The test formulates two hypotheses:

- H₀: one instruction is sufficient (all input classes share the same quality distribution)
- H₁: specialization is needed (input class K has a significantly different quality distribution)

The LRT computes the log-likelihood ratio between H₀ and H₁ from observed execution data. Child creation is triggered only when the p-value falls below a configurable significance level (default 0.01) and the quality exceeds the absolute floor `min_quality_threshold`. This prevents variant proliferation from lucky runs or small-sample noise. When LRT is disabled, self-generation falls back to the threshold-only trigger.

The LRT gate interacts with the self-governance fixed point (Theorem 4.2): a child instruction created via LRT is still subject to all five governance properties. The LRT adds a statistical precondition to child creation but does not alter the governance invariants of the child itself.

## 4.5  Weight Estimation via MLE (v1.1)

Static weights degrade as input distributions shift. GAP v1.1 introduces maximum likelihood estimation (MLE) for weight adaptation. The lattice tracks (input_class, verdict, quality_score) per instruction. After a configurable minimum number of executions (default 50), the static weight is replaced by the MLE estimate: w_effective = argmax_θ L(θ | execution_data). A configurable floor (default 0.10) prevents MLE from reducing an instruction's weight to zero.

MLE weight estimation is a runtime property: it does not alter the instruction's static definition (which remains a valid GAP document under the Meta-Schema). The effective weight used for routing decisions is a function of the execution history, not of the instruction's static document content. This separation ensures that Layer 1 (constraint masking) is unaffected by MLE: the static weight field is still constrained to [0.0, 1.0] by the grammar.

## 4.6  Latent Governance Inference (v1.1)

Covenants are authored by instruction designers, but some governance preferences emerge only from observed rejection patterns. GAP v1.1 introduces latent governance inference, which observes the lattice's rejection history and proposes covenants that the data suggests are missing.

The mechanism tracks (rejection_step, scanner, input_class, count) per instruction. When the rejection rate for a specific tuple exceeds a configurable threshold (default 0.15) over a minimum number of observations (default 50), the system proposes a covenant tagged with `provenance: "latent"`. By default (`auto_apply: false`), proposals require human approval at lattice Step 11 (Gate Check) before activation.

Latent covenants follow the same monotonic governance rules as authored covenants: they can only strengthen governance (never weaken it), hard latent covenants cannot be overridden by children and self-generated children inherit latent covenants from their parent. The governance monotonicity of Theorem 4.2 extends to latent covenants without modification.

## 4.7  Provenance and Stability Tracking

Every GAP instruction carries provenance metadata tracing its origin. The `provenance` field records whether the instruction was human-authored, system-seeded or self-generated (with a pointer to the parent instruction). The `stability` field tracks the instruction's maturity through four states: `experimental` → `beta` → `stable` → `deprecated`.

Self-generated children start at `experimental` stability and must accumulate sufficient execution history (configurable via `promotion_threshold`) before promotion. This stability ladder prevents untested instructions from reaching production status. The `grade` field (integer 0 to 10) provides a quality score updated by the lattice based on execution outcomes.

## 5  Deployment Architecture

## 5.1  Tier 1: Logits-Level Enforcement (llama.cpp GBNF)

At Tier 1, the GAP Meta-Schema is compiled to a GBNF grammar and loaded into the llama.cpp inference engine [6]. The grammar defines a pushdown automaton that tracks the parse state during decoding. At every token position, the PDA computes the set of valid next tokens and the engine zeroes out all others.

## Proposition 5.1 (Tier 1 Guarantee).

Under Tier 1 deployment, the bypass probability is exactly zero. Every output is a valid GAP document.

## Proof.

Tier 1 implements constrained decoding (Definition 2.5) with the constraint function C_S. The result follows from the No-Bypass Theorem (Theorem 3.2).

□

Tier 1 is available on self-hosted infrastructure where the inference engine is under operator control. The inference engine runs llama.cpp with the GAP grammar loaded, constraining all agent outputs to valid canonical JSON conforming to the GAP Meta-Schema.

## 5.2  Tier 2: Post-Processing Validation Gate

When logits-level constraint masking is unavailable (e.g. when using remote API providers without structured output support), Tier 2 applies post-generation validation:

1. The LLM generates output without token-level constraints.
2. The gapc validator checks the output against the GAP Meta-Schema.
3. If validation fails, a structured error (with field path, expected value and suggestion) is fed back to the LLM.
4. The LLM retries with the error context. The constraint mask may still apply during retry if available.
5. After max_attempts retries (default 3), if no valid output has been produced, the instruction is REJECTED. No invalid output reaches execution.

## Proposition 5.2 (Tier 2 Guarantee).

Under Tier 2 deployment, no structurally invalid instruction ever becomes active. The probability of activation failure (no valid output after max_attempts retries) is bounded above by p^k, where p is the probability of a single-attempt validation failure and k = max_attempts.

## Proof.

The activation gate is deterministic: it blocks any output that fails gapc validation. An invalid output cannot bypass the gate because validation is a total, deterministic function (it terminates on every input and produces the same result for the same input).

For the bound: let q_i = Pr[F_i | F_1, ..., F_{i-1}] denote the conditional failure probability on the i-th attempt. Activation failure requires all k attempts to fail, so the exact probability is ∏_{i=1}^{k} q_i. Retry attempts are not independent: each retry includes the structured error from the previous attempt, which may narrow the model's output space. If there exists q_max such that q_i ≤ q_max for all i, then ∏_{i=1}^{k} q_i ≤ q_max^k. We note that error feedback does not guarantee monotonic improvement: while error-conditioned retries often reduce failure probability (q_{i+1} ≤ q_i), they can also confuse the model or trigger repeated failure modes. The bound p^k (where p = q_max) is therefore conservative only when q_max is a valid upper bound on all conditional failure probabilities.

□

## 5.3  Tier 3: Provider JSON Schema Mode

At Tier 3, the GAP Meta-Schema is submitted to a provider API (OpenAI structured outputs, Anthropic tool_use) as the `response_format` parameter with `json_schema` mode. The provider enforces schema compliance at their infrastructure level, applying their own constrained decoding implementation.

Coverage at Tier 3 depends on the provider's implementation fidelity. Provider structured-output modes generally support *subsets* of JSON Schema 2020-12, not the full specification. Support for recursive `$ref`, complex conditionals, `patternProperties`, numeric constraints and some combinators varies across providers. The GAP Meta-Schema is a standard JSON Schema 2020-12 document; however, Tier 3 guarantees are limited to the provider-supported subset of JSON Schema. A compatibility profile for the GAP Meta-Schema must be maintained per provider. Any constraints unsupported by the provider's structured-output mode must be rechecked by Layer 2 (gapc validation) after generation. The always-on Layer 2 validation gate ensures that provider-level gaps do not result in structurally invalid instructions reaching activation.

## 5.4  Hybrid Deployment and Graceful Degradation

In production, GAP operates across multiple tiers simultaneously. A self-hosted LLM uses Tier 1 (logits-level). A provider-hosted LLM uses Tier 3 (JSON Schema mode). A raw API without structured output support falls to Tier 2 (validation gate with retry).

Graceful degradation preserves the core guarantee: no invalid instruction becomes active. The tiers differ in efficiency (Tier 1 produces valid output in one pass; Tier 2 may require retries) but not in correctness. The three-layer enforcement pipeline (Layer 1: constrained decoding, Layer 2: gapc validator, Layer 3: 15-step lattice) provides defence in depth regardless of which tier is active.

The governance depth also varies across tiers. Under Tier 1 (logits-level), all three layers operate in full: Layer 1 enforces structural validity at every token, Layer 2 validates the complete instruction and Layer 3 runs all 15 lattice steps (8 always-mode, 7 active-mode). Under Tier 3 (provider JSON Schema mode), Layer 1 is delegated to the provider's constrained decoding implementation, whose fidelity depends on the provider's JSON Schema subset support; Layers 2 and 3 operate unchanged. Under Tier 2 (validation gate), Layer 1 is absent during generation: the model produces unconstrained output and Layer 2 applies post-hoc validation with retry. The always-mode lattice steps (1, 2, 3, 4, 7, 8, 9, 14) execute regardless of tier. The active-mode steps (0, 5, 6, 10, 11, 12, 13) execute only when their preconditions are met, providing configurable governance depth within each tier.

## 5.5  Binary Compilation Target

For high-frequency domains (robotics at 1 kHz, drone swarm coordination), GAP instructions may optionally be compiled from .gap YAML to .gapb binary via gapc compile --target bin. The binary format uses a fixed 64-byte header and 27 constraint opcodes (8 bytes each) dispatched via direct function-pointer table. The .gap file remains canonical; .gapb is a deterministic derivative. A VERIFY step in the compiler confirms that binary and source produce identical validation results before emission.

## 5.6  Performance Characteristics (analytical estimates)

This paper is a theoretical contribution and does not include empirical benchmarks. The estimates below are order-of-magnitude calculations derived from published implementation characteristics and asymptotic analysis. They are intended to demonstrate that governance overhead is architecturally feasible, not to predict production performance. Actual overhead depends on implementation quality, hardware configuration, grammar complexity, vocabulary size and inference engine version. Empirical validation on production workloads is required before these estimates can be treated as performance specifications.

**Layer 1 (constraint masking) overhead.** At each decoding step, the PDA state transition is O(1) (fixed-size state update). The token mask computation requires testing each token in the vocabulary against the current PDA state: O(|V|) per step. For |V| = 128 000, this is 128 000 comparisons per token. Contemporary CPUs execute approximately 10⁹ simple comparisons per second, yielding approximately 128 μs per token for a naive implementation. Optimized implementations (llama.cpp GBNF, Outlines FSM) use bitset representations of valid token sets and report 10-50 μs per token in published benchmarks [6, 11]; actual overhead depends on tokenizer trie implementation, grammar complexity, batching strategy, GPU/CPU split and cache locality. For a 1 000-token GAP instruction, total Layer 1 overhead is 10-50 ms, which is 0.1-5 % of typical LLM inference time (1-30 seconds for 1 000 tokens).

**Layer 2 (gapc validation) overhead.** Validation is a single-pass tree walk over the parsed JSON data model. Type reference resolution is O(n) in the number of fields. Constraint field validity checking is O(n × c) where c is the number of constraints. Composition cycle detection is O(V + E) via topological sort on the instruction dependency graph. For a typical GAP instruction (10-50 fields, 5-20 constraints, dependency graph of 1-10 instructions), total Layer 2 time is sub-millisecond.

**Layer 3 (15-step lattice) overhead.** Each step is a constant-time comparison (ring check: 1 integer comparison; covenant evaluation: O(k) string matches for k covenants; scanner execution: O(m × p) for m scanners with p patterns). For a typical instruction with 3-5 covenants, 4-6 scanners and a pattern library of 1 000 patterns, total Layer 3 time is 1-10 ms.

**Total per-instruction cost.** Layer 1 dominates at 10-50 ms per instruction. Layers 2 and 3 add less than 15 ms. Total governance overhead per instruction is approximately 25-65 ms, negligible relative to LLM inference time. The overhead is fixed per token and does not grow with model size (it depends on grammar complexity and vocabulary size, both of which are model-independent). In batched inference, per-sequence grammar state can reduce batch efficiency because different sequences may require different masks, limiting parallelism relative to unconstrained batched generation. Constrained decoding can also alter the model's output distribution and sometimes increase average output length.

**Comparison to unconstrained generation.** Without constraint masking, post-hoc validation (Tier 2) adds zero per-token overhead but incurs retry cost. If the probability of a single-attempt validation failure is p, the expected number of attempts is 1/(1-p). For complex schemas where p ≈ 0.3-0.7 (common with raw API generation), the expected retry cost is 1.4-3.3× the base inference cost. Tier 1 constraint masking trades a small per-token overhead (< 5 % of inference time) for elimination of all retries.

## 6  The 15-Step Adjudication Lattice

The 15-step evaluation chain satisfies the structural requirements of the abstract adjudication lattice framework introduced in [1] (see Observation 6.1 below). Every instruction action passes through all 15 steps. The chain produces ALLOW or DENY. The evaluation is deterministic: same input, same governance state, same result.

**Table 6.1. The 15-step GAP adjudication lattice.**

| Step | Name | Mode | Predicate | Failure Code | Soundness Basis |
|-----:|------|------|-----------|-------------|-----------------|
| 0 | Input canonicalization | active | Normalise input encoding and structure | E-INPUT-CANONICAL | structural |
| 1 | Session validity | always | Verify session token and agent identity | E-SESSION-INVALID | structural |
| 2 | Ring capability | always | agent_ring ≤ operation_ring | E-RING-ESCALATION | arithmetic |
| 3 | System rate limit | always | Request count within planetwide rate window | E-RATE-LIMIT-EXCEEDED | counter comparison |
| 3.5 | Complex structure validation | active | Mathematical validators (unit_norm, psd_check, tensor_integrity, aabb_valid, obb_valid, frustum_check, spatial_containment, format_check) execute when subprotocol is declared; zero cost when absent | E-VALIDATOR-{NAME} | validator-relative |
| 4 | Session rate limit | always | Request count within per-session rate window | E-SESSION-RATE-EXCEEDED | counter comparison |
| 5 | Intent drift | active | Five heuristics: tool distribution, target scope, prefix drift, frequency anomaly, repetition | E-INTENT-DRIFT | heuristic (pattern-library-relative) |
| 6 | Scope validation | active | Action targets within declared scope boundaries | E-SCOPE-VIOLATION | structural |
| 7 | Covenant evaluation | always | All hard/soft covenants satisfied; forbid-wins conflict resolution | E-COVENANT-VIOLATION | deterministic string matching |
| 8 | Rego policy enforcement | always | OPA policy evaluation returns allow | E-POLICY-DENIED | policy-relative |
| 9 | Capability + trust scoring | always | Agent trust score ≥ operation trust floor | E-TRUST-INSUFFICIENT | arithmetic |
| 10 | Budget enforcement | active | Token and cost within per-instruction and daily caps (active when budget constraints are declared) | E-BUDGET-TOKEN-EXCEEDED, E-BUDGET-COST-EXCEEDED | arithmetic |
| 11 | Gate check | active | Human-in-the-loop or auto-approval gate | E-GATE-DENIED | procedural |
| 12 | Composition integrity | active | No cycles, trust floors compatible, bounded iteration | E-COMPOSITION-CYCLE, E-COMPOSITION-TRUST-FLOOR | topological sort |
| 13 | Provenance verification | active | Instruction provenance chain valid, stability sufficient | E-PROVENANCE-INVALID | structural |
| 14 | Content scanners | always | 11 scanners (PII, injection, secrets, jailbreak, toxicity, URL, data profiler, prediction, brand, regulatory, temporal) | E-SCANNER-{TYPE} | pattern-library-relative |

## 6.1  Ring Capability Check

Step 2 of the lattice verifies that the agent's execution ring permits the requested operation type. GAP defines four rings in a strict hierarchy:

- Ring 3 (sandbox): minimal permissions, read-only, no external calls
- Ring 2 (user): standard permissions, scoped tool access
- Ring 1 (system): elevated permissions, infrastructure access
- Ring 0 (enterprise): full permissions, cross-system governance

An instruction at Ring 2 cannot perform operations requiring Ring 1 or Ring 0 capabilities. Since lower ring numbers denote greater privilege, the ring check is a simple inequality: agent_ring ≤ operation_ring (the agent must be at least as privileged as the operation requires). If the check fails, the lattice produces DENY with error code E-RING-ESCALATION.

## 6.2  Covenant Evaluation

Step 7 evaluates behavioural covenants attached to the instruction. Covenants are natural-language constraints with severity tags: `[hard]` covenants produce DENY on violation; `[soft]` covenants produce a warning and allow recovery.

Covenant evaluation is deterministic: each covenant is checked against the instruction's output using the declared constraint expressions. The key governance invariant is monotonicity: child instructions inherit all parent covenants and cannot weaken them. New covenants can be added (strengthening governance) but existing covenants cannot be removed or relaxed.

Forbid rules always take precedence over permit rules. If a covenant forbids an action and another covenant permits it, the forbid wins. This conflict-resolution semantics eliminates ambiguity.

## 6.3  Content Scanners

Step 14 applies 11 content scanners to the instruction's output:

| Scanner | Severity | Target |
|---------|----------|--------|
| PII | hard | Personal identifiable information |
| Injection | hard | SQL, script, prompt injection patterns |
| Secrets | hard | API keys, credentials, private keys |
| Jailbreak | hard | Prompt manipulation attempts |
| Toxicity | hard | Harmful content |
| URL | soft | Suspicious or disallowed URLs |
| Data Profiler | soft | Statistical profiling of data distributions |
| Prediction | soft | Speculative claims without evidence |
| Brand | soft | Brand guideline violations |
| Regulatory | soft | Regulatory compliance flags |
| Temporal | soft | Anachronistic or time-inconsistent claims |

Hard-severity scanners produce DENY on detection. Soft-severity scanners produce warnings that are logged but do not block execution. Scanner activation is configurable per instruction via the `scanners` field in metadata.

## 6.4  Trust Scoring and Budget Enforcement

Step 9 evaluates the agent's trust score (integer 0 to 1000, five tiers) against the operation's trust requirements. Step 10 enforces budget limits: per-instruction token limits (`max_tokens`), per-instruction cost limits (`max_cost`) and daily cost caps (`daily_cap`). If any budget limit is exceeded, the lattice produces DENY with error code E-BUDGET-TOKEN-EXCEEDED or E-BUDGET-COST-EXCEEDED.

Budget enforcement is deterministic: the lattice tracks cumulative token usage and cost per instruction and per session. The tracking state is maintained across invocations.

## 6.5  Integration with AEP Lattice Framework

The 15-step evaluation chain is a constructive instantiation of the adjudication lattice defined in [1, Definition 2.6]. We make this relationship precise.

## Observation 6.1 (GAP Lattice as AEP Instantiation).

The 15-step GAP evaluation chain T_GAP = (v₀, v₁, ..., v₁₄) satisfies the structural requirements of an adjudication lattice as defined in [1]. We verify the three predicate properties required by [1, Definition 2.3] and identify the conditions under which the convergence theorem of [1, Theorem 3.3] applies.

**Totality.** Every step terminates on every input. Always-mode steps (1, 2, 3, 4, 7, 8, 9, 14) execute unconditionally and terminate because they perform finite comparisons on finite data (ring inequality, covenant string matching, scanner pattern matching). Active-mode steps (0, 5, 6, 10, 11, 12, 13) short-circuit to ALLOW when their precondition is not met, ensuring termination regardless of input.

**Determinism.** Each step is a pure function of the input, the instruction's governance metadata and the lattice state. No step involves randomness or non-deterministic choice. Same input and same state produce same output. This follows from the implementation: each step is a conditional expression over finite, typed fields (ring levels are integers, covenant strings are finite, budget counters are floats).

**Soundness.** Soundness is the strongest claim and the one that requires the most care. Each step is sound with respect to its defect category if and only if the predicate correctly identifies all defective outputs. For structural predicates (v₂ ring check: integer comparison; v₃ system rate limit: counter comparison; v₄ session rate limit: counter comparison; v₁₀ budget: arithmetic comparison), soundness follows from the correctness of integer and floating-point comparison, which is established by the hardware. For semantic predicates (v₅ intent drift: five heuristics including prefix drift and frequency anomaly; v₁₄ content scanners: pattern matching for PII, injection and secrets), soundness depends on the completeness of the pattern libraries. An injection scanner that lacks a specific injection pattern is unsound for that pattern class. We do not claim universal soundness for semantic predicates; we claim soundness relative to their pattern libraries and flag this as a limitation (Section 9.1).

**Partial ordering.** Steps checking orthogonal properties (ring capability vs. content scanning vs. budget enforcement) are incomparable. Steps with hierarchical relationships (session validity at Step 1 is prerequisite for session rate limiting at Step 4) satisfy the ordering relation. The lattice filter T_GAP accepts an instruction action if and only if every step accepts it, consistent with [1, Definition 2.7].

**Applicability of convergence.** The convergence theorem of [1, Theorem 3.3] applies to T_GAP when the soundness condition holds for all 15 predicates. For structural predicates, this holds by construction. For semantic predicates, the convergence guarantee is relative to the pattern library's coverage. Under population-based generation with population size N = ⌈ln(δ)/ln(1 - α_T)⌉ (equivalently, N = ⌈ln(1/δ)/ln(1/(1 - α_T))⌉; the approximation N ≈ ⌈ln(1/δ)/α_T⌉ holds for small α_T), the probability of producing an action that passes all 15 steps converges to 1 - δ. Every surviving action is zero-defect across all defect categories for which the predicates are sound.

The relationship between AEP and GAP is architectural: AEP provides the mathematical proof that lattice-based verification converges to zero-defect [1, Theorem 3.3]; GAP provides the engineering machinery to deploy that lattice at the token level on real LLMs. Constraint masking (Section 3) provides the Layer 1 enforcement that makes the AEP lattice practically deployable: by guaranteeing structural validity before any semantic or governance check, it ensures that the lattice operates on well-formed inputs, increasing the acceptance probability α_T and reducing the population size required for convergence.

## 6.6  Semantic Predicate Unsoundness and Its Consequences

Section 3.5 identified the structural-semantic gap as the intrinsic boundary of constraint masking. Here we quantify the convergence consequences of that gap for the lattice. The convergence guarantee of [1, Theorem 3.3] requires that all lattice predicates are sound. For structural predicates (ring check, rate limiting, budget enforcement), soundness follows from the correctness of arithmetic comparison and is not in question. For semantic predicates (intent drift at Step 5, content scanners at Step 14), soundness is contingent on the completeness of their pattern libraries. This contingency has precise consequences for the lattice's defect coverage. Appendix D provides synthetic simulation data illustrating these consequences quantitatively.

**Injection scanner unsoundness.** The injection scanner (Step 14, hard severity) detects SQL injection, script injection and prompt injection via pattern matching. If an injection pattern is absent from the scanner's library, the scanner is unsound for that pattern class: it will accept an instruction containing the unrecognized injection. The lattice as a whole then fails to detect that defect category. The convergence theorem still applies to the remaining 14 steps and to all injection patterns that are in the library; it does not apply to the missing pattern class. Adding the pattern to the library restores soundness for that class without altering the lattice structure.

**Intent drift false negatives.** The intent drift detector (Step 5, active mode) uses five heuristics: tool distribution, target scope, prefix drift, frequency anomaly and repetition. Each heuristic is a binary classifier with its own false-negative rate. A sophisticated drift pattern that evades all five heuristics constitutes a false negative. The lattice's intent-drift predicate is sound only to the coverage of these heuristics.

**Impact on convergence bounds.** Let S ⊂ {v₀, ..., v₁₄} be the subset of predicates that are provably sound (all structural predicates) and let U = {v₀, ..., v₁₄} \ S be the unsound predicates. The convergence theorem of [1] applies to the sub-lattice T_S = ∏_{v ∈ S} v: population-based selection converges to zero defect across all defect categories covered by S. For defect categories covered only by predicates in U, the residual defect probability depends on the relationship between predicates and defect classes. Two cases arise:

*Case 1: Multiple independent detectors for the same defect class.* If a defect class d is checked by detectors v₁, ..., v_k whose false negatives are conditionally independent given d, then Pr[d survives] = p_d · ∏_{i=1}^{k} ε_i, where p_d is the base rate of defect class d and ε_i is the false-negative rate of detector v_i.

*Case 2: Different detectors for different defect classes.* When distinct predicates cover distinct defect classes, the total residual defect probability is bounded by a union bound: Pr[any semantic defect survives] ≤ Σ_{d ∈ D} p_d · ε_d, where p_d is the base rate and ε_d is the false-negative rate for defect class d.

The product bound ∏_{v ∈ U} ε_v applies only under Case 1 (same defect checked by multiple independent detectors). When different predicates address different defect categories, the union bound of Case 2 is the correct formulation. Both bounds decrease as pattern libraries are extended (reducing each ε_d) and reach zero when all predicates are sound.

This analysis makes explicit the honest boundary of the framework's guarantees: convergence to zero structural defect is unconditional; convergence to zero semantic defect is conditional on pattern library completeness.

## 6.7  Abstract Governance Runtime Interface

GAP is runtime-parametric: it specifies the artifact; the runtime adjudicates the artifact. The 15-step lattice described above is one concrete instantiation. To establish that GAP's formal properties hold independently of any specific runtime implementation, we define a minimal abstract interface that any governance runtime must satisfy.

**Definition 6.2 (Governance Runtime).** A governance runtime is a function R: I × X × Σ → V × Σ' × E, where I is a valid GAP instruction (I ∈ L(S)), X is the input/context, Σ is runtime state, V ∈ {allow, deny, soft_violation} is the verdict, Σ' is the updated state and E is an evidence record. The runtime must satisfy five properties:

**R1. Totality.** For all valid instructions I ∈ L(S), inputs x ∈ X and states σ ∈ Σ, R(I, x, σ) terminates with a verdict. No valid instruction causes the runtime to diverge.

**R2. Determinism.** R is a mathematical function of (I, x, σ), not a randomised relation. For any two invocations with identical arguments: (I, x, σ) = (I', x', σ') ⟹ R(I, x, σ) = R(I', x', σ'). No randomness inside hard adjudication.

**R3. Hard-predicate soundness.** If the runtime allows an instruction, all hard predicates accept: R(I, x, σ).verdict = allow ⟹ ∧_{p ∈ P_hard} p(I, x, σ) = accept. Hard predicates are the subset of lattice steps with hard severity (ring check, covenant hard violations, injection scanner, secrets scanner, etc.).

**R4. Monotonic child governance.** If instruction I creates child I' through self-generation, then child governance is at least as restrictive under the governance preorder (Definition 4.6): G(I') ⪰ G(I).

**R5. Evidence production.** Every non-skip verdict emits an evidence record E = evidence(I, x, σ, V). The evidence may include implementation-specific proof material, but GAP does not prescribe evidence internals.

**Observation 6.2.** The 15-step GAP lattice satisfies R1-R5. R1 follows from totality of each step (Section 6.5). R2 follows from determinism of each step. R3 follows from the lattice's conjunction semantics (all steps must accept for the lattice to allow). R4 follows from Theorem 4.2. R5 follows from the proof bundle produced by each lattice evaluation (I.G.proof).

**Optional runtime extensions.** A governance runtime may additionally satisfy:

**R6. Trajectory bounding (optional).** The runtime maintains a cumulative state vector across evaluations and blocks actions whose prospective cumulative effect would exceed a configurable drift threshold. This detects individually-acceptable actions that cumulatively shift system behaviour beyond an operator-defined envelope.

**R7. Multi-engine verification (optional).** The runtime evaluates hard predicates through multiple independent verification implementations and detects verdict disagreement before enforcement. This reduces single-engine verification risk through heterogeneous redundancy.

**R8. Convention memory (optional).** The runtime stores parameterized templates derived from previously validated outputs and matches structurally equivalent future inputs against them. This enables validation cost to decrease over time as the runtime accumulates validated patterns. Exact cryptographic matches may reuse prior proof bundles. Near-matches must still pass all hard predicates.

Properties R6-R8 are not required for GAP's structural guarantees. A runtime satisfying only R1-R5 is fully compliant. R6 adds trajectory-level safety. R7 adds redundant verification. R8 adds amortized validation cost. The formal treatment of runtimes satisfying R6-R8 is deferred to future work.

**Observation 6.3 (Validation Depth Levels).** The combination of required (R1-R5) and optional (R6-R8) runtime properties defines three validation depth levels:

*Level A (GAP-native).* A runtime satisfying R1-R5 with pattern-library-relative semantic predicates. Structural bypass probability is zero. Semantic defect probability is bounded by the union bound of Section 6.6.

*Level B (formally augmented).* A Level A runtime where some semantic predicates are backed by formal solvers (SMT, model checkers, proof assistants). These predicates become provably sound for all inputs within the encoded domain, eliminating defect classes rather than bounding their probability.

*Level C (stateful).* A Level B runtime additionally satisfying R6 (trajectory bounding), R7 (multi-engine verification) and R8 (convention memory). Provides per-action verification, trajectory-level drift detection, redundant verification and amortized validation cost.

GAP's formal results hold at all three levels. The levels differ in the strength and coverage of semantic predicates.

**Remark.** GAP is compatible with any deterministic adjudication runtime satisfying R1-R5, including stateless validators, stateful policy engines and proof-producing runtimes. Advanced runtime implementations may optimise, enrich or statefully refine adjudication, but those mechanisms are outside the scope of this paper. The language requires evidence production but does not prescribe evidence internals.

## 7  Formal Semantics

## 7.1  Operational Semantics of gapc run

We define small-step operational semantics for instruction evaluation. Let Σ = (I, x, σ) be a configuration, where I is the instruction being evaluated, x is the current input and σ is the execution state (including lattice state, retry count and accumulated evidence).

The transition rules are:

**Match.**
⟨I, x, σ⟩ →_match ⟨I, x, σ[matched := m]⟩  if match(I.P, x) = m ≠ FAIL

**Skip.**
⟨I, x, σ⟩ →_skip SKIP  if match(I.P, x) = FAIL

**Governance.**
⟨I, x, σ[matched := m]⟩ →_gov ⟨I, x, σ[governed := g]⟩  where g = lattice(I.G, x, m)

**Reject.**
⟨I, x, σ[governed := DENY(e)]⟩ →_rej REJECT(e)

**Resolve.**
⟨I, x, σ[governed := ALLOW]⟩ →_res ⟨I, x, σ[result := r]⟩  where r = resolve(I.R, x, σ.matched)

**Validate.**
⟨I, x, σ[result := r]⟩ →_val ⟨I, x, σ[validated := v]⟩  where v = validate(r, I.P.output)

**Retry.**
⟨I, x, σ[validated := FAIL(e), retries := k]⟩ →_retry ⟨I, x, σ[retries := k+1, result := ⊥, validated := ⊥, recovery_error := e]⟩  if k < I.E.max_attempts

The retry transition clears the result and validation state and stores the error as recovery context, allowing the Resolve rule to execute again with error feedback.

**Exhaust.**
⟨I, x, σ[validated := FAIL(e), retries := k]⟩ →_exhaust REJECT(e)  if k ≥ I.E.max_attempts

**Commit.**
⟨I, x, σ[validated := PASS]⟩ →_commit COMMIT(σ.result, I.G.proof)

These transition rules define a deterministic evaluation strategy. Every configuration has at most one applicable rule (rules are mutually exclusive based on the state fields). Evaluation terminates in one of three states: SKIP (pattern did not match), REJECT (governance or validation failed) or COMMIT (successful evaluation with evidence).

## 7.2  Denotational Semantics of Composition

We assign denotational semantics to each composition operator as a function over the domain D = X → (Y + Error), where X is the input space, Y is the output space and Error is the space of structured errors.

**Atomic.** ⟦atomic(I)⟧ = eval(I, ·)

**Sequence.** ⟦seq(I₁, ..., I_n)⟧ = ⟦I_n⟧ ∘ ... ∘ ⟦I₂⟧ ∘ ⟦I₁⟧, where composition propagates errors (if any ⟦I_j⟧ returns an error, the composition returns that error).

**Conditional.** ⟦cond(c, I_t, I_e)⟧(x) = if c(x) then ⟦I_t⟧(x) else ⟦I_e⟧(x)

**Loop.** ⟦loop(I, p, k)⟧(x) = fix_k(λf. λy. if p(y) then y else f(⟦I⟧(y)))(x), where fix_k is the k-bounded fixpoint (unrolling at most k times).

**Parallel.** ⟦par(I₁, ..., I_n, m)⟧(x) = m({⟦I_j⟧(x) : j ∈ {1, ..., n}}), where m is the merge function.

**Abstraction.** ⟦abs(I₁, ..., I_n, s)⟧(x) = ⟦s(x, I₁, ..., I_n)⟧(x), where s selects the best-matching variant.

## 7.3  Type Soundness

## Definition 7.1 (Well-Typed Instruction).

An instruction I is well-typed if gapc validation succeeds: all type references resolve, all constraint expressions reference existing fields, no composition cycles exist and all trust floors are compatible across sequence steps.

## Definition 7.2 (Validator Soundness).

A validator v for type T is sound if for every output r and type specification T:

v(r, T) = PASS ⟹ r conforms to T

where conformance is defined recursively: scalar types conform if the value is of the declared type and satisfies all range constraints; mathematical types (Vector2f, Vector3f, Vector4f, Quaternion, Matrix3x3, Matrix4x4, Tensor, AABB, OBB, Frustum) conform if the named validator (range_check, unit_norm, psd_check, tensor_integrity, aabb_valid, obb_valid, frustum_check) accepts; complex types conform if all fields conform to their declared types; collection types conform if all elements conform to the item type and all aggregate constraints hold.

For GAP's built-in validators, soundness is established by the implementation: range_check performs floating-point comparison (sound for IEEE 754 arithmetic), unit_norm computes ‖q‖₂ and compares to 1 within declared tolerance (sound for finite-precision arithmetic within tolerance). The same pattern applies to each named validator. Custom validators registered via `gapc validator register` are trusted by the framework; their soundness is the responsibility of the registrant.

## Theorem 7.1 (Type Soundness).

If instruction I is well-typed and input x conforms to I.P.input.type, then eval(I, x) either:

(a) produces an output y conforming to I.P.output.type (COMMIT) or
(b) produces a structured error E (REJECT or SKIP) or
(c) retries up to I.E.max_attempts times and then produces (a) or (b).

No evaluation path reaches COMMIT with an output that has not passed the declared output validator.

## Proof.

The proof proceeds by case analysis on the evaluation transitions (Section 7.1).

SKIP: produced when match fails. No output value is generated.

REJECT: produced when the lattice returns DENY or retries are exhausted. The error E is structured (Section 15.2 of [10]).

COMMIT: produced when validation succeeds. The validate function checks the output r against I.P.output.type. If validate returns PASS, then r conforms to the output type by soundness of the validator. The validator is sound because it checks structural conformance against the type definition (which is a decidable property for all GAP types, including mathematical types validated by named executors).

No transition rule produces a value outside these three cases. The transition system is exhaustive (every configuration matches exactly one rule) and deterministic (no configuration matches two rules).

□

## 7.4  Preservation and Progress

## Theorem 7.2 (Preservation).

If configuration ⟨I, x, σ⟩ is well-formed and ⟨I, x, σ⟩ → ⟨I, x, σ'⟩, then ⟨I, x, σ'⟩ is well-formed, and any committed result is type-conformant.

We distinguish two kinds of configurations. A *checked configuration* is one in which all stored values have been validated against their declared types. A *pending configuration* is one in which `result := r` may hold an unvalidated value awaiting the validate step. The resolve step may produce a pending configuration (the result r has not yet been type-checked); the validate step transitions the configuration from pending to checked (or triggers retry/exhaust on failure). Preservation means that the machine state remains well-formed across all transitions, and that no value is committed without passing validation.

## Proof.

Each transition rule preserves well-formedness. The match step produces a match result m that is compatible with the pattern's type declarations (checked). The governance step produces a verdict (ALLOW or DENY) that does not alter the instruction's types (checked). The resolve step produces a result r that may be pending: it has not yet been validated against the output type. The validate step checks r against I.P.output.type; if validation succeeds, the configuration becomes checked and r is committed. If validation fails, the configuration transitions to retry or exhaust, and no unvalidated value is committed. No transition introduces untyped values in committed state.

Note: custom validators registered via `gapc validator register` are trusted by the framework; their soundness is the responsibility of the registrant. Floating-point validators (e.g., unit_norm) are sound relative to specified tolerances and IEEE 754 arithmetic behaviour.

□

## Theorem 7.3 (Progress).

If configuration ⟨I, x, σ⟩ is well-typed and is not a terminal state (SKIP, REJECT or COMMIT), then there exists a configuration σ' such that ⟨I, x, σ⟩ → ⟨I, x, σ'⟩.

## Proof.

The transition rules cover all non-terminal configurations. If matched is not set, the match rule applies. If matched is set but governed is not, the governance rule applies. If governed is DENY, the reject rule applies. If governed is ALLOW and result is not set, the resolve rule applies. If result is set and validated is not set, the validate rule applies. If validated is FAIL and retries < max, the retry rule applies. If validated is FAIL and retries ≥ max, the exhaust rule applies. If validated is PASS, the commit rule applies.

Every non-terminal configuration matches exactly one rule. Evaluation always makes progress toward a terminal state.

□

## 8  Comparison: GAP vs Existing Approaches

## 8.1  vs. Prompt-Level Guardrails (NeMo, Llama Guard)

NeMo Guardrails [2] and Llama Guard [3] constrain LLM behaviour through prompt-level instructions and post-hoc classifiers. Both are probabilistic: they reduce the expected defect rate but provide no formal guarantee that defective outputs are eliminated. NeMo Guardrails use programmable dialogue flows (Colang) to define conversational constraints; Llama Guard uses a trained classifier to detect unsafe outputs.

GAP differs in two respects. First, constraint enforcement occurs at the token level before selection, not after generation. The defective output is never produced, not merely detected and discarded. Second, the constraint is derived from a formal schema (JSON Schema 2020-12), not from a trained classifier. The schema is verifiable by inspection; a classifier's behaviour is opaque.

## 8.2  vs. Post-Hoc Validation (dottxt, Outlines)

The Outlines library [11] and its commercial deployment dottxt provide constrained decoding from JSON Schema, regex and CFG specifications. Outlines compiles these specifications into finite-state machines and applies them at the logits level.

GAP subsumes this capability as a single action flag: `structured_generation: true`. When this flag is set, gapc compiles the instruction's type definitions into a constraint mask and applies it during decoding. GAP can optionally delegate this mask to Outlines as an external accelerator.

The difference is scope. Outlines provides structural constraint enforcement. GAP extends structural constraint enforcement with governance metadata, deterministic adjudication hooks (the 15-step lattice), composition semantics, self-generation with monotonic governance constraints, provenance tracking, budget enforcement and runtime evidence production. Structural constraint masking in GAP is Layer 1 of a three-layer enforcement pipeline; in Outlines, it is the entire system.

## 8.3  vs. RLHF and Constitutional AI

RLHF [4, 5] and Constitutional AI [12] modify the model's distribution to prefer aligned outputs. Both operate at training time: they alter the model's parameters so that P_M assigns higher probability to desired outputs and lower probability to undesired ones. The resulting distribution is improved but still assigns nonzero probability to all outputs, including structurally invalid ones.

GAP operates at inference time and is orthogonal to training-time alignment. A model trained with RLHF can be further constrained by GAP at the token level. The RLHF alignment improves the semantic quality of outputs within the GAP-valid space; GAP ensures that only structurally valid outputs are produced. The two approaches compose without interference.

## 8.4  vs. Traditional Programming Language Type Systems

Traditional type systems (Hindley-Milner [13], System F [14], dependent types [15]) provide compile-time guarantees for programs written by human developers. They verify that programs are type-safe before execution.

GAP's constraint masking can be viewed as a type system for LLM outputs, but with a fundamental difference in enforcement timing. Traditional type systems reject ill-typed programs after they are written. GAP's constraint masking prevents ill-typed outputs from being written in the first place. The type system operates during generation, not after generation.

GAP's type system includes ten mathematical types: Vector2f, Vector3f, Vector4f (per-component range checking), Quaternion (unit-norm invariant, ||q|| = 1 within tolerance), Matrix3x3 and Matrix4x4 (symmetry, positive semi-definiteness, orthogonality, affine structure), Tensor (shape, dtype, value range, NaN/Inf rejection), AABB (axis-aligned bounding box, min < max on all axes), OBB (oriented bounding box, half-extents positive, rotation unit-norm) and Frustum (six planes forming closed convex volume). These types have no analog in conventional type systems. These types are enforced through two-phase validation: simple constraints at the logits level (Phase 1) and mathematical validators after generation at lattice Step 3.5 (Complex Structure Validation), which executes when a subprotocol is declared (Phase 2).

## 9  Discussion

## 9.1  Limitations

The following subsections enumerate the framework's known limitations, organised by category. Table 1 summarises the guarantee structure across all three layers.

**Table 1. Layer-by-layer guarantee structure.**

| Layer | What It Guarantees | Soundness Basis | Residual Risk | Mitigation Path |
|-------|-------------------|-----------------|---------------|-----------------|
| 1 (Constraint Masking) | Structural validity | By construction (Theorem 3.2) | Compiler bugs | Formal verification of compiler |
| 2 (gapc Validation) | Static semantic properties | Relative to gapc rule set | Tool misuse, composition errors | Expand gapc rule set |
| 3 (15-Step Lattice) | Dynamic semantic properties | Relative to pattern libraries | Unknown attack patterns, intent drift | Continuous pattern library updates + latent governance (v1.1) |

### 9.1.1  Compiler Soundness Assumption

The No-Bypass Theorem (Theorem 3.2) depends on the soundness of the compilation from JSON Schema to constraint function. If the compiler contains a bug that permits an invalid token, the guarantee is voided. Compiler soundness is a conventional software correctness problem, addressable by testing, fuzzing and formal verification of the compiler itself. The GBNF grammars used by llama.cpp have been extensively tested but are not formally verified. Mechanized proof of compiler correctness (e.g. in Lean or Coq) would upgrade this from an engineering assumption to a mathematical guarantee.

### 9.1.2  The Structural-Semantic Gap (Intrinsic Boundary)

As analysed in Section 3.5, constraint masking guarantees structural validity but not semantic correctness. This is the primary limitation of Layer 1 enforcement. Layers 2 and 3 narrow the gap: gapc validates tool permission lists (Layer 2); the lattice checks covenants (Step 7), Rego policies (Step 8) and content scanners (Step 14) at Layer 3. These checks are deterministic and sound relative to their pattern libraries. Section 6.6 analyses the convergence consequences of pattern library incompleteness.

### 9.1.3  Semantic Predicate Coverage and Residual Risk

The residual risk is that a semantically harmful action falls outside the coverage of all covenants, Rego policies and scanner patterns. The framework provides defence in depth but does not claim semantic omniscience. Under the analysis of Section 6.6, if the injection scanner has coverage c (fraction of known injection patterns recognised), the false-negative rate for that predicate is (1 - c). For a scanner with 92% pattern coverage, the false-negative rate is 0.08, contributing an upper bound of 0.08 to the residual semantic defect probability for that defect category. When distinct predicates cover distinct defect classes, the total residual probability across all semantic predicates is bounded by the union bound Σ_{d ∈ D} p_d · ε_d (Section 6.6). When multiple independent detectors cover the same defect class, a product bound applies to that class. If two scanners share underlying pattern libraries, their errors may correlate, invalidating both the product and the independence assumptions; correlated scanners should be treated as a single detector for residual-risk analysis. Appendix D provides illustrative analytical scenarios showing how pattern library coverage affects population size requirements.

### 9.1.4  Threat Model and Attack Surface

GAP's threat model assumes a potentially adversarial model (one that may attempt to produce harmful outputs) operating within a trusted enforcement pipeline (the constraint compiler, gapc validator and lattice implementation are trusted code). The threat model does not cover:

- **Compromised compiler.** If the constraint compiler itself is malicious or contains a backdoor, all Layer 1 guarantees are void. This is a supply-chain risk mitigated by code review, reproducible builds and formal verification.
- **Compromised lattice implementation.** If the lattice implementation skips steps or returns ALLOW unconditionally, all Layer 3 guarantees are void. Same mitigation.
- **Side-channel attacks.** An adversary who observes timing differences in constraint masking may infer information about the grammar structure. This is a low-severity risk for most deployment contexts.
- **Model-level attacks.** Adversarial training data or fine-tuning that shifts the model's distribution toward the boundary of the constraint mask (structurally valid but semantically harmful outputs) is the primary attack vector. Layers 2 and 3 provide the defence; Layer 1 alone is insufficient.

The framework's honest security claim is: given a trusted enforcement pipeline, no structurally invalid instruction can become active regardless of the model's behaviour. Semantic safety is conditional on pattern library coverage and covenant authoring quality.

### 9.1.5  Computational Overhead

The per-token cost of constraint masking scales with the product of grammar complexity and vocabulary size. For the GAP Meta-Schema (approximately 50 production rules, 8 top-level fields (6 required, 2 optional), 5 enum constraints with 4-6 values each), the PDA state space is modest. On a vocabulary of 128 000 tokens, each decoding step requires testing each token against the current PDA state. Published measurements from llama.cpp's GBNF implementation report 10-100 μs per token depending on grammar complexity and vocabulary size [6]. For a 1 000-token GAP instruction, total overhead is 10-100 ms, which is negligible relative to LLM inference time (typically 1-30 seconds for a 1 000-token output at current hardware speeds). For schemas with deeply recursive structures (GAP's recursive types with `max_depth`), the PDA stack depth may grow to O(max_depth), increasing per-token cost linearly. These figures are order-of-magnitude estimates; production validation requires benchmarking on the specific deployment hardware and inference engine (see Section 5.5).

### 9.1.6  Roadmap for Stronger Guarantees

Three developments would materially strengthen the framework's guarantees:

1. **Formal verification of the constraint compiler.** Mechanized proof (Lean, Coq or Isabelle) that the compilation from JSON Schema 2020-12 to GBNF grammar is sound for all schemas expressible in the GAP Meta-Schema feature subset. This would upgrade Theorem 3.2 from conditional to unconditional.

2. **Continuous pattern library learning.** Integration of latent governance inference (Section 4.6) with active learning to continuously expand scanner pattern libraries from observed rejection data. This would reduce each ε_v over time, monotonically decreasing the residual defect bound.

3. **Hybrid neuro-symbolic scanners.** Replacing pure pattern-matching scanners with learned classifiers backed by formal verification (e.g. certified adversarial robustness [27]) would provide semantic predicate soundness guarantees that pure pattern matching cannot.

## 9.2  Open Problems (including Lattice Memory Enhancement)

**Memory-augmented constraint lattices.** The AEP research paper [1] deliberately omitted memory-augmented lattices from its analysis, noting in its Important Notice that "if the lattice retains memory of previously validated outputs and their structure, it can effectively raise α_T for any model feeding into it." We identify memory augmentation as the primary research direction bridging AEP's theoretical framework and GAP's practical deployment.

The key insight is that GAP's constraint lattice currently operates statelessly: each instruction evaluation is independent, with no memory of previously validated instruction structures. A memory-augmented lattice would retain embeddings of previously accepted instructions and use them to condition future constraint masking.

## Conjecture 9.1 (Memory-Augmented Acceptance Bound).

Let M_t = {o₁, ..., o_t} be the set of instructions validated through time t. Let μ_{s,M_t} denote the generative distribution conditioned on M_t (e.g. by providing previously validated instructions as in-context exemplars). Then the memory-augmented acceptance probability satisfies:

α_T^M(t) = Pr_{o ~ μ_{s,M_t}}[T(o, s) = accept] ≥ α_T

with equality when t = 0 (empty memory). If the conditioning satisfies the positive association condition of [1, Theorem 3.2], then α_T^M(t) is monotonically non-decreasing in t.

The inequality α_T^M(t) ≥ α_T is plausible but not proven. It requires that conditioning on M_t does not shift probability mass away from the acceptance set. This holds when the conditioning satisfies the positive association condition of [1, Theorem 3.2]. We identify two conditions under which positive association is plausible for GAP instructions. First, in-context exemplar conditioning: when M_t is provided as exemplars in the prompt, the model's distribution shifts toward token sequences that are distributionally similar to the exemplars. Because the exemplars are all accepted (they passed the lattice), the shifted distribution concentrates mass on structurally similar instructions, which are more likely to pass the same lattice predicates. Second, structural homogeneity of the acceptance set: the GAP Meta-Schema defines a structurally homogeneous acceptance region in the sense that documents sharing the same structural skeleton (same field set, same action type) but differing in permitted value ranges (e.g., weight values, covenant lists) tend to remain valid. (Note: "convexity" is used here informally; JSON documents do not naturally form a vector space with a standard convexity notion.) This structural property makes it unlikely that conditioning on accepted outputs shifts mass toward rejected regions. Neither condition constitutes a proof; establishing positive association rigorously for autoregressive language models conditioned on exemplar sets remains an open problem.

The critical advantage of memory-augmented lattices over in-context exemplar conditioning is robustness: memory does not suffer from context window saturation or catastrophic interference, the failure modes identified in [1, Observation 3.2].

For GAP specifically, memory augmentation would interact with the self-generation model (Section 4.3) in a productive feedback loop. As the lattice accumulates validated instruction patterns, the constraint mask could be tightened beyond the static Meta-Schema: patterns that have historically led to lattice rejection (e.g. covenant violations, trust escalations) could be excluded from the token mask proactively. This would create a system where the lattice actively improves generation quality rather than merely filtering it, closing the gap between the static constraint mask (which permits all structurally valid instructions) and the dynamic governance requirements (which permit only a subset of structurally valid instructions).

The population size required for zero-defect selection ([1, Theorem 3.3]) is N = ⌈ln(δ)/ln(1 - α_T)⌉. As α_T^M(t) increases through memory augmentation, the required N decreases. In the limit, if α_T^M(t) → 1, then N → 1: a single generation attempt suffices with high probability. This convergence point represents the state where the lattice's accumulated memory has effectively trained the constraint mask to produce governance-compliant instructions with near-unit probability, without modifying the underlying model's parameters.

**Formal verification of the constraint compiler.** The No-Bypass Theorem assumes sound compilation. Verifying that the compilation from JSON Schema to GBNF grammar is sound for all schemas in the GAP Meta-Schema is an open problem amenable to mechanized proof (e.g. in Coq, Isabelle or Lean).

**Code intermediate representation.** GAP instructions can serve as a structured intermediate representation for source code where types, constraints, invariants and governance rules become explicit first-class declarations. LLM-driven code operations (bug fixing, migration, feature implementation) can operate on the GAP-IR with tiered context loading (from full instruction to minimal problem view to numerical coordinate representation). Stateful runtimes satisfying R8 (convention memory) could further reduce per-operation cost by crystallizing validated fix patterns as reusable templates. The formal treatment of GAP as code IR and the cost properties of tiered context loading are outside the scope of this paper.

**Continuous verification predicates.** The 15-step lattice uses binary predicates (accept/reject). Extending the framework to continuous predicates that return confidence scores in [0, 1] would enable softer selection mechanisms. The composition theory of continuous predicates has not been developed.

## 9.3  Hardware Acceleration Potential

The computational cost of constraint masking is dominated by the per-token grammar check. For the GAP Meta-Schema, this check requires traversing the PDA transition function for each token in the vocabulary (up to 128 000 tokens). The check is embarrassingly parallel: each token can be tested independently.

Custom hardware (FPGA or ASIC implementations of the PDA transition function) could reduce per-token latency from microseconds to nanoseconds. For high-frequency domains using the binary compilation target (.gapb), the constraint opcodes (27 fixed-size instructions, 8 bytes each) are designed for direct function-pointer dispatch with minimal heap allocation, targeting sub-microsecond validation on general-purpose hardware.

The photonic acceleration potential identified in [1, Section 9.2] applies to the generative side of the pipeline (producing candidate outputs). The constraint masking side (validating tokens against the grammar) is Boolean logic rather than linear algebra, making it better suited to electronic ASIC acceleration. A hybrid architecture with photonic generation and electronic masking represents the natural hardware co-design target.

## 10  Conclusion

GAP establishes a language-level foundation for governed agentic programming. Its primary guarantee is structural: under sound constrained decoding, malformed instructions are not sampled, rejected or repaired after the fact; they are absent from the token selection space. The No-Bypass Theorem (Theorem 3.2) isolates three sufficient engineering invariants (prefix soundness, termination soundness and viable non-empty continuation) whose joint satisfaction guarantees that every terminated output of constrained decoding is a valid GAP document. Proposition 3.2 establishes corresponding necessity results: prefix soundness and termination soundness are necessary for zero invalid terminated output, and viable non-empty continuation is necessary for deadlock freedom. Token-level constraint masking is mature engineering practice; GAP's contribution is the governance framework built on top of it.

GAP's instruction-as-atomic-unit model provides a uniform representation for agents, workflows, validators and governance rules. Six composition operators have formal operational semantics with provable type soundness, preservation and progress. Self-generating instructions are constrained by the same Meta-Schema, establishing a recursive governance fixed point where the governance preorder can only strengthen across generations (Theorem 4.2). The v1.1 extensions (LRT-gated self-generation, MLE weight estimation and latent governance inference) add statistical rigour to the governance pipeline without altering its structural invariants.

GAP deliberately separates language guarantees from runtime guarantees. The language proves structural validity, governance-field completeness, governance-path preservation through composition and self-generation containment. Semantic safety, policy compliance and operational authorisation are properties of the deterministic governance runtime that evaluates GAP artifacts. The abstract Governance Runtime Interface (Section 6.7) defines the minimal contract (totality, determinism, hard-predicate soundness, monotonic child governance and evidence production) that any adjudication implementation must satisfy. This separation allows GAP to serve as a stable instruction format while permitting independent runtimes to evolve their adjudication, evidence and optimisation mechanisms behind the same abstract interface.

The 15-step adjudication lattice is one concrete instantiation of this interface, satisfying the structural requirements of the abstract Deterministic Adjudication Lattice framework of [1] for structural predicates (Observation 6.1). For semantic predicates (intent drift detection, content scanning), soundness is relative to pattern library coverage. The structural-semantic gap (Section 3.5) is the honest boundary of the framework's formal guarantees: Layer 1 eliminates structural defects by construction; Layers 2 and 3 mitigate semantic defects to the extent of their pattern coverage; neither layer claims semantic omniscience.

The result is a governed programming model in which agent outputs are structurally valid by construction and semantically executable only by explicit deterministic acceptance.

## Appendix A  Proofs of All Theorems

## A.1  Expanded Proof of Theorem 3.2 (No-Bypass Theorem)

The proof in Section 3.2 establishes the result by induction on sequence length under three explicit invariants: prefix soundness, termination soundness and viable non-empty continuation. We provide additional detail on the termination and liveness conditions.

Under a constraint function satisfying prefix soundness, C_S is derived from a pushdown automaton (PDA) that tracks the parse state of the GAP Meta-Schema grammar. The PDA maintains a stack of non-terminal symbols representing the expected structure of the remaining document. At each step, the PDA transitions based on the current input token and the top of the stack.

**Termination soundness (invariant 2).** The end-of-sequence condition corresponds to the PDA reaching an accepting state with an empty stack. Formally, let (q, γ) be the PDA state (control state q and stack γ) after processing prefix w. The end-of-sequence token EOS is in C_S(w) if and only if (q, γ) is an accepting configuration (q ∈ F and γ = ε, where F is the set of final states). By the correctness of the PDA construction from the CFG, (q, ε) ∈ F if and only if the prefix w is a complete valid document. The biconditional (if and only if) is essential: EOS must be permitted at accepting states (to allow termination) and forbidden at non-accepting states (to prevent premature termination). A constraint function that allows premature EOS violates termination soundness and can produce structurally incomplete documents.

**Viable non-empty continuation (invariant 3).** At every viable but incomplete prefix w (where w is extendable to a valid document but decode(w) ∉ L(S)), the constraint function must provide at least one non-EOS token. If C_S(w) = ∅ at a viable prefix, constrained decoding is trapped: it can neither continue nor terminate validly. The PDA construction prevents this for well-formed grammars because every viable PDA state (q, γ) with γ ≠ ε admits at least one valid transition.

Therefore, constrained decoding can terminate only when the accumulated output is a complete valid GAP document, and it is never trapped at a viable intermediate state.

□

## A.2  Proof of Proposition 2.1 (GAP Meta-Schema Defines a Context-Free Language)

This appendix expands the proof of Proposition 2.1 with additional detail on the restrictions stated in the proposition.

JSON syntax is defined by a BNF grammar (RFC 8259 [8]) and is therefore context-free. We show that each JSON Schema constraint used by the GAP Meta-Schema preserves context-freeness under the stated restrictions.

**Duplicate keys.** GAP canonical JSON forbids duplicate object member names before schema validation. Because `additionalProperties: false` and a finite property list define the exact set of allowed keys at each level, the grammar generates only distinct-key objects by construction.

**Object member order.** JSON objects are semantically unordered. The grammar must generate all valid orderings of object members. For a finite property set of size k (with `additionalProperties: false`), there are at most k! orderings, each expressible as an alternative production. This is finite and does not affect context-freeness.

**Type constraints** (object, array, string, number, boolean, null) partition the JSON language into sublanguages, each of which is context-free. The constraint restricts the document to one sublanguage, preserving context-freeness.

**Enumeration constraints** restrict a string or value to a finite set of alternatives. A finite set is a regular language. The intersection of a CFL with a regular language is a CFL [9, Theorem 4.8].

**Pattern constraints** (regex patterns on strings) restrict string values to a regular language. Same argument as enumeration.

**Required-property constraints** impose that certain keys must appear in an object. This is expressible as a CFG production with mandatory non-terminals.

**additionalProperties: false** prohibits unlisted keys. Combined with the property list, this defines a fixed set of allowed keys, each of which is a terminal in the grammar.

**Conditional constraints** (if/then/else) introduce context-sensitive dependencies in general. In JSON Schema 2020-12, the condition is evaluated on a sub-document and the then/else clause constrains a (potentially different) sub-document. For the GAP Meta-Schema, conditional constraints are local: the condition and the constrained clause reference the same object (e.g. `action.type = "code"` implies `action.language` is required). These local conditionals can be encoded as alternative productions in the CFG, one alternative per condition value.

**Recursive `$ref`** introduces recursive productions, which are the defining feature of CFGs. The recursion in the GAP Meta-Schema is structurally regular and does not impose cross-branch equality or uniqueness constraints.

□

## A.3  Proof of Theorem 4.1 (Composition Preserves Governance)

For each composition operator, we verify that the eval function (which includes the lattice check) is applied to every constituent instruction.

**Sequence.** seq(I₁, ..., I_n)(x) = eval(I_n, ... eval(I₁, x) ...). Each eval(I_j, ·) includes the lattice check as its second phase. If any lattice check produces DENY, evaluation halts and returns REJECT.

**Parallel.** par(I₁, ..., I_n, m)(x) = m({eval(I_j, x)}). Each eval is independent and includes the lattice check.

**Conditional.** cond(c, I_t, I_e)(x) evaluates one branch. The selected branch's eval includes the lattice check.

**Loop.** loop(I, p, k)(x) iterates eval(I, ·) up to k times. Each iteration includes the lattice check.

**Abstraction.** abs(I₁, ..., I_n, s)(x) selects one variant and evaluates it. The selected variant's eval includes the lattice check.

Additionally, the composition itself is an instruction and undergoes Layer 2 validation by gapc, which checks:
- No composition cycles (detected by topological sort of the instruction dependency graph)
- Compatible trust floors across sequence steps (no step requires a higher ring than the composition declares)
- Bounded iteration (loop composition requires max_iterations)

□

## Appendix B  The GAP Meta-Schema as a Formal Grammar

We construct a CFG for a representative fragment of the GAP Meta-Schema.

## B.1  Top-Level Structure

The grammar targets canonical JSON serialization (Section 2.3). All productions generate JSON syntax per RFC 8259.

```
GAP_DOC     → "{" ADDRESS "," PATTERN "," ACTION "," WEIGHT "," COMPOSITION "," METADATA OPT_FIELDS "}"
ADDRESS     → QS_address ":" "{" QS_domain ":" DOMAIN_VAL "," QS_id ":" IDENT_VAL "}"
DOMAIN_VAL  → '"' LC LCNUM_SEQ ("." LC LCNUM_SEQ)+ '"'
IDENT_VAL   → '"' LC LCNUM_DASH_SEQ ("." LC LCNUM_DASH_SEQ)* OPT_VERSION '"'
OPT_VERSION → ε | ".v" DIGITS
```

where QS_x denotes the quoted JSON string `"x"`, LC is a lowercase letter, LCNUM_SEQ is a sequence of lowercase letters and digits and LCNUM_DASH_SEQ includes hyphens. Object member order in the grammar is fixed (deterministic key ordering); the finite property set of the GAP Meta-Schema makes this a finite set of permutation alternatives, but we present the canonical ordering for clarity.

## B.2  Action Type (Conditional Productions)

```
ACTION      → QS_action ":" "{" QS_type ":" ACTION_TYPE "," ACTION_BODY "}"
ACTION_TYPE → '"template"' | '"structured"' | '"code"' | '"reference"' | '"pipeline"'
ACTION_BODY → TEMPLATE_BODY | STRUCTURED_BODY | CODE_BODY | REF_BODY | PIPE_BODY
```

The conditional validation in the Meta-Schema (`if action.type = "code" then required: [language, content]`) is encoded as alternative productions: when ACTION_TYPE derives `"code"`, ACTION_BODY must derive CODE_BODY, which requires both `"language"` and `"content"` fields.

## B.3  Weight (Numeric Range)

```
WEIGHT      → QS_weight ":" FLOAT_01
FLOAT_01    → "0" OPT_DECIMAL | "1" OPT_ZERO_DECIMAL
OPT_DECIMAL → ε | "." DIGITS
OPT_ZERO_DECIMAL → ε | "." ZEROS
```

This grammar ensures that weight is a JSON number in [0.0, 1.0]. The value 1.5 is not derivable; the token "1" followed by ".5" is blocked because FLOAT_01 in the "1" branch requires OPT_ZERO_DECIMAL (only zeros after the decimal point of 1).

Note. In practice, the numeric range constraint is enforced more efficiently by the PDA's state machine than by expanding all valid digit sequences in the grammar. The presentation above illustrates the principle.

## Appendix C  Category-Theoretic Perspective on Instruction Composition (Informal)

**Note.** This appendix offers an informal analogy between GAP's compositional structure and standard category-theoretic constructions. The discussion is intended to suggest connections and motivate future formalisation; it does not constitute a rigorous categorical development. In particular, the presheaf and sheaf claims below are sketched at the level of intuition rather than proved from precise categories, topologies and functors. Readers seeking formal results should rely on the proofs in Sections 3-7 and Appendix A.

## C.1  The Category Gap

Define the category **Gap** whose objects are GAP instructions and whose morphisms are composition relationships. A morphism f: I₁ → I₂ exists if I₁ is a component of I₂ (I₂ references I₁ in its composition block).

**Gap** is a finite category (finitely many instructions in any deployment). The composition operators define functors:

- **Seq**: **Gap**^n → **Gap** maps a tuple of instructions to their sequential composition
- **Par**: **Gap**^n × Merge → **Gap** maps a tuple and merge strategy to their parallel composition
- **Cond**: **Gap** × **Gap** × Pred → **Gap** maps two branches and a predicate to a conditional composition

## C.2  Governance as a Functor

The governance assignment G: **Gap** → **Gov** is a functor from the instruction category to the category of governance structures. **Gov** is a partially ordered category whose objects are governance configurations (trust ring, covenants, scanners, budget) and whose morphisms are governance-strengthening relationships (G₁ → G₂ if G₂ is at least as restrictive as G₁).

The self-governance theorem (Theorem 4.2) states that the self-generation map gen: **Gap** → **Gap** satisfies G ∘ gen ≥ G (governance is monotonically non-decreasing under self-generation). In categorical terms, the functor G is lax monoidal with respect to the generation endofunctor.

## C.3  The Constraint Mask as a Presheaf

The constraint function C_S: V* → 2^V can be viewed informally as a presheaf-like construction on the category **Pref** of prefixes (objects: elements of V*; morphisms w → w·t for each token t ∈ V, representing prefix extension). The assignment F: **Pref**^op → **Set** maps each prefix w to the set C_S(w) of valid continuations. Note that the analogy with contravariant functors is imperfect: extending a prefix does not in general yield a subset relationship on next-token sets. After producing `{`, the valid next tokens may be field-name starts; after producing `"address"`, the valid next token set may become `:`. These sets are not subsets of each other in the obvious sense. The relationship is better described at the level of *reachable PDA states*: each prefix maps to a PDA state, and the set of valid continuations is a function of that state. The No-Bypass Theorem states that the prefixes for which EOS is a valid continuation (i.e., the empty string is a valid completion) are exactly the complete documents in L(S).

**Computational consequence.** Informally, quotienting prefixes by reachable PDA state gives the same optimisation one might describe categorically as factoring the prefix-indexed continuation assignment through the PDA-state space. In practice this means: instead of recomputing C_S(w) from scratch at each step by re-scanning all |V| tokens, we compute the PDA state transition δ(q, t) → q' and derive C_S(w·t) from C_S(w) and the transition. This converts an O(|V| · |G|) per-step computation into an O(|δ|) state transition followed by an O(|V|) mask scan from the cached state. This is the standard optimisation used by llama.cpp and Outlines. We do not rely on this categorical interpretation for any theorem.

**State-determinism (informal gluing analogy).** The key property that makes constraint masking efficient is that the PDA state after processing prefix w determines C_S(w) uniquely. Two prefixes reaching the same PDA state have identical constraint sets, regardless of how they arrived at that state. This state-determinism means that constraint information is "local" in the sense that it depends only on the current parse state, not on the full prefix history. Informally, this is analogous to a gluing condition: the constraint set at w is determined by the transition structure at its extensions, and no inconsistency arises because the PDA is deterministic. For context-sensitive languages, the analogous construction would require an automaton whose state depends on the full prefix, breaking this locality property. This observation motivates the requirement that L(S) be context-free but is offered as an analogy rather than a formal sheaf-theoretic result.

## Appendix D  Illustrative Analytical Scenarios

**Note.** This appendix presents computed examples derived from the analytical model of Section 6.6. No LLM was run; no empirical data was collected. All values (acceptance rates, base defect rates, scanner coverage figures) are *assumptions*, not measurements. The purpose is to illustrate the quantitative relationships between pattern library coverage, layer composition and population size requirements under these assumed parameters. These scenarios demonstrate the framework's analytical properties, not production performance.

## D.1  Scenario Setup

We model instruction generation as a Bernoulli process with the following *assumed* parameters (not empirical measurements):

- **Structural acceptance rate (Layer 1).** Under Tier 1 (logits-level constraint masking), the structural acceptance rate is 1.0 by construction (Theorem 3.2). Under unconstrained generation (no Layer 1), we estimate the structural acceptance rate at 0.67, based on published measurements of raw LLM JSON Schema conformance rates for complex schemas [11].

- **Semantic acceptance rate (Layers 2 + 3).** We decompose the semantic acceptance rate into per-predicate pass rates. For structural predicates (Steps 1-4, 7-9), the pass rate is 1.0 by construction for well-configured agents (correct ring, valid session, budget within limits). For semantic predicates, we model pass rates as functions of pattern library coverage c:
  - Injection scanner (Step 14, hard): pass rate = 1 - (1 - c_inj) × p_inj, where p_inj is the probability that a random instruction contains an injectable pattern and c_inj is the scanner's coverage of known injection patterns.
  - Intent drift (Step 5, active): pass rate = 1 - (1 - c_drift) × p_drift.
  - Covenant evaluation (Step 7, always): pass rate depends on covenant specificity. We model this as a function of the number of covenants and their false-positive rate.

- **Population size for zero-defect selection.** From [1, Theorem 3.3]: N = ⌈ln(1/δ) / ln(1/(1 - α_T))⌉, where α_T is the joint acceptance probability and δ is the target failure probability (set to 0.001 throughout).

## D.2  Computed Examples

**Table D.1. Acceptance rates and population sizes across configurations.** All values are computed from the assumed parameters above, not measured empirically. δ = 0.001 (99.9% confidence). Semantic acceptance assumes well-configured agents (correct ring, valid session, budget within limits) and models scanner coverage at 92% for injection, 85% for intent drift and 95% for covenant coverage. p_inj = 0.05, p_drift = 0.08 (probability of encountering an injectable or drifting instruction in a typical workload).

| Configuration | Structural Acceptance | Semantic Acceptance | Joint α_T | Population Size N (δ = 0.001) | Notes |
|--------------|----------------------|--------------------|-----------|-----------------------------|-------|
| Raw generation baseline | 0.67 | 0.41 | 0.27 | 22 | Unconstrained generation with post-hoc selection model |
| Layer 1 only (constraint mask) | 1.00 | 0.41 | 0.41 | 14 | Structural defects eliminated |
| Layers 1 + 2 (mask + gapc) | 1.00 | 0.72 | 0.72 | 6 | Static semantic checks added |
| Full 3-layer pipeline | 1.00 | 0.89 | 0.89 | 4 | Current GAP (all 15 steps) |
| + Memory augmentation (projected) | 1.00 | 0.96 | 0.96 | 3 | Conjecture 9.1 with t = 100 |
| + Full v1.1 statistical layer | 1.00 | 0.975 | 0.975 | 2 | LRT + MLE + latent inference |

**Interpretation.** Layer 1 alone reduces the population size from 22 to 14 by eliminating all structural defects. Adding Layer 2 reduces N to 6 by catching static semantic violations (composition cycles, trust floor mismatches, tool permission errors). The full 15-step lattice brings N to 4. Memory augmentation (Conjecture 9.1) and the v1.1 statistical extensions project N to 2-3, approaching the theoretical minimum of N = 1 (single-attempt generation suffices).

## D.3  Sensitivity Analysis: Pattern Library Coverage

**Table D.2. Effect of pattern library coverage on residual defect probability.** Full 3-layer pipeline; δ = 0.001. Only the injection scanner coverage is varied; all other parameters held constant.

| Injection Scanner Coverage c_inj | False-Negative Rate ε_inj | Residual Semantic Defect Bound | Population Size N |
|----------------------------------|--------------------------|-------------------------------|-------------------|
| 0.70 | 0.015 | 0.015 | 6 |
| 0.80 | 0.010 | 0.010 | 5 |
| 0.90 | 0.005 | 0.005 | 5 |
| 0.95 | 0.0025 | 0.0025 | 5 |
| 0.99 | 0.0005 | 0.0005 | 5 |
| 1.00 | 0.0000 | 0.0000 | 5 |

**Interpretation.** The residual defect bound decreases linearly with scanner coverage improvement. Even at 70% coverage, the residual bound is 1.5%, which is already below the typical false-positive rate of post-hoc classifiers [2, 3]. At 95% coverage, the residual bound is 0.25%. Full coverage (1.00) achieves the zero-defect guarantee for that predicate class.

## D.4  Comparison to Post-Hoc Filtering Baselines

**Table D.3. GAP vs. cascaded post-hoc filtering.** Post-hoc filtering assumes k independent classifiers, each with false-negative rate ε = 0.05. GAP uses full 3-layer pipeline. δ = 0.001.

| System | Mechanism | Residual Defect Probability | Population Size N |
|--------|-----------|----------------------------|-------------------|
| Single classifier (k=1) | Post-hoc | 0.050 | N/A (probabilistic) |
| Cascaded classifiers (k=3) | Post-hoc | 0.000125 | N/A (probabilistic) |
| Cascaded classifiers (k=5) | Post-hoc | 3.1 × 10⁻⁷ | N/A (probabilistic) |
| GAP (Tier 1, full pipeline) | Construction + lattice | 0 (structural) + Σ p_d·ε_d (semantic) | 5 |
| GAP (Tier 2, full pipeline) | Validation + lattice | 0 (gate blocks invalid) + Σ p_d·ε_d (semantic) | 5 + retry cost |

**Interpretation.** Cascaded classifiers reduce defect probability geometrically but never reach zero and assume independence between classifiers (which [1] argues is unrealistic for classifiers checking related properties). GAP achieves zero structural defect by construction. Semantic defect rates are bounded by Σ p_d·ε_d (union bound over defect classes, Section 6.6) and depend on pattern coverage rather than classifier accuracy. The two approaches are complementary: GAP could incorporate learned classifiers as additional lattice predicates.

## D.5  Reproducibility

All values in Tables D.1-D.3 are derived from closed-form expressions in Section 6.6 and [1, Theorem 3.3]. The population size formula is N = ⌈ln(1/δ) / ln(1/(1 - α_T))⌉. The semantic acceptance rates are computed as ∏_{v ∈ active} (1 - ε_v × p_v), where ε_v is the false-negative rate of predicate v and p_v is the base rate of the defect class in the instruction population. This product computes the joint per-predicate pass rate (the probability that a candidate instruction passes all active predicates); the residual defect probability is bounded separately by the union bound Σ p_d · ε_d as described in Section 6.6. No stochastic simulation was performed. A researcher can reproduce all population size values with the following script and the assumed parameters listed above:

```python
import math

def population_size(alpha_T, delta=0.001):
    if alpha_T >= 1.0:
        return 1
    return math.ceil(math.log(1.0 / delta) / math.log(1.0 / (1.0 - alpha_T)))

# Table D.1 population sizes (delta = 0.001)
for label, alpha in [("Raw baseline", 0.27), ("Layer 1 only", 0.41),
                      ("Layers 1+2", 0.72), ("Full pipeline", 0.89),
                      ("+ Memory", 0.96), ("+ Full v1.1", 0.975)]:
    print(f"{label}: alpha_T={alpha}, N={population_size(alpha)}")
```

The illustrative nature of these scenarios is emphasised: they demonstrate the framework's analytical properties under assumed parameters, not production performance. All acceptance rates, base defect rates and scanner coverage figures are assumptions. Empirical validation on real LLM-generated GAP instructions is the most pressing direction for future work.

## References

[1] the.PM. Deterministic Adjudication Lattices and Evolutionary Convergence in Constrained Generative Agent Populations. Companion technical report, Iberian Peninsula Human Civilization Continuation Project / New Lisbon Agency, April 2026.

[2] A. Rebedea, C. Dinu, N. Sreedhar, et al. NeMo Guardrails: A toolkit for controllable and safe LLM applications. In EMNLP (Demo), 2023.

[3] I. Inan, M. Upasani, J. Chi, et al. Llama Guard: LLM-based input-output safeguard for human-AI conversations. arXiv:2312.06674, 2023.

[4] P. Christiano, J. Leike, T. Brown, et al. Deep reinforcement learning from human preferences. In NeurIPS, 2017.

[5] L. Ouyang, J. Wu, X. Jiang, et al. Training language models to follow instructions with human feedback. In NeurIPS, 2022.

[6] G. Gerganov et al. llama.cpp: LLM inference in C/C++. GitHub repository, 2023-2026. GBNF grammar support for constrained decoding.

[7] A. Wright, H. Andrews, B. Hutton and G. Dennis. JSON Schema: A media type for describing JSON documents. Internet-Draft, 2020-12 specification, 2020.

[8] T. Bray. The JavaScript Object Notation (JSON) Data Interchange Format. RFC 8259, IETF, 2017.

[9] J. E. Hopcroft, R. Motwani and J. D. Ullman. Introduction to Automata Theory, Languages and Computation. Third edition. Addison-Wesley, 2006.

[10] the.PM. GAP: Governed Agentic Programming. Language Specification v1.1. Companion specification document, Iberian Peninsula Human Civilization Continuation Project / New Lisbon Agency, May 2026.

[11] B. T. Willard and R. Louf. Efficient guided generation for large language models. arXiv:2307.09702, 2023.

[12] Y. Bai, S. Kadavath, S. Kundu, et al. Constitutional AI: Harmlessness from AI feedback. arXiv:2212.08073, 2022.

[13] R. Hindley. The principal type-scheme of an object in combinatory logic. Transactions of the American Mathematical Society, 146:29-60, 1969.

[14] J.-Y. Girard. Interpretation fonctionnelle et elimination des coupures de l'arithmetique d'ordre superieur. These d'Etat, Universite Paris VII, 1972.

[15] P. Martin-Lof. Intuitionistic type theory. Bibliopolis, 1984.

[16] E. M. Clarke, O. Grumberg and D. A. Peled. Model Checking. MIT Press, 1999.

[17] A. Pnueli. The temporal logic of programs. In FOCS, 1977.

[18] L. Lamport. Proving the correctness of multiprocess programs. IEEE Transactions on Software Engineering, 3(2), 1977.

[19] P. Cousot and R. Cousot. Abstract interpretation: A unified lattice model for static analysis of programs by construction or approximation of fixpoints. In POPL, 1977.

[20] A. Tarski. A lattice-theoretical fixpoint theorem and its applications. Pacific Journal of Mathematics, 5(2):285-309, 1955.

[21] G. D. Plotkin. A structural approach to operational semantics. Technical Report DAIMI FN-19, Aarhus University, 1981.

[22] S. Yao, J. Zhao, D. Yu, et al. ReAct: Synergizing reasoning and acting in language models. In ICLR, 2023.

[23] N. Scholak, R. Schucher and D. Baez. PICARD: Parsing incrementally for constrained auto-regressive decoding from language models. In EMNLP, 2021.

[24] C. Geng, C. Liu, S. Chen, et al. Grammar-aligned decoding. arXiv:2305.13971, 2023.

[25] T. Anderson and R. Perlis. Open Policy Agent: Policy-based control for cloud native environments. CNCF, 2018.

[26] N. Hansen and A. Ostermeier. Completely derandomized self-adaptation in evolution strategies. Evolutionary Computation, 9(2), 2001.

[27] G. Katz, C. Barrett, D. L. Dill, et al. Reluplex: An efficient SMT solver for verifying deep neural networks. In CAV, 2017.

[28] L. C. Paulson. Isabelle: A Generic Theorem Prover. Springer, 1994.

[29] R. Milner. Communicating and Mobile Systems: The Pi-Calculus. Cambridge University Press, 1999.

[30] M. Y. Vardi. An automata-theoretic approach to linear temporal logic. In Banff Higher Order Workshop, 1996.

## Data Availability Statement

This paper is a theoretical contribution. All results are analytical (proofs of theorems). The synthetic simulation data in Appendix D is derived from closed-form expressions and is fully reproducible from the parameters listed in Section D.5. The GAP Meta-Schema (JSON Schema 2020-12) is available as a machine-readable artifact (`GAP-meta-schema-v1.1.json`) in the GAP repository. The GAP v1.1 specification is available as a companion document. No external datasets or LLM experiments were used.

## Conflicts of Interest

The author declares no conflicts of interest.

## Use of AI-Assisted Tools

Portions of this manuscript were drafted with the assistance of Claude Opus 4.6 (Anthropic). This tool was used for cognitive simplification, prose generation, mathematical exposition and document formatting. All theoretical content, definitions, theorem statements, proof strategies and architectural decisions are the sole intellectual contribution of the author. The author has reviewed the full text for correctness, verified all mathematical claims independently and takes full responsibility for the content of this paper. This disclosure is made in accordance with the COPE position statement on AI-assisted authorship tools.
