# AGENT INSTRUCTIONS — JiBe ERP Interactive Training Simulator Project

This document serves as the **master operational instruction manual** for any AI agent or software engineer working on this repository. All rules, conventions, business logic, and architectural constraints documented here MUST be followed strictly.

---

## 1. Project Objective & Core Mission

* **Objective:** Build a high-fidelity, functional, and fully interactive look-alike of the JiBe ERP web application designed specifically to train maritime employees on **"How to Create and Submit a Purchase Order (PO) / Requisition"**.
* **Role:** Interactive employee training simulator. It allows trainees to perform real-world actions (fill forms, make dropdown selections, attach files, browse equipment catalogues, handle policy errors, and generate requisitions) in a realistic browser sandbox.
* **Architecture:** 4-page interactive multi-page web application (`page1.html` through `page4.html`) driven by shared design tokens in [`common.css`](file:///e:/Jibe/source/pages/common.css) and client-side state synchronization (`localStorage`).

---

## 2. Strict Scope Boundaries & Restrictions

> [!CAUTION]
> **STRICT SCOPE RESTRICTION:**
> Focus ONLY on the requisition creation workflow: **"How to Create and Submit a Procurement Requisition / Order"**.
>
> **DO NOT IMPLEMENT:**
> * Closing Purchase Orders ("Delivered Onboard") unless requested as future training modules.
> * Receiving Goods or Warehouse Check-ins.
> * Invoicing, Bill of Lading, or Accounts Payable approvals.
> * Third-party vendor quoting portals.

---

## 3. Directory Layout & File Structure

All web simulator pages live in `e:\Jibe\source\pages\`:

```text
e:\Jibe\
├── AGENT.md                                 # This file: Master guidelines & simulator operational manual
├── STRUCTURE.md                             # Project directory structure, page roles & file mapping
├── TECHNICAL_DETAILS.md                     # Technical architecture: State machine, design tokens & UI components
├── project flow chart.md                    # Detailed step-by-step Mermaid diagrams & sequence table
├── project_flow_chart.md                    # Synchronized markdown flow chart copy
├── JiBe_Procurement_Order_Creation_Guide.md # Comprehensive SOP guide with business rules
├── How_to_Create_a_Procurement_Order.txt    # Quick-reference text summary
├── How to Create PO.pdf                     # Ground-truth source reference document (21 pages)
│
└── source\
    └── pages\
        ├── common.css                       # Global design system, theme tokens & component styles
        ├── page1.html                       # Phase 1: Dashboard, Onboarding Prompt ("What should we call you?")
        ├── page2.html                       # Phase 2: LMS Lists Table & "+ Create New" modal
        ├── page3.html                       # Phase 3, 6, 7, 8: List Details, Items Table, Modals & Requisition Dialogs
        └── page4.html                       # Phase 5 & 6: Catalogues, Browse Items (Chemicals) & Services Policy Check
```

### Strict Naming Rules:
* Pages **MUST** be named `pageX.html` (`page1.html`, `page2.html`, `page3.html`, `page4.html`).
* **NEVER** use `.index` extensions or create files like `page1.index`, `page2.index`, or `page1.index.html`.
* **NEVER** create redundant duplicate CSS files per page. Always link to [`common.css`](file:///e:/Jibe/source/pages/common.css).

---

## 4. Master Workflow Phases (From User Prompts)

The simulator implements the end-to-end user journey across 8 logical phases:

### Phase 1: Landing Page & User Onboarding (`page1.html`)
1. **Onboarding Modal Prompt ("What should we call you?"):**
   - Whenever the user visits the site, an onboarding modal dialog prompts: *"What should we call you?"* with a person icon, input field, and `Proceed` button.
   - The user enters their custom username and clicks `Proceed` (or presses Enter).
   - The username is stored in variable `jibe_username` and initials in `jibe_user_initials` (`localStorage`).
   - The username replaces every instance of `"Ivan Xavier"` across all pages (top-right header, LMS table "Created By", discussion feed author), and initials replace `"IX"` in all avatar badges.
   - Clicking the user profile in the top header on `page1.html` re-opens the prompt to edit the username at any time.
2. **Procurement Navigation:**
   - Hover and click **Procurement** on the top navigation bar.
   - Click **Lists** from the dropdown menu to navigate to [`page2.html`](file:///e:/Jibe/source/pages/page2.html).

### Phase 2: LMS Lists Table & New List Creation (`page2.html`)
1. **LMS Table View:**
   - Displays existing table records with list name and status colored `#2f79bb`.
   - Dynamic user profile and table rows display the active username and initials.
2. **List Creation:**
   - Click **+ Create New** button in the top right.
   - Select Vessel: `SESP Demo Vessel` (saved dynamically into variable for cross-page persistence).
   - Enter List Name (saved dynamically into variable for cross-page persistence).
   - Select Urgency: `D-Regular Spares/Stores` (saved dynamically into variable for cross-page persistence).
   - Select Vessel Department: `Engine` (saved dynamically into variable for cross-page persistence).
   - Click **Create** &rarr; Captures creation date (`02-10-2026`) and navigates to [`page3.html`](file:///e:/Jibe/source/pages/page3.html).
3. **Draft Discard Rule:**
   - If the user returns directly to the Landing page (`page1.html`) from here, in-progress draft list data is automatically discarded from client storage. (User identity `jibe_username` is protected and preserved).

### Phase 3: List Details Configuration (`page3.html`)
1. **Initial Button States:**
   - **Save List** button is **DISABLED** by default on initial arrival.
   - **Create Requisition** button is **DISABLED** until items are added.
2. **Metadata & Remarks:**
   - Click **Tags** box under General Information.
   - General Remarks expands rich text editor (2,500-character limit).
   - Select `MARPOL` tag from dropdown &rarr; **Save List** button transitions to **ENABLED**.
3. **Checklist:**
   - Scroll to Checklist Card &rarr; Click **Edit** button. Action bar toggles with "Discard Changes" and "Save".
   - Click **Add checklist item** button &rarr; type item text &rarr; click **Save** (saved dynamically).
   - **Save List** button remains **ENABLED**.
4. **Attachments:**
   - Scroll to Attachments Card &rarr; Click **Attach File** button.
   - Upload modal opens &rarr; Click **Browse** and select local file &rarr; Enter remarks &rarr; Click **Upload**.
   - Modal closes, file appears in table, and **Save List** button remains **ENABLED**.
5. **Saving List Details:**
   - Click **Save List** button (`#btn-save-list`).
   - Green success toast displays: *"ItemList saved successfully."*
   - **Save List** button **REVERTS TO DISABLED STATE** unless further changes are made.

### Phase 4: Left Menu & Navigation to Browse Items
1. Click **Items** in the left sidebar menu.
2. Expands sub-options: *Items*, *Services*, *Machinery*.
3. View switches to Items subdivision on [`page3.html`](file:///e:/Jibe/source/pages/page3.html).
4. Click **Browse Item** button in Items card header &rarr; Navigates to [`page4.html`](file:///e:/Jibe/source/pages/page4.html).

### Phase 5: Catalogues & Adding Chemicals (`page4.html`)
1. Active List Name dynamically reflects the variable from Page 2.
2. Select Segment: Click **Select Segment** dropdown &rarr; select **Chemicals**.
3. Middle tree updates; table displays initial state.
4. **Item 1:** Click `A&T Upgrades` in tree &rarr; click `EnviroLogic HF HP 32` &rarr; click **Add to list** &rarr; blue toast: *"Item Added to List"*.
5. **Item 2:** Click `Chemicals` under Active GL2 in tree &rarr; click `Test UAT` &rarr; click **Add to list** &rarr; blue toast: *"Item Added to List"*.
6. **Item 3:** Click `Demo Catalogue` in tree &rarr; click `DCSI003` &rarr; click **Add to list** &rarr; blue toast: *"Item Added to List"*.
7. Click **Back to List** (upper-left corner) to return to [`page3.html`](file:///e:/Jibe/source/pages/page3.html).
   - Saves this list on [`page2.html`](file:///e:/Jibe/source/pages/page2.html) with 3 items & segment `Chemicals`.

### Phase 6: Services Policy Check & Machinery Verification (`page3.html` & `page4.html`)
1. [`page3.html`](file:///e:/Jibe/source/pages/page3.html) Items table reflects items selected from Page 4 (3 items).
2. **Create Requisition** button **BECOMES ENABLED** now that items are present.
3. Verify **Machinery section is permanently embedded** under Services section.
4. **Optional Services Policy Check:**
   - Click **Browse Services** button &rarr; opens [`page4.html`](file:///e:/Jibe/source/pages/page4.html) in Services mode.
   - Click `Bridge` under `SESP Demo Vessel` &rarr; select a service item &rarr; click **Add to list**.
   - **Segment Policy Error Popup** triggers (Services cannot be mixed with Chemicals).
   - Click **Back to List** to return to [`page3.html`](file:///e:/Jibe/source/pages/page3.html). List remains saved and Create Requisition remains enabled.

### Phase 7: Create Requisition Parameters (`page3.html`)
1. Click **Create Requisition** button (`#btn-create-req`).
2. Requisition modal initializes with Vessel, Urgency, and pre-filled Subject.
3. Select **Port Call** (e.g. `SESP-PC-2026-001 (Singapore)`).
4. Click **Purchase Type \*** &rarr; select `High Value Stores`.
5. Click **Delivery Port \*** &rarr; Ports modal opens (14,664 records).
   - Search `Singapore` &rarr; select `Pasir Panjang` &rarr; click **Confirm**.
   - Modal closes &rarr; field populated with `Pasir Panjang | Singapore`.
6. Click **Delivery Date \*** &rarr; Calendar popover opens (October 2026).
   - Click Day `2` (`02-10-2026`) &rarr; popover closes.
7. Enter/edit Subject: `Chemicals for deck and eng`.
8. Form validates &rarr; **Save** button turns solid blue.

### Phase 8: Confirmation & Requisition Creation Popups (`page3.html`)
1. Click **Save** button (`#btn-req-save`).
2. **Confirmation Dialog** appears: *"Are you sure you want to create a requisition?"* with Cancel and Confirm buttons.
3. Click **Confirm** button.
4. Confirmation dialog vanishes and **New Requisition Dialog** appears.
5. Dialog displays cyan message: *"Requisition SPDM-60037/26 has been created successfully"*.
6. LMS List status transitions to **Completed**; Discussion Feed logs the event with the dynamic username and avatar.
7. **BOTH Create Requisition and Save List buttons lock PERMANENTLY DISABLED**.
8. Click **Close** or **Open Link** button &rarr; Dialogs close with final success toast.

---

## 5. Key State & Business Rules Enforced

1. **Button Lifecycle & Dirty State Management:**
   - **Save List:** Disabled on initial arrival. Becomes enabled whenever the user edits List Details (Remarks, Tags, Checklist, Attachments). Reverts to disabled state upon clicking Save.
   - **Create Requisition:** Disabled on initial arrival. Enables only when items are added from the Items section (`lms_items_list.length > 0`).
   - **Permanent Locking:** Once Requisition is confirmed, both Save List and Create Requisition are permanently locked/disabled.
2. **Automatic Draft Discard Rule:**
   - Direct navigation back to the Landing page (`page1.html`) discards in-progress draft list data.
3. **Dynamic User Identity & Onboarding Prompt:**
   - Onboarding modal prompts: *"What should we call you?"*.
   - Stores custom username and initials in `localStorage`.
   - Replaces all occurrences of "Ivan Xavier" and "IX" across all pages, tables, and feeds.
   - Persists independently of draft list discards.
4. **Segment Policy Isolation:**
   - Items under `Chemicals` cannot be mixed with `Services`. Attempting to add service line items on `page4.html` triggers the Segment Policy Error popup banner.
5. **LMS Table Synchronization:**
   - Saved lists dynamically prepend to the top of the LMS table on [`page2.html`](file:///e:/Jibe/source/pages/page2.html) with List Number `LIST-SPDM-60028/26`, item count, segment, and status badge.
