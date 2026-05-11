# BRD Comparison: Our BRD vs. OmniTranslate BRD

**Date:** 2026-05-11
**Purpose:** Identify key differences between the two Business Requirement Documents to inform the final technical specification.

---

## Summary

Both documents address the same core problem (manual clinical trial localization breaks OIDs and takes weeks) but differ significantly in depth, scope, and architectural specificity. Our BRD is broader and more process-oriented; the OmniTranslate BRD is narrower but more technically prescriptive.

---

## Key Differences

### 1. Scope & Breadth

| Dimension | Our BRD | OmniTranslate BRD |
|-----------|---------|-------------------|
| Functional requirements | 21 requirements across 5 categories | 5 focused requirements |
| Non-functional requirements | 6 (performance, scalability, security, compliance, quality, usability) | 4 (reliability, performance, compliance, scalability) |
| User stories | 6 personas with explicit stories | None |
| Stakeholder analysis | 7 roles mapped | None |
| Risk assessment | 5 risks with likelihood/impact matrix | 2 risks with mitigations |
| RTL support | Explicitly out of scope (v1) | In scope (50+ languages including RTL) |
| Review workflow | Full human-in-the-loop workflow (FR-15 through FR-18) | Single mention of human-in-the-loop for high-risk strings |
| Rollback capability | Explicit requirement (FR-21) | Not mentioned |
| Incremental translation | Explicit requirement (FR-09) for study amendments | Not mentioned |

### 2. AI Architecture

| Aspect | Our BRD | OmniTranslate BRD |
|--------|---------|-------------------|
| Architecture detail | High-level model requirements + prompt strategy | Specific multi-agent pipeline (Translator → Reviewer → Validator) |
| RAG/Knowledge Base | Mentioned as glossary adherence | Explicitly calls out RAG with Study Protocol as knowledge base |
| Back-translation | Listed as quality assurance technique | Dedicated functional requirement (FR.4) with its own agent |
| Agent pattern | Not specified | Three-agent loop (A: Translator, B: Reviewer, C: Validator) |
| Tokenization | Described as "identify and protect non-translatable tokens" (FR-04) | Explicit "Key-Value" tokenization engine where Key=OID, Value=translatable string |
| Logic masking | Covered under FR-04 (protect OIDs, variable names, regex) | Dedicated requirement (FR.2) with specific example: `if (AGE < 18)` |
| Context example | Generic ("therapeutic area, study phase") | Specific ("Arm" → "Study Group" not "Upper Limb") |

### 3. Performance Targets

| Metric | Our BRD | OmniTranslate BRD |
|--------|---------|-------------------|
| Translation speed | 5,000+ segments in < 1 hour | 1,000 fields in < 15 minutes |
| Language support | 20+ languages | 50+ languages (including RTL) |
| Acceptance rate | ≥95% without human modification | >98% back-translation accuracy |
| OID integrity | 0 structural errors | 0% modification (same target, different wording) |

### 4. Compliance & Audit

| Aspect | Our BRD | OmniTranslate BRD |
|--------|---------|-------------------|
| Regulatory framing | General audit trail + flagging instruments needing certified translation | Explicit 21 CFR Part 11 readiness with traceability matrix |
| Audit detail | "All AI model invocations logged" (NFR-04) | "Traceability matrix of original vs. translated strings" |

### 5. Input Format Specificity

| Aspect | Our BRD | OmniTranslate BRD |
|--------|---------|-------------------|
| Input formats | "Structured, exportable format (XML, JSON, or API)" (assumption) | Explicitly names CDISC ODM-XML, JSON, XLSX |
| Industry standards | Not named | Calls out CDISC ODM-XML and MedDRA dictionaries |

### 6. Hackathon Success Criteria

| Aspect | Our BRD | OmniTranslate BRD |
|--------|---------|-------------------|
| Demo scope | End-to-end flow, 3 languages, context-aware translation, confidence flagging | Schema validation pass on import, >98% accuracy, 20+ days → <1 day |
| Validation method | OID preservation + confidence flagging | EDC schema validation test on re-import |

---

## Unique Strengths of Each

### Our BRD — Strengths

1. **Comprehensive stakeholder & user story coverage** — maps who uses it and why
2. **Review & approval workflow** — full human-in-the-loop process, not just a mention
3. **Incremental translation** — handles study amendments without full re-translation
4. **Rollback capability** — safety net for failed translations
5. **Future vision** — translation memory, real-time preview, continuous localization
6. **Confidence scoring** — flags uncertain translations for review
7. **Field length validation** — ensures translated text fits UI constraints (FR-13)
8. **Glossary per therapeutic area** — not just one global dictionary

### OmniTranslate BRD — Strengths

1. **Specific multi-agent architecture** — prescribes the Translator/Reviewer/Validator pattern
2. **RAG with Study Protocol** — concrete grounding strategy
3. **21 CFR Part 11 compliance** — names the regulation explicitly
4. **CDISC ODM-XML** — names the industry standard format
5. **Logic masking as first-class concept** — dedicated requirement with example
6. **Concrete context example** — "Arm" disambiguation makes the problem tangible
7. **RTL support in scope** — more ambitious language coverage
8. **Regex-based protection layers** — specific technical mitigation for syntax corruption

---

## Gaps to Address in Technical Spec

Based on this comparison, the technical specification should incorporate:

| # | Gap | Source |
|---|-----|--------|
| 1 | Multi-agent architecture (Translator → Reviewer → Validator) | OmniTranslate |
| 2 | RAG grounding with Study Protocol as knowledge base | OmniTranslate |
| 3 | Explicit CDISC ODM-XML and MedDRA dictionary support | OmniTranslate |
| 4 | 21 CFR Part 11 traceability matrix | OmniTranslate |
| 5 | Regex-based logic masking with non-translatable tags | OmniTranslate |
| 6 | Full review/approval workflow with confidence scoring | Our BRD |
| 7 | Incremental translation for study amendments | Our BRD |
| 8 | Rollback capability | Our BRD |
| 9 | Field length constraint validation | Our BRD |
| 10 | Therapeutic area-specific glossaries | Our BRD |

---

## Recommendation

The technical spec should use **our BRD's breadth** (stakeholders, workflow, incremental processing, rollback) as the process foundation, and layer in **OmniTranslate's architectural specificity** (multi-agent pipeline, RAG grounding, CDISC formats, 21 CFR Part 11 framing) as the implementation blueprint. The combination produces a spec that is both operationally complete and technically actionable.
