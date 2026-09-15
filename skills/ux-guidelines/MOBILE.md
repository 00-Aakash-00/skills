# Mobile Feel

Loaded when a surface ships to phones: touch interactions, app-like viewport layout, or "feels like a website, not an app" complaints. Web-platform mechanics only — they apply in any framework.

## Touch is not hover

- Hover styles fire on tap and then stick until the next tap. Scope every hover style to devices that actually hover: `@media (hover: hover) and (pointer: fine)`.
- The gray/blue tap flash marks the page as a website: set `-webkit-tap-highlight-color: transparent` and provide your own pressed state instead.
- Taps feel laggy when feedback waits for the click event: give visual feedback on pointer-down, and set `touch-action: manipulation` on interactive elements to remove the double-tap delay.
- Long-press on a control selects its label: `user-select: none` on buttons and controls — never on content, where selecting text is a feature.

## Viewport honesty

- Mobile browser chrome makes `100vh` lie. Size app shells with `100dvh` (tracks the collapsing toolbar) and heroes with `100svh` (the stable smallest size, so nothing jumps on scroll).
- An input with a font size below 16px makes iOS zoom into the field on focus: keep every input at 16px or larger.
- Content stops at the notch unless asked: `viewport-fit=cover` on the viewport meta plus `env(safe-area-inset-*)` padding on fixed edges — backgrounds extend under the hardware, controls stay clear of it.
- Pull-to-refresh and scroll chaining hijack app-like scrolling: `overscroll-behavior: none` on `html, body` — only where the page owns its scroll; keep the native refresh where it is genuinely wanted.

## Gesture surfaces

A horizontal carousel or slider that also scrolls the page vertically fights the user on every drag: `touch-action: pan-y` on the gesture surface, so horizontal drags belong to the widget and vertical drags to the page.

## The device is the truth

- Set `theme-color` per color scheme so the browser's status bar matches the app's surface in both light and dark.
- Right in desktop Chrome means nothing on a phone: verify on real hardware — device emulation misses tap highlights, zoom-on-focus, safe areas, and toolbar-resize behavior.

Bar: on a real phone — no stuck hover, no tap flash, no zoom-on-focus, no selected button labels, no hijacked scroll, no content trapped by the notch, and browser chrome that matches the app's colors.
