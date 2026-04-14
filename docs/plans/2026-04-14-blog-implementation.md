# Blog Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add markdown-based blog at `/blog/` with tags, categories, syntax highlighting, RSS, and a persistent navbar.

**Architecture:** Pure Pelican — configure URL routing, create Jinja2 templates using DaisyUI/Tailwind, generate Pygments CSS, add a sample post. No new dependencies.

**Tech Stack:** Pelican 4.11+, Jinja2, DaisyUI 5, TailwindCSS (browser CDN), Pygments

---

### Task 1: Generate Pygments CSS

**Files:**
- Create: `themes/acepython/static/css/pygments.css`

**Step 1: Create the static CSS directory**

```bash
mkdir -p themes/acepython/static/css
```

**Step 2: Generate the Pygments CSS file**

```bash
cd /Users/phildini/code/acepython/acepython.com
uv run pygmentize -S friendly -f html -a .highlight > themes/acepython/static/css/pygments.css
```

**Step 3: Verify the file was created and has content**

Run: `wc -l themes/acepython/static/css/pygments.css`
Expected: ~70+ lines of CSS

**Step 4: Commit**

```bash
git add themes/acepython/static/css/pygments.css
git commit -m "feat: add Pygments syntax highlighting CSS"
```

---

### Task 2: Configure Pelican for blog URLs and Markdown extensions

**Files:**
- Modify: `pelicanconf.py`

**Step 1: Add blog URL configuration to `pelicanconf.py`**

Add the following after `DEFAULT_PAGINATION = 10`:

```python
# Blog URL structure
ARTICLE_URL = 'blog/{slug}/'
ARTICLE_SAVE_AS = 'blog/{slug}/index.html'
INDEX_SAVE_AS = 'blog/index.html'
TAG_URL = 'blog/tags/{slug}/'
TAG_SAVE_AS = 'blog/tags/{slug}/index.html'
TAGS_URL = 'blog/tags/'
TAGS_SAVE_AS = 'blog/tags/index.html'
CATEGORY_URL = 'blog/category/{slug}/'
CATEGORY_SAVE_AS = 'blog/category/{slug}/index.html'

# Disable unused default pages
AUTHOR_SAVE_AS = ''
AUTHORS_SAVE_AS = ''
ARCHIVES_SAVE_AS = ''
CATEGORIES_SAVE_AS = ''

# Markdown extensions
MARKDOWN = {
    'extension_configs': {
        'markdown.extensions.codehilite': {'css_class': 'highlight'},
        'markdown.extensions.extra': {},
        'markdown.extensions.meta': {},
    },
    'output_format': 'html5',
}

# Static files
STATIC_PATHS = ['images']
```

**Step 2: Verify config is valid**

Run: `cd /Users/phildini/code/acepython/acepython.com && uv run python -c "exec(open('pelicanconf.py').read()); print('OK')"`
Expected: `OK`

**Step 3: Commit**

```bash
git add pelicanconf.py
git commit -m "feat: configure Pelican blog URLs and Markdown extensions"
```

---

### Task 3: Update `base.html` with navbar, Pygments CSS, and feed link

**Files:**
- Modify: `themes/acepython/templates/base.html`
- Create: `themes/acepython/templates/navbar.html`

**Step 1: Create `navbar.html`**

Create `themes/acepython/templates/navbar.html` with:

```html
<div class="navbar bg-base-100 shadow-sm">
    <div class="navbar-start">
        <a href="/" class="btn btn-ghost text-xl">{{ SITENAME }}</a>
    </div>
    <div class="navbar-end">
        <ul class="menu menu-horizontal px-1">
            <li><a href="/">Home</a></li>
            <li><a href="/blog/">Blog</a></li>
            <li><a href="/feeds/all.atom.xml" title="Atom Feed">RSS</a></li>
        </ul>
    </div>
</div>
```

**Step 2: Update `base.html`**

Replace the current `base.html` with:

