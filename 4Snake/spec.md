# Snake Game Specification

## Overview
A classic Snake game built with HTML5 Canvas, featuring both single-player and multiplayer modes.

## Game Modes

### Single Player
- One snake (green) controlled by the player
- Eat apples to grow and earn points
- Game speeds up as you collect more apples
- Scores can be added to the Top 5 leaderboard

### Multiplayer (2 Players)
- Two snakes compete on the same screen
- Player 1: Green snake
- Player 2: Purple snake
- Both players collect apples for their own score
- First player to crash loses

## Controls

| Player | Up | Down | Left | Right |
|--------|-----|------|------|-------|
| Player 1 | W | S | A | D |
| Player 2 | I | K | J | L |

- **Space**: Pause/Resume game

## Gameplay Rules

### Winning/Losing Conditions
- **Single Player**: Game ends when the snake hits a wall or itself
- **Multiplayer**: Game ends when either snake:
  - Hits a wall
  - Hits itself
  - Hits the other snake
  - The surviving player wins

### Scoring
- Each apple eaten = 1 point
- Single player: Scores qualify for Top 5 leaderboard (no duplicate scores allowed)
- Multiplayer: Both players' scores displayed at game over

### Game Mechanics
- Snake grows by one segment when eating an apple
- Single player mode: Game speed increases as score increases
- Multiplayer mode: Constant speed for fair competition
- Apples spawn randomly, avoiding both snakes

## Visual Elements

### Snake Design
- Gradient coloring with brighter head
- Eyes that follow movement direction
- Player 1: Green (#4ecca3)
- Player 2: Purple (#9b59b6)

### Apple Design
- Red circular body with glow effect
- Brown stem
- Green leaf tilted to the right

### UI Elements
- Score display for each active player
- Previous game score tracker
- Top 5 leaderboard (single player only)
- Start screen with mode selection
- Game over screen with replay options

## Screens

### Start Screen
- Game title
- Control instructions
- "1 Player" button
- "2 Players" button

### Game Screen
- 400x400 pixel canvas with subtle grid
- Score display at top
- Leaderboard on the right side

### Game Over Screen
- Winner announcement (multiplayer) or "Game Over" (single player)
- Final score(s)
- "Play Again" button (same mode)
- Mode switch button

### High Score Screen (Single Player Only)
- Appears when score qualifies for Top 5
- Name input field
- Submit button

## Technical Details

- **Canvas Size**: 400x400 pixels
- **Grid Size**: 20x20 pixels (20 tiles across)
- **Initial Game Speed**: 100ms per frame
- **Minimum Game Speed**: 50ms per frame (single player only)
- **Speed Increase**: 2ms faster per apple (single player only)
