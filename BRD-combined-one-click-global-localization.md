# Business Requirement Document: OmniTranslate — One-Click Global Clinical Localization

**Document Version:** 2.0 (Combined)
**Date:** 2026-05-11
**Status:** Draft
**Author:** Platform Assistant Team
**Project Type:** Hackathon Initiative → Production Roadmap

---

## 1. Executive Summary

Expanding a clinical trial globally requires translating thousands of labels, forms, and error messages across eCOA and EDC environments. The current manual process breaks underlying system OIDs and logic, fails to preserve medical intent, and takes 4–8 weeks per language to complete.

OmniTranslate is an AI-powered orchestration layer that achieves "One-Click Global Localization" by separating technical logic from clinical content, leveraging a multi-agent translation pipeline grounded in study protocol context, and ensuring 100% structural integrity through automated validation. The system replaces weeks of manual effort with a single operation that produces production-ready, schema-validated output files.

---

## 2. Problem Statement

### 2.1 Current State

When expanding a clinical trial to new geographies, teams must manually translate:

- **Form labels and field names** — thousands per study
- **Error messages and validation text** — tightly coupled to system logic
- **eCOA instruments** — patient-facing content requiring medical accuracy
- **EDC annotations and help text** — operational guidance for site staff

### 2.2 Root Causes

| Root Cause | Example | Impact |
|------------|---------|--------|
| No separation of translatable content from technical structure | Translators see raw exports where OIDs and display text are interleaved | Accidental OID modification renders environments non-functional |
| Context loss | "Arm" translated as "Upper Limb" instead of "Study Group" | Patient-facing errors that can invalidate trial data |
| No automated quality gate | Broken logic discovered only during UAT | Weeks of re-work |
| Fragmented tooling | EDC, eCOA, RTSM each have different export/import formats | No unified translation layer |
| Generic translation tools | No awareness of MedDRA, CDISC, or therapeutic area terminology | Domain-sensitive mistranslation |

### 2.3 Operational Impact

- **Timeline:** 4–8 weeks per language, per study
- **Cost:** Specialized bilingual + technical staff as resource bottleneck
- **Risk:** Regulatory rejection, data integrity failures, patient safety concerns

---

## 3. Business Objectives

| # | Objective | Success Metric |
|---|-----------|----------------|
| 1 | Reduce localization cycle time from weeks to hours | ≥90% reduction (20+ days → <1 day) |
| 2 | Eliminate OID/logic breakage during translation | 0% modification of OIDs or system-level identifiers |
| 3 | Preserve medical intent and regulatory terminology | >98% back-translation accuracy verified by clinical lead |
| 4 | Enable single-action localization for entire study environments | One-click initiation covering all translatable assets |
| 5 | Support iterative refinement without re-translation of unchanged content | Delta-only re-processing on study amendments |
| 6 | Achieve 21 CFR Part 11 compliance readiness | Full traceability matrix of original vs. translated strings |

---

## 4. Scope

### 4.1 In Scope

- EDC form labels, field labels, help text, error messages, and validation messages
- eCOA instrument text (patient-facing questionnaires, instructions, response options)
- System-generated messages (notifications, alerts, status text)
- Input formats: **CDISC ODM-XML**, JSON, XLSX
- Translation to 50+ languages including Right-to-Left (RTL) support (Arabic, Hebrew)
- Preservation of OIDs, variable names, edit checks, skip logic, and derivations
- RAG-grounded translation using Study Protocol, MedDRA/CDISC dictionaries, and **Medidata Approved Translation Library**
- Human-in-the-loop review workflow for medical/regulatory content
- Automated back-translation for semantic drift detection
- 21 CFR Part 11 audit trail and traceability matrix
- Rollback capability to revert to pre-translation state

### 4.2 Out of Scope (v1)

- RTSM/IRT system localization (future phase)
- Source document translation (protocols, ICFs managed separately)
- Voice/audio localization for eCOA

---

## 5. Stakeholders

| Role | Interest | Involvement |
|------|----------|-------------|
| Clinical Data Manager | Study integrity, timeline | Requirements, UAT |
| Globalization/Localization Lead | Quality, consistency | Review workflow, acceptance |
| eCOA Product Team | Instrument fidelity | Technical integration |
| EDC Configuration Team | OID/logic preservation | Technical integration |
| Regulatory Affairs | 21 CFR Part 11 compliance, terminology | Review gate |
| Site Operations | Usability of translated content | Feedback |
| Engineering (Platform) | Architecture, AI integration | Build |

---

