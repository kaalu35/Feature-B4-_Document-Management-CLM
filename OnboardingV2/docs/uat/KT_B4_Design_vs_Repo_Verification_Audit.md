# KT Onboarding Enterprise - B4 Document Management & CLM
# Design vs. Repo Verification Audit

Audit date: 2026-05-26

Scope: Repo metadata and source verification against the B4 design and tracker checklist. This is an audit-only report; no fixes were applied.

## SECTION 1: CUSTOM OBJECTS

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| 8 B4 object folders | All 8 objects with `.object-meta.xml` | PRESENT | All required object metadata folders exist. |
| `KT_Document_Request__c` | Required 10 fields and listed picklist values | PRESENT | All expected fields, types, and status/direction values present. |
| `KT_DocumentTemplate__c` | Required 13 fields | PARTIAL | `Template_Binary__c` exists as `Text(18)`, not Lookup to `ContentDocument`. Other expected fields present. |
| `KT_DocumentVault__c` | Required 14 fields | PRESENT | All expected fields present. `Is_Expired__c` is a formula checkbox. Status has expected values plus extra lifecycle values. |
| `KT_Signature_Request__c` | Required 11 fields | PARTIAL | `Completed_Signers__c` exists as Number, not Roll-Up Summary. Other expected fields present. |
| `KT_Signer__c` | Required 13 fields | PRESENT | All expected fields present, including geolocation and signer status values. |
| `KT_Signature_Audit__c` | Required 10 fields | PRESENT | All expected audit fields present. |
| `KT_OCR_Job__c` | Required 13 fields | PRESENT | All expected OCR job fields present. |
| `KT_Bulk_Document_Job__c` | Required 10 fields | PRESENT | All expected bulk job fields present. |

## SECTION 2: CUSTOM METADATA TYPES

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| 4 CMT definitions | Document template, OCR mapping/provider, signature config | PRESENT | All 4 CMT object definitions exist. |
| `KT_Document_Template_Config__mdt` fields | Includes `Requires_OCR__c`, `Document_Direction__c` | PRESENT | Design gap for `Requires_OCR__c` and `Document_Direction__c` is resolved in repo. |
| `KT_OCR_FieldMapping__mdt` fields | 7 expected fields | PARTIAL | `Transform_Function__c` is missing. |
| `KT_OCR_Provider_Config__mdt` fields | 6 expected fields | PRESENT | All expected fields present. |
| `KT_Signature_Config__mdt` fields | Includes `Require_Trusted_Identity__c` | PARTIAL | `Require_Trusted_Identity__c` is missing. |
| OCR provider CMT records | AWS Textract and Azure records | PRESENT | Both provider records exist. |
| Signature config CMT records | At least 1 record per domain | MISSING | No `KT_Signature_Config__mdt.*.md-meta.xml` records found. |

## SECTION 3: NAMED CREDENTIALS

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| `KT_DocuSign_NC` | OAuth 2.0 JWT Bearer | PARTIAL | File exists, but metadata is `NoAuthentication`. |
| `KT_AdobeSign_NC` | OAuth 2.0 Auth Code | PARTIAL | File exists, but metadata is `NoAuthentication`. |
| `KT_AWS_Textract_NC` | AWS Signature V4 | PARTIAL | File exists, but metadata is `NoAuthentication`. |
| `KT_Azure_FormRecognizer_NC` | Named Principal/API key | PARTIAL | File exists, but metadata is `NoAuthentication`; endpoint is placeholder-style. |
| `KT_VirusScan_NC` | Named Principal/API key | PARTIAL | File exists, but metadata is `NoAuthentication`; endpoint is placeholder-style. |

