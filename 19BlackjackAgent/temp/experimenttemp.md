# Science Experiment Generator — Product Specification

## Overview

A web-based tool that generates safe, classroom-ready science experiments using the OpenAI API. Teachers and educators input their constraints (grade level, available supplies, topic, time) and receive tailored experiment ideas complete with instructions, learning objectives, and safety notes. Generated experiments are automatically saved to a local library for future reference.

---

## Features

### 1. API Key Input

Users must provide an OpenAI API key to power the generator. Multiple input methods are supported:

**Option A — Paste directly:**
- A masked text input field displaying the key as dots (using CSS text-security)
- Inline validation: check that the key matches the `sk-...` format before allowing submission

**Option B — Upload a file:**
- A modal with drag-and-drop zone accepting `.env` and `.csv` files only
- Parser behavior:
  - `.env` files: extract the value of `OPENAI_API_KEY=...`
  - `.csv` files: scan all cells for a value matching the `sk-...` pattern
- If a valid key is found, auto-populate the paste field and show a success indicator
- If no valid key is found, show a descriptive error popup

**API Key Actions:**
- **Test Key** button: Validates the API key against OpenAI's API and shows success/error feedback
- **Upload API Key** button: Opens the file upload modal
- **Clear Key** button: Clears the current API key from the input field

**Security notes:**
- The API key is stored only in memory (never written to localStorage, cookies, or any backend)
- Display a small disclaimer: *"Your API key is used only in your browser session and is never stored or transmitted to our servers."*

---

### 2. Model Selector

A dropdown menu to choose which OpenAI model powers the generation.

**Options to include:**
- `gpt-4o` *(recommended, default)*
- `gpt-4o-mini`
- `gpt-4-turbo`
- `gpt-3.5-turbo`

**UI details:**
- Label: *"Model"*
- Show a short descriptor next to each option (e.g., "Fast & affordable", "Most capable", "Balanced")
- Default selection: `gpt-4o`
- Hint message below: *"Specify at least one: grade level, topic, or available supplies"*

---

### 3. Grade Level Selector

A control for selecting the target student grade level. This informs experiment complexity, vocabulary, and safety considerations.

**Options:**
- K–2 (Ages 5–8)
- 3–5 (Ages 8–11)
- 6–8 (Ages 11–14)
- 9–12 (Ages 14–18)
- University (Ages 18+)

**UI details:**
- Rendered as a segmented button group or large toggle buttons (not a dropdown)
- Only one grade band may be selected at a time
- Clicking an already-selected grade level will deselect it
- Default: no selection (optional — user may generate without selecting)

---

### 4. Experiment Time Selector

A dropdown menu for specifying the desired experiment duration.

**Options:**
- No preference (default)
- 15 minutes
- 30 minutes
- 45 minutes
- 1 hour
- 1.5 hours
- 2 hours

**UI details:**
- Label: *"Experiment Time"*
- The selected time is included in the prompt and displayed in the generated experiment below the title

---

### 5. Available Supplies Input

A multi-line text area where the user lists materials they have on hand, with a companion list of common household items.

**UI details:**
- Label: *"Available Supplies"*
- Placeholder text: *"e.g., baking soda, vinegar, plastic cups, balloons, food coloring…"*
- The generator uses this list to constrain experiment suggestions to only what's available
- No strict formatting required — comma-separated, line-separated, or freeform text are all acceptable
- Character limit: 1,000 characters, with a live counter

**Common Items List:**
- Positioned to the left of the supplies textarea
- Contains 24 clickable common household items: Baking soda, Vinegar, Food coloring, Plastic cups, Balloons, Paper towels, Salt, Sugar, Water, Ice, Dish soap, Vegetable oil, Plastic bottles, Straws, Rubber bands, String, Tape, Scissors, Ruler, Magnifying glass, Magnets, Paper clips, Coins, Aluminum foil
- Each item has a "+" icon; clicking adds the item to the supplies textarea
- Duplicate items are not added (case-insensitive check)
- Scrollable list with always-visible scrollbar
- Height matches the textarea height

---

### 6. Topic Autocomplete Search Bar

A search input that lets users narrow experiments to a specific science topic or concept.

**Behavior:**
- As the user types, a dropdown appears showing matching topic suggestions
- Suggestions are drawn from a predefined list of common K–12 science topics (see below)
- Fuzzy matching: partial matches and minor typos should surface relevant results
- User may select a suggestion from the dropdown **or** type a custom topic not on the list
- Only one topic may be active at a time; selecting a new one replaces the previous

**Predefined topic list (non-exhaustive):**

*Life Science:* Photosynthesis, Cell biology, Ecosystems, Food webs, Genetics, Plant growth, Animal adaptation, Human body systems, Microorganisms, Pollination

*Physical Science:* Density, States of matter, Chemical reactions, Magnetism, Electricity, Sound waves, Light & optics, Force & motion, Simple machines, Thermal energy

*Earth & Space Science:* Erosion, Volcanoes, Weather & climate, Rock cycle, Water cycle, Solar system, Phases of the moon, Plate tectonics, Fossils, Soil composition

*Engineering & Design:* Structures & bridges, Buoyancy, Aerodynamics, Insulation, Circuits, Water filtration

---

### 7. Classroom Safety Constraint (Always On)

All generated experiments must be safe for standard classroom use. This is a non-negotiable system-level constraint baked into every API prompt — it is **not** a user-facing toggle.

