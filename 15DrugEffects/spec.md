# Drug Information Portal — Implementation Documentation

## Overview

A React single-page application for searching drugs and viewing side effects, drug classes, similar drugs, and drug-drug interactions. All data comes exclusively from free, public government and open-source APIs — no AI-generated content.

---

## Pages

### About Page (Landing)
- Header with "Drug Information Portal" title and blue border top and bottom
- Sticky table of contents sidebar with anchor links to all sections
- Educational content sections:
  - Credits & Disclaimer
  - About This Tool
  - How to Interpret Adverse Event Data
  - What Drug Labels Actually Tell You
  - Why Some Drugs Have More Reports Than Others
  - Understanding Drug Recall Classifications
  - Drug Pairs with Known Dangerous Interactions
  - Data Sources (table format)
- Call-to-action section with button to navigate to Search page

### Search Page
- Navigation button to return to About page
- Header with title and description
- Search mode toggle (Drug / Symptom)
- Search bar with inline Filter button
- Rx/OTC key legend
- Quick-launch buttons (drugs or symptoms based on mode)
- Symptom results panel (when searching by symptom)
- Cross-drug interaction panel (when 2+ drugs selected)
- Drug cards grid
- Disclaimers footer

---

## Data Sources

| Source | What it provides | Base URL |
|---|---|---|
| **openFDA drug label** | FDA-approved label text: indications, adverse reactions, warnings, contraindications, Rx/OTC status, brand/generic names, route | `https://api.fda.gov/drug/label.json` |
| **openFDA FAERS** | Counts of adverse event reports submitted to the FDA by reaction term (MedDRA) | `https://api.fda.gov/drug/event.json` |
| **RxNorm API** | Canonical drug name, RxCUI identifier, brand name resolution | `https://rxnav.nlm.nih.gov/REST` |
| **RxClass API** | Drug class membership (EPC, MOA, VA, ATC), similar drugs in the same class, all drug classes list | `https://rxnav.nlm.nih.gov/REST/rxclass` |
| **RxNav interaction API** | Drug-drug interaction pairs from DrugBank and ONCHigh | `https://rxnav.nlm.nih.gov/REST/interaction` |

All APIs are free and require no authentication key.

---

## Drug Lookup Pipeline

For each searched drug, the following requests are made in parallel after resolving the RxCUI:

1. `GET /rxcui/{rxcui}/properties.json` — canonical name
2. `GET /rxcui/{rxcui}/related.json?tty=SBD+GPCK+SBG+BN` — brand names
3. `GET /rxclass/class/byRxcui.json?rxcui={rxcui}&relaSource=MEDRT+ATC+VA` — drug classes
4. `GET api.fda.gov/drug/label.json?search=openfda.generic_name:"{name}"` — FDA label
5. `GET api.fda.gov/drug/event.json?search=...&count=patient.reaction.reactionmeddrapt.exact` — FAERS top reactions
6. `GET /interaction/interaction.json?rxcui={rxcui}&sources=DrugBank+ONCHigh` — single-drug interactions

After class data returns, a 7th request fetches class members (similar drugs):

7. `GET /rxclass/classMembers.json?classId={id}&relaSource={src}&ttys=IN` — same-class drugs

### RxCUI Resolution

First tries exact match via `/rxcui.json?name=...&search=1`. Falls back to approximate match via `/approximateTerm.json?term=...` if no exact match is found. Returns `null` if neither resolves — the drug is not added and an error is shown.

---

## Features

### Search Modes

**Drug Search**
- Free-text input for generic or brand names
- Real-time spelling suggestions from RxNav `/spellingsuggestions.json` (debounced 280ms, minimum 2 characters)
- Quick-launch buttons for 8 common drugs when no drugs are selected
- Duplicate prevention — adding an already-selected drug is a no-op

**Symptom Search**
- Free-text input for symptoms/conditions
- Searches FDA labels for drugs with matching indications
- Quick-launch buttons for 8 common symptoms
- Results panel showing matching drugs with:
  - Generic name and Rx/OTC badge
  - Brand name
  - Indication snippet (truncated to 200 chars)
  - "Add to Compare" button to add drug to the grid

### Filter System

