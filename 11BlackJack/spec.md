# Online Blackjack Game — Product Specification

## Overview

A browser-based single-player Blackjack game where the player competes against a virtual dealer. The game follows standard casino Blackjack rules with a **1.5× payout on a winning hand** (e.g., a $100 bet returns $150 in winnings, for a total of $250 returned to the player).

---

## Game Rules

### Objective
Beat the dealer by having a hand value closer to 21 without exceeding it (busting).

### Card Values
- **Number cards (2–10):** Face value
- **Face cards (J, Q, K):** 10
- **Ace:** 11 or 1 (whichever keeps the hand ≤ 21)

### Blackjack (Natural 21)
A two-card hand totaling 21 (Ace + 10-value card, in any order) is a **Blackjack**. It pays **1.5× the original bet** (same as a standard win). If both player and dealer have Blackjack, the result is a Push.

### Dealer Rules
- Dealer must **hit on 16 or below**.
- Dealer must **stand on 17 or above** (including soft 17).
- Dealer's first card is face-up; second card is face-down (the "hole card"), revealed after the player stands.

---

## Winning Hand Scenarios

A player wins the round under any of the following conditions, each paying out at **1.5× the original bet**:

| # | Scenario | Example |
|---|---|---|
| 1 | **Natural Blackjack** — Player's opening two cards are an Ace + any 10-value card (10, J, Q, K), in any order, and the dealer does not also have Blackjack. Both A♠ K♦ and K♦ A♠ are valid natural Blackjacks. | Player: A♠ K♦ (21) vs. Dealer: 9♣ 7♥ (16 → hits to 20) → Player wins |
| 2 | **Higher total, no bust** — Player stands without busting and their total is strictly greater than the dealer's final total. | Player: 19 vs. Dealer: 17 → Player wins |
| 3 | **Dealer busts** — Player stands without busting and the dealer's hand exceeds 21. | Player: 15 (stands) vs. Dealer: 10 → hits to 23 → Player wins |
| 4 | **Double Down win** — Player doubles their bet, receives one card, and the resulting hand satisfies scenario 2 or 3 above. | Player doubles on 11, draws a 10 for 21 vs. Dealer: 18 → Player wins |
| 5 | **Split hand win** — One or both hands resulting from a Split satisfy scenario 2 or 3 above. Each hand is evaluated and paid out independently. | Player splits 8s; one hand reaches 19, the other busts. The 19 hand wins; the busted hand loses. |

### Non-Winning Outcomes (for completeness)
- **Push:** Player and dealer finish with equal totals (neither wins nor loses; bet returned). If both have Blackjack, the result is also a Push.
- **Player loses:** Player's final total is less than the dealer's, or the player busts. Full bet is forfeited.

---

## Payout Structure

| Outcome | Payout |
|---|---|
| Player wins | +1.5× bet (net profit) |
| Blackjack (natural) | +1.5× bet (net profit) |
| Push (tie) | Bet returned (no gain/loss) |
| Player busts | −1× bet (lose full bet) |
| Dealer wins | −1× bet (lose full bet) |

> **Example:** Player bets $100 and wins → receives $150 profit + $100 original bet = $250 total returned.

---

## Player Actions

| Action | Description |
|---|---|
| **Hit** | Draw one additional card from the deck. |
| **Stand** | End the player's turn; dealer plays. |
| **Double Down** | Double the current bet, draw exactly one more card, then stand. Available on the initial two-card hand only. |
| **Split** | If the two initial cards have equal value, split them into two separate hands, each with its own bet equal to the original. Each hand is then played independently. |

---

## Game Flow

```
1. Player sets bet amount
2. Deal: Player receives 2 cards (face-up); Dealer receives 2 cards (1 face-up, 1 face-down)
3. Check for Blackjack (natural 21) on either side
4. Player chooses actions (Hit / Stand / Double Down / Split) until standing or busting
5. Dealer reveals hole card and plays by house rules
6. Outcome is determined and payout is applied
7. Option to play again or adjust bet
```

---

## Wallet & Betting

- **Starting Balance:** Player begins with a default balance (e.g., $1,000 in chips).
- **Minimum Bet:** $10
- **Maximum Bet:** $500
- **Bet Increment:** $10 steps via +/− buttons or direct input in the editable bet field.
- **Editable Bet Field:** The bet amount can be typed directly into the input field. The field automatically expands to accommodate larger numbers.
- **Debt System:** Players can continue playing into negative balance (debt). The game does not stop at $0.
- Balance and current bet are displayed in an info box in the bottom-left corner, labeled "Current Bet" and "Net Profit".

---

## Deck

- Standard 52-card deck, reshuffled each round (no card counting advantage).
- Alternatively, a configurable 6-deck shoe may be used with a shuffle point at ~75% penetration.

---

## UI / UX Requirements

### Layout
- **Table:** Green felt-style background with designated player and dealer card areas.
- **Top Bar:** Contains game mode toggle buttons (Dealer Tells, Card Counting), settings icon, and sound toggle icon.
- **Info Box:** Positioned in the bottom-left corner, displays Current Bet and Net Profit with labels.
- **Bet Display:** Current bet shown in the betting zone with an editable input field.
- **Action Buttons:** Hit, Stand, Double Down, Split — enabled/disabled contextually.
- **Deal Button:** Initiates the round after bet is placed.
- **Modal Close Buttons:** Help and Settings modals include an X close button in the top-right corner.

