# JiBe ERP Interactive Training Simulator — Technical Details & Architecture

This document contains in-depth technical documentation covering the design system, DOM architecture, dynamic user identity engine, button lifecycle state machine, cross-page client persistence, and modal dialog subsystems for the **JiBe ERP Look-Alike Training Simulator**.

---

## 1. Design System & Sampled Color Tokens

All colors are sampled directly from Seaspan JiBe ERP production screens:

```css
:root {
  /* Navigation & Brand */
  --jibe-nav-bg: #343957;           /* Deep maritime navy (Top navbar background) */
  --jibe-nav-text: #c5cbe3;         /* Subtle slate/periwinkle nav link text */
  --jibe-nav-hover: #409dd6;        /* Bright ocean blue (Navbar item active/hover) */
  --jibe-nav-active: #23273c;       /* Dark charcoal slate (Active tab background) */
  --jibe-blue: #409dd6;             /* Primary JiBe accent blue */
  --jibe-dropdown-hover: #3f9dda;   /* Dropdown menu item active hover blue */

  /* Surface & Borders */
  --jibe-body-bg: #eff3f6;          /* Soft light ERP body background */
  --jibe-card-bg: #ffffff;          /* Pure white card backgrounds */
  --jibe-border: #e2e8f0;           /* Global subtle border */
  --jibe-card-border: #e1e7ec;      /* Card container border */

  /* Typography Colors */
  --jibe-text-dark: #1e293b;        /* Header titles & dark text */
  --jibe-text-body: #334155;        /* Standard body table text */
  --jibe-text-muted: #64748b;       /* Muted table labels & metadata */

  /* Status & Badges */
  --jibe-red-badge: #ef6a6b;        /* Overdue dates & critical findings pill badge */
  --jibe-green: #10b981;            /* JiBe leaf dot logo & Active status dot */
  --seaspan-red: #cc0000;           /* Seaspan brand logo red */

  /* Buttons & Interactive States */
  --btn-primary-bg: #1a73e8;        /* Standard active blue button */
  --btn-primary-hover: #1557b0;     /* Hover state for primary blue button */
  --btn-disabled-bg: #dbe3ec;       /* Disabled button background */
  --btn-disabled-text: #8c9ba5;     /* Disabled button label color */
  --btn-success-bg: #10b981;        /* Success actions */
}
```

---

## 2. Dynamic User Identity & Onboarding Engine

The simulator personalizes the training experience by allowing users to specify their identity upon arrival, which propagates reactively across every screen.

### A. Onboarding Modal Prompt ("What should we call you?")
- Triggered automatically on `page1.html` whenever `localStorage.getItem('jibe_username')` is unset or blank.
- Modal markup is structured with high-z-index backdrop (`.username-modal-backdrop`), centered card (`.username-modal-card`), title, person avatar SVG icon, input field, and `Proceed` button (`.username-proceed-btn`).
- Pressing `Enter` in `#username-input` or clicking `#username-proceed-btn` commits the user's name.

```javascript
function applyUsername(name) {
  const finalName = name && name.trim() ? name.trim() : 'Ivan Xavier';
  const initials = getInitials(finalName);
  
  localStorage.setItem('jibe_username', finalName);
  localStorage.setItem('jibe_user_initials', initials);
  
  document.querySelectorAll('.jibe-display-username').forEach(el => {
    el.textContent = finalName;
  });
  document.querySelectorAll('.jibe-display-initials').forEach(el => {
    el.textContent = initials;
  });
}
```

### B. Initials Derivation Algorithm
```javascript
function getInitials(name) {
  if (!name || !name.trim()) return 'IX';
  const parts = name.trim().split(/\s+/).filter(Boolean);
  if (parts.length === 1) {
    return parts[0].slice(0, 2).toUpperCase();
  }
  return (parts[0][0] + parts[parts.length - 1][0]).toUpperCase();
}
```

### C. Site-Wide Dynamic Updates
- **`page1.html`:** Top-right profile header (`.jibe-display-username`). Clicking this element re-opens the prompt with the existing name pre-filled for easy editing.
- **`page2.html`:** Top-right profile header, LMS table `Created By` text (cell 5), and avatar pill badges (cell 8). When a list is saved, the newly prepended row displays the dynamic username and initials.
- **`page3.html`:** Top-right profile header, Feed & Discussions input box avatar, pre-existing discussion cards, and dynamically generated Requisition Created feed entries.

---

## 3. Button Lifecycle & Dirty-State Tracking Machine

A central feature of the simulator is the strict enforcement of JiBe's business button state machine on [`page3.html`](file:///e:/Jibe/source/pages/page3.html):

