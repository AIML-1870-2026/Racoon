# Confrontation Decision Neuron - Specification

## Overview
A web-based tool that uses a simple perceptron (single neuron) to predict the probability of a person confronting their superior based on 12 adjustable factors. The Emotional State factor directly controls the neuron's bias term, while the other 11 factors have learned weights.

The tool features a split-screen design:
- **Left side**: Interactive controls including a dedicated Emotional State (Bias) section, 11 weighted factors, and neuron weight display
- **Right side**: Visual representation with three modes - Home view (figures and speech bubble), Decision Boundary visualization, and Training Mode

## Core Features

### 1. Probability Display
- Large, prominent display showing the calculated probability (0-100%)
- Updates in real-time as users adjust sliders
- Visually distinct from the rest of the interface

### 2. Emotional State (Bias Control)
Located in its own dedicated section under the header:
- **Slider**: Controls the neuron's bias term (Calm to Emotional)
- **Bias Display**: Shows the current bias value in real-time
- **Range**: Calm (0) → Bias = -1, Emotional (1) → Bias = +1
- **Effect**: Higher emotional states increase overall confrontation likelihood

### 3. Factor System
Each factor has:
- **Factor Value Slider**: Controls the intensity/level of that factor (0.0-1.0 scale, displayed as 0-100%)
- **Factor Weight Display**: Shows the learned weight for each factor (controlled by the neuron)
- **Factor Name**: Clear, descriptive title
- **Information Button**: Click to reveal factor description

### 4. Factors (11 Weighted + 1 Bias)
The following factors influence confrontation likelihood:

