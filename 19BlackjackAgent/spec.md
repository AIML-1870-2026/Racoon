# Blackjack AI — Product Specification

## Overview

A web-based, single-player Blackjack game powered by an AI agent that provides real-time move suggestions and can optionally play the game autonomously on the user's behalf. Users provide an API key to unlock AI features; the core card game is fully playable without one. The interface uses a casino-green felt aesthetic with card table atmosphere, a clean two-column layout, and real-time feedback from the AI advisor panel.

---

## Features

### 1. API Key Input

Users must provide an OpenAI API key to enable AI suggestions and autopilot mode. Multiple input methods are supported.

**Option A — Paste directly:**
- A masked text input field displaying the key as dots (using CSS `-webkit-text-security: disc`)
- Inline validation: checks that the key matches the `sk-...` format before allowing submission
- Label: *"API Key"*

**Option B — Upload a file:**
- A modal with a drag-and-drop zone accepting `.env` and `.csv` files only
- Upload button uses Apple-style icon (square with arrow pointing up)
- Parser behavior:
  - `.env` files: extract the value of `OPENAI_API_KEY=...`
  - `.csv` files: scan all cells for a value matching the `sk-...` pattern
- If a valid key is found, auto-populate the paste field and display a success indicator
- If no valid key is found, show a descriptive error popup

**API Key Actions:**
- **Upload** button (icon): Opens the file upload modal
- **Test** button: Validates the key against OpenAI's API and shows success/error feedback
- **Clear** button: Clears the current key from the input field

**Security notes:**
- The API key is stored only in memory (never written to localStorage, cookies, or any backend)

---

### 2. Model Selector

A dropdown to choose which OpenAI model powers the AI advisor.

**Options:**
- `gpt-4o` *(recommended, default)*
- `gpt-4o-mini` *(fast)*
- `gpt-4-turbo`
- `gpt-3.5-turbo`

**UI details:**
- Label: *"Model"*
- Located in the header alongside the API key input
- Default selection: `gpt-4o`

---

### 3. Risk Level Selector

A dropdown to configure the AI's playing style, located in the AI Advisor panel.

**Options:**
- **Conservative**: Prioritizes minimizing losses, avoids risky doubles/splits, stands on borderline hands
- **Balanced** *(default)*: Uses standard basic strategy with mathematically optimal plays
- **Aggressive**: Takes calculated risks, doubles/splits more liberally, pushes edges when odds are marginally favorable

**UI details:**
- Label: *"Risk Level"*
- Located at the top of the AI Advisor panel body
- The selected risk level is included in the AI prompt to influence recommendations

---

### 4. Game Configuration Panel

A collapsible settings panel (open by default) where users configure the game before starting.

**Settings:**

| Setting | Options | Default |
|---|---|---|
| Number of Decks | 1, 2, 4, 6, 8 | 6 |
| Starting Balance | $50, $100, $250, $500, $1,000 | $100 |
| Minimum Bet | $1, $5, $10, $25 | $5 |
| Dealer Stands On | Soft 17, Hard 17 | Soft 17 |
| Allow Surrender | Yes / No | Yes |

**UI details:**
- Rendered as a compact card above the game table
- A **"New Game"** button applies settings and resets the table
- Settings are disabled mid-hand; they can only be changed between hands

---

### 5. Betting Interface

Before each hand, the user places a bet using chip buttons.

**Chip denominations:** $1, $5, $10, $25, $100 (shown as colored casino chips)

**UI details:**
- Clicking a chip adds its value to the current bet
- A **"Clear Bet"** button resets the bet to $0
- A **"Deal"** button locks the bet and starts the hand (disabled if bet is $0 or exceeds current balance)
- Current balance and current bet are always visible above the chip row
- If the balance is $0, a **"Rebuy"** button resets balance to the starting amount

---

### 6. Card Table & Hand Display

The main game area renders both the dealer's hand and the player's hand as visual playing cards.

**Dealer Hand:**
- Cards rendered as styled HTML/CSS card components (suit symbol + rank)
- During the player's turn, the dealer's second card is face-down (shown as a card back)
- After the player stands or busts, the hole card flips and the dealer plays out their hand with animated card reveals

**Player Hand:**
- Cards displayed in a fan or spread layout
- Current hand value shown below the cards (e.g., "Hand: 16" or "Hand: 7 / 17" for soft hands)
- If the player has been dealt a split hand, both hands are displayed side-by-side with the active hand highlighted