```html
<!DOCTYPE html>
<html lang="en" data-theme="emerald">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}{{ SITENAME }}{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/daisyui@5/themes.css" rel="stylesheet" type="text/css" />
    <link href="https://cdn.jsdelivr.net/npm/daisyui@5" rel="stylesheet" type="text/css" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script src="https://unpkg.com/htmx.org@2"></script>
    <script defer src="https://cloud.umami.is/script.js" data-website-id="19229676-7247-4ebf-9293-b3436ae96c22"></script>
    <link rel="stylesheet" href="{{ SITEURL }}/theme/css/pygments.css">
    {% if FEED_ALL_ATOM %}
    <link rel="alternate" type="application/atom+xml" title="{{ SITENAME }} Feed" href="{{ SITEURL }}/{{ FEED_ALL_ATOM }}">
    {% endif %}
</head>
<body>
    {% include "navbar.html" %}
    {% block content %}{% endblock %}
</body>
</html>
```

**Step 3: Build and verify no errors**

Run: `cd /Users/phildini/code/acepython/acepython.com && uv run pelican content -o output -s pelicanconf.py`
Expected: Builds without errors

**Step 4: Commit**

```bash
git add themes/acepython/templates/base.html themes/acepython/templates/navbar.html
git commit -m "feat: add navbar, Pygments CSS link, and Atom feed autodiscovery"
```

---

### Task 4: Create blog listing template (`article_list.html`)

**Files:**
- Create: `themes/acepython/templates/article_list.html`

**Step 1: Create the template**

Pelican uses `index.html` for the article listing by default. Since our `index.html` is the homepage, we need to use a different approach. Create `themes/acepython/templates/article_list.html` — but actually, Pelican's `INDEX_SAVE_AS` uses the template named `index.html` for the article index. We need to override this behavior.

The correct approach: Pelican renders the article index using the `index.html` template. Since we want `index.html` to remain the homepage, we need to configure Pelican to use a **direct template** for the blog listing. Update `pelicanconf.py` to add:

```python
# Use direct templates for custom pages
DIRECT_TEMPLATES = ['index', 'blog']
PAGINATED_TEMPLATES = {'blog': 10}
BLOG_SAVE_AS = 'blog/index.html'
```

And remove the earlier `INDEX_SAVE_AS = 'blog/index.html'` line (keep `INDEX_SAVE_AS` as default so the homepage works).

Then create `themes/acepython/templates/blog.html`:

```html
{% extends "base.html" %}

{% block title %}Blog — {{ SITENAME }}{% endblock %}

{% block content %}
<div class="max-w-3xl mx-auto px-4 py-12">
    <h1 class="text-4xl font-bold mb-8">Blog</h1>

    {% if articles %}
    <div class="space-y-8">
        {% for article in articles %}
        <article class="card bg-base-100 shadow-sm">
            <div class="card-body">
                <h2 class="card-title">
                    <a href="{{ SITEURL }}/{{ article.url }}" class="hover:text-primary">
                        {{ article.title }}
                    </a>
                </h2>
                <div class="text-sm opacity-60">
                    <time datetime="{{ article.date.isoformat() }}">{{ article.date.strftime('%B %d, %Y') }}</time>
                    {% if article.category %}
                    &middot; <a href="{{ SITEURL }}/{{ article.category.url }}" class="link link-hover">{{ article.category }}</a>
                    {% endif %}
                </div>
                {% if article.summary %}
                <p>{{ article.summary }}</p>
                {% endif %}
                {% if article.tags %}
                <div class="card-actions justify-start mt-2">
                    {% for tag in article.tags %}
                    <a href="{{ SITEURL }}/{{ tag.url }}" class="badge badge-outline badge-sm">{{ tag }}</a>
                    {% endfor %}
                </div>
                {% endif %}
            </div>
        </article>
        {% endfor %}
    </div>

    {% if articles_paginator.num_pages > 1 %}
    <div class="flex justify-center mt-12">
        <div class="join">
            {% if articles_page.has_previous() %}
            <a href="{{ SITEURL }}/{{ articles_previous_page.url }}" class="join-item btn">Previous</a>
            {% endif %}
            {% for num in range(1, articles_paginator.num_pages + 1) %}
            <a href="{{ SITEURL }}/blog/{% if num > 1 %}page/{{ num }}/{% endif %}"
               class="join-item btn {% if num == articles_page.number %}btn-active{% endif %}">
                {{ num }}
            </a>
            {% endfor %}
            {% if articles_page.has_next() %}
            <a href="{{ SITEURL }}/{{ articles_next_page.url }}" class="join-item btn">Next</a>
            {% endif %}
        </div>
    </div>
    {% endif %}

    {% else %}
    <p class="text-lg opacity-70">No posts yet. Check back soon!</p>
    {% endif %}
</div>
{% endblock %}
```

