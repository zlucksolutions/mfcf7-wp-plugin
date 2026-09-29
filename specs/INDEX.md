# Spec Index

This file is maintained automatically by Claude.
Every time a new spec is created or a spec status changes,
this file is updated. Do not edit manually.

Every spec under specs/** (excluding WORKFLOW.md, TEMPLATE.md,
and this file) must have exactly one row here. Descriptions must
stay to one line; do not append change history into this file.

## Format
Group | File Path | Description | Status

## Specs

admin | specs/admin/deactivation-feedback-privacy.md | Deactivation pop-up sends email and site address only with an unticked opt-in checkbox; correct the Privacy Policy (WordPress.org violation) | Implemented
admin | specs/admin/deactivation-popup-redesign.md | Modern, accessible WordPress-style design for the deactivation feedback pop-up (3.1.2) | Draft
i18n | specs/i18n/text-domain-fix.md | Renamed text domain from zl-mfcf7 to multiline-files-for-contact-form-7 to match the plugin slug (Plugin Check requirement) | Implemented
core | specs/core/plugin-check-file-system-fixes.md | Removed dead file-upload code and switched to WP_Filesystem/wp_delete_file, added License header, fixed Tested up to format | Implemented
core | specs/core/plugin-check-security-and-naming-fixes.md | Nonce-protected admin notice dismiss links, documented CF7/filter-name false positives, prefixed globals, removed redundant load_plugin_textdomain, fixed plugin name mismatch | Implemented