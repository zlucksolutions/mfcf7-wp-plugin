# Spec: Deactivation Pop-up Redesign

## Status
Draft

## Type
Improvement

## Goal
Give the deactivation feedback pop-up a clean, modern look that matches current WordPress admin dialogs, and make it easier to use (keyboard, screen readers, small screens). What is sent and when stays exactly as set by `specs/admin/deactivation-feedback-privacy.md`. Ships in 3.1.2, after the 3.1.1 privacy release.

## Current State
- Markup: `mfcf7_zl_deactivation_popup()` in `multiline-admin.php`, printed on every admin page through `admin_footer`.
- Styles: the "For deactivation popup" block in `css/admin-style.css` (`.admin-popup-container`, `.admin-popup-content`, `.admin-popup-close`, `.mfcf7-modal-header`, `.mfcf7-modal-footer`, `#loader`, `.loader-circle`, `.mfcf7-zl-consent`).
- Behaviour: `js/zl-multine-admin-files.js` shows and hides the pop-up by the classes and IDs above.
- Known problems:
  - No dimmed backdrop. `#custom-plugin-modal-overlay` exists but has no styles, so the page behind stays fully visible.
  - No fixed width on desktop; the box grows and shrinks with its text.
  - Plain radio list with little spacing; hard to scan and small click targets.
  - Both buttons look the same (`button-secondary`), so the main action isn't clear.
  - The close "X" is a `<span>`, not a button: it can't be reached with the keyboard and has no label for screen readers.
  - No Escape-key close, no click-outside close, no focus moved into the dialog, and no `role="dialog"`.
  - On short screens the box can't scroll, so the buttons can end up off-screen.
  - The loading spinner is `position: fixed` at an odd spot on the page, not next to the buttons.
  - Heading "MFCF7 Feedback" uses the short product name.
  - "Cancel & Deactivate" calls `location.reload()` straight after starting its AJAX request, so the reload can cut the request off before deactivation is saved (follow-up noted in the privacy spec).

## Proposed Change
Layout and style only, plus the small accessibility and reload fixes. The data sent, the consent checkbox (unticked by default), the radio `value`s and the AJAX actions do not change.

1. **Backdrop and card.** Style `#custom-plugin-modal-overlay` as a full-screen dimmed backdrop. The pop-up becomes a centred white card, 520px wide (max 100% minus 32px), rounded corners, soft shadow, `max-height: 90vh`, with the body scrolling if needed so the header and buttons stay visible.
2. **Header.** Title "Quick feedback", with a one-line subtitle: "Before you deactivate MultiLine Files for Contact Form 7, tell us why. It takes a few seconds." The close "X" becomes a real `<button>` with `aria-label="Close"` in the top-right corner.
3. **Reasons as selectable rows.** Each reason is a full-width bordered row, with the whole row clickable. On hover the row gets a light grey background; when selected it gets a border in the admin's own theme colour. The "Other" text box appears inside its row when chosen.
4. **Consent area.** Keep the checkbox and the privacy note unchanged in wording. Show the note in WordPress's standard small grey `.description` style, under the checkbox.
5. **Footer.** Buttons right-aligned. "Submit & Deactivate" becomes the main button (`button button-primary`), and "Cancel & Deactivate" stays `button button-secondary`. While sending, show the existing spinner inside the footer next to the buttons and disable both buttons, so they can't be double-clicked.
6. **Keyboard and screen reader support.**
   - Add `role="dialog"`, `aria-modal="true"` and `aria-labelledby` (the title) to the card.
   - When the pop-up opens, focus moves to the first reason. When it closes, focus returns to the Deactivate link.
   - Escape and a click on the backdrop close the pop-up, the same as the "X": nothing is sent and the plugin stays active.
   - Tab stays inside the pop-up while it is open.
7. **Small screens (480px and below).** The card uses the full width minus a 16px margin on each side, and the buttons stack full width with "Submit & Deactivate" on top.
8. **Fix the Cancel reload race.** In the "Cancel & Deactivate" handler, remove the extra `location.reload()` that runs straight away, so the page reloads only after the AJAX request finishes (success or error).
9. **Keep existing hooks.** Keep all current IDs and classes that the JS uses (`.admin-popup-container`, `#custom-plugin-modal`, `#custom-plugin-modal-overlay`, `#custom-plugin-deactivate-form`, `.cancel-deactivate-button`, `.button-close`, `#loader`, `input[name="selected-reason"]`, `input[name="mfcf7_zl_consent"]`). New classes use the `mfcf7-zl-` prefix.
10. **Translations.** New or changed text ("Quick feedback", the subtitle, "Close") is wrapped in `zl-mfcf7` functions and added to both `.po` files. The `.mo` files are rebuilt.
11. **Release as 3.1.2.** Update the `Version:` header, `Stable tag:` and add a changelog entry ("Improved: modern, accessible design for the deactivation feedback pop-up.").

