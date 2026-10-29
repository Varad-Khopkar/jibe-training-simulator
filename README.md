# JiBe ERP Training Simulator

[![Simulator Status](https://img.shields.io/badge/Simulator-Interactive%20ERP%20Training-blue.svg)](#)
[![Workflow](https://img.shields.io/badge/Workflow-Procurement%20%26%20Requisition-emerald.svg)](#)
[![Tech Stack](https://img.shields.io/badge/Stack-HTML5%20%7C%20CSS3%20%7C%20Vanilla%20JS-orange.svg)](#)

An interactive, high-fidelity maritime ERP training simulator replicating the **JiBe Marine ERP** web application. Designed specifically to train vessel crew (Captains, Chief Engineers, Officers) and shore-side procurement managers on **"How to Create and Submit a Procurement Requisition / Purchase Order"** in a risk-free training sandbox.

---

## 🌟 Key Features

* **Dynamic User Onboarding:**  
  When trainees open the simulator, a *"What should we call you?"* modal prompts for their name. The custom username and computed initials automatically personalize all top headers, LMS table audit columns (`Created By`), and discussion activity feeds.
* **Realistic LMS List Management:**  
  Interactive table grid displaying active and completed procurement lists with dynamic status badges, list number generation (`LIST SPDM-60028/26`), and vessel assignments (`SESP Demo Vessel`).
* **Strict Button Lifecycle & Dirty-State Tracking:**  
  - **Save List Button:** Disabled by default upon arrival; illuminates solid blue when list details (Tags, Remarks, Checklist, or Attachments) are edited; reverts to disabled once saved.
  - **Create Requisition Button:** Remains disabled until line items are added from the catalogue.
  - **Permanent Locking:** Once a requisition is created and confirmed, both action buttons lock permanently to prevent unauthorized post-submission edits.
* **Catalogue Browsing & Segment Policy Isolation:**  
  Traverse equipment and chemical categories (*EnviroLogic HF HP 32*, *Test UAT*, *DCSI003*). Enforces JiBe's strict segment policy—attempting to mix Services with Chemicals triggers a validation block.
* **Two-Stage Requisition Engine:**  
  - Searchable delivery ports modal indexing **14,664 global ports** (e.g. *Pasir Panjang | Singapore*).
  - Interactive October 2026 calendar popover.
  - Two-stage confirmation dialog followed by the cyan success dialog (`Requisition SPDM-60037/26 created successfully`).

---

## 🚀 Quick Start

No heavy dependencies or build tools required. Run with any standard static HTTP server:

### Using Python:
```bash
# From repository root
python -m http.server 8080
```

### Using Node / npx:
```bash
npx serve .
```

Open your browser at:
```text
http://localhost:8080/source/pages/page1.html
```

---

## 📂 Project Architecture

```text
├── source/
│   └── pages/
│       ├── common.css       # Unified design system & JiBe ERP color tokens
│       ├── page1.html       # Dashboard & User Onboarding Modal
│       ├── page2.html       # LMS Lists Grid & New List Creation Modal
│       ├── page3.html       # List Details, Items Table & Requisition Generator
│       └── page4.html       # Catalogue Browsing & Segment Policy Check
├── AGENT.md                 # Master operational instructions & AI guidelines
├── STRUCTURE.md             # Project structure, file roles & storage schema
├── TECHNICAL_DETAILS.md     # In-depth state machine & technical architecture
├── project flow chart.md    # Detailed Mermaid diagrams & action-by-action steps
└── JiBe_Procurement_Order_Creation_Guide.md # Standard Operating Procedure (SOP)
```

---

## 📖 Complete Documentation

* **[Project Flow Chart](project%20flow%20chart.md):** Complete visual Mermaid diagrams and sequence table for all 8 workflow phases.
* **[Technical Details](TECHNICAL_DETAILS.md):** Design tokens, button state machine, initials algorithm, and modal subsystems.
* **[Procurement Order Creation Guide](JiBe_Procurement_Order_Creation_Guide.md):** Standard Operating Procedure (SOP) covering both vessel and shore-side workflows.
* **[Agent Instructions](AGENT.md):** Rules, constraints, and development guidelines.
