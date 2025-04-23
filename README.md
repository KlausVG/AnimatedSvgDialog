      
# SVG Interactive Dialogue System (v2 - Choices & Explicit Flow)

This project provides a lightweight, browser-based engine for creating interactive dialogue scenes using SVG graphics with animations. It is driven by a JSON file that defines the sequence of dialogue, text effects, animations (SMIL and CSS), branching choices, and user interaction triggers.

## Features

*   **JSON-Driven Sequences:** Define dialogue steps, speakers, effects, animations, choices, and explicit flow control in a simple JSON file using `stepId`s.
*   **SVG Integration:** Loads and displays external SVG files specified in the JSON.
*   **Text Effects:** Supports various effects for displaying dialogue text:
    *   `typewriter`: Text appears character by character.
    *   `blurReveal`: Text fades in, unblurs, and translates up character by character.
    *   `staticBlurReveal`: Text is positioned correctly, then fades in and unblurs character by character.
    *   `fadeIn`: The entire line fades in at once.
    *   `none`: Text appears instantly.
*   **Variable Text Speed:** Control the speed of `typewriter`, `blurReveal`, and `staticBlurReveal` effects via the optional `textEffectSpeed` property in JSON steps.
*   **Animation Triggering:**
    *   Trigger SMIL animations (`<animate>`, `<animateTransform>`, etc.) embedded within the SVG using `beginElement()`.
    *   Trigger CSS animations by adding/removing classes to SVG elements.
*   **Multiple Animations:** Trigger several animations simultaneously within a single dialogue step by using an array for the `animation` property.
*   **Animation Control:**
    *   **Stop on Next:** Most animations automatically stop when the user proceeds to the next step.
    *   **Persistent Animations:** Mark specific animations (`persistent: true`) to continue playing across steps.
    *   **Synced Animations:** Synchronize SMIL animations (`sync: "dialogue"`) to start with the text effect and end *only* when the text finishes displaying (requires compatible SMIL setup, e.g., no `fill="freeze"` on loops).
*   **Dialogue Choices:** Implement branching narratives by presenting choices to the player within a step using the `choices` array.
*   **Choice Input:** Players can select choices using:
    *   Mouse clicks.
    *   Keyboard number keys ('1', '2', '3'...).
*   **Progression Triggers:** For non-choice steps, define how to advance using `nextTrigger`:
    *   Requires an explicit `nextStepId` to define the target step.
    *   Supports trigger types: `click` (on dialogue box), `keypress` (configurable key, defaults to Space), `timer`.
    *   Supports **multiple triggers** (e.g., advance on *either* click *or* keypress) by providing an array of trigger objects.
*   **Visual Indicators:**
    *   Displays a blinking `▶` when waiting for a standard `nextTrigger` (click/key/timer).
    *   Displays a blinking `▼` when waiting for the player to make a choice.
    *   Displays a solid `■` at the final step of a sequence path (when no `nextTrigger` or `choices` are defined).
*   **Local Storage Persistence:** Remembers the player's current step index and resumes from that point on page reload. Includes a "Reset Progress" button.
*   **Self-Contained Runtime:** Runs entirely in the browser using plain JavaScript, HTML, and CSS.

## File Structure

```
your-project-folder/
├── index.html # Main HTML file containing the runtime engine (CSS+JS)
├── your-scenario.json # Defines the dialogue sequence, animations, choices, etc.
└── your-image.svg # Your SVG graphic (potentially with SMIL animations)
```
      
## Setup

1.  **Prepare Files:** Place `index.html`, `your-scenario.json`, and `your-image.svg` in the same folder.
2.  **Crucially: Use a Local Web Server:** Browsers restrict loading local files (`.json`, `.svg`) directly from an HTML file opened via `file:///`. You **must** serve the files using a local HTTP server.
    *   **Method 1: Python 3**
        *   In terminal: `cd your-project-folder/`
        *   Run: `python3 -m http.server`
        *   Open browser to `http://localhost:8000`.
    *   **Method 2: Node.js/npm**
        *   In terminal: `cd your-project-folder/`
        *   Run: `npx serve`
        *   Open browser to the `http://localhost:xxxx` address provided.
    *   **Method 3: VS Code Live Server Extension**
        *   Install "Live Server".
        *   Open folder in VS Code.
        *   Right-click `index.html` -> "Open with Live Server".

## Usage Workflow

1.  **Create/Edit SVG (`your-image.svg`):**
    *   Design graphic.
    *   **Add unique `id` attributes** to elements targeted by CSS or containing SMIL animations.
    *   If adding SMIL (`<animate>`):
        *   Nest inside the element it modifies.
        *   Add unique `id` if triggering via JSON.
        *   Set `begin="indefinite"` if triggering via JSON.
        *   Avoid `fill="freeze"` on looping animations intended for `sync: "dialogue"` control.
        *   Be wary of `attributeName="d"` morphing compatibility.
