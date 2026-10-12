I read all four `plan2_*` reviews in full, line by line, and consolidated the **net-valid findings only**. The important result is that the reviewers did **not** identify a flaw requiring us to reopen the three-project architecture. Their remaining findings were about making the boundaries mechanically precise, and those have now been converted into explicit architectural decisions.

The consolidated document is now **v1.0 — Architecture Closed / Implementation Plan Authorized**.

The major closures are:

- **Project A — TOE-Agent:** hypothesis construction, hypothesis chains, attacks, counterexamples, correspondence proposals, research state, and formalization handoff.
- **Project B — PhysMath:** reusable mathematics only; no physical ontology or trust metadata.
- **Project C — `phys-*`:** reusable physical-domain semantics such as `phys-qm`, `phys-bohm`, `phys-gr`, `phys-cst`, `phys-cdt`.
- **Frozen Phys:** the sole owner of physical assumptions, conventions, provenance, corpus status, MRC/trust, and trusted physical formalization.
- **Numerical Layer:** quantitative computation only.

The reviews' key issues are explicitly closed, including the PhysMath→Phys dependency tension, external-object shape, metadata attachment, interpretation ambiguity, multi-sorted mathematical IR, canonicalization, conditions/scoping, asymptotics, two replay domains, correspondence artifacts, `t`/`k` separation, TOE joint-dynamics acceptance, AI-agent protocol, numerical handoff, Growth Gate routing, and resource containment. DeepSeek's review identified these as blockers preventing implementation without agent invention. plan2_deepseek Gemini likewise confirmed the three-project separation but required explicit decision closure before implementation. plan2_gemini

I also resolved the specific Z findings rather than merely restating them: the PhysMath/Phys dependency is now isolated through an explicit interoperability boundary; external-domain trust remains contained because the frozen Phys MVP has no new-domain curation mechanism; the provenance/trace terminology is separated; the dependency diagram is normative; replay/handoff precedence is pinned; and research-status vocabulary is separated from obstacle-class vocabulary. Those were the principal structural problems the review identified. plan2_z plan2_z

The final architectural principle is:

\[
\boxed{
\text{TOE-Agent}
\rightarrow
\text{PhysMath}
\rightarrow
\text{phys-*}
\rightarrow
\text{Phys formalization}
\rightarrow
\text{Numerical consequences}
}
\]

with the much more precise ownership model:

\[
\boxed{
\text{Research} \neq \text{Math} \neq \text{Physics} \neq \text{Trust} \neq \text{Numerics}
}
\]

Most importantly, the document now says **there are no open architectural decisions for the coding agent to invent**. Remaining choices such as private helper names, exact algorithms, test-file organization, and transport mechanisms are explicitly classified as implementation choices rather than architecture.

:chatgpt-content-reference{index="4"}[Download the consolidated, implementation-authorized architecture v1.0](sandbox:/mnt/data/PHYSMATH_REQUIREMENTS_ARCHITECTURAL_BLOCKERS_v1.0.md)