#### Personal Factors (4 weighted + Emotional State as bias)
- **Issue Confidence**: How confident you are on the issue (Low to High)
- **Social Confidence**: How confident you are with interacting with people (Low to High)
- **Job Security**: How likely you are to keep your job (In Jeopardy to Highly Secure)
- **Issue Importance**: How much you care about the issue (Don't Care to Deeply Care)
- *Emotional State*: Controls the bias term (see section 2 above)

#### Superior Factors (3)
- **Superior Approachability**: How easy it is to talk to your superior (Extremely Intimidating to Extremely Approachable)
- **Bias Alignment**: How much your biases on the issue align with your superior (Strongly Opposing to Strongly in Agreement)
- **Superior Accountability**: How likely the superior is to implement changes based on your suggestion (Very Unlikely to Highly Likely)

#### Outside Factors (4)
- **Peer Support**: How much your peers support you on the issue (No Support to Highly Supported)
- **Alternative Options**: How many other ways there are to approach the issue (No Alternatives to Many Good Alternatives)
- **Outside Superior Support**: Support from other superiors or managers (No Support to Highly Supportive)
- **Escalation Probability**: How likely the issue is to escalate (Very Unlikely to Highly Likely) - *Controls speech bubble color*

## Calculation Method

The tool uses a single perceptron (neuron) with sigmoid activation:

### Perceptron Formula
```
z = Σ(wᵢ × xᵢ) + bias
probability = σ(z) = 1 / (1 + e^(-z))
```

Where:
- wᵢ = learned weight for factor i (11 weighted factors)
- xᵢ = factor value (0-1 normalized)
- bias = derived from Emotional State: `(emotionalState - 0.5) * 2` (maps 0-1 to -1 to +1)
- σ = sigmoid activation function

### Bias Calculation
The bias is not randomly learned but directly controlled by the Emotional State factor:
- Calm (0%) → Bias = -1 (decreases confrontation probability)
- Neutral (50%) → Bias = 0 (no baseline shift)
- Highly Emotional (100%) → Bias = +1 (increases confrontation probability)

### Speech Bubble Display
- **Size**: Based on confrontation probability from neuron output (small: <25%, medium: 25-50%, large: 50-75%, xlarge: >75%)
- **Color**: Based on Escalation Probability factor value:
  - Green (#16a34a): Low risk (0-33%) - safe to confront
  - Yellow (#eab308): Medium risk (33-66%) - caution advised
  - Red (#dc2626): High risk (66-100%) - dangerous to confront
- **Position**: Positioned above the scene using `bottom: calc(100% + 20px)`, grows upward to avoid covering figures

### Example Calculation
```
Given:
- Weights: [0.5, 0.3, -0.2, 0.4, 0.6]
- Bias: -0.5
- Factor Values: [0.8, 0.6, 0.4, 0.7, 0.9]

z = (0.5×0.8) + (0.3×0.6) + (-0.2×0.4) + (0.4×0.7) + (0.6×0.9) + (-0.5)
z = 0.4 + 0.18 - 0.08 + 0.28 + 0.54 - 0.5 = 0.82

probability = 1 / (1 + e^(-0.82)) ≈ 0.69 = 69%

Result: Large green speech bubble showing "69%"
```

## User Interface Requirements

### Layout
- Single-page application split into two halves
- Left half: Factor controls
- Right half: Visual graphic display
- Mobile-responsive (stacks vertically on small screens)
- Clean, modern design

### Components

#### Left Half - Controls
1. **Header Section**
   - Title: "Confrontation Decision Neuron"
   - Font size: Large (28px)
   - Sticky header that remains visible when scrolling
   - Shadow appears under header only when scrolled

2. **Slider Definitions**
   - Placed directly under the title
   - Two definitions:
     - **Factor Value**: The factor's current level/intensity
     - **Factor Weight**: Magnitude of factor's influence on the probability

3. **Mode Switcher**
   - Three buttons: Prediction Mode, Decision Boundary, Training Mode
   - Active mode highlighted
   - Switches both left panel controls and right panel view

4. **Emotional State (Bias) Section**
   - Located in the header, below the mode switcher
   - Title: "Emotional State (Bias)"
   - Description explaining its effect on the neuron
   - Slider with "Calm" to "Emotional" labels
   - Real-time bias value display
   - Yellow accent border to distinguish from other sections

5. **Factor List (Prediction Mode)**
   - 11 factors displayed vertically (excludes Emotional State)
   - Each factor includes:
     - Factor name with information button (ℹ️)
     - Value slider (0-100%)
     - Weight display bar showing learned weight

6. **Neuron Parameters Section**
   - Displays current weights for each factor (11 weights)
   - Shows bias value (controlled by Emotional State)
   - Formula display: σ(Σ wᵢxᵢ + b)
   - Reset Weights button

6. **Layout Stability**
   - Both panels fixed at 50vw width
   - Scrollbar gutter reserved to prevent layout shift
   - Consistent width across all modes

#### Right Half - Visual Display
Three view modes available:

1. **Home View (Prediction Mode)**
   - Simplified illustration showing:
     - Tall figure in suit (representing superior) - right side
     - Shorter figure in black polo (representing employee) - left side
     - Speech bubble above the scene showing probability
   - Background: Green striped pattern with tan floor
   - Speech bubble positioned to never cover figures

2. **Decision Boundary View**
   - 2D heatmap showing probability landscape (red → white → green)
   - Black contour line at the 50% decision threshold
   - Draggable crosshair dot with dashed crosshair lines
   - Axis selectors for X/Y factors
   - Legend showing confront (green) / don't confront (red) regions

3. **Training Mode View**
   - Interactive 2D plot for adding training points
   - Click to add points, click existing points to remove
   - Visual feedback showing decision boundary updates

### Visual Design Specifications

#### Typography Hierarchy
- **Title**: 32-36px, bold, prominently placed at top of left panel
- **Factor Names & Definitions**: 14-16px, regular or semi-bold weight
- **Slider Labels & Values**: 12-14px
- **Speech Bubble Percentage**: Scales with bubble size (20-44px)

#### Character Design
- **Superior (Right Side)**
  - Tall silhouette figure
  - Dark suit (jacket and pants)
  - Side profile facing left
  - Simplified geometric shapes (rectangles for body, circle/oval for head)
  - Height: Approximately 70-75% of viewport height

- **Employee (Left Side)**
  - Shorter silhouette figure
  - Black polo shirt
  - Side profile facing right (toward superior)
  - Simplified geometric shapes
  - Height: Approximately 50-55% of viewport height
  - Positioned to create clear height differential

#### Speech Bubble Design
- **Position**: Positioned above the scene using `bottom: calc(100% + 20px)`, ensuring it never covers the figures and grows upward as size increases
- **Shape**: Circular speech bubble with centered tail that stays attached at all sizes
- **Content**: Large, bold percentage number (e.g., "67%")
- **Size Scaling** (based on confrontation probability):
  - 0-25%: 105×105px, font-size 26px
  - 26-50%: 145×145px, font-size 37px
  - 51-75%: 184×184px, font-size 47px
  - 76-100%: 224×224px, font-size 58px
- **Animation**: Smooth CSS transition (0.3s ease) when size changes
- **Color** (based on probability):
  - >66%: Green (#16a34a) - likely to confront
  - 33-66%: Yellow (#eab308) - uncertain
  - <33%: Red (#dc2626) - unlikely to confront
- **Tail**: Centered at bottom of bubble using CSS pseudo-element, stays attached regardless of bubble size

#### Layout Positioning
- Characters positioned in bottom-center of right half
- Space at top for speech bubble to grow
- Adequate spacing between characters (facing each other with ~100-150px gap)
- Background: Subtle gradient or solid color, non-distracting

### Slider Design
- **Factor Value Slider**
  - Label: "Level" or just the range endpoints (e.g., "Low → High")
  - Range: 0-100
  - Display current numeric value
  - Compact design to fit multiple factors on screen

- **Weight Slider**
  - Label: "Importance" or "Weight"
  - Range: 0-100
  - Display current numeric value
  - Visually distinct from value slider (different color/style)

### Information Button
- Small circular (i) icon next to each factor name
- On hover (desktop): Display description in tooltip
- On click (mobile): Display description in modal or expandable section
- Subtle, non-intrusive design

### Default Values
- All factor values: 50 (neutral)
- All weights: 50 (moderate importance)
- Starting probability: ~50%

## Technical Specifications

### Technology Stack
- **Frontend**: HTML5, CSS3, JavaScript (vanilla or framework)
- **No backend required**: All calculations client-side
- **Optional**: Local storage for saving configurations

### Browser Compatibility
- Modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile browsers (iOS Safari, Chrome Mobile)

### Performance
- Instant calculation updates (<16ms for 60fps)
- Smooth slider interactions
- No page reloads required

## User Experience Considerations

### Interactivity
- Real-time updates as sliders move
- Visual feedback on all interactions
- Smooth animations and transitions

### Clarity
- Tooltips or info icons explaining each factor
- Clear labeling of all controls
- Visual distinction between value and weight sliders

### Accessibility
- Keyboard navigation support
- ARIA labels for screen readers
- Sufficient color contrast
- Clear focus indicators

## Advanced Features (Implemented)

### Mode Switcher
Three buttons in the header allow switching between views:
- **Prediction Mode**: The default view with employee/superior figures and speech bubble
- **Decision Boundary**: 2D visualization of the decision space
- **Training Mode**: Interactive perceptron training interface

The left panel content updates based on the selected mode, showing relevant controls for each view. The right panel width remains consistent across all modes.

### Decision Boundary Visualization
A 2D heatmap showing the "landscape" of the neuron's decision across two selected factors:

#### Visual Elements
- **Heatmap Gradient**: Smooth color gradient showing probability landscape
  - Red (rgb(220, 38, 38)): Low probability - Don't Confront
  - White: 50% threshold
  - Green (rgb(22, 163, 74)): High probability - Confront
- **Decision Boundary Contour**: Black line marking the 50% decision threshold
- **Crosshair Dot**: Draggable marker showing current slider position
  - Black dot with white center
  - Dashed crosshair lines extend to axes
  - Drag to update X/Y factor values in real-time
- **Axis Selectors**: Dropdowns to choose which two factors map to X and Y axes

#### Interaction
- **Dragging the Dot**: Updates the corresponding factor sliders and probability display
- **Changing Emotional State (Bias)**: Shifts the entire decision boundary
- **Adjusting Other Factors**: Modifies the boundary position based on their weights
- **Real-time Updates**: View updates automatically as any slider changes

#### Legend
- Green: Confront (>50%)
- Red: Don't Confront (<50%)

### Training Mode
An interactive machine learning training interface that teaches a simple perceptron to classify confrontation decisions.

#### How to Train the Model
1. **Select your axes**: Choose which two factors you want to visualize using the X-Axis and Y-Axis dropdowns
2. **Add "Yes" points**: Click the "Yes" button, then click on the graph to place green points where you believe confrontation is appropriate
3. **Add "No" points**: Click the "No" button, then click on the graph to place red points where you believe confrontation is not appropriate
4. **Train the model**:
   - Click "Step" to run one training iteration and watch the decision boundary adjust
   - Click "Train" to run 100 iterations automatically
5. **Observe the results**: Watch as the decision boundary (dashed line) moves to separate your Yes and No points
6. **Check accuracy**: The accuracy percentage shows how well the model classifies your training points
7. **Iterate**: Add more points or click "Reset" to start over with a new configuration

#### Adding/Removing Points
- Click anywhere on the 2D plot to add training points
- Select "Yes" or "No" label before clicking to assign the point's class
- Click on an existing point to remove it

#### Training Controls
- **Step Button**: Advances one perceptron training iteration
- **Train Button**: Runs 100 iterations automatically
- **Reset Button**: Clears all points and resets weights to zero

#### Statistics Display
- **Weight 1 (X)**: Current weight for the X-axis factor
- **Weight 2 (Y)**: Current weight for the Y-axis factor
- **Bias**: Current bias term
- **Step Counter**: Number of training iterations performed
- **Accuracy**: Percentage of correctly classified training points

#### Visual Feedback
- Decision regions update as weights change during training
- Decision boundary line shows current classification threshold
- Points are color-coded (green = Yes/Confront, red = No/Don't Confront)

### Perceptron Learning Algorithm
The training mode implements a simple perceptron:
```
For each training point:
  activation = w1 * x + w2 * y + bias
  prediction = activation >= 0 ? 1 : 0
  error = label - prediction

  if error != 0:
    w1 += learning_rate * error * x
    w2 += learning_rate * error * y
    bias += learning_rate * error
```
- Learning rate: 0.1
- Points are processed sequentially in a cycle

## Optional Enhancements

### Phase 2 Features
- **Presets**: Pre-configured scenarios (e.g., "Minor Complaint", "Serious Violation", "Friendly Relationship")
- **Explanations**: Show which factors are contributing most to the probability
- **History**: Track how probability changes over time
- **Comparison**: Compare multiple scenarios side-by-side
- **Export**: Download results as PDF or share via link
- **Data Persistence**: Save configurations to local storage
- **Advanced Mode**: Allow users to add custom factors
- **Breakdown View**: Visual chart showing contribution of each factor
- **Recommendations**: Suggest which factors to improve for higher success probability

### Visual Enhancements
- Animated probability meter
- Color gradient based on probability level
- Charts showing factor contributions
- Dark mode option

## Use Cases

1. **Personal Decision-Making**: Individual contemplating confronting their manager
2. **Training Tool**: HR or management training on workplace dynamics
3. **Research**: Academic study of workplace confrontation patterns
4. **Coaching**: Career coaches helping clients navigate difficult conversations

## Ethical Considerations

### Disclaimers
- Tool provides estimates, not guarantees
- Should be used as one input among many for decision-making
- Not a substitute for professional advice
- Results depend entirely on user's subjective assessments

### Privacy
- No data collection or tracking
- All calculations performed locally
- No user information stored on servers

## Success Metrics

### User Engagement
- Time spent adjusting factors
- Number of different scenarios explored
- Repeat usage

### Usability
- Ease of understanding the interface
- Clarity of factor descriptions
- Intuitiveness of calculation logic

## Implementation Phases

### Phase 1 (MVP) - COMPLETED
- Core perceptron-based probability calculation
- All 12 input factors with value sliders (5 Personal, 3 Superior, 4 Outside)
- Basic UI with split-screen design
- Dynamic speech bubble sizing (based on confrontation probability)
- Dynamic speech bubble coloring (based on escalation probability)
- Neuron parameters display (weights, bias)

### Phase 2 - COMPLETED
- Decision Boundary visualization with draggable marker
- Training Mode with perceptron learning
- Mode switcher buttons for view navigation
- Real-time updates across all views
- Stable layout with consistent panel widths
- Speech bubble positioning that never covers figures

### Phase 3 (Future)
- Advanced features (export, save, compare)
- Detailed analytics
- Preset scenarios
- Mobile-responsive improvements

## File Structure
```
8DecisionNeuron/
├── index.html          # Single-file application (HTML, CSS, JS combined)
└── spec.md             # This specification document
```

Note: The application is implemented as a single HTML file containing all CSS and JavaScript inline, following the project's coding standards for single-file projects.

## Sample Factor Definitions (JSON)
```json
{
  "factors": [
    { "id": "issue_confidence", "name": "Issue Confidence", "description": "How confident you are on the issue", "range": "Low to High", "category": "Personal" },
    { "id": "social_confidence", "name": "Social Confidence", "description": "How confident you are with interacting with people", "range": "Low to High", "category": "Personal" },
    { "id": "emotional_state", "name": "Emotional State (Bias)", "description": "Controls the neuron bias term - higher values increase confrontation likelihood", "range": "Calm to Highly Emotional", "category": "Bias", "controlsBias": true },
    { "id": "job_security", "name": "Job Security", "description": "How likely you are to keep your job", "range": "In Jeopardy to Highly Secure", "category": "Personal" },
    { "id": "issue_importance", "name": "Issue Importance", "description": "How much you care about the issue", "range": "Don't Care to Deeply Care", "category": "Personal" },
    { "id": "superior_approachability", "name": "Superior Approachability", "description": "How easy it is to talk to your superior", "range": "Extremely Intimidating to Extremely Approachable", "category": "Superior" },
    { "id": "bias_alignment", "name": "Bias Alignment", "description": "How much your biases on the issue align with your superior", "range": "Strongly Opposing to Strongly in Agreement", "category": "Superior" },
    { "id": "superior_accountability", "name": "Superior Accountability", "description": "How likely the superior is to implement changes based on your suggestion", "range": "Very Unlikely to Highly Likely", "category": "Superior" },
    { "id": "peer_support", "name": "Peer Support", "description": "How much your peers support you on the issue", "range": "No Support to Highly Supported", "category": "Outside" },
    { "id": "alternative_options", "name": "Alternative Options", "description": "How many other ways there are to approach the issue", "range": "No Alternatives to Many Good Alternatives", "category": "Outside" },
    { "id": "outside_superior_support", "name": "Outside Superior Support", "description": "Support from other superiors or managers", "range": "No Support to Highly Supportive", "category": "Outside" },
    { "id": "escalation_probability", "name": "Escalation Probability", "description": "How likely the issue is to escalate (controls speech bubble color)", "range": "Very Unlikely to Highly Likely", "category": "Outside" }
  ]
}
```

## Conclusion
This specification documents the Confrontation Decision Neuron, an interactive tool that demonstrates how a simple perceptron makes decisions based on 12 input factors across three categories (Personal, Superior, Outside). The tool provides three modes: prediction (see the probability), visualization (understand the decision boundary), and training (teach the model). The speech bubble size reflects confrontation probability while its color reflects escalation risk. The implementation uses a single HTML file with embedded CSS and JavaScript, following the project's single-file coding standards.