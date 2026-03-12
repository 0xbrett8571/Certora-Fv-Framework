> **Purpose:** 
> This defines the **mandatory process** for authoring **execution-closed** Certora specifications.
> Any specification written without a **closed execution surface** is **invalid by construction**, regardless of property quality.

> **Core Principle:** 
> **Execution reality must be closed before properties are even named.**
> No invariant, rule, ghost, dispatcher, or assumption may be written before **all reachable EVM interactions are enumerated and owned**.

---

## NON-NEGOTIABLE AXIOMS

> **AXIOM 1 — EXECUTION CLOSURE**
>
> *Every external call, external storage read, or external view function that can influence control-flow, state, or accounting MUST be explicitly modeled or explicitly trusted.*
>
> *Unmodeled interactions are adversarial by definition and invalidate the specification.*

> **AXIOM 2 — OWNERSHIP OF TRUTH**
>
> *Every value used in a property must have a single on-chain owner.*
>
> *If the owner is not the contract under verification, that owner must be modeled.*

> **AXIOM 3 — HAVOC IS A SIGNAL**
> 
> Any counterexample relying on HAVOC indicates missing execution modeling, not a protocol bug.
> 
---
## PHASE 0 — CONTRACT REALITY PRE-FLIGHT 

> **Goal:** Extract *execution reality* — not intent — from the contract.

🚨 **Hard Stop:**
If execution reality is incomplete → **Reject the spec**.

---

## 0.1 — VERIFICATION BOUNDARY IDENTIFICATION 

☐ Identify **primary contract(s)**
☐ List **in-scope contracts**
☐ List **out-of-scope contracts**

**RULE:**
Anything out-of-scope **must be modeled**.
Out-of-scope ≠ out-of-execution.

---

## 0.2 — ENTRY POINT ENUMERATION

> **View functions ARE entry points if they influence control-flow or accounting.**

Include:

* `view` pricing
* `view` permission checks
* `view` balance checks

If a view function is used in a `require`, it is **security-critical**.

---

## 0.3 — ENTRY POINT CLASSIFICATION 

### ☐ External Read Entry Point 

* Reads external storage
* Influences require/assert/branching

📌 **Rule:**
External reads are as dangerous as external writes.

---

## 0.4 — STATE MUTATION MAP (WRITE GRAPH) 

Add:

| Function | Writes | Reads | External Writes | External Reads |
| -------- | ------ | ----- | --------------- | -------------- |

🚨 **READ/WRITE PAIRING**

> Every external read must have a corresponding modeled write or be provably immutable.

---

## 0.5 — ASSET FLOW TRACE 

For each asset:

☐ Identify **owning contract**
☐ Identify **mutation functions**
☐ Identify **read-only checks**
☐ Identify **reentrancy window**

🚫 If asset owner ≠ this contract → must be modeled.

---

## 0.6 — TRUST BOUNDARY IDENTIFICATION 

**NEW DISTINCTION**

| Category            | Meaning                     |
| ------------------- | --------------------------- |
| Code-Enforced       | Solidity restricts behavior |
| Modeled-Adversarial | Modeled but hostile         |
| Threat-Model        | Assumed honest              |

🚨 Properties may **not** silently depend on Threat-Model actors.

---

## 0.7 — FLOW COLLISION ENUMERATION 

> If two flows can interleave on-chain, Certora must consider the worst ordering.

If not modeled → spec invalid.

---

# PHASE 0 HARD STOP RULE 

> No property discovery may begin until:
>
> * execution universe is closed
> * ownership of truth is assigned
> * modeling obligations are listed
---
## PHASE −1 — EXECUTION CLOSURE PRE-FLIGHT (MANDATORY)
> Each objective must list the **external truths it depends on**.

> **Hard Rule:**
> If Phase −1 is incomplete → **Abort spec authoring.**

### −1.1 — EXECUTION UNIVERSE ENUMERATION

List **all contracts that may participate in execution**, directly or indirectly:

* Direct call targets
* Delegatecall targets
* Callback targets
* Upgrade controllers
* Token contracts
* Oracles
* Registries
* Routers / hubs

📌 **Rule:**
If a contract appears in the call graph **even once**, it is part of the execution universe.

---

### −1.2 — INTERACTION OWNERSHIP TABLE 

For each external contract:

| Contract | Owns Which Truth? | Reads | Writes | Callbacks |
| -------- | ----------------- | ----- | ------ | --------- |

