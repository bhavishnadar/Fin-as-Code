# Intercom-Managed Content

This folder is the source of truth for your Intercom Articles, Internal Articles, and Content Snippets. Edit the files here and open a pull request — when the PR merges, your changes apply to your Intercom workspace.

This is **infrastructure as code**: the file IS the desired state. If you delete a file, the corresponding entity is archived in Intercom on merge. Renames and content edits apply on merge too. Files outside this folder are ignored by the connector.

> A pre-merge validator and GitHub Checks integration is on the roadmap. Today, your PR can merge with content the connector can't fully apply — check the post-merge sync status comment for any drops.

## Directory layout

```
intercom/
  articles/
    <collection-slug>/<article-slug>/
      meta.yaml
      en.md
      es.md
      fr.md
  internal-articles/
    <title-slug>-<id>/
      meta.yaml
      en.md
  snippets/
    <title-slug>-<id>/
      meta.yaml
      en.md
  images/
    <filename>.png
```

- **Articles** live under a collection-slug folder, with a `meta.yaml` (shared IaC fields) plus one `<lang>.md` per locale. Articles in the same collection share the parent folder.
- **Internal articles** and **snippets** each live in **their own folder** named `<title-slug>-<id>/`, holding a `meta.yaml` (identity + shared fields) plus a single `<locale>.md`. They are per-locale entities — one folder per `(entity, locale)` row — so each folder contains exactly one locale file, matching the `locale` declared in its `meta.yaml`.
- `intercom/images/` is workspace-shared — referenced from any article via relative path (see "Images" below).

## Frontmatter schema

Every entity's identity lives in its folder's `meta.yaml`. Like the `.md` locale files, `meta.yaml` is YAML wrapped in `---` frontmatter fences — keep the leading and trailing `---`, or the connector parses the file as empty and silently ignores every field. The `id` is **type-prefixed** so it's self-documenting at a glance:

- Articles: `id: art_<numeric-id>` (e.g. `id: art_12345`)
- Internal articles: `id: ia_<numeric-id>` (e.g. `id: ia_67890`)
- Snippets: `id: snip_<numeric-id>` (e.g. `id: snip_555`)

Leave `id` blank when creating a new entity. On merge, Intercom assigns one and, on the next sync, writes the entity out at its canonical `<slug>-<id>/` folder with the `id` filled into `meta.yaml`.

### Article — folder shape (`intercom/articles/<collection-slug>/<article-slug>/`)

Articles are multi-locale entities, so they split shared fields into `meta.yaml` and locale-specific fields into one `<lang>.md` per locale.

`meta.yaml`:

```yaml
---
id: art_12345                   # Intercom internal id; leave blank when creating a new article
default_locale: en
collection: getting-started     # collection slug (matches the parent folder)
target_user_type: user          # one of: user, lead, both
help_centers: []                # help-center slugs the article publishes to
exclude_from_home_screen_suggestions: false
show_related_articles: true
---
```

`<lang>.md` (one file per locale):

```markdown
---
locale: en
title: Getting started with Intercom
summary: A short summary shown in the help center search results.
author_id: 67890
published: true
chatbot_availability: true
copilot_availability: true
sales_agent_availability: false
with_table_of_contents: true
---

# Getting started

Body content here. Markdown + Intercom directives — see below.
```

### Internal article — folder shape (`intercom/internal-articles/<title-slug>-<id>/`)

Internal articles are per-locale entities — one row per `(logical-IA, locale)` pair — so each row renders to its own folder: a `meta.yaml` carrying identity and shared fields, plus a single `<locale>.md` (matching the `locale` in `meta.yaml`) carrying the locale-specific fields and body.

`meta.yaml`:

```yaml
---
id: ia_67890                    # Intercom internal id; leave blank when creating a new internal article
locale: en
audience: []                    # array of ai_content_segment IDs; [] means admin-only
owner_id: 42
---
```

`<locale>.md` (one file, named for the locale in `meta.yaml`):

```markdown
---
locale: en
title: How to escalate to senior support
author_id: 67890
published: true
chatbot_availability: true
copilot_availability: true
sales_agent_availability: false
---

# Body content
```

### Snippet — folder shape (`intercom/snippets/<title-slug>-<id>/`)

Snippets are per-locale entities, same folder shape as internal articles (a `meta.yaml` plus a single `<locale>.md`). Their `meta.yaml` carries only identity — snippets have no audience or owner fields.

Snippet bodies support a limited block set: paragraphs, headings, flat (non-nested) lists, code, inline links, and horizontal rules. Images, tables, nested lists, and the rich directives documented below (`:::callout:::`, `:::button:::`, `:::collapsible:::`, `:::video:::`, `:::attachment:::`) aren't supported and are dropped on sync — don't use them in snippets. (Images are dropped as standalone blocks, but an image written inline within a paragraph can slip through as raw `<img>` HTML rather than render, so avoid images in snippets entirely.)

`meta.yaml`:

```yaml
---
id: snip_555                    # Intercom internal id; leave blank when creating a new snippet
locale: en
---
```

`<locale>.md` (one file, named for the locale in `meta.yaml`):

```markdown
---
locale: en
title: Support phone number
author_id: 67890
published: true
chatbot_availability: true
copilot_availability: true
sales_agent_availability: true
---

# Body content
```

## Markdown vocabulary

Standard CommonMark renders as you'd expect:

