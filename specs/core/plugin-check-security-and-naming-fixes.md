# Spec: Plugin Check Security and Naming Fixes

## Status
Implemented

## Type
Bug Fix

## Goal
Fix the last round of Plugin Check warnings: unverified `$_GET`/`$_FILES` reads, non-prefixed global variables, non-prefixed hook names, a discouraged `load_plugin_textdomain()` call, and a readme/header plugin-name mismatch.

## Current State
- `multiline-admin.php`'s `mfcf7_zl_notice_ignor_temp()` (hooked to `admin_init`) read `$_GET['mfcf7_zl_pro_ver_notice_ignor']` / `$_GET['mfcf7_zl_rating_notice_ignor']` and called `update_option()` with no nonce check. The three links that set these ("No Thanks", "Maybe Later", "Already Rated") carried no nonce either.
- `multiline-files-upload-for-contact-form-7.php`'s `mfcf7_zl_warning_if_cf7_deactivated()` checked/unset `$_GET['activate']` with no nonce — this only suppresses WordPress's own "Plugin activated" notice and never stores or outputs the value.
- Both `wpcf7_validate_multilinefile` filter callbacks read `$_FILES[$name]` directly. These run inside Contact Form 7's own submission pipeline, which verifies the submission before invoking any `wpcf7_validate_*` filter; there's no separate nonce for our plugin to check here.
- Two global variables, `$latest_contact_form_7` and `$latest_contact_form_7new`, had no plugin prefix.
- Three filters (`cf7_multilinefile_atts`, `cf7_multilinefile_input`, `cf7_multilinefile_max_size`) are flagged for the same reason, but they're documented public contracts (see `wordpress-plugin.md`) — sites may already hook into them under these exact names.
- `mfcf7_plugin_init()` called `load_plugin_textdomain()`, which WordPress has discouraged since 4.6: once the text domain matches the plugin's WordPress.org slug (done in [[text-domain-fix]]), translations load automatically.
- `readme.txt`'s `=== MultiLine Files for Contact Form 7 ===` didn't match the main file header's `Plugin Name: MultiLine files for Contact Form 7` (lowercase "files").

## Proposed Change
1. Add a nonce (`wp_nonce_url()`, action `mfcf7_zl_notice_ignor`) to the three notice-dismiss links, and verify it with `wp_verify_nonce()` in `mfcf7_zl_notice_ignor_temp()` before any `update_option()` call.
2. Add a `phpcs:ignore` comment to the `$_GET['activate']` check, explaining why no nonce applies (nothing read or stored).
3. Add `phpcs:disable`/`phpcs:enable` blocks around the `$_FILES` reads in both validation filters, explaining that Contact Form 7 core verifies the submission before calling them.
4. Rename `$latest_contact_form_7` → `$mfcf7_zl_latest_contact_form_7` and `$latest_contact_form_7new` → `$mfcf7_zl_latest_contact_form_7new` everywhere (declarations, `global` statements, commented-out lines, for consistency).
5. Leave the three `cf7_multilinefile_*` filter names as they are (protected public contracts); add a `phpcs:ignore` comment at each call site recording why, instead of silently leaving an unexplained warning.
6. Delete `mfcf7_plugin_init()` and its `plugins_loaded` hook entirely, since it's now redundant.
7. Change the main file's `Plugin Name:` header to `MultiLine Files for Contact Form 7` to match `readme.txt` (the casing `CLAUDE.md` already designates as the preferred form for new text).

Why this approach: fixes the genuine gaps (the two `update_option()` calls behind plain GET links) with real nonce checks, while treating the two false-positive categories (CF7-verified `$_FILES` access, protected public filter names) with documented suppressions instead of risky or contract-breaking changes.

## Affected Files
- `multiline-admin.php`: nonce added to notice-dismiss links and their handler
- `multiline-files-upload-for-contact-form-7.php`: phpcs ignores, global variable renames, filter-name ignores, `load_plugin_textdomain()` removed, plugin name header fixed

## Risks
- If the nonce action string (`mfcf7_zl_notice_ignor`) ever differs between the link and the check, the dismiss links silently stop working (existing option-based fallback via `get_transient()` still applies after the timed re-show). Verified both link-generation and check sites use the same string.
- Manual test: click "No Thanks" / "Maybe Later" / "Already Rated" on a fresh install and confirm each notice stays dismissed for its expected duration.
- Manual test: submit a `[multilinefile]` form normally to confirm the `$_FILES` phpcs changes didn't alter behavior (they're comment-only).

## Implementation Notes
- Done as planned. `php -l` passes on both files; grep confirms no remaining reference to the removed `load_plugin_textdomain()`/`mfcf7_plugin_init()`.

## Change Request History