2.  **Define Scenario (`your-scenario.json`):**
    *   Specify `svgUrl`.
    *   Create `steps` array.
    *   **Crucially:** Give each step a unique `stepId`.
    *   Ensure every step that is *not* the final step has **either** a `choices` array (with valid `nextStepId`s) **or** a `nextTrigger` object/array containing valid `nextStepId`(s). Use `null` for `nextTrigger` on the final step of a path.
    *   (See **JSON Structure** below).
3.  **Configure HTML (`index.html`):**
    *   Update `data-scenario-url` in `<body>` to point to your JSON file.
4.  **Run:** Start local server and open `index.html` via `http://localhost:...`. Use the "Reset Progress" button if needed.

## JSON Structure (`your-scenario.json`)

```json
{
  "svgUrl": "./your-image.svg",
  "steps": [
    // --- Example Step with Trigger(s) ---
    {
      "stepId": "intro", // REQUIRED unique ID for this step
      "dialogue": "Text here. <b>HTML</b> ok.",
      "speaker": "Character", // Optional
      "textEffect": "typewriter", // "typewriter","blurReveal","staticBlurReveal","fadeIn","none"
      "textEffectSpeed": 50, // Optional speed override
      "animation": [ /* null, single object, or array */
        {
          "type": "smil", // "smil" or "css"
          "targetId": "smil-anim-id",
          "action": "begin",
          "sync": "dialogue", // Optional
          "persistent": false // Optional
        },
        { /* ... other animations ... */ }
      ],
      // --- Use EITHER nextTrigger OR choices ---
      "nextTrigger": { // Single trigger example
        "type": "click", // "click", "keypress", "timer"
        "nextStepId": "next-step-after-click" // **** REQUIRED ****
        // "key": "Enter", // Optional for keypress
        // "duration": 3000 // Required for timer
      }
      // --- OR Multiple Triggers Example ---
      // "nextTrigger": [
      //   { "type": "click", "nextStepId": "next-step-id" }, // REQUIRED
      //   { "type": "keypress", "key": "Enter", "nextStepId": "next-step-id" } // REQUIRED
      // ]
    },
    // --- Example Step with Choices ---
    {
        "stepId": "decision-point", // REQUIRED unique ID
        "dialogue": "Make your choice.",
        "textEffect": "staticBlurReveal",
        "animation": null,
        // --- Use EITHER nextTrigger OR choices ---
        "choices": [
            {
              "text": "Option One", // Text displayed (numbering added automatically)
              "nextStepId": "step-after-option-one" // REQUIRED: Target step ID
            },
            {
              "text": "Option Two (<b>HTML</b> ok)",
              "nextStepId": "step-after-option-two" // REQUIRED
            }
        ]
        // No nextTrigger needed here
    },
    // --- Final Step Example ---
    {
        "stepId": "end-of-path", // REQUIRED unique ID
        "dialogue": "The sequence ends here.",
        "textEffect": "fadeIn",
        "animation": null,
        "nextTrigger": null // **** Indicates the end of this sequence path ****
        // No choices here either
    }
    // ... more step objects
  ]
}

``` 

## SVG Requirements

   * Elements targeted by CSS or containing SMIL need unique ids.

  * SMIL <animate>/<animateTransform> tags triggered by JSON need unique ids.

  * SMIL <animate> tags triggered by JSON must have begin="indefinite".

  * SMIL tags must be nested inside the SVG element they animate.

  * Avoid fill="freeze" on looping SMIL animations using sync: "dialogue" if clean stopping is desired.

## Customization

  * Styling: Modify CSS in index.html's <style> tags. Includes styles for dialogue, indicators (#next-indicator), choices (.choice-item), text effects, etc.

  * Behavior: Modify JavaScript in index.html's <script> tag.


## Troubleshooting

  * NetworkError / 404 Errors Loading Files: Run via a local web server (http://...), not file:///.... See Setup.

  * SMIL Animation Not Found/Working: Check id matching (case-sensitive), nesting in SVG, begin="indefinite". Verify correct file is loaded (clear cache). Use browser dev tools (Elements/Console) to inspect loaded SVG and script logs. Check fill="freeze" on synced animations.

  * Incorrect Branching / Step Flow: Ensure every non-final step has either choices or a nextTrigger with a valid nextStepId. Check stepIds match exactly. Use console logs in goToStep to trace flow.

  * Syntax Errors / No Dialogue: Check the browser console (F12) for JavaScript errors. Validate JSON syntax. Check for script copy-paste errors.

  * Progress Not Saving/Resuming: Ensure localStorage is enabled. Check console for saving/loading messages/errors. Use "Reset Progress" button if data seems corrupted.

  * Choices Not Working: Ensure the step uses the choices array (not nextTrigger). Check JSON structure for text and nextStepId. Verify click/keydown listeners are being attached correctly (check logs from setupChoiceInputHandlers).