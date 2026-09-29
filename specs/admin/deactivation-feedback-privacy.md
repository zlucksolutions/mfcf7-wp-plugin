# Spec: Deactivation Feedback Privacy Fix

## Status
Implemented

## Type
Bug Fix

## Goal
Fix the WordPress.org guideline violation reported against version 3.1.0: the deactivation feedback pop-up secretly sends the site address and the admin's email address to our Google Form. After this change, the reason is sent only on "Submit & Deactivate", and the email and site address are sent only if the admin ticks an unticked consent checkbox. "Cancel & Deactivate" sends nothing. The readme's Privacy Policy is corrected to match.

## Current State
- When an admin clicks Deactivate on the Plugins screen, a feedback pop-up opens (`mfcf7_zl_deactivation_popup()` in `multiline-admin.php`, loaded on every admin page through `admin_footer`).
- The pop-up only asks for a reason. It never says that anything will be sent anywhere.
- "Submit & Deactivate" calls the AJAX action `custom_plugin_deactivate`, handled by `mfcf7_zl_custom_handle_deactivation_plugin_form_submission()`. That function sends three things to a Google Form with `wp_remote_post()`:
  - the site URL (`get_site_url()`)
  - the current admin's email address
  - the chosen reason, or the "Other" text the admin typed
- WordPress's default HTTP user-agent is `WordPress/<version>; <site URL>`, so the site address also reaches Google in the request header, on top of the form field.
- "Cancel & Deactivate" calls `deactive_plugin_without_feedback`, which sends nothing but has **no nonce check**.
- Both handlers check `current_user_can('administrator')`, which is a role name, not a capability.
- The pop-up function starts with `echo get_option('mfcf7_zl_plugin_deactivate_request');`, which prints the raw option value, unescaped, into the footer of every admin page.
- `multiline-admin.php` has no `ABSPATH` direct-access guard.
- Most pop-up strings are not translatable.
- `readme.txt` `== Privacy Policy ==` says the plugin "does not collect, store, or transmit any personal data". That is false today.
- The WordPress.org review team has asked us to either remove the transmission, or get explicit opt-in consent and correct the Privacy Policy. Their key point: "submitting feedback is not consent to disclosing one's email address, which the modal never mentions."

Relevant files: `multiline-admin.php`, `js/zl-multine-admin-files.js`, `readme.txt`, `multiline-files-upload-for-contact-form-7.php` (version header), `languages/*.po` / `*.mo`.

## Proposed Change
We take the review team's second option: **explicit opt-in consent**. The reason is anonymous feedback. The email and site address are only sent if the admin ticks a checkbox that is **unticked by default**.

1. **Add a consent checkbox to the pop-up**, below the reason list and right above the buttons. It is always unticked when the pop-up opens. Wording:
   > ☐ You can contact me by email about my feedback (this also shares my email and website address).
   - The checkbox must never be pre-ticked, and must not be ticked by JS or remembered from a previous visit. A pre-ticked box is not valid consent under WordPress.org rules or GDPR.
2. **Add a short notice under the checkbox** so it is clear what is sent in each case:
   > When you click "Submit & Deactivate", the reason you chose is sent to Zluck Solutions (through Google Forms) to help us improve the plugin. Your email and website address are sent only if you tick the box above. Nothing is sent if you click "Cancel & Deactivate" or close this window.
   - Include a link to the plugin's Privacy Policy section and to Google's privacy policy.
3. **Server sends only what the admin agreed to.** In `mfcf7_zl_custom_handle_deactivation_plugin_form_submission()`:
   - Always send the reason (`entry.1682553995`).
   - Send the site URL (`entry.1315009358`) and email (`entry.144564863`) **only if** the posted consent value is exactly `'1'`. Otherwise leave those fields out of the request entirely.
   - Always pass `'user-agent' => 'WordPress'` to `wp_remote_post()` so the site address isn't sent in the request header when the box is unticked.
   - Add a short `timeout` (5 seconds) so a slow Google response doesn't hold up deactivation.
4. **Only send when a reason is given.** If no reason is chosen, or "Other" is chosen with an empty text box, send nothing (even if the box is ticked) and just deactivate.
5. **JS sends the checkbox state.** `js/zl-multine-admin-files.js` sends `consent: 1` only when the box is ticked, and nothing (or `0`) otherwise.
6. **"Cancel & Deactivate" and the close (X) button send nothing.** This is already true; keep it and test it.
7. **Correct the readme `== Privacy Policy ==` section.** Replace the current paragraph with an accurate one:
   - Uploaded files stay on the site and are never sent to us or any third party.
   - When an admin deactivates the plugin, a feedback pop-up appears. Only if they click "Submit & Deactivate", the chosen reason is sent to a Google Form owned by Zluck Solutions.
   - The admin's email address and the website address are sent **only if** the admin ticks the unticked "you can contact me" box. They are used only to reply about the feedback.
   - Nothing is sent if they click "Cancel & Deactivate" or close the pop-up.
   - Google processes the data under its privacy policy (link). To have your data removed, contact Zluck Solutions (contact link).