## 6. Functional Requirements

### 6.1 Content Extraction & Structural Parsing

| ID | Requirement |
|----|-------------|
| FR-01 | System SHALL ingest CDISC ODM-XML, JSON, and XLSX study definition formats. |
| FR-02 | System SHALL programmatically isolate translatable text labels from system OIDs using a Key-Value tokenization engine (Key = immutable OID, Value = translatable string). |
| FR-03 | System SHALL classify each text segment by type: label, error message, help text, instruction, response option. |
| FR-04 | System SHALL identify and "mask" logical operators and scripts (e.g., `if (AGE < 18)`) using regex-based protection layers that wrap code in non-translatable tags. |
| FR-05 | System SHALL protect non-translatable tokens (OIDs, variable names, code list values, regular expressions, placeholders like `{FIELD_NAME}`). |

### 6.2 Context-Aware AI Translation

| ID | Requirement |
|----|-------------|
| FR-06 | System SHALL translate text using a multi-agent AI pipeline: Agent A (Translator), Agent B (Medical Reviewer), Agent C (Back-Translation Validator). |
| FR-07 | System SHALL use RAG (Retrieval-Augmented Generation) grounded in the Study Protocol, **Medidata Approved Translation Library**, and domain dictionaries to resolve ambiguity. When a Medidata-approved translation already exists for a given string, the system SHALL use it directly rather than generating a new translation. |
| FR-08 | System SHALL query the Medidata Approved Translation Library as the first step before AI translation. If an approved translation exists for the source string + target language pair, it SHALL be used without modification and marked as "pre-approved" in the audit trail. |
| FR-09 | System SHALL accept therapeutic area-specific glossaries aligned with MedDRA and CDISC dictionaries to enforce consistent terminology. |
| FR-10 | System SHALL provide surrounding technical context to the translation model (what the label is for, what validation it triggers, target audience). |
| FR-11 | System SHALL support batch translation of an entire study environment in a single operation. |
| FR-12 | System SHALL support incremental translation — only processing new or changed content on study amendments. |

### 6.3 Automated Quality Assurance

| ID | Requirement |
|----|-------------|
| FR-13 | System SHALL automatically perform back-translation (target → source) and flag semantic drift exceeding a configurable threshold. |
| FR-14 | System SHALL validate that all OIDs remain unchanged post-translation. |
| FR-15 | System SHALL validate that edit check logic references and skip-logic scripts are intact. |
| FR-16 | System SHALL validate that character encoding is correct for the target language. |
| FR-17 | System SHALL validate that translated text fits within field length constraints. |
| FR-18 | System SHALL produce a structural diff report comparing pre- and post-translation study definitions. |
| FR-19 | System SHALL assign confidence scores to each translation and flag low-confidence segments for mandatory human review. |
| FR-20 | System SHALL verify terminology consistency — the same source term maps to the same target term across the entire study. |

### 6.4 Review & Approval Workflow

| ID | Requirement |
|----|-------------|
| FR-21 | System SHALL present translated content in a side-by-side review interface (source vs. target) with technical context. |
| FR-22 | System SHALL allow reviewers to accept, reject, or modify individual translations. |
| FR-23 | System SHALL route high-risk strings (primary endpoint questions, patient-facing safety content) to clinical lead review. |
| FR-24 | System SHALL maintain a 21 CFR Part 11-ready audit trail: traceability matrix of original string, AI translation, human modifications, and approver identity. |

### 6.5 Export & Integration

| ID | Requirement |
|----|-------------|
| FR-25 | System SHALL re-assemble translated values into the original XML/JSON schema based on the OID key, producing a "Mirror Environment" file structure. |
| FR-26 | System SHALL generate export files that pass the target EDC/eCOA platform's schema validation test on import. |
| FR-27 | System SHALL support direct re-import into the source system without manual intervention. |
| FR-28 | System SHALL provide a rollback capability to revert to the pre-translation state. |

---

## 7. Non-Functional Requirements

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-01 | Translation of a standard 1,000-field study environment SHALL complete in < 15 minutes. Full study (5,000+ segments) in < 1 hour. | Performance |
| NFR-02 | System SHALL support 50+ global languages including RTL (Arabic, Hebrew). | Scalability |
| NFR-03 | 0% modification of OIDs or system-level unique identifiers. | Reliability |
| NFR-04 | All data SHALL remain within approved cloud boundaries (no external API calls with PHI). | Security |
| NFR-05 | System SHALL maintain a detailed traceability matrix for 21 CFR Part 11 readiness. | Compliance |
| NFR-06 | System SHALL log all AI model invocations for auditability. | Compliance |
| NFR-07 | System SHALL achieve ≥95% translation acceptance rate without human modification; >98% back-translation accuracy. | Quality |
| NFR-08 | System SHALL be available as a self-service capability (no engineering intervention required). | Usability |

