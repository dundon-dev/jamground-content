# jamground-content

**Content only.** This repository holds `content/` and nothing else — no `package.json`,
no `astro.config`, no build scripts, and no workflows beyond the single thin gate
described below.

That is a security boundary, not tidiness. Editors get write access to branches here,
and a preview build runs on push. If build configuration sat beside editor-writable
bytes, an editor's token — or anyone who obtained one — could modify the build that
executes on the box which also serves production. Keeping the two apart makes "content
is data, never executed" structural rather than aspirational. This repository will
never carry a build config for that reason, regardless of how convenient it would be to
add one.

See ADR-0020 in the site repository.

## Layout

```
content/
  pages/
    <locale>/          # one directory per configured locale, e.g. en-US/
  posts/
    <locale>/
  authors/
    <locale>/
  navigation/
    <locale>/
  media/               # originals, locale-neutral
  settings/
```

The kind comes first, the locale second — `content/pages/en-US/home.yaml`, not
`content/en-US/pages/home.yaml`. Exactly one locale directory deep within each kind —
the site's collection glob is `<kind>/*/*`, not `<kind>/**/*`, and that is what enforces
it. `media/` is a directory, not a collection; `settings/` is locale-neutral and is read
at config time rather than as a collection.

## The envelope

Every entity under `content/` — a page, a post, an author, a navigation menu — carries
the same metadata block ahead of its own fields, serialised in the order the contract
declares:

- **`id`** — an immutable ULID, assigned once at creation and never reused. Never the
  filename; filenames carry no meaning here.
- **`translationOf`** — the ULID of the entity's **translation group**: the id shared by
  every locale's version of the same conceptual entity. This is what a `ref:` (in
  navigation items and in link-bearing blocks) or a post's `author:` points at —
  **never** the entity's own `id`.
- **`locale`** — must match the containing locale directory.
- **`slug`** — mutable and URL-facing, unique per collection and locale. Changing it is
  an ordinary content edit, not a change of identity: `id` and `translationOf` do not
  move when `slug` does.
- **`title`**
- **`status`** — `draft` or `published`, decided per locale.
- **`publishedAt`** — required once `status` is `published`; on a draft it doubles as a
  scheduling date.
- **`updatedAt`**

Optional, in addition: **`slugHistory`** (every previous slug, so a rename still
redirects), **`seo`** (`title`, `description`, `ogImage`, `noindex`), and
**`sourceHash`** (set on a translation when it is created, to detect drift from its
source).

Everything after that is specific to the kind: a page's `blocks`, a post's `author` /
`excerpt` / `tags` / `related`, an author's `name` / `role` / `bio` / `avatar`,
navigation's `items`.

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
