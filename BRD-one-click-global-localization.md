# Business Requirement Document: One-Click Global Localization

**Document Version:** 1.0
**Date:** 2026-05-11
**Status:** Draft
**Author:** Platform Assistant Team
**Project Type:** Hackathon Initiative

---

## 1. Executive Summary

Clinical trial globalization requires translating thousands of labels, forms, and error messages across systems like eCOA and EDC. The current manual process is error-prone, breaks underlying system OIDs and logic, and takes weeks to complete. This initiative explores an AI-driven "One-Click Global Localization" solution that preserves technical context and medical intent while automating translation of entire study environments.

---

## 2. Problem Statement

### 2.1 Current State

When expanding a clinical trial to new geographies, teams must manually translate:

- **Form labels and field names** — thousands per study
- **Error messages and validation text** — tightly coupled to system logic
- **eCOA instruments** — patient-facing content requiring medical accuracy
- **EDC annotations and help text** — operational guidance for site staff

This manual translation process:

| Pain Point | Impact |
|------------|--------|
| Breaks OID references and system logic | Data integrity failures, re-work cycles |
| Takes weeks to complete | Delays trial activation in new regions |
| Requires specialized bilingual + technical staff | Resource bottleneck, high cost |
| Inconsistent terminology across forms | Regulatory risk, patient confusion |
| No preservation of technical metadata | Downstream integration failures |

### 2.2 Root Causes

1. **No separation of translatable content from technical structure** — translators see raw exports where OIDs, variable names, and display text are interleaved.
2. **Context loss** — translators lack visibility into how a label relates to validation logic, skip patterns, or regulatory requirements.
3. **No automated quality gate** — broken OIDs and logic errors are only caught during UAT, weeks after translation.
4. **Fragmented tooling** — different systems (EDC, eCOA, RTSM) each have their own export/import formats with no unified translation layer.

---

## 3. Business Objectives

| # | Objective | Success Metric |
|---|-----------|----------------|
| 1 | Reduce localization cycle time from weeks to hours | ≥90% reduction in elapsed time |
| 2 | Eliminate OID/logic breakage during translation | 0 structural integrity errors post-translation |
| 3 | Preserve medical intent and regulatory terminology | ≥95% acceptance rate by medical reviewers |
| 4 | Enable single-action localization for entire study environments | One-click initiation covering all translatable assets |
| 5 | Support iterative refinement without re-translation of unchanged content | Delta-only re-processing on study amendments |

---

## 4. Scope

### 4.1 In Scope

- EDC form labels, field labels, help text, error messages, and validation messages
- eCOA instrument text (patient-facing questionnaires, instructions, response options)
- System-generated messages (notifications, alerts, status text)
- Translation to any language supported by the target geography
- Preservation of OIDs, variable names, edit checks, skip logic, and derivations
- Human-in-the-loop review workflow for medical/regulatory content
- Audit trail of all translation decisions

### 4.2 Out of Scope (v1)

- RTSM/IRT system localization (future phase)
- Source document translation (protocols, ICFs managed separately)
- Voice/audio localization for eCOA
- Right-to-left (RTL) layout rendering changes (UI concern, not translation)

---

## 5. Stakeholders

| Role | Interest | Involvement |
|------|----------|-------------|
| Clinical Data Manager | Study integrity, timeline | Requirements, UAT |
| Globalization/Localization Lead | Quality, consistency | Review workflow, acceptance |
| eCOA Product Team | Instrument fidelity | Technical integration |
| EDC Configuration Team | OID/logic preservation | Technical integration |
| Regulatory Affairs | Terminology compliance | Review gate |
| Site Operations | Usability of translated content | Feedback |
| Engineering (Platform) | Architecture, AI integration | Build |

---

## 6. Functional Requirements

### 6.1 Content Extraction & Parsing

