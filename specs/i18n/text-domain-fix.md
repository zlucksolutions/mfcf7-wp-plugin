# Spec: Text Domain Rename to Match Plugin Slug

## Status
Implemented

## Type
Bug Fix

## Goal
Fix the `WordPress.WP.I18n.TextDomainMismatch` errors reported by Plugin Check: the plugin's text domain must be `multiline-files-for-contact-form-7` (the WordPress.org slug), not `zl-mfcf7`. Since WordPress 4.6, translate.wordpress.org only auto-loads official translations when the text domain matches the plugin slug exactly, so the current domain silently blocks any translations submitted there.

## Current State
- `Text Domain: zl-mfcf7` in the main file header (`multiline-files-upload-for-contact-form-7.php`).
- Every `__()`, `_e()`, `esc_html__()`, `esc_html_e()`, `esc_attr_e()` call across `multiline-admin.php` (57 occurrences) and `multiline-files-upload-for-contact-form-7.php` (17 occurrences) uses `'zl-mfcf7'`.
- Translation files are `languages/zl-mfcf7-en_US.po/.mo` and `languages/zl-mfcf7-es_ES.po/.mo`.
- `.claude/rules/wordpress-plugin.md` states "Text domain is always `'zl-mfcf7'`", which is now out of date and must be corrected as part of this fix.
- Two unrelated Plugin Check findings, fixed alongside this since they're in the same files and just as small:
  - Missing `translators:` comments above two `__()`/`sprintf()` calls with a `%s` placeholder (`multiline-admin.php:57` and `:204`).
  - Two `__()` calls in `mfcf7_plugin_meta_links()` (`multiline-files-upload-for-contact-form-7.php:783`) have no domain argument at all.

## Proposed Change
1. Change `Text Domain: zl-mfcf7` to `Text Domain: multiline-files-for-contact-form-7` in the main file header.
2. Replace every `'zl-mfcf7'` text-domain argument with `'multiline-files-for-contact-form-7'` in both PHP files.
3. Rename the translation files to match: `languages/multiline-files-for-contact-form-7-en_US.po/.mo` and `-es_ES.po/.mo`. Update the `msgid`-less header block in each `.po` (`X-Domain`, `Project-Id-Version` if it names the old domain) and recompile the `.mo` files so they still match the `.po` content exactly.
4. Add the missing `translators:` comments (describing what `%s` is) above the two flagged `__()`/`sprintf()` calls.
5. Add the missing `'multiline-files-for-contact-form-7'` domain argument to the two `__()` calls in `mfcf7_plugin_meta_links()`.
6. Update `.claude/rules/wordpress-plugin.md` so "Text domain is always `'zl-mfcf7'`" reads `'multiline-files-for-contact-form-7'`.

No visible strings change (only the domain argument and file names), so no new `.po` entries are needed beyond the two new translator comments, which don't change the `msgid`s either.

Why this approach: it's a pure mechanical rename with no behavior change, matches what WordPress.org's Plugin Check requires, and unblocks official translations going forward.

## Affected Files
- `multiline-files-upload-for-contact-form-7.php`: header, ~17 domain arguments, `mfcf7_plugin_meta_links()` domain args
- `multiline-admin.php`: ~57 domain arguments, two translator comments
- `languages/zl-mfcf7-en_US.po/.mo` → `languages/multiline-files-for-contact-form-7-en_US.po/.mo`
- `languages/zl-mfcf7-es_ES.po/.mo` → `languages/multiline-files-for-contact-form-7-es_ES.po/.mo`
- `.claude/rules/wordpress-plugin.md`: text domain naming rule

## Risks
- If a `.po`/`.mo` file pair goes out of sync during the rename (recompiled from a stale copy), translations could silently stop showing. Verify the rebuilt `.mo` matches the renamed `.po` for both locales.
- If any occurrence of `'zl-mfcf7'` is missed, that string won't translate; verify with a repo-wide search after the change.
- Anyone with `zl-mfcf7-*.mo` already cached by a translation plugin loses it — expected and unavoidable with a domain rename.

## Implementation Notes
- Done as planned: header and all ~74 domain arguments renamed in both PHP files, `.po`/`.mo` files renamed with `git mv` (content unchanged, so the existing `.mo` compilation still matches), `wordpress-plugin.md` and `CLAUDE.md` updated. Both files pass `php -l`.
- Added the two missing `translators:` comments and the two missing domain arguments found in the same Plugin Check run, since they were trivial and in the same files.
- `$mfcf7_btn_tag_name = 'zl-mfcf7-upld-btn'` in the main file was left as is — it's a form-tag name suffix, not the text domain, and is a protected public name per `wordpress-plugin.md`.
- The still-Draft `specs/admin/deactivation-popup-redesign.md` was updated to reference the new domain/filenames so it doesn't reintroduce the old ones when implemented.

## Change Request History