---

## 8. AI Architecture

### 8.1 Logical Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     ONE-CLICK TRIGGER                            │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 1: Tokenization Engine                                    │
│  • Parse CDISC ODM-XML / JSON / XLSX                            │
│  • Build Key-Value pairs (Key=OID, Value=translatable string)   │
│  • Mask logic scripts with regex protection layers              │
│  • Classify segments by type                                    │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 2: Contextual Grounding (RAG) + Approved Translation      │
│  • Query Medidata Approved Translation Library FIRST             │
│  • If approved translation exists → use directly (skip Step 3)  │
│  • If no match → load Study Protocol as Knowledge Base           │
│  • Load MedDRA + CDISC dictionaries                             │
│  • Load therapeutic area glossary                               │
│  • Resolve ambiguity: "Arm" → "Study Group" (not "Upper Limb") │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 3: Multi-Agent Translation Loop                           │
│  (Only for strings WITHOUT a pre-approved translation)          │
│                                                                 │
│  Agent A (Translator)                                           │
│    → Generates target language string with context              │
│    → Respects glossary, preserves placeholders                  │
│                                                                 │
│  Agent B (Medical Reviewer)                                     │
│    → Compares translation against medical dictionary            │
│    → Validates domain terminology accuracy                      │
│    → Assigns confidence score                                   │
│                                                                 │
│  Agent C (Back-Translation Validator)                            │
│    → Translates target → source                                 │
│    → Compares semantic similarity to original                   │
│    → Flags deviations exceeding threshold                       │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 4: Integrity Validation                                   │
│  • OID preservation check (100%)                                │
│  • Edit check / skip-logic integrity                            │
│  • Field length constraint validation                           │
│  • Character encoding verification                              │
│  • Structural diff report                                       │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 5: Human Review (conditional)                             │
│  • Low-confidence segments → mandatory review                   │
│  • High-risk strings → clinical lead review                     │
│  • Pre-approved strings bypass review (already validated)       │
│  • Side-by-side interface with context                          │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 6: Re-assembly & Export                                   │
│  • Re-insert translated values by OID key                       │
│  • Generate Mirror Environment file structure                   │
│  • Run schema validation                                        │
│  • Produce traceability matrix                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Model Requirements

- **Domain specialization**: Must understand clinical trial terminology, medical abbreviations, and regulatory language.
- **Context window**: Must process entire form context (not isolated strings) to maintain coherence.
- **Glossary adherence**: Must respect provided terminology glossaries over general translation.
- **Structural awareness**: Must distinguish translatable text from technical tokens within the same string.

### 8.3 Prompt Engineering Strategy

- System-level context: therapeutic area, study phase, target audience (patient vs. site staff).
- Structural metadata: "This is an error message triggered when field X fails edit check Y."
- Output format constraints: translated text must maintain placeholder positions and formatting markers.
- Disambiguation examples: provide protocol-specific term mappings.

---

## 9. User Stories

| ID | As a... | I want to... | So that... |
|----|---------|--------------|------------|
| US-01 | Clinical Data Manager | Initiate localization of my entire study with one click | I can activate new countries without weeks of manual work |
| US-02 | Localization Reviewer | See AI translations alongside source text with technical context | I can efficiently validate medical accuracy |
| US-03 | EDC Configuration Lead | Confirm that no OIDs or edit checks were broken via structural diff | I can trust the translated study will function correctly |
| US-04 | Regulatory Affairs | Enforce approved terminology and access a 21 CFR Part 11 traceability matrix | Translations comply with regulatory expectations |
| US-05 | Study Manager | Translate only the amendments, not the entire study again | Country activations for amended studies are fast |
| US-06 | eCOA Product Owner | Ensure patient-facing translations are culturally appropriate and back-translation verified | Patient comprehension and compliance are maintained |
| US-07 | Clinical Lead | Review high-risk strings flagged by the AI with confidence scores | Patient safety content is never mistranslated |

---

## 10. Assumptions & Constraints

### Assumptions

1. Source study definitions are available in CDISC ODM-XML, JSON, or XLSX format.
2. Study Protocol documents are available for RAG ingestion.
3. Medidata Approved Translation Library is accessible and contains previously approved string translations indexed by source text and target language.
4. MedDRA and CDISC dictionaries are accessible; therapeutic area glossaries exist or can be curated.
5. AI translation models (AWS Bedrock, Claude) are available within approved infrastructure.
6. Human reviewers are available for the approval workflow.