**Step 2: Update `pelicanconf.py`**

Replace the `INDEX_SAVE_AS = 'blog/index.html'` line with the direct template config above.

**Step 3: Build and verify**

Run: `cd /Users/phildini/code/acepython/acepython.com && uv run pelican content -o output -s pelicanconf.py`
Expected: Builds without errors, `output/blog/index.html` exists

**Step 4: Commit**

```bash
git add themes/acepython/templates/blog.html pelicanconf.py
git commit -m "feat: add blog listing page template"
```

---

### Task 5: Create single article template

**Files:**
- Create: `themes/acepython/templates/article.html`

**Step 1: Create `article.html`**

```html
{% extends "base.html" %}

{% block title %}{{ article.title }} — {{ SITENAME }}{% endblock %}

{% block content %}
<div class="max-w-3xl mx-auto px-4 py-12">
    <article>
        <header class="mb-8">
            <h1 class="text-4xl font-bold mb-4">{{ article.title }}</h1>
            <div class="text-sm opacity-60">
                <time datetime="{{ article.date.isoformat() }}">{{ article.date.strftime('%B %d, %Y') }}</time>
                {% if article.category %}
                &middot; <a href="{{ SITEURL }}/{{ article.category.url }}" class="link link-hover">{{ article.category }}</a>
                {% endif %}
                &middot; {{ article.author }}
            </div>
            {% if article.tags %}
            <div class="mt-3">
                {% for tag in article.tags %}
                <a href="{{ SITEURL }}/{{ tag.url }}" class="badge badge-outline badge-sm">{{ tag }}</a>
                {% endfor %}
            </div>
            {% endif %}
        </header>

        <div class="prose max-w-none">
            {{ article.content }}
        </div>
    </article>

    <div class="divider mt-12"></div>

    <div class="flex justify-between items-center">
        <a href="{{ SITEURL }}/blog/" class="btn btn-ghost">&larr; All Posts</a>
    </div>
</div>
{% endblock %}
```

**Step 2: Build and verify**

Run: `cd /Users/phildini/code/acepython/acepython.com && uv run pelican content -o output -s pelicanconf.py`
Expected: Builds without errors

**Step 3: Commit**

```bash
git add themes/acepython/templates/article.html
git commit -m "feat: add single article template"
```

---

### Task 6: Create tag and category templates

**Files:**
- Create: `themes/acepython/templates/tag.html`
- Create: `themes/acepython/templates/category.html`
- Create: `themes/acepython/templates/tags.html`

**Step 1: Create `tag.html`**

```html
{% extends "base.html" %}

{% block title %}Posts tagged "{{ tag }}" — {{ SITENAME }}{% endblock %}

{% block content %}
<div class="max-w-3xl mx-auto px-4 py-12">
    <h1 class="text-4xl font-bold mb-8">Tag: {{ tag }}</h1>

    <div class="space-y-4">
        {% for article in articles %}
        <div class="flex justify-between items-baseline">
            <a href="{{ SITEURL }}/{{ article.url }}" class="link link-hover text-lg">{{ article.title }}</a>
            <time datetime="{{ article.date.isoformat() }}" class="text-sm opacity-60 ml-4 shrink-0">{{ article.date.strftime('%B %d, %Y') }}</time>
        </div>
        {% endfor %}
    </div>

    <div class="mt-8">
        <a href="{{ SITEURL }}/blog/tags/" class="btn btn-ghost">&larr; All Tags</a>
    </div>
</div>
{% endblock %}
```

**Step 2: Create `category.html`**

```html
{% extends "base.html" %}

{% block title %}{{ category }} — {{ SITENAME }}{% endblock %}

{% block content %}
<div class="max-w-3xl mx-auto px-4 py-12">
    <h1 class="text-4xl font-bold mb-8">Category: {{ category }}</h1>

    <div class="space-y-4">
        {% for article in articles %}
        <div class="flex justify-between items-baseline">
            <a href="{{ SITEURL }}/{{ article.url }}" class="link link-hover text-lg">{{ article.title }}</a>
            <time datetime="{{ article.date.isoformat() }}" class="text-sm opacity-60 ml-4 shrink-0">{{ article.date.strftime('%B %d, %Y') }}</time>
        </div>
        {% endfor %}
    </div>

    <div class="mt-8">
        <a href="{{ SITEURL }}/blog/" class="btn btn-ghost">&larr; All Posts</a>
    </div>
</div>
{% endblock %}
```

