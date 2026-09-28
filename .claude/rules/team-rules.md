# Team Rules

These rules apply to every task in this repo, by any team member or Claude session.

## Workflow
- Follow `specs/WORKFLOW.md`. Don't change behaviour that has a spec without going through that spec first.
- Implementation notes go in the spec's Implementation Notes, never in `CLAUDE.md` or a rules file.
- Each rule lives in exactly one rules file. Elsewhere, point to it; don't copy it.

## Comments
- Comment *why*, not *what*. Don't add comments that repeat what the code says.
- Match the comment density of the surrounding code. Don't add a comment block to every function you touch.
- Don't leave commented-out code behind. Delete it; git keeps the history. (Old commented-out code already exists here. Remove it only as part of a spec'd cleanup, not as a side effect of an unrelated change.)
- Don't leave dated or throwaway comments (for example `/*01-03-2024*/`, `//old`, `//new`, `//debug`).

## Naming
- **Never rename an existing name just to match product wording or to fix spelling.** This covers files, functions, hooks, CSS classes and IDs, option keys and form-tag names. Many are public contracts that live on customers' sites (see `wordpress-plugin.md`, "Public contracts"). Examples that stay as they are: `zl-multine-*.js`, `mfcf7_zlchange_attachments`, `multilinefile`.
- New names follow the conventions in `wordpress-plugin.md`, "Naming".

## AI prompts
- This plugin makes no AI calls today. If one is ever added, keep each prompt in its own clearly named constant or file (not inline in logic), with a version number. Record any prompt change in the owning spec's Change Request History.

## Secrets and outbound data
- Never commit keys, tokens or credentials. The plugin has none today; keep it that way.
- Any new request that sends data off the customer's site must be listed in the owning spec and in the `== Privacy Policy ==` section of `readme.txt`. The existing Google Form feedback submission in `multiline-admin.php` is not currently disclosed there.
