# Rave Study/Environment XML Research

**Date:** 2026-05-11
**Status:** Research Complete
**Goal:** Understand how to generate and translate study/environment XMLs from Rave EDC

---

## What Are Study XMLs?

Study XMLs in Rave EDC are **CDISC ODM (Operational Data Model)** files with Medidata extensions (`mdsol:` namespace). They define the entire structure of a clinical study — forms, fields, visits, edit checks, and crucially, all user-facing text that needs translation.

---

## XML Structure (Translatable Elements)

The translatable content lives in `<TranslatedText xml:lang="...">` elements:

```xml
<ODM FileType="Snapshot" Granularity="Metadata"
     xmlns:mdsol="http://www.mdsol.com/ns/odm/metadata"
     xmlns="http://www.cdisc.org/ns/odm/v1.3">
  <Study OID="Mediflex">
    <MetaDataVersion OID="Draft" Name="Draft1">
      <ItemDef OID="PRIMARY.FIELD1" Name="FIELD1" DataType="text">
        <Question>
          <TranslatedText xml:lang="en">FIELD1</TranslatedText>
          <TranslatedText xml:lang="ja">FIELD1</TranslatedText>
        </Question>
      </ItemDef>
      <mdsol:ConfirmationMessage xml:lang="en">Data saved</mdsol:ConfirmationMessage>
      <mdsol:ConfirmationMessage xml:lang="ja">Data saved</mdsol:ConfirmationMessage>
    </MetaDataVersion>
  </Study>
</ODM>
```

### Elements That Need Translation

| Element | Location | Purpose |
|---------|----------|---------|
| `<TranslatedText>` in `<Question>` | Inside `<ItemDef>` | Field labels shown to users |
| `<TranslatedText>` in `<Description>` | Inside `<ItemDef>` | Help text / tooltips |
| `<mdsol:ConfirmationMessage>` | Inside `<MetaDataVersion>` | Confirmation messages |
| `<TranslatedText>` in `<CodeListItem>` | Inside `<CodeList>` | Dropdown/codelist display values |
| `<TranslatedText>` in `<StudyEventDef>` | Visit/folder names | Visit display names |

---

## How to Generate/Obtain These Files

### Option 1: Rave Web Services (RWS) API

The primary programmatic way. Call the metadata endpoint:

```
GET https://{rave-url}/RaveWebServices/metadata/studies/{study-oid}/versions/{version-oid}
```

Returns the full ODM XML for a study draft/version. Code lives in `mdsol/RaveEDC` under `Medidata.RaveWebServices`.

### Option 2: Architect Loader Spreadsheet (ALS)

Studies are designed in Excel (the ALS), then uploaded to Rave. The library `mdsol/alslib` (Python) can parse these spreadsheets and extract all translatable text from worksheets.

### Option 3: Rave UI Export

In the Rave EDC UI, export a study draft as ODM XML from the Architect module.

### Option 4: Test Fixtures in Repos

Sample XMLs exist in:
- `mdsol/Rave` → `Medidata.RaveWebServices/features/support/files/`
- `mdsol/Rave` → `Medidata.RaveWebServices/SystemTests/`

---

## Key Repos

| Repo | Language | Purpose |
|------|----------|---------|
| `mdsol/ODMSchema` | XSD | CDISC ODM schemas with Medidata extensions — defines the XML structure |
| `mdsol/RaveEDC` | C# | Main Rave EDC source, contains RWS that generates/consumes ODM XML |
| `mdsol/Rave` | C# | Legacy Rave Classic — has sample study XMLs in test fixtures |
| `mdsol/alslib` | Python | Library for reading Architect Loader Spreadsheets |
| `mdsol/hinterland` | — | MCP server for ALS analysis |
| `mdsol/mdsol-odm-builders` | Python | Programmatic ODM XML creation (archived) |
| `mdsol/RaveEDC_E2E_Test_Data_Seeding` | — | Test data seeding with sample study XMLs |
| `mdsol/olympus` | — | Unifying study builds across Designer and Rave (new) |
| `mdsol/RaveDataService` | C# | Rave EDC SIMT API for internal integrations |

---

## Localization Approach for Hackathon

### The Problem
Study XMLs contain `<TranslatedText xml:lang="en">` elements that need equivalent entries for other languages. Currently this is done manually by study builders.

### Proposed Solution
1. **Parse** the ODM XML and extract all `<TranslatedText>` elements
2. **Identify** which languages are present vs. which are needed
3. **Translate** the English text to target languages (using AI/LLM)
4. **Inject** new `<TranslatedText xml:lang="{target}">` elements back into the XML
5. **Validate** against the ODM schema (`mdsol/ODMSchema`)

### Key Considerations
- The `xml:lang` attribute uses standard language codes (en, ja, fr, de, etc.)
- Each parent element (`<Question>`, `<Description>`, etc.) can have multiple `<TranslatedText>` children — one per language
- OIDs and structural elements should NOT be translated
- Some text is clinical/medical terminology that needs domain-aware translation
- The ODM schema at `mdsol/ODMSchema` (versions 1.2.1, 1.3, 1.3.1) defines valid structure

---

## Next Steps

- [ ] Get a real study XML from a sandbox environment via RWS API
- [ ] Build XML parser to extract translatable strings
- [ ] Integrate with translation service (Bedrock/Claude for AI translation)
- [ ] Build XML writer to inject translations back
- [ ] Validate output against ODM schema
- [ ] Test round-trip: export → translate → re-import to Rave