Why this approach: it looks at home in the WordPress admin, reuses WordPress's own buttons, colours and `.description` style, and fixes the real usability gaps without touching the privacy behaviour that was just reviewed.

## Affected Files
- `multiline-admin.php`: pop-up markup (header, close button, reason rows, ARIA attributes, button classes)
- `css/admin-style.css`: rewrite of the "For deactivation popup" block, including the 480px rules
- `js/zl-multine-admin-files.js`: selected-row class, Escape/backdrop close, focus handling, disabled buttons while sending, Cancel reload fix
- `.claude/rules/design-system.md`: add the new admin colour values below to the Admin table
- `languages/zl-mfcf7-en_US.po`, `languages/zl-mfcf7-es_ES.po` and their `.mo` files: new strings
- `readme.txt`: `Stable tag: 3.1.2`, changelog entry
- `multiline-files-upload-for-contact-form-7.php`: `Version: 3.1.2`

## Design
- **Screen:** the deactivation feedback pop-up on the Plugins screen.
- **Reused:** WordPress `button`, `button-primary`, `button-secondary` and `.description` classes; the existing `#eee` divider, `#fff` panel, spinner, and 480px breakpoint.
- **New colour values** (all taken from WordPress core admin; to be added to the Admin table in `design-system.md`):

| Use | Value |
|---|---|
| Backdrop | `rgba(0, 0, 0, 0.6)` |
| Selected row border | `var(--wp-admin-theme-color, #2271b1)` (follows the admin's chosen colour scheme) |
| Row border | `#dcdcde` |
| Row hover background | `#f6f7f7` |
| Card shadow | `0 8px 24px rgba(0, 0, 0, 0.2)` |

- **Spacing:** card padding 24px; 8px gap between reason rows; row padding 10px 12px; card corner radius 8px, row radius 4px.
- **Type:** inherits WordPress admin fonts and sizes. Title uses the admin's standard `h2` size. No new fonts or font sizes.
- **Reference:** WordPress core's own modal dialogs (for example the "Add Media" and block editor dialogs).
- **Layout sketch:**

```
┌──────────────────────────────────────┐
│ Quick feedback                    ✕  │
│ Before you deactivate …, tell us why │
├──────────────────────────────────────┤
│ ┌──────────────────────────────────┐ │
│ │ ○ I found a better plugin        │ │
│ └──────────────────────────────────┘ │
│ ┌──────────────────────────────────┐ │
│ │ ● This plugin does not work …    │ │ ← selected: theme-colour border
│ └──────────────────────────────────┘ │
│   … more reasons, then "Other"       │
│ ──────────────────────────────────── │
│ ☐ You can contact me by email …      │
│ small grey privacy note + links      │
├──────────────────────────────────────┤
│   [Cancel & Deactivate] [Submit & Deactivate] │
└──────────────────────────────────────┘
```

## Risks
- Renaming or removing an ID or class the JS relies on would break the buttons. Item 9 lists what must stay.
- The CSS must not leak onto other admin screens: keep every new rule scoped under `.admin-popup-container`.
- Any change that accidentally ticks the consent box, or sends data on Escape/backdrop close, would undo the privacy fix. Re-run the privacy spec's manual tests.
- Manual tests:
  1. The pop-up opens with a dimmed backdrop and focus on the first reason.
  2. Clicking anywhere on a reason row selects it and highlights the row.
  3. "Other" shows its text box inside the row.
  4. Escape, the "X" button and a backdrop click all close it: nothing is sent and the plugin stays active.
  5. Tab and Shift+Tab stay inside the pop-up.
  6. "Submit & Deactivate" disables both buttons, shows the spinner, then deactivates.
  7. "Cancel & Deactivate" deactivates reliably (the page reloads only after the request finishes).
  8. At 480px and below, the card fits the screen and the buttons stack.
  9. On a short screen, the body scrolls and the buttons stay visible.
  10. With a non-default admin colour scheme (Users → Profile), the selected row uses that scheme's colour.
  11. The privacy spec's manual tests still pass.
  12. No layout changes on other admin screens.

## Open Questions
- **Should "Submit & Deactivate" be disabled until a reason is chosen?** Today it deactivates without sending anything if no reason is picked. Disabling it is clearer, but changes behaviour. Recommendation: keep today's behaviour.
- **Only print the pop-up on the Plugins screen?** Today the markup and admin JS load on every admin page. Printing the pop-up only on `plugins.php` is lighter and follows the asset rule in `wordpress-plugin.md`. Recommendation: yes, include it here.
- **Title wording:** is "Quick feedback" with the subtitle above OK, or do you prefer another title?

## Implementation Notes

## Change Request History
