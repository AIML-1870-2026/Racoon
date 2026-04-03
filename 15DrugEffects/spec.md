# Drug Information Website — Specification

## Overview

A React single-page application for searching drugs and viewing side effects, drug classes, similar drugs, and drug-drug interactions. All data comes exclusively from free, public government and open-source APIs — no AI-generated content.

---

## Data Sources

| Source | What it provides | Base URL |
|---|---|---|
| **openFDA drug label** | FDA-approved label text: indications, adverse reactions, warnings, contraindications, Rx/OTC status, brand/generic names, route | `https://api.fda.gov/drug/label.json` |
| **openFDA FAERS** | Counts of adverse event reports submitted to the FDA by reaction term (MedDRA) | `https://api.fda.gov/drug/event.json` |
| **RxNorm API** | Canonical drug name, RxCUI identifier, brand name resolution | `https://rxnav.nlm.nih.gov/REST` |
| **RxClass API** | Drug class membership (EPC, MOA, VA, ATC), similar drugs in the same class | `https://rxnav.nlm.nih.gov/REST/rxclass` |
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

### Drug Search

- Free-text input for generic or brand names
- Real-time spelling suggestions from RxNav `/spellingsuggestions.json` (debounced 280ms, minimum 2 characters)
- Quick-launch buttons for 8 common drugs when no drugs are selected
- Duplicate prevention — adding an already-selected drug is a no-op

### Drug Card

Three tabs per drug:

**Overview**
- Canonical name (from RxNorm), brand names (from related terms), Rx/OTC status badge, route of administration
- Primary drug class name from RxNorm (preferred: EPC > MOA > VA > first available)
- Full indications & usage text from FDA label (expandable)
- All drug class badges (up to 6, with class type as tooltip)
- Similar drugs list: up to 6 drugs in the same primary class, each with a "+ Add" button

**Side effects**
- Top 10 adverse event terms from FDA FAERS with relative frequency bar chart and report counts
- Full adverse reactions section from FDA label (expandable)
- Warnings section from FDA label, highlighted in amber (expandable)

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
| `query` | string | Current search input |
| `selected` | string[] | Lowercase drug keys currently selected |
| `drugInfos` | object | Map of key → resolved drug data |
| `loading` | object | Map of key → boolean |
| `errors` | object | Map of key → error message string |
| `suggestions` | string[] | Current RxNav autocomplete results |

---

## Disclaimers

- FAERS report counts reflect submitted reports, not incidence rates. Common drugs appear more often. No causal relationship is implied.
- FDA label text is as submitted by manufacturers and may not reflect the most current approved labeling.
- RxNav interaction data is for educational and research purposes (DrugBank non-commercial license).
- This product uses publicly available data from the U.S. National Library of Medicine.
- Not a substitute for professional medical advice.
