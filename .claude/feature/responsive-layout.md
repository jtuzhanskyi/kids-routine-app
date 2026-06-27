# Feature: Responsive Layout

**Status: Implemented**

## Problem

The app was built around a fixed `width:1180px; height:812px` "device screen" container with a decorative bezel frame. On any screen narrower than ~1280px (tablets, smaller laptops), this caused horizontal overflow and an unreadable layout.

## Layout layers (outermost → innermost)

| Layer | Desktop style | Responsive override |
|---|---|---|
| Page wrapper `.krb-page` | `padding:32px; flex center` | `padding:0; align-items:flex-start` |
| Device bezel `.krb-bezel` | `padding:16px; border-radius:46px; shadow` | Hidden (transparent, no padding/shadow) |
| Screen `.krb-screen` | `width:1180px; height:812px` | `width:100vw; height:100dvh` |
| Kids row `.krb-kids` | `gap:24px` | `gap:12px` (8px on mobile) |

## Implementation

Since all layout styles are inline JavaScript objects (not CSS classes), the approach was:

1. **Added CSS class names** (`krb-page`, `krb-bezel`, `krb-screen`, `krb-kids`) to the four structural divs in the `__bundler/template` HTML string.
2. **Added `@media` query overrides** with `!important` to the app `<style>` block inside the template, targeting those classes.

Breakpoints:
- `≤ 1280px` — tablet / small laptop: full-viewport layout, decorative bezel removed
- `≤ 600px` — mobile: tighter padding

## Decision: portrait tablet layout

Two kid panels remain **side-by-side** at all screen sizes (user preference). The internal task lists already have `overflow:auto; flex:1; min-height:0` so they scroll within the fixed-height columns without affecting anything.

## Editing note

When re-encoding the template JSON, Python's `json.dumps` does not escape `/` to `\/` by default. A `.replace('</', '<\\/')` call is required after `json.dumps` to prevent `</script>` inside the template from being misinterpreted by the HTML parser as closing the outer `<script>` tag.