**Step 3: Create `tags.html`**

```html
{% extends "base.html" %}

{% block title %}Tags — {{ SITENAME }}{% endblock %}

{% block content %}
<div class="max-w-3xl mx-auto px-4 py-12">
    <h1 class="text-4xl font-bold mb-8">Tags</h1>

    <div class="flex flex-wrap gap-3">
        {% for tag, articles in tags|sort %}
        <a href="{{ SITEURL }}/{{ tag.url }}" class="badge badge-lg badge-outline gap-2">
            {{ tag }}
            <span class="badge badge-sm">{{ articles|count }}</span>
        </a>
        {% endfor %}
    </div>

    <div class="mt-8">
        <a href="{{ SITEURL }}/blog/" class="btn btn-ghost">&larr; Blog</a>
    </div>
</div>
{% endblock %}
```

**Step 4: Build and verify**

Run: `cd /Users/phildini/code/acepython/acepython.com && uv run pelican content -o output -s pelicanconf.py`
Expected: Builds without errors

**Step 5: Commit**

```bash
git add themes/acepython/templates/tag.html themes/acepython/templates/category.html themes/acepython/templates/tags.html
git commit -m "feat: add tag, category, and tags listing templates"
```

---

### Task 7: Create a sample blog post and verify everything works

**Files:**
- Create: `content/hello-world.md`

**Step 1: Create the sample post**

Create `content/hello-world.md`:

```markdown
Title: Hello, World!
Date: 2026-04-14
Category: announcements
Tags: python, acepython
Slug: hello-world
Summary: Welcome to the AcePython blog — where we'll share Python tutorials, tips, and project updates.

Welcome to the AcePython blog! This is where I'll be sharing Python tutorials, code tips, and updates about upcoming courses and templates.

## What to Expect

I'll be writing about:

- **Python fundamentals** — clear explanations of core concepts
- **Web development** — Django, Flask, and modern Python web frameworks
- **Project updates** — new courses and templates as they launch

## A Quick Code Sample

Here's a taste of what tutorial posts will look like:

```python
def greet(name: str) -> str:
    """Return a friendly greeting."""
    return f"Hello, {name}! Welcome to AcePython."

if __name__ == "__main__":
    print(greet("World"))
```

Stay tuned for more posts, and [sign up for the newsletter](/) to get notified when new content drops.
```

**Step 2: Build the full site**

Run: `cd /Users/phildini/code/acepython/acepython.com && uv run pelican content -o output -s pelicanconf.py`
Expected: Builds without errors

**Step 3: Verify output files exist**

Run: `ls output/blog/ && ls output/blog/hello-world/ && ls output/blog/tags/ && ls output/blog/category/`
Expected: `index.html` in each directory

**Step 4: Start dev server and visually verify**

Run: `cd /Users/phildini/code/acepython/acepython.com && uv run pelican -l content -o output -s pelicanconf.py`

Check in browser:
- `http://localhost:8000/` — homepage with navbar, hero section unchanged
- `http://localhost:8000/blog/` — blog listing showing the hello world post
- `http://localhost:8000/blog/hello-world/` — full article with syntax-highlighted code
- `http://localhost:8000/blog/tags/` — tags listing page
- `http://localhost:8000/blog/tags/python/` — posts tagged "python"
- `http://localhost:8000/blog/category/announcements/` — posts in "announcements" category

**Step 5: Commit**

```bash
git add content/hello-world.md
git commit -m "feat: add sample blog post for testing"
```

---

### Task 8: Final verification and cleanup

**Step 1: Build with production config**

Run: `cd /Users/phildini/code/acepython/acepython.com && uv run pelican content -o output -s publishconf.py`
Expected: Builds without errors, feed file exists at `output/feeds/all.atom.xml`

**Step 2: Verify feed**

Run: `head -5 output/feeds/all.atom.xml`
Expected: Valid XML with Atom feed content

**Step 3: Final commit if any cleanup was needed**

```bash
git add -A
git commit -m "chore: final blog setup cleanup"
```