Examples:

* ERC20 → balances, allowances, supply
* Oracle → price
* PositionManager → ownership
* Registry → permissions

🚫 If ownership is unclear → spec must stop here.

---

### −1.3 — MODELING OBLIGATION 

For **each owned truth**:

| Truth     | Owner         | Modeling Required      |
| --------- | ------------- | ---------------------- |
| balances  | ERC20         | Dispatcher             |
| ownership | NFT / manager | Dispatcher             |
| price     | Oracle        | Adversarial or trusted |
| flags     | Registry      | Constant or dispatcher |

🚨 Leaving any row blank invalidates the spec.
---

## PHASE 2 — PROPERTY DISCOVERY/CATEGORIZATION (NO CVL YET)

> No property may reference:
>
> * balances
> * ownership
> * prices
> * flags
>   unless their owner is identified and modeled.

List candidate security properties **without choosing CVL syntax**.

> ❌ Do **not** write `invariant`, `rule`, `require`, or `assume` yet.

🚨 Develop a comprehensive list of properties in plain English, then categorize them using **`categorizing-properties.md`**

**NEW v1.3 Enhancements:**
- **Prioritization:** Assign HIGH/MEDIUM/LOW priority (Section 7)
- **Dual Mindset:** "Should Always" + "Should Never" enumeration (Section 5)
- **Test Mining:** Extract properties from existing tests (Section 6)
- **Best Practices:** See `best-practices-from-certora.md` Section 1 for property discovery techniques and common pitfalls
> These categories are NOT a commitment to invariant vs rule.
Right now, that rule is implicit, not explicit.
> Categorization at this stage is semantic only and **MUST NOT** be used to infer invariant vs rule.
> These categories describe what kind of harm occurs, not how the property will be encoded.

---
## HOW TO THINK ABOUT CATEGORIZATION (MENTAL MODEL)

| Phase        | Question Being Answered                        |
| ------------ | ---------------------------------------------- |
| Phase 0 / −1 | *What is real?*                                |
| Phase 2      | *What would be catastrophic if violated?*      |
| Step 2.5     | *How must it be enforced (invariant vs rule)?* |
| Phase 3      | *How do we encode it?*                         |

---

### A. STATE-INDUCTIVE SAFETY PROPERTIES

*Must hold in every reachable state*

* Solvency
* Supply caps
* Conservation laws
* Mutually exclusive modes

---

### B. STATE TRANSITION PROPERTIES

*Depend on specific calls*

* Access control
* Monotonicity
* Post-conditions of a function

---

### C. GLOBAL / CROSS-USER PROPERTIES

*Require aggregation or history*

* Global accounting
* Reward distribution correctness
* Cross-user conservation

➡️ Often requires **ghosts**

---

### D. ATTACK SURFACE INVERSION

*Adversarial reasoning*

* Theft
* Griefing
* Unexpected call sequences

> Phase 2 produces candidate laws, not enforceable properties.
> No item in this phase is guaranteed to survive Step 2.5.
---

## STEP 2.5: INVARIANT vs RULE DECISION GATE (MANDATORY)

For **each candidate property**, answer **in order**:

0️⃣ **Is all state referenced by this property owned or modeled?**

> ❌ No → **STOP** (model first)
> ✔ Yes → continue

1️⃣ **Is the question:**

> *“Can this EVER happen?”*
> ✔ Yes → **INVARIANT**
> ❌ No → continue

2️⃣ **Does it depend on a specific function call, caller, or sequence?**
> ✔ Yes → **RULE**
> ❌ No → continue

2️⃣.5 **Does it depend on external state not fully enforced by this contract**

(ERC20 balances, oracle prices, AMM reserves, lending protocol state)?

> ✔ Yes → **NOT an invariant**
> → **RULE or THREAT MODEL**
> ❌ No → continue

> 📌 Even if the external system is “standard”, “battle-tested”, or “widely used”.

3️⃣ **Would a single violation leave the protocol irrecoverably broken?**
> ✔ Yes → **INVARIANT**
> ❌ No → continue

4️⃣ **Does violation require a trusted role to misbehave?**
> ✔ Yes → **OUT OF SCOPE (Threat Model)**
> ❌ No → continue

5️⃣ **Are you tempted to write `require(property)`?**
> ✔ Yes → **Bug hiding detected**
> → Must be an `assert` or `invariant`
> ❌ No → classification stands

> 📌 **If unsure, default to RULE.**