Combined filter button inline with search bar:
- Shows badge with count of active filters
- Turns blue when filters are active
- Dropdown panel contains:

**Prescription Type Filter**
- All (default)
- Rx Only — prescription required
- OTC Only — over-the-counter

**Drug Class Filter**
- Searchable input field
- All drug classes fetched from RxClass API on page load (`/rxclass/allClasses.json?classTypes=EPC`)
- Type to filter classes or scroll through dropdown
- Selected class shown as removable tag
- Clear All Filters button when filters active

### Rx/OTC Key Legend
- Visual guide below search bar showing:
  - Blue "Rx" badge = Prescription required
  - Green "OTC" badge = Over-the-counter

### Drug Card

Three tabs per drug:

**Overview**
- Canonical name (from RxNorm), brand names (from related terms), Rx/OTC status badge, route of administration
- Primary drug class name from RxNorm (preferred: EPC > MOA > VA > first available)
- Full indications & usage text from FDA label (expandable)
- All drug class badges (up to 6, with class type as tooltip)
- Similar drugs list: up to 6 drugs in the same primary class, each with a "+ Add" button

**Side Effects**
- Top 10 adverse event terms from FDA FAERS with relative frequency bar chart and report counts
- Full adverse reactions section from FDA label (expandable)
- Warnings section from FDA label, highlighted in amber (expandable)
- Shows "No side effect data" when no data available

**Interactions**
- Contraindications from FDA label (expandable)
- Up to 10 known interaction pairs from RxNav (DrugBank + ONCHigh), each showing interacting drug names, severity (if available), and description

### Cross-Drug Interaction Panel

- Appears automatically when 2 or more drugs are selected
- Calls `GET /interaction/list.json?rxcuis={rxcui1}+{rxcui2}+...` with all selected RxCUIs
- Displays all returned interaction pairs with drug names, severity badge, description, and source label
- Shows "No interactions found" if the RxNav database returns nothing for the combination
- Manual refresh button
- Re-runs automatically when the selected drug set changes

---

## RxNav Interaction API Note

The RxNav drug-drug interaction *UI tab* was removed in January 2024. The underlying REST API endpoints remain active and are used here:

- `GET /REST/interaction/interaction.json` — interactions for a single RxCUI
- `GET /REST/interaction/list.json` — interactions across a list of RxCUIs

Sources: **DrugBank** (non-commercial, educational use) and **ONCHigh** (expert-curated high-priority interactions, updated via CredibleMeds for TdP risk). Neither source carries severity ratings in the non-commercial tier; severity is shown when present.

---

## State

| Variable | Type | Description |
|---|---|---|
| `currentPage` | string | 'about' or 'search' |
| `query` | string | Current search input |
| `searchMode` | string | 'drug' or 'symptom' |
| `selectedDrugs` | object | Map of key → drug data, loading state, error |
| `suggestions` | string[] | Current RxNav autocomplete results |
| `symptomResults` | array | Drugs matching symptom search |
| `symptomQuery` | string | Current symptom being searched |
| `rxFilter` | string | 'all', 'rx', or 'otc' |
| `classFilter` | string | 'all' or selected class name |
| `allDrugClasses` | string[] | All EPC drug classes from RxClass API |
| `classSearchQuery` | string | Search input for class filter |
| `showFilters` | boolean | Filter dropdown visibility |
| `showClassDropdown` | boolean | Class search dropdown visibility |
| `crossInteractions` | array | Interactions between selected drugs |

---

## UI Components

| Component | Description |
|---|---|
| `AboutPage` | Landing page with educational content and TOC |
| `DrugCard` | Tabbed card displaying single drug info |
| `Expandable` | Collapsible content section |
| `SymptomResultsPanel` | Grid of drugs matching symptom search |
| `CrossInteractionPanel` | Drug-drug interactions between selected drugs |

---

## Disclaimers

- FAERS report counts reflect submitted reports, not incidence rates. Common drugs appear more often. No causal relationship is implied.
- FDA label text is as submitted by manufacturers and may not reflect the most current approved labeling.
- RxNav interaction data is for educational and research purposes (DrugBank non-commercial license).
- This product uses publicly available data from the U.S. National Library of Medicine.
- Not a substitute for professional medical advice.