- Headings (`#` through `####`)
- Paragraphs, hard breaks (two trailing spaces)
- Bold (`**bold**`), italic (`_italic_`), inline `code`
- Links: `[text](url)` and `[text](url "title")`
- Images: `![alt](url)` (top-level) — see "Images" for inline forms
- Lists (ordered, unordered, nested)
- Code fences with language hints (```` ```ruby ````)
- Tables (GitHub-flavored)
- Horizontal rules (`---`)
- Blockquotes (rendered as paragraphs — Intercom has no blockquote block)

## Directive vocabulary

Block types beyond CommonMark use **CommonMark directive syntax**: a fence opened with `:::name` and closed with a bare `:::` line. Attributes go on the opener line as `attr="value"` pairs.

### `:::callout :::` — Highlighted callout box

```markdown
:::callout
Heads up. This is a default-styled callout.
:::
```

Custom colours via `backgroundColor` and `borderColor` attributes (hex strings, matching the `Blocks::Callout#style` model fields):

```markdown
:::callout backgroundColor="#fff8c5" borderColor="#d4a72c"
A yellow-tinted callout for emphasis.
:::
```

Body is Markdown — supports nested directives (e.g. a `:::collapsible :::` inside a callout).

### `:::button :::` — Call-to-action button

Leaf directive (no body):

```markdown
:::button href="https://example.com/signup" label="Sign up"
:::
```

Required: `href`, `label`. Optional: `align`, `tracking_link_url`, `tracking_link_id`.

### `:::collapsible :::` — Expandable section

```markdown
:::collapsible title="Advanced options"
Content shown when the user expands the section.
:::
```

Required: `title`. Body is Markdown.

### `:::video :::` — Embedded video

Leaf directive:

```markdown
:::video id="dQw4w9WgXcQ" provider="youtube"
:::
```

Required: `id`, `provider`. Optional: `text`.

### `:::attachment :::` — File attachment (standalone)

Leaf directive:

```markdown
:::attachment href="https://example.com/file.pdf" name="file.pdf" content_type="application/pdf" size="12345"
:::
```

Required: `href`, `name`. Optional: `content_type`, `size`, `text`, `id`.

### `:::attachments :::` — Group of attachments

Body is one or more `:::attachment :::` directives:

```markdown
:::attachments
:::attachment href="https://example.com/a.pdf" name="a.pdf"
:::

:::attachment href="https://example.com/b.pdf" name="b.pdf"
:::
:::
```

### `:::intercom-block :::` — Opaque round-trip carrier

You should not need to write this directive by hand. Intercom emits it on outbound for block types that don't have a friendly directive form (legacy embeds, internal cards, etc.). The block payload is base64-encoded inside the `data` attribute. Editing it usually breaks parsing — leave these blocks alone unless you intend to remove the block entirely.

## Cross-references

To link to another Article, Internal Article, or Snippet from your content, use the **hybrid-link** form: a normal Markdown link whose `title` attribute carries an `intercom:<entity-type>:<id>` reference.

```markdown
See also our [pricing guide](https://example.com/pricing "intercom:article:12345").
```

The `href` is what readers see — typically your live help-center URL. The `title` is the Intercom-internal reference. On merge, the connector validates that the referenced entity exists and updates the link target to its current canonical URL.

## Images

Images are workspace-shared. Drop image files into `intercom/images/` and reference them from an Article or Internal Article using a relative path:

```markdown
![Architecture overview](../../../images/architecture.png)
```

For Articles, the relative path is `../../../images/<filename>` (three levels up from `<collection>/<article>/<lang>.md`). For Internal Articles, the relative path is `../../images/<filename>` (two levels up from `internal-articles/<slug>-<id>/<locale>.md`). Snippets don't support images, so don't add them — see the Snippet section for what snippet bodies can contain.

When a referenced image file doesn't yet exist in `intercom/images/`, add it in the same PR.

## How to add new content

### A new article

1. Pick (or create) a collection folder under `intercom/articles/`.
2. Create a new article folder named after a slug for the article (e.g. `getting-started/welcome/`).
3. Write `meta.yaml` with the article fields (leave `id` blank).
4. Write `en.md` (or your default locale) with the locale-specific frontmatter + body.
5. Add additional `<lang>.md` files for translations as needed.
6. Commit and open a PR.

### A new internal article or snippet

1. Under `intercom/internal-articles/` or `intercom/snippets/`, create a folder named after a human-readable slug (e.g. `how-to-escalate/`).
2. Write `meta.yaml` (leave `id` blank) and a `<locale>.md` for the entity's locale — see the Frontmatter schema section.
3. Commit and open a PR.

On merge, Intercom creates the entity and assigns an `id`. On the next sync it writes the entity out at its canonical `<slug>-<id>/` folder, with the `id` filled into `meta.yaml`, in a follow-up commit on your default branch. If the folder you created used a different name, your original folder is left untouched alongside the canonical one.

## How to update or delete content

- **Update**: edit the files in the entity's folder. The change applies on merge.
- **Rename a slug** (Articles only): rename the article folder. The folder name is a human-readable hint — the connector resolves the entity by `id` (in `meta.yaml`), so the rename applies cleanly. For Internal Articles and Snippets the folder name is derived from the entity's title (not a slug you control), so rename by editing the `title` in the `<locale>.md` rather than renaming the folder by hand.
- **Delete**: delete the folder. The entity is archived in Intercom on merge.
- **Move between collections** (Articles only): move the article folder into a different collection folder.

## What this connector doesn't do (yet)

- Pre-merge state (drafts, content review requests, releases) lives in Intercom — the connector applies published state only.
- The pre-merge validator and GitHub Checks integration are roadmap items.
- Marketplace install and customer-self-serve setup flows are not yet shipped.

For support, contact the team that owns this integration in your workspace's `intercom-sync.yaml` file.