### Stats Side Panel

A persistent side panel is displayed alongside the game table to track the player's performance across rounds in the current session.

#### Displayed Statistics
- **Wins** — Total number of rounds won (including natural Blackjacks).
- **Losses** — Total number of rounds lost (including busts).
- **Pushes** — Total number of tied rounds.
- **Win Rate** — Wins as a percentage of total completed rounds, updated after each round.
- **Net Profit/Loss** — Running total of chip gain or loss since the session started, displayed in green (positive) or red (negative).
- **Biggest Win** — Highest single-round payout achieved in the session.
- **Current Streak** — Number of consecutive wins or losses, labeled accordingly (e.g., "3 Win Streak" or "2 Loss Streak").

#### UI Behavior
- The panel sits to the right of the game table on desktop, and collapses into a toggleable drawer (accessible via a "Stats" tab) on mobile.
- Stats update immediately after each round resolves.
- A **"Reset Stats"** button appears below the statistics.
- A **"Keyboard Shortcuts"** reference section is displayed below the Reset Stats button, showing all available key bindings.
- The panel is formatted to fit all content without scrolling on standard displays.
- Stats are stored in `localStorage` and persist between sessions unless manually reset.
- Natural Blackjack wins are counted as wins but also tracked separately with a **"Blackjacks"** sub-counter beneath the Wins total.

---

### Win Celebration — Particle Fireworks

When the player wins a round, a full-viewport **particle system fireworks animation** plays over the game table to celebrate the result.

#### Trigger Conditions
- Standard win (player total beats dealer)
- Dealer busts
- Double Down win
- Split hand win (fires once per winning hand)
- Natural Blackjack — triggers an enhanced version of the animation (see below)

#### Particle System Behavior
- On win, **3–5 firework bursts** launch from the bottom of the viewport and explode at staggered heights and horizontal positions across the screen.
- Each burst emits **80–120 particles** that fan outward in a radial spread, then arc downward under simulated gravity.
- Particles fade out over **1.2–1.8 seconds** with a smooth opacity easing curve.
- Burst colors cycle through a gold, white, and green palette to reinforce a "winning" aesthetic.
- The animation runs for approximately **2 seconds** total before fully clearing, ensuring it doesn't obstruct gameplay for long.

#### Natural Blackjack Enhancement
A natural Blackjack triggers a more dramatic celebration:
- **6–8 bursts** fire in rapid succession with a slight cascade delay (100ms between each).
- Particles are larger and include **star-shaped and sparkle variants** alongside standard circular particles.
- A brief **golden shimmer sweep** passes across the player's card area from left to right as the burst begins.
- Animation duration extends to approximately **3 seconds**.

#### Performance & Accessibility
- The animation is rendered on a transparent `<canvas>` overlay positioned above the game table, ensuring no DOM layout is disrupted.
- Particle count automatically scales down on low-powered devices based on a frame rate check at launch (target: 60fps; fallback: reduced to 40 particles per burst at <30fps).
- A **"Reduce Animations"** toggle in settings disables the fireworks entirely and replaces them with a simple green win banner for players who prefer minimal motion or have accessibility needs.
- The animation does not block any action buttons — the player can immediately place a new bet while particles are still fading.

---

### Card Animations
- Cards slide in smoothly when dealt.
- Dealer's hole card flips with a brief animation on reveal.

### Feedback
- Hand totals displayed beneath each hand in real time.
- Win/Loss/Push result displayed clearly with color coding (green / red / yellow).
- Net change in balance shown after each round.

### Responsiveness
- Fully playable on desktop and mobile browsers.
- Touch-friendly buttons with appropriate sizing on small screens.

---

## Visual Design

### Color Palette

The game uses CSS custom properties (variables) for consistent theming across all elements.

#### Classic Casino Theme (Default)
| Element | Color | Hex Code |
|---|---|---|
| Felt Green (primary) | Deep casino green | `#1a5f2a` |
| Felt Dark | Dark green accent | `#0d3d16` |
| Gold | Primary accent color | `#d4af37` |
| Gold Light | Hover/highlight state | `#f4cf67` |
| Card White | Card background | `#ffffff` |
| Win Green | Victory indicator | `#4caf50` |
| Lose Red | Loss indicator | `#f44336` |
| Push Yellow | Tie indicator | `#ffc107` |
| Overlay BG | Modal/overlay backdrop | `rgba(0,0,0,0.85)` |

#### Modern Dark Theme
| Element | Color | Hex Code |
|---|---|---|
| Background | Dark navy | `#1a1a2e` |
| Background Dark | Deeper navy | `#0f0f1a` |
| Accent (replaces gold) | Cyan | `#00d4ff` |
| Secondary Accent | Purple | `#7c4dff` |
| Card Background | Dark slate | `#2a2a3e` |

#### Retro Arcade Theme
| Element | Color | Hex Code |
|---|---|---|
| Background | Deep purple | `#2d1b69` |
| Background Dark | Darker purple | `#1a0f3c` |
| Accent | Coral red | `#ff6b6b` |
| Secondary Accent | Bright yellow | `#ffd93d` |
| Card Background | Cream | `#fffef0` |

