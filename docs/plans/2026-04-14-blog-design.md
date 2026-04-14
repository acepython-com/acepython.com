# Blog Design for AcePython.com

## Summary

Add markdown-based blog capabilities to acepython.com using Pelican's built-in features. The blog lives at `/blog/` as a separate section. The homepage remains unchanged. A persistent navbar provides navigation between Home and Blog.

## Approach

Pure Pelican — no new dependencies. Pelican already supports Markdown articles, tags, categories, pagination, feeds, and Pygments syntax highlighting. We create the missing templates and configure URL routing.

## URL Structure

| Page | URL |
|------|-----|
| Blog listing | `/blog/` (paginated, 10/page) |
| Individual post | `/blog/{slug}/` |
| Tag page | `/blog/tags/{tag}/` |
| Category page | `/blog/category/{category}/` |
| All tags | `/blog/tags/` |
| RSS feed | `/feeds/all.atom.xml` |

## Markdown Post Format

```markdown
Title: My First Post
Date: 2026-04-14
Category: tutorials
Tags: python, django
Slug: my-first-post

Post content here with **markdown** and code blocks.
```

Files live in `content/` directory.

## Templates

### New Templates (in `themes/acepython/templates/`)

- **`navbar.html`** — DaisyUI navbar partial with "Home" and "Blog" links. Included in `base.html`.
- **`article_list.html`** — Blog listing page. Posts in reverse chronological order showing title, date, category, tags, summary. Paginated.
- **`article.html`** — Single post view. Title, date, author, category, tags at top. Rendered markdown with Pygments syntax highlighting.
- **`tag.html`** — Posts filtered by tag.
- **`category.html`** — Posts filtered by category.
- **`tags.html`** — Overview of all tags.

### Modified Templates

- **`base.html`** — Add navbar include before content block. Add Pygments CSS link. Add Atom feed autodiscovery link tag.
- **`index.html`** — No changes.

## Syntax Highlighting

- Pelican uses Pygments via the `codehilite` Markdown extension (included with `pelican[markdown]`).
- Generate a Pygments CSS file at `themes/acepython/static/css/pygments.css`.
- Use a theme that complements DaisyUI's emerald theme (e.g., `default` or `friendly`).
- Reference from `base.html`.

## RSS/Atom Feeds

- Already configured in `publishconf.py`: `FEED_ALL_ATOM = "feeds/all.atom.xml"`.
- Add `<link rel="alternate">` autodiscovery tag in `base.html` `<head>`.
- Add RSS link in navbar or blog listing.

## Styling

- All templates use existing DaisyUI + Tailwind classes.
- Blog content area uses Tailwind `prose` class for clean markdown typography.
- Pygments theme chosen to complement emerald DaisyUI theme.

## Pelican Config Changes

Add to `pelicanconf.py`:

- `ARTICLE_URL = 'blog/{slug}/'`
- `ARTICLE_SAVE_AS = 'blog/{slug}/index.html'`
- `INDEX_SAVE_AS = 'blog/index.html'`
- `TAG_URL = 'blog/tags/{slug}/'`
- `TAG_SAVE_AS = 'blog/tags/{slug}/index.html'`
- `TAGS_URL = 'blog/tags/'`
- `TAGS_SAVE_AS = 'blog/tags/index.html'`
- `CATEGORY_URL = 'blog/category/{slug}/'`
- `CATEGORY_SAVE_AS = 'blog/category/{slug}/index.html'`
- Enable `codehilite` and `extra` Markdown extensions.

## Sample Content

Create one example blog post in `content/` so templates can be tested during development.

## No New Dependencies

Everything is built into Pelican 4.11+ and the existing `pelican[markdown]` install.
