# jamground-content

**Content only.** This repository holds `content/` and nothing else — no `package.json`,
no `astro.config`, no build scripts, and no workflows beyond the single thin gate
described below.

That is a security boundary, not tidiness. Editors get write access to branches here,
and a preview build runs on push. If build configuration sat beside editor-writable
bytes, an editor's token — or anyone who obtained one — could modify the build that
executes on the box which also serves production. Keeping the two apart makes "content
is data, never executed" structural rather than aspirational.

See ADR-0020 in the site repository.

## Layout

```
content/
  <locale>/          # one directory per configured locale, e.g. en-US/
    pages/
    posts/
    authors/
    navigation/
  media/             # originals, locale-neutral
  settings/
```

Exactly one locale directory deep — the site's collection glob is `*/*`, not `**/*`,
and that is what enforces it. `media/` is a directory, not a collection; `settings/` is
read at config time rather than as a collection.

## Working here

Every file is schema-validated and canonically serialised. Both are enforced in CI, so a
hand edit that drifts from canonical form fails rather than being silently rewritten.

- The contract, the schemas and the invariants live in the **site** repository under
  `docs/` — `02-content-contract.md` is the one to read first.
- The build finds this repository through `JAMGROUND_CONTENT_DIR`, which points at this
  repository's **root** and defaults to `../jamground-content` (`OD-29`). Contract paths
  such as `content/media/hero-a1b2c3.jpg` resolve against it directly.

Editors do not work here by hand. They work in wp-admin through the editing shell, which
opens a change, writes the diff and opens the pull request on their behalf — no git
vocabulary, no github.com. This README is for developers.