## SECTION 4: APEX CLASSES

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| Document generation/template/checklist services | Required methods and tests | PRESENT | Core services exist with expected methods and test coverage. |
| E-signature orchestrator/service | Workflow, delegation, expiry, audit/security logic | PRESENT | Required methods and test classes found. |
| Virus scan service | Callout via `KT_VirusScan_NC`, quarantine, pre-DML scan | PRESENT | Upload service scans before `ContentVersion` insert. |
| OCR ingestion/polling/callback | Required methods, queueable polling, HMAC callback | PRESENT | Polling is Queueable with 12 retry default; callback validates HMAC. |
| DocuSign envelope service | Envelope methods plus B12 LicenseManager gate | PARTIAL | Required methods exist, but no B12 `LicenseManager` feature gate found. |
| DocuSign webhook | REST URL, HMAC, envelope correlation | PRESENT | Expected REST mapping and correlation found. |
| Adobe Sign service/webhook | Agreement methods and REST webhook | PRESENT | Required classes/methods exist. |
| Audit export service | Multi-document PDF assembly and certification footer | PARTIAL | Export/certification logic exists, but true multi-document PDF assembly was not confirmed. |
| Bulk document batch | Batchable, AllowsCallouts, batch size 25 | PRESENT | `BATCH_SIZE = 25`; implements required interfaces. |
| Bulk queue/job service | Queueable and Aura status APIs | PARTIAL | Classes exist, but exact corresponding test classes are missing for some services. |
| MCP Apex handlers | 4 Apex handler classes | MISSING | Apex classes `KT_MCPDocumentRequestHandler`, `KT_MCPDocumentVerifyHandler`, `KT_MCPVaultHandler`, `KT_MCPSigningHandler` not found. |
| `KT_AXLDocPayloadBuilder` | JSON payload builder and one-time deeplink | MISSING | Requested class not found; related AXL signing tile service exists but is not the specified artefact. |
| Sharing model/tests | With sharing and test class per Apex class | PARTIAL | Main classes use `with sharing`; missing classes/tests prevent full compliance. |

## SECTION 5: LIGHTNING WEB COMPONENTS

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| 9 LWC folders | `.js`, `.html`, `.js-meta.xml` | PRESENT | All expected LWC folders and files exist. |
| `ktSignaturePad` | Canvas, session token, legal checkbox, no `innerHTML` | PRESENT | All requested checks pass. |
| `ktDocumentVault` | PDF.js and version history panel | PARTIAL | Version history exists; PDF preview uses iframe/shepherd URL, not PDF.js. |
| `ktOCRReview` | Confidence colors and Accept All High Confidence | PRESENT | Green/amber/red logic and accept handler present. |

## SECTION 6: SALESFORCE FLOWS

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| `KT_Document_Request_Flow` | Record-triggered on `KT_Onboarding__c` | PRESENT | Flow exists and triggers on `KT_Onboarding__c`. |
| `KT_DocumentVault_StatusSync` | Record-triggered on `KT_Signer__c` update | PRESENT | Flow exists and triggers on `KT_Signer__c`. |
| `KT_Document_Uploaded_Handler` | Platform event flow using `KT_Document_Uploaded__e` and `Requires_OCR__c` | PARTIAL | Exact flow name missing; equivalent `KT_OCR_Upload_Pipeline` exists for `KT_Document_Uploaded__e`. |
| Scheduled flows | 4 daily scheduled flows, `processType=ScheduledFlow` | PARTIAL | All 4 exist and are scheduled daily, but metadata shows `AutoLaunchedFlow` with schedule trigger. |
| `KT_Document_Generation_Wizard` | Screen flow with 5 screens | PRESENT | Exists with 5 screens. |
| `KT_Bulk_Send_Wizard` | Screen flow with 5 screens | PRESENT | Exists with 5 screens. |
| `KT_Document_Upload_Portal_Flow` | Onboardee guided upload flow | MISSING | Exact flow file not found. |

## SECTION 7: PLATFORM EVENTS

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| `KT_Document_Uploaded__e` | 5 payload fields | PRESENT | All expected fields present; extra `VaultId__c` also exists. |
| `KT_Document_Generated__e` | 5 payload fields | PRESENT | All expected fields present. |
| `KT_Signature_Complete__e` | 5 payload fields | PRESENT | All expected fields present. |
| `KT_OCR_Job_Complete__e` | 5 payload fields | PRESENT | All expected fields present. |