```mermaid
stateDiagram-v2
    [*] --> InitialState: Arrive on page3.html
    
    state InitialState {
        SaveList: Disabled (#dbe3ec)
        CreateRequisition: Disabled (#dbe3ec)
    }
    
    InitialState --> DirtyState: User edits Remarks, Tag, Checklist, or Uploads File
    state DirtyState {
        SaveList: Enabled (#1a73e8)
        CreateRequisition: Disabled (0 items)
    }
    
    DirtyState --> SavedState: Click "Save List"
    state SavedState {
        SaveList: Disabled (#dbe3ec)
        CreateRequisition: Disabled (0 items)
    }
    
    SavedState --> DirtyState: Further edits made
    
    SavedState --> ItemsAddedState: User returns from page4.html with 3 Items
    state ItemsAddedState {
        SaveList: Disabled (or Enabled if unsaved)
        CreateRequisition: Enabled (#1a73e8)
    }
    
    ItemsAddedState --> RequisitionConfirmed: Click Create Req -> Fill -> Confirm
    state RequisitionConfirmed {
        SaveList: Permanently Locked / Disabled
        CreateRequisition: Permanently Locked / Disabled
        Status: Completed
    }
```

### Implementation Details:
1. **`setSaveListEnabled(bool)`**:
   Toggles CSS classes `p3-btn-disabled` / `p3-btn-enabled` and inline background styles (`#dbe3ec` vs `#1a73e8`).
2. **`setCreateReqEnabled(bool)`**:
   Evaluates `itemsList.length > 0` and `requisitionCreated === false`.
3. **Dirty Event Listeners**:
   - Tags select/clear: triggers `markDirty()`.
   - Remarks `input` / `change`: triggers `markDirty()`.
   - Checklist item addition/editing: triggers `markDirty()`.
   - File upload modal confirmation: triggers `markDirty()`.
4. **Requisition Confirmation Lock**:
   Sets `localStorage.setItem('lms_requisition_created', 'true')` and invokes `lockBothButtonsPermanently()`.

---

## 4. Cross-Page State Persistence & Discard Logic

### Storage Keys & Schema:
- `jibe_username` *(string)*: Training user's display name.
- `jibe_user_initials` *(string)*: Two-letter uppercase initials.
- `lms_vessel` *(string)*: Selected vessel (`SESP Demo Vessel`).
- `lms_list_name` *(string)*: User's list title.
- `lms_urgency` *(string)*: Priority rating (`D-Regular Spares/Stores`).
- `lms_department` *(string)*: Owning department (`Engine`).
- `lms_creation_date` *(string)*: Timestamp (`02-10-2026`).
- `lms_items_list` *(JSON array)*: Added item objects with id, code, description, quantity, and unit.
- `lms_marpol_tag` *(string)*: Selected regulatory tag (`MARPOL`).
- `lms_remarks` *(string)*: Operational comments.
- `lms_saved_list` *(boolean flag)*: Triggers LMS table row prepending on `page2.html`.
- `lms_requisition_created` *(boolean flag)*: Marks workflow completion.

### Automatic Draft Discard Rule:
When navigating directly back to the landing page (`page1.html`), in-progress draft list data must be reset to avoid stale training sessions:

```javascript
function clearDraftData() {
  const keysToDiscard = [
    'lms_draft_active',
    'lms_vessel',
    'lms_list_name',
    'lms_urgency',
    'lms_department',
    'lms_creation_date',
    'lms_marpol_tag',
    'lms_remarks',
    'lms_items_list',
    'lms_saved_list',
    'lms_requisition_created'
  ];
  keysToDiscard.forEach(k => localStorage.removeItem(k));
  // Note: jibe_username and jibe_user_initials are user identity and are NEVER discarded!
}
```

---

## 5. Modal Dialogs & Subsystems

### A. Ports Selection Modal (14,664 Records)
- Hosted in [`page3.html`](file:///e:/Jibe/source/pages/page3.html).
- Features live debounced filtering across international ports.
- Row click highlights the port (`Pasir Panjang | Singapore`) and enables the **Confirm** button.

### B. Delivery Date Calendar Popover
- Renders October 2026 calendar grid matching JiBe production datepicker.
- Selecting day `2` populates `02-10-2026` and smoothly closes the popover.

### C. Two-Stage Requisition Creation Flow
1. **Requisition Modal:**
   Validates Port Call, Purchase Type (`High Value Stores`), Delivery Port, Delivery Date, and Subject (`Chemicals for deck and eng`). Save button illuminates solid blue.
2. **Confirmation Popup:**
   Displays *"Are you sure you want to create a requisition?"* with Cancel and Confirm buttons.
3. **New Requisition Success Dialog:**
   Replaces confirmation popup with cyan message:
   `Requisition SPDM-60037/26 has been created successfully`.
   Synchronizes status to `Completed` on both `page3.html` and `page2.html`.

---

## 6. Catalogue Browsing & Segment Policy Isolation (`page4.html`)

- **Catalogue Segment Switching:** Supports `Chemicals` and `Services`.
- **Catalogue Items:** Traverses category tree (*A&T Upgrades > EnviroLogic HF HP 32*, *Chemicals > Test UAT*, *Demo Catalogue > DCSI003*).
- **Segment Restriction Policy:**
  JiBe strictly isolates incompatible segments. Attempting to add Bridge services under `SESP Demo Vessel` when chemical items already exist triggers the **Segment Policy Error Popup Banner**:
  *"Adding this item violates the allowed segments policy and is therefore restricted."*
- **Back to List:** Commits all items to `lms_items_list` and navigates back to `page3.html`.