| ID | Requirement |
|----|-------------|
| FR-01 | System SHALL extract all translatable text from EDC study definitions while preserving structural metadata (OIDs, form/field hierarchy, edit check references). |
| FR-02 | System SHALL extract all translatable text from eCOA instruments while preserving question logic, skip patterns, and scoring algorithms. |
| FR-03 | System SHALL classify each text segment by type: label, error message, help text, instruction, response option. |
| FR-04 | System SHALL identify and protect non-translatable tokens (OIDs, variable names, code list values, regular expressions). |

### 6.2 AI-Powered Translation

| ID | Requirement |
|----|-------------|
| FR-05 | System SHALL translate text using AI models with clinical/medical domain context. |
| FR-06 | System SHALL accept a medical terminology glossary per therapeutic area to enforce consistent term usage. |
| FR-07 | System SHALL provide the surrounding technical context (what the label is for, what validation it triggers) to the translation model to preserve intent. |
| FR-08 | System SHALL support batch translation of an entire study environment in a single operation. |
| FR-09 | System SHALL support incremental translation — only processing new or changed content on study amendments. |

### 6.3 Integrity Validation

| ID | Requirement |
|----|-------------|
| FR-10 | System SHALL validate that all OIDs remain unchanged post-translation. |
| FR-11 | System SHALL validate that edit check logic references are intact. |
| FR-12 | System SHALL validate that character encoding is correct for the target language. |
| FR-13 | System SHALL validate that translated text fits within field length constraints. |
| FR-14 | System SHALL produce a structural diff report comparing pre- and post-translation study definitions. |

### 6.4 Review & Approval Workflow

| ID | Requirement |
|----|-------------|
| FR-15 | System SHALL present translated content in a side-by-side review interface (source vs. target). |
| FR-16 | System SHALL allow reviewers to accept, reject, or modify individual translations. |
| FR-17 | System SHALL flag low-confidence translations for mandatory human review. |
| FR-18 | System SHALL maintain an audit trail of all translation decisions (AI-generated, human-modified, approved-by). |

### 6.5 Export & Integration

| ID | Requirement |
|----|-------------|
| FR-19 | System SHALL export translated study definitions in the native format required by the target system (EDC, eCOA). |
| FR-20 | System SHALL support re-import into the source system without manual intervention. |
| FR-21 | System SHALL provide a rollback capability to revert to the pre-translation state. |

---

## 7. Non-Functional Requirements

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-01 | Translation of a full study (5,000+ text segments) SHALL complete within 1 hour. | Performance |
| NFR-02 | System SHALL support at least 20 target languages concurrently. | Scalability |
| NFR-03 | All data SHALL remain within approved cloud boundaries (no external API calls with PHI). | Security/Compliance |
| NFR-04 | System SHALL log all AI model invocations for auditability. | Compliance |
| NFR-05 | System SHALL achieve ≥95% translation acceptance rate without human modification. | Quality |
| NFR-06 | System SHALL be available as a self-service capability (no engineering intervention required). | Usability |

---

## 8. AI/ML Considerations

### 8.1 Model Requirements

- **Domain specialization**: Model must understand clinical trial terminology, medical abbreviations, and regulatory language.
- **Context window**: Must process entire form context (not isolated strings) to maintain coherence.
- **Glossary adherence**: Must respect provided terminology glossaries over general translation.
- **Structural awareness**: Must distinguish translatable text from technical tokens within the same string (e.g., `"Please enter a value for {OID_FIELD_NAME}"` — translate the sentence, preserve the token).

### 8.2 Prompt Engineering Strategy

- Provide system-level context: therapeutic area, study phase, target audience (patient vs. site staff).
- Include structural metadata as context: "This is an error message triggered when field X fails edit check Y."
- Enforce output format constraints: translated text must maintain placeholder positions and formatting markers.

### 8.3 Quality Assurance

- **Back-translation validation**: Translate target → source and compare semantic similarity.
- **Terminology consistency check**: Verify the same source term maps to the same target term across the study.
- **Confidence scoring**: Flag segments where model confidence is below threshold for human review.

---

## 9. User Stories

