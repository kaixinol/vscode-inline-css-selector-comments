# Changelog

## 0.1.0

First release.

- Highlight `/* ... */` comments inside CSS selector string literals in JS/TS.
- Marker comment goes directly in front of the string: `/* css selector */` or `/* css-selector */`.
- Supports double-quoted, single-quoted and template strings.
- Works in JavaScript / TypeScript (including JSX and TSX) and the `<script>` block of
  Vue / Svelte / Astro files.
- The selector itself keeps the theme's string color; only the inner comments are recolored.