**Card animations:**
- Cards slide in from a deck position in the upper right on deal
- Hole card flips with a CSS 3D rotation animation
- Bust and Blackjack outcomes trigger a brief shake or glow animation on the hand

---

### 7. Player Action Buttons

Standard Blackjack actions rendered as clearly labeled buttons beneath the player's hand. Buttons are enabled or disabled based on game state.

**Actions:**

| Button | Condition to Enable |
|---|---|
| **Hit** | Player's turn, not busted |
| **Stand** | Player's turn |
| **Double Down** | First two cards only; sufficient balance |
| **Split** | First two cards are a pair; sufficient balance; max 3 splits per hand |
| **Surrender** | First two cards only; enabled in settings |

**UI details:**
- Buttons are styled prominently with distinct colors (e.g., Hit = green, Stand = red, Double Down = gold, Split = blue, Surrender = gray)
- Disabled buttons are visually muted, not hidden
- After the hand resolves, all action buttons are hidden and replaced with a **"Next Hand"** button

---

### 8. AI Advisor Panel

A panel displayed to the right of (or below, on mobile) the card table that shows the AI's real-time suggestion and reasoning.

**Panel Header:**
- Title: *"AI Advisor"*
- Status indicator: Shows "Enter API key" or "AI Ready" (in green) based on API key state

**Panel Body (top to bottom):**
1. **Risk Level** dropdown
2. **Autopilot** toggle with bet selector (visible when API key is provided)
3. **Strategy Matrix** (collapsible)
4. **AI Recommendation** display (when playing)
5. **Execute Recommendation** button

**Suggestion content from the AI:**
The AI is prompted with the current game state (player hand, dealer upcard, deck count, risk level, and available actions) and responds with:
1. **Recommended Action** — one of: Hit, Stand, Double Down, Split, Surrender
2. **Reasoning** — 2–4 sentences explaining the decision using basic strategy or expected value reasoning
3. **Hand Strength** — a brief label like "Soft 18 — strong hand" or "Hard 16 — danger zone"
4. **Confidence** — Low / Medium / High (displayed as a simple badge)

**Execute Recommendation button:**
- A prominently styled **"Execute Recommendation"** button appears once a suggestion is ready
- Clicking it performs the AI's recommended action exactly as if the player had clicked that action button manually
- The button is disabled while the AI is still fetching or when it is not the player's turn
- In autopilot mode, this button is hidden (autopilot executes automatically)

**Trigger behavior:**
- Suggestions are fetched automatically each time the player must make a decision (after initial deal, after each hit)
- The AI does not re-query if the player has already acted; it waits for the next decision point
- Suggestions are non-blocking — the player can act before the AI responds

---

### 9. Strategy Visualization

A collapsible section in the AI Advisor panel that displays visual decision aids.

**Tabs:**
1. **Logic** — A step-by-step decision tree showing the AI's reasoning path with YES/NO branches and arrows
2. **Hard** — Basic strategy matrix for hard hand totals (5-21 vs dealer 2-A)
3. **Soft** — Basic strategy matrix for soft hand totals (13-21 vs dealer 2-A)
4. **Pairs** — Basic strategy matrix for pair splitting decisions

**Matrix Display:**
- Color-coded cells: Hit (green), Stand (red), Double (yellow), Split (blue), Surrender (purple)
- Current situation is highlighted with a gold outline
- Legend showing what each color means

**Logic Flow Display:**
- Questions with YES/NO answers in colored badges
- Green path for YES answers, red for NO
- Final recommendation box with action and reasoning
- Updates in real-time as cards are dealt

---

### 10. Autopilot Mode

When enabled, the AI plays the game autonomously on the user's behalf, making all decisions without manual input.

**UI details:**
- A toggle labeled **"Autopilot"** appears in the AI advisor panel body (only visible when a valid API key is provided)
- **Bet selector** appears next to the toggle when autopilot is enabled (Min / 2x / 5x)
- The toggle stays in the same position whether on or off (bet selector uses visibility, not display)
- When toggled on:
  - The **"Execute Recommendation"** button is hidden
  - The AI automatically bets based on the selected multiplier
  - After each API response, the AI's chosen action is executed with a 1.2-second delay (so the user can follow along)
  - The advisor panel narrates each decision in real time
- When toggled off, control returns to the player immediately (after the current hand resolves)

**Autopilot stop conditions:**
- Balance reaches $0
- User toggles autopilot off
- An API error occurs (autopilot pauses with an error message)

