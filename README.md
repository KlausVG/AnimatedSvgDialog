      
# SVG Interactive Dialogue System

This project provides a lightweight, browser-based engine for creating interactive dialogue scenes using SVG graphics with animations. It is driven by a JSON file that defines the sequence of dialogue, text effects, animations (SMIL and CSS), and user interaction triggers.

## Features

*   **JSON-Driven Sequences:** Define dialogue steps, speakers, and flow in a simple JSON file.
*   **SVG Integration:** Loads and displays external SVG files.
*   **Text Effects:** Supports various effects for displaying dialogue text:
    *   `typewriter`: Text appears character by character.
    *   `blurReveal`: Text fades in, unblurs, and translates up character by character.
    *   `staticBlurReveal`: Text is positioned correctly, then fades in and unblurs character by character.
    *   `fadeIn`: The entire line fades in at once.
    *   `none`: Text appears instantly.
*   **Variable Text Speed:** Control the speed of `typewriter` and `blurReveal`/`staticBlurReveal` effects via JSON.
*   **Animation Triggering:**
    *   Trigger SMIL animations (`<animate>`, `<animateTransform>`, etc.) embedded within the SVG using `beginElement()`.
    *   Trigger CSS animations by adding/removing classes to SVG elements.
*   **Multiple Animations:** Trigger several animations simultaneously within a single dialogue step.
*   **Animation Control:**
    *   **Stop on Next:** Most animations automatically stop when the user proceeds to the next step.
    *   **Persistent Animations:** Mark specific animations (`persistent: true`) to continue playing across steps.
    *   **Synced Animations:** Synchronize SMIL animations (`sync: "dialogue"`) to start with the text effect and end when the text finishes displaying.
*   **Progression Triggers:** Advance the dialogue sequence via:
    *   `click` (on the dialogue box)
    *   `keypress` (configurable key, defaults to Space)
    *   `timer` (automatic advance after a duration)
*   **Visual Indicators:** Displays a blinking `▶` when waiting for user input, and `■` at the end of the sequence.
*   **Self-Contained Runtime:** Runs entirely in the browser using plain JavaScript, HTML, and CSS (no external JS libraries required unless you add GSAP support later).

## File Structure

```
your-project-folder/s
├── index.html # The main HTML file containing the runtime engine (CSS+JS)
├── your-scenario.json # Defines the dialogue sequence, animations, and triggers
└── your-image.svg # Your SVG graphic, potentially with embedded SMIL animations
```
      
## Setup

1.  **Prepare Files:** Make sure you have `index.html`, your `your-scenario.json` file, and your `your-image.svg` file ready.
2.  **Crucially: Use a Local Web Server:** Browsers restrict loading local files (`.json`, `.svg`) directly from an HTML file opened via `file:///`. You **must** serve the files using a local HTTP server.
    *   **Method 1: Python 3 (Commonly Available)**
        *   Open your terminal/command prompt.
        *   Navigate (`cd`) to `your-project-folder/`.
        *   Run: `python3 -m http.server`
        *   Open your browser to `http://localhost:8000` (or the port specified).
    *   **Method 2: Node.js/npm**
        *   Navigate (`cd`) to `your-project-folder/`.
        *   Run: `npx serve`
        *   Open your browser to the `http://localhost:xxxx` address it provides.
    *   **Method 3: VS Code Live Server Extension**
        *   Install the "Live Server" extension.
        *   Open `your-project-folder/` in VS Code.
        *   Right-click `index.html` and choose "Open with Live Server".

## Usage Workflow

1.  **Create/Edit SVG (`your-image.svg`):**
    *   Design your SVG graphic.
    *   **Add unique `id` attributes** to any element you want to target with CSS animations (e.g., `<path id="pupille">`) or that contains SMIL animations you want to trigger.
    *   If adding SMIL animations (`<animate>`, `<animateTransform>`):
        *   Place the `<animate>` tag **inside** the element it modifies.
        *   Give the `<animate>` tag a unique `id` if you want to trigger it via JSON (e.g., `<animate id="speak">`).
        *   Set `begin="indefinite"` on animations you want to trigger via JSON.
        *   For path morphing (`attributeName="d"`), ensure the path data structures in `values` are compatible (this is tricky without JS libraries!).
