Your Complete Framework

📁 **File Organization**

```
/home/brett/dual-governance/
│
├── 📚 METHODOLOGY (Root Level)
│   ├── CERTORA_QUICKSTART_TEMPLATE.md  ← START HERE for new contracts
│   ├── CERTORA_WORKFLOW.md             ← Detailed phase explanations
│   ├── CERTORA_SPEC_FRAMEWORK.md       ← CVL patterns & templates
│   ├── CERTORA_CE_DIAGNOSIS_FRAMEWORK.md ← Debug counterexamples
│   └── SPEC AUTHORING (CERTORA).md     ← Deep theory
│
├── 📝 spec_authoring/                  ← Analysis workspace
│   ├── {contract}_spec_authoring.md    ← Main analysis
│   ├── {contract}_candidate_properties.md ← Plain English
│   ├── {contract}_causal_validation.md ← Mutation paths
│   └── DEMONSTRATION_WALKTHROUGH.md    ← Example with Escrow
│
└── 🔧 certora/
    ├── specs/
    │   ├── validation_{contract}.spec  ← Run FIRST
    │   └── {Contract}.spec             ← Real verification
    └── confs/
        ├── validation_{contract}.conf
        └── {Contract}.conf
```
🚀 **Quick Start for New Contract**
```bash
# 1. Set your contract name
CONTRACT="YourContractName"

# 2. Create analysis files
touch spec_authoring/${CONTRACT,,}_spec_authoring.md
touch spec_authoring/${CONTRACT,,}_candidate_properties.md  
touch spec_authoring/${CONTRACT,,}_causal_validation.md

# 3. Create CVL files
touch certora/specs/validation_${CONTRACT,,}.spec
touch certora/confs/validation_${CONTRACT,,}.conf

# 4. Analyze contract (find entry points)
grep -n "function.*external" contracts/${CONTRACT}.sol

# 5. Follow CERTORA_QUICKSTART_TEMPLATE.md phases 1-7

# 6. Run validation
certoraRun certora/confs/validation_${CONTRACT,,}.conf

# 7. If all pass, write real spec
```
📋 **Which Document When?**
```
| Situation |	Use This
| Starting a new contract |	CERTORA_QUICKSTART_TEMPLATE.md |
| Need CVL syntax help	| CERTORA_SPEC_FRAMEWORK.md |
| Rule is failing	| CERTORA_CE_DIAGNOSIS_FRAMEWORK.md |
| Want to understand why	| CERTORA_WORKFLOW.md |
| See a worked example	| DEMONSTRATION_WALKTHROUGH.md |
```
**⚡ Essential Commands**

```bash
# Clear cache before running
rm -rf .certora_internal

# Run validation (always first!)
certoraRun certora/confs/validation_{contract}.conf

# Run real spec
certoraRun certora/confs/{Contract}.conf

# Run specific rule only
certoraRun certora/confs/{Contract}.conf --rule "ruleName"

# Save output to file
certoraRun certora/confs/{Contract}.conf 2>&1 | tee output.log
```
The `CERTORA_QUICKSTART_TEMPLATE.md` contains everything you need to apply this workflow to any contract - just copy the templates and fill in the blanks!
