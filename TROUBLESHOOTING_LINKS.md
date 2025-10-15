# Troubleshooting: Markdown Link Rendering Issue

## Problem

When rendering markdown content, particularly with external links, a `TypeError: href.startsWith is not a function` was encountered in the `MarkdownRenderer.tsx` component.

This error occurred within the custom `marked.Renderer().link` function, specifically at the line attempting to call `href.startsWith('http')`.

## Cause

The `marked` library's `link` renderer function, under certain circumstances (e.g., when `href` might be `undefined` or `null` for malformed markdown links), was providing a non-string value for the `href` parameter.

Since `startsWith` is a string method, calling it on a non-string value resulted in the `TypeError`.

## Solution

To prevent this `TypeError`, a type check was added to ensure that `href` is indeed a string before attempting to call `startsWith` on it.

**File:** `src/components/MarkdownRenderer.tsx`

**Change:**
```typescript
// Before
renderer.link = (href, title, text) => {
  const html = linkRenderer.call(renderer, href, title, text);
  if (href.startsWith('http')) { // Error occurred here
    return html.replace(/^<a /, '<a target="_blank" rel="noopener noreferrer" class="markdown-link" ');
  }
  return html.replace(/^<a /, '<a class="markdown-link" ');
};

// After
renderer.link = (href, title, text) => {
  const html = linkRenderer.call(renderer, href, title, text);
  if (typeof href === 'string' && href.startsWith('http')) {
    return html.replace(/^<a /, '<a target="_blank" rel="noopener noreferrer" class="markdown-link" ');
  }
  return html.replace(/^<a /, '<a class="markdown-link" ');
};
```

By adding `typeof href === 'string'`, the code now safely checks the type of `href` before attempting to use string-specific methods, thus resolving the `TypeError`.
