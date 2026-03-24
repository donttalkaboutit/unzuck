# Instagram Scanning Guide

**URL**: `https://www.instagram.com`

Requires login. Instagram is **opt-in** (default maxItems: 0) because it's primarily visual with limited text content. Only scan if the user has set maxItems > 0 in their profile or explicitly requests it.

## CRITICAL — Slow Loading

Instagram takes 3-4 seconds to load the feed. Wait for the stories bar to appear at the top before extracting.

## Scanning

1. Navigate to the home feed
2. **Wait 3-4 seconds** for the feed to load
3. Take a screenshot to confirm feed is visible
4. Use `javascript_tool` to extract posts from `<article>` elements (see script below)
5. Scroll 2-3 times and extract again
6. Focus on posts with captions longer than ~30 characters

## Content extraction using `javascript_tool`

```javascript
const articles = document.querySelectorAll('article');
const results = [];
articles.forEach(art => {
  const allLinks = art.querySelectorAll('a[href]');
  let username = '';
  allLinks.forEach(l => {
    const href = l.getAttribute('href');
    if (!username && href && href.match(/^\/[a-zA-Z0-9_.]+\/$/) &&
        !href.includes('/p/') && !href.includes('/reel/')) {
      username = l.textContent.trim() || href.replace(/\//g, '');
    }
  });
  const pLink = art.querySelector('a[href*="/p/"], a[href*="/reel/"]');
  const permalink = pLink ? 'https://www.instagram.com' + pLink.getAttribute('href') : '';
  const time = pLink ? pLink.textContent.trim() : '';
  const postIdMatch = permalink.match(/\/(p|reel)\/([^/]+)\//);
  const postId = postIdMatch ? postIdMatch[2] : '';
  let caption = '';
  art.querySelectorAll('span').forEach(s => {
    const t = s.textContent.trim();
    if (t.length > 20 && t.length < 2000 && !t.includes('like') && !t.includes('comment')) {
      if (t.length > caption.length) caption = t;
    }
  });
  const likeBtn = art.querySelector('a[href*="liked_by"]');
  const likes = likeBtn ? likeBtn.textContent.trim() : '';
  if (permalink && postId) {
    results.push({ username, permalink, time, caption, likes, postId });
  }
});
JSON.stringify(results);
```

## Expanding captions

```javascript
document.querySelectorAll('[role="button"]').forEach(b => { if(b.textContent.includes('more')) b.click(); });
```
Wait 1 second, then re-extract.

## Media — Embed iframe is PRIMARY

Do NOT use Instagram CDN image URLs — they expire. Always capture the `postId`. The dashboard uses embed iframes (`https://www.instagram.com/p/{postId}/embed/`) which render the full post.

## Getting the direct URL

Permalink links with `/p/{id}/` or `/reel/{id}/` are in the DOM on timestamp links within `<article>`. Prepend `https://www.instagram.com`. Never use `instagram.com/{username}/` as a post URL.

## DOM changes

If `article` selectors stop working, fall back to `document.querySelectorAll('a[href*="/p/"]')` and work outward.

## Realistic expectations

3-5 items with valid permalinks.