2.  **Define Scenario (`your-scenario.json`):**
    *   Specify the `svgUrl` pointing to your SVG file.
    *   Create an array of `steps`, defining dialogue, effects, animations, and triggers (see structure below).
3.  **Configure HTML (`index.html`):**
    *   Find the `<body>` tag.
    *   Update the `data-scenario-url` attribute to point to the correct path of your JSON file (e.g., `data-scenario-url="./your-scenario.json"`).
4.  **Run:** Start your local web server in the project folder and open `index.html` via the `http://localhost:...` address.

## JSON Structure (`your-scenario.json`)

```json
{
  "svgUrl": "./path/to/your-image.svg", // Relative path from HTML to your SVG file
  "steps": [
    // --- Example Step ---
    {
      "stepId": "unique-step-name", // Optional, but good for reference/debugging
      "dialogue": "This is the text that will appear. Can include <b>HTML</b> tags.", // The dialogue line
      "speaker": "Character Name", // Optional: Speaker name (not currently displayed, but available)
      "textEffect": "typewriter", // Effect to use: "typewriter", "blurReveal", "staticBlurReveal", "fadeIn", "none"
      "textEffectSpeed": 50, // Optional: Speed override (ms/char for typewriter/blur, stagger delay for reveals)
      "animation": [ // Can be null, a single animation object, or an array of objects
        {
          // --- SMIL Example ---
          "type": "smil",
          "targetId": "smil-animation-id-in-svg", // ID of the <animate> tag
          "action": "begin", // Currently only "begin" is implemented for triggering
          "sync": "dialogue", // Optional: Set to "dialogue" to start with text and stop when text finishes.
          "persistent": false // Optional: Set to true to prevent animation from stopping on nextTrigger
        },
        {
          // --- CSS Example ---
          "type": "css",
          "targetId": "svg-element-id", // ID of the SVG element (<path>, <circle>, etc.)
          "action": "addClass", // Currently only "addClass" is implemented
          "className": "your-css-animation-class", // CSS class defined in index.html <style>
          "persistent": false // Optional: Set true to prevent class removal on nextTrigger
        }
      ],
      "nextTrigger": { // Defines how to move to the next step
        "type": "click", // "click", "keypress", "timer", or null for the last step
        // --- Options based on type ---
        "key": "Enter", // Optional (for "keypress"): Specify key (e.g., "Enter", "ArrowRight"). Defaults to Space (" ").
        "duration": 3000 // Required (for "timer"): Duration in milliseconds before auto-advancing.
      }
    }
    // ... more step objects
  ]
}
```
    


## SVG Requirements 

*  Elements targeted by CSS or containing SMIL need unique ids.

*  SMIL animate / animateTransform tags triggered by JSON need unique ids.

*  SMIL animate tags triggered by JSON must have begin="indefinite".

*  SMIL tags must be nested inside the SVG element they are intended to animate.

## Customization

* Styling: Modify the CSS within the style tags in index.html to change the appearance of the dialogue box, text, SVG container, etc.

* Behavior: Modify the JavaScript within the script tags in index.html to change text effects, add new animation types, or alter the core engine logic.

## Limitations

* No UI Editor: Scenario creation requires manually editing the JSON file.

* SMIL Path Morphing: Reliably morphing complex paths (attributeName="d") with SMIL is difficult due to potential path structure mismatches. 


## Troubleshooting

* NetworkError / 404 Errors Loading JSON/SVG: You MUST run index.html via a local web server (http://localhost...), not directly as a file:///... URL. See Setup section.

* SMIL Animation Not Found: Double-check the targetId in JSON matches the id in the SVG exactly (case-sensitive). Ensure the animate tag is correctly nested inside the element it animates in the SVG file. Ensure the SVG file saved correctly and clear browser cache (hard refresh). Use browser dev tools (Elements tab) to inspect the loaded SVG structure.

* Animation Doesn't Stop: Ensure the persistent flag is not set to true in the JSON for the animation you expect to stop. Check the nextStep function's cleanup logic in the script.