---

# PHASE 3 — PROPERTY SPECIFICATION BLOCK

For each surviving property, output the following block:

**Name:** (e.g., `invariant_solvency`)
**Type:** Invariant | Rule
**Impact Category:** Theft / Insolvency / Privilege Escalation / Griefing
**Plain English Description:** The law being enforced
**External Truths Used:** (list contracts + fields)
**Revert/Failure Behavior:** (describe expected revert conditions, if any)

If non-empty → reviewer must check modeling.

> **NEW v1.6 — Revert Specification Requirement:**
> For each state-changing function referenced in a rule, document:
> 1. Under what conditions should it **succeed**?
> 2. Under what conditions should it **revert**?
> 3. Is the revert behavior verified via `@withrevert` + `lastReverted`?
>
> A property that only verifies the success path leaves the failure behavior undefined.
> If an attacker can manipulate *whether* a function reverts, that’s a security-critical
> behavior that MUST be specified.
>
> **Minimum standard:** Every rule for a state-changing function should either:
> - Use the complete Liveness/Effect/No-Side-Effect pattern (`@withrevert` + `<=>`), OR
> - Have a companion `_revert` rule that exhaustively enumerates revert conditions

---
## PHASE 3.5 — CAUSAL CLOSURE VALIDATION (MANDATORY)

> **This phase PROVES causal closure for each property before CVL is written.**

For each property in Phase 3:
1. Create entry in `causal_validation_[project].md`
2. Enumerate ALL mutation paths for involved variables
3. Write and run validation rules (templates below)
4. ONLY proceed if ALL validation passes

🚫 If validation fails → STOP and fix modeling.

---

### 3.5.1 — PROPERTY ANALYSIS TEMPLATE

For EACH property, document in `causal_validation_[project].md`:

```markdown
## Property: [PROPERTY_NAME]
**Type:** [INVARIANT/RULE]
**Source:** [Security property document reference]

### Variables Involved:
| Variable | Type | Location | Purpose in Property |
|----------|------|----------|---------------------|
| `variable1` | `uint256` | Storage | [Purpose] |
| `variable2` | `mathint` | Ghost | [Purpose] |

### Mutation Paths Analysis:
**`variable1`:**
| Function | Effect | Modeled? | Hook? |
|----------|--------|----------|-------|
| `function1(args)` | [effect] | ✅/❌ | ✅/❌ |
| `function2(args)` | [effect] | ✅/❌ | ✅/❌ |
| `constructor()` | [initial] | ✅/❌ | init_state axiom |

### Causal Closure Status:
- [ ] All mutation paths enumerated
- [ ] All paths have hooks (if ghost involved)
- [ ] Constructor effects modeled
- [ ] Validation rules written
- [ ] Validation rules PASS
```

---

### 3.5.2 — CVL 2.0 VALIDATION RULE TEMPLATES

**Template 1: Mutation Path Completeness**

```cvl
/**
 * @title Mutation Path Analysis for [PROPERTY_NAME]
 * @notice Verifies all changes to [variable] go through expected functions
 * @dev If this fails with unexpected function, there's an unmodeled mutation path
 */
rule mutation_paths_[variable_name](method f) 
    filtered { 
        f -> f.contract == currentContract &&
             f.selector != sig:viewOnlyFunction().selector  // Exclude view functions
    } 
{
    // Capture state before
    uint256 valueBefore = relevantVariable;
    
    env e;
    calldataarg args;
    f(e, args);
    
    uint256 valueAfter = relevantVariable;
    
    // If value changed, it must be through an expected mutator
    assert valueBefore != valueAfter => (
        f.selector == sig:expectedMutator1(address,uint256).selector ||
        f.selector == sig:expectedMutator2(address).selector ||
        f.selector == sig:expectedMutator3().selector
    ), "Unmodeled mutation path detected for [variable]";
}
```

**Template 2: Ghost Synchronization**

```cvl
/**
 * @title Ghost Synchronization for [GHOST_NAME]
 * @notice Verifies ghost stays synchronized with storage after any function
 */
rule ghost_sync_[ghost_name](method f, address anyUser) 
    filtered { f -> f.contract == currentContract }
{
    // Require invariant holds before (proves inductive step)
    requireInvariant ghostMatchesStorage();
    
    env e;
    calldataarg args;
    f(e, args);
    
    // Verify ghost still matches storage
    assert to_mathint(storageVariable()) == ghostVariable,
        "Ghost [GHOST_NAME] desynchronized from storage";
}
```