### Constraints

1. No patient data (PHI/PII) is present in the translatable content (labels and messages only).
2. All processing must occur within Medidata's approved cloud environment.
3. Must integrate with existing EDC and eCOA systems via their native import/export mechanisms.
4. Regulatory submissions may still require certified human translation for specific instruments — system must flag these.
5. 21 CFR Part 11 compliance requires electronic signatures and complete audit trail.

---

## 11. Risk Assessment & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| AI mistranslates a medical term (patient safety) | Medium | Critical | Multi-agent review pipeline; mandatory clinical lead review for high-risk strings; back-translation validation; glossary enforcement |
| OID/logic breakage not caught | Low | Critical | Regex-based logic masking; structural diff engine; schema validation on export; automated regression tests |
| Hallucination introduces non-existent text | Low | High | Output length validation; source-target alignment checks; back-translation comparison |
| Syntax corruption in nested logic scripts | Medium | High | Non-translatable tag wrapping; regex protection layers; logic masking as pre-processing step |
| Regulatory rejection of AI-translated content | Medium | Medium | 21 CFR Part 11 traceability matrix; human approval audit trail; flagging instruments requiring certified translation |
| Performance degradation with large studies | Low | Medium | Batch processing with parallelization; incremental translation; <15 min target for 1,000 fields |

---

## 12. Success Criteria

### Hackathon Demo

1. **End-to-end flow**: Ingest CDISC ODM-XML → Tokenize → Translate → Validate → Export for a sample study.
2. **OID preservation**: 100% of exported files pass EDC schema validation test on import.
3. **Multi-language**: Demonstrate at least 3 target languages in a single operation.
4. **Context-aware translation**: Show "Arm" correctly translated as "Study Group" based on protocol context.
5. **Back-translation**: Demonstrate automated semantic drift detection with flagging.
6. **Confidence scoring**: At least one segment flagged for human review with explanation.

### Production Readiness

| Metric | Target |
|--------|--------|
| Schema validation pass rate | 100% |
| Back-translation accuracy | >98% verified by clinical lead |
| Time savings | 20+ days → <1 day |
| OID integrity | 0% modification |
| Translation acceptance (no human edit needed) | ≥95% |

---

## 13. Future Vision (Post-Hackathon)

- **Translation memory**: Learn from reviewer corrections to improve over time.
- **Real-time preview**: See translated forms rendered in the target system before committing.
- **Multi-system orchestration**: Single operation localizes EDC + eCOA + RTSM together.
- **Regulatory intelligence**: Auto-detect which instruments require certified translation per country.
- **Continuous localization**: Automatically translate new content as studies are built, not as a separate phase.
- **Adaptive glossaries**: Auto-update terminology databases from approved reviewer corrections.

---

## 14. Glossary

| Term | Definition |
|------|------------|
| OID | Object Identifier — unique technical reference for forms, fields, and code lists |
| EDC | Electronic Data Capture — system for collecting clinical trial data |
| eCOA | Electronic Clinical Outcome Assessment — patient-reported outcome instruments |
| CDISC ODM-XML | Clinical Data Interchange Standards Consortium Operational Data Model — industry standard study definition format |
| MedDRA | Medical Dictionary for Regulatory Activities — standardized medical terminology |
| RTSM/IRT | Randomization and Trial Supply Management / Interactive Response Technology |
| RAG | Retrieval-Augmented Generation — grounding LLM output in domain-specific knowledge |
| Medidata Approved Translation Library | Repository of previously human-reviewed and approved translations indexed by source string and target language, used as the primary lookup before AI translation |
| Edit Check | Programmatic validation rule applied to collected data |
| Skip Logic | Conditional display rules that show/hide questions based on prior answers |
| Back-translation | Translating the target language back to source to verify semantic accuracy |
| 21 CFR Part 11 | FDA regulation on electronic records and electronic signatures |
| Traceability Matrix | Document mapping original strings to translations with decision audit trail |

---

## 15. Approvals

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Product Owner | | | |
| Engineering Lead | | | |
| Localization Lead | | | |
| Regulatory Affairs | | | |
| Clinical Lead | | | |

---

*This document serves as the unified foundation for a Technical Specification that will detail the multi-agent AI architecture, RAG implementation, CDISC ODM-XML parsing, data models, and integration approach.*
