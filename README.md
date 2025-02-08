# YouTube Quality, PiP & Playback Speed Hotkeys 3

This is a Violentmonkey user script that allows you to control YouTube video quality, enable Picture-in-Picture (PiP) mode, and adjust playback speed using keyboard shortcuts.

## Installation

1. Install [Violentmonkey](https://www.tampermonkey.net/](https://violentmonkey.github.io) on your browser.
2. Create a new script in Tampermonkey.
3. Copy and paste the following script into the editor.
4. Save and enable the script.

## Usage

### Quality Shortcuts
Press the following keys to change video quality:
- `z` → 240p
- `a` → 360p
- `q` → 480p
- `e` → 720p
- `x` → 144p
- `v` → 1080p

### Playback Speed
- `[` → Decrease speed by 0.25x
- `]` → Increase speed by 0.25x
- `Backspace` → Reset speed to normal (1x)
- `\` → Set speed to 2x

### Picture-in-Picture (PiP)
- `p` → Toggle Picture-in-Picture mode

```javascript
// ==UserScript==
// @name         YouTube Quality, PiP & Playback Speed Hotkeys 3
// @namespace    http://tampermonkey.net/3
// @version      1.2
// @description  Assign keyboard shortcuts to change YouTube video quality, enable PiP mode, and adjust playback speed with overlay
// @author       Lavkush Singh
// @match        *://*.youtube.com/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // Shortcuts and their respective quality
    const qualitySettings = {
        'z': '240p', // Changed from q to z
        'a': '360p',
        'q': '480p', // Changed from z to q
        'e': '720p',
        'x': '144p',
        'v': '1080p'
    };

    // Helper function to click a button with a specific label
    function clickButtonByText(buttonText) {
        const buttons = [...document.querySelectorAll('.ytp-menuitem')];
        for (let button of buttons) {
            if (button.innerText.includes(buttonText)) {
                button.click();
                break;
            }
        }
    }

    // Function to change playback speed and show the overlay
    function changePlaybackSpeed(increase) {
        const player = document.querySelector('video');
        if (player) {
            let newSpeed = player.playbackRate + (increase ? 0.25 : -0.25);
            newSpeed = Math.min(2.0, Math.max(0.25, newSpeed)); // Limit speed between 0.25x and 2x
            player.playbackRate = newSpeed;

            // Simulate the YouTube overlay by dispatching the same action
            const event = new KeyboardEvent('keydown', {
                bubbles: true,
                cancelable: true,
                keyCode: increase ? 190 : 188, // > for increase, < for decrease (these are the default keys)
                shiftKey: true // YouTube uses Shift + ,/. for speed adjustment
            });
            document.dispatchEvent(event);
        }
    }

    document.addEventListener('keydown', function(event) {
        // Skip if focus is on input or textarea or other editable elements
        if (event.target.tagName.toLowerCase() === 'input' ||
            event.target.tagName.toLowerCase() === 'textarea' ||
            event.target.isContentEditable) {
            return; // Skip if typing in an input box or editable element
        }

        const player = document.querySelector('video');
        if (!player) return;

        const qualityKey = event.key;

        // Check if the pressed key is for quality adjustment
        if (qualitySettings[qualityKey]) {
            const desiredQuality = qualitySettings[qualityKey];

            // Open settings menu
            const settingsButton = document.querySelector('.ytp-settings-button');
            if (settingsButton) settingsButton.click();

            // Wait a moment for the menu to appear and then click the "Quality" option
            setTimeout(() => {
                clickButtonByText('Quality');

                // Wait for the quality options to appear, then select the desired quality
                setTimeout(() => {
                    clickButtonByText(desiredQuality);
                }, 300); // Adjust the delay if needed
            }, 300); // Adjust the delay if needed
        }

        // Enable/Disable Picture-in-Picture mode with 'p' key
        if (event.key === 'p') {
            if (document.pictureInPictureElement) {
                document.exitPictureInPicture(); // Exit PiP if it's already enabled
            } else {
                player.requestPictureInPicture(); // Enter PiP
            }
        }

        // Custom shortcuts for playback speed adjustment
        if (event.key === '[') {
            changePlaybackSpeed(false); // Decrease playback speed
        }
        if (event.key === ']') {
            changePlaybackSpeed(true); // Increase playback speed
        }

        // Reset playback speed to 1x with Backspace key
        if (event.key === 'Backspace') {
            player.playbackRate = 1.0; // Set speed to normal (1x)
        }

        // Set playback speed to 2x with "\" key
        if (event.key === '\\') {
            player.playbackRate = 2.0; // Set speed to 2x
        }
    });
})();
```
