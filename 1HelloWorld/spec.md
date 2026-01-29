# Terminal Chatbot

## Overview
A browser-based chatbot that simulates a command-line interface with conversational capabilities and Windows-style window controls.

## Features

### Visual Design
- Dark terminal-style interface with monospace font
- Windows-style window frame with minimize, maximize, and close buttons
- Cyan text output on dark background
- Green blinking cursor animation
- Hidden "Peekaboo!" message in the background

### Window Controls
- **Close button (X)**: Hides the terminal window; click "Peekaboo!" to reopen
- **Minimize button (-)**: Collapses the window to just the header bar
  - Minimizing from fullscreen moves window to bottom-right corner
  - Clicking the header bar reopens the window
- **Maximize button (square)**: Toggles fullscreen mode
  - Icon changes to stacked squares when maximized
  - Window repositions to stay visible when leaving minimized state

### Interactivity
- **Draggable window**: Click and drag the header bar to move the terminal
- **Keyboard input**: Type directly on the page to enter text
- **Backspace support**: Delete characters with backspace key
- **Enter to submit**: Press enter to send messages

### Chat Responses
The chatbot responds to:
- Greetings (hi, hello, hey, howdy, yo, sup)
- "How are you" questions
- Name introductions ("my name is...")
- "What's your name" / "who are you" - responds with "I'm Chatbot!"
- Time and date requests
- Jokes ("tell me a joke")
- Compliments ("you're awesome")
- Thank you / goodbye
- Help command
- Unrecognized input returns "no"

### Commands
- `clear` - Resets the chat to initial state
- `help` - Shows available commands and features

## Tech Stack
- HTML5
- CSS3 (animations, flexbox, transitions)
- Vanilla JavaScript (no dependencies)

## Live URL
https://racoonma392.github.io/Test-Project/
