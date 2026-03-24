# LinkedIn Scanning Guide

**URL**: `https://www.linkedin.com/feed/`

Requires login. Look for "Start a post" box and feed content.

## Scanning

1. Navigate to the feed
2. Scroll through 1-2 times (more scrolling has diminishing returns)
3. Use JavaScript execution to extract posts — `read_page` alone misses the key `data-urn` attributes
4. Note the "LinkedIn News" section in the right sidebar for trending professional topics
5. Note the user's profile info (job title, company) visible in the left sidebar — useful for bootstrap

## Expand truncated posts

LinkedIn truncates most posts. You MUST expand before extracting:
```javascript
document.querySelectorAll('.feed-shared-inline-show-more-text button, [class*="see-more"]').forEach(b => b.click());
```
Wait 1 second after expanding, then extract.

## Getting the direct URL — the `data-urn` method

LinkedIn post containers have a `data-urn` attribute containing `urn:li:activity:{id}`. This is on a `<div>`, NOT on any `<a>` link. Permalink = `https://www.linkedin.com/feed/update/{data-urn}/`

Use `javascript_tool` to extract:
```javascript
document.querySelectorAll('.feed-shared-inline-show-more-text button, [class*="see-more"]').forEach(b => b.click());
setTimeout(() => {}, 1000);
const posts = document.querySelectorAll('[data-urn^="urn:li:activity"]');
const results = [];
posts.forEach(el => {
  const urn = el.getAttribute('data-urn');
  const permalink = 'https://www.linkedin.com/feed/update/' + urn + '/';
  const authorEl = el.querySelector('.update-components-actor__title span[aria-hidden="true"]')
    || el.querySelector('[class*="actor"] span[aria-hidden="true"]');
  const author = authorEl ? authorEl.textContent.trim() : '';
  const textEl = el.querySelector('.update-components-text span[dir="ltr"]')
    || el.querySelector('[class*="update-components-text"]');
  const text = textEl ? textEl.textContent.trim() : '';
  const timeEl = el.querySelector('[class*="actor__sub-description"] span[aria-hidden="true"]');
  const time = timeEl ? timeEl.textContent.trim() : '';
  const reactEl = el.querySelector('[class*="social-details-social-counts"]');
  const reactions = reactEl ? reactEl.textContent.trim() : '';
  results.push({ author, permalink, text, time, reactions });
});
JSON.stringify(results);
```

**IMPORTANT**: Do NOT look for `/feed/update/` links in `read_page` output — they don't exist in LinkedIn's DOM.

## Realistic expectations

5-10 items with valid permalinks per scroll session.