**Template 3: Constructor Initialization**

```cvl
/**
 * @title Constructor Establishes [PROPERTY_NAME]
 * @notice Verifies property holds in initial state
 */
rule constructor_establishes_[property_name] {
    // Represent fresh deployment state
    require totalSupply() == 0;
    require ghostSum == 0;
    // Add other initial state requirements
    
    // Verify property holds initially
    assert [property_condition],
        "Property [NAME] not established by constructor";
}
```

**Template 4: Invariant Preservation with Dependencies**

```cvl
/**
 * @title Preservation Test for [INVARIANT_NAME]
 * @notice Explicitly tests that dependent invariants enable preservation
 */
rule preservation_test_[invariant_name](method f, address user) 
    filtered { f -> f.contract == currentContract }
{
    // Load dependent invariants (these must be proven first)
    requireInvariant dependentInvariant1();
    requireInvariant dependentInvariant2(user);
    
    // Assume this invariant holds before
    require [invariant_condition_before];
    
    env e;
    calldataarg args;
    f(e, args);
    
    // Assert it holds after
    assert [invariant_condition_after],
        "Invariant [NAME] not preserved";
}
```

---

### 3.5.3 — VALIDATION EXECUTION

1. **Create** `validation_[project].spec` with validation rules
2. **Run** Certora prover on validation rules
3. **Interpret results:**

| Result | Meaning | Action |
|--------|---------|--------|
| All rules PASS | Causal closure verified | Proceed to Phase 4 |
| Mutation path rule fails | Unmodeled function mutates variable | Add to mutation path list, add hook if needed |
| Ghost sync rule fails | Missing Sstore hook | Add hook for the mutation path |
| Constructor rule fails | Initial state not modeled | Add init_state axiom or constructor model |
| Preservation rule fails | Missing dependency | Add requireInvariant for the dependency |

🚫 **Do NOT proceed until ALL validation rules pass.**
### Multi-Epoch Attack Awareness

Before proceeding, check whether the protocol requires multi-epoch analysis:

* Time-delayed operations (withdrawals, unlocking, vesting)?
* Interest/reward accrual over time?
* Oracle price feeds that update between blocks?
* Governance voting with token accumulation?

If YES → flag for `multi-step-attacks-template.md` multi-epoch patterns in Phase 8.
**Reference:** `certora-master-guide.md` Section 7.6 (Multi-Epoch Attack Modeling)
---

## REALITY CONSTRAINTS (ANTI-FALSE-POSITIVE FILTER)

Constraints **must not encode correctness**.

Allowed:

* Deployment facts
* EVM-enforced constraints
* Legitimate reverts
* Explicit threat-model trust

**Unrealistic Scenarios (Physics Only):**

Only exclude:

* EVM impossibilities
* Solidity type violations
* Explicit code-enforced execution laws

> If the blockchain could produce the state, Certora must consider it.

🚨 Any constraint whose removal reintroduces a **real exploit** is **invalid**.

---

**CVL Logic**

* Invariant → Global, inductive
* Rule → Pre / Action / Post
* ❌ Never `require` the property

**Ghosts Required:** Yes / No
**Attack Vector Caught:** Concrete exploit

---

# PHASE 4 — MODELING DECISION MATRIX

**HARD RULE**

> 🚫 It is forbidden to model correctness.
> Only execution behavior may be modeled.

For each external dependency:

### ERC20-like

✅ Dispatcher (default)
✅ Constant summaries only if immutable + pure read
🚫 Never `NONDET`

### Lending Protocols (Aave, Compound, etc.)

🚫 Never assume solvency or price correctness
🚫 Never assume interest monotonicity
✅ Model as adversarial external state

### ERC721 / ERC1155

✅ Dispatcher required
Model ownership changes

### IERC165

Never assume compliance

### Oracles / Bridges / Admin Registries

Model behavior, not honesty
Document trust explicitly

---

# PHASE 5 — GHOST ANALYSIS

**HARD RULE**

> If a ghost mirrors storage owned by a contract, the spec is invalid.

Ghosts may only represent:

* aggregates
* cross-contract totals
* history not stored on-chain

☐ Is all global truth derivable from storage?
If no → introduce ghosts

Rules:

* Ghost only reality
* Attach only to real mutation points
* Never ghost “correctness”

---

# PHASE 6 — FINAL SANITY GATE

