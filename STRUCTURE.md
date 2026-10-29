# JiBe ERP Interactive Training Simulator — Project Structure

This document outlines the project directory tree, file roles, page architecture, and module responsibilities for the **JiBe ERP Look-Alike Training Simulator** ("How to Create and Submit a Purchase Order / Requisition").

---

## 1. Project Directory Tree

```text
e:\Jibe\
├── AGENT.md                                 # Master operational manual & AI developer guidelines
├── STRUCTURE.md                             # This file: Project directory & file role mapping
├── TECHNICAL_DETAILS.md                     # Technical architecture: State machines, styling & components
├── project flow chart.md                    # Detailed step-by-step Mermaid diagrams & sequence table
├── project_flow_chart.md                    # Exact synchronized copy of the project flow chart
├── JiBe_Procurement_Order_Creation_Guide.md # Standard Operating Procedure (SOP) & business rules
├── How_to_Create_a_Procurement_Order.txt    # Plain-text quick reference summary
├── How to Create PO.pdf                     # Ground-truth source reference document (21 pages)
├── How to Create a Procurement Order.docx   # Ground-truth Word document reference
├── JiBe_Procurement_-_Close_PO.mp4          # Video reference for procurement closing workflow
├── mermaid-diagram_HowToCreatePO_Jibe.png   # Workflow infographic diagram
│
└── source\
    └── pages\
        ├── common.css                       # Central stylesheet: JiBe ERP design system, tokens & components (86KB)
        ├── page1.html                       # Phase 1: Landing Page Dashboard & Onboarding Prompt Modal
        ├── page2.html                       # Phase 2: LMS Lists Table, New List Creation Modal & User Identity Display
        ├── page3.html                       # Phase 3, 6, 7, 8: List Details, Items Table, Machinery Section, Requisition Modals
        └── page4.html                       # Phase 5 & 6: Catalogues (Chemicals / Services) & Segment Isolation Policy
```

---

## 2. Core Modules in `source/pages/`