| ID | As a... | I want to... | So that... |
|----|---------|--------------|------------|
| US-01 | Clinical Data Manager | Initiate localization of my entire study with one click | I can activate new countries without weeks of manual work |
| US-02 | Localization Reviewer | See AI translations alongside source text with context | I can efficiently validate medical accuracy |
| US-03 | EDC Configuration Lead | Confirm that no OIDs or edit checks were broken | I can trust the translated study will function correctly |
| US-04 | Regulatory Affairs | Enforce approved terminology for my therapeutic area | Translations comply with regulatory expectations |
| US-05 | Study Manager | Translate only the amendments, not the entire study again | Country activations for amended studies are fast |
| US-06 | eCOA Product Owner | Ensure patient-facing translations are culturally appropriate | Patient comprehension and compliance are maintained |

---

## 10. Assumptions & Constraints

### Assumptions

1. Source study definitions are available in a structured, exportable format (XML, JSON, or API).
2. A medical terminology glossary exists or can be curated per therapeutic area.
3. AI translation models (e.g., AWS Bedrock, Claude) are available within the approved infrastructure.
4. Human reviewers are available for the approval workflow (the system augments, not replaces, human judgment).

### Constraints

1. No patient data (PHI/PII) is present in the translatable content (labels and messages only).
2. All processing must occur within Medidata's approved cloud environment.
3. Must integrate with existing EDC and eCOA systems via their native import/export mechanisms.
4. Regulatory submissions may still require certified human translation for specific instruments — system must flag these.

---

## 11. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| AI mistranslates a medical term, causing patient safety issue | Medium | High | Mandatory human review for patient-facing content; glossary enforcement; back-translation validation |
| OID breakage not caught by validation | Low | High | Structural diff engine compares full study definition pre/post; automated regression tests |
| Regulatory rejection of AI-translated content | Medium | Medium | Audit trail showing human approval; option to flag instruments requiring certified translation |
| Model hallucination introduces non-existent text | Low | Medium | Output length validation; source-target alignment checks |
| Performance degradation with large studies | Low | Medium | Batch processing with parallelization; incremental translation |

---

## 12. Success Criteria (Hackathon Demo)

For the hackathon proof-of-concept, success is defined as:

1. **End-to-end demo**: Extract → Translate → Validate → Export for a sample EDC study definition.
2. **OID preservation**: 100% of OIDs intact in the translated output.
3. **Multi-language**: Demonstrate at least 3 target languages in a single operation.
4. **Context-aware translation**: Show that the same English word translates differently based on its technical context (e.g., "study" as a noun vs. "study" in "study drug").
5. **Confidence flagging**: At least one segment flagged for human review with explanation.

---

## 13. Future Vision (Post-Hackathon)

- **Translation memory**: Learn from reviewer corrections to improve over time.
- **Real-time preview**: See translated forms rendered in the target system before committing.
- **Multi-system orchestration**: Single operation localizes EDC + eCOA + RTSM together.
- **Regulatory intelligence**: Auto-detect which instruments require certified translation per country.
- **Continuous localization**: Automatically translate new content as studies are built, not as a separate phase.

---

## 14. Glossary

| Term | Definition |
|------|------------|
| OID | Object Identifier — unique technical reference for forms, fields, and code lists |
| EDC | Electronic Data Capture — system for collecting clinical trial data |
| eCOA | Electronic Clinical Outcome Assessment — patient-reported outcome instruments |
| RTSM/IRT | Randomization and Trial Supply Management / Interactive Response Technology |
| Edit Check | Programmatic validation rule applied to collected data |
| Skip Logic | Conditional display rules that show/hide questions based on prior answers |
| Localization | Adapting content for a specific language and cultural context |
| Back-translation | Translating the target language back to source to verify semantic accuracy |

---

## 15. Approvals

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Product Owner | | | |
| Engineering Lead | | | |
| Localization Lead | | | |
| Regulatory Affairs | | | |

---

*This document serves as the foundation for a Technical Specification that will detail the AI architecture, system integration points, data models, and implementation approach.*