---

### 11. Hand History & Statistics

A collapsible panel (collapsed by default) beneath the game table showing running statistics for the current session.

**Stats displayed:**
- Hands played
- Hands won / lost / pushed
- Win rate (%)
- Biggest win
- Current streak (win or loss)
- Net profit/loss for the session

**Hand log:**
- A scrollable list of the last 20 hands showing: hand number, player total, dealer total, outcome, bet amount, and profit/loss
- Most recent entry at the top

**Storage:**
- Stats are **in-memory only** — they reset when the page is refreshed
- A **"Reset Stats"** button clears history and restores defaults

---

### 12. Outcome Display

After each hand resolves, an outcome banner overlays the game table (centered within the table area) briefly before auto-dismissing.

**Outcomes and messages:**

| Result | Banner Text | Color |
|---|---|---|
| Player Blackjack | Blackjack! | Gold |
| Player wins | You Win! | Green |
| Push | Push | Gray |
| Dealer wins | Dealer Wins | Red |
| Player busts | Bust! | Red |
| Dealer busts | You Win! | Green |
| Surrender | Surrendered | Orange |

**Behavior:**
- Banner appears for 2 seconds with a fade-in/fade-out animation
- Banner is centered within the game table area (not the full screen)
- Balance is updated immediately when the banner appears
- The **"Next Hand"** button becomes active once the banner dismisses

---

## Output Format (AI API Response)

The AI is prompted to return a structured JSON object. The system prompt enforces this format:

```json
{
  "action": "Hit | Stand | Double Down | Split | Surrender",
  "reasoning": "String — 2–4 sentence explanation",
  "handStrength": "String — brief label",
  "confidence": "Low | Medium | High"
}
```

**System prompt includes:**
- Risk level (Conservative / Balanced / Aggressive) with specific strategy instructions
- Basic Blackjack strategy as a baseline
- Adjustment reasoning for deck count and dealer upcard
- Never suggest illegal moves (e.g., Split when no pair exists)
- Keep language accessible — avoid dense probability notation; explain in plain terms

---

## UI Design

**Layout:**
- Two-column layout on wider screens (>= 1100px): game table on the left (60%), AI advisor on the right (40%)
- Single-column layout on smaller screens: table above, advisor below
- Configuration panel and stats panel collapse/expand independently
- API key + model selector grouped together in a compact header card at the top

**Visual Theme:**
- Casino felt green (`#1a5c38`) as the primary table surface
- Dark charcoal (`#1a1a2e`) for the background and panels
- Gold (`#c9a84c`) as the accent color for wins, Blackjack, and highlights
- Card faces: clean white with classic suit symbols; red suits in `#d72b2b`
- Card backs: a subtle repeating diamond pattern in dark navy
- Typography: a serif display font for the title and hand values; clean sans-serif for UI labels
- Subtle felt texture applied via CSS noise/grain overlay on the table area
- Ambient glow effect on active hands using `box-shadow`
- No emojis in text — plain text only for all messages and labels

**Error Display:**
- Centered popup on full screen with red-tinted background (`#fee2e2`)
- Auto-dismisses after 2 seconds with a fade-out animation
- Manual close button (x)

---

## User Flow

1. User opens the app
2. User optionally provides API key (paste or file upload) to enable AI features
3. User optionally tests the key and selects a model
4. User optionally adjusts game configuration (decks, balance, bet limits)
5. User clicks **"New Game"** to start
6. User selects chips to place a bet and clicks **"Deal"**
7. Cards are dealt; AI advisor (if key provided) fetches a suggestion automatically
8. User can view Strategy Matrix to understand optimal play
9. User selects an action (or follows the AI suggestion); AI updates suggestion after each hit
10. Hand resolves; outcome banner displays in game table; balance updates
11. User clicks **"Next Hand"** to continue
12. User may enable **AI Autopilot** to let the AI play automatically
13. User may review session stats in the collapsible history panel at any time
14. When balance reaches $0, user is offered a **"Rebuy"** to continue

---

## Error States

| Scenario | Message |
|---|---|
| No API key provided (AI feature triggered) | *"Please enter or upload your OpenAI API key to enable AI suggestions."* |
| Invalid API key format | *"API keys must start with sk-."* |
| API key rejected (401) | *"Your API key was not accepted. Please check it and try again."* |
| Rate limited (429) | *"Too many requests. Please wait a moment and try again."* |
| API timeout / network error | *"Something went wrong. Please check your connection and try again."* |
| File upload — no key found | *"No API key found in that file. Check that it contains OPENAI_API_KEY=..."* |
| File upload — wrong type | *"Only .env and .csv files are supported."* |
| Test key — no key entered | *"Please enter an API key first."* |
| Bet exceeds balance | *"Your bet cannot exceed your current balance."* |