**What this enforces (via system prompt):**
- No open flames, unless explicitly described as a supervised teacher demonstration
- No toxic, corrosive, or controlled chemicals
- No experiments requiring specialized lab equipment (fume hoods, autoclaves, etc.)
- Age-appropriate complexity and risk level, matched to the selected grade band
- Each experiment output must include a **Safety Notes** section flagging any minor precautions (e.g., "wear safety goggles", "adult supervision recommended for cutting")

---

### 8. Experiment Library

A local storage-based library that automatically saves generated experiments for future reference.

**Features:**
- Experiments are automatically saved when generated
- Library button in the header shows the count of saved experiments
- Modal displays all saved experiments with title, grade level, topic, and date
- **View** button loads a saved experiment into the output panel
- **Delete** button removes an experiment from the library
- Data persists across browser sessions using localStorage

---

### 9. Editable Output

Users can edit the generated experiment directly in the output panel.

**Behavior:**
- **Edit** button appears after an experiment is generated
- Clicking Edit switches to a contenteditable mode where users can modify the formatted HTML directly
- Clicking **Done** saves the changes and returns to view mode
- Edited content maintains the same formatting and styling as the original

---

## Output Format

The model returns a markdown response with a specific section order. The system prompt enforces the following structure:

**Required sections (in this exact order):**
1. Title (engaging experiment name)
2. Time Estimate (displayed as "**Time:** X minutes" below the title)
3. Introduction (brief explanation of the science concept)
4. Learning Objectives (what students will learn)
5. Materials List (uses supplies provided when possible)
6. Safety Notes (any necessary precautions)
7. Procedure (step-by-step instructions as a numbered list)
8. Expected Results (what to observe)
9. Discussion Questions (questions to reinforce learning)
10. Extensions (optional ways to expand the experiment)

**Markdown rendering:**
- The UI parses and renders the model's raw markdown output using a custom markdown renderer
- All standard markdown elements render correctly: `##` headings, `**bold**`, `*italic*`, `-` and `1.` lists (with proper nesting), `>` blockquotes, `` `inline code` ``, and `---` dividers
- Procedure steps appear as numbered lists; sub-items appear as nested bullet points
- The rendered output appears inside a clean, readable panel with edit capability

---

## UI Design

**Layout:**
- Two-column layout on wider screens (≥ 1100px): inputs on left, output on right
- Single-column layout on smaller screens: inputs stack above output
- Output panel is hidden until the first generation completes
- The **Generate Another** button appears only after a result is shown

**Visual Theme:**
- Science-themed molecule SVG background pattern (hexagons, benzene rings, molecular structures)
- Green accent color (#10b981) for interactive elements
- Light red background (#fee2e2) for error popups

**Error Display:**
- Centered popup with red-tinted background
- Auto-dismisses after 2 seconds with fade-out animation
- Dismiss button (×) for manual close

---

## User Flow

1. User opens the app
2. User provides API key (paste or file upload)
3. User optionally tests the API key
4. User selects a model from the dropdown
5. User provides **at least one** of: grade level, topic, or available supplies
6. User optionally selects an experiment time
7. User optionally clicks common items to add to supplies
8. User clicks **"Generate Experiment"**
9. A loading state is shown while the API call is in progress
10. The generated experiment is displayed and automatically saved to the library
11. User may edit the experiment using the Edit button
12. User may click **"Generate Another"** to get a new experiment, or adjust inputs and regenerate
13. User may access saved experiments from the Library button in the header

---

## Error States

| Scenario | Message |
|---|---|
| No API key provided | *"Please enter or upload your OpenAI API key to continue."* |
| Invalid API key format | *"That doesn't look like a valid API key. Keys start with `sk-`."* |
| API key rejected (401) | *"Your API key was not accepted. Please check it and try again."* |
| Missing required input | *"Please provide at least one of: grade level, topic, or available supplies."* |
| Rate limited (429) | *"Too many requests. Please wait a moment and try again."* |
| API timeout / network error | *"Something went wrong. Please check your connection and try again."* |
| File upload — no key found | *"No API key found in that file. Check that it contains `OPENAI_API_KEY=...`."* |
| File upload — wrong type | *"Only `.env` and `.csv` files are supported."* |
| Test key — no key entered | *"Please enter an API key first."* |

---

## Technical Notes

- **Framework:** Vanilla HTML/CSS/JS (single `index.html` file)
- **API calls:** Made directly from the browser to `https://api.openai.com/v1/chat/completions`
- **No backend required**
- **Storage:**
  - API key: in-memory only (never persisted)
  - Experiment library: localStorage
- **Accessibility:** All inputs have visible labels; keyboard navigation works throughout
- **Responsive:** Two-column layout on screens ≥ 1100px; single-column on smaller screens

---

## Implementation Summary

**Completed Features:**
- API key input with masking, test, upload (drag-and-drop modal), and clear functionality
- Model selector with 4 OpenAI model options
- Grade level segmented buttons with toggle/deselect capability
- Experiment time dropdown selector
- Topic autocomplete with fuzzy search
- Available supplies textarea with character counter
- Common household items clickable list (24 items)
- Experiment library with localStorage persistence
- Editable output using contenteditable
- Custom markdown renderer with proper numbered list handling
- Centered error popups with red background and auto-dismiss
- Science-themed molecule SVG background pattern
- Responsive two-column layout