---

### Typography

#### Font Families
| Theme | Primary Font | Fallback |
|---|---|---|
| Classic Casino | Times New Roman | Georgia, serif |
| Modern Dark | Times New Roman | Georgia, serif |
| Retro Arcade | Return of Ganon (codeman38) | cursive |

#### Font Sizing
- **Area Labels:** 0.9rem, uppercase, letter-spacing 2px
- **Card Rank/Value:** 2rem (center), 0.85rem (corners)
- **Hand Total:** 1.3rem
- **Bet Amount:** 2rem
- **Action Buttons:** 1.1rem, uppercase, letter-spacing 1px
- **Info Box Labels:** 1rem
- **Info Box Values:** 1.5rem, bold

---

### Card Design

Cards are styled to evoke the classic Bicycle playing card aesthetic.

#### Dimensions
- Width: 90px
- Height: 130px
- Border radius: 8px
- Card overlap (stacked): -30px margin

#### Card Face
- White background with bold center rank and suit
- Corner pips showing rank and suit (top-left and bottom-right, rotated 180°)
- Traditional colors: Red (`#c41e3a`) for hearts/diamonds, Black (`#000000`) for spades/clubs
- Center suit displayed at 2.5rem

#### Card Back
- Blue gradient (`#1565c0` to `#0d47a1`)
- Diagonal stripe pattern overlay (45° repeating)
- Inner border with 8px inset, 2px solid white at 30% opacity

#### Card Animations
- **Deal animation:** 0.4s ease-out, slides from -200px above with -20° rotation
- **Flip animation:** 0.5s ease-in-out rotateY from 0° → 90° → 0°

---

### UI Components

#### Buttons

**Action Buttons**
- Padding: 15px 30px
- Border radius: 8px
- Primary style: Gold background, dark text
- Secondary style: Semi-transparent with white border
- Hover: Slight upward translation (-2px)
- Disabled: 50% opacity, no hover effects

**Bet Adjustment Buttons (+/−)**
- Circular: 50px × 50px
- Gold border (3px solid)
- Semi-transparent background
- Flexbox centering for symbols
- Arial font for consistent symbol rendering

**Icon Buttons (settings, sound)**
- Circular: 44px × 44px
- Gold border accent
- SVG icons matching theme
- Hover: Gold fill with dark icon

**Mode Toggle Buttons**
- Pill-shaped: padding 8px 15px, border-radius 20px
- Inactive: Semi-transparent, subtle border
- Active: Gold background, dark text

#### Info Box
- Fixed position: bottom-left corner (30px from edges)
- Semi-transparent black background (`rgba(0,0,0,0.5)`)
- Border radius: 14px
- Padding: 22.5px 30px
- Two rows: Current Bet, Net Profit
- Values displayed in gold (or red when negative)

#### Hand Total Indicator
- Pill-shaped background (`rgba(0,0,0,0.5)`)
- Border radius: 20px
- Padding: 8px 20px
- Special states: Bust (red background), Blackjack (gold background with dark text)

#### Side Panel
- Fixed width on desktop
- Semi-transparent background
- Contains statistics, reset button, and keyboard shortcuts
- Scrolling disabled (content fits viewport)

---

### Animations & Effects

#### Card Dealing
```css
@keyframes dealCard {
    from { transform: translateY(-200px) rotate(-20deg); opacity: 0; }
    to { transform: translateY(0) rotate(0); opacity: 1; }
}
```

#### Card Flip (Hole Card Reveal)
```css
@keyframes flipCard {
    0% { transform: rotateY(0); }
    50% { transform: rotateY(90deg); }
    100% { transform: rotateY(0); }
}
```

#### Win Celebration
- Particle fireworks rendered on transparent canvas overlay
- 3-5 bursts for standard wins, 6-8 for natural Blackjack
- Gold, white, and green particle palette
- 1.2-1.8 second fade duration

#### Button Transitions
- All interactive elements: 0.2s transition duration
- Hover states provide immediate visual feedback

---

### Shadows & Depth

| Element | Shadow |
|---|---|
| Cards | `0 4px 15px rgba(0,0,0,0.3)` |
| Modals | Dark overlay backdrop with centered content |
| Top Bar | `rgba(0,0,0,0.3)` background |
| Side Panel | Consistent with overall theme opacity |

---

### Responsive Considerations

- Game container uses flexbox for fluid layout
- Minimum viewport height: 100vh
- Side panel collapses to toggleable drawer on mobile
- Touch-friendly button sizing (minimum 44px tap targets)
- Cards scale appropriately for smaller screens

---

### Accessibility

- Color contrast ratios meet WCAG guidelines for text legibility
- Focus states visible on all interactive elements
- No reliance on color alone for critical information (icons accompany states)
- "Reduce Animations" setting available for motion-sensitive users

---

## Edge Cases & Rules Clarifications

- **Bust:** If player's hand exceeds 21, the round ends immediately and the bet is lost regardless of the dealer's hand.
- **Split Aces:** Each split Ace receives exactly one additional card; no further hitting allowed on split Aces.
- **Double Down after Split:** Not permitted.
- **Insurance:** Not offered.
- **Surrender:** Not offered.

