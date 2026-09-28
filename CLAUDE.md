# CLAUDE.md

## What This Is
MultiLine Files for Contact Form 7 (free edition) is a WordPress plugin by Zluck Solutions. It adds a `[multilinefile]` form field to Contact Form 7, so visitors can attach many files, adding them a batch at a time. When two or more files are sent, the site owner receives them as one ZIP attachment. A paid Pro edition, sold on Envato, lives in a separate codebase. Full product description: `PRODUCT_OVERVIEW.md`.

## Stack
- PHP 7.4+ WordPress plugin with no Composer and no autoloading; everything is procedural functions
- Hard dependency: Contact Form 7 (uses its form-tag, validation, submission and Tag Generator v2 APIs)
- jQuery (WordPress's bundled copy) for the front-end and admin scripts
- Plain CSS with no build step
- PHP `ZipArchive` extension for zipping attachments
- Translations: gettext `.po`/`.mo` files, text domain `zl-mfcf7`

## Key Commands
There is no package manager, build, lint or test tooling. Edit the files directly.
- PHP syntax check: `/d/wamp64/bin/php/php8.3.28/php.exe -l <file>.php`
- Manual testing: copy or symlink the repo into a WordPress site's `wp-content/plugins/multiline-files-for-contact-form-7/` folder, with Contact Form 7 active.
- After editing a `.po` file, regenerate its `.mo` with Poedit or `msgfmt`.

## Folder Map
- `/` (root): the two PHP files that make up the whole plugin, plus `readme.txt` (the WordPress.org listing)
  - `multiline-files-upload-for-contact-form-7.php`: main file, the form field, validation and zipping
  - `multiline-admin.php`: the Tag Generator panel, admin notices and the deactivation feedback pop-up
- `css/`: front-end and admin styles
- `js/`: front-end upload behaviour and the admin deactivation pop-up
- `images/`: the icon shown in the Pro upsell notice
- `languages/`: English and Spanish translations
- `specs/`: specs for spec-driven development (see Rules)

## Naming Rules
- The product name is written several ways in the code: "MultiLine files for Contact Form 7", "Multiline files upload for contact form 7", and the short form "MFCF7". In new user-facing text, use **MultiLine Files for Contact Form 7**.
- The WordPress.org slug is `multiline-files-for-contact-form-7`, but the main file is `multiline-files-upload-for-contact-form-7.php`, and `readme.txt` tells FTP users to use the folder name `multiline-files-upload-for-contact-form-7`. The admin JS finds the Deactivate link by an ID made from the folder name (`#deactivate-multiline-files-for-contact-form-7`), so the popup only works when the folder is named after the slug.
- Script file names are spelled `multine` (missing an "li"). This is known; don't fix it (see `.claude/rules/team-rules.md`).

## Undocumented Architecture
This file is read in full every session, every line here is a fixed cost paid on every task. Keep this section lean; a feature-specific invariant belongs in that feature's own spec, not here.
- **Contact Form 7 saves the uploaded files, not this plugin.** The field is registered with the `file-uploading` feature, so Contact Form 7 core moves the files. This plugin's validation functions only check type, size and count. The code after `continue;` / `return $result;` inside them never runs.
- **Zipping happens at send time** in `mfcf7_zlchange_attachments` (`wpcf7_before_send_mail`). It **replaces** the Mail tab's attachment list with the ZIP or single files, and it also zips Contact Form 7's own `[file]` fields. Mail (2) is not handled.
- **Validation errors are attached to the button, not the field.** The tag name is renamed to `{name}-zl-mfcf7-upld-btn` before calling `invalidate()`, so the error shows next to the upload button.
- **One field per form or page.** The front-end JS clones a hidden template input using the fixed IDs `#mfcf7_zl_add_file` and `#mfcf7_zl_multifilecontainer`. Supporting multiple fields is a Pro feature.
- **There are two near-identical validation functions** (`..._validation_filter` and `..._validation_filtero`), chosen by Contact Form 7 version. A fix to one usually needs the same fix in the other.
- **Deactivation is deferred.** The pop-up sets the option `mfcf7_zl_plugin_deactivate_request`, and the plugin deactivates itself on the next `admin_init`. "Submit & Deactivate" sends the reason to an external Google Form, plus the admin's email and site URL only if the unticked consent box is ticked (see `specs/admin/deactivation-feedback-privacy.md`).

## Rules
Coding and team rules are in `.claude/rules/` (`team-rules.md`, `wordpress-plugin.md`, `design-system.md`). Feature specs are in `specs/`, with the process in `specs/WORKFLOW.md`.

## Session Startup
At the start of every session:
1. Read this file.
2. Read `specs/WORKFLOW.md`.
3. Read `specs/INDEX.md` to learn which specs exist (names, paths and one-line descriptions only; don't open the specs).
4. Wait for a task. Only read a specific spec when a relevant task comes in.

Available commands:
- `/sdd-audit`: read and report on a module; changes nothing
- `/sdd-spec`: create a new spec for a new feature
- `/sdd-change`: update an existing spec when requirements change
- `/sdd-implement`: implement an approved spec
- `/sdd-cleanup`: repo-wide housekeeping pass; run occasionally, not part of daily work

## Communication
Always respond in plain, simple English. Avoid technical jargon where possible. Keep responses short and clear unless the task genuinely needs detail.