---

## Technical Notes

- **Framework:** Vanilla HTML/CSS/JS (single `index.html` file)
- **API calls:** Made directly from the browser to `https://api.openai.com/v1/chat/completions`
- **No backend required**
- **Deck logic:** Standard Fisher-Yates shuffle; shoe is reshuffled when ~75% of cards have been dealt (configurable penetration)
- **Card rendering:** Pure CSS card components — no images required
- **Storage:**
  - API key: in-memory only (never persisted)
  - Session stats and hand history: in-memory only (reset on page refresh)
  - Game configuration: localStorage (persists between sessions)
- **Accessibility:** All interactive elements have visible labels and ARIA attributes; keyboard navigation works throughout
- **Responsive:** Two-column layout on screens >= 1100px; single-column on smaller screens
- **API Key Input:** Uses CSS `-webkit-text-security: disc` for masking (not `type="password"` to avoid browser warnings)

---

## Console Logging

All key interactions must be logged to the browser console (`console.log` / `console.group`) to support debugging and verification. This is a required technical behavior, not optional.

**Events to log:**

| Event | Log Content |
|---|---|
| API key loaded | `[AUTH] API key loaded (sk-...xxxx)` — last 4 chars only |
| API key test result | `[AUTH] Key test: success` or `[AUTH] Key test: failed — <status>` |
| Hand started | `[GAME] New hand — Player: [cards] (total), Dealer up: [card]` |
| AI request sent | `[AI] Sending request — Player: [cards], Dealer up: [card], Risk: [level], Actions: [list]` |
| Raw AI response | `[AI] Raw response: <full JSON string>` |
| Parsed recommendation | `[AI] Parsed action: <action> — Confidence: <level>` |
| Execute Recommendation clicked | `[AI] Executing recommendation: <action>` |
| Player action taken | `[GAME] Player action: <action>` |
| Dealer play-out | `[GAME] Dealer draws: [card] — Dealer total: <n>` |
| Hand outcome | `[GAME] Outcome: <result> — Bet: $<n>, Balance: $<n>` |
| Autopilot toggled | `[GAME] Autopilot: ON` or `[GAME] Autopilot: OFF` |
| API error | `[ERROR] API call failed — Status: <code>, Message: <msg>` |
| JSON parse failure | `[ERROR] Failed to parse AI response — Raw: <string>` |

**Grouping:**
- Each hand's logs are wrapped in a `console.group("Hand #N")` / `console.groupEnd()` block for easy inspection in DevTools
- AI request and response logs are nested inside a `console.group("[AI] Analysis")` sub-group within the hand group

**Purpose:**
Console output lets an instructor (or developer) open DevTools and verify at a glance: what the AI was told, what it responded, whether the parsed action matched the reasoning, and whether the balance updated correctly — without needing to instrument the code manually.

---

## Implementation Summary

**Core Game Engine:**
- Full Blackjack rules: Hit, Stand, Double Down, Split (up to 3 times), Surrender
- Dealer plays by configurable hard/soft 17 rule
- Blackjack pays 3:2
- Push returns bet
- Shoe-based multi-deck play with configurable penetration reshuffle

**AI Features:**
- API key input with CSS masking, test, upload (drag-and-drop modal), and clear functionality
- Model selector with 4 OpenAI model options
- Risk level selector (Conservative / Balanced / Aggressive) affecting AI recommendations
- Real-time move suggestions with action, reasoning, hand strength, and confidence
- Strategy visualization with decision matrix tables and logic flow explanation
- **Execute Recommendation** button to apply the AI's suggestion in one click
- Autopilot mode with configurable bet sizing and per-action delay
- Non-blocking suggestion fetch (player can act before AI responds)
- Full console logging of all game and AI interactions for debugging

**UI & UX:**
- Animated card dealing and hole-card flip
- Casino chip betting interface with clear/deal controls
- Outcome banners centered in game table with auto-dismiss
- Collapsible stats panel with in-memory session history
- Responsive two-column to single-column layout
- Casino felt table aesthetic with gold accents and card texture
- Centered error popups with red background and auto-dismiss
- No emojis — clean text-only interface