---

## Technical Requirements

- **Frontend:** HTML5, CSS3, JavaScript (vanilla or framework of choice, e.g., React).
- **State Management:** All game state managed client-side (no backend required for single-player).
- **Persistence:** Player balance persisted in `localStorage` between sessions.
- **Randomness:** Cards drawn using a cryptographically seeded shuffle (Fisher-Yates algorithm).
- **No external dependencies required** beyond a modern browser.

---

## Tutorial Mode

### Overview
Tutorial Mode is an optional, guided experience designed for first-time players. It walks the player through a scripted game round step by step, explaining each rule and decision point with contextual tooltips and commentary. No real chips are wagered during the tutorial; a separate "play money" balance is used.

### Entry Point
- On first launch, the player is prompted: **"New to Blackjack? Try Tutorial Mode"** with options to start the tutorial or skip to regular play.
- Tutorial Mode can also be accessed at any time via a **"?" / Help** button in the main UI.
- Progress is saved in `localStorage` so returning players are not prompted again unless they reset.

### Tutorial Structure

The tutorial is broken into **6 sequential lessons**, each covering one core concept. The player cannot advance to the next lesson until they complete the required action in the current one.

#### Lesson 1 — The Goal
- A tooltip overlay appears explaining the objective: "Get closer to 21 than the dealer without going over."
- Introduces the concept of busting.
- Player reads the explanation and clicks **"Got it → Next"** to continue.

#### Lesson 2 — Card Values
- An animated card reference panel slides in, showing example cards (2–10, J/Q/K, Ace).
- Each card type highlights as its value is narrated in the tooltip: number cards = face value, face cards = 10, Ace = 1 or 11.
- A short interactive quiz: player is shown a two-card hand and asked to select the correct total from three options. Incorrect answers show a gentle correction with an explanation.

#### Lesson 3 — The Deal
- A scripted deal animation plays: player receives two face-up cards, dealer receives one face-up and one face-down card.
- Tooltip explains the hole card: "The dealer keeps one card hidden until it's their turn."
- Hand totals are shown with a callout arrow pointing to them.