| File | Primary Role | Key Features & Implementations |
| :--- | :--- | :--- |
| **[`common.css`](file:///e:/Jibe/source/pages/common.css)** | Global Design System & Styling | CSS custom properties, typography (Roboto/Inter), JiBe top navbar (`#343957`), 3-column dashboard widgets, LMS data tables, custom scrollbars, modal overlays (`.username-modal-backdrop`, `.lms-modal-backdrop`, `.req-modal-backdrop`), button states (`#dbe3ec` disabled vs `#1a73e8` active), animations and toast notifications. |
| **[`page1.html`](file:///e:/Jibe/source/pages/page1.html)** | Landing Page & Onboarding | • **"What should we call you?"** onboarding modal with user icon, input, and `Proceed` button.<br>• Dynamic user identity capture (`jibe_username`, `jibe_user_initials`) replacing "Ivan Xavier" and "IX".<br>• Profile click-to-edit feature in top-right header.<br>• Full JiBe maritime dashboard metrics, pending tasks, overdue findings, port calls.<br>• Top navigation with interactive Procurement &rarr; Lists dropdown routing to `page2.html`.<br>• Automatic draft discard routine for in-progress list drafts (protects user identity). |
| **[`page2.html`](file:///e:/Jibe/source/pages/page2.html)** | LMS Table & List Creation | • Full LMS Lists table grid with realistic data rows.<br>• Dynamic `Created By` column and avatar badges displaying custom username.<br>• Prepended dynamic row when a list is saved from `page3.html` or `page4.html`.<br>• **+ Create New** button opening list creation modal.<br>• Vessel selector (`SESP Demo Vessel`), List Name input, Urgency (`D-Regular Spares/Stores`), Department (`Engine`).<br>• Dynamic cross-page variable persistence into `localStorage`. |
| **[`page3.html`](file:///e:/Jibe/source/pages/page3.html)** | List Details & Requisition Engine | • Dynamic header title reflecting user's list name.<br>• **Button Lifecycle State Machine:** `Save List` disabled by default, enables on edits, disables on save; `Create Requisition` disabled until items added; both permanently locked on requisition confirmation.<br>• General Information card with 2,500-char remarks editor and MARPOL tag.<br>• Interactive Checklist card (inline item creation, discard/save).<br>• Attachments card with file upload modal dialog.<br>• Left sidebar navigation expanding *Items*, *Services*, *Machinery*.<br>• Items table reflecting line items added from `page4.html`.<br>• Permanent Machinery section embedded under Services.<br>• **Create Requisition Modal:** Port call dropdown, Purchase Type menu, searchable Delivery Port modal (14,664 ports), delivery date popover, subject input.<br>• Two-stage confirmation dialog and cyan success dialog (`Requisition SPDM-60037/26 created successfully`).<br>• Dynamic Discussion Feed logging requisition creation with active user avatar and username. |
| **[`page4.html`](file:///e:/Jibe/source/pages/page4.html)** | Catalogues & Browsing | • Dynamic subheader displaying active list name from Page 2.<br>• Segment selector (`Chemicals` vs `Services`).<br>• Hierarchical category tree navigation.<br>• Item drawer expansion with specifications and **Add to list** action.<br>• Adds Chemicals items (*EnviroLogic HF HP 32*, *Test UAT*, *DCSI003*) with blue feedback toast.<br>• **Segment Policy Isolation Check:** Bridge services under `SESP Demo Vessel` trigger the policy violation banner when attempting to mix with chemicals.<br>• **Back to List** button persisting items and returning to `page3.html`. |

---

## 3. Cross-Page Variable Architecture

All client state is synchronized across pages using browser `localStorage`:

| Storage Key | Variable Role | Source Page | Consumer Pages |
| :--- | :--- | :--- | :--- |
| `jibe_username` | Custom username | `page1.html` (modal prompt) | `page1.html`, `page2.html`, `page3.html` |
| `jibe_user_initials` | Computed 2-letter user initials | `page1.html` (derived) | `page1.html`, `page2.html`, `page3.html` |
| `lms_vessel` | Selected Vessel name | `page2.html` (modal) | `page3.html`, `page4.html` |
| `lms_list_name` | List Name | `page2.html` (modal) | `page2.html`, `page3.html`, `page4.html` |
| `lms_urgency` | Priority / Urgency | `page2.html` (modal) | `page3.html` |
| `lms_department` | Owning department | `page2.html` (modal) | `page3.html` |
| `lms_creation_date` | List creation timestamp | `page2.html` (auto) | `page2.html`, `page3.html` |
| `lms_marpol_tag` | Selected MARPOL tag | `page3.html` (details) | `page2.html`, `page3.html` |
| `lms_remarks` | Internal operational remarks | `page3.html` (editor) | `page2.html`, `page3.html` |
| `lms_items_list` | Serialized JSON array of added items | `page4.html` (catalogue) | `page2.html`, `page3.html` |
| `lms_requisition_created` | Requisition generation status flag | `page3.html` (confirmation) | `page2.html`, `page3.html` |
| `lms_saved_list` | Flag indicating list is saved to LMS | `page3.html` / `page4.html` | `page2.html` |

---

## 4. Architectural Rules & Conventions

1. **Strict File Naming:** All interactive pages MUST be named `page1.html`, `page2.html`, `page3.html`, `page4.html`. Never use alternative extensions like `.index` or subdirectories.
2. **Unified Styling:** All styles, animations, and modal overlays reside in [`common.css`](file:///e:/Jibe/source/pages/common.css).
3. **No External Server Dependencies:** The simulator runs entirely in vanilla HTML, CSS, and modern JavaScript, compatible with standard static file servers or local development servers (`http://localhost:8080`).
4. **Draft Discard vs Identity Persistence:** Draft list keys are purged when returning to `page1.html`, but user identity keys (`jibe_username`, `jibe_user_initials`) remain intact to ensure seamless cross-session user personalization.