Before CVL is written:

### Execution Closure Checks
☐ All entry points enumerated
☐ All state mutations mapped
☐ All external reads modeled or trusted
☐ Every external read has a modeled write
☐ All NONDET usages justified (see acceptable/forbidden rules)
☐ All contract-typed addresses bound or constrained

### Causal Closure Checks
☐ Causal validation completed for ALL properties
☐ **Function reachability: `satisfy` rules PASS for every state-changing function** ← NEW v1.8
☐ **Failure-path reachability: `satisfy lastReverted` rules PASS for critical revert conditions** ← NEW v1.9
☐ All mutation paths enumerated and validated
☐ All ghosts have Sstore hooks for every mutation path
☐ All ghosts have Sload hooks enforcing relationships
☐ Constructor effects modeled (init_state axioms)
☐ No causally unconstrained state changes
☐ No HAVOC-dependent paths to property variables

### Custom Summary Accuracy Checks (NEW v1.9)
☐ For each custom summary: documented as Exact / Overapproximation / Underapproximation
☐ Custom summaries do not assume determinism the real function doesn't guarantee
☐ Custom summary accuracy justified in Phase -1.3 Modeling Obligations
☐ Removing any custom summary constraint does not reintroduce a real exploit

### Invariant Dependency Safety (NEW v1.9)
☐ Every invariant annotated with `@dev Level: N` (dependency depth)
☐ Invariant dependency DAG documented (no cycles)
☐ `requireInvariant` only references strictly lower-level invariants
☐ Base invariants (Level 1) proven in isolation first
☐ Higher-level invariants proven only after their dependencies pass

### Bounded State Checks
☐ Array lengths bounded realistically (< 1000)
☐ Timestamps bounded (<= max_uint40)
☐ Counter/ID values bounded realistically
☐ Loop iterations bounded in preserved blocks where needed

### Property Quality Checks
☐ No property mixes two truth owners
☐ No ghost competes with real storage
☐ No oracle / protocol assumptions hidden
☐ No invariant relies on documentation
☐ No exploit required away

### Edge Case Checks
☐ `Block.timestamp` dependence analyzed
☐ `Block.number` dependence analyzed
☐ `msg.value` influence on logic analyzed
☐ Silent rounding / truncation analyzed
☐ Zero-value edge cases analyzed

### Revert/Failure Coverage Checks (NEW in v1.6)
☐ Every state-changing function has `@withrevert` coverage (dedicated rule or Liveness pattern)
☐ Revert conditions documented in Phase 3 property blocks
☐ Liveness assertions use biconditional `<=>` (not just implication `=>`)
☐ `lastReverted` captured immediately after `@withrevert` calls (not overwritten)
☐ Non-payable function reverts included in revert condition enumeration
☐ Access control revert behavior verified (unauthorized → must revert)
☐ No rule silently relies on default revert pruning for critical behavior

🚨 If ANY check fails → **STOP** and fix before writing CVL

---

## ADVERSARIAL DESIGN INTERROGATION

> **MANDATORY before Phase 7.** The code may be correct and the design may still be extractable.

Answer these questions and record in `spec_authoring/design_interrogation.md`:

1. Who benefits most if the protocol behaves *as specified*?
2. What invariant would users *assume* exists but is not enforced?
3. Does the protocol reward adversarial strategy by design?
4. Can "optimal behavior" be economically malicious?
5. What happens if all actors behave selfishly and rationally?

**Output:** Candidate attacker objectives, profit metrics, state variables of leverage.

These outputs directly inform offensive specifications in Phase 8.

**Reference:** `certora-master-guide.md` Section 8.4

---

# PHASE 7 — CVL SPECIFICATION WRITING

> **Only enter this phase after Phase 6 sanity gate passes.**  
> **This begins as a MINIMAL defensive hypothesis, not a full spec.**  
> **The full defensive spec is written LAST, after the adversarial verification loop converges.**

**Use** `certora-spec-framework.md` **for:**
- Methods block structure
- Ghost and hook patterns
- validState() and validEnv() functions
- Invariant and rule templates
- CVL 2.0 syntax reference

**Workflow:**
1. Import Common.spec if using shared summaries
2. Write methods block (all summaries from Phase 4)
3. Write ghosts with init_state axioms (from Phase 5)
4. Write hooks for all mutation paths
5. Write validState() function bundling all invariants
6. Write validEnv() function for env constraints
7. Write invariants (Level 1 first, then dependent levels)
8. Write rules (call validState() at start of each)