#### Lesson 4 — Player Actions
- Tooltips appear over each action button as it is introduced in sequence:
  - **Hit:** "Draw another card to increase your total. Be careful not to bust!"
  - **Stand:** "Keep your current hand and let the dealer play."
  - **Double Down:** "Double your bet and receive exactly one more card. A bold move when you're feeling confident." (grayed out with a note if the scripted hand doesn't qualify.)
  - **Split:** "If your two starting cards have the same value, you can split them into two separate hands." (grayed out with a note if not applicable.)
- Player is prompted to perform a **Hit** on the scripted hand to see a new card added.
- Player is then prompted to **Stand** to end their turn.

#### Lesson 5 — The Dealer's Turn
- The hole card flips with a highlight and tooltip: "Now the dealer reveals their hidden card and must follow strict rules."
- Dealer plays automatically (hitting on ≤16, standing on ≥17) with each action narrated in a tooltip bubble.
- Result is shown with a clear win/loss/push callout and the payout explained: "You win! Your $10 bet earns $15 in profit — that's 1.5× your wager."

#### Lesson 6 — Payouts & Wrap-Up
- A summary screen displays the full payout table (mirroring the Payout Structure section).
- Each row is briefly highlighted with a one-line plain-English explanation.
- A **"Blackjack!" demo** is shown: a scripted Ace + King hand is dealt, labeled as a natural Blackjack, with the 1.5× payout highlighted.
- Player is shown their tutorial completion and prompted: **"You're ready to play! Start a real game →"**

### Tutorial UI Behavior

- **Overlay & Spotlight:** Non-relevant UI elements are dimmed; the active element is spotlit with a soft glow.
- **Tooltip Bubbles:** Appear adjacent to the relevant element with a pointer arrow. Each tooltip has a **"Next"** or **"Try it"** CTA.
- **Skip Option:** A **"Skip Tutorial"** button is always visible in the top-right corner of the tutorial overlay. Skipping jumps directly to the regular game.
- **Replay:** Players can replay individual lessons or the full tutorial from the Help menu at any time.
- **No Chip Risk:** Tutorial uses a separate $500 fake balance clearly labeled "Tutorial Chips — not real money." This balance resets on each tutorial run.
- **No Distractions:** Bet adjustment controls and the Deal button are pre-set and locked during tutorial steps so the player focuses only on the concept being taught.

### Help Menu (Non-Tutorial Mode)

When the player is not in Tutorial Mode, the **"?" / Help** button opens a compact, non-blocking rules overview panel (modal or slide-out drawer). This gives experienced players a quick reference without launching the full step-by-step tutorial.

#### Contents

**The Goal**
Get a hand total closer to 21 than the dealer's without going over. Going over 21 is a "bust" and you lose the round immediately.

**Card Values**
Number cards are worth their face value, face cards (J, Q, K) are worth 10, and Aces are worth 11 or 1 — whichever keeps your hand at 21 or below.

**Your Actions**
- **Hit** — Draw another card.
- **Stand** — Keep your hand and end your turn.
- **Double Down** — Double your bet and receive exactly one more card (available on your opening two cards only).
- **Split** — If your two starting cards have equal value, split them into two independent hands, each with its own bet.

**Dealer Rules**
The dealer must hit on 16 or below and stand on 17 or above. The dealer's second card stays hidden until your turn ends.

**Payouts**
- Win → +1.5× your bet
- Push (tie) → bet returned
- Lose or bust → bet lost

#### UI Behavior
- The panel opens as an overlay or side drawer without interrupting an active round.
- A **"Full Tutorial"** link at the bottom of the panel lets the player launch the complete Tutorial Mode at any time.
- The panel is dismissed by clicking outside it, pressing Escape, or clicking a close button.

---

### Help Me Win (Strategy Assistant)

A **"Help Me Win"** button inside the Help Menu provides context-aware strategy suggestions based on the current state of the game. It is only active during a live round, after the player's opening two cards have been dealt.

#### How It Works
When the player clicks "Help Me Win", the panel reads the current game state — the player's hand total, whether the hand is soft (contains an Ace counted as 11), whether a Split or Double Down is available, and the dealer's visible (face-up) card — and displays a recommended action with a plain-English explanation.

#### Strategy Logic
Suggestions follow standard Basic Strategy, the statistically optimal set of decisions for each possible player hand vs. dealer upcard combination.

**Hard Hands (no Ace counted as 11)**

| Player Total | Dealer Upcard | Suggestion |
|---|---|---|
| 8 or less | Any | Hit |
| 9 | 3–6 | Double Down; otherwise Hit |
| 10 | 2–9 | Double Down; otherwise Hit |
| 11 | 2–10 | Double Down; otherwise Hit |
| 12 | 4–6 | Stand; otherwise Hit |
| 13–16 | 2–6 | Stand; otherwise Hit |
| 17 or more | Any | Stand |

**Soft Hands (Ace counted as 11)**

| Player Total | Dealer Upcard | Suggestion |
|---|---|---|
| Soft 13–14 (A+2, A+3) | 5–6 | Double Down; otherwise Hit |
| Soft 15–16 (A+4, A+5) | 4–6 | Double Down; otherwise Hit |
| Soft 17 (A+6) | 3–6 | Double Down; otherwise Hit |
| Soft 18 (A+7) | 2–8 | Stand; vs. 9–Ace Hit |
| Soft 19–21 | Any | Stand |

**Pairs (Split eligible)**

| Pair | Dealer Upcard | Suggestion |
|---|---|---|
| Aces | Any | Split |
| 8s | Any | Split |
| 10s / J / Q / K | Any | Stand (never Split) |
| 9s | 2–6, 8–9 | Split; otherwise Stand |
| 7s | 2–7 | Split; otherwise Hit |
| 6s | 2–6 | Split; otherwise Hit |
| 5s | 2–9 | Double Down; otherwise Hit |
| 4s | 5–6 | Split; otherwise Hit |
| 2s or 3s | 2–7 | Split; otherwise Hit |

#### Suggestion Display
Each suggestion is shown as a highlighted action label (e.g., **"Stand"**, **"Hit"**, **"Double Down"**, **"Split"**) followed by a one- or two-sentence plain-English explanation. Examples:

- *"Stand — Your 16 is weak, but the dealer is showing a 6, meaning they're likely to bust. Hold your ground."*
- *"Double Down — You have 11 and the dealer shows a 5. This is one of the best spots to double; you're likely to land near 21 and the dealer is in a tough spot."*
- *"Split — Always split Aces. Two separate chances to hit 21 is far better than a soft 12."*

#### UI Behavior
- The strategy suggestion appears inline within the Help panel as a distinct highlighted callout box.
- The suggestion updates in real time if the player Hits and the hand state changes.
- If it is not the player's turn (e.g., dealer is playing or between rounds), the button is grayed out with the label "Suggestions available on your turn."
- The feature is advisory only — it never automatically takes an action. The player always makes the final decision.
- A small disclaimer reads: *"Suggestions are based on Basic Strategy and represent statistically optimal play over time. Individual round outcomes may vary."*

---

### Tutorial Completion State
- On completion, `tutorialComplete: true` is written to `localStorage`.
- A subtle completion indicator appears in the Help menu to indicate it has been finished.
- First-launch prompt is suppressed for all future sessions.

---

## Debt Pause Screen

When a player's balance goes negative and reaches **−$10,000 in debt**, the game pauses and displays a full-screen overlay titled **"Reflection Time"**. The overlay cannot be dismissed until the countdown timer expires and the player makes a choice.

### Behavior

- The game state is frozen — no actions, betting, or navigation are possible while the overlay is active.
- A countdown timer starts at **1 minute**. During the countdown, only the title "Reflection Time" and the timer are displayed.
- Once the timer expires, the message changes to **"Would you like to continue?"** and two buttons appear:
  - **"Yes"** — dismisses the overlay and resumes the game.
  - **"No"** — closes the browser tab.
- The overlay is triggered again at each additional **−$10,000 threshold** (i.e., −$20,000, −$30,000, etc.).
- Each subsequent trigger adds **1 additional minute** to the countdown (2 min at −$20k, 3 min at −$30k, and so on).

### Deep Debt Recovery
- When debt reaches **−$100,000**, a **"Start New Game"** option becomes available in the settings menu.
- This option resets the balance to the starting amount and clears all statistics.

### UI
- The overlay covers the full viewport with a dimmed background.
- During countdown: The title **"Reflection Time"** is displayed prominently in the center with the countdown timer beneath.
- After countdown: The title changes to **"Would you like to continue?"** and response buttons appear.
- The countdown timer is displayed in a large, legible format (e.g., `1:00` counting down).
- No close button, Escape key, or other exit mechanism is available during the countdown.

---

## Dealer Tells Mode

Dealer Tells is an optional game mode that adds a layer of psychological skill to the game. The virtual dealer exhibits subtle behavioral cues — visual "tells" — that hint at the strength of their hidden hole card. Players who pay close attention can use these tells to inform their decisions, rewarding observation alongside strategy.

Tells are probabilistic hints, not guarantees. They are designed to be plausible and consistent enough to be learnable, but never 100% reliable, preserving the house edge and game balance.

### Enabling the Mode
- Toggle button displayed in the **top bar** alongside Card Counting mode.
- Can be toggled on/off between rounds.
- When active, the button appears highlighted/active.
- Compatible with all other features including Tutorial Mode (tells are explained in a bonus Lesson 7 if this mode is active during the tutorial) and Help Me Win (suggestions factor in the observed tell when one is detected).
- No emoji indicators; visual treatment matches the current theme.

### Tell Categories

Tells are delivered through four channels — animation, timing, audio, and UI micro-cues — so they feel organic rather than mechanical.

#### 1. Dealing Animation
- **Weak hole card (2–6):** The dealer's dealing hand lingers briefly over the hole card after placing it, as if reluctant.
- **Strong hole card (10, J, Q, K, A):** The card is placed quickly and cleanly with no hesitation.
- **Mid hole card (7–9):** Neutral animation with no notable variation.

#### 2. Dealer Expression / Avatar Reaction
If a dealer avatar is shown, it reacts subtly after peeking at the hole card:
- **Weak card:** Avatar glances slightly to the side or shifts posture, suggesting mild discomfort.
- **Strong card:** Avatar settles back with a slight stillness or composed expression.
- **Blackjack:** No tell is shown — the game resolves immediately as normal.

#### 3. Timing Delay
- **Weak hole card:** A slightly longer pause (300–500ms above baseline) occurs after the dealer peeks, as if the dealer is recalibrating.
- **Strong hole card:** The peek resolves at or below baseline speed.
- Timing variation is randomized within a ±100ms noise window to avoid being machine-readable.

#### 4. UI Micro-Cue
- A very faint color tint pulses once on the dealer's card back immediately after the deal:
  - **Warm tint (amber):** Suggests a weaker hole card.
  - **Cool tint (blue):** Suggests a stronger hole card.
  - **No tint:** Neutral / mid-range card.
- The tint is intentionally subtle — visible on close attention, easy to miss otherwise.

### Tell Reliability
Tells are accurate approximately **65–70% of the time**, making them useful signals but not certainties. The reliability is consistent enough that an attentive player gains a small statistical edge, but not so reliable that they override Basic Strategy.

| Tell Signal | Approximate Accuracy |
|---|---|
| Weak card tell (2–6) | 68% |
| Strong card tell (10, J, Q, K, A) | 65% |
| Mid card tell (7–9) | ~50% (near random) |

### Integration with Help Me Win
When Dealer Tells Mode is active, the **Help Me Win** suggestion engine incorporates the current tell signal into its recommendation:
- If a weak tell is detected and it changes the optimal action (e.g., standing on 15 vs. a dealer 10 would normally be a hit, but a weak tell might suggest standing), the suggestion updates accordingly.
- The explanation notes the tell: *"The dealer's hesitation suggests a weak hole card. Consider standing and letting them bust."*
- If no tell is detected or the tell is neutral, suggestions fall back to standard Basic Strategy.

### Tutorial Integration
If Dealer Tells Mode is enabled when the player launches Tutorial Mode, a bonus **Lesson 7 — Reading the Dealer** is appended to the tutorial sequence:
- Explains what tells are and that they are probabilistic, not guaranteed.
- Walks through each tell channel (animation, expression, timing, tint) with annotated examples.
- Presents a short interactive exercise where the player identifies the tell on a scripted hand and selects an action based on it.
- Concludes with: *"Tells give you an edge — but never a sure thing. Use them alongside strategy, not instead of it."*

---

## Card Counting Educational Simulator

The Card Counting Simulator is a read-only educational mode that teaches players how card counting works and why casinos consider it cheating. It does not affect the live game or provide a real in-game advantage — the deck still reshuffles every round as normal. All counting is demonstrated on scripted or illustrative hands clearly labeled as a learning sandbox.

This mode is accessible via a toggle button in the **top bar** alongside Dealer Tells mode.

---

### What the Mode Teaches

The simulator covers card counting in three parts: the concept, the mechanics, and the real-world consequences.

#### Part 1 — What Is Card Counting?
An introductory screen explains the concept in plain language:

> Card counting is a strategy used by some players to track the ratio of high to low cards remaining in the deck. When more high cards remain, the player has a statistical advantage — so a counter raises their bet. When more low cards remain, the house has the edge — so a counter bets low.

Key points covered:
- Card counting is not illegal, but casinos treat it as a violation of their house rules and will ban players caught doing it.
- It requires no special ability — it is a learned, practiced skill based on simple arithmetic.
- It only works meaningfully against a multi-deck shoe, not a reshuffled single deck (which is why this game's standard mode reshuffles every round).

#### Part 2 — The Hi-Lo System
The simulator teaches the most widely known counting method, the **Hi-Lo system**:

| Card | Count Value |
|---|---|
| 2, 3, 4, 5, 6 | +1 (low cards favor the house — count goes up) |
| 7, 8, 9 | 0 (neutral — count unchanged) |
| 10, J, Q, K, A | −1 (high cards favor the player — count goes down) |

The **Running Count** is the cumulative sum of all card values seen so far. A positive running count means more low cards have been dealt, so more high cards remain — favorable for the player.

The **True Count** adjusts for the number of decks remaining: `True Count = Running Count ÷ Decks Remaining`. This is the figure a counter actually uses to size their bets.

#### Part 3 — Real-World Consequences
A brief explainer covers how casinos detect and respond to card counting:
- Surveillance teams watch for bet-sizing patterns that correlate with count swings.
- Casinos may shuffle the deck early, restrict bet sizes, or ask a suspected counter to leave.
- Counting teams (like the famous MIT Blackjack Team) use signals and role separation to avoid detection.
- Online and reshuffled-deck games are immune to card counting, which is the primary reason continuous shufflers exist.

---

### The Card Counter Display

While in the simulator, a **Card Counter HUD** appears alongside the table. It is clearly labeled **"Educational Display — Not Active in Real Game"** and updates in real time as cards are revealed during scripted demo rounds.

#### HUD Elements

| Element | Description |
|---|---|
| **Running Count** | Live cumulative Hi-Lo count across all revealed cards. Color-coded: green for positive, red for negative, white for zero. |
| **Cards Remaining (est.)** | Approximate number of cards left in the simulated shoe. |
| **True Count** | Running Count ÷ estimated decks remaining, rounded to one decimal. |
| **Count Interpretation** | A plain-English label beneath the true count: "Favor: Player", "Favor: House", or "Neutral". |
| **Suggested Bet Size** | An illustrative indicator showing what a counter would do: "Bet High", "Bet Low", or "Bet Neutral". This is for education only and does not affect the actual bet. |
| **Card History Strip** | A scrolling row of the last 10 revealed cards, each tagged with its Hi-Lo value (+1, 0, −1) for reference. |

#### Demo Mode
The simulator includes a fully scripted **Auto-Play Demo** that deals a sequence of hands from a simulated 6-deck shoe without requiring player input. Each card is dealt with a brief pause, and an animated callout highlights:
- The card's Hi-Lo value as it lands.
- The running count ticking up or down.
- Moments where the count crosses a meaningful threshold and a counter would change their bet size.

The demo can be paused, rewound, or stepped through card by card using playback controls.

---

### UI Behavior
- The mode is entirely non-interactive with the live game — no bets are placed and no chips are affected.
- An **"Exit Simulator"** button is always visible and returns the player to the main menu.
- A **"Test Yourself"** sub-mode presents a sequence of cards one at a time and asks the player to input the running count at each step, providing instant feedback on accuracy.
- Progress through the three educational parts is tracked and shown as a simple checklist on the mode's home screen.

---

## Sound Effects

Audio cues reinforce game events and enhance immersion. All sounds are optional and toggled via a dedicated **Mute Button** in the UI.

### Mute Button
- Displayed as a persistent icon button in the top bar, always visible regardless of game state.
- Icon style matches the current theme (no emojis; uses theme-appropriate symbols).
- Clicking the button toggles all sound effects on or off instantly.
- Visual treatment indicates the current muted/unmuted state.
- Mute state is persisted in `localStorage` so the player's preference is remembered across sessions.
- Keyboard shortcut `M` triggers the same toggle (see Keyboard Shortcuts).

| Event | Sound |
|---|---|
| Card dealt | Soft card sliding/flicking sound |
| Player hits | Same card deal sound |
| Chip placed (bet) | Chip clink |
| Player wins | Short upbeat chime or fanfare |
| Natural Blackjack | Extended celebratory fanfare |
| Player loses / busts | Low, muted thud or descending tone |
| Push | Neutral soft chime |
| Dealer hole card revealed | Card flip sound |
| Button click | Subtle click tick |
| Debt Pause Screen appears | Slow, sobering tone |

- Sounds are loaded as short `.mp3` or `.ogg` files.
- Volume is adjustable via a slider in the settings panel.
- All sounds respect the OS-level mute state.

---

## Keyboard Shortcuts

Keyboard shortcuts allow faster play for desktop users. A shortcuts reference is shown in the Help Menu.

| Key | Action |
|---|---|
| `H` | Hit |
| `S` | Stand |
| `D` | Double Down (if available) |
| `P` | Split (if available) |
| `Enter` or `Space` | Deal (when in betting phase) |
| `+` / `=` | Increase bet by one increment |
| `-` | Decrease bet by one increment |
| `M` | Toggle mute |
| `?` | Open / close Help Menu |
| `Escape` | Close any open panel or drawer |

- Shortcuts are disabled during animations to prevent accidental double-inputs.
- Shortcuts are disabled while the Debt Pause Screen countdown is active.
- A keyboard shortcuts reference is displayed in the side panel for easy reference.

---

## Theme Options

Players can choose from three visual themes, selectable from the settings panel. The chosen theme is persisted in `localStorage`.

### Classic Casino
- Deep green felt table, gold trim accents.
- Traditional red and black card suits styled to match Bicycle playing cards.
- Serif typography (Times New Roman/Georgia) matching classic playing card aesthetics.
- Default theme.

### Modern Dark
- Dark charcoal and slate background with neon accent colors (cyan, purple).
- Minimalist card design with thin-stroke suits.
- Clean sans-serif typography throughout.

### Retro Arcade
- Bright pixel-art inspired aesthetic using the **"Return of Ganon"** font by codeman38.
- Chunky card borders and saturated colors.
- Fireworks animation uses pixel-burst particles instead of smooth circles.
- Chip stack rendered as pixel-art icons.
- All UI elements use pixelated image rendering.

All three themes share identical layout and functionality — only visual styling differs. Themes apply instantly on selection with no page reload required.

---

## Testing Scenarios

The following scenarios must be validated before release to ensure rules, payouts, and edge cases are handled correctly.

### Payout Tests

| # | Scenario | Expected Result |
|---|---|---|
| T1 | Player gets A♠ K♦ (Blackjack), dealer gets 18 | Player wins, payout = 1.5× bet |
| T2 | Player gets K♥ A♣ (Blackjack, reverse order), dealer gets 18 | Player wins, payout = 1.5× bet |
| T3 | Player gets Q♦ A♠ (Blackjack), dealer also gets A♣ K♣ (Blackjack) | Push — bet returned, no payout |
| T4 | Player wins with 20 vs dealer 18 (no Blackjack) | Player wins, payout = 1.5× bet |
| T5 | Player busts (hand > 21) | Player loses full bet immediately |
| T6 | Dealer busts, player stands on any total ≤ 21 | Player wins, payout = 1.5× bet |
| T7 | Player and dealer both stand on 19 | Push — bet returned |
| T8 | Player doubles down, resulting hand beats dealer | Payout = 1.5× doubled bet |
| T9 | Player splits, one hand wins, one hand loses | Winning hand pays 1.5×; losing hand forfeits its bet independently |

### Natural Blackjack Combination Coverage

All of the following two-card combinations must be recognized as natural Blackjack (in either deal order):

- A + 10, A + J, A + Q, A + K (all four suits for each)
- Reverse order: 10 + A, J + A, Q + A, K + A

A total of **32 unique two-card combinations** (8 Ace cards × 4 ten-value ranks × both orders) must resolve as Blackjack when neither card is a duplicate draw from an impossible deck state.

### Game State Tests

| # | Scenario | Expected Result |
|---|---|---|
| G1 | Player clicks Hit when it is not their turn | Button is disabled; no action taken |
| G2 | Player attempts to bet more than their balance | Bet capped at current balance; Deal button remains disabled until valid bet |
| G3 | Player balance reaches $0 | Player can continue playing into debt; no prompt appears |
| G4 | Player reaches −$10,000 | Debt Pause Screen fires with 1-minute countdown |
| G5 | Player reaches −$20,000 after continuing | Debt Pause Screen fires again with 2-minute countdown |
| G5a | Player reaches −$100,000 in debt | "Start New Game" option appears in settings menu |
| G6 | Split attempted on non-equal cards | Split button remains disabled |
| G7 | Double Down attempted after first hit | Double Down button is disabled after initial two-card deal phase |
| G8 | Dealer stands on soft 17 (e.g., A+6) | Dealer does not hit; round resolves correctly |
| G9 | Dealer hits on hard 16 | Dealer draws until ≥ 17 or busts |

### Deck & Randomness Tests

| # | Scenario | Expected Result |
|---|---|---|
| D1 | Full deck audit — deal all 52 cards | Exactly 52 unique cards dealt with no duplicates |
| D2 | Shuffle distribution test — run 1,000 shuffles | Each card appears in each position with roughly equal frequency (~1.9%) |
| D3 | Deck resets between rounds | Cards dealt in round 2 are drawn from a freshly shuffled deck independent of round 1 |

### UI / UX Tests

| # | Scenario | Expected Result |
|---|---|---|
| U1 | Current bet is visible before cards are dealt | Bet amount displayed in betting zone prior to clicking Deal |
| U2 | Hand totals update in real time after each Hit | Running total shown beneath player and dealer hands |
| U3 | Win/Loss/Push result is clearly shown after round | Color-coded result banner displayed with net chip change |
| U4 | Reduce Animations setting disables fireworks | No particle animation on win; simple banner shown instead |
| U5 | Keyboard shortcut `H` triggers Hit | Same result as clicking the Hit button |
| U6 | Mute toggle silences all sound effects | No audio plays after muting |
| U7 | Theme change applies without page reload | Visual style updates instantly across all elements |
| U8 | Stats panel updates after every round | Wins, losses, pushes, win rate, and streak all reflect latest round |

---

## Out of Scope (v1)

- Multiplayer / live dealer
- Real-money wagering
- Account registration / login
- Leaderboards
- Side bets (Perfect Pairs, 21+3, etc.)