## SECTION 8: PERMISSION SETS

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| `KT_Admin` | CRUD on all 8 B4 objects | PARTIAL | All objects included, but Signature Audit is not full CRUD. |
| `KT_HR_User` | Create/Read/Edit, no delete on request/vault/signer | PRESENT | Required access present. |
| `KT_Compliance_Officer` | Read-only on all B4 objects | PRESENT | Required access present. |
| `KT_Manager` | Read on vault and request | PRESENT | Required access present. |
| `KT_Onboardee` | Limited read on request/vault | PRESENT | Required object access present. |

## SECTION 9: STATIC RESOURCES

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| PDF.js static resource | Used by `ktDocumentVault` | MISSING | No PDF.js static resource found. |

## SECTION 10: TRACKER CROSS-CHECK

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| Sprint 4 core metadata/services/UI | Object model, generation, upload, vault, flows | PARTIAL | Most artefacts exist; gaps remain around template binary lookup, upload portal flow, PDF.js preview, and some scheduled flow metadata expectations. |
| Sprint 6 OCR/signature integrations | OCR, DocuSign, Adobe, audit export | PARTIAL | Code exists for most services; named credentials are metadata placeholders, DocuSign lacks B12 gate, signature CMT records missing. |
| Sprint 7 MCP/Agent/AXL | MCP handlers and AXL payload builder | PARTIAL | Node MCP tooling exists, but required Apex MCP handlers and `KT_AXLDocPayloadBuilder` are missing. |
| Sprint 8 hardening/UAT/security | Regression, security, deploy readiness | PARTIAL | Repo has broad coverage, but unresolved auth, static resource, CMT, and missing-class gaps remain. |

## SECTION 11: KNOWN DESIGN GAPS

| Artefact | Expected | Status | Delta / Notes |
|---|---|---|---|
| GAP 1: `Requires_OCR__c` | Field exists if flow references it | PRESENT | Repo has `KT_Document_Template_Config__mdt.Requires_OCR__c`. No critical missing-field failure here. |
| GAP 2: `Document_Direction__c` | Field exists and service references it | PRESENT | Repo has field and checklist service references document direction. |
| GAP 3: `Require_Trusted_Identity__c` | Field exists if orchestrator references it | MISSING | Field is missing from `KT_Signature_Config__mdt`; this is a design/repo gap. |

## SUMMARY

| Category | Total Expected | Present | Partial | Missing | Gap % |
|---|---:|---:|---:|---:|---:|
| Custom Objects | 8 | 6 | 2 | 0 | 25% |
| Custom Metadata | 7 | 4 | 2 | 1 | 43% |
| Named Credentials | 5 | 0 | 5 | 0 | 100% |
| Apex Classes | 22 | 13 | 4 | 5 | 41% |
| LWCs | 9 | 8 | 1 | 0 | 11% |
| Flows | 10 | 4 | 5 | 1 | 60% |
| Platform Events | 4 | 4 | 0 | 0 | 0% |
| Permission Sets | 5 | 4 | 1 | 0 | 20% |
| Static Resources | 1 | 0 | 0 | 1 | 100% |

## CRITICAL ISSUES

1. `KT_Signature_Config__mdt.Require_Trusted_Identity__c` is missing while trusted identity behavior is part of the design.
2. Named Credentials exist but are configured as `NoAuthentication`, so live external callouts are not metadata-complete.
3. Required Apex MCP handler classes are missing.
4. `KT_AXLDocPayloadBuilder.cls` is missing.
5. `KT_Document_Upload_Portal_Flow` is missing.
6. PDF.js static resource is missing while the design expects PDF.js-based vault preview.
7. `KT_DocuSignEnvelopeService` does not show the required B12 LicenseManager feature gate check.