8. **Security fixes in the same code** (the review team will re-check the whole plugin, so these ship together):
   - Add a nonce check to `mfcf7_zl_handle_deactivation_plugin_without_feedback()`, and send the nonce from the "Cancel & Deactivate" click in the JS.
   - Change both handlers from `current_user_can('administrator')` to `current_user_can('activate_plugins')`.
   - Remove the stray `echo get_option('mfcf7_zl_plugin_deactivate_request');` from the pop-up.
   - Add the `if (!defined('ABSPATH')) { exit; }` guard to the top of `multiline-admin.php`.
9. **Make the pop-up strings translatable** (heading, question, reasons, checkbox label, notice, buttons, placeholder) with the `zl-mfcf7` text domain. The radio `value`s sent to Google stay in English so our feedback sheet stays readable; only the visible labels are translated. Add the new strings to both `.po` files and rebuild the `.mo` files.
10. **Release as 3.1.1.** Update the `Version:` header, `Stable tag:` and add a changelog entry, e.g. "Privacy: the deactivation feedback pop-up no longer sends your email address or website address unless you tick the new, unticked 'you can contact me' box. The pop-up now explains what is sent. Nothing is sent with Cancel & Deactivate. Privacy Policy updated."

Why this approach: it matches the reviewer's words "explicit opt-in consent" directly. We keep anonymous reasons from everyone, and still get email and site address from admins who choose to share them.

Out of scope (leave for later specs): loading admin CSS/JS only on the Plugins screen, the `time()` script version, escaping in the Pro/review notices, and the same fix in the Pro edition (separate codebase, but it almost certainly has the same code and should be fixed too).

## Affected Files
- `multiline-admin.php`: consent checkbox and notice in the pop-up, handler changes, nonce, capability, `ABSPATH` guard, stray `echo` removed, strings made translatable
- `js/zl-multine-admin-files.js`: send the checkbox state with "Submit & Deactivate", and the nonce with "Cancel & Deactivate"
- `readme.txt`: Privacy Policy rewrite, `Stable tag: 3.1.1`, changelog entry
- `multiline-files-upload-for-contact-form-7.php`: `Version: 3.1.1`
- `languages/zl-mfcf7-en_US.po`, `languages/zl-mfcf7-es_ES.po` and their `.mo` files: new strings

## Design
- Screen affected: the deactivation feedback pop-up on the Plugins screen.
- Reuse the existing pop-up markup and classes. The checkbox is a normal WordPress admin `<label><input type="checkbox"> ...</label>` inside `.mfcf7-modal-body`, after the reason list, followed by the notice as a plain `<p>`. Separate them from the reasons with the existing `1px solid #eee` divider style.
- No new CSS colours or fonts (see `.claude/rules/design-system.md`). It must still read fine at 480px wide, and long email addresses or URLs must wrap, not overflow.

## Risks
- If the server trusts anything other than an exact `'1'` for consent, the email could be sent without a tick. Test with the box unticked.
- If the nonce is added to the PHP handler but not sent by the JS, "Cancel & Deactivate" will stop working. Test both buttons.
- Fewer feedback rows will have an email and site address. That is expected.
- Manual tests (log outbound requests with the `pre_http_request` filter, as the reviewer did):
  1. The pop-up opens with the checkbox unticked.
  2. Submit & Deactivate, reason chosen, box **unticked**: one request with only the reason; no email or site URL in the body; user-agent is plain `WordPress`; plugin deactivates.
  3. Submit & Deactivate, reason chosen, box **ticked**: one request with reason, email and site URL; plugin deactivates.
  4. Submit & Deactivate with "Other" + text: the typed text is sent as the reason.
  5. Submit & Deactivate with no reason chosen (ticked or not): no request, plugin still deactivates.
  6. Cancel & Deactivate: no request, plugin deactivates.
  7. Close (X), then reopen the pop-up: box is unticked again; no request sent.
  8. A user without `activate_plugins` (or a bad nonce) can't trigger either action.
  9. No stray "1" or other text printed in the admin footer.
  10. Pop-up is readable at 480px, with long emails/URLs wrapping.

## Open Questions
- **Reply to the review team.** After releasing 3.1.1, reply to their email with the version number and a short list of what changed (unticked opt-in checkbox, notice in pop-up, Privacy Policy rewrite, security fixes). Who sends this?

## Implementation Notes
- Done as planned in 3.1.1. The pop-up reasons now render from a `$reasons` array (English value → translated label); the handlers were checked with stubbed WordPress functions for all send/no-send cases.
- File-upload/attachment behavior confirmed working on a real site after the later `WP_Filesystem` changes (single file and multi-file ZIP both arrived by email). The deactivation pop-up's own manual test list (consent checkbox, nonce, Cancel/close send-nothing cases) is still to be run separately.
- Deviation (agreed): added a `.mfcf7-zl-consent` rule to `css/admin-style.css` for the divider and email/URL wrapping. Also added a 3.1.1 Upgrade Notice to `readme.txt`.
- The "Plugin privacy details" link points to the plugin's WordPress.org page, because WordPress.org has no direct anchor for the readme's Privacy Policy section.
- Follow-up: the "Cancel & Deactivate" JS still calls `location.reload()` straight after starting the AJAX request, so the reload can cut the request off before deactivation is saved (existing behaviour, not changed here).

## Change Request History
- 2026-09-28 | Shortened the consent checkbox label; it no longer shows the admin's email and site address in full | Shorter, simpler pop-up text; consent is still explicit | Divyesh Kakrecha
