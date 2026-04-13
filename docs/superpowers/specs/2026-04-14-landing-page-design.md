# AcePython Landing Page Design

## Overview

A single-page landing site for acepython.org built with Pelican, DaisyUI, HTMX, and Buttondown. The page serves as a pre-launch email signup for Philip James's upcoming Python learning courses and code templates.

## Tone

Friendly and approachable first, backed by professional credibility. Not minimal or mysterious — Philip introduces himself directly and warmly.

## Page Structure

Three sections stacked vertically on a single viewport:

### 1. Hero Section
- Philip's name and the AcePython brand
- A warm one-liner about what's coming: Python courses and code templates
- The email signup form (Buttondown) as the primary CTA, front and center
- This is above the fold — the signup form should not require scrolling

### 2. Credibility Strip
- Brief section establishing Philip's authority
- Link to pyvideo speaking history (https://pyvideo.org/speaker/philip-james.html)
- Short statement about speaking background (PyCon, DjangoCon, Python meetups)

### 3. Footer
- Minimal: copyright line
- Optional social/GitHub links

## Technical Architecture

### Theme Structure

```
themes/acepython/
  templates/
    base.html        # HTML shell: <head> with CDN links, <body> wrapper
    index.html       # Extends base, contains the landing page layout
  static/            # Empty for now, future custom CSS/JS
```

### CDN Dependencies (no build step)

Loaded in `base.html` `<head>`:
- **Tailwind CSS + DaisyUI** — DaisyUI ships as a Tailwind plugin, single CDN link
- **HTMX** — ~14kb script for form submission without page reload

### DaisyUI Theming

- Use the `emerald` built-in theme (`data-theme="emerald"` on `<html>`)
- Clean greens, friendly typography — fits the approachable-yet-professional tone
- Easy to swap themes later by changing the attribute value

### Buttondown Email Signup + HTMX

- `<form>` with single email input and submit button
- `hx-post` pointing to Buttondown's embed subscribe endpoint: `https://buttondown.com/api/emails/embed-subscribe/<newsletter-id>`
- `hx-swap="outerHTML"` to replace the form with a success message on submit
- No API key needed on the client side — Buttondown's embed endpoint accepts form POSTs directly
- No custom JavaScript required

### Pelican Configuration Changes (`pelicanconf.py`)

- Add `THEME = "themes/acepython"` to point at the custom theme
- The landing page lives entirely in the theme's `index.html` template
- `content/` directory stays empty for now
- Future course pages and blog posts will use Pelican's content system when ready

## What This Design Does NOT Include

- No blog functionality (future)
- No course listing pages (future)
- No custom CSS build pipeline — CDN only
- No analytics or tracking
- No social media meta tags (can add later)

## Success Criteria

- Visiting acepython.com shows the landing page with a working email signup
- Submitting an email adds the subscriber to the Buttondown newsletter
- The form provides feedback on successful signup without a page reload
- The page looks clean and professional on both desktop and mobile
