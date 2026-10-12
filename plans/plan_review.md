Done. I consolidated the four reviews around the **three separate software projects** rather than treating PhysMath/Phys/TOE as one implementation.

The key architectural correction is now explicit:

```text
Project A — TOE AI Reasoning / Research Program
    reasons about hypotheses and research chains

Project B — PhysMath
    reusable theoretical mathematics

Project C — Phys External Libraries
    reusable physical-domain semantics
```

with the existing `Phys` trust substrate and Numerical Layer remaining distinct.

This resolves several review findings by **moving responsibilities to the correct project**, rather than adding another layer. In particular, the proposed “Candidate Physical Object” / “Interpretation Layer” is not accepted as a new PhysMath layer; research-level correspondence and handoff artifacts belong to the TOE-Agent project, while physical semantics remain in `phys-*` and canonical trust metadata remains in Phys. The reviews independently converged on the need to make these boundaries mechanically enforceable rather than leaving them as prose. plan_deepseek plan_z

I also retained the valid blockers around PhysMath IR/sorts, canonicalization, external physical-object contracts, metadata attachment, formalization bridge/loss accounting, `t` vs `k`, TOE joint dynamics, AI-agent schemas, and numerical handoff. The proposed Qwen “domain tagging” is reframed as **generic mathematical typing**, not Dancer/Dance/Stage ontology. plan_qwen

The revised document is now **v0.5** and remains **not implementation-authorized** until the decision-level blockers are closed.

:chatgpt-content-reference{index="3"}[Download the consolidated PhysMath architectural requirements v0.5](sandbox:/mnt/data/PHYSMATH_REQUIREMENTS_ARCHITECTURAL_BLOCKERS_v0.5.md)