---

# PHASE 8 — ATTACK SYNTHESIS (OFFENSIVE) ← NEW v3.0

> **Enter after Phase 6 sanity gate AND adversarial design interrogation (\u00a78.4) pass.**  
> **Offensive and defensive specs are mutually adversarial hypotheses over the same causal model.**  
> **Final defensive proof comes LAST, after the attack surface is exhausted and profit boundary established.**  
> **See `certora-master-guide.md` Section 1.4 for the Adversarial Verification Model.**

**Adversarial Verification Loop:**
```
─────────────────────────────────────────────────────────────────────
DEFENSIVE HYPOTHESIS:  "What must the design never allow?"
OFFENSIVE EXISTENTIAL: "Can an attacker extract profit?"
FEEDBACK LOOP:         SAT → exploit or refine. UNSAT → weaken or expand.
FINAL PROOF:           Defensive verification — ALWAYS LAST.
─────────────────────────────────────────────────────────────────────
```

**Workflow:**
1. **Economic Impact Assessment:** List assets, values, entry/exit points
2. **Create impact.spec:** Ghost tracking for actor_value, total_system_value
3. **Write Anti-Invariants:** Rules expected to FAIL if exploit exists
4. **Run Attack Search:** If anti-invariant fails, CE contains exploit params
5. **CE→Exploit Conversion:** Convert CE to Foundry PoC, validate on fork
6. **Multi-Step Attacks:** Flash loan, sandwich, staged, governance patterns

**Key Templates:**
- `impact-spec-template.md` — Value tracking infrastructure
- `multi-step-attacks-template.md` — Attack pattern library
- `poc-template-foundry.md` — CE→PoC conversion

**Reference:** `certora-master-guide.md` Section 9.5 (Phase 8: Attack Synthesis)

### Profit Escalation (v3.2)

Do not stop at existence. For every SAT anti-invariant:

1. Record the profit value from the CE
2. Increase the threshold: `satisfy attacker_profit >= higher_value`
3. Repeat until UNSAT — this establishes the maximum extractable value
4. If ALL thresholds SAT → UNBOUNDED vulnerability (CRITICAL)

**Reference:** `certora-master-guide.md` Section 9.5.10 (Attacker Optimization & Profit Escalation)

---

## ✅ RESULT OF THIS CONTRACT

A valid spec must end with:

1. Complete execution-reality map
2. Explicit trust boundaries
3. Sound modeling choices
4. Correct invariant vs rule classification
5. Zero ambiguity about adversarial behavior
6. **Causal validation PASSED for all properties**
7. **All sanity gate checks PASSED**

---

## FRAMEWORK INTEGRATION

This document is part of the complete framework:

| Document | Purpose | When to Use |
|----------|---------|-------------|
| **spec-authoring-certora.md** | Understand contract, model execution, classify properties | Before writing any CVL |
| **certora-spec-framework.md** | CVL 2.0 syntax, templates, patterns | When writing CVL spec |
| **cvl-language-deep-dive.md** | Complete CVL language reference (types, ghosts, hooks, invariants) | CVL mastery ⭐ NEW v1.5 |
| **verification-playbooks.md** | Production-ready worked examples (ERC-20, WETH, ERC-721) | Copy & adapt ⭐ NEW v1.5 |
| **certora-ce-diagnosis-framework.md** | Debug counterexamples systematically | When prover returns CEs |
| **best-practices-from-certora.md** | Official tutorial techniques & patterns | Throughout verification |
| **advanced-cli-reference.md** | Performance optimization, timeout mitigation | CLI flags ⭐ NEW v1.4 |
| **quick-reference-v1.3.md** | Cheat sheet & quick syntax lookup | Keep open while coding |

**Workflow:**
```
SPEC AUTHORING → Pass Phase 6 → SPEC FRAMEWORK (write CVL) → Run Prover → CE DIAGNOSIS (if needed)
                     ↓                     ↓                                    ↓
              BEST_PRACTICES        CVL_LANGUAGE_DEEP_DIVE              BEST_PRACTICES
               Section 1           + VERIFICATION_PLAYBOOKS               Section 2
```

---

> **If execution reality is wrong, the proof is meaningless.**
> **Causal validation defines reality. Defensive spec states intent.**
> **Offensive spec attacks intent. The loop refines both. Proof comes last.**
