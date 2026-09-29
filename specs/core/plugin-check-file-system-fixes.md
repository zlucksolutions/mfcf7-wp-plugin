# Spec: Plugin Check File-System and Header Fixes

## Status
Implemented

## Type
Bug Fix

## Goal
Fix the remaining Plugin Check errors found after the text-domain rename ([[text-domain-fix]]): a missing License header field, an invalid "Tested up to" version format, and direct PHP file-system calls (`chmod()`, `move_uploaded_file()`, `unlink()`, `rmdir()`) that WordPress.org's coding standards forbid in favor of `WP_Filesystem`/`wp_delete_file()`.

## Current State
- The main file's header docblock had no `License:` / `License URI:` fields (Plugin Check: `plugin_header_no_license`), even though `readme.txt` already declared GPLv2.
- `readme.txt` `Tested up to:` was `7.1.2`, a three-segment version; Plugin Check requires major.minor only (`invalid_tested_upto_minor`).
- `mfcf7_zl_multilinefile_validation_filter()` and `mfcf7_zl_multilinefile_validation_filtero()` (the two near-identical validation functions noted in `CLAUDE.md`'s Undocumented Architecture section) each contain an unconditional `continue;` inside their file loop and a `return $result;` right after it. Everything written after those two points — a second, unused upload/zip implementation using `move_uploaded_file()` and `chmod()` — never executes; Contact Form 7 core handles the actual file move.
- `mfcf7_zl_multilinefile_remove()`, a helper called from several live validation-failure branches, used raw `unlink()`/`rmdir()`. Its `$new_files` argument is always an empty array in every current call site (the only line that ever populated it lived inside the dead code above), so its body has never actually run, but it's live, reachable code and was still flagged.
- `mfcf7_zlchange_attachments()` (the real, live zip-and-attach logic that runs on `wpcf7_before_send_mail`) calls `@chmod($zipped_files, 0440)` on the freshly created zip. This one genuinely executes on every multi-file submission.

## Proposed Change
1. Add `License: GPLv2 or later` / `License URI: https://www.gnu.org/licenses/gpl-2.0.html` to the main file's header docblock, matching `readme.txt`.
2. Change `Tested up to: 7.1.2` to `Tested up to: 7.1` (confirmed by the plugin owner as the actual WordPress version tested against).
3. Delete the dead code after `continue;` / `return $result;` in both validation functions (the unused `move_uploaded_file()`/`chmod()` path). No behavior change: this code never ran.
4. Rewrite `mfcf7_zl_multilinefile_remove()` to lazily init `WP_Filesystem` and use `wp_delete_file()` and `$wp_filesystem->rmdir()` instead of raw `unlink()`/`rmdir()`. Since `$new_files` is always empty today, this still never executes in practice — the change only satisfies the scanner and keeps the helper usable if it's ever wired up correctly later.
5. In `mfcf7_zlchange_attachments()`, lazily init `WP_Filesystem` the same way and use `$wp_filesystem->chmod()` instead of raw `chmod()`. This one is live, so it was verified to behave the same (owner/group-read-only permission on the generated zip) with direct filesystem access, which is what almost all hosts use.

Why this approach: two validation functions are explicitly called out in `CLAUDE.md` as having dead code after `continue`/`return`; removing it is a direct cleanup rather than working around it. The one genuinely live `chmod()` gets a real fix since it can't be deleted.

## Affected Files
- `multiline-files-upload-for-contact-form-7.php`: header License fields, dead code removed from both validation functions, `mfcf7_zl_multilinefile_remove()` and `mfcf7_zlchange_attachments()` rewritten to use `WP_Filesystem`/`wp_delete_file()`
- `readme.txt`: `Tested up to: 7.1`

## Risks
- `WP_Filesystem()` can prompt for FTP credentials on hosts without direct file-system access. This only matters for the live `chmod()` call in `mfcf7_zlchange_attachments()`; on direct-access hosts (the large majority) it's a no-op wrapper. If `$wp_filesystem` fails to init, the code now skips the `chmod()` silently instead of erroring, which is safe but means the permission tightening is skipped on those rare hosts (previously `@chmod()` would have silently failed there too).
- Manual test: submit a Contact Form 7 form with 2+ files in a `[multilinefile]` field and confirm the ZIP still attaches to the email as before.

## Implementation Notes
- Done as planned. `php -l` passes; final sweep confirms no remaining raw `chmod()`/`move_uploaded_file()`/`unlink()`/`rmdir()` calls in the file.
- Deviation: `mfcf7_zl_multilinefile_remove()` was kept (not deleted) even though it's currently a no-op, since removing it meant touching 12 call sites across both validation functions for no functional gain.
- Manual test passed: submitted a form with 2+ files, received the ZIP attachment by email; single-file submission arrived as a single attachment. Confirms the `WP_Filesystem`-based `chmod()` in `mfcf7_zlchange_attachments()` didn't break attachment delivery.

## Change